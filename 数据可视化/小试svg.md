### 基本概念

svg(Scalable Vector Graphics)是一种基于XML语法的图像格式，全称是可缩放矢量图，其它图像格式都是基于像素处理的，SVG则是属于对图像的形状描述，所以它本质上是文本文件，体积较小，且不管放大多少倍都不会失真.SVG是面向图形，HTML时面向文本。

### 嵌入到HTML

SVG可以写在一个独立的文件中，然后用<img>, <object>, <embed>, <iframe>等标签插入网页

```javascript
  <img src="circle.svg">
  <object id="object" data="circle.svg" type="image/svg+xml"></object>
  <embed id="embed" src="icon.svg" type="image/svg+xml">
  <iframe id="iframe" src="icon.svg"></iframe>
```

SVG文件可以转为base64编码，然后作为Data URI写入网页

```javascript
  <img src="data:image/svg+xml;base64,[data]" />
```

### SVG书写的注意点

  - SVG的元素和属性必须按照标准格式来写，因为XML是确认大小写的
  - SVG里的属性值必须用引号引起来，就算是数值也必须这么做
  - SVG图像的默认大小是300像素（宽）x 150像素（高）
  - 后面的元素会渲染在前面元素之上

### SVG的所有元素

  [SVG的所有元素](https://developer.mozilla.org/zh-CN/docs/Web/SVG/Element)

### SVG的所有属性

  [SVG的所有属性](https://developer.mozilla.org/zh-CN/docs/Web/SVG/Attribute)


### 常用的形状元素

  ![常用的形状元素](http://img.souche.com/f2e/3707937a28d2b2cf4acd48eaf6a41272.png)

其实上图只是对一些常用svg标签的初步认识，因为svg所提供的标签不止这些，而且比如path标签是在svg中最为通用的形状标签，因为它可以通过设置路径画出其它图形，比如矩形，圆，椭圆，多边形，多线段，甚至是复杂的贝塞尔曲线等等

### <path>

第一次看到svg的<path ...>标签的时候，打开控制台，也是一脸懵逼，首先这里面的d属性是个啥，M是啥，L是啥，Z是啥，H是啥，V是啥，C是啥，S是啥，Q是啥，T是啥，A是啥，我...  打扰了，打扰了

  ![path](http://img.souche.com/f2e/696bd22a9cc33291655e5b0cf4c6c8dd.png)

嗯，26个兄弟快凑齐了，马上就可以召唤神龙了。当然，path这条神龙在svg界就是“爸爸”，啥玩意都能给你弄出来，

想要通过path勾勒出美妙的图形，需要了解d这个属性，path标签中的d属性可以定义一系列的指令和参数，每一个指令通过一个字母来指定，比如上面说的M，它表示移动到，也就是"move to"的意思，比如让我们移动到（10, 10）的坐标点，就可以这样写：

```javascript
  <rect d="M10 10" />'
```

当然每一种字母都是区分大小写的，比如M是基于画布上的一个绝对坐标，而m则是基于上一个点的坐标，也就是相对坐标。比如有下面两种指令

```javascript
  <path d="M20,20 L40 40 M60 60 L80 80" fill="none" stroke="blue" stroke-width="5"/>

  <path d="M20,20 L40 40 m60 60 L80 80" fill="none" stroke="blue" stroke-width="5"/>
```

![path](http://img.souche.com/f2e/614e999f183c9e1466d6f6d2c89bb64f.png)

两个path唯一的区别就是第三个指令，一个是M60 60, 一个是m60 60

#### 线段指令（Line commands）

  - L：L指令会拿到两个参数，x坐标和y坐标，然后从当前位置到指定参数坐标位置来绘制线段
  - H：H其实是horizontal的缩写，意为绘制出水平方向的线段，因为方向已确定，所以只需一个参数就能完成线段的绘制
  - V：同H同理，只不过表示垂直方向（vertical）的线段绘制

比如用H和V来绘制一个矩形, 我们一步一步来

  - step1

  ```javascript
    <path d="M10 10 H 90" fill="none" stroke="blue"/>
  ```

  ![step1](http://img.souche.com/f2e/9edcc46307a37cfef748cd763a25784c.png)

  - step2

  ```javascript
    <path d="M10 10 H 90 V 90" fill="none" stroke="blue"/>
  ```

  ![step2](http://img.souche.com/f2e/95c6b0385d824a5b76290401f0e75ec2.png)

  - step3

  ```javascript
    <path d="M10 10 H 90 V 90 H 10" fill="none" stroke="blue"/>
  ```

  ![step3](http://img.souche.com/f2e/98f310ac5cb2a8c288a8cb53d85e2019.png)

  - step4

  ```javascript
    <path d="M10 10 H 90 V 90 H 10 V 10" fill="none" stroke="blue"/>
  ```

  ![step4](http://img.souche.com/f2e/c86a8d55c63fba7ebdeddc8d670f073e.png)


上面的写法也可以通过一个指令来简写一下，这就用到了Z指令

**Z：该指令的作用是从当前位置向起始点画出一条线段，它一般都被放置在一连串节点的末尾，并且不区分大小写。可以理解为”闭环“指令**

所以上例可以写成这样，也能达到同样的效果

  ```javascript
    <path d="M10 10 H 90 V 90 H 10 Z" fill="none" stroke="blue"/>
  ```
同样，上例也可以通过相对定位的形式进行改写，效果是一致的

  ```javascript
    <path d="M10 10 h 80 v 80 h -80 Z" fill="transparent" stroke="blue"/>
  ```

![relative](http://img.souche.com/f2e/baf5f2123adbd2245eed6547b8289f72.png)


#### 曲线指令（Curve commands）

一说到曲线，那贝塞尔曲线是绕不开的，对于曾高数挂科的我来说是很排斥的，但好在闲着蛋疼，遂学之。

path标签中有两类贝塞尔曲线，一种叫做“三次贝塞尔曲线（cubic curve）“， 一种叫做”二次贝塞尔曲线（quadratic curve）“，这名字听起来就不接地气。

那先从三次贝塞尔曲线说起

**C：该指令用于创建一个三次贝塞尔曲线，需指定三组参数**

比如：

  ```javascript
    <path d="M10 10 C 20 20, 40 20, 50 10" stroke="black" fill="transparent"/>
  ```

首先，（20 20）和（40 20）表示控制节点，一个是描述曲线起始点的斜率，另一个是描述曲线终止点的斜率，最后一组（50 10）表示曲线的终点。总结一下这段示例，就是有一条从（10 10）到（50 10）的一条线段，通过设置两个控制点的斜率，使这条线段的各个点弯曲成正确的（符合斜率趋势的）曲线。

MDN上有多组曲线的对比示例。

![MDN上有多组曲线的对比示例](http://img.souche.com/f2e/aad38bbec9e8bac6c04e195086ee7f3d.png)

这里面我们再添加一种情况，就是设置两个水平的控制节点，来看看线段是如何变化的

  ```javascript
    <path d="M10 10 C 10 10, 40 10, 50 10" stroke="black" fill="transparent"/>
  ```
![水平](http://img.souche.com/f2e/23ee5fe3e0f64154e6155b794705a74c.png)

通过S指令能生成和上述示例中同样的平滑曲线，使用S指令分为以下两种情况

  - S指令跟在C或者另一个S指令之后：那S指令的开始控制节点就是基于前一个控制节点的对称点，并且S指令指定的第一组节点是结束控制节点
  - 单独的S指令：两个控制节点会被设置为同一个点

比如如下代码

  ```javascript
    <svg
      xmlns="http://www.w3.org/2000/svg"
      width="190"
      height="160"
    >
      <path d="M10 80 C 40 10, 65 10, 95 80 S 150 150, 180 80" stroke="black" fill="transparent"/>

      <circle cx="10" cy="80" r="2" fill="red"/>
      <circle cx="95" cy="80" r="2" fill="red"/>
      <circle cx="180" cy="80" r="2" fill="red"/>
      <circle cx="150" cy="150" r="2" fill="red"/>
    </svg>
  ```
我们通过不断改变S的第一组节点来看图形的变化趋势

![S](http://img.souche.com/f2e/79c449edc1ac317fab72d6b0e2af8212.png)

我们可以看到，随着不断给S指令结束控制节点的横坐标累加，曲线会向右偏移。

接下来看下S指令前面没有其他C或者S指令的情况,代码如下

  ```javascript
    <svg
      xmlns="http://www.w3.org/2000/svg"
      width="300"
      height="300"
    >
      <path d="M10 80 S 95 150, 180 80" stroke="black" fill="transparent"/>

      <circle cx="10" cy="80" r="2" fill="red"/>
      <circle cx="95" cy="150" r="2" fill="red"/>
    </svg>
  ```

![S](http://img.souche.com/f2e/30cca1678f7233e27dbd2b35bfe9deb0.png)

### 另一种曲线是二次贝塞尔曲线（quadratic curve）

它通过指令Q来来进行描述，相较于三次贝塞尔曲线，它更为简单。

**Q：只需要指定两组参数，第一组表示控制节点的坐标，第二组表示终点坐标。**

示例：

  ```javascript
    <svg
      xmlns="http://www.w3.org/2000/svg"
      width="300"
      height="300"
    >
      <path d="M10 80 Q 95 10 180 80" stroke="black" fill="transparent"/>

      <circle cx="10" cy="80" r="2" fill="red"/>
      <circle cx="95" cy="20" r="2" fill="red"/>
      <circle cx="180" cy="80" r="2" fill="red"/>
    </svg>
  ```

![Q](http://img.souche.com/f2e/83c7c43e73634030efd05ec37583e2b5.png)

和三次贝塞尔类似，二次贝塞尔也提供了快捷的玩法，那就是T指令

**T：通过找到前一个控制节点，来推断出一个新的控制点，T指令后面只需要指定一组结束点坐标即可，由于T指令是基于前一个控制点的基础上来生成的，所以T指令之前必须要有Q指令或者其他T指令，否则生成的控制节点就和前一个控制节点就会重合，在画布上看到的就仅仅是一条直线。**


示例：

  ```javascript
    <svg
      xmlns="http://www.w3.org/2000/svg"
      width="300"
      height="300"
    >
      <path d="M10 80 Q 52.5 10, 95 80 T 180 80" stroke="black" fill="transparent"/>

      <circle cx="10" cy="80" r="2" fill="red"/>
      <circle cx="52.5" cy="10" r="2" fill="red"/>
      <circle cx="95" cy="80" r="2" fill="red"/>
      <circle cx="180" cy="80" r="2" fill="red"/>

    </svg>
  ```

![S](http://img.souche.com/f2e/9ef8040dc10cf11cb724cc13a66fd71d.png)

在上面几个例子中，两种曲线都生成了同样的结果，虽然三次贝塞尔允许更多的自由度，但是决定使用哪种曲线还要依照具体情形以及对称曲线的数量来定

### 弧度（Arcs）

在svg中也可以创建弧度这种曲线，它通过A指令来指定，A指令可以接收7个参数

  1. rx：x轴半径
  2. ry：y轴半径
  3. x-axis-rotation：弧形的旋转角度
  4. large-arc-flag：决定弧线是大于180度好事小于180度，0表示小角度弧，1表示大角度弧
  5. sweep-flag：表示弧线的方向，0表示从起点到终点沿逆时针画弧，1表示从起点到终点沿顺时针画弧
  6. x：弧形终点的横坐标
  7. y：弧形终点的纵坐标


示例：

  ```javascript
  <?xml version="1.0" standalone="no"?>
  <svg width="325px" height="325px" version="1.1" xmlns="http://www.w3.org/2000/svg">
    <path d="M80 80
            A 45 45, 0, 0, 0, 125 125
            L 125 80 Z" fill="green"/>
    <path d="M230 80
            A 45 45, 0, 1, 0, 275 125
            L 275 80 Z" fill="red"/>
    <path d="M80 230
            A 45 45, 0, 0, 1, 125 275
            L 125 230 Z" fill="purple"/>
    <path d="M230 230
            A 45 45, 0, 1, 1, 275 275
            L 275 230 Z" fill="blue"/>
  </svg>
  ```

![S](http://img.souche.com/f2e/73ce1c6eede3924c0ad6ac5cf727bfdd.png)

### 饼图

通过学习path，我们来绘制一个简单的饼图

  ```javascript
    <svg width="325" height="325" xmlns="http://www.w3.org/2000/svg">
      <path d="M80 80
              A 45 45, 0, 0, 0, 125 125
              L 125 80 Z" fill="green"/>
      <path d="M170 80
              A 45 45, 0, 0, 1, 125 125
              L 125 80 Z" fill="red"/>
      <path d="M170 80
              A 45 45, 0, 0, 0, 125 35 
              L 125 80 Z" fill="blue" />
      <path d="M80 80
              A 45 45, 0, 0, 1, 125 35
              L 125 80 Z" fill="pink"/>
    </svg>
  ```

![pie](http://img.souche.com/f2e/8c6b964d8d6d19739e5e1f24e315884e.png)

### 小结

  在最近的一些项目中，接触到了部分有关svg的需求，所以这篇文章就是记录下自己在学习svg的一部分总结，比较基础，方便自己今后的复习和查阅。



































































