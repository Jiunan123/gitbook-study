1.区别：vuex存储在内存，localstorage（本地存储）则以文件的方式存储在本地,永久保存（不主动删除，则一直存在）；sessionstorage( 会话存储 ) ,临时保存。localStorage和sessionStorage只能存储字符串类型，对于复杂的对象可以使用ECMAScript提供的JSON对象的stringify和parse来处理

2.应用场景：vuex用于组件之间的传值，localstorage，sessionstorage则主要用于不同页面之间的传值。

3.永久性：当刷新页面（这里的刷新页面指的是 --> F5刷新,属于清除内存了）时vuex存储的值会丢失，sessionstorage页面关闭后就清除掉了，localstorage不会。

注：大家可能觉得用localstorage可以代替vuex, 对于不变的数据确实可以，但是当两个组件共用一个数据源（对象或数组）时，如果其中一个组件改变了该数据源，希望另一个组件响应该变化时，localstorage，sessionstorage无法做到，原因就是区别1。