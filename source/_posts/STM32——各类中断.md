---
title: STM32——各类中断
tags: [STM32]
category: [嵌入式开发, STM32]
top: 18
index_img: /img/正点原子STM32F103.jpg
excerpt: 这篇文章标志着博主迈出了研究生学习的第一步，凭借大一学习51单片机的基础，正式开启STM32的学习。
---

# 各类中断

## 外部中断

* STM32的每个IO都可以作为外部中断输入。

* STM32的中断控制器支持19个外部中断/事件请求：
  线0~15：对应外部IO口的输入中断。
  
  ![](https://s2.loli.net/2022/06/21/L2tBESiCdhoHV4b.jpg)
  
  线16：连接到PVD输出。
  线17：连接到RTC闹钟事件。
  线18：连接到USB唤醒事件。

* IO口外部中断在中断向量表中只分配了7个中断向量，也就是只能使用7个中断服务函数

* 中断服务函数列表

  ```c
  EXTI0_IRQHandler           
  EXTI1_IRQHandler
  EXTI2_IRQHandler           
  EXTI3_IRQHandler           
  EXTI4_IRQHandler           
  EXTI9_5_IRQHandler         
  EXTI15_10_IRQHandler       
  ```
  
* 外部中断常用库函数

  ```c
  void GPIO_EXTILineConfig(uint8_t GPIO_PortSource, uint8_t GPIO_PinSource);
     //设置IO口与中断线的映射关系
  
     exp:  GPIO_EXTILineConfig(GPIO_PortSourceGPIOE,GPIO_PinSource2);
  
  void EXTI_Init(EXTI_InitTypeDef* EXTI_InitStruct);
   //初始化中断线：触发方式等
  
  ITStatus EXTI_GetITStatus(uint32_t EXTI_Line);
  //判断中断线中断状态，是否发生
  
  void EXTI_ClearITPendingBit(uint32_t EXTI_Line);
  //清除中断线上的中断标志位
  ```

* EXTI_Init函数

  ```c
  void EXTI_Init(EXTI_InitTypeDef* EXTI_InitStruct)；
  typedef struct
  {
    uint32_t EXTI_Line;   //指定要配置的中断线           
    EXTIMode_TypeDef EXTI_Mode;   //模式：事件 OR中断
    EXTITrigger_TypeDef EXTI_Trigger;//触发方式：上升沿/下降沿/双沿触发
    FunctionalState EXTI_LineCmd;  //使能 OR失能
  }EXTI_InitTypeDef;
  ```

* 外部中断的一般配置步骤

  ![](https://s2.loli.net/2022/06/21/y2EqUtG3z4ieCAl.jpg)

## 看门狗🐕

### 独立看门狗

* 在键值寄存器（**IWDG_KR**）中写入0xCCCC，开始启用独立看门狗。此时计数器开始从其复位值0xFFF递减，当计数器值计数到尾值0x000时会产生一个复位信号（IWDG_RESET）。

* 无论何时，只要在键值寄存器（**IWDG_KR**）中写入0xAAAA（通常说的喂狗）, 自动重装载寄存器IWDG_RLR的值就会重新加载到计数器，从而避免看门狗复位。

* 如果程序异常，就无法正常喂狗，从而系统复位。

* 键值寄存器IWDG_KR: 0~15位有效

  ![](https://s2.loli.net/2022/06/22/zsUMBmvEH1RCr8n.png)

* 预分频寄存器IWDG_PR：0~2位有效。具有写保护功能，要操作先取消写保护

  ![](https://s2.loli.net/2022/06/22/qG8VWswOdaT2leX.png)

* 重装载寄存器IWDG_RLR：0~11位有效。具有写保护功能，要操作先取消写保护。

  ![](https://s2.loli.net/2022/06/22/5VHZsbEYzdDQL3F.png)

* 状态寄存器IWDG_SR：0~1位有效

  ![](https://s2.loli.net/2022/06/22/9NXcy6FOptIJ5iV.png)

* 独立看门狗超时时间

  ![](https://s2.loli.net/2022/06/22/asR8hq2rTjibXPd.png)

  溢出时间计算：

    *Tout=((4×2^prer)×rlr)/40*  (M3)
  
  其中，Tout为看门狗溢出时间（单位为 ms） prer为看门狗时钟预分频值（ IWDG_PR值），范围为 0~7 rlr为看门狗的重装载值（ IWDG_RLR的值）；

* IWDG独立看门狗操作库函数

  ```c
  void IWDG_WriteAccessCmd(uint16_t IWDG_WriteAccess);//取消写保护：0x5555使能
  void IWDG_SetPrescaler(uint8_t IWDG_Prescaler);//设置预分频系数：写PR
  void IWDG_SetReload(uint16_t Reload);//设置重装载值：写RLR
  void IWDG_ReloadCounter(void);//喂狗：写0xAAAA到KR
  void IWDG_Enable(void);//使能看门狗：写0xCCCC到KR
  FlagStatus IWDG_GetFlagStatus(uint16_t IWDG_FLAG);//状态：重装载/预分频 更新
  ```

* 独立看门狗操作步骤

  ![](https://s2.loli.net/2022/06/22/ewvgHlyCQROZouD.jpg)

### 窗口看门狗

之所以称为窗口就是因为其喂狗时间是一个有上下限的范围内(窗口），可以通过设定相关寄存器，设定其上限时间（下限固定）。喂狗的时间不能过早也不能过晚。

![](https://s2.loli.net/2022/06/22/yI9a1XBegxY8PpQ.png)

* 窗口看门狗超时时间

  ![](https://s2.loli.net/2022/06/22/kIKhWvQoDJUlFXL.png)

* 控制寄存器**WWDG_CR**

  ![](https://s2.loli.net/2022/06/22/6ldQkh9fCjZWrw7.png)

  ```c
  void WWDG_Enable(uint8_t Counter);//启动并设置初始值
  void WWDG_SetCounter(uint8_t Counter);//喂狗
  ```

* 配置寄存器WWDG_CFR

  ![](https://s2.loli.net/2022/06/22/FESBmlxQtwpG6aj.png)

  ```c
  void WWDG_EnableIT(void);//使能提前唤醒中断
  void WWDG_SetPrescaler(uint32_t WWDG_Prescaler);
  void WWDG_SetWindowValue(uint8_t WindowValue);
  ```

* 状态寄存器WWDG_SR

  ![](https://s2.loli.net/2022/06/22/Iq5rCw1UnSXcdeO.png)

  ```c
  FlagStatus WWDG_GetFlagStatus(void);
  void WWDG_ClearFlag(void);
  ```

* 窗口看门狗配置过程

  ![](https://s2.loli.net/2022/06/22/2Z1wRgeSMsp3Nba.jpg)

## 通用定时器

### 定时器的功能特点

* 位于低速的APB1总线上(APB1)
* 16 位向上、向下、向上/向下(中心对齐)计数模式，自动装载计数器（TIMx_CNT）。
* 16 位可编程(可以实时修改)预分频器(TIMx_PSC)，计数器时钟频率的分频系数 为 1～65535 之间的任意数值。
* 4 个独立通道（TIMx_CH1~4），这些通道可以用来作为： 
  * 输入捕获 
  * 输出比较
  * PWM 生成(边缘或中间对齐模式) 
  * 单脉冲模式输出

* 可使用外部信号（TIMx_ETR）控制定时器和定时器互连（可以用 1 个定时器控制另外一个定时器）的同步电路。

### 产生中断条件 

* 更新：计数器向上溢出/向下溢出，计数器初始化(通过软件或者内部/外部触发) 

* 触发事件(计数器启动、停止、初始化或者由内部/外部触发计数) 

* 输入捕获 

* 输出比较 

* 支持针对定位的增量(正交)编码器和霍尔传感器电路 

* 触发输入作为外部时钟或者按周期的电流管理

STM32 的通用定时器可以被用于：**测量输入信号的脉冲长度(输入捕获)或者产生输出波形(输出比较和 PWM)等。 **

### 计数器模式

通用定时器可以向上计数、向下计数、向上向下双向计数模式。

### 工作流程图

![](https://s2.loli.net/2022/07/04/UlQcE5Xa6WYm8IP.png)

### 内部时钟选择

![](https://s2.loli.net/2022/07/04/XVEq68IbgQtJHef.jpg)

### 定时器中断实验相关寄存器（核心）

* 计数器当前值寄存器CNT（修改该寄存器，定时器重新计时）

  ![](https://s2.loli.net/2022/07/04/eEyvTiLpUQRPY3G.png)

* 预分频寄存器TIMx_PSC

  ![](https://s2.loli.net/2022/07/04/ve1wI6GCLR9Mm2Q.png)

* 自动重装载寄存器（TIMx_ARR)

  ![](https://s2.loli.net/2022/07/04/myzgLZBRoYOTFJG.png)

* 控制寄存器1（TIMx_CR1）

  ![](https://s2.loli.net/2022/07/04/YOosxQlqHcgvM83.png)

* DMA中断使能寄存器（TIMx_DIER）

  ![](https://s2.loli.net/2022/07/04/yeBjxwJYu967Hls.png)

### 定时器中断实现步骤

```c
//能定时器时钟。
RCC_APB1PeriphClockCmd();
//初始化定时器，配置ARR,PSC。
TIM_TimeBaseInit();
//开启定时器中断，配置NVIC。
void TIM_ITConfig();
NVIC_Init();
//使能定时器。
TIM_Cmd();
//编写中断服务函数。
TIMx_IRQHandler();
```
