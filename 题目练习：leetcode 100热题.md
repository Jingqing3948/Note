---
title: 题目练习：Leetcode 100热题
date: 2025-01-18
tags: 
- Algorithm
- Java
categories:
- 算道（课外IT技能学习）
- Practice
description: Leetcode 100热题练习
---

重点学习 100 热题中的优化解。

## 哈希

### 1. 两数之和

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506172044608.png" alt=" " style="zoom:80%;" />

优化方法：把数组的下标和值放到一个哈希表中。为什么是哈希表？因为哈希表的键唯一，满足我们这道题的需求。

比如数组是 [3,3,6]，找到之和 = 10 的下标。我用最简单的遍历方法获取：

```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        Map<Integer, Integer> hash_table=new HashMap<Integer, Integer>();
        for(int i=0;i<nums.length;i++){
            if(hash_table.containsKey(target-nums[i])){
                return new int[] {hash_table.get(target-nums[i]),i};
            }
            hash_table.put(nums[i],i);
        }
        return new int[0];
    }
}
```

第一次遍历，3+3,3+6 都不符合。

第二次遍历，又出现了 3+6，这个计算结果我们之前已经出现过，可以优化掉不用再计算一次的。

所以我们可以把数组的下标-值放到哈希表的值-键映射中，每次遍历挑一个元素寻找哈希表中有没有对应值的键，查找速度更快，而且相同值的数组元素也无法同时放入哈希表，避免重复遍历查找。

那么对于两个相同值的数组元素相加 = 目标值的情况，如何处理？

采取这样的逻辑：

1. 并不是一开始就把所有元素放入哈希表中的。
2. 每次遍历，用当前元素和哈希表中的元素进行配对，如果成功配对则返回。不成功配对，则将当前元素存入哈希表，再继续遍历下一个元素。
3. 也就是说，相当于每个元素都和“他之前出现过的所有元素”进行遍历匹配。如果当前元素是 3，之前哈希表中存有元素 3 了，也可以成功匹配。

代码：

```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        Map<Integer, Integer> hash_table=new HashMap<Integer, Integer>();
        for(int i=0;i<nums.length;i++){
            if(hash_table.containsKey(target-nums[i])){
                return new int[] {hash_table.get(target-nums[i]),i};
            }
            hash_table.put(nums[i],i);
        }
        return new int[0];
    }
}
```

### 49. 字母异位词分组

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506172042130.png" alt=" " style="zoom:80%;" />

一眼看上去也是哈希的思路。

键：排序过后的字符串。

值：一个装有所有对应组合的字符串的列表。

难点主要就是语法。好久不刷 Java 了，想了半天，哎。

```java
class Solution {
    public List<List<String>> groupAnagrams(String[] strs) {
        Map<String, List<String>> map=new HashMap<String, List<String>>();
        for (String s : strs){
            char[] temp_char_array=s.toCharArray();
            Arrays.sort(temp_char_array);
            String sp=String.valueOf(temp_char_array);
            if(!map.containsKey(sp)){
                map.put(sp, new ArrayList<>());
            }
            map.get(sp).add(s);
        }
        return new ArrayList<List<String>>(map.values());
    }
}
```

### 128. 最长连续序列

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506172042165.png" alt=" " style="zoom:80%;" />

首先，时间复杂度 O(n) 就绝对不能排序了。

我们可以先遍历一遍，把所有元素存入 hashSet；

然后遍历 hashSet 中的每个元素，让这个元素无限向后延伸找到最大连续长度。如果这个元素不是最大连续长度的起始点就不予考虑。这样相当于每个点其实只求了一次最大连续长度，要么是作为起始点向后延伸求的，要么是作为中间点被其他点求过了，跳过。

```java
class Solution {
    public int longestConsecutive(int[] nums) {
        Set<Integer> set=new HashSet<>();
        for(int num : nums)set.add(num);
        int max_length=0;
        for(int p: set){
            if(!set.contains(p-1)){
                int temp_length=0;
                while(set.contains(p++))temp_length++;
                max_length=max_length>temp_length?max_length:temp_length;
            }
        }
        return max_length;
    }
}
```

## 双指针

### 283. 移动零

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506172044115.png" alt=" " style="zoom:67%;" />

不能复制数组，即在原数组上进行操作。那么遍历的顺序就有讲究，我们要把原数组划分成“待处理的序列”和”已经处理过的序列“进行区分，每次指针 1 位于已经处理过的序列尾部等待指针 2 给他换数字过来，指针 2 在待处理的序列不断遍历寻找新的非零数给指针 1 传过来。

题目要求的最终输出结果是：左侧全部是数字且保留原数组顺序；右侧全部是 0。

那么我们可以从左到右遍历，指针 2 检测到数字的时候和指针 1 交换，从未处理过的序列中提取出所有的数字拼接到已经处理过的数字序列中。

```java
class Solution {
    public void moveZeroes(int[] nums) {
        if (nums == null || nums.length <= 1)
            return;
        int p1 = 0, p2 = 0, n = nums.length;
        while (p2 < n) {
            if (nums[p2] != 0) {
                swap(nums, p1, p2);
                p1++;
            }
            p2++;
        }
    }

    public void swap(int[] nums, int p1, int p2) {
        int temp = nums[p1];
        nums[p1] = nums[p2];
        nums[p2] = temp;
    }
}
```

### 11. 盛最多水的容器

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506172043476.png" alt=" " style="zoom:67%;" />

最基础的做法当然是双层循环遍历找到容量最大值。

不过容量计算有意思的地方在于“短板”思想。假设我们固定一个维度：p1 p2 指针之间的距离不断减小，那么容器的高度只有不断增加才有可能让容积增加，也就是两块板之间较小的那一块的高度增加。不然就算长板长度增加了，短板长度还是不变，容器可用高度还是取决于短板。

我们假设 p1 从最左端开始往右遍历，p2 从最右端开始向左遍历。遍历的整个过程 p1 p2 的间距是不断减小的，如果想找到容积更大的容器，就必须持续不断地让 p1 p2 中数值更小的“短板”向内移动变为新值。所以每次循环，我们都比较两板的长度，并选择长度较小的那一方移动。

