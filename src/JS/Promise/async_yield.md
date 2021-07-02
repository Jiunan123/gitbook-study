# async & yield

## 案例

```js
async function async1() { // 协程1
    console.log('2'); // 同步
    await async2(); // async2() 同步
    console.log('10');
    return 'end'
}
async function async2() { // 协程2
    console.log('3'); // 同步
    await new Promise((resolve) => {
        console.log('4'); // 同步
        resolve(); // 同步
    }).then(() => {
        console.log('7'); // 由于promise是立即决策，所以回调函数加入微任务队列[log(7)]
    })
    console.log('9');
}
// 主协程
console.log('1'); // 同步

async1().then((res) => console.log('11:', res));
// 同步：async1中的await交出代码控制权，且返回的promise未被决策，绑定回调函数

new Promise((resolve) => {
    console.log('5'); // 同步
    resolve(); // 同步
}).then(() => {
    console.log('8'); // 由于promise是立即决策，所以回调函数加入微任务队列[log(7), log(8)]
})
console.log('6'); // 同步

// macros = [log(7), log(8)]
// 开始执行微任务log(7), Promise被决策，async2_it.next加入微任务队列=[log(8), async2_it.next]
// 开始执行微任务log(8)
// 开始执行微任务 async2_it.next 即log(9), async2()的微任务被决策，async1_it.next 加入微任务[async1_it.next]
// 开始执行微任务 async1_it.next 即log(10), async1()的微任务被决策，其回调被加入微任务[log(11)]
// 开始执行微任务 log(11)
```

## await 内置生成器模拟

核心：(**递归**+**it.next放到微任务队列**)

- 执行async函数
- 首次立刻执行一次it.next，进入子协程，直到遇到第一个await
- await 紧跟的表达式会被转成promise，并绑定回调函数it.next
- await 紧跟的表达式立即执行，执行完成之后会交出代码控制权，即返回父协程
- 当promise 被决策，则将it.next放入微任务队列

```js
function executor(generator, ...args) {
    let iter = generator(...args); // 创建协程
    return new Promise((resolve => {
        (function _run(value) {
            let state = iter.next(value);
            if (state.done) {
                resolve(state.value);
            } else {// 防止传递传来的数据不是promise,进行封装。
                Promise.resolve(state.value).then((res) => {
                    _run(res)
                })
            }
        })();
    }))
} // 当且仅当所有状态遍历完成之后，对executor返回Promise的resolve加入微任务。

function* async1() {
    console.log('2');
    yield executor(async2);
    yield executor(async2);
    console.log('10');
    return 'end';
}
function* async2() {
    console.log('3');
    yield new Promise((resolve) => {
        console.log('4');
        resolve(); // 微任务1
    }).then(() => {
        console.log('7'); // 该then决断，产生微任务3
    })
    console.log('9');
}
// 主协程
console.log('1');
executor(async1).then((res) => {
    console.log('11:', res)
});
new Promise((resolve) => {
    console.log('5');
    resolve(); // 微任务2
}).then(() => {
    console.log('8');
})
console.log('6'); 
```

