> 源码位置： [https://github.com/vuejs/vue-next/blob/master/packages/reactivity/src/reactive.ts](https://github.com/vuejs/vue-next/blob/master/packages/reactivity/src/reactive.ts)

## reactive 方法

接收一个对象，如果对象说 `readonly` 类型的，就返回对象本身，否则返回 对象深代理，通过 `createReactiveObject` 方法

```typescript
export function reactive(target: object) {
  if (target && (target as Target)[ReactiveFlags.IS_READONLY]) {
    return target
  }
  return createReactiveObject(
    target,
    false,
    mutableHandlers,
    mutableCollectionHandlers
  )
}
```

#### mutableHandlers 

在 `target` 是一个正常的 `Object` 或者 `Array` 时，使用这个对象做 `proxy` 的 `handle`



## shallowReactive 方法

参数接收一个对象，返回对象的浅代理，通过 `createReactiveObject` 方法

```typescript
export function shallowReactive<T extends object>(target: T): T {
  return createReactiveObject(
    target,
    false,
    shallowReactiveHandlers,
    shallowCollectionHandlers
  )
}
```

## readonly 方法

参数接收一个对象，返回一个只读 `readonly` 对象，通过 `createReactiveObject` 方法

```typescript
export function readonly<T extends object>(
  target: T
): DeepReadonly<UnwrapNestedRefs<T>> {
  return createReactiveObject(
    target,
    true,
    readonlyHandlers,
    readonlyCollectionHandlers
  )
}
```

## shallowReadonly 方法

参数接收一个对象，返回一个只读 `readonly` 的对象，这个只读，只针对对象的第一层，不做深层的代理， 通过 `createReactiveObject` 方法

```typescript
export function shallowReadonly<T extends object>(
  target: T
): Readonly<{ [K in keyof T]: UnwrapNestedRefs<T[K]> }> {
  return createReactiveObject(
    target,
    true,
    shallowReadonlyHandlers,
    readonlyCollectionHandlers
  )
}
```

#### createReactiveObject 方法

参数

| 参数               | 描述                                                         |
| ------------------ | ------------------------------------------------------------ |
| target             | 需要代理的源数据（一般是对象，可以是Map/Set/WeakMap/WeakSet） |
| isReadonly         | 是否创建只读对象                                             |
| baseHandlers       | target 是 Object/Array 时，proxy 的 handle 对象              |
| collectionHandlers | target 是 Map/Set/WeakMap/WeakSet时，proxy 的 handle 对象    |

返回一个 `proxy` 对象，会对返回的 `proxy` 和 `target` 做一个缓存，如果对同一个对象多次调用该方法，获取到的是同一个对象

```typescript
function createReactiveObject(
  target: Target,
  isReadonly: boolean,
  baseHandlers: ProxyHandler<any>,
  collectionHandlers: ProxyHandler<any>
) {
  // 如果不是对象类型，就直接返回 参数本身
  if (!isObject(target)) {
    if (__DEV__) {
      console.warn(`value cannot be made reactive: ${String(target)}`)
    }
    return target
  }
  // 如果target是只读对象，并且要获取只读对象的时候，就直接返回 target
  if (
    target[ReactiveFlags.RAW] &&
    !(isReadonly && target[ReactiveFlags.IS_REACTIVE])
  ) {
    return target
  }
  // 如果 target 已经有了对象的代理对象，直接返回这个代理
  const proxyMap = isReadonly ? readonlyMap : reactiveMap
  const existingProxy = proxyMap.get(target)
  if (existingProxy) {
    return existingProxy
  }
  /**
   * targetType 指定要跳过或者不可扩展，或者不是Object/Array/Map/WeakMap/Set/WeakSet TargetType.INVALID
   * Object/Array TargetType.COMMON
   * Map/Set/WeakMap/WeakSet TargetType.COLLECTION
   */
  const targetType = getTargetType(target)
  if (targetType === TargetType.INVALID) {
    return target
  }
  const proxy = new Proxy(
    target,
    targetType === TargetType.COLLECTION ? collectionHandlers : baseHandlers
  )
  // 保存一下这个代理对象
  proxyMap.set(target, proxy)
  return proxy
}
```



## isReactive 方法

## isReadonly 方法

## isProxy 方法

## toRaw 方法

> 源码位置： [https://github.com/vuejs/vue-next/blob/master/packages/reactivity/src/reactive.ts](https://github.com/vuejs/vue-next/blob/master/packages/reactivity/src/reactive.ts)

方法接收一个参数，如果参数是 `reactive` 对象会返回 `reactive` 对象的原始值，如果不是 `reactive` 对象，就返回接收的参数

`ReactiveFlags.RAW` 在 `reactive` 对象的 `get` 方法中做了处理 

```typescript
export function toRaw<T>(observed: T): T {
  return (
    (observed && toRaw((observed as Target)[ReactiveFlags.RAW])) || observed
  )
}
```

## markRaw 方法





