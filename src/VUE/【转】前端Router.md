# [javascript基础修炼(6)——前端路由的基本原理](https://www.cnblogs.com/dashnowords/p/9671213.html)

![img](http://i2.51cto.com/images/blog/201809/18/72c50a0bd74600abae342a163816fc89.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

> 【造轮子】是笔者学习和理解一些较复杂的代码结构时的常用方法，它很慢，但是效果却胜过你读十几篇相关的文章。为已知的API方法自行编写实现，遇到自己无法复现的部分再有针对性地去查资料，最后当你再去学习官方代码的时候，就会明白这样做的价值，总有一天，你也将有能力写出大师级的代码。

## 一. 前端路由

现代前端开发中最流行的页面模型，莫过于`SPA`单页应用架构。单页面应用指的是应用只有一个主页面，通过动态替换DOM内容并同步修改url地址，来模拟多页应用的效果，切换页面的功能直接由前台脚本来完成，而不是由后端渲染完毕后前端只负责显示。前端三驾马车`Angular`,`Vue`,`React`均基于此模型来运行的。`SPA`能够以模拟多页面应用的效果，归功于其**前端路由机制**。

> 前端路由，顾名思义就是一个**前端不同页面的状态管理器**,可以不向后台发送请求而直接通过前端技术实现多个页面的效果。angularjs中的`ui-router`,vue中的`vue-router`,以及react的`react-router`均是对这种功能的具体实现。

既然`前端路由`这么牛逼，那必须的好好研究一下。

## 二. 两种实现方式及其原理

> 常见的路由插件中两种方式都是支持且可以切换的，例如`angularjs1.x`中就可以通过以下代码从**Hash**模式切换到**H5**模式：
> $locationProvider.html5Mode(true);
> 切换到**HTML5**的路由模式，主要用于避免url地址中包含**#**而引发的问题。

### 1.HashChange

#### 1.1 原理

`HTML`页面中通过锚点定位原理可进行无刷新跳转，触发后url地址中会多出`# + 'XXX'`的部分，同时在全局的`window`对象上触发`hashChange`事件，这样在页面锚点哈希改变为某个预设值的时候，通过代码触发对应的页面DOM改变，就可以实现基本的路由了,基于`锚点哈希`的路由比较直观，也是一般前端路由插件中最常用的方式。

#### 1.2 应用

下面通过一个实例看一下,当点击`angularjs`的连接时，可以看到控制台打印出了相应的信息。
![img](http://i2.51cto.com/images/blog/201809/18/dccf958c41d5737b9427a20263991231.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)
![img](http://i2.51cto.com/images/blog/201809/18/0c734e08a3cae1a46da3af1ad84203bb.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

### 2.HTML5 HistoryAPI

#### 2.1 原理

`HTML5`的`History API`为浏览器的全局`history`对象增加的扩展方法。**一般用来解决ajax请求无法通过`回退`按钮回到请求前状态的问题**。

在HTML4中，已经支持`window.history`对象来控制页面历史记录跳转，常用的方法包括：

- **history.forward()**; //在历史记录中前进一步
- **history.back()**; //在历史记录中后退一步
- **history.go(n)**: //在历史记录中跳转n步骤，n=0为刷新本页,n=-1为后退一页。

在HTML5中，`window.history`对象得到了扩展，新增的API包括：

- **history.pushState(data[,title][,url])**;//向历史记录中追加一条记录
- **history.replaceState(data[,title][,url])**;//替换当前页在历史记录中的信息。
- **history.state**;//是一个属性，可以得到当前页的state信息。
- **window.onpopstate**;//是一个事件，在点击浏览器后退按钮或js调用forward()、back()、go()时触发。监听函数中可传入一个event对象，event.state即为通过pushState()或replaceState()方法传入的data参数。

#### 2.2 应用

浏览器访问一个页面时，当前地址的状态信息会被压入`历史栈`,当调用`history.pushState()`方法向历史栈中压入一个新的`state`后，历史栈顶部的指针是指向新的`state`的。可以将其作用简单理解为 *假装已经修改了url地址并进行了跳转* ,除非用户点击了浏览器的`前进`,`回退`,或是显式调用HTML4中的操作历史栈的方法，否则不会触发全局的`popstate`事件。

在下面的示例中，点击导航按钮，可以看到`url`地址栏发生了变化，且控制台打印出了响应的信息。
![img](http://i2.51cto.com/images/blog/201809/18/b0c4fb18462ba620339eda2b75f28263.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)
![img](http://i2.51cto.com/images/blog/201809/18/b288ef5dccfabc6c609648d178c252a4.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

### 3.hash 和 history API对比

| 对比         | hash路由                               | History API 路由                                            |
| ------------ | :------------------------------------- | :---------------------------------------------------------- |
| url字符串    | 丑                                     | 正常                                                        |
| 命名限制     | 通常只能在同一个`document`下进行改变   | url地址可以自己来定义，只要是同一个域名下都可以，自由度更大 |
| url地址变更  | 会改变                                 | 可以改变，也可以不改变                                      |
| 状态保存     | 无内置方法，需要另行保存页面的状态信息 | 将页面信息压入历史栈时可以附带自定义的信息                  |
| 参数传递能力 | 受到url总长度的限制，                  | 将页面信息压入历史栈时可以附带自定义的信息                  |
| 实用性       | 可直接使用                             | 通常服务端需要修改代码以配合实现                            |
| 兼容性       | IE8以上                                | IE10以上                                                    |

## 三.亲手造一个简单的前端路由插件

> 造轮子,不是为了把它装在你的车上，而是当你在荒郊野外开车而轮子出了问题时多一种选择。

接下来就自己动手实现一个前端路由的插件吧~

### 3.1基于Hash的前端路由插件`myHashRouter.js`

我们希望实现的功能是:

- 1.引入`MyHashRouter.js`库
- 2.通过`when()`方法来定义若干不同的路由状态
- 3.通过`init()`方法启动路由功能
- 4.通过点击导航实现**前端路由**切换

首先编写js骨架，如图所示:

```js
;(function() {
    function Router() {
        //记录路由的跳转历史
        this.historyStack = [];
        //记录已注册的路由信息
        this.registeredRouter = [];
        //路由匹配失败时跳转项
        this.otherwiseRouter = {
            path: '/',
            content: 'home page'
        }
    }

    /*
     * 启动路由功能
     */
    Router.prototype.init = function() {

    }

     /*
     * 绑定window.onhashchange事件的回调函数
     */
    Router.prototype._bindEvents = function() {

    }

    /**
     * 路由注册方法
     */
    Router.prototype.when = function(path, content) {

    }

    /**
     * 判断新添加的路由是否已存在
     */
    Router.prototype._hasThisRouter = function(path) {

    }

    /**
     * 路由不存在时的指定地址
     */
    Router.prototype.otherwise = function(path, content) {

    }

    /**
     * 路由跳转方法，主动调用时可用于跳转路由
     */
    Router.prototype.go = function(topath) {

    }

    /**
     * 用于将对应路由信息渲染至页面,实现路由切换
     */
    Router.prototype.render = function (content) {
     
    }

    var router = new Router();

    //将接口暴露至全局
    window.$router = router;
})();
```

完成了路由插件的编写后，我们在demo中引入该库，然后使用`when()`方法注册几个路由地址，再使用`init()`方法启动路由，脚本部分代码如下：
![img](http://i2.51cto.com/images/blog/201809/18/b67fea3fd75e1dadd2387c4a5e712896.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

效果：
运行附件中的`router-demo-hash.html`,点击导航按钮，即可看到url地址栏以及内容区域同步更改。
![img](http://i2.51cto.com/images/blog/201809/18/3b66cc291801565296dcafcf41b262cb.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

### 3.2基于History API的前端路由插件`myHistoryRouter.js`

由于`History API`不支持低于IE10以下版本的浏览器（其他大多数现代浏览器基本都支持）,所以我们在`init()`方法启动时先进行可用性判断，基本代码框架与基于`Hash`的路由插件一致。每个方法的实现并不难写，这里不再赘述，笔者自己的代码实现放在附件**`myHashRouter.js`**中，水平有限，仅供参考。
![img](http://i2.51cto.com/images/blog/201809/18/7facde2acbf9df812e88855710a2cd5d.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

### 3.3集成说明

为方便理解，本例中将两种模式分开编写，如果是插件库的开发，可以模仿**ui-router**增加一个`html5mode()`的方法,在`init()`方法启动路由时，根据所传的参数生成不同的路由插件的单例，也就是我们常说的**工厂模式**来实现即可。

## 四.后记

- `造车轮`是一个很好的学习方式，虽然自己造的车轮很简陋，但是对于理解工具的底层原理却很有帮助。
- 本例只是编写了一个路由工具的基本骨架，真正的路由工具还需要做很多功能扩展，个别功能的复杂度也会很高，例如**路径的正则匹配**,**懒加载**,**组合视图**,**嵌套视图**,**路由动画**等等，有兴趣的小伙伴可以在本例提供的框架上进行学习扩展。
- 附件说明：
  - **index_h5history.html** —— history API基本用法演示demo
  - **index_hashchange.html** —— hashchange基本用法演示demo
  - **router-demo-hash.html** —— 引用了`myHashRouter.js`的demo
  - **myHashRouter.js** —— 自己开发的基于hash简易路由插件
  - **router-demo-hash.html** —— 引用了`myHashRouter.js`的demo
  - **myHistoryRouter.js** —— 自己开发的基于historyAPI的简易路由插件
  - **router-demo-history.html** —— 引用了`myHistoryRouter.js`的demo