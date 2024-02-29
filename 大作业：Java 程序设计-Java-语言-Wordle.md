---
title: 大作业：Java 程序设计-Java-语言-Wordle
date: 2022-07-20
tags:
- Java
categories:
- Project
---

# Wordle 游戏介绍
> Wordle的游戏规则很简单，玩家需要猜出程序每天指定的一个5位英语单词谜底。
>
> 玩家可以随意提交一个英语单词，但必须是字典里有的，不能胡乱拼写。
>
> 如果字母在谜底中出现且位置对了就显示绿色，字母出现了但位置不对就显示黄色，字母在答案的单词中没出现就显示灰色。
>
> 根据反馈信息再进行下一轮猜测，在6次尝试之内猜出就算赢。
> 来源：https://news.mydrivers.com/1/813/813695.htm#:~:text=Wordle%E7%9A%84%E6%B8%B8,%E4%BD%8D%E8%8B%B1%E8%AF%AD%E5%8D%95%E8%AF%8D%E8%B0%9C%E5%BA%95%E3%80%82

![在这里插入图片描述](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/494e131e4b2641d48becd4aef3f90c7d.png)
# 成果图示
博主大二期间学习的java课程大作业，就是写一款 wordle 游戏，主要考察 GUI 界面的开发。
最终成果图示：
![在这里插入图片描述](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/008abdff020b4e05bc56265450bc7442.png)
![在这里插入图片描述](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/1840388526344189a27121987da22d47.png)
![在这里插入图片描述](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/74f2dd38063349298b84d23cafb28db2.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/e9aacefcb2984adf913ea8c80fee24ff.png)
![在这里插入图片描述](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/07e7dd5c4a4348afb35cfc7cc009ca88.png)
![在这里插入图片描述](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/f163f8a63ff14878b74691ba432fd882.png)
![在这里插入图片描述](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/de75c6c50c9b45a3bbd2c58f95104217.png)

# 玩法介绍
点击HELP 查看帮助，点击 START 开始游戏。
本游戏中没有虚拟键盘，输入框也不能获取鼠标焦点，只能通过键盘键入字母，回车检查答案。
敲下回车后，字母就会呈现灰色、黄色、绿色三种状态，并换到下一行开始下一次猜单词机会。当猜中答案或六轮游戏结束时，弹出小窗，提示用户游戏胜利/失败。
注意：
1. 检查用户输入。本游戏中用户只能输入字母，当用户试图键入数字或符号时会提示只能输入字母。当用户输入不足5位就尝试检查，或尝试输入超出5位时会提示输入必须为5位字母。当用户输入非单词尝试检查会提示输入必须为词库中存在的单词（词库：words.txt，可以自己更新）
2. 一些小的注意点。当用户输入中有两个e，都不在正确的位置上，那这两个e只有一个会显示黄色就够了，另一个显示灰色，否则会影响用户的判断。
# 代码
[Java_Wordle_Game github下载地址](https://github.com/Jingqing3948/Java_Wordle_Game)
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