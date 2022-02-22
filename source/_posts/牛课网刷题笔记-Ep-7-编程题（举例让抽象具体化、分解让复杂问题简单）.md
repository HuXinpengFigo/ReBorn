---
title: 牛课网刷题笔记-Ep-7-编程题（举例让抽象具体化、分解让复杂问题简单）
date: 2020-04-06 22:42:03
index_img: /img/Xnip2020-02-05_19-55-51.png
banner_img: /img/1200px-ISO_C++_Logo.svg.png
tags: [C++,牛课网]
categories: 实习
---

### 包含min函数的栈<!-- more -->

#### 题目描述

> 定义栈的数据结构，请在该类型中实现一个能够得到栈中所含最小元素的min函数（时间复杂度应为O（1））。
>
> 注意：保证测试中不会当栈为空的时候，对栈调用pop()或者min()或者top()方法。

#### 思路

写到时间复杂度为O（1），但是没具体要求空间复杂度，那么就可以用两个栈，辅助栈为局部最小值（比如第一个元素之间入栈，第二个元素如果大于第一个元素，也就是大于辅助栈的top，则入辅助栈）。主栈出栈的时候，判断一下当前要出栈的元素是不是辅助栈的top，如果是的话两个栈一起出栈，不是的话只针对主栈出栈。

#### 代码

```c++
class Solution {
public:
    stack<int> A,B;//B为辅助栈
    void push(int value) {
        if(B.empty()){
            B.push(value);
            A.push(value);
        }
        if(value > B.top()){
            A.push(value);
        }else{
            A.push(value);
            B.push(value);
        }
    }
    void pop() {
        if(A.top()==B.top()){
            A.pop();
            B.pop();
        }else{
            A.pop();
        }
    }
    int top() {
        return A.top();
    }
    int min() {
        return B.top();
    }
};
```

### 栈的压入，弹出序列

#### 题目描述

> 输入两个整数序列，第一个序列表示栈的压入顺序，请判断第二个序列是否可能为该栈的弹出顺序。假设压入栈的所有数字均不相等。例如序列1,2,3,4,5是某栈的压入顺序，序列4,5,3,2,1是该压栈序列对应的一个弹出序列，但4,3,5,1,2就不可能是该压栈序列的弹出序列。（注意：这两个序列的长度是相等的）

#### 思路

需要针对给的弹出序列，一个一个来判断。

借用一个辅助的栈，遍历压栈顺序，先将第一个放入栈中，这里是1，然后判断栈顶元素是不是出栈顺序的第一个元素，这里是4，很显然1≠4，所以我们继续压栈，直到相等以后开始出栈，出栈一个元素，则将出栈顺序向后移动一位，直到不相等，这样循环等压栈顺序遍历完成，如果辅助栈还不为空，说明弹出序列不是该栈的弹出顺序。

#### 代码

```c++
class Solution {
public:
    bool IsPopOrder(vector<int> pushV,vector<int> popV) {
        stack<int> tem;
        if(pushV.empty()){
            return false;
        }
        int j = 0;
        for(int i = 0;i<pushV.size();i++){
            tem.push(pushV[i]);
            while(j<popV.size() && popV[j]==tem.top()){
                tem.pop();
                j++;
            }
        }
        return tem.empty();
    }
};
```

### 从上往下打印二叉树

#### 题目描述

> 从上往下打印出二叉树的每个节点，同层节点从左至右打印。

#### 思路

层序遍历的思想，有一个vector和一个queue足矣。

#### 代码

```c++
class Solution {
public:
    vector<int> PrintFromTopToBottom(TreeNode* root) {
        vector<int> res;
        queue<TreeNode*> que;
        if(!root)
            return res;
        que.push(root);
        while(!que.empty()){
            res.push_back(que.front()->val);
            if(que.front()->left) que.push(que.front()->left);
            if(que.front()->right) que.push(que.front()->right);
            que.pop();
        }
        return res;
    }
};
```

#### 思路

针对给出遍历序列，来判断二叉树是否符合标准的，肯定都需要按下标来分成根节点、左子树、右子树，之后再递归地对其进行更小维度的判断。

根据二叉搜索树的后序遍历的性质可得，最后一个节点是根节点，然后之前的节点可以分为两个部分
左子树和右子树，分别在左边一段和右边一段（除根节点），且对于每一小段，也可以按这个性质来判断于是我们按照递归的思想分段，只要有一段不符合，就是false，从头到尾都符合，则true。

#### 代码

```c++
class Solution {
public:
    bool judge(vector<int> sequence,int l,int r){
        if(l >= r)
            return true;
        int i = r;//设置指向左右子树分割点的i
        while(i>l && sequence[i-1]>sequence[r])//把这一段大于根节点的都归类于右子树
            --i;
        for(int k=l;k<i;k++)//检查是否左子树都小于根节点
            if(sequence[k]>sequence[r])
                return false;
        return judge(sequence, l, i-1)&&judge(sequence, i,r-1);//注意此处的r-1，因为r已经是根节点了，所以不能放进去继续递归
    }
    bool VerifySquenceOfBST(vector<int> sequence) {
        if(sequence.empty())
            return false;
        return judge(sequence, 0, sequence.size()-1);
    }
};
```

