> 原始链接：[🔗](https://www.cnblogs.com/jiangyiming/p/15790845.html)


## 1. 什么是端口复用？

STM32有很多的**内置外设**(把一些功能ADC、看门狗…集中到芯片里面)，<u>这些内置外设的外部引脚都是与GPIO复用的</u>。

<mark>也就是说，一个GPIO如果可以复用为内置外设的功能引脚，那么当这个GPIO作为内置外设使用的时候，就叫做复用。</mark>

> 例如串口1 的发送接收引脚是PA9,PA10，当我们把PA9,PA10不用作GPIO，而用做复用功能串口1的发送接收引脚的时候，叫端口复用。


## 2. STM32引脚可以复用为哪些功能？

> 可在芯片STM32767IGT6资料中的pin and ball definitions中找到：
>  ![](https://raw.githubusercontent.com/DustOfStars/ObsPicGo/master/Gavin_Obs/20221111104138.png)


### STM32(M4内核以上)的端口复用映射原理

STM32系列微控制器***IO引脚通过一个复用器连接到内置外设或模块***。**该复用器一次只允许一个外设的复用功能（AF）连接到对应的IO口。这样可以确保共用同一个IO引脚的外设之间不会发生冲突。** 

--> <mark>也就是说，每个IO引脚连接一个多路选择器，就形成了端口复用！！</mark>

每个IO引脚都有一个复用器，该复用器采用16路复用功能输入（AF0到AF15），**可通过GPIOx_AFRL(针对引脚0-7)和GPIOx_AFRH（针对引脚8-15）寄存器对这些输入进行配置，每四位控制一路复用。**

![](https://raw.githubusercontent.com/DustOfStars/ObsPicGo/master/Gavin_Obs/20221111104527.png)

对于每一组GPIO都有GPIO复用功能，
> 比如GPIOA有16个IO口，
> GPIO 复用功能低位寄存器 (GPIOx_AFRL) (x = A…K):是32位，可以配置引脚0-7。
> GPIO 复用功能高位寄存器 (GPIOx_AFRH) (x = A…J)：

`GPIOx_AFRL`寄存器：

![](https://raw.githubusercontent.com/DustOfStars/ObsPicGo/master/Gavin_Obs/20221111110551.png)


`GPIOx_AFRH` 寄存器：

![](https://raw.githubusercontent.com/DustOfStars/ObsPicGo/master/Gavin_Obs/20221111111739.png)
> **比如配置PA9连接到AF7，那么就是配置GPIOA_AFRH的AFR9，配置成0111**
> GPIOA的Pin9，选择端口为0111，得到AF7选通！！


## 3. 复用功能配置：

1. 系统功能：将I/O连接到AF0，然后根据所用功能进行配置：
	-   JTAG/SWD:在各器件复位之后，会将这些引脚指定为专用引脚，可供片上调试模块立即使用 （不受GPIO控制器控制）
	-   RTC_REFIN：此引脚应配置为输入浮空模式。
	-   MCO1和MCO2：这些引脚必须配置为复用功能模式。
2. GPIO：在GPIOx_MODER寄存器中将所需I/O配置为输出或输入。
3. 外设复用功能：
	对于ADC和DAC，在GPIOx_MODER寄存器中将所需I/O配置为**模拟通道**。
	对于其它外设：
	-   在GPIOx_MODER寄存器中将所需I/O配置为**复用功能**
	-   通过GPIOx_OTYPER、GPIOx_PUPDR和GPIOx_OSPEEDER寄存器，分别配置类型、上拉/下拉以及输出速度。
	-   在GPIOx_AFRL或GPIOx_AFRH寄存器中，将I/O连接到所需AFx


## 4. 端口复用功能配置过程

以PA9、PA10配置为串口1为例：

1. GPIO端口的时钟使能。  
	`__HAL_RCC_GPIOA_CLK_ENABLE(); //使能GPIO时钟`
2. 复用外设的时钟使能。  
	1. 比如你要将端口PA9,PA10复用为串口，所以要使能串口时钟
		`__HAL_RCC_USART1_CLK_ENABLE(); //使能串口1时钟`
3. 端口模式配置为**复用功能**。 `HAL_GPIO_Init`函数。  
	`GPIO_Initure.Mode=GPIO_MODE_AF_PP; //复用推挽输出`
4. 配置GPIOx_AFRL或者GPIOx_AFRH寄存器，将IO连接到所需的AFx。HAL_GPIO_Init函数。`GPIO_Initure.Alternate=GPIO_AF7_USART1;//复用为USART1`


<mark>最最**重要**！！！</mark>
在文件system-usart.c中可以找到：`HAL_UART_MspInit` :
> **HAL_UART_MspInit函数会被HAL_UART_Init函数调用**

```c
void HAL_UART_MspInit(UART_HandleTypeDef *huart)
{
    //GPIO端口设置
	GPIO_InitTypeDef GPIO_Initure;
	
	if(huart->Instance==USART1)//如果是串口1，进行串口1 MSP初始化
	{
		__HAL_RCC_GPIOA_CLK_ENABLE();			//使能GPIOA时钟
		__HAL_RCC_USART1_CLK_ENABLE();			//使能USART1时钟
	
		GPIO_Initure.Pin=GPIO_PIN_9;			//PA9
		GPIO_Initure.Mode=GPIO_MODE_AF_PP;		//复用推挽输出
		GPIO_Initure.Pull=GPIO_PULLUP;			//上拉
		GPIO_Initure.Speed=GPIO_SPEED_FAST;		//高速
		GPIO_Initure.Alternate=GPIO_AF7_USART1;	//复用为USART1
		HAL_GPIO_Init(GPIOA,&GPIO_Initure);	   	//初始化PA9

		GPIO_Initure.Pin=GPIO_PIN_10;			//PA10
		HAL_GPIO_Init(GPIOA,&GPIO_Initure);	   	//初始化PA10
		
#if EN_USART1_RX
		HAL_NVIC_EnableIRQ(USART1_IRQn);				//使能USART1中断通道
		HAL_NVIC_SetPriority(USART1_IRQn,3,3);			//抢占优先级3，子优先级3
#endif	
	}

}
```




## 其他前置结构体说明：

- GPIO_InitTypeDef结构体为：

（Pin为引脚、Mode为输入输出模式、Pull为上/下拉、Speed为速度、Alternate为功能）:

```c
  1 typedef struct
  2 {
  3   uint32_t Pin;       /*!< Specifies the GPIO pins to be configured.
  4                            This parameter can be any value of @ref GPIO_pins_define */
  5 
  6   uint32_t Mode;      /*!< Specifies the operating mode for the selected pins.
  7                            This parameter can be a value of @ref GPIO_mode_define */
  8 
  9   uint32_t Pull;      /*!< Specifies the Pull-up or Pull-Down activation for the selected pins.
 10                            This parameter can be a value of @ref GPIO_pull_define */
 11 
 12   uint32_t Speed;     /*!< Specifies the speed for the selected pins.
 13                            This parameter can be a value of @ref GPIO_speed_define */
 14 
 15   uint32_t Alternate;  /*!< Peripheral to be connected to the selected pins.
 16                             This parameter can be a value of @ref GPIO_Alternate_function_selection */
 17 }GPIO_InitTypeDef;
```

- **AF（0-15）功能选择**

```c
  1 /**
  2   * @brief   AF 0 selection
  3   */
  4 #define GPIO_AF0_RTC_50Hz      ((uint8_t)0x00U)  /* RTC_50Hz Alternate Function mapping                       */
  5 #define GPIO_AF0_MCO           ((uint8_t)0x00U)  /* MCO (MCO1 and MCO2) Alternate Function mapping            */
  6 #define GPIO_AF0_SWJ           ((uint8_t)0x00U)  /* SWJ (SWD and JTAG) Alternate Function mapping             */
  7 #define GPIO_AF0_TRACE         ((uint8_t)0x00U)  /* TRACE Alternate Function mapping                          */
  8 
  9 /**
 10   * @brief   AF 1 selection
 11   */
 12 #define GPIO_AF1_TIM1          ((uint8_t)0x01U)  /* TIM1 Alternate Function mapping */
 13 #define GPIO_AF1_TIM2          ((uint8_t)0x01U)  /* TIM2 Alternate Function mapping */
 14 #if defined (STM32F765xx) || defined(STM32F767xx) || defined(STM32F769xx) || defined(STM32F777xx) || defined(STM32F779xx)
 15 #define GPIO_AF1_UART5         ((uint8_t)0x01U)  /* UART5 Alternate Function mapping */
 16 #define GPIO_AF1_I2C4          ((uint8_t)0x01U)  /* I2C4 Alternate Function mapping  */
 17 #endif /* STM32F767xx || STM32F769xx || STM32F777xx || STM32F779xx */
 18 
 19 /**
 20   * @brief   AF 2 selection
 21   */
 22 #define GPIO_AF2_TIM3          ((uint8_t)0x02U)  /* TIM3 Alternate Function mapping */
 23 #define GPIO_AF2_TIM4          ((uint8_t)0x02U)  /* TIM4 Alternate Function mapping */
 24 #define GPIO_AF2_TIM5          ((uint8_t)0x02U)  /* TIM5 Alternate Function mapping */
 25 
 26 /**
 27   * @brief   AF 3 selection
 28   */
 29 #define GPIO_AF3_TIM8          ((uint8_t)0x03U)  /* TIM8 Alternate Function mapping  */
 30 #define GPIO_AF3_TIM9          ((uint8_t)0x03U)  /* TIM9 Alternate Function mapping  */
 31 #define GPIO_AF3_TIM10         ((uint8_t)0x03U)  /* TIM10 Alternate Function mapping */
 32 #define GPIO_AF3_TIM11         ((uint8_t)0x03U)  /* TIM11 Alternate Function mapping */
 33 #define GPIO_AF3_LPTIM1        ((uint8_t)0x03U)  /* LPTIM1 Alternate Function mapping */
 34 #define GPIO_AF3_CEC           ((uint8_t)0x03U)  /* CEC Alternate Function mapping */
 35 #if defined (STM32F765xx) || defined(STM32F767xx) || defined(STM32F769xx) || defined(STM32F777xx) || defined(STM32F779xx)
 36 #define GPIO_AF3_DFSDM1         ((uint8_t)0x03U)  /* DFSDM1 Alternate Function mapping */
 37 #endif /* STM32F767xx || STM32F769xx || STM32F777xx || STM32F779xx */
 38 /**
 39   * @brief   AF 4 selection
 40   */
 41 #define GPIO_AF4_I2C1          ((uint8_t)0x04U)  /* I2C1 Alternate Function mapping */
 42 #define GPIO_AF4_I2C2          ((uint8_t)0x04U)  /* I2C2 Alternate Function mapping */
 43 #define GPIO_AF4_I2C3          ((uint8_t)0x04U)  /* I2C3 Alternate Function mapping */
 44 #define GPIO_AF4_I2C4          ((uint8_t)0x04U)  /* I2C4 Alternate Function mapping */
 45 #define GPIO_AF4_CEC           ((uint8_t)0x04U)  /* CEC Alternate Function mapping */
 46 #if defined (STM32F765xx) || defined(STM32F767xx) || defined(STM32F769xx) || defined(STM32F777xx) || defined(STM32F779xx)
 47 #define GPIO_AF4_USART1        ((uint8_t)0x04)  /* USART1 Alternate Function mapping */
 48 #endif /* STM32F767xx || STM32F769xx || STM32F777xx || STM32F779xx */
 49 
 50 /**
 51   * @brief   AF 5 selection
 52   */
 53 #define GPIO_AF5_SPI1          ((uint8_t)0x05U)  /* SPI1 Alternate Function mapping        */
 54 #define GPIO_AF5_SPI2          ((uint8_t)0x05U)  /* SPI2/I2S2 Alternate Function mapping   */
 55 #define GPIO_AF5_SPI3          ((uint8_t)0x05U)  /* SPI3/I2S3 Alternate Function mapping   */
 56 #define GPIO_AF5_SPI4          ((uint8_t)0x05U)  /* SPI4 Alternate Function mapping        */
 57 #define GPIO_AF5_SPI5          ((uint8_t)0x05U)  /* SPI5 Alternate Function mapping        */
 58 #define GPIO_AF5_SPI6          ((uint8_t)0x05U)  /* SPI6 Alternate Function mapping        */
 59 
 60 /**
 61   * @brief   AF 6 selection
 62   */
 63 #define GPIO_AF6_SPI3          ((uint8_t)0x06U)  /* SPI3/I2S3 Alternate Function mapping  */
 64 #define GPIO_AF6_SAI1          ((uint8_t)0x06U)  /* SAI1 Alternate Function mapping       */
 65 #if defined (STM32F765xx) || defined(STM32F767xx) || defined(STM32F769xx) || defined(STM32F777xx) || defined(STM32F779xx)
 66 #define GPIO_AF6_UART4         ((uint8_t)0x06U)   /* UART4 Alternate Function mapping     */
 67 #define GPIO_AF6_DFSDM1        ((uint8_t)0x06U)  /* DFSDM1 Alternate Function mapping     */
 68 #endif /* STM32F767xx || STM32F769xx || STM32F777xx || STM32F779xx */
 69 
 70 /**
 71   * @brief   AF 7 selection
 72   */
 73 #define GPIO_AF7_USART1        ((uint8_t)0x07U)  /* USART1 Alternate Function mapping     */
 74 #define GPIO_AF7_USART2        ((uint8_t)0x07U)  /* USART2 Alternate Function mapping     */
 75 #define GPIO_AF7_USART3        ((uint8_t)0x07U)  /* USART3 Alternate Function mapping     */
 76 #define GPIO_AF7_UART5         ((uint8_t)0x07U)  /* UART5 Alternate Function mapping      */
 77 #define GPIO_AF7_SPDIFRX       ((uint8_t)0x07U)  /* SPDIF-RX Alternate Function mapping   */
 78 #define GPIO_AF7_SPI2          ((uint8_t)0x07U)  /* SPI2 Alternate Function mapping       */
 79 #define GPIO_AF7_SPI3          ((uint8_t)0x07U)  /* SPI3 Alternate Function mapping       */
 80 #if defined (STM32F765xx) || defined(STM32F767xx) || defined(STM32F769xx) || defined(STM32F777xx) || defined(STM32F779xx)
 81 #define GPIO_AF7_SPI6          ((uint8_t)0x07U)  /* SPI6 Alternate Function mapping       */
 82 #define GPIO_AF7_DFSDM1         ((uint8_t)0x07U) /* DFSDM1 Alternate Function mapping      */
 83 #endif /* STM32F767xx || STM32F769xx || STM32F777xx || STM32F779xx */
 84 
 85 /**
 86   * @brief   AF 8 selection
 87   */
 88 #define GPIO_AF8_UART4         ((uint8_t)0x08U)  /* UART4 Alternate Function mapping  */
 89 #define GPIO_AF8_UART5         ((uint8_t)0x08U)  /* UART5 Alternate Function mapping  */
 90 #define GPIO_AF8_USART6        ((uint8_t)0x08U)  /* USART6 Alternate Function mapping */
 91 #define GPIO_AF8_UART7         ((uint8_t)0x08U)  /* UART7 Alternate Function mapping  */
 92 #define GPIO_AF8_UART8         ((uint8_t)0x08U)  /* UART8 Alternate Function mapping  */
 93 #define GPIO_AF8_SPDIFRX       ((uint8_t)0x08U)  /* SPIDIF-RX Alternate Function mapping */
 94 #define GPIO_AF8_SAI2          ((uint8_t)0x08U)  /* SAI2 Alternate Function mapping   */
 95 #if defined (STM32F765xx) || defined(STM32F767xx) || defined(STM32F769xx) || defined(STM32F777xx) || defined(STM32F779xx)
 96 #define GPIO_AF8_SPI6          ((uint8_t)0x08U)  /* SPI6 Alternate Function mapping   */
 97 #endif /* STM32F767xx || STM32F769xx || STM32F777xx || STM32F779xx */
 98 
 99 
100 /**
101   * @brief   AF 9 selection
102   */
103 #define GPIO_AF9_CAN1          ((uint8_t)0x09U)  /* CAN1 Alternate Function mapping    */
104 #define GPIO_AF9_CAN2          ((uint8_t)0x09U)  /* CAN2 Alternate Function mapping    */
105 #define GPIO_AF9_TIM12         ((uint8_t)0x09U)  /* TIM12 Alternate Function mapping   */
106 #define GPIO_AF9_TIM13         ((uint8_t)0x09U)  /* TIM13 Alternate Function mapping   */
107 #define GPIO_AF9_TIM14         ((uint8_t)0x09U)  /* TIM14 Alternate Function mapping   */
108 #define GPIO_AF9_QUADSPI       ((uint8_t)0x09U)  /* QUADSPI Alternate Function mapping */
109 #if defined(STM32F746xx) || defined(STM32F756xx) || defined(STM32F767xx) || defined(STM32F769xx) || defined(STM32F777xx) || defined(STM32F779xx)
110 #define GPIO_AF9_LTDC          ((uint8_t)0x09U)  /* LCD-TFT Alternate Function mapping */
111 #endif /* STM32F746xx || STM32F756xx || STM32F767xx || STM32F769xx || STM32F777xx || STM32F779xx */
112 #if defined(STM32F746xx) || defined(STM32F756xx) || defined(STM32F765xx) || defined(STM32F765xx) || defined(STM32F767xx) || defined(STM32F769xx) || defined(STM32F777xx) || defined(STM32F779xx)
113 #define GPIO_AF9_FMC           ((uint8_t)0x09U)   /* FMC Alternate Function mapping     */
114 #endif /* STM32F746xx || STM32F756xx || STM32F767xx || STM32F769xx || STM32F777xx || STM32F779xx */
115 /**
116   * @brief   AF 10 selection
117   */
118 #define GPIO_AF10_OTG_FS        ((uint8_t)0xAU)  /* OTG_FS Alternate Function mapping */
119 #define GPIO_AF10_OTG_HS        ((uint8_t)0xAU)  /* OTG_HS Alternate Function mapping */
120 #define GPIO_AF10_QUADSPI       ((uint8_t)0xAU)  /* QUADSPI Alternate Function mapping */
121 #define GPIO_AF10_SAI2          ((uint8_t)0xAU)  /* SAI2 Alternate Function mapping */
122 #if defined (STM32F765xx) || defined(STM32F767xx) || defined(STM32F769xx) || defined(STM32F777xx) || defined(STM32F779xx)
123 #define GPIO_AF10_DFSDM1         ((uint8_t)0x0AU)  /* DFSDM1 Alternate Function mapping  */
124 #define GPIO_AF10_SDMMC2        ((uint8_t)0x0AU)  /* SDMMC2 Alternate Function mapping */
125 #endif /* STM32F767xx || STM32F769xx || STM32F777xx || STM32F779xx */
126 
127 /**
128   * @brief   AF 11 selection
129   */
130 #define GPIO_AF11_ETH           ((uint8_t)0x0BU)  /* ETHERNET Alternate Function mapping */
131 #if defined(STM32F765xx) || defined (STM32F767xx) || defined (STM32F769xx) || defined (STM32F777xx) || defined (STM32F779xx)
132 #define GPIO_AF11_CAN3          ((uint8_t)0x0BU)  /* CAN3 Alternate Function mapping     */
133 #define GPIO_AF11_SDMMC2        ((uint8_t)0x0BU)  /* SDMMC2 Alternate Function mapping   */
134 #define GPIO_AF11_I2C4          ((uint8_t)0x0BU)  /* I2C4 Alternate Function mapping     */
135 #endif /* STM32F767xx || STM32F769xx || STM32F777xx || STM32F779xx */
136 
137 /**
138   * @brief   AF 12 selection
139   */
140 #define GPIO_AF12_FMC           ((uint8_t)0xCU)  /* FMC Alternate Function mapping                      */
141 #define GPIO_AF12_OTG_HS_FS     ((uint8_t)0xCU)  /* OTG HS configured in FS, Alternate Function mapping */
142 #define GPIO_AF12_SDMMC1        ((uint8_t)0xCU)  /* SDMMC1 Alternate Function mapping                   */
143 #if defined(STM32F765xx) || defined (STM32F767xx) || defined (STM32F769xx) || defined (STM32F777xx) || defined (STM32F779xx)
144 #define GPIO_AF12_MDIOS        ((uint8_t)0xCU)  /* SDMMC1 Alternate Function mapping                    */
145 #define GPIO_AF12_UART7        ((uint8_t)0xCU)  /* UART7 Alternate Function mapping                     */
146 #endif /* STM32F767xx || STM32F769xx || STM32F777xx || STM32F779xx */
147 
148 /**
149   * @brief   AF 13 selection
150   */
151 #define GPIO_AF13_DCMI          ((uint8_t)0x0DU)  /* DCMI Alternate Function mapping */
152 #if defined (STM32F769xx) || defined (STM32F779xx)
153 #define GPIO_AF13_DSI           ((uint8_t)0x0DU)  /* DSI Alternate Function mapping  */
154 #endif /* STM32F767xx || STM32F769xx || STM32F777xx || STM32F779xx */
155 #if defined(STM32F746xx) || defined(STM32F756xx) || defined(STM32F767xx) || defined(STM32F769xx) || defined(STM32F777xx) || defined(STM32F779xx)
156 #define GPIO_AF13_LTDC          ((uint8_t)0x0DU)  /* LTDC Alternate Function mapping */
157 
158 /**
159   * @brief   AF 14 selection
160   */
161 #define GPIO_AF14_LTDC          ((uint8_t)0x0EU)  /* LCD-TFT Alternate Function mapping */
162 #endif /* STM32F746xx || STM32F756xx || STM32F767xx || STM32F769xx || STM32F777xx || STM32F779xx */
163 /**
164   * @brief   AF 15 selection
165   */
166 #define GPIO_AF15_EVENTOUT      ((uint8_t)0x0FU)  /* EVENTOUT Alternate Function mapping */
```






[[2022-11-11_星期五]]
