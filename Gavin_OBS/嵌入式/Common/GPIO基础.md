## 1、GPIO入门知识

> GPIO是什么？

GPIO全称general purpose input output通用输入输出端口，*GPIO口既可以做输入也可以做输出，这些都是通过配置GPIO的工作模式来实现的。*

- STM32FXXXIGT6：一共有**9组**IO口：PA~PI(PI只有PI0~PI11);一共有140个IO口：16*8+12=140

- STM32的大部分引脚都***除了可以当作GPIO口来使用，还可以复用为外设功能引脚，如串口等***。

- 集体引脚复用功能查找对应的芯片数据手册即可。

## 2、GPIO的8种工作模式（GPIO_Mode）

看IO口电路图更容易理解

四种输入模式：
- 输入浮空 :外部输入直接进入IDR寄存器，施密特触发器打开
- 输入上拉：外部输入经过上拉进入IDR寄存器，施密特触发器打开
- 输入下拉：外部输入经过下拉进入IDR寄存器，施密特触发器打开
- 模拟输入：模拟输入时，施密特触发器关闭，直接进入模拟通道

四种输出模式：
- 开漏输出（带上拉或者下拉）：只可以输出强低电平，高电平只能由外部上拉，适合做电流型驱动，吸收电流能力较强（一般20mA以内）
- 开漏复用功能（带上拉或者下拉）
- 推挽输出（带上拉或者下拉）：可以输出高低电平
- 推挽复用功能（带上拉或者下拉）

四种最大输出速度F429：
- 2MHz   低速
- 25MHz 中速
- 50MHz 快速
- 100MHz 高速

对于stm32来说，上电复位后，**GPIO默认为输入浮空状态，部分特殊功能引脚为特定状态**。

**注意复位后，调试引脚处于复用功能上拉或是下拉状态，不能直接当作IO口使用（ PA15,PA14,PA13,PB4,PB3 ）**

## 3、GPIO寄存器

一般而言M3,M4,M7每组都有十个寄存器，共9 * 10 = 90个寄存器

### 3.1 四个32位配置寄存器：
- **GPIOx_MODE**------端口模式寄存器
	![](https://raw.githubusercontent.com/DustOfStars/ObsPicGo/master/Gavin_Obs/20221111141513.png)
- **GPIOx_OTYPER**------端口输出类型寄存器
	![](https://raw.githubusercontent.com/DustOfStars/ObsPicGo/master/Gavin_Obs/20221111141625.png)
- **GPIOx_OSPEEDR**------端口输出速度寄存器
	![](https://raw.githubusercontent.com/DustOfStars/ObsPicGo/master/Gavin_Obs/20221111141815.png)
- **GPIOx_PUPDR**------端口上拉下拉寄存器
	![](https://raw.githubusercontent.com/DustOfStars/ObsPicGo/master/Gavin_Obs/20221111141920.png)



### 3.2 两个32位数据寄存器：

- **GPIOx_IDR**-------输入数据寄存器

[](https://img2018.cnblogs.com/blog/1593746/201902/1593746-20190221222113212-1411795687.png)[![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg2018.cnblogs.com%2Fblog%2F1593746%2F201902%2F1593746-20190221222037214-498186248.png)](https://img2018.cnblogs.com/blog/1593746/201902/1593746-20190221222037214-498186248.png)

GPIOx_ODR-------输出数据寄存器

[](https://img2018.cnblogs.com/blog/1593746/201902/1593746-20190221222114325-1054729312.png)[![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg2018.cnblogs.com%2Fblog%2F1593746%2F201902%2F1593746-20190221222038144-612406166.png)](https://img2018.cnblogs.com/blog/1593746/201902/1593746-20190221222038144-612406166.png)

一个32位置位/复位寄存器：GPIOx_BSRR

[](https://img2018.cnblogs.com/blog/1593746/201902/1593746-20190221222115617-328738181.png)[![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg2018.cnblogs.com%2Fblog%2F1593746%2F201902%2F1593746-20190221222039143-2023091311.png)](https://img2018.cnblogs.com/blog/1593746/201902/1593746-20190221222039143-2023091311.png)

一个32位锁存寄存器：GPIOx_LCKR

[](https://img2018.cnblogs.com/blog/1593746/201902/1593746-20190221222117224-189142087.png)[![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg2018.cnblogs.com%2Fblog%2F1593746%2F201902%2F1593746-20190221222040139-106109718.png)](https://img2018.cnblogs.com/blog/1593746/201902/1593746-20190221222040139-106109718.png)

两个32位复用功能寄存器：低位GPIOx_AFRL&高位GPIOx_AFRH

[](https://img2018.cnblogs.com/blog/1593746/201902/1593746-20190221222118173-859462968.png)[![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg2018.cnblogs.com%2Fblog%2F1593746%2F201902%2F1593746-20190221222041194-1920708582.png)](https://img2018.cnblogs.com/blog/1593746/201902/1593746-20190221222041194-1920708582.png)

