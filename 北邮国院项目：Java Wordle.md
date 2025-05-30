---
title: 北邮国院项目：Java Wordle
date: 2022-07-20
tags: 
- Java
categories:
- 邮途（北邮国院课内学习）
- Project
description: 北京邮电大学国际学院物联网工程专业Java大作业项目整理
---


# Wordle 游戏介绍
> Wordle的游戏规则很简单，玩家需要猜出程序每天指定的一个5位英语单词谜底。
>
> 玩家可以随意提交一个英语单词，但必须是字典里有的，不能胡乱拼写。
>
> 如果字母在谜底中出现且位置对了就显示绿色，字母出现了但位置不对就显示黄色，字母在答案的单词中没出现就显示灰色。
>
> 根据反馈信息再进行下一轮猜测，在6次尝试之内猜出就算赢。
> 来源：[数学大神攻克猜字游戏Wordle，求解算法成绩逼近理论极限，连信息论都用上了 -量子位](https://mp.weixin.qq.com/s/iddHGL4IaibPq_A59efuyg)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b0d7914573006aedb8e26edd3f4c2a31.png)
# 成果图示
xxxxxxxxxx int zong(int i)//纵坐标y1{    double x=(i-51);    double y=12.0*sin(x/7);    int y1=(int)(15-y);    return y1;}​int heng(int i)//横坐标j{    int j=i;    if(i>101&&i<=202)j=203-i;    else if(i>202&&i<=303)j=i-202;    else if(i>303&&i<=404)j=405-i;    return j;}​void dayin(char ch,int j,int y1)//在(y1,j)处输出给定字符{        gotoxy(y1,j);        if(ch==' ')        {            if(y1==15)printf("-");            else if(j==51)printf("|");            else printf(" ");        }        else printf("#");        gotoxy(29-y1,102-j);        if(ch==' ')//防止坐标点和坐标轴被空格覆盖        {            if(29-y1==15)printf("-");            else if(102-j==51)printf("|");            else if(29-y1==26&&102-j==49)printf("-");            else if((29-y1==26&&102-j==50)||(29-y1==3&&102-j==50))printf("1");            else if(29-y1==16&&102-j==100)printf("X");            else printf(" ");        }        else printf("#");        gotoxy(1,1);}​​​​int main(){    float x,y;    int tim=20;    system("color F0");    system("mode con cols=101 lines=30");    clear();        for(int i=0;i<=100;i++)//打印x轴     {        gotoxy(15,i+1);        if(i!=50)printf("-");        else printf("+");        gotoxy(1,1);        rest(1);    }        for(int i=29;i>=0;i--)//打印y轴     {        gotoxy(i+1,51);        if(i!=14)printf("|");        gotoxy(1,1);         rest(1);    }    //打印轴上的点    gotoxy(16,52); printf("O");        gotoxy(16,100);printf("X");        gotoxy(1,49);printf("Y");        gotoxy(3,50);printf("1");        gotoxy(26,49);printf("-1");                    int i=0,j,y1;​    for(i=52;i<=404;i++)//打印函数     {        j=heng(i),y1=zong(i);        dayin('#',j,y1);        rest(4);                if(i-51-tim>0)        {            y1=zong(i-tim);            j=heng(i-tim);            dayin(' ',j,y1);        }    }       for(int r=i-tim;r<=i;r++)    {        j=heng(r);        y1=zong(r);        dayin(' ',j,y1);        rest(4);    }     gotoxy(1,1);    rest(1000);    return 0;}c
# 玩法介绍
点击HELP 查看帮助，点击 START 开始游戏。
本游戏中没有虚拟键盘，输入框也不能获取鼠标焦点，只能通过键盘键入字母，回车检查答案。
敲下回车后，字母就会呈现灰色、黄色、绿色三种状态，并换到下一行开始下一次猜单词机会。当猜中答案或六轮游戏结束时，弹出小窗，提示用户游戏胜利/失败。
注意：
1. 检查用户输入。本游戏中用户只能输入字母，当用户试图键入数字或符号时会提示只能输入字母。当用户输入不足5位就尝试检查，或尝试输入超出5位时会提示输入必须为5位字母。当用户输入非单词尝试检查会提示输入必须为词库中存在的单词（词库：words.txt，可以自己更新）
2. 一些小的注意点。当用户输入中有两个e，都不在正确的位置上，那这两个e只有一个会显示黄色就够了，另一个显示灰色，否则会影响用户的判断。
# 代码
[Java_Wordle_Game github下载地址](https://github.com/Jingqing3948/Java_Wordle_Game)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/41a2ad5623f9074db53001b89c6d06a6.png)

README：帮助信息
javadoc：生成的 javadoc 
Test：测试图片及说明
words.txt：词库

运行方法：
```java
javac Main.java
java Main
```
在此感谢老师同学对此项目的帮助指导！
欢迎大家star支持[Doge]有问题也可以与博主交流~