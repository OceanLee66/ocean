#include "led.h"
#include "delay.h"
#include "LCD1602.h" 
#include "key4.h"
#include "timer.h"
#include "Password.h"
#include "stmflash.h"


#include "string.h" 	


int main(void)
{	
	unsigned char cnt=0;
	delay_init();	    			//延时函数初始化	  
	NVIC_PriorityGroupConfig(NVIC_PriorityGroup_2);//设置中断优先级分组为组2：2位抢占优先级，2位响应优先级
	 
	Key_Init(); 			 		//矩阵按键初始化
	TIM2_Int_Init(499,7199);		//10Khz的计数频率，计数到500为50ms 
    LED_Init();		  				//初始化与控制设备连接的硬件接口
	Lcd1602_Init();
	test();

 	while(1)
	{	
		
		Lcd1602_String(0,0," Password Lock !");
		if(Lock_Flag==1)						//键盘可用
		{
			cnt=KeyScan();						//扫描按键键值 值赋给cnt
			
			if(cnt!=0xff)  						//如果有按键按下
			{
				if(cnt>='0'&&cnt<='9')		  	//如果按下了0-9之间的数字键 则直接输入密码开锁
				{
					Beep_Alram(40);
					Password_Insert();			//输入密码
				}
				else if(cnt=='B')				//如果按下了'B'键	修改密码
				{
					Beep_Alram(40);
					Password_Changed();			//密码修改函数
				}
				else if(cnt=='D')				//如果按下了'D'键	紧急开锁并重置密码
				{
					Beep_Alram(40);
					Password_Open();
					Password_Right = 1;
				}
			}
			if(Password_Right==1)				//输入密码正确？
			{	
				Jd0_Set(GPIO_ON);			   	//开继电器
				Password_Right = 0;	  			//清除标志
				Lcd1602_String(0,0,"  Openning...   ");
				delay_ms(1000);					//延时一会儿
				Jd0_Set(GPIO_OFF);				//关继电器 由于电磁锁不能长时间通电并且由于门锁的特性，并不需要一直开着继电器。
			}
			if(Password_Error3 == 1)			//密码输错误3次标志位
			{
				Password_Error3 = 0;			//清除标志
				Lock_Flag = 0;					//锁死键盘不可用
				TIM_Cmd(TIM2, ENABLE);  		//使能TIMx	使能中断
			}
		}
		else
		{
			Lcd1602_Write_Com(0xc0+1);		 	//第二行0xc0开头
			Lcd1602_Write_Data(0+0x30);
			Lcd1602_Write_Data(0+0x30);
			Lcd1602_Write_Data(':');
			Lcd1602_Write_Data(Sec/10+0x30);
			Lcd1602_Write_Data(Sec%10+0x30);	
		}
		
	}	
}
#include "led.h"

GPIO_STATUS gpioStatus;


// IO初始化
void LED_Init(void)
{

	GPIO_InitTypeDef  GPIO_InitStructure;

	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA|RCC_APB2Periph_GPIOC, ENABLE);	 //使能PB C端口时钟


	
	GPIO_InitStructure.GPIO_Pin =GPIO_Pin_13;				 //PC13    --  STM32板载LED 
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP; 		 //推挽输出
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;		 //IO口速度为50MHz
	GPIO_Init(GPIOC, &GPIO_InitStructure);					 //根据设定参数初始化
	GPIO_SetBits(GPIOC,GPIO_Pin_13);						 //PC13 输出高

	
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_11|GPIO_Pin_12;   //PA11 BEEP  PA12继电器
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP; 		 //推挽输出
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;		 //IO口速度为50MHz
	GPIO_Init(GPIOA, &GPIO_InitStructure);					 //根据设定参数初始化GPIO
	GPIO_SetBits(GPIOA,GPIO_Pin_11|GPIO_Pin_12);			 //PA11、12 输出高

}
void Led0_Set(GPIO_ENUM status)
{

	GPIO_WriteBit(GPIOC, GPIO_Pin_13, status != GPIO_ON ? Bit_SET : Bit_RESET);
	gpioStatus.Led0Sta = status;

}
void Jd0_Set(GPIO_ENUM status)
{

	GPIO_WriteBit(GPIOA, GPIO_Pin_12, status != GPIO_ON ? Bit_SET : Bit_RESET);
	gpioStatus.Jd0Sta = status;

}
void Beep_Set(GPIO_ENUM status)
{

	GPIO_WriteBit(GPIOA, GPIO_Pin_11, status != GPIO_ON ? Bit_SET : Bit_RESET);
	gpioStatus.Jd0Sta = status;

}

