---
title: C变长数组
date: 2018-05-16 18:18:02
tags: 
- c
- c++
---

​	在C/Cpp中定义一个数组：

```c
Type array[const_size];
```

其中const_size必须是一个常量或者常量表达式（constexpr），或许在不同的编译器上有不同的规定，但是在标准里，一个数组的长度必须是在一开始就确定下来的，后续就不能改变了。在C里想实现动态的数组来存放不定长的数据，可以使用malloc函数，下面就对它进行一个简单的包装以实现存放和访问不定长的数据。

## 类型

​	一个变长的数组不可能只支持一种数据，在cpp里，使用泛型可以很简单的实现，但是在C里没有这样的特性，cpp中尖括号里的类型参数在C里只能手动的写到类型名称中去，另外考虑到对多种类型的支持，所以最后写出来可能会有很多的类型定义，比如：int_array，double_array，char_array ...除了保存数组信息的结构定义，每一个类型的数组还有若干对应的处理函数，如果一个个写下来，代码量*&￥#%，关键是这些代码的逻辑都是一样的，那么有没有办法来模仿泛型呢？答案当然是没有，好的本篇博客到此结束，谢谢观看。





























hhh，下面是解决办法：

```c
#define ARRAY_SPECIALIZATION(_Ty) \
\
struct _Ty##_array\
{\
	_Ty* pointer = NULL;\
	unsigned int size = 0;\
	unsigned long capacity = 0; \
}; \
```

"\"实现多行宏定义，"##"在宏里表示简单的字符串拼接。通过这一段宏定义，并由以下代码来实现手动实例化：

```c
ARRAY_SPECIALIZATION(int)
ARRAY_SPECIALIZATION(float)
ARRAY_SPECIALIZATION(long)
ARRAY_SPECIALIZATION(char)
ARRAY_SPECIALIZATION(double)
```

这样就有了type_array结构的五份定义，接着上面的宏定义，编写出对应类型的处理函数。



## 变长

​	变长的实现就很简单了。最开始初始化时malloc出一片大小为capacity*sizeof(type)的内存，添加数据时，如果size将要大于capacity了，就另外开辟更大的内存，并将原内存的数据复制到新内存中。相关代码：

```c
void _Ty##_array_expand(_Ty##_array* _Ta)\
{\
	if (_Ta->capacity==0)\
		_Ta->capacity = 1;\
	else\
		_Ta->capacity *= 2;\
	_Ty* pNew=_Ty##_allocator(_Ta->capacity);\
	assert(pNew);\
	if (_Ta->pointer!=NULL)\
	{\
		memcpy((void*)pNew,(const void*)_Ta->pointer,_Ta->size*sizeof(_Ty));\
	}\
	_Ty##_deleter(_Ta->pointer);\
	_Ta->pointer=pNew;\
}\
void _Ty##_array_add( _Ty##_array* _Ta,unsigned int num_of_ele, ...)\
{\
	va_list args;\
	va_start(args,num_of_ele);\
	while(_Ta->size + num_of_ele > _Ta->capacity ) _Ty##_array_expand(_Ta);\
	while(num_of_ele!=0)\
	{\
		_Ty _Te=va_arg(args,_Ty);\
		*(_Ta->pointer+_Ta->size)=_Te;\
		_Ta->size++;\
		num_of_ele--;\
	}\
	va_end(args);\
}\
```

扩宽内存的策略是将当前大小扩大一倍，如果一次加入很多数据，扩宽函数将会执行很多次造成效率低下，修改也很简单，这里就不实现了。

​	除此以外，添加元素使用了va_list，实现了类似printf的变参函数，有关va_list的用法请自己搜索。



### 使用

#### 添加元素

```c
void type_array_add(type_array* _Ta,unsigned int num_of_ele, ...);
```

#### 访问元素

```c
type type_array_at(type_array* _Ta,unsigned int index);
type* type_array_pat(type_array* _Ta,unsigned int index);
```

at访问元素值，pat访问元素指针。

#### 排序

使用qsort:

```c
int _Ty##_cmp(const void* ln,const void* rn)\
{\
	return (*(_Ty*)ln)-(*(_Ty*)rn)>0?1:0;\
}\
void _Ty##_array_sort(_Ty##_array* _Ta)\
{\
	qsort(_Ta->pointer,_Ta->size,sizeof(_Ta),_Ty##_cmp);\
}\
```
### 最后

​	如果这样用：

```c
ARRAY_SPECIALIZATION(long long)
```

那么宏替代后的部分代码：

```c
struct long long_array
{
	long long* pointer = NULL;
	unsigned int size = 0;
	unsigned long capacity = 0; 
}; 
```

其中struct long long_array肯定是有问题的，如果类型中有空格，一种解决办法就是：

```c
typedef long long ll;
```

就可以正常使用了。

### [代码地址](https://github.com/cildhdi/Code/blob/master/C_Array.h)
（图简单写在了一个头文件里）