
> 大纲：
> 1. 终端机功能
> 2. ASCII码与字符转换
> 3. 系统规划和效率分析
> 4. 程序解析
> 5. 实验结果


## Section 1. 终端机功能


## Section 2. ASCII码与字符转换

![ASCII表](https://raw.githubusercontent.com/DustOfStars/ObsPicGo/master/Gavin_Obs/20221116135841.png)

> 每个位数转换为ASCII的方法，是 加上0x30，因为从48开始，表示‘0’；
> ch = x + '0'
> ch = x + 48

- 大写字母从65开始；
- 小写字母从97开始
- 因为16进制的A是10进制的10，所以，0xa要先减去10，再加上‘A'或者’a'；
- 这样16进制a变成了ASCII的A；


### 2.1 字符转换示例--10进制

把十进制的1024转为一个`char[]`表示每个位的真实ASCII数字的字符串；

```c
int a = 1024;
char txt[5];

txt[0] = (a / 1000) + 0x30;
txt[1] = ((a/100) % 10) + 48;
txt[2] = ((a/10) % 10) + '0';
txt[3] = (a % 10) + 0x30;
txt[4] = '\0';
```


### 2.2 字符转换示例--16进制

把a的数值转换为16进制字符串；

```c
txt[0] = (a >> 12) & 0x0f;    // 右移12位，只剩下最高的4位了，与上1111，即保持原始值
if(txt[0] >= 10) {
	txt[0] = txt[0] - 10 + 'a';
}
else {
	txt[0] += '0';
}    // 因为a和9的ASCII不连续，所以做了分段处理
```


### 2.3 使用sprintf函数

- 要inculd `string.h` 头文件；
- sprintf函数的格式： `sprintf(目标字符串, '格式10 or 16', 变量)`

```c
int a = 1024;
char txt[10];
sprintf(txt, "%4d  %4x", a, a);    // d是十进制接收， x是十六进制接收
```

> 中文字符使用Unicode存储，一个中文字符占用两个byte；


## Section 3. 系统规划和效率分析

### 3.1 系统输入与输出

输入：
- 由ADC读取开发板上可变电阻的分压电压；
- 在PC终端软件输入触发采样的信号，通过RS-232经过UART通知处理器采样；

输出：
- 处理器经过UART通过RS-232传值给PC，在终端显示，包括ADC采样的值和转换后的电压值；

### 3.2 系统操作

- 规定A或者a来作为采样触发信号；
- 采样完成ADC转换为电压值，存放到另一个变量，把ADC转换的结果和电压值传回PC终端显示；
- 规定m或者M，处理器传回前面十次的平均值；

### 3.3 外围设备

- ADC设为单次采样，软件触发；
- 解析度设为12bit；
- UART：9600 8 n 1
- ADC要设置转换完成中断；

### 3.4 比较转换字符串的效率

- 使用两种方式进行变量内容转为字符串；
- 开发系统提供了Performance Analysis功能，可以比较两种方式的转换速度；
- 速度不同以外，程式码大小也有不同，观察IDE的Map档比较大小；


## Section 4. 程序代码解析

![](https://raw.githubusercontent.com/DustOfStars/ObsPicGo/master/Gavin_Obs/20221116144438.png)

![](https://raw.githubusercontent.com/DustOfStars/ObsPicGo/master/Gavin_Obs/20221116144657.png)

![](https://raw.githubusercontent.com/DustOfStars/ObsPicGo/master/Gavin_Obs/20221116144541.png)

![](https://raw.githubusercontent.com/DustOfStars/ObsPicGo/master/Gavin_Obs/20221116144830.png)

[[2022-11-16_星期三]]




