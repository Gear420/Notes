# 微机原理及接口技术notes
## 1.1基本概念

``选择合适的微处理器，增加外围芯片，构成应用系统。``

8088/8086 CPU

* 微处理器：不等于cpu
* 微型计算机：集成到芯片上—> 单片机
* 微型计算机系统：➕软件

主要是接口技术

* 北桥：高速处理通道
* 南桥：慢速设备

小端存储 
## 2.1 8086/8088 CPU

` CPU `->`系统总线` <- `接口电路` + `寄存器`
`-------------汇编程序------------`

>1. 了解功能
>2. 引脚特性 
>3. 结构部分
>4. 电路设计
>5. 编程开发

1.1. 引脚功能
1.2. 信号流向(单向or双向)
1.3. 有效电平
1.4. 三态能力（低电平，高电平，高阻三态）

### 8086引脚功能：
最大最小模式 -> 特殊功能引脚（两种功能 低电平 高电平）
地址与数据线 16位
高字节的数据选择 （BHE A0选择读写数据）
中断功能

最大模式与最小模式接口设计
准备信号与测试信号 用于协调速度





					

