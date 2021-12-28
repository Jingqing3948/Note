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
