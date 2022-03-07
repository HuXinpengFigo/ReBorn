---
title: 牛课网刷题笔记-Ep-4-编程题（树、栈和队列）
date: 2020-04-04 10:09:12
index_img: /img/368-3682760_c-language-hd-png-download.png.jpeg
banner_img: /img/1200px-ISO_C++_Logo.svg.png
tags: [C++,牛课网]
categories: 实习
---

### 重建二叉树<!-- more -->

#### 题目描述

> 输入某二叉树的前序遍历和中序遍历的结果，请重建出该二叉树。假设输入的前序遍历和中序遍历的结果中都不含重复的数字。例如输入前序遍历序列{1,2,4,7,3,5,6,8}和中序遍历序列{4,7,2,1,5,3,8,6}，则重建二叉树并返回。

#### 思路

利用分治的思想，联合前序序列第一个在中序序列中分割开左右子树的性质，设定四个vector来保存每次分割后剩下的先序序列和中序序列的左右两部分，之后再把四个vector按照左右分好，递归针对左右继续分割。

#### 代码

```c++
class Solution {
public:
    TreeNode* reConstructBinaryTree(vector<int> pre,vector<int> in) {
        if(pre.size()==0||in.size()==0)
            return NULL;
        vector<int> preleft, preright, inleft, inright;
        TreeNode* root = new TreeNode(pre[0]);
        int gen;
        for(int i = 0; i < in.size();i++){
            if(in[i] == pre[0]){
                gen = i; //找到前序首位在中序中的位置
                break;
            }
        }
        
        for(int i = 0; i < gen; i++){//分割头节点的左子树
            inleft.push_back(in[i]);
            preleft.push_back(pre[i+1]);//注意这里的+1，头节点不需要
        }
        for(int i = gen + 1; i < in.size(); i++){//分割头节点的右子树
            inright.push_back(in[i]);
            preright.push_back(pre[i]);
        }
        root->left = reConstructBinaryTree(preleft, inleft);//递归
        root->right = reConstructBinaryTree(preright, inright);
        return root;
    }
};
```

### 二叉树的下一个节点

#### 题目描述

> 给定一个二叉树和其中的一个结点，请找出中序遍历顺序的下一个结点并且返回。注意，树中的结点不仅包含左右子结点，同时包含指向父结点的指针。

#### 思路

三种情况，建议把图画下来：

1. 二叉树为空，则返回空；
2. 节点右孩子存在，则设置一个指针从该节点的右孩子出发，一直沿着指向左子结点的指针找到的叶子节点即为下一个节点；
3. 节点不是根节点。如果该节点是其父节点的左孩子，则返回父节点；否则继续向上遍历其父节点的父节点，重复之前的判断，返回结果。

#### 代码

```c++
class Solution {
public:
    TreeLinkNode* GetNext(TreeLinkNode* pNode)
    {
        if(!pNode)
            return NULL;
        if(pNode->right){
            TreeLinkNode* tem = pNode->right;
            while(tem->left){
                tem = tem->left;
            }
            return tem;
        }
        while(pNode->next){
            TreeLinkNode* roof = pNode->next;
            if(pNode==roof->left)
                return roof;
            else
                pNode = roof;
        }
        return NULL;
    }
};
```

### 对称的二叉树

#### 题目描述

> 请实现一个函数，用来判断一颗二叉树是不是对称的。注意，如果一个二叉树同此二叉树的镜像是同样的，定义其为对称的。

#### 思路

对于这种需要逐个判断的，首先要想到递归。然后开始考虑递归条件，从对称上来看，首先需要两边都有或者两边都没有（都是NULL或者都不是NULL），之后才看左右两边不是NULL的情况下值是否相等,相等的话就继续向下递归，知道全部都符合条件。

#### 代码

```c++
class Solution {
public:
    bool judge(TreeNode* pRoot1, TreeNode* pRoot2){
        if(!pRoot1&&!pRoot2){
            return true;
        }else if((!pRoot1&&pRoot2)||(pRoot1&&!pRoot2)){
            return false;
        }else{
            if(pRoot1->val == pRoot2->val){
                return judge(pRoot1->left,pRoot2->right)&&judge(pRoot2->left,pRoot1->right);
            }else{
                return false;
            }
        }
    }
    bool isSymmetrical(TreeNode* pRoot)
    {
      	if(!pRoot)
          return false;
        return judge(pRoot->left, pRoot->right);
    }

};
```

### 按之字形顺序打印二叉树

#### 题目描述

> 请实现一个函数按照之字形打印二叉树，即第一行按照从左到右的顺序打印，第二层按照从右至左的顺序打印，第三行按照从左到右的顺序打印，其他行以此类推。

#### 思路

利用层序遍历的思想，再在每层之间遍历的时候加入一个vector用来记录当层的元素值（利用一个for循环就可以把单层的元素直接遍历完，并且把当前元素的子节点也添加进入了queue），之后在每层最后判断当前层数的奇偶性，偶数就直接调用stl里面的reverse(begin(),end())翻转vector，然后再push_back进返回的二维vector

#### 代码

