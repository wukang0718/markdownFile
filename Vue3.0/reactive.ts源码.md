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

#### 

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