```java
class Solution {
    public int maxArea(int[] height) {
        int p1 = 0, p2 = height.length - 1;
        int[] max_container = { 0, 0, 0 };// p1, p2, volume
        while (p1 < p2) {
            int volume = (height[p1] > height[p2] ? height[p2] : height[p1]) * (p2 - p1);
            if (volume > max_container[2]) {
                max_container[0] = p1;
                max_container[1] = p2;
                max_container[2] = volume;
            }
            if (height[p1] > height[p2]) {
                p2--;
            } else
                p1++;
        }
        return max_container[2];
    }
}
```

### 15. 三数之和

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506172042643.png" alt=" " style="zoom:67%;" />

基本做法还是很简单的三个指针遍历。在此基础上要注意两点：

1. 注意不能出现重复组合。三个指针不要出现重叠的情况；同时指针遍历移动的时候，如果下一个位置和上一个位置值相同，就继续移动直到出现新值为止。
2. 时间复杂度优化：数组排序后，我们固定三个指针为：p1 p2 p3，且 p1 < p2 < p3，nums [p1] < nums [p2] < nums [p3]。每次循环我们先固定 p1，然后 p2 从 p1+1 到 nums.length-2 开始遍历，p3 从 nums.length-1 到 p1+1 开始遍历，且 p2 < p3 （即：**假设 p1 是小数，p2 是中等数，p3 是大数**。不然如果 p3 到了 p2 左边，就会出现重复的情况了）. 这样可以跳过更多重复情况。
3. 时间复杂度优化：如果 nums [p1]+nums [p2] > 0 了，那就不用找 p2 和 p3 了，因为 p2 指针再向右移动，求和值也只会增加；且 nums [p3] > nums [p2]，nums [p1]+nums [p2]+nums [p3] > nums [p1]+nums [p2]+nums [p2] > 0，也一定找不到符合条件的 p3 了。所以可以直接 **不用继续遍历剩下的 p2 和 p3，开始遍历下一次 p1 即可**。

```java
class Solution {
    public List<List<Integer>> threeSum(int[] nums) {
        int n=nums.length;
        Arrays.sort(nums);
        List<List<Integer>> list = new ArrayList<>();
        for (int i = 0; i < n-2; i++) {
            if (i != 0 && nums[i - 1] == nums[i])
                continue;
            int target = -nums[i];
            int k = n - 1;
            for (int j = i + 1; j < n; j++) {
                if (j != i + 1 && nums[j - 1] == nums[j])
                    continue;
                while (k > j && nums[j] + nums[k] > target)
                    k--;
                if (k <= j)
                    break;
                if (nums[j] + nums[k] == target) {
                    List temp_list = new ArrayList<Integer>();
                    temp_list.add(nums[i]);
                    temp_list.add(nums[j]);
                    temp_list.add(nums[k]);
                    list.add(temp_list);
                }
            }
        }
        return list;
    }
}
```

双指针还有一道接雨水，不过我想先刷完简单中等，再往进阶进发吧，这样效率高一些。

## 滑动窗口

久等了~

滑动窗口的特点是左右两端有两个指针标注着窗口左右界限。

如果我们要找一个满足需求的固定长度的窗口：记录下当前窗口与目标窗口之间的差异，每次循环剔除掉左侧元素，加入右侧新元素并再次比较。这样不用双层循环每次都重新遍历目标窗口长度。

如果要找一个非固定长度的窗口：右侧不断延伸，如果出现不满足条件的情况，则左侧指针也向右移动直到新窗口再次满足条件为止。随后右侧指针继续延伸。

### 3. 无重复字符的最长子串

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506172044091.png" alt=" " style="zoom:80%;" />

这道题对应上面我们说过的第二种滑动窗口情况。

1. 左指针从 0 开始，右指针从 1 开始滑动（当然首先要考虑字符串长度 <= 1 的情况）。
2. 右侧窗口指针不断滑动，如果每次滑动没有出现重复字符则当前长度+1，并判断是否要更新最大长度。
3. 如果右侧窗口指针出现了该窗口中已经出现过的重复元素，则左侧窗口指针向右移动直到略过重复元素出现的位置，形成新窗口。此时不用判断新窗口长度是否为最大长度，因为右侧向右移动了 1，左侧向右至少移动了 1，长度不可能变得比上一轮循环的窗口长度更长。

比如对于示例 1：

起始窗口： ptr_left = 0, ptr_right = 1, (ab), window_length = 2

第一次移动：    ptr_left = 0, ptr_right = 2, (abc), window_length = 3

第二次移动：    ptr_left = 0, ptr_right = 3, (abca)，出现重复元素。则左指针向右移动直到不再出现重复元素为止：ptr_left = 1, ptr_right = 3, (bca)

第三次移动：    ptr_left = 1, ptr_right = 4, (bcab)，出现重复元素。则左指针向右移动直到不再出现重复元素为止：ptr_left = 2, ptr_right = 4, (cab)

……

至于窗口中是否重复元素的判断方式，最优解还是哈希表查找。遍历我觉得也还行吧。

```java
class Solution {
    public int lengthOfLongestSubstring(String s) {
        if (s == null || s.length() == 0)
            return 0;
        int left = 0, right = 1, max_length = 1;
        Set<Character> set = new HashSet<Character>();
        set.add(s.charAt(left));
        while (right < s.length()) {
            if (!set.add(s.charAt(right))) {
                int i = left;
                while (s.charAt(i) != s.charAt(right)) {
                    set.remove(s.charAt(i));
                    i++;
                }
                left = i + 1;
            } else {
                if (right - left + 1 > max_length) {
                    max_length = right - left + 1;
                }
            }
            right++;
        }
        return max_length;
    }
}
```

### 438. 找到字符串中所有字母异位词

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506172042360.png" alt=" " style="zoom:80%;" />

这种属于我们介绍的滑动窗口第一种情况：固定窗口长度。

对于待检索的字符串 s，我们可以先存储 [0, p.length()-1] 范围的一个窗口范围的子串。随后每轮循环中舍弃掉左侧的一个元素，添加右侧的一个新元素，重点关注增加这两处变动后新子串和目标子串的 **差异变化**。

