# 剑指offer哈哈

## 03. 数组中重复的数字

### 题目描述

在一个长度为 n 的数组 nums 里的所有数字都在 0～n-1 的范围内。数组中某些数字是重复的，但不知道有几个数字重复了，也不知道每个数字重复了几次。请找出数组中任意一个重复的数字。

> ```
> 输入：
> [2, 3, 1, 0, 2, 5, 3]
> 输出：2 或 3 
> ```


### Java代码
```java
class Solution {
    public int findRepeatNumber(int[] nums) {
        for(int i = 0;i<nums.length;i++){
            int m = nums[i];
            if(m != i){
                if (nums[m] != m ){
                    swapArray(i,m,nums);
                }
                else{
                    return m;
                }
            }
        }
        return -1;
    }

    public void swapArray(int i, int m,int[] array){
        int t;
        t = array[i];
        array[i] = array[m];
        array[m] = t;
    }
}
```

### 思路：

从头到尾扫描数组nums，扫描到第i个数字值为m

1. 比较m是否等于i

   - 相等，转到第3步
   - 不等，比较m和第m个数字nums[m],即第2步

2. 比较m和第m个数字nums[m]

   - 相等，重复数字即m
   - 不等，交换第i个数字m和第m个数字nums[m]

3. 继续扫描第i+1个数字

复杂度总结:

- 时间复杂度O(n)
- 空间复杂度O(1)



## 04. 二维数组的查找

### 题目描述：

在一个 n * m 的二维数组中，每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。

> 现有矩阵 matrix 如下：
>
> [
>   [1,   4,  7, 11, 15],
>   [2,   5,  8, 12, 19],
>   [3,   6,  9, 16, 22],
>   [10, 13, 14, 17, 24],
>   [18, 21, 23, 26, 30]
> ]
> 给定 target = 5，返回 true。
>
> 给定 target = 20，返回 false。
>

### Java代码

```java
class Solution {
    public boolean findNumberIn2DArray(int[][] matrix, int target) {
   		if (matrix == null || matrix.length == 0 || matrix[0].length == 0) {
            return false;
        }
        int rowlen = matrix.length - 1;
        int collen = matrix[0].length - 1;
        int row = 0;
        int col = collen;
        
        while(row <= rowlen && col >=0){
            int rc = matrix[row][col];
            if(rc == target){
                return true;
            }
            else if(target < rc){
                col --;
            }
            else{
                row ++;
            }
        }
        return false;
   
    }
}
```

### 思路

右上角寻找法



## 05. 替换空格

### 题目描述

请实现一个函数，把字符串 `s` 中的每个空格替换成"%20"。

> **示例 1：**
>
> ```
> 输入：s = "We are happy."
> 输出："We%20are%20happy."
> ```

### Java代码

```java
class Solution {
    public String replaceSpace(String s) {
        int count = 0;
        for(int i=0;i<s.length();i++){
            if(s.charAt(i)==' '){
                count++;
            }
        }
        int oldlen = s.length() -1;
        int newlen = oldlen + 2*count + 1;
        char[] array = new char[newlen];
        int j = 0;
        for(int i=0;i<s.length();i++){
            char c = s.charAt(i);
            if(c == ' '){
                array[j++] = '%';
                array[j++] = '2';
                array[j++] = '0';
            }
            else{
                array[j++] = c;
            }
        }

        String sn = new String(array,0,j);

        return sn;
    }
}
```