/******************************************************************************

 ******************************************************************************
*  文 件 名   : timer.c
timer定时器相关子程序
******************************************************************************/
#include "timer.h"
#include "led.h"
#include "key4.h"


_Bool Lock_Flag = 1;				//键盘可用标志位  1：可用 0：不可用
unsigned char Sec = 60;				//60秒 倒计时

void TIM2_Int_Init(u16 arr,u16 psc)
{
    TIM_TimeBaseInitTypeDef  TIM_TimeBaseStructure;
	NVIC_InitTypeDef NVIC_InitStructure;

	RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM2, ENABLE); 			//时钟使能
	
	//定时器TIM3初始化
	TIM_TimeBaseStructure.TIM_Period = arr; 						//设置在下一个更新事件装入活动的自动重装载寄存器周期的值	
	TIM_TimeBaseStructure.TIM_Prescaler =psc; 						//设置用来作为TIMx时钟频率除数的预分频值
	TIM_TimeBaseStructure.TIM_ClockDivision = TIM_CKD_DIV1; 		//设置时钟分割:TDTS = Tck_tim
	TIM_TimeBaseStructure.TIM_CounterMode = TIM_CounterMode_Up;  	//TIM向上计数模式
	TIM_TimeBaseInit(TIM2, &TIM_TimeBaseStructure); 				//根据指定的参数初始化TIMx的时间基数单位
 
	TIM_ITConfig(TIM2,TIM_IT_Update,ENABLE ); 						//使能指定的TIM3中断,允许更新中断

	//中断优先级NVIC设置
	NVIC_InitStructure.NVIC_IRQChannel = TIM2_IRQn;  				//TIM3中断
	NVIC_InitStructure.NVIC_IRQChannelPreemptionPriority = 1; 		//先占优先级0级
	NVIC_InitStructure.NVIC_IRQChannelSubPriority = 3;  			//从优先级3级
	NVIC_InitStructure.NVIC_IRQChannelCmd = ENABLE;					//IRQ通道被使能
	NVIC_Init(&NVIC_InitStructure); 								//初始化NVIC寄存器


	TIM_Cmd(TIM2, DISABLE);  //失能TIMx					 
}
//定时器2中断服务程序
void TIM2_IRQHandler(void)   //TIM2中断		50ms中断
{
	static unsigned char Cnt = 0;
	if (TIM_GetITStatus(TIM2, TIM_IT_Update) != RESET) 				//检查TIM3更新中断发生与否
	{
		TIM_ClearITPendingBit(TIM2, TIM_IT_Update  );  				//清除TIMx更新中断标志 
		Cnt++;
		if(Cnt>=20)			//达到1秒
		{
			Cnt = 0;		//清0
			Sec--;			//秒--
			if(Sec<=0)		//达到1分钟 后 键盘恢复
			{
				Sec = 60;
				Lock_Flag = 1;	//键盘恢复可用
				TIM_Cmd(TIM2, DISABLE);  //失能TIMx关中断
			}	
		}
	}
}

/******************************************************************************

 ******************************************************************************
*  文 件 名   : key4.c
key4矩阵键盘子程序
******************************************************************************/
#include "key4.h"




unsigned char KeyCode[4][4]=      //按键编码
{


    '1', '4' , '7' , '*' ,
    '2', '5' , '8' , '0' ,
    '3', '6' , '9' , '#' ,	   
    'A', 'B' , 'C' , 'D' ,  		
};

/************************************************************************
* 函数: void Key_Init(void) 
* 描述: 键盘接口初始化
* 参数: none.
* 返回: none.
************************************************************************/
void Key_Init(void) 			 //矩阵按键初始化
{

	GPIO_InitTypeDef      GPIO_InitStructure;
//PA0  PA1  PA4 PA5    1234行   
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA,ENABLE);	 	//使能PC端口时钟	

	GPIO_InitStructure.GPIO_Pin   = GPIO_Pin_0 | GPIO_Pin_1 | GPIO_Pin_4 | GPIO_Pin_5;				 
	GPIO_InitStructure.GPIO_Mode  = GPIO_Mode_Out_PP;      		//推挽输出
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;		 	//IO口速度为50MHz
	GPIO_Init(GPIOA, &GPIO_InitStructure);			     		//根据设定参数初始化端口
	GPIO_SetBits(GPIOA,GPIO_Pin_0 | GPIO_Pin_1 | GPIO_Pin_4 | GPIO_Pin_5); 

