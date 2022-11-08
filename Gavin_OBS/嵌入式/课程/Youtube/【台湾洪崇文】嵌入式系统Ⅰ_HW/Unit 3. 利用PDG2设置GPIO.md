## Section 1. 通用型输入输出GPIO

 什么是GPIO?
- 可由软件控制接脚成为数位输入接脚或者数位输出接脚;
- Input可以用来读取外部输入的高低电平;
- Output可以输出高低电平给外围设备;
- 高低电平由系统的电压决定;

RX62T的GPIO:
- 大多数对外脚位兼具不同功能,功能由处理器内部控制寄存器选择来设置;
	- 数位输入或输出(GPIO);
	- 周边输入输出
	- 类比输入
	- 中断输入

GPIO电路:
- 输出与输入可以使用PDG2设定电平与读取电平;
- 输入电平可以由PORT寄存器去读取;
- <mark>输出电平由DR(Data Register)数据寄存器决定</mark>;
- <u>设置DDR(Data Direction Register)数据方向寄存器中的相应bit,决定接脚为输入还是输出</u>;

输出设置:
- 开启PDG2设置完时钟之后,在下方外围页面的部分,选择"I/O"
- **因为LED外围设备是装在P30--P33,所以使用Port3 Pn0~Pn3**;
- LED对应的Port3 的 Pn0到Pn3 Direction选择为**Output**;


## Section 2. 建立HEW的一个project和硬件连接


### 硬件连接
- 无需其他部件和连线,使用开发板上的LED;
- 不需要外部电源, 使用PC电源; ( 注意GPIO的电流驱动能力 )
- 一种改变并联LED数量的方法,使用GPIO做栅极电压控制开漏, 不用担心GPIO驱动能力不足了

![](https://raw.githubusercontent.com/DustOfStars/ObsPicGo/master/Gavin_Obs/20221108172648.png)


### LED动作逻辑:
- 设计为**Low Active**, 当GPIO输出为low时,电流由Vcc流经LED流向Gnd, LED发光
- GPIO为High时, LED两边都是高电平, 没有电流, LED不亮;


## Section 3.  HandsOn 1 -- LED明灭控制

Steps:
1. 从HEW新建工程;
2. PDG2 设定周边然后汇入程序码;
3. 撰写程序代码;
4. 解说程序;
5. 结果展示;






[[2022-11-08_星期二]]

