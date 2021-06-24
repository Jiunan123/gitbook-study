# HTML 模块加载

## 格式


```html
<!-- 模式1 -->
<script src="srcPath" type="module"></script>

<!-- 模式2 -->
<script type="module">
	import XX from "srcPath"
</script>
```

## 加载defer,module,async（[参考来源](https://www.cnblogs.com/sunshq/p/10076569.html)）

注意点：

- type=module 与 defer 一样，会在DOM解析完成后才开始解析script
- 普通的inline 的普通脚本（即非module）会忽略defer属性
- async 对普通脚本和module脚本都生效，加载完成后立即执行
- async、defer都不会阻塞Dom解析

例子1:

```html
<!-- 执行顺序 2.js 1.js 3.js -->
<script type="module" src="1.js"></script>
<script src="2.js"></script>
<script defer src="3.js"></script>
```

解析：

- 1.js 设置了module, 所以相当于标记了defer
- 2.js 没有设置defer，立即执行
- 3.js 设置了defer, 由于defer是会按先后顺序执行，所有1.js执行顺序优先于3.js

例子2:

```html
<!-- 1.js inline2 inline1 2.js -->
<script type="module">
  // inline1
  addTextToBody("Inline module executed");
</script>

<script src="1.js"></script>

<script defer>
  // inline2
  addTextToBody("Inline script executed");
</script>

<script defer src="2.js"></script>
```

解析：

- inline1 是module，所以相当于标记了defer
- 1.js 没有设置defer，立即执行
- Inline2 设置了defer，但是inline 普通脚本的defer不生效，立即执行
- 2.js 设置了defer，所以排在了inline1之后

例子3

```html
<!-- 这个脚本将会在imports完成后立即执行 -->
<script async type="module">
  import {addTextToBody} from './utils.js';

  addTextToBody('Inline module executed.');
</script>

<!-- 这个脚本将会在脚本加载和imports完成后立即执行 -->
<script async type="module" src="1.js"></script>
```

解析：由于async的作用，两个脚本都会在加载完成之后立即执行。

## 多次引人相同文件

- module脚本多次引入，只会执行一次
- 普通脚本多次引入，每次都会执行

## 跨域问题

- 普通脚本不受跨域影响
- module脚本会受跨域影响（跨域的脚本需携带 Access-Control-Allow-Origin: * 的CORS头信息）

## 凭证

```text
来源：https://www.cnblogs.com/sunshq/p/10076569.html
todo: 没看懂，以后再看。

没有凭证信息(credentials)
<!-- 有凭证信息 (cookies等) -->
<script src="1.js"></script>

<!-- 没有凭证信息 -->
<script type="module" src="1.js"></script>

<!-- 有凭证信息 -->
<script type="module" crossorigin src="1.js?"></script>

<!-- 没有凭证信息 -->
<script type="module" crossorigin src="https://other-origin/1.js"></script>

<!-- 有凭证信息-->
<script type="module" crossorigin="use-credentials" src="https://other-origin/1.js?"></script>
观看演示
当请求在同一安全域下，大多数的CORS-based APIs都会发送凭证信息 (cookies 等)，但 fetch() 和 module scripts 例外 – 它们并不发送凭证信息，除非我们要求它们。

我们可以通过添加crossorigin属性来让同源module脚本携带凭证信息，如果你也想让非同源module脚本也携带凭证信息，使用 crossorigin="use-credentials" 属性。需要注意的是，非同源脚本需要具有 Access-Control-Allow-Credentials: true 头信息。

同样，“Modules只执行一次”的规则也会影响到这一特征。URL是Modules的唯一标志，如果你先请求的Modules没有携带凭证信息，然后你第二次请求希望携带凭证信息，你仍然会得到一个没有凭证信息的module。这就是为什么上面的有个URL上我使用了一个?号，是让URL变的不同。
```