//PA6 PA7 PB0 PB1 列信号读取

	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);	 	//使能PA,PD端口时钟
	GPIO_InitStructure.GPIO_Pin =  GPIO_Pin_6 | GPIO_Pin_7;		
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU;           	//上拉输入 
	GPIO_Init(GPIOA, &GPIO_InitStructure);		
	
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB, ENABLE);	 	//使能PA,PD端口时钟
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_0 | GPIO_Pin_1 ;		
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU;           	//上拉输入 
	GPIO_Init(GPIOB, &GPIO_InitStructure);		

}
/************************************************************************
* 函数: static unsigned char Key_ReadLine(void)
* 描述: 键盘行状态读取
* 参数: none.
* 返回: Key：读取到的值
************************************************************************/
static unsigned char Key_ReadLine(void)
{
	unsigned char Key = 0;

	if(LINE0==0)
	{
		Key = 0;   
	}      
	else if(LINE1==0)
	{
		Key = 1;
	}
	else if(LINE2==0)
	{
		Key = 2; 
	}
	else if(LINE3==0)
	{
		Key = 3;
	}
	else
	{
		Key = KEY_INVALID;
	}
	return Key;
}

/************************************************************************
* 函数: static void Key_SetLine(unsigned char num)
* 描述: 设定行扫描
* 参数: num：行列标识
* 返回: none.
************************************************************************/
static void Key_SetLine(unsigned char num)
{
	ROW0_SET;
	ROW1_SET;
	ROW2_SET;
	ROW3_SET;
	switch(num)
	{
		case 0:  ROW0_CLR;  break;
		case 1:  ROW1_CLR;  break;
		case 2:  ROW2_CLR;  break;
		case 3:  ROW3_CLR;  break;
		default:break;
	}
}

/************************************************************************
* 函数: static void KeySetLine(unsigned char num)
* 描述: 按键扫描程序 不支持连续按
* 参数: none.
* 返回: none.
************************************************************************/
unsigned char KeyScan(void)   					//按键扫描程序
{
	static unsigned char KeyState = KEY_UP;
	static unsigned char Row=0;		 
	static unsigned char Line=0;	 
	unsigned char KeyVal = 0;
	switch(KeyState)
	{
		case KEY_UP:   					//按键检测
		{
			for(Row=0;Row<4;Row++)  	//行扫描
			{  	
				Key_SetLine(Row);       //设置扫描行
				Line = Key_ReadLine();  //读取按键值
				if(KEY_INVALID!=Line)
				{
					KeyState = KEY_PRESS; //如果有按键按下记录按键值并进入下一状态
					break;
				}
			}
		}break;

		case KEY_PRESS:	  			 	//按键消抖处理
		{
			Key_SetLine(Row);        	//设置扫描行//KEY_Port = ScanCode[Row];   
			if(Line == Key_ReadLine())
			{
				KeyVal = KeyCode[Row][Line];
				KeyState = KEY_WAIT_UP; //按键确实按下
//		  		KBeep = 1;
			}
			else
			{
				Line = 0;
				KeyState = KEY_UP;     
			}
		}break;

		case KEY_WAIT_UP:				//等待按键抬起释放
		{
			Key_SetLine(Row);         	 
			if(Line == Key_ReadLine())  //按键和开始读到的按键值相同
			{
				KeyState = KEY_WAIT_UP; //等待按键抬起
			}
			else
			{
				Line = 0;
				KeyState = KEY_UP;    	//按键抬起
			}
		}break;

		default:
		{  KeyState = KEY_UP; }break;
	}
	return KeyVal;
}

/******************************************************************************

 ******************************************************************************
*  文 件 名   : LCD1602.c
lcd1602子程序
******************************************************************************/
#include "LCD1602.h"
#include "delay.h"