关于差异，题解采取了用长度为 26 的数组记录字符差异的方法。目标子串中有而当前子串中没有的记为+1，当前子串中有而目标子串中没有的记为-1. 当数组所有元素求和 = 0 的话说明当前窗口子串和目标子串无差异。

```java
class Solution {
    public List<Integer> findAnagrams(String s, String p) {
        List<Integer> return_list = new ArrayList<Integer>();

        if (s.length() < p.length())
            return return_list;
        int[] count = new int[26];

        for (int i = 0; i < p.length(); i++) {
            count[p.charAt(i) - 'a']++;
            count[s.charAt(i) - 'a']--;
        }
        int differ = 0;
        for (int i = 0; i < 26; i++) {
            if (count[i] != 0)
                differ++;
        }
        if (differ == 0)
            return_list.add(0);

        for (int i = 0; i < s.length() - p.length(); i++) {
            if (count[s.charAt(i) - 'a'] == -1) {
                --differ;
            } else if (count[s.charAt(i) - 'a'] == 0)
                ++differ;
            count[s.charAt(i) - 'a']++;

            if (count[s.charAt(i + p.length()) - 'a'] == 1) {
                --differ;
            } else if (count[s.charAt(i + p.length()) - 'a'] == 0)
                ++differ;
            count[s.charAt(i + p.length()) - 'a']--;
            if (differ == 0)
                return_list.add(i + 1);
        }
        return return_list;
    }
}
```

## 子串

### 560. 和为 K 的子数组

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506172042131.png" alt=" " style="zoom:80%;" />

直接解很简单，双重循环遍历求和。

优化解很容易想到是存储部分子串计算结果避免重复计算，但是官方题解的解题思路更加巧妙，利用两个前缀作差获取连续子串。

我们只遍历一次数组，每次循环中记录从 0 开始到当前位置的前缀子串求和值 pre，并在此轮循环中查找有多少个求和 = pre-目标值 k 的前缀子串。因为用两个前缀子串作差的结果一定是一个连续子串。

```java
class Solution {
    public int subarraySum(int[] nums, int k) {
        int count = 0;
        HashMap<Integer, Integer> map = new HashMap<>();
        map.put(0, 1);
        int pre = 0;
        for (int i = 0; i < nums.length; i++) {
            pre += nums[i];
            if (map.containsKey(pre - k))
                count += map.get(pre - k);
            map.put(pre, map.getOrDefault(pre, 0) + 1);
        }
        return count;
    }
}
```

## 数组

### 53. 最大子数组和

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506172043243.png" alt=" " style="zoom:80%;" />

普通做法：O(n^2) 双层循环，找到所有可能的子数组和其求和，依次与最大数组求和比较是否更新。

我选择动态规划做法，不研究分治了，贪心也不错而且写起来还简单。

简单来说就是从数组开头开始依次加每个元素，如果当前数组和为负数就可以把这部分数组和全部舍弃掉，从 0 开始累加下一个数组元素。因为左侧部分如果求和是负数，那么加到右侧部分只可能会让右侧的数组总和变得更小，只有舍弃掉负数部分才能取到最大值。

还有一种额外情况是形如：[-1] 这样的数组。数组中只有一个元素而且是负数，最大和就是-1 而不是 0，所以我们找不到比 0 大的数组最大和。所以我们可以设定初始数组值为一个非常小的负数，先判断当前数组和是否大于最大值，再判断当前数组和是否小于零，小于零舍弃，继续遍历。

```java
class Solution {
    public int maxSubArray(int[] nums) {
        int sum = 0, pre = 0, temp_sum = 0, max_sum = -10000;
        for (int i = 0; i < nums.length; i++) {
            temp_sum += nums[i];
            if (max_sum < temp_sum)
                max_sum = temp_sum;
            if (temp_sum < 0)
                temp_sum = 0;
        }
        return max_sum;
    }
}
```

### 56. 合并区间

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506172042886.png" alt=" " style="zoom:80%;" />

普通做法：可以用一个大数组记录所有区间包含的所有元素，再把这个大数组转化成若干个区间的形式。

进阶做法：先对所有的区间进行排序（按左侧起点大小顺序排序。这里难点在于 Comparator 的语法），而后对于两个区间项前一项的终点 > 后一项的起点的情况就可以合并。合并结果是 [前一项的起点，Math.max(前一项的终点，后一项的终点)].

```java
class Solution {
    public int[][] merge(int[][] intervals) {
        if (intervals.length == 0)
            return new int[0][2];

        Arrays.sort(intervals, new Comparator<int[]>() {
            public int compare(int[] interval1, int[] interval2) {
                return interval1[0] - interval2[0];
            }
        });

        List<int[]> result = new ArrayList<int[]>();
        for (int i = 0; i < intervals.length; i++) {
            if (result.size() == 0 || result.get(result.size() - 1)[1] < intervals[i][0]) {
                result.add(new int[] { intervals[i][0], intervals[i][1] });
            } else
                result.get(result.size() - 1)[1] = Math.max(intervals[i][1], result.get(result.size() - 1)[1]);
        }

        return result.toArray(new int[result.size()][]);
    }
}
```

### 189. 轮转数组

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506172042439.png" alt=" " style="zoom:67%;" />

普通做法：开辟一块新的数组区间，把原数组 i 位置元素放到新数组 (i+k)%nums.length 的位置。时间复杂度 O(n) 空间复杂度 O(n).

reverse 做法：数组轮转其实相当于整个数组 reverse 一下，然后 [0, k-1] 区间的子数组 reverse 一下，[k, nums.length-1] 区间的子数组 reverse 一下。这个非常巧妙，最好记住。

```java
class Solution {
    public void rotate(int[] nums, int k) {
        if (nums.length <= 1)
            return;
        k %= (nums.length);
        reverse(nums, 0, nums.length - 1);
        reverse(nums, 0, k - 1);
        reverse(nums, k, nums.length - 1);
    }

    public void reverse(int[] nums, int L, int R) {
        while (L < R) {
            int temp = nums[L];
            nums[L] = nums[R];
            nums[R] = temp;
            L++;
            R--;
        }
    }
}
```

还有一种环状替换方法感兴趣的同学可以研究一下。

