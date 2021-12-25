# Tree
*熟练掌握树的遍历*
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
