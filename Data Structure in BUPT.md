---
title: Data Structure in BUPT
date: 2021-10-01
tags: 
- C
categories:
- Data Structure
---



# 前言

因为考试要求，博主学习的是双语的数据结构，因此自己总结一些英文的概念、知识点。希望也可以对读者学习这门课程有所帮助~
特别感谢许可老师的教导！

理解时间复杂度的定义
（本课程涉及到的所有算法，都需要记住时间复杂度O）

# Data structure notes

Program = Algorithm + Data Structure
An algorithm is a finite set of instructions that, fi followed, accomplishes a particular task. In addition, all algorithms must satisfy the following criteria:

-	Input
-	Output
-	Definiteness
-	Finiteness
-	Effectiveness

*Note:
A program is written in some programming language, and does not have to be finite.
An algorithm can be described by human languages, flow charts, some programming languages, or pseudocode.*

引入 time complexities 和 space complexities 的概念。

$2^n^>n^2^>nlogn>n>logn>1$

# 单词：

Algorithm 算法
Definiteness 确定性
Finiteness 有限性
Effectiveness 有效性
Finite 有限的
Pseudocode 伪代码
Assumptions 假设
Asymptotic 渐近线的，渐近的
Notation 记号，标记法
For loops for循环
Nested 嵌套
Consecutive statements 顺序语句
Fibonacci 斐波那契
Euclid 欧几里得
Exponentiation 幂
Pow 指数

【表】
理解链表、数组的特点（优缺点）
会代码实现：单链表 循环链表 双向链表……

*【栈和队列】*
*理解原理，会算法：两种实现方法*
*会应用算法，如：中序后序互转；前中后缀表达式互换；栈（队列）操作后的状态*

# ADT

**ADT(Abstract Data Type)** is a data type that is organized in such a way that the **specification**  on the objects and **specification** of the operations on the objects are **separated from** the **representation** of the objects and the **implementation** on the operations.

# The List ADT

1.	Array implementation
   - 	**MaxSize** has to be estimated. （大小必须事先给出。给小了数组越界；(可以再申请)给大了太浪费空间）
   - 	**Find_Kth** takes O(1) time. （随机存取）
   - 	**Insertion and Deletion** not only take O(N) time, but also involve a lot of data movements which takes time.（插入删除不方便，需要移动大量元素；存储密度高）
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

树
【基本概念】各类树的度、高、深、节点数与叶子节点的关系；孩子兄弟节点树；
【二分检索树】：概念 平衡二叉树 完全二叉树 满二叉树 AVL树…… 会插入 删除；
【遍历树】：先序 后序 中序 层序；会代码创建树和检索、会执行画图 
【B-树】：会画图 构建B-树，增加节点，删除节点 B-树
【霍夫曼树】：理解原理 给出数据 会画霍夫曼树的图；节点关系
理清各种树之间的关系

# Tree

## Conception

- degree of a node: number of its subtrees
- degree of a tree: max{degree(node)}
- parent;children;siblings
- leaf:a node with degree 0
- path from n_1 to n_k: a **unique** sequence of n_1, n_2 …… n_k (n_i is the parent of n_{i+1})
- length of path: number of **edges** on the path
- depth of n_i: path to root (depth(root)=0)
- height of n_i：length of the longest path from n_i to a leaf.(Height(leaf)=0)
- ancestors of anode: all the nodes along the path from the node up to the root
- descendants of a node: all the nodes in its subtrees

## Representation

### FirstChild-NextSibling Representation

Each Node carries its element and 2 pointers. The first points its FirstChild; The second points its NextSibling.
*This representation is **not unique** since the children in a tree can be of any order.*

### Binary Trees

a tree in wh ich no node can have more than 2 children.
*Rotate the FirstChild-NextSibling tree clockwise by 45°*

## Tree Traversals (significant!)

### Preorder Traversal

visit This Node first;
visit its child recursively.

### Postorder Traversal

visit its child recursively;
At last, visit this node itself.

### Levelorder Traversal

visit Tree via a queue.

- Enqueue root node;
- while(queue is not empty){
- visit first node in queue and Dequeue;
- for its each child, enqueue.}

### Inorder Traversal(for binary trees)

Recursive:

