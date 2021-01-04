## 前言

关注 `vue2.0` 的同学都知道， `vue2.0` 实现响应式的方式的是通过 `Object.defineProperties`，而 `vue3.0` 的响应式实现是用的 ES6 的新语法 `proxy` 代理实现的。

> 其实不是所有vue3.0 的 API 都是用 `proxy` 实现的，比如： `ref` 、`computed`，我们之后在解读 vue3 源码的时候详细解释

在 vue3.0 生态圈不断完善的今天，我们不可忽视的需要了解 vue3.0 的语法和实现方式，3.0 中最核心的 API 不过就是 `Proxy`了，你真的了解了 `Proxy` 了吗？

## Proxy 可以解决什么问题



## 参考文章

[【你不知道的 Proxy】：用 ES6 Proxy 能做哪些有意思的事情？](https://juejin.cn/post/6844904101218631694)