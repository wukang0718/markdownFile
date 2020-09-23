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
      	rule: {
          validator // function | string | 
          message: 
        }
    },
		name: "" // 表单校验显示的字段名称
}]
```



`form` 组件根据传入的 `type` 参数，动态渲染子组件



`Vue` 原型挂载方法 `$extendRule` 可以扩展校验规则



`validateInput` 组件接收数据

```js
props: {
		options: {}, // md组件配置参数
    label: "", // 表单校验显示的字段名称
      props："", // 获取数据的 key 值
  	rules: Object | String | Array // 校验规则
}
```

`rules` 如果是 `array` =>  `join("|")`

`string` 直接赋值

`object` 判断值是 `string` 或者是 `object` 中包含 `message` 字段



抛出 `getData` 方法，给父组件获取数据使用