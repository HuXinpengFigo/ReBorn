---
title: 牛客网刷题笔记--Episode 1
date: 2019-12-02 17:19:11
index_img: /img/368-3682760_c-language-hd-png-download.png.jpeg
banner_img: /img/1200px-ISO_C++_Logo.svg.png
tags: [C++,牛课网]
categories: 实习
---

### `深/浅拷贝`和`赋值操作`

题目：

<!-- more -->

```c++
#include<iostream>
using namespace std;
class MyClass
{
public:
    MyClass(int i = 0)
    {
        cout << i;
    }
    MyClass(const MyClass &x)
    {
        cout << 2;
    }
    MyClass &operator=(const MyClass &x)
    {
        cout << 3;
        return *this;
    }
    ~MyClass()
    {
        cout << 4;
    }
};
int main()
{
    MyClass obj1(1), obj2(2);
    MyClass obj3 = obj1;
    return 0;
}
```

运行时的输出结果是（**12244**）

> 没有重载=之前：
>
> ```c++
> A a;
> A b;
> a = b;
> ```
>
> 这里是赋值操作。
>
> ```c++
> A a;
> A b = a; 
> ```
>
> 这里是浅拷贝操作。

> 重载 = 之后：
>
> ```c++
> A a ;
> A b;
> a = b;
> ```
>
> 这里是深拷贝操作（当然这道题直接返回了，通常我们重载赋值运算符进行深拷贝操作）。 
>
> ```c++
> A a;
> A b = a; 
> ```
>
> 这里还是浅拷贝操作。

换而言之，

```c++
C MyClass obj3 = obj1;
obj3还不存在，所以调用拷贝构造函数输出2，
如果obj3存在，obj3=obj，则调用复制运算符重载函数，输出3
```

如果写成 

```c++
MyClass obj3; 
obj3 = obj1; 
```

输出的结果就是 1203444

### `虚函数`与`动态绑定`

```c++
class A
{
public:
 void FuncA()
 {
     printf( "FuncA called\n" );
 }
 virtual void FuncB()
 {
     printf( "FuncB called\n" );
 }
};
class B : public A
{
public:
 void FuncA()
 {
     A::FuncA();
     printf( "FuncAB called\n" );
 }
 virtual void FuncB()
 {
     printf( "FuncBB called\n" );
 }
};
void main( void )
{
 B  b;
 A  *pa;
 pa = &b;
 A *pa2 = new A;
 pa->FuncA(); （ 3）
 pa->FuncB(); （ 4）
 pa2->FuncA(); （ 5）
 pa2->FuncB();
 delete pa2;
}
```

程序的输出结果是:

> FuncA called   //pa=&b**动态绑定**但是FuncA不是虚函数，所以FuncA called
> FuncBB called   //FuncB是虚函数所以调用B中的FuncB，FuncBB called
> FuncA called  //pa2是A类指针，不特殊，直接调用A中的函数，所以FuncA called
> FuncB called //FuncB called

### `重载`与`重写`

| 方法重载                                     | 方法重写                         |
| -------------------------------------------- | -------------------------------- |
| 同一类中的相同的方法名，参数和返回值均可不同 | 子类对父类已经实现的方法重新定义 |

> 方法重载（overload）：
>
> 1. 必须是同一个类
> 2. 方法名（也可以叫函数）一样
> 3. 参数类型不一样或参数数量不一样

> 方法重写（override）的两同两小一大原则:
>
> 1. 方法名相同，参数类型相同
> 2. 子类返回类型小于等于父类方法返回类型，
> 3. 子类抛出异常小于等于父类方法抛出异常，
> 4. 子类访问权限大于等于父类方法访问权限。
>
> ```c++
> public class TTTTT extends SuperC{
>  public String get(){
>      return null;
>  }
> }
> class SuperC{
>  Object get(){
>      return null;
>  }
> }
> 
> ```

> 方法重载的`返回值的类型`可以`不同`，因为判断方法重载的方法主要是根据方法的参数不同来判定；
>
> 方法重写的返回值类型需要相同，重写就是子类继承了父类的方法，并在此方法上重写属于自己的特征，既然是继承过来的，那么它的`返回值类型`就必须要`相同`。

### `sizeof`和`strlen`

32位系统下下面程序的输出结果为多少？

```c++
void Func(char str_arg[100])
{
       printf("%d\n",sizeof(str_arg));
}
int main(void)
{
     char str[]="Hello";
     printf("%d\n",sizeof(str));
    printf("%d\n",strlen(str));
    char*p=str;
    printf("%d\n",sizeof(p));
    Func(str);
}

```

输出结果为：6 5 4 4

> 对字符串进行`sizeof`操作的时候，会把字符串的`结束符"\0"`计算进去的;
>
> 进行`strlen`操作求字符串的长度的时候，`不计算"\0"`的。
>
> 数组作为函数参数传递的时候，已经退化为指针了，Func函数的参数`str_arg`只是表示一个指针，那个100`不起任何作用`的。
>
> 指针不管什么类型的使用sizeof都为4（32位系统下）。

> 1. str是复合类型数组char[6]，维度6是其类型的一部分，sizeof取其 维度*sizeof(char)，故为6；*
> 2. *strlen 求c类型string 的长度，不含尾部的'\0'，故为5；*
> 3. *p只是个指针，32位机上为4；*
> 4. *c++中不允许隐式的数组拷贝，所以Func的参数会被隐式地转为char*，故为4；
>
> note：若Func的原型为void Func(char (&str_arg) [6])（若不为6则调用出错），则结果为6。

