# vue之SEO优化-预渲染

> [原文链接](https://zhuanlan.zhihu.com/p/337866915)

## 前言

- 先了解什么是seo？
- 再了解搜索引擎蜘蛛的工作原理？
- seo为啥对vue单页面不友好?
- vue项目怎么做seo优化？
- prerender-spa-plugin怎么使用，
- prerender-spa-plugin原理探究
- prerender-spa-plugin不能解决的问题
- 静态页面分配title和meta标签----[vue-meta-info](https://link.zhihu.com/?target=https%3A//www.npmjs.com/package/vue-meta-info)

### 什么是seo？

SEO是由英文Search Engine
Optimization缩写而来， 中文意译为“搜索引擎优化”。SEO是指通过对网站进行站内优化和修复(网站Web结构调整、网站内容建设、网站代码优化和编码等)和站外优化，从而提高网站的网站关键词排名以及公司产品的曝光度。通过搜索引擎查找信息是当今网民们寻找网上信息和资源的主要手段。

## 引擎蜘蛛的工作原理？

搜索引擎蜘蛛的爬行是被输入了一定的规则的，它需要遵从一些命令或文件的内容。
网络爬虫在爬取网页内容的时候，需要分析页面内容，主要有以下几点：

- 从 meta 标签中读取 keywords 、 description 的内容。
- 根据**语义化的 html 的标签**爬取和分析内容。一个整体都是用 div 标签的网站和正确使用了 html5 标签的效果是不一样的。
- 读取**a 标签里的链接**，通过 a 标签的链接可以跳转到别的网站。（爬虫是先跳转，还是继续爬内容再跳转，就看算法是广度优先还是深度优先了）
- 像**h1 - h6 标签**是具有不同程度的强调意义的。
- 一般将 **h1** 视为重要内容。同样有强调内容还有**strong 、 em**标签。

### seo为啥对vue单页面不友好?

- 爬虫在爬取的过程中，不会去执行js，所以隐藏在js中的跳转也不会获取到
- vue通过js控制路由然后渲染出对应的页面，而搜索引擎蜘蛛是不会去执行页面的js的，导致搜索引擎蜘蛛只能收录index.html一个页面，在百度中就搜索不到相关的子页面的内容。
- 我们加载页面的时候,浏览器的渲染包含:html的解析、dom树的构建、cssom构建、javascript解析、布局、绘制,当解析到javascript的时候才回去触发vue的渲染,然后元素挂载到id为app的div上,这个时候我们才能看到我们页面的内容,所以即使vue渲染机制很快我们仍然能够看到一段时间的白屏情况,用户体验不好

### 引起的问题

1. 收录的页面少了->被抓取的页面就少了->点击量之类的也就少了；

2. 不能对对应的页面做TDK(title, keywords, description)不同的配置，每个页面的title和meta标签都是一样的，不利于网络爬虫的爬取

   ```html
   <head>
     <title>xxxx公司</title>
     <meta name="keywords" content="关键字列表">
     <meta name="description" content="详情">
   </head>
   ```

## vue项目怎么做seo优化？

html就不能通过js生成，我们需要在加载js之前做一下页面的预渲染,目前了解到的有两种方法

常见的解决方案：

1. 页面预渲染，prerender-spa-plugin插件实现([配置参考](https://link.zhihu.com/?target=https%3A//www.npmjs.com/package/prerender-spa-plugin))

2. 服务端渲染，vue的ssr渲染([配置参考](https://link.zhihu.com/?target=https%3A//cn.vuejs.org/v2/guide/ssr.html))，SSR比较复杂。

3. 路由采用h5 history模式

## prerender-spa-plugin的使用

vue-cli3的解决方案

使用 webpack +prerender-spa-plugin + vue-meta-info 轻松地添加预渲染

[prerender-spa-plugingithub.com/chrisvfritz/prerender-spa-plugin](https://link.zhihu.com/?target=https%3A//github.com/chrisvfritz/prerender-spa-plugin)

[vue-meta-infowww.npmjs.com/package/vue-meta-info![img](https://pic3.zhimg.com/v2-338e4905a2684ca96e08c7780fc68412_180x120.jpg)](https://link.zhihu.com/?target=https%3A//www.npmjs.com/package/vue-meta-info)

```text
// 安装依赖
npm install prerender-spa-plugin --save
```

vue.config.js配置如下

```text
const PrerenderSPAPlugin = require('prerender-spa-plugin')
const Renderer = PrerenderSPAPlugin.PuppeteerRenderer
// eslint-disable-next-line no-unused-vars
const webpack = require('webpack')
const path = require('path')

module.exports = {
  configureWebpack: config => {
    if (process.env.NODE_ENV !== 'production') return
    return {
      plugins: [
        new PrerenderSPAPlugin({
          // 生成文件的路径，也可以与webpakc打包的一致。
          // 这个目录只能有一级，如果目录层次大于一级，在生成的时候不会有任何错误提示，在预渲染的时候只会卡着不动。
          staticDir: path.join(__dirname, 'dist'),
          // outputDir: path.join(__dirname, './'),
          // 对应自己的路由文件，比如a有参数，就需要写成 /a/param1。
          routes: ['/testData',  '/contact'],
          // 这个很重要，如果没有配置这段，也不会进行预编译
          renderer: new Renderer({
              inject: { //默认挂在window.__PRERENDER_INJECTED对象上，可以通过window.__PRERENDER_INJECTED.foo在预渲染页面取值
              foo: 'bar'
            },
            headless: false,
            // 在 main.js 中 document.dispatchEvent(new Event('render-event'))，两者的事件名称要对应上。
            renderAfterDocumentEvent: 'render-event'//等到事件触发去渲染，此处我理解为是Puppeteer获取页面的时机
          })
        })
      ]
    }
  },
}
```

- staticDir 指的是预渲染输出的页面地址，
- routes 指的是需要预渲染的路由地址，
- renderer 则是所采用的渲染引擎是什么，目前用的是 V3.4.0 版本支持 PuppeteerRenderer。
- inject 则是预渲染过程中都能拿到的值，该值提供给你了机会，让你觉得是否渲染这部分代码。例如下面的代码，是不会被预渲染进 HTML 中的。
- renderAfterDocumentEvent 这个则很关键，这个是监听 document.dispatchEvent 事件，决定什么时候开始预渲染

main.js中mounted触发

```text
new Vue({
  router,
  store,
  render: h => h(App),
//添加到这里,这里的render-event和vue.config.js里面的renderAfterDocumentEvent配置名称一致
  mounted () {
    document.dispatchEvent(new Event('render-event'))
  }
}).$mount('#app')
```

注意：

1、如果打包之后，刷新页面样式丢失，请配置对应的webpack的资源路径publicPath: '/',字段；

2、router.js里面把mode要改为：'history', 因为hash模式打包的时候会生成同样的页面；

## prerender-spa-plugin原理探究

它是如何做到将运行时的 html 打包到文件中的呢？

- prerender-spa-plugin 利用了 Puppeteer[4] 的爬取页面的功能。 Puppeteer 是一个 Chrome官方出品的 headlessChromenode 库。它提供了一系列的 API, 可以在无 UI 的情况下调用 Chrome 的功能, 适用于爬虫、自动化处理等各种场景。它很强大，所以很简单就能将运行时的 HTML 打包到文件中。
- 原理是在 Webpack 构建阶段的最后，在本地启动一个 Puppeteer 的服务，访问配置了预渲染的路由，然后将 Puppeteer 中渲染的页面输出到 HTML 文件中，并建立路由对应的目录。
- 每个路由对应的 HTML，然后我们可以更改每个路由文件里的 title 、 meta keyword等 。
- 另外页面的内容都已经在 HTML 中直接呈现，也可以解决 js 等资源加载慢导致白屏的问题。

## prerender-spa-plugin不能解决的问题

- 不同的用户看到不同的页面，动态数据页面（预渲染在获取用户权限数据之前就进行渲染了，所以这个不能）
- 动态路由也不可以（webpack编译的时候 路由还没挂载）
- 经常发生变化的页面，数据实时性展示（比如体育比赛等 我们现在的方式是前端拿到组件后进行组装数据，然后在进行渲染 像这种实时数据的会不准确）
- 路由过多，构建时间过长

## 静态页面分配title和meta标签----[vue-meta-info](https://link.zhihu.com/?target=https%3A//www.npmjs.com/package/vue-meta-info)

```text
// 安装依赖
npm install vue-meta-info --save

// main.js中引入注册
import MetaInfo from 'vue-meta-info'

Vue.use(MetaInfo)



// 需要seo的组件中使用
<template>
...
</template>

<script>
export default {
  metaInfo: {
    title: '我是contact头', // set a title
    meta: [{             // set meta
      name: 'keyWords',
      content: '我是contact关键字'
    },
    {
      name: 'description',
      content: '我是contact描述'
    }],
    link: [{ // set link
      rel: 'asstes',
      href: 'https://assets-cdn.github.com/'
    }]
  }
}
</script> 
```

注意：

如果需要改动的页面太多，给页面设置keywords和description的。也可以在router中配置，结合vuex去设置更加优雅一点。