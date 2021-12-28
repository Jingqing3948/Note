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


