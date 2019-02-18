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

注意：tslint过去会提示我们给每一个interface名称前面加上一个i，比如IFormProps和IFormState。然而默认是不强制加的

#### 给组件应用interface
我们既可以给类组件也可以给无状态组件应用interface。对于类组件，我们利用尖括号语法去分别应用我们的props和state的interface。

```javascript
  export class MyForm extends React.Component<FormProps, FormState> {
    ...
  }
```

注意：在只有state而没有props的情况下，props的位置可以用{}或者object占位，这两个值都表示有效的空对象。

对于纯函数组件，我们可以直接传递props interface

```javascript
  function MyForm(props: FormProps) {
    ...
  }
```

#### 引入interface
按照约定，我们一般会创建一个 **src/types/**目录来将你的所有interface分组：

```javascript
  // src/types/index.tsx
  export interface FormProps {
    first_name: string;
    last_name: string;
    age: number;
    agreetoterms?: boolean;
  }
```

然后引入组件所需要的interface

```javascript
  // src/components/MyForm.tsx
  import React from 'react';
  import { StoreState } from '../types/index';
  ...
```

#### enums
枚举Enums是另外一个typescript有用的功能，假设我们想针对 **MyForm**组件来定一个枚举，然后对提交的表单值进行验证

```javascript
  // define enum
  enum HeardFrom {
      SEARCH_ENGINE = "Search Engine",
      FRIEND = "Friend",
      OTHER = "Other"
  }
  //construct heardFrom array
  let heardFrom = [HeardFrom.SEARCH_ENGINE, 
                  HeardFrom.FRIEND, 
                  HeardFrom.OTHER];

  //get submitted form value
  const submitted_heardFrom = form.values.heardFrom;

  //check if value is valid
  heardFrom.includes(submitted_heardFrom)
    ? valid = true
    : valid = false;
```

#### iterables
在typescript中我们可以使用 **for...of**和 **for...in**方法来进行循环遍历。这两个方法有一个很重要的区别：
  - for...of方法会返回被迭代对象的键(key)的列表
  - for...in方法会返回被迭代对象的值(value)的列表

```javascript
  for (let i in heardFrom) {
   console.log(i); // "0", "1", "2",
  }
  for (let i of heardFrom) {
    console.log(i); // "Search Engine", "Friend", "Other"
  }
```

#### Typing Events
如果你希望比如**onChange**或者**onClick**事件利用语法工具可以获取明确的你所需要的事件。
可以考虑下面这个例子，通过将光标悬浮在**handleChange()**方法上，我们就可以清晰的看到真实的事件类型**React.ChangeEvent<HTMLInputElement>:**：

![event type](https://img.souche.com/f2e/a554b54a7c5782faf9fd81e58933c8a1.png)

然后在我们的handleChange函数定义中传入e这个参数的时候会用到这个类型

我们也可以给e对象中的name和value指定类型，通过下面的语法：

```javascript
  const {name, value}: {name: string; value: string;} = e.target;
```

如果你不知道对象该指定什么类型，你可以使用**any**类型，就像下面这样

```javascript
  const {name, value}: any = e.target;
```

现在我们已经学会了一些基本的示例，接下来一起来看看typescript如何与redix搭配。探索typescript更多的功能

### Redux with Typescript

##### Step1:给Store指定类型
首先，我们想要给我们的Redux store定义一个interface。定义合理的state结构将有利于你的团队及更好的维护应用的状态

这部分可以在我们先前讨论过的 **/src/types/index.tsx**文件中完成，下面是一个试图解决位置与身份认证的示例：

```javascript
  // src/types/index.tsx
  export interface MyStore {
    language: string;
    country: string;
    auth: {
        authenticated: boolean;
        username?: string;
    };
  }
```

##### Step2:定义action的类型以及actions
所有的action类型可以用一种 **const & type**的模式来进行定义，我们首先在 **src/constants/index.tsx**文件中定义action types：

```javascript
  // src/constants/index.tsx
  export const SET_LANGUAGE = 'INCREMENT_ENTHUSIASM';
  export type SET_LANGUAGE = typeof SET_LANGUAGE;
  export const SET_COUNTRY = 'SET_COUNTRY';
  export type SET_COUNTRY = typeof SET_COUNTRY;
  export const AUTHENTICATE = 'AUTHENTICATE';
  export type AUTHENTICATE = typeof AUTHENTICATE;
```

注意到如何让我们刚刚定义的常量被用作interface类型还是字符串字面量，我们会在后面进行使用讲解

这些**const & type**所组成的对象现在可以在**src/actions/index.tsx**文件中进行导入了，这里我们定义了action interface以及action自身，以及对它们都指定了类型

