---
title: STM32入门——基于野火 F407 霸天虎课程学习
date: 2023-11-26
tags:
- C
categories:
- Embeded System
---

## 前言

博主开始探索嵌入式以来，其实很早就开始玩 stm32 了。但是学了一段时间之后总是感觉还是很没有头绪，不知道在学什么。前前后后分别尝试了江协科技、正点原子、野火霸天虎三次 stm32 的课程学习。江协科技的 stm32f103c8t6 课程看了一段时间，感觉对一些外设的调用方法有一个基础的认知了，但是没有很明白到底在学什么；正点原子则是有点听不懂，半字也借给同学了就有一段时间没学，感觉自认为 stm32 学的有两把刷子了。后来听前辈说江协科技的 stm32 课程不如 51 单片机的质量好，其实课程讲的一般，我就想：是不是应该重新好好学一下 stm32 课程了。

这次选择的是野火的  F407 霸天虎课程，第一是听大家说，入门选野火或者正点最好。第二是野火的大师进阶篇的一些内容，涉及到一些原理等的学习讲解，我觉得对我会非常有帮助，因此正好就买了这款开发板从入门到中级到大师原理一起学习了。

本系列博文笔记主要基于野火相应课程，b站地址：[野火F407开发板-霸天虎视频-【入门篇】_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1Vt411X7PK/?spm_id_from=333.337.search-card.all.click&vd_source=05baa16b109824edfe68909895264c33) ，仅供学习参考不做任何商业用途使用，侵删！

### 调试器介绍

我选择的是高速版，支持 SW 和 JTAG 两种连接方式。SW 模式则只需要连接 VREF（3V3）, TMS（数据）, TCK, RESET, GND 五个引脚。

### 程序烧录配置

芯片型号：STM32F407ZGTX。

**DAP 仿真烧录**自然非常简单。

**串口一键下载 ISP** 下载速度慢，不能调试，但是成本很低。可以使用 FlyMcu 等软件。

**2023.11.2 补充。**

**警告，建议如果 flymcu 不能烧录，就不要尝试这种玩法了，看看课学学得了。因为我自己乱捣鼓一通后把开发板锁了。**

**下面的内容我不太清楚具体是哪一步出现了锁死 flash 的问题，总之不要尝试！学习一下理论得啦。**

**如果和我一样锁死了，请见野火大师篇程序，里面有一个解除写保护的代码，运行一下。**

ISP 下载方式：允许我们不拆下芯片来下载。对于上个世纪嵌入式学习来说这是一个很大的突破，因为当时是要把芯片拆下来烧录编程的。

ISP 厂商出产的时候就选定了一种串行外设对芯片内部 FLASH 进行编程，我们不能修改。常用串口下载方式，成本低，但是不能调试仿真。

普通 ISP 需要手动配置 boot loader，一键 ISP 不用，硬件电路和上位机配合达到一键下载的效果（手动配置：00是用户闪存启动，10是系统 SRAM/ISP 启动，普通 ISP 要手动改跳线帽）。

一键下载电路的具体原理流程如下：

1. RTS 低电平，Q1 是一个 PNP 三极管，导通，BOOT0 拉高。
2. DTS 高电平，Q2 NPN 导通，U18 是一个由 EN 控制开关的模拟开关，2 脚被导通为低电平，连接1脚拉低 NRST 复位。程序下载执行。
3. U18 模拟开关的作用是稳定电路。开发板复位的时候 DTR RTS 是不稳定的状态，如果没有这个模拟开关，DTR RTS 可能进入 ISP 状态，复位，进入 ISP 状态，复位，进入……一直运行不起来了。模拟开关右侧电容使得 VCC 需要花一点时间充电给 EN，而不是立刻激活 EN（EN 1.8V 左右）。这时候 DTR RTS 已经稳定了，可以导通 U18 12 引脚来给 NRST 复位了。

![1698858387784](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/1698858387784.png)

不过 FlyMcu 实际配置方式是反过来的，因为他的协议是 232（+3~+15 是 0，-3~-15 是 1），和 TTL（3.3v 是1,0v 是0）正相反。

*但是实际操作的时候可能遇到一种状况：部分开发板无法使用 FlyMcu 写入。我就碰到了。解决办法是使用 stm32 cube programmer 烧录程序。*

*配置如下：开发板上 boot 连接 3v3，RTS DTR=0，选中 read unprotect，建立连接后再烧录程序。*

*但是不知道是波特率或者校验位的问题，我每次能成功烧录进去，然后过一会就显示断开找不到设备了。可能是因为波特率没有76800的选项。*

## STM32 介绍

