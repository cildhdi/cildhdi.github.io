---
title: placement new
date: 2018-08-18 11:52:18
tags: 'C++'
---

在《more effective c++》里讨论默认构造函数的部分中，提到了一种对象构造方法：placement new。

## new
比较常用的new操作符的流程大致应该是
- 分配内存
- *调用构造函数构造对象
- *返回内存区域的地址

为什么说这是大致流程呢，因为上面打了星号的两步执行顺序是不确定的，可能在对象还没有构造完成的时候就已经返回了内存地址，这涉及到cpu的乱序执行，学习过Singleton模式的应该都了解过。

与之对应的delete操作符的流程也应该是

- 调用析构函数析构对象

- 回收内存

## placement new
>布置 new：若提供 `placement_params` ，则将它们作为额外参数传递给分配函数。这些分配函数被称作“布置 new ”，根据标准分配函数 void\* [operator new](http://zh.cppreference.com/w/cpp/memory/new/operator_new)([std::size_t](http://zh.cppreference.com/w/cpp/types/size_t), void\*) ，它们简单地返回不更改的第二参数。这被用于在已分配的存储构造对象：

placement new代表的就应该是调用构造函数构造对象这一步，因此我们使用 placement new之前需要手动分配内存，之后也需要手动析构对象和释放内存。

## 实践

开始我写的代码是这个样子的：

```c++
void* rm = operator new[](10 * sizeof(std::string));
for (size_t i = 0; i < 10; i++)
	new(rm + i * sizeof(std::string)) std::string(std::to_string(i));
```

报错：
> error C2036: “void *”: 未知的大小

这是因为(void\* + int)编译器无法计算偏移的大小，虽然我写上了i \* sizeof(std::string)，编译器也无法通过void\*来计算出具体值。

修改之后：

```c++
//分配内存
void* rm = operator new[](10 * sizeof(std::string));
std::string* ps = static_cast<std::string*>(rm);
for (size_t i = 0; i < 10; i++)
	//placement new
	new(ps + i) std::string(std::to_string(i));
for (size_t i = 0; i < 10; i++)
	std::cout << *(ps + i) << std::endl;
for (int i = 9; i >= 0; i--)
	//手动析构
	(ps + i)->~basic_string();
//释放内存
operator delete[](rm);
```

可能有人会疑惑上面手动析构的部分为啥调用的析构函数是~basic_string()而不是~string()，这其实是因为并不存在string这个类（我用的是vs2017，不知道其他编译器的情况，提一下防止打脸），在头文件中有如下定义：

```c++
using string = basic_string<char, char_traits<char>, allocator<char>>;
using wstring = basic_string<wchar_t, char_traits<wchar_t>, allocator<wchar_t>>;
```

string只是后面那个模板特化的一个别名，所以调用的析构函数是~basic_string()。

虽然可以手动调用析构函数，但是构造函数是不可以显式调用的，这也是使用placement new 的原因。

另外，对于值对象，比如int等，是可以不用placement new来直接使用的：

```c++
//分配内存
void* rm = operator new[](10 * sizeof(int));
int* is = static_cast<int*>(rm);
for (size_t i = 0; i < 10; i++)
	*(is + i) = i;
for (size_t i = 0; i < 10; i++)
	std::cout << *(is + i) << std::endl;
//释放内存
operator delete[](rm);
```

输出：

```
0
1
2
3
4
5
6
7
8
9
```

