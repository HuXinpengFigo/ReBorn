---
title: 剑指Offer刷题集锦Char-3
date: 2022-03-24 21:34:42
index_img: /img/1626948535.png
banner_img: /img/1200px-ISO_C++_Logo.svg.png
tags: [C++,牛课网,剑指Offer]
categories: 实习
---

## 栈

### JZ9 用两个栈实现一个队列

#### 题目描述

> 用两个栈来实现一个队列，完成队列的Push和Pop操作。 队列中的元素为int类型。

#### 思路

> 使用两个栈A、B，A是中间栈（push操作一律先插入A中），B是平时要pop的那个容器栈，当其不为空的时候pop操作一律使用B的top值，当B为空时，将A中之前push操作插入的元素全部出栈，加入B栈中。
>
> 入栈：将元素进栈A
>
> 出栈：判断栈A中元素的个数是否为1，如果等于1，则出栈，否则将栈A中的元素
> 出栈并放入栈B，直到栈A中的元素留下一个，然后栈A出栈，再把栈B中的元素出栈以此放入栈A中。

#### 代码

```c++
class Solution
{
public:
    void push(int node) {
        stack1.push(node);
    }

    int pop() {
        int ret;
        if(stack2.empty()){
            while(!stack1.empty()){
                ret = stack1.top();
                stack2.push(ret);
                stack1.pop();
            }
        }

        ret = stack2.top();
        stack2.pop();
        return ret;
    }

private:
    stack<int> stack1;
    stack<int> stack2;
};
```

### JZ30 包含min函数的栈

#### 题目描述

> 定义栈的数据结构，请在该类型中实现一个能够得到栈中所含最小元素的min函数（时间复杂度应为O（1））。
>
> 注意：保证测试中不会当栈为空的时候，对栈调用pop()或者min()或者top()方法。

#### 思路

> 写到时间复杂度为O（1），但是没具体要求空间复杂度，那么就可以用两个栈，辅助栈为局部最小值（比如第一个元素之间入栈，第二个元素如果小于等于第一个元素，也就是小于等于辅助栈的top，则入辅助栈）。主栈出栈的时候，判断一下当前要出栈的元素是不是辅助栈的top，如果是的话两个栈一起出栈，不是的话只针对主栈出栈。

#### 代码

```c++
class Solution {
public:
    stack<int> minVal;
    stack<int> stackWithMin;
    void push(int value) {
        if ( stackWithMin.empty() ) {
            minVal.push(value);
        } else {
            if ( minVal.top() >= value ) { //新value小于等于辅助栈top
                minVal.push(value);
            }
        }
        stackWithMin.push(value);
    }
    void pop() {
        if( stackWithMin.top() == minVal.top() ) {
            minVal.pop();
        }
        
        stackWithMin.pop();
    }
    int top() {
        return stackWithMin.top();
    }
    int min() {
        return minVal.top();
    }
};
```

### JZ73 翻转单词序列

#### 描述

>  牛客最近来了一个新员工Fish，每天早晨总是会拿着一本英文杂志，写些句子在本子上。同事Cat对Fish写的内容颇感兴趣，有一天他向Fish借来翻看，但却读不懂它的意思。例如，“nowcoder. a am I”。后来才意识到，这家伙原来把句子单词的顺序翻转了，正确的句子应该是“I am a nowcoder.”。Cat对一一的翻转这些单词顺序可不在行，你能帮助他么？
>
> 数据范围：1≤*n*≤100 
> 进阶：空间复杂度 O(n) ，时间复杂度 O(n) ，保证没有只包含空格的字符串

#### 思路

>  使用一个string栈来存放各个单词，遇到空格就把这个单词压入栈，最后再逐个出栈，记住输出的最后一个单词之后不用加上空格。

#### 代码

```c++
class Solution {
public:
    string ReverseSentence(string str) {
        if ( str.empty() ) {
            return str;
        }
        stack<string> reverseStack;
        
        bool spaceFlag = false;
        for ( int i = 0; i < str.size(); ++i) {
            if ( str[i] == ' ') {
                spaceFlag = true;
            }
        }
        if ( !spaceFlag ) return str;

        string temp ="";
        for ( int i = 0; i < str.size(); ++i) { //单词入栈
            if (str[i] != ' ') {
                temp = temp + str[i];
            }
            if ( str[i] == ' ' || i == str.size()-1) {
                reverseStack.push(temp);
                temp = "";
            }
        }
        
        string ret = "";
        while ( reverseStack.size() > 1 ) { //除了最后一个单词，都要带着空格入栈
            ret = ret + reverseStack.top() + " ";
            reverseStack.pop();
        }
        ret = ret + reverseStack.top(); //特殊处理最后一个单词，不用加空格
        return ret;
    }
};
```

