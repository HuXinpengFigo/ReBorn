---
title: 牛课网刷题笔记-Ep-3-编程题（数组、字符串、链表）
date: 2020-03-31 16:33:08
index_img: /img/368-3682760_c-language-hd-png-download.png.jpeg
banner_img: /img/1200px-ISO_C++_Logo.svg.png
tags: [C++,牛课网]
categories: 实习
---

### 数组

#### 二维数组中的查找<!-- more -->

##### 题目描述

> 在一个二维数组中（每个一维数组的长度相同），每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。

##### 思路

先从后往前一行一行找，当每一行的第一个一个数字大于当前要找的数字，则行数自减，如果第一个数字小于或者等于当前的要找的数字，则说明目标数字行数可以确定下来了；之后再列数逐个增加来寻找就可以了。

##### 代码

```c++
class Solution {
public:
    bool Find(int target, vector<vector<int> > array) {
        int RowLen = array.size();
        int ColLen = array[0].size();
        for(int i = RowLen-1, j = 0; i>=0&&j<ColLen; ){//从后往前找，也可从前往后，但是这样就要和每行最后一个比
            if(array[i][j]==target)
                return true;
            else if(array[i][j]>target){//寻找target所在行
                i--;
                continue;
            }else{//寻找target具体定位
                j++;
                continue;
            }
        }
        return false;
    }
};
```

#### 数组中重复的数字

##### 题目描述

> 在一个长度为n的数组里的所有数字都在0到n-1的范围内。 数组中某些数字是重复的，但不知道有几个数字是重复的。也不知道每个数字重复几次。请找出数组中任意一个重复的数字。 例如，如果输入长度为7的数组{2,3,1,0,2,5,3}，那么对应的输出是第一个重复的数字2。

##### 思路

该题采用了比较特殊的思路，主要思想是针对题目的数字范围在0～n-1这个条件，把数组中值为k的数重新放到位置也为k的位置。由于肯定有一个重复的数字（k属于0～n-1），所以第一个出现要交换的时候已经被一个值为k的数字占坑的情况下，该数字就是第一个重复的数字。

主要代码：`numbers[i]==numbers[numbers[i]]`

##### 代码

```c++
class Solution {
//此方法只针对数字范围是0～n-1的数组
bool duplicate(int numbers[], int length, int* duplication) {
    if(length<=0||!numbers)
        return false;
    for(int i=0;i<length;i++){
        while(numbers[i]!=i){
            if(numbers[i]==numbers[numbers[i]]){// 如果数组中第i位置的元素值A等于数组中第A位置的元素值
                *duplication = numbers[i];//这里不能用取址duplication=&numbers[i]，会出问题
                return true;
            }else{// 如果数组中第i位置的元素值A不等于数组中第A位置的元素值，则互相交换这两个位置的元素，使最后位置都对应上
                swap(numbers[i],numbers[numbers[i]]);
            }
        }
    }
    return false;
}
};
```

#### 构建乘积数组

##### 题目描述

> 给定一个数组A[0,1,...,n-1],请构建一个数组B[0,1,...,n-1],其中B中的元素B[i]=A[0]*A[1]*...*A[i-1]*A[i+1]*...*A[n-1]。不能使用除法。（注意：规定B[0] = A[1] * A[2] * ... * A[n-1]，B[n-1] = A[0] * A[1] * ... * A[n-2];）

##### 思路

该题需要用到数学上的连乘思想，把矩阵分为上下两个三角子矩阵。先把B0～Bn-1用矩阵表示，然后可以发现是个主对角线全为1的矩阵，先连乘算出下三角，再回来连乘上三角。

![](/img/841505_1472459965615_8640A8F86FB2AB3117629E2456D8C652.jpeg)

##### 代码

```c++
class Solution {
    
public:
    vector<int> multiply(const vector<int>& A) {
        vector<int> B(A.size());
        if(A.size()==0)
            return B;
        B[0] = 1;
        for(int i = 1;i<A.size();i++){//计算下三角
            B[i] = B[i-1]*A[i-1];
        }
        int temp = 1;
        for(int i = A.size()-2;i>=0;i--){//计算上上三角
            temp = temp*A[i+1];
            B[i] = B[i]*temp;
        }
        return B;
    }
};
```

### 字符串

#### 替换空格

##### 题目描述

> 请实现一个函数，将一个字符串中的每个空格替换成“%20”。例如，当字符串为We Are Happy.则经过替换之后的字符串为We%20Are%20Happy。

##### 思路

首先得出当前字符串长度（包括空格）和空格数，得到之后就可以由此计算出全部替换完的字符串长度了。之后再利用新长度，从字符串末尾开始遍历，依次将字符后移或者替换（空格）。

##### 代码

```c++
class Solution {
public:
	void replaceSpace(char *str,int length) {//先算出替换完成之后的字符串长度，再从后往前遍历，减少移动次数
        int spaceLen = 0;
        int originalLen = 0;
        if(str==NULL||length<=0){
            return;
        }
        int i = 0;
        while(str[i]!='\0'){
            originalLen++;
            if(str[i]==' '){
                spaceLen++;
            }
            i++;
        }
        int newLen = originalLen + spaceLen*2;
        while(originalLen>=0){//按原字符串的倒序来遍历，一个一个后移
            if(str[originalLen] == ' '){
                str[newLen--] = '0';
                str[newLen--] = '2';
                str[newLen--] = '%';
            }else{
                str[newLen--] = str[originalLen];
            }
            originalLen--;
        }
        return;
	}
};
```

#### 正则表达式匹配

##### 题目描述

> 请实现一个函数用来匹配包括'.'和'*'的正则表达式。模式中的字符'.'表示任意一个字符，而'*'表示它前面的字符可以出现任意次（包含0次）。 在本题中，匹配是指字符串的所有字符匹配整个模式。例如，字符串"aaa"与模式"a.a"和"ab*ac*a"匹配，但是与"aa.a"和"ab*a"均不匹配

##### 思路

需要用到递归，针对不同情况。

```
首先，考虑特殊情况：
         1>两个字符串都为空，返回true
         2>当第一个字符串不空，而第二个字符串空了，返回false（因为这样，就无法
            匹配成功了,而如果第一个字符串空了，第二个字符串非空，还是可能匹配成
            功的，比如第二个字符串是“a*a*a*a*”,由于‘*’之前的元素可以出现0次，
            所以有可能匹配成功）
之后就开始匹配第一个字符，这里有两种可能：匹配成功或匹配失败。但考虑到pattern
下一个字符可能是‘*’， 这里我们分两种情况讨论：pattern下一个字符为‘*’或不为‘*’：
          1>pattern下一个字符为‘*’时，稍微复杂一些，因为‘*’可以代表0个或多个。
            这里把这些情况都考虑到：
               a>当‘*’匹配0个字符时，str当前字符不变，pattern当前字符后移两位，
                跳过这个‘*’符号；
               b>当‘*’匹配1个或多个时，str当前字符移向下一个，pattern当前字符
                不变。（这里匹配1个或多个可以看成一种情况，因为：当匹配一个时，
                由于str移到了下一个字符，而pattern字符不变，就回到了上边的情况a；
                当匹配多于一个字符时，相当于从str的下一个字符继续开始匹配）
          2>pattern下一个字符不为‘*’：这种情况比较简单，直接匹配当前字符。如果
            匹配成功，继续匹配下一个；如果匹配失败，直接返回false。注意这里的
            “匹配成功”，除了两个字符相同的情况外，还有一种情况，就是pattern的
            当前字符为‘.’,同时str的当前字符不为‘\0’。
之后再写代码就很简单了。
```

##### 代码

```c++
class Solution {
public:
    bool match(char* str, char* pattern)
    {
        if(*str=='\0' && *pattern =='\0')
            return true;
        if(*str!='\0' && *pattern =='\0')
            return false;
        if(*(pattern+1) == '*'){
            if(*pattern == *str || (*str != '\0' && *pattern == '.')){//两种可能：1.双方匹配2.是任意字符。之后因为下一个是‘*’，所以也有两种可能。
                return match(str, pattern+2) || match(str+1,pattern);
            }else{
                return match(str, pattern+2);
            }
        }else{
            if(*pattern == *str || (*str != '\0' && *pattern == '.')){
                return match(str+1, pattern +1);
            }else{
                return false;
            }
        }
    }
};
```

#### 表示数字的字符串

##### 题目描述

> 请实现一个函数用来判断字符串是否表示数值（包括整数和小数）。例如，字符串"+100","5e2","-123","3.1416"和"-1E-16"都表示数值。 但是"12e","1a3.14","1.2.3","+-5"和"12e+4.3"都不是。

##### 思路

此题没有捷径，只能把每种情况都考虑到：

1. ‘E’和‘e’后面必须接正负号数字，且无小数点
2. 第二次出现正负号，必须在‘E’和‘e’后面
3. 第一次出现+-符号，且不是在字符串开头，则也必须紧接在‘E’和‘e’之后
4. e后面不能接小数点，小数点不能出现两次
5. str>'9'||str<'0'非法
6. 只输入一个符号（非数字）非法