/************************************************************************
* 函数: void Lcd1602_InitPort(void)
* 描述: 液晶1602接口初始化
* 参数: none.
* 返回: none.
************************************************************************/
void Lcd1602_InitPort(void)
{
	GPIO_InitTypeDef  GPIO_InitStructure;						//定义结构体		
	
	RCC_APB2PeriphClockCmd(GPIOCLK|RCC_APB2Periph_AFIO, ENABLE);//使能功能复用IO时钟，不开启复用时钟不能显示
//  GPIO_PinRemapConfig(GPIO_Remap_SWJ_JTAGDisable,ENABLE);     //禁止JTAG使能SWD 如果用到PB3 PB4需要加这句话
// 	GPIO_PinRemapConfig(GPIO_Remap_SWJ_Disable,ENABLE);     	//禁止JTAG和SWD
	
	GPIO_InitStructure.GPIO_Pin  = LCD_GPIO_DAT;				//数据口配置成开漏输出模式，此模式下读输入寄存器的值得到IO口状态
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_OD;   			//开漏输出
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;  
	GPIO_Init(LCD1602_GPIO , &GPIO_InitStructure);   			//IO口初始化函数（使能上述配置）

	GPIO_InitStructure.GPIO_Pin  = LCD_GPIO_CMD;
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;  			//推挽输出   
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(LCD1602_GPIO , &GPIO_InitStructure); 
		
	GPIO_Write(LCD1602_GPIO ,0xffff);	 

}
/************************************************************************
* 函数: bit Lcd1602_Check_Busy(void)
* 描述: 查忙函数
* 参数: none.
* 返回: 1 ：忙
		0 ：不忙
************************************************************************/ 
_Bool Lcd1602_Check_Busy(void)     //查忙函数
{
	unsigned char busy;
	LCD_RS(0); 
	LCD_RW(1);  
	LCD_EN(1); 
	delay_us(10);
	busy=Text_Busy;
	LCD_EN(0); 
	return (busy);
}
/************************************************************************
* 函数: void Lcd1602_Lcd1602_Write_Com(unsigned char com)
* 描述: 液晶写命令
* 参数: com : 要写的命令
* 返回: none.
************************************************************************/
void Lcd1602_Write_Com(unsigned char dat)
{
	while(Lcd1602_Check_Busy());	//忙检测
	LCD_RS(0);      				//写指令时 RS = 0 RW = 0 ..EN 1->0
	LCD_RW(0);
	LCD_EN(0);
	delay_us(10);
	LCD_WriteData(dat);	  
	delay_us(10);
	LCD_EN(1);
	delay_us(10);
	LCD_EN(0);
}
/************************************************************************
* 函数: void Lcd1602_Write_Data(unsigned char dat)
* 描述: 液晶写数据  要显示什么字符就写入
* 参数: dat ：要写的数据
* 返回: none. 
************************************************************************/
void Lcd1602_Write_Data(unsigned char dat)
{
	while(Lcd1602_Check_Busy());	//忙检测
	LCD_RS(1);      				//写数据时 RS = 1 RW = 0 ..EN 1->0
	LCD_RW(0);
	LCD_EN(0);
	delay_us(10);
	LCD_WriteData(dat);	 
	delay_us(10);
	LCD_EN(1);
	delay_us(10);
	LCD_EN(0);
}
/************************************************************************
* 函数: void Lcd_Init(void)	
* 描述: LCD1602液晶初始化
* 参数: none.
* 返回: none. 
************************************************************************/
void Lcd1602_Init(void)
{	
	Lcd1602_InitPort();    			//初始化LCD1602 GPIO
	Lcd1602_Write_Com(0x38);    	//扩充指令操作
	Lcd1602_Write_Com(0x0c);    	//基本指令操作
	Lcd1602_Write_Com(0x06);    	//显示开，关光标;
	Lcd1602_Write_Com(0x01);    	//清除LCD的显示内容
}
/************************************************************************
* 函数: void write_string(unsigned char x,unsigned char y,unsigned char *string)
* 描述: 显示字符串函数
* 参数: x : x坐标
		y : y坐标
		*string : 要显示的字符串
* 返回: none.
************************************************************************/
void Lcd1602_String(unsigned char x,unsigned char y,unsigned char *string)	
{
	if(y==0)                  			 //设置行
		Lcd1602_Write_Com(0x80+x);       //第一行0x80开头
	else
		Lcd1602_Write_Com(0xc0+x);		 //第二行0xc0开头 
	while(*string)                       //字符串传地址			
	{
		Lcd1602_Write_Data(*string);
		string++;
	}
}

/******************************************************************************

 ******************************************************************************
*  文 件 名   : Password.c
矩阵键盘密码锁相关子程序
******************************************************************************/
#include "sys.h"
#include "Password.h"
#include "LED.h"
#include "delay.h"
#include "key4.h"
#include "LCD1602.h"
#include "stmflash.h"

//要写入到STM32 FLASH的字符串数组
#define SIZE sizeof(Password)	 		//数组长度
#define FLASH_SAVE_ADDR  0X0800f400 	//设置FLASH 保存地址(必须为偶数，且其值要大于本代码所占用FLASH的大小+0X08000000)

