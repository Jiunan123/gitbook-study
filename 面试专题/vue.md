# VUE

## vue

### 生命周期

父子间的生命周期执行顺序(双V，单执行直到 mounted, 转折)

```text
p->beforeCreate
p->created
p->beforeMount
s->beforeCreate
s->created
s->beforeMount
s->mounted
p->mounted

p->beforeDestroy
s->beforeDestroy
s->destroyed
p->destroyed

// 若父亲组件传参给子组件，更新
p->beforeUpdate
s->beforeUpdate
s->updated
p->updated
```

### nextTick

- 触发nextTick两种形式：
  - 隐式：触发响应式属性的setter属性
  - 显式：直接调用nextTick

### v-for、v-if 为什么不能一起使用

影响性能：首先，v-for 的优先级高于 v-if，则先进行遍历，再进行判断。

- 若v-if判断条件与遍历内容无关时，则原本1次判断就够的问题，进行了N次的判断
- 若v-if判断条件与遍历内容有关时，且判断在某种情况下才进行呈现。则若这个数据中的有一半数据需要呈现，一半数据不需要呈现，当修改数组中不需要呈现的元素时，render 函数也会被重新触发。若采用computed 方式先对数组进行一次过滤，再使用v-for进行遍历渲染的话，则更改非呈现的元素时，则不会重新触发render。

## vue-router

### router.beforeEach

### router.afterEach

### route.beforeEnter

### component.beforeRouteEnter

### component.beforeRouteUpdate

### component.beforeRouteLeave

### 完整的导航解析流程

1. 导航被触发。
2. 在失活的组件里调用 `beforeRouteLeave` 守卫。
3. 调用全局的 `beforeEach` 守卫。
4. 在重用的组件里调用 `beforeRouteUpdate` 守卫 (2.2+)。
5. 在路由配置里调用 `beforeEnter`。
6. 解析异步路由组件。
7. 在被激活的组件里调用 `beforeRouteEnter`。
8. 调用全局的 `beforeResolve` 守卫 (2.5+)。
9. 导航被确认。
10. 调用全局的 `afterEach` 钩子。
11. 触发 DOM 更新。
12. 调用 `beforeRouteEnter` 守卫中传给 `next` 的回调函数，创建好的组件实例会作为回调函数的参数传入。