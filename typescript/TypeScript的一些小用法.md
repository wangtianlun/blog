### 预先定义的条件类型（Predefined conditional types）

TS提供了几种内置的预定义的条件类型

- Exclude<T, U> - 用于从类型T中去除不在U类型中的成员
- Extract<T, U> - 用于从类型T中取出可分配给U类型的成员
- NonNullable<T> - 用于从类型T中去除undefined和null类型
- ReturnType<T> - 获取函数类型的返回类型
- InstanceType<T> - 获取构造函数的实例类型

下面分别来举例：

1. Exclude<T, U>
  
![Exclude<T, U>](https://pic4.zhimg.com/80/v2-80f73a35ec22a222f93d61025da37ba3_hd.jpg)

在这个例子中，因为用到了Exclude这个条件类型，会尝试寻找在T类型中有，但在U类型中没有的成员，最后将获取到的Union类型"b" | "d"赋给T00

2. Extract<T, U>

![Extract<T, U>](https://pic4.zhimg.com/80/v2-55430b0620faa5b72cbe3b64d5c7c0ff_hd.jpg)

Extract首先是取出的意思，应用Extract条件类型，会尝试寻找T类型成员和U类型成员的交集，在该示例中，取到的是"a" | "c"

3. NonNullable<T>

![NonNullable<T>](https://pic4.zhimg.com/80/v2-a556f2f9a5969316cd60a94e8ecb523b_hd.jpg)

通过运行NonNullable，清除了undefined类型成员

4. ReturnType<T>

![ReturnType<T>](https://pic4.zhimg.com/80/v2-50bfcccfb3acdc2a33f4fd0c9d1026f7_hd.jpg)
![ReturnType<T>](https://pic2.zhimg.com/80/v2-3de33fcf897c4190f3c1d2844d3fdc35_hd.jpg)

通过ReturnType，返回了范型参数T的返回值类型

5. InstanceType<T>

![InstanceType<T>](https://pic4.zhimg.com/80/v2-72b617095f675beaa14d7e72437de7b7_hd.jpg)

### Omit

目前的ts版本（3.2及以前）并没有内置Omit，那么Omit是什么呢？开始我对这个Omit也很好奇，在很多开源的实现里都能看到它的身影。Omit本身有省略和删除的意思，那在ts里这个Omit也很有可能有着相似的操作。查了一些资料之后，学习到，Omit确实是用来删除指定成员。ts虽然没有原生内置Omit，但却可以通过其他内置类型进行实现，比如：

![Omit](https://pic4.zhimg.com/80/v2-5ffcaf1c98840b403ea09d5014675923_hd.jpg)

搭配Pick这个映射类型，就可以实现Omit的效果，我们一点点来拆分这个类型实现。

首先看等号的右侧，Pick是一个ts内置的映射类型，Pick的实现为

```javascript
  type Pick<T, K extends keyof T> = {
    [P in K]: T[P];
  }
```

首先这个 "K extends keyof T"说明这个类型值必须为T类型属性的子集，也就是说假如有一个interface定义如下

```javascript
  interface Student {
    name: string;
    age: number;
  }
```

在传入这个Student到Pick中时

```javascript
  type PickStudent1 = Pick<Student, "name"> // 正确
  type PickStudent2 = Pick<Student, "score"> // 错误
```

在上面的Omit实现中，我们用到了Exclude这个条件类型，根据上文中的说明，Exclude的效果就是寻找在keyof T中有，但在K类型中没有的成员，这样就将剩余的类型过滤了出来，再去应用Pick，就获得了我们的Omit实现。

完整例子：


![Omit](https://pic2.zhimg.com/80/v2-5f7502a89519af11aed0ae8154a5d951_hd.jpg)