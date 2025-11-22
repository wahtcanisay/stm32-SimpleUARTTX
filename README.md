# stm32-SimpleUART TX
基于stm32实现串口异步通信，单片机写入程序发送至电脑当中
## 具体实现

### cubemx部分
首先设置调试接口，选中`SYS`, `Debug`选中`Serial Wire`。
然后在`Connectivity`中选中`USART1`，为串口选择工作模式`Mode`中选`Asynchronous`，即异步模式。
`PA9`作为数据发送引脚，`PA10`作为数据接收引脚。

接线图如下：
![接线图](https://github.com/wahtcanisay/stm32-SimpleUARTTX/blob/main/%E6%8E%A5%E7%BA%BF%E5%9B%BE.png)

两处TX与RX交叉连接
在`USART1`中设置参数：`115200`，`8 Bits`， `None`， `1`， `Receive and Transmit`
也就是最常用的8位无校验

在电脑中的串口调试中选用相同的串口设置

### Keil5实现
在`mian.c`中在43行会有`UART_HandleYtpeDef huart1;` 生成了串口1的句柄

**新接口：**
```c
HAl_statusTypeDef HAL_UART_Transmit(UART_HandleTypeDef *huart,
                                    uint8_t *pData,
                                    uint16_t Size,
                                    uint32_t Timeout)
```
作用：通过串口向外发送数据  
参数：      
  *huart 串口句柄指针  
  *pData 要发送的数据指针  
  Size 要发送的数据数量，以字节为单位  
  Timeout 超时时间，单位ms，填写HAL_MAX_Delay代表无限期等待下去直到发送完成为止  

返回：返回数据发送结果   
HAL_OK - 成功  
HAL_ERROR - 发送出错  
HAL_BUSY - 串口忙  
HAL_TIMEOUT - 发送超时  

**声明变量内容并发送** ：
```c
//声明变量
uint8_t byteNumber = 0x5a;
uint8_t byteArray = {1, 2, 3, 4, 5};
char ch = 'a';
char *str = "Hello World";
//发送数据0x5a
HAL_UART_Transmit(&huart1, &byteNumber, 1, HAL_MAX_Delay);
//发送字节数组
HAL_UART_Transmit(&huart1, &byteArray, 5, HAL_MAX_Delay);
//发送字母
HAL_UART_Transmit(&huart1, (uint8_t*)&ch, 1, HAL_MAX_Delay);
//发送字符串
HAL_UART_Transmit(&huart1, (uint8_t*)str, strlen(str), HAL_MAX_Delay);
```
**注意!** 
* 使用了`strlen()`则要在头文件中加入``#include<string.h>``,也要写在23行的BEGIN Includes和END注释之间

* 发送数据的指针必须为`uint8_t`类型，不符合想要发送要进行强制类型转换，比如本实验中将字符以及字符数组的`char*`进行转换


将代码写入main.c中UART初始化之后，while(1)循环之前的BEGIN和END注释之间

随后进行调试代码，点击魔术棒找到C/C++将 Level3 改为 Level 0，再进行编译代码  
选择调试按钮，直接运行到发送数据的位置(右键点击Run to cursor line)  
在串口调试中首先以HEX十六进制接收数据，在keil5中点击两次单步调试就能收到5A和01 02 03 04 05  
再选择ASC2格式以字符格式接收数据，再进行单步调试就能收到a 和 hello world  
