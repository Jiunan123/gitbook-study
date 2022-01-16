# postcss

## postcss-pxtorem

将css文件中的px转成rem

```js
// https://github.com/michael-ciniawsky/postcss-load-config

module.exports = {
    "plugins": {
        "postcss-import": {},
        "postcss-url": {},
        // to edit target browsers: use "browserslist" field in package.json
        /*"autoprefixer": {
            browsers: ['Android >= 4.0', 'iOS >= 7']
        },*/
        "postcss-pxtorem": {
            rootValue: 37.5, // 1rem = 37.5px 作为换算
            propList: ["*"]
        }
    }
}

```

```css
.c-test {
    height: 10rem;
    width: 10rem;
    background: red;
    display: inline-block;
}
.c-test2 {
    display: inline-block;
    // 375px / 37.5px/rem = 10rem。实际浏览器设备的rem=16px,则10rem = 160px
    height: 375px;
    width: 375px;
    background: blue;
    border: 1px solid red;
}
```

因此，设计稿若为375px宽度，则rootValue = 37.5

vant 的rootValue设置的比例是37.5

### 配置说明

1）`rootValue`（Number | Function）表示根元素字体大小或根据`input`参数返回根元素字体大小。

2）`unitPrecision` （Number）允许REM单位增加的十进制数字。

3）`propList` （Array）可以从px更改为rem的属性。

- 值必须完全匹配。
- 使用通配符`*`启用所有属性。例：`['*']`
- `*`在单词的开头或结尾使用。（`['*position*']`将匹配`background-position-y`）
- 使用`!`不匹配的属性。例：`['*', '!letter-spacing']`
- 将“ not”前缀与其他前缀组合。例：`['*', '!font*']`

4）`selectorBlackList` （Array）要忽略的选择器，保留为px。

- 如果value是字符串，它将检查选择器是否包含字符串。
  - `['body']` 将匹配 `.body-class`
- 如果value是regexp，它将检查选择器是否匹配regexp。
  - `[/^body$/]`将匹配`body`但不匹配`.body`

`5）replace` （Boolean）替换包含rems的规则。

6）`mediaQuery` （Boolean）允许在媒体查询中转换px。

7）`minPixelValue`（Number）设置要替换的最小像素值。

8）`exclude`（String, Regexp, Function）要忽略并保留为px的文件路径。

- 如果value是字符串，它将检查文件路径是否包含字符串。
  - `'exclude'` 将匹配 `\project\postcss-pxtorem\exclude\path`
- 如果value是regexp，它将检查文件路径是否与regexp相匹配。
  - `/exclude/i` 将匹配 `\project\postcss-pxtorem\exclude\path`
- 如果value是function，则可以使用exclude function返回true，该文件将被忽略。
  - 回调函数会将文件路径作为参数传递，它应该返回一个布尔结果。
  - `function (file) { return file.indexOf('exclude') !== -1; }`