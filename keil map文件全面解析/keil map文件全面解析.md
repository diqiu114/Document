# Keil系列教程12_map文件全面解析



 发表于 2018-12-07 | 分类于 [Keil系列教程 ](https://www.strongerhuang.com/categories/Keil系列教程/)| *阅读次数:* 3051
- [Keil系列教程12\_map文件全面解析](#keil系列教程12_map文件全面解析)
- [一、写在前面](#一写在前面)
- [二、关于Keil中的map文件](#二关于keil中的map文件)
  - [2.1 如何打开map文件](#21-如何打开map文件)
  - [2.2 map文件输出内容配置](#22-map文件输出内容配置)
  - [2.3 map文件内容分类](#23-map文件内容分类)
- [三、Section Cross References](#三section-cross-references)
- [四、Removing Unused input sections from the image](#四removing-unused-input-sections-from-the-image)
- [五、Image Symbol Table](#五image-symbol-table)
  - [5.1 Symbol分为两大类](#51-symbol分为两大类)
  - [5.2 Symbol内容要点](#52-symbol内容要点)
- [六、Memory Map of the image](#六memory-map-of-the-image)
  - [6.1 主要介绍](#61-主要介绍)
  - [6.2 内容要点](#62-内容要点)
- [七、Image component sizes](#七image-component-sizes)
- [八、说明](#八说明)
- [九、最后](#九最后)

# 一、写在前面

**什么是map文件？**

简单的说：map文件是通过编译器编译之后，集程序、数据及IO空间的一种映射文件。

很多技术牛逼的工程师在遇到**内存越界，或溢出的情况**，首先想到的就是分析map文件。通过map文件可以知道函数大小，入口地址等一些重要信息。

我们在Keil中最常见的就是在编译之后，编译窗口会显示类似如下一段关于程序和数据大小的信息：
**Program Size: Code=1112 RO-data=320 RW-data=0 ZI-data=1632**

这一段提示信息其实是汇总了程序和数据的信息，这些信息其实是单个模块汇总而成，在map文件里有详细列表。



# 二、关于Keil中的map文件

## 2.1 如何打开map文件

最直接，也是最简单的办法：**双击工程目标，出现map文件**（注意，双击的是工程目标，不要双击工程，或文件组）。如下图：


[![img](keil%20map%E6%96%87%E4%BB%B6%E5%85%A8%E9%9D%A2%E8%A7%A3%E6%9E%90.assets/1.%E5%8F%8C%E5%87%BB%E5%B7%A5%E7%A8%8B%E7%9B%AE%E6%A0%87%EF%BC%8C%E5%87%BA%E7%8E%B0map%E6%96%87%E4%BB%B6.gif)](https://github.com/strongerHuang/strongerHuang.github.io/blob/master/Keil/Keil系列教程12_map文件全面解析/1.双击工程目标，出现map文件.gif?raw=true)

当然，可以找到map文件位置（Listings文件夹），用编辑器工具打开。

## 2.2 map文件输出内容配置

在之前文章[《Keil系列教程05_工程目标选项配置（一）》](https://www.strongerhuang.com/Keil/Keil系列教程05_工程目标选项配置（一）.html)中的Listing 列表章节有提到map文件。

map文件输出信息的配置位于：Project -> Options for Target -> Listing，如下图：


[![img](keil%20map%E6%96%87%E4%BB%B6%E5%85%A8%E9%9D%A2%E8%A7%A3%E6%9E%90.assets/2.map%E6%96%87%E4%BB%B6%E8%BE%93%E5%87%BA%E9%85%8D%E7%BD%AE.png)](https://github.com/strongerHuang/strongerHuang.github.io/blob/master/Keil/Keil系列教程12_map文件全面解析/2.map文件输出配置.png?raw=true)

主要包含配置：
**Memory Map：**内存映射
**Callgraph：**图像映射

**Symbols：**符号
**Cross Reference：**交叉引用

**Size Info：**大小信息
**Totals Info：**统计信息
**Unused Section Info：**未调用模块信息
**Veneers Info：**装饰信息

我们可根据自己情况，想要输出什么信息，勾选对应信息即可。

**提示：**
A.默认情况，输出所有信息；
B.这些配置是一个组合关系；

## 2.3 map文件内容分类

从上面输出配置可以看得出来map文件大概包含了哪些信息。map文件将其分为如下五大类：
**1.Section Cross References：**模块、段(入口)交叉引用

**2.Removing Unused input sections from the image：**移除未调用模块

**3.Image Symbol Table：**映射符号表

**4.Memory Map of the image：**内存（映射）分布

**5.Image component sizes：**存储组成大小

下面章节针对Keil DMK-ARM、 ARM Compiler 5组件生成的map文件五大类内容展开详细讲述。

# 三、Section Cross References

**Section Cross References：**模块、段(入口)交叉引用
配置中需勾选：**Cross Reference**


[![img](keil%20map%E6%96%87%E4%BB%B6%E5%85%A8%E9%9D%A2%E8%A7%A3%E6%9E%90.assets/3.Section%20Cross%20References.png)](https://github.com/strongerHuang/strongerHuang.github.io/blob/master/Keil/Keil系列教程12_map文件全面解析/3.Section Cross References.png?raw=true)

**Section Cross References：**模块、段(入口)交叉引用，指的是各个源文件生成的模块、段（定义的入口）之间相互引用的关系。

比如：
**main.o(i.System_Initializes) refers to bsp.o(i.BSP_Initializes) for BSP_Initializes**

意思是：
main模块(main.o)中的System_Initializes函数(i.System_Initializes)，引用（或者说调用）了bsp模块(bsp.o)中的BSP_Initializes函数。

**提示：**
**A.**main.o是main.c源文件生成的目标文件模块；
**B.**I.System_Initializes是System_Initializes函数的入口。

# 四、Removing Unused input sections from the image

**Removing Unused input sections from the image：**移除未调用模块
配置中需勾选：**Unuaed Sections Info**


[![img](keil%20map%E6%96%87%E4%BB%B6%E5%85%A8%E9%9D%A2%E8%A7%A3%E6%9E%90.assets/4.Removing%20Unused%20input%20sections%20from%20the%20image.png)](https://github.com/strongerHuang/strongerHuang.github.io/blob/master/Keil/Keil系列教程12_map文件全面解析/4.Removing Unused input sections from the image.png?raw=true)

这一类很好理解，就是我们代码中，没有被调用的模块（或者说函数）会在map文件中生成一个列表。

比如：
**Removing stm32f10x_gpio.o(i.GPIO_AFIODeInit), (20 bytes).**

意思是：
stm32f10x_gpio.c文件中GPIO_AFIODeInit模块（函数）未被调用，其代码大小20字节。

最后还有一个统计信息：
**52 unused section(s) (total 2356 bytes) removed from the image.**

1.总共有52段没有被调用；
2.没有被调用的大小为2356 字节；

# 五、Image Symbol Table

**Image Symbol Table：**映射符号表
配置中需勾选：**Symbols**


[![img](keil%20map%E6%96%87%E4%BB%B6%E5%85%A8%E9%9D%A2%E8%A7%A3%E6%9E%90.assets/5.Image%20Symbol%20Table.png)](https://github.com/strongerHuang/strongerHuang.github.io/blob/master/Keil/Keil系列教程12_map文件全面解析/5.Image Symbol Table.png?raw=true)

Image Symbol Table：映射符号表，也就是各个段所存储对应地址的表（图片删除了中间部分内容）。

## 5.1 Symbol分为两大类

**1.Local Symbols：**局部
**2.Global Symbols：**全局

## 5.2 Symbol内容要点

**1.Symbol Name：符号名称**
名称命名及分类请看最后给出的官方参考文档。

**2.Value：存储对应的地址**
大家会发现有0x0800xxxx、0x2000xxxx这样的地址。
0x0800xxxx指存储在FLASH里面的代码、变量等。
0x2000xxxx指存储在内存RAM中的变量Data等。

**3.Ov Type：符号对应的类型**
符号类型大概有几种：Number、Section、Thumb Code、Data等；
细心的朋友会发现：全局、静态变量等位于0x2000xxxx的内存RAM中。

**4.Size：存储大小**
这个容易理解，就是当前行Symbol占用大小。

**5.Object(Section)：段目标**
这里一般指所在模块（源文件）。

# 六、Memory Map of the image

**Memory Map of the image：**内存（映射）分布
配置中需勾选：**Memory Map**


[![img](keil%20map%E6%96%87%E4%BB%B6%E5%85%A8%E9%9D%A2%E8%A7%A3%E6%9E%90.assets/6.Memory%20Map%20of%20the%20image.png)](https://github.com/strongerHuang/strongerHuang.github.io/blob/master/Keil/Keil系列教程12_map文件全面解析/6.Memory Map of the image.png?raw=true)

**Memory Map of the image：**内存（映射）分布，内容相对较多，比较重要的一项。

## 6.1 主要介绍

**Image Entry point : 0x08000131：**指映射入口地址。

**Load Region LR_IROM1 (Base: 0x08000000, Size: 0x00000598, Max: 0x00080000, ABSOLUTE):**
指加载区域位于LR_IROM1开始地址0x08000000，大小有0x00000598，这块区域最大为0x00080000.

**执行区域：**
A.Execution Region ER_IROM1
B.Execution Region RW_IRAM1
这个区域，其实就是对应我们目标配置中的区域，如下如：


[![img](keil%20map%E6%96%87%E4%BB%B6%E5%85%A8%E9%9D%A2%E8%A7%A3%E6%9E%90.assets/7.%E6%89%A7%E8%A1%8C%E5%8C%BA%E5%9F%9F.png)](https://github.com/strongerHuang/strongerHuang.github.io/blob/master/Keil/Keil系列教程12_map文件全面解析/7.执行区域.png?raw=true)

## 6.2 内容要点

**1.Base Addr：存储地址**
0x0800xxxxFLASH地址和0x2000xxxx内存RAM地址。

**2.Size：存储大小**

**3.Type：类型**
Data：数据类型
Code：代码类型
Zero：未初始化变量类型
PAD：这个类型在map文件中放在这个位置，其实它不能算这里的类型。要翻译的话，只能说的“补充类型”。

ARM处理器是32位的，如果定义一个8位或者16位变量就会剩余一部分，这里就是指的“补充”的那部分，会发现后面的其他几个选项都没有对应的值。

**4.Attr：属性**
RO：存储与ROM中的段
RW：存储与RAM中的段

**5.Section Name：段名**
这里也可以说为入口分类名，与第一章节“Section Cross References”指的模块、段一样。
大概包含：RESET、.ARM、 .text、 i、 .data、 .bss、 HEAP、 STACK等。

**6.Object：目标**

# 七、Image component sizes

**Image component sizes：**存储组成大小
配置中需勾选：**Size Info**

[![img](keil%20map%E6%96%87%E4%BB%B6%E5%85%A8%E9%9D%A2%E8%A7%A3%E6%9E%90.assets/8.Image%20component%20sizes.png)](https://github.com/strongerHuang/strongerHuang.github.io/blob/master/Keil/Keil系列教程12_map文件全面解析/8.Image component sizes.png?raw=true)

**Image component sizes：**存储组成大小，其实主要就是对模块进行汇总存储大小信息。

这一章节内容相信大家都能理解，我们编译工程后，在编译窗口一般会看到类似如下一段信息：
**Program Size: Code=1112 RO-data=320 RW-data=0 ZI-data=1632**

**Code：**指代码的大小；
**Ro-data：**指除了内联数据(inline data)之外的常量数据；
**RW-data：**指可读写（RW）、已初始化的变量数据；
**ZI-data：**指未初始化（ZI）的变量数据；

**提醒：**
**A.Code、Ro-data：**位于FLASH中；
**B.RW-data、ZI-data：**位于RAM中；
**C.RW-data**已初始化的数据会存储在Flash中，上电会从FLASH搬移至RAM。

关系如下：
RO Size = Code + RO Data
RW Size = RW Data + ZI Data
ROM Size = Code + RO Data + RW Data

可以参看我公众号之前分享的一篇文章[《Keil编译大小及存储说明》](https://mp.weixin.qq.com/s/Ey9ZqHt05sXdwescFl2vZA)

上面map信息是比较全面的汇总，如果不想看那些模块的详细，只看汇总统计的信息，可以在配置中只勾选“Totals Info”，对比信息：


[![img](keil%20map%E6%96%87%E4%BB%B6%E5%85%A8%E9%9D%A2%E8%A7%A3%E6%9E%90.assets/9.%E5%8F%AA%E7%9C%8B%E7%BB%9F%E8%AE%A1%E4%BF%A1%E6%81%AF.png)](https://github.com/strongerHuang/strongerHuang.github.io/blob/master/Keil/Keil系列教程12_map文件全面解析/9.只看统计信息.png?raw=true)

**最后提示：**
本文内容请参看Keil自带的一篇PDF文章**《ARM® Compiler v5.06 for µVision® armlink User Guide》。**

# 八、说明

1.该文档仅供个人学习使用，版权所有，禁止商用。

2.本文由我一个人编辑并整理，难免存在一些错误。

3.为了方便大家平时公交、地铁、外出办事也能用手机随时随地查看该教程，该教程同步更新于微信公众号【EmbeddedDevelop】，关注微信公众号回复【Keil系列教程】即可查看全系列教程。

# 九、最后

我的博客：[http://www.strongerhuang.com](http://www.strongerhuang.com/)
我的GitHub：https://github.com/EmbeddedDevelop

　　我的微信公众号（ID：strongerHuang）还在分享STM8、STM32、Keil、IAR、FreeRTOS、UCOS、RT-Thread、CANOpen、Modbus…等更多精彩内容，如果想查看更多内容，可以关注我的微信公众号。

[![strongerHuang wechat](keil%20map%E6%96%87%E4%BB%B6%E5%85%A8%E9%9D%A2%E8%A7%A3%E6%9E%90.assets/strongerHuang.jpg)](https://www.strongerhuang.com/uploads/strongerHuang.jpg)

关注微信公众号，查看更多精彩内容!

本文对你有帮助吗？

打赏

[# Keil](https://www.strongerhuang.com/tags/Keil/) [# MDK-ARM](https://www.strongerhuang.com/tags/MDK-ARM/) [# µVision](https://www.strongerhuang.com/tags/µVision/)