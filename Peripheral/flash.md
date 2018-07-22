# 3、嵌入式Flash接口

## 3.1 前言

    Flash 接口可管理 CPU 通过 AHB I-Code 和 D-Code 对 Flash 进行的访问。
    该接口可针对Flash 执行擦除和编程操作，并实施读写保护机制。
    Flash 接口通过指令预取和缓存机制加速代码执行。

## 3.2 主要特征

    ● Flash 读操作
    ● Flash 编程/擦除操作
    ● 读/写保护
    ● I-Code 上的预取操作
    ● I-Code 上的 64 个缓存（128 位宽）
    ● D-Code 上的 8 个缓存（128 位宽）
    图 3 所示为系统架构内的 Flash 接口连接。

![图3](/Pictures/图3.PNG)

## 3.3 嵌入式Flash

    Flash 具有以下主要特性：
    ● 对于 STM32F40x 和 STM32F41x，容量高达 1 MB；对于 STM32F42x 和 STM32F43x，
    容量高达 2 MB
    ● 128 位宽数据读取
    ● 字节、半字、字和双字数据写入
    ● 扇区擦除与全部擦除
    ● 存储器组织结构
    Flash 结构如下：
    — 主存储器块，分为 4 个 16 KB 扇区、1 个 64 KB 扇区和 7 个 128 KB 扇区
    — 系统存储器，器件在系统存储器自举模式下从该存储器启动
    — 512 字节 OTP（一次性可编程），用于存储用户数据
    OTP 区域还有 16 个额外字节，用于锁定对应的 OTP 数据块。
    — 选项字节，用于配置读写保护、BOR 级别、软件/硬件看门狗以及器件处于待机或
    停止模式下的复位。
    ● 低功耗模式（有关详细信息，请参见参考手册的“电源控制 (PWR)”部分）

## 3.4 读接口

### 3.4.1 CPU时钟频率与Flash读取时间之间的关系

    为了准确读取 Flash 数据，必须根据 CPU 时钟 (HCLK) 频率和器件电源电压在
    Flash 存取控制寄存器 (FLASH_ACR) 中正确地编程等待周期数(LATENCY)。
    当电源电压低于 2.1 V 时，必须关闭预取缓冲器。
    Flash 等待周期与 CPU 时钟频率之间的对应关系。
    注意： STM32F405xx/07xx  和 STM32F415xx/17xx  器件：
    -  当 VOS = “ 0 ”时， f HCLK 最大值 = 144 MHz
    -  当 VOS = “ 1 ”时， f HCLK 最大值 = 168 MHz
    STM32F42xxx  和 STM32F43xxx  器件：
    -  当 VOS[1:0] = “ 0x01 ”时， f HCLK 最大值为 120 MHz
    -  当 VOS[1:0] = “ 0x10 ”时， f HCLK 最大值为 144 MHz
    -  当 VOS[1:0] = “ 0x11 ”时， f HCLK 最大值为 168 MHz

### 3.4.2 自适应实时存储器加速器(ART Accelerator™)

    专有的自适应实时 (ART) 存储器加速器面向 STM32 工业标准 ARM ® Cortex™-M4F 处理器
    进行了优化。该加速器很好地体现了 ARM Cortex M4F 的固性能优势，克服了通常条件下，
    高速的处理器在运行中需要经常等待 FLASH 读取的情况。为了发挥处理器的全部性能，该加
    速器将实施指令预取队列和分支缓存，从而提高了 128 位Flash 的程序执行速度。
    根据 CoreMark 基准测试，凭借 ART 加速器所获得的性能相当于Flash 在 CPU 频率
    高达 168 MHz 时以 0 个等待周期执行程序。

## 3.5 擦除和编程操作

    执行任何 Flash 编程操作（擦除或编程）时，CPU 时钟频率 (HCLK) 不能低于 1 MHz。
    如果在 Flash 操作期间发生器件复位，无法保证 Flash 中的内容。在对 STM32F4xx
    的 Flash 执行写入或擦除操作期间，任何读取 Flash 的尝试都会导致总线阻塞。只有
    在完成编程操作后，才能正确处理读操作。意味着，写/擦除操作进行期间不能从 Flash
    中执行代码或数据获取操作。

