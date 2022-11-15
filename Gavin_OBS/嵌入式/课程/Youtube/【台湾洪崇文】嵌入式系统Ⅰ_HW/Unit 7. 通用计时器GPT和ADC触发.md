> 总纲：
> 1. 基本脉冲宽度调整PWM的概念
> 2. 脉波宽度调整PWM
> 3. 通用计时器GPT功能介绍
> 4. 使用PDG2设定GPT功能
> 5. 使用GPT自动触发ADC实例


## Section 1. 基本脉冲宽度调整PWM的概念

> GPT ： general PWM Timer, 用来做ADC的触发控制；
> PWM ： Pulse-width modulation， 脉宽调制

处理器产生模拟信号：
- 模数转换器ADC将模拟世界的连续信号，转换为抽象的数字信号，可供处理器使用；
- 处理器如果要产生模拟信号，需要数模转换器DAC；
- **DAC技术成熟，但是成本较高，只有少数处理器内建DAC，同时电流驱动能力受限**；
- 加权电阻型、R-2R为常见的DAC架构；

使用脉宽调制器PWM，产生和DAC类似的效果；

PWM：
- 是DAC的替代方案；
- **利用高频连续周期性脉冲产生等效的模拟信号**；
- **通过改变工作周期（duty）获得等效的模拟输出的改变**；

![](https://raw.githubusercontent.com/DustOfStars/ObsPicGo/master/Gavin_Obs/20221115160142.png)

PWM的应用：
- 早期用在通讯领域
- 数字电视、手机荧幕背光
- 直流马达驱动（调整电压）
- 交流马达驱动（SPWM、SVPWM）
- 直流电源转换器（BUCK、BOOST）
- 伺服马达控制信号（角度、转速）


## Section 2. 脉波宽度调制器PWM

> PWM使用固定的脉波频率，改变工作周期，产生对应的电压；

### 2.1 PWM参数

- **载波频率（切换频率， fp = 1/Tp）**
- Tp是PWM的周期；
- Td ： 工作周期，duty，是逻辑1所占的时间长度；
- 工作周期比（duty ratio/cycle） 用D来表示
	- 逻辑1在整个周期中所占时间长度比；
	- 也叫作用比、工作比、占空比；

![](https://raw.githubusercontent.com/DustOfStars/ObsPicGo/master/Gavin_Obs/20221115161148.png)

### 2.2 SPWM（弦波的PWM）

> 实际电压是方波，但是通过改变某一刻的占空比，等效为不同电压值，连起来拟合正弦波；
- 10个point模拟：

![](https://raw.githubusercontent.com/DustOfStars/ObsPicGo/master/Gavin_Obs/20221115161400.png)

- 100个point模拟：

![](https://raw.githubusercontent.com/DustOfStars/ObsPicGo/master/Gavin_Obs/20221115161633.png)


### 2.3 DAC低通滤波

- 可以通过低通滤波器。让PWM输出逼近模拟信号；
- 简单的一阶RC滤波就可以获得效果；
- <mark>低通滤波器的截止频率： fc = 1 / (2paiRC)</mark>
- 放大器可以提高驱动能力；

![](https://raw.githubusercontent.com/DustOfStars/ObsPicGo/master/Gavin_Obs/20221115161946.png)


## Section 3. 通用计时器GPT功能介绍

### 3.1 通用PWM计时器

- RX62T中提供了三种计时器 CMT、GPT、 MTU3
- 通用PWM计时器（GPT）可以提供PWM输出，并附加其他功能：
	- 例如**互补式PWM**
	- ***短路保护时间（dead-time，死区时间）***
- 支援外部硬件启动、停止、清除定时器；

### 3.2 互补式PWM

- GPT可以输出两个互补的PWM信号
	- 在马达驱动和电力电子应用中十分常见；

![](https://raw.githubusercontent.com/DustOfStars/ObsPicGo/master/Gavin_Obs/20221115163150.png)


### 3.3 短路保护时间 dead time

- 电力电子和马达驱动电路中常常使用两个开关串联在电源和地之间，（上下臂），中介接点提供正电或接地；
- 若两个开关同时导通，会造成电源到地短路；
- 开关先天的导通与截止延迟，造成其中一个开关没完全关闭，另一个已经开始导通，会短路；
- 因此，在切换开关时，要错开上下臂信号时间，称为短路保护时间dead time；

![](https://raw.githubusercontent.com/DustOfStars/ObsPicGo/master/Gavin_Obs/20221115170539.png)

- 以桥式电路做示范：
	- S1 S4导通，切换为S2 S3导通
	- 如果没有dead time保护，因为开关特性，上下臂会造成电源短路；
	- 所以，加入短路保护时间：
		- 先将S1 S4 截止
		- delay之后，S2 S3导通

![](https://raw.githubusercontent.com/DustOfStars/ObsPicGo/master/Gavin_Obs/20221115170913.png)

- 若高准位控制开关通道，橘色为下臂，蓝色为上臂；
- 短路保护时间避免两个信号同时导通；
- GPT可以自动产生dead time；

### 3.4 三角波和锯齿波

- GPT的计数方式可以简单分为两种；
	- 三角波：计时器先数到寄存器数值，然后倒数到0再正数；
	- 锯齿波：正数到寄存器数值，归零再继续；


## Section 4. 使用PDG2设定GPT功能






[[2022-11-15_星期二]]