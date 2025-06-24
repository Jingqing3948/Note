---
title: 北邮国院项目：程序设计课程设计 C语言订餐系统
date: 2021-09-11
tags: 
- C
categories:
- 邮途（北邮国院课内学习）
- Project
description: 北京邮电大学国际学院物联网工程专业小学期：C语言订餐系统项目整理

---

尽管快半年没有学c了，好在回顾起来还是蛮快的。而且也发现比去年的自己熟练了许多。
小学期要求5人合作，用C语言做一个订餐系统，能够实现用户的登录、选择餐馆、订餐、按各种规则查询与排序、修改个人信息，管理员的登录、添加菜品、按各种规则查询与排序、修改个人信息。
首先，我们调研了几个流行的订餐系统，~~并解决了午饭~~分析其页面逻辑顺序，借此设定了我们自己的订餐系统的逻辑顺序。
感谢blue、peppa、214、Mar鼎力相助！

![](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202505302235979.png)

至于写法，主要尝试了两个新知识点：**文件操作与链表**。

想要实现这样的订餐系统，必须有地方可以存储其数据。不能说放在程序里吧，每次一关一开又恢复原状了。我们使用FILE文件操作，对存放在指定位置的txt进行读写操作，读取其中的数据。

至于数据结构采用结构体数组还是链表，在之前学习翁恺老师的网课时我们认识到了结构体数组的一些弊端。数组大小固定，一旦初始化无法修改，少了可能越界，多了浪费内存空间，链表就可以想用一个结点开辟一个结点。而且对于插入、删除等操作还是链表香啊，数组中间插入或删除一个，后面的都要跟着动。

至于如何实现多人合作，可参考翁恺老师网课12.3.1多个源代码文件用devc++创建项目的方法。我们每人负责了几个函数，最后对好接口和返回值合并。其实就像同学说的，在合并之前是最没底的，也没法测试知道自己写的什么样。好在最终完成的还算顺利。

链表中嵌套链表，是这回我们做的一个非常有意思的尝试。我们发现每次订单中所点的菜品及其数量，和每次订单结束后推荐的菜品数量都是不固定的。对于数据结构还没开始学的我们来说有些困难。后来去问了老师，老师给出了“那你们只能在链表里套一个链表了吧”的解决方案。没想到这个方案真的能做，虽然这铁不是最佳方案。但是也非常有趣。

```c
typedef struct list_dish //订单中的菜品链表
{
	int dish_id; //菜品号
	int number;	 //该菜品数量
	struct list_dish *next;
} LISTDISH;
typedef struct recommend_d
{
	char dish_name[len]; //推荐的菜品名称
	struct recommend_d *next;
} RECOMMEND;
typedef struct list //订单链表
{
	int number;				//订单号
	char user_id[len];		//用户id，据此可以找到用户地址
	int restaurant_id;		//餐馆id
	int score;				//本次订单对餐馆的打分
	int price;				//本次订单的总价格
	TIME delivery_time_s;	//起始配送时间。TIME是一个包含月、日、时、分成员的结构体
	TIME delivery_time_e;	//结束配送时间
	RECOMMEND *recommend_d; //本次用户推荐菜品头结点指针 
	LISTDISH *head_ld;		//本次订单的菜品链表头结点指针
	struct list *next;
} LIST;
```

如上，在订单链表的每个结点中，都包含着一个listdish（存储本次订单所选菜品及其数量）链表的头结点和一个recommend（存储本次订单推荐的菜品）链表的头结点。访问的时候可以通过list->recommend->dish_name这样的方法直接得到本次订单中推荐菜品的菜品名称。

这样的结构，如何从文件中读取？

其他的链表都还好办。至于这三个链表，我们建立了3个txt文件（虽然listdish和recommend有许多链表，但是最好还是都存在一个txt里，不然每次新点订单还要新建txt文件。）其中listdish和recommend文本文档是这样的：

![ ](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202505302235904.png)

每一行代表第几个订单。每两个数字为一组，第一个数字为菜品号，第二个数字为该菜品所点数量。由于不固定长度，我们采用两个-1作为结尾，当读到-1时本订单结束。

