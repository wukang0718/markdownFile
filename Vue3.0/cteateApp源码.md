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

```typescript
/**
 * 惰性 创建 renderer
 */
function ensureRenderer() {
  return renderer || (renderer = createRenderer<Node, Element>(rendererOptions))
}
```

惰性创建 `renderer` 对象，这个对象的创建和运行的平台有关系，在 WEB 平台时，参数 `rendererOptions` 这个是 `DOM` 操作的 API

### createRenderer

```typescript
export function createRenderer<
  HostNode = RendererNode,
  HostElement = RendererElement
>(options: RendererOptions<HostNode, HostElement>) {
  return baseCreateRenderer<HostNode, HostElement>(options)
}
```

这个方法直接返回了 `baseCreateRenderer` 方法，``baseCreateRenderer` ` 方法有几个重载的方法。

### baseCreateRenderer

> 源码位置： [https://github.com/vuejs/vue-next/blob/master/packages/runtime-core/src/renderer.ts](https://github.com/vuejs/vue-next/blob/master/packages/runtime-core/src/renderer.ts)

```typescript
function baseCreateRenderer(
  options: RendererOptions,
  createHydrationFns?: typeof createHydrationFunctions
): any {
  // WEB 平台获取到的是操作 DOM 的方法
  const {
    insert: hostInsert,
    remove: hostRemove,
    patchProp: hostPatchProp,
    forcePatchProp: hostForcePatchProp,
    createElement: hostCreateElement,
    createText: hostCreateText,
    createComment: hostCreateComment,
    setText: hostSetText,
    setElementText: hostSetElementText,
    parentNode: hostParentNode,
    nextSibling: hostNextSibling,
    setScopeId: hostSetScopeId = NOOP,
    cloneNode: hostCloneNode,
    insertStaticContent: hostInsertStaticContent
  } = options
  const patch = () => {...}
  const processText = () => {...}
  const processCommentNode = () => {...}
  const mountStaticNode = () => {...}
  const patchStaticNode = () => {...}
  const moveStaticNode = () => {...}
  const removeStaticNode = () => {...}
  const processElement = () => {...}
  const mountElement = () => {...}
  const setScopeId = () => {...}
  const mountChildren = () => {...}
  const patchElement = () => {...}
  const patchBlockChildren = () => {...}
  const patchProps = () => {...}
  const processFragment = () => {...}
  const processComponent = () => {...}
  const mountComponent = () => {...}
  const updateComponent = () => {...}
  const setupRenderEffect = () => {...}
  const updateComponentPreRender = () => {...}
  const patchChildren = () => {...}
  const patchUnkeyedChildren = () => {...}
  const patchKeyedChildren = () => {...}
  const move = () => {...}
  const unmount = () => {...}
  const remove = () => {...}
  const removeFragment = () => {...}
  const unmountComponent = () => {...}
  const unmountChildren = () => {...}
  const getNextHostNode = () => {...}
  const render = () => {...}
  return {
    render,
    hydrate,
    // createApp 入口
    createApp: createAppAPI(render, hydrate)
  }
}
```

在 `createApp` 中调用了 `ensureRenderer().createApp(...args)` 方法获取 `app` 的实例，就是 `baseCreateRenderer` 返回的对象中的 `createApp` 函数，通过 `createAppAPI` 函数生成的一个函数

### createAppAPI

> 源码位置：[https://github.com/vuejs/vue-next/blob/master/packages/runtime-core/src/apiCreateApp.ts](https://github.com/vuejs/vue-next/blob/master/packages/runtime-core/src/apiCreateApp.ts)

```typescript
/**
 * 返回 app 实例
 * @param render 
 * @param hydrate 
 */
export function createAppAPI<HostElement>(
  render: RootRenderFunction,
  hydrate?: RootHydrateFunction
): CreateAppFunction<HostElement> {
  /**
   * 接收两个参数
   * rootComponent 根组件
   * rootProps 传递给根组件的 props 
   */
  return function createApp(rootComponent, rootProps = null) {
    if (rootProps != null && !isObject(rootProps)) {
      __DEV__ && warn(`root props passed to app.mount() must be an object.`)
      rootProps = null
    }

    const context = createAppContext()
    // 安装的插件
    const installedPlugins = new Set()
    // 是否挂载
    let isMounted = false
    
    const app = (context.app = {
      _uid: uid++,
      _component: rootComponent as ConcreteComponent,
      _props: rootProps,
      _container: null,
      _context: context,

      version,

      get config() {
        return context.config
      },

      set config(v) {
        if (__DEV__) {
          warn(
            `app.config cannot be replaced. Modify individual options instead.`
          )
        }
      }, 
    })
    
    return app
  }
}
```

















