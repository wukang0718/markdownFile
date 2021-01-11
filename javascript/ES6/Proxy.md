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

`Proxy` 可以代理对象的13种行为：

### `get` 代理对象属性的读取

接收三个参数

- target：源对象
- key：获取的对象的属性名称
- receiver：proxy 对象实例本身

> 如果违背了以下的约束，proxy会抛出 `TypeError`:
>
> - 如果要访问的目标属性是不可写以及不可配置的，则返回的值必须与该目标属性的值相同，即当源对象的属性的描述符 `configurable` 和 `writable` 为 `false`，必须返回和源对象的属性一样的值 。
> - 如果要访问的目标属性没有配置访问方法，即get方法是undefined的，则返回值必须为undefined，即当通过 `Object.defineProperty` 给源对象设置属性，并且没有设置 `get` 是，代理必须返回 `undefined`。

#### 三种可以触发 `get` 的方法

```

```



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

### `set` 代理对象属性的设置

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

### `has` 代理对象 `in` 操作符

接收两个参数

- target：源对象
- key： 要判断的key

返回 `Boolean` 类型的值

> 如果违反了下面这些规则,  proxy 将会抛出 `TypeError`:
>
> - 如果目标对象的某一属性本身不可被配置，则该属性不能够被代理隐藏，即如果对象树形的描述符 `configurable` 值为 `false`，`has` 代理不能返回 `false` .
> - 如果目标对象为不可扩展对象，则该对象的属性不能够被代理隐藏，即源对象或代理对象被 `Object.preventExtensions` 方法调用后，不能返回`false`

#### 两种可以触发 `has`的方法

```
属性查询: foo in proxy
继承属性查询: foo in Object.create(proxy)
with 检查: with(proxy) { (foo); }
Reflect.has()
```

示例代码：

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

#### **五种可以触发 `getPrototypeOf` 的方式**

```js
Object.getPrototypeOf(p) // Object.getPrototypeOf
Reflect.getPrototypeOf(p) // 反射方法 Reflect.getPrototypeOf
p.__proto__ // __proto__ 属性
Array.prototype.isPrototypeOf(p) // isPrototypeOf 方法
p instanceof Array    // instanceof 操作符
```

示例代码:

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

### `setPrototypeOf` 代理 `Object.setPrototypeOf` 设置原型对象的行为

接收两个参数

- target：源对象
- v：要设置成原型的对象可以是 对象或者 `null`

返回一个 `Boolean` 类型

#### 两种可以触发 `setPrototypeOf` 的方法

```js
Object.setPrototypeOf()
Reflect.setPrototypeOf() // 反射方法
```

示例代码：

```js
const target = {};

const proxy = new Proxy(target, {
    setPrototypeOf (target, v) {
        return Reflect.setPrototypeOf(target, v) // 反射也是ES6中提供的新的操作对象的方法
    }
})
Object.setPrototypeOf(proxy, null)
console.log(Object.getPrototypeOf(proxy)) // null
console.log(Object.getPrototypeOf(target)) // null
Object.setPrototypeOf(proxy, {a: 1})
console.log(Object.getPrototypeOf(proxy)) // {a: 1}
console.log(Object.getPrototypeOf(target)) // {a: 1}
```

### `isExtensible` 代理 `Object.isExtensible()` 判断对象是否可扩展的行为

接收一个参数：

- target：源对象

返回一个 `Boolean`

> 如果违背了以下的约束，`proxy` 会抛出 `TypeError`:
>
> - `Object.isExtensible(proxy)` 必须同`Object.isExtensible(target)`返回相同值。也就是必须返回`true`或者为`true`的值,返回`false`和为`false`的值都会报错。

#### 两种可以触发 `isExtensible`  方法的操作：

```js
Object.isExtensible()
Reflect.isExtensible()
```

示例代码：

```js
const target = {};

const proxy = new Proxy(target, {
    isExtensible (target) {
        return true
    }
})

console.log(Object.isExtensible(proxy)) // true
console.log(Object.isExtensible(target)) // true
```

**错误示例**

```js
const target = {};

const proxy = new Proxy(target, {
    isExtensible (target) {
        return false
    }
})

console.log(Object.isExtensible(proxy))
```

