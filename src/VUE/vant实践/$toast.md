# $toast 高级用法

```js
// 设置加载提示，直到手动清除
const toast = Toast.loading({
  duration: 0, // 持续展示 toast
  forbidClick: true,
  message: '倒计时 3 秒',
});

let second = 3;
const timer = setInterval(() => {
  second--;
  if (second) {
    toast.message = `倒计时 ${second} 秒`;
  } else {
    clearInterval(timer);
    // 手动清除 Toast
    Toast.clear();
  }
}, 1000);
```

```js
/**
 * 请求前拦截处理
 * 设置loading
 */
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
      forbidClick: false,
      duration: 0
    });
  }
  return config;
});

/**
 * 响应后拦截处理
 * 手动清除loading
 */
instance.interceptors.response.use(res => {
  // 合并配置参数（转换JSON，确保默认参数不被改变）
  let options = Object.assign(JSON.parse(JSON.stringify(defaultOptions)), userOptions);
  // 关闭加载提示
  if (options.loading) {
    Vue.prototype.$toast.clear();
  }
  if (res.status === 200) {
    // 弹出成功提示
    if (options.success) {
      Vue.prototype.$toast({
        message: res.data.msg,
        forbidClick: true,
        duration: options.duration
      });
    }
    // 重置用户参数
    userOptions = {};
    return Promise.resolve(res.data);
  } else {
    // 弹出错误提示
    if (options.success) {
      Vue.prototype.$toast({
        message: res.data.msg,
        forbidClick: true,
        duration: options.duration
      });
    }
    // 重置用户参数
    userOptions = {};
    return Promise.reject(res.data);
  }
}, err => {
  // 合并配置参数（转换JSON，确保默认参数不被改变）
  let options = Object.assign(JSON.parse(JSON.stringify(defaultOptions)), userOptions);
  // 关闭加载提示
  if (options.loading) {
    Vue.prototype.$toast.clear();
  }
  // 判断是否登陆异常
  if (err.response && err.response.data && err.response.data.status === 401) {
    // 移除Token令牌
    Vue.prototype.$storage.remove(Vue.prototype.$config.TOKEN);
    // 保存授权登录地址
    Vue.prototype.$storage.set(Vue.prototype.$config.OAUTH_URL, err.response.data.data);
    // 记录失效页面地址
    Vue.prototype.$storage.set(Vue.prototype.$config.REDIRECT_URI, location.href);
    // 跳转到登录页
    if (options.error) {
      Vue.prototype.$toast({
        message: err.response.data.message,
        forbidClick: true,
        duration: options.duration,
        onClose: function () {
          router.replace('/login');
        }
      });
    } else {
      router.replace('/login');
    }
  } else {
    if (options.error) {
      Vue.prototype.$toast({
        message: err.response.data.message,
        forbidClick: true,
        duration: options.duration
      });
    }
  }
  // 重置用户参数
  userOptions = {};
  return Promise.reject(err.response.data);
});
```

