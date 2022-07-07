---
title: STM32——IIC、SPI通信
tags: [STM32]
category: [嵌入式开发, STM32]
top: 19
index_img: /img/正点原子STM32F103.jpg
excerpt: 这篇文章标志着博主迈出了研究生学习的第一步，凭借大一学习51单片机的基础，正式开启STM32的学习。
---

# IIC通信

I2C(IIC,Inter－Integrated Circuit),两线式串行总线，它是由数据线SDA和时钟SCL构成的串行总线，可发送和接收数据，IIC是半双工通信方式。

## IIC协议

* 空闲状态

  I2C总线总线的SDA和SCL两条信号线同时处于高电平时，规定为总线的空闲状态。

* 开始信号

  l当SCL为高期间，SDA由高到低的跳变；启动信号是一种电平跳变时序信号，而不是一个电平信号。

* 停止信号

  当SCL为高期间，SDA由低到高的跳变；停止信号也是一种电平跳变时序信号，而不是一个电平信号。

  ![](https://s2.loli.net/2022/07/01/H5vuVkwtqP3W2ab.png)

* 应答信号ACK

  发送器每发送一个字节，就在时钟脉冲9期间释放数据线，由接收器反馈一个应答信号。 应答信号为低电平时，规定为有效应答位（ACK简称应答位），表示接收器已经成功地接收了该字节；应答信号为高电平时，规定为非应答位（NACK），一般表示接收器接收该字节没有成功。 

  ![](https://s2.loli.net/2022/07/01/z2LpfcTyJiqRKFY.png)

* 数据的有效性

  I2C总线进行数据传送时，时钟信号为高电平期间，数据线上的数据必须保持稳定，只有在时钟线上的信号为低电平期间，数据线上的高电平或低电平状态才允许变化。

  ![](https://s2.loli.net/2022/07/01/yTSKxDzdNhOmljb.png)

* 数据传输

  在I2C总线上传送的每一位数据都有一个时钟脉冲相对应（或同步控制），即在SCL串行时钟的配合下，在SDA上逐位地串行传送每一位数据。

## EEPROM（24C02）

总容量是256（2K/8)个字节，IIC接口

* 设备地址

  ![](https://s2.loli.net/2022/07/02/JqUteIgwmVQHzxy.png)

* 24C02字节写时序

  ![](https://s2.loli.net/2022/07/02/QH8dcXOl3BjpgGu.png)

* 24C02读时序

  ![](https://s2.loli.net/2022/07/02/QKle2unUTxmvN4p.png)

# SPI通信

SPI 是英语Serial Peripheral interface的缩写，顾名思义就是串行外围设备接口。SPI，是一种高速的，全双工，同步的通信总线，并且在芯片的管脚上只占用四根线。

## SPI接口原理

* SPI内部结构简明图

  ![](https://s2.loli.net/2022/07/02/P8FnoSusOWxDqdM.png)

  **SPI接口一般使用4条线通信：**

  MISO 主设备数据输入，从设备数据输出。

  MOSI 主设备数据输出，从设备数据输入。

  SCLK时钟信号，由主设备产生。

  CS从设备片选信号，由主设备控制。

* 从选择（NSS）脚管理

  不仅可以使用特定的NSS脚控制片选信号，也可以使用任意的IO口控制片选信号。

* 时钟信号的相位和极性

  SPI_CR寄存器的CPOL和CPHA位，能够组合成四种可能的时序关系。

  CPOL位确定时钟在空闲状态下的电平：CPOL位为0时，时钟信号在空闲时为低电平；CPOL位为1时，时钟信号在空闲时为高电平。

  CPHA位确定数据在时钟的第几个边沿被采集：CPHA位为0时，在第一个边沿被采集；CPHA位为1时，在第二个边沿被采集。

* 数据帧格式

  ![](https://s2.loli.net/2022/07/02/ibJY6WmauqTEOSs.png)

   LSB：least significant bit 表示二进制数据的最低位

  MSB : most significant bit 表示二进制数据的最高位

* 状态标志

  ![](https://s2.loli.net/2022/07/02/Lg1Ue4Tj2fsCIni.png)

* SPI中断

  ![](https://s2.loli.net/2022/07/02/8B7GEpzvOLi9In5.png)

* SPI引脚配置

  ![](https://s2.loli.net/2022/07/02/qyh6mjC3AYIcs4n.png)

* 常用寄存器

  ![](https://s2.loli.net/2022/07/02/azwYEnhDM2OIHyi.jpg)

* 程序配置过程

  ```c
  //配置相关引脚的复用功能，使能SPIx时钟
  void GPIO_Init(GPIO_TypeDef* GPIOx, GPIO_InitTypeDef* GPIO_InitStruct);
  //初始化SPIx,设置SPIx工作模式
  void SPI_Init(SPI_TypeDef* SPIx, SPI_InitTypeDef* SPI_InitStruct);
  //使能SPIx
  void SPI_Cmd(SPI_TypeDef* SPIx, FunctionalState NewState);
  //SPI传输数据
  void SPI_I2S_SendData(SPI_TypeDef* SPIx, uint16_t Data);
  uint16_t SPI_I2S_ReceiveData(SPI_TypeDef* SPIx) ;
  //查看SPI传输状态
  SPI_I2S_GetFlagStatus(SPI2, SPI_I2S_FLAG_RXNE);
  ```

## W25QXX芯片

W25Q128(W25Q64)将16M(8M)的容量分为256 (128)个块（Block），每个块大小为64K字节，每个块又分为16个扇区（Sector），每个扇区4K个字节。W25Qxx的最小擦除单位为一个扇区，也就是每次必须擦除4K个字节。

W25QXX芯片的操作需要不断查看芯片的数据手册。
