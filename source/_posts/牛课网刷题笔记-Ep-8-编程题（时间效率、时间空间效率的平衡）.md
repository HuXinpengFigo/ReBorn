---
title: 牛课网刷题笔记-Ep-8-编程题（时间效率、时间空间效率的平衡）
date: 2020-04-07 20:35:42
index_img: /img/368-3682760_c-language-hd-png-download.png.jpeg
banner_img: /img/1200px-ISO_C++_Logo.svg.png
tags: [C++,牛课网]
categories: 实习
---

### 数组中出现次数超过一半的数字<!-- more -->

#### 题目描述

> 数组中有一个数字出现的次数超过数组长度的一半，请找出这个数字。例如输入一个长度为9的数组{1,2,3,2,2,2,5,4,2}。由于数字2在数组中出现了5次，超过数组长度的一半，因此输出2。如果不存在则输出0。

#### 思路

数组排序后，如果符合条件的数存在，则一定是数组中间那个数。
（比如：1，2，2，2，3；或2，2，2，3，4；或2，3，4，4，4等等）
这种方法虽然容易理解，但由于涉及到快排sort，其时间复杂度为O(NlogN)并非最优；

#### 代码

```c++
class Solution {
public:
    int MoreThanHalfNum_Solution(vector<int> numbers) {
        if(numbers.size()==0)
            return 0;
        sort(numbers.begin(),numbers.end());
        int middle = numbers[numbers.size()/2];
        int count = 0;
        for(int i=0;i<numbers.size();i++){
            if(numbers[i]==middle)
                count++;
        }
        if(count>numbers.size()/2)
            return middle;
        else
            return 0;
    }
};
```

### 最小的k个数

#### 题目描述

> 输入n个整数，找出其中最小的K个数。例如输入4,5,1,6,2,7,3,8这8个数字，则最小的4个数字是1,2,3,4,。

#### 思路

全排序  时间复杂度O（nlogn）

#### 代码

```c++
class Solution {
public:
    vector<int> GetLeastNumbers_Solution(vector<int> input, int k) {
        vector<int> res;
        if(input.empty()||k==0||k>input.size())
            return res;
        sort(input.begin(),input.end());
        for(int i=0;i<k;i++){
            res.push_back(input[i]);
        }
        return res;
    }
};
```

### 整数中1出现的次数（从1到n整数中出现1的个数）

#### 题目描述

> 求出1~13的整数中1出现的次数,并算出100~1300的整数中1出现的次数？为此他特别数了一下1~13中包含1的数字有1、10、11、12、13因此共出现6次,但是对于后面问题他就没辙了。ACMer希望你们帮帮他,并把问题更加普遍化,可以很快的求出任意非负整数区间中1出现的次数（从1 到 n 中1出现的次数）。

#### 思路

可以直接暴力搜索0～n每个数字中有几个1，之后累加就行

#### 代码

```c++
class Solution {
public:
    int count1(int n){
        int count = 0;
        while(n){
            if(n%10==1)
                ++count;
            n = n/10;
        }
        return count;
    }
    int NumberOf1Between1AndN_Solution(int n)
    {
        int count = 0;
        for(int i=1;i<=n;i++){
            count += count1(i);
        }
        return count;
    }
};
```

### 把数组排成最小的数

#### 题目描述

> 输入一个正整数数组，把数组里所有数字拼接起来排成一个数，打印能拼接出的所有数字中最小的一个。例如输入数组{3，32，321}，则打印出这三个数字能排成的最小数字为321323。

#### 思路

对vector容器内的数据进行排序，按照 将a和b转为string后
 若 a＋b<b+a  a排在在前 的规则排序,
 如 2 21 因为 212 < 221 所以 排序后为 21 2 
  to_string() 可以将int 转化为string

sort中的比较函数compare要声明为静态成员函数或全局函数，不能作为普通成员函数，否则会报错。
因为：非静态成员函数是依赖于具体对象的，而std::sort这类函数是全局的，因此无法再sort中调
用非静态成员函数。静态成员函数或者全局函数是不依赖于具体对象的, 可以独立访问，无须创建任何
对象实例就可以访问。同时静态成员函数不可以调用类的非静态成员。

#### 代码

```c++
class Solution {
public:
    static bool cmp(int a,int b){//必须加static
        string A = "";
        string B = "";
        A += to_string(a);
        A += to_string(b);
        B += to_string(b);
        B += to_string(a);
        return A < B;
    }
    string PrintMinNumber(vector<int> numbers) {
        string answer = "";
        sort(numbers.begin(), numbers.end(),cmp);
        for(int i=0;i<numbers.size();i++)
            answer += to_string(numbers[i]);
        return answer;
    }
};
```

### 丑数

#### 题目描述

> 把只包含质因子2、3和5的数称作丑数（Ugly Number）。例如6、8都是丑数，但14不是，因为它包含质因子7。 习惯上我们把1当做是第一个丑数。求按从小到大的顺序的第N个丑数。

#### 思路

首先从丑数的定义我们知道，一个丑数的因子只有2,3,5，那么丑数p = 2 ^ x * 3 ^ y * 5 ^ z，换句话说一个丑数一定由另一个丑数乘以2或者乘以3或者乘以5得到，那么我们从1开始乘以2,3,5，就得到2,3,5三个丑数，在从这三个丑数出发乘以2,3,5就得到4，6,10,6，9,15,10,15,25九个丑数，我们发现这种方法得到重复的丑数，而且我们题目要求第N个丑数，这样的方法得到的丑数也是无序的。

