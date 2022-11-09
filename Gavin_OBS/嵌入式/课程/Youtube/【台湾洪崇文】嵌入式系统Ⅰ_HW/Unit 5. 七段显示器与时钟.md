
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








## Section 2. 七段显示器的驱动和线路


## Section 3. 系统分析和规划


## Section 4. PDG2规划周边驱动





[[2022-11-09_星期三]]

