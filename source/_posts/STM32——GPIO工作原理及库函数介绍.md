---
title: STM32——GPIO工作原理及库函数介绍
tags: [STM32]
category: [嵌入式开发, STM32]
top: 14
index_img: /img/正点原子STM32F103.jpg
excerpt: 这篇文章标志着博主迈出了研究生学习的第一步，凭借大一学习51单片机的基础，正式开启STM32的学习。
---

# STM32——GPIO工作原理

![GPIO基本结构](https://s2.loli.net/2022/05/25/lv6ihsOFrq5dKHp.png)

## GPIO工作模式

* 4种输入模式：输入浮空、 输入上拉、输入下拉、模拟输入
* 4种输出模式：开漏输出（当NMOS管导通时，输出低电平；当NMOS管关断时，呈现高阻态）、开漏复用功能、推挽式输出（当PMOS管导通时，输出高电平；当NMOS管导通时，呈现低电平）、推挽式复用功能
* 3种最大翻转速度：-2MHZ、-10MHz、-50MHz

## GPIO相关配置寄存器

### 两个32位配置寄存器(GPIOx_CRL,GPIOx_CRH)

GPIOx_CRL 端口配置低寄存器 ，GPIOx_CRH 端口配置高寄存器。

![](https://s2.loli.net/2022/05/25/7ibaHkBnDUuPRKE.png)

### 两个32位数据寄存器(GPIOx_IDR,GPIOx_ODR)

GPIOx_IDR端口输入寄存器，寄存器位上的值为对应IO口的状态：

![](https://s2.loli.net/2022/05/25/uC32OFjBmPWpfIM.png)

GPIOx_ODR端口输出寄存器，寄存器位上的值为对应IO口的输出电平：

![](https://s2.loli.net/2022/05/25/h7XbC31ByLj4OUl.png)

### 一个32位置位/ 复位寄存器(GPIOx_BSRR)

可单独修改某一IO口的输出：

![](https://s2.loli.net/2022/05/25/s4UEIyTmNcFOhAu.png)

### 一个16位复位寄存器(GPIOx_BRR)

该寄存器实际上和GPIOx_BSRR高16位具有同样的功能：

![](https://s2.loli.net/2022/05/25/jU6Yr1ukwORmoa7.png)

### 一个32位锁定寄存器(GPIOx_LCKR)

# STM32——GPIO库函数介绍

## 1个初始化函数GPIO_Init：

### GPIO_Init函数

```c
void GPIO_Init(GPIO_TypeDef* GPIOx, GPIO_InitTypeDef* GPIO_InitStruct);
```

作用：初始化一个或者多个IO口（同一组）的工作方式和速度。

该函数主要是操作GPIO_CRL(CRH)寄存器，在上拉或者下拉 的时候有设置BSRR或者BRR寄存器。

  GPIOx: GPIOA~GPIOG

```c
typedef struct{     //GPIO的七个寄存器
  __IO uint32_t CRL;
  __IO uint32_t CRH;
  __IO uint32_t IDR;
  __IO uint32_t ODR;
  __IO uint32_t BSRR;
  __IO uint32_t BRR;
  __IO uint32_t LCKR;
} GPIO_TypeDef;

typedef struct{
    uint16_t GPIO_Pin;            //指定要初始化的IO口         
    GPIOSpeed_TypeDef GPIO_Speed; //设置IO口输出速度
    GPIOMode_TypeDef GPIO_Mode;    //设置工作模式：8种中的一个
}GPIO_InitTypeDef;
```

### GPIO_Init函数初始化样例

```c
GPIO_InitTypeDef  GPIO_InitStructure;
GPIO_InitStructure.GPIO_Pin = GPIO_Pin_5; //LED0-->PB.5 端口配置
GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;  //推挽输出
GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz; //IO口速度为50MHz
GPIO_Init(GPIOB, &GPIO_InitStructure);	 //根据设定参数初始化GPIOB.5
```

## 2个读取输入电平函数

### GPIO_ReadInputDataBit函数

```c
uint8_t GPIO_ReadInputDataBit(GPIO_TypeDef* GPIOx, uint16_t* GPIO_Pin);
```

作用：读取某个GPIO的输入电平。实际操作的是GPIOx_IDR寄存器。

例如：GPIO_ReadInputDataBit(GPIOA, GPIO_Pin_5);//读取GPIOA.5的输入电平。

### GPIO_ReadInputData函数

```c
uint16_t GPIO_ReadInputData(GPIO_TypeDef* GPIOx);
```

作用：读取某组GPIO的输入电平。实际操作的是GPIOx_IDR寄存器。

例如：GPIO_ReadInputData(GPIOA);//读取GPIOA组中所有io口输入电平。

## 2个读取输出电平函数

### GPIO_ReadOutputDataBit函数

```c
uint8_t GPIO_ReadOutputDataBit(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin);
```

作用：读取某个GPIO的输出电平。实际操作的是GPIO_ODR寄存器。

例如：GPIO_ReadOutputDataBit(GPIOA, GPIO_Pin_5);//读取GPIOA.5的输出电平。

### GPIO_ReadOutputData函数

```c
uint16_t GPIO_ReadOutputData(GPIO_TypeDef* GPIOx);
```

作用：读取某组GPIO的输出电平。实际操作的是GPIO_ODR寄存器。

例如：GPIO_ReadOutputData(GPIOA);//读取GPIOA组中所有io口输出电平。

## 4个设置输出电平函数

### GPIO_SetBits函数

```c
void GPIO_SetBits(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin);
```

作用：设置某个IO口输出为高电平（1），实际操作BSRR寄存器。

### GPIO_ResetBits函数

```c
void GPIO_ResetBits(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin);
```

作用：设置某个IO口输出为低电平（0），实际操作的BRR寄存器。

### GPIO_WriteBit，GPIO_Write函数

```c
void GPIO_WriteBit(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin, BitAction BitVal);
void GPIO_Write(GPIO_TypeDef* GPIOx, uint16_t PortVal);
```

这两个函数不常用，也是用来设置IO口输出电平。

# STM32 命名规则

![STM32 命名规则](https://s2.loli.net/2022/05/24/6FjI35sY8vGnfOy.png)