### 二叉树中和为某一值的元素

#### 题目描述

> 输入一颗二叉树的根节点和一个整数，打印出二叉树中结点值的和为输入整数的所有路径。路径定义为从树的根结点开始往下一直到叶结点所经过的结点形成一条路径。(注意: 在返回值的list中，数组长度大的数组靠前)

#### 思路

相当于带记忆的DFS，利用一个array进行每条路径的遍历，大小合适成功了则记录到arrayAll中，失败了则将array回溯到上一个节点，继续尝试。

#### 代码

```c++
class Solution {
public:
    vector<vector<int>> arrayAll;
    vector<int> array;
    
    vector<vector<int> > FindPath(TreeNode* root,int expectNumber) {
        if(!root)
            return arrayAll;
        array.push_back(root->val);
        expectNumber -= root->val;
        if(expectNumber == 0 && !root->left && !root->right)
            arrayAll.push_back(array);
        FindPath(root->left,expectNumber);//不用担心root->left为NULL有什么影响，并没有利用到它的返回值
        FindPath(root->right,expectNumber);
        array.pop_back();//回溯array,去掉尾元素
        return arrayAll;
    }
};
```

### 复杂链表的复制

#### 题目描述

> 输入一个复杂链表（每个节点中有节点值，以及两个指针，一个指向下一个节点，另一个特殊指针指向任意一个节点），返回结果为复制后复杂链表的head。（注意，输出结果中请不要返回参数中的节点引用，否则判题程序会直接返回空）

#### 思路

解题思路：
1、遍历链表，复制每个结点，如复制结点A得到A1，将结点A1插到结点A后面；如A->B->C变成A->A1->B->B1->C->C1
2、重新遍历链表，复制老结点的随机指针给新结点，如A1->random = A->random->next;注意此处是random->next,不是直接复制random
3、拆分链表，将链表拆分为原链表和复制后的链表

#### 代码

```c++
class Solution {
public:

    RandomListNode* Clone(RandomListNode* pHead)
    {
        if(!pHead) return NULL;
        //复制
        RandomListNode *currNode = pHead;
        while(currNode){
            RandomListNode *node = new RandomListNode(currNode->label);
            node->next = currNode->next;
            currNode->next = node;
            currNode = node->next;
        }
        //设定好random
        currNode = pHead;
        while(currNode){
            RandomListNode *node = currNode->next;
            if(currNode->random){               
                node->random = currNode->random->next;
            }
            currNode = node->next;
        }
        //拆分
        RandomListNode *pCloneHead = pHead->next;
        RandomListNode *tmp;
        currNode = pHead;
        while(currNode->next){
            tmp = currNode->next;
            currNode->next =tmp->next;
            currNode = tmp;
        }
        return pCloneHead;
    }
};
```

### 二叉搜索树于双向链表

#### 题目描述

> 输入一棵二叉搜索树，将该二叉搜索树转换成一个排序的双向链表。要求不能创建任何新的结点，只能调整树中结点指针的指向。

#### 思路

利用中序遍历的思想，主要是连接好左右子树使其不断链

#### 代码

```c++
class Solution {
public:
    void ConvertHelper(TreeNode* root,TreeNode*& pre){//注意这里的pre传的是指针引用，不传引用的话难以改变指针
                                                    //只能输出三个（root,root->left,root->right）
        if(!root)
            return;
        ConvertHelper(root->left, pre);//先到达二叉搜索树的最左边
        
        root->left = pre;//重点，连接起子树左右两边
        if(pre)
            pre->right = root;
        pre = root;
        
        ConvertHelper(root->right, pre);
    }
    TreeNode* Convert(TreeNode* pRootOfTree)
    {
        if(!pRootOfTree)
            return NULL;
        TreeNode *root = pRootOfTree,*pre = NULL;
        ConvertHelper(pRootOfTree, pre);
        
        TreeNode *res = pRootOfTree;
        while(res->left)//取最左边节点，那是链表的头节点
            res = res->left;
        return res;
    }
};
```

### 字符串的排列

#### 题目描述

> 输入一个字符串,按字典序打印出该字符串中字符的所有排列。例如输入字符串abc,则打印出由字符a,b,c所能排列出来的所有字符串abc,acb,bac,bca,cab和cba。

#### 输入描述:

```
输入一个字符串,长度不超过9(可能有字符重复),字符只包括大小写字母。
```

#### 思路

先把字符串sort一下（达到字典序），然后直接使用stl里面的全排列函数`next_permutation(begin,end)`

#### 代码

```c++
class Solution {

public:
    vector<string> Permutation(string str) {
        if (str.empty()) return {};
        sort(str.begin(), str.end());
        vector<string> ans;
        ans.push_back(str);
        while (next_permutation(str.begin(), str.end()))
            ans.push_back(str);
        return ans;
    }
    
};
```

