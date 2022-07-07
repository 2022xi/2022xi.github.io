---
title: STM32——MQTT协议
tags: [STM32]
category: [嵌入式开发, STM32]
top: 20
index_img: /img/正点原子STM32F103.jpg
excerpt: 这篇文章标志着博主迈出了研究生学习的第一步，凭借大一学习51单片机的基础，正式开启STM32的学习。
---

# STM32——MQTT协议

**工作流程：**

![](https://s2.loli.net/2022/07/04/3Nlin8AvFfjZ7yC.jpg)

## 阿里云报文

### 14种报文概述

![](https://s2.loli.net/2022/07/04/yHYTeBpLPv5MVX6.jpg)

* 固定报头格式

  ![](C:\Users\math2\AppData\Roaming\Typora\typora-user-images\image-20220705165447628.png)

* 剩余长度

  剩余长度（Remaining Length）表示当前报文剩余部分的字节数，包括可变报头和负载的数据。剩余长度不包括用于编码剩余长度字段本身的字节数。

  ![](https://s2.loli.net/2022/07/05/ihNRqOUGScXAz67.png)

  剩余长度字段使用一个变长度编码方案，对小于128的值它使用单字节编码。更大的值按下面的方式处理。低7位有效位用于编码数据，最高有效位用于指示是否有更多的字节。因此每个字节可以编码128个数值和一个延续位（continuation bit）。剩余长度字段最大4个字节。

### 报文服务等级

![](https://s2.loli.net/2022/07/04/tjK8XswLJYN6e74.jpg)

QoS值0：不管对方是否收到，最多只分发一次。

QoS值1：必须收到对方应答，否则将再次分发。

QoS值2：需要经过对方的两次确认。

### CONNECT – 连接服务端

* 固定报头：0x10  ?? （剩余长度值）

* 可变报头（十个字节，固定不变）

  CONNECT报文的可变报头按下列次序包含四个字段：协议名（Protocol Name），协议级别（Protocol Level），连接标志（Connect Flags）和保持连接（Keep Alive）。

  * 协议名：0x00 04 (0x)MQTT，协议名是表示协议名 MQTT 的UTF-8编码的字符串
  
  * 协议级别：0x04
  
  * 连接标志：服务端必须验证CONNECT控制报文的保留标志位（第0位）是否为0，如果不为0必须断开客户端连接。
  
    ![](https://s2.loli.net/2022/07/06/gije27StYpfrZyo.png)
  
  * 保持连接：保持连接（Keep Alive）是一个以秒为单位的时间间隔，表示为一个16位的字，它是指在客户端传输完成一个控制报文的时刻到发送下一个报文的时刻，两者之间允许空闲的最大时间间隔。
  
    ![](https://s2.loli.net/2022/07/06/BsaHXKY3IGmnhq9.png)

* 有效载荷

  ![](https://s2.loli.net/2022/07/06/4LJrjAfclTyaZpm.png)

  * 客户端标识符： *|securemode=3,signmethod=hmacsha1|                         *设备名称     注意替换
  * 用户名： *&#                     *设备名称 #ProductKey  注意替换  
  * 密码：用DeviceSecret做为秘钥对clientId*deviceName*productKey#进行hmacsha1加密后的结果      *设备名称 #ProductKey  注意替换

### CONNACK – 确认连接请求

* 固定报头：0x20 02

* 可变报头：

  ![](https://s2.loli.net/2022/07/06/NfuboV5wgIrEqAj.png)

  * 连接确认标志：第1个字节是 连接确认标志，位7-1是保留位且必须设置为0；第0 (SP)位 是当前会话（Session Present）标志。

  * 连接返回码：

    ![](https://s2.loli.net/2022/07/06/O64oT5jim2lrERP.png)

### DISCONNECT –断开连接

* 固定报头：0xE0 00

### PINGREQ – 心跳请求

* 固定报头：0xC0 00

### PINGRESP – 心跳响应

* 固定报头：0xD0 00

### SUBSCRIBE - 订阅主题

* 固定报头：0x82  ?? （剩余长度值）

* 可变报头：报文标识符（两个字节），确定哪条报文。

* 有效载荷

  ![](https://s2.loli.net/2022/07/06/arB4ch7QCENAUzp.png)

  主题过滤器（物模型通信Topic）如：/sys/hgerCvFPASO/${deviceName}/thing/service/property/set

  **可以发送多对主题过滤器 和 QoS等级字段组合。**

### SUBACK – 订阅确认

* 固定报头：0x90  03 （剩余长度值）

* 可变报头

  ![](https://s2.loli.net/2022/07/06/kS9EGRLKJmAjz7P.png)

* 有效载荷

  允许的返回码值：
  0x00 - 最大QoS 0
  0x01 - 成功 – 最大QoS 1
  0x02 - 成功 – 最大 QoS 2
  0x80 - Failure 失败
  
### UNSUBSCRIBE –取消订阅

* 固定报头：0xA2  ?? （剩余长度值）

* 可变报头：报文标识符（两个字节），确定哪条报文。

* 有效载荷

  ![](https://s2.loli.net/2022/07/06/blohOnv63Z2eVWf.png)

  **可以发送多对主题过滤器 ，同时取消订阅。**

### UNSUBACK – 取消订阅确认

* 固定报头：0xB0  02 （剩余长度值）

* 可变报头

  ![](https://s2.loli.net/2022/07/06/yHmSTzX3swG2jUq.png)

* 有效载荷：没有有效载荷。

### PUBLISH – 发布消息

* 固定报头：0x3?   ?? （剩余长度值）

  ![](https://s2.loli.net/2022/07/06/f5MHKziDNUbF3YW.png)

  * 重发标志：第1个字节，第3位。为0表示第一次发送，为1表示不是第一次。
  * 服务质量等级：第1个字节，第2-1位。
  * 保留标志：第1个字节，第0位。置1则该条报文会发送给后续订阅的设备。

* 可变报头（可变报头按顺序包含主题名和报文标识符）

  * 主题名：和主题过滤器一个概念。

  * 报文标识符：只有当QoS等级是1或2时，报文标识符（Packet Identifier）字段才能出现在PUBLISH报文中。

* 有效载荷：有效载荷包含将被发布的应用消息。数据的内容和格式是应用特定的。有效载荷的长度这样计算：用固定报头中的剩余长度字段的值减去可变报头的长度。包含零长度有效载荷的PUBLISH报文是合法的。


### PUBACK –发布确认

* 固定报头：0x40  02 （剩余长度值）

* 可变报头

  ![](https://s2.loli.net/2022/07/07/ZoRkBFUVaeuA6Db.png)