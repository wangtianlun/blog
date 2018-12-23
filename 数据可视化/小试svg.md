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




