- visit left tree recursively.
- visit root node.
- visit left tree recursively.
  Iterative:

```c
void iter_inorder(Tree)
{
	Stack S;
	for(;;)
	{
		for(;tree;tree=tree->Left)Push(tree,S);
		tree=Top(S);Pop(S);
		if(!tree)break;
		visit(tree->Element);
		tree=tree->Right;
	}
}
```

inorder traversal: infix expression
preorder traversal: prefix expression
postorder traversal: postfix expression
*Depth is a evry important conception. we can use a function to calculate it:*

```c
static void ListDir(DirOrFile D,int Depth)//at first, Depth is 0
{
	if(D is a legitimate entry){
		PrintName(D,Depth);
		if(D is a directory)
			for(each child C of D)
				ListDir(C,Depth+1);
	}
}
```


## Threaded Binary Trees

- If tree->Left is null, 指向中序遍历的前一个结点  a pointer to the inorder predecessor of Tree.
- If tree->Right is null, 指向中序遍历的前一个结点  a pointer to the inorder successor of Tree.
- 最开头、最结尾结点的左、右指针指向head node. There must not be any loose threads. Therefore a threaded binary tree must have a **head node** of which the left child points to the first node.

## The Search Tree ADT -- Binary Search Trees

- Every node has a key which is an integer, and it's distinct.
- The keys in a nonempty **left** subtree must be **smaller** than the key in the root of the subtree.
- The keys in a nonempty **right** subtree must be **larger** than the key in the root of the subtree.
- The left and right subtrees are also binary search trees.

insert is quite easy while delete not.
Delete:

- if it's leaf node:just delete it.
- if it's degree is 1: use its child to replace it.
- if it's degree is 2: use the **largest** node in its **left** subtree or the **smallest** node in its **right** subtree to replace it.
  *will the lazy deletion be in the scope of final exam?*
  **Internal path length: O(Nlog N);depth of any node is O(log N)**
  要会计算Average Search Time (AST)
  *If you print Binary Search Tree inorder, then it is an increasing order.*

Get Depth: preorder traversal

```c
int Height(Tree T)
{
	if(T==NULL)return 1;
	ekse return 1+Max(Height(T->Left),Height(T->Right));
}
```

## AVL Trees

If the tree is too high, AST can be very large and time complexities is O(N). So we need to balance the height by rotating.
**If |h_L-h_R|>1,rotate.**

- Single Rotation: Trouble is left subtree's left subtree or right subtree's right subtree.

```c
Tree SingleRotatewithLeft(Tree T)
{
	TreeNode LeftTree = T->LeftChild;
	T->LeftChild = LeftTree->RightChild;
	LeftTree->RightChild = T;
	return LeftTree;
}

Tree SingleRotatewithRight(Tree T)
{
	TreeNode RightTree = T->RightChild;
	T->RightChild = RightTree->LeftChild;
	RightTree->LeftChild = T;
	return RightTree;
}
```

- Double Rotation: Trouble is left subtree's right subtree or right subtree's left subtree.

```c
Tree DoubleRotatewithLeft(Tree T) 			
{
	T->LeftChild = SingleRotatewithRight(T->LeftChild);
	return SingleRotatewithLeft(T);
}

Tree DoubleRotatewithRight(Tree T) 			
{
	T->RightChild = SingleRotatewithLeft(T->RightChild);
	return SingleRotatewithRight(T);
}
```

## B Trees 要会插入、建立

### Insertion

考完试补全
$$
Depth(M,N)=O(log_{[M/2]}N)
$$

$$
T_{Find}(M,N)=O(log N)
$$

## Forest Empty

## Huffman Tree

Weighted Path Length, WPL
$$
WPL=\sum^k_{i=1}w_i*l_i
$$
Huffman Tree: the binary tree with the minimum weighted path length.
0 is appended to left branch, 
1 is appended to right branch.

哈希
要求：
会算法 给出散列函数 会画图 会计算比较成功的次数 比较失败的次数 平均查找时间
【分离链接法】
【开放定址法】线性探测法 平方探测法 双散列
【再散列】

# HASHING