### 238. 除自身以外数组的乘积

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506172043812.png" alt=" " style="zoom:67%;" />

普通做法：O(n^2)，对于新数组每个元素，用原数组所有元素（除对应位置上的元素）遍历相乘获得。或者说是这个元素左侧的所有元素遍历乘积（L [i]）*右侧所有元素遍历乘积（R [i]）。

除法做法（当然题里不让用 hh）：先让数组所有元素相乘，然后新数组每个元素 = 原数组所有元素乘积/原数组该位置上的元素。这里有一种情况需要考虑：原数组中某个元素 = 0 的情况，比如原数组是 [-1,1,0,-3,3]，那么计算新数组 index = 2 位置上的元素就不能用原数组所有元素相乘再/0 计算了，乘的时候就要跳过当前位置元素 = 0 的情况。

空间复杂度 O(1) 做法：其实就是再普通做法左右列表乘积的基础上稍做改动。我们可以发现，第 i 个元素的左侧列表元素乘积 = 第 i-1 个元素的左侧列表乘积 =*第 i-1 个元素。这样我们就不需要存储每个元素的左侧列表右侧列表了，可以直接用一个 L 变量一个 R 变量迭代获取所有元素的左侧和右侧列表乘积。

思路不难，不过还是要自己能想到才行。

```java
class Solution {
    public int[] productExceptSelf(int[] nums) {
        // left
        int[] ans = new int[nums.length];
        ans[0] = 1;
        for (int i = 1; i < nums.length; i++) {
            ans[i] = ans[i - 1] * nums[i - 1];
        }

        // right
        int R = 1;
        for (int i = nums.length - 2; i >= 0; i--) {
            R *= nums[i + 1];
            ans[i] *= R;
        }

        return ans;
    }
}
```

## 矩阵

### 73. 矩阵置零

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506172043822.png" alt=" " style="zoom:80%;" />

原地置零的方法是：

1. 首先记录首行首列是否存在 0。
2. 对于非首行非首列的每个元素，如果其值 = 0，则令该行行首和该列列首元素 = 0，用于标识“这一行/列应当被归零”。
3. 遍历非首行非首列的所有元素，如果发现该元素所在的行首或列首元素 = 0，说明这个元素所在的行和列应当被全部归零，则令该元素 = 0.
4. 最后再根据之前记录的首行或首列是否存在 0 元素的情况，把首行或首列置零。

```java
class Solution {
    public void setZeroes(int[][] matrix) {
        if (matrix.length == 0)
            return;
        int row = matrix.length;
        int col = matrix[0].length;
        int is_first_row_contains_zero = 0, is_first_column_contains_zero = 0;
        for (int i = 0; i < row; i++) {
            if (matrix[i][0] == 0) {
                is_first_column_contains_zero = 1;
                break;
            }
        }
        for (int i = 0; i < col; i++) {
            if (matrix[0][i] == 0) {
                is_first_row_contains_zero = 1;
                break;
            }
        }
        for (int i = 1; i < row; i++) {
            for (int j = 1; j < col; j++) {
                if (matrix[i][j] == 0) {
                    matrix[i][0] = 0;
                    matrix[0][j] = 0;
                }
            }
        }
        for (int i = 1; i < row; i++) {
            for (int j = 1; j < col; j++) {
                if (matrix[i][0] == 0 || matrix[0][j] == 0)
                    matrix[i][j] = 0;
            }
        }
        if (is_first_column_contains_zero == 1) {
            for (int i = 0; i < row; i++)
                matrix[i][0] = 0;
        }
        if (is_first_row_contains_zero == 1) {
            for (int i = 0; i < col; i++)
                matrix[0][i] = 0;
        }
    }
}
```

### 54. 螺旋矩阵

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506172042368.png" alt=" " style="zoom: 80%;" />

我自己想的方法：矩阵元素全都 >=-100 对吧，所以我给遍历过的矩阵元素设置值 =-101.

每次遍历指针朝着右/下/左/上的一个方向遍历，如果数组越界，或者碰到 =-101 的元素（说明碰到之前遍历过的元素了）则切换到下一个方向。

```java
class Solution {
    public List<Integer> spiralOrder(int[][] matrix) {
        if (matrix.length == 0)
            return null;
        int[] directions = new int[] { 0, 1, 1, 0, 0, -1, -1, 0 };
        int current_direction = 0, p_x = 0, p_y = 0;
        int num = matrix.length * matrix[0].length;
        List<Integer> list = new ArrayList<>();
        for (int i = 0; i < num; i++) {
            list.add(matrix[p_x][p_y]);
            matrix[p_x][p_y] = -101;
            if (p_x + directions[current_direction] < 0 || p_x + directions[current_direction] >= matrix.length
                    || p_y + directions[current_direction + 1] < 0
                    || p_y + directions[current_direction + 1] >= matrix[0].length
                    || matrix[p_x + directions[current_direction]][p_y + directions[current_direction + 1]] <= -101) {
                current_direction = (current_direction + 2) % 8;
            }
            p_x += directions[current_direction];
            p_y += directions[current_direction + 1];
        }
        return list;
    }
}
```

方向向量的方法是之前写吃豆人的时候悟出来的哈哈，用 [0,1] [1,0] [0,-1] [-1,0] 分别代表四个方向。

### 48. 旋转图像

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506172042384.png" alt=" " style="zoom:80%;" />

这道题我还记得上次的做法：先对角翻转，再水平翻转。

```java
class Solution {
    public void rotate(int[][] matrix) {
        for (int i = 0; i < matrix.length; i++) {
            for (int j = i + 1; j < matrix.length; j++)
                swap(matrix, i, j, j, i);
        }
        for (int i = 0; i < matrix.length; i++) {
            for (int j = 0; j < matrix.length / 2; j++) {
                swap(matrix, i, j, i, matrix.length - 1 - j);
            }
        }
    }

    public void swap(int[][] matrix, int i, int j, int a, int b) {
        int temp = matrix[i][j];
        matrix[i][j] = matrix[a][b];
        matrix[a][b] = temp;
    }
}
```

### 240. 搜索二维矩阵 II

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506172043296.png" alt=" " style="zoom:67%;" />

