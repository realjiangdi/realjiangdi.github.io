---
layout: post
title: 基于STM32G030F6 HAL库开发
tags: 单片机

---

STM32G030F6 单片机开发记录。

<!--more-->

| 日期     | 修订版本 | 修订内容                                                                       | 修订人 |
| :------- | :------- | :----------------------------------------------------------------------------- | :----- |
| 20200706 | V1.0.0   | 文章框架搭建；<br>基于HAL库开发环境搭建；<br>创建Demo工程模板；                | 蒋迪   |
| 20200806 | V1.1.0   | 基于Hal库GPIO的使用；<br> 基于Hal库GPIO 中断的使用; <br>基于Hal库的ADC的使用； | 蒋迪   |

# 环境搭建

## 1. keil5 安装

可自行百度

参考：

* [MDK5(keil5)环境安装及破解](https://blog.csdn.net/weixin_42108484/article/details/80475394)

## 2. Stm32CubeMX安装

可自行百度，相关IO操作函数可以参考`GPIO`章节

参考：

* [STM32CubeMX_环境搭建](https://blog.csdn.net/weifengdq/article/details/102902936)
* [STM32CubeMX系列教程](https://zhuanlan.zhihu.com/STM32CubeMX)

## 3. 创建工程模板Demo

可自行百度

参考：

* [STM32CubeMX系列教程](https://zhuanlan.zhihu.com/STM32CubeMX)

## 4. 点亮LED灯

可自行百度

参考：

# HAL库开发实例

## GPIO

通过STM32CubeMX 配置好相应的GPIO的模式，完成相应初始化的配置后。业务代码可以通过一些HAL库提供的 `IO Function` 来操纵GPIO。 以下是一些常用的API函数。

``` c
/**

  + @brief  Read the specified input port pin.
  + @param  GPIOx where x can be (A..F) to select the GPIO peripheral for STM32G0xx family
  + @param  GPIO_Pin specifies the port bit to read.
  +         This parameter can be any combination of GPIO_Pin_x where x can be (0..15).
  + @retval The input port pin value.

  */
GPIO_PinState HAL_GPIO_ReadPin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin)；

/**

  + @brief  Set or clear the selected data port bit.

  *

  + @note   This function uses GPIOx_BSRR and GPIOx_BRR registers to allow atomic read/modify
  +         accesses. In this way, there is no risk of an IRQ occurring between
  +         the read and the modify access.

  *

  + @param  GPIOx where x can be (A..F) to select the GPIO peripheral for STM32G0xx family
  + @param  GPIO_Pin specifies the port bit to be written.
  +         This parameter can be any combination of GPIO_Pin_x where x can be (0..15).
  + @param  PinState specifies the value to be written to the selected bit.
  +         This parameter can be one of the GPIO_PinState enum values:
  +            @arg GPIO_PIN_RESET: to clear the port pin
  +            @arg GPIO_PIN_SET: to set the port pin
  + @retval None

  */
void HAL_GPIO_WritePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin, GPIO_PinState PinState);

/**

  + @brief  Toggle the specified GPIO pin.
  + @param  GPIOx where x can be (A..F) to select the GPIO peripheral for STM32G0xx family
  + @param  GPIO_Pin specifies the pin to be toggled.
  +         This parameter can be any combination of GPIO_Pin_x where x can be (0..15).
  + @retval None

  */
void HAL_GPIO_TogglePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin);

```

## 中断
在STM32CubeMX中，将GPIO的某个引脚配置为外部中断源，并设置电气模式（上拉/下拉），设置触发边沿（下降沿/上升沿）。然后在NVIC中使能外部中断，并分配优先级。然后在中断处理函数`HAL_GPIO_EXTI_Falling_Callback`中撰写相对的用户代码。

## USART  
在STM32CubeMx中，找到相对应的外设模块（Connectivity）里配置串口模式为（Asynchronous）。

USART 相应的处理函数
```c
/**
  * @brief Send an amount of data in blocking mode.
  * @note   When UART parity is not enabled (PCE = 0), and Word Length is configured to 9 bits (M1-M0 = 01),
  *         the sent data is handled as a set of u16. In this case, Size must indicate the number
  *         of u16 provided through pData.
  * @note When FIFO mode is enabled, writing a data in the TDR register adds one
  *       data to the TXFIFO. Write operations to the TDR register are performed
  *       when TXFNF flag is set. From hardware perspective, TXFNF flag and
  *       TXE are mapped on the same bit-field.
  * @note   When UART parity is not enabled (PCE = 0), and Word Length is configured to 9 bits (M1-M0 = 01),
  *         address of user data buffer containing data to be sent, should be aligned on a half word frontier (16 bits)
  *         (as sent data will be handled using u16 pointer cast). Depending on compilation chain,
  *         use of specific alignment compilation directives or pragmas might be required to ensure proper alignment for pData.
  * @param huart   UART handle.
  * @param pData   Pointer to data buffer (u8 or u16 data elements).
  * @param Size    Amount of data elements (u8 or u16) to be sent.
  * @param Timeout Timeout duration.
  * @retval HAL status
  */
HAL_StatusTypeDef HAL_UART_Transmit(UART_HandleTypeDef *huart, uint8_t *pData, uint16_t Size, uint32_t Timeout);
uint8_t TxData[10]= "01234abcde";
HAL_UART_Transmit(&huart2,TxData,sizeof(TxData),0xffff);

```

### 重映射printf
```c
#include "stdio.h"
#include "main.h"

#ifdef __GNUC_
#define PUTCHAR_PROTOTYPE int __io_putchar(int ch)
#else
#define PUTCHAR_PROTOTYPE int fputc(int ch, FILE *f)
#endif

//重映射printf的功能
PUTCHAR_PROTOTYPE
{
  HAL_UART_Transmit(&huart1, (uint8_t *)&ch, 1, 0xFFFF);
 
  return ch;
}
```
## ADC 
### ADC 查询模式
1. 启动ADC
2. 等待EOC标志位、
3. 读取寄存器数据

HAL库ADC查询模式
```c
HAL_ADC_Start(&hadc1);
if(HAL_OK == HAL_ADC_PollForConversion(&hadc1, 0xff)){
	adcValue = HAL_ADC_GetValue(&hadc1);
	printf("ADC Value is %d \r" , adcValue);
}
```
### ADC 中断模式
待更新

### ADC DMA模式
待更新

## SPI 

## I2C 

# 参考资料

* [野火《STM32 HAL库开发实战指南》系列](https://pan.baidu.com/s/1tw68lIFJHXGqnbaf3oxXXQ?errno=0&errmsg=Auth%20Login%20Sucess&&bduss=&ssnerror=0&traceid=#list/path=%2F&parentPath=%2Fsharelink1463230356-229805398784803)
* [【STM32】HAL库 STM32CubeMX系列学习教程](https://blog.csdn.net/as480133937/article/details/99935090)
* [STM32教程](https://www.xmf393.com/category/jiaoxue/stm32/)
* [HAL库教程4：外部中断](https://blog.csdn.net/geek_monkey/article/details/89164659)
* [HAL驱动库学习-ADC](https://www.cnblogs.com/cat-li/p/4982510.html)
* [STM32使用HAL库实现ADC单通道转换](https://www.cnblogs.com/xingboy/p/10018749.html)
* [【STM32】HAL库 STM32CubeMX教程九---ADC](https://blog.csdn.net/as480133937/article/details/99627062)
* [HAL库 STM32CubeMX教程四---UART串口通信详解](https://blog.csdn.net/as480133937/article/details/99073783)
* [STM32CubeMX系列教程5:串行通信(USART)](https://www.waveshare.net/study/article-644-1.html)
