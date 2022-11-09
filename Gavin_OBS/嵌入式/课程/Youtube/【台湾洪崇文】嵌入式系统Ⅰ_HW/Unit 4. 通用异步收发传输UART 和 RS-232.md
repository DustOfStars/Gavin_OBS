
## Section 1. 串口通讯介绍

- 串口通讯：Serial Communication
	- 只使用一条信号线，**每个单位时间传送一个bit**；
	- 一个byte需要8个单位时间；
- 并口通讯：Parallel Communication
	- 每个单位时间有多条信号线，同事并行传输多个bit；
	- 如果有8条信号线，一个byte只需要一个单位时间就可以传输完毕；
- 常见的串口通讯：
	- Chip Level：在同一个PCB上，IC与IC之间的通讯；
		- **UART I2C SPI**
	- Device Level：原件对原件，或者PC对原件
		- **Ethernet USB SATA RS232**
- 串口通讯的分类：
	- 同步：通讯的装置之间有时钟信号同步，会多一条时钟线；
	- 非同步：通讯的装置之间以协定好的速率传输，不需要时钟线；
	- 单工：只能单向传递；
	- 半双工：可以双向传递，但是不能同时；
	- 全双工：可以同时双向传递；

RX62T上的串口通讯：
1. SCI：Serial Communications Interface
	- 为全双工非同步通讯接口，包括常用的UART；
	- 2条信号线，没有同步线；
2. IIC： Inter Integrated Circuit
	- I2C，**是半双工同步通信接口**；
	- 1条信号线 + 1条时钟线；
3. SPI： Serial Peripheral Interface
	- **全双工通讯同步接口**；
	- 2条信号线 + 1条时钟线
4. 应用于车辆与工业控制的非同步串口通信接口：
	- LIN ： Local Interconnect Network
	- CAN ： Controller area network


## Section 2. UART通讯与RS-232

### 2.1 UART 格式

可调整的格式参数：
- 数据帧的bit数：7 bit or 8 bit；
- 校验位：奇校验、偶校验、不校验；
- 停止位：1 bit or 2 bit；
- 数据方向： 高位优先或者低位优先；


### 2.2 非同步传输格式

一个UART的Frame包含了：
- 一个起始位、一个data、同位元检测、停止位

![UART Frame](https://raw.githubusercontent.com/DustOfStars/ObsPicGo/master/Gavin_Obs/20221109113637.png)


### 2.3 同位元检测

同位元检测步骤：
- 先将data中的所有位元相加：
	- 比如： 0xA5  --> 10100101 --> 4（和为4）
- 奇同位检测：检测位 + data位 的 和 必须为奇数， 所有此时检测位设为1，保持4+1=5奇数；
- 偶同位检测：加起来必须是偶数；


### 2.4 波特率 Baud Rate

- 计算传输速度的单位；
- 每秒传输的bit数：  bit per second， bps
- 因为发送和接收两端不是同步的，所以有误差，一般需要保证误差小于5%；
- 可使用的传输速度由两边硬件效能和传输路径干扰状况而定；


### 2.5 RS232介绍

- RS232是美国电子工业联盟EIA订立的标准
- 最初使用25只脚的D-SUB街头作为标准街头
	- 现在简化为9只脚的DE-9接头；
- RS232的线路不能太长，**限制在15米以内**；
- 信号高低准位为 **正的3-15V 和 负的3-15V**；

DE-9接头：
- 九只脚的公母顺序不同；
	- 公头是PC端，使用2、3、5 脚；
	- 母头是device端，使用2 3 5 脚；
- 公头Pin脚说明：（母头Tx Rx 相反）
	- Pin2 Receiver 接收 RX
	- Pin3 Transmit 发送 TX
	- Pin5 Ground 地线
![](https://raw.githubusercontent.com/DustOfStars/ObsPicGo/master/Gavin_Obs/20221109152904.png)
PC端 USB转为RS232信号之后，就会看到COM Port口；


## Section3. 中断机制

中断：
- 中断interrupt就是处理器收到外围设备或者外界的触发信号，加以响应的机制；
- 可以设定当周边线路完成硬件程序，或者外界预期外的状况，对处理器核心发出中断信号；
- **处理器收到中断触发信号，必须暂停当前程序，处理中断副程序，处理完再返回原本的程序，继续执行**；

![](https://raw.githubusercontent.com/DustOfStars/ObsPicGo/master/Gavin_Obs/20221109154241.png)

> 中断可以方便开发者不浪费资源随时检测周边设备是否执行完毕，是否有外部输入，可以发挥处理器真正的效率；

**中断来源**：
- 周边中断
	- 比如： Timer计数、UART收到命令、ADC转换完成；
- 软件中断：
	- 在程序中，加入中断触发；
- 外部中断：
	- 侦测引脚状态：由外部信号上升或者下降沿触发中断；

中断等级：
- 当同时发生几个中断时，根据优先级判断哪个先处理；
- 在RX62T中分为16个等级：
	- 0等级最低
	- 15等级最高

> **常常会有一些系统等级的中断发生，这些中断不希望被软件错误的关掉，所以需要NMI（Non-Maskable Interrupt）**

**NMI**：
- 一般的中断有disable的机制，可以决定关闭中断，使它不能通知到CPU；
- 不可遮蔽中断，NMI，用来避免软件错误导致无法处理系统等级的中断；
- 来源：
	- NMI pin （外部硬件触发）
	- 电源电压过低
	- 时钟震荡停止


## Section 4. PDG2设置与实例



## Section 5. 通过PC控制LED明灭

**看看这个原始链接**： [Youtube](https://www.youtube.com/watch?v=NPrDM727dA8&list=PLI6pJZaOCtF3vI21_rUg1mj-GTpnGnvBp&index=25)

框图：

![](https://raw.githubusercontent.com/DustOfStars/ObsPicGo/master/Gavin_Obs/20221109160732.png)

转接板线路图：

![](https://raw.githubusercontent.com/DustOfStars/ObsPicGo/master/Gavin_Obs/20221109160920.png)









[[2022-11-09_星期三]]