```c
LIST *read_inf_l()
{
	LIST *head = (LIST *)malloc(sizeof(LIST))
	, *last = head, *p, *temp = (LIST *)malloc(sizeof(LIST));
	head->next = NULL;
	FILE *fp_l = fopen("D:\\小学期\\list.txt", "r");
	while (fscanf(fp_l, "%d %s %d %d %d %d %d %d %d %d %d %d %d "
	, &temp->number, temp->user_id, &temp->restaurant_id
	, &temp->score, &temp->price
	, &temp->delivery_time_s.month, &temp->delivery_time_s.day
	, &temp->delivery_time_s.hour, &temp->delivery_time_s.minute
	, &temp->delivery_time_e.month, &temp->delivery_time_e.day
	, &temp->delivery_time_e.hour, &temp->delivery_time_e.minute) != EOF)
	{
		p = (LIST *)malloc(sizeof(LIST));
		p->next = NULL;
		p->head_ld = read_inf_ld(temp->number);//跳转到read_inf_ld(int number)函数中获得本次订单listdish链表的头结点，number是订单号
		p->recommend_d = read_inf_recommend(temp->number);//跳转到read_inf_recommend(int number)函数中获得本次订单recommend链表的头结点，number是订单号
		p->number = temp->number;
		strcpy(p->user_id, temp->user_id);
		p->restaurant_id = temp->restaurant_id;
		p->score = temp->score;
		p->price = temp->price;
		p->delivery_time_s.month = temp->delivery_time_s.month;
		p->delivery_time_s.day = temp->delivery_time_s.day;
		p->delivery_time_s.hour = temp->delivery_time_s.hour;
		p->delivery_time_s.minute = temp->delivery_time_s.minute;
		p->delivery_time_e.month = temp->delivery_time_e.month;
		p->delivery_time_e.day = temp->delivery_time_e.day;
		p->delivery_time_e.hour = temp->delivery_time_e.hour;
		p->delivery_time_e.minute = temp->delivery_time_e.minute;
		last->next = p;
		last = last->next;
		last->next = NULL;
	}
	fclose(fp_l);
	return head;
}
```

```c
LISTDISH *read_inf_ld(int id) //Read the Information of List-Dish linklist
{
	LISTDISH *head = (LISTDISH *)malloc(sizeof(LISTDISH))
	, *last = head, *p, *temp = (LISTDISH *)malloc(sizeof(LISTDISH));
	head->next = NULL;
	FILE *fp_ld;
	fp_ld = fopen("D:\\小学期\\listdish.txt", "r");
	int cnt = 1;
	if (id != 0)
		while (cnt != id)
		{

			temp->dish_id = 0;
			while (temp->dish_id != -1)
			{
				fscanf(fp_ld, "%d %d ", &temp->dish_id, &temp->number);
			}
			cnt++;
		}
	fscanf(fp_ld, "%d %d ", &temp->dish_id, &temp->number);
	while (temp->dish_id != -1)
	{
		p = (LISTDISH *)malloc(sizeof(LISTDISH));
		p->dish_id = temp->dish_id;
		p->number = temp->number;
		last->next = p;
		last = last->next;
		last->next = NULL;
		fscanf(fp_ld, "%d %d ", &temp->dish_id, &temp->number);
	}
	fclose(fp_ld);
	return head;
}
```

比如要读取第三个订单所点菜品信息，传入read_inf_ld()的number订单号为3.`int cnt = 1`代表当前文件指针读取的订单号，从1开始计数，每次读到-1 -1证明又读完了一单，cnt++，直到cnt == number为止。然后读取接下来的这个链表，最后返回头结点，作为该订单中的一个成员。

![ ](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202505302235325.png)

```c
RECOMMEND *read_inf_recommend(int id)
{
	RECOMMEND *head = (RECOMMEND *)malloc(sizeof(RECOMMEND)),*last=head
	,*p,*temp = (RECOMMEND *)malloc(sizeof(RECOMMEND));
	head->next=NULL;
	FILE *fp_recommend = fopen("D:\\小学期\\recommend.txt", "r"); 
	int cnt = 1;
    if (id != 0)
        while (cnt != id)
        {
            strcpy(temp->dish_name,"0");
            while (strcmp(temp->dish_name , "-1")!=0)
            {
                fscanf(fp_recommend, "%s ", temp->dish_name);
            }
            cnt++;
        }
    fscanf(fp_recommend, "%s ", temp->dish_name);
    while (strcmp(temp->dish_name ,"-1")!=0)
    {
        p = (RECOMMEND *)malloc(sizeof(RECOMMEND));
        strcpy(p->dish_name, temp->dish_name);
        last->next = p;
        last = last->next;
        last->next=NULL;
        fscanf(fp_recommend, "%s ", temp->dish_name);
    }
	fclose(fp_recommend);
	return head;
}
```

读取推荐菜品链表也是类似。

得到的链表全都包括头结点（里链表也是），不包含结尾的-1，只要通过->->就能访问hh。

总之，结束了~
