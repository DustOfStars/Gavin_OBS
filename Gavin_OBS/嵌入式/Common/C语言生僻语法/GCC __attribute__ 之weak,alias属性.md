# GCC \_\_attribute\_\_ 之weak,alias属性

[blog.csdn.net](https://blog.csdn.net/BingoAmI/article/details/78683906)

Weak Alias 跟 Weak Reference 完全没有任何关系，不过是我在看到 Weak Reference 的时候想到的而已。

Weak Alias 是 gcc 扩展里的东西，实际上是函数的属性。这个东西在库的实现里面可能会经常用到，比如 glibc 里面就用了不少。抄录一段 [gcc 手册](http://gcc.gnu.org/onlinedocs/gcc-4.0.0/gcc/Function-Attributes.html)里面的话解释下函数属性是干啥的，

> In GNU C, you declare certain things about functions called in your program which help the compiler optimize function calls and check your code more carefully.

先上代码，看看 weak alias 怎么写。第一个文件 dummy.c 内容，

`#include int \_\_foo() { puts(”I do no thing.”); } int foo() \_\_attribute\_\_ ((weak, alias(”\_\_foo”)));`

weak 和 alias 分别是两个属性。**weak 使得 `foo` 这个符号在目标文件中作为 weak symbol 而不是 global symbol**。用 `nm` 命令查看编译 dummy.c 生成的目标文件可用看到 foo 是一个 weak symbol，它前面的标记是 W。给函数加上weak属性时，即使函数没定义，函数被调用也可以编译成功。-

00000000 T \_\_foo 00000000 W foo U puts

**而 alias 则使 `foo` 是 `__foo` 的一个别名**，\_\_foo 和 foo 必须在同一个编译单元中定义，否则会编译出错。

那么这个东西的用处是？

看第二个文件，func.c,

```c
#include int foo() { puts(”I do something.”); }
```


这里有一个函数名字是 `foo`。如果我们编译 func.c 和 dummy.c 得到两个目标文件，当我们同时使用 func.o 和 dummy.o 和其他目标文件进行链接时，如果其他目标文件里面引用符号 `foo`，最终使用到的是 func.c 中定义的函数，而不是 `__foo`，虽然它有一个别名 `foo`。也就是说，我们最终使用到的函数会是“实际做事”的那个函数。当然，单独使用 dummy.o 链接的话使用的是那个“不做事”的函数。**如果 dummy.o 中的 `foo` 不是 weak symbol 的话，在链接时会产生冲突，这就是我们要使用 weak 的原因**。

glibc 的实现里面经常用 weak alias。比如它的 `socket` 函数，在 C 文件里面你会看到一个 `__socket` 函数，它几乎什么都没有做，只是设置了一些错误代码，返回些东西而已。在同一个 C 文件里面会再声明一个 `__socket` 的 weak alias 别名 `socket`。实际完成工作的代码通过汇编来实现，在另外的汇编文件里面会有设置系统调用号，执行 `sysenter` 或者 `int` 等动作去请求系统调用。以前看 glibc 里面系统调用的实现的时候郁闷过很久，就是那个时候才知道了 weak alias 这个东西。

[查看原网页: blog.csdn.net](https://blog.csdn.net/BingoAmI/article/details/78683906)



[[2022-11-07_星期一]]