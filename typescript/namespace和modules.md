### 介绍

这篇文章概述了多种在typescript中，使用namespaces和modules组织代码的方式，我们将会重温一些进阶的如何使用namespaces和modules的主题，还有处理一些在typescript中使用它们时的一些陷阱

### 使用Namespaces

Namespaces简化了js对象在全局命名空间里的命名，这使得namespaces可以很容易的去构造使用。它们可以跨越多个文件，也可以通过使用--outFile标识去串联起来使用。Namespaces在web程序中是一种很好的组织代码的方式，把所有的依赖包含进script标签插入到你的html页面上。

像所有的全局命名空间污染一样，它很难去确定组件的依赖，尤其是在一个大型的应用程序中


### 使用Modules

和namespaces类似，modules既可以包含代码也可以包含声明。两者最主要的区别就是modules声明了它的依赖

modules依赖于模块加载器（比如CommonJs或者Require.js），对于小型的JS应用来说，modules或许不是最佳实践，但是对于大型应用来说，这里所耗费的成本同时也带来了代码的模块及良好的可维护性的好处。modules提供了更好的代码复用，强作用域隔离和更好的打包工具支持。

值得一提的是，对于nodejs应用来说，modules已经作为组织代码的默认方式

从ECMAScript 2015开始，modules已经作为内置的一部分，通过所有遵循规范的引擎实现都应该已经支持modules，这样，对于新项目来说，modules已经是组织代码的标配

### namespaces和modules的使用陷阱

在这个章节中，我们将介绍几个比较常见的在使用namespaces和modules的陷阱，还有就是如何避免这些陷阱

#### <reference>-ing a module

一个比较常见的错误就是，试图使用"/// <reference ... />"语法来引用一个模块文件，而不是使用import声明语句。为了理解这两者的区别，我们首先需要明白如何让编译器可以锁定基于此路径的模块的类型信息。比如“import x from ...”中的...

编译器会尝试在相应的路径下查找.ts, .tsx和.d.ts文件，如果指定的文件并没有被找到，编译器就会去其他的模块声明文件中去查找。

  - myModules.d.ts

  ```javascript
    // In a .d.ts file or .ts file that is not a module:
    declare module "SomeModule" {
        export function fn(): string;
    }
  ```

  - myOtherModule.ts

  ```javascript
    /// <reference path="myModules.d.ts" />
    import * as m from "SomeModule";
  ```

  这里的reference tag允许我们去定位这个包含了这个模块声明的文件

  
### 没必要的Namespacing

如果你正在转化一个程序从namespaces到modules，这个过程会非常简单，类似于下面文件这样

  - shapes.ts

  ```javascript
    export namespace Shapes {
      export class Triangle { /* ... */ }
      export class Square { /* ... */ }
    }
  ```

这里顶层的module Shapes毫无原因的包装了Triangle和Square，这会让使用你模块的人比较迷惑

  - shapeConsumer.ts

  ```javascript
    import * as shapes from "./shapes";
    let t = new shapes.Shapes.Triangle(); // shapes.Shapes?
  ```

typescript中模块的一个关键特性就是，两个不同的模块绝不会赋予名称给相同的作用域，因为模块的消费者决定了该分配什么名称，没必要主动在一个namesapce中去包裹导出的标记

为了解释为什么你不应该在你的模块内容中用namespace来包裹，一般来说namesapce会提供合理的构造分组，防止命名冲突，由于模块文件自身已经是一个合理的分组，它的顶层名称通过import的代码来定义，所以对于对于导出的对象没必要再去包装额外的模块层。

下面是一个改进版的例子：

  - shapes.ts

  ```javascript
    export class Triangle { /* ... */ }
    export class Square { /* ... */ }
  ```

  - shapeConsumer.ts

  ```javascript
    import * as shapes from "./shapes";
    let t = new shapes.Triangle();
  ```

### 模块的权衡

正如在js文件和modules之间是一对一的关系，在typescript中模块源文件和它们对应的最终生成文件之间也是一对一的关系，这样所带来的影响就是根据目标的模块系统不能将多个模块源文件连接起来。当你的模块目标是commonjs或umd时，你不能使用outFile这个选项来输出文件，但随着typescript1.8及更高版本的发布，当目标target为amd或是system时就有可能可以使用outFile选项

### 参考

[typescript](https://www.typescriptlang.org/docs/handbook/namespaces-and-modules.html)