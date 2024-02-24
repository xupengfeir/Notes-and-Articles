# stm32启动文件详解

在stm32程序执行main函数之间，需要执行一段汇编程序和完成C语言资源硬件的初始化工作。下面以**startup_stm32f429_439xx.s** 启动文件为例，讲解stm32的启动流程。   

启动文件工作流程主要分为以下几个步骤：  

【1】初始化栈指针 **MSP=_initial_sp**   
【2】初始化复位程序计数寄存器值 **=Reset_Handler**   
【3】初始化异常/中断向量表  
【4】系统时钟配置  
【5】C库函数 **_main** 初始化用户堆栈的调用   

**文件启动步骤**  
1、在启动的时候，先对堆栈的大小定义，并在代码区的起始位置建立异常中断向量表。然后在复位中断服务程序中跳转执行C标准库_main函数，以上这些完成后，跳转到主程序中的main函数执行相关函数应用。  
2、假设单片机被设置成从内部flash启动，这时候片内flash被映射到程序启动空间， **异常/中断向量表实际的开始地址为0x8000000,复位中断存放在0x8000004，** 若STM32F4遇到复位信号，**则从0x8000004处取出复位中断服务函数的入口地址**，继而执行中断服务函数，随后跳转到_main函数，最终进入main函数。  
![v2-e13be5844f31ed6aa70ee5e64b6fe198_r](https://gcore.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/v2-e13be5844f31ed6aa70ee5e64b6fe198_r.jpg)

**详解**  
**1、栈（Stack）  **  
在startup_stm32f429_439xx.s文件中，默认将栈大小设置成0x00000400（1KB），stm32F4xx芯片内部有192KB SRAM  ，因此栈最大可设置为192KB大小。   

Stack_Mem为栈名，不初始化可读可写，8字节对齐。Stack_Size是栈的大小，__initial_sp表示结束地址（栈顶地址，栈是由高字节向低字节生长的）。

栈的主要作用是用于局部变量、函数调用、函数形参的开销，大小应小于内部RAM大小，考虑到局部变量的需求，防止栈溢出。
![20240224161032](https://gcore.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240224161032.png)

**2、堆（Heap）**  
在栈的代码后面便是初始化堆的代码，其中堆的大小设置为0x00000200(512B)。  
堆名为Heap_ Mem，不初始化，可读可写，8（23）字节对齐。Heap_Size为堆的大小，heap_base为堆的起始地址，heap_limit为堆的结束地址，因为堆是由低地址向高地址生长。堆的作用是用于malloc()函数申请的动态内存的分配。
![20240224161622](https://gcore.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240224161622.png)

**3、中断向量表**  
![20240224162243](https://gcore.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240224162243.png)

PRESERVE8： 指定当前文件的堆栈按照 8 字节对齐    
THUMB： 表示后面指令为 THUMB 指令。    

EXPORT： 声明一个标号具有全局属性，可被外部的文件使用。如果是 IAR 编译器，则使用的是 GLOBAL 这个指令
![20240224162400](https://gcore.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240224162400.png)

__Vectors是异常/中断向量表的起始位置，__Vectors_End是中断向量表的结束位置， __vectors__Size中断向量表的大小。  

中断向量表的每个成员都是一个字长（32bit）
![20240224162516](https://gcore.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240224162516.png)![20240224162540](https://gcore.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240224162540.png) 

【WEAK】：表示弱定义，如果外部文件优先定义了该标号则首先引用该标号，如果外部文件没有声明也不会出错。这里表示复位子程序可以由用户在其他文件重新实现，这里并不是唯一。  
因此，程序编译时会优先将其他文件中的同名函数首地址放置到异常或中断向量表中对应中断服务函数的地址。
![20240224162829](https://gcore.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240224162829.png)

**4、复位中断服务程序**    
定义一个名为.text代码段，可读。   
![20240224163529](https://gcore.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240224163529.png)

**复位中断服务程序是系统上电后第一个执行的程序，调用 **SystemInit**函数初始化系统时钟，然后调用C库函数_main,最终调用main函数进入C程序。**   
![20240224163647](https://gcore.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240224163647.png)

LDR：从存储器加载字到一个寄存器。   
BL：跳转到由寄存器/标号给出的地址，并把跳转前的下条指令地址保存到链接寄存器。   
BLX：跳转到由寄存器给出的地址，并根据寄存器的LSE确定处理器的状态，还要把跳转前的下条指令地址保存到链接寄存器。   
BX：跳转到由寄存器/标号给出的地址，不用返回。  

IMPORT:表示该标号来自外部文件，跟C语言中的关键字EXTERN类似。这里表示SystemInit和_main这两个函数均是来自外部的文件。

Systemlnit是一个标准的库函数，在system_stm32f4xx.c这个库文件中定义，主要作用是配置系统时钟，在调用这个函数之后，STM32F429的系统时钟被配置为180MHz。

main是一个标准的C库函数，主要作用是初始化用户堆栈，最终调用main函数进入C程序。在C应用程序中，必须有一个main函数。需要注意的是，_main不是用户C程序的main 函数。  

**5、异常和中断服务程序**  
异常和中断服务程序（NMI_Handler）的地址存储在0x08000008地址处。  
【0x08000000：存储异常/中断向量表地址；0x08000004：存储符复位中断服务程序地址】

**6、用户堆栈初始化**  

判断是否定义了__MICROLIB,如果定义了，则赋予标号__initial_sp(栈顶地址)、__heap_base(堆起始地址)、__heap_limit(堆结束地址)全局属性。可供外部文件调用。

若没有定义，则使用默认的C库函数，然后初始化用户堆栈大小，这部分由C库函数__main来完成，当初始化堆栈之后，就调用main函数进入C程序。

![20240224165121](https://gcore.jsdelivr.net/gh/xupengfeir/Notes-and-Articles/Image/20240224165121.png)

注：  
在STM32微控制器中，系统堆栈（System Stack）和用户堆栈（User Stack）是两个不同的堆栈区域。

1、系统堆栈（System Stack）：

·系统堆栈是用于保存中断服务函数和异常处理函数的局部变量、寄存器值和返回地址等信息的堆栈区域。  
·系统堆栈由硬件自动管理，用于处理各种中断请求和异常情况。  
·系统堆栈通常位于内存的高地址区域，由硬件和启动文件初始化并维护。  
·系统堆栈的大小通常由编译器和链接器在生成可执行文件时确定，并且通常比用户堆栈的大小要小。   

2、用户堆栈（User Stack）：

·用户堆栈是用于保存程序执行过程中函数调用的局部变量、函数参数、返回地址等信息的堆栈区域。   
·用户堆栈由用户自行管理，用于保存程序执行过程中的临时数据。  
·用户堆栈通常位于内存的低地址区域，可以由用户根据实际需求分配和管理。  
·用户堆栈的大小可以根据程序的需求进行调整，通常由用户在程序设计和开发过程中确定。
