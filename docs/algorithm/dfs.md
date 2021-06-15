# 深度优先搜索

## 0、简介

深度优先搜索算法（英语：Depth-First-Search，DFS）是一种用于遍历或搜索树或图的算法。
沿着树的深度遍历树的节点，尽可能深的搜索树的分支。当节点v的所在边都己被探寻过，搜索将回溯到发现节点v的那条边的起始节点。这一过程一直进行到已发现从源节点可达的所有节点为止。如果还存在未被发现的节点，则选择其中一个作为源节点并重复以上过程，整个进程反复进行直到所有节点都被访问为止。属于盲目搜索。

> 一般考察的都是树的dfs或者使用回溯算法解决对矩阵的搜索问题等。

## 1、排列组合问题

> 全排列问题：1.1 1.2 

需要借助辅助数据`used` , `dfs`内部遍历整个数组，所以递归参数一般不需要`index`进行辅助，根据重复考虑是否剪枝。

### 1.1、**剑指offer 38 字符串的排列**

***题目介绍***

> 输入一个字符串，打印出该字符串中字符的所有排列。
>
> 你可以以任意顺序返回这个字符串数组，但里面不能有重复元素。

***分析：***

本质属于**全排列问题**。本题字符串中的字符可能存在重复的（比如"`aab`")，所以需要做去重处理。一种方法是使用`HashSet`，另一种则是先排序再在`dfs`过程中使用剪枝。下面的代码则是实现了第二种方法。

- 辅助数组`used/visited[]`

- 递归参数

  - 字符数组
  - 暂存某轮搜索路径的`StringBuilder`

- 递归终止条件

  搜索路径的长度等于字符串长度时终止（注意剪枝）

- `dfs`主体

  - for循环从0开始，到字符数组结尾结束。

  - 根据`used`判断是否搜索过
  - 剪枝
    - 判断当前字符等于上一位字符，且上一位在本轮未访问
    - 直接continue

***代码如下：***

```java
class Solution {
    List<String> res = new ArrayList<>();
    boolean[] used; // 记录某一位字符是否在本次搜索中已经被使用
    public String[] permutation(String s) {
        int len = s.length();
        used = new boolean[len];
        char[] c = s.toCharArray();
        // 先将字符串转为字符数组，然后进行排序
        Arrays.sort(c);
        StringBuilder sb = new StringBuilder();
        dfs(c, sb);
        return res.toArray(new String[res.size()]);
    }

    private void dfs(char[] c, StringBuilder sb) {
        if(c.length == sb.length()) {
            res.add(sb.toString());
            return;
        }

        for(int i = 0 ; i < c.length ;i++) {
            if(!used[i]) {
                // 剪枝
                if(i > 0 && c[i] == c[i - 1] && !used[i - 1]) continue;
                used[i] = true;
                sb.append(c[i]);
                dfs(c, sb);
                sb.deleteCharAt(sb.length() - 1);
                used[i] = false;
            }
        }
    }
}
```
### 1.2、46 全排列

***题目介绍***

> 给定一个不含重复数字的数组 `nums` ，返回其 **所有可能的全排列** 。你可以 **按任意顺序** 返回答案。

***分析：***

本质上与1.1、字符串的排列思想相同，只是本题声明了**数组元素不重复**，因此不需要剪枝。

***代码如下：***

```java
class Solution {
    List<List<Integer>> res = new ArrayList<>();
    boolean[] used;
    public List<List<Integer>> permute(int[] nums) {
        used = new boolean[nums.length];
        LinkedList<Integer> t = new LinkedList<>();
        dfs(nums, t);
        return res;
    }

    private void dfs(int[] nums, LinkedList<Integer> t) {
        if(nums.length == t.size()) {
            res.add(new LinkedList<>(t));
            return;
        }

        for(int i = 0; i < nums.length;i++ ) {
            if(!used[i]) {
                t.add(nums[i]);
                used[i] = true;
                dfs(nums, t);
                used[i] = false;
                t.removeLast();
            }
        }
    }
}
```

### 1.3、17 电话号码的字母组合

***题目介绍***

> 给定一个仅包含数字 2-9 的字符串，返回所有它能表示的字母组合。答案可以按 任意顺序 返回。
>
> 给出数字到字母的映射如下（与电话按键相同）。注意 1 不对应任何字母。

***分析***

- 辅助数组需要

- 递归参数

  - 数组 `digits[]`
  - `index`辅助，记录在数字数组的位置，保证顺序
  - `StringBuilder`暂存路径

- 终止条件：

  `index` 和 数组的长度相等

- `dfs`主体

  从头到尾遍历数字对应的字符串

***代码***

