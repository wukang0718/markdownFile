# Vue3使用h函数渲染响应式数据

vue3 使用 `h` 函数渲染的组件 `props`传入 `ref`但是没有响应修改

要用`effect`函数把 `h` 函数包裹起来才可以

只有在`effect`函数中执行的函数中的依赖才会被收集

```javascript
import { h, render, ref, effect, stop } from 'vue'
const modelValue = ref(1)
// 渲染响应式数据
const runner = effect(() => {
  const instance = h(CarTraceSlider, {
    max: 1,
    min: 0,
    modelValue: modelValue.value,
  })
  render(instance, element)
})
modelValue.value = 2;// 数据修改可以被响应
if (runner) {
  stop(runner)
}
```