```javascript
  // src/actions/index.tsx
  import * as constants from '../constants';

  //define action interfaces
  export interface SetLanguage {
      type: constants.SET_LANGUAGE;
      language: string;
  }
  export interface SetCountry {
      type: constants.SET_COUNTRY;
      country: string;
  }
  export interface Authenticate{
      type: constants.AUTHENTICATE;
      username: string;
      pw: string;
  }

  //define actions
  export function setLanguage(l: string): SetLanguage ({
      type: constants.SET_LANGUAGE,
      language: l
  })
  export function setCountry(c: string): SetCountry ({
      type: constants.SET_COUNTRY,
      country: c
  })
  export function authenticate(u: string, pw: string): Authenticate ({
      type: constants.SET_COUNTRY,
      username: u,
      pw: pw
  })
```

在authenticate action中，我们传入了username和password两个参数，两个参数都是string类型，返回值也指定了类型，在这个示例中是**Authenticate**

在Authenticate interface内部，我们也包括了有效的action所需要的username和pw的值

##### Step3:定义Reducers

为了简化在reducer中指定一个action type的过程，我们可以利用联合类型，这个特性是在typescript1.4版本之后引入进来的，联合类型允许我们将两种或两种以上的类型合并为一个类型

回到我们的actions文件，给我们表示位置的interface添加一个联合类型

```javascript
  // src/actions/index.tsx
  export type Locality = SetLanguage | SetCountry;
```

现在我们就可以将**Locality**类型应用到我们reducer函数中的action

```javascript
  // src/reducers/index.tsx
  import { Locality } from '../actions';
  import { StoreState } from '../types/index';
  import { SET_LANGUAGE, SET_COUNTRY, AUTHENTICATE} from '../constants/index';
  export function locality(state: StoreState, action: Locality):     StoreState {
    switch (action.type) {
      case SET_LANGUAGE:
        return return { ...state, language: action.language};
      case SET_COUNTRY:
        return { ...state, language: action.country};
      case AUTHENTICATE:
        return { 
          ...state, 
          auth: {
              username: action.username, 
              authenticated: true 
            }
        };
    }
    return state;
  }
```

尽管已经全部指定了类型，这个reducer相对来说也是非常直观

- 这个命名为locality的reducer，将state指定为StoreState类型，以及将action指定为Locality类型
- 这个reducer将会返回一个StoreState类型的对象，如果并没有匹配到任何的action就将原state返回
- 我们的 constant & type（常量和类型）对在这里也被得到应用，作为action间切换的条件


##### Step4:创建初始化Store

利用尖括号传入类型联同**createStore()**，在**index.ts**文件中我们可以初始化store了

```javascript
  // src/index.tsx
  import { createStore } from 'redux';
  import { locality } from './reducers/index';
  import { StoreState } from './types/index';
  const store = createStore<StoreState>(locality, {
    language: 'British (English)',
    country: 'United Kingdom',
    auth: {
        authenticated: false
    }
  });
```

已经快要完成了，现在已经覆盖了集成typescript到redux中的大部分步骤了，再坚持一下，让我们来看一下容器组件(container component)所需要的**mapStateToProps**和**mapDispatchToProps**

### Mapping State and Dispatch
在**mapStateToProps**内部，记得将state参数设定为StoreState类型，第二个参数ownProps也可以指定一个props的interface：

```javascript
  //mapStateToProps example
  import { StoreState } from '../types/index';
  interface LocalityProps = {
      country: string;
      language: string;
  }
  export function mapStateToProps(state: StoreState, ownProps: LocalityProps) {
    return {
      language: state.language,
      country: state.country,
    }
  }
```

**mapDispatchToProps**有些不同，我们利用尖括号想Dispatch方法中传入一个interface，然后，在返回代码块中dispatch我们**Locality**类型的action：

```javascript
  //mapDispatchToProps example
  export function mapDispatchToProps(dispatch: Dispatch<actions.Locality>) {
      return {
          setLanguage: (l: string) => 
              dispatch(actions.setLanguage(l)),
          
          setCountry: (c: string) => 
              dispatch(actions.setCountry(c))
      }
  }
```

最后，我们就可以和组件进行连接

```javascript
  export default connect(mapStateToProps, mapDispatchToProps)(MyComponent);
  ...
```

### 总结

这篇文章介绍了typescript和react如何联合以及如何利用tslint-react进行更加平缓的开发。我们已经了解到如何在组件中让props和state应用interface，同样也了解到了在Typescript中如何处理事件。最终，我们了解了typescript如何集成到Redux中。

将typescript集成到react项目中，的确会增加一些额外的成本，但随着应用范围的扩大，支持typescript语言一定会增加项目的维护性和管理性

使用typescript会促进模块化和代码分隔，记住，随着项目的扩大。如果你发现了维护性上面的问题，typescript不仅可以提升代码的可读性，同时也会降低错误发生的可能性。