#### 代码

```c++
class Solution {
public:
    int GetUglyNumber_Solution(int index) {
        if(index<7)
            return index;
        int p2 = 0, p3 = 0, p5 = 0;
        int newNum = 1;
        vector<int> arr;
        arr.push_back(newNum);
        while(arr.size() < index){
            newNum = min(arr[p2]*2,min(arr[p3]*3,arr[p5]*5));
            if(arr[p2]*2==newNum) p2++;
            if(arr[p3]*3==newNum) p3++;
            if(arr[p5]*5==newNum) p5++;
            arr.push_back(newNum);
        }
        return newNum;
    }
};
```

### 第一个只出现一次的字符

#### 题目描述

> 在一个字符串(0<=字符串长度<=10000，全部由字母组成)中找到第一个只出现一次的字符,并返回它的位置, 如果没有则返回 -1（需要区分大小写）。

#### 思路

用一个map来记录每个元素出现的次数，然后再遍历一遍，第一个出现次数为一的数返回。

#### 代码

```c++
class Solution {
public:
    int FirstNotRepeatingChar(string str) {
        map<char, int> mp;
        for(int i=0; i<str.size();i++){
            mp[str[i]]++;
        }
        for(int i=0; i<str.size();i++){
            if(mp[str[i]]==1)
                return i;//注意是返回位置
        }
        return -1;
    }
};
```

### 数组中的逆序对

#### 题目描述

> 在数组中的两个数字，如果前面一个数字大于后面的数字，则这两个数字组成一个逆序对。输入一个数组,求出这个数组中的逆序对的总数P。并将P对1000000007取模的结果输出。 即输出P%1000000007

#### 输入描述:

```
题目保证输入的数组中没有的相同的数字数据范围：
对于%50的数据,size<=10^4	
对于%75的数据,size<=10^5	
对于%100的数据,size<=2*10^5
```

#### 思路

一般方法：直接暴力穷举，O(n^2),超时
优化方法：类归并排序方法，O(nlog(n))

#### 代码

```c++
class Solution {
public:
    int InversePairs(vector<int> data) {
        if(data.size()<=1) return 0;//如果少于等于1个元素，直接返回0
        int* copy=new int[data.size()];
        //初始化该数组，该数组作为存放临时排序的结果，最后要将排序的结果复制到原数组中
        for(unsigned int i=0;i<data.size();i++)
            copy[i]=0;
        //调用递归函数求解结果
        int count=InversePairCore(data,copy,0,data.size()-1);
        delete[] copy;//删除临时数组
        return count;
    }
     //程序的主体函数
    int InversePairCore(vector<int>& data,int*& copy,int start,int end)
    {
        if(start==end)
        {
            copy[start]=data[start];
            return 0;
        }
        //将数组拆分成两部分
        int length=(end-start)/2;//这里使用的下标法，下面要用来计算逆序个数；也可以直接使用mid=（start+end）/2
        //分别计算左边部分和右边部分
        int left=InversePairCore(data,copy,start,start+length)%1000000007;
        int right=InversePairCore(data,copy,start+length+1,end)%1000000007;
        //进行逆序计算
        int i=start+length;//前一个数组的最后一个下标
        int j=end;//后一个数组的下标
        int index=end;//辅助数组下标，从最后一个算起
        int count=0;
        while(i>=start && j>=start+length+1)
        {
            if(data[i]>data[j])
            {
                copy[index--]=data[i--];
                //统计长度
                count+=j-start-length;
                if(count>=1000000007)//数值过大求余
                    count%=1000000007;
            }
            else
            {
                copy[index--]=data[j--];
            }
        }
        for(;i>=start;--i)
        {
            copy[index--]=data[i];
        }
        for(;j>=start+length+1;--j)
        {
            copy[index--]=data[j];
        }
        //排序
        for(int i=start; i<=end; i++) {
            data[i] = copy[i];
        }
        //返回最终的结果
        return (count+left+right)%1000000007;
    }
};
```

### 两个链表的第一个公共节点

#### 题目描述

> 输入两个链表，找出它们的第一个公共结点。（注意因为传入数据是链表，所以错误测试数据的提示是用其他方式显示的，保证传入数据是正确的）

#### 思路

两个链表相交，肯定是一个Y型，所以只要找到长度差，之后长的先走，走到两者剩下的长度相同时，开始同时走，直到遇到第一个公共节点。

#### 代码

```c++
class Solution {
public:
    int calculateLen(ListNode* root){
        int count = 0;
        while(root){
            count++;
            root=root->next;
        }
        return count;
    }
    
    ListNode* walkNStep(ListNode* root, int steps){
        for(int i=0;i<steps;i++){
            root = root->next;
        }
        return root;
    }
    
    ListNode* FindFirstCommonNode( ListNode* pHead1, ListNode* pHead2) {
        int len1 = calculateLen(pHead1);
        int len2 = calculateLen(pHead2);
        if(len1>len2){
            pHead1 = walkNStep(pHead1, len1-len2);
        }else{
            pHead2 = walkNStep(pHead2, len2-len1);
        }
        while(pHead1){
            if(pHead1==pHead2)
                return pHead1;
            pHead1 = pHead1->next;
            pHead2 = pHead2->next;
        }
        return NULL;
    }
};
```

