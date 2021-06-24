# Vue模拟

## 模块
- mvue.html
- mvue.js
- compile.js
- observer.js
- watcher.js
- dep.js
- util.js

## 代码

### mvue.html

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=0,minimal-ui:ios">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <meta http-equiv="Content-Type" content="text/html;charset=UTF-8">
    <title>Document</title>
    <link rel="stylesheet" href="">
    <script src=""></script>
</head>
<body>
    <div id="app">
        <p @click="add">{{counter}}</p>
        <p>{{message.warning}}</p>
        <p m-text="counter"></p>
        <p m-html="desc"></p>
        <input type="text" m-model="desc">
      </div>
</body>
<script type="module">
  import MVue from './mvue.js'
  var app = new MVue({
    el: '#app',
    data: {
      counter: 1,
      desc: '<span style="color:red">red</span>',
      message: {
        warning: 'warning'
      },
      arr: [1, 2, {
        x: 1
      }]
    },
    methods: {
      add() {
        this.counter++
        this.arr.push('4')
        console.log(this.arr)
      }
    },
  })

  // setInterval(() => {
  //   app.counter++
  // }, 1000);
  app.message.warning = 'hhhh'
</script>
</html>
```

### mvue.js

```js
import { observe } from './observer.js';
import { Complie } from './compile.js';

export default class MVue {
    constructor(option) {
        // 1. 赋值
        this.$option = option;
        this.$data = option.data;
        this.$el = document.querySelector(option.el);

        // 2. 响应式数据
        observe(this.$data);

        // 3. 代理数据
        Reflect.ownKeys(this.$data).forEach((key) => {
            Reflect.defineProperty(this, key, {
                get() {
                    return this.$data[key];
                },
                set(newVal) {
                    this.$data[key] = newVal;
                }
            })
        })

        // 4. complie
        new Complie(this, this.$el);
    }
}
```

### observer.js

```js
import { Dep } from './dep.js'

const originArrayPrototype = Array.prototype;
// 保护Array.prototype不变，生成新的原型链
const arrayPrototype = Object.create(originArrayPrototype);
const methodsToPatch = [
    'push',
    'pop',
    'shift',
    'unshift',
    'splice',
    'sort',
    'reverse'
  ]
  
  methodsToPatch.forEach(method => {
    return arrayPrototype[method] = function (...args) {
        const res = originArrayPrototype[method].apply(this, args);
        let inserted; // 数组
        let ob = this.__ob__;
        switch(method) {
            case 'push':
            case 'unshift':
                inserted = args;
                break;
            case 'splice':
                inserted = args.slice(2);
        }
        /** 对新增的数组成员进行监听 */
        ob.observeArray(inserted);
        ob.dep.notify();

        return res;
    }
})

export function observe(value) {
    if (typeof value !== 'object' || value == null) {
        return;
    }
    return value.__ob__ ? value.__ob__ : new Observer(value)
}

export class Observer {
    constructor(value) {
        /** 这个细节很关键 */
        Object.defineProperty(value, '__ob__', {
            value: this,
            enumerable: false,
            writable: true,
            configurable: true
        });
        this.dep = new Dep();

        if (Array.isArray(value)) {
            Object.setPrototypeOf(value, arrayPrototype);
            this.observeArray(value);
        } else {
            this.walk(value);
        }
    }

    walk (value) {
        const keys = Object.keys(value);
        for(let i = 0; i < keys.length; i++) {
            observe(value[keys[i]])
        }
    }

    observeArray (value) {
        for(let v of value) {
            observe(v);
        }
    }
}

function defineReactive(obj, key, val) {
    let dep = new Dep();
    
    let childOb = observe(val);
    Object.defineProperty(obj, key, {
        get() {
            if (Dep.target) {
                dep.depend();
                childOb && childOb.dep.depend();
            }
            return val;
        },
        set(newVal) {
            val = newVal;
            dep.notify();
        }
    })
}
```

### watcher.js

```js
import { Dep } from './dep.js'
import Util from './util.js'

export class Watcher {
    constructor(app, exp, cb) {
        this.cb = cb;
        this.app = app;
        this.exp = exp;

        Dep.target = this;
        this.getVal();
        Dep.target = null;
    }