散列表（hashing table）的实现叫散列（hashing）散列是一种用于以常数时间执行插入、删除和查找的技术，但不能排序。
散列函数（hashing function）是关键字被映射到0～TableSize-1范围内的函数。

- must be easy to compute and minimizes the number of collisions.
- Should be unbiased. For any x and any i, we have that probability(f(x)=i)=1/b. (Uniform hash function)
  ** f(x)=x%TableSize**
  TableSize=prime number is good for random integer keys.

## 处理冲突的办法

### open addressing 开放定址法

find another empty cell to solve collision 

#### Linear Probing 线性探测法

f(i)=i (a linear function)
如果对应位被占了，则往下继续寻找空位。
下一位是否为空？若为空放入，若不为空检查下一位；
再下一位是否为空？……
查找：先去对应位找，如果没找到再继续往下遍历。

#### Quadratic Probing 平方探测法

f(i)=i^2^
如果对应位被占了，则+1^1；
若这一位也被占了，则原位置+2^2；
……
（也有的算法是+-i^2）
最大加到（n/2）的平方
查找：先去对应位找，如果没找到再去加1、加4等等位找。

#### Double Hashing 双散列法

第一个散列函数冲突时，使用第二个散列函数。
**hash2(x)必须不恒等于0！**
*Tips: hash2(x)=R-(x%R) with R a prime smaller than TableSize, will work well.*

### Rehashing 再散列

当插入数据达到N/2时，再散列。

- 建立一个原表的二倍大的表（接近原表二倍大的素数）
- 从原表中读取数据，用新hash function 存到新表中。

堆
【二叉堆】
要求：会构建堆 插入 删除； 画图，手动执行中间数据；

# Piority Queue(Heap)

Can find the element with the highest \ lowest priority.

|                                                              | Insertion             | Deletion              |
| ------------------------------------------------------------ | --------------------- | --------------------- |
| Array                                                        | O(1)                  | find:O(n) delete:O(n) |
| Linked List                                                  | O(1)                  | find:O(n) delete:O(n) |
| Ordered Array                                                | find:O(1) insert:O(n) | O(1)                  |
| Ordered Linked List                                          | find:O(1) insert:O(n) | O(1)                  |
| ## complete binary tree                                      |                       |                       |
| Its nodes correspond to the nodes numbered from 1 to n in the perfect binary tree of height h. |                       |                       |
| A complete binary tree of height h has between 2^h and 2^{h+1}-1 nodes. |                       |                       |
| Array representation: BT[n+1](BT[0]is not used)              |                       |                       |
| For node i:                                                  |                       |                       |
| - parent: i/2                                                |                       |                       |
| - left child: 2*i                                            |                       |                       |
| - right child: 2*i+1                                         |                       |                       |

**A min tree** is a tree in which the key value in each node is no larger than the key values in its children.
**A min heap** :a complete binary tree + a min tree. And the max heap is visa versa.

## Insertion(Min Heap)

Because a heap is a complete binary tree, the nth node only has one possible position to insert. Then, if nth node is smaller than its parent:exchange upward will its location is proper.

```c
for ( i = ++H->Size; H->Elements[ i / 2 ] > X; i /= 2 ) 
	H->Elements[ i ] = H->Elements[ i / 2 ]; 

     H->Elements[ i ] = X; 
```

*Another kind of Insertion is: insert all nodes first, then adjust their location. We need to check from n/2 node. If its children node is smaller than it, choose the smallest node and exchange them. n/2 node go downward until reach the proper location. Then do the same thing to the n/2-1 node until the first node.*

## Deletion

We only need to delete the root node——the smallest node.

- Exchange the root node with the last node——the nth node.
- do the same thing mentioned above in the Italic text: re-arrange the new node.
- Iteratively delete root node until delete the whole heap.
  **Those 2 operations' time complexity are O(log N). Insertion all node is O(nlogn).*

*For the perfect binary tree of height h containing 2h+1 - 1 nodes, the sum of the heights of the nodes is 2h+1 - 1 - (h + 1).*

# Graph

G:graph

V:finite nonempty set of vertices

E:finite set of edges

Undirected graph: $(v_i,v_j)$ *v_i and v_j are adjacent*

Directed graph: $<v_i,v_j>$ *v_i is adjacent to v_j;v_j is adjacent from v_i*

