7、排序
要求：会代码会执行，能得出中间数据；记住各算法时间复杂度 稳定性
|Name|Time complexity|Others|
|--|--|--|
|【起泡排序】|O(N2)|
|【插入排序】|O(N2)|
|【希尔排序】|O(N3/2) O(N4/3) ……||
|【堆排序】|N log N -O( N log log N )  ~  2N log N -O( N log log N )|【稳定】|
|【归并排序】|O(NlogN)|【稳定，快，浪费空间】|
|【快速排序】|O(NlogN)  ~  O(N2)|【好的时候快】|
# Insertion sort
每次循环将一个元素插入前面已经排好序的序列中
```c
void InsertionSort ( ElementType A[ ], int N ) 
{ 
      int  j, P; 
      ElementType  Tmp; 

      for ( P = 1; P < N; P++ ) { 
	Tmp = A[ P ];  /* the next coming card */
	for ( j = P; j > 0 && A[ j - 1 ] > Tmp; j-- ) 
	      A[ j ] = A[ j - 1 ]; 
	      /* shift sorted cards to provide a position 
                       for the new coming card */
	A[ j ] = Tmp;  /* place the new card at the proper position */
      }  /* end for-P-loop */
}
```
best time complexities: O(N) 刚好不用移动，已经是排好序的情况了
worst time complexities: O(n^2) 正好是反序
Average：O(N^2)
Stable.
# Shell sort
First we define a increment(usually = N/2)
Then we divide the original sequence into increment subsequences:
1 1+increment 1+2*increment ……
 2 2+increment 2+2*increment ……
  3 3+increment 3+2*increment ……
Then we do insertion sort to every subsequences.
Then increment/=2 and do the same thing until increment = 1. then do the last sort and finish.
worst time complexities: O(N^2)
# Heap sort
T(N)=O(Nlog N)
But the space requirement is doubled.
# Merge sort
We divide the sequence into 2 sequences recursively; finally merge them.
```c
/* Lpos = start of left half, Rpos = start of right half */ 
void Merge( ElementType A[ ], ElementType TmpArray[ ], 
	       int Lpos, int Rpos, int RightEnd ) 
{   int  i, LeftEnd, NumElements, TmpPos; 
    LeftEnd = Rpos - 1; 
    TmpPos = Lpos; 
    NumElements = RightEnd - Lpos + 1; 
    while( Lpos <= LeftEnd && Rpos <= RightEnd ) /* main loop */ 
        if ( A[ Lpos ] <= A[ Rpos ] ) 
	TmpArray[ TmpPos++ ] = A[ Lpos++ ]; 
        else 
	TmpArray[ TmpPos++ ] = A[ Rpos++ ]; 
    while( Lpos <= LeftEnd ) /* Copy rest of first half */ 
        TmpArray[ TmpPos++ ] = A[ Lpos++ ]; 
    while( Rpos <= RightEnd ) /* Copy rest of second half */ 
        TmpArray[ TmpPos++ ] = A[ Rpos++ ]; 
    for( i = 0; i < NumElements; i++, RightEnd - - ) 
         /* Copy TmpArray back */ 
        A[ RightEnd ] = TmpArray[ RightEnd ]; 
}
```
S(N)=O(Nlog N) (Waste of space)
T(N)=O(Nlog N)
# Quick sort
- First, choose a node as pivot.
- Every node smaller than pivot should be arranged on the left side of pivot; every node larger than pivot should be arranged on the right side.
- Do this recursively to the left and right side segment.
- If segment length is smaller than 3, then use insertion sort is more convenient.
```c
void Quicksort ( ElementType A[ ], int N )
{
     if ( N < 2 )  return;
     pivot = pick any element in A[ ]; 
     Partition S = { A[ ] \ pivot } into two disjoint sets:
	A1={ a in S | a <= pivot } and A2={ a in S | a >= pivot };
     A = Quicksort ( A1, N1) Union { pivot } Union Quicksort ( A2, N2);
}
```
Which node should be the pivot?
- A wrong way: A[0]
	the worst case is A[] is presorted ——quicksort take O(N^2)time and do nothing.
- A safe maneuver: random select from A[]
	but random number generation is expensive
- Median of Three Partitioning:
	Pivot = median ( left, center, right )
```c
/* Return median of Left, Center, and Right */ 
/* Order these and hide the pivot */ 

ElementType Median3( ElementType A[ ], int Left, int Right ) 
{ 
    int  Center = ( Left + Right ) / 2; 
    if ( A[ Left ] > A[ Center ] ) 
        Swap( &A[ Left ], &A[ Center ] ); 
    if ( A[ Left ] > A[ Right ] ) 
        Swap( &A[ Left ], &A[ Right ] ); 
    if ( A[ Center ] > A[ Right ] ) 
        Swap( &A[ Center ], &A[ Right ] ); 
    /* Invariant: A[ Left ] <= A[ Center ] <= A[ Right ] */ 
    Swap( &A[ Center ], &A[ Right - 1 ] ); /* Hide pivot */ 
    /* only need to sort A[ Left + 1 ] … A[ Right – 2 ] */
    return  A[ Right - 1 ];  /* Return pivot */ 
}
```
T(N)=O(Nlog(N)).
# Bubble sort
easy.
Time complexity: O(N^2)
Space complexity: O(1)
stable.
# Selection sort
Every time, choose the smallest node and placed it at front.
Time complexity: O(N^2)
Space complexity: O(1)
Not Stable.
