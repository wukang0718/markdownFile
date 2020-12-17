## createApp

> 源码位置：[https://github.com/vuejs/vue-next/blob/master/packages/runtime-dom/src/index.ts](https://github.com/vuejs/vue-next/blob/master/packages/runtime-dom/src/index.ts)

> 都是源码，比较干。。。
>
> v 3.0.4

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

在组件不是一个函数，并且没有设置 `render` 函数和 `tempalte` 参数的时候，默认 `container` 中的 `innerHTML` 就是组件的 `template`。

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

这个方法直接返回了 `baseCreateRenderer` 方法，`baseCreateRenderer`  方法有几个重载的方法。

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
    const context = createAppContext() // 返回一个对象
    // 安装的插件
    const installedPlugins = new Set()
    // 是否挂载
    let isMounted = false
    
    const app = (context.app = {
        _uid: uid++, // 唯一id
        _component: rootComponent as ConcreteComponent,
        _props: rootProps,
        _container: null,
        _context: context,

        version, // vue 版本

        get config() { // config 是一个只读对象，设置 config 在开发环境会报警告
            return context.config
        },

        use() {...},
        mixin() {...},
		component() {...},
        directive() {...},
        mount() {...},
        unmount() {...},
        provide() {...}
    })
    
    return app
  }
}
```

在 `runtime-dom` 的 `createApp` 中，重写了 `mount` 方法，在其中调用了 `app.mount` 方法。

#### mount

```typescript
/**
   * 组件挂载
   * @param rootContainer 
   * @param isHydrate 
   */
mount(rootContainer: HostElement, isHydrate?: boolean): any {
    if (!isMounted) {
        // 创建 vnode 节点   
        const vnode = createVNode(
            rootComponent as ConcreteComponent,
            rootProps
        )
        // 节点的 vnode 挂载 context
        vnode.appContext = context
        // 忽略其他平台的执行，和开发环境的警告
        // ...
        // 执行 render 函数
        render(vnode, rootContainer)
        isMounted = true
        app._container = rootContainer
        // 返回 ???
        return vnode.component!.proxy
    }
}
```

在 `mount` 方法中，执行了 `createVNode` 方法创建了一个 组件的 `vnode`，然后执行了 `render` 方法。

### createVNode

创建 `vnode` 节点

```typescript
export const createVNode = (__DEV__
  ? createVNodeWithArgsTransform
  : _createVNode) as typeof _createVNode
```

不考虑开发环境的情况，直接看 `_createVNode` 方法

### _createVNode

```typescript
/**
 * 创建 vnode 的方法
 * @param type 
 * @param props 
 * @param children 
 * @param patchFlag 
 * @param dynamicProps 
 * @param isBlockNode 
 */
