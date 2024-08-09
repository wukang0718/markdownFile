# safari中的z-index设置没有生效的问题

解决方案：

```css
-webkit-transform: translate3d(0,0,0); // fix safari中zIndex失效的问题
```

设置`translate3d`属性会触发`GPU`加速

设置 `transform`会生成一个新的层叠上下文？

### 参考链接：

https://stackoverflow.com/questions/40895387/z-index-not-working-on-safari-fine-on-firefox-and-chrome