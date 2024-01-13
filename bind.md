---
title: bind
date: 2024-01-07 11:58:30
tags:
---

### 背景
作为一名C++开发工程狮，尤其是桌面端软件的开发，面对各种各样神奇的dll太习以为常了，因为上层软件避免不了需要对接各种SDK，而SDK提供出来的接口中，避免不了使用回调函数，所以借此背景，简单讲讲对接SDK时，我们上层软件对回调函数的使用。

### C98的回调函数
```c++
#include <iostream>

int func(int val)
{
    val++;
    std::cout << "val: " << val << std::endl;
    return val;
}

typedef int funcptr(int val);
int dllFunc(int val, funcptr callback)
{
    return callback(val);
}

int main()
{
    std::cout << "Hello Mac!" << std::endl;
    int res = dllFunc(12, func);
    std::cout << "dll result: " << res << std::endl; // 输出13
    std::cin.get();
    return 0;
}
```
相信大家对上述代码都不陌生，是一个C98版本的回调函数的使用，在这里我的dllFunc函数模拟了接口函数，然后我们将我们的数据和函数指针作为参数传递进去，dllFunc函数通过函数指针回调给main函数数据。

### C++11的回调函数
当然，在C++11版本的编译器中你也可以用上述函数指针的方式实现相同的功能，但是C++11提供了更易容易理解的方式，来让你优雅的使用回调函数。
## 先说一个关键字using
```c++
#include <iostream>
#include <functional>

int func(int val)
{
    val++;
    std::cout << "val: " << val << std::endl;
    return val;
}

typedef int funcptr(int val);
//55行这里！！！
using funcCallBack = int (int val);
int dllFunc(int val, funcptr callback, funcCallBack callback2)
{
    int res1 = callback(val);
    int res2 = callback2(val);
    return res1 + res2;
}

int main()
{
    std::cout << "Hello Mac!" << std::endl;
    int res = dllFunc(12, func, func);
    std::cout << "dll result: " << res << std::endl; //输出26
    std::cin.get();
    return 0;
}
```
看到上面代码第55行没，是不是更符合人类语言了一点，有没有像我们通常定义变量一样int i = 12;然后主函数中使用没有一点区别。

## 再说C++11的std::function
```c++
#include <iostream>
#include <functional>

int func(int val)
{
    val++;
    std::cout << "val: " << val << std::endl;
    return val;
}

typedef int funcptr(int val);
using funcCallBack = int (int val);
int dllFunc(int val, funcptr callback, funcCallBack callback2, std::function<int(int)> callback3)
{
    int res1 = callback(val);
    int res2 = callback2(val);
    int res3 = callback3(val);
    return res1 + res2 + res3;
}

int main()
{
    std::cout << "Hello Mac!" << std::endl;
    // 这里！！！
    std::function<int(int)> funcptr = func;
    int res = dllFunc(12, func, func, funcptr);
    std::cout << "dll result: " << res << std::endl;
    std::cin.get();
    return 0;
}
```
C++11提供了std::function这种的模版类，接受一个函数类型，并且可以像函数指针一样可以调用，是不是更方便了一些😄

## 最后说一下标题中的bind
```c++
#include <iostream>
#include <functional>

using funcCallBack = int (int val, void* pUser);
int dllFunc(int val, void* pUser, funcCallBack callback, std::function<int(int,void*)> callback2)
{
    int res1 = callback(val, pUser);
    int res2 = callback2(val, pUser);
    return res1 + res2;
}

class Client
{
public:
    Client() {}
    ~Client() {}

    void SetCallBack()
    {
        auto class_func = std::bind(&Client::bindFunc, this, std::placeholders::_1, std::placeholders::_2);
        int res = dllFunc(12, this, classFunc, class_func);
        std::cout << "dll result: " << res << std::endl;
    }

    static int classFunc(int val, void* pUser)
    {
        Client* client = static_cast<Client*> (pUser);
        if (client)
        {
            client->processData(val);
        }
    }

    int bindFunc(int val, void* pUser)
    {
        this->processData(val);
        return 0;
    }

    void processData(int val)
    {
        val++;
        m_Val = val;
        std::cout << "m_Val: " << m_Val << std::endl;
    }

private:
    int m_Val = 0;
};

int main()
{
    std::cout << "Hello Mac!" << std::endl;
    Client client;
    client.SetCallBack();
    std::cin.get();
    return 0;
}
```
进入正式环境进行开发，大家肯定都是在面向对象进行编程，所以大多数时间都是在处理类之间的关系，所以对于对接接口的处理中，肯定也避免不了和类之间进行打交道，上述代码分别描述了两种类中处理回调函数的方式，分别为C98版本与C++11中的bind+function方式。

- C98方式
大家肯定都知道，在C++中，类成员函数一般是不可以作为回调函数使用的，因为类成员函数中默认第一个参数为this，所以会导致参数列表不匹配导致编译不通过，所以对于此种情况，一般的做法是，使用静态函数作为回调函数，这里又引入一个问题，如果我还想往类中成员进行赋值怎么办？因为静态函数中没有this，所以又有一个做法，新加一个参数void* pUser，设置回调函数的时候，将this传进去，然后在回调函数中再强转回来，来对成员进行赋值或者调用setter接口进行赋值，这是一种操作（脱裤子放屁的操作😄）
- C++11方式
为了解决上述痛点，C++11中有了更优雅的解决方案，也就引出来了我们今天的主题，std::bind，这个bind可以绑定一个成员函数来作为回调函数，例如上述代码中的bindFunc成员函数就作为了回调函数进行使用，所以再也不用传递这个pUser啦😄，bind返回一个std::function，所以直接传入到接口中即可，这里有个注意点，std::placeholders::_1占位符的个数要与回调函数的参数个数一致才可以。

### 总结
本次总结了回调函数与C++11中的新特性，即作为自己的学习笔记，也希望可以为大家提供一个容易理解的指南。
