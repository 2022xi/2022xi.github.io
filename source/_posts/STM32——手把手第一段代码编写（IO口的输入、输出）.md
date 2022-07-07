---
title: STM32——手把手第一段代码编写（IO口的输入、输出）
tags: [STM32]
category: [嵌入式开发, STM32]
top: 15
index_img: /img/正点原子STM32F103.jpg
excerpt: 这篇文章标志着博主迈出了研究生学习的第一步，凭借大一学习51单片机的基础，正式开启STM32的学习。
---

# STM32——手把手第一段代码编写（跑马灯IO口输出）

## 库函数版本

### IO口操作流程

*  使能IO口时钟，调用函数RCC_APB2PeriphColckCmd(), 不同的IO组，调用的时钟使能函数不一样

  ```c
  RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA|RCC_APB2Periph_GPIOD, ENABLE);	 //使能PB,PE端口时钟
  ```

* 初始化IO口模式，调用函数GPIO_Init()

  ```c
  GPIO_InitTypeDef  GPIO_InitStructure;
  GPIO_InitStructure.GPIO_Pin = GPIO_Pin_8;				 //LED0-->PB.5 端口配置
  GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP; 		 //推挽输出
  GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;		 //IO口速度为50MHz
  GPIO_Init(GPIOA, &GPIO_InitStructure);					 //根据设定参数初始化GPIOB.5
  ```

*  操作IO口，输出高低电平

  ```c
  GPIO_SetBits(GPIOA,GPIO_Pin_8);						 //PA.8 输出高
  GPIO_ResetBits(GPIOA,GPIO_Pin_8);				     //PA.8 输出低 
  ```

### 硬件驱动程序编写

通常会同时编写一个.c文件和.h文件。

* 编写.h文件时避免头文件被重复引用

  ```c
  #ifndef __XX_H     //若文件中没有引用xx.h文件，则执行#define __XX_H
  #define __XX_H
  
  
  #endif             //若文件中已经引用xx.h文件，则中间内容不被执行
  ```

* 编写.c文件时，按照IO口操作流程进行初始化

## 寄存器版本

### IO口操作流程

* 使能IO口时钟，配置寄存器RCC_APB2ENR

  ```c
  #include "stm32f10x.h"
  RCC->APB2ENR|=1<<6+1<<3 ;
  ```

* 初始化IO口模式，配置寄存器GPIOx_CRH/CRL

  ```c
  //GPIOB.5设置成推挽输出
  GPIOB->CRL&=0xFF0FFFFF;
  GPIOB->CRL|=0x00300000;
  //GPIOE.9设置成推挽输出
  GPIOE->CRH&=0xFFFFFFF0;
  GPIOE->CRH|=0x00000003;
  ```
  
*  操作IO口，输出高低电平，配置寄存器GPIOX_ODR或者BSRR/BRR

  ```c
  GPIOB->ODR|=1<<5;
  ```
  
  ## 位操作版本
  
   IO口操作流程
  
  * 使能IO口时钟，调用函数RCC_APB2PeriphColckCmd()
  
  * 初始化IO口模式，调用函数GPIO_Init()
  
  * 操作IO口，输出高低电平，使用位带操作

# STM32——手把手第一段代码编写（按键IO口输入）

### IO口操作流程

* 使能按键对应IO口时钟，调用函数：RCC_APB2PeriphClockCmd()

* 初始化IO模式：上拉/下拉输入，调用函数：GPIO_Init()

* 扫描IO口电平（库函数/寄存器/位操作）

```c
// 库函数版本
RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE)；
GPIO_InitTypeDef  GPIO_InitStructure;
GPIO_InitStructure.GPIO_Pin = GPIO_Pin_5;				 //按键0-->PA.5 端口配置
GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU; 		     //上拉输入
GPIO_Init(GPIOA, &GPIO_InitStructure);					 //根据设定参数初始化GPIOA.5
GPIO_ReadInputDataBit(GPIOA,GPIO_Pin_5)//读取按键0

// 寄存器版本
void KEY_Init(void)
{
	RCC->APB2ENR|=1<<2;     //使能PORTA时钟
	GPIOA->CRL&=0XFF0FFFFF;	//PC5设置成输入	  
	GPIOA->CRL|=0X00800000;   
	GPIOA->ODR|=1<<5;	   	//PC5上拉 
} 
```

### 按键扫描思路

![](https://s2.loli.net/2022/05/28/VtMHro6bONqluZK.png)

记录电平跳变并持续一小段时间，则可判断按键按下。

# MDK中寄存器地址名称映射

![](https://s2.loli.net/2022/05/28/kp7mhwo2ldsiKRj.jpg)
