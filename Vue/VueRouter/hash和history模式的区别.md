## 面试题：VueRouter中的 hash 模式和 history 模式有什么区别

### hash模式

1. hash 模式的路由中带有 `#` 号
2. hash 模式通过 `window.onhashchange` 方法监听路由的修改
3. hash 模式在页面刷新的时候，发送的请求 url 是不带 # 后面的内容的
4. hash 模式可以兼容部分低版本的浏览器

### history模式

1. history 模式是使用正常的 url 路径显示
2. history 模式通过

### 总结