##### 代码

```c++
class Solution {
public:
    bool isNumeric(char* string)
    {
        int len = strlen(string);
        bool sign = false, point = false, hasE = false;
        if(len==0 || (len==1&&(string[0]>'9'&&string[0]<'0')))
            return false;
        for(int i = 0; i < len; i++){
            if(string[i]=='e'||string[i]=='E'){
                if(i == len-1) return false;
                if(hasE) return false;
                hasE = true;
            }else if(string[i]=='+'||string[i]=='-'){
                if(sign && string[i-1]!='e'&&string[i-1]!='E') return false;
                if(!sign && i>0 && string[i-1]!='e' && string[i-1]!='E') return false;
                sign = true;
            }else if(string[i] == '.'){
                if(hasE) return false;
                if(point) return false;
                point = true;
            }else if(string[i]>'9'||string[i]<'0'){
                return false;
            }
        }
        return true;
    }
};
```

#### 从尾到头打印链表

##### 题目描述

> 输入一个链表，按链表从尾到头的顺序返回一个ArrayList。

##### 思路

此题可以有多种做法，可以用栈来暂存也可以直接用vector来存，之后栈依次出栈、vector使用stl中的reverse函数翻转即可。

##### 代码

```c++
class Solution {
public:
    vector<int> printListFromTailToHead(ListNode* head) {
        vector<int> ret;
        if(!head){
            return ret;
        }
        while(head){
            ret.push_back(head->val);
            head = head->next;
        }
        reverse(ret.begin(),ret.end());
        return ret;
    }
};
```

#### 链表环的入口结点

##### 题目描述

> 给一个链表，若其中包含环，请找出该链表的环的入口结点，否则，输出null。

##### 思路

使用快慢指针，首先快慢指针相遇确定相遇点。相遇之后快指针不变（但是速度变为和慢指针相同，相当于也是一个慢指针），继续走，同时新建一个慢指针，从头节点继续走，这次两个指针相遇的地方就是环的入口了。

具体原理：

链表头到环入口长度为--**a**  

环入口到相遇点长度为--**b**  

相遇点到环入口长度为--**c**  

![](/img/kuaiman.jpeg)

则：相遇时 

**快指针路程=a+(b+c)k+b** ，k>=1 其中b+c为环的长度，k为绕环的圈数（k>=1,即最少一圈，不能是0圈，不然和慢指针走的一样长，矛盾）。 

**慢指针路程=a+b**  

快指针走的路程是慢指针的两倍，所以： 

**（a+b）\*2=a+(b+c)k+b**  

化简可得： 

**a=(k-1)(b+c)+c**  这个式子的意思是：  **链表头到环入口的距离=相遇点到环入口的距离+（k-1）圈环长度**。其中k>=1,所以**k-1>=0**圈。所以两个指针分别从链表头和相遇点出发，最后一定相遇于环入口。

##### 代码

```c++
class Solution {
public:
    ListNode* EntryNodeOfLoop(ListNode* pHead)
    {
        ListNode *fast=pHead,*low=pHead;
        while(fast&&fast->next){
            fast = fast->next->next;
            low = low->next;
            if(fast == low)
                break;
        }
        if(!fast||!fast->next)//无环
            return NULL;
        low = pHead;
        while(low!=fast){
            low = low->next;
            fast = fast->next;
        }
        return low;
    }
};
```

#### 删除链表中重复的结点

##### 题目描述

> 在一个排序的链表中，存在重复的结点，请删除该链表中重复的结点，重复的结点不保留，返回链表头指针。 例如，链表1->2->3->3->4->4->5 处理后为 1->2->5

##### 思路

此题较简单，因为是排序的链表，所以重复的结点都相邻。其中一个坑就是“去出重复的节点（包括节点本身）”，所以头节点也可能被删掉于是新增一个Head，使Head->next才等于pHead。

##### 代码

```c++
class Solution {
public:
    ListNode* deleteDuplication(ListNode* pHead)
    {
        ListNode* Head = new ListNode(0);
        Head->next = pHead;
        ListNode* pre = Head;
        ListNode* last = Head->next;
        while(last){
            if((last->next) && last->val==last->next->val){ //记住只要是用到了next的val，一定要把last->next!=NULL作为先决条件
                while((last->next) && last->val==last->next->val){
                    last = last->next;
                }
                pre->next = last->next;
                last = last->next;
            }else{
                pre = pre->next;
                last = last->next;
            }
        }
        return Head->next;
    }
};
```