答案中比较优秀的一种做法是 z 字形遍历。

首先，我们想象一下：如果一个点作为一个矩阵最右下角的点，那么所有比他小的点都在他左上方的矩阵里。其他部分的元素，右侧也好下侧也好，都只会比他大。

我们假设这个点一开始位于整个矩阵的右上角。那么它作为子矩阵的右下角元素，涵盖的比他小的子矩阵就只有第一行。

接着，如果我们要找的目标元素比这个元素大，说明目标元素不在这个元素的左上角子矩阵里，我们令这个元素往下移动（让 [x, y+1] 位置的元素作为新的子矩阵右下角元素），一点点向下移动扩大范围。

如果我们要找的目标元素比这个元素小，说明这个元素在子矩阵中，我们令这个元素向左移动（让 [x-1, y] 位置的元素作为新的子矩阵右下角元素），缩小子矩阵范围。

这样 z 字形遍历，可以有效锁定目标元素的位置。反之，如果遍历元素越界了，说明矩阵中不存在目标元素，return false.

```java
class Solution {
    public boolean searchMatrix(int[][] matrix, int target) {
        int x = 0, y = matrix[0].length - 1;
        while (x < matrix.length && y >= 0) {
            if (matrix[x][y] == target)
                return true;
            else if (matrix[x][y] < target)
                x++;
            else
                y--;
        }
        return false;
    }
}
```

## 链表

很久没做了，再上手发现链表部分还是很熟的，居然所有的都做出来了，也可能是简单吧。

链表部分我觉得的一些重点：

1. 注意添加一个 `dummyNode` 的做法。有的时候头节点可能不太方便，需要额外考虑，这个时候可以创建一个 `dummyNode` 指向头节点，作为一个空头节点。这样写代码的时候不用特殊处理头节点情况会比较方便。不过最后返回链表的时候，不要忘记返回 `dummyNode.next`！
2. 链表中删除元素的方法要比较熟悉。保存 `prev curr next` 三个节点，如果要删除 `curr` 的话，让 `prev.next = next`。不保存前后节点，单链表实现就比较麻烦。
3. 快慢指针在找链表中的一些结构的时候会非常有用，比如环，链表相交，通过快慢指针的不同起点或者速度让它们在我们想要的位置相遇。
4. 链表反转也要掌握，也是保存 `prev curr next` 三个节点.

### 160. 相交链表

判断两链表是否相交。

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506172047742.png" alt=" " style="zoom:67%;" />

方便的做法自然是弄一个 HashSet 存储所有节点，然后同时遍历两个链表，如果发现有一个节点无法存入 HashSet 已经存在，这个节点就是相交的起始点。不过这个方法时空复杂度都不是特别好。

第二种做法就是双指针，感觉链表很多题都可以双指针解决。第一次我们遍历两个链表，并记下两个链表到达结尾的总长度，比如上图中一个是 5 一个是 6.那么我们可以判断出，B 链表长于 A 链表的部分肯定不会是相交的起始点出现的地方。因为两个链表相交之后的部分长度相等。所以我们可以再次遍历两个链表，B 链表从 b2 开始遍历，A 链表从 a1 开始遍历，两个指针每次同时往后挪动一个单位后判断是否相等。也就是说把长链表过长的那部分去掉了，变成一个和短链表等长的链表，然后两个链表比对到底是在什么地方最先出现的相交点。

时间复杂度 O(n) 空间复杂度 O(1)。

```java
public class Solution {
    public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
        int lenA = 0, lenB = 0;
        ListNode tempA = headA, tempB = headB;
        while(tempA != null){
            lenA++;
            tempA = tempA.next;
        }
        while(tempB != null){
            lenB++;
            tempB = tempB.next;
        }

        ListNode longerListNode = lenA > lenB? headA : headB;
        ListNode shorterListNode = lenA > lenB? headB : headA;

        for(int i =0;i<Math.abs(lenA-lenB);i++)longerListNode = longerListNode.next;
        while(longerListNode != null){
            if(longerListNode == shorterListNode)return longerListNode;
            longerListNode = longerListNode.next;
            shorterListNode = shorterListNode.next;
        }
        return null;
    }
}
```

### 206. 反转链表

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506172052587.png" alt=" " style="zoom:67%;" />

不限制空间复杂度的前提下，这道题还是非常好做的。我们一边读取原链表，一边创造一个新链表，不过新链表不是从头到尾逐渐添加元素的，而是每次在头的位置添加一个新的头元素，从尾到头添加。时空复杂度均为 O(N)。

如果要求空间复杂度 O(1) 就是在原来的链表基础上反转了，从头遍历地柜实现。主要难点就是需要 `prev curr next` 三个指针，每次遍历的时候 `curr->next = prev, curr = next, prev = curr`。不用额外变量保存前后节点就会更复杂。

```java
class Solution {
    public ListNode reverseList(ListNode head) {
        if(head == null || head.next == null)return head;
        ListNode pre = head, cur = head.next, tmp = head.next;
        pre.next = null;
        while(cur != null){
            tmp = cur.next;
            cur.next = pre;
            pre = cur;
            cur = tmp;
        }
        return pre;
    }
}
```

### 234. 回文链表

判断一个链表是否回文。

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506172102238.png" alt=" " style="zoom:67%;" />

不限制空间复杂度的话也很简单，可以创建个集合记录一下顺序，然后再次遍历看集合出栈顺序和原链表遍历顺序是否一致。都是 O(N)。

限制空间复杂度常数的话，可以用快慢指针实现。慢指针移动速度是快指针的二分之一，这样快指针走完了慢指针才刚走到链表中间。然后再创建一个慢指针2号从头遍历，慢指针1号反转后半部分链表，反转完成后慢指针1号和2号同步遍历看前后两部分链表是否构成回文。

用到了之前反转链表的思想。

