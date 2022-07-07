---
title: STM32——时钟系统
tags: [STM32]
category: [嵌入式开发, STM32]
top: 16
index_img: /img/正点原子STM32F103.jpg
excerpt: 这篇文章标志着博主迈出了研究生学习的第一步，凭借大一学习51单片机的基础，正式开启STM32的学习。
---

# STM32——时钟系统

## 时钟系统框图

![](https://s2.loli.net/2022/05/28/Flc5V7svKOeCJMT.png)

* STM32 有5个时钟源:HSI、HSE、LSI、LSE、PLL
  ① HSI是高速内部时钟，RC振荡器，频率为8MHz，精度不高
  ② HSE是高速外部时钟，可接石英/陶瓷谐振器，或者接外部时钟源，频率范围为4MHz~16MHz
  ③ LSI是低速内部时钟，RC振荡器，频率为40kHz，提供低功耗时钟，WDG
  ④ LSE是低速外部时钟，接频率为32.768kHz的石英晶体，RTC
  ⑤ PLL为锁相环倍频输出，其时钟输入源可选择为HSI/2、HSE或者HSE/2，倍频可选择为2~16倍，但是其输出频率最大不得超过72MHz

* 系统时钟SYSCLK可来源于三个时钟源：
  ① HSI振荡器时钟
  ② HSE振荡器时钟
  ③ PLL时钟
* STM32可以选择一个时钟信号输出到MCO脚(PA8)上，可以选择为PLL输出的2分频、HSI、HSE、或者系统时钟
* 任何一个外设在使用之前，必须首先使能其相应的时钟

### 蓝色方框（时钟源）：

* HSI RC（内部高速时钟），频率约等于8MHz（由RC振荡器产生的时钟，不十分稳定，可关闭）
* LSI RC（内部低速时钟），频率约等于40KHz，提供给IW DGCLK独立看门狗时钟
* HSE OSC（外部高速时钟），外接晶振产生4-16MHz
* LSE OSC（外部低速时钟），外接晶振32.768KHz，提供给RTC内部实时时钟单元
* PLL（锁相环），可以用于倍频

### 灰色梯形（选择器）

### 黄色方框（时钟监视系统）
一旦HSE失效自动将系统时钟切换至HSI

### MCO输出内部时钟

### 绿色方框（外设）

* USB时钟，通常48MHz，可预分频 ÷1 or ÷1.5
* AHB预分频器，能产生HCLK时钟
* APB1预分频器，能产生PCLK1时钟，给低频外设提供时钟
* APB2预分频器，能产生PCLK2时钟，给高频外设提供时钟

## RCC相关配置寄存器


```c
typedef struct
{
  __IO uint32_t CR;                //HSI,HSE,CSS,PLL等的使能和就绪标志位 
  __IO uint32_t CFGR;           //PLL等的时钟源选择，分频系数设定
  __IO uint32_t CIR;               // 清除/使能 时钟就绪中断
  __IO uint32_t APB2RSTR;  //APB2线上外设复位寄存器
  __IO uint32_t APB1RSTR;   //APB1线上外设复位寄存器
  __IO uint32_t AHBENR;    //常用使能，DMA，SDIO等时钟使能
  __IO uint32_t APB2ENR;   //常用使能，APB2线上外设时钟使能
  __IO uint32_t APB1ENR;   //常用使能，APB1线上外设时钟使能
  __IO uint32_t BDCR;        //备份域控制寄存器
  __IO uint32_t CSR;           //控制状态寄存器
} RCC_TypeDef;
```

### RCC相关头文件和固件库源文件

![](https://s2.loli.net/2022/05/28/3qkf4WHKbI1ARQX.jpg)

## SystemInit时钟系统初始化函数

```c
void SystemInit (void)
{
  /* Reset the RCC clock configuration to the default reset state(for debug purpose) */
  /* Set HSION bit */
  RCC->CR |= (uint32_t)0x00000001;  //打开HSI RC（内部高速时钟）

  /* Reset SW, HPRE, PPRE1, PPRE2, ADCPRE and MCO bits */
#ifndef STM32F10X_CL     
  RCC->CFGR &= (uint32_t)0xF8FF0000;
#else
  RCC->CFGR &= (uint32_t)0xF0FF0000;
#endif /* STM32F10X_CL */   
  
  /* Reset HSEON, CSSON and PLLON bits */
  RCC->CR &= (uint32_t)0xFEF6FFFF;

  /* Reset HSEBYP bit */
  RCC->CR &= (uint32_t)0xFFFBFFFF;

  /* Reset PLLSRC, PLLXTPRE, PLLMUL and USBPRE/OTGFSPRE bits */
  RCC->CFGR &= (uint32_t)0xFF80FFFF;

#ifdef STM32F10X_CL
  /* Reset PLL2ON and PLL3ON bits */
  RCC->CR &= (uint32_t)0xEBFFFFFF;

  /* Disable all interrupts and clear pending bits  */
  RCC->CIR = 0x00FF0000;

  /* Reset CFGR2 register */
  RCC->CFGR2 = 0x00000000;
#elif defined (STM32F10X_LD_VL) || defined (STM32F10X_MD_VL) || (defined STM32F10X_HD_VL)
  /* Disable all interrupts and clear pending bits  */
  RCC->CIR = 0x009F0000;

  /* Reset CFGR2 register */
  RCC->CFGR2 = 0x00000000;      
#else
  /* Disable all interrupts and clear pending bits  */
  RCC->CIR = 0x009F0000;
#endif /* STM32F10X_CL */
    
#if defined (STM32F10X_HD) || (defined STM32F10X_XL) || (defined STM32F10X_HD_VL)
  #ifdef DATA_IN_ExtSRAM
    SystemInit_ExtMemCtl(); 
  #endif /* DATA_IN_ExtSRAM */
#endif 

  /* Configure the System clock frequency, HCLK, PCLK2 and PCLK1 prescalers */
  /* Configure the Flash Latency cycles and enable prefetch buffer */
  SetSysClock();

#ifdef VECT_TAB_SRAM
  SCB->VTOR = SRAM_BASE | VECT_TAB_OFFSET; /* Vector Table Relocation in Internal SRAM. */
#else
  SCB->VTOR = FLASH_BASE | VECT_TAB_OFFSET; /* Vector Table Relocation in Internal FLASH. */
#endif 
}
```

## SysTick定时器

Systick定时器就是系统滴答定时器，一个24 位的倒计数定时器，计到0 时，将从RELOAD 寄存器中自动重装载定时初值。

### 4个寄存器

* SysTick 控制和状态寄存器（CTRL）

  ![](https://s2.loli.net/2022/05/30/duKBZSHLIqXfCzT.png)

  对于STM32，外部时钟源是 HCLK(AHB总线时钟)的1/8内核时钟是 HCLK时钟；
  配置函数：SysTick_CLKSourceConfig()

* SysTick 重装载数值寄存器（LOAD）

  ![](https://s2.loli.net/2022/05/30/8LFcpaQfeW7dbi9.png)
  
* SysTick 当前值寄存器（VAL）

  ![](https://s2.loli.net/2022/06/01/vaYoqI94LygiJAN.png)

### 固件库中的Systick相关函数
* SysTick_CLKSourceConfig()    //Systick时钟源选择  misc.c文件中

  ```c
  void SysTick_CLKSourceConfig(uint32_t SysTick_CLKSource)
  {
    /* Check the parameters */
    assert_param(IS_SYSTICK_CLK_SOURCE(SysTick_CLKSource));
    if (SysTick_CLKSource == SysTick_CLKSource_HCLK)
    {
      SysTick->CTRL |= SysTick_CLKSource_HCLK;
    }
    else
    {
      SysTick->CTRL &= SysTick_CLKSource_HCLK_Div8;
    }
  }
  ```

* SysTick_Config()    //初始化并启动SysTick计数器及其中断，ticks就是LOAD值，即重载值，表示两次中断的计数。

  ```c
  static __INLINE uint32_t SysTick_Config(uint32_t ticks)
  { 
    if (ticks > SysTick_LOAD_RELOAD_Msk)  return (1);            /* Reload value impossible */                             
    SysTick->LOAD  = (ticks & SysTick_LOAD_RELOAD_Msk) - 1;      /* set reload register */
    NVIC_SetPriority (SysTick_IRQn, (1<<__NVIC_PRIO_BITS) - 1);  /* set Priority for Cortex-M0 System Interrupts */
    SysTick->VAL   = 0;                                          /* Load the SysTick Counter Value */
    SysTick->CTRL  = SysTick_CTRL_CLKSOURCE_Msk | 
                     SysTick_CTRL_TICKINT_Msk   | 
                     SysTick_CTRL_ENABLE_Msk;                    /* Enable SysTick IRQ and SysTick Timer */
    return (0);                                                  /* Function successful */
  }
  ```

* Systick中断服务函数，SysTick_Handler(void);