```c++
class Solution {
public:
    vector<vector<int> > Print(TreeNode* pRoot) {
        vector<vector<int>> ret;
        queue<TreeNode*> que;
        if(!pRoot){
            return ret;
        }
        que.push(pRoot);
        bool even = false;
        while(!que.empty()){
            vector<int> vec;
            int size = que.size();
            for(int i=0;i<size;i++){
                TreeNode* tem = que.front();
                que.pop();
                vec.push_back(tem->val);
                if(tem->left)
                    que.push(tem->left);
                if(tem->right)
                    que.push(tem->right);
            }
            if(even)
                std::reverse(vec.begin(),vec.end());
            ret.push_back(vec);
            even = !even;
        }
        return ret;
    }
};
```

### 把二叉树打印成多行

#### 题目描述

> 从上到下按层打印二叉树，同一层结点从左至右输出。每一层输出一行。

#### 思路

与上题基本相同，还少了一个翻转的过程，记住要点是while里面带一个for循环就可以了

#### 代码

```c++
class Solution {
public:
        vector<vector<int> > Print(TreeNode* pRoot) {
            vector<vector<int>> ret;
            if(!pRoot)
                return ret;
            queue<TreeNode*> que;
            que.push(pRoot);
            while(!que.empty()){
                vector<int> vec;
                int size = que.size();
                for(int i=0;i<size;i++){
                    TreeNode* tem = que.front();
                    que.pop();
                    vec.push_back(tem->val);
                    if(tem->left)
                        que.push(tem->left);
                    if(tem->right)
                        que.push(tem->right);
                }
                ret.push_back(vec);
            }
            return ret;
        }
};
```

### 序列化二叉树

#### 题目描述

> 请实现两个函数，分别用来序列化和反序列化二叉树
>
> 二叉树的序列化是指：把一棵二叉树按照某种遍历方式的结果以某种格式保存为字符串，从而使得内存中建立起来的二叉树可以持久保存。序列化可以基于先序、中序、后序、层序的二叉树遍历方式来进行修改，序列化的结果是一个字符串，序列化时通过 某种符号表示空节点（#），以 ！ 表示一个结点值的结束（value!）。
>
> 二叉树的反序列化是指：根据某种遍历顺序得到的序列化字符串结果str，重构二叉树。

#### 思路

序列化：

首先序列话就是先序遍历+字符串操作，遇到NULL就放#号进字符串，主要难点在于字符串操作。

反序列化：

将之前的string转换回二叉树，涉及二级指针（因为需要对字符串本身的指针和每个字符的指针进行操作），较为繁琐，还是使用先序序列的方法，针对一个字符元素能够处理完，之后就直接递归给其左子树右子树调用函数递归就行。

#### 代码

```c++
class Solution {
public:
char* Serialize(TreeNode *root) {
       if(root == NULL)
           return NULL;
        string str;
        Serialize(root, str);
        char *ret = new char[str.length() + 1];
        int i;
        for(i = 0; i < str.length(); i++){
            ret[i] = str[i];
        }
        ret[i] = '\0';
        return ret;
    }
    void Serialize(TreeNode *root, string& str){
        if(root == NULL){
            str += '#';
            return ;
        }
        string r = to_string(root->val);
        str += r;
        str += ',';
        Serialize(root->left, str);
        Serialize(root->right, str);
    }
     
    TreeNode* Deserialize(char *str) {
        if(str == NULL)
            return NULL;
        TreeNode *ret = Deserialize(&str);
 
        return ret;
    }
    TreeNode* Deserialize(char **str){//由于递归时，会不断的向后读取字符串
        if(**str == '#'){  //所以一定要用**str,
            ++(*str);         //以保证得到递归后指针str指向未被读取的字符
            return NULL;
        }
        int num = 0;
        while(**str != '\0' && **str != ','){
            num = num*10 + ((**str) - '0');
            ++(*str);
        }
        TreeNode *root = new TreeNode(num);
        if(**str == '\0')
            return root;
        else
            (*str)++;
        root->left = Deserialize(str);//左子树遍历完了自然下一个就是其右子树
        root->right = Deserialize(str);
        return root;
    }
    
};
```

### 二叉搜索树的第k个节点

#### 题目描述

> 给定一棵二叉搜索树，请找出其中的第k小的结点。例如， （5，3，7，2，4，6，8）  中，按结点数值大小顺序第三小结点的值为4。

#### 思路

利用二叉搜索树的性质，递归到最左边的元素，肯定是第一个元素，这个时候开始计数，每回溯一个节点计数++，直到计数大小与k相同，则该元素就是第k个节点

#### 代码

```c++
#include <stack>
class Solution {
public:
    TreeNode* KthNode(TreeNode* pRoot, int k)
    {
        if(!pRoot||k==0){
            return NULL;
        }
        int count = 0;
        TreeNode* node = pRoot;
        stack<TreeNode*> st;
        while(node||!st.empty()){
            if(node){
                st.push(node);
                node = node->left;
            }else{
                node = st.top();//左子树遍历完了，回溯
                st.pop();//注意要先取top()，再pop（）_
                count++;
                if(count == k){
                    return node;
                }
                node = node->right;
            }
        }
        return NULL;
    }
    
};
```

