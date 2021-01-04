### 前言

关注 `vue2.0` 的同学都知道， `vue2.0` 实现响应式的方式的是通过 `Object.defineProperties`，而 `vue3.0` 的响应式实现是用的 ES6 的新语法 `proxy` 代理实现的（其实不是所有的 API 都是用 `proxy` 实现的，比如： `ref` 、`computed`，我们之后在解读 vue3 源码的时候详细解释）。