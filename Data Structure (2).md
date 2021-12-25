*【表】*
*理解链表、数组的特点（优缺点）*
*会代码实现：单链表 循环链表 双向链表……*

*【栈和队列】*
*理解原理，会算法：两种实现方法*
*会应用算法，如：中序后序互转；前中后缀表达式互换；栈（队列）操作后的状态*

# ADT
**ADT(Abstract Data Type)** is a data type that is organized in such a way that the **specification**  on the objects and **specification** of the operations on the objects are **separated from** the **representation** of the objects and the **implementation** on the operations.

# The List ADT
1.	Array implementation
	- 	**MaxSize** has to be estimated. （大小必须事先给出。给小了数组越界；(可以再申请)给大了太浪费空间）
	- **Find_Kth** takes O(1) time. （随机存取）
	- **Insertion and Deletion** not only take O(N) time, but also involve a lot of data movements which takes time.（插入删除不方便，需要移动大量元素；存储密度高）
2.	Linked Lists（是否带有头结点）
	-	三种操作：Find_Kth、Find_Value、Find_Length take O(N) time. 注意循环逻辑，以及有无头结点时对边界情况的处理
	-	找前一个结点不方便，只能重新从头遍历、
## Doubly Linked Lists
找前一个结点更方便。注意初始化、插入、删除、双向遍历的区别。
## Circular Lists
头尾相接的单链表。注意判空条件（`L->Next==L`）、判断表尾结点条件(`p->Next==L`)以及插入删除时修改的操作。
双向循环链表不再赘述。
## Application: The polynomial ADT
多项式求和、求积函数，使用链表更好，因为如果出现形如x^100000^+x^5^+x^1^这种，数组会浪费大量空间，而单链表只需要将系数非0的部分申请空间创造结点。
（也可以建立十字链表，两个维度分别存储系数和指数）
## Cursor Implementation of Linked Lists (no pointer)
可以使用一个游标int Cursor来作为指针，游标移动访问数组，从而实现类似于指针移动的操作。
The cursor implementation is usually significantly **faster** because of the lack of memory management routines.
# The Stack ADT
A Stack is a **Last-in-First-Out (LIFO)** list. An ordered list in which insertions and deletion are made at the top only.
类似于一堆书，放书和拿书都只能从最顶上进行。最后放上的书最早被拿出来。
Push: 入栈
Top: 读取并返回栈顶元素
Pop: 栈顶元素出栈
## Application 1: Balancing symbols 括号匹配
输入一串表达式，判断其中的括号()[]{}是否匹配。
检验方法：每个右括号都和离他最近的，也就是最晚输入的左括号匹配，正符合栈的后进先出原则。
读到左括号的时候入栈，读到右括号的时候与栈顶的左括号比较，比较后出栈。
最后结束输入，还要检查一下栈是否为空，若非空，栈内剩下的括号全都不匹配。
[7-1 Balancing symbols 代码实现](https://github.com/Jingqing3948/Data-Structure-in-BUPT/blob/main/1%20STACK/7-1%20Balancing%20Symbols.cpp)
## Application 2: Postfix Evaluation 后缀表达式的计算
Infix expression中缀表达式： a + b * c - d / e
Prefix expression 前缀表达式：- + a * b c / d e
Postfix expression 后缀表达式： a b c * + d e / -
中缀表达式更符合人脑的计算方法，而后缀表达式更符合计算机的计算方法。
实际上在计算机中计算表达式，是先转化为postfix expression，再进行计算的。
转换方法：
建立堆栈。
1)	读入的字符是操作数：直接输出
2)	读入的字符是操作符：
a)	堆栈是空的，则直接存入堆栈中（push）
b)	该操作符的优先级>栈顶元素操作符，则直接入栈（push）
c)	该操作符的优先级<=栈顶元素操作符，则栈顶元素出栈（pop），直到该操作符的优先级大于栈顶元素（这个栈顶元素也要出栈），然后该操作符入栈（push）
d)	操作符是左括号，则直接入栈（pop），遇到右括号之前永不输出
e)	操作符是右括号，则一直出栈（pop）直到左括号出栈
f)	最终输入结束，栈若非空，栈中元素全部出栈。

[7-2 Infix to postfix Conversion 代码实现](https://github.com/Jingqing3948/Data-Structure-in-BUPT/blob/main/1%20STACK/7-2%20Infix%20to%20Postfix%20Conversion.cpp)
# The Queue ADT
A Queue is a **First-In-First-Out (FIFO)** list. An ordered list in which insertions take place at one end and deletions take place at the opposite end.
就像日常生活中的排队，先来后到，先排的有机会先出。
EnQueue: 入队
DeQueue: 出队

## Circular Queue：循环链表
## Application: who is the last
输入n、m；n个人坐成一个圈，从第一个人开始报数，每m个人淘汰掉第m个人，然后下一个人继续重新报数。问：谁能留到最后？
[7-4 who is the last 代码实现](https://github.com/Jingqing3948/Data-Structure-in-BUPT/blob/main/2%20QUEUE/7-4%20who%20is%20the%20last.cpp)