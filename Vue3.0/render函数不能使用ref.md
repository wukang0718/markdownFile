

## 一次在 Vue3 中使用render函数的事故，从源码中找到了答案

第一遍看 `vue3` 的源码，有不对的地方，希望各位大佬能指出，感谢！！！

**Vue3 版本 3.0.4**

在 `Vue3.0` 创建组件时，在 `render` 函数中使用 `setup` 函数中返回的 `ref` 数据，无法动态绑定

示例：

```js
Vue.createApp({
    setup() {
        const elRef = Vue.ref(null)
        Vue.onMounted(() => {
        	elRef.value.innerHTML = "123"
        })
        return {
        	elRef
        }
    },
    render() {
        return Vue.h("div", {
            ref: this.elRef
        })
    }
}).mount("#app")
```

浏览器报错

![image-20201209111428308](https://gitee.com/wu_kang0718/image/raw/master//20201209111428973.png)

### 查找问题

### 1、找到 `render` 函数的执行

源码位置：[https://github.com/vuejs/vue-next/blob/master/packages/runtime-core/src/componentRenderUtils.ts](https://github.com/vuejs/vue-next/blob/master/packages/runtime-core/src/componentRenderUtils.ts) 77行

删减后的代码：

```js
export function renderComponentRoot(
  instance: ComponentInternalInstance
): VNode {
  const {
    proxy,
    withProxy,
    props,
    render,
    renderCache,
    data,
    setupState,
    ctx
  } = instance

  let result
  try {
    let fallthroughAttrs
    if (vnode.shapeFlag & ShapeFlags.STATEFUL_COMPONENT) {
	  const proxyToUse = withProxy || proxy
      result = normalizeVNode(
        render!.call(
          proxyToUse,
          proxyToUse!,
          renderCache,
          props,
          setupState,
          data,
          ctx
        )
      )
      
      fallthroughAttrs = attrs
    } else {
      // 函数组件
    }
  return result
}
```

`render` 函数在执行的时候，将 `this` 改变成了 `proxy`，（在生产环境是通过在 `with` 块中执行改变）

### 2、找到 `proxy` 的定义

源码位置 [https://github.com/vuejs/vue-next/blob/master/packages/runtime-core/src/component.ts](https://github.com/vuejs/vue-next/blob/master/packages/runtime-core/src/component.ts) 565行

删减后的代码：

```js
function setupStatefulComponent(
  instance: ComponentInternalInstance,
  isSSR: boolean
) {
  const Component = instance.type as ComponentOptions // type 就是组件的options
  instance.accessCache = Object.create(null)
  // proxy 在这里定义 
  instance.proxy = new Proxy(instance.ctx, PublicInstanceProxyHandlers)
  // setup 方法在这里执行
  const { setup } = Component
  if (setup) {
    const setupContext = (instance.setupContext =
      setup.length > 1 ? createSetupContext(instance) : null)

    currentInstance = instance
    pauseTracking()
    const setupResult = callWithErrorHandling( // 这个方法内部只是使用 try catch 捕捉 setup 函数时的异常
      setup,
      instance,
      ErrorCodes.SETUP_FUNCTION,
      [__DEV__ ? shallowReadonly(instance.props) : instance.props, setupContext]
    )
    handleSetupResult(instance, setupResult, isSSR) // 这里处理 setup 的返回值
  }
}
```

`PublicInstanceProxyHandlers` 对 `get` 方法的处理

源码位置：[https://github.com/vuejs/vue-next/blob/master/packages/runtime-core/src/componentPublicInstance.ts](https://github.com/vuejs/vue-next/blob/master/packages/runtime-core/src/componentPublicInstance.ts) 143行

删减后的代码：

```js
export const PublicInstanceProxyHandlers: ProxyHandler<any> = {
  get({ _: instance }: ComponentRenderContext, key: string) {
    const {
      ctx,
      setupState,
      data,
      props,
      accessCache,
      type,
      appContext
    } = instance
    let normalizedProps
    if (key[0] !== '$') {
      // 如果缓存里有 key 对应的数据类型，就不需要在判断 key 的来源
      const n = accessCache![key]
      if (n !== undefined) {
        switch (n) {
          case AccessTypes.SETUP:
            return setupState[key]
          case AccessTypes.DATA:
            return data[key]
          case AccessTypes.CONTEXT:
            return ctx[key]
          case AccessTypes.PROPS:
            return props![key]
        }
      } else if (setupState !== EMPTY_OBJ && hasOwn(setupState, key)) {
        // 缓存中没有 key 的数据来源，把 key 的数据来源存入到缓存中，返回数据
        accessCache![key] = AccessTypes.SETUP
        return setupState[key]
      }
    }
  }
}
```

这里返回的 `setupState` 的值，`setupState` 也是一个 `proxy` 的类型，在 `setup` 执行完成后创建

源码位置：[https://github.com/vuejs/vue-next/blob/master/packages/runtime-core/src/component.ts](https://github.com/vuejs/vue-next/blob/master/packages/runtime-core/src/component.ts) 610行

删减后的代码：

```js
export function handleSetupResult(
  instance: ComponentInternalInstance,
  setupResult: unknown,
  isSSR: boolean
) {
  if (isFunction(setupResult)) {
    // setup 中返回一个函数，这个函数作为render函数使用
    instance.render = setupResult as InternalRenderFunction
  } else if (isObject(setupResult)) {
    // 这里定义 setupState
    instance.setupState = proxyRefs(setupResult) // proxyRefs 函数返回一个 proxy 对象
  }
  finishComponentSetup(instance, isSSR)
}
```

`proxyRefs` 函数 返回一个 `proxy` 对象，这个 `proxy` 对象的 `get` 方法，会返回 `ref` 数据的 `value` 值

源码位置： [https://github.com/vuejs/vue-next/blob/master/packages/reactivity/src/ref.ts](https://github.com/vuejs/vue-next/blob/master/packages/reactivity/src/ref.ts) 105行

删减后的代码：

```tsx
export function unref<T>(ref: T): T extends Ref<infer V> ? V : T {
  // 会返回 ref 数据的 value 属性，而不是整个对象
  return isRef(ref) ? (ref.value as any) : ref
}

const shallowUnwrapHandlers: ProxyHandler<any> = {
  get: (target, key, receiver) => unref(Reflect.get(target, key, receiver)),
}

export function proxyRefs<T extends object>(
  objectWithRefs: T
): ShallowUnwrapRef<T> {
  return isReactive(objectWithRefs)
    ? objectWithRefs
    : new Proxy(objectWithRefs, shallowUnwrapHandlers)
}
```

到这里，就可以知道 为什么在 `render` 函数中使用 `this.` 的方法获取到的不是 `ref` 对象，而是原始值了

### 原因总结：

在 `render` 方法中使用 `this.` 的方式获取的是 `setup` 中返回的 `ref` 数据的 `value` 属性，所以不能给元素做动态绑定

### 怎么解决：

> 在上面提到的 `setup` 方法执行完成后，会执行 `handleSetupResult` 方法，处理 `setup` 函数返回的数据，再看一遍

删减后的代码：

```js
export function handleSetupResult(
  instance: ComponentInternalInstance,
  setupResult: unknown,
  isSSR: boolean
) {
  if (isFunction(setupResult)) {
    // setup 中返回一个函数，这个函数作为render函数使用
    instance.render = setupResult as InternalRenderFunction
  } else if (isObject(setupResult)) {
    // 这里定义 setupState
    instance.setupState = proxyRefs(setupResult) // proxyRefs 函数返回一个 proxy 对象
  }
  finishComponentSetup(instance, isSSR)
}
```

在 `setup` 的函数返回值是一个函数的时候，这个函数会被作为 `render` 函数处理（这个返回值会覆盖在 `opitons` 中的 `render` 函数）

所以我们之前的示例就可以改成下面这个样子：

```js
Vue.createApp({
    setup() {
        const elRef = Vue.ref(null)
        Vue.onMounted(() => {
        	elRef.value.innerHTML = "123"
        })
        return () =>  Vue.h("div", {
        	ref: elRef
        })
    }
}).mount("#app")
```

![image-20201209105641740](https://gitee.com/wu_kang0718/image/raw/master//20201209105642409.png)

页面中就可以正确的显示内容了

#### 总结：

`setup` 中可以返回一个 对象 或者 函数，当返回 函数是，这个函数就是这个组件的 `render` 函数

当返回 对象的时候，这个对象的数据通过 `this` 获取值的时候，只能获取到 `.value` 的值，获取不到 `ref` 对象