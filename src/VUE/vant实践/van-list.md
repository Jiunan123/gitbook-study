# van-list

## 1. 首屏无限加载数据

导致问题伪代码：

```html
 <div class="c-list j-main_content">
   <van-list v-model="loading" :finished="finished" finished-text="没有更多了" ref="working-list" class="c-list__list" @load="onLoad" :offset="20">
     ...
   </van-list>
</div>

<style>
  .c-list {
    height: 500px;
    overflow-y: auto;
  }
  .c-list__list {
    overflow-y: auto;
  }
</style>
```

问题根源：

Van-list 源码：

```js
// 从van-list绑定的元素开始，向上查找，找到计算属性overflowY=scroll或者auto的元素。
// 因此，问题代码中，找到的scroller为.c-list__list
const overflowScrollReg = /scroll|auto/i;
function getScroller(el: HTMLElement, root: ScrollElement = window) {
  let node = el;

  while (
    node &&
    node.tagName !== 'HTML' &&
    node.tagName !== 'BODY' &&
    node.nodeType === 1 &&
    node !== root
  ) {
    const { overflowY } = window.getComputedStyle(node);
    if (overflowScrollReg.test(overflowY)) {
      return node;
    }
    node = node.parentNode as HTMLElement;
  }

  return root;
}
// render时，列表尾部插入<div ref="placeholder" />
function render() {
    const Placeholder = (
      <div ref="placeholder" key="placeholder" class={bem('placeholder')} />
    );

    return (
      <div class={bem()} role="feed" aria-busy={this.innerLoading}>
        {this.direction === 'down' ? this.slots() : Placeholder}
        {this.genLoading()}
        {this.genFinishedText()}
        {this.genErrorText()}
        {this.direction === 'up' ? this.slots() : Placeholder}
      </div>
    );
}

function check() {
  this.$nextTick(() => {
    if (this.innerLoading || this.finished || this.error) {
      return;
    }

    const { $el: el, scroller, offset, direction } = this;
    let scrollerRect;
		
    if (scroller.getBoundingClientRect) {
      // 滚动容器相对视口的位置
      scrollerRect = scroller.getBoundingClientRect();
    } else {
      scrollerRect = {
        top: 0,
        bottom: scroller.innerHeight,
      };
    }

    const scrollerHeight = scrollerRect.bottom - scrollerRect.top;

    /* istanbul ignore next */
    if (!scrollerHeight || isHidden(el)) {
      return false;
    }

    let isReachEdge = false;
    // placeholder 相对视口的位置
    const placeholderRect = this.$refs.placeholder.getBoundingClientRect();

    if (direction === 'up') {
      isReachEdge = scrollerRect.top - placeholderRect.top <= offset;
    } else {
      // 比对滚动容器的底部与实际列表的底部距离
      isReachEdge = placeholderRect.bottom - scrollerRect.bottom <= offset;
    }

    if (isReachEdge) {
      this.innerLoading = true;
      this.$emit('input', true);
      this.$emit('load');
    }
  });
}

```

由于问题代码中，

1. 父元素.c-list是overflow-y: auto，所以父元素的外部盒子大小为指定的height: 500px; 但是内部容器盒子的大小是实际内容长度。
2. 子元素.c-list__list的高度为auto, overflow-y: auto, 所以外部盒子大小为内容实际长度，内部容器盒子大小也是。
3. 所以scroller为.c-list__list时，滚动并没有生效，所以列表相对于视口的位置的bottom与placeholderRect的bottom一直都是一样的。因此 isReachEdge = 0 <= offset; // true

```html
<style>
  .c-list {
    height: 500px;
    overflow-y: auto;
  }
  .c-list__list {
    // 方法1: 注释掉子元素的滚动设置
    // 方法2: 指定子元素的高度
    // overflow-y: auto;
  }
</style>
```

## 2. 滚动到底部，触发加载数据后，滚动条回弹到列表顶部

问题代码：

```js
// axios的拦截器设置了加载提示

instance.interceptors.request.use(config => {
  // 合并配置参数（转换JSON，确保默认参数不被改变）
  let options = Object.assign(JSON.parse(JSON.stringify(defaultOptions)), userOptions);
  // 添加请求头Token令牌
  let token = Vue.prototype.$storage.get(Vue.prototype.$config.TOKEN);
  if (token) {
    config.headers.token = token;
  }
  // 弹出加载提示
  if (options.loading) {
    Vue.prototype.$toast({
      type: 'loading',
      message: '加载中...',
      forbidClick: true,
      duration: 0
    });
  }
  return config;
});
```

$toast 源码：

```js
{
  watch: {
    value: 'toggleClickable',
    forbidClick: 'toggleClickable',
  },
  toggleClickable() {
    const clickable = this.value && this.forbidClick;

    if (this.clickable !== clickable) {
      this.clickable = clickable;
      lockClick(clickable);
    }
  },
};
  
function lockClick(lock: boolean) {
  if (lock) {
    if (!lockCount) {
      document.body.classList.add('van-toast--unclickable');
    }

    lockCount++;
  } else {
    lockCount--;

    if (!lockCount) {
      document.body.classList.remove('van-toast--unclickable');
    }
  }
}


```

```scss
.van-toast {
    &--unclickable {
    // lock scroll
    overflow: hidden;

    // should not add pointer-events: none directly to body tag
    // that will cause unexpected tap-highlight-color in mobile safari
    * {
      pointer-events: none;
    }
  }
}
```

1. 当forbidClick为true时，则body元素会被添加.van-toast--unclickable
2. pointer-events: none; 这个属性是元凶
3. Todo: 听说是因为重绘引起列表的scrollTop = 0; pointer-events属性待学习
4. 总结，滚动触发axios请求后，导致 pointer-events 属性被应用，引起列表的scrollTop = 0。

修复代码:

```js
Vue.prototype.$toast({
  type: 'loading',
  message: '加载中...',
  forbidClick: false, // true -> false
  duration: 0
});
```

## 3. 注意事项

1. 当去请求数据的时候，需要设置 loading = true, 否则会产生多个请求（onload一直触发）。

