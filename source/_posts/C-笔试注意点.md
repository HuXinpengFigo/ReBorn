---
title: C++ 笔试注意点
date: 2022-03-29 10:58:37
index_img: /img/LeetCode_Sharing.png
banner_img: /img/h4ear4i3g4q7r04utgpm.png
tags: [C++,LeetCode]
categories: 实习
---

### 头文件

万能头文件：

```c++
#include <bits/stdc++.h>
```

基本头文件：

```c++
#include <iostream>
#include <cstring>
#include <algorithm>
```

### 输入

普通输入：

```c++
cin >> a >> b;
```

string按行输入 (有空格)：

```c++
string str;
getline(cin,str);
```

char输入：

```c++
char a[20];
cin>>a;
```

### 输出

保留两位小数输出

#### c++:

```c++
cout<<fixed<<setprecision(2)<<a;
```

> 需要头文件 `#include <iomanip>`

#### c:

```c
printf("%.2f", a);
```

### 小数四舍五入

保留两位小数并且按四舍五入来

```c++
a = round(a*100)/100;
```

### vector利用set去重

```c++
//定义并初始化一个vector
vector<int> vec(10,1);      //vec里有10个值为1的元素
set<int> s(vec.begin(), vec.end());
vec.assign(s.begin(), s.end());
//完成去重
```