```java
class Solution {
    List<String> res = new ArrayList<>();
    String[] map = new String[] {
        "",
        "",
        "abc",
        "def",
        "ghi",
        "jkl",
        "mno",
        "pqrs",
        "tuv",
        "wxyz"
    };

    public List<String> letterCombinations(String digits) {
        if(digits.length() == 0) return res;
        StringBuilder sb = new StringBuilder();
        dfs(digits, 0, sb);
        return res;
    }

    private void dfs(String digits, int index, StringBuilder  sb) {
        // 停止
        if(index == digits.length()) {
            res.add(sb.toString());
            return;
        }
        String s = map[digits.charAt(index) - '0'];
        for(int j = 0; j < s.length();j++ ) {
            sb.append(s.charAt(j));
            dfs(digits, index + 1, sb);
            sb.deleteCharAt(sb.length() - 1);
        }

    }
}
```

### 1.4、39 组合总和

***题目介绍***

> 给定一个无重复元素的数组 candidates 和一个目标数 target ，找出 candidates 中所有可以使数字和为 target 的组合。candidates 中的数字可以无限制重复被选取。
>
> 说明：
>
> 所有数字（包括 target）都是正整数。
> 解集不能包含重复的组合。

***分析***

本题数组不存在重复元素，但是元素可以重复选取。这种情况需要讲数组排序，再在主体中进行剪枝，提前终止搜索

- 不需要辅助数组（因为可以重复选取）

- 递归参数

  - 数组
  - `index`记录在数组中的位置
    - 不能往前遍历，只能往后遍历
    - 因为可以重复选取，所以当前位置可以选，即`i = index`；
  - 目标和剩余值，每次减去选取的数组元素值

- 剪枝

  在下一次`dfs`之前，判断当前剩余值剪去当前数组元素值是否小于 0 ，若是则直接返回。

- 终止条件：

  目标剩余值为0

***代码***

```java
class Solution {
    List<List<Integer>> res = new ArrayList<>();
    public List<List<Integer>> combinationSum(int[] candidates, int target) {
        LinkedList<Integer> t = new LinkedList<>();
        // 数组排序
        Arrays.sort(candidates);
        dfs(candidates, 0, target, t);
        return res;
    }

    private void dfs(int[] candidates, int index, int target, LinkedList<Integer> t) {
        if(target == 0) {
            res.add(new LinkedList<>(t));
            return;
        }
        for(int i = index; i < candidates.length; i++) {
            // 剪枝
            if(target - candidates[i] < 0 ) return;
            t.add(candidates[i]);
            dfs(candidates,i, target - candidates[i], t );
            t.removeLast();
        }
    }
}
```

### 1.5、78 子集

***题目介绍***

> 给你一个整数数组 `nums` ，数组中的元素 **互不相同** 。返回该数组所有可能的子集（幂集）。
>
> 解集 **不能** 包含重复的子集。你可以按 **任意顺序** 返回解集。

***分析***

- 辅助数组不需要
- 递归终止条件：不需要，因为所有的都要存到结果中
- 参数
  - 数组
  - `index`记录在数组的位置
    - 只能向后遍历
    - 下一轮递归就是以下一个位置为参数
- 主体
  - `i = index`
  - `dfs(..,index + 1, ...)`

***代码如下***

```java
class Solution {
    List<List<Integer>> res = new ArrayList<>();;
    public List<List<Integer>> subsets(int[] nums) {
        LinkedList<Integer> t = new LinkedList<>();
        dfs(nums, 0 , t);
        return res;
    }

    private void dfs(int[] nums, int index, LinkedList<Integer> t) {
        res.add(new LinkedList<>(t));
        for(int i = index ; i < nums.length;i++){
            t.add(nums[i]);
            dfs(nums, i + 1, t);
            t.removeLast();
        }
    }
}
```

## 2、路径搜索问题

### 2.1、树路径

#### 2.1.1、剑指offer  34. 二叉树中和为某一值的路径

***题目介绍***

> 输入一棵二叉树和一个整数，打印出二叉树中节点值的和为输入整数的所有路径。从树的根节点开始往下一直到叶节点所经过的节点形成一条路径。

***分析***

***代码***

```java
class Solution {
    List<List<Integer>> res;
    public List<List<Integer>> pathSum(TreeNode root, int target) {
        res = new ArrayList<>();
        LinkedList<Integer> t = new LinkedList<>();
        dfs(root, target, t);
        return res;
    }

    private void dfs(TreeNode root, int target, LinkedList<Integer> t) {
        if(root == null) return;
        t.addLast(root.val);
        target -= root.val;
        // 当前节点为也节点 且target为0
        if(target == 0 && root.left == null && root.right == null) {
            res.add(new LinkedList<>(t));
        }
        dfs(root.left, target, t);
        dfs(root.right, target, t);
        t.removeLast();
    }
}
```

