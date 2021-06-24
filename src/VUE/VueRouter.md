# VueRouter

## 1. 原理简述

- install
  - Vue.mixin:
    - 根组件属性注入，全局属性注入
    - 将vnode根实例（app）注入给vueRouter实例
    - Vue.util.defineReactive(this, '_route', vueRouter.history.current)
  - 注入全局组件：必须使用render方式，因为这部分代码没有预编译
    - RouterView:(functional)
      - data.routerView = true
      - depth = (循环遍历parent, 判断$parent.data是否为routerView，是depth++)
      - component = _route.matched[depth]
      - return render(component ? component : null）
    - RouterLink
- constructor:
  - history:
    - hashchange监听：
      - location.path 与pathList的路径进行匹配，若匹配到，则根据pathMap获取对应的routeRecord 记录，生成匹配结果记录（matched, path, parent, ...）,更新到current
      - 将匹配出的结果记录更新到app._route = history.current
  - matcher:
    - 处理routes数据，整理成routeRecord（path（转成绝对路径）, regex，parent）根据children递归添加pathMap, pathList

## 2. 代码实现

- MVueRouter 模块

```js
//=> mvue-router/index.js

import History from './lib/History'
import Matcher from './lib/Matcher'

class MVueRouter {
    constructor(options) {
        if (options === void 0) options = {};

        this.app = null;
        this.options = options;
        this.matcher = new Matcher(options.routes || [], this);
        this.history = new History(this, options.base, (route) => { 
            this.app && (this.app._route = route); 
        });
    }

    init(app) {
        this.app = app;
    }
}

var View = {
    name: 'RouterView',
    functional: true,
    props: {
        name: {
            type: String,
            default: 'default'
        }
    },
    render: function render(_, ref) {
        // console.log('ref', ref) // 本身无状态（不能设置响应式数据），不能被实例化。
        // console.log('ref.parent', ref.parent) // 父节点即包含本<route-view>组件的节点。
        var children = ref.children;
        var parent = ref.parent;
        var data = ref.data;

        data.routerView = true;

        var h = parent.$createElement;
        var route = parent.$route;

        var depth = 0;
        while (parent?._routerRoot !== parent) {
            var vnodeData = parent.$vnode?.data ?? {};
            if (vnodeData.routerView) {
                depth++;
            }

            parent = parent.$parent;
        }
        data.routerViewDepth = depth;

        // render previous view if the tree is inactive and kept-alive

        var matched = route.matched[depth];
        var component = matched?.component;
        // render empty node if no matched route or no config component
        if (!component) {
            // path: /user routes: /user、/user/posts 
            return h()
        }
        return h(component, data, children)
    }
};
MVueRouter.install = function (Vue) {
    Vue.component('router-link', {
        props: ['to'],
        render(h) {
            let self = this
            return h(
                'a',
                {
                    attrs: {
                        href: '/#' + self.to
                    },
                },
                self.$slots.default
            )
        }
    })
    Vue.mixin({
        beforeCreate() { // mixin $router
            if (this.$options.router) {
                this._routerRoot = this;
                this._router = this.$options.router;
                this._router.init(this);
                Vue.util.defineReactive(this, '_route', this._router.history.current);
            } else {
                this._routerRoot = (this.$parent?._routerRoot) || this
            }
        }
    })

    Object.defineProperty(Vue.prototype, '$router', {
        get: function get() { return this._routerRoot._router }
    });
    Object.defineProperty(Vue.prototype, '$route', {
        get: function get() { return this._routerRoot._route }
    });

    Vue.component('RouterView', View);
}

export default MVueRouter;

/* --------------------------------------------- */
// => mvue-router/lib/History.js
class History {
    constructor(router, base, cb) {
        this.router = router;
        this.base = base;
        this.matcher = this.router.matcher;
        this.current = this.createRoute(null, {
            path: '/'
        });
        this.cb = cb;
        window.addEventListener('hashchange', () => {
            // eslint-disable-next-line
            let path = this._getHash();
            let matchedRecord = this.matcher.match(path);
            let route = this.createRoute(matchedRecord, path)
            this.updateRoute(route)
            this.cb(route)
        })
    }

    updateRoute (route) {
        this.current = route
    }

    createRoute(
        record,
        path
    ) {
        var route = {
            path: path || '/',
            matched: record ? this._formatMatch(record) : []
        };
        return Object.freeze(route)
    }

    _formatMatch (record) {
        let matched = [];
        while(record) {
            matched.unshift(record);
            record = record.parent;
        }
        return matched;
    }

    _getHash() {
        // We can't use window.location.hash here because it's not
        // consistent across browsers - Firefox will pre-decode it!
        var href = window.location.href;
        var index = href.indexOf('#');
        if (index < 0) { return '' }
        href = href.slice(index + 1);
        return href
    }
}


export default History;

/* --------------------------------------------- */
//=> mvue-router/lib/Matcher.js

class Matcher {
    constructor(routes, router) {
        this.routes = routes;
        this.router = router;
        this.pathMap = {};
        this.pathList = new Set();
        this.init();
    }

    init() {
        for (let route of this.routes) {
            if (!route.path) {
                throw 'route without path.'
            }
            this.addRouteRecord(route)
        }
    }
    _normalizePath(
        path,
        parent,
        strict
    ) {
        if (!strict) { path = path.replace(/\/$/, ''); }
        if (path[0] === '/') { return path }
        if (parent == null) { return path }
        return this._cleanPath(((parent.path) + "/" + path))
    }

    _cleanPath(path) {
        return path.replace(/\/\//g, '/')
    }

    addRouteRecord(route, parent) {
        let normalizedPath = this._normalizePath(route.path, parent);

        var record = {
            path: normalizedPath,
            regex: normalizedPath,
            component: route.component,
            parent: parent,
        };
        this.pathList.add(normalizedPath);
        this.pathMap[normalizedPath] = record;

        if (route.children) {
            for(let childRoute of route.children) {
                this.addRouteRecord(childRoute, route)
            }
        }
    }
    match (path) {
        path = path.replace(/\/$/, '')
        return this.pathMap[path] ?? {}
    }
}

export default Matcher;
```

- Main.js/app.vue

```js
// main.js

import Vue from 'vue'
import App from './App.vue'
import router from './mrouter'
import store from './mstore'

Vue.config.productionTip = false

new Vue({
    router,
    store,
    render: h => h(App)
}).$mount('#app')
```

```vue
<!-- app.vue -->
<template>
    <div id="app">
    <div id="nav">
        <router-link to="/">Home</router-link> |
        <router-link to="/about">About</router-link>
    </div>
    <router-view/>
    </div>
</template>
```
