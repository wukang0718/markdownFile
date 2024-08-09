### JSON.stringify

- 会把 NaN,Infinity,-Infinity都变成null
- 把function和Symbol删除
- 只能序列化自有可枚举属性

```js
const s = Symbol()
const a = {
    a: NaN,
    b: Infinity,
    c: -Infinity,
    d: function() {},
    e: 1,
    s: s
}
Object.defineProperty(a, 'h', {
    enumerable: false,
})
// json.stringify 方法会把 NaN,Infinity,-Infinity都变成null
// 把function和Symbol删除
// 只能序列化自有可枚举属性
console.log('json.stringify 方法会把 NaN,Infinity,-Infinity都变成null，把function和Symbol删除,只能序列化自有可枚举属性');
console.log(JSON.stringify(a));
```

