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

*For the perfect binary tree of height h containing 2h+1  1 nodes, the sum of the heights of the nodes is 2h+1 - 1 - (h + 1).*
