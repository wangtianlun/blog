### 主进程和渲染进程
在Electron应用中，通过执行package.json中的main字段所指向的文件，可以开启electron的主进程（main process）。在主进程中可以通过创建web页面的方式来展示出图形用户界面。而且一个electron应用有且只能有一个主进程。

由于electron使用Chromium来展示web页面，Chromium多进程架构也会被用到。每一张web页面都运行在它自己的进程里，该进程称为渲染进程（renderer process）

对于一般的浏览器来说，web网页是跑在一个沙盒环境下的并且不允许与系统层面的资源进行交互。然而electron通过使用nodejs在页面上，赋予了用户与系统底层进行交互的能力。



### 主进程和渲染进程的区别

主进程通过创建BrowserWindow实例来创建页面，每一个BrowserWindow实例都会在一个独立的渲染进程中去渲染页面。当一个BrowserWindow实例被删除，它对应的渲染进程也会终止运行

主进程管理着所有的页面以及页面对应的渲染进程，每一个渲染进程都是独立的，而且只关心跑在它上面的web页面

在web页面里，调用系统底层的API是不被允许的，这是因为在web页面上处理底层GUI资源是非常危险的，很容易导致资源泄漏。如果你想要在web页面上执行GUI操作，相应web页面的渲染进程必须与主进程进行通信，向主进程发起请求去执行那些操作.在electron中，有几种主进程与渲染进程通信的方法，比如用ipcRenderer和ipcMain模块来发送信息，还有RPC风格的远程通信模块。



### 使用electron的API

Electron提供了多个API供开发者在主进程及渲染进程上面的开发。在两种进程中，你都可以通过require('electron')来引入electron模块。所有的Electron API都会被分配到一个进程类型。一些API只能用在主进程中，一些API只能用于渲染进程中。还有一些API两种进程可以通用。electron的API文档会声明每一个API可以被用于哪一种进程

Electron中通过BrowserWindow class来实例化创建一个窗口（window），实例化窗口的动作只能在主进程中进行.实际上，如果在渲染进程的代码里引入BrowserWindow对象，获得的其实是一个undefined。

由于进程间通信是可能的，所以一个渲染进程可以基于主进程执行一些task。electron提供了一个叫做remote模块，remote模块提供了一种简单的进程间通讯的方式。在Electron中，GUI相关的模块（比如dialog，menu等）只能在主进程中进行使用，为了在渲染进程中也能使用它们，那就要通过ipc模块给主进程发送消息。但有了remote模块，你可以直接使用主进程的对象而不需要发送确切的进程消息，比如在renderer.js中去加载一个远程页面

![electron](https://pic2.zhimg.com/80/v2-20d486f6c283d1bcef3857aca325378d_hd.jpg)
展示效果为下图：
![electron](https://pic1.zhimg.com/80/v2-de5f36e6311d92bf6d8cb17883fca4d4_hd.jpg)

### 使用node

不管是在主进程中还是渲染进程中，都可以使用nodejs模块

- node所提供的API都适用于electron，如果你曾尝试过加载远程内容的时候，可能会带来一些安全问题。这部分问题可以看这个https://electronjs.org/docs/tutorial/security
- 你可以在应用中使用node模块，npm也提供了全世界最大的开源仓库

  这里还有一点需要注意，Native Node模块需要被编译之后才能在electron中进行使用。但好在大多数的nodejs模块代码不是底层的，如果确实是需要这部分功能，可以参考这个https://electronjs.org/docs/tutorial/using-native-node-modules

### vscode

我们日常开发使用的编辑器vscode也是通过electron构建的，这是vscode的架构图

![vscode](https://pic1.zhimg.com/80/v2-a2583443bc73c19fa18141d72a1a9b5c_hd.jpg)

可以看到vscode中也包含主进程，渲染进程，同时因为vscode提供了插件的扩展能力，又出于安全稳定性的考虑，上图中又多了一个Extension Host，其实这个Extension Host也是一个独立的进程，用于运行我们的插件代码。并且同渲染进程一样，彼此都是独立互不影响的。Extension Host Process暴露了一些vscode的API供插件开发者去使用。