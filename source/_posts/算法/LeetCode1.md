---
title: LeetCode练习1
date: 2022-06-18 10:00:00
categories:
- 算法
---

### [剑指 Offer 09. 用两个栈实现队列 - 力扣（LeetCode）](https://leetcode.cn/problems/yong-liang-ge-zhan-shi-xian-dui-lie-lcof/)

> 队列头进尾出，两个栈一个栈只进，一个只出；出栈时，从statckIn：pop（）到statckOut中。
>
> 这样刚好满足队列要求。

```java
class CQueue {

     Stack<Integer> stackIn;
    Stack<Integer> stackOut;
    public CQueue(){
        stackIn = new Stack();
        stackOut = new Stack();
    }
    public void appendTail(int value) {
        stackIn.push(value);
    }

    public int deleteHead() {
        if (stackOut.isEmpty()){
            if (stackIn.isEmpty()){
                return -1;
            }
            while (!stackIn.isEmpty())
                stackOut.push(stackIn.pop());
        }
        return stackOut.pop();
    }
}

/**
 * Your CQueue object will be instantiated and called as such:
 * CQueue obj = new CQueue();
 * obj.appendTail(value);
 * int param_2 = obj.deleteHead();
 */
```



### [剑指 Offer 30. 包含min函数的栈 - 力扣（LeetCode）](https://leetcode.cn/problems/bao-han-minhan-shu-de-zhan-lcof/)

> 输出min函数的栈，就要搞一个min栈，每次入栈的时候比较栈内元素，保存最小的栈数据。
>
> peek：输出栈顶元素。

```java
class MinStack {

     Stack<Integer> stack1;
    Stack<Integer> stackMin;

    public MinStack() {
        stack1 = new Stack<>();
        stackMin = new Stack<>();
        
        stackMin.push(Integer.MAX_VALUE);
    }

    public void push(int x) {
        stack1.push(x);
        stackMin.push(Math.min(stackMin.peek(), x));
    }

    public void pop() {
        stack1.pop();
        stackMin.pop();
    }

    public int top() {
        return stack1.peek();
    }

    public int min() {
        return stackMin.peek();
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

