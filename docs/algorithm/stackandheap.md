# 堆栈相关算法题

## 一、栈

### 1、栈本身

#### 1.1、剑指 Offer 09. 用两个栈实现队列

***题目介绍***

> 用两个栈实现一个队列。队列的声明如下，请实现它的两个函数 `appendTail` 和 `deleteHead` ，分别完成在队列尾部插入整数和在队列头部删除整数的功能。(若队列中没有元素，`deleteHead` 操作返回 -1 )

***分析***

这题比较简单，使用两个栈实现一个队列，只需要用栈1保存入队顺序，出队时再将栈1的元素一一弹出并压入栈2，最后弹出栈2的栈顶元素即可。

***代码***

```java
class CQueue {
    Stack<Integer> stk1;
    Stack<Integer> stk2;
    
    public CQueue() {
        stk1 = new Stack<>();
        stk2 = new Stack<>();
    }
    
    public void appendTail(int value) {
        // stk1负责入队 队列先进先出
        // stk2负责出队
        while(!stk2.isEmpty()) stk1.push(stk2.pop());
        stk1.push(value);
    }
    
    public int deleteHead() {
        while(!stk1.isEmpty()) stk2.push(stk1.pop());
        if(stk2.isEmpty()) return - 1;
        else return stk2.pop();
    }
}

/**
 * Your CQueue object will be instantiated and called as such:
 * CQueue obj = new CQueue();
 * obj.appendTail(value);
 * int param_2 = obj.deleteHead();
 */
```

#### 1.2、剑指 Offer 30. 包含min函数的栈

***题目介绍***

> 定义栈的数据结构，请在该类型中实现一个能够得到栈的最小元素的 min 函数在该栈中，调用 min、push 及 pop 的时间复杂度都是 O(1)。

***分析***

使用辅助栈存储最小值即可，出栈时判断该元素和辅助栈的栈顶元素**是否相等**（包装类要使用equals()进行判断），若是则将辅助栈栈顶元素弹出；入栈时，判断压入元素是否**小于等于**辅助栈栈顶元素，若是则将该元素也压入辅助栈。

***代码***

```java
class MinStack {
    Stack<Integer> s1;
    Stack<Integer> s2;

    /** initialize your data structure here. */
    public MinStack() {
        s1 = new Stack<>();
        s2 = new Stack<>();
    }
    
    public void push(int x) {
        s1.push(x);
        if(s2.isEmpty() || x <= s2.peek() ) s2.push(x);
    }
    
    public void pop() {
        if(s1.pop().equals(s2.peek())) s2.pop(); 
    }
    
    public int top() {
        return s1.size() == 0 ? -1 : s1.peek();
    }
    
    public int min() {
        return s2.size() == 0 ? -1 : s2.peek();
    }
}

/**
 * Your MinStack object will be instantiated and called as such:
 * MinStack obj = new MinStack();
 * obj.push(x);
 * obj.pop();
 * int param_3 = obj.top();
 * int param_4 = obj.min();
 */
```

#### 1.3、剑指 Offer 59 - II. 队列的最大值

***题目介绍***

> 请定义一个队列并实现函数 `max_value` 得到队列里的最大值，要求函数`max_value`、`push_back` 和 `pop_front` 的**均摊**时间复杂度都是O(1)。若队列为空，`pop_front` 和 `max_value` 需要返回 -1

***分析***

本题可以借助一个**单调双向队列dq**，主队列为q。

- q入队时，循环判断dq不为空且入队元素大于dq的首元素，移除dq首元素， 最后将该元素入dq队列；
- q出队时，判断出队元素和**dq队尾元素**是否相等，若是则将dq队尾元素出队。
- 所以，最大元素就是当前**dq的队首元素**

***代码***

```java
class MaxQueue {
    LinkedList<Integer> q;
    LinkedList<Integer> dq;

    public MaxQueue() {
        q = new LinkedList<>();
        dq = new LinkedList<>();
    }
    
    public int max_value() {
        return dq.isEmpty() ? -1 : dq.getFirst();
    }
    
    public void push_back(int value) {  
        q.add(value);
        while(!dq.isEmpty() && value > dq.getLast()) dq.removeLast();
        dq.addLast(value);
    }
    
    public int pop_front() {
        if(q.isEmpty()) return -1;
        int ret = q.removeFirst();
        if(ret == dq.getFirst()) dq.removeFirst();
        return ret;
    }
}
```

### 2、栈的应用

#### 2.1、20. 有效的括号

***题目介绍***

> 给定一个只包括 '('，')'，'{'，'}'，'['，']' 的字符串 s ，判断字符串是否有效。
>
> 有效字符串需满足：
>
> 左括号必须用相同类型的右括号闭合。
> 左括号必须以正确的顺序闭合。

***分析***

遇到左边就入栈，遇到右边就判断与栈顶元素是否配对

- 若是，则栈顶元素弹出
- 若不是，则返回false

***代码***

```java
class Solution {
    public boolean isValid(String s) {
        Stack<Character> stk = new Stack<>();
        for(int i = 0 ; i < s.length();i++) {
            char c = s.charAt(i);
            if(c == '(' || c == '[' || c == '{') stk.push(c);
            else{
                if(c == ')') {
                    if(!stk.isEmpty() && stk.peek() == '(') stk.pop();
                    else return false;
                }else if(c == ']') {
                    if(!stk.isEmpty() && stk.peek() == '[') stk.pop();
                    else return false;
                }else if(c == '}') {
                    if(!stk.isEmpty() && stk.peek() == '{') stk.pop();
                    else return false;
                }
            }
        }
        return stk.isEmpty();
    }
}
```

#### 2.2、739 每日温度

***题目介绍***

> 请根据每日 气温 列表，重新生成一个列表。对应位置的输出为：要想观测到更高的气温，至少需要等待的天数。如果气温在这之后都不会升高，请在该位置用 0 来代替。
>
> 例如，给定一个列表 temperatures = [73, 74, 75, 71, 69, 72, 76, 73]，你的输出应该是 [1, 1, 4, 2, 1, 1, 0, 0]。
>
> 提示：气温 列表长度的范围是 [1, 30000]。每个气温的值的均为华氏度，都是在 [30, 100] 范围内的整数。
>

***分析***

温度数组倒序入栈，栈存序号

- 循环判断当前元素大于等于栈顶元素对应的值，栈顶元素弹出
- 若栈为空，结果为0， 否则为栈顶元素剪去当前索引
- 将当前索引入栈

***代码***

```java
class Solution {
    public int[] dailyTemperatures(int[] temperatures) {
        int[] res = new int[temperatures.length];
        Stack<Integer> s = new Stack<>();

        for(int i = temperatures.length - 1; i >= 0;i--) {
            while(!s.isEmpty() && temperatures[i] >= temperatures[s.peek()]) s.pop();
            res[i] = s.isEmpty() ? 0 :  (s.peek() - i);
            s.push(i);
        }
        return res;
    }
}
```











***题目介绍***

***分析***

***代码***
