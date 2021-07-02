# Array

##  会改动原数组方法

- 处理数组第一个元素

  - 插入到首部：unshift
  - 移除第一个元素：shift

- 处理数组最后一个元素

  - 插入到尾部：push
  - 移除尾部元素：pop

- 处理数组一系列元素：arr.splice(start, deleteLength, new1, new2, ...)

- 排序：

  - 按指定顺序排序：sort
  - 数据翻转：reserve

- arr.length=0 与arr=[]之间的差异

  arr.length会影响到引用，arr=[]不会影响到引用。

  ```js
  var foo = [1,2,3];
  var bar = [1,2,3];
  var foo2 = foo;
  var bar2 = bar;
  foo = [];
  bar.length = 0;
  console.log(foo, bar, foo2, bar2); // [], [], [1, 2, 3], []
  ```

  