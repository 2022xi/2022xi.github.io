---
title: STM32——串口通信
tags: [STM32]
category: [嵌入式开发, STM32]
top: 17
index_img: /img/正点原子STM32F103.jpg
excerpt: 这篇文章标志着博主迈出了研究生学习的第一步，凭借大一学习51单片机的基础，正式开启STM32的学习。
---

# STM32——串口通信

## 端口复用配置过程

* GPIO端口时钟使能

  ```c
  RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
  ```

* 复用外设时钟使能

  ```c
  RCC_APB2PeriphClockCmd(RCC_APB2Periph_USART1, ENABLE);
  ```

* 端口模式配置， GPIO_Init()函数

    查表：《STM32中文参考手册V10》P110的表格“8.1.11外设的GPIO配置”

  ![](https://s2.loli.net/2022/06/06/QkgwiK4VnDqS5rx.png)

### PA9,PA10复用为串口1配置过程

```c
RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);//①IO时钟使能

RCC_APB2PeriphClockCmd(RCC_APB2Periph_USART1, ENABLE);//②外设时钟使能

//③初始化IO为对应的模式
GPIO_InitStructure.GPIO_Pin = GPIO_Pin_9; //PA.9//复用推挽输出
GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_PP; 
GPIO_Init(GPIOA, &GPIO_InitStructure);
  
GPIO_InitStructure.GPIO_Pin = GPIO_Pin_10;//PA10 PA.10 浮空输入
GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IN_FLOATING;//浮空输入
GPIO_Init(GPIOA, &GPIO_InitStructure);  
```

## 端口重映射配置过程（串口1为例）

* 使能GPIO时钟（重映射后的IO);

* 使能功能外设时钟（例如串口1);

* 使能AFIO时钟（复用功能时钟），重映射必须使能AFIO时钟

  ```c
  RCC_APB2PeriphClockCmd(RCC_APB2Periph_AFIO, ENABLE);
  ```
  
  ![](https://s2.loli.net/2022/06/06/WPbwk4N6ZTJrQUE.jpg)


* 开启重映射

  ```c
  GPIO_PinRemapConfig(GPIO_Remap_USART1, ENABLE);
  ```

  根据第一个参数，来确定是部分重映射还是全部重映射

## 中断优先级管理NVIC

首先，对STM32中断进行分组，组0~4。同时，对每个中断设置一个抢占优先级和一个响应优先级值。

* 分组配置是在寄存器SCB->AIRCR中配置：

  ![](https://s2.loli.net/2022/06/06/yUweVIX75obKEOH.jpg)

* 抢占优先级 & 响应优先级区别：
  * 高优先级的抢占优先级是可以打断正在进行的低抢占优先级中断的
  * 抢占优先级相同的中断，高响应优先级不可以打断低响应优先级的中断
  * 抢占优先级相同的中断，当两个中断同时发生的情况下，哪个响应优先级高，哪个先执行
  * 如果两个中断的抢占优先级和响应优先级都是一样的话，则看哪个中断先发生就先执行

* 中断优先级分组函数： 

  ```c
  void NVIC_PriorityGroupConfig(uint32_t NVIC_PriorityGroup)
  {
    assert_param(IS_NVIC_PRIORITY_GROUP(NVIC_PriorityGroup));
    SCB->AIRCR = AIRCR_VECTKEY_MASK | NVIC_PriorityGroup;
  }
  ```

* 中断参数初始化函数

  ```C
  void NVIC_Init(NVIC_InitTypeDef* NVIC_InitStruct);
  ```

* 中断优先级设置步骤

  * 系统运行后先设置中断优先级分组

    ```C
    void NVIC_PriorityGroupConfig(uint32_t NVIC_PriorityGroup);
    //整个系统执行过程中，只设置一次中断分组。
    ```
  
  * 针对每个中断，设置对应的抢占优先级和响应优先级：
  
    ```C
    void NVIC_Init(NVIC_InitTypeDef* NVIC_InitStruct);
    ```
  
  * 如果需要挂起/解挂，查看中断当前激活状态，分别调用相关函数

## 串口通信UART

### 常见的串行通信接口：

![](https://s2.loli.net/2022/06/07/GthL4ycmQzYdJeN.jpg)

### STM32的串口通信接口

大容量STM32F10x系列芯片，包含3个USART和2个UART

* UART:通用异步收发器
* USART:通用同步异步收发器

### UART异步通信方式引脚

* -RXD:数据输入引脚，数据接受。
* -TXD:数据发送引脚，数据发送。

![](https://s2.loli.net/2022/06/07/d6jFtkirhPKGCle.png)

### STM32串口异步通信需要定义的参数

* 起始位
* 数据位（8位或者9位）
* 奇偶校验位（第9位）
* 停止位（1,15,2位）
* 波特率设置

![](https://s2.loli.net/2022/06/07/XuqSPBKOeo7UxJh.png)

## 串口通信寄存器/库函数配置

### 常用的串口相关寄存器

* USART_SR状态寄存器

* USART_DR数据寄存器

* USART_BRR波特率寄存器

  **波特率计算方法：**

  ![](https://s2.loli.net/2022/06/07/m8LPtVbcCihuYHv.png)

* USART_CR使能寄存器

### 串口配置的一般步骤

![](https://s2.loli.net/2022/06/07/Uq5ZfjlTMhFSeE6.jpg)