```java

class Solution {
    public boolean isPalindrome(ListNode head) {
        ListNode fastPtr = head, slowPtr = head, prev =new ListNode(-1, head);
        while (fastPtr != null && fastPtr.next != null) {
            fastPtr = fastPtr.next.next;
            slowPtr = slowPtr.next;
            prev = prev.next;
        }

        ListNode cur = slowPtr, tmp = slowPtr.next;
        // 截断前半部分和后半部分数组。不然的话就是要从 head 比较到 tail 了，我这样截断之后就从 head 比较到 mid 就行。
        // 不过这道题最大长度也就是9，没感觉出来太多区别。
        prev.next = null;
        prev = null;

        while (cur != null) {
            tmp = cur.next;
            cur.next = prev;
            prev = cur;
            cur = tmp;
        }
        while (head != null && prev != null) {
            if (head.val != prev.val)
                return false;
            head = head.next;
            prev = prev.next;
        }
        return true;
    }
}
```

慢指针1号在快指针到达结尾的时候刚好会到达后半部分回文子串的开始。

### 141. 环形链表

判断链表中是否有环。

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506172117884.png" alt=" " style="zoom:67%;" />

非常经典的题了，快慢指针，快指针走得比慢指针快一倍，看会不会相遇。如果会，说明有环。

```java
public class Solution {
    public boolean hasCycle(ListNode head) {
        ListNode slowPtr = head, fastPtr = head;
        while (fastPtr != null && fastPtr.next != null) {
            fastPtr = fastPtr.next.next;
            slowPtr = slowPtr.next;
            if (fastPtr == slowPtr) {
                return true;
            }
        }
        return false;
    }
}
```

### 142. 环形链表 II

简单说就是不仅要判断是否为环形链表，还要找到入环点。

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506172119992.png" alt=" " style="zoom:67%;" />

不考虑空间复杂度的话，还是弄个 Set 存点，第一个存不进的重复元素就是入环点了呗。

考虑空间复杂度的话，这道题会有一个比较有意思的解法。

官方题解图示如下：

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506172121642.png" alt=" " style="zoom: 33%;" />

首先我们还是快慢指针遍历。两者相遇的时候，慢指针走了 a+b，快指针走了 a+b+n(b+c) （多走了 n 圈）。

我们要求 a：

$$
2x = a+b+n(b+c)\\
x = a+b\\
a = (n-1)b+nc
$$

也就是说 a 部分的路程相当于 n 圈的路程 -c。

那么我们就让一个指针从紫色的快慢指针相遇点开始转圈，一个指针从原点出发。那么两者就一定会在入环点相遇。原点出发的点走了距离 a 到达了入环点，绕圈的点走了 n 圈+c，不过绕圈点本身起点就是距离入环点 c 距离的位置，所以这个时候也会在入环点。

这样我们两次遍历就找到入环点了，时间复杂度 O(N)。

```java
public class Solution {
    public ListNode detectCycle(ListNode head) {
        // a: 环前的元素
        // loop: 环长度
        // x: slowPtr 走过的距离
        // b: slowPtr 在环中走过的距离
        // x = a + b
        // 2*x = a + n*(b+c) + b
        // loop = b+c
        // x = n*(b+c) 
        // a = (n-1)*b + n*c = (n-1) * (b+c) + c = (n-1)*b+n*c
        ListNode slowPtr = head, fastPtr = head;
        while(fastPtr != null && fastPtr.next != null){
            fastPtr = fastPtr.next.next;
            slowPtr = slowPtr.next;
            if(slowPtr == fastPtr)break;
        }
        if(fastPtr == null || fastPtr.next == null)return null;
        ListNode slowPtr1 = head;
        while(slowPtr1 != slowPtr){slowPtr = slowPtr.next; slowPtr1 = slowPtr1.next;}
        return slowPtr1;
    }
}
```

### 21. 合并两个有序链表

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506172128919.png" alt=" " style="zoom:67%;" />

没啥难度，就双指针遍历嘛，创建一个新链表作为输出结果，或者直接修改原来链表元素。

这个题主要是给后面归并排序做铺垫呢。

```java
class Solution {
    public ListNode mergeTwoLists(ListNode list1, ListNode list2) {
        if (list1 == null && list2 == null)
            return null;
        ListNode newList = new ListNode();
        ListNode pre = newList;
        while (list1 != null || list2 != null) {
            if (list1 == null || (list2 != null && list2.val <= list1.val)) {
                newList.next = list2;
                list2 = list2.next;
            } else {
                newList.next = list1;
                list1 = list1.next;
            }
            newList = newList.next;
            newList.next = null;
        }
        return pre.next;
    }
}
```

### 2. 两数相加

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506172133495.png" alt=" " style="zoom:67%;" />

就是类似一个进位器机制，有一定计组基础的同学应该很容易做，就是用一个 carry 变量存储之前两次链表求和的进位。最后遍历完两个链表了别忘了再检查一下 carry 是否为空。

空间复杂度上呢，要么就自己新建个链表，要么就麻烦点挑那个更长的链表在其基础之上修改。

```java
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        // if(l1 == null && l2 == null)return null;
        ListNode preListNode = new ListNode();
        ListNode tmp = preListNode;
        int carry = 0;
        while (l1 != null || l2 != null) {
            if (l1 == null) {
                tmp.next = new ListNode((carry + l2.val) % 10);
                carry = (carry + l2.val) / 10;
                l2 = l2.next;
            } else if (l2 == null) {
                tmp.next = new ListNode((carry + l1.val) % 10);
                carry = (carry + l1.val) / 10;
                l1 = l1.next;
            } else {

                tmp.next = new ListNode((carry + l1.val + l2.val) % 10);
                carry = (carry + l1.val + l2.val) / 10;

                l1 = l1.next;

                l2 = l2.next;
            }
            tmp = tmp.next;
        }
        if (carry != 0) {
            tmp.next = new ListNode(carry);
        }
        return preListNode.next;
    }
}
```

### 19. 删除链表的第 n 个节点

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506172136336.png" alt=" " style="zoom:67%;" />

这个仍然没啥难度，主要是给后面的难题做铺垫呢。主要还是要保存 prev 和 next 节点进行拼接。

```java
class Solution {
    public ListNode removeNthFromEnd(ListNode head, int n) {
        ListNode slowPtr = new ListNode(0, head), fastPtr = head;
        for (int i = 0; i < n; i++) {
            if (fastPtr == null)
                return head;
            fastPtr = fastPtr.next;
        }
        while (fastPtr != null) {
            slowPtr = slowPtr.next;
            fastPtr = fastPtr.next;
        }
        if (slowPtr.next == head) {
            return slowPtr.next.next;
        } else if (slowPtr.next != null)
            slowPtr.next = slowPtr.next.next;

        return head;
    }
}
```