    update() {
        this.cb.call(this.app, this.getVal())
    }

    getVal() {
        return Util.getValueFromExpress(this.app, this.exp)
    }
}
```

### dep.js

```js
export class Dep {
    constructor() {
        this.watchers = []
    }

    depend() {
        this.addWatcher(Dep.target)
    }

    addWatcher(target) {
        this.watchers.push(target)
    }

    notify() {
        this.watchers.forEach((watcher) => {
            watcher.update();
        })
    }
}
```

### util.js

```js
function getValueFromExpress(obj, exp) {
    return exp.split('.').reduce((res, key) => res[key], obj);
}

function getTargetFromExpress(obj, exp) {
    const keys = exp.split('.');
    const lastKey = keys.pop();
    const target = keys.reduce((res, key) => res[key], obj);
    return {
        target,
        key: lastKey
    }
}

export default {
    getValueFromExpress,
    getTargetFromExpress
}
```

### compile.js

```js
import Util from './util.js';
import { Watcher } from './watcher.js';

const NODE_TYPE = {
    element: 1,
    attribute: 2,
    text: 3,
}

const COMPILE_TEXT_REG = /{{(\s*([^\{\}\s])+\s*)}}/g;

export class Complie {
    constructor(app, root) {
        this.app = app;
        this.root = root;

        this.complie(this.root);
    }

    complie(node) {
        if (node == null) { return; }
        switch (node.nodeType) {
            case NODE_TYPE.element:
                this.compileAttr(node);
                [...node.childNodes].forEach((chileNode) => {
                    this.complie(chileNode);
                })
                break;
            case NODE_TYPE.text:
                // 查找{{}},转变成数据
                this.compileText(node);
                break;
            default:
                break;
        }
    }

    // 目前只考虑单个 {{ obj.xx }}的情况
    compileText(node) {
        let textContent = node.textContent;
        if (COMPILE_TEXT_REG.test(textContent)) {
            this.update(node, RegExp.$1, 'text', { textContent });
        }
    }

    compileAttr(node) {
        [...node.attributes].forEach((attr) => {
            let name = attr.name;
            if (name.substring(0, 2) == 'm-') {
                const dir = name.substring(2) + 'Directive';
                if (this[dir]) {
                    this[dir](node, attr.value, dir)
                } else {
                    this.update(node, attr.value, dir)
                }

                // 指令处理3个步骤
                // 1. 指令影响响应式数据，以及其他
                // 2. new watcher(..., 响应式影响视图的回调事件)
            }
            if (name[0] == '@') {
                // @click = 'xx'
                const dir = name.substring(1);
                this.eventHandle(node, dir, attr.value)
            }
        })
    }

    update(node, exp, dir, payload) {
        // 1. init
        let fn = this[dir + 'Updater'];
        fn?.(node, Util.getValueFromExpress(this.app, exp), payload)
        // 2. new Watcher
        new Watcher(this.app, exp, (val) => {
            fn?.(node, val, payload)
        });
    }

    eventHandle(node, dir, val) {
        const fn = this.app.$option.methods?.[val];
        if (!fn) { throw `${val} method is undefined` }
        node.addEventListener(dir, fn.bind(this.app))
    }

    textUpdater(node, val, { textContent }) {
        node.textContent = textContent.replace(COMPILE_TEXT_REG, val)
    }

    textDirectiveUpdater(node, val) {
        // val的encode模块略过。
        node.innerHTML = val;
    }
    htmlDirectiveUpdater(node, val) {
        node.innerHTML = val;
    }
    // 视图节点驱动响应式数据更新。
    modelDirective(node, exp, dir) {
        // 1. 监听input事件
        // 2. 更新响应式数据
        node.addEventListener('input', (e) => {
            const { target, key } = Util.getTargetFromExpress(this.app, exp)
            if (target && target[key]) {
                target[key] = e.target.value;
            }
        })
        this.update(node, exp, dir)
    }
    // 响应式驱动视图变化的回调
    modelDirectiveUpdater(node, val) {
        node.value = val;
    }
}
```



