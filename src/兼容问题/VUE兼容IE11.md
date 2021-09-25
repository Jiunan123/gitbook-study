# VUE兼容IE11

## 问题1: 解决ie下vue列表数据不能即时刷新的问题

[转载来源](https://www.cnblogs.com/yanggb/p/11563110.html)

项目上要兼容IE浏览器（客户要求），发现之前在谷歌浏览器下，操作（增删改查）列表后列表能即时刷新（双向绑定），IE下却不行。

自己调试一下发现，在IE11下，如果GET请求请求相同的URL，默认会使用之前请求来的缓存数据，而不会去请求接口获取最新数据。

另外，在F12开发者模式一直打开着的情况下，是能够正常即时刷新列表的，上面的假设也得到了进一步论证。

解决方法是，给每个请求的URL后加一个时间戳【new Date().getTime()】，这样就保证了每一次请求的URL都不同，IE11就会不断的请求接口而不使用缓存数据。

另外，更高级的做法是统一拦截所有请求，在请求发送前给每一个请求先加一个时间戳，更高效率地解决问题。

```js
export function api_getList(params) { // 获取记录列表
  return request({
    url: 'list?_t=' + new Date().getTime(),
    method: 'get',
    params: params
  })
}
```

