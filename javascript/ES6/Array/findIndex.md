# Array.prototype.findIndex

## 作用

`findIndex()`方法返回数组中满足测试函数的第一个元素的**索引**.否则返回-1

## 语法

> ​	arr.findIndex(callback[, thisArg])

### 参数

| 参数          | 作用                                       |
| ------------- | ------------------------------------------ |
| callback      | 针对数组的每个元素都会执行的函数的回调函数 |
| thisArg(可选) | 执行callback时作为this的对象的值           |



#### callback的参数

| 参数    | 作用           |
| ------- | -------------- |
| element | 当前遍历到的值 |
| index   | 当前遍历的索引 |
| array   | 当前数组本身   |

### 返回值

数组中通过测试函数的第一个元素的索引,否则返回-1

## 示例

```js
let arr = [1, 2, 3, 4];
console.log(arr.findIndex(item => item % 3 === 0)) // 2
```

