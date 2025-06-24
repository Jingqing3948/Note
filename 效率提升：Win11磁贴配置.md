---
title: 效率提升：Win11磁贴配置
date: 2023-10-09
tags:
- Windows
categories:
- 踏星（创新探索）
- Efficiency
description: Win11 磁贴配置办公效率提升
---

## 前言

最近电脑还是升级到 win11 了。我之前采用的美化方案是桌面上的图标全部移到 win10 开始菜单里的全屏菜单上，用磁贴贴一排。每次要访问文件的时候都去开始菜单里找，而不是放在桌面上，这样桌面也可以空出来欣赏壁纸。参考配置链接：[如何让Windows 10系统桌面变得更好看？ - 知乎 (zhihu.com)](https://www.zhihu.com/question/45120814/answer/1007955630)

但是升级到 win11 对我而言影响最大的就是压根没有全屏桌面和磁贴功能了。因此我搜了很多解决方案，加上一些自己的改进，最终把桌面磁贴恢复成如上图所示。一个自己喜欢看的桌面还是会对生产力的提高有很大帮助的。

![](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/image-20230712234132339.png)

## 软件安装：start11

首先，恢复开始菜单这一操作最离不开的就是 start11 这款软件。他让 win11 的开始菜单又有了全屏菜单选项，也支持了自定义磁贴在上面。虽然调整磁贴位置的时候偶尔会花几秒重启，但大多数时候还是没问题的。

![](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/image-20230712235022366.png)

正版软件是免费试用的，欢迎付费支持原作者。或采用博主的同款方案：

链接：<https://pan.baidu.com/s/1HY0WuV7ynlXdBI7qfuWZzg?pwd=1fru>
提取码：1fru
--来自百度网盘超级会员V2的分享

首先如果是按博主的磁贴方法配置，就要选“win10配置”。或者你觉得其他风格也还不错都可以选。并且 ENABLE start11.

![](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/image-20230712235704628.png)

点击“配置菜单”，进行如下配置：

![](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/image-20230712235738298.png)

在“自定义菜单视觉外观”里，可以设置全屏菜单的颜色、透明度等，比如我使用的是有一定透明度的毛玻璃的样式。

![](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/image-20230713000601091.png)

然后在“控制”栏里设定如何打开 start11，确保可以打开：

![](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/image-20230712235840844.png)

接着，点击 win 图标就可以打开全屏菜单了。

## 磁贴配置

对于大部分软件，只要右键-固定到开始屏幕/固定到 start11，就可以在全屏菜单里看到刚刚贴上的磁贴了。

如果贴失败，可以尝试以下的方法：

- 右键快捷方式，点击“打开文件所在位置”，再尝试把该文件的 .exe 文件固定到开始屏幕。
- 反复尝试，因为可能有一定的延迟。可以取消固定再次固定，等待一会看开始菜单是否出现。

然后可以手动分组磁贴（把他们移到临近的位置），调整磁贴背景色，调整磁贴大小（有小正方形，中正方形，长方形，大正方形四种可以选择），调整磁贴布局位置。

![](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/image-20230713000428410.png)

## 图片磁贴配置

这里是最自由发挥的部分。高情商：自由发挥。低情商：都要自己做很麻烦。

之前 win10 是有一款快捷工具可以输入自定义图片，按自己想要的格式裁剪并自动在全屏菜单中输出的，叫 Tile Genie.

![](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/image-20230713000854968.png)

但是它好像并不能在 start11 中使用。我尝试了一下导出的都是不能显示的图片块，所以只能放弃这种方法。如果读者的 Tile Genie 是没有问题可以正常显示图片那再好不过了，后面的内容都可以不用看了。

如果导出失败……我采用的方法是手动裁剪固定图片。很笨，但是有结果。

![](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/image-20230713001102790.png)

首先，自行裁剪图片，计算公式为：中正方形 150\*150，长方形 306\*150，大正方形 306\*306，边界线是6（像素）.

我采用的图像裁剪方法是：[免费在线裁剪图像文件 (iloveimg.com)](https://www.iloveimg.com/zh-cn/crop-image)

![1689178422454](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/1689178422454.png)

然后把导出图片找一个合适的地方存储起来，注意贴上磁贴之后就不能再移动修改这些图片了。

在全屏菜单中右键-固定文件，选定文件路径添加。

![](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/image-20230713001454756.png)

刚固定上是这种形式：

![1689178534062](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/1689178534062.png)

然后右键-调整大小，调整为想要的大小。

![](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/image-20230713001613170.png)

![](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/image-20230713001632462.png)

最后一步，右键-图标-选择自定义磁贴图像，再次选择此文件，然后他就被当做图标全屏显示了。

![](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/image-20230713001653804.png)
