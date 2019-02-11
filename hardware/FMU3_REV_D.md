# FMU3_REV_D

[TOC]

## FMU_FMU3_REV_C
主要是对主芯片STM32F4引脚的定义，晶振电路的设计和供电电路的选择

**问题：FM25V10芯片不知道干什么，推测是稳压电路**

## SD_USB_FMU3_REV_C
SD卡和USB接口的电路

1.USB接口
引出了两个数据网络，用到了芯片**NUF2042XV6**，可能一个是高速一个是低速。

2.SD卡
常规SD卡插口，上拉电阻

3.其他
有电流和电压传感器接口，电流使用ADC2，电压使用ADC1。
还有压力传感器接口
还有一个电机或者舵机的传感器，接口是**VDD_SERVO_SENS**，目前作用未知

##SERIAL_FMU3_REV_C
CAN1,CAN2,I2C1,I2C2,USART2,USART3,USART4,USART7,USART8接口

##SENSORS_FMU3_REV_C
四个传感器电路，分别为：
1.MPU6000
2.H11905CT-ND
3.342-1092-1-ND
4.MS5611-01BA
**后面几个传感器未知，待查**

## FMU_POWER_FMU3_REV_C
这里是电源电路，还有芯片的供电滤波电路和重启电路

## LEDs_FMU3_REV_C
led灯电路
环绕一周LED还有电源指示灯

## IO_FMU3_REV_C
这里是STM32F1的最小系统板，包括电源、晶振和重启电路。

## PWM_PPM_FMU3_REV_C
这里是PWM波生成的电路，**但是具体实现过程还没弄懂。**
其中FMU是F4的IO口，IO是F1的IO口。
有一个芯片不清楚，**568-8979-1-ND**，猜测是PWM控制芯片

## 80PIN_FMU3_REV_C
这是有一个80pin的芯片**CON-DF17-80F**，具体作用未知。

## IO_PWR_FMU3_REV
F1板供电，并且会在输入电压过高的时候截止。**具体原理未知**

##总结
整个电路板比较简单清晰，我们需要搞清楚电源的种类和地的种类，所有传感器都较为简单。















