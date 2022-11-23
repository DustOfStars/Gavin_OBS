> MKW38A4 startup code for use with MCUXpresso IDE

## 一、 涉及语法

### 1.1 if define 和 pragma

```c
#if defined (DEBUG)
#pragma GCC push_options
#pragma GCC optimize ("Og")
#endif // (DEBUG)
```

#### if defined

> 如果您决定定义这个 `DEBUG` 变量，那么将编译 `#ifdef/#endif` 块中包含的任何内容。否则不会。您可以通过在代码顶部开头（在源代码中）包含 `#define DEBUG` 来定义它，**或者将其作为编译参数中的标志来定义**。
> 
> 因此，不必每次都来更改源文件。 
> 
> 这些 `#ifdef/#endif` 块可能包含用于输出描述性消息，以帮助跟踪和验证程序执行的诊断代码。

**在编译时，使用命令 `gcc -Dflag -o out.exe` 来生成可执行文件，如果没有 `-D` ，则不编译 `#if defined （）` 中的内容；**

举例：

```c
#include <stdio.h>

int main()
{
    for(int i = 0; i < 10; i++) {
    #if defined RANDOM_NAME
        printf("%d\n", i);
    #endif
    } 

    return 0;
}

// 使用 gcc -DRANDOM_NAME foo.c -o out.exe 执行一次；
// 使用 gcc foo.c -o out.exe 执行一次；
// 对比，只有 -D的有打印
```


#### pragma

> [参考链接🔗](https://www.geeksforgeeks.org/pragma-directive-in-c-c/)

该指令是一个**特殊用途指令，用于打开或关闭某些功能**。这些类型的指令是**特定于编译器的**，即它们因编译器而异。

下面讨论了一些 `#pragma` 指令：

##### 1. `#pragma startup` 和 `#pragma exit`

这些指令帮助我们指定在程序启动之前（<u>***在控制传递给 main() 之前***</u>）和程序退出之前（就在控制从 main() 返回之前）需要运行的函数. 
注意：以下程序不适用于 GCC 编译器。 

看下面的程序：
```c
#include<stdio.h>
void func1();
void func2();

#pragma startup func1
#pragma exit func2

void func1() 
{
    printf("Inside func1()\n");
}

void func2()
{
	printf("Inside func2()\n"); 
}

int main()
{
	printf ("Inside main()\n");
	return 0; 
}
```

输出：
```
Inside func1()
Inside main()
Inside func2()
```

在GCC上的输出是：
```
Inside main()
```
发生这种情况是因为 GCC 不支持`#pragma startup` 或 `exit`。但是，您可以使用以下代码在 GCC 编译器上获得类似的输出：

```c
#include <stdio.h>

void func1();
void func2();

void __attribute__((constructor)) func1();
void __attribute__((destructor)) func2();

void func1()
{
	printf("Inside func1()\n");
}

void func2()
{
	printf("Inside func2()\n");
}

int main()
{
	printf("Inside main()\n");
	return 0;
}
```

输出：
```
Inside func1()
Inside main()
Inside func2()
```


##### 2. `#pragma warn Directive`

该指令**用于隐藏编译期间显示的警告消息**。当我们有一个大型程序并且我们想在查看警告之前解决所有错误，然后通过使用它我们**可以通过隐藏所有警告来先只关注于错误**时，这可能对我们有用。我们可以通过对语法进行轻微更改来再次让警告可见。 

句法：
```c
#pragma warn +xxx //(To show the warning)
#pragma warn -xxx //(To hide the warning)
#pragma warn .xxx //(To toggle between hide and show)
```

我们可以隐藏警告，如下所示：
- `#pragma warn -rvl` ：该指令隐藏了该返回value的函数不返回value时引发的警告。 
- `#pragma warn -par` ：该指令隐藏了当函数未使用到传递给它的参数时引发的警告。
- `#pragma warn -rch` ：该指令隐藏了代码无法访问时发出的警告。例如：函数中return语句之后写的任何代码都是不可达的。

示例：
```c
// Example to explain the working of
// #pragma warn directive
// This program is compatible with C/C++ compiler

#include<stdio.h>

#pragma warn -rvl /* return value */
#pragma warn -par /* parameter never used */
#pragma warn -rch /*unreachable code */
		
int show(int x)
{
	// parameter x is never used in
	// the function
	
	printf("GEEKSFORGEEKS");
	
	// function does not have a
	// return statement
}
			
int main()
{
	show(10);
	
	return 0;
}

```

> 关于GCC编译和警告处理的用法，推荐下面两篇文章：
> [Linux系统平台下关于GCC编译及使用的方法](http://h.leomei.com/a20264701.html)
> [GCC编译警告选项总结](https://blog.51cto.com/u_15061934/3793701)

##### 3. `#pragma GCC poison`
该指令受 GCC 编译器支持，用于从程序中完全删除标识符。如果我们想阻止一个identifier，那么我们可以使用`#pragma GCC poison`指令。

示例：
```c
// Program to illustrate the
// #pragma GCC poison directive

#include<stdio.h>

#pragma GCC poison printf

int main()
{
	int a=10;
	
	if(a==10)
	{
		printf("GEEKSFORGEEKS");
	}
	else
		printf("bye");

	return 0;
}
```

##### 4. `#pragma GCC dependency`
`#pragma GCC dependency` ：`#pragma GCC dependency`允许您检查当前文件和另一个文件的相对日期。如果其他文件比当前文件更新，则会发出warning。如果当前文件是从其他文件派生的，并且应该重新生成，这将很有用。

语法：
```c
#pragma GCC dependency "parse.y"
#pragma GCC dependency "/usr/include/time.h" rerun fixincludes
```


##### 5. `#pragma GCC system_header`
这个 pragma 没有参数。**它会导致当前文件中的其余代码被视为来自系统头文件**。

##### 6. `#pragma once`
`#pragma once` 指令有一个非常简单的概念。**包含此指令的头文件仅包含一次**，即使程序员在编译期间多次包含它也是如此。

这不包括在任何 ISO C++ 标准中。**该指令的工作方式类似于#include 防护习惯用法**。使用`#pragma once`可以使程序免于多次包含优化。

语法：
```c
#pragma once
```


### 1.2 extern “C” {

> \_\_cplusplus 用于在用 C++ 编译的 C++ 代码中使用 C 外部 - 名称修改， 这个用法是，当使用C++编译时，指定以下extern中的内容为C代码，防止编译出错。

```
#ifdef __cplusplus
// if we are being compiled with a C++ compiler then declare the
// following functions as C functions to prevent name mangling.
extern "C" {
#endif

// exported C function list.
int foo (void);

#ifdef __cplusplus
// if this is a C++ compiler, we need to close off the extern declaration.
};
#endif
```



## 二、 ARM相关


## 三、 代码流程

1. 闪存配置块：16字节的闪存配置区域，存储默认保护设置（复位时加载）和安全信息，允许MCU限制对Flash Memory模块的访问。**由链接器脚本放置在地址`0x400`处**。
```c
__attribute__ ((used,section(".FlashConfig"))) const struct {
    unsigned int word1;
    unsigned int word2;
    unsigned int word3;
    unsigned int word4;
} Flash_Config = {0xFFFFFFFF, 0xFFFFFFFF, 0xFFFFFFFF, 0xFFFFFFFE};
```
2. 声明外部 SystemInit 函数；
3. **核心异常处理程序**的前向声明。
	1. 当application定义了一个处理程序（具有相同的名称），这将自动优先于这些弱定义。
	2. 如果你的应用程序是一个C++程序，那么在你的主程序中，任何在C++文件中定义的中断处理程序都需要有C链接，而不是C++链接。要做到这一点，请确保你在中断处理程序周围使用extern "C" { .... }在你的**主程序代码中的中断处理程序周围**。
	3. `WEAK void IntDefaultHandler(void);`
4. application IRQ handlers 的前向声明。当应用程序定义了一个handlers（具有相同的名称），这将自动优先于下面的弱定义；
	1. `WEAK void PORTB_PORTC_IRQHandler(void);`
5. driver IRQ handlers 的前向声明。这些被别名为`IntDefaultHandler`，它是一个 for循环。当驱动程序定义了一个handler（具有相同的名称），它将自动优先于这些弱定义。
	1. `void PORTB_PORTC_DriverIRQHandler(void) ALIAS(IntDefaultHandler);`
6. 应用程序的入口点。
	1. \_\_main() 是基于 Redlib 的应用程序的入口点。
	2. main()是基于Newlib的应用程序的入口点。
	3. `extern int main(void);`
7. 来自Linker Script的堆栈顶部指针的外部声明;
	1. `extern void _vStackTop(void);`
8. 矢量表。依赖于链接器脚本在内存中的正确位置。
	1. `extern void (* const g_pfnVectors[])(void);`
	2. `extern void * __Vectors __attribute__ ((alias ("g_pfnVectors")));`
	3. `__attribute__ ((used, section(".isr_vector")))`
	4. `void (* const g_pfnVectors[])(void) = {...`
	5. **注意： g_pfnVectors是一个指针数组，指向不带参数且不返回任何内容的函数，即原型为 void fn(void);**
9. 用于执行RW和BSS数据部分的初始化的函数。这些函数被写成独立的函数，而不是在`ResetISR()`函数中被内联，以应对具有多个内存库的MCU。
10. 下面的符号是由linker生成的结构，表示 "Global Section Table"中各点的位置。这个表是由链接器通过Code Red管理的链接器脚本机制创建的。它包含每个RW数据部分的加载地址、执行地址和长度，以及每个BSS（零初始化）部分的执行和长度。
	1. `extern unsigned int __bss_section_table_end;`
11. 为你的代码重置入口点。设置一个简单的运行环境，并初始化C/C++库。
	1. `__attribute__ ((section(".after_vectors.reset")))`
	2. `void ResetISR(void) {...`
12. 默认的核心异常处理程序。通过在你的应用程序代码中定义你自己的处理程序来覆盖这里的处理程序。
	1. `WEAK_AV void NMI_Handler(void) { while(1) {}}`
13. 如果一个意外的中断发生或一个特定的处理程序没有出现在应用程序代码中，处理器就会在这里结束。
	1. `WEAK_AV void IntDefaultHandler(void) { while(1) {} }`
14. 默认的应用程序异常处理程序。通过在你的应用程序代码中定义你自己的handler来覆盖这里的处理程序。这些例程会调用驱动程序的异常处理程序，**如果没有驱动程序的异常处理程序，则调用IntDefaultHandler()**。
	1. `WEAK_AV void PORTB_PORTC_IRQHandler(void) {   PORTB_PORTC_DriverIRQHandler(); }`



[[2022-11-23_星期三]]



