组建接收的数据

```js
props: {
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
    }
}
```