### 3.5.1  Flash控制寄存器解锁

    复位后，Flash 控制寄存器 (FLASH_CR) 不允许执行写操作，以防因电气干扰等原因出现对
    Flash 的意外操作。此寄存器的解锁顺序如下：
    1. 在 Flash 密钥寄存器 (FLASH_KEYR) 中写入 KEY1 = 0x45670123
    2.  在 Flash 密钥寄存器 (FLASH_KEYR) 中写入 KEY2 = 0xCDEF89AB
    如果顺序出现错误，将返回总线错误并锁定 FLASH_CR 寄存器，直到下一次复位。
    也可通过软件将 FLASH_CR 寄存器中的 LOCK 位置为 1 来锁定 FLASH_CR 寄存器。
    注意： 当 FLASH_SR  寄存器中的 BSY  位为 1  时，将不能在写模式下访问 FLASH_CR 寄存器。
    BSY  位为 1  时，对该寄存器的任何写操作尝试都会导致 AHB  总线阻塞，直到 BSY  位清零。

### 3.5.2  编程/ 擦除并行位数

    通过 FLASH_CR 寄存器中的 PSIZE 字段配置并行位数。并行位数表示每次对 Flash 进行写
    操作时将编程的字节数。PSIZE 受限于电源电压以及是否使用外部 V PP 电源。因此，在进
    行任何编程/擦除操作前，必须在 FLASH_CR 寄存器中对其进行正确配置。Flash 擦除操作只
    能针对扇区或整个 Flash（批量擦除）执行。擦除时间取决于 PSIZE 编程值。有关擦除时间
    的详细信息，请参见器件数据手册的电气特性部分。注意： 如果在编程并行位数 / 电压范围
    设置不一致的情况下启动任何编程或擦除操作，可能会导致出现意外结果。即使后续的读操作
    指示逻辑值已有效写入存储器中，也无法确定写入操作确实成功。要使用 V PP ，必须
    在 V PP 引脚施加一个外部高压电源（ 8 V  到 9 V  之间）。该外部电源必须在直流电耗
    超过 10 mA  时也能维持该电压范围。建议仅在工厂生产线上进行初始编程时使用VPP 。
     V PP 电源的供电时间不得超过一小时，否则 Flash  可能会损坏。

