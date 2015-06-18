---
layout: post
title:  链接指示：extern 'C'
date:   2015-06-18 14:30
categories: coding
---

## 1.声明非C++函数

链接指示有两种形式：单个的和复合的。链接指示不能出现在类定义或函数定义的内部，它必须出现在函数的第一次声明上。

```c
// single statement linkage directive
extern "C" size_t strlen(const char *);
// compound statement linkage directive
extern "C" {
    int strcmp(const char *, const char *);
    char *strcat(char *, const char *);
}
```

## 2.链接指示与头文件

可以将多重声明形式应用于整个头文件

```c
// compound statement linkage directive
extern "C" {
#include <string.h>
}
```

链接指示可以嵌套，所以，如果头文件包含了带链接指示的函数，该函数的链接不受影响

## 3.导入C++函数到其他语言

通过对函数定义使用链接指示，使得用其他语言编写的程序可以使用C++函数：

```c
// the calc function can be called from C programs
extern "C" double calc(double dparm) { /* ... ...  */ }
```

当编译器为该函数产生代码的时候，它将产生适合于指定语言的代码。

## 4.链接指示支持的语言

要求编译器支持对C语言的链接指示。编译器可以为其他语言提供链接说明。例如：extern "Ada"、extern "FORTRAN"等。
对链接到C的预处理器支持：
有时需要在C和C++中编译同一源文件。当编译C++时，自动定义预处理器名字__cplusplus，所以可以根据是否正在编译C++有条件地包含代码。

```c
#ifdef __cplusplus
// ok : we're compiling C++
extern "C"
#endif
int strcmp(const char *, const char *);
```

## 5.重载函数与链接指示

链接指示与函数重载之间的相互作用依赖于目标语言。如果语言支持重载函数，则为该语言实现链接指示的编译很可能也支持C++的这些函数重载。
C++保证支持的唯一语言是C，C语言不支持函数重载，所以，不应该对下面的情况感到惊讶：在一组重载函数中只能为一个C函数指定链接指示。用带给定名字的C链接声明多于一个函数是错误的：

```c
// error : two extern "C" functions in set of overloaded functions
extern "C" void print(const char *);
extern "C" void print(int);
```

在C++程序中，重载C函数很常见，但是，重载集合中的其他函数必须都是C++函数：

```c
class SmallInt { /* */ };
// the C function can be called from C and C++ programs
// the C++ functions overloaded that function and are callable from C++
extern "C" double calc(double);
extern SmallInt calc(const SmallInt &);
```

可以从C程序和C++程序调用calc的C版本。其余函数是带类型形参的C++函数，只能从C++程序调用。声明的次序不重要。

## 6.extern "C" 函数指针
编写函数所用的语言是函数类型的一部分。为了声明用其他程序设计语言编写的函数的指针，必须使用链接指示：

```c
// pf points to a C function returning void taking an int
extern "C" void (*pf)(int);
```

使用pf调用函数的时候，假定该调用是一个C函数调用而编译该函数。
note:C函数的指针与C++函数的指针具有不同的类型，不能将C函数的指针初始化或赋值为C++函数的指针(反之亦然)
存在这种不匹配的时候，会给出编译的错误：

```c
void (*pf1)(int);               // points to a C++ function
extern "C" void (*pf2)(int);    // points to a C funtion
pf1 = pf2;                  	// error: pf1 and pf2 have different types
```

## 7.应用于整个声明的链接指示
使用链接指示的时候，它应用于函数和任何函数指针，作为返回类型或形参类型使用：

```c
// f1 is a C function; its parameter is a pointer to a C function
extern "C" void f1(void(*)(int));
// FC is a pointer to C function
extern "C" typedef void FC (int);
// f2 is a C++ function with a parameter that is a pointer to a C function
void f2(FC *);
```