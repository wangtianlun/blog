### 为什么要用typescript

这里列举了两个主要的目的

- typescript为js提供了可选的类型系统(type system)
- typescript为当前的js引擎提供了未来JS版本才能使用的特性

### typescript的类型系统

你获取想知道为什么要给javascript添加类型系统呢？

类型系统已经被证明是一种可以增强代码质量和可读性的能力，大型团队（例如谷歌，微软，facebook）都在印证着这个结论，更具体点说：

  - 当要进行代码重构时，类型系统能给予更高的灵活度，因为在编译时进行异常捕获要好于在运行时
  - 类型系统是最好的文档格式之一，函数签名（function signature）是一个定理，而函数体则是相应的证明（proof）

typescript会尽量保持一个低门槛，来保证开发者可以低成本的学习编写ts代码

### 你的js代码就是ts代码

typescript为js提供了编译时的类型检查，最棒的是类型完全是可选的，你的js代码（.js文件）可以重命名成（.ts）文件，typescript同样会返回和原有js文件一样的输出。通过可选的类型检查，typescript就是严格的js超集。

### 类型可以是隐式的

在代码开发阶段，typescript会尽可能用比较低的成本去推断尽可能多的类型信息，例如，在接下来的例子中，typescript将会知道foo是number类型，当在第二行代码中又给foo赋值为一个字符串类型的值时，就会报出错误

```javascript
  var foo = 123;
  foo = '456'; // Error: cannot assign `string` to `number`
```

![example]( https://img.souche.com/f2e/9b83e87a01ed2240296cceb97c122271.png
)

这种类型推断具有良好的动机，如果你也有像上述例子相似的场景，在接下来的代码中，并不确定foo到底是number类型还是string类型，这样的问题在大型的多文件代码库中经常能碰到，我们稍后会继续深入了解类型推断的规则。

### 类型可以是显式的

根据我们在之前提到的，typescript将会尽能安全的进行类型的推断，然而，也可以使用注解去明确达到下面两个目的

  - 有助于编译器进行友好的提示，还有就是对于那些不得不阅读你代码的开发者，文档的填充也是非常重要的
  - 强制让在编译器上看到的提示就是你希望让用户（阅读你代码的开发者）看到的提示，也就是说，你对于代码的理解匹配了代码的算法分析（这一步通过编译器来完成）

typescript是用尾随式的类型注解

```javascript
  var foo: number = 123;
```

下面这个例子编译器将会抛出一个error

```javascript
  var foo: number = '123'; // Error: cannot assign a `string` to a `number`
```

我们会在后续章节讨论注解语法的细节

### typescript是结构化的

在typescript中，我们想让js开发者以更小的学习成本来编写ts代码，所以类型都是结构化的，这意味着，”鸭子类型“是一种类(class)语言优先的结构，考虑下面这个例子，函数iTakePoint2D将会接收任何包含了x和y的对象作为参数

```javascript
  interface Point2D {
    x: number;
    y: number;
  }
  interface Point3D {
      x: number;
      y: number;
      z: number;
  }
  var point2D: Point2D = { x: 0, y: 10 }
  var point3D: Point3D = { x: 0, y: 10, z: 20 }
  function iTakePoint2D(point: Point2D) { /* do something */ }

  iTakePoint2D(point2D); // exact match okay
  iTakePoint2D(point3D); // extra information okay
  iTakePoint2D({ x: 0 }); // Error: missing information `y`
```

### 类型错误不会阻止js代码的运行

为了使js代码迁移到ts代码更为简单，即使是有编译错误，默认的，typescript也会触发有效的js代码使其正常执行

```javascript
  var foo = 123;
  foo = '456'; // Error: cannot assign a `string` to a `number`
```

等价于触发下面这段js代码

```javascript
  var foo = 123;
  foo = '456';
```

所以从js代码过渡到ts代码可以采用逐渐更新升级的策略，这也是ts不同于其他语言编译器工作以及迁移到ts的原因


### 类型可以是调节粒度的

typescript的一个主要的设计目标就是可以在typescript尽可能简单和安全的使用已经存在的js库，typescript通过声明（declaration）来达到这个目的，typescript提供了一个可变的比例针对你想在声明文件中放置多少声明信息，声明的越具体，类型检测和代码提示就越详细，注意，针对大多数流行的js库已经有写好的声明文件[https://github.com/borisyankov/DefinitelyTyped](DefinitelyTyped community), 所以针对大多数的目的：

  - 声明文件已经存在
  - 或者至少，已经有大量的经过了review的声明模版可用了

为了快速定义一个自己的声明文件，以jQuery为例，默认的，在你使用一个变量之前，typescript都期望你首先要声明它

```javascript
  $('.awesome').show(); // Error: cannot find name `$`
```

为了快速解决这个问题，你可以告诉typescript，这里确实有一个叫做$的家伙

```javascript
  declare var $: any;
  $('.awesome').show(); // Okay!
```

如果你想基于这个基础的定义来提供更多的信息以防止出现编译error，可以这样做

```javascript
  declare var $: {
    (selector:string): any;
  };
  $('.awesome').show(); // Okay!
  $(123).show(); // Error: selector needs to be a string
```

### 现在就能使用js的新特性

typescript提供了很多新特性针对当前的js引擎，typescript团队也在积极的添加这些特性，这份特性列表也会随着时间变得越来越丰富，这里以一个class为例。

```javascript
  class Point {
      constructor(public x: number, public y: number) {

      }
      add(point: Point) {
        return new Point(this.x + point.x, this.y + point.y);
      }
  }

  var p1 = new Point(0, 10);
  var p2 = new Point(10, 20);
  var p3 = p1.add(p2); // { x: 10, y: 30 }
```