# Vue nextTick与更新机制源码分析

## 批量更新模块

代码阅读入口：instance/observer/index.js

模块：dep.notify

dep.notify -> subs[i].update-> ...收集进队列，且在nextTick进行更新。... -> run() -> get()

```js
export default class Watcher {
  update () {
    /* computed设置的lazy */
    if (this.lazy) {
      this.dirty = true
    } else if (this.sync) {
      this.run()
    } else {
      /*业务入口*/
      queueWatcher(this)
    }
  }
  
  /* 真正触发更新 */
  run () {
    if (this.active) {
      /* 执行更新 */
      const value = this.get()
      if (
        value !== this.value ||
        // Deep watchers and watchers on Object/Arrays should fire even
        // when the value is the same, because the value may
        // have mutated.
        isObject(value) ||
        this.deep
      ) {
        // set new value
        const oldValue = this.value
        this.value = value
        /* 省略部分代码 */
        /* cb 是$watch是传入的handle, 'xx': function (val, oldVal) {...} 
         * 执行完更新后，执行的回调
         */
        this.cb.call(this.vm, value, oldValue)
      }
    }
  }
  
  /* 创建watcher的时候，会立即调用一次。（更新暨初始化）
   * update真正触发的时候，也会调用。
   */
  get () {
    pushTarget(this)
    let value
    const vm = this.vm
    try {
      /* 最终的更新函数
       * renderWatcher: updateComponent -> 执行render...
       * computed: get ()
       */
      value = this.getter.call(vm, vm)
    } catch (e) {
      if (this.user) {
        handleError(e, vm, `getter for watcher "${this.expression}"`)
      } else {
        throw e
      }
    } finally {
      // "touch" every property so they are all tracked as
      // dependencies for deep watching
      if (this.deep) {
        traverse(value)
      }
      popTarget()
      this.cleanupDeps()
    }
    return value
  }
}
```

路径：instance/observer/scheduler.js

模块：queueWatcher

```js
/* 是否开始收集脏袜子 */
let waiting = false
/* 是否开始洗袜子 */
let flushing = false
/* 收集脏袜子的盆 */
const queue: Array<Watcher> = []
/* 记录洗到第几个脏袜子 */
let index = 0
/* 脏袜子从小到大洗（排序watcher.id）*/

// 新一轮update，第一次触发响应式数据的set
export function queueWatcher (watcher: Watcher) {
  const id = watcher.id
  // 去重
  if (has[id] == null) {
    has[id] = true
    if (!flushing) {
      queue.push(watcher)
    } else {
      /* 如果已经开始洗袜子，则把脏袜子插入到待洗袜子中（sort）*/
      /* 若比该袜子小的袜子已经全部洗干净（i > index），则把该袜子放在队首 */
      let i = queue.length - 1
      while (i > index && queue[i].id > watcher.id) {
        i--
      }
      queue.splice(i + 1, 0, watcher)
    }
    if (!waiting) {
    	/* 设置开始收集脏袜子 */
      waiting = true
      /* ...省略部分代码 */
      
      /* 将洗袜子的任务加入微任务队列。*/
      nextTick(flushSchedulerQueue)
    }
  }
}
```

路径：instance/observer/scheduler.js

模块：flushSchedulerQueue

```js
/* 异步执行阶段 */
function flushSchedulerQueue () {
  currentFlushTimestamp = getNow()
  flushing = true
  let watcher, id
  
  /* 排序目的：
   * 1. 保证组件更新顺序：父->子；（父组件created > 子组件created，所以watcher.id会比较小）
   * 2. 组件本身watcher更新顺序：userWatcher > renderWatcher
   *    (userWatcher在initState时创建，而renderWatcher在mounted时创建。)
   * 3. 当组件被销毁时，正在更新父组件的watcher，可以把该组件的watcher移除。
   */
  queue.sort((a, b) => a.id - b.id)

  /* 遍历清洗袜子。
   * 由于该模块的代码时异步执行的，所以在洗袜子期间，可能会增加新的袜子。
   * queue.length可能会变大，所以不能缓存queue.length
   */
  for (index = 0; index < queue.length; index++) {
    watcher = queue[index]
    if (watcher.before) {
      watcher.before()
    }
    id = watcher.id
    /* 被洗过的袜子要清除标记 */
    has[id] = null
    watcher.run()
    /* ...省略部分代码 */
  }

  // keep copies of post queues before resetting state
  const activatedQueue = activatedChildren.slice()
  const updatedQueue = queue.slice()

  resetSchedulerState()

  // call component updated and activated hooks
  callActivatedHooks(activatedQueue)
  callUpdatedHooks(updatedQueue)

  // devtool hook
  /* istanbul ignore if */
  if (devtools && config.devtools) {
    devtools.emit('flush')
  }
}

function resetSchedulerState () {
  index = queue.length = activatedChildren.length = 0
  has = {}
  if (process.env.NODE_ENV !== 'production') {
    circular = {}
  }
  waiting = flushing = false
}
```