### 2.2、矩阵路径

#### 2.2.1、剑指offer 12 .矩阵中的路径 / 79  单词搜索

***题目介绍***

> 给定一个 m x n 二维字符网格 board 和一个字符串单词 word 。如果 word 存在于网格中，返回 true ；否则，返回 false 。
>
> 单词必须按照字母顺序，通过相邻的单元格内的字母构成，其中“相邻”单元格是那些水平相邻或垂直相邻的单元格。同一个单元格内的字母不允许被重复使用。

***分析***

***代码***
```java
class Solution {
    int m ,n;
    boolean[][] used;
    int[][] direct = {{0,1},{0,-1},{1,0},{-1,0}};
    public boolean exist(char[][] board, String word) {
        if(board == null || board.length == 0 || board[0].length == 0) 
            return false;
        m = board.length;
        n = board[0].length;
        used = new boolean[m][n];
        char[] wordChar =  word.toCharArray();
        for(int i = 0; i < m;i++)
            for(int j = 0; j < n;j++) 
                if(dfs(board,wordChar,0,i,j)) return true;
        return false;
    }

    private boolean checkEdge(int i , int j ) {
        return i >= 0 && i < m && j >= 0 && j < n;
    }

    private boolean dfs(char[][] board, char[] wordChar, int index, int i, int j) {
        if( !checkEdge(i , j ) || used[i][j] || wordChar[index] != board[i][j])
            return false;
        if(index == wordChar.length - 1) return true;
        used[i][j] = true;
        for(int k = 0; k < 4;k++) {
            if(dfs(board,wordChar, index + 1, i + direct[k][0],j + direct[k][1])) return true;
        }
        used[i][j] = false;
        return false;
    }
}
```

#### 2.2.2、200 岛屿数量

***题目介绍***

  > 给你一个由 '1'（陆地）和 '0'（水）组成的的二维网格，请你计算网格中岛屿的数量。
  >
  > 岛屿总是被水包围，并且每座岛屿只能由水平方向和/或竖直方向上相邻的陆地连接形成。
  >
  > 此外，你可以假设该网格的四条边均被水包围。

***分析***

***代码***
```java
  class Solution {
      int[][] direct = {{0,1},{0,-1},{-1,0},{1,0}};
      boolean[][] vis;
      public int numIslands(char[][] grid) {
          int res = 0;
          int m = grid.length;
          int n = grid[0].length;
          vis = new boolean[m][n];
  
          for(int i = 0; i < m ; i++) {
              for(int j = 0; j < n ; j ++) {
                  if(!vis[i][j] && grid[i][j] == '1') {
                      dfs(grid, i , j);
                      res++;
                  }
                  }
              }
          return res;
      }
  
      private void dfs(char[][] grid,int i , int j) {
          if(!check(grid, i , j) || vis[i][j]) return;
          vis[i][j] = true;
          for(int k = 0; k < 4;k++) {
              dfs(grid, i + direct[k][0], j + direct[k][1]);
          }
      }
  
      private boolean check(char[][] grid, int i , int j ) {
          int m = grid.length;
          int n = grid[0].length;
          return i >= 0 && i < m && j >= 0 && j < n && grid[i][j] == '1';
      }
  }
```

#### 2.2.3、394 字符串编码

***题目介绍***

  > 给定一个经过编码的字符串，返回它解码后的字符串。
  >
  > 编码规则为: k[encoded_string]，表示其中方括号内部的 encoded_string 正好重复 k 次。注意 k 保证为正整数。
  >
  > 你可以认为输入字符串总是有效的；输入字符串中没有额外的空格，且输入的方括号总是符合格式要求的。
  >
  > 此外，你可以认为原始数据不包含数字，所有的数字只表示重复的次数 k ，例如不会出现像 3a 或 2[4] 的输入。
  >
  > 
  >

***分析***

***代码***

```java
  class Solution {
      public String decodeString(String s) {
          return dfs(s , 0)[0];
      }
  
      private String[] dfs(String s , int i ) {
          int multi = 0;
          StringBuilder sbe = new StringBuilder();
          while(i < s.length()) {
              char c = s.charAt(i);
              if(c >= '0' && c <= '9') {
                  multi = multi * 10 + (c - '0');
              }else if(c == '[') {
                  String[] t = dfs(s, i + 1);
                  i = Integer.parseInt(t[0]);
                  while(multi > 0) {
                      multi--;
                      sbe.append(t[1]);
                  }
              }else if(c == ']') {
                  return new String[]{String.valueOf(i), sbe.toString()};
              }else sbe.append(c);
              i++;
          }
          return new String[]{sbe.toString()};
      }
  }
```

