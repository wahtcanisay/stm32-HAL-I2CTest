# stm32-HAL-I2CTest
使用stm32单片机I2C总线控制OLED显示屏，并通过板载LED亮灭判断显示屏状态  
## 硬件部分
使用四针脚OLED屏幕，VCC接电源，GND接地，SCL代表I2C接口的串行时钟线，SDA代表I2C串行数据线。  

## 软件部分

## cubemx部分
首先再`SYS`中`Debug`选择`Serial Wire`  
将PC13设置为输出，参数选择输出高电平，开漏模式，低速  
在`Connectivity`选择`I2C1`,`Mode`选择`I2C`  
`Master Features`参数(作为主机参数)选择快速模式`Fast Mode`，时钟速率选择`400000`,占空比选择2:1  
