---
title: KrUI-列表绘制
tags: 'gui'
date: 2018-05-25 22:12:34
---


### GDI+实现双缓冲

​	在Windows系统中，有设备上下文（DC）的概念，可以把它理解为画图的环境。比如一个窗口就有设备上下文，使用该设备上下文绘图，就可以把绘制的内容显示在窗口上。另外还有内存设备上下文，在它上绘制一次要比在窗口设备上下文上绘制一次要快。所以，在收到WM_PAINT刷新一次窗口时，就可以把所有的操作一次性写入内存设备上下文中，最后再使用位块传输把内存设备上下文中的内容复制到窗口设备上下文中，这样就避免了一次更新窗口中对窗口设备上下文的大量的绘图操作导致的屏幕闪烁。

​	GDI+是对GDI的一些封装，不仅让绘图更加方便，还加入了像抗锯齿这样的特性。不过既然是封装，效率上肯定会有一定的损失，所以在[KrUI](https://github.com/cildhdi/KrUI)中，控件和窗口都是双缓冲绘制。具体的实现很简单，就是在GDI+中寻找一个能绘图的地方来取代GDI中使用的内存设备上下文，最后是选择了Gdiplus::Bitmap，与Gdiplus::Graphics绑定后具有很多方便的绘图成员函数，并且win7及以后的系统中都自带了gdiplus.dll。

​	下面就说说最开始想到的两种方案。

### 1. 全部绘制，部分显示

​	每一个列表项都由以下结构体定义：

```c++
struct KrListItem
{
	unsigned int m_Index;
	std::wstring m_ItemName;
	unsigned int m_Height;
	bool m_bSelected;
	KrListItem(std::wstring wStr, unsigned int Index, unsigned int Height) : m_ItemName(wStr), m_Index(Index), m_Height(Height), m_bSelected(false) {}
};
```

​	其中有m_Height成员，代表每一项的高度。因此对所有列表项的m_Height求和得到height，则需要一块宽等于列表控件的宽、高为height的Gdiplus::Bitmap缓冲，并在显示区域改变时，把这个缓冲区的内容部分的画到最终控件中。因此，每一次添加或者删除列表项（尤其是初始化时的大量添加），都需要resize然后repaint这块缓冲区，并且在列表项特别多时，repaint操作将会花费大量资源。最后实现出来在我的老人机上运行了一下，意料之中的卡......

### 2. 实时计算

​	后来想到另外一种解决方式，既然全部绘制特别慢，那么就部分绘制，这样就需要在界面更新时计算需要绘制的列表项。由于上面的结构中含有了列表项的高度信息，可以很容易的根据当前显示位置来遍历计算出需要绘制的列表项。另外还有一点需要注意，当出现这种情况时：

<img src="http://img02.sogoucdn.com/app/a/100520146/5e3538ea44eb484b4e740b5873ff187d" width="50%">

计算出来的第一项的绘制开始位置变成了负数，好在Gdiplus的绘图函数是支持负数坐标的，省得再写分割的代码。



​	最后实现出来也是第二种的更加流畅（这个鼠标位置不是我写的有问题...是截图的ShareX有问题）：

<img src="http://img03.sogoucdn.com/app/a/100520146/e6768b865052abb00075c429d5870b03" width="50%">

​	（随便添加了500项仍然纵享丝滑hhhh）



### [具体地址](https://github.com/cildhdi/KrUI/blob/master/source/KrList.h)