function _createVNode(
  type: VNodeTypes | ClassComponent | typeof NULL_DYNAMIC_COMPONENT,
  props: (Data & VNodeProps) | null = null,
  children: unknown = null,
  patchFlag: number = 0,
  dynamicProps: string[] | null = null,
  isBlockNode = false
): VNode {
  if (isVNode(type)) {
    // 如果 type 已经是一个 vnode，返回 clone 的 vnode
    const cloned = cloneVNode(type, props, true /* mergeRef: true */)
    if (children) {
      normalizeChildren(cloned, children)
    }
    return cloned
  }
  // 处理 class 组件，vue3 中已经取消了 class 组件
  // ...
  // class & style normalization.
  // 处理 props class 转成 字符串, style 转成 对象
  
  // encode the vnode type information into a bitmap
  const shapeFlag = isString(type)
    ? ShapeFlags.ELEMENT
    : __FEATURE_SUSPENSE__ && isSuspense(type)
      ? ShapeFlags.SUSPENSE
      : isTeleport(type)
        ? ShapeFlags.TELEPORT
        : isObject(type)
          ? ShapeFlags.STATEFUL_COMPONENT
          : isFunction(type)
            ? ShapeFlags.FUNCTIONAL_COMPONENT
            : 0
  // 新的 vnode 节点
  const vnode: VNode = {
    __v_isVNode: true,
    [ReactiveFlags.SKIP]: true,
    type,
    props,
    key: props && normalizeKey(props),
    ref: props && normalizeRef(props),
    scopeId: currentScopeId,
    children: null,
    component: null,
    suspense: null,
    ssContent: null,
    ssFallback: null,
    dirs: null,
    transition: null,
    el: null,
    anchor: null,
    target: null,
    targetAnchor: null,
    staticCount: 0,
    shapeFlag,
    patchFlag,
    dynamicProps,
    dynamicChildren: null,
    appContext: null
  }

  // validate key 校验 key 不是 NaN
  // ...
  // 处理子节点
  normalizeChildren(vnode, children)
  if (__FEATURE_SUSPENSE__ && shapeFlag & ShapeFlags.SUSPENSE) {
    const { content, fallback } = normalizeSuspenseChildren(vnode)
    vnode.ssContent = content
    vnode.ssFallback = fallback
  }

  if (
    shouldTrack > 0 
    !isBlockNode &&
    currentBlock &&
    (patchFlag > 0 || shapeFlag & ShapeFlags.COMPONENT) &&
    patchFlag !== PatchFlags.HYDRATE_EVENTS
  ) {
    currentBlock.push(vnode)
  }
  return vnode
}
```

这个方法返回了一个新的 `vnode`，即使传入的参数已经是一个 `vnode` 节点，也会 clone 一份新的 `vnode` 节点，并返回。

然后看在 `mount` 中调用的另一个方法 `render`

### render

`mount`中调用的这个 `render`  方法是在调用 `createAppAPI` 方法 的时候，传入的参数。也就是 `baseCreateRenderer` 方法中定义的 `render` 方法

```typescript
const render: RootRenderFunction = (vnode, container) => {
    // vnode === null 卸载组件
    if (vnode == null) {
        if (container._vnode) {
            // 卸载组件需要执行
            unmount(container._vnode, null, null, true)
        }
    } else {
        // patch 更新和或者创建组件
        patch(container._vnode || null, vnode, container)
    }
    flushPostFlushCbs()
    // 保存组件的 vnode
    container._vnode = vnode
}
```

`render` 方法中通过 `patch` 方法，将 `vnode` 转化成真实的 `DOM`，并挂载在页面上。

### patch

```typescript
  const patch: PatchFn = (
    n1,
    n2,
    container,
    anchor = null,
    parentComponent = null,
    parentSuspense = null,
    isSVG = false,
    optimized = false
  ) => {
    // 新旧节点类型不同的时候，直接删除旧的节点
    if (n1 && !isSameVNodeType(n1, n2)) {
      anchor = getNextHostNode(n1)
      unmount(n1, parentComponent, parentSuspense, true)
      n1 = null
    }
    const { type, ref, shapeFlag } = n2
    switch (type) {
      case Text:
        // ...
        break
      case Comment:
        // ...
        break
      case Static:
        // ...
        break
      case Fragment:
        // ...
        break
      default:
        // 处理其他类型的节点
        // ...
        if (shapeFlag & ShapeFlags.COMPONENT) {
          processComponent( // 处理component
            n1,
            n2,
            container,
            anchor,
            parentComponent,
            parentSuspense,
            isSVG,
            optimized
          )
        }
    }
    // 处理 绑定的 ref
    if (ref != null && parentComponent) {
      setRef(ref, n1 && n1.ref, parentSuspense, n2)
    }
  }
```

`patch` 过程中，会根据 `vnode` 的 `type` 不同，调用不同的处理节点的方法，这里主要看处理 `component` 的方法 `processComponent`，因为这里会执行 `setup` 和收集依赖。

#### processComponent

```typescript
const processComponent = (
    n1: VNode | null,
    n2: VNode,
    container: RendererElement,
    anchor: RendererNode | null,
    parentComponent: ComponentInternalInstance | null,
    parentSuspense: SuspenseBoundary | null,
    isSVG: boolean,
    optimized: boolean
  ) => {
    if (n1 == null) {
        // ...
      	// 挂载 component
        mountComponent(
          n2,
          container,
          anchor,
          parentComponent,
          parentSuspense,
          isSVG,
          optimized
        )
    } else {
      // 更新组件
      updateComponent(n1, n2, optimized)
    }
  }
