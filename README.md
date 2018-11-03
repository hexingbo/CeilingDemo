# CeilingDemo
With a head at the top of the suspension layout

1.使用的过程中会涉及到design包中的控件，请自行了解

2.实现的过程中遇到问题可留言解决具体的地址在http://blog.csdn.net/bydbbb/article/details/78709732

3.由于项目需求需要，需要一个带有头部的吸顶布局，在网上搜索了好多实现办法，都不太理想，最终使用对RecyclerView添加分割线的方式，重写了RecyclerView的分割线来实现这个悬浮栏。效果比较简单，但是比较实用，下面主要介绍实现过程。
首先先贴上效果图：

## Demo
用模拟器运行的效果，鼠标拨动和模拟器太卡等原因，实际效果比效果图更炫哦～～
![](https://img-blog.csdn.net/20171204133548754?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYnlkYmJi/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
