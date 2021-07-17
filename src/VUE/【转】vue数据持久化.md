vuex是在中大型项目中必不可少的状态管理组件，刷新会重新更新状态，但是有时候我们并不希望如此。例如全局相关的，如登录状态、`token`、以及一些不常更新的状态等，我们更希望能够固化到本地，减少无用的接口访问，以及更佳的用户体验。

### 安装起步

```
npm i -S vuex-persistedstate
```

目前的环境版本：



```javascript
"vue": "2.2.4",
"vuex": "^2.5.0",
"vuex-persistedstate": "^2.5.4"
```

vuex已经升级到了`3.0.1`，看了release note没有发现太过分的升级，预计不会出现不兼容的情况，有感兴趣的小伙伴可以升级体验一下。

### 配置使用

在vuex初始化时候，作为组件引入。



```javascript
import persistedState from 'vuex-persistedstate'
export default new Vuex.Store({
    // ...
    plugins: [persistedState()]
})
```

### 自定义存储方式

`vuex-persistedstate`默认使用`localStorage`来固化数据，一些特殊情况要如何应对呢？（如：safari的无痕浏览模式）

> 需要使用`sessionStorage`的情况



```javascript
plugins: [
    persistedState({ storage: window.sessionStorage })
]
```

> 使用`cookie`的情况



```javascript
import persistedState from 'vuex-persistedstate'
import * as Cookies from 'js-cookie'

export default new Vuex.Store({
  // ...
  plugins: [
    persistedState({
      storage: {
        getItem: key => Cookies.get(key),
        setItem: (key, value) => Cookies.set(key, value, { expires: 7 }),
        removeItem: key => Cookies.remove(key)
      }
    })
  ]
})
```



作者：FE小帅
链接：https://www.jianshu.com/p/c22861ec5f21
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。