#	Array.prototype.find

##	作用

`find()`方法返回数组中满足提供的测试函数的第一个元素的值，如果没有符合的值，则返回undefined

该方法不会修改原数组

##	语法

> ​	`arr.find(callback[, thisArg])`

###	参数

| 参数            | 作用                                   |
| --------------- | -------------------------------------- |
| callback        | 在数组的每一项上执行的函数             |
| thisArg（可选） | 执行回调函数时this的值，对箭头函数无效 |

####	callback函数参数

| 参数    | 作用             |
| ------- | ---------------- |
| element | 当前遍历到的元素 |
| index   | 当前遍历到的索引 |
| array   | 数组本身         |

###	返回值

数组中第一满足所提供的测试函数的值，否则返回undefined

## 示例

```js
let arr = [1, 2, 3, 4];
console.log(arr.find((item, index, array) => item % 2 === 0)) // 2 只返回第一个值
conoole.log(arr.find(item => item > 10)) // undefined
```