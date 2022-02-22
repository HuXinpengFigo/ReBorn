---
title: 牛课网刷题笔记--Episode 2
date: 2019-12-20 15:18:31
index_img: /img/368-3682760_c-language-hd-png-download.png.jpeg
banner_img: /img/1200px-ISO_C++_Logo.svg.png
tags: [C++,牛课网]
categories: 实习
---

### default和protected的区别

前者只要是外部包，就不允许访问。

后者只要是子类就允许访问，即使子类位于外部包。

<!-- more -->

> 总结：default拒绝一切包外访问；protected接受包外的子类访问

### 数组传参退化为指针

在32位机器中，如下代码：

```c++
void example(char acWelcome[]){
    printf("%d",sizeof(acWelcome));
    return;
}
void main(){
    char acWelcome[]="Welcome to Huawei Test";
    example(acWelcome);
    return;
}
```

的输出是`4`

> 当数组当作参数传递时，它就退化成指针了，要求数组长度的话，可以在main函数内部求得

#### 测试：

```c++
void example(char acWelcome[]) {
    printf("%d", sizeof(acWelcome));
    return;
}
int main() {
    char acWelcome[] = "Welcome to Huawei Test";
    example(acWelcome);
    printf("\n%d\n", sizeof(acWelcome));
    return 0;
}
```

![](http://uploadfiles.nowcoder.com/images/20160221/959856_1456054089628_BAAF0699667FE20620684FE95520D507)

### 逗号运算符

以下程序的输出结果为（`13`）

```c++
int func(int x,int y)
{
return(x+y);
}
 
main()
{int a=1,b=2,c=3,d=4,e=5;
printf("%d\n",func((a+b,b+c,c+a),(d+e)));
}
```

 c语言提供一种特殊的运算符，**逗号运算符**，优先级别**最低**，它将两式联接起来，如：（3+5,6+8）称为逗号表达式，其求解过程先表达式1，后表达式2，整个表达式值是表达式2的值。

> 如：（3+5，6+8）的值是14，（a=3*5,a*4）的值是60，原因在于赋值运算优先级高于逗号表达式。

逗号表达式的要领：

- 从左到右逐个计算；


- 逗号表达式作为一个整体，它的值为最后一个表达式的值；


- 逗号表达式的优先级别在所有运算符中最低。

对于本题：

(a+b,b+c,c+a)=c+a

func((c+a),(d+e))=13

### 静态外部变量

静态外部变量只在本文件内可用。（**T**）

> 在外部变量的定义前面加上关键字static，就表示定义了一个外部静态变量。外部静态变量具有全局的作用域和全局的生存期，定义成static类型的外部变量将无法再使用extern将其作用范围扩展到其他文件中，而是被限制在了本身所在的文件内，为程序的模块化、通用性提供方便。

### const *

语言中哪一种形式声明了一个指向char类型变量的指针p，p的值不可修改，但p指向的变量值可修改？

`char*const p`

- const出现在*左边，如const char* p，表示p所指向的变量内容不可变，指针指向可以改变；

- const出现在*右边，如char* const p，表示p是个常量指针，即不能指向其他变量，而指向的变量内容可变；

- const出现在*左边和右边，如const char* const p，表示p的指向不能改变，指向的变量内容也不能改变。

### STL

STL中的哪种结构在增加成员时可能会引起原有成员的存储位置发生改变

1. map
2. set
3. list
4. vector

(**4**)

vector是STL中最常见的容器，它是一种顺序容器，支持随机访问。vector是一块连续分配的内存，从数据安排的角度来讲，和数组极其相似。

> 不同的地方就是：数组是静态分配空间，一旦分配了空间的大小，就不可再改变了；而vector是动态分配空间，随着元素的不断插入，它会按照自身vector的扩充机制：按照容器现在容量的一倍进行增长。vector容器分配的是一块连续的内存空间，每次容器的增长，并不是在原有连续间的迭代器已经失 效，所以当操作容器时，迭代器要及时更新。

### 文件流

要求打开文件D:\file.dat，并能够写入数据，正确的语句是（`fstream infile("D:\\file.dat", ios::in|ios::out );`）。

- 如果想以输入方式打开，就用ifstream来定义;

- 如果想以输出方式打开，就用ofstream来定义;

- 如果想以输入/输出方式来打开，就用fstream来定义。

- [x] ios::in：　　　 文件以输入方式打开(文件数据输入到内存)
- [x] ios::out：　　　文件以输出方式打开(内存数据输出到文件)

可以用“或”把以上属性连接起来，如ios::in|ios::out

### 关键字mutalbe

以下代码编译有错误，哪个选项能解决编译错误？

```c++
class A {
    public:
        int GetValue() const {
            vv = 1;
            return vv;
         }
    private:
        int vv;
};
```

1. 改变成员变量"vv"为"mutable int vv"
2. 改变成员函数"GetValue"的声明，以使其不是const的
3. 都不能修复编译错误
4. 都可以修复编译错误

`(4)`

**mutalbe**的中文意思是“可变的，易变的”，跟constant（既 [C](http://product.yesky.com/product/225/225032/) ++中的const）是反义词。

> 在C++中，mutable也是为了突破const的限制而设置的。被mutable修饰的变量，将永远处于可变的状态，即使在一个const函数中。

> 我们知道，如果类的成员函数不会改变对象的状态，那么这个成员函数一般会声明成const的。但是，有些时候，我们需要在const的函数里面修改一些跟类状态无关的数据成员，那么这个数据成员就应该被mutalbe来修饰。

### fork()&clone()

下列关于 clone 和 fork 的区别描述正确的:

> clone是fork的升级版本，不仅可以创建进程或者线程，还可以指定创建新的命名空间（**namespace**）、有选择的继承父进程的内存、甚至可以将创建出来的进程变成父进程的兄弟进程等等

`fork() `函数复制时将父进程的所以资源都通过复制**数据结构**进行了复制，然后传递给子进程，所以 `fork() `函数不带参数； 

`clone()` 函数则是将部分父进程的资源的数据结构进行复制，复制哪些资源是可选择的，这个可以通过参数设定，所以 `clone() `函数带参数，没有复制的资源可以通过指针共享给子进程。

### operator

计算机假定要对类AB定义加号操作符重载成员函数，实现两个AB类对象的加法，并返回相加结果，则该成员函数的声明语句为（`AB operator+(AB &A)`）

> 成员函数操作两个对象时，只需传递一个对象参数，另一个是调用成员的的**this**。友员函数则需要两个对象参数。