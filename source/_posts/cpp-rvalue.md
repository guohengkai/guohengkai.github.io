title: C++11中右值的传参及返回值优化
date: 2016-05-13 21:01:20
tags:
- C++
categories:
- Engineering
---
自C++11引入右值概念之后，很多代码有了优化空间，但同时对于编译器在传参和返回值时的优化有了一些疑惑，于是写了点代码做了实验。
<!--more-->
首先定义一个没有实际意义的类A：
``` cpp
class A
{
    public:
        A()
        {
            printf("default construct\n");
        }
        A(const A&)
        {
            printf("copy construct\n");
        }
        A(A&&)
        {
            printf("move construct\n");
        }
        A& operator=(const A& other)
        {
            printf("copy assignment\n");
            return *this;
        }
        A& operator=(A&& other)
        {
            printf("move assignment\n");
            return *this;
        }
};
```

## 传参
定义场景为：将类传进函数里，插入到vector中。函数定义如下：
``` cpp
void PassRightPushLeft(A&& a)
{
    printf("in PassRightPushLeft\n");
    vec.push_back(a);
}

void PassRightPushRight(A&& a)
{
    printf("in PassRightPushRight\n");
    vec.push_back(std::move(a));
}

void PassLeftPushLeft(A& a)
{
    printf("in PassLeftPushRight\n");
    vec.push_back(a);
}

void PassLeftPushRight(A& a)
{
    printf("in PassLeftPushRight\n");
    vec.push_back(std::move(a));
}

void PassConstLeftPushLeft(const A& a)
{
    printf("in PassConstLeftPushRight\n");
    vec.push_back(a);
}

void PassConstLeftPushRight(const A& a)
{
    printf("in PassConstLeftPushRight\n");
    vec.push_back(std::move(a));
}
```

函数调用如下：
``` cpp
A a1;
PassRightPushLeft(std::move(a1));
A a2;
PassRightPushRight(std::move(a2));
A a3;
PassLeftPushLeft(a3);
A a4;
PassLeftPushRight(a4);
```

无论参数为右值引用还是非常数左值引用，若使用普通的push_back，则调用的是该类的复制构造函数；若在push_back中使用std::move，则调用的是该类的移动构造函数。这两种方法的区别在于：调用该函数时，右值引用必须使用std::move或临时对象，而左值引用必须使用左值变量。若需要使用临时变量则需要在左值参数加上const，但这样的话即便在里面调用std::move，调用的也只是复制构造函数，因为编译器判断出传入的值为常数，便使用了复制来替代移动。因此，在此场景下最灵活的应该使用PassRightPushRight的策略。

## 返回值
定义场景为：将函数返回的类构造新的类。函数定义如下：
``` cpp
A ReturnLeftLeft()
{
    printf("in ReturnLeftLeft\n");
    return A();
}

A ReturnRightLeft()
{
    printf("in ReturnRightLeft\n");
    return std::move(A());
}

A&& ReturnRightRight()
{
    printf("in ReturnRightRight\n");
    return std::move(A());
}

A&& ReturnLeftRight()
{
    printf("in ReturnLeftRight\n");
    return A();
}
```

函数调用如下：
``` cpp
A a1(ReturnLeftLeft());
A a2(ReturnLeftRight());
A a3(ReturnRightLeft());
A a4(ReturnRightRight());
```

当使用最普通的看似为复制形式的返回值（ReturnLeftLeft）效率最高，只调用了一次默认构造函数。而其他想通过右值来优化的均调用了一次默认构造函数和一次移动构造函数。这是因为在引入了右值之后，C++编译器会自动对返回值进行优化，将临时变量直接移动给外层的需要构造的变量中，而其他方式下均得先构造出该变量再将其转为右值。因此，在此场景下应该使用ReturnLeftLeft的策略。

## 关于构造与赋值
顺便把构造和赋值函数也做了实验，代码如下：
``` cpp
A a;
A a1(a);
A a2 = a;
A a3(std::move(a1));
A a4 = std::move(a2);
```
结果发现，以上所有代码均调用的是构造函数版本而不是赋值函数版本。
