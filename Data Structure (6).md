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

**Topological order**: a linear ordering of the vertices of a graph such that, for any 2 vertices, i, j, if i is a predecessor of j in the network then i precedes j in the linear ordering.  所有时间都是按完成顺序排序的

If we use a special box (queue or stack) to sort, T=O(|V|+|E|)

## Shortest Path Algorithms 最短路径算法
两类问题：
1. 从物流中心（一个顶点）往其他所有顶点运输物品，怎样路径最短？
2. 怎样让所有顶点之间路径总和最短？

### 1. Single-Source Shortest-Path Problem
Given as input a weighted graph, G = ( V, E ), and a distinguished vertex, s, find the shortest weighted path from s to every other vertex in G.
**Breadth-first search 广度优先搜索**
Similar to the topological order. We will visit every node connect to the fist node (if they haven't been visit) and Enquque; and Dequeue this node, do the same thing to every node in the Queue.

**Dijkstra's Algorithm 迪杰斯特拉算法**