### 总结：

完整一轮更新：（初始：waiting=false, queue=[]，flashing=false）

1. 当响应式属性更新（set）时，触发dep.notify，将该属性dep关联的watchers都添加到queue(去重)队列
2. 标记waiting=true，nextTick(冲刷任务)（ps: 冲刷任务：遍历执行queue任务）
   1. 即冲刷任务进入微任务队列
3. 重复步骤1，直到执行到冲刷任务被执行(开始执行时，设置flashing=true)
   1. 执行时，需要先给queue排序，保证执行顺序：父>子，userWatcher > renderWatcher
   2. 结束执行前重置：waiting=false, queue.length=0, flashing=false
   3. flushing开始时，仍然会继续收集watcher

## nextTick模块

路径：util/next-tick.js

模块：nextTick

```js
/* nextTick的调用队列(微.微任务队列)
 * 回调任务收集，直到前面的微任务执行完毕，开始执行当前的nextTick的微任务。
 * callbacks: [..., flushSchedulerQueue(即nextTickCb), nextTickCb,...]
 * queue(flushSchedulerQueue): [watcher,...]
 */
const callbacks = []
/* 模拟promise的决策状态，pending = true, 表示创建了Promise，但是待决*/
let pending = false

export function nextTick (cb?: Function, ctx?: Object) {
  let _resolve
  /* 把cb包装，进行异常情况处理 */
  callbacks.push(() => {
    if (cb) {
      try {
        cb.call(ctx)
      } catch (e) {
        handleError(e, ctx, 'nextTick')
      }
    } else if (_resolve) {
      _resolve(ctx)
    }
  })
  if (!pending) {
    pending = true
    timerFunc()
  }
  /* 感觉这段代码整合到上面的 else if 内，运行也没有差别。*/
  if (!cb && typeof Promise !== 'undefined') {
    return new Promise(resolve => {
      _resolve = resolve
    })
  }
}

```

路径：util/next-tick.js

模块：timerFunc

```js
function flushCallbacks () {
  pending = false
  /* 数组备份，而不是引用。*/
  const copies = callbacks.slice(0)
  /* 清空原数组：使用length=0，则该数组的引用也被清空 */
  callbacks.length = 0
  /* 遍历执行所有回调函数 */
  for (let i = 0; i < copies.length; i++) {
    copies[i]()
  }
}

let timerFunc

if (typeof Promise !== 'undefined' && isNative(Promise)) {
  /* 1.使用原生Promise */
  const p = Promise.resolve()
  timerFunc = () => {
    /* p.then(() => {Promise.resolve(flushCallbacks)})*/
    p.then(flushCallbacks) 
    if (isIOS) setTimeout(noop)
  }
  isUsingMicroTask = true
} else if (!isIE && typeof MutationObserver !== 'undefined' && (
  isNative(MutationObserver) ||
  MutationObserver.toString() === '[object MutationObserverConstructor]'
)) {
  /* MutationObserver接口提供了监视对DOM树所做更改的能力：
   * 监听某个Dom节点（包含属性变更），一旦变更便触发回调。
   * 使用原生MutationObserver
   */
  let counter = 1
  /* 创建原生mutationObserver */
  const observer = new MutationObserver(flushCallbacks)
  /* 创建文本节点：*/
  const textNode = document.createTextNode(String(counter))
  /* 监听文本节点的变更 */
  observer.observe(textNode, {
    characterData: true
  })
  /* 通过变更文本节点的属性data，触发回调函数的执行 */
  timerFunc = () => {
    counter = (counter + 1) % 2
    textNode.data = String(counter)
  }
  isUsingMicroTask = true
} else if (typeof setImmediate !== 'undefined' && isNative(setImmediate)) {
  /* 使用setImmediate
   * 在浏览器完成后面的其他语句后，就立刻执行这个回调函数
   */
  timerFunc = () => {
    setImmediate(flushCallbacks)
  }
} else {
  /* 定时器宏任务 */
  timerFunc = () => {
    setTimeout(flushCallbacks, 0)
  }
}
```