### 数据流中的中位数

#### 题目描述

> 如何得到一个数据流中的中位数？如果从数据流中读出奇数个数值，那么中位数就是所有数值排序之后位于中间的数值。如果从数据流中读出偶数个数值，那么中位数就是所有数值排序之后中间两个数的平均值。我们使用Insert()方法读取数据流，使用GetMedian()方法获取当前读取数据的中位数。

#### 思路

利用优先队列来创造一个大顶堆一个小顶堆，在插入的时候就进行控制。要快速得到数据流的中位数，要保证两个优先队列长度要么相等要么差距为1.利用大顶堆队头作为标志，如果小于大顶堆队队头，就将该元素插入大顶堆当中；如果大于大顶堆队头，则将元素插入小顶堆当中。

最后小顶堆的长度只能小于或等于大顶堆，不能大于大顶堆，出现超过的情况，哪个更多，就把哪个的top插入到另一个之中。

最后按照大顶堆和小顶堆的长度来判断返回两个top除以2或者大顶堆的top

#### 代码

```c++
#include <queue>
class Solution {
public:
    priority_queue<int, vector<int>, less<int>> p;//大顶堆
    priority_queue<int, vector<int>, greater<int>> q;//小顶堆
    
    void Insert(int num)
    {
        if(p.empty()||num<=p.top())
            p.push(num);
        else
            q.push(num);
        if(p.size() == q.size()+2){
            q.push(p.top());
            p.pop();
        }
        if(p.size()+1 == q.size()){
            p.push(q.top());
            q.pop();
        }
    }

    double GetMedian()
    { 
        if(p.size()==q.size())
            return (p.top()+q.top())/2.0;//注意此处一定要用2.0,不然结果是int，会把小数部分截去
        else
            return p.top();
    }
};
```

### 用两个栈实现一个队列

#### 题目描述

> 用两个栈来实现一个队列，完成队列的Push和Pop操作。 队列中的元素为int类型。

#### 思路

使用两个栈A、B，A是中间栈（push操作一律先插入A中），B是平时要pop的那个容器栈，当其不为空的时候pop操作一律使用B的top值，当B为空时，将A中之前push操作插入的元素全部出栈，加入B栈中。

入栈：将元素进栈A

出栈：判断栈A中元素的个数是否为1，如果等于1，则出栈，否则将栈A中的元素
出栈并放入栈B，直到栈A中的元素留下一个，然后栈A出栈，再把栈B中的元素出栈以此放入栈A中。

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

### 滑动窗口的最大值

#### 题目描述

> 给定一个数组和滑动窗口的大小，找出所有滑动窗口里数值的最大值。例如，如果输入数组{2,3,4,2,6,2,5,1}及滑动窗口的大小3，那么一共存在6个滑动窗口，他们的最大值分别为{4,4,6,6,6,5}； 针对数组{2,3,4,2,6,2,5,1}的滑动窗口有以下6个： {[2,3,4],2,6,2,5,1}， {2,[3,4,2],6,2,5,1}， {2,3,[4,2,6],2,5,1}， {2,3,4,[2,6,2],5,1}， {2,3,4,2,[6,2,5],1}， {2,3,4,2,6,[2,5,1]}。

#### 思路

本题两种方法：

1. 按照窗口大小，把每个窗口的最大值都算出来，如果窗口大小为n，数组大小为m则时间复杂度为O(mn)
2. 利用一个队列来存数组元素的下标，如果当前下标的元素大小大于队头，则将队头出栈（有几个更小的队头就出队几次，直到当前队头大于该下标元素），记录当前扫描到第几个元素了，要是大于等于窗口大小，说明可以开始记录窗口最大值了，将当前队头值记录到返回的result当中。且要记录当前队头是否不属于该窗口了（利用队头下标值和当前遍历的数组元素下标之差来确定），要是不属于了，就将队头出队。

#### 代码

1. 简单版：

```c++
int findMaxInWindow(vector<int> &num,int begin,int size){//寻找每个窗口最大值
    int max = 0;
    for(int i=begin;i<size+begin;i++){
        if(num[i]>max)
            max = num[i];
    }
    return max;
}

vector<int> maxInWindow(vector<int> & num,int size){//主函数
    vector<int> res;
    if(num.size()<size)
        return res;
    for(int i=0;i<num.size()-size;i++){
        res.push_back(findMaxInWindow(num,i,size));
    }
    return res;
}
```

2. 进阶版：

```c++
vector<int> maxInWindowsBetter(vector<int> &num, int size){
    vector<int> res;
    deque<int> s;//必须存的是num元素的下标，存num元素值之后会难以判断
    for(unsigned int i = 0; i < num.size(); ++i){
        while(!s.empty() && num[s.back()] <= num[i]) s.pop_back();//操作窗口右边

        if(!s.empty() && i - s.front() + 1 > size) s.pop_front();//操作窗口左边，注意第二个判断，重点。
        //解决了随着窗口右移判断是否要改变窗口最大值的问题。
        s.push_back(i);
        if(size && i + 1 >= size) res.push_back(num[s.front()]);
    }
    return res;
}
```