我这里命名不好，其实应该 prev 和 curr 的。别在意。

### 24. 两两交换链表节点

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506172137721.png" alt=" " style="zoom:67%;" />

这个其实也不难，我直接递归实现了，空间复杂度有点麻烦，迭代可能会好些。就是2个2个元素交换，然后处理其之前和之后的元素的指向拼接。

```java
class Solution {
    public ListNode swapPairs(ListNode head) {
        if(head == null || head.next == null)return head;
        ListNode tmpHead = head.next;
        head.next = head.next.next;
        tmpHead.next = head;
        if(head.next != null){
            head.next = swapPairs(head.next);
        }
        return tmpHead;
    }
}
```

### 25. K 个一组翻转链表

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506172138184.png" alt=" " style="zoom:67%;" />

这个是在之前那道题上的进阶。两两交换还很简单，多了就得用到反转链表的方法了。

反转链表，还有前后链表段处理这些前面都有出现过，没什么非常新奇的东西，就是写起来麻烦，不要犯错就行。

```java
class Solution {
    public ListNode reverseKGroup(ListNode head, int k) {
        ListNode dummyNode = new ListNode(-1, head);
        ListNode prev = dummyNode;
        ListNode currNode = head;

        while (currNode != null) {
            for (int i = 0; i < k - 1 && currNode != null; i++) {
                currNode = currNode.next;
            }
            if (currNode == null)
                break;
            ListNode next = currNode.next;
            currNode.next = null;
            prev.next = reverseSubGroup(prev.next);
            while (prev.next != null)
                prev = prev.next;
            prev.next = next;
            currNode = next;
        }
        return dummyNode.next;
    }

    public ListNode reverseSubGroup(ListNode head) {
        if (head == null || head.next == null)
            return head;
        ListNode prev = null;
        ListNode currNode = head;
        while (currNode != null) {
            ListNode tmpNode = currNode.next;
            currNode.next = prev;
            prev = currNode;
            currNode = tmpNode;
        }
        return prev;
    }
}
```

### 138. 随机链表的复制

从这道题开始有点意思。

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506172141142.png" alt=" " style="zoom: 67%;" />

深拷贝一个链表比较简单，但是这个链表中还有 random，这个 random 我们不一定知道是指向链表中哪一个元素。

我想到的比较蠢的实现方法就是遍历链表去记录 random 指向的链表元素的索引，然后我在我自己深拷贝的数组里也找到对应索引的位置，用 random 指向这些位置。时间复杂度炸完了，O(n2) 了吧。

不过其实最简单的方法是哈希表映射。我用 key value 分别对应原链表的某一个节点和对应的新链表的节点。然后我发现有一个 B 节点 random 指向 A 节点，我去 map 中看看 A 节点对应的新链表中是什么节点， 发现是 A' 节点。那我新链表中的 B'.random 就指向 A' 节点即可。

```java
class Solution {
    HashMap<Node, Node> map = new HashMap<>();
    public Node copyRandomList(Node head) {
        if(head == null)return null;
        Node tmpPtr = head;
        Node resPtr = new Node(1);
        Node curPtr = resPtr;
        if(map.get(tmpPtr)==null){
            curPtr.next = new Node(tmpPtr.val);
            curPtr = curPtr.next;
            map.put(tmpPtr, curPtr);
            curPtr.next = copyRandomList(tmpPtr.next);
            
            curPtr.random = copyRandomList(tmpPtr.random);
            return resPtr.next;
        }
        else {
            return map.get(tmpPtr);
        }

    }
}
```

### 148. 排序链表

题目挺简单的。

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506172146676.png" alt=" " style="zoom:67%;" />

实现方法有很多，比如插入排序，每次插入一个节点；`Collections.sort` 帮我排序；不过题目中有提到：建议尝试 O(nlogn) 的时间复杂度，常数级空间复杂度的做法。这个数字比较眼熟，很明显是要挑战链表的归并排序。

归并排序简单说就是把链表分成两段，分别排序，然后这两段再合并起来排序。比如链表总长度16，我们自底向上归并排序，首先是16个一元元素（不用排序）。然后两两合并，变为8个2元元素，每个都要遍历2次（要合并的两个元素各访问一次）也就是遍历16次。然后两两合并，4个4元数组，遍历16次。然后合并。2个8元数组，遍历16次。最后一个16元素数组不用遍历。整个过程中需要拆分合并的次数近似等于 Log2n，所以时间复杂度总的是 O(nlogn)。

具体实现……你要说有啥难点，其实也没啥难点，都是之前的知识，比如截断后两两排序，递归排序返回头结点什么的。就是小问题一堆，写起来也麻烦

```java
class Solution {
    public ListNode sortList(ListNode head) {
        int len = 0;
        ListNode tmpHead = head;
        ListNode dummyHead = new ListNode(-1, head);
        while (tmpHead != null) {
            tmpHead = tmpHead.next;
            len++;
        }
        for (int subLength = 1; subLength < len; subLength <<= 1) {
            // prev: 之前已经排好序的链表
            // head1 head2: 本轮循环准备归并排序的两部分链表
            // next: 之后的链表

            // 找到 prev
            ListNode prev = dummyHead, cur = dummyHead.next;

            while (cur != null) {
                // 找到 head1 head2
                ListNode head1 = cur;
                for (int i = 1; i < subLength && cur.next != null; i++)
                    cur = cur.next;
                ListNode head2 = cur.next;
                cur.next = null;

                // 找到 next
                cur = head2;
                for (int i = 1; i < subLength && cur != null && cur.next != null; i++)
                    cur = cur.next;
                ListNode next = null;
                if (cur != null) {
                    next = cur.next;
                    // 并且截断归并排序的第二个链表
                    cur.next = null;
                }

                ListNode merged = merge(head1, head2);
                prev.next = merged;
                while (prev.next != null)
                    prev = prev.next;
                cur = next;
            }
        }
        return dummyHead.next;

    }

    public ListNode merge(ListNode node1, ListNode node2) {
        ListNode dummyNode = new ListNode(-1);
        ListNode currNode = dummyNode;
        while (node1 != null && node2 != null) {
            if (node1.val < node2.val) {
                currNode.next = node1;
                node1 = node1.next;
            } else {
                currNode.next = node2;
                node2 = node2.next;
            }
            currNode = currNode.next;

        }
        if (node1 != null)
            currNode.next = node1;
        else if (node2 != null)
            currNode.next = node2;
        return dummyNode.next;
    }
}
```

