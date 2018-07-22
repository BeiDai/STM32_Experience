## STM32F407ZGT6 MCU配置

- 集成FPU和DSP指令、192KB SRAM、1024FLASH
- 12个16位定时器、2个32位定时器、2个DMA控制器（共16通道）
- 3个SPI、3个全双工I2S、3个IIC、6个串口、2个USB(支持HOST/SLAVE)
- 2个CAN、3个12位ADC、2个12位DAC、1个RTC（带日历功能）、1个SDIO接口
- 1个FSMC接口、1个10/100M以太网MAC控制器、1个摄像头、11个随机数生成器
- 12个通用IO口等，刷屏速度3300W像素/秒

## STM32F4xx中文参考手册

- 2、存储器和总线架构


- 3、嵌入式Flash接口
  - [FLASH](/Peripheral/flash.md)

- 4、CRC计算单元

- 5、电源控制器（PWR）
  - [BKP](/Peripheral/bkp.md)
  
- 6、复位和时钟控制（RCC）
  - [RCC](/Peripheral/rcc.md)

- 7、通用IO（GPIO）
  - [GPIO](/Peripheral/gpio.md)

- 8、系统配置控制器（SYSCFG）

- 9、DMA控制器（DMA）
  - [DMA](/Peripheral/dma.md)

- 10、中断和事件（NVIC&EXIT）
  - [NVIC](/Peripheral/nvic.md)
  - [EXTI](/Peripheral/exti.md)

- 11、模数装换器（ADC）
  - [ADC](/Peripheral/adc.md)

- 12、数模装换器（DAC）

- 13、数字摄像头（DCMI）

- 14、高级控制定时器（TIM1和TIM8）
  - [TIM1_8](/Peripheral/tim.md)

- 15、通用定时器（TIM2和TIM5）
  - [TIM2_5](/Peripheral/tim.md)

- 16、通用定时器（TIM9和TIM14）
  - [TIM9_19](/Peripheral/tim.md)

- 17、基本定时器（TIM6和TIM7）
  - [TIM6_7](/Peripheral/tim.md)

- 18、独立看门狗（IWDG）
  - [IWDG](/Peripheral/iwdg.md)

- 18、窗口看门狗（WWDG）
  - [WWDG](/Peripheral/wwdg.md)

- 20、加密处理器（CRYP）

- 21、随机数发生器（RNG）

- 22、散列处理器（HASH）

- 23、实时时钟（RTC）
  - [RTC](/Peripheral/rtc.md)

- 24、控制器区域网络（bxCAN）
  - [CAN](/Peripheral/can.md)

- 25、内部集成电路（I2C）接口
  - [I2C](/Peripheral/i2c.md)

- 26、通用同步异步收发器（USART）
  - [USART](/Peripheral/usart.md)

- 27、串行外设接口（SPI）
  - [SPI](/Peripheral/spi.md)

- 28、安全数字输入输出接口（SDIO）

- 29、以太网（ETH）：通过DMA控制器进行介质访问控制（MAC）

- 30、全速 USB on-the-go (OTG_FS)

- 31、高速 USB on-the-go (OTG_HS)

- 32、灵活的静态存储控制器（FSMC）
  - [FSMC](/Peripheral/fsmc.md)

- 33、调试支持（DBG）

- 34、设备电子签名

## STM32使用笔记
### 系统时钟使用 (RCC)

![SystemClock](/Pictures/CLOCK.JPG)

### 中断系统 (EXTI)

![EXTI](/Pictures/EXTI.JPG)

### 串口通讯 (USART)

![USART](/Pictures/USART.JPG)

### (PWM) 输出

![PWM](/Pictures/PWM.JPG)

### 定时器 (TIME)

![TIME](/Pictures/TIME.JPG)
![TIME](/Pictures/TIME1.JPG)

### 独立看门狗 (IWDG)

![IWDG](/Pictures/IWDG.JPG)

### 窗口看门狗 (WWDG)

![WWDG](/Pictures/WWDG.JPG)
