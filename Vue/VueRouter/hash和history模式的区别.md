## 面试题：VueRouter中的 hash 模式和 history 模式有什么区别

### hash模式

1. hash 模式的路由中带有 `#` 号
2. hash 模式通过 `window.onhashchange` 方法监听路由的修改
3. hash 模式在页面刷新的时候，发送的请求 url 是不带 # 后面的内容的
4. hash 模式可以兼容部分低版本的浏览器

### history模式

1. history 模式是使用正常的 url 路径显示
2. history 模式通过 `pushState` 和 `replaceState` 方式修改路由改变
3. history 模式在页面刷新的时候，会请求当前地址栏中完成的 url，这时需要服务器对这个 url 有处理，如果没有对应的文件，需要返回 index.html
4. history 模式因为是使用的 HTML5 的新规范，所以不能兼容低版本的浏览器

### 其他注意事向

1. 有些

### 总结

