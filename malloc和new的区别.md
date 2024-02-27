# malloc、new、free、delete区别
## malloc和new使用方法
```
#include <cstdlib>

int main() {
    // 使用 malloc 分配内存
    int *ptr = (int *)malloc(sizeof(int));
    // 记得在使用完之后释放内存
    free(ptr);
}
```
```
int main() {
    // 使用 new 分配内存并调用构造函数
    int *ptr = new int;
    // 记得在使用完之后释放内存
    delete ptr;
}
```

## malloc和free不同点
**1、申请内存所在位置**  

new操作符从自由存储区（free store）上为对象动态分配内存空间，而malloc函数从堆上动态分配内存。

自由存储区是C++基于new操作符的一个**抽象概念**，凡是通过new操作符进行内存申请，该内存即为自由存储区。

堆是操作系统中的术语，是操作系统所维护的一块特殊内存，用于程序的内存动态分配，C语言使用malloc从堆上分配内存，使用free释放已分配的对应内存。

那么自由存储区是否能够是堆（等价于new是否能在堆上动态分配内存），这取决于operator new的实现细节。

自由存储区不仅可以使堆，还可以是静态存储区，这都看operator new在哪里为对象分配内存。（不过一般情况可以将自由存储区等价与堆区）。

特别的，new甚至可以部位对象分配内存。`placement_new`的功能可以办到：
```
new (place_address) type
```
`place_address`为一个指针，代表一块内存的地址。当使用上面这种仅以一个地址调用new操作符时，new操作符调用特殊的operator new。
```
void * operator new (size_t,void *) //不允许重定义这个版本的operator new
```
这个operator new 不分配任何的内存，它只是简单地返回指针实参，然后new表达式负责在`place_address`指定的地址进行对象的初始化工作。

**注：堆是操纵系统维护的一块内存，而自由存储区是C++中通过new/delete动态分配和释放对象的抽象概念，（自由存储区可以在堆或者全局区或其他可读区域中实现）。堆与自由存储区并不等价。**

**2、返回类型安全性**  

new操作符内存分配成功时，返回的是对象类型的指针，类型严格与对象匹配，无须进行另外类型转换，故new是符合**类型安全性**的操作符。malloc内存分配成功则是返回void*，需要通过强制类型转换将void*指针转换成我们需要的类型。

类型安全很大程度上可以等价于内存安全，类型安全的代码不会试图访问自己未授权的内存区域。

**3、内存分配失败时的返回值**  
new内存分配失败时，会抛出`bac_alloc`异常，**不会返回NULL**；malloc分配内存失败时返回NULL。  

使用`malloc`分配内存后会判断分配是否成功：  
```
int *a  = (int *)malloc ( sizeof (int ));
if(NULL == a)
{
    ...
}
else 
{
    ...
}
```
使用`free`时应该使用**异常机制**：
```
try
{
    int *a = new int();
} 
catch (bad_alloc)
{
    ...
}
```

**4、是否需要指定内存大小**  
使用new操作符申请内存分配时无须指定内存块的大小，编译器会根据类型自行计算，而malloc则需要显式地指出所需内存的大小。
```
class A{...};
A *ptr = new A;
A *ptr = (A*)malloc(sizeof(A)); // 需要显式指定内存大小
```

**5、是否调用构造函数/析构函数**  

使用`new`操作符来分配对象内存时会经历三个步骤：
* 第一步：调用`operator new`函数(对于数组是 `operator new[]`)分配一块足够大的、原始的、未命名的内存空间以便存储特定类型的对象。
* 第二步：编译器运行相应的**构造函数**以构造对象，并为其传入初值。
* 第三步：对象构造完成后，返回一个指向该对象的指针；
  
使用`delete`操作符来释放对象内存时有两个步骤：
* 第一步：调用对象的析构函数；
* 第二步：编译器调用 `operator delete（operator delete[]）` 函数释放内存空间；

`new/delete`会调用构造/析构函数以完成对象的构造/析构。而malloc不会。

**6、对数组的处理**  
```
A *ptr = new A[10];
delete [] ptr;
```
`delete[]`和`new[]`要配套使用，不然会造成数组对象部分释放的现象，造成内存泄漏；

而`malloc`申请一块原始的内存，再返回内存的地址。
```
int *ptr = (int *)malloc(sizeof(int) * 10);
```

**7、new与malloc是否可以相互调用**  

operator new / operator delete的实现可以基于malloc，而malloc的实现不可以调用new。
```
void * operator new (size_t size)
{
    if (void *mem = malloc(size))
        return mem;
    else
        throw bad_alloc();
}
void operator delete (void *mem) noexcept
{
    free(mem);
}
```

**8、是否可以被重载**  
`operator new/operator delete `可以被重载。标准库定义了 `operator new`函数和`operator delete`函数的8个重载版本；

```
//这些版本可能抛出异常
void * operator new(size_t);
void * operator new[](size_t);
void * operator delete (void * )noexcept;
void * operator delete[](void *0）noexcept;
//这些版本承诺不抛出异常
void * operator new(size_t ,nothrow_t&) noexcept;
void * operator new[](size_t, nothrow_t& );
void * operator delete (void *,nothrow_t& )noexcept;
void * operator delete[](void *0,nothrow_t& ）noexcept;
```
我们可以重定义上面函数版本中的任意一个，前提是自定义版本必须位于全局作用域或者类作用域。

**9、能够直观地重新分配内存**

使用`malloc`分配内存后，如果在使用过程中发现内存不足，可以使用`realloc`函数重新进行内存重新分配实现内存的扩充。

`realloc` 先判断当前的指针所指向的内存是否有连续空间，如果有，原地扩大可分配的内存地址，并且返回原来的地址指针；如果空间不够，先按照新指定的大小分配空间，将原有数据全部拷贝到新分配的内存区域，而后释放原来的内存区域。
