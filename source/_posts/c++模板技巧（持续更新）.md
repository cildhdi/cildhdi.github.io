---
title: c++模板技巧（持续更新）
date: 2018-09-01 19:18:53
tags:
- c++
---



下面是我在模板元编程的过程中的一些问题的解决办法，以后应该是遇到了有价值的就会添加到这里。

## 限制模板参数的类型

下面有一个简单的虚数模板类（省略了无关的东西）：

```c++
template<typename T>
class Complex
{
	T v, r;
};
```

我们希望别人在使用这个类的时候传递一个数值型的模板参数，尽管你在注释里写的清清楚楚，但也总是会有人什么类型都敢往里传。如果编译失败都还好说，但是如果传入的类型参数刚好符合要求（比如重载了+-\*/等操作符）而编译通过了，那么问题很可能会在运行时出现，更加麻烦。因此我们要限制T的类型，这里使用了c++11的一个定义于头文件[<type_traits>](https://zh.cppreference.com/w/cpp/header/type_traits)中的[is_arithmetic](https://zh.cppreference.com/w/cpp/types/is_arithmetic) 模板结构以及 [static_assert](https://zh.cppreference.com/w/cpp/language/static_assert) 语句。

```c++
template<typename T>
class Complex
{
	T v, r;
	Complex()
	{
		static_assert(std::is_arithmetic<T>::value, "T should be numeric type")
	}
};
```

这样当我试图用一个结构体类型来实例化时：

```c++
struct MyStruct
{
	int i, j;
};
Complex<MyStruct> c;
```

就会编译出错并显示我们指定的提示：

> error C2338: T should be numeric type
> note: 编译 类 模板 成员函数 "Complex<main::MyStruct>::Complex(void)" 时
> note: 参见对正在编译的函数 模板 实例化“Complex<main::MyStruct>::Complex(void)”的引用
> note: 参见对正在编译的 类 模板 实例化 "Complex<main::MyStruct>" 的引用

除了is_arithmetic，在<type_traits>头文件中还有许多类似的模板结构，通过组合它们的value可以对类型产生多种多样的限制，另外还需要注意static_assert的第一个参数必须是编译期常量。

##类型推导

用一个简单的代码片段来引入问题：

```c++
template<typename T1, typename T2>
Type add(T1 a, T2 b)
{
	return a + b;
}
```

这是一个非常简单的模板函数，但是它的返回值Type应该是T1或者是T2呢？如果T1 = T2 = int，那么毫无疑问Type就应该也是int，但是若T1 = int, T2 = double，那么Type就应该是double，那么怎么在代码里表现这种变化呢？有一种解决办法：

```c++
template<typename T1, typename T2>
auto add(T1 a, T2 b)
{
	return a + b;
}
```

这是利用了编译器的类型推导，通过a+b的值自动推导出返回值的类型：

```c++
std::cout << typeid(add(1, 1.1)).name();
```

输出：

> double

但是面对更加复杂的情况呢？例如：

```c++
template<typename T>
class Pos
{
private:
	T x, y;
public:
	template<typename _T>
	Type operator+(const Pos<_T>& rhs)
	{
		Type res = *this;
		res.x += rhs.x;
		res.y += rhs.y;
		return res;
	}
	//...
};
```

其中的Type又应该是什么类型？显然上面直接return a + b的解决方法不行了，但是我们可以不用编译器来自动做类型推导，自己来写：

```c++
template<typename T>
class Pos
{
private:
	T x, y;
public:
	template<typename _T>
	auto operator+(const Pos<_T>& rhs)
	{
		using E = decltype(T() + _T());
		Pos<E> res = *this;
		res.x += rhs.x;
		res.y += rhs.y;
		return res;
	}
	//...
};
```

decltype能够推导出括号里的数据类型。另外，根据函数宏的特点，可以写如下宏：

```c++
#define COMBINE_TYPE(T1, op, T2) decltype(T1() ## op ## T2())
//op为具体的操作符
```

那么上面的E可以直接由COMBINE_TYPE(T, +, _T)来代替，增加了可读性。