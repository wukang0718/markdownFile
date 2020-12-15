## reactive 方法

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

## track 方法

收集依赖

```typescript
export function track(target: object, type: TrackOpTypes, key: unknown) {
  if (!shouldTrack || activeEffect === undefined) {
    return
  }
  let depsMap = targetMap.get(target)
  if (!depsMap) {
    targetMap.set(target, (depsMap = new Map()))
  }
  let dep = depsMap.get(key)
  if (!dep) {
    depsMap.set(key, (dep = new Set()))
  }
  if (!dep.has(activeEffect)) {
    dep.add(activeEffect)
    activeEffect.deps.push(dep)
    // 开发环境才会执行，方便追踪依赖调试，忽略
    if (__DEV__ && activeEffect.options.onTrack) {
      activeEffect.options.onTrack({
        effect: activeEffect,
        target,
        type,
        key
      })
    }
  }
}
```





