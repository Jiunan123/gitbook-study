# mocha + chai

## 概念

1. 测试套件：一组相关的测试，称为测试套件（describe）
2. 测试用例：一个用例（it）
3. 套件、用例钩子：before、after、beforeEach、afterEach
4. Mocha: 测试框架
5. Chai: 断言
   1. 三种断言风格：should、expect、assert
   2. 判断内容：大、小、等、包含
6. mock：跨组件请求或者前端请求后端，通过mock进行模拟返回。
7. 超时配置：

## 例子

Mocha：

```js
import { expect } from 'chai'
import { shallowMount } from '@vue/test-utils'
import HelloWorld from '@/components/HelloWorld.vue'

// 套件
describe('HelloWorld.vue', () => {
  // 用例
  it('renders props.msg when passed', () => {
    const msg = 'new message'
    // shallowMount 渲染组件
    // 断言页面包含哪些内容。
    const wrapper = shallowMount(HelloWorld, {
      propsData: { msg }
    })
    // 断言
    expect(wrapper.text()).to.include(msg)
  })
})

```

官方例子：

```vue
<template>
  <div>
    <input v-model="username">
    <div
      v-if="error"
      class="error"
    >
      {{ error }}
    </div>
  </div>
</template>

<script>
export default {
  name: 'Hello',
  data () {
    return {
      username: ''
    }
  },

  computed: {
    error () {
      return this.username.trim().length < 7
        ? 'Please enter a longer username'
        : ''
    }
  }
}
</script>
```

测试脚本

```js
import { shallowMount } from '@vue/test-utils'
import Hello from './Hello.vue'

test('Hello', () => {
  // 渲染这个组件
  const wrapper = shallowMount(Hello)

  // `username` 在除去头尾空格之后不应该少于 7 个字符
  wrapper.setData({ username: ' '.repeat(7) })

  // wrapper.find方式可以对DOM元素进行获取。进行一些模拟操作
  // 确认错误信息被渲染了
  expect(wrapper.find('.error').exists()).toBe(true)

  // 将名字更新至足够长
  wrapper.setData({ username: 'Lachlan' })

  // 断言错误信息不再显示了
  expect(wrapper.find('.error').exists()).toBe(false)
})
```

- shallowMount 和 mount 的区别：
  - mount是完整的渲染
  - shallowMount渲染的子组件是假的，也就是只mount了这一层
- 
