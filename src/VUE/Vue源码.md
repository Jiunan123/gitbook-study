# Vue



## 模块解析

1. 数据响应式模块（Observe）

2. 数据代理（proxy）

3. 编译（Compile）

   遍历挂载的元素childNodes：

   ​	元素节点: 判断是否有指令

   ​	文本节点：判断是否有插入语法{{xx.xx}}

   ** 若使用到对应的响应式数据，则new Watcher, 并主动触发一次属性的get, 收集依赖 **

4. 事件订阅发布模式（Watcher）

   Depend：在get中进行收集

   Notify：在set中进行发布

5. 数据关系

   一个响应式属性包含一个dep

   dep中包含多个Watcher

6. 关系图

   ![vue响应式渲染关系图](../assets/img/img01-vue.png)

### 注意

Vue2的compile模块是将模板代码转换成render函数。

runtime版本仅包含运行时，不包含编译器。所以不能在runtime版本中使用template模版代码。

其中，插件VueRouter就是runtime版本。

## Vue2初始化

【问】：new Vue() 做了哪些事情

- 执行了_init(): 

  - 组件实例初始化属性initLifecycle（生命周期相关的属性 $parent, $chidren, ...）

  - 处理自定义事件initEvents（<comp @myclick='xxx' />）

  - 处理与render相关的属性initRender（this.$slots, this.$scopedSlots, this._c, this.$createElement )

  - 调用生命周期钩子beforeCreate

  - **组件状态初始化，响应式初始化（inject > props, data, computed, watch > provide)**

    - Inject 父辈属性注入。可在data中使用

    - Provide 给子辈提供属性，可以指向响应式的数据

      ``````js
      var Provider = {
        provide: {
          vmData: this.data, 
          /* provide 和 inject 绑定并不是可响应的。
          如果你传入了一个可监听的对象，
          那么其对象的 property 还是可响应的 */
        },
      // ...
      ``````

  - 调用生命周期钩子created

  - 如果选项存在el，则自动调用$mount：

    - 取用优先级（el < template < render）

## Vue mount

源码路径：core/instance/lifecycle.js

模块： mountComponent

```js
export function mountComponent (
  vm: Component,
  el: ?Element,
  hydrating?: boolean
): Component {
  vm.$el = el
  // ...省略
  
  // 1. 调用beforeMount生命周期钩子
  callHook(vm, 'beforeMount')

  // ...省略判断逻辑
  let updateComponent = () => {
    /* vm.render() 得到虚拟Dom, _update将虚拟dom变成真实Dom/
    vm._update(vm._render(), hydrating)
  }
  
  /* 2. new Watcher(vue2中，一个组件只有1个watcher)
   * 这样的watcher称为render watcher. 组件级别的watcher
   */ 
  new Watcher(vm, updateComponent, noop, {
    before () {
      if (vm._isMounted && !vm._isDestroyed) {
        callHook(vm, 'beforeUpdate')
      }
    }
  }, true /* isRenderWatcher */)
  hydrating = false

  if (vm.$vnode == null) {
    vm._isMounted = true
    callHook(vm, 'mounted')
  }
  return vm
}
```

源码路径：core/instance/lifecycle.js

模块：Vue.prototype._update

```js
Vue.prototype._update = function (vnode: VNode, hydrating?: boolean) {
    const vm: Component = this
    const prevEl = vm.$el
    /* update前的虚拟dom */
    const prevVnode = vm._vnode
    const restoreActiveInstance = setActiveInstance(vm)
    /* update后的虚拟dom */
    vm._vnode = vnode
  	/* 判断是初始化还是更新 */
    if (!prevVnode) {
      // initial render
      vm.$el = vm.__patch__(vm.$el, vnode, hydrating, false /* removeOnly */)
    } else {
      // updates
      vm.$el = vm.__patch__(prevVnode, vnode)
    }
  
    restoreActiveInstance()
    // update __vue__ reference
    if (prevEl) {
      prevEl.__vue__ = null
    }
    if (vm.$el) {
      vm.$el.__vue__ = vm
    }
    // if parent is an HOC, update its $el as well
    if (vm.$vnode && vm.$parent && vm.$vnode === vm.$parent._vnode) {
      vm.$parent.$el = vm.$el
    }
    // updated hook is called by the scheduler to ensure that children are
    // updated in a parent's updated hook.
  }
```

### 注意

性能优化：

- 组件拆分：由于1个组件对应一个watcher，所以频繁更新的模块应该拆分成独立的子组件。
- watcher 和 dep 是多对多的关系：
  - 1个组件对应一个watcher，则多个数据对应一个watcher
  - props、watch、computed 响应式数据，可关联多个组件数据，render watcher, 即一个数据对应多个watcher

### 流程总结

 1. 有el配置，或者$.mount(el)
  2. 执行mountComponent：
       1. 创建一个Render Watcher
            1. 绑定更新函数updateComponent
                 1. 执行render函数生成虚拟Dom。
                     2. 虚拟Dom转成真实Dom。
                        2. 立即执行update：
                             1. 触发响应式数据的get请求，收集依赖。
  3. 当对应响应式数据触发set请求时，则会通知该render watcher执行update

## 响应式数据

源码路径：core/instance/state.js

模块：initState

```js
function initState (vm: Component) {
  vm._watchers = []
  const opts = vm.$options
  if (opts.props) initProps(vm, opts.props)
  if (opts.methods) initMethods(vm, opts.methods)
  if (opts.data) {
    initData(vm)
  } else {
    observe(vm._data = {}, true /* asRootData */)
  }
  if (opts.computed) initComputed(vm, opts.computed)
  if (opts.watch && opts.watch !== nativeWatch) {
    initWatch(vm, opts.watch)
  }
}
```

模块源码：core/instance/state.js

模块：initData

```js
function initData (vm: Component) {
  let data = vm.$options.data
	// ...省略代码
  
  // proxy data on instance
  const keys = Object.keys(data)
  const props = vm.$options.props
  const methods = vm.$options.methods
  let i = keys.length
  while (i--) {
    const key = keys[i]
   	// ...省略属性名重复警告逻辑
    
    // 把data属性代理到实例对象上。vm.xx
    proxy(vm, `_data`, key)
  }
  // 递归遍历响应处理
  observe(data, true /* asRootData */)
}
```

模块源码：core/instance/observer/index.js

模块：observe

``````js
export function observe (value: any, asRootData: ?boolean): Observer | void {
  if (!isObject(value) || value instanceof VNode) {
    return
  }
  let ob: Observer | void
  /* 每个响应式对象都有一个__ob__属性
   * 用于响应式对象的动态属性的新增与删除 和 响应式数据的元素的增加与删除的变更通知
   * Vue.set(obj, 'key', val);
   */
  if (hasOwn(value, '__ob__') && value.__ob__ instanceof Observer) {
    ob = value.__ob__
  } else if (
    shouldObserve &&
    !isServerRendering() &&
    (Array.isArray(value) || isPlainObject(value)) &&
    Object.isExtensible(value) &&
    !value._isVue
  ) {
    ob = new Observer(value)
  }
  if (asRootData && ob) {
    ob.vmCount++
  }
  return ob
}
``````

模块源码：core/instance/observer/index.js

模块：Observer

```js
export class Observer {
  value: any;
  dep: Dep;
  vmCount: number; // number of vms that have this object as root $data

  constructor (value: any) {
    this.value = value
    /* 这里有个一个dep。
     * 收集该属性本身的依赖，当该属性对应的对象的子属性新增与删除，
     * 则能够通过该实例变量，通知到对应的依赖。
     */
    this.dep = new Dep()
    this.vmCount = 0
    /* 将该实例存储到数据的__ob__中 */
    def(value, '__ob__', this)
    if (Array.isArray(value)) {
      if (hasProto) {
        protoAugment(value, arrayMethods)
      } else {
        copyAugment(value, arrayMethods, arrayKeys)
      }
      this.observeArray(value)
    } else {
      this.walk(value)
    }
  }

  /* 遍历对象的属性，并改造它的getter，setter */
  walk (obj: Object) {
    const keys = Object.keys(obj)
    for (let i = 0; i < keys.length; i++) {
      defineReactive(obj, keys[i])
    }
  }

  /**
   * Observe a list of Array items.
   */
  observeArray (items: Array<any>) {
    for (let i = 0, l = items.length; i < l; i++) {
      observe(items[i])
    }
  }
}
```

模块源码：core/instance/observer/index.js

模块：defineReactive

```js
export function defineReactive (
  obj: Object,
  key: string,
  val: any,
  customSetter?: ?Function,
  shallow?: boolean
) {
  /* dep 与 对下对象的属性关系 1:1*/
  const dep = new Dep()

  const property = Object.getOwnPropertyDescriptor(obj, key)
  if (property && property.configurable === false) {
    return
  }
	// ...省略部分代码
  
  /* observe(val) 递归遍历该属性对应的值，并获得子对象的__ob__ 
   * 问：为什么要在为子对象收集触发getter调用的watcher
   */
  let childOb = !shallow && observe(val)
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter () {
      const value = getter ? getter.call(obj) : val
      if (Dep.target) {
        dep.depend()
        /* 子对象的__ob__存在，则将此次的watcher收集 */
        if (childOb) {
          childOb.dep.depend()
          if (Array.isArray(value)) {
            dependArray(value)
          }
        }
      }
      return value
    },
    set: function reactiveSetter (newVal) {
      const value = getter ? getter.call(obj) : val
      /* eslint-disable no-self-compare */
      if (newVal === value || (newVal !== newVal && value !== value)) {
        return
      }
      /* eslint-enable no-self-compare */
      if (process.env.NODE_ENV !== 'production' && customSetter) {
        customSetter()
      }
      // #7981: for accessor properties without setter
      if (getter && !setter) return
      if (setter) {
        setter.call(obj, newVal)
      } else {
        val = newVal
      }
      childOb = !shallow && observe(newVal)
      dep.notify()
    }
  })
}
```



### 注意

- 数据初始化的优先级：
  - Inject > props > methods > data > computed > watch
