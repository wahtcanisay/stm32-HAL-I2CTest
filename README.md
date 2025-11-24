# stm32-HAL-I2CTest
使用stm32单片机I2C总线控制OLED显示屏，并通过板载LED亮灭判断显示屏状态  
## 硬件部分
使用四针脚OLED屏幕，VCC接电源，GND接地，SCL代表I2C接口的串行时钟线，SDA代表I2C串行数据线。  
![电路接线图](https://github.com/wahtcanisay/stm32-HAL-I2CTest/blob/main/image.png)  
***实际上常用型号的OLED屏幕内置电阻，该处上拉电阻可以不连***  
OLED屏幕地址：0x78  
## 软件部分

## cubemx部分
首先再`SYS`中`Debug`选择`Serial Wire`  
将PC13设置为输出，参数选择输出高电平，开漏模式，低速  
在`Connectivity`选择`I2C1`,`Mode`选择`I2C`  
`Master Features`参数(作为主机参数)选择快速模式`Fast Mode`，时钟速率选择`400000`,占空比选择2:1  

## keil5部分

OLED点亮的驱动代码：  
```c
uint8_t commands[] = {
    0x00,       //命令流
    0x8d, 0x14, //使能电荷泵
    0xaf,       //打开屏幕开关
    0xa5        //屏幕全亮
}
```

新接口：  
```c
HAL_StatusTypeDef HAL_I2C_Master_Transmit(I2C_HandleTypeDef *hi2c,
                                          uint16_t DevAddress,
                                          uint8_t *pData,
                                          uint16_t Size,
                                          uint32_t Timeout)
```
作用：向从机写入数据  
参数：  
*hi2c-i2c的句柄指针  
DevAddress-从机地址  
*pData-发送数据地址  
Size-发送数据数量  
Timeout-超时时间ms  
返回：数据发送结果   

```c
HAL_StatusTypeDef HAL_I2C_Master_Receive(I2C_HandleTypeDef *hi2c,
                                         uint16_t DevAddress,
                                         uint8_t *pData,
                                         uint16_t Size,
                                         uint32_t Timeout)
```
作用：主机向从机读数据  
参数：  
*hi2c-i2c的句柄指针  
DevAddress-从机地址  
*pData-接收数据地址  
Size-接收数据数量  
Timeout-超时时间ms  
返回：数据发送结果   

整体流程：  
```c
uint8_t commands[] = {0x00, 0x8d, 0x14, 0xaf, 0xa5}

HAL_I2C_Master_Transmit(&hi2c1, 0x78, commands, sizeof(commands)/sizeof(commands[0]), HAL_MAX_DELAY);

/*OLED屏幕被点亮之后会一直发送一个字节，OLED被成功点亮第六位为0，否则第六位为1*/
uint8_t dataRcvd;

HAL_I2C_Master_Receive(&hi2c1, 0x78, &dataRcvd, 1, HAL_MAX_DELAY);

if((dataRcvd & (0x01<<6)) == 0){   // 0x01<<6相当于将最后一位的1左移6位，变为0x40
                                   //然后再与dataRcvd进行位与运算，第六位若为0则输出0x00,第六位为1则输出0x10
    //第六位为0，点亮LED
    HAL_GPIO_WritePin(GPIOC, GPIO_PIN_13, GPIO_PIN_RESET);
}
else{
    //第六位为1，熄灭LED
    HAL_GPIO_WritePin(GPIOC, GPIO_PIN_13, GPIO_PIN_SET);
}
while(1){} //防止程序跑飞
```