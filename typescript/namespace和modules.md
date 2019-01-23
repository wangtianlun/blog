### 介绍

这篇文章概述了多种在typescript中，使用namespaces和modules组织代码的方式，我们将会重温一些进阶的如何使用namespaces和modules的主题，还有处理一些在typescript中使用它们时的一些陷阱

### 使用Namespaces

Namespaces简化了js对象在全局命名空间里的命名，这使得namespaces可以很容易的去构造使用。它们可以跨越多个文件，也可以通过使用--outFile标识去串联起来使用。Namespaces在web程序中是一种很好的组织代码的方式，把所有的依赖包含进script标签插入到你的html页面上。

像所有的全局命名空间污染一样，它很难去确定组件的依赖，尤其是在一个大型的应用程序中


### 使用Modules

和namespaces类似，modules既可以包含代码也可以包含声明。两者最主要的区别就是modules声明了它的依赖