### 总结：

- nextTick: Promise. > MutationObserver > setImmadiate > setTimeout(cb,0)

- 一轮nextTick（初始：pending=false, callbacks=[]）
  - nextTick被调用
    - pending = true
    - callbacks.push(callback)
    - 冲刷任务进入微任务队列
  - nextTick再次被调用，callbacks.push(callback)
  - 直到冲刷任务被执行：遍历callbacks，执行回调，重置：pending=false, callbacks.length=0

## 思考

```js
<!DOCTYPE html>
<html>

<head>
  <title>Vue源码剖析</title>
  <script src="../../dist/vue.js"></script>
</head>

<body>
  <div id="demo">
    <h1>异步更新</h1>
    <p id="p1">{{foo}}</p>
  </div>
  <script>
    /* 案例1: $nextTick在最前面
     * 影响了nextTick的callbacks=[userNextTickCb, defaultNextTickCb]
     * defaultNextTickCb是指变更属性关联的watcher的update
     * watcher包含renderWatcher。
     */
    function mounted1 () {
      // 1. $nextTick触发一个promise,并处于pending,callbacks=[userNextTickCb]
      this.$nextTick(() => {
        console.log('p1.innerHTML:' + p1.innerHTML)
        // ready~~
      })
      
      // 2. 属性set,触发nextTick，callbacks=[userNextTickCb, defaultNextTickCb]
      this.foo = Math.random()
      console.log('1:' + this.foo);
      this.foo = Math.random()
      console.log('2:' + this.foo);
      
      // 3. renderWatcher的更新是在微任务中,当前代码运行完之后，所以此时视图还没有更新
      console.log('p1.innerHTML:' + p1.innerHTML)
      // ready~~
    }

    /* 案例2: $nextTick在更新数据的中间
     * 影响了nextTick的callbacks=[defaultNextTickCb, userNextTickCb]
     */
    function mounted2 () {
      // 1. 属性set,触发nextTick，callbacks=[defaultNextTickCb]
      this.foo = Math.random()
      console.log('1:' + this.foo);

      // 2. $nextTick，callbacks=[defaultNextTickCb, userNextTickCb]
      this.$nextTick(() => {
        console.log('p1.innerHTML:' + p1.innerHTML)
        // 视图已更新，第二次random的值。
      })

      // this.foo属性立刻更新
      this.foo = Math.random()
      console.log('2:' + this.foo);
    }

    /* 案例3: 同案例2，测试相同属性多次变更
     * 影响了nextTick的callbacks=[defaultNextTickCb, userNextTickCb]
     */
    function mounted3 () {
      // 1. 属性set,触发nextTick，callbacks=[defaultNextTickCb]
      this.foo = Math.random()
      console.log('1:' + this.foo);

      // this.foo属性立刻更新
      this.foo = Math.random()
      console.log('2:' + this.foo);
    
      // 2. $nextTick，callbacks=[defaultNextTickCb, userNextTickCb]
      this.$nextTick(() => {
        console.log('p1.innerHTML:' + p1.innerHTML)
        // 视图已更新，第二次random的值。
      })
    }

	  /* 案例4: Promise vs $nextTick
     * 影响了微任务列表 mcros=[Promise.resolve(callbacks), Promise.resolve()]
     */
    function mounted4 () {
      // 1. 属性set,触发nextTick，callbacks=[属性更新cb（包含renderWatcher）]
      // 微任务队列 [nextTickCb]
      this.foo = Math.random()
      console.log('1:' + this.foo);

      // this.foo属性立刻更新
      this.foo = Math.random()
      console.log('2:' + this.foo);
    
      //2. 微任务队列[nextTickCb, promise.resolve()]
      Promise.resolve().then(() => {
          console.log('promise p1.innerHTML:' + p1.innerHTML)
      })

      // 3. nextTickCb的callbacks=[属性更新cb（包含renderWatcher）,cb1]
      this.$nextTick(() => {
        console.log('nextTick p1.innerHTML:' + p1.innerHTML)
      })
      
      // => nextTick先打印，然后才是promise
    }
    // 创建实例
    const app = new Vue({
      el: '#demo',
      data: {
        foo: 'ready~~'
      },
      mounted() {
        mounted3.call(this)
      }
    });

  </script>
</body>

</html>
```

