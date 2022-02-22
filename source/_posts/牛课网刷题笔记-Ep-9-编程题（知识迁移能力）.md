---
title: 牛课网刷题笔记-Ep-9-编程题（知识迁移能力）
date: 2020-04-11 19:22:26
index_img: /img/Xnip2020-02-05_19-55-51.png
banner_img: /img/1200px-ISO_C++_Logo.svg.png
tags: [C++,牛课网]
categories: 实习
---

### 数字在排序数组中出现的次数<!-- more -->

#### 题目描述

> 统计一个数字在排序数组中出现的次数。

#### 思路

还是直接遍历好，二分法因为查到的相同的很多，可能不对，可能查到的是第二个k

#### 代码

```c++
class Solution {
public:
    int GetNumberOfK(vector<int> data ,int k) {
        int count=0;
        if(data.empty())
            return count;
        bool flagK=false;
        int i;
        for(i=0;i<data.size()&&k!=data[i];i++);//遍历写法
        if(i<data.size())
            while(k==data[i]){
                ++i;
                ++count;
            }
        return count;
    }
};
```

### 二叉树的深度

#### 题目描述

> 输入一棵二叉树，求该树的深度。从根结点到叶结点依次经过的结点（含根、叶结点）形成树的一条路径，最长路径的长度为树的深度。

#### 思路

简单递归问题，要先想好return的条件，之后再想进入递归改如何累加。

#### 代码

```c++
class Solution {
public:
    int TreeDepth(TreeNode* pRoot) {
        int depth=0;
        if(!pRoot){
            return 0;
        }
        return 1+TreeDepth(pRoot->left) > 1+TreeDepth(pRoot->right)?
            1+TreeDepth(pRoot->left) : 1+TreeDepth(pRoot->right);
    }
};
```

### 平衡二叉树

#### 题目描述

> 输入一棵二叉树，判断该二叉树是否是平衡二叉树。

#### 思路

1. 最直接的方法，算出左右子树的高度，然后逐一判断左右子树高度差是否小于等于1
2. 更优做法，从下往上判断，一旦有一个子树不是平衡树则直接返回

#### 代码

```c++
//最直接的方法，算出左右子树的高度，然后逐一判断左右子树高度差是否小于等于1

class Solution {
public:
    int getDepth(TreeNode* root){
        if(!root)
            return 0;
        return max(1+getDepth(root->left), 1+getDepth(root->right));
    }
    bool IsBalanced_Solution(TreeNode* pRoot) {
        if(!pRoot)
            return true;
        return abs(getDepth(pRoot->left)-getDepth(pRoot->right))<=1 
            && IsBalanced_Solution(pRoot->left) && IsBalanced_Solution(pRoot->right);
    }
};


//更优做法，从下往上判断，一旦有一个子树不是平衡树则直接返回

class Solution {
public:
    bool IsBalanced_Solution(TreeNode* root) {
        return getDepth(root) != -1;
    }
     
private:
    int getDepth(TreeNode* root) {
        if (root == NULL) return 0;
        int left = getDepth(root->left);
        if (left == -1) return -1;
        int right = getDepth(root->right);
        if (right == -1) return -1;
        return abs(left - right) > 1 ? -1 : 1 + max(left, right);
    }
};
```

### 数组中只出现一次的数字

#### 题目描述

> 一个整型数组里除了两个数字之外，其他的数字都出现了两次。请写程序找出这两个只出现一次的数字。

#### 思路

利用异或的思想，用map空间限制无法满足。先利用一个标志int来对所有元素异或一次，肯定有一位以上是1，这就是那两个只出现一次的元素的不一样的位数。然后利用这个位来将整个数组分为两个部分，分别将这两个部分所有元素互相异或，剩下的就是出现一次的元素。

#### 代码

```c++
class Solution {
public:
    void FindNumsAppearOnce(vector<int> data,int* num1,int *num2) {
        if(data.size()<2)
            return ;
        int resultExclusiveOR=0;
        for(int i=0;i<data.size();i++){
            resultExclusiveOR = resultExclusiveOR ^ data[i];
        }
        unsigned int indexOf1 = findFirstBit1(resultExclusiveOR);
        *num1 = *num2 = 0;
        for(int i = 0;i<data.size();i++){
            if(isBit1(data[i],indexOf1))
                *num1 = *num1^data[i];
            else
                *num2 = *num2^data[i];
        }
    }
    unsigned int findFirstBit1(int num){
        int index = 0;
        while((num&1)==0&&(index < 8*sizeof(unsigned int))){//注意此处==比&优先级要高，所以必须把num&1用括号扩起来
            num = num>>1;
            index++;
        }
        return index;
    }
    bool isBit1(int num,unsigned int index){
        num = num>>index;
        return (num&1);
    }
};
```

### 和为S的连续正数数列

#### 题目描述

