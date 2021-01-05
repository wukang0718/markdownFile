## 前言

关注 `vue2.0` 的同学都知道， `vue2.0` 实现响应式的方式的是通过 `Object.defineProperty，而 `vue3.0` 的响应式实现是用的 ES6 的新语法 `proxy` 代理实现的。

> 其实不是所有vue3.0 的 API 都是用 `proxy` 实现的，比如： `ref` 、`computed`，我们之后在解读 vue3 源码的时候详细解释

在 vue3.0 生态圈不断完善的今天，我们不可忽视的需要了解 vue3.0 的语法和实现方式，3.0 中最核心的 API 不过就是 `Proxy`了，你真的了解了 `Proxy` 了吗？

## 什么时Proxy

> **`Proxy`** 对象用于创建一个对象的代理，从而实现基本操作的拦截和自定义（如属性查找、赋值、枚举、函数调用等）   ----MDN

## Proxy 的语法

### 通过 `new` 关键字调用

可以接收两个参数

| 参数名称 | 参数描述                                                     |
| -------- | ------------------------------------------------------------ |
| target   | 需要 `proxy` 包装的对象（可以是任意类型的对象，包括数组、函数，甚至是另一个 `Proxy` 的实例） |
| handle   | 一个以函数作为属性的**对象**，属性定义了在执行代理操作时的行为 |

```javascript
const target = {};

const proxy = new Proxy(target, {
    get (target, key, receiver) {
        return 123;
    }
})
// 不管是通过 proxy 获取任何的属性的值，这里都会返回 123
console.log(proxy.getData)
```

### 静态方法 `Proxy.revocable`

接收和通过 `new` 调用一样的参数，返回值时一个**对象**，包含 `proxy` 和 `revoke` 两个属性

| 属性   | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| proxy  | `Proxy` 对象的实例，和通过 `new` 关键字调用返回一样的代理对象 |
| revoke | 函数，执行会销毁 `proxy` 属性的代理行为，在执行了 `revoke` 方法后，在使用 `proxy` 对象，会报错 |

```javascript
const target = {};

const {proxy, revoke} = Proxy.revocable(target, {
    get (target, key, receiver) {
        return 123;
    }
})
// 不管是通过 proxy 获取任何的属性的值，这里都会返回 123
console.log(proxy.getData); // 123
revoke(); // 销毁 proxy 代理
console.log(proxy.getData); // Uncaught TypeError: Cannot perform 'get' on a proxy that has been revoked
// 被销毁的代理对象不能在执行代理方法
```

## Proxy 可以代理哪些行为

`proxy` 顾名思义，就是“代理”的意思，那他都可以代理那些行为呢？

- `get` 代理对象属性的读取

```javascript
const target = {};

const proxy = new Proxy(target, {
    get (target, key, receiver) {
        return 123;
    }
})
// 不管是通过 proxy 获取任何的属性的值，这里都会返回 123
console.log(proxy.getData)
```

- 

## Proxy 对比 Object.defineProperty



## 参考文章

[【你不知道的 Proxy】：用 ES6 Proxy 能做哪些有意思的事情？](https://juejin.cn/post/6844904101218631694)

[MND Proxy 文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy)