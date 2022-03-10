---
title: Linux和Windows安装tensorflow-cpu 2.x
tags: [tensorflow]
category: [深度学习,tensorflow]
top: 3
index_img: /img/tensorflow-cpu.jpg
---
# Linux和Windows安装tensorflow-cpu 2.x

网络上有许多安装tensorflow的教程，我写这篇博客主要是想记录下自己安装tensorflow过程中踩的坑，现在回过头看tensorflow的安装其实无比简单。在这里将我的经验写下来，供大家借鉴参考，linux和Windows同样适用。
<!-- more -->
在初次安装tensorflow时，网络上的许多教程都讲可以通过anaconda进行安装。按照教程的步骤，首先是使用conda创建一个新的虚拟环境，对于刚刚接触的我来说并不明白虚拟环境是什么意思，所以就导致了我即使是成功安装了tensorflow而不自知。

下面我将自己的安装步骤详细记录下来，并对一些问题进行解释。

## 一、anaconda的安装

不同的项目用到的环境可能不一样，但有些环境在同一台计算机里面可能会导致冲突，这就是为什么有时候会产生一些莫名其妙错误的原因，所以我们希望将这些环境隔离开来。anaconda就是这样一个很好的工具。

![](https://s2.loli.net/2022/02/12/x3r4PSn582wmUoc.png)

[Anaconda | Individual Edition](https://www.anaconda.com/products/individual)

从这个网站上下载，根据自己的操作系统选择相应的版本，可以直接使用最新版本。

Windows是寻常的.exe文件，按照提示进行安装；Linux是.sh文件，在shell终端使用sh 文件名.sh安装。Linux操作系统下需要注意更改安装路径，建议放在用户目录下，这样可以避免后续安装过程中始终被要求获得root权限。具体安装步骤这里不详加叙述，安装成功并添加好环境变量后，可以使用conda -V命令验证。

## 二、虚拟环境的创建

顺利安装anaconda后，在命令行终端输入

```shell
conda create -n tensorflow python=3.9
```

创建一个虚拟环境，python=3.9是指定在虚拟环境安装python3.9

```shell
conda info -e
```

查看已安装的环境。

![](https://s2.loli.net/2022/02/12/FJQjiU9l3hZDdMb.png)

```shell
activate tensorflow
```

切换虚拟环境（deactivate退出当前环境）。

![](https://s2.loli.net/2022/02/12/3GpnKfa29PFiv14.png)

后续在“tensorflow”这个虚拟环境中安装的任何库，均仅在此虚拟环境中生效。

## 三、tensorflow安装

tensorflow分为cpu和gpu两个版本。gpu需要计算机有英伟达的显卡，我将在下面一篇博客中介绍如何安装和配置环境。

[在公共服务器上使用非root用户离线安装tensorflow-gpu 2.x - 舍功利之心，注一腔热情。 (2022xi.github.io)](https://2022xi.github.io/2022/02/12/在公共Linux服务器上使用非root用户离线安装tensorflow-gpu/)

进入虚拟环境后，在命令行终端输入

```shell
conda install tensorflow
```

等待自动安装一些依赖。

安装完成后，我们来验证一下。

```shell
python
```

进入python环境

```python
import tensorflow as tf
a = tf.constant(1)
print(a)
```

成功安装输出结果

![](https://s2.loli.net/2022/02/12/tcLQ4g2RzJIkMSv.png)

不想让中间警告信息显示时，可以通过添加如下两行代码，设置TensorFlow日志输出级别。

```python
impot os
os.environ["TF_CPP_MIN_LOG_LEVEL"] = "3"
```

到这里tensorflow的安装就完成了，需要注意在使用时激活虚拟环境。