> 小明很喜欢数学,有一天他在做数学作业时,要求计算出9~16的和,他马上就写出了正确答案是100。但是他并不满足于此,他在想究竟有多少种连续的正数序列的和为100(至少包括两个数)。没多久,他就得到另一组连续正数和为100的序列:18,19,20,21,22。现在把问题交给你,你能不能也很快的找出所有和为S的连续正数序列? Good Luck!

#### 输出描述:

```
输出所有和为S的连续正数序列。序列内按照从小至大的顺序，序列间按照开始数字从小到大的顺序
```

#### 思路

重点是序列是连续的，那么就可以用双指针的思想来框一个滑动窗口出来，如果窗口内数字和小了，则把右边的指针右移，如果窗口内的数字大了，则把左边的指针右移，最后就能得到序列了。

#### 代码

```c++
class Solution {
public:
    vector<vector<int> > FindContinuousSequence(int sum) {
        vector<vector<int>> result;
        int pLow = 1, pHigh = 2;
        //两个起点，相当于动态窗口的两边，根据其窗口内的值的和来确定窗口的位置和大小
        while(pLow < pHigh){
            int cur = (pLow + pHigh) * (pHigh - pLow +1)/2;//由于是连续的，差为1的一个序列，那么求和公式是(a0+an)*n/2
                                    //注意这里是pHigh - pLow +1，这个1不要忘记
            if(cur == sum){//相等，那么就将窗口范围的所有数添加进结果集
                vector<int> temResult;
                for(int i=pLow;i<=pHigh;i++){
                    temResult.push_back(i);
                }
                result.push_back(temResult);
                pLow++;
            }else if(cur < sum){ //如果当前窗口内的值之和小于sum，那么右边窗口右移一下
                pHigh++;
            }else{//如果当前窗口内的值之和大于sum，那么左边窗口右移一下
                pLow++;
            }
        }
        return result;
    }
};
```

### 和为S的两个数字

#### 题目描述

> 输入一个递增排序的数组和一个数字S，在数组中查找两个数，使得他们的和正好是S，如果有多对数字的和等于S，输出两个数的乘积最小的。

#### 思路

使用两个指向指针，前面为low，后面为high，分别从头尾像中间靠拢；如果两者相加之和大于S，则high--，小于S则low++

#### 代码

```c++
class Solution {
public:
    vector<int> FindNumbersWithSum(vector<int> array,int sum) {
        vector<int> res;
        int low = 0, high = array.size()-1;
        while(low < high){
            if(array[low] + array[high] == sum){
                res.push_back(array[low]);
                res.push_back(array[high]);
                return res; //一定要一找到就return，不然会说超过内存限制
            }
            else if(array[low] + array[high] < sum ) low++;
            else high--;
        }
        return res;
    }
};
```

### 左旋转字符串

#### 题目描述

> 汇编语言中有一种移位指令叫做循环左移（ROL），现在有个简单的任务，就是用字符串模拟这个指令的运算结果。对于一个给定的字符序列S，请你把其循环左移K位后的序列输出。例如，字符序列S=”abcXYZdef”,要求输出循环左移3位后的结果，即“XYZdefabc”。是不是很简单？OK，搞定它！

#### 思路

首先要看左旋的数目n为多大，是否已经超过了字符串的len，如果已经超过了，则只需要算左旋`n%len`就可以了；

然后一个取巧的方法，直接在str后面加上str，然后左旋了多少，直接取`substr(n,len)`返回就可以了。

```c++
class Solution {
public:
    string LeftRotateString(string str, int n) {
        int len = str.size();
        if(len == 0) return "";
        n = n%len;//算出实际上需要循环右移的次数
        str += str;
        return str.substr(n,len);//substr(a,b) a是从第几个开始，b是子字符串的长度
    }
};
```

### 旋转单词顺序序列

#### 题目描述

> 牛客最近来了一个新员工Fish，每天早晨总是会拿着一本英文杂志，写些句子在本子上。同事Cat对Fish写的内容颇感兴趣，有一天他向Fish借来翻看，但却读不懂它的意思。例如，“student. a am I”。后来才意识到，这家伙原来把句子单词的顺序翻转了，正确的句子应该是“I am a student.”。Cat对一一的翻转这些单词顺序可不在行，你能帮助他么？

#### 思路

首先字符串整体反序，之后再把各个单词反序就可以了。需要注意，各个单词反序的时候，一开始是遇到空格就开始反序之前的，而到了最后一个单词，需要特殊对待，直接在循环外反序。

#### 代码

```c++
#include <iostream>
#include <stack>
using namespace std;

void reverseS(string &str, int l,int r){
    while(l<r){
        swap(str[l],str[r]);
        l++;
        r--;
    }
}

int main(){
    string str;
    getline(cin, str);
    reverseS(str,0,str.size()-1);
    int count = 0;
    int i;
    for(i=0;i<str.size();i++){
        if(str[i]!=' '){
            count++;
        }else if(str[i]==' '){
            reverseS(str,i-count,i-1);
            count=0;
        }
    }
    reverseS(str,str.size()-count,str.size()-1);
    cout<<str;
    return 0;
}

```