_Bool Password_Right = 0;					//密码正确标志位     1：密码正确  
_Bool Password_Error3 = 0;					//密码输错三次标志位 1：密码输出三次
_Bool Password_ChangeOK = 0;				//密码修改完成标志位 1：密码修改完成

unsigned char Password_Input[6];			//输入密码缓存数组
unsigned char Password_Change[6];			//修改密码缓存数组
unsigned char Password[7]={6,6,6,6,6,6,12};	//保存密码数组  默认密码666666 12为存储标志位

/************************************************************************
* 函数: void Beep_Alram(unsigned shor)
* 描述: 按键声
* 参数: ms:响声时长  单位ms
* 返回: none.
************************************************************************/
void Beep_Alram(unsigned short ms)
{
	Beep_Set(GPIO_ON);
	delay_ms(ms);
	Beep_Set(GPIO_OFF);
}
void Password_Read(void)			 //读EEPROM中保存的数据(密码)
{
	STMFLASH_Read(FLASH_SAVE_ADDR,(u16*)Password,SIZE);  //flash中读出数据	
}
void Password_Write(void)			//密码写进eepROM
{
	STMFLASH_Write(FLASH_SAVE_ADDR,(u16*)Password,SIZE);
}
void test(void)						//检测 是不是第一次 并读取EEPROM
{
	STMFLASH_Read(FLASH_SAVE_ADDR,(u16*)Password,SIZE);  //flash中读出数据
	delay_ms(100);					//必须延时
	if(Password[6]!=12)	
	{		
		Password_Write();			//写密码进eepROM		
	}
	else
		Password_Read();			//读取密码
}

/************************************************************************
* 函数: void Disp_char(unsigned char x,unsigned char dat)
* 描述: 显示单个字符函数
* 参数: x：第x+5行的位置 
*		dat：要显示的字符
* 返回: none.
************************************************************************/
void Disp_char(unsigned char x,unsigned char dat)	//显示符号
{
	Lcd1602_Write_Com(0xc5+x);
    Lcd1602_Write_Data(dat);
}

