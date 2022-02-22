---
title: C/C++函数指针与指针函数等区别
date: 2020-03-31 16:30:14
index_img: /img/IMG_0474-2.jpeg
banner_img: /img/IMG_0474-2.jpeg
tags: [C++]
categories: 代码
---

### （1）函数指针与指针函数

**（1）函数指针**

一种特殊的指针，它指向函数的入口；

<!-- more -->

***要声明指向特定类型的函数指针，可以首先编写这个函数的原型，然后用(\*p)来替换函数名，这样p就是这类函数的指针。***

```c++
/*
* 定义一个函数指针p，只能指向返回值为int，形参为两个int的函数
*/
int (*p)(int,int);


/* 
 * 求最大值 
 * 返回值是int类型，返回两个整数中较大的一个 
 */  
int max(int a, int b) {  
    return a > b ? a : b;  
}  
  
/* 
 * 求最小值 
 * 返回值是int类型，返回两个整数中较小的一个 
 */  
int min(int a, int b) {  
    return a < b ? a : b;  
}

int main(){
	f = max; // 函数指针f指向求最大值的函数max  
    int c = (*f)(1, 2);

	f = min; // 函数指针f指向求最小值的函数min  
    c = (*f)(1, 2); 
}
```

**（2）指针函数**

返回指针的函数，一个函数，它的返回值是指针；

```c++
//这是一个形参为两个int类型，返回值是int型指针的函数
int *p(int,int);

/* 
 * 指针函数的定义 
 * 返回值是指针类型int * 
 */  
int *f(int a, int b) {  
    int *p = (int *)malloc(sizeof(int));  
    memset(p, 0, sizeof(int));  
    *p = a + b;    
    return p;  
} 

int main(){
	int *p1 = NULL;  
    p1 = f(1, 2);  
}
```

### （2）指针数组与数组指针

**（1）指针数组**

```c++
//一个数组，它包含的元素是指针
int* point[10]; 
```

**（2）数组指针**

```c++
//一个指针，它指向的是一个数组
int (*point)[10]; 
```

### （3）函数模板与模板函数

**（1）函数模板**

表示一个模板，专门用来生成函数；

```c++
template <typename T> 
void fun(T a) 
{ 
}
```

**（2）模板函数**

是一个函数，表示由一个模板生成而来的函数；

如：fun <int> , fun <double> , fun <Shape*>等；

### （4）类模板与模板类

**（1）类模板**

表示一个模板，专门用于生产类的模板；

```c++
template <typename T> 
class Vector 
{ 
}; 
```

**（2）模板类**

是一个类，表示由一个模板生成而来的类；

如：Vector<int> , Vector<double> , Vector<Shape*>等；

----

转载自[雪舞飞影](https://blog.csdn.net/dongxianfei)

