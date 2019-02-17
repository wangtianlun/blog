[原文地址](https://medium.com/@rossbulat/how-to-use-typescript-with-react-and-redux-a118b1e02b76)

作者：Ross Bulat

注：本文并非直译

一份用Typescript开发react和redux应用的指南

typescript在增强react应用的稳定性，可读性以及易管理性方面一直都处在非常重要的位置上，typescript为React和其他javascript前端框架逐步引入了更多的支持；从3.0版本和3.1版本之后显著增强了许多功能。在过去，typescript集成到react应用中是一件很头疼的任务，但在这篇文章中，我门就会去探讨一种更为直截了当的方式让typescript和react更轻松的融合。

Create React App现在已经内置支持了typescript，从react-scripts2.1开始，通过添加一个typescript选项，就可以在一个新项目中开启typescript，在探索如何将types和interface集成到React的props和state之前，我们将会使用Create React App去构造一个基于react和typescript应用

### 用Create React App创建typescript项目

Create React App官网有一个专门的页面用来介绍项目创建过程以及给现有的项目添加typescript支持的迁移步骤，创建一个新的app并开启typescript，先要执行以下命令：

```javascript
  yarn create react-app app_name --typescript
  #or
  npx create-react-app app_name --typescript
```

基于Create React App模板创建的项目，针对javascript有几个需要注意的变化

- 现在一份配置了typescript编译器选项的 **tsconfig.json** 文件被引入了进来
- **.js**文件现在要统一变为以 **.tsx** 为后缀的文件，这样typescript编译器就能够在编译时识别出所有的.tsx文件
- **package.json**包含了针对各种@types包的依赖，包括对node，jest，react和react-dom的支持，并且是开箱即用的
- 一份叫做 **react-app-env.d.ts**的文件会去引用 **react-scripts**的类型，通过yarn start启动本地开发服务，这个文件会自动生成。

**yarn start**执行阶段将会编译和运行你的应用，并且会生成一个和原有js版本应用相同的副本

在我们继续往下展开之前，最好先停止开发服务，重新考虑一些针对Typescript和React的检查工具。


### 下载tslint-react
引入检查工具对于开发typescript和react应用来说非常有帮助，你可以根据提示去得到一个确切的类型，尤其是事件（events）类型，检查工具有极其严格的默认配置，所以我们在安装过程中忽略一些规则。

注意：这些检查工具通过NPM或者Yarn进行安装

全局安装typescript，tslint，tslint-react

```javascript
  yarn global add tslint typescript tslint-react
```

现在在你的项目目录下，初始化tslint

```javascript
  tslint --init
```

上述命令会生成一个拥有一些默认配置选项的tslint.json文件，将文件内容替换成如下内容

```javascript
  {
    "defaultSeverity": "error",
    "extends": [
      "tslint-react"
    ],
    "jsRules": {
    },
    "rules": {
      "member-access": false,
      "ordered-imports": false,
      "quotemark": false,
      "no-console": false,
      "semicolon": false,
      "jsx-no-lambda": false
    },
    "rulesDirectory": [
    ],
    "linterOptions": {
      "exclude": [
        "config/**/*.js",
        "node_modules/**/*.ts"
      ]
    }
  }
```

简单说明一下文件中的各个选项都是用来干嘛的

- **defaultSeverity**规定了错误处理的级别，**error**作为默认值将会在你的IDE里出现红色的错误提示，然而**warning**将会展现橘黄色的警告提示
- **"extends": ["tslint-react"]**: 我们扩展的规则是基于已经删除了的**tslint-recommended**库，之所以没有使用tslint-recommended库，是因为该库有些规则并没有遵循React语法
- **"rules": {"rule-name": false, ...}**: 我们可以在rules对象内忽略一些规则，比如，忽略**member-access**规则，以阻止tslint报出缺少函数访问类型的提示，因为在react中成员访问关键字（public，privaye）并不常用，另外一个例子，**ordered-imports**，这个规则会提示我们根据字母顺序排列我们的import的语句，所有可用的规则可以点击这里进行查看[这里](https://palantir.github.io/tslint/rules/)
- **"linterOptions": {"exclude": [...]}**: 在这里我们排除了所有在config目录下的js后缀的文件和在node_modules目录下的ts文件以避免tslint的检查

我们可以在组件的props以及state上应用interface和type

#### 定义interface

当我们传递props到组件中去的时候，如果想要使props应用interface，那就会强制要求我们传递的props必须遵循interface的结构,确保成员都有被声明，同时也会阻止未期望的props被传递下去。

interface可以定义在组件的外部或是一个独立文件，可以像这样定义一个interface

```javascript
  interface FormProps {
    first_name: string;
    last_name: string;
    age: number;
    agreetoterms?: boolean;
  }
```

这里我们创建了一个**FormProps**接口，包含一些值，agreeToterms后面跟着？，代表该成员是可选的，非必传，我们也可以给组件的state应用一个interface

```javascript
  interface FormState {
      submitted?: boolean;
      full_name: string;
      age: number;
  }
```