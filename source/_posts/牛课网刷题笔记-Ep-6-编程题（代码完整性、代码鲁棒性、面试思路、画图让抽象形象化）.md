---
title: 牛课网刷题笔记-Ep-6-编程题（代码完整性、代码鲁棒性、面试思路、画图让抽象形象化）
date: 2020-04-04 20:34:31
index_img: /img/368-3682760_c-language-hd-png-download.png.jpeg
banner_img: /img/1200px-ISO_C++_Logo.svg.png
tags: [C++,牛课网]
categories: 实习
---

### 数值的整数次方<!-- more -->

#### 题目描述

> 给定一个double类型的浮点数base和int类型的整数exponent。求base的exponent次方。
>
> 保证base和exponent不同时为0

#### 思路

需要用到快速幂的思想，但是首先要把系数的正负数确定下来，最后返回数值时再进行操作

快速幂公式(时间复杂度Log（n），连乘时间复杂度是n)：

![img](https://uploadfiles.nowcoder.com/images/20170907/3270776_1504748011304_B16239181123036ECE974C89CC410F1F)

#### 代码

```c++
class Solution {
    double quickPower(double base, int exp){
        if(exp==1){
            return base;
        }else if((exp&1)==0){
            double tem = quickPower(base,exp>>1);//exp>>1即exp/2
            return tem*tem;
        }else{
            double tem = quickPower(base,exp>>1);
            return tem*tem*base;
        }
    }
public:
    double Power(double base, int exponent) {
        double res = 1.0;
        int abs;
        if(exponent>0){
            abs = exponent;
        }else if(exponent==0){
            return 1;
        }else{
            if(base<0)
                return 0; 
            abs = -exponent;
        }
        res = quickPower(base, abs);
        return exponent>1 ? res:1/res;
    }
};
```

### 调整数组顺序使奇数位于偶数之前

#### 题目描述

> 输入一个整数数组，实现一个函数来调整该数组中数字的顺序，使得所有的奇数位于数组的前半部分，所有的偶数位于数组的后半部分，并保证奇数和奇数，偶数和偶数之间的相对位置不变。

#### 思路

设定两个指向下标的类似双指针的变量 i，j负责遍历一遍数组。i负责正常遍历，j负责应对连续出现偶数的情况下可以一次性后移这连续出现的多个偶数。

#### 代码

```c++
class Solution {
public:
void reOrderArray(vector<int> &array) {
        int len = array.size();
        if(len <= 1){ // 数组空或长度为1
            return;
        }
        int i = 0;
        while(i < len){
            int j = i + 1;
            if(array[i]%2 == 0){ // a[i]为偶数，j前进，直到替换
                while(array[j]%2 == 0){ // j为偶数，前进
                    if(j==len-1)// i为偶数，j也为偶数，一直后移到了末尾，证明后面都是偶数
                         return;
                    j++;
                }
                // 此时j为奇数
                int tem = array[j];//把j遇到的奇数保存下来，以备之后前插
                for(int k=j;k>i;k--){//j扫描到的偶数全部后移
                   array[k] = array[k-1];
                }
                array[i] = tem;
            }
            i++;
        }
    }
};
```

### 链表中倒数第k个元素

#### 题目描述

> 输入一个链表，输出该链表中倒数第k个结点。

#### 思路

用类似标尺的方法，两个指针，其中一个先走k-1格（注意是k-1），之后再同步向后走

#### 代码

```c++
class Solution {
public:
    ListNode* FindKthToTail(ListNode* pListHead, unsigned int k) {
        if(!pListHead||k<=0){
            return NULL;
        }
        ListNode* pre,* end;
        pre = pListHead;
        end = pre;
        for(int i=0;i<k-1;i++){//注意此处是k-1，两者相对距离是k-1
            if(!end->next)
                return NULL;
            end = end->next;
        }
        while(end->next){
            end = end->next;
            pre = pre->next;
        }
        return pre;
    }
};
```

### 反转链表

#### 题目描述

> 输入一个链表，反转链表后，输出新链表的表头。

#### 思路

需要先设计好三个指针，pre、root、next，其中root是要反转的链表的表头，pre和next用于之后链表反转。首先把pre置NULL，next置为root->next，之后将root->next赋值pre，pre赋值root，再将next->next置为root，最后将root赋值next，一个流程就走完了，之后循环遍历于整个链表就可以了。

#### 代码

```c++
class Solution {
public:
    ListNode* ReverseList(ListNode* pHead) {
        ListNode* pre = NULL;
        ListNode* next = NULL;
        while(pHead){
            next = pHead->next;//逻辑有点复杂，建议画图理一下，本质上就是三个指针交换指向
            pHead->next = pre;
            pre = pHead;
            pHead = next;
        }
        return pre;//一定返回pre，因为之前pHead已经被赋值了最后一个next，也就是NULL，而此时最后一个pHead已经赋给了pre
    }
};
```

### 合并两个排序的链表

#### 题目描述

> 输入两个单调递增的链表，输出两个链表合成后的链表，当然我们需要合成后的链表满足单调不减规则。

#### 思路

思路最重要的是创建一个新节点当头节点，之后两个链表哪个当前元素小哪个插到头节点之后，最后返回头节点的next就行。

#### 代码

```c++
class Solution {
public:
    ListNode* Merge(ListNode* pHead1, ListNode* pHead2)
    {
        ListNode* head=new ListNode(-1);
        head->next = NULL;
        ListNode* root = head;
        ListNode* tem = root;
        if(!pHead1)
            return pHead2;
        if(!pHead2)
            return pHead1;
        while(pHead1&&pHead2){
            if(pHead1->val <= pHead2->val){
                tem->next = pHead1;
                pHead1 = pHead1->next;
                tem = tem->next;
            }else{
                tem->next = pHead2;
                pHead2 = pHead2->next;
                tem = tem->next;
            }
        }
        if(pHead1){
            tem->next=pHead1;
        }
        if(pHead2){
            tem->next=pHead2;
        }
        return root->next;
    }
};
```

### 树的字结构

#### 题目描述

> 输入两棵二叉树A，B，判断B是不是A的子结构。（ps：我们约定空树不是任意一个树的子结构）

#### 思路

要查找树A中是否存在和树B结构一样的子树，可以分成两步：

第一步在树A中找到和B的根节点的值一样的结点R；
第二步再判断树A中以R为根结点的子树是不是包含和树B一样的结构。

第一步在树A中查找与根结点的值一样的结点，这实际上就是树的遍历。递归调用HasSubTree遍历二叉树A。
如果发现某一结点的值和树B的头结点的值相同，则调用DoesTreeHavaTree2，做第二步判断。
第二步是判断树A中以R为根结点的子树是不是和树B具有相同的结构。

#### 代码

```c++
class Solution {
public:
    bool HasSubtree(TreeNode* pRoot1, TreeNode* pRoot2){
        bool result = false;
        if(pRoot1 != NULL && pRoot2 != NULL)
        {
            if(pRoot1->val == pRoot2->val)
                result = DoesTree1HaveTree2(pRoot1, pRoot2);
            if(!result)
                result = HasSubtree(pRoot1->left, pRoot2);
            if(!result)
                result = HasSubtree(pRoot1->right, pRoot2);//注意这种调用方式，直接return DoesTree1HaveTree2(pRoot1, pRoot2)结果失败
            }
        return result;
    }
    bool DoesTree1HaveTree2(TreeNode* pRoot1, TreeNode* pRoot2){
        if(pRoot2 == NULL)//如果B已经遍历完了，也没有出现false的情况，就是true
            return true;
        if(pRoot1 == NULL)//如果A已经遍历完了，但是B还没有，则说明A没有B中的所有元素
            return false;
        if(pRoot1->val != pRoot2->val)
            return false;
        return DoesTree1HaveTree2(pRoot1->left, pRoot2->left) &&
            DoesTree1HaveTree2(pRoot1->right, pRoot2->right);
    }
};
```

### 二叉树的镜像

#### 题目描述

> 操作给定的二叉树，将其变换为源二叉树的镜像。

#### 输入描述

>```
>二叉树的镜像定义：源二叉树 
>	    8
>	   /  \
>	  6   10
>	 / \  / \
>	5  7 9 11
>	镜像二叉树
>	    8
>	   /  \
>	  10   6
>	 / \  / \
>	11 9 7  5
>```

#### 思路

直接按照递归的方式就行了，针对每个节点，将其左子树和右子树交换

#### 代码

```c++
class Solution {
public:
    void Mirror(TreeNode *pRoot) {
        if(!pRoot){
            return;
        }
        TreeNode* tem = pRoot->left;
        pRoot->left = pRoot->right;
        pRoot->right = tem;
        Mirror(pRoot->left);
        Mirror(pRoot->right);
    }
};
```

### 顺时针打印矩阵

#### 题目描述

> 输入一个矩阵，按照从外向里以顺时针的顺序依次打印出每一个数字，例如，如果输入如下4 X 4矩阵： 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 则依次打印出数字1,2,3,4,8,12,16,15,14,13,9,5,6,7,11,10.

#### 思路

设置四个标记，作为指向四个角的指针，然后再按相对位置，从左到右，从上到下，从右到左，从下到上把元素依次放入vector，注意一些特殊情况，比如只有一行，只有一列，都要判断一下。

#### 代码

```c++
class Solution {
public:
    vector<int> printMatrix(vector<vector<int> > matrix) {
        int row = matrix.size();
        int col = matrix[0].size();
        vector<int> res;
        if(row==0||col==0){
            return res;
        }
        int left=0,top=0,right=col-1,bottom=row-1;
        while(left<=right&&top<=bottom){
            for(int i=left;i<=right;i++) res.push_back(matrix[top][i]);
            for(int i=top+1;i<=bottom;i++) res.push_back(matrix[i][right]);
            if(top!=bottom){//防止重复放入
                for(int i=right-1;i>=left;i--) res.push_back(matrix[bottom][i]);
            }
            if(left!=right){//防止重复放入
                for(int i=bottom-1;i>=top+1;i--) res.push_back(matrix[i][left]);
            }
            left++,right--,top++,bottom--;
        }
        return res;
    }
};
```

