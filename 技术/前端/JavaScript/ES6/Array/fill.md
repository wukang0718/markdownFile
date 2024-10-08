#	Array.prototype.fill

##	作用

> `fill()` 方法用一个固定值填充一个数组中从起始索引到终止索引内的全部元素。不包括终止索引。

##	语法

> ​	`arr.fill(value[, start[, end]])`

###		参数

| 参数          | 作用                          |
| ------------- | ----------------------------- |
| value         | 用来填充数组元素的值          |
| statr（可选） | 起始索引，默认收0             |
| end（可选）   | 结束索引，默认值是this.length |

###	返回值

返回修改后的数组（返回的是原数组）

##	示例

```javascript
let arr = Array(5);
let arr1 = arr.fill(1);
console.log(arr === arr1); // true
let arr2 = [1, 2, 3, 4]
arr2.fill(6, 2, 4);
console.log(arr2); //  [1, 2, 6, 6]
```



##	注意事项

* 该方法修改的是原数组