/************************************************************************
* 函数: void Password_Insert(void)
* 描述: 密码输入函数
* 参数: none.
* 返回: none.
************************************************************************/
void Password_Insert(void)
{
	char temp,i=0,j=0;

	Lcd1602_String(0,0," Input Password ");		//刷新屏幕显示
	Lcd1602_String(0,1,"                ");	
	while(1)  									//进入while死循环 进入之后如果没有退出 则程序只执行这个while里边的程序（中断函数除外）
	{
			
		temp=KeyScan();			 			//获取键值
		if(temp>='0'&& temp<='9')  			//如果按键按下的是数字0-9 说明是在输入数字
		{	
			
			if(i<6)	 						//如若输入小于6位数 则可以继续输入
			{
				Beep_Alram(40);
				Password_Input[i]=temp-48;	//把键值符号转换成ASCII对应的值	  
				Disp_char(i,temp); 			//显示输入的数字
				delay_ms(200);				//延时一会儿
				Disp_char(i,'*');  			//显示/*/号 遮挡密码
				i++;
			}
			else			   				//输入满了 长响
			{
				Beep_Alram(300);
			}	
		}
		if(temp=='A')						//删除
		{
			Beep_Alram(40);
			if(i>0)						  	//判断O是否大于0 大于0 说明有输入
			{
				i--;					   	//前面可以看到输入了一次后i被加一了 所以这里减一
				Password_Input[i] = 0;		//清除对应数组的信息
			}
			Disp_char(i,' ');				//显示空白
		}
		if(temp=='C')						//重新输入
		{
			Beep_Alram(40);
			if(i>0)						  	//判断O是否大于0 大于0 说明有输入
			{
				for(;i>0;i--)			    //前面可以看到输入了一次后i被加一了 所以这里减一
				{
					Password_Input[i-1] = 0;//清除对应数组的信息 
					Disp_char(i-1,' ');		//显示空白	
				}					   		
			}
			
		}
		if(temp=='*')						//*退出
		{
			Beep_Alram(40);
			Lcd1602_Write_Com(0x01); 		//清屏
			break;							//退出循while循环 回到主界面 
		}
		if(temp=='#'&&i>5)					//按#确认
		{
			Beep_Alram(40);
			if((Password_Input[0]==Password[0])&&(Password_Input[1]==Password[1])		//如果密码正确
			&&(Password_Input[2]==Password[2])&&(Password_Input[3]==Password[3])
			&&(Password_Input[4]==Password[4])&&(Password_Input[5]==Password[5]))
			{
				i = 0;			
				Password_Right = 1; 		//密码正确标志位
				Lcd1602_String(0,0," Password Right ");		//提示一下
				Lcd1602_String(0,1,"                ");
				delay_ms(800);								//延时一会儿
				Lcd1602_Write_Com(0x01); 	//清屏			
				break;						//退出循while循环 回到主界面 
			}
			else  							//如果密码输入错误
			{
				j++;
				//这里可以加报警
				Beep_Alram(300);delay_ms(100);	
				Beep_Alram(300);delay_ms(100);
				Beep_Alram(300);delay_ms(100);
				if(j>=3)					//输入三次错误
				{
					j=0;
					Password_Error3 = 1;	//密码输错误3次标志位
					Lcd1602_Write_Com(0x01);//清屏
					break;					//退出循while循环 回到主界面 
				}
				i=0;	  
				if(j==1)Lcd1602_String(0,1,"Password Error1!");		//显示提示密码输错一次	
				if(j==2)Lcd1602_String(0,1,"Password Error2!");		//显示提示密码输错二次
				delay_ms(800);
				Lcd1602_String(0,0," Input Password ");
				Lcd1602_String(0,1,"                ");			
			}
		}

	}	
}
/************************************************************************
* 函数: void Password_Change(void)
* 描述: 密码修改函数
* 参数: none.
* 返回: none.
************************************************************************/
void Password_Changed(void)
{
	unsigned char temp,i=0,j=0;;

	Lcd1602_String(0,0," Input Password ");		//刷新屏幕显示
	Lcd1602_String(0,1,"                ");	
	while(1)  									//进入while死循环 进入之后如果没有退出 则程序只执行这个while里边的程序（中断函数除外）
	{
			
		temp=KeyScan();			 			//获取键值

		if(temp>='0'&& temp<='9')  			//如果按键按下的是数字0-9 说明是在输入数字
		{	
			if(i<6)	 						//如若输入小于6位数 则可以继续输入
			{
				Beep_Alram(40);
				Password_Input[i]=temp-48;	//把键值符号转换成ASCII对应的值
				Disp_char(i,temp); 			//显示输入的数字
				delay_ms(200);				//延时一会儿
				Disp_char(i,'*');  			//显示/*/号 遮挡密码
				i++;
			}
			else			   				//输入满了 长响
			{
				Beep_Alram(300);
			}	
		}
		if(temp=='A')						//删除
		{
			if(i>0)						  	//判断O是否大于0 大于0 说明有输入
			{
				Beep_Alram(40);
				i--;					   	//前面可以看到输入了一次后i被加一了 所以这里减一
				Password_Input[i] = 0;		//清除对应数组的信息
			}
			Disp_char(i,' ');				//显示空白
		}
		if(temp=='C')						//重新输入
		{
			if(i>0)						  	//判断O是否大于0 大于0 说明有输入
			{
				Beep_Alram(40);
				for(;i>0;i--)			    //前面可以看到输入了一次后i被加一了 所以这里减一
				{
					Password_Input[i-1] = 0;//清除对应数组的信息 
					Disp_char(i-1,' ');		//显示空白	
				}					   		
			}
			
		}
		if(temp=='*')						//*退出
		{
			Beep_Alram(40);
			Lcd1602_Write_Com(0x01); 		//清屏
			break;							//退出循while循环 回到主界面 
		}
		if(temp=='#'&&i>5)					//按#确认
		{
			Beep_Alram(40);
			if((Password_Input[0]==Password[0])&&(Password_Input[1]==Password[1])		//如果密码正确
			&&(Password_Input[2]==Password[2])&&(Password_Input[3]==Password[3])
			&&(Password_Input[4]==Password[4])&&(Password_Input[5]==Password[5]))
			{
				i = 0;			
				Lcd1602_String(0,0," Password Right ");		//提示一下
				Lcd1602_String(0,1,"                ");
				delay_ms(800);								//延时一会儿
				Lcd1602_Write_Com(0x01); 					//清屏			
				Lcd1602_String(0,0,"  New Password  ");		//显示输入新密码提示
				Lcd1602_String(0,1,"                ");
				while(1)	  								//进入输入新密码界面  进入之后如果没有退出 则程序只执行这个while里边的程序（中断函数除外）
				{
						
					temp=KeyScan();			 			//获取键值
					
					if(temp>='0'&& temp<='9')  			//如果按键按下的是数字0-9 说明是在输入数字
					{	
						if(i<6)	 						//如若输入小于6位数 则可以继续输入
						{
							Beep_Alram(40);
							Password_Change[i]=temp-48;	//把键值符号转换成ASCII对应的值
							Disp_char(i,temp); 			//显示输入的数字
							delay_ms(200);				//延时一会儿
							Disp_char(i,'*');  			//显示/*/号 遮挡密码
							i++;
						}
						else			   				//输入满了 长响
						{
							Beep_Alram(300);
						}	
					}
					if(temp=='A')						//删除
					{
						if(i>0)						  	//判断O是否大于0 大于0 说明有输入
						{
							Beep_Alram(40);
							i--;					   	//前面可以看到输入了一次后i被加一了 所以这里减一
							Password_Change[i] = 0;		//清除对应数组的信息
						}
						Disp_char(i,' ');				//显示空白
					}
					
					if(temp=='C')						//重新输入
					{
						if(i>0)						  	//判断O是否大于0 大于0 说明有输入
						{
							Beep_Alram(40);
							for(;i>0;i--)			    //前面可以看到输入了一次后i被加一了 所以这里减一
							{
								Password_Input[i-1] = 0;//清除对应数组的信息 
								Disp_char(i-1,' ');		//显示空白	
							}					   		
						}
						
					}
					if(temp=='*')						//*退出
					{
						Beep_Alram(40);
						Lcd1602_Write_Com(0x01); 		//清屏
						break;							//退出循while循环 回到主界面 
					}
					if(temp=='#'&&i>5)					//按#确认
					{
						Beep_Alram(40);
						i = 0;
						for(i=0;i<6;i++)
						{
							Password[i]=Password_Change[i];		//临时密码赋给password数组
						}
						i=0;
						//写入EEPROM
						Password_Write();						//密码写进eepROM
						Lcd1602_Write_Com(0x01);				//清屏
						Lcd1602_String(0,0,"Change  Succeed ");	//设置成功提示
						delay_ms(1500);
						
						Password_ChangeOK = 1;   				//密码修改完成
						Lcd1602_Write_Com(0x01);				//清屏
						break;									//跳出输入新密码界面 			
					}


				}
				if(Password_ChangeOK==1) 							//如果密码修改完了   
				{ 
					Password_ChangeOK=0; 							//清除标志 
					break; 											//跳出输入原密码界面 	
				}   		  
			}
			else  													//如果密码输入错误
			{
				j++;
				//这里可以加报警
				Beep_Alram(300);delay_ms(100);	
				Beep_Alram(300);delay_ms(100);
				Beep_Alram(300);delay_ms(100);
				if(j>=3)											//输入三次错误
				{
					j=0;
					Password_Error3 = 1;							//密码输错误3次标志位
					Lcd1602_Write_Com(0x01);						//清屏
					break;											//退出循while循环 回到主界面 
				}
				i=0;	  
				if(j==1)Lcd1602_String(0,1,"Password Error1!");		//显示提示密码输错一次	
				if(j==2)Lcd1602_String(0,1,"Password Error2!");		//显示提示密码输错二次
				delay_ms(800);
				Lcd1602_String(0,0," Input Password ");
				Lcd1602_String(0,1,"                ");			
			}
		}

	}	
}
/************************************************************************
* 函数: void Password_Open(void)
* 描述: 一键开锁并复位密码
* 参数: none.
* 返回: none.
************************************************************************/
void Password_Open(void)
{
	unsigned char i = 0;
	while(1)
	{
		for(i=0;i<6;i++)
		{
			Password[i]=6;						//默认密码赋给password数组
		}
		i=0;
		//写入EEPROM
		Password_Write();						//密码写进eepROM
		break;									//跳出循环		
	}
}
#include "stmflash.h"
#include "delay.h"
#include "usart.h"
 