### JZ31 栈的压入、弹出序列

#### 描述

> 输入两个整数序列，第一个序列表示栈的压入顺序，请判断第二个序列是否可能为该栈的弹出顺序。假设压入栈的所有数字均不相等。例如序列1,2,3,4,5是某栈的压入顺序，序列4,5,3,2,1是该压栈序列对应的一个弹出序列，但4,3,5,1,2就不可能是该压栈序列的弹出序列。
>
> 1. 0<=pushV.length == popV.length <=1000
>
> 2. -1000<=pushV[i]<=1000
>
> 3. pushV 的所有数字均不相同

示例1

输入：

```
[1,2,3,4,5],[4,5,3,2,1]
```

返回值：

```
true
```

说明：

```
可以通过push(1)=>push(2)=>push(3)=>push(4)=>pop()=>push(5)=>pop()=>pop()=>pop()=>pop()
这样的顺序得到[4,5,3,2,1]这个序列，返回true      
```

示例2

输入：

```
[1,2,3,4,5],[4,3,5,1,2]
```

返回值：

```
false
```

说明：

```
由于是[1,2,3,4,5]的压入顺序，[4,3,5,1,2]的弹出顺序，要求4，3，5必须在1，2前压入，且1，2不能弹出，但是这样压入的顺序，1又不能在2之前弹出，所以无法形成的，返回false      
```

#### 思路

> 使用一个真实的栈来进行模拟。在一个一个压栈的同时，对出栈序列进行头元素判定，如果相同，则能消去多少消去多少（模拟栈的头元素也要出栈），直到出栈序列的每个元素都判定过了，如果模拟栈全空了，则说明出栈序列就是真实栈的弹出序列，否则就不是。

#### 代码

```c++
class Solution {
public:
    bool IsPopOrder(vector<int> pushV,vector<int> popV) {
        if ( pushV.empty() ) return false;
        stack<int> stackSim;
        for ( int i=0, j=0; i<pushV.size();i++ ) { //i,j设定重点
            stackSim.push(pushV[i]);
            //此处要判断stackSim是不是非空，如果空了就不能进行后面top()的调用
            while ( j<popV.size() && !stackSim.empty() && stackSim.top() == popV[j] ) {
                stackSim.pop();
                j++;
            }
        }
        return stackSim.empty();
    }
};
```

### JZ59 滑动窗口的最大值

#### 描述

> 给定一个长度为 n 的数组 nums 和滑动窗口的大小 size ，找出所有滑动窗口里数值的最大值。
>
> 例如，如果输入数组{2,3,4,2,6,2,5,1}及滑动窗口的大小3，那么一共存在6个滑动窗口，他们的最大值分别为{4,4,6,6,6,5}； 针对数组{2,3,4,2,6,2,5,1}的滑动窗口有以下6个： {[2,3,4],2,6,2,5,1}， {2,[3,4,2],6,2,5,1}， {2,3,[4,2,6],2,5,1}， {2,3,4,[2,6,2],5,1}， {2,3,4,2,[6,2,5],1}， {2,3,4,2,6,[2,5,1]}。

示例1

输入：

```
[2,3,4,2,6,2,5,1],3
```

返回值：

```
[4,4,6,6,6,5]
```

示例2

输入：

```
[9,10,9,-7,-3,8,2,-6],5
```

返回值：

```
[10,10,9,8]
```

示例3

输入：

```
[1,2,3,4],3
```

返回值：

```
[3,4]
```

#### 思路

> 简单思路就是遍历每个窗口求最大值；
>
> 进阶版是维护一个单调队列（使用双端队列），这个队列队头是当前窗口最大元素的下标。注意单调队列的定义，如果入队的值大于队尾，就删掉队中比它小的所有元素直到它成为队头。然后还要考虑在滑动窗口已经滑出当前最大值所在下标的情况，这个地方也要出队。

#### 代码

```c++
class Solution {
public:
    vector<int> maxInWindows(const vector<int>& nums, int size) {
        vector<int> ret;
        deque<int> subsequence;
        for ( int i=0; i < nums.size(); i++) {
            // 重点在双端删除，这里删的是比当前入队要小的元素
            while ( !subsequence.empty() && nums[i] > nums[subsequence.back()]) {
                subsequence.pop_back();
            }
            
            // 这里删的是最大但是已经不在滑动窗口的元素(删队头)
            if ( !subsequence.empty() && i - subsequence.front() > size - 1) {
                subsequence.pop_front();
            }
            subsequence.push_back(i);
            
            //当i>size-1即当滑动窗口成立（之前还没到size的大小）
            if ( size!=0 && i >= size -1)
                ret.push_back(nums[subsequence.front()]);
        }
        return ret;
    }
};
```

