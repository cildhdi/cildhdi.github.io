---
title: c++模板技巧(二)
date: 2019-10-10 18:51:30
tags: c++
---

最近写了几题累死编译器系列的很有意思的模板题目（编译期求值），列一下新学到的一些黑魔法。

## constexpr 函数

例如以下[函数](http://en.cppreference.com/w/cpp/language/constexpr)，
``` c++
constexpr int factorial(int n)
{
    return n <= 1 ? 1 : (n * factorial(n - 1));
}
```
如果 n 也是编译期常量，那么 factorial(n) 也是编译期常量，`factorial(n)` 编译之后将会直接被替换为对应的值。

## 编译期 itoa
这个直接就是一道题目，要求实现一个类，`itoax<x, y>::value` 直接就是 x 在 y 进制的字符串表示，看到题我甚至怀疑真的可以实现吗... 同一个系列还有一道阶乘的题目，
``` c++
template <int x>
struct factorial {
  static constexpr unsigned long long value = x * factorial<x - 1>::value;
};

template <int x>
constexpr unsigned long long factorial<x>::value;

template <>
struct factorial<0> {
  static constexpr unsigned long long value = 1;
};

constexpr unsigned long long factorial<0>::value;
```
我就在想会不会也是一个套路，利用模板递归获得上一个状态的 value，但是 char[] 和 char 没有办法再把他们在编译期组合成 char[]，实在没有头绪就放弃了，看了别人的解法自己写了一遍：
``` c++
using integer = unsigned long long;

constexpr char get_digit(integer n) {
  return n < 10 ? '0' + static_cast<char>(n) : 'a' + static_cast<char>(n) - 10;
}

template <integer X, integer Base = 10, char... Cs>
struct itoaxx : public itoaxx<X / Base, Base, get_digit(X % Base), Cs...> {};

template <integer Base, char... Cs>
struct itoaxx<0, Base, Cs...> {
  static constexpr char value[] = {Cs..., 0};
};

template <integer Base, char... Cs>
constexpr char itoaxx<0, Base, Cs...>::value[];

template <integer X, integer Base = 10>
struct itoax : public itoaxx<X, Base> {};

template <integer Base>
struct itoax<0, Base> {
  static constexpr char value[] = {'0', 0};
};

template <integer Base>
constexpr char itoax<0, Base>::value[];
```
其实只用一个类就可以实现除了 0 之外的所有的进制转换，但 0 是终止条件，转换会得到一个空字符串，所以再套一个类做一个 0 的偏特化就可以了。
在这个解法里，利用继承和参数包传递解决了上面char[] 和 char 无法组合成 char[]的问题，主要思想就是把所有的字符利用继承以参数包的形式全部传递给终止条件：X=0的特化，在这个特化里把参数包展开到 value 的初始化列表里。

## 使用 enum
上面很多类里的 value 除了要在类里面声明，还需要在类外定义一下（通常在 cpp 文件里），非常麻烦，使用 enum 就可以避免多写一句（还有可能要新建个 cpp 文件）：
### 使用前
``` c++
// A.h
class A{
    static constexpr int value = 0;
};

// A.cpp
constexpr int A::value;
```
### 使用后
``` c++
// A.h
class A{
    enum{
        value = 0 // 自动 static constexpr
    };
};
```

## 编译器：我太难了