![image-20210107190807219](https://gitee.com/wu_kang0718/image/raw/master//20210107190807908.png)

### `preventExtensions` 代理 `Object.preventExtensions` 让对象变成不可扩展的行为

接收一个参数：

- target：源对象

返回一个 `Boolean`

> 如果违反了下列规则, proxy则会抛出一个 `TypeError`:
>
> - 如果目标对象是可扩展的，那么只能返回 `false`

#### 两种可以触发 `preventExtensions` 的方式：

```js
Object.preventExtensions()
Reflect.preventExtensions()
```

示例代码：

```js
const target = {};

const proxy = new Proxy(target, {
    preventExtensions (target) {
        return Reflect.preventExtensions(target)
    }
})

Object.preventExtensions(proxy)
console.log(Object.isExtensible(proxy)) // false
console.log(Object.isExtensible(target)) // false
```

### `getOwnPropertyDescriptor` 代理 `Object.getOwnPropertyDescriptor` 获取对象上的一个自有属性的描述符的行为

接收两个参数：

- target：源对象
- key：要获取属性描述符的对象的key

返回 `PropertyDescriptor` 或 `undefined`， 获取的 key 不存在的时候返回 `undefined`

> 如果下列行为被违反，代理将抛出一个 `TypeError`：
>
> - `getOwnPropertyDescriptor` 必须返回一个 `object` 或 `undefined`
>
> - 如果属性作为目标对象的不可配置的属性存在，则该属性无法报告为不存在，即如果源对象中的属性配置符的 `configurable` 属性为 `false` 时，`getOwnPropertyDescriptor` 返回的配置符必须和源对象属性的配置符一致。
> - 如果属性作为目标对象的属性存在，并且目标对象不可扩展，则该属性无法报告为不存在，即当源对象时不可扩展的（使用了 `Object.preventExtensions` 方法后）并且对象上有要查找的属性，那么`getOwnPropertyDescriptor` 返回的配置符必须和源对象属性的配置符一致。
> - 属性不能被报告为不可配置，如果它不作为目标对象的自身属性存在，或者作为目标对象的可配置的属性存在，即：
>   - 当源对象上有要查找的属性，并且属性描述符的 `configurable` 是 `true`，那么`getOwnPropertyDescriptor` 返回的配置符中的 `configurable` 也必须是 `true`
>   - 或者是当源对象上不存在这个属性的时候，那么`getOwnPropertyDescriptor` 返回的配置符中的 `configurable` 必须是 `true`
> - `Object.getOwnPropertyDescriptor(target)`的结果可以使用 `Object.defineProperty` 应用于目标对象，也不会抛出异常，即通过 `getOwnPropertyDescriptor` 方法返回的不是 `undefined` 的结果，可以被 `Object.defineProperty` 方法调用，设置为任何对象的属性操作符。

#### 两种可以触发 `getOwnPropertyDescriptor` 的方式

```js
Object.getOwnPropertyDescriptor()
Reflect.getOwnPropertyDescriptor()
```

示例代码：

```js
const target = {
    a: 123
};

const proxy = new Proxy(target, {
    getOwnPropertyDescriptor (target, key) {
        return Reflect.getOwnPropertyDescriptor(target, key)
    }
})
console.log(Object.getOwnPropertyDescriptor(proxy, 'a')) // {value: 123, writable: true, enumerable: true, configurable: true}
```

### `defineProperty` 代理 `Object.defineProperty` 在对象上添加一个新的属性或者修改对象上现有属性的行为

接口三个参数

- target： 源对象
- key：要添加或者修改的属性
- attributes： 要添加或修改的属性的描述符

返回 `Boolean`

> 如果违背了以下的行为，proxy会抛出 `TypeError`:
>
> - 如果目标对象不可扩展， 将不能添加属性，即对源对象或者 `proxy` 示例调用过 `Object.preventExtensions` 或者 `Reflect.preventExtensions` 方法后，不能再使用 `defineProperty` 
> - 不能添加或者修改一个属性为不可配置的，如果它不作为一个目标对象的不可配置的属性存在的话，即当源对象上没有要扩展的属性的时，不能通过 `defineProperty` 方法设置 `configurable` 为 `false` 的描述符
> - 如果目标对象存在一个对应的可配置属性，这个属性可能不会是不可配置的，即当要配置的属性在源对象上的描述符中的 `configurable` 为 `true`，那么就不能通过 `defineProperty` 设置 `configurable` 为 `false` 的描述符
> - 如果一个属性在目标对象中存在对应的属性，那么 `Object.defineProperty(target, prop, descriptor)` 将不会抛出异常。
> - 在严格模式下， `false` 作为` handler.defineProperty` 方法的返回值的话将会抛出 `TypeError`

## Proxy 对比 Object.defineProperty



## 参考文章

[【你不知道的 Proxy】：用 ES6 Proxy 能做哪些有意思的事情？](https://juejin.cn/post/6844904101218631694)

[MND Proxy 文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy)















