### String类可以被继承吗？

不可以，String类是final的，不能被继承。

### String类为什么是不可变的？

1. 为了实现字符串池。
2. 为了线程安全。
3. 为了避免安全问题。
4. 为了实现字符串常量池。

### 如果String是final的为什么可以用 + 连接？

因为 + 号实际使用的是StringBuilder或者StringBuffer的append方法。