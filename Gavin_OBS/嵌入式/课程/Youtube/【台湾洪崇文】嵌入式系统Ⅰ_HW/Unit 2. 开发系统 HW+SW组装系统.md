## Section 1. 硬件线路设计

### 1.1 处理器

- 瑞萨RX62T（R5F562T6DDFM）作为开发板的处理器；
	- 最大系统工作频率 80MHz；
	- 工作电压 5V；
	- 脚位 64 PIN；
	- FLASH 64KB；
	- RAM/Data Flash 8KB；（RAM随机存取，下电丢失，Data Flash读写次数大于FLASH）

### 1.2 使用处理器

- 接脚
	- 印刷在PCB布线上，处理器共有40个接脚连接至排针，提供硬件设计使用；
	- 两组Vcc/Gnd；
	- 64-44 ，其余的脚位都是烧录程序，硬件开发系统和外部时钟需要的；

### 1.3 开发板线路图

1. LED + 限流电阻保护
2. 10MHz 晶振 + 8倍频电路 （晶振不直接使用80MHz是防止频率过高影响其他电路）；
3. 可变电阻 --> 分压 + 测试ADC + 旋钮UI；
4. 电容一般用来滤掉杂波；

## 1.4 开发板 + Debugger转接板

1. Debug Tool 脚座；
2. 供电可以有两个来源：
	1. Debug Tool --> 从PC的USB过来，但是电流只有0.5A，超过这个电流的话Debug Tool就不可以了；
	2. 外部电源：Vcc/Gnd
3. 转接板：为了配合J-Link，脚位重新配，将Debug Tool的14pin转为符合J-Link的20Pin；


## Section 2. 硬件开发系统 Debug tool（J-LINK）

### 2.1 Debug Tool Ⅰ

> 具有下载程序，调试加断点等功能

早期工具：
- ICE ： In-Circuit Emulator 线上仿真器，提供嵌入式系统软件debug的硬件装置；
- 模拟处理器的动作，通过排针连接在处理器的脚座上；
- 可以由设计者操控执行处理器可以的操作，完全模拟处理器的动作；
- 由于速度和复杂度，近代ICE跟不上了；

近代Debug 工具：
- OCD：On-chip Debugger，内建于处理器中的debug线路，便于开发者开发和调试；
- Debug Tool：将OCD讯息传入电脑的开发环境中；或者PC--> HW
- 功能：
	- 将程序下载到处理器中；
	- 程序执行时，追踪寄存器和数据变化；
	- 使用中断点Break Point、单步执行Step等功能对程序debug
- 瑞萨提供的Debug Tool：
	- E1/E20 Debug Emulator
	- SEGGER公司的J-Link

### 2.2 Debug Tool Ⅱ 安装和使用

安装 J-Link 驱动：
1. 先安装通用版本驱动器，J-Link主控台CMD；
2. 安装瑞萨RX版本的驱动；

J-Link操作准备：
1. 点击 SEGGER -> J-Link ARM V4.34d -> **J-Link Commander**
2. 将J-Link连接USB到PC上；

J-Link Commander 操作：
1. `?` 可以查看指令；
2. 本课程使用的指令为：
	1. `POWER ON` 通过J-Link提供电源给处理器；（注意PC供电不会超过0.5A，如果需要超过，需要使用外部电源）
	2. `POWER OFF` 关闭J-Link提供的电源；
3. 其余指令在IDE中，使用图形化界面即可操作；

J-Link 重置：
1. 可能会出现J-Link硬件出现问题不能连线的状况；
2. 可以重新烧录J-Link固件


## Section 3. IDE介绍

通常完整IDE包括：
- 程序编辑器；
- 编译器、汇编器；
- 程序下载和烧录功能；
- Debug功能；


## Section 4. 瑞萨专用IDE HEW安装与使用

- HEW version 4.08
- http://goo.g;/PdLoZa
- 安装路径建议使用预设地址
- <mark>软件安装顺序也是需要注意的！</mark>


## Section 5. 程序码产生器 Peripheral Driver Generator 2（PDG2）

- 用来对外围设备的设置修改进行图形化自动生成代码,省去了查找datasheet的时间,不易出错;
- 勾选使用的外围设备,会产生专用的 `.c` 和 `.h` 文件;
	- `.c` 文件包含: 函数的说明和函数的实现;
	- `.h` 文件: 引入RPDL和函数声明;
- 当需要直接存取寄存器,可以在主程序中加入 `iodefine.h` 


## Section 6. PDG2安装与操作说明



## Section 7. 安装影片





[[2022-11-08_星期二]]
