### 生产环境的 `keep-alive`  组件无效

#### 原因

`keep-alive` 组件的 `include` prop 需要指定的组件的 `name` 属性，在 开发环境的时候，回将 类组件的类名作为组件的 `name` 属性， 执行 `build`	的时候回忽略组件的类名

#### 解决

需要显示的声明组件的 `name` 

```jsx
@Component({
	name: "NameComponent"
})
export default class NameComponent extend Vue{}
```

