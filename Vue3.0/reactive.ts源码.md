> 源码位置： [https://github.com/vuejs/vue-next/blob/master/packages/reactivity/src/reactive.ts](https://github.com/vuejs/vue-next/blob/master/packages/reactivity/src/reactive.ts)

## reactive 方法



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



## shallowReactive 方法

## readonly 方法

## shallowReadonly 方法、

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





