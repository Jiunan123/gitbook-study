- mount和shallowMount的区别
   mount是完整的渲染（推荐使用）
   shallowMount渲染的子组件是假的，也就是只mount了这一层
- 测试props的时候只能通过html来测试
   不能直接通过wrapper.props().属性，来测试，要通过html上的原生属性来测试
   比如测试input接受value



```csharp
//正确的测试
it('接受value', () => {
  const wrapper = mount(Input, {
    propsData: {value: '123'}
  })
    const vm = wrapper.vm
    const input = vm.$el.querySelector('input')
    expect(input.value).to.equal('123')
})

//错误的测试
it('接受value', () => {
  const wrapper = mount(Input, {
    propsData: {value: '123'}
  })
    expect(wrapper.props().value).to.equal('123')
})
```

而且只有dom元素本身有这个属性的时候才能测试，上面的value就是input自有的属性



```csharp
it('接受readonly', () => {
  const wrapper = mount(Input, {
    propsData: {
      readonly: false
    }
  })
    const vm = wrapper.vm
    const inputElement = vm.$el.querySelector('input')
    //因为input没有readonly属性，所以是undefined，测试失败
    console.log(inputElement.readonly)
    expect(inputElement.readonly).to.equal(false)
})
```

而attributes只能获取到dom本身自有的属性，比如id，name什么的，自定义的也获取不到



作者：sweetBoy_9126
链接：https://www.jianshu.com/p/f6a4c9499868
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。