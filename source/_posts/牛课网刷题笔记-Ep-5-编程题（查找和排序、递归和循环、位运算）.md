---
title: 牛课网刷题笔记-Ep-5-编程题（查找和排序、递归和循环、位运算）
date: 2020-04-04 16:41:04
index_img: /img/Xnip2020-02-05_19-55-51.png
banner_img: /img/1200px-ISO_C++_Logo.svg.png
tags: [C++,牛课网]
categories: 实习
---

### 旋转数组的最小数字<!-- more -->

#### 题目描述

> 把一个数组最开始的若干个元素搬到数组的末尾，我们称之为数组的旋转。
> 输入一个非递减排序的数组的一个旋转，输出旋转数组的最小元素。
> 例如数组{3,4,5,1,2}为{1,2,3,4,5}的一个旋转，该数组的最小值为1。
> NOTE：给出的所有元素都大于0，若数组大小为0，请返回0。

#### 思路

利用二分查找算法的思想，结合该数组为一个非递减的数组的旋转，来进行查询。用mid和high来进行比较：

- mid>high 说明mid是未旋转的那部分的一个元素，最小值肯定在右边，则low = mid+1；
- mid==high 难以判断，但是最小值肯定在high之前，high = high-1;
- mid<high 说明最小值在mid之前（并不能看出来是否旋转），high = mid-1.

#### 代码

```c++
class Solution {
public:
    int minNumberInRotateArray(vector<int> rotateArray) {
        if(rotateArray.size()==0){
            return 0;
        }
        int low = 0, high = rotateArray.size()-1;
        while(low < high){
            int mid = (low+high)/2;
            if(rotateArray[mid]>rotateArray[high]){//mid>high(元素值),说明最小值一定在右边
                low = mid+1;
            }else if (rotateArray[mid]==rotateArray[high]){//mid==high(元素值)，难以判断，但是肯定在high之前能找到小于等于的元素
                high = high-1;
            }else{//mid<high(元素值)，说明最小值在左边，考虑只有两个元素的情况，用high=mid而不是high=mid-1
                high = mid;
            }
        }
        return rotateArray[low];
    }
};
```

### 斐波那契数列

#### 题目描述

> 大家都知道斐波那契数列，现在要求输入一个整数n，请你输出斐波那契数列的第n项（从0开始，第0项为0）。
>
> n<=39

#### 思路

先设定好第0号、第1号元素的结果，之后按照利用递归或者循环就行，每个元素等于其前两个数据之和

#### 代码

循环：

```c++
//用循环，不要用递归
class Solution {
public:
    int Fibonacci(int n) {
        if(n==0)
            return 0;
        if(n==1)
            return 1;
        int first = 0,second = 1;
        int tem;
        for(int i=2;i<=n;i++){//核心，要从i=2开始,之前的0，1是直接返回
            tem = first+second;
            first = second;
            second = tem;
        }
        return tem;
    }
};
```

递归：

```c++
public class Solution {
    public int Fibonacci(int n) {
        if(n<=1){
            return n;
        }
        return Fibonacci(n-1) + Fibonacci(n-2);
    }
}
```

### 跳台阶

#### 题目描述

> 一只青蛙一次可以跳上1级台阶，也可以跳上2级。求该青蛙跳上一个n级的台阶总共有多少种跳法（先后次序不同算不同的结果）。

#### 思路

对于本题,前提只有1阶或者2阶的跳法。
a.如果两种跳法，1阶或者2阶，那么假定第一次跳的是一阶，那么剩下的是n-1个台阶，跳法是f(n-1);
b.假定第一次跳的是2阶，那么剩下的是n-2个台阶，跳法是f(n-2)
c.由a\b假设可以得出总跳法为: f(n) = f(n-1) + f(n-2) 
d.然后通过实际的情况可以得出：只有一阶的时候 f(1) = 1 ,只有两阶的时候可以有 f(2) = 2
e.可以发现最终得出的是一个斐波那契数列：
           | 1, (n=1)
f(n) =  | 2, (n=2)
           | f(n-1)+f(n-2) ,(n>2,n为整数)

#### 代码

```c++
class Solution {
public:
    int jumpFloor(int number) {
        if(number == 1)
            return 1;
        if(number == 2)
            return 2;
        int first=1,second=2;
        int tem;
        for(int i=3;i<=number;i++){
            tem = first+second;
            first = second;
            second = tem;
        }
        return tem;
    }
};
```

### 变态跳台阶

#### 题目描述

> 一只青蛙一次可以跳上1级台阶，也可以跳上2级……它也可以跳上n级。求该青蛙跳上一个n级的台阶总共有多少种跳法。

#### 思路

普通跳台阶是斐波那契数列，变态跳台阶是利用告诉里面的无穷数列的思想把累加化简为公式

f(n-1) = f(0) + f(1)+f(2)+f(3) + ... + f((n-1)-1) = f(0) + f(1) + f(2) + f(3) + ... + f(n-2)
    f(n) = f(0) + f(1) + f(2) + f(3) + ... + f(n-2) + f(n-1) 

​           = f(n-1) + f(n-1) 

​           = 2 f(n-1)

#### 代码

```c++
class Solution {
public:
    int jumpFloorII(int number) {
        if(number ==1)
            return 1;
        if(number ==2)
            return 2;
        int res=1;
        for(int i=1;i<number;i++){
            res *= 2;
        }
        return res;
    }
};
```

#### 题目描述

> 我们可以用2*1的小矩形横着或者竖着去覆盖更大的矩形。请问用n个2*1的小矩形无重叠地覆盖一个2*n的大矩形，总共有多少种方法？
>
> 比如n=3时，2*3的矩形块有3种覆盖方法：
>
> ![img](https://uploadfiles.nowcoder.com/images/20200218/6384065_1581999858239_64E40A35BE277D7E7C87D4DCF588BE84)

#### 思路

找规律，多画几种情况，最后发现还是斐波那契数列的延展
f(n) = f(n-1) + f(n-2)， (n > 2);
如果是其它情况：
（1）1 * 3方块 覆 盖3*n区域：f(n) = f(n-1) + f(n - 3)， (n > 3)
（2） 1 *4 方块 覆 盖4*n区域：f(n) = f(n-1) + f(n - 4)，(n > 4)
最合得出接论：
如果用1*m的方块覆盖m*n区域，递推关系式为f(n) = f(n-1) + f(n-m)，(n > m)

#### 代码

```c++
class Solution {
public:
    int rectCover(int number) {
        if(number == 0)
            return 0;
        if(number == 1)
            return 1;
        if(number ==2)
            return 2;
        int first=1,second=2;
        int tem;
        for(int i=3;i<=number;i++){
            tem = first+second;
            first = second;
            second = tem;
        }
        return tem;
    }
};
```

### 二进制中1的个数

#### 题目描述

> 输入一个整数，输出该数二进制表示中1的个数。其中负数用补码表示。

#### 思路

简单题，直接&运算就可以了

#### 代码

```c++
class Solution {
public:
     int  NumberOf1(int n) {
         int flag = 1;
         int count = 0;
         while (flag!=0){//因为计算机中存储字长有限，flag一直左移，补的是0，最后flag会变为全0
             if(flag&n)//flag与n的每位都与运算一下，可以得到每位是不是1
                 count++;
             flag = flag << 1;
         }
         return count;
     }
};
```