**Complete Graph**: a graph that has the maximum number of edges

**Subgraph**

**Path from v_p to v_q**

**Length of a path**

**Simple path**: v_i1, v_i2 …… v_in are distinct (on the path)  路径上的顶点不重复出现

**Cycle** Simple path with v_p = v_q

**A graph is connected**: every pair of distinct v_i and v_j are connected  图中任意两点都连通

**Connected Component of an undirected G**: the maximal connected subgraph

**A tree**: a graph that is connected and  acyclic

**A DAG**: a directed acyclic graph  有向无环图

**Strongly connected directed graph G**: for every pair of v_i and v_j in V(G), there exist directed paths from v_i to v_j and from v_j to v_i.
(Graph without direction: **weakly connected**)

**Stronhly connected component**: the maximal subgraph that is strongly connected

**Degree(v)**: in-degree and out-degree  进入某个顶点的边数和传出该顶点的边数

## Representation of Graphs

### Adjacency Matrix 邻接矩阵法

二维矩阵存储边信息。如果v_i to v_j 有边，adj_mat[i][j]=1，否则=0.（undirected graph还要考虑双向，同时也要让adj_mat[j][i]=1）

### Adjacency Lists 邻接表法

每个顶点结点都是一个单链表的头结点，后面挂着他连接的顶点的节点。

如：

0->1->2

1->0

2->0

这就是一个三个顶点的无向图，01、02相连。

S(n+2e) Space complexities

## Topological Sort

**AOV Network**: digraph in which V(G) represents activities and E(G) represents precedence relations 顶点表示事件，边表示事件之间的进程关系。比如A->B，就是要先做完A才能做B

**i is a predecessor of j**: there is a path from i to j  前驱

**i is an immediate predecessor**: <i,j>=1.  直接前驱 Then j is called a **successor (immediate successor)** of i 直接后继

**Partial order**: transitive (i->k,k->j: i->j) && irreflexive (i->j is impossible)

AOV network must be a dag (directed acyclic graph)

**Topological order**: a linear ordering of the vertices of a graph such that, for any 2 vertices, i, j, if i is a predecessor of j in the network then i precedes j in the linear ordering.  所有时间都是按完成顺序排序的.
算法：

1. 找出入度为0的节点
2. visit 该节点，并删除该节点的所有出度边
3. 再重新寻找入度为0，没有 visit 过的节点，循环往复

If we use a special box (queue or stack) to sort, T=O(|V|+|E|)

## Shortest Path Algorithms 最短路径算法

两类问题：

1. 从物流中心（一个顶点）往其他所有顶点运输物品，怎样路径最短？
2. 怎样让所有顶点之间路径总和最短？

### 1. Single-Source Shortest-Path Problem

Given as input a weighted graph, G = ( V, E ), and a distinguished vertex, s, find the shortest weighted path from s to every other vertex in G.

**Breadth-first search 广度优先搜索**

