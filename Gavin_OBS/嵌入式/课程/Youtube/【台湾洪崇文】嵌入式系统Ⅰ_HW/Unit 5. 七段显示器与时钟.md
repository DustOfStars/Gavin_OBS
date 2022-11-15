
## Section 1. 比较匹配计时器 Compare Match Timer（CMT）

### 1.1 计时器 Timer

- 计时器是处理器常见的内建周边；
- 当计数器的计数来自于clock时，就是计时器Timer；
- 可设定Timer周期时间
	- 背光控制：50ms ，人眼就会以为是常亮；
	- CO2检测的话，5min一次就可以；
- **配合不同的比较器与输出设定，产生控制所需要的硬件波形**；
- RX62T中内建的定时器有：CMT、GPT（General Purpose Timer 通用型计数器）、MTU（多功能计时器）

![比较计时器产生方波](https://raw.githubusercontent.com/DustOfStars/ObsPicGo/master/Gavin_Obs/20221109162904.png)

### 1.2 比较匹配计时器

- CMT是处理器常见的内建周边，**加入比较器之后，可以控制该计时器周期时间**；
- 可以由软件控制计时器计时长短；
- **当时间达到Timer的周期时，可以产生中断**；
- 设计中断子程序进行控制动作；

RX62T的CMT：
- 可以有不同的时钟来源；
- 使用者自行规划计时长短；
- CMT让处理器有更多应用：
	- 闹钟；
	- LED闪烁、亮度；
	- 电力电子
	- 马达控制；


### 1.3 CMT工作原理

CMT方框图：

![](https://raw.githubusercontent.com/DustOfStars/ObsPicGo/master/Gavin_Obs/20221109172052.png)

> 周边时钟分频，作为CMT的时钟源，CMCNT计数器一直增加，和CMCOR常量进行比较，输出进入控制单元；
> 当CMCN增加到CMCOR时，**CMCNT会清零，同时触发中断**；
> 所以周期时N+1 倍的Count Clock周期；（N是CMCOR中储存的常量数字）

![](https://raw.githubusercontent.com/DustOfStars/ObsPicGo/master/Gavin_Obs/20221114103503.png)



## Section 2. 七段显示器的驱动和线路

### 2.1 七段显示器

常见的七段显示器种类：
- 一位、二位、四位
- 共阴极、共阳极

![](https://raw.githubusercontent.com/DustOfStars/ObsPicGo/master/Gavin_Obs/20221114111504.png)

七段显示器的数字显示：
- 一位数字由**八颗LED**组成；（有个小数点H）
- 制定对应LED亮来表示数字；
- **用一个8bit char来写数字显示，每个bit位对应一个LED**；

四位共阳极七段显示器：
- **搭配三极管使用。避免处理器驱动能力不足**；
- 三极管导通时该位的七段显示器才会有动作；
- 利用视觉暂留现象与轮询技巧，**每次三极管控制只点亮某一个七段，再由八个接脚选通某个LED，闪烁**，即可全部看到，避免了硬件接脚的不足；
- **4个控制七段 + 8个控制bit位 = 12个接脚即可！**

![](https://raw.githubusercontent.com/DustOfStars/ObsPicGo/master/Gavin_Obs/20221114111830.png)


## Section 3. 按钮与线路

### 3.1 按钮电路

- 一般按钮只有短路和开路功能；
- 为了随时从GPIO接脚读入、确定准位，加入了上拉下拉电阻，可以定义未按按钮时的输入准位；

![](https://raw.githubusercontent.com/DustOfStars/ObsPicGo/master/Gavin_Obs/20221114113046.png)

### 3.2 硬件弹跳现象

- 弹簧式按钮会在按下和放开时，有短暂的机械弹跳现象，产生不正确的信号；

如果用来计数count，很可能按一下记录了好几次！！

![](https://raw.githubusercontent.com/DustOfStars/ObsPicGo/master/Gavin_Obs/20221114113421.png)


### 3.3 硬件防弹跳

1. 增加一个RC缓冲滤掉弹跳！
	![](https://raw.githubusercontent.com/DustOfStars/ObsPicGo/master/Gavin_Obs/20221114113634.png)

2. **设计flip flop来做操作**；？？？
	![](https://raw.githubusercontent.com/DustOfStars/ObsPicGo/master/Gavin_Obs/20221114113807.png)

### 3.3 软件防弹跳

- 读取到按钮信号转换之后，开始**以CMT计时器20ms之后才能再次读取按钮信号**；

![](https://raw.githubusercontent.com/DustOfStars/ObsPicGo/master/Gavin_Obs/20221114114007.png)


## Section 4. 系统分析和规划

设计系统动作用简单电子计时器为例：
- 系统动作模式：
	1. 时钟
	2. 闹钟模式
	3. 设定时钟时间
	4. 设定闹钟时间
- 系统输入：
	- 平时工作于时钟模式
		- 按钮1进入时钟设定模式、按钮2进入闹钟设定模式
	- 时间设定：
		- 按钮1返回时钟模式，按钮2更换位数，按钮3数值+1，按钮4数值-1；
	- 闹钟模式：
		- 当计时符合闹钟设定的时间，启动闪烁，按任意键取消闪烁；
- 系统输出：
	- 显示当前时间，只显示时分，秒由小数点闪烁代替；
- 闹钟模式：
	- 四个位数都闪烁显示当前时间；
- 调整时钟、调整闹钟：
	- 闪烁显示当前修改的位数；
	- 四个小数点全亮，表示进入闹钟设定；

使用的周边：
- 输入：
	- 四个按钮，需要四个GPIO作为输入脚；
- 输出：
	- 四位数七段显示器，12个GPIO作为输出脚；
- 计时器，CMT的2组计时器：
	- 毫秒的计数，防弹跳的使用；
	- 秒的计数，时钟使用；

输出设定：
- 一共需要12个输出接脚，分为三组：
	- 七段显示器的数字显示，7个接脚；
	- 小数点，1个接脚；
	- 七段显示器的显示位数切换，4个接脚；
- 必须注意并不是所有的接脚都能作为输出：
	- 比如Port4只能作为输入；
	- P40已经连接开发板的可变电阻，不能作为其他用途；
- 规划：
	- 设定Port7的7个脚作为数字显示；
	- Port9的4只脚作为显示位数切换；
	- P3-0作为小数点显示；

输入设定：
- 需要4个GPIO作为输入脚；
- 考虑其他Port脚位大多无法从0开始，在编写程序时有点不方便；
- 规划：
	- 选择PortB作为输入；
	- 使用PortB 0~3

**系统的硬件规划图：**

![](https://raw.githubusercontent.com/DustOfStars/ObsPicGo/master/Gavin_Obs/20221114162336.png)



## Section 5. PDG2规划周边驱动

- PDG2设定GPIO
- 选到Port3，输入输出等等选中；
- CMT设定
	- 第一组设定在10ms，用于软件防弹跳；
	- **第二组希望设定1m，需要分频到512倍，但是不足，所以以0.5m为一次中断，再用软件处理**；

## Section 6. 程序代码分析

> 总纲：

- PDG2设定
- 程序流程
- 七段显示器的轮询显示
- 输入的处理（软件防弹跳）

1. 所需变量编写：

![](https://raw.githubusercontent.com/DustOfStars/ObsPicGo/master/Gavin_Obs/20221114163922.png)

2. 主函数：
![](https://raw.githubusercontent.com/DustOfStars/ObsPicGo/master/Gavin_Obs/20221114164201.png)
- 因为3-0是输入，不用管；
- 其他都是输出，set为高，表示初始LED熄灭；

3. CMT中断
![](https://raw.githubusercontent.com/DustOfStars/ObsPicGo/master/Gavin_Obs/20221114164839.png)







[[2022-11-09_星期三]]