### 3.5.3  擦除

    Flash 擦除操作可针对扇区或整个 Flash（批量擦除）执行。执行批量擦除时，
    不会影响OTP 扇区或配置扇区。
    扇区擦除
    扇区擦除的具体步骤如下：
    1. 检查 FLASH_SR 寄存器中的 BSY 位，以确认当前未执行任何 Flash 操作
    2.  在 FLASH_CR 寄存器中，将 SER 位置 1，并从主存储块的 12 个 (STM32F405xx/07xx
    和 STM32F415xx/17xx) 或 24 个 (STM32F42xxx 和 STM32F43xxx) 扇区中选择要擦除
    的扇区 (SNB)
    3.  将 FLASH_CR 寄存器中的 STRT 位置 1
    4.  等待 BSY 位清零
    批量擦除
    要执行批量擦除，建议采用以下步骤：
    1. 检查 FLASH_SR 寄存器中的 BSY 位，以确认当前未执行任何 Flash 操作
    2.  将 FLASH_CR 寄存器中的 MER 位置 1(STM32F405xx/07xx 和 STM32F415xx/17xx
    器件）
    3.  将 FLASH_CR 寄存器中的 MER 和 MER1 位置 1（STM32F42xxx 和 STM32F43xxx
    器件）
    4.  将 FLASH_CR 寄存器中的 STRT 位置 1
    5.  等待 BSY 位清零
    注意： 如果 FLASH_CR  寄存器中的 MERx  位和 SER  位均置为 1 ，则无法执行
    扇区擦除和批量擦除。

### 3.5.4  编程

    标准编程
    Flash 编程顺序如下：
    1.  检查 FLASH_SR 中的 BSY 位，以确认当前未执行任何主要 Flash 操作。
    2.  将 FLASH_CR 寄存器中的 PG 位置 1。
    3.  针对所需存储器地址（主存储器块或 OTP 区域内）执行数据写入操作：
    — 并行位数为 x8 时按字节写入
    — 并行位数为 x16 时按半字写入
    — 并行位数为 x32 时按字写入
    — 并行位数为 x64 时按双字写入
    4.  等待 BSY 位清零。
    注意： 把 Flash  的单元从“ 1 ”写为“ 0 ”时，无需执行擦除操作即可进行连续写操作。把
     Flash  的单元从“ 0 ”写为“ 1 ”时，则需要执行 Flash  擦除操作。如果同时发出擦除和
     编程操作请求，首先执行擦除操作。

    编程错误
    不允许针对 Flash 执行跨越 128 位行界限的数据编程操作。如果出现这种情况，写操作将不
    会执行，并且 FLASH_SR 寄存器中的编程对齐错误标志位 (PGAERR) 将置 1。
    写访问宽度（字节、半字、字或双字）必须与所选并行位数类型（x8、x16、x32 或 x64）相
    符。否则，写操作将不会执行，并且 FLASH_SR 寄存器中的编程并行位数错误标志位
    (PGPERR) 将置 1。
    如果未遵循标准的编程顺序（例如，在 PG 位未置 1 时尝试向 Flash 地址写入数据），则操
    作将中止并且 FLASH_SR 寄存器中的编程顺序错误标志位 (PGSERR) 将置 1。

    编程与缓存
    如果 Flash 写访问涉及数据缓存中的某些数据，Flash 写访问将修改 Flash 中的数据和缓存中的数据。
    如果 Flash 中的擦除操作也涉及数据或指令缓存中的数据，则必须确保在代码执行期间访问
    这些数据之前将它们重新写入缓存。如果无法可靠执行这一操作，建议将 FLASH_CR 寄存
    器中的 DCRST 和 ICRST 位置 1，以刷新缓存。
    注意： I/D  缓存只有在被禁止 (I/DCEN = 0)  的情况下才能刷新。

### 3.5.5  中断

    如果将 FLASH_CR 寄存器中的操作结束中断使能位 (EOPIE) 置 1，则在擦除或编程操作结
    束时，即 FLASH_SR 寄存器中的繁忙位 (BSY) 清零（操作正确或非正确完成）时，将产生
    中断。此时，FLASH_SR 寄存器中的操作结束 (EOP) 位置 1。
    如果在请求编程、擦除或读操作期间出现错误，则 FLASH_SR 寄存器中的以下错误标志位
    之一将置 1：

    ● PGAERR、PGPERR、PGSERR（编程错误标志）
    ● WRPERR（保护错误标志）

    这种情况下，FLASH_SR 寄存器中的操作错误位 (OPERR) 置 1，并且如果 FLASH_SR 寄
    存器中的错误中断使能位 (ERRIE) 置 1，则将产生一个中断。
    注意： 如果检测到多个连续错误（例如，在对 Flash  进行 DMA  传输期间），则直到连续写操作请
    求结束，这些错误标志才会清零。

## 3.6 选项字节

### 3.6.1  关于用户选项字节的说明

### 3.6.2  用户选项字节编程

    要针对此扇区执行任何操作，Flash 选项控制寄存器 (FLASH_OPTCR) 中的选项锁定位
    (OPTLOCK) 必须清零。为了能够将该位清零，用户需要顺序执行以下步骤：

    1.  在 Flash 选项密钥寄存器 (FLASH_OPTKEYR) 中写入 OPTKEY1 = 0x0819 2A3B
    2.  在 Flash 选项密钥寄存器 (FLASH_OPTKEYR) 中写入 OPTKEY2 = 0x4C5D 6E7F
    通过软件将 OPTLOCK 位置 1 后，可防止用户选项字节发生意外的擦除/编程操作。
    修改 STM32F405xx/07xx  和 STM32F415xx/17xx  上的用户选项字节
    要修改用户选项值，请顺序执行以下步骤：
    1.  检查 FLASH_SR 寄存器中的 BSY 位，以确认当前未执行任何 Flash 操作
    2.  在 FLASH_OPTCR 寄存器中写入所需的选项值。
    3.  将 FLASH_OPTCR 寄存器中的选项启动位 (OPTSTRT) 置 1。
    4.  等待 BSY 位清零。

    注意： 硬件会自动先擦除用户配置扇区，然后以 FLASH_OPTCR  寄存器中包含的值对所有选项字
    节进行编程。
    修改 STM32F42xxx  和 STM32F43xxx  上的用户选项字节
    要修改用户选项字节值，请顺序执行以下步骤：

    1.  检查 FLASH_SR 寄存器中的 BSY 位，以确认当前未执行任何 Flash 操作。
    2.  在 FLASH_OPTCR 和/或 FLASH_OPTCR1 寄存器中写入选项字节值。
    3.  将 FLASH_OPTCR 寄存器中的选项启动位 (OPTSTRT) 置 1。
    4.  等待 BSY 位清零。

    注意： 硬件会自动先擦除用户配置扇区，然后以 FLASH_OPTCR  和 FLASH_OPTCR1  寄存器中包
    含的值对所有选项字节进行编程。

### 3.6.3  读保护 (RDP)

    可对 Flash 中的用户区域实施读保护，以防不受信任的代码读取其中的数据。读保护分三个
    级别，具体定义如下：

    ● 级别 0 ：无读保护
    将 0xAA 写入读保护选项字节 (RDP) 时，读保护级别即设为 0。此时，在所有自举配置
    （用户 Flash 自举、调试或从 RAM 自举）中，均可执行对 Flash 或备份 SRAM 的读/
    写操作（如果未设置写保护）。

    ● 级别 1 ：使能读保护
    这是擦除选项字节后的默认读保护级别。将任意值（分别用于设置级别 0 和级别 2 的
    0xAA 和 0xCC 除外）写入 RDP 选项字节时，即激活读保护级别 1。设置读保护级别
    1 后：
    — 在连接调试功能或从 RAM 或系统存储器自举时，不能对 Flash 或备份 SRAM 进行
    访问（读取、擦除、编程）。读请求将导致总线错误。
    — 从 Flash 自举时，允许通过用户代码对 Flash 和备份 SRAM 进行访问（读取、擦
    除、编程）。
    激活级别 1 后，如果将保护选项字节 (RDP) 编程为级别 0，则将对 Flash 和备份 SRAM
    执行全部擦除。因此，在取消读保护之前，用户代码区域会清零。批量擦除操作仅擦除
    用户代码区域。包括写保护在内的其它选项字节将不受影响。OTP 区域不受批量擦除操
    作的影响，同样保持不变。只有在已激活级别 1 并请求级别 0 时，才会执行批量擦除。
    当提高保护级别 (0->1、1->2、0->2) 时，不会执行批量擦除。

    ● 级别 2 ：禁止调试/ 芯片读保护
    将 0xCC 写入 RDP 选项字节时，可激活读保护级别 2。设置读保护级别 2 后：
    — 级别 1 提供的所有保护均有效。
    — 不再允许从 RAM 或系统存储器自举。
    — JTAG、SWV（单线查看器）、ETM 和边界扫描处于禁止状态。
    — 用户选项字节不能再进行更改。
    — 从 Flash 自举时，允许通过用户代码对 Flash 和备份 SRAM 进行访问（读取、擦
    除、编程）。
    存储器读保护级别 2 是不可更改的。激活级别 2 后，保护级别不能再降回级别 0 或级别 1。
    注意： 激活级别 2  后，将永久性禁止 JTAG  端口（相当于 JTAG  熔断）。这样，将无法执行边界扫
    描。意法半导体无法对设为保护级别 2  的器件做失效分析。

![图5](/Pictures/图5.PNG)

### 3.6.4  写保护

    Flash 中有多达 24 个用户扇区具备写保护功能，可防止因程序指针错乱而发生意外的写
    操作。当 FLASH_OPTCR 或 FLASH_OPTCR1 寄存器中的低有效写保护位 nWRPi
    (0 i 11) 为低电平时，无法对相应的扇区执行擦除或编程操作。因此，如果某个扇区处于
    写保护状态，则无法对整个器件执行全部擦除。
    如果尝试对 Flash 中处于写保护状态的区域执行擦除/编程操作（由写保护位保护的扇区、锁
    定的 OTP 区域或永远不能执行写操作的 Flash 区域，例如 ICP），则 FLASH_SR 寄存器中
    的写保护错误标志位 (WRPERR) 将置 1。
    注意： 选择存储器读保护级别（ RDP  级别 = 1 ）后，如果已连接 CPU  调试功能（ JTAG  调试或单线
    调试）或者正在从 RAM  执行启动代码，则即使 nWRPi = 1 ，也无法对 Flash  扇区 i  执行编程
    或擦除操作。

    写保护错误标志
    如果对 Flash 的写保护区域执行擦除/编程操作，则 FLASH_SR 寄存器中的写保护错误标志
    位 (WRPERR) 将置 1。
    如果请求执行擦除操作，则以下情况下 WRPERR 位置 1：
    ● 配置全部擦除、块擦除、扇区擦除（MER 或 MER/MER1 和 SER = 1）
    ● 请求执行扇区擦除但扇区编号 SNB 字段无效
    ● 请求执行全部擦除，但至少一个用户扇区通过选项位实施了写保护（FLASH_OPTCRx
    寄存器中的 MER 或 MER/MER1 = 1 且 nWRPi = 0，0  i  11 位）
    ● 请求针对写保护扇区执行扇区擦除。（FLASH_OPTCRx 寄存器中 SER = 1、SNB = i
    且 nWRPi = 0，0  i  11 位）
    ● Flash 处于读保护状态，但检测到擦除操作企图。
    如果请求执行编程操作，则以下情况下 WRPERR 位置 1：
    ● 针对系统存储器或用户特定扇区的保留区域执行写操作
    ● 针对用户配置扇区执行写操作
    ● 针对通过选项位实施写保护的扇区执行写操作
    ● 请求针对已锁定的 OTP 区域执行写操作
    ● Flash 处于读保护状态，但检测到写操作企图

## 3.7 一次性可编程字节

    OTP 区域划分为 16 个 32 字节的 OTP 数据块和 1 个 16 字节的 OTP 锁定块。OTP 数据块
    和锁定块均无法擦除。锁定块中包含 16 字节的 LOCKBi (0  i  15)，用于锁定相应的 OTP
    数据块（块 0 到 15）。每个 OTP 数据块均可编程，除非相应的 OTP 锁定字节编程为
    0x00。锁定字节的值只能是 0x00 和 0xFF，否则这些 OTP 字节无法正确使用。

## 3.8  Flash

    ### 3.8.1  Flash  访问控制寄存器 (FLASH_ACR)
    ### 3.8.2  Flash  密钥寄存器 (FLASH_KEYR)
    ### 3.8.3  Flash  选项密钥寄存器 (FLASH_OPTKEYR)
    ### 3.8.4  Flash  状态寄存器 (FLASH_SR)
    ### 3.8.5  用于 STM32F405xx/07xx  和 STM32F415xx/17xx  的 Flash  控制寄存器(FLASH_CR)
    ### 3.8.6  用于 STM32F42xxx  和 STM32F43xxx 的Flash  控制寄存器 (FLASH_CR)
    ### 3.8.7  Flash  选项控制寄存器 (FLASH_OPTCR)
    ### 3.8.8  用于 STM32F42xxx  和 STM32F43xxx  的 Flash  选项控制寄存器(FLASH_OPTCR1)
    ### 3.8.9  Flash  接口寄存器映射

# FLASH 库函数编程

    讲解使用 STM32F4 的官方固件库操作 FLASH 的几个常用函
    数。这些函数和定义分布在文件 stm32f4xx_flash.c 以及 stm32f4xx_flash.h 文件中

### 1）锁定解锁函数

对 FLASH 进行写操作前必须先解锁，解锁操作也就是必须在 FLASH_KEYR 寄
存器写入特定的序列（KEY1 和 KEY2）,固件库函数实现很简单：

    void FLASH_Unlock(void)；

同样的道理，在对 FLASH 写操作完成之后，我们要锁定 FLASH，使用的库函数是：

    void FLASH_Lock(void)；

### 2） 写操作函数

    FLASH_Status FLASH_ProgramDoubleWord(uint32_t Address, uint64_t Data);
    FLASH_Status FLASH_ProgramWord(uint32_t Address, uint32_t Data);
    FLASH_Status FLASH_ProgramHalfWord(uint32_t Address, uint16_t Data);
    FLASH_Status FLASH_ProgramByte(uint32_t Address, uint8_t Data);

    这几个函数从名字上面还是比较好理解意思，分别为写入双字，字，半字，字节的函数。

### 3） 写操作函数

    FLASH_Status FLASH_EraseSector(uint32_t FLASH_Sector, uint8_t VoltageRange);
    FLASH_Status FLASH_EraseAllSectors(uint8_t VoltageRange);
    FLASH_Status FLASH_EraseAllBank1Sectors(uint8_t VoltageRange);
    FLASH_Status FLASH_EraseAllBank2Sectors(uint8_t VoltageRange);

    对于前面两个函数比较好理解，一个是用来擦除某个 Sector，一个使用来擦除全部的 sectors。
    对于第三个和第四个函数，这里的话主要是针对 STM32F42X 系列和 STM32F43X 系列芯片而
    言的，因为它们将所有的 sectors 分为两个 bank。所以这两个函数用来擦除 2 个 bank 下的 sectors
    的 。 第 一 个 参 数 取 值 范 围 在 固 件 库 有 相 关 宏 定 义 标 识 符 已 经 定 义 好 ， 为
    FLASH_Sector_0~FLASH_Sector_11（对于我们使用的 STM32F407 最大是 FLASH_Sector_11），
    对于这些函数的第二个参数，我们这里电源电压范围是 3.3V，所以选择 VoltageRange_3 即可。

### 4） 获取FLASH状态

    FLASH_Status FLASH_GetStatus(void)；

返回值是通过枚举类型定义的：

    typedef enum
    {
      FLASH_BUSY = 1,//操作忙
      FLASH_ERROR_RD,//读保护错误
      FLASH_ERROR_PGS,//编程顺序错误
      FLASH_ERROR_PGP,//编程并行位数错误
      FLASH_ERROR_PGA,//编程对齐错误
      FLASH_ERROR_WRP,//写保护错误
      FLASH_ERROR_PROGRAM,//编程错误
      FLASH_ERROR_OPERATION,//操作错误
      FLASH_COMPLETE//操作结束
    }FLASH_Status;

### 5） 等待操作完成函数

在执行闪存写操作时，任何对闪存的读操作都会锁住总线，在写操作完成后读操作才能正
确地进行；既在进行写或擦除操作时，不能进行代码或数据的读取操作。
所以在每次操作之前，我们都要等待上一次操作完成这次操作才能开始。使用的函数是：

    FLASH_Status FLASH_WaitForLastOperation(void)

返回值是 FLASH 的状态，这个很容易理解，这个函数本身我们在固件库中使用得不多，但是
在固件库函数体中间可以多次看到  

### 6） 等待操作完成函数

有写就必定有读，而读取 FLASH 指定地址的数据的函数固件库并没有给出来，这里我们
提供从指定地址一个读取一个字的函数

    u32 STMFLASH_ReadWord(u32 faddr)
    {
    return *(vu32*)faddr;
    }

### 7） 写选项字节操作

固件库还提供了一些列选项字节区域操作函数

# 硬件设计

    本章实验功能简介：开机的时候先显示一些提示信息，然后在主循环里面检测两个按键，
    其中 1 个按键（KEY1）用来执行写入 FLASH 的操作，另外一个按键（KEY0）用来执行读出
    操作，在 TFTLCD 模块上显示相关信息。同时用 DS0 提示程序正在运行。
    所要用到的硬件资源如下：
    1） 指示灯 DS0
    2） KEY1 和 KEY0 按键
    3） TFTLCD 模块
    4） STM32F4 内部 FLASH
    本章需要用到的资源和电路连接，在之前已经全部有介绍过了，接下来我们直接开始软件
    设计。

# 软件设计

    打开我们的 FLASH 模拟 EEPROM 实验工程，可以看到我们添加了两个文件 stmflash.c 和
    stm32flash.h 。 同 时我 们还 引入 了固 件库 flash 操 作文件 stm32f4xx_flash.c 和头文 件
    stm32f4xx_flash.h。
    打开 stmflash.c 文件，代码如下：
----
    //读取指定地址的半字(16 位数据)
    //faddr:读地址
    //返回值:对应数据.
    u32 STMFLASH_ReadWord(u32 faddr)
    {
      return *(vu32*)faddr;
    }
    //获取某个地址所在的 flash 扇区
    //addr:flash 地址
    //返回值:0~11,即 addr 所在的扇区
    uint16_t STMFLASH_GetFlashSector(u32 addr)
    {
      if(addr<ADDR_FLASH_SECTOR_1)return FLASH_Sector_0;
      else if(addr<ADDR_FLASH_SECTOR_2)return FLASH_Sector_1;
      else if(addr<ADDR_FLASH_SECTOR_3)return FLASH_Sector_2;
      else if(addr<ADDR_FLASH_SECTOR_4)return FLASH_Sector_3;
      else if(addr<ADDR_FLASH_SECTOR_5)return FLASH_Sector_4;
      else if(addr<ADDR_FLASH_SECTOR_6)return FLASH_Sector_5;
      else if(addr<ADDR_FLASH_SECTOR_7)return FLASH_Sector_6;
      else if(addr<ADDR_FLASH_SECTOR_8)return FLASH_Sector_7;
      else if(addr<ADDR_FLASH_SECTOR_9)return FLASH_Sector_8;
      else if(addr<ADDR_FLASH_SECTOR_10)return FLASH_Sector_9;
      else if(addr<ADDR_FLASH_SECTOR_11)return FLASH_Sector_10;
      return FLASH_Sector_11;
    }
    //从指定地址开始写入指定长度的数据
    //特别注意:因为 STM32F4 的扇区实在太大,没办法本地保存扇区数据,所以本函数
    // 写地址如果非 0XFF,那么会先擦除整个扇区且不保存扇区数据.所以
    // 写非 0XFF 的地址,将导致整个扇区数据丢失.建议写之前确保扇区里
    // 没有重要数据,最好是整个扇区先擦除了,然后慢慢往后写.
    //该函数对 OTP 区域也有效!可以用来写 OTP 区!
    //OTP 区域地址范围:0X1FFF7800~0X1FFF7A0F
    //WriteAddr:起始地址(此地址必须为 4 的倍数!!)
    //pBuffer:数据指针
    //NumToWrite:字(32 位)数(就是要写入的 32 位数据的个数.)
    void STMFLASH_Write(u32 WriteAddr,u32 *pBuffer,u32 NumToWrite)
    {
      FLASH_Status status = FLASH_COMPLETE;
      u32 addrx=0;
      u32 endaddr=0;
      if(WriteAddr<STM32_FLASH_BASE||WriteAddr%4)return; //非法地址
      FLASH_Unlock();  //解锁
      FLASH_DataCacheCmd(DISABLE);//FLASH 擦除期间,必须禁止数据缓存
      addrx=WriteAddr;  //写入的起始地址
      endaddr=WriteAddr+NumToWrite*4;  //写入的结束地址
    if(addrx<0X1FFF0000)  //只有主存储区,才需要执行擦除操作!!
    {
      while(addrx<endaddr) //扫清一切障碍.(对非 FFFFFFFF 的地方,先擦除)
    {
    if(STMFLASH_ReadWord(addrx)!=0XFFFFFFFF)
      //有非 0XFFFFFFFF 的地方,要擦除这个扇区
    {
      status=FLASH_EraseSector(STMFLASH_GetFlashSector(addrx),VoltageRange_3);
      //VCC=2.7~3.6V 之间!!
      if(status!=FLASH_COMPLETE)break; //发生错误了
    }else addrx+=4;
    }
    }
    if(status==FLASH_COMPLETE)
    {
        while(WriteAddr<endaddr)//写数据
        {
        if(FLASH_ProgramWord(WriteAddr,*pBuffer)!=FLASH_COMPLETE)//写入数据
          {
            break;  //写入异常
          }
        WriteAddr+=4; pBuffer++;
      }
    }
      FLASH_DataCacheCmd(ENABLE);  //FLASH 擦除结束,开启数据缓存
      FLASH_Lock();//上锁
    }
    //从指定地址开始读出指定长度的数据
    //ReadAddr:起始地址
    //pBuffer:数据指针
    //NumToRead:字(4 位)数
    void STMFLASH_Read(u32 ReadAddr,u32 *pBuffer,u32 NumToRead)
    {
      u32 i;
      for(i=0;i<NumToRead;i++)
      {
        pBuffer[i]=STMFLASH_ReadWord(ReadAddr);//读取 4 个字节.
        ReadAddr+=4;//偏移 4 个字节.
      }
    }
    //////////////////////////////////////////测试用///////////////////////////////////////////
    //WriteAddr:起始地址
    //WriteData:要写入的数据
    void Test_Write(u32 WriteAddr,u32 WriteData)
    {
      STMFLASH_Write(WriteAddr,&WriteData,1);//写入一个字*
    }
----
    该部分代码，我们重点介绍一下 STMFLASH_Write 函数，该函数用于在 STM32F4 的指定
    地址写入指定长度的数据，该函数的实现基本类似第 30 章的 W25QXX_Flash_Write 函数，不
    过该函数使用的时候，有几个要注意要注意：
    1， 写入地址必须是用户代码区以外的地址。
    2， 写入地址必须是 4 的倍数。
    第 1 点比较好理解，如果把用户代码给卡擦了，可想而知你运行的程序可能就被废了，从
    而很可能出现死机的情况。不过，因为 STM32F4 的扇区都比较大（最少 16K，大的 128K），
    所以本函数不缓存要擦除的扇区内容，也就是如果要擦除，那么就是整个扇区擦除，所以建议
    大家使用该函数的时候，写入地址定位到用户代码占用扇区以外的扇区，比较保险。
    第 2 点则是每次必须写入 32 位，即 4 字节，所以地址必须是 4 的倍数。
    关于STMFLASH_GetFlashSector函数，这个就比较好理解了，根据地址确定其sector编号。
    其他函数我们就不做介绍了。
    对 于 头 文 件 stmflash.h ， 这 里 面 有 一 点 提 一 下 ， 就 是 我 们 定 义 了 从
    ADDR_FLASH_SECTOR_0~ ADDR_FLASH_SECTOR_11 等一系列宏定义标识符，实际上这些
    标识符的值就是对应的 sector 的起始地址值，相信也比较好理解。
    最后我们打开 main.c 文件，代码如下：
----
    //要写入到 STM32 FLASH 的字符串数组
    const u8 TEXT_Buffer[]={"STM32 FLASH TEST"};
    #define TEXT_LENTH sizeof(TEXT_Buffer)  //数组长度
    #define SIZE TEXT_LENTH/4+((TEXT_LENTH%4)?1:0)
    /*设置 FLASH 保存地址(必须为偶数，且所在扇区,要大于本代码所占用到的扇区.否则,
    *写操作的时候,可能会导致擦除整个扇区,从而引起部分程序丢失.引起死机.*/
    #define FLASH_SAVE_ADDR 0X0800C004
    int main(void)
    {
      u8 key=0, datatemp[SIZE];
      u16 i=0;
      NVIC_PriorityGroupConfig(NVIC_PriorityGroup_2);//设置系统中断优先级分组 2
      delay_init(168); //初始化延时函数
      uart_init(115200); //初始化串口波特率为 115200
      LED_Init();  //初始化 LED
      LCD_Init();  //LCD 初始化
      KEY_Init();  //按键初始化
      POINT_COLOR=RED;//设置字体为红色
      LCD_ShowString(30,50,200,16,16,"Explorer STM32F4");
      LCD_ShowString(30,70,200,16,16,"FLASH EEPROM TEST");
      LCD_ShowString(30,90,200,16,16,"ATOM@ALIENTEK");
      LCD_ShowString(30,110,200,16,16,"2014/5/9");
      LCD_ShowString(30,130,200,16,16,"KEY1:Write KEY0:Read");
      while(1)
      {
          key=KEY_Scan(0);
          if(key==KEY1_PRES) //KEY1 按下,写入 STM32 FLASH
          {
            LCD_Fill(0,170,239,319,WHITE);//清除半屏
            LCD_ShowString(30,170,200,16,16,"Start Write FLASH....");
            STMFLASH_Write(FLASH_SAVE_ADDR,(u32*)TEXT_Buffer,SIZE);
            LCD_ShowString(30,170,200,16,16,"FLASH Write Finished!");//提示传送完成
          }
          if(key==KEY0_PRES) //KEY0 按下,读取字符串并显示
          {
            LCD_ShowString(30,170,200,16,16,"Start Read FLASH.... ");
            STMFLASH_Read(FLASH_SAVE_ADDR,(u32*)datatemp,SIZE);
            LCD_ShowString(30,170,200,16,16,"The Data Readed Is: ");//提示传送完成
            LCD_ShowString(30,190,200,16,16,datatemp);//显示读到的字符串
          }
          i++; delay_ms(10);
          if(i==20)
            {
            LED0=!LED0;//提示系统正在运行
            i=0;
            }
      }
    }
至此，我们的软件设计部分就结束了。