Similar to the topological order. We will visit every node connect to the fist node (if they haven't been visit) and Enquque; and Dequeue this node, do the same thing to every node in the Queue.

Just simply scan the whole table

T+(|V|^2+|E|)

**Dijkstra's Algorithm 迪杰斯特拉算法**

建立三个数组，final[]标记哥哥顶点是否已经找到最短路径；dist[]最短路径长度；path[]路径上的前驱
第一次循环：遍历所有结点，找到还没有确定最短路径且dist最小的顶点，final[i]=true
检查i结点的所有final=false邻接节点，如果把i作为路径上的前驱，最短路径会不会缩短？如果会，更新最短路径长度dist和路径上的前驱path信息。
进入下一轮循环。

```c
void Dijkstra( Table T )
{   /* T is initialized by Figure 9.30 on p.303 */
    Vertex  V, W;
    for ( ; ; ) {
        V = smallest unknown distance vertex;
        if ( V == NotAVertex )
	break; 
        T[ V ].Known = true;
        for ( each W adjacent to V )
	if ( !T[ W ].Known ) 
	    if ( T[ V ].Dist + Cvw < T[ W ].Dist ) {
	    	Decrease( T[ W ].Dist  to
			 T[ V ].Dist + Cvw );
		T[ W ].Path = V;
	    } /* end-if update W */
    } /* end-for( ; ; ) */
}
```

**not work for edge with negative cost!**

T=O(|E|log|V|)

## Acylic Graphs

If the graph is acyclic, vertices amy be selected in topological order since when a vertex is selected, its distance can no longer be lowered without any incoming edges from unknown nodes.

T=O(|E|+|V|)

Application: AOE(Activity On Edge)Networks

EC[ j ] \ LC[ j ] :: the earliest \ latest completion time for node v_j

**CPM (Critical Path Method)**

Calculation of EC: start from v_0, for any a_i=<v,w>, we have $EC[w]=max_{(v,w)\in E(EC[v]+C_{v,w})}$

Calculation of LC: Start from the last vertex v_e, for any a_i = <v,w>, $LC[v]=min_{(v,w)\in E}(LC[w]-C_{v,w})$

Slack Time of <v,w>: $LC[w]-EC[v]-C_{v,w}$

Critical Path: path consisting entirely of zero-slack edges.

**All-Pairs Shortest Path Problem**

Method 1: Use single-source algorithm  for |V| times.
T = O(|V|^3) – works fast on sparse graph. 稀疏图

Method 2  O( |V|^3 ) algorithm given in Ch.10, works faster on dense graphs.

## Network Flow Problems

Find Maximum Path from s to t.

Step 1:  Find any path s -> t in Gr ;

Step 2:  Take the minimum edge on this path as the amount of flow and add to G_f ;

Step 3:  Update G_r and remove the 0 flow edges;

Step 4:  If (there is a path s -> t in G_r )
                    Goto Step 1;
               Else
                     End.

## Minimum Spanning Tree

is a tree which consists of V(G) and a subset of E(G). 

- It is acyclic -- the number of edges is |V|-1.
- It is minimum for the total cost of edges is minimized.
- A minimum spanning tree exists if G is connected.

## 2 Method of calculating the minimum spanning tree: 

1. Prim's Algorithm
   very samiliar to Dijkstra’s algorithm. We choose a vertex to start, and every time add a a least cost vertex until all vertex is connected.

2. Kruskal’s Algorithm – maintain a forest
   every time add a a least cost edge <v,w> from E until all vertex is connected.

## DFS -- Depth-First Search

```c
void ListComponents ( Graph G ) 
{   for ( each V in G ) 
        if ( !visited[ V ] ) {
	DFS( V );
               printf(“\n“);
        }
}

void DFS ( Vertex V )  /* this is only a template */
{   visited[ V ] = true;  /* mark this vertex to avoid cycles */
    for ( each W adjacent to V )
        if ( !visited[ W ] )
	DFS( W );
} /* T = O( |E| + |V| ) as long as adjacency lists are used */
```

Applications:

1. Undirected Graphs
2. Biconnectivity
3. Euler Circuits

## NP-Completeness

# KMP

## Basic String Operations

The StrAssign、Strcopy、StrCompare、StrLength、Concat and SubString

## String Operations Library (in C)

gets(str)  //Input a string;

puts(str) //Output a string；

strcat(str1, str2)  // string Concatenation；

strcpy(str1, str2, k)  //String copy；

strcmp(str1, str2) //String comparision；

strlen(str)  //Get the length of string 

## Pattern Matching algorithm

旧模式匹配算法：每次不匹配时，子串都从头重新匹配。效率很低

Basic match algorithm is inefficient

### Knuth-Morris-Pratt (KMP Algorithms)

When dismatch happens, we don't need to go back to first. We can use a next array to decide where to go back.

next[]: compare s[i] and t[j]，if equals then continue comparing following items, else compare s[i] and t [ next[j] ], till end.

```c
void  Get_next( SString T, int  &next[] )
{    j = 1 ;  // postfix pointer
     k = 0;   //prefix pointer
     next[1]=0;
    while ( j <= T[0] )
     {  if ( k == 0 || T[j] == T[k] ) 
             { ++j ;  ++k ;  next[j]=k; }
         else  
             k = next[k];  // k go back, then compare T[j] with T[ next[k] ]
     }// end while
}
```

For example:
a  a  b  c  a  a  a  b  c  a  d
0  1  2  1  1  2  3  3  4  5  6

