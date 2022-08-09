# cache-lab

## 预备说明

关于这个lab其实我大部分得思路都是从其他人的博客得来的，因此我在这里只是简述一下我遇到得一些困难和理解性的东西

[上课的ppt](http://www.cs.cmu.edu/afs/cs/academic/class/15213-f15/www/recitations/rec07.pdf)包含关于第一部分一些帮助函数和第二部分最重要思想blocking

### 博客推荐

1. 十分重要的博客：https://www.cnblogs.com/liqiuhao/p/8026100.html?utm_source=debugrun&utm_medium=referral

2. 推荐博客：https://zhuanlan.zhihu.com/p/42754565，但是这个博客关于64^64的图是错误的请看另一个。

## 正文

我主要说明过于cache-lab第二部分64*64的问题，这里最难理解的是逆序存储，其实这里很关键的点就是在3和4号转置是如果我们都从上往下，那么每一个元素读取都会发生抖动，但是当我们逆序读取时，我们对于3号是从下往下上的，对于四号是从上往下的，因此在第一行的时候，对于3号而言替换的是第四行的cache，而对于四号而言替换的第一行的cache，同时对于4号来说2号此时缓存的cache刚好就是4号所需要的。而如果我们正序读取，那么我们在2号放入是就会刷新缓存，同时在四号放入是还会刷新缓存。

**如果看不明白请看第二个博客的解释和第一个博客的图，第二个博客的图是错误的**

## 总结

虽然这个lab我更多的是了解的居多但是lab我也是自己手敲的代码，同时对于一些简单抽象cache缓存组和blocking又了很深刻的理解。