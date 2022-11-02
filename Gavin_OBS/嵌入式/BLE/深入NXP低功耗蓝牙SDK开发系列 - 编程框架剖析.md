> [原始链接](https://xiongyu0523.github.io/2018/09/30/%E6%B7%B1%E5%85%A5NXP%E4%BD%8E%E5%8A%9F%E8%80%97%E8%93%9D%E7%89%99SDK%E5%BC%80%E5%8F%91%E7%B3%BB%E5%88%97-%E7%BC%96%E7%A8%8B%E6%A1%86%E6%9E%B6%E5%89%96%E6%9E%90/)


## 前言

讲解NXP的低功耗蓝牙SDK开发的方方面面，目前NXP官方仅有一个英文版的《BLE Application Developer Guide》介绍SDK的使用，对于国内开发者来说不太友好。希望这个系列文章能作为官方文档的补充，帮助到正在使用和将要使用NXP蓝牙的芯片的朋友们。

假设用户已经具备了基本的BLE概念，如Profile、GAP、GATT、HCI、LL等**各个层次的职责，广播与连接的行为**，基于GATT数据传输的方式等。如果读者对这个概念还不熟悉，可以阅读《Getting Started with Bluetooth Low Energy》一书的第1至3章共75页内容，利用业余时间在2-3天内可以轻松完成。


## NXP BLE SoC概况

目前NXP面向物联网和可穿戴设备数据传输应用的BLE SoC芯片要有QN和KW两大系列（型号特点见下表），除了QN902x使用原Quintic自带的协议栈SDK外，其他几个型号都采用了NXP MCUXpresso SDK和NXP自研的Bluetooth协议栈，呈现给用户的编程接口完全一致。*无论是开发超低功耗的QN9080，支持Zigbee/Thread/BLE多模的KW41，还是面向汽车应用的KW36，对固件工程师来说，掌握了一款芯片的开发，拿下其他的也就是轻而易举之事*。


| 型号      | 协议版本     | 内核频率                            | 存储器资源                               | 特色      |
|---------|----------|---------------------------------|-------------------------------------|---------|
| **QN902x**  | BLE 4.2  | Cortex-M0 @32MHz                | 128KB Flash / 64KB RAM / 128KB ROM  | 低成本     |
| **QN908x**  | BLE 5.0  | Cortex-M4F @32MHz               | 512KB Flash / 128KB RAM             | 超低功耗    |
| **KW41**    | BLE 4.2  | Cortex-M0+ @48MHz               | 512KB Flash / 128KB RAM             | 多模多协议   |
| **KW36**    | BLE 5.0  | Cortex-M0+ @48MHz               | 512KB Flash / 64KB RAM              | 汽车网络    |
| **K32W**    | BLE 5.0  | Cortex-M4F & Cortex-M0+ @72MHz  | 1.25MB Flash / 384KB RAM            | 单芯片高集成度 |



## MCUXpresso SDK与NXP BLE协议栈

MCUXpresso SDK是NXP面向MCU市场推出的一套完整的软件开发套件，***支持Kinetis, LPC, i.MX RT等通用微控制器以及QN和JN系列的无线连接微控制器***。一个SDK包内包含了**芯片硬件抽象层，外设驱动库，多个中间件库**（根据芯片功能的不同包含不同的中间件，比如带Ethernet外设的MCU则会有lwip网络协议栈中间件），以及丰富的示例代码。

对于低功耗蓝牙MCU，它的BLE协议栈作为MCUXpresso SDK的一个中间件而存在。在用户解压缩SDK包后，在`./middleware/wireless/bluetooth_x.y.z`下可以找到协议栈的代码和库文件。<u>低功耗蓝牙应用将包含这个目录下的文件，以使用协议栈提供的API和服务。</u>所有低功耗蓝牙的示例代码可以在 `./boards/wireless_examples/bluetooth` 目录下找到。同时，低功耗蓝牙应用与通用MCU SDK还有一点差异，它需要一些额外的通用组件作为支撑，**如OS环境抽象、非易失存储、队列管理，低功耗管理等，因此多引入了一个connectivity framework的中间件层**。在 `./middleware/wireless/framework_x.y.z` 下可以找到framework包含的所有功能模块源代码。下图展示了SDK的文件结构中与低功耗蓝牙相关的目录。

![MCUXpresso SDK文件目录组织](https://raw.githubusercontent.com/DustOfStars/ObsPicGo/master/Gavin_Obs/20221102151836.png)

> 如果基于MCUXpresso IDE开发，通常会导出某个示例工程到workspace目录下，被导出的工程与解压缩后的SDK的layout稍有不同，它**仅包含了该示例代码所需的驱动和中间件**，并以一个平铺的方式组织文件夹，但文件夹下的内容并无区别。


## NXP BLE SDK系统架构

NXP提供了一套完整的BLE协议栈和编程框架，并已通过了Bluetooth Core Spec 5.0的认证。用户第一次开发自己的应用时，应当先大致了解整个系统的组成，知道每个部分的职责，分别有哪些文件。有了一些基础，再通过查询API文档，很快便能掌握BLE应用开发。下图为BLE SDK总体的框架图，图中使用了几种不同的颜色来区分：

*   应用（灰）
*   低功耗蓝牙服务框架（黄）
*   Connectivtiy Framework（绿）
*   BLE协议栈（蓝）
*   通用SDK（白）

![NXP BLE SDK系统架构](https://raw.githubusercontent.com/DustOfStars/ObsPicGo/master/Gavin_Obs/20221102152452.png)  


## OSA

OSA属于***Connectivity Framework***，单独介绍它是<u>因为OSA服务于整个SDK和协议栈，并且也是系统复位后的入口</u>。整个蓝牙协议栈和SDK的设计使用了RTOS下才有的多任务，消息和事件等服务。***为了减少代码大小和栈的内存，适配一些资源紧张的SoC，在设计时才引入了OSA***。

OSA提供一套接口封装，底层可以使用FreeRTOS或者Bare Metal前后台两种实现。
* 如果底层为FreeRTOS，OSA只是简单的将FreeRTOS API转换为OSA API。
* 如果底层底层为Bare Metal系统，OSA则内部实现了一套多任务、信号量、事件等RTOS API的模拟实现，当用户调用`OSA_Start`启动多任务调度环境时，实际上是陷入了一个`while(1)`的循环，在这个循环中会<u>不断的检查任务队列的状态并选择优先级最高的任务来运行，这些任务之间无法抢占，因此每个任务的设计需要额外小心，占用CPU的时间最好不要超过2ms，以确保处理BLE事件的任务能及时得到调度</u>。

在编写OSA的任务代码时，有几个不同与一般RTOS的地方需要注意。
* 任务体`while(1)`循环的最后需要加上一个条件判断，如果是`gUseRtos_c == 0`，则直接跳出循环，相当于每次任务只执行一次。
* 任务体`while(1)`循环之前的初始化工作需要特殊化处理，定义一个静态或者全局变量`initailized`的，并对它进行测试，确保在当OSA运行在`Bare Metal`配置时，这些初始化代码仅会运行一次。
* `OSA_EventWait`, `OSA_SemphoreWait`,  `OSA_MutexLock`等函数在OSA的FreeRTOS实现时会阻塞任务，而在Bare Metal实现中，**调用这些API不会阻塞**，而是通过内部一个状态机来检测资源，如果资源目前是不可获得状态，则直接返回。
* `OSA_TimeDelay`函数在BM实现时是不会切换到其他任务运行的，而是在原地等待直到超时。

OSA的源码可以在实现 `./middleware/wireless/framework_x.y.z/OSAbstraction`下找到，如感兴趣可以深入研读。

> SDK提供的每一份BLE示例代码中都提供了freertos和bm两个工程。


## Connectivity Framework

Connectivity Framework包含了众多的子模块，服务于应用和BLE协议栈。本系列文章将会抽取其中几个比较重要的几个子模块作为单独的文章来分析。本文希望先给读者一个总体概念，通过表格列出了各个模块的职责和必要性。标注为“必要”的模块如果缺失，最后整个BLE工程构建将会失败，说明该模块已被协议栈或者其他的代码引用了。标注为“非必要”的模块目的是服务应用层，如果用户的应用不需要这个功能，则可以移除。

| 子模块                | 职责                    | 必要性  | 说明                           |
|--------------------|-----------------------|------|------------------------------|
| FunctionLib        | memcpy等标准函数的封装        | 必要   | 通过头文件配置选择私有实现或直接使用C库         |
| List               | 通用双向链表实现              | 必要   | List是MemManager和Messaging的基础 |
| MemManager         | 基于块的内存分配              | 必要   | 协议栈所需的动态内存都从这里分配             |
| Messaging          | 消息队列服务                | 必要   | 不具备任务同步功能，需配合OSA Event使用     |
| OSAbstraction      | 提供操作系统服务抽象            | 必要   | 整个系统调度都依赖于OSA                |
| TimersManager      | 提供软定时服务               | 必要   | 协议栈超时机制需要使用到                 |
| SecLib             | 封装SMP加密操作             | 必要   | 部分芯片可以选择使用硬件加密加速引擎           |
| LowPower           | 实现低功耗管理               | 非必要  | 仅当系统需要使用低功耗时用到               |
| OtaSupport         | 提供OTA服务               | 非必要  | 仅包含OTA的应用需要用到                |
| SerialsManager     | 提供不同串行通讯链路层的封装        | 非必要  | 当配置协议栈为DTM模式或Host Only模式时需要） |
| Flash              | 对不同芯片内部和外部Flash操作的封装  | 非必要  | 当包括NVM模块时为必要                 |
| NVM                | 在Flash上提供一套非易失存储机制    | 非必要  | 仅当系统需要用到Bonding时才为必要         |
| MWS                | 多协议共存协议               | 非必要  | 当需要与外部WLAN芯片配                |
| Shell              | 提供控制台交互和格式化打印功能       | 非必要  | 服务于应用程序                      |
| GPIO/LED/Keyboard  | 提供简单输入输出能力            | 非必要  | 服务于应用程序                      |
| Reset/Panic/RNG    | 对系统各个功能的简单封装          | 非必要  | 服务于应用程序                      |

  

## BLE协议栈

BLE协议栈以闭源库的形式提供，***分为Host和Controller两部分***。下图描述了组成Stack的主要文件和相互之间的依赖关系。

![](https://raw.githubusercontent.com/DustOfStars/ObsPicGo/master/Gavin_Obs/20221102160828.png)


- Controller Stack负责与硬件和时序相关的蓝牙链路层处理，它通过HCI接口与Host Stack进行协同工作完成BLE数据交互。<u>由于使用BLE SDK时几乎不需要直接与Controller Stack打交道，我们知道它的存在即可。</u>
- Host Stack是一个与硬件无关的C库，它通过HCI接口向Controller Stack发送命令和接收消息，***负责完成Bluetooth Core Spec中的L2CAP, ATT, SM, GATT, GAP等多个层次职责***。Host Stack通过一组头文件暴露出协议栈各个层的API和回调事件接口，理解Host Stack提供的服务和数据结构对于掌握BLE SDK开发至关重要。本系列专题将以独立文章分享各个部分服务的具体应用，这里先将协议栈的两类主要接口介绍给大家。

1.  API
	- API即应用程序编程接口，**是外部应用主动向协议栈发起请求的入口**。协议栈所提供的绝大部分API都是异步的，**利用Messaging消息队列向协议栈的Host任务发送一条后便返回**。实际的处理是在Host任务中完成，再通过某个注册的callback回调函数来反馈执行的结果。
2.  Callback注册回调函数
	- Callback注册回调函数是由应用程序提供的一个钩子函数，**在协议栈内部处理中如果有需要通知到用户层事件发生或者数据变化时，将调用注册的钩子函数通知应用程序做出响应动作**。

下面一段示例代码演示了 `Gap_Connect()` 和 `Gap_Disconnect()` 两个异步API的行为和他们所触发的callback回调函数后对应事件的响应。通过这段代码可以大致了解应用代码如何与协议栈进行交互。

```c
// 1. initiate API to request and connection and register a callback

bleResult_t rt;

rt = Gap_Connect(&gConnReqParams, Gap_ConnectionCallback);

if (rt != gBleSuccess_c) {

    printf("Gap_Connect API fail, reason = %d\r\n", rt);

}

// when PC(R15) arrives here, the connection may NOT be setup

... 

// 2. initiate API to request a disconnection action.

rt = Gap_Disconnect(peerDeviceId);

if (rt != gBleSuccess_c) {

    printf("Gap_Disconnect fail, reason = %d\r\n", rt);

}

// when PC(R15) arrives here, the connection may NOT be disconnected

... 

// 3. Event will be triggered when connection and disconnection happens

void Gap_ConnectionCallback(deviceId_t peerDeviceId, gapConnectionEvent_t* pConnectionEvent)

{

    switch (pConnectionEvent->eventType)

    {

        case gConnEvtConnected_c:

            // async event to notify that link is connected

            printf("Connected to device (%d)\r\n", peerDeviceId);

            break;

        case gConnEvtDisconnected_c:

             // async event to notify that link is disconnected

            printf("Disconnected to device (%d)\r\n", peerDeviceId);

            break;

            ...
```

## 低功耗蓝牙服务框架

低功耗蓝牙服务框架这个名称是笔者**取给SDK中非协议栈部分的蓝牙代码取一个名字，涉及内容主要包括有HCI、FSCI、Profiles、Connection Manager, Service Discovery, App Thread, BLE Initialization, Stack Runtime Environment等几个部分，顾名思义是这些代码都是为实现BLE应用而服务的，目的是简化用户开发的难度**。

*   **Stack Runtime Environment**
**上一小节介绍的Host和Controller协议栈库提供的是一组可链接的符号，需要外部创建任务以提供运行时环境和创建必要的队列和事件资源，这就是Stack Runtime Environment的职责**。协议栈提供了两个任务处理函数 `Host_TaskHandler()` 和 `Controller_TaskHandler()` ，分别在创建的 `Host_task` 和 `Controller_task` 里调用。另外任务处理函数还需要用到队列和事件资源，用户任务与 `Host_task` 交互、以及 `Host_task` 与 `Controller_task` 的交互都依赖于他们。下表说明了所创建资源的用途。

| 资源                    | 类别     | 用途                                                       |
|-----------------------|--------|----------------------------------------------------------|
| gApp2Host_TaskQueue   | 消息队列   | 用户任务调用Host Stack API时，向这个队列插入一个请求                        |
| gHci2Host_TaskQueue   | 消息队列   | Controller Taskdan产生HCI事件时，向这个队列插入一个请求                   |
| gHost_TaskEvent       | OSA事件  | 用于通知Host Stack队列中有新的消息插入                                 |
| mControllerTaskEvent  | OSA事件  | Host Task向Controller Task发送HCI命令时或中断向Controller Task请求服务 | 

*   **App Thread**
App Thread是在系统启动后由OSA创建的**第一个任务，它在完成BLE系统初始化后创建了一个应用后台，主要目的是服务Host stack触发的各类callback回调事件**。

通过Host_stack提供的Registeration API注册的回调函数被调用的上下文都是前面介绍的Host task里，如果在这些callback做一些耗时较长的用户动作（比如上面的printf打印输出），Host task其他部分会得不到响应，从而影响BLE协议交互。因此很有必要将这些callback回调事件的处理放到一个相对低优先级任务中，这个任务就是**App Thread**。首先 `Applmain.c` 中对 `Host stack` 提供的回调函数接口都做了实现，这些实现并不作实际处理，**而是转发一个消息到App Thread**。<u>App Thread一直处于等待事件的状态，收到事件后再分发到各个自定义的callback中作相应处理。App Thread同时还可以通过等待另外一个队列来接收其他任务发送的Application消息并处理，以方便用户实现类似于协议栈的异步处理机制</u>。

下表列出了App Thread的队列和时间资源以及他们的用途。

| 资源                  | 类别     | 用途                                 |
|---------------------|--------|------------------------------------|
| **mHostAppInputQueue**  | 消息队列   | Host Stack产生的callback里将向这个队列插入一个请求 |
| **mAppCbInputQueue**    | 消息队列   | App Thread本身或者其他用户任务都可以向这个队列插入一个请求 |
| **mAppEvent**           | OSA事件  | 用于通知App Thread有队列里有新的消息插入          |

下面代码段以App_Connect为例来展示了代码是如何通过App Thread来处理Host stack回调函数的。

```c

// User call App_Connect, instead of Gap_Connect

bleResult_t App_Connect(gapConnectionRequestParameters_t*   pParameters,

                        gapConnectionCallback_t             connCallback)

{

    pfConnCallback = connCallback;

    return Gap_Connect(pParameters, App_ConnectionCallback);

}

// Simplified App_ConnectionCallback, the function insert a message and notify App Thread

void App_ConnectionCallback (deviceId_t peerDeviceId, gapConnectionEvent_t* pConnectionEvent)

{

    appMsgFromHost_t *pMsgIn = NULL;   

    uint8_t msgLen = GetRelAddr(appMsgFromHost_t, msgData) + sizeof(connectionMsg_t);

    pMsgIn = MSG_Alloc(msgLen);

    // use Applmain defined message type

    pMsgIn->msgType = gAppGapConnectionMsg_c; 

    pMsgIn->msgData.connMsg.deviceId = peerDeviceId;

    FLib_MemCpy(&pMsgIn->msgData.connMsg.connEvent, pConnectionEvent, sizeof(gapConnectionEvent_t));

    MSG_Queue(&mHostAppInputQueue, pMsgIn);

    OSA_EventSet(mAppEvent, gAppEvtMsgFromHostStack_c);  

}

// Pseduo code of App_Thread

void App_Thread (uint32_t param)

{

    while(1) {

        OSA_EventWait(mAppEvent, osaEventFlagsAll_c, FALSE, osaWaitForever_c , &event);

        if (event & gAppEvtMsgFromHostStack_c) {

            while (MSG_Pending(&mHostAppInputQueue))  {

                pMsgIn = MSG_DeQueue(&mHostAppInputQueue);

                // check msgType and dispatch callback handler

                if (pMsg->msgType == gAppGapConnectionMsg_c) {

                    pfConnCallback(pMsg->msgData.connMsg.deviceId, &pMsg->msgData.connMsg.connEvent);

                } else if (pMsg->msgType == ...) {
```

通过下图可以全面的**了解到App Thread, Host task和Controller task三者之间是如何完成交互的**。掌握了这张图的消息交互流程，开发基于NXP BLE SDK的应用也就变的非常容易了。

![ky](https://raw.githubusercontent.com/DustOfStars/ObsPicGo/master/Gavin_Obs/20221102162253.png)

*   **HCI**

    HCI是低功耗蓝牙Controller协议栈和Host协议栈之间的沟通桥梁，对于SoC，两个协议栈都运行在同一芯片上，这部分功能实际上是简单的函数调用。当BLE SDK被配置为只作为controller Only模式（用于DTM），或者Host Only模式（controller使用另外一颗芯片）时，HCI的就需要与通过SerialManager来完成与外界通讯。

*   **FSCI**

    FSCI是**F**ramework **S**erial **C**ommunication **I**nterface的缩写，该模块**定义了一套统一的接口**将无线通讯微控制器（BLE，Thread，Zigbee）所提供的服务提供给另外一颗主处理器或者PC系统。对于BLE SoC而言，片上运行了完整的BLE Host和Controller协议栈，此时处于Network Processor网络处理器模式。**外部处理器在只需要遵循FSCI协议便能控制SoC发起广播、建立连接和进行数据交换。**

*   **Profiles**

    在GATT之上，Bluetooth SIG定义了多个标准的GATT based Profile帮助应用层互联互通，同时每个厂家也会定义一些简单的私有Profile来实现raw数据传输、OTA等功能。由于所有的Callback回调事件处理都是在应用中完成的，Profile在NXP BLE SDK中职责比较简单，主要负责完成Profile数据到GATT数据库的操作转换。

*   **Conneciton Manager** 与 **Service Discovery**

	**连接建立和服务发现是每个BLE应用都需要经历的过程，BLE SDK将这部分通用的代码从应用层分离了出来，形成两对独立的.c/.h文件ble_conn_manager.c/h和ble_service_discovery.c/h**。这样减少了应用层重复的代码，提高代码可维护性。这两个部分服务的功能在后面介绍蓝牙SDK具体编程的文章中都会有所涉及。

*   **GATT Database**
    
    通过一套宏定义的方法，完成静态或者动态GATT数据库的建立。第一次看到gatt_db.h会很难以理解里面的特殊格式，但要是“照猫画虎”的增加自己的service或者characeteristic还是比较简单的，这就是它的神奇之处。写一篇文章专门讲解gatt_db模块到底是如何通过一系列的宏定义来成这个工作的。
    

## 应用

BLE SDK的每个应用示例都包含了十分类似的应用层文件，下表简单描述了各个文件的职责。其中.c/h是整个应用的核心。

| 文件名               | 职责                                                  |
|-------------------|-----------------------------------------------------|
| app_config.c      | BLE广播参数、扫描参数、安全性相关的配置                               |
| app_preinclude.h  | 全局参数配置文件（其他文件不需要include此文件，在IDE里已配置）                |
| gatt_db.h         | GATT database描述文件                                   |
| gatt_uuid128.h    | 私有128位UUID描述文件                                      |
| .c/h              | 应用示例代码，如heart_rate_sensor.c，广播发起、扫描，以及各种BLE协议栈事件的处理 |


## 小结

本文从零开始介绍NXP BLE SDK系统架构，逐步分析了各个子模块的功能。对于BLE协议栈和服务框两部分作了较为细致的讲解，以帮助大家了解自己编写的应用是如何与框架进行交互的。本文是《深入NXP低功耗蓝牙SDK开发系列》的第一篇，涵盖内容较广。如果您觉得其中某部分功能的介绍太过简单，不要心急，留言给我，后续会有多篇文章一步步教大家如何上手BLE SDK开发，敬请期待吧：）。



[[2022-11-02_星期三]]

