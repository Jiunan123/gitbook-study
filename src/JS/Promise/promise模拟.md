# promise模拟

## 模拟代码

要点：

- 使用浏览器的原生API：MutationObserver，实现微任务模式（监听Dom节点的属性，变更则执行回调）
- resolve、reject 作用是变更promise的状态（一个错误概念：resolve, reject产生微任务）
- then 作用
  - 绑定回调函数：onFulfilled、onRejected
  - 立即返回一个新的promise，且该promise的result是回调函数的结果
- 微任务被决策（!pending），则按需将绑定的回调函数加入微任务队列。
- 构造函数提供外部的两个函数resolve、rejected必须bind(this)

```js
// MPromise.mjs

const MPromistState = {
    pending: 'pending',
    fulfilled: 'fulfilled',
    rejected: 'rejected'
}

let counter = 0;

export default class MPromise {
    constructor (cb) {
        // 1. 初始化
        this["[[PromiseState]]"] = MPromistState.pending;
        this["[[PromiseResult]]"] = undefined;
        this.resolvedCallbacks = [];
        this.rejectedCallbacks = [];
        this.callbacks = [];
        this.subResolves = [];

        // 2. 准备微任务配置
        this.timeFunc = this._createTimeFunc();

        // 3. 立即执行用户传入的回调函数
        cb(this.resolve.bind(this), this.reject.bind(this))
    }

    // promise决策。并将then绑定的cb加入微任务队列
    // cb执行的结果作为新的promise的结果。
    resolve (val) {
        // 1. 设置状态与值
        this["[[PromiseState]]"] = MPromistState.fulfilled;
        this["[[PromiseResult]]"] = val;

        // 2. 将对应状态的任务们加入微任务列表
        this.callbacks = this.resolvedCallbacks;
        this.timeFunc()
    }

    reject (val) {
        this["[[PromiseState]]"] = MPromistState.rejected;
        this["[[PromiseResult]]"] = val;
        this.callbacks = this.rejectedCallbacks;
        this.timeFunc()
    }

    // 异步执行
    flushCallbacks () {
        let copies = this.callbacks.slice(0);
        let resolves = this.subResolves.slice(0);
        this.callbacks.length = 0;
        this.subResolves.length = 0;
        this.resolvedCallbacks.length = 0;
        this.rejectedCallbacks.length = 0;

        for(let i = 0; i < copies.length; i++) {
            // 将结果返回作为下一个promise的result
            let result = copies[i]?.(this["[[PromiseResult]]"])
            resolves[i](result);
        }
    }

    _createTimeFunc () {
        let observer = new MutationObserver(this.flushCallbacks.bind(this));
        let textNode = document.createTextNode(String(counter));
        observer.observe(textNode, {
            characterData: true
        })
        return () => {
            counter = (counter + 1) % 2;
            textNode.data = String(counter);
        }
    }
	
    // 允许两个回调函数都为空，则result=undefined, state=fulfilled
    then (onFulfilled, onRejected) {
        return new MPromise((res, rej) => {
            this.resolvedCallbacks.push(onFulfilled);
            this.rejectedCallbacks.push(onRejected);
            this.subResolves.push(res);

            if (this["[[PromiseState]]"] !== MPromistState.pending) {
                this.timeFunc()
            }
        })
    }
}
```

```html
<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=0,minimal-ui:ios">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <link rel="stylesheet" href="">
    <script src=""></script>
</head>

<body>

</body>
<script type="module">
  	// 测试模块
    import MPromise from './MPromise.mjs'

    let p = new MPromise((res) => {
        res(1)
    })
    p.then((val) => {
        console.log('1', val)
        return 3
    }).then((val) => {
        console.log('3', val)
    })
    p.then((val) => {
        console.log('2', val)
    })

    let p2 = new MPromise((res) => {
        setTimeout(() => {
            res(2)
        }, 0)
    })
    p2.then((val) => {
        console.log('4', val)
    })
</script>
</html>
```

## 测试

then与微任务关系的测试

```js
let p1 = new Promise((res, rej) => {
    res(1)
})

// 同步：p1已被决策

p1.then(() => {
    console.log('p1.then')
}).then(() => {
    console.log('p1.then.then')
})

// mcros=[log(p1.then)]
// p1_1 在log('p1.then')执行之后才会被决策

let p2 = new Promise((res, rej) => {
    res(2)
})
// 同步：p2被决策

p2.then(() => {
    console.log('p2.then')
}).then(() => {
    console.log('p2.then.then')
})
// mcros=[log('p1.then')，log('p2.then')]
// p2_1 在log('p2.then')执行之后才会被决策

// 解析：微任务队列
// 1 [log('p1.then')，log('p2.then')]

// p1_1决策，将其绑定的回调加入微任务队列
// 2 [log('p2.then')，log('p1.then.then')]

// p2_1决策，将其绑定的回调加入微任务队列
//  2 [log('p1.then.then')，log('p2.then.then')]
```

```js
function log(i) {
    console.log(i)   
}

let p1 = new Promise((resolve) => {
    resolve()
})
log(1)
Promise.resolve().then(log.bind(null, 3)).then(log.bind(null, 6))
log(2)
p1.then(log.bind(null, 4))
p1.then(log.bind(null, 5))

// p1 同步决策
// p2 同步决策
// 1. [log(3), log(4), log(5)]
// p2_then1 被决策后才能把其绑定的回调加入微任务
// 2. [log(4), log(5), log(6)]
```

Resolve 测试

```js
let _resolve1;
let _resolve2;
let p1 = new Promise((res, rej) => {
    _resolve1 = res;
})
let p2 = new Promise((res, rej) => {
    _resolve2 = res;

    _resolve1(2)
})
p1.then(() => {
    console.log('p1')
})
p2.then(() => {
    console.log('p2')
})
console.log(_resolve1 == _resolve2)
```