我这个主要瓶颈在于合并之后 prev 要疯狂遍历找到这部分合并链表的结尾作为新的 prev ，除此之外符合归并排序。用递归可能会好些。

### 23. 合并 K 个升序列表

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506172207459.png" alt=" " style="zoom:67%;" />

其实这个也是考验递归。

先写一个两链表合并的方法，然后每次放两个链表进去排序。如果是结果返回值链表每次拿一个链表集合中的元素到自己这里“插入排序”，那么时间复杂度会很高，还是两两归并效率会高很多。

```java
class Solution {
    public ListNode mergeKLists(ListNode[] lists) {
        return mergeTwoSets(lists, 0, lists.length-1);
    }

    public ListNode mergeTwoSets(ListNode[] lists, int l, int r){
        if(l == r)return lists[l];
        else if(l > r)return null;
        int mid = (l+r)>>1;
        return mergeTwoLists(mergeTwoSets(lists, l, mid),mergeTwoSets(lists, mid+1, r));
    }

    public ListNode mergeTwoLists(ListNode head1, ListNode head2){
        ListNode dummyHead = new ListNode(-1, null);
        ListNode curNode = dummyHead;
        while(head1!=null && head2 != null){
            if(head1.val<=head2.val){
                curNode.next = head1;
                head1 = head1.next;
            }else {
                curNode.next = head2;
                head2 = head2.next;
            }
            curNode = curNode.next;
        }
        if(head1!=null){
            curNode.next = head1;
        }
        else if(head2!=null){
            curNode.next = head2;
        }
        else if(curNode!=null)curNode.next = null;
        return dummyHead.next;
    }
}
```

### 146. LRU 缓存

看似中等，实则真正的困难！

让我们实现一个类似操作系统中 LRU（least recently used）缓存的算法。比如缓存大小设置为10，现在存入了10个缓存了，然后又要存入一个新缓存，就要把最久没用过的那个淘汰掉。

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506172210112.png" alt=" " style="zoom:67%;" />

我们可以采取类似 LinkedHashMap 的做法，既保存了顺序又可以依靠哈希表快速访问。

当用户 put 新添加一个元素进缓存的时候，我们就将其存入 HashMap 和双链表，双链表用于记载其顺序位置，头是最新存入的或者最近访问的，尾是最久没用的，HashMap 用于根据这个键值对的键快速找到对应对象的引用，用双链表实现就是头尾可以快速访问，不然我们要删除尾部元素的话每次都要 O(n) 遍历到结尾。添加完元素之后判断一下当前容量是否超出最大容量，如果超出则删除尾部元素并删除其在 HashMap 中的映射。

用户 get 一个元素的时候，先去 HashMap 里查找看有没有，快速定位，如果有则把这个元素在链表中删掉，放到头节点位置并返回 value 值给用户。如果没有直接返回 -1。*缓存穿透问题无法避免吧，哈哈。必要的话可以缓存空对象，不过这里做题就不用考虑了。*

用户 put 一个已经存在的键值对的时候，更新该元素，然后和 get 一样，将其从链表中间删去，放到头节点。

如下未完整代码。说实话，这个也没有什么高深的算法，都是之前我们练习过的，但是重点还是两点：1. 思想，要能够想到这种用法。2. 细节，我估计真面试遇到了我得写上30分钟都不一定纠正得了所有错误。所以还是要常练。

```java
class LRUCache {

    class DoubleLinkedNode {
        int key;
        int value;
        DoubleLinkedNode prev;
        DoubleLinkedNode next;

        public DoubleLinkedNode() {
        }

        public DoubleLinkedNode(int _key, int _value) {
            key = _key;
            value = _value;
        }
    }

    private Map<Integer, DoubleLinkedNode> cache = new HashMap<>();
    private int capacity;
    private int size;
    private DoubleLinkedNode head,tail;

    public LRUCache(int capacity) {
        this.size = 0;
        this.capacity = capacity;

        head = new DoubleLinkedNode();
        tail = new DoubleLinkedNode();
        head.next = tail;
        tail.prev = head;
    }

    public int get(int key) {
        DoubleLinkedNode getNode = cache.get(key);
        if(getNode == null)return -1;
        int res = getNode.value;
        deleteNodeInCache(getNode);
        updateHeadNodeInCache(getNode);
        return res;
    }

    public void put(int key, int value) {
        
        DoubleLinkedNode getNode = cache.get(key);
        if(getNode == null){
            DoubleLinkedNode newHeadNode = new DoubleLinkedNode(key, value);
            cache.put(key, newHeadNode);
            size++;
            if(size>capacity){
                int lastKey = tail.prev.key;
                deleteLastNodeInCache();
                cache.remove(lastKey);
                size--;
            }
            
            updateHeadNodeInCache(newHeadNode);
        }
        else {
            getNode.value = value;
            deleteNodeInCache(getNode);
            updateHeadNodeInCache(getNode);

        }
    }

    private void deleteNodeInCache(DoubleLinkedNode node){
        DoubleLinkedNode prev = node.prev;
        DoubleLinkedNode next = node.next;
        prev.next = next;
        next.prev = prev;
    }

    private void updateHeadNodeInCache(DoubleLinkedNode node){
        DoubleLinkedNode oldFirst = head.next;
        oldFirst.prev = node;
        node.prev = head;
        node.next = oldFirst;
        head.next = node;
    }
    private void deleteLastNodeInCache(){
        deleteNodeInCache(tail.prev);
    }
}

/**
 * Your LRUCache object will be instantiated and called as such:
 * LRUCache obj = new LRUCache(capacity);
 * int param_1 = obj.get(key);
 * obj.put(key,value);
 */
```
