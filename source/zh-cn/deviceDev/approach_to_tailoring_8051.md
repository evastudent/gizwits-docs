title: 8051平台最小资源裁剪说明
---

# 前序

由于8051等型号MCU片上资源有限（以 **STC15F2K60S2** 为例，SRAM空间大小：2K = 2048字节），当云端定义的产品中数据点较多时会导致编译后的程序剩余内存过少，进而导致串口数据解析异常、程序卡死等情况，故 **STC15F2K60S2** 平台的**数据点长度建议*少于20个字节***，对应的RAM的使用率需在60%以内。

对于想进一步降低内存消耗的开发者，我们本文提供一种“资源裁剪最小集”的方式。

本文档主要说明如何在自动生成的代码基础上裁剪为最小资源集的代码工程，以便节省更多的资源用于应用层的开发或是移植到更低资源的平台（如 SRAM为1K的MCU）。

在阅读本文档前建议开发者已深入了解机智云MCU版的程序逻辑，请参考：[《GoKit-MCU程序详解》](http://docs.gizwits.com/zh-cn/deviceDev/Gokit3Voice/GoKit-MCU-explanation.html)一文。本文并不过多讲解MCU程序实现的细节。

下面是裁剪的相关步骤。



# 1 查看当前源码工程资源使用情况

首先定义相应的数据点：

![1](/assets/zh-cn/deviceDev/8051/1.png)接下来生成 **STC15F2K60S2** 平台的源码工程：

![2](/assets/zh-cn/deviceDev/8051/2.png)

数据点长度的方式：在开发者中心下载《xxx- 机智云独立MCU方案接入通信协议文档》 ,在“3.2 WiFi模组控制设备”中可以看到“attr_flags”+“attr_vals”之和即为数据点长度的总大小（字节）：

![20](/assets/zh-cn/deviceDev/8051/20.png)

> 可以看到占**2个字节**的数据点长度

编译后的工程 **SRAM** 占用的大小如下图所示：

![3](/assets/zh-cn/deviceDev/8051/3.png)

> 可以看到定义的数据点所对应的 **SRAM** 使用量为988字节（ **STC15F2K60S2** SRAM总量为2048字节）

如果我们需要进一步降低内存使用率，就要进行资源最小集的裁剪。



# 2 裁剪资源到最小集

## 2.1 裁剪透传处理代码

- 去除全局结构体中用于透传处理的结构体成员：

![4](/assets/zh-cn/deviceDev/8051/4.png)

- 屏蔽相应的处理代码：

![5](/assets/zh-cn/deviceDev/8051/5.png)

![6](/assets/zh-cn/deviceDev/8051/6.png)

`减少内存（26字节）：`

![7](/assets/zh-cn/deviceDev/8051/7.png)



## 2.2 裁剪”数据点事件处理“代码

- 去除全局结构体中用于事件处理的结构体成员：

![8](/assets/zh-cn/deviceDev/8051/8.png)

- 去除相应的处理逻辑：

> 提示：可以全局搜索关键字，如：issuedProcessEvent

![9](/assets/zh-cn/deviceDev/8051/9.png)

![10](/assets/zh-cn/deviceDev/8051/10.png)

- 修改P0转事件的处理函数：

![12](/assets/zh-cn/deviceDev/8051/12.png)

`减少内存（19字节）：`

![13](/assets/zh-cn/deviceDev/8051/13.png)



- 裁剪用户区全局P0变量（替换为"共用协议区全局P0变量"）：

![14](/assets/zh-cn/deviceDev/8051/14.png)

![15](/assets/zh-cn/deviceDev/8051/25.png)

![img](file:///D:/Gizwits_Work/%E6%96%87%E6%A1%A3%E7%9B%B8%E5%85%B3/8051%E5%B9%B3%E5%8F%B0%E6%9C%80%E5%B0%8F%E8%B5%84%E6%BA%90%E8%A3%81%E5%89%AA%E8%AF%B4%E6%98%8E/15.png?lastModify=1511148170?lastModify=1511148250)

![15](/assets/zh-cn/deviceDev/8051/24.png)

- 屏蔽控制型事件处理相关代码：

![17](/assets/zh-cn/deviceDev/8051/17.png)



`减少内存（2字节）：`

![18](/assets/zh-cn/deviceDev/8051/18.png)



- 在"P0转事件处理"函数中添加控制处理逻辑：

![19](/assets/zh-cn/deviceDev/8051/19.png)



## 2.3 裁剪日志打印功能

对于类似Arduino的mcu平台，使用日志打印时会默认将打印内容存入RAM中，故会暂用大量的内存。

> 注： **STC15F2K60S2** 平台并没有此问题，其打印内容会存储在 `code` 区，即FLASH中，不占用 `xdata` 区的内存空间。

可以通过屏蔽打印语句或修改重定向宏来关闭日志打印功能：

![23](/assets/zh-cn/deviceDev/8051/23.png)



# 3 总结

以上内容通过裁剪**透传处理**、**数据点生成时间处理**、**用户区全局变量**，来达到资源最小集的生成。

由上文中的裁剪例子我们看到，当只定义一个布尔型的数据点时，协议中的数据点长度如下所示：

![20](/assets/zh-cn/deviceDev/8051/20.png)

> 可以看到占**2个字节**的数据点长度

> 裁剪后的内存消耗从 `988字节` 降低到了 `941字节` ，共减少了 `47字节` 的内存使用量。



经测试，当定义20个字节的扩展型数据点时，协议中的数据点长度如下所示：

![21](/assets/zh-cn/deviceDev/8051/26.png)

> 可以看到占**21个字节**的数据点长度
>
> 我们使用**相同的裁剪方式**，裁剪后的内存消耗能从 `1332字节` 降低到了 `1248字节` ，共减少了 `84字节` 的内存使用量。



**结论：当定义的数据点越多时，用这种方式节省出的内存越可观。**