> 正点原子网课：单片机和电脑的类比：内存是 [SRAM](https://so.csdn.net/so/search?q=SRAM&spm=1001.2101.3001.7020)，硬盘是 FLASH，主板是外设。

st：意法半导体公司，SoC 厂商。

m：微控制器。微控制器和微处理器相比性能比较拉一点，主频低，微处理器能跑一些大 os（linux）。

32：32位微控制器。

> 正点原子网课：8051，X86 属于 CISC；ARM, MIPS, RISC-V 属于 RISC.
>
> ![image-20230401221223643](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202304012212735.png)
>
> 冯诺依曼和哈佛结构的主要区别：程序存储器和数据存储器是否分开存储。不分开是冯诺依曼，分开是哈佛。哈佛执行效率更高，冯诺伊曼资源占据更少。
>
> CORTEX-M 系列介绍
> ARM 公司（做精简指令集计算机的）只设计内核架构和授权知识产权，不参与设计芯片，给其他合作公司授权设计芯片。半导体厂商再根据架构完善周边电路并制作芯片。现在95%手机、平板都是 ARM 架构的， ARM 公司是真的牛。
>
> ![image-20230401221658811](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202304012216885.png)
>
>
> 其优点在于低功耗低成本高性能，且支持16/32位双指令集。
>
> ARM 有9个版本，从 v6 开始出现 cortex 的命名。
>
> ![image-20230401221918218](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202304012219288.png)

随着需求不断发展，stm32 在一众 8/16位 MCU 中脱颖而出。

stm32 自带许多通信接口，如 spi i2c uart 等；扫地机，无人机，手环等都可以是 stm32 的作品。

如何选型？以下是几大类 stm32 的特点。

![image-20231102013709264](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/image-20231102013709264.png)

本课程学习使用的开发板命名方式：

![image-20231102013830412](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/image-20231102013830412.png)

选型：满足项目需求的前提下，尽可能选便宜的，比如主频低，功耗低，引脚少，flash 少。

引脚分配：

![1698860405030](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/1698860405030.png)

看手册的重点：

![1698860693691](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/1698860693691.png)

外设资源，芯片功能，引脚，引脚大致分类，内存，封装……

哎想起前两天面试被问，如果选型 MCU 我应该看哪些因素。我只想到了外设和内存hhh。属于是只会写代码的笨比了。这也是我开始重新看野火课程的原因之一。

## 寄存器

虽然正式编程没有必要用寄存器编程，通常都是库函数或者 hal 库。但是还是有必要学一下原理的。

### 寄存器映射

芯片视图如下。

丝印：芯片上印的信息。型号，内核，生产批次等。

引脚：左上角是有小圆点的，从左上-左下-右下-右上逆时针看。或者如果没有小圆点，把丝印方向摆正，从左上角开始看。

![image-20231102121423731](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/image-20231102121423731.png)

芯片内部组成：

![image-20231102121841662](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/image-20231102121841662.png)

寄存器映射：32位，2^32^=4GB，因此所有程序都需要通过内存 4GB 去映射访问。

![image-20231102122058475](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/image-20231102122058475.png)

block7：M4 芯片内外设，比如一些通信总线这些都算外设。

block1：内存。

block0：代码。不过实际上由于设计工艺的问题，block0 block1 都只用了很少的一部分来存代码或者作为内存。

外设寄存器放在 block2 中。根据不同块速度不一样，又具体分为不同速度的外设（AHB APB）。

总线速度：AHB>APB2>APB1. APB1 是较低速的外设，包括 I2C UART SPI 看门狗等。

我们想要操作特定的外设，其实就是控制他的寄存器。控制寄存器就要找到寄存器相应的地址往里面写入数据，寄存器地址就是内存中的地址映射。

比如 GPIOF 我们想让其端口全部输出高电平。我们查找 stm32f407 手册，发现 GPIOF 的地址是 0x40021400，GPIOF 的 ODR（output data register）相对起始地址的偏移地址是14，则我们需要给 0x40021414 的地址写入数据 0xFFFF.

![image-20231102123404148](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/image-20231102123404182.png)

51 单片机库函数中封装的 reg51.h 中，利用 sfr 定义寄存器地址；而 stm32 库函数中使用宏定义，这些就是寄存器映射操作。**对芯片里一个特殊功能的内存单元起别名的过程就是寄存器映射。** 给这个地址再分配一个地址交重映射，stm32 中不咋常用。

### C语言对寄存器的封装

这样逐个地址，哪怕已经进行了寄存器映射，还是很复杂。

c 语言库函数实际进行的封装操作是使用结构体批量定义。

```c
/* GPIO 外设基地址 */
#define GPIOA_BASE (AHB1PERIPH_BASE + 0x0000)
#define GPIOB_BASE (AHB1PERIPH_BASE + 0x0400)
#define GPIOC_BASE (AHB1PERIPH_BASE + 0x0800)
#define GPIOD_BASE (AHB1PERIPH_BASE + 0x0C00)
#define GPIOE_BASE (AHB1PERIPH_BASE + 0x1000)
#define GPIOF_BASE (AHB1PERIPH_BASE + 0x1400)
#define GPIOG_BASE (AHB1PERIPH_BASE + 0x1800)
#define GPIOH_BASE (AHB1PERIPH_BASE + 0x1C00)

/* GPIO 寄存器列表 */
typedef struct {
    uint32_t MODER; /*GPIO 模式寄存器 地址偏移: 0x00 */
    uint32_t OTYPER; /*GPIO 输出类型寄存器 地址偏移: 0x04 */
    uint32_t OSPEEDR; /*GPIO 输出速度寄存器 地址偏移: 0x08 */
    uint32_t PUPDR; /*GPIO 上拉/下拉寄存器 地址偏移: 0x0C */
    uint32_t IDR; /*GPIO 输入数据寄存器 地址偏移: 0x10 */
    uint32_t ODR; /*GPIO 输出数据寄存器 地址偏移: 0x14 */
    uint16_t BSRRL; /*GPIO 置位/复位寄存器低 16 位部分 地址偏移: 0x18 */
    uint16_t BSRRH; /*GPIO 置位/复位寄存器高 16 位部分 地址偏移: 0x1A */
    uint32_t LCKR; /*GPIO 配置锁定寄存器 地址偏移: 0x1C */
	uint32_t AFR[2]; /*GPIO 复用功能配置寄存器 地址偏移: 0x20-0x24 */
} GPIO_TypeDef;

/* 使用 GPIO_TypeDef 把地址强制转换成指针 */
#define GPIOA ((GPIO_TypeDef *) GPIOA_BASE)
#define GPIOB ((GPIO_TypeDef *) GPIOB_BASE)
#define GPIOC ((GPIO_TypeDef *) GPIOC_BASE)
#define GPIOD ((GPIO_TypeDef *) GPIOD_BASE)
#define GPIOE ((GPIO_TypeDef *) GPIOE_BASE)
#define GPIOF ((GPIO_TypeDef *) GPIOF_BASE)
#define GPIOG ((GPIO_TypeDef *) GPIOG_BASE)
#define GPIOH ((GPIO_TypeDef *) GPIOH_BASE)
```

所有外设都是如此，首先定义总线 APB AHB 地址，然后每个外设在其基础上进行偏移，每个外设的不同部分再在该外设基址上进行偏移。

## 新建工程

### 寄存器方式

要命啊，一看名字我就不想试。寄存器新建不得麻烦死。

哎算了为了学习原理，干了。

我们尝试自己写一个寄存器的库函数来引用。

首先我们需要引用 st 官方启动文件 stmf4xx.s，具体用途后面章节再展开讲解。然后我们自己新建一个 stm32f4xx.h 文件来映射寄存器。不过只是把这个文件包含进项目，编译会报错：

````
.\Objects\led_reg.axf: Error: L6218E: Undefined symbol SystemInit (referred from startup_stm32f40xx.o).
````

进入启动文件后，可以看到这么一个函数：

```assembly
; Reset handler
Reset_Handler    PROC
                 EXPORT  Reset_Handler             [WEAK]
        IMPORT  SystemInit
        IMPORT  __main

                 LDR     R0, =SystemInit
                 BLX     R0
                 LDR     R0, =__main
                 BX      R0
                 ENDP
```

import 的作用相当于 extern，所以没有找到这个函数的定义，需要我们自己去定义。这就是为什么简单引入了启动文件会报错。

而 __main 是当我们定义了 main() 函数后，编译器会自动链接一些c语言库定义好的函数，用于初始化堆栈并且调用我们的 main().

**注意，如果想要生成 __main 函数，必须勾选下面这一项。**

![image-20231102200243099](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/image-20231102200243099.png)

野火你讲的是真好啊。我之前草草学了学 stm32 单片机用法，比赛的时候自己想移植代码，改了启动文件也不好使，就是报错。原来是这个原因。

那么我们只需要定义这么一个函数，哪怕内容是空都无所谓。

最终我们定义的初步项目框架如下：

![1698926683862](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/1698926683862.png)

stm32f4xx.h：内容为空，有这么个东西就行。

main.c：

```c
#include "stm32f4xx.h"

int main(){
	while(1){
		
	}
}

void SystemInit(){
	
}
```

好了，这个程序可以烧录到板子上的。烧录成功之后没有任何反应（因为本来程序也没做什么哈哈），但是这就是一个大进步了。

#### 点灯——51单片机版

51单片机版就是引用 reg51.h 头文件，在其中声明了各个引脚的地址。我们只需要直接给引脚赋值即可。

调用代码：

```c
#include "reg51.h"

#ifdef 0 
void main(){
	PA0=0xFE;
	while(1){}
}
#endif
```

接下来我们需要定义 LED 灯的寄存器位置。阅读原理图如下：

![1698939342199](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/1698939342199.png)

大致可以看出，板子上的这个 RGB LED 通过三个引脚来控制 RGB 亮度。输出低电平则导通点亮。

具体输出方式是通过 ODR 进行输出。查找 stm32f4xx 中文参考手册可见：

![1698940158585](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/1698940158585.png)

![1698940257044](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/1698940257044.png)

那么我们就要给 0x4002 1400 +14 的地址赋值，让 1<<6 1<<7 1<<8 的位分别赋值为低电平.

```c
int main(){
	 *(unsigned int *)(0x40021400+14)&=~(1<<6); 
	while(1){
		
	}
}
```

然而这样也不亮。亮就怪了，stm32 寄存器是需要先做初始化配置的。

#### 点灯——stm32 版

首先我们要设置 GPIO 模式。

![1698940961543](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/1698940961543.png)

想点灯 输出高低电平，是 01 通用输出模式。

```c
*(unsigned int *)(0x40021400+0)&=~(3<<(6*2)); 
*(unsigned int *)(0x40021400+0)|=(1<<(6*2)); 
```

意思是先把 PF6 模式位置为00，然后赋值为01通用输出。

配置完模式之后，还需要配置时钟，stm32 每个外设都需要配置时钟。

前面提到过 GPIO 是在 AHB1.

![1699103516061](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/1699103516061.png)

![1699103724261](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/1699103724261.png)

全部代码如下：

```c
#include "stm32f4xx.h"

int main(){
    //RCC
    *(unsigned int *)(0x40023800+0x30)|=(1<<5); 
    
    //Mode
    *(unsigned int *)(0x40021400+0)&=~(3<<(6*2)); 
    *(unsigned int *)(0x40021400+0)|=(1<<(6*2)); 
    
    
    *(unsigned int *)(0x40021400+0x14)&=~(1<<6); 
    while(1){
		
	}
}

void SystemInit(){
	
}

```

接下来，我们把这几个地址值提取出来，宏定义映射寄存器。

```c
//stm32f4xx.h
/* 用来存放寄存器映射相关的代码 */
#define RCC_AHB1_ENR    *(unsigned int *)(0x40023800+0x30)
#define GPIOF_MODER     *(unsigned int *)(0x40021400+0)
#define GPIOF_ODR       *(unsigned int *)(0x40021400+0x14)

//main.c
#include "stm32f4xx.h"

int main(){
    //RCC
    RCC_AHB1_ENR|=(1<<5); 
    
    //Mode
    GPIOF_MODER&=~(3<<(6*2)); 
    GPIOF_MODER|=(1<<(6*2)); 
    
    
    GPIOF_ODR&=~(1<<6); 
    while(1){
		
	}
}

void SystemInit(){
	
}
```

#### 点灯——流水灯闪烁

利用软件延时实现 RGB 流水灯闪烁。很简单，前面已经看了3个 LED 通道 PF678 了。

```c
#include "stm32f4xx.h"

void delay_ms(int time);

int main(){
    //RCC
    RCC_AHB1_ENR|=(1<<5); 
    
    //Mode
    GPIOF_MODER&=~(3<<(6*2)); 
    GPIOF_MODER|=(1<<(6*2)); 
    GPIOF_MODER&=~(3<<(7*2)); 
    GPIOF_MODER|=(1<<(7*2)); 
    GPIOF_MODER&=~(3<<(8*2));   
    GPIOF_MODER|=(1<<(8*2)); 
    
    while(1){
        GPIOF_ODR|=(7<<6);
        GPIOF_ODR&=~(1<<6);
        delay_ms(1000);
        GPIOF_ODR|=(7<<6);
        GPIOF_ODR&=~(1<<7); 
        delay_ms(1000);
        GPIOF_ODR|=(7<<6);
        GPIOF_ODR&=~(1<<8);
        delay_ms(1000);
	}
}

void SystemInit(){
	
}

//毫秒级的延时
void delay_ms(int time)
{    
   int i=0;  
   while(time--)
   {
      i=4000;
      while(i--) ;    
   }
}
```

#### 点灯——GPIO 具体功能框图对应

GPIO：通用输入输出引脚。我们可以通过编程来输出或者读取数据。大部分 GPIO 是已经连接、定义好了一些功能（比如上面尝试过的 PF6 LED），有的引脚有多个功能支持重新映射。

STM32 GPIO 除了 adc 是 3.3v，其他 GPIO 都是 5v 容忍。

GPIO 框图（**重点**）如下：

![image-20231104221140325](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/image-20231104221140325.png)

先从输出开始看。最右侧的 IO 引脚是连接在芯片周围一圈的144个引脚之一。除了 IO 引脚，此图中其他所有部分都是封装在芯片内部我们看不到的。

往左有两个保护二极管。当电压大于 5V，电流会往上 VDD_FT 走。当电压为负电压，电流会由 VSS 往 IO 引脚走。

上下拉电阻：比武外接一个低电平工作的设备，但是我们不希望一上电外设就工作，可以设置上拉电阻，稳定一段时间。

GPIO 输出的数据来源：复位寄存器 BSRR，或者 ODR 设置（图中的3下路部分）。复位寄存器高16位复位（写1置0）低16位置位（写1置1），置位优先级更高。

配置 GPIO 模式（输入/输出，选择哪一路）通过前面用过的 MODER 配置。

输出模式（图中输出控制部分）配置端口输出类型寄存器 OTYPER，比如推挽输出，开漏输出。

推挽输出：有直接驱动能力，输出0就是低电平，输出1就输出可以工作的高电平。原理是采用了一个放大的电路？

![1699356797818](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/1699356797818.png)



输入（INT）为高电平时，反向后 PMOS 导通，输出高电平。输入为低电平时，反向后 NMOS 导通，输出低电平。我们可以用一个小电流去驱动出来一个大电流。

开漏输出：自己本身没有输出高电平的手段。低电平可以接地，高电平没有 PMOS 管，是浮空状态。需要外接一个电阻。

![1699357078909](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/1699357078909.png)

stm32 输出 5V 电压的方法就是开漏输出外接电阻。通过接两个三极管的方式反向。

![1699357296750](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/1699357296750.png)

框图中的模拟部分输入输出则不用配置这些模式信息，直接由外设接到保护二极管再接到输出引脚。

框图中的输入部分经过保护电压后，还需要施密特触发器调整一下。比如原来电压的数值并非精确的0或 3.3V，施密特触发器将高于 1.8V 的全部视作1，低于的全部视作0后输入芯片。模拟部分则不需要经过施密特触发器。

因此配置 GPIO 输出的步骤如下：

1. GPIO 功能，通用输出、复用功能、模拟输入等 MODER；
2. 输出推挽 or 开漏 OTYPER；
3. 输出速度 OSPEEDR；
4. 上下拉电阻是否需要开启 PUPDR；
5. 具体输出内容 BSRR or ODR.

输入部分后面输入实验介绍~

按整个流程重新串一遍代码，如下：（其实和前面差不多，就是重新按照流程串了一遍）

```c
/* 用来存放寄存器映射相关的代码 */
#define RCC_BASE    (unsigned int *)    0x40023800
#define GPIOF_BASE  (unsigned int *)    0x40021400
    
#define RCC_AHB1ENR         *(RCC_BASE+0x30)
    
#define GPIOF_MODER         *(GPIOF_BASE+0x00)
#define GPIOF_OSPEEDR       *(GPIOF_BASE+0x08)
#define GPIOF_PUPDR         *(GPIOF_BASE+0x0C)
#define GPIOF_ODR           *(GPIOF_BASE+0x14)
#define GPIOF_BSRR          *(GPIOF_BASE+0x18)

//main.c
#include "stm32f4xx.h"

int main()
{
    RCC_AHB1ENR |= (1<<5);
    GPIOF_MODER &= ~(3<<(6*2));
    GPIOF_MODER |= (1<<(6*2));
    while (1)
    {
    }
}

void SystemInit()
{
}
```

烧录前记得勾选：use MicroLib.

### 构建库方式

#### 点灯——自己尝试构建库函数版

寄存器方法了解到这里就好，野火课程主要是库函数写代码。首先我们自己尝试构建一下库函数。

还是基于上次实验代码修改即可。首先对 .h 文件做一些修改：

```c
#ifndef __STM32F4XX_H__
#define __STM32F4XX_H__

#endif
```

这个是防止多次引用头文件重复定义。

然后，像之前一条条定义太麻烦了。其实我们注意到每个寄存器都是4字节，我们可以用固定大小的结构体定义。比如 GPIO ABCDEF 结构都一样，我们只需要统一定义结构体和各自的基址即可。

```c
#include<stdint.h>	// 包含了 uint32_t uint16_t
/* GPIO寄存器列表 */
typedef struct
{
    uint32_t MODER;    /*GPIO模式寄存器						地址偏移: 0x00      */
    uint32_t OTYPER;   /*GPIO输出类型寄存器				地址偏移: 0x04      */
    uint32_t OSPEEDR;  /*GPIO输出速度寄存器				地址偏移: 0x08      */
    uint32_t PUPDR;    /*GPIO上拉/下拉寄存器			地址偏移: 0x0C      		*/
    uint32_t IDR;      /*GPIO输入数据寄存器				地址偏移: 0x10      		*/
    uint32_t ODR;      /*GPIO输出数据寄存器				地址偏移: 0x14      		*/
    uint16_t BSRRL;    /*GPIO置位/复位寄存器 低16位部分	地址偏移: 0x18 	*/
    uint16_t BSRRH;    /*GPIO置位/复位寄存器 高16位部分	地址偏移: 0x1A  */
    uint32_t LCKR;     /*GPIO配置锁定寄存器				地址偏移: 0x1C      */
    uint32_t AFR[2];   /*GPIO复用功能配置寄存器		地址偏移: 0x20-0x24 		*/
} GPIO_TypeDef;

# define GPIOF ((GPIO_TypeDef *)GPIOF_BASE)
```

main.c 中可以把对应寄存器替换为 GPIOF->寄存器名了。

然后我们直接对寄存器做操作，还是有点直接了，最好是我们不需要关注寄存器有哪些，直接调用一个 GPIO 设置函数即可使用，封装性可移植性都会好很多。

```c
//stm32f4xx_gpio.c
void GPIO_SetBits(GPIO_TypeDef * GPIOx, uint16_t GPIO_Pin){
    GPIOx->BSRRL=GPIO_Pin;
}

void GPIO_ResetBits(GPIO_TypeDef * GPIOx, uint16_t GPIO_Pin){
    GPIOx->BSRRH=GPIO_Pin;
}

//stm32f4xx_gpio.h
#include "stm32f4xx.h"

#ifndef __STM32F4XX_GPIO_H__
#define __STM32F4XX_GPIO_H__

void GPIO_SetBits(GPIO_TypeDef * GPIOx, uint16_t GPIO_Pin);
void GPIO_ResetBits(GPIO_TypeDef * GPIOx, uint16_t GPIO_Pin);

#endif
```

比如首先我们简单写了这样一个置位函数，使用方法为 `GPIO_SetBits(GPIOF_Base,1<<6)` .

以及我们可以在 `stm32f4xx_gpio.h` 里批量定义：

```c
#define GPIO_Pin_6          (uint16_t)(1<<6)
#define GPIO_Pin_7          (uint16_t)(1<<7)
#define GPIO_Pin_8          (uint16_t)(1<<8)
```

这样 GPIO 使用用 Set Reset 函数已经非常规范了。那么初始化操作我们也可以封装成一个函数。

初始化需要设置 MODER PUPDR OSPEEDR OTYPER，我们可以定义一个结构体用于存储这些初始化变量，初始化的时候新建一个这样的结构体并赋值，传入初始化函数。

```c
//stm32f4xx_gpio.h
//每个模块具体的值可以定义一个枚举类型。
typedef enum
{
    GPIO_Mode_IN=0x00;
    GPIO_Mode_OUT=0x01;
    GPIO_Mode_AF=0x02;
    GPIO_Mode_AN=0x03;
}GPIOMode_TypeDef;

typedef struct
{
    uint16_t GPIO_Pin;
    GPIOMode_TypeDef MODER;
    GPIOPuPd_TypeDef PUPDR;
    GPIOOType_TypeDef OTYPER;
    GPIOOSpeed_TypeDef OSPEEDR;
}GPIO_InitTypeDef;
```

具体使用的时候首先我们初始化一个 GPIO_InitTypeDef 变量，并且给其中的每一个子元素都赋值。然后传入 GPIO_Init 函数中，里面就是一系列根据手册而来的位操作，这里我感觉前面原理懂差不多就不用非跟着敲了。

#### 分析 stm32 固件库函数

前面基本上都是了解固件库编程，从51过渡到 stm32. 后面所有固件编程固件库的使用方法都和前面的 GPIO 类似。

固件是什么？其实就是程序，固化到 EEPROM 或 FLASH 中，操作最底层的设备。**不是具体的应用，而是只操作最底层的设备。比如点灯算应用，给应用工程师提供库函数的工作是固件工程师的。**

stm32 官方 stmf4 固件库下载地址：

[STM32标准外设软件库: 相关产品](https://www.st.com/zh/embedded-software/stm32-standard-peripheral-libraries/products.html?querycriteria=productId=LN1939)

![image-20231108222206160](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/image-20231108222206160.png)

```shell
$ ls
Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         2023/11/8     22:16                Libraries
d-----         2023/11/8     22:16                Project
d-----         2023/11/8     22:17                Utilities
d-----         2023/11/8     22:17                _htmresc
-ar---         2023/11/8     22:15          88007 Package_license.html
-ar---         2023/11/8     22:15          19611 Package_license.md
-ar---         2023/11/8     22:15         152599 Release_Notes.html
-ar---         2023/11/8     22:15       37185187 stm32f4xx_dsp_stdperiph_lib_um.chm
```

.chm：使用帮助文档。

.html .md：一些版本更新，包许可证相关信息。

Utilities：一些第三方其他软件。

Project：样例，模板。

Libraries：库，CMSIS 是一些 ARM 公司的标准，Driver 是固件。inc 是头文件，src 是c文件。

我们根据上节课写的项目来进行库函数文件功能分析。

| 文件名                                  | 所属类别 | 功能                                 |
| --------------------------------------- | -------- | ------------------------------------ |
| startup_stm32f40xx.s                    | 片上外设 | 汇编启动文件                         |
| stm32f4xx.h                             | 片上外设 | 外设寄存器映射                       |
| system_stm32f4xx.c / system_stm32f4xx.h | 片上外设 | 初始化系统时钟                       |
| stm32f4xx_xxxx.c / stm32f4xx_xxxx.h     | 内核     | 内核寄存器映射                       |
| core_cm4.h                              | 内核     | 内核寄存器映射                       |
| core_cmFunc.h / core_cmSimd.h           | 内核     | 内核外设的一些操作函数               |
| misc.c / misc.h                         | 内核     | 中断相关函数（优先级分组，系统中断） |
| stm32f4xx_it.c / stm32f4xx_it.h         | 内核     | 中断服务函数（所有中断入口）         |
| main.c                                  |          | main 函数                            |



1. startupxxxx.s：启动文件。
2. stm32f4xx.h：外设寄存器映射。
3. 跳到 system_Init 函数，这个函数当时我们为了执行只写了一个空函数，而 stm32 官方固件库模板里面是有的，在 system_stm32f4xx.c 里，初始化系统时钟。
4. stm32f4xx.c：具体外设驱动，比如上节课写的 gpio。
5. core_cm4.h：内核寄存器映射。
6. misc：中断。

#### 构建库函数

创建一个通用的模板，后面写程序直接使用这个模板。

```shell
$ ls
Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         2023/11/8     23:27                Libraries
d-----         2023/11/8     23:27                Listing
d-----         2023/11/8     23:27                Output
d-----         2023/11/8     23:27                Project
d-----         2023/11/8     23:27                User
-a----         2020/2/27     13:45            401 keilkill.bat
```

前面都是新建的文件夹，keilkill.bat 是从 keil 编译程序中复制出来的一个脚本，可以删掉中间文件。

把固件库 Lib 里的 CMSIS 和 Driver 文件拷贝到 Libraries 文件夹中。CMSIS 中只保留 Device Include 文件夹。Device 中包含外设相关（比如 stm32f4xx.h system_stm32f4xx），Include 中只包含内核相关。

把 main.c stm32f4xx_it.c / stm32f4xx_it.h stm32f4xx_conf.h 拷贝到 User 文件夹中。

在 Project 文件夹里可以包含多给项目文件，不光只有 Keil 的。比如 IAR 的我们新建一个 IAR 文件夹，Keil 我们新建一个 RVMDK(uv5) 文件夹。RealView 是包含不止 MDK 的开发工具集合的称呼，MDK 是 MCU 开发工具集成包，uVersion 是 IDE，Keil 是公司名字。

uVision 里新建工程，新建在 RVMDK(uv5) 文件夹下。

新建组、添加文件如下：

![image-20231109002542997](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/image-20231109002542997.png)

STM32F4xx_StdPeriph_Driver 添加 STM32F4xx_StdPeriph_Driver/src 下的所有文件，屏蔽掉 dma2d fmc ltdc，后两个是 sd 和 lcd 屏幕组件。

头文件如下：

![image-20231109002823229](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/image-20231109002823229.png)

宏定义如下：

USE_STDPERIPH_DRIVER,STM32F40_41xxx

我现在好想明白为什么宏定义在这里了，这样后面换单片机型号的时候可以直接修改这个宏定义。

**PS：我下载的是 1.8.1 版本 stm32f4xx.h 库函数，里面出现了一段重复定义导致编译产生了200多个 warning。我把下面那一段删掉了就好了。**

![jingqing3948_1-1699463889291.png](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/true)

Output 里设置 Output 文件夹，不然都在 Proj 里太乱。

记得勾选 MicroLib。

#### 点灯——官方库函数版

在 User 文件夹中新建 LED 文件夹，里面新建 bsp_led.c，代表板级支持包 LED 代码，也就是只针对我们当前这一款开发板的点灯程序。

1. 设置时钟：rcc 时钟，在 stm32f4xx_rcc.c 中：

2. ```c
   /**
     * @brief  Enables or disables the AHB1 peripheral clock.
     * @note   After reset, the peripheral clock (used for registers read/write access)
     *         is disabled and the application software has to enable this clock before 
     *         using it.   
     * @param  RCC_AHBPeriph: specifies the AHB1 peripheral to gates its clock.
     *          This parameter can be any combination of the following values:
     *            @arg RCC_AHB1Periph_GPIOA:       GPIOA clock
     *            @arg RCC_AHB1Periph_GPIOB:       GPIOB clock 
     *            @arg RCC_AHB1Periph_GPIOC:       GPIOC clock
     *            @arg RCC_AHB1Periph_GPIOD:       GPIOD clock
     *            @arg RCC_AHB1Periph_GPIOE:       GPIOE clock
     *            @arg RCC_AHB1Periph_GPIOF:       GPIOF clock
     *            @arg RCC_AHB1Periph_GPIOG:       GPIOG clock
     *            @arg RCC_AHB1Periph_GPIOG:       GPIOG clock
     *            @arg RCC_AHB1Periph_GPIOI:       GPIOI clock
     *            @arg RCC_AHB1Periph_GPIOJ:       GPIOJ clock (STM32F42xxx/43xxx devices) 
     *            @arg RCC_AHB1Periph_GPIOK:       GPIOK clock (STM32F42xxx/43xxx devices)  
     *            @arg RCC_AHB1Periph_CRC:         CRC clock
     *            @arg RCC_AHB1Periph_BKPSRAM:     BKPSRAM interface clock
     *            @arg RCC_AHB1Periph_CCMDATARAMEN CCM data RAM interface clock
     *            @arg RCC_AHB1Periph_DMA1:        DMA1 clock
     *            @arg RCC_AHB1Periph_DMA2:        DMA2 clock
     *            @arg RCC_AHB1Periph_DMA2D:       DMA2D clock (STM32F429xx/439xx devices)  
     *            @arg RCC_AHB1Periph_ETH_MAC:     Ethernet MAC clock
     *            @arg RCC_AHB1Periph_ETH_MAC_Tx:  Ethernet Transmission clock
     *            @arg RCC_AHB1Periph_ETH_MAC_Rx:  Ethernet Reception clock
     *            @arg RCC_AHB1Periph_ETH_MAC_PTP: Ethernet PTP clock
     *            @arg RCC_AHB1Periph_OTG_HS:      USB OTG HS clock
     *            @arg RCC_AHB1Periph_OTG_HS_ULPI: USB OTG HS ULPI clock
     * @param  NewState: new state of the specified peripheral clock.
     *          This parameter can be: ENABLE or DISABLE.
     * @retval None
     */
   void RCC_AHB1PeriphClockCmd(uint32_t RCC_AHB1Periph, FunctionalState NewState)
   ```

   其他部分基本也是参照库函数（主要是 stm32f4xx_gpio.h）最终呈现如下：

   ```c
   #include "bsp_led.h"
   
   void LED_GPIO_Config(void){
       //RCC set function in stm32f4xx_rcc.h
       RCC_AHB1PeriphClockCmd(RCC_AHB1Periph_GPIOF,ENABLE);
       {
           //Init structure
           GPIO_InitTypeDef GPIO_InitStructure;
           GPIO_InitStructure.GPIO_Mode=GPIO_Mode_OUT;
           GPIO_InitStructure.GPIO_OType=GPIO_OType_PP;
           GPIO_InitStructure.GPIO_Pin=GPIO_Pin_6;
           GPIO_InitStructure.GPIO_PuPd=GPIO_PuPd_UP;
           GPIO_InitStructure.GPIO_Speed=GPIO_Speed_50MHz;
           //init function
           GPIO_Init(GPIOF,&GPIO_InitStructure);
       }
       
   }
   ```

   置位可以使用 `GPIO_SetBits` 或 `GPIO_ResetBits`。

   ```c
   int main()
   {
       int i;
       LED_GPIO_Config();
       
       
       *(unsigned int *)(0x40021400+0x14)&=~(1<<6); 
     while (1)
     {
         GPIO_ResetBits(GPIOF,GPIO_Pin_6);
         i=12000000;
         while(i--);
         GPIO_SetBits(GPIOF,GPIO_Pin_6);
         i=12000000;
         while(i--);
     }
   }
   ```

   没有上下拉的时候推挽输出会直接被 ODR 值所影响，哪怕没有赋值其中本来的值也会影响。所以推挽输出无上下拉，不置位 LED 也会被点亮，因为 ODR 默认值0.

### 输入——按键点灯

开发板按键电路如下：

![image-20231109191525189](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/image-20231109191525189.png)

按键未按下接地，按下后为高电平。电容起到消抖作用，软件处理就不需要手动延时消抖了。

编程没啥难度，就是改了一下输入模式。使用 `ReadInputDataBits` 读取。

```c
//bsp_button.c
#include "bsp_button.h"

void Button_GPIO_Config(void){
    RCC_AHB1PeriphClockCmd(RCC_AHB1Periph_GPIOA,ENABLE);
    {
        GPIO_InitTypeDef GPIOInitStruct;
        GPIOInitStruct.GPIO_Mode=GPIO_Mode_IN;
        GPIOInitStruct.GPIO_Pin=GPIO_Pin_0;
        GPIOInitStruct.GPIO_OType=GPIO_OType_PP;
        GPIOInitStruct.GPIO_Speed=GPIO_Speed_50MHz;
        GPIOInitStruct.GPIO_PuPd=GPIO_PuPd_NOPULL;
        
        GPIO_Init(GPIOA,&GPIOInitStruct);
    }
}

//main.c
#include "stm32f4xx.h"

#include "bsp_led.h"
#include "bsp_button.h"

int main()
{
    //RCC
    LED_GPIO_Config();
    Button_GPIO_Config();
    while (1)
    {
        if(GPIO_ReadInputDataBit(GPIOA,GPIO_Pin_0))GPIO_SetBits(GPIOF, GPIO_Pin_6);
        else GPIO_ResetBits(GPIOF, GPIO_Pin_6);
    }
}
```

实现按键按下后翻转：

```c
int main()
{
    //RCC
    LED_GPIO_Config();
    Button_GPIO_Config();
    while (1)
    {
        if(GPIO_ReadInputDataBit(GPIOA,GPIO_Pin_0)==Bit_SET){
            while(GPIO_ReadInputDataBit(GPIOA,GPIO_Pin_0)==Bit_SET);
            GPIO_ToggleBits(GPIOF, GPIO_Pin_6);
        }
    }
}
```

要等到按键松开的时候再翻转，轮询直到松开。

### 位带操作

之前51单片机常见位定义。比如 PA 引脚有8个 IO 口，我们可以定义 `sbit LED1=PA^0` 这样单独操作某一位。

stm32 里没有直接的位定义方式。一种解决办法是我们利用与或操作不影响其他位的同时操作特定位；另一种就是位带操作。

stm32 里有一部分别名区域，用于映射外设、SRAM 中特定的位带区，我们操作这一部分别名区域时就可以实现对外设、SRAM 位带区与的位操作。

片上外设位带区：0X4000 0000~0X400F 0000，别名区：0X4200 0000~0X43FF FFFF，包含 APB12，AHB1 外设。

SRAM 位带区：0X2000 0000~0X200F 0000，别名区：0X2200 0000~0X23FF FFFF

![image-20231109220918350](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/image-20231109220918350.png)

外设地址 A 别名地址为：AliasAddr= =0x42000000+ (A-0x40000000)*8*4 +n*4  （n是位序号）

SRAM 地址 A 别名地址为：AliasAddr= =0x22000000+ (A-0x20000000)*8*4 +n*4

扩大了32倍，可以对32位寄存器中的每一位进行操作。

统一公式：`\#define BITBAND(addr, bitnum) ((addr & 0xF0000000)+0x02000000+((addr & 0x000FFFFF)<<5)+(bitnum<<2))`

使用：比如我们操作一个 GPIO 的位操作。

```c
#define BITBAND(addr, bitnum) ((addr & 0xF0000000)+0x02000000+((addr & 0x000FFFFF)<<5)+(bitnum<<2))
// 把一个地址转换成一个指针
#define MEM_ADDR(addr) *((volatile unsigned long *)(addr))
// 把位带别名区地址转换成指针
#define BIT_ADDR(addr, bitnum) MEM_ADDR(BITBAND(addr, bitnum))

// GPIO ODR 和 IDR 寄存器地址映射
#define GPIOF_ODR_Addr (GPIOF_BASE+20)
#define GPIOF_IDR_Addr (GPIOF_BASE+16)

// 单独操作 GPIO 的某一个 IO 口，n(0,1,2...15),
// n 表示具体是哪一个 IO 口
#define PFout(n) BIT_ADDR(GPIOF_ODR_Addr,n) //输出
#define PFin(n) BIT_ADDR(GPIOF_IDR_Addr,n) //输入

//使用示例
PFout(6)= 0;
```

这个概念学学就好，好像实际应用没啥意义。我们微处理器考试也考过hh。
