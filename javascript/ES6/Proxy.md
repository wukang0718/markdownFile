## 前言

关注 `vue2.0` 的同学都知道， `vue2.0` 实现响应式的方式的是通过 `Object.defineProperty，而 `vue3.0` 的响应式实现是用的 ES6 的新语法 `proxy` 代理实现的。

> 其实不是所有vue3.0 的 API 都是用 `proxy` 实现的，比如： `ref` 、`computed`，我们之后在解读 vue3 源码的时候详细解释

在 vue3.0 生态圈不断完善的今天，我们不可忽视的需要了解 vue3.0 的语法和实现方式，3.0 中最核心的 API 不过就是 `Proxy`了，你真的了解了 `Proxy` 了吗？

## 什么时Proxy

> **`Proxy`** 对象用于创建一个对象的代理，从而实现基本操作的拦截和自定义（如属性查找、赋值、枚举、函数调用等）   ----MDN

ts 中对 `proxy` 接口的定义

```typescript

interface ProxyHandler<T extends object> {
    getPrototypeOf? (target: T): object | null;
    setPrototypeOf? (target: T, v: any): boolean;
    isExtensible? (target: T): boolean;
    preventExtensions? (target: T): boolean;
    getOwnPropertyDescriptor? (target: T, p: PropertyKey): PropertyDescriptor | undefined;
    has? (target: T, p: PropertyKey): boolean;
    get? (target: T, p: PropertyKey, receiver: any): any;
    set? (target: T, p: PropertyKey, value: any, receiver: any): boolean;
    deleteProperty? (target: T, p: PropertyKey): boolean;
    defineProperty? (target: T, p: PropertyKey, attributes: PropertyDescriptor): boolean;
    ownKeys? (target: T): PropertyKey[];
    apply? (target: T, thisArg: any, argArray?: any): any;
    construct? (target: T, argArray: any, newTarget?: any): object;
}

interface ProxyConstructor {
    revocable<T extends object>(target: T, handler: ProxyHandler<T>): { proxy: T; revoke: () => void; };
    new <T extends object>(target: T, handler: ProxyHandler<T>): T;
}
declare var Proxy: ProxyConstructor;
```

## Proxy 的语法

通过ts 的接口的定义可以发现 `Proxy` 有两种使用方式，可以通过 `new` 关键词调用，或者使用 `Proxy.revocable` 的静态方法。

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

`Proxy` 可以代理所有 `ProxyHandler` 中定义的行为，下面会详细的解释一下。

- `get` 代理对象属性的读取

  接收三个参数

  - target：源对象
  - key：获取的对象的属性名称
  - receiver：proxy 对象实例本身

  返回值可以是任意值，表示代理对象 `key` 属性的值

```javascript
const target = {};

const proxy = new Proxy(target, {
    get (target, key, receiver) {
        console.log(receiver === proxy) // true
        return 123;
    }
})
// 不管是通过 proxy 获取任何的属性的值，这里都会返回 123
console.log(proxy.getData)
```

- `set` 代理对象属性的设置

  接收四个参数

  - target：源对象
  - key：要设置的对象的属性
  - value：要设置的对象属性的值
  - receiver：proxy 实例本身

  返回 `Boolean` 类型的值。

```js
const target = {};

const proxy = new Proxy(target, {
    set (target, key, value, receiver) {
        target[key] = value;
        return true;
    }
})

proxy.getData = 123
console.log(target.getData) // 123
console.log(proxy.getData) // 123
```

- `has` 代理对象 `in` 操作符

  接收两个参数

  - target：源对象
  - key： 要判断的key

  返回 `Boolean` 类型的值

```js
const target = {};

const proxy = new Proxy(target, {
    has (target, key) {
        return key === 'getData'
    }
})

console.log("getData" in proxy)
console.log("a" in proxy)
```

- `getPrototypeOf` 代理 `Object.getPrototypeOf` 获取原型对象的行为

  接收一个参数

  - target：源对象

    返回一个对象或者 `null`

```js
const target = {};

const proxy = new Proxy(target, {
    getPrototypeOf (target) {
        return {}
    }
})
console.log(Object.getPrototypeOf(proxy))
console.log(Object.getPrototypeOf(target))
```

输出结果：

![image-20210107185116501](https://gitee.com/wu_kang0718/image/raw/master//20210107185117191.png)

- `setPrototypeOf` 代理 `Object.setPrototypeOf` 设置原型对象的行为

  接收两个参数

  - target：源对象
  - v：要设置成原型的对象可以是 对象或者 `null`

## Proxy 对比 Object.defineProperty



## 参考文章

[【你不知道的 Proxy】：用 ES6 Proxy 能做哪些有意思的事情？](https://juejin.cn/post/6844904101218631694)

[MND Proxy 文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy)