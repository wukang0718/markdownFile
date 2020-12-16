## createApp

> 源码位置：[https://github.com/vuejs/vue-next/blob/master/packages/runtime-dom/src/index.ts](https://github.com/vuejs/vue-next/blob/master/packages/runtime-dom/src/index.ts)

代码实现：

```typescript
/**
 * createApp 函数
 */
export const createApp = ((...args) => {
  const app = ensureRenderer().createApp(...args)

  // 开发环境 校验 组件的 name 是不是内置的标签同名
  if (__DEV__) {
    injectNativeTagCheck(app)
  }

  const { mount } = app
  /**
   * 重写了 mount 函数
   * @param containerOrSelector 
   */
  app.mount = (containerOrSelector: Element | ShadowRoot | string): any => {
    // container 是真实的 DOM 元素
    const container = normalizeContainer(containerOrSelector)
    if (!container) return
    const component = app._component // 组件的options
    // 默认 组件的 template 是 挂载元素的内容
    if (!isFunction(component) && !component.render && !component.template) {
      component.template = container.innerHTML
    }
    // 清空 容器中的内容
    container.innerHTML = ''
    const proxy = mount(container)
    if (container instanceof Element) {
      // 删除元素上的 v-cloak 指令
      container.removeAttribute('v-cloak')
      container.setAttribute('data-v-app', '')
    }
    return proxy
  }

  return app
}) as CreateAppFunction<Element>
```

