> 源码位置： [https://github.com/vuejs/vue-next/blob/master/packages/reactivity/src/reactive.ts](https://github.com/vuejs/vue-next/blob/master/packages/reactivity/src/reactive.ts)

## reactive 方法

接收一个对象，返回 对象深代理，通过 `createReactiveObject` 方法

- 对象的 `reactive` 代理，会被缓存，所以对同一个对象执行多次 `reactive` 返回的是同一个值

示例：

```typescript
setup() {
    const a = {b: 12}
    const reactive1 = Vue.reactive(a)
    const reactive2 = Vue.reactive(a)
    console.log(reactive1 === reactive2) // true
}
```

- 如果对象是 `readonly` 类型的，就返回对象本身

示例：

```typescript
setup() {
    const a = {b: 12}
    const readonly = Vue.readonly(a)
    const reactive = Vue.reactive(readonly)
    console.log(readonly === reactive) // true
}
```



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

### mutableHandlers 

在 `target` 是一个正常的 `Object` 或者 `Array` 时，使用这个对象做 `proxy` 的 `handle`

```typescript
const get = /*#__PURE__*/ createGetter() // 在下面
const set = /*#__PURE__*/ createSetter() // 在下面

// delete / Reflect.deleteProperty() 操作符的时候调用
function deleteProperty(target: object, key: string | symbol): boolean {
  const hadKey = hasOwn(target, key) // Object.prototype.hasOwnProperty
  const oldValue = (target as any)[key]
  const result = Reflect.deleteProperty(target, key)
  if (result && hadKey) {
    // 如果删除成功，触发依赖更新 
    trigger(target, TriggerOpTypes.DELETE, key, undefined, oldValue)
  }
  return result
}
// in / Reflect.has() 操作符调用
function has(target: object, key: string | symbol): boolean {
  const result = Reflect.has(target, key)
  if (!isSymbol(key) || !builtInSymbols.has(key)) {
    // key 和 value 不是 symbol 类型 收集依赖
    track(target, TrackOpTypes.HAS, key)
  }
  return result
}
/** 拦截下面操作
	Object.getOwnPropertyNames()
    Object.getOwnPropertySymbols()
    Object.keys()
    Reflect.ownKeys()
 */
function ownKeys(target: object): (string | number | symbol)[] {
  track(target, TrackOpTypes.ITERATE, isArray(target) ? 'length' : ITERATE_KEY)
  return Reflect.ownKeys(target)
}

export const mutableHandlers: ProxyHandler<object> = {
  get,
  set,
  deleteProperty,
  has,
  ownKeys
}
```

#### createGetter

参数接收 `isReadonly` 是否只读， `shallow` 是否只做前代理， 返回一个 `get` 函数

当返回值时一个对象，并且 `shallow`  为 false 时，会对返回值再做一次 `readonly` 或者 `reactive` 处理，实现深代理

```typescript
/**
 * 返回一个 get 函数
 * @param isReadonly 只读
 * @param shallow 浅代理
 */
function createGetter(isReadonly = false, shallow = false) {
  return function get(target: Target, key: string | symbol, receiver: object) {
    if (key === ReactiveFlags.IS_REACTIVE) {
      // 这个key __v_isReactive 表示对象是不是 reactive 类型
      return !isReadonly
    } else if (key === ReactiveFlags.IS_READONLY) {
      // 这个key __v_isReadonly 表示对象是不是只读类型
      return isReadonly
    } else if (
      key === ReactiveFlags.RAW &&
      receiver === (isReadonly ? readonlyMap : reactiveMap).get(target)
    ) {
      // 这个keym __v_raw 获取 代理的源对象
      return target
    }
    // 上面的三个 特殊值，在 is 开头的函数 和 to 开头的函数中用到
    // 例：isReactive / toRaw

    const targetIsArray = isArray(target)

    // 不是只读，是数组，并且调用了数组的方法，需要特殊的方法处理
    if (!isReadonly && targetIsArray && hasOwn(arrayInstrumentations, key)) {
      return Reflect.get(arrayInstrumentations, key, receiver)
    }

    const res = Reflect.get(target, key, receiver)
    
    // key 是 symbol 类型，并且值也是 symbol 或者 获取的是  __proto__ 或者 __v_isRef 时
    // 直接返回结果
    if (
      isSymbol(key)
        ? builtInSymbols.has(key as symbol)
        : key === `__proto__` || key === `__v_isRef`
    ) {
      return res
    }

    // 不是只读的数据，收集依赖
    if (!isReadonly) {
      track(target, TrackOpTypes.GET, key)
    }
    
    // 浅代理，返回结果
    if (shallow) {
      return res
    }

    if (isRef(res)) {
      // ref展开-不适用于数组 和 整数键。
      const shouldUnwrap = !targetIsArray || !isIntegerKey(key)
      // 这里对 ref 做了 返回原始值的处理
      return shouldUnwrap ? res.value : res
    }
    
    // 如果结果时对象，对这个对象在做一次 reactive 处理，实现深代理
    if (isObject(res)) {
      // Convert returned value into a proxy as well. we do the isObject check
      // here to avoid invalid value warning. Also need to lazy access readonly
      // and reactive here to avoid circular dependency.
      return isReadonly ? readonly(res) : reactive(res)
    }

    return res
  }
}
```

这里用到了一个特殊的值 `arrayInstrumentations`

这个值表示需要特殊处理的 `Array` 的方法

```typescript
/**
 * 数组的一些方法需要做特殊的处理
 */
const arrayInstrumentations: Record<string, Function> = {}
;(['includes', 'indexOf', 'lastIndexOf'] as const).forEach(key => {
  const method = Array.prototype[key] as any
  arrayInstrumentations[key] = function(this: unknown[], ...args: unknown[]) {
    const arr = toRaw(this) // 获取到数组的原值
    for (let i = 0, l = this.length; i < l; i++) {
      // 循环对数组中的每一项添加依赖
      track(arr, TrackOpTypes.GET, i + '')
    }
    // 先不对参数做处理，直接运行方法
    const res = method.apply(arr, args)
    if (res === -1 || res === false) {
      // 如果方法没有期望的返回值，在对参数做一次 toRaw（获取原始值） 转换
      return method.apply(arr, args.map(toRaw))
    } else {
      return res
    }
  }
})
// 这些方法会改变 Array 的length 属性，在某种情况下，会导致依赖循环触发 #2137
;(['push', 'pop', 'shift', 'unshift', 'splice'] as const).forEach(key => {
  const method = Array.prototype[key] as any
  arrayInstrumentations[key] = function(this: unknown[], ...args: unknown[]) {
    pauseTracking() // 停止依赖收集
    const res = method.apply(this, args) // 获取到处理结果
    resetTracking() // 恢复依赖收集
    return res
  }
})
```

#### createSetter

函数接收一个参数 `shallow` ，是否只做浅代理，返回一个 `set` 函数

```typescript
/**
 * 返回一个 set 方法
 * @param shallow 是否做浅代理操作
 */
function createSetter(shallow = false) {
  return function set(
    target: object,
    key: string | symbol,
    value: unknown,
    receiver: object
  ): boolean { 
    const oldValue = (target as any)[key]
    if (!shallow) {
      value = toRaw(value)
      // 深代理模式下，如果数组中的每一项都是 ref 对象，修改只修改该项的 value 值，不替换整个对象
      if (!isArray(target) && isRef(oldValue) && !isRef(value)) {
        oldValue.value = value
        return true
      }
    } else {
      // in shallow mode, objects are set as-is regardless of reactive or not
    }

    // 判断 key 是数组的下标或者 对象/数组的属性
    const hadKey =
      isArray(target) && isIntegerKey(key)
        ? Number(key) < target.length
        : hasOwn(target, key)
      

    const result = Reflect.set(target, key, value, receiver)
    // don't trigger if target is something up in the prototype chain of original
    // 不能直接触发依赖更新，有可能是原型链上的proxy被调用
    // 要判断 receiver 的原始值，就是当前的target对象， 这样确保是当前的 target 的 proxy 被触发
    if (target === toRaw(receiver)) {
      //
      if (!hadKey) {
        // 没有 key 的情况是添加属性，或者 数组添加一项
        trigger(target, TriggerOpTypes.ADD, key, value)
      } else if (hasChanged(value, oldValue)) {
        // 有 key 并且 新旧值不一样，修改成功，需要触发 set 的依赖
        trigger(target, TriggerOpTypes.SET, key, value, oldValue)
      }
    }
    return result
  }
}
```

#### mutableCollectionHandlers

```typescript
export const mutableCollectionHandlers: ProxyHandler<CollectionTypes> = {
  get: createInstrumentationGetter(false, false)
}
```

### createInstrumentationGetter

接收两个参数 `isReadonly` 是否只读和 `shallow` 是否浅代理

返回一个 `get` 方法，这里是对 `Map.` 的代理，`get` 方法中对 `set`/`has`...方法做了处理，通过 `instrumentations` 对象

```typescript
/**
 * 返回一个 Collection 的get 函数，在 get 中，对 set/has... 方法做处理
 * @param isReadonly 是否只读
 * @param shallow 是否浅代理
 */
function createInstrumentationGetter(isReadonly: boolean, shallow: boolean) {
  const instrumentations = shallow
    ? shallowInstrumentations
    : isReadonly
      ? readonlyInstrumentations
      : mutableInstrumentations

  return (
    target: CollectionTypes,
    key: string | symbol,
    receiver: CollectionTypes
  ) => {
    if (key === ReactiveFlags.IS_REACTIVE) {
      return !isReadonly
    } else if (key === ReactiveFlags.IS_READONLY) {
      return isReadonly
    } else if (key === ReactiveFlags.RAW) {
      return target
    }
    // 处理三个特殊 key 在 is... 和 to... 的方法中用到

    // 如果是 instrumentations 中有的方法，并且是 target 中的原始存在的方法，就使用代理，否则直接放回对象本身的值
    return Reflect.get(
      hasOwn(instrumentations, key) && key in target
        ? instrumentations
        : target,
      key,
      receiver
    )
  }
}
```

#### instrumentations  => mutableInstrumentations

返回了一个 `handle` 的对象，对象方法中的 `this`，都是指向 `reactive` 对象 

```typescript
const mutableInstrumentations: Record<string, Function> = {
  get(this: MapTypes, key: unknown) {
    return get(this, key)
  },
  get size() {
    return size((this as unknown) as IterableCollections)
  },
  has,
  add,
  set,
  delete: deleteEntry,
  clear,
  forEach: createForEach(false, false)
}
const iteratorMethods = ['keys', 'values', 'entries', Symbol.iterator]
iteratorMethods.forEach(method => {
  mutableInstrumentations[method as string] = createIterableMethod(
    method,
    false,
    false
  )
})
```

##### get

代理 `get` 方法

```typescript
/**
 * 获取值的方法
 * @param proxy 对象 reactive 对象
 * @param key 获取的key
 * @param isReadonly 是否只读
 * @param isShallow 是否浅代理
 */
function get(
  target: MapTypes,
  key: unknown,
  isReadonly = false,
  isShallow = false
) {
  target = (target as any)[ReactiveFlags.RAW] // 获取到原始值 （可能是 reactive 对象） #1772
  const rawTarget = toRaw(target) // 获取到最终的原始值
  const rawKey = toRaw(key) // 获取 key 的原始值
  if (key !== rawKey) {
    // 如果 key 是 reactive 类型，且代理不是 readonly ，收集 key 的依赖
    !isReadonly && track(rawTarget, TrackOpTypes.GET, key)
  }
  // 再次收集一次依赖 ???
  !isReadonly && track(rawTarget, TrackOpTypes.GET, rawKey)
  const { has } = getProto(rawTarget)
  const wrap = isReadonly ? toReadonly : isShallow ? toShallow : toReactive
  // 先获取 key 不是 reactive 的情况
  if (has.call(rawTarget, key)) {
    return wrap(target.get(key))
  } else if (has.call(rawTarget, rawKey)) {
    return wrap(target.get(rawKey))
  }
}
```

###### toReactive

获取到最终要返回的值的方法，如果是对象/数组类型，返回一个 reactive 类型，否则返回原始值

```typescript
const toReactive = <T extends unknown>(value: T): T =>
  isObject(value) ? reactive(value) : value
```

##### size

拦截 `.size` 的获取

```typescript
/**
 * 拦截 .size 的获取
 * @param proxy 对象 reactive 对象
 * @param isReadonly 是否只读
 */
function size(target: IterableCollections, isReadonly = false) {
  target = (target as any)[ReactiveFlags.RAW]
  // 不是  只读 收集依赖
  !isReadonly && track(toRaw(target), TrackOpTypes.ITERATE, ITERATE_KEY)
  // 返回原始的 size 属性
  return Reflect.get(target, 'size', target)
}
```

##### has

拦截 `has` 操作

```typescript
/**
 * 拦截 has 操作
 * @param proxy 对象 reactive 对象
 * @param key 判断的key
 * @param isReadonly 是否只读
 */
function has(this: CollectionTypes, key: unknown, isReadonly = false): boolean {
  const target = (this as any)[ReactiveFlags.RAW] // 获取原始值，可能是一个 reactive 对象
  const rawTarget = toRaw(target) // 获取到最终的原始值
  const rawKey = toRaw(key) // 获取到 key 的原始值
  if (key !== rawKey) {
    // key 是 reactive 类型时，收集 key 的依赖
    !isReadonly && track(rawTarget, TrackOpTypes.HAS, key)
  }
  // 收集 rawKey 的依赖
  !isReadonly && track(rawTarget, TrackOpTypes.HAS, rawKey)
  // 先 判断 有没有 reactive 的 key 的值，然后判断 原始值的 key 的值
  return key === rawKey
    ? target.has(key)
    : target.has(key) || target.has(rawKey)
}
```

##### add

拦截 `Set`/`WeakSet` 的 add 操作

```typescript
/**
 * 拦截 Set/WeakSet 的 add 操作
 * @param proxy 对象 reactive 对象
 * @param value 添加的数据
 */
function add(this: SetTypes, value: unknown) {
  value = toRaw(value) // 获取到 value 的原始值
  const target = toRaw(this) // 获取 target 的原始值
  const proto = getProto(target) //  获取原型
  const hadKey = proto.has.call(target, value)
  target.add(value) // 给 set 中添加原始值
  if (!hadKey) {
    // 如果 set 中之前没有这个值，触发一次依赖
    trigger(target, TriggerOpTypes.ADD, value, value)
  }
  return this
}
```

##### set

拦截 `Map` 和 `WeakMap` 的操作

```typescript
/**
 * 拦截 Map 和 WeakMap 的操作
 * @param proxy 对象 reactive 对象
 * @param key key
 * @param value value
 */
function set(this: MapTypes, key: unknown, value: unknown) {
  value = toRaw(value) // 获取到 value 的原始值
  const target = toRaw(this) // 获取到 target 的原始值
  const { has, get } = getProto(target)

  let hadKey = has.call(target, key) // boolean 元数据是否存在 key
  if (!hadKey) {
    key = toRaw(key) // key 的原始值
    hadKey = has.call(target, key) // 如果没有 reactive 的key ，尝试获取 原始类型的key 是否存在
  } else if (__DEV__) {
    // 开发环境做的操作，忽略
    checkIdentityKeys(target, has, key)
  }

  const oldValue = get.call(target, key)
  target.set(key, value)
  if (!hadKey) {
    // 触发一次 添加的依赖
    trigger(target, TriggerOpTypes.ADD, key, value)
  } else if (hasChanged(value, oldValue)) {
    // 触发一次修改的依赖
    trigger(target, TriggerOpTypes.SET, key, value, oldValue)
  }
  return this
}
```

##### delete

拦截 `delete` 操作

```typescript
/**
 * 拦截 delete 操作
 * @param proxy 对象 reactive 对象
 * @param key key
 */
function deleteEntry(this: CollectionTypes, key: unknown) {
  const target = toRaw(this)
  const { has, get } = getProto(target)
  let hadKey = has.call(target, key)
  if (!hadKey) {
    key = toRaw(key)
    hadKey = has.call(target, key)
  } else if (__DEV__) {
    checkIdentityKeys(target, has, key)
  }

  const oldValue = get ? get.call(target, key) : undefined
  const result = target.delete(key)
  if (hadKey) {
    // 如果删除的是之前存在的字段，触发一次 delete 依赖
    trigger(target, TriggerOpTypes.DELETE, key, undefined, oldValue)
  }
  return result
}
```



##### clear

拦截 `clear` 操作

```typescript
/**
 * 拦截 clear 操作
 * @param this proxy 对象 reactive 对象
 */
function clear(this: IterableCollections) {
  const target = toRaw(this)
  const hadItems = target.size !== 0
  const oldTarget = __DEV__
    ? isMap(target)
      ? new Map(target)
      : new Set(target)
    : undefined
  const result = target.clear()
  if (hadItems) {
    // 如果之前有元素，触发一次依赖事件
    trigger(target, TriggerOpTypes.CLEAR, undefined, undefined, oldTarget)
  }
  return result
}
```



##### forEach

拦截 `forEach` 操作

```typescript
/**
 * 拦截 forEach 操作
 * @param isReadonly 是否只读
 * @param isShallow 是否浅代理
 */
function createForEach(isReadonly: boolean, isShallow: boolean) {
  return function forEach(
    this: IterableCollections, // proxy 对象
    callback: Function, // 回调函数
    thisArg?: unknown // 改变的 this 的值
  ) {
    const observed = this as any
    const target = observed[ReactiveFlags.RAW]
    const rawTarget = toRaw(target) // 原始的值
    const wrap = isReadonly ? toReadonly : isShallow ? toShallow : toReactive
    // 不是只读，触发一次iterate依赖
    !isReadonly && track(rawTarget, TrackOpTypes.ITERATE, ITERATE_KEY)
    return target.forEach((value: unknown, key: unknown) => {
      // forEach callback 中接收到的值，是 reactive / readonly 类型
      return callback.call(thisArg, wrap(value), wrap(key), observed)
    })
  }
}
```



##### keys 、values、entries

###### createIterableMethod

返回一个方法 拦截 `iterate` 操作，返回一个遍历器，`for of` 遍历就是基于遍历器实现的

```typescript
/**
 * 返回一个方法 拦截 iterate 操作
 * @param method 方法名称 keys values entries
 * @param isReadonly 是否只读
 * @param isShallow 是否浅代理
 */
function createIterableMethod(
  method: string | symbol,
  isReadonly: boolean,
  isShallow: boolean
) {
  return function(
    this: IterableCollections, // this 是 proxy 对象
    ...args: unknown[]
  ): Iterable & Iterator {
    const target = (this as any)[ReactiveFlags.RAW]
    const rawTarget = toRaw(target)
    const targetIsMap = isMap(rawTarget)
    // 是否 map 遍历 iterate
    const isPair =
      method === 'entries' || (method === Symbol.iterator && targetIsMap)
    // 是否获取map的keys
    const isKeyOnly = method === 'keys' && targetIsMap
    const innerIterator = target[method](...args)
    const wrap = isReadonly ? toReadonly : isShallow ? toShallow : toReactive
    // 收集依赖
    !isReadonly &&
      track(
        rawTarget,
        TrackOpTypes.ITERATE,
        isKeyOnly ? MAP_KEY_ITERATE_KEY : ITERATE_KEY
      )
    // return a wrapped iterator which returns observed versions of the
    // values emitted from the real iterator
    // 返回的是一个next函数，next函数返回 value 和 done ，这是 iterable 接口规范
    // value 的值还是一个 reactive / readonly / ... 的值
    return {
      // iterator protocol
      next() {
        const { value, done } = innerIterator.next()
        return done
          ? { value, done }
          : {
              value: isPair ? [wrap(value[0]), wrap(value[1])] : wrap(value),
              done
            }
      },
      // iterable protocol
      [Symbol.iterator]() {
        return this
      }
    }
  }
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

### shallowReactiveHandlers

```typescript
export const shallowReactiveHandlers: ProxyHandler<object> = extend(
  {},
  mutableHandlers,
  {
    get: shallowGet,
    set: shallowSet
  }
)
```

用了新的 `get` 和 `set` 方法

```typescript
const shallowGet = /*#__PURE__*/ createGetter(false, true)
const shallowSet = /*#__PURE__*/ createSetter(true)
```

用了上面提到的 `createGetter` 和 `createSetter` 方法

### shallowCollectionHandlers

```typescript
export const shallowCollectionHandlers: ProxyHandler<CollectionTypes> = {
  get: createInstrumentationGetter(false, true)
}
```

用的还是上面提到的 `createInstrumentationGetter` 方法

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

### readonlyHandlers

```typescript
export const readonlyHandlers: ProxyHandler<object> = {
  get: readonlyGet,
  set(target, key) { // 不做任何改变，直接返回 true
    if (__DEV__) {
      console.warn(
        `Set operation on key "${String(key)}" failed: target is readonly.`,
        target
      )
    }
    return true
  },
  deleteProperty(target, key) { // 不做任何改变，直接返回 true
    if (__DEV__) {
      console.warn(
        `Delete operation on key "${String(key)}" failed: target is readonly.`,
        target
      )
    }
    return true
  }
}
```

`get` 方法 `readonlyGet`

```typescript
const readonlyGet = /*#__PURE__*/ createGetter(true)
```

使用的是上面提到的  `createGetter` 方法

### readonlyCollectionHandlers

```typescript
export const readonlyCollectionHandlers: ProxyHandler<CollectionTypes> = {
  get: createInstrumentationGetter(true, false)
}
```

用的 `createInstrumentationGetter` 方法，创建的 `handle` 对象

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

### shallowReadonlyHandlers

```typescript
export const shallowReadonlyHandlers: ProxyHandler<object> = extend(
  {},
  readonlyHandlers,
  {
    get: shallowReadonlyGet
  }
)
```

`get` 方法 `shallowReadonlyGet`

```typescript
const shallowReadonlyGet = /*#__PURE__*/ createGetter(true, true)
```

### readonlyCollectionHandlers

```typescript
export const readonlyCollectionHandlers: ProxyHandler<CollectionTypes> = {
  get: createInstrumentationGetter(true, false)
}
```

上面已经提到过

### createReactiveObject 方法

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

通过 `ReactiveFlags.IS_REACTIVE` / `__v_isReactive` 属性判断是不是 `reactive` 对象，这个对象的获取在 `proxy` 的 `handle` 中做了处理

```typescript
export function isReactive(value: unknown): boolean {
  if (isReadonly(value)) {
    return isReactive((value as Target)[ReactiveFlags.RAW])
  }
  return !!(value && (value as Target)[ReactiveFlags.IS_REACTIVE])
}
```

示例：

```typescript
setup() {
	const b = Vue.reactive({a: 123})
    console.log(Vue.isReactive(b)) // true
    console.log(Vue.isReactive({__v_isReactive: true})) // true
    console.log(Vue.isReactive({__v_isReactive: 1})) // true
}
```



## isReadonly 方法

通过 `ReactiveFlags.IS_READONLY` / `__v_isReadonly` 判断是否是 `readonly` 类型

```typescript
export function isReadonly(value: unknown): boolean {
  return !!(value && (value as Target)[ReactiveFlags.IS_READONLY])
}
```

示例：

```typescript
setup() {
	const b = Vue.readonly({a: 123})
    console.log(Vue.isReadonly(b)) // true
    console.log(Vue.isReadonly({__v_isReadonly: true})) // true
    console.log(Vue.isReadonly({__v_isReadonly: 1})) // true
}
```



## isProxy 方法

判断数据是不是 `reactive` 或者 `readonly` 类型的数据

```typescript
export function isProxy(value: unknown): boolean {
  return isReactive(value) || isReadonly(value)
}
```

示例：

```typescript
setup() {
    const a = Vue.reactive({a: 123})
    const b = Vue.readonly({a: 123})
    console.log(Vue.isProxy(a)) // true
    console.log(Vue.isProxy(b)) // true
    console.log(Vue.isProxy({__v_isReadonly: true})) // true
    console.log(Vue.isProxy({__v_isReactive: true})) // true
}
```



## toRaw 方法

方法接收一个参数，如果参数是 `reactive` / `readonly` 对象会返回 `reactive` 对象的原始值，如果不是 `reactive` 对象，就返回接收的参数

`ReactiveFlags.RAW` 在 `reactive` 对象的 `get` 方法中做了处理 

```typescript
export function toRaw<T>(observed: T): T {
  return (
    (observed && toRaw((observed as Target)[ReactiveFlags.RAW])) || observed
  )
}
```

示例：

```typescript
setup() {
    const a = Vue.reactive({a: 123})
    const b = Vue.readonly({b: 123})
    console.log(Vue.toRaw(a)) // {a: 123}
    console.log(Vue.toRaw(b)) // {b: 123}
    console.log(Vue.toRaw({__v_isReadonly: true})) // {__v_isReadonly: true}
    console.log(Vue.toRaw({__v_isReactive: true})) // {__v_isReactive: true}
}
```



## markRaw 方法

接收一个参数对象，标记这个对象是不需要代理的，通过 `ReactiveFlags.SKIP` / `__v_skip` 实现，会给对象添加一个 ` __v_skip` 属性，值是  `true`

```typescript
export function markRaw<T extends object>(value: T): T {
  def(value, ReactiveFlags.SKIP, true)
  return value
}
```

示例：

```typescript
setup() {
	const a = Vue.markRaw({a: 123})
    console.log(a) // {a: 123, __v_skip: true}
    console.log(Vue.reactive({a: 123})) // Proxy {a: 123}
    console.log(Vue.reactive(a)) // {a: 123, __v_skip: true}
    console.log(Vue.reactive({__v_skip: true})) // {__v_skip: true}
}
```

