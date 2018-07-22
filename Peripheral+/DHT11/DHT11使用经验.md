# DH11温湿度传感器使用经验

##  DHT11  简介

    DHT11 是一款湿温度一体化的数字传感器。该传感器包括一个电阻式测湿元件和一个 NTC
    测温元件，并与一个高性能 8 位单片机相连接。通过单片机等微处理器简单的电路连接就能够
    实时的采集本地湿度和温度。DHT11 与单片机之间能采用简单的单总线进行通信，仅仅需要一
    个 I/O 口。传感器内部湿度和温度数据 40Bit 的数据一次性传给单片机，数据采用校验和方式
    进行校验，有效的保证数据传输的准确性。DHT11 功耗很低，5V 电源电压下，工作平均最大
    电流 0.5mA。
    DHT11 的技术参数如下：
      工作电压范围：3.3V-5.5V
      工作电流 ：平均 0.5mA
      输出：单总线数字信号
      测量范围：湿度 20~90％RH，温度 0~50℃
      精度 ：湿度±5%，温度±2℃
      分辨率 ：湿度 1%，温度 1℃

    正面从左到右引脚分别为：

    VCC   Dout    NC    GND   （电源接反了DH11会发烫，融化塑料外壳）

    DHT11 数字湿温度传感器采用单总线数据格式。即，单个数据引脚端口完成输入输出双向
    传输。其数据包由 5Byte（40Bit）组成。数据分小数部分和整数部分，一次完整的数据
    传输为0bit，高位先出。
    **DHT11 的数据格式为：8bit 湿度整数数据  +  8bit 湿度小数数据  +
    8bit温度整数数据  +  8bit 温度小数数据  +  8bit 校验和
    其中校验和数据为前四个字节相加**

    例如：
          byte4      byte3      byte2      byte1      byte0
        00101101 + 00000000 + 00011100 + 00000000 + 01001001
          整数        小数       整数       小数      校验和
                湿度                  温度

    传感器数据输出的是未编码的二进制数据。

    湿度= byte4 . byte3=45.0 (％RH)
    温度= byte2 . byte1=28.0 ( ℃)
    校验= byte4+ byte3+ byte2+ byte1=73(=湿度+温度)(校验正确)

    DHT11和MCU的一次通信最大为3ms左右，建议主机连续读取时间间隔不要小于 100ms

    DHT11 的数据发送流程：

    首先主机发送开始信号，即：拉低数据线，保持 t1（至少 18ms）时间
    然后拉高数据线 t2（20~40us）时间，然后读取 DHT11 的响应
    正常的话，DHT11 会拉低数据线，保持 t3（40~50us）时间
    作为响应信号，然后 DHT11 拉高数据线，保持 t4（40~50us）时间后，开始输出数据。

    0： 12-14us低电平  --  26-28us高电平
    1： 12-14us低电平  --  116-118us高电平


个人代码：

    //读取温湿度值		
    if(Update == 0)
    {
      if(!DHT11_Read_Data(&temperature,&humidity))
      {
        OLED_ShowNum(30,14,temperature,2,12);		//显示温度	 	
        OLED_ShowNum(30,30,humidity,2,12);		//显示湿度
        OLED_Refresh_Gram();		
      }else{
            OLED_ShowNum(30,14,00,2,12);		//显示温度	 	
            OLED_ShowNum(30,30,00,2,12);		//显示湿度
            OLED_Refresh_Gram();		
      }
      Update = 60;

    }else{
      Update--;
    }

-----    
实例代码：

    //////  头函数配置
    //IO方向设置
    #define DHT11_IO_IN()  {GPIOG->MODER&=~(3<<(9*2));GPIOG->MODER|=0<<9*2;}	//PG9输入模式
    #define DHT11_IO_OUT() {GPIOG->MODER&=~(3<<(9*2));GPIOG->MODER|=1<<9*2;} 	//PG9输出模式
    ////IO操作函数											   
    #define	DHT11_DQ_OUT PGout(9) //数据端口	PG9
    #define	DHT11_DQ_IN  PGin(9)  //数据端口	PG9

    u8 DHT11_Init(void);//初始化DHT11
    u8 DHT11_Read_Data(u8 *temp,u8 *humi);//读取温湿度*
    u8 DHT11_Read_Byte(void);//读出一个字节
    u8 DHT11_Read_Bit(void);//读出一个位
    u8 DHT11_Check(void);//检测是否存在DHT11
    void DHT11_Rst(void);//复位DHT11    
