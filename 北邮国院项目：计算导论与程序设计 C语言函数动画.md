---
title: 北邮国院项目：计算导论与程序设计 C语言函数动画
date: 2020-12-15
tags: 
- C
categories:
- 邮途（北邮国院课内学习）
- Project
description: 北京邮电大学国际学院物联网工程专业计算导论与程序设计大作业C语言函数动画项目整理
---

**绘制一个函数动画。**
要求：1.碰到边界反弹；
2.会像贪吃蛇一样消失。

老师已给出提示代码：三个辅助函数

```c
#include<stdio.h>
#include<windows.h>
#include<math.h>
// two functions to clear screen and set cursor position for printing.
// one function to take a rest in 10 microseconds.

// use this function to clear screen
void clear()//清屏
{
#if defined _WIN32
    system("cls");
#elif defined (__LINUX__) || defined(__gnu_linux__) || defined(__linux__)
    system("clear");
#elif defined (__APPLE__)
    system("clear");
#endif
}

#if defined _WIN32
    #include <conio.h>
    #include <windows.h>
    void setcursor(int x, int y)
    {
        HANDLE hCon = GetStdHandle(STD_OUTPUT_HANDLE);
        COORD setps;
        setps.X = x; setps.Y = y;
        SetConsoleCursorPosition(hCon, setps);
    }
#endif


// use this function to positioning before calling printf  
void gotoxy(int x, int y)//光标移动到x,y处
{
#if defined _WIN32
    setcursor(y - 1, x - 1);
#elif defined (__LINUX__) || defined(__gnu_linux__) || defined(__linux__)
    printf("\033[%d;%dH", x, y);
#elif defined (__APPLE__)
    printf("\033[%d;%dH", x, y);
#endif
}

int factor = 10;

// rest msec * 10 microseconds
void rest(int msec)//当前画面静止一定时间
{
#if defined _WIN32
    Sleep(msec * factor);
#elif defined (__LINUX__) || defined(__gnu_linux__) || defined(__linux__)
    usleep(msec * 1000 * factor);
#elif defined (__APPLE__)
    usleep(msec * 1000* factor);
#endif
}
```
其中`clear()`函数清屏；`gotoxy()`函数使光标移动到指定位置；`rest()`当前画面静止。

**思路**
1.动画绘制
动画的原理是一帧帧的图片有短暂的时间间隔，连起来放映之后就产生了会动的感觉。我们可以每打印一个点之后`rest()`一小段时间来形成动画的效果。

2.边界反弹
即为碰到边界反向输出。我想的方法是纵坐标y1不变，只需要设计分段函数让x反向即可。

3.延时消失
要求的消失是像贪吃蛇一样，前面一边画着点，后面尾巴一边消失。但自己怎么想都只能做到全部绘制完成之后再让它开始消失。
请教助教与Mc大佬之后得知他的思路是保留点数。屏幕中最多出现50个点，第51个点之后，打印第x个点时，x-50号点打印空格覆盖，让他消失。

**注意事项**
1.审题。给定的`gotoxy()`函数的横纵坐标是反着的，而且从左上角开始为(1,1)。
2.`rest()`的用法非常灵活，比如高人指点：用户可以输入打印速度speed，rest(100-speed)。（妙啊）
3.程序中出现几段几乎完全相似的代码（除了字母不同），“代码复制”是程序不良的表现。因为将来做修改、维护的时候要维护很多处。所以比起重复一段好多遍，使用函数会更好。

感谢Mc、fOrever_jAckY、olinr1222、--_--等人指点！
我の代码

```c
int zong(int i)//纵坐标y1
{
	double x=(i-51);
	double y=12.0*sin(x/7);
	int y1=(int)(15-y);
	return y1;
}

int heng(int i)//横坐标j
{
	int j=i;
	if(i>101&&i<=202)j=203-i;
	else if(i>202&&i<=303)j=i-202;
	else if(i>303&&i<=404)j=405-i;
	return j;
}

void dayin(char ch,int j,int y1)//在(y1,j)处输出给定字符
{
		gotoxy(y1,j);
		if(ch==' ')
		{
			if(y1==15)printf("-");
			else if(j==51)printf("|");
			else printf(" ");
		}
		else printf("#");
		gotoxy(29-y1,102-j);
		if(ch==' ')//防止坐标点和坐标轴被空格覆盖
		{
			if(29-y1==15)printf("-");
			else if(102-j==51)printf("|");
			else if(29-y1==26&&102-j==49)printf("-");
			else if((29-y1==26&&102-j==50)||(29-y1==3&&102-j==50))printf("1");
			else if(29-y1==16&&102-j==100)printf("X");
			else printf(" ");
		}
		else printf("#");
		gotoxy(1,1);
}




int main()
{
	float x,y;
	int tim=20;
	system("color F0");
	system("mode con cols=101 lines=30");
	clear();
	
	for(int i=0;i<=100;i++)//打印x轴 
	{
		gotoxy(15,i+1);
		if(i!=50)printf("-");
		else printf("+");
		gotoxy(1,1);
		rest(1);
	}
	
	for(int i=29;i>=0;i--)//打印y轴 
	{
		gotoxy(i+1,51);
		if(i!=14)printf("|");
		gotoxy(1,1); 
		rest(1);
	}
	//打印轴上的点
	gotoxy(16,52); printf("O");
	
	gotoxy(16,100);printf("X");
	
	gotoxy(1,49);printf("Y");
	
	gotoxy(3,50);printf("1");
	
	gotoxy(26,49);printf("-1");
	
	
	
	
	int i=0,j,y1;

	for(i=52;i<=404;i++)//打印函数 
	{
		j=heng(i),y1=zong(i);
		dayin('#',j,y1);
		rest(4);
		
		if(i-51-tim>0)
		{
			y1=zong(i-tim);
			j=heng(i-tim);
			dayin(' ',j,y1);
		}
	}	
	for(int r=i-tim;r<=i;r++)
	{
		j=heng(r);
		y1=zong(r);
		dayin(' ',j,y1);
		rest(4);
	} 
	gotoxy(1,1);
	rest(1000);
	return 0;
}
```