```

可以看出创建组件使用了 `mountComponent` 方法，更新组件使用了 `updateComponent`  方法，先看 `mountComponent`，再看 `updateComponent`。

#### mountComponent

```typescript
const mountComponent: MountComponentFn = (
    initialVNode,
    container,
    anchor,
    parentComponent,
    parentSuspense,
    isSVG,
    optimized
  ) => {
    // createComponentInstance 会对 instance 做处理, ctx 的不一致，就是在这个方法处理的
    const instance: ComponentInternalInstance = (initialVNode.component = createComponentInstance(
      initialVNode,
      parentComponent,
      parentSuspense
    ))
	// ...
    // 这里调用 setup 方法，setup 返回的值，会保存在 instance.setupState 中
    setupComponent(instance)
    // ...
    if (__FEATURE_SUSPENSE__ && instance.asyncDep) {
      // 处理 setup 是 promise 的情况，在 promise 的状态 resolve 之后，才执行 setupRenderEffect 函数
      parentSuspense && parentSuspense.registerDep(instance, setupRenderEffect)
	  // ...
      return
    }

    setupRenderEffect(
      instance,
      initialVNode,
      container,
      anchor,
      parentSuspense,
      isSVG,
      optimized
    )
  }
```

这个方法里通过 `createComponentInstance` 生成了 `instance` 

这个 `instance` 是通过 `getCurrentInstance` 获取到的 `instance` ，`instance` 的 `ctx` 属性，在 `dev` 环境和 `prod` 是两个东西，**生产环境中不要使用**

函数最后执行了 `setupRenderEffect` 方法，在这个方法的执行过程中，会收集 `vnode` 中使用到的依赖

##### createComponentInstance

```typescript
export function createComponentInstance(
  vnode: VNode,
  parent: ComponentInternalInstance | null,
  suspense: SuspenseBoundary | null
) {
  // ...
  const instance: ComponentInternalInstance = {
    // ...
  }
  // 开发环境对 ctx 做的特殊处理
  // 项目开发中不能使用这个 ctx，生产环境不支持
  if (__DEV__) {
    instance.ctx = createRenderContext(instance)
  } else {
    instance.ctx = { _: instance }
  }
  instance.root = parent ? parent.root : instance
  instance.emit = emit.bind(null, instance)

  return instance
}
```

#### setupRenderEffect

```typescript
  const setupRenderEffect: SetupRenderEffectFn = (
    instance,
    initialVNode,
    container,
    anchor,
    parentSuspense,
    isSVG,
    optimized
  ) => {
    instance.update = effect(function componentEffect() {
      // 新建组件
      if (!instance.isMounted) {
        let vnodeHook: VNodeHook | null | undefined
        const { el, props } = initialVNode
        // 对子节点处理  这里会执行 组件的 render 函数
        // render 函数对 ref / reactive 的值的获取，都会把当前函数作为依赖变更需要触发的函数收集
        const subTree = (instance.subTree = renderComponentRoot(instance))
        patch( // 这个patch执行会完成DOM的挂载
            null,
            subTree,
            container,
            anchor,
            instance,
            parentSuspense,
            isSVG
          )
          initialVNode.el = subTree.el
        instance.isMounted = true
        initialVNode = container = anchor = null as any
      } else {
        // 更新组件
        let { next, bu, u, parent, vnode } = instance
        let originNext = next
        let vnodeHook: VNodeHook | null | undefined
        const nextTree = renderComponentRoot(instance)
        const prevTree = instance.subTree
        instance.subTree = nextTree
        patch(
          prevTree,
          nextTree,
          // parent may have changed if it's in a teleport
          hostParentNode(prevTree.el!)!,
          // anchor may have changed if it's in a fragment
          getNextHostNode(prevTree),
          instance,
          parentSuspense,
          isSVG
        )
        next.el = nextTree.el
      }
    }, __DEV__ ? createDevEffectOptions(instance) : prodEffectOptions)
  }
```

`setupRenderEffect` 函数执行会调用 `effect` 函数，只有在 `effect` 中执行的函数，才可以做依赖收集，通过 `renderComponentRoot` 方法创建组件的子节点，这个方法执行了组件的 `render` 方法，`render`方法中对 `reactive` 类型的值的获取和 `ref`  / `computed` 类型的 `.value` 的获取，都会把这个 `effect` 函数作为变更的依赖做收集。

在执行 `effect` 的时候，传递了第二个参数 `prodEffectOptions`，这个参数中，有一个 `scheduler` 方法，这个是依赖更新之后会调用的调度器，这个调度器决定什么时候执行 `DOM` 更新，而不是每次依赖变化都对 `DOM` 做修改。

在这个方法也会执行 `beforeMount` 的 hooks 函数，之后执行的 `renderComponentRoot`  结束之后，再次执行一个 `patch` 方法，这个方法中，完成了组件创建到 `DOM` 的动作。并且对组件模板中绑定的 `ref` 做处理。通过 `setRef` 方法。

在之后执行了 `mounted` 的 hooks 函数。

组件的渲染就结束了！！！

