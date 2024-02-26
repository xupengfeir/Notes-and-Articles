# malloc-free底层原理-动态内存管理

`本文参考自文章 https://jacktang816.github.io/post/mallocandfree/，原文作者：JackTang816`

内存管理一般可以从两个方面讲解：  
* 虚拟内存机制：物理和虚拟地址空间、TLB页表、内存映射  
* 动态内存管理：内存管理、分配方式、内存回收、GC等

下面着重讲解动态内存管理这部分，从malloc和free的底层原理出发。  

## malloc和free
`malloc`和`free`是C语言中用于动态内存分配和释放内存的两个函数。它们是C语言标准库的一部分，**用于在程序运行期间请求和释放堆内存。**  

### 进程地址空间
![20240226094648](https://gcore.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240226094648.png)

由于虚拟内存的存在，每个进程就像独占整个地址空间一样。  

如上图所示，在一个32位系统中，可寻址的空间大小是4GB，linux系统下0~3GB是用户模式，3~4GB是内核模式。  

在用户模式下，可分为代码段、数据段(.data)、.bss段、堆、栈。  
代码段主要存放进程的可执行二进制代码，字符串字面值和只读变量；  
数据段存放已经初始化且初始值非0的全局变量和局部静态变量；
bss段存放未初始化或初始值为0的全局变量和局部静态变量；  
堆段存放由用户动态分配内存存储的变量；  
栈段主要存储局部变量、函数参数、返回地址等；

### 内存映射 mmap
* 内存映射段（mmap）的作用：内核将硬盘文件的内容直接映射到内存，任何应用程序都可以通过linux的mmap()系统调用请求这种映射。  
* 内存映射是一种方便高效的文件I/O方式，因而被用于装载动态共享库。  
* 用户也可创建匿名内存映射，该映射没有对应的文件，可用于存放程序数据。  
```
在linux中，若通过malloc()请求一大块内存，C运行库将创建一匿名内存映射，而不是用堆内存。“大块”意味着比阈值MMAP_THRESHOLD还大，缺省值为128KB，可通过mallopt()调整。
```
* mmap()映射区向下扩展，堆向上扩展，两者相对扩展，直到耗尽虚拟地址空间中的剩余区域。

在linux中进程由进程控制块（PCB）描述，用一个task_struct数据结构表示，这个数据结构记录了所有进程信息，包括进程状态、进程调度信息标识符、进程通信相关信息、进程连接信息、时间和定时器、文件系统信息、虚拟内存信息等，和malloc密切相关的就是虚拟内存信息，定义为`struct mm_struct *mm`描述进程的地址空间。  

mm_struct 结构是对整个用户空间（进程空间）的描述。
```
//include/linux/sched.h 

struct mm_struct {
  struct vm_area_struct * mmap;  /* 指向虚拟区间（VMA）链表 */
  rb_root_t mm_rb;         /*指向red_black树*/
  struct vm_area_struct * mmap_cache;     /* 指向最近找到的虚拟区间*/
  pgd_t * pgd;             /*指向进程的页目录*/
  atomic_t mm_users;                   /* 用户空间中的有多少用户*/                                     
  atomic_t mm_count;               /* 对"struct mm_struct"有多少引用*/                                     
  int map_count;                        /* 虚拟区间的个数*/
  struct rw_semaphore mmap_sem;
  spinlock_t page_table_lock;        /* 保护任务页表和 mm->rss */       
  struct list_head mmlist;            /*所有活动（active）mm的链表 */
  unsigned long start_code, end_code, start_data, end_data; /* 代码段、数据段 起始地址和结束地址 */
  unsigned long start_brk, brk, start_stack; /* 栈区 的起始地址，堆区 起始地址和结束地址 */
  unsigned long arg_start, arg_end, env_start, env_end; /*命令行参数 和 环境变量的 起始地址和结束地址*/
  unsigned long rss, total_vm, locked_vm;
  unsigned long def_flags;
  unsigned long cpu_vm_mask;
  unsigned long swap_address;

  unsigned dumpable:1;
  /* Architecture-specific MM context */
  mm_context_t context;
};
```  
其中`start_brk`和`brk`分别是堆的起始地址和终止地址，我们使用malloc动态分配的内存就在这之间。

`start_stack`是进程栈的其实地址，**栈大小在编译器确定的** ，在运行时不能改变。

而堆的大小由`start_brk`和`brk`决定，但是可以使用系统调用`sbrk()`和`brk()`增加`brk`的值，达到增大对空间的效果，但是系统调用代价太大，涉及到用户态和内核态的相互转换。  

所以，**实际中系统分配较大的堆空间，进程通过`malloc`库函数在对上进行空间动态分配，堆如果不够用，`malloc`可以进行系统调用，增大`brk`的值。**  

```malloc只知道stark_brk和brk之间连续可用的内存空间可以任意分配，如果不够用了就向系统申请增大brk。```

### 相关系统调用 
**(1) brk()和sbrk()**

由前文对进程地址空间结构的分析可知，要增加一个进程实际的可用堆大小，就需要将`break`指针向高地址移动。

linux通过`brk`和`sbrk`系统调用操作break指针。两个系统调用的原型如下：
```
#include <unistd.h>
int brk(void *addr);
void *sbrk(intptr_t increment);
```
`brk`函数将`break`指针直接设置为某个地址，而`sbrk`将`break`指针从当前位置移动`increment`所指定的增量。  

`brk`在执行成功时返回0，否则返回-1并设置`errno`为`ENOMEM`;`sbrk`成功时返回break指针移动之前所指向的地址，否则返回(void *)-1

```
如果将increment设置为0，则可以获得当前break的地址。  
另外需要注意的是，由于linux是按页进行内存映射的，所以如果break被设置为没有按页大小对齐时，则系统实际上会在最后映射一个完整的页，从而实际已映射的内存空间比break指向的地方要大一些。
但是使用break之后的地址是很危险的（尽管break之后确实有一小块可用内存地址）。
```

进程所面对的虚拟内存地址空间，只有按页映射到物理内存地址，才能真正使用。  
受物理存储容量限制，整个堆虚拟内存空间不可能全部映射到实际的物理内存。因此每个进程有一个rlimit表示当前进程可用资源上限。这个限制可以通过getrlimit系统调用得到。  
![20240226144545](https://gcore.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240226144545.png)  
其中rlimit是一个结构体：  
```
struct rlimit {
  rlim_t rlim_cur;  /* Soft limit */
  rlim_t rlim_max;  /* Hard limit (ceiling for rlim_cur) */
};
```
每种资源有软限制和硬限制，并且可以通过`setrlimit`对`rlimit`进行有条件设置。其中硬限制作为软限制的上限，非特权进程只能设置软限制，且不能超过硬限制。

**（2）mmap()函数**  
```
#include <sys/mman.h>
void *mmap(void *addr, size\_t length, int prot, int flags, int fd, off\_t offset);
int munmap(void *addr, size_t length);
```
mmap函数函数的第一种用法是映射磁盘文件到内存中；而malloc使用的mmap函数的第二种用法，即匿名映射，匿名映射不映射磁盘文件，而是向映射区申请一块内存。  

munmap函数是用于释放内存，第一个参数为内存首地址，第二个参数为内存的长度。

**当申请小内存时，malloc使用sbrk分配内存；当申请大内存时，malloc使用mmap函数申请内存**；

但是这只是分配了虚拟内存，还没有映射到物理内存，当访问申请的内存时，才会因为缺页异常，内核分配物理内存。

![20240226150312](https://gcore.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240226150312.png)  
* 分配内存 < DEFAULT_MMAP_THRESHOLD, 走__brk,从内存池获取，失败的话走brk系统调用
* 分配内存 < DEFAULT_MMAP_THRESHOLD, 走__mmap,直接调用mmap系统调用  
* 
**其中，DEFAULT_MMAP_THRESHOLD默认为128k，可通过mallopt进行设置。**

## malloc实现方案
由于`brk/sbrk/mmap`属于系统调用，如果每次申请内存，都调用这三个函数中的一个，那么每次都要产生调用开销（即CPU从用户态切换到内核态的上下文切换，这里要保存用户态数据，等会还要切换回用户态），非常影响性能。

鉴于此，**malloc采用的是内存池的实现方式**，malloc内存池实现方式更类似于STL分配器和memcached的内存池，先申请一大块内存，然后将内存分成不同大小的内存块，然后用户申请内存时，直接从内存池中选择一块相近的内存块即可。

![20240226151709](https://gcore.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240226151709.png)  
内存池保存在`bins`这个长度128的数组中，每个元素都是一个双向链表。  
* bins[0]目前没有使用
* bins[1]的链表称为unsorted_list,用于维护free释放的chunk
* bins[2,63]的区间称为small_bins,用于维护<512字节的内存块,其中每个元素对应的链表中的chunk大小相同，均为index*8
* bins[64,127]称为large_bins,用于维护>=512字节的内存块，每个元素对应的链表中的chunk大小不同，index越大，链表中chunk的内存相差越大。

**例如: 下标为64的chunk大小介于[512, 512+64)，下标为95的chunk大小介于[2k+1,2k+512)。同一条链表上的chunk，按照从小到大的顺序排列。**  

malloc将内存分成了大小不同的chunk，然后通过bins来组织起来。

malloc将相似大小的chunk（图中可以看出同一链表上的chunk大小差不多）用双向链表链接起来，这样一个链表被称为一个bin。malloc一共维护了128个bin，并使用一个数组来存储这些bin。

数组中第一个为unsorted bin，数组编号前2到前64的bin为small bins，同一个small bin中的chunk具有相同的大小，两个相邻的small bin中的chunk大小相差8bytes。

small bins后面的bin被称作large bins。large bins中的每一个bin分别包含了一个给定范围内的chunk，其中的chunk按大小序排列。

malloc除了有unsorted bin，small bin，large bin三个bin之外，还有一个**fast bin** 。

一般情况下，程序在运行时会经常需要申请和释放一些较小的内存空间。当分配器合并了相邻的几个小的chunk之后，也许立即会有另一个小块内存的请求，这样分别分配器又需要从大的空间内存中切分出一块，这样是低效的，因此，malloc在分配过程中引入了fast bins，不大于max_fast(默认值为64B)的chunk被释放后，首先会被放到fast bins中，fast bins并不改变它的使用标志P。这样就无法将它们合并，当需要给用户分配的chunk小于或等于max_fast时，malloc首先会在fast bins中查找相应的空闲块，然后才会去查找bins中空闲chunk。在某个特定的时候，malloc会遍历fast bins中的chunk，将相邻的空闲chunk进行合并，并将合并后的chunk加入unsorted bin中，然后再将unsorted bin里的chunk加入bins中。

unsorted bin的队列使用bins数组的第一个，如果被用户释放的chunk或者fast bins中的空闲chunk合并后大于max_fast，这些chunk首先会被放到unsorted bin中。在进行 malloc 操作的时候，如果在 fast bins 中没有找到合适的 chunk，则malloc 会先在 unsorted bin 中查找合适的空闲 chunk，然后才查找 bins。如果 unsorted bin 不能满足分配要求。 malloc便会将 unsorted bin 中的 chunk 加入 bins 中。然后再从 bins 中继续进行查找和分配过程。  

从这个过程可以看出来，unsorted bin可以看作是bins的一个缓冲区，增加它只是为了加快分配速度。（利用了程序的局部性原理，常用的内存块大小差不多，类似于TLB的作用）

malloc的三种内存区：
* 当fast bin和bins都不能满足内存需求时，malloc会设法在top chunk中分配一块内存给用户；top chunk为在mmap区域分配一块较大的空闲内存模拟sub-heap。（比较大的时候） >top chunk是堆顶的chunk，堆顶指针brk位于top chunk的顶部。移动brk指针，即可扩充top chunk的大小。当top chunk大小超过128k(可配置)时，会触发malloc_trim操作，调用sbrk(-size)将内存归还操作系统。
* 当chunk足够大，fast bin和bins都不能满足要求，甚至top chunk都不能满足时，malloc会从mmap来直接使用内存映射来将页映射到进程空间，这样的chunk释放时，直接解除映射，归还给操作系统。（极限大的时候）
* Last remainder是另外一种特殊的chunk，就像top chunk和mmaped chunk一样，不会在任何bins中找到这种chunk。当需要分配一个small chunk,但在small bins中找不到合适的chunk，如果last remainder chunk的大小大于所需要的small chunk大小，last remainder chunk被分裂成两个chunk，其中一个chunk返回给用户，另一个chunk变成新的last remainder chunk。（这个应该是fast bins中也找不到合适的时候，用于极限小的）

由之前的分析可知malloc利用chunk结构来管理内存块，malloc就是由不同大小的chunk链表组成。malloc会给  用户分配的空间的前后加上一些控制信息，用这样的方法来记录分配的信息，以便完成分配和释放工作。chunk指针指向chunk开始的地方，图中的mem指针才是真正返回给用户的内存指针。  
![20240226161541](https://gcore.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240226161541.png)

* chunk 的第二个域的最低一位为P，它表示前一个块是否在使用中，P 为 0 则表示前一个 chunk 为空闲，这时chunk的第一个域 prev_size 才有效，prev_size 表示前一个 chunk 的 size，程序可以使用这个值来找到前一个 chunk 的开始地址。当 P 为 1 时，表示前一个 chunk 正在使用中，prev_size程序也就不可以得到前一个 chunk 的大小。不能对前一个 chunk 进行任何操作。malloc分配的第一个块总是将 P 设为 1，以防止程序引用到不存在的区域。
* Chunk 的第二个域的倒数第二个位为M，他表示当前 chunk 是从哪个内存区域获得的虚拟内存。M 为 1 表示该 chunk 是从 mmap 映射区域分配的，否则是从 heap 区域分配的。
* Chunk 的第二个域倒数第三个位为 A，表示该 chunk 属于主分配区或者非主分配区，如果属于非主分配区，将该位置为 1，否则置为 0。

当chunk空闲时，其M状态是不存在的，只有AP状态，原本是用户数据区的地方存储了四个指针，指针fd指向后一个空闲的chunk，而bk指向前一个空闲的chunk，malloc通过这两个指针将大小相近的chunk连成一个双向链表。在large bin中的空闲chunk，还有两个指针，fd_nextsize和bk_nextsize，用于加快在large bin中查找最近匹配的空闲chunk。不同的chunk链表又是通过bins或者fastbins来组织的。

## malloc内存分配流程 
1、如果分配内存 < 512字节，则通过内存大小定位到small bins对应的index上。（index=size/8）
* 如果 **smallbins[index]** 为空，进入步骤3；
* 如果 **smallbins[index]** 非空，直接返回第一个chunk；  
  
2、如果分配内存 > 512字节，则定为到large bins对应的index上。
* 如果 **largebins[index]** 为空，进入步骤3；
* 如果 **largebins[index]** 非空，扫描链表，找到第一个合适大小的chunk，如size=12.5K，则使用13K大小的chunk B，剩余0.5K放入 unsorted_list中；

3、遍历unsorted_list,查找合适大小的chunk，如果找到则返回；否则，将这些chunk归类到small bins和large bins中。

4、index++从更大的链表中查找，直到找到合适大小的chunk为止，找到后将chunk拆分，并将剩余的加入到unsorted_list中。

5、如果还没找到，使用 top chunk。

6、或者， 内存 < 128K,使用brk；内存 > 128K，使用mmap获取新内存；

此外，调用free函数时，它将用户释放的内存块连接到空闲链上。到最后，空闲链会被切成很多的小内存片段，如果这时用户申请一个大的内存片段，那么空闲链上可能没有可以满足用户要求的片段了。于是，malloc函数请求延时，并开始在空闲链上翻箱倒柜地检查各内存片段，对它们进行整理，将相邻的小空闲块合并成较大的内存块。

* 虚拟内存并不是每次malloc后都增长，是与堆顶有没有发生变化有关，因为可重用堆顶内剩余的空间，这样的malloc可以很快速；
* 如果虚拟内存发生变化，基本与分配内存量相当，因为虚拟内存是计算虚拟地址空间总大小；
* 物理内存的增量很少，是因为malloc分配的内存并不是马上分配实际储存空间，只有第一次使用才会分配；
* 由于每个物理内存页面大小是4K，不管memset其中的1k还是5k、7k，实际占用物理内存总是4K的倍数。所以物理内存的增量总是4K的倍数；

**因此，不是malloc后就马上占用实际内存，而是第一次使用时发现虚拟内存对应的物理页面未分配，产生缺页中断，才真正分配物理页面，同时更新进程页面的映射关系。这是linux虚拟内存管理的核心概念之一。**

## 内存碎片
free释放内存时，有两种情况：  
* chunk和top chunk相邻，则和 top chunk合并
* chunk和top chunk不相邻，则直接插入到unsorted_list  
  ![20240226165302](https://gcore.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240226165302.png)  

如上图示: top chunk是堆顶的chunk，堆顶指针brk位于top chunk的顶部。移动brk指针，即可扩充top chunk的大小。**当top chunk大小超过128k(可配置)时，会触发malloc_trim操作，调用sbrk(-size)将内存归还操作系统。**

考虑如下场景（假设brk起始地址是512KB）：  
```
malloc 40k内存，即chunkA，brk = 512k + 40k = 552k；
malloc 50k内存，即chunkB，brk = 552k + 50k = 602k;
malloc 60k内存，即chunkD，brk = 602k + 60k = 662k;
free chunkA.
此时chunk D是Top chunk
```
此时，由于brk=662k，而释放的内存地址是位于[512k,552k]，无法通过移动brk指针将区域内内存交还操作系统，此时在地址空间[512k,552k]形成了一个内存空洞即内存碎片。

按照glibc的策略，free后的chunk A区域由于不和top chunk相邻，因此无法和top chunk合并，应该挂在unsorted_list链表上。
