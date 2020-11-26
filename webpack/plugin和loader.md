## 面试题： Webpack 的 plugin 和 loader 有什么区别

### webpack 的打包原理

1. 识别入口文件
2. 通过逐层识别模块依赖(Commonjs、amd或者es6的import，webpack都会对其进行分析，来获取代码的依赖)
3. webpack做的就是分析代码，转换代码，编译代码，输出代码
4. 最终形成打包后的代码

### 什么是 loader

loader 是文件加载器，能够加载资源文件，并对这些文件进行一些处理，诸如编译、压缩等，最终一起打包到指定的文件中。

1. 处理一个文件可以使用多个 loader，loader 的加载顺序和配置顺序是相反的
2. 第一个执行的 loader 接收源文件的内容作为参数，下一个 loader 接收前一个 loader 的返回值作为参数，最后一个 loader 会返回此模块的 javaScript 的源码

### 什么是 plugin

在 webpack 运行的生命周期中会广播很多的事件（ hooks中的事件监听 ），plugin 可以监听这些事件（在 apply 方法中监听 hooks 的声明周期），在合适的时机通过 webpack 提供的 Api 改变输出结果 。

### loader 和 plugin 的区别

对于 loader 来说，他是一个转换器，