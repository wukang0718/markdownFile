`form`组建接收的数据

```js
data: [{
  	type: {input | NumberKeyboard | Radio | Slider | Switch | TextareaItem},
    options: { // md-[type] 对应的设置
    	  
    },
    rules: {
      /**
       * rule： 规则名称
       */
      	rule: String | Array // 校验规则
    },
		name: "" // 表单校验显示的字段名称
}]
```



`form` 组件根据传入的 `type` 参数，动态渲染子组件



`Vue` 原型挂载方法 `$extendRule` 可以扩展校验规则



`validateInput` 组件接收数据

```js
props: {
  type: string, // 和 md 组件库的form 表单做一次映射，根据type渲染子组件
      options: {}, // md组件配置参数
    label: "", // 表单校验显示的字段名称
      props："", // 获取数据的 key 值
  	rules:   String | Array // 校验规则
}
```

`rules` 如果是 `array` =>  `join("|")`

`string` 直接赋值



抛出 `getData` 方法，给父组件获取数据使用