---
    ///// GPIOG9引脚初始化
    //初始化DHT11的IO口 DQ 同时检测DHT11的存在
    //返回1:不存在
    //返回0:存在  
    {
      GPIO_InitTypeDef GPIO_InitStructure;
      RCC_AHB1PeriphClockCmd(RCC_AHB1Periph_GPIOG, ENABLE);//使能 GPIOG 时钟
      //GPIOG9初始化设置
      GPIO_InitStructure.GPIO_Pin = GPIO_Pin_9 ;
      GPIO_InitStructure.GPIO_Mode = GPIO_Mode_OUT;//普通输出模式
      GPIO_InitStructure.GPIO_OType = GPIO_OType_PP;//推挽输出
      GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;//50MHz
      GPIO_InitStructure.GPIO_PuPd = GPIO_PuPd_UP;//上拉
      GPIO_Init(GPIOG, &GPIO_InitStructure);//初始化
      DHT11_Rst();
      return DHT11_Check();
    }
---
    //复位DHT11
    void DHT11_Rst(void)	   
    {                 
    	DHT11_IO_OUT(); 	//SET OUTPUT
      DHT11_DQ_OUT=0; 	//拉低DQ
      delay_ms(20);    	//拉低至少18ms
      DHT11_DQ_OUT=1; 	//DQ=1
    	delay_us(30);     	//主机拉高20~40us
    }

    //等待DHT11的回应
    //返回1:未检测到DHT11的存在
    //返回0:存在
    u8 DHT11_Check(void) 	   
    {   
    	u8 retry=0;
    	DHT11_IO_IN();//SET INPUT	 
        while (DHT11_DQ_IN&&retry<100)//DHT11会拉低40~80us
    	{
    		retry++;
    		delay_us(1);
    	};	 
    	if(retry>=100)return 1;
    	else retry=0;
        while (!DHT11_DQ_IN&&retry<100)//DHT11拉低后会再次拉高40~80us
    	{
    		retry++;
    		delay_us(1);
    	};
    	if(retry>=100)return 1;	    
    	return 0;
    }

---
    //从DHT11读取一个位
    //返回值：1/0
    u8 DHT11_Read_Bit(void) 			 
    {
     	u8 retry=0;
    	while(DHT11_DQ_IN&&retry<100)//等待变为低电平
    	{
    		retry++;
    		delay_us(1);
    	}
    	retry=0;
    	while(!DHT11_DQ_IN&&retry<100)//等待变高电平
    	{
    		retry++;
    		delay_us(1);
    	}
    	delay_us(40);//等待40us
    	if(DHT11_DQ_IN)return 1;
    	else return 0;		   
    }


    //从DHT11读取一个字节
    //返回值：读到的数据
    u8 DHT11_Read_Byte(void)    
    {        
        u8 i,dat;
        dat=0;
    	for (i=0;i<8;i++)
    	{
       		dat<<=1;
    	    dat|=DHT11_Read_Bit();
        }						    
        return dat;
    }


    //从DHT11读取一次数据
    //temp:温度值(范围:0~50°)
    //humi:湿度值(范围:20%~90%)
    //返回值：0,正常;1,读取失败
    u8 DHT11_Read_Data(u8 *temp,u8 *humi)*    
    {        
     	u8 buf[5];
    	u8 i;
    	DHT11_Rst();
    	if(DHT11_Check()==0)
    	{
    		for(i=0;i<5;i++)//读取40位数据
    		{
    			buf[i]=DHT11_Read_Byte();
    		}
    		if((buf[0]+buf[1]+buf[2]+buf[3])==buf[4])
    		{
    			*humi=buf[0];
    			*temp=buf[2];*
    		}
    	}else return 1;
    	return 0;	    
    }
