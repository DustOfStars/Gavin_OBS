---
annotation-target: /PDF/Programming Embedded Systems in C and C++.pdf
---

>%%
>```annotation-json
>{"created":"2022-11-30T09:25:43.624Z","text":"什么是嵌入式系统？","updated":"2022-11-30T09:25:43.624Z","document":{"title":"Programming Embedded Systems in C and C++.pdf","link":[{"href":"urn:x-pdf:3bc9ca85000f7343bdef3dbc72be1c4a"},{"href":"vault:/PDF/Programming Embedded Systems in C and C++.pdf"}],"documentFingerprint":"3bc9ca85000f7343bdef3dbc72be1c4a"},"uri":"vault:/PDF/Programming Embedded Systems in C and C++.pdf","target":[{"source":"vault:/PDF/Programming Embedded Systems in C and C++.pdf","selector":[{"type":"TextPositionSelector","start":34504,"end":34665},{"type":"TextQuoteSelector","exact":"An embedded system is a combination of computer hardware and software, andperhaps additional mechanical or other parts, designed to perform a specific func-tion.","prefix":"ples.What Is an Embedded System?","suffix":" A good example is the microwave"}]}]}
>```
>%%
>*%%PREFIX%%ples.What Is an Embedded System?%%HIGHLIGHT%% ==An embedded system is a combination of computer hardware and software, andperhaps additional mechanical or other parts, designed to perform a specific func-tion.== %%POSTFIX%%A good example is the microwave*
>%%LINK%%[[#^y9u27mldqtl|show annotation]]
>%%COMMENT%%
>什么是嵌入式系统？
>%%TAGS%%
>#定义
^y9u27mldqtl


>%%
>```annotation-json
>{"created":"2022-12-01T02:50:18.352Z","text":"实时系统是一个有时间限制的计算机系统。\n换句话说，一个实时系统的部分规定，是在其及时进行某些计算或决策的能力方面。\n这些重要的计算有完成的最后期限。","updated":"2022-12-01T02:50:18.352Z","document":{"title":"Programming Embedded Systems in C and C++.pdf","link":[{"href":"urn:x-pdf:3bc9ca85000f7343bdef3dbc72be1c4a"},{"href":"vault:/PDF/Programming Embedded Systems in C and C++.pdf"}],"documentFingerprint":"3bc9ca85000f7343bdef3dbc72be1c4a"},"uri":"vault:/PDF/Programming Embedded Systems in C and C++.pdf","target":[{"source":"vault:/PDF/Programming Embedded Systems in C and C++.pdf","selector":[{"type":"TextPositionSelector","start":39626,"end":39835},{"type":"TextQuoteSelector","exact":"a real-time system is a computer system that has timing con-straints. In other words, a real-time system is partly specified in terms of its abilityto make certain calculations or decisions in a timely manner.","prefix":"this point. Ascommonly defined, ","suffix":" These important cal-culations a"}]}]}
>```
>%%
>*%%PREFIX%%this point. Ascommonly defined,%%HIGHLIGHT%% ==a real-time system is a computer system that has timing con-straints. In other words, a real-time system is partly specified in terms of its abilityto make certain calculations or decisions in a timely manner.== %%POSTFIX%%These important cal-culations a*
>%%LINK%%[[#^h8q7n0g3ygp|show annotation]]
>%%COMMENT%%
>实时系统是一个有时间限制的计算机系统。
>换句话说，一个实时系统的部分规定，是在其及时进行某些计算或决策的能力方面。
>这些重要的计算有完成的最后期限。
>%%TAGS%%
>#real-time system
^h8q7n0g3ygp


>%%
>```annotation-json
>{"created":"2022-12-01T02:54:01.995Z","text":"1. 每个嵌入式系统的硬件都是专门为应用而定制的，以保持系统的低成本。因此，不必要的电路被消除了，\n2. 硬件资源被尽可能地共享。\n\n在本节中，将了解所有嵌入式系统中哪些硬件特征是共同的，以及为什么在其他方面会有如此多的差异。","updated":"2022-12-01T02:54:01.995Z","document":{"title":"Programming Embedded Systems in C and C++.pdf","link":[{"href":"urn:x-pdf:3bc9ca85000f7343bdef3dbc72be1c4a"},{"href":"vault:/PDF/Programming Embedded Systems in C and C++.pdf"}],"documentFingerprint":"3bc9ca85000f7343bdef3dbc72be1c4a"},"uri":"vault:/PDF/Programming Embedded Systems in C and C++.pdf","target":[{"source":"vault:/PDF/Programming Embedded Systems in C and C++.pdf","selector":[{"type":"TextPositionSelector","start":41387,"end":41601},{"type":"TextQuoteSelector","exact":"hehardware in each embedded system is tailored specifically to the application, inorder to keep system costs low. As a result, unnecessary circuitry is eliminated andhardware resources are shared wherever possible.","prefix":"ty in the underlying hardware. T","suffix":" In this section you will learnw"}]}]}
>```
>%%
>*%%PREFIX%%ty in the underlying hardware. T%%HIGHLIGHT%% ==hehardware in each embedded system is tailored specifically to the application, inorder to keep system costs low. As a result, unnecessary circuitry is eliminated andhardware resources are shared wherever possible.== %%POSTFIX%%In this section you will learnw*
>%%LINK%%[[#^vt10j49rmnq|show annotation]]
>%%COMMENT%%
>1. 每个嵌入式系统的硬件都是专门为应用而定制的，以保持系统的低成本。因此，不必要的电路被消除了，
>2. 硬件资源被尽可能地共享。
>
>在本节中，将了解所有嵌入式系统中哪些硬件特征是共同的，以及为什么在其他方面会有如此多的差异。
>%%TAGS%%
>
^vt10j49rmnq


>%%
>```annotation-json
>{"created":"2022-12-01T02:56:39.995Z","text":"ROM : 存放可执行代码\nRAM : 临时存储运行时的数据；\n\n可以片上，也可以片外","updated":"2022-12-01T02:56:39.995Z","document":{"title":"Programming Embedded Systems in C and C++.pdf","link":[{"href":"urn:x-pdf:3bc9ca85000f7343bdef3dbc72be1c4a"},{"href":"vault:/PDF/Programming Embedded Systems in C and C++.pdf"}],"documentFingerprint":"3bc9ca85000f7343bdef3dbc72be1c4a"},"uri":"vault:/PDF/Programming Embedded Systems in C and C++.pdf","target":[{"source":"vault:/PDF/Programming Embedded Systems in C and C++.pdf","selector":[{"type":"TextPositionSelector","start":41925,"end":42062},{"type":"TextQuoteSelector","exact":"theremust be a place to store the executable code and temporary storage for runtimedata manipulation. These take the form of ROM and RAM,","prefix":"nly, in order to have software, ","suffix":" respectively; anyembedded syste"}]}]}
>```
>%%
>*%%PREFIX%%nly, in order to have software,%%HIGHLIGHT%% ==theremust be a place to store the executable code and temporary storage for runtimedata manipulation. These take the form of ROM and RAM,== %%POSTFIX%%respectively; anyembedded syste*
>%%LINK%%[[#^38y38ixt7st|show annotation]]
>%%COMMENT%%
>ROM : 存放可执行代码
>RAM : 临时存储运行时的数据；
>
>可以片上，也可以片外
>%%TAGS%%
>#ROM, #RAM
^38y38ixt7st


>%%
>```annotation-json
>{"created":"2022-12-01T03:00:46.683Z","updated":"2022-12-01T03:00:46.683Z","document":{"title":"Programming Embedded Systems in C and C++.pdf","link":[{"href":"urn:x-pdf:3bc9ca85000f7343bdef3dbc72be1c4a"},{"href":"vault:/PDF/Programming Embedded Systems in C and C++.pdf"}],"documentFingerprint":"3bc9ca85000f7343bdef3dbc72be1c4a"},"uri":"vault:/PDF/Programming Embedded Systems in C and C++.pdf","target":[{"source":"vault:/PDF/Programming Embedded Systems in C and C++.pdf","selector":[{"type":"TextPositionSelector","start":42714,"end":42927},{"type":"TextQuoteSelector","exact":"The inputs to the system usually take the form of sensors and probes,communication signals, or control knobs and buttons. The outputs are typicallydisplays, communication signals, or changes to the physical world.","prefix":"e, current tempera-ture, etc.). ","suffix":" See Figure 1-1for a general exa"}]}]}
>```
>%%
>*%%PREFIX%%e, current tempera-ture, etc.).%%HIGHLIGHT%% ==The inputs to the system usually take the form of sensors and probes,communication signals, or control knobs and buttons. The outputs are typicallydisplays, communication signals, or changes to the physical world.== %%POSTFIX%%See Figure 1-1for a general exa*
>%%LINK%%[[#^5hm0v9x69bt|show annotation]]
>%%COMMENT%%
>
>%%TAGS%%
>#输入输出
^5hm0v9x69bt


>%%
>```annotation-json
>{"created":"2022-12-01T03:03:28.898Z","text":"硬件千变万化，来自于各种要求的妥协和权衡；\n\n成本、功耗、面积等等","updated":"2022-12-01T03:03:28.898Z","document":{"title":"Programming Embedded Systems in C and C++.pdf","link":[{"href":"urn:x-pdf:3bc9ca85000f7343bdef3dbc72be1c4a"},{"href":"vault:/PDF/Programming Embedded Systems in C and C++.pdf"}],"documentFingerprint":"3bc9ca85000f7343bdef3dbc72be1c4a"},"uri":"vault:/PDF/Programming Embedded Systems in C and C++.pdf","target":[{"source":"vault:/PDF/Programming Embedded Systems in C and C++.pdf","selector":[{"type":"TextPositionSelector","start":43089,"end":43152},{"type":"TextQuoteSelector","exact":"This variation is the result of many competing design crite-ria","prefix":"ed hard-ware is usually unique. ","suffix":". Each system must meet a comple"}]}]}
>```
>%%
>*%%PREFIX%%ed hard-ware is usually unique.%%HIGHLIGHT%% ==This variation is the result of many competing design crite-ria== %%POSTFIX%%. Each system must meet a comple*
>%%LINK%%[[#^57gajbbrule|show annotation]]
>%%COMMENT%%
>硬件千变万化，来自于各种要求的妥协和权衡；
>
>成本、功耗、面积等等
>%%TAGS%%
>
^57gajbbrule


>%%
>```annotation-json
>{"created":"2022-12-01T03:08:03.867Z","text":"MIPS 和 寄存器位宽 作为衡量 处理器power的标志","updated":"2022-12-01T03:08:03.867Z","document":{"title":"Programming Embedded Systems in C and C++.pdf","link":[{"href":"urn:x-pdf:3bc9ca85000f7343bdef3dbc72be1c4a"},{"href":"vault:/PDF/Programming Embedded Systems in C and C++.pdf"}],"documentFingerprint":"3bc9ca85000f7343bdef3dbc72be1c4a"},"uri":"vault:/PDF/Programming Embedded Systems in C and C++.pdf","target":[{"source":"vault:/PDF/Programming Embedded Systems in C and C++.pdf","selector":[{"type":"TextPositionSelector","start":43947,"end":43989},{"type":"TextQuoteSelector","exact":"MIPS (millions of instructions persecond) ","prefix":"compare processing power is the ","suffix":"rating. If two processors have r"}]}]}
>```
>%%
>*%%PREFIX%%compare processing power is the%%HIGHLIGHT%% ==MIPS (millions of instructions persecond)== %%POSTFIX%%rating. If two processors have r*
>%%LINK%%[[#^i8y400hd2mp|show annotation]]
>%%COMMENT%%
>MIPS 和 寄存器位宽 作为衡量 处理器power的标志
>%%TAGS%%
>
^i8y400hd2mp


>%%
>```annotation-json
>{"created":"2022-12-01T03:11:16.954Z","updated":"2022-12-01T03:11:16.954Z","document":{"title":"Programming Embedded Systems in C and C++.pdf","link":[{"href":"urn:x-pdf:3bc9ca85000f7343bdef3dbc72be1c4a"},{"href":"vault:/PDF/Programming Embedded Systems in C and C++.pdf"}],"documentFingerprint":"3bc9ca85000f7343bdef3dbc72be1c4a"},"uri":"vault:/PDF/Programming Embedded Systems in C and C++.pdf","target":[{"source":"vault:/PDF/Programming Embedded Systems in C and C++.pdf","selector":[{"type":"TextPositionSelector","start":44780,"end":44890},{"type":"TextQuoteSelector","exact":" In general, the register widthof a processor establishes the upper limit of the amount of memory it canaccess","prefix":" affect the processor selection.","suffix":" (e.g., an 8-bit address registe"}]}]}
>```
>%%
>*%%PREFIX%%affect the processor selection.%%HIGHLIGHT%% ==In general, the register widthof a processor establishes the upper limit of the amount of memory it canaccess== %%POSTFIX%%(e.g., an 8-bit address registe*
>%%LINK%%[[#^6o1ht2aszpj|show annotation]]
>%%COMMENT%%
>
>%%TAGS%%
>
^6o1ht2aszpj


>%%
>```annotation-json
>{"created":"2022-12-01T03:13:32.137Z","updated":"2022-12-01T03:13:32.137Z","document":{"title":"Programming Embedded Systems in C and C++.pdf","link":[{"href":"urn:x-pdf:3bc9ca85000f7343bdef3dbc72be1c4a"},{"href":"vault:/PDF/Programming Embedded Systems in C and C++.pdf"}],"documentFingerprint":"3bc9ca85000f7343bdef3dbc72be1c4a"},"uri":"vault:/PDF/Programming Embedded Systems in C and C++.pdf","target":[{"source":"vault:/PDF/Programming Embedded Systems in C and C++.pdf","selector":[{"type":"TextPositionSelector","start":45518,"end":45671},{"type":"TextQuoteSelector","exact":"Of course, the smaller the register width, the more likely it is that the processor employs tricks like mul-tiple address spaces to support more memory. ","prefix":"nents for alow-volume product.* ","suffix":"A few hundred bytes just isn’t e"}]}]}
>```
>%%
>*%%PREFIX%%nents for alow-volume product.*%%HIGHLIGHT%% ==Of course, the smaller the register width, the more likely it is that the processor employs tricks like mul-tiple address spaces to support more memory.== %%POSTFIX%%A few hundred bytes just isn’t e*
>%%LINK%%[[#^re8un3wgg1q|show annotation]]
>%%COMMENT%%
>
>%%TAGS%%
>
^re8un3wgg1q