//读取指定地址的半字(16位数据)
//faddr:读地址(此地址必须为2的倍数!!)
//返回值:对应数据.
u16 STMFLASH_ReadHalfWord(u32 faddr)
{
	return *(vu16*)faddr; 
}
#if STM32_FLASH_WREN	//如果使能了写   
//不检查的写入
//WriteAddr:起始地址
//pBuffer:数据指针
//NumToWrite:半字(16位)数   
void STMFLASH_Write_NoCheck(u32 WriteAddr,u16 *pBuffer,u16 NumToWrite)   
{ 			 		 
	u16 i;
	for(i=0;i<NumToWrite;i++)
	{
		FLASH_ProgramHalfWord(WriteAddr,pBuffer[i]);
	    WriteAddr+=2;//地址增加2.
	}  
} 
//从指定地址开始写入指定长度的数据
//WriteAddr:起始地址(此地址必须为2的倍数!!)
//pBuffer:数据指针
//NumToWrite:半字(16位)数(就是要写入的16位数据的个数.)
#if STM32_FLASH_SIZE<256
#define STM_SECTOR_SIZE 1024 //字节
#else 
#define STM_SECTOR_SIZE	2048
#endif		 
u16 STMFLASH_BUF[STM_SECTOR_SIZE/2];//最多是2K字节
void STMFLASH_Write(u32 WriteAddr,u16 *pBuffer,u16 NumToWrite)	
{
	u32 secpos;	   //扇区地址
	u16 secoff;	   //扇区内偏移地址(16位字计算)
	u16 secremain; //扇区内剩余地址(16位字计算)	   
 	u16 i;    
	u32 offaddr;   //去掉0X08000000后的地址
	if(WriteAddr<STM32_FLASH_BASE||(WriteAddr>=(STM32_FLASH_BASE+1024*STM32_FLASH_SIZE)))return;//非法地址
	FLASH_Unlock();						//解锁
	offaddr=WriteAddr-STM32_FLASH_BASE;		//实际偏移地址.
	secpos=offaddr/STM_SECTOR_SIZE;			//扇区地址  0~127 for STM32F103RBT6
	secoff=(offaddr%STM_SECTOR_SIZE)/2;		//在扇区内的偏移(2个字节为基本单位.)
	secremain=STM_SECTOR_SIZE/2-secoff;		//扇区剩余空间大小   
	if(NumToWrite<=secremain)secremain=NumToWrite;//不大于该扇区范围
	while(1) 
	{	
		STMFLASH_Read(secpos*STM_SECTOR_SIZE+STM32_FLASH_BASE,STMFLASH_BUF,STM_SECTOR_SIZE/2);//读出整个扇区的内容
		for(i=0;i<secremain;i++)//校验数据
		{
			if(STMFLASH_BUF[secoff+i]!=0XFFFF)break;//需要擦除  	  
		}
		if(i<secremain)//需要擦除
		{
			FLASH_ErasePage(secpos*STM_SECTOR_SIZE+STM32_FLASH_BASE);//擦除这个扇区
			for(i=0;i<secremain;i++)//复制
			{
				STMFLASH_BUF[i+secoff]=pBuffer[i];	  
			}
			STMFLASH_Write_NoCheck(secpos*STM_SECTOR_SIZE+STM32_FLASH_BASE,STMFLASH_BUF,STM_SECTOR_SIZE/2);//写入整个扇区  
		}else STMFLASH_Write_NoCheck(WriteAddr,pBuffer,secremain);//写已经擦除了的,直接写入扇区剩余区间. 				   
		if(NumToWrite==secremain)break;//写入结束了
		else//写入未结束
		{
			secpos++;				//扇区地址增1
			secoff=0;				//偏移位置为0 	 
		   	pBuffer+=secremain;  	//指针偏移
			WriteAddr+=secremain;	//写地址偏移	   
		   	NumToWrite-=secremain;	//字节(16位)数递减
			if(NumToWrite>(STM_SECTOR_SIZE/2))secremain=STM_SECTOR_SIZE/2;//下一个扇区还是写不完
			else secremain=NumToWrite;//下一个扇区可以写完了
		}	 
	};	
	FLASH_Lock();//上锁
}
#endif

//从指定地址开始读出指定长度的数据
//ReadAddr:起始地址
//pBuffer:数据指针
//NumToWrite:半字(16位)数
void STMFLASH_Read(u32 ReadAddr,u16 *pBuffer,u16 NumToRead)   	
{
	u16 i;
	for(i=0;i<NumToRead;i++)
	{
		pBuffer[i]=STMFLASH_ReadHalfWord(ReadAddr);//读取2个字节.
		ReadAddr+=2;//偏移2个字节.	
	}
}

//////////////////////////////////////////////////////////////////////////////////////////////////////
//WriteAddr:起始地址
//WriteData:要写入的数据
void Test_Write(u32 WriteAddr,u16 WriteData)   	
{
	STMFLASH_Write(WriteAddr,&WriteData,1);//写入一个字 
}


