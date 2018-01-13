## STM32新建MDK工程

- 下载芯片固件库
- 新建工程
- MDK导入工程
- 添加头文件目录设置宏
- 修改固件库

#### 一、下载芯片固件库

  [stm32固件下载ST官网](http://www.st.com/zh/embedded-software/stm32-standard-peripheral-libraries.html?querycriteria=productId=LN1939)

#### 二、新建工程

1. 新建文件夹

| 文件夹    | 包含内容   |
| --------- | ---------- |
| User      | 主函数     |
| Hardware  | 个人固件库 |
| StdPeriph | 官方固件库 |
| CMSIS     | 系统内核   |
| Readme    | 读我       |
| System    | 正点原子库 |
| MDK-ARM   | 工程文件夹 |

##### 2. STM32F4复制官方固件库到工程

**CMSIS** :

core_cm4.h / core_cmFunc.h / core_cmInstr.h / core_cmSimd.h  /startup_stm32f40_41xxx.s

**User** :

stm32f4xx.h / stm32f4xx_conf.h / stm32f4xx_it.h / stm32f4xx_it.c  / system_stm32f4xx.h  /
system_stm32f4xx.c

**StdPeriph** :

./STM32F4xx_StdPeriph_Driver

##### 3. STM32F1复制官方固件库到工程

**CMSIS**

core_cm3.c  / core_cm3.h  / system_stm32f10x.h

**User**

stm32f10x.h  /  stm32f10x_conf.h  / stm32f10x_it.c  / stm32f10x_it.h  / system_stm32f10x.c

**StdPeriph**

./StdPeriph_Driver
