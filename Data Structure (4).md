*哈希*
*要求：*
*会算法 给出散列函数 会画图 会计算比较成功的次数 比较失败的次数 平均查找时间*
*【分离链接法】*
*【开放定址法】线性探测法 平方探测法 双散列*
*【再散列】*

# HASHING
散列表（hashing table）的实现叫散列（hashing）散裂是一种用于以常数时间执行插入、删除和查找的技术，但不能排序。
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
如果对应位被占了，则从头开始寻找空位。
第0位是否为空？若为空放入，若不为空检查下一位；
第1位是否为空？……
查找：先去对应位找，如果没找到再从头遍历。
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
#### Rehashing 再散列
当插入数据达到N/2时，再散列。
- 建立一个原表的二倍大的表（接近原表二倍大的素数）
- 从原表中读取数据，用新hash function 存到新表中。
