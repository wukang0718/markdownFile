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

调用 `ensureRenderer().createApp(...args)` 方法，获取到了 `app` 的实例；然后重写了 `app` 的 `mount` 方法，在新的 `mount` 方法中，先对 `container` 做了一次处理（如果传的是css选择器，通过 ` document.querySelector` 方法获取到的DOM元素 ），让 `container` 是一个真实的 `DOM元素`。

在组件不是一个函数，并且没有设置 `render` 函数和 `tempalte` 参数的时候，默认 `container` 中的 html 元素就是组件的 `template`。

调用 `app` 中返回的 `mount` 方法，完成 `DOM` 的挂载

### ensureRenderer

惰性创建 `renderer` 对象，这个对象的创建和运行的平台有关系

```typescript
/**
 * 惰性 创建 renderer
 */
function ensureRenderer() {
  return renderer || (renderer = createRenderer<Node, Element>(rendererOptions))
}
```

























