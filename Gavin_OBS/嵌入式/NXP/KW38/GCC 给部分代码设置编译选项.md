>  [GCC设置部分代码的编译选项](https://www.cnblogs.com/DF11G/p/14421342.html)

GCC可以对部分代码设置不同的编译选项，在编译时使用其指明的选项，而不用编译命令里指定的参数。

## 1. 使用`#pragma`指令

参数值可以是数字，也可以是字符串。数字就是优化级别，以O开头的字符串也被认为是一个优化级别（例如（“O1”）（“-O1”）和（“1”）是相同的作用），其他的字符串选项如果没有-前缀，编译器自动添加-f，例如("unroll-loops")代表-funroll-loops。

**该指令最小的作用域为函数，也就是说无法在函数内部使用，但可以作用于多个函数。**

```c
#pragma GCC push_options
#pragma GCC optimize ("O0")

// ...your code...

#pragma GCC pop_options
```

## 2.  使用__attibute__属性

参数值可以是数字，也可以是`-`开头的字符串。数字就是优化级别。以-O开头的字符串也被认为是一个优化选项（例如（“O1”）（“-O1”）和（“1”）是相同的作用），其他的字符串选项如果没有`-`前缀，编译器自动添加`-f`，例如("unroll-loops")代表-funroll-loops。

该选项作用于单个函数，需要在函数开头添加选项属性。

```c
#define OPTIMIZE_ZERO     __attribute__ ((__optimize__ ("-O0")))
#define OPTIMIZE_UNROLL   __attribute__ ((__optimize__ ("-funroll-loops")))
#define OPTIMIZE_UNROLL_ZERO   __attribute__ ((__optimize__ ("unroll-loops","O0")))
#define SECTION_ITCM      __attribute__ ((section (".itcm")))
#define SECTION_DTCM      __attribute__ ((section (".dtcm")))

int test_int[4] SECTION_DTCM;

OPTIMIZE_ZERO  int main(void)
{
    ......
 }
 
OPTIMIZE_UNROLL_ZERO  int test(void)
{
    ......
}

SECTION_ITCM int sec_tcm(void)
{
    ......
}
```

### 3. 修改Linker脚本

对于指定目标文件，如果我们想**让该文件中所有的`text/data/bss`等段内容放置在指定地址**，应该怎么实现？

第一种方法，可以用本文第二段中的__attribute__属性，**对文件中的每个函数或者变量加上指定前缀，然后在Linker脚本安排好对应的Section即可，这样虽然可以满足需求但是稍显繁琐。**

**查阅GCC的文档，没有找到使用`#pragma`指令对多个函数或者变量设置`Section属性`的办法，因此第二种办法需要从Linker文件下手**。

GCC使用的Linker脚本为**LD文件**，从文件中摘出`.text`段分配关系为例：
```ld
.text :
  {
    . = ALIGN(4);
    *(.text)           /* .text sections (code) */
    *(.text*)          /* .text* sections (code) */
    *(.rodata)         /* .rodata sections (constants, strings, etc.) */
    *(.rodata*)        /* .rodata* sections (constants, strings, etc.) */
    *(.glue_7)         /* glue arm to thumb code */
    *(.glue_7t)        /* glue thumb to arm code */
    *(.eh_frame)

    KEEP (*(.init))
    KEEP (*(.fini))

    . = ALIGN(4);
    _etext = .;        /* define a global symbols at end of code */
  } >FLASH
```

上述的LD文件内容指定了所有文件的.text**和**.text\*段放置在FLASH所在基地址。

其中\*符号为多字符通配符，加入.text\*是因为如果编译器开启-ffunction-sections选项，会给所有的函数生成单独的text段，例如main函数会生成.text.main段，加入.text\*就可以覆盖这些text段。

假如我们想把目标文件中**test.o**这个文件从中取出来放在另外的**FLASH2**地址，可以这么实现：
#### 1. 在FLASH段的描述中使用**EXCLUDE_FILE**把该文件排除：

```ld
.text :
  {
    . = ALIGN(4);
    *(EXCLUDE_FILE(*test.o) .text)    /* .text sections (code) */
    *(EXCLUDE_FILE(*test.o) .text*)    /* .text* sections (code) */
    *(.rodata)         /* .rodata sections (constants, strings, etc.) */
    *(.rodata*)        /* .rodata* sections (constants, strings, etc.) */
    *(.glue_7)         /* glue arm to thumb code */
    *(.glue_7t)        /* glue thumb to arm code */
    *(.eh_frame)

    KEEP (*(.init))
    KEEP (*(.fini))

    . = ALIGN(4);
    _etext = .;        /* define a global symbols at end of code */
  } >FLASH
```

#### 2. 在**FLASH2段**的描述中填入该文件text段
```ld
.text_back :
  {
    . = ALIGN(4);
    *test.o(.text)    /* .text sections (code) */
    *test.o(.text*)    /* .text* sections (code) */
    . = ALIGN(4);
  } >FLASH2
```



[[2022-11-23_星期三]]
