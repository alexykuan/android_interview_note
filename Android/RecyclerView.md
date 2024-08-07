### RecyclerView介绍

RecyclerView是用来显示多个相同布局的列表项的是视图。它有以下的特点：

1. 支持多种以及自定义的布局模式。 线性布局、网格布局、瀑布流布局等。
2. 支持多重刷新方式。有增加、删除、更新等操作。
3. 动态加载以及ViewHolder缓存，提高性能，减少内存消耗。
4. 支持动画效果。

### RecyclerView的缓存

RecyclerView的缓存分为四级：

1. Scrap。包括mAttachedScrap和mChangedScarp。在item被删除或者重新布局的时候会将ViewHolder放入mAtachedScrap。
2. mCachedViews最大容量为2，存储的内容为屏幕上和下第一个不可见的ViewHolder，在屏幕滚动的时候通过position和id来判断来直接复用。在朝一个方向滚动的时候会持续的将ViewHolder放入`RecyclerPool`中。
3. ViewCacheExtension是用户自定义的缓存。
4. RecycledViewPool中的mScarp。里面是根据不同的ViewType来储存的ViewHolder数组，这个ViewHolder数组默认大小是5个。

### RecyclerView的动画