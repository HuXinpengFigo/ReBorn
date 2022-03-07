---
title: 三种方法（递归、迭代、Morris Traversal）遍历二叉树
date: 2019-11-23 14:13:17
index_img: /img/WALLHAVEN-392LJV.jpeg
banner_img: /img/WALLHAVEN-392LJV.jpeg
tags: [LeetCode,算法,C++]
categories: 代码
---

### Morris Traversal

* 非递归，不用栈，空间O(1)时间O(n)
* 二叉树的形状不能被破坏（中间过程允许改变形状）
* Morris Traveral只提供了中序遍历的算法，在中序遍历的基础上稍加修改可以实现前序，而后序就要再费点心思了。

  <!-- more -->

> 二叉树结构：
>
> ```c++
> struct TreeNode {
>   int val;
>   TreeNode *left;
>   TreeNode *right;
>   TreeNode(int x) : val(x), left(NULL), right(NULL) {}
> };
> ```

#### 中序遍历步骤：

> 1. 如果当前节点的左孩子为空，则输出当前节点并将其右孩子作为当前节点。
>
> 2. 如果当前节点的左孩子不为空，在当前节点的左子树中找到当前节点在中序遍历下的前驱节点。
>
>    a) 如果前驱节点的右孩子为空，将它的右孩子设置为当前节点。当前节点更新为当前节点的左孩子。
>
>    b) 如果前驱节点的右孩子为当前节点，将它的右孩子重新设为空（恢复树的形状）。输出当前节点。当前节点更新为当前节点的右孩子。
>
> 3. 重复以上1、2直到当前节点为空。

##### 例如二叉树：

```
       4
     /   \
    2     6
   / \   / \
  1   3 5   7
```

> 寻找前驱的过程中： 
> 节点 3 的右孩子应该指向节点 4； 
> 节点 1 的右孩子应该指向节点 2； 
> 节点 5 的右孩子应该指向节点 6； 
> 在第二次访问的时候以上前驱的右孩子重新被置为NULL；

##### 图示：

> 下图为每一步迭代的结果（从左至右，从上到下），cur代表当前节点，深色节点表示该节点已输出
>
> ![tra](/img/tra.jpg)

##### 代码：

```c++
void inorderMorrisTraversal(TreeNode *root) {
    TreeNode *cur = root, *prev = NULL;
    while (cur != NULL)
    {
        if (cur->left == NULL)          // 1.
        {
            printf("%d ", cur->val);
            cur = cur->right;
        }
        else
        {
            // 寻找cur的前驱（指向其的线索）
            prev = cur->left;
            while (prev->right != NULL && prev->right != cur)
                prev = prev->right;

            if (prev->right == NULL)   // 2.a)
            {
                prev->right = cur; //设置右孩子线索
                cur = cur->left;
            }
            else                       // 2.b)
            {
                prev->right = NULL; //线索用过了，重设
                printf("%d ", cur->val);
                cur = cur->right;
            }
        }
    }
}
```

**复杂度分析：**

>  空间复杂度：O(1)，因为只用了两个辅助指针。

> 时间复杂度：O(n)。证明时间复杂度为O(n)，最大的疑惑在于寻找中序遍历下二叉树中所有节点的前驱节点的时间复杂度是多少，即以下两行代码：

```c++
while (prev->right != NULL && prev->right != cur)
    prev = prev->right;
```

> 直觉上，认为它的复杂度是O(nlgn)，因为找单个节点的前驱节点与树的高度有关。但事实上，寻找所有节点的前驱节点只需要O(n)时间。n个节点的二叉树中一共有n-1条边，整个过程中每条边最多只走2次，一次是为了定位到某个节点，另一次是为了寻找上面某个节点的前驱节点，如下图所示，其中**红色**是为了定位到某个节点，**黑色**线是为了找到前驱节点。所以复杂度为O(n)。
>
> ![](/img/15150628-5285f29bab234750a62e2309394b6e14.jpg)

#### 前序遍历

前序遍历与中序遍历相似，代码上只有一行不同，不同就在于输出的顺序。

**步骤：**

> 1. 如果当前节点的左孩子为空，则输出当前节点并将其右孩子作为当前节点。
> 2. 如果当前节点的左孩子不为空，在当前节点的左子树中找到当前节点在中序遍历下的前驱节点。
>    a) 如果前驱节点的右孩子为空，将它的右孩子设置为当前节点。**输出当前节点（在这里输出，这是与中序遍历唯一一点不同）。**当前节点更新为当前节点的左孩子。
>    b) 如果前驱节点的右孩子为当前节点，将它的右孩子重新设为空。当前节点更新为当前节点的右孩子。
> 3. 重复以上1、2直到当前节点为空。

**图示：**

> ![](/img/zx.jpg)

##### 代码：

```c++
void preorderMorrisTraversal(TreeNode *root) {
    TreeNode *cur = root, *prev = NULL;
    while (cur != NULL)
    {
        if (cur->left == NULL)
        {
            printf("%d ", cur->val);
            cur = cur->right;
        }
        else
        {
            prev = cur->left;
            while (prev->right != NULL && prev->right != cur)
                prev = prev->right;

            if (prev->right == NULL)
            {
                printf("%d ", cur->val);  // the only difference with inorder-traversal
                prev->right = cur;
                cur = cur->left;
            }
            else
            {
                prev->right = NULL;
                cur = cur->right;
            }
        }
    }
}
```

##### 复杂度分析：

> 时间复杂度与空间复杂度都与中序遍历时的情况相同。

#### 后续遍历

> 后续遍历稍显复杂，需要建立一个临时节点dump，令其左孩子是root。并且还需要一个子过程，就是倒序输出某两个节点之间路径上的各个节点。

**步骤：**

> 当前节点设置为临时节点dump。
>
> 1.  如果当前节点的左孩子为空，则将其右孩子作为当前节点。
> 2.  如果当前节点的左孩子不为空，在当前节点的左子树中找到当前节点在中序遍历下的前驱节点。
>     a) 如果前驱节点的右孩子为空，将它的右孩子设置为当前节点。当前节点更新为当前节点的左孩子。
>     b) 如果前驱节点的右孩子为当前节点，将它的右孩子重新设为空。**倒序输出从当前节点的左孩子到该前驱节点这条路径上的所有节点。**当前节点更新为当前节点的右孩子。
> 3.  重复以上1、2直到当前节点为空。

##### 图示：

> ![](/img/hx.jpg)

##### 代码

```c++
void reverse(TreeNode *from, TreeNode *to) // reverse the tree nodes 'from' -> 'to'.
{
    if (from == to)
        return;
    TreeNode *x = from, *y = from->right, *z;
    while (true)
    {
        z = y->right;
        y->right = x;
        x = y;
        y = z;
        if (x == to)
            break;
    }
}

void printReverse(TreeNode* from, TreeNode *to) // print the reversed tree nodes 'from' -> 'to'.
{
    reverse(from, to);
    
    TreeNode *p = to;
    while (true)
    {
        printf("%d ", p->val);
        if (p == from)
            break;
        p = p->right;
    }
    
    reverse(to, from);
}

void postorderMorrisTraversal(TreeNode *root) {
    TreeNode dump(0);
    dump.left = root;
    TreeNode *cur = &dump, *prev = NULL;
    while (cur)
    {
        if (cur->left == NULL)
        {
            cur = cur->right;
        }
        else
        {
            prev = cur->left;
            while (prev->right != NULL && prev->right != cur)
                prev = prev->right;

            if (prev->right == NULL)
            {
                prev->right = cur;
                cur = cur->left;
            }
            else
            {
                printReverse(cur->left, prev);  // call print
                prev->right = NULL;
                cur = cur->right;
            }
        }
    }
}	
```

##### 复杂度分析：

> 空间复杂度同样是O(1)；时间复杂度也是O(n)，倒序输出过程只不过是加大了常数系数。

----

鸣谢与参考：

​	[AnnieKim](http://www.cnblogs.com/AnnieKim/archive/2013/06/15/MorrisTraversal.html)

----

### 递归

##### 代码：

> 中序遍历（前序、后序遍历可以在此基础上更改）

```c++
class Solution {
public:
    vector<int> inorderTraversal(TreeNode* root) {
        vector<int> nodes;
        inorder(root, nodes);
        return nodes;
    }
private:
    void inorder(TreeNode* root, vector<int>& nodes) {
        if (!root) {
            return;
        }
        inorder(root -> left, nodes);
        nodes.push_back(root -> val);
        inorder(root -> right, nodes);
    }
};
```

#### 迭代（栈+迭代）

##### 代码

> 中序遍历（前序、后序遍历可以在此基础上更改）

```c++
class Solution {
public:
    vector<int> inorderTraversal(TreeNode* root) {
        vector<int> nodes;
        stack<TreeNode*> todo;
        while (root || !todo.empty()) {
            while (root) {
                todo.push(root);
                root = root -> left;
            }
            root = todo.top();
            todo.pop();
            nodes.push_back(root -> val);
            root = root -> right;
        }
        return nodes;
    }
};
```



