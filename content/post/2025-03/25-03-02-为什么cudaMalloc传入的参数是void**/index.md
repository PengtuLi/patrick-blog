---
title: "为什么cudaMalloc传入的参数是void**"
author : "tutu"
description:
date: '2025-03-02'
lastmod: '2025-03-02'
image:
math: true
hidden: false
comments: true
draft: false
categories : ['program language']
---

- 函数原型

`cudaError_t cudaMalloc (void **devPtr, size_t  size );`

之前一直在想为什么不是直接传入void*呢，把指针传进去直接赋值就好了嘛，后来经过思考，错误的原因在于函数的形参和传入的参数不是一个参数（地址不同），就算是指针也一样，是两个定义在栈上的指针，虽然指向的地址一样。

```c++
#include<iostream>

void my_malloc(void *int_ptr, int size)
{
    std::cout << "--函数调用--"<< std::endl;

    //不再和传入的指针是同一个指针，虽然指向的地址相同！！
    std::cout << "函数参数的值: " << int_ptr << std::endl;
    std::cout << "函数参数本身的地址是: " << &int_ptr << std::endl;

    void * res = malloc(size*sizeof(int));
    int_ptr=res;

    std::cout << "malloc返回指针的值: " << res << std::endl;
    std::cout << "函数参数的值: " << int_ptr << std::endl;
    
    std::cout << "--函数调用结束--"<< std::endl;
}

int main()
{
    int a = 10;        // 定义一个整数变量
    int* pointer = &a; // 定义一个指针，指向变量 a

    // 打印指针的地址
    std::cout << "指针的值: " << pointer << std::endl;
    std::cout << "指针变量本身的地址是: " << &pointer << std::endl;
    std::cout << "变量的地址是: " << &a << std::endl;
    std::cout << "变量的值: " << a << std::endl;

    *pointer = 20;

    std::cout << "指针的值: " << pointer << std::endl;
    std::cout << "指针变量本身的地址是: " << &pointer << std::endl;
    std::cout << "变量的地址是: " << &a << std::endl;
    std::cout << "变量的值: " << a << std::endl;

    my_malloc(pointer,5);

    std::cout << "指针的值: " << pointer << std::endl;
    std::cout << "指针变量本身的地址是: " << &pointer << std::endl;
    std::cout << "变量的地址是: " << &a << std::endl;
    std::cout << "变量的值: " << a << std::endl;

    return 0;
}

/*
指针的值: 0x16efe31b8
指针变量本身的地址是: 0x16efe31b0
变量的地址是: 0x16efe31b8
变量的值: 10
指针的值: 0x16efe31b8
指针变量本身的地址是: 0x16efe31b0
变量的地址是: 0x16efe31b8
变量的值: 20
--函数调用--
函数参数的值: 0x16efe31b8
函数参数本身的地址是: 0x16efe3148
malloc返回指针的值: 0x135e05e10
函数参数的值: 0x135e05e10
--函数调用结束--
指针的值: 0x16efe31b8
指针变量本身的地址是: 0x16efe31b0
变量的地址是: 0x16efe31b8
变量的值: 20
*/
```
