## ref 方法

> 源码地址：[https://github.com/vuejs/vue-next/blob/master/packages/reactivity/src/ref.ts](https://github.com/vuejs/vue-next/blob/master/packages/reactivity/src/ref.ts)

接收一个参数，返回参数的深代理，如果参数是对象，`.value` 是 `reactive` 的值

```typescript
export function ref(value?: unknown) {
  return createRef(value)
}
```

## shallowRef 方法

> 源码地址：[https://github.com/vuejs/vue-next/blob/master/packages/reactivity/src/ref.ts](https://github.com/vuejs/vue-next/blob/master/packages/reactivity/src/ref.ts)

接收一个参数，返回 对参数 浅代理

```typescript
export function shallowRef(value?: unknown) {
  return createRef(value, true)
}
```

调用 `createRef` 方法

#### createRef 方法

判断如果已经是 `ref` 对象，直接返回这个对象

否则，返回一个 `RefImpl` 类型的实例

```typescript
function createRef(rawValue: unknown, shallow = false) {
  if (isRef(rawValue)) { // 判断是否已经是 ref 对象
    return rawValue
  }
  return new RefImpl(rawValue, shallow)
}
```

#### RefImpl 类

`new` 接收两个参数

| 参数      | 描述                                            |
| --------- | ----------------------------------------------- |
| _rawValue | 原始值                                          |
| _shallow  | 是否只做浅层的代理，默认值 false 做 深层 的代理 |

实例属性：

| 属性      | 描述                                                    |
| --------- | ------------------------------------------------------- |
| _value    | 原始值                                                  |
| __v_isRef | ref 对象的表示，在判断对象是不是 ref 对象的时候，会用到 |
| _rawValue | 原始值 或 修改后的原始值                                |
| _shallow  | 是否只做浅层的监听                                      |
| value     | 暴露给外层访问的属性，会调用 get、set 方法              |

```typescript
class RefImpl<T> {
  private _value: T

  public readonly __v_isRef = true

  constructor(private _rawValue: T, public readonly _shallow = false) {
    this._value = _shallow ? _rawValue : convert(_rawValue) // convert 函数做深层的代理
  }

  get value() {
    track(toRaw(this), TrackOpTypes.GET, 'value') // track 函数收集依赖，后续描述
    return this._value
  }

  set value(newVal) {
    // hasChanged 方法会对 NaN 做处理
    if (hasChanged(toRaw(newVal), this._rawValue)) { // 如果修改后的值，和当前值不一致
      this._rawValue = newVal
      this._value = this._shallow ? newVal : convert(newVal)
      trigger(toRaw(this), TriggerOpTypes.SET, 'value', newVal) // trigger 函数触发依赖，后续描述
    }
  }
}
```

#### convert 函数

接收一个参数，如果参数是`对象/数组`类型，调用 reactive 方法，否则返回参数本身

```typescript
const convert = <T extends unknown>(val: T): T =>
  isObject(val) ? reactive(val) : val
```

示例：

```typescript
setup() {
	const ref0 = Vue.ref({a: 1})
    const ref1 = Vue.shallowRef({a: 1})
    console.log(ref0); // {..., value: Proxy{a: 1}}
    console.log(ref1); // {..., value: Object{a: 1}}
}
```

## isRef 方法

> 源码位置： [https://github.com/vuejs/vue-next/blob/master/packages/reactivity/src/ref.ts](https://github.com/vuejs/vue-next/blob/master/packages/reactivity/src/ref.ts)

方法接收一个参数，返回 `boolean`，判断参数是不是 `ref` 类型

```typescript
export function isRef(r: any): r is Ref {
  return Boolean(r && r.__v_isRef === true)
}
```

示例：

```typescript
setup() {
	const ref = Vue.ref(0)
    console.log(isRef(ref)); // true
    console.log(isRef(0)); // false
    console.log(isRef({__v_isRef: true})) // true
}
```

## triggerRef 方法

> 源码位置： [https://github.com/vuejs/vue-next/blob/master/packages/reactivity/src/ref.ts](https://github.com/vuejs/vue-next/blob/master/packages/reactivity/src/ref.ts)

方法接收一个 `ref` 对象，触发这个 `ref` 对象上收集的依赖

```typescript
export function triggerRef(ref: Ref) {
  trigger(toRaw(ref), TriggerOpTypes.SET, 'value', __DEV__ ? ref.value : void 0)
}
```

示例：

```typescript
setup() {
    const ref = Vue.ref(0)
    Vue.watchEffect(() => { // watchEffect 默认会收集依赖
        console.log(ref.value); // 0
    })
    Vue.triggerRef(ref) // 0
}
```

## unref 方法

> 源码位置： [https://github.com/vuejs/vue-next/blob/master/packages/reactivity/src/ref.ts](https://github.com/vuejs/vue-next/blob/master/packages/reactivity/src/ref.ts)

方法接收一个参数，如果参数是 `ref` 类型，返回 `.value` 的值，否则返回参数自己

```typescript
export function unref<T>(ref: T): T extends Ref<infer V> ? V : T {
  return isRef(ref) ? (ref.value as any) : ref
}
```

示例：

```typescript
setup() {
    const ref = Vue.ref(0)
    console.log(Vue.unref(ref)) // 0
    console.log(Vue.unref(0)) // 0
}
```

## proxyRefs 方法

> 源码位置： [https://github.com/vuejs/vue-next/blob/master/packages/reactivity/src/ref.ts](https://github.com/vuejs/vue-next/blob/master/packages/reactivity/src/ref.ts)

方法接收一个参数，返回一个 `proxy` 对象

> 这个方法返回的 `proxy` 对象，不会触发依赖的收集和触发

```typescript
export function proxyRefs<T extends object>(
  objectWithRefs: T
): ShallowUnwrapRef<T> {
  return isReactive(objectWithRefs)
    ? objectWithRefs
    : new Proxy(objectWithRefs, shallowUnwrapHandlers)
}

```

`proxy` 的 `handle` 是 `shallowUnwrapHandlers`

`get` 返回数据的原值

`set` 方法针对代理对象的类型做修改

```typescript
const shallowUnwrapHandlers: ProxyHandler<any> = {
  get: (target, key, receiver) => unref(Reflect.get(target, key, receiver)),
  set: (target, key, value, receiver) => {
    const oldValue = target[key]
    if (isRef(oldValue) && !isRef(value)) {
      oldValue.value = value
      return true
    } else {
      return Reflect.set(target, key, value, receiver)
    }
  }
}
```

示例：

```typescript
setup() {
	console.log(Vue.proxyRefs({a: 12})) // Proxy{a: 12}
}
```



## customRef 方法

自定义 `ref` 类型，接收一个函数，函数可以接收两个参数

- track `Function` 调用可以收集依赖
- trigger `Function` 调用可以派发依赖

函数需要返回一个对象，对象需要包含 `get` 和 `set` 方法，如果需要响应式，应该在 `get` 方法中调用 `track `，在 `set` 方法中调用 `trigger`

- get `Function` 在获取返回值的 .`value` 时调用
- set `Function` 在设置返回值的 `.value` 时调用

```typescript
export function customRef<T>(factory: CustomRefFactory<T>): Ref<T> {
  return new CustomRefImpl(factory) as any
}
```

返回了一个 `CustomRefImpl` 实例

### CustomRefImpl 类

```typescript
class CustomRefImpl<T> {
  private readonly _get: ReturnType<CustomRefFactory<T>>['get']
  private readonly _set: ReturnType<CustomRefFactory<T>>['set']

  public readonly __v_isRef = true

  constructor(factory: CustomRefFactory<T>) {
    const { get, set } = factory(
      () => track(this, TrackOpTypes.GET, 'value'),
      () => trigger(this, TriggerOpTypes.SET, 'value')
    )
    this._get = get
    this._set = set
  }

  get value() {
    return this._get()
  }

  set value(newVal) {
    this._set(newVal)
  }
}
```

示例：

```typescript
Vue.createApp({
    template: "#item-template",
    setup() {
        const text = useDebouncedRef("123")
        Vue.watchEffect(() => {
            console.log(text.value)
            console.timeEnd("input")
        })
        console.time("input")
        text.value = "345"
        return {
            text
        }
    }
}).use(ElementPlus).mount("#app")
// customRef 实现防抖函数
function useDebouncedRef(value, delay = 200) {
    let timer;
    return Vue.customRef((track, trigger) => ({
        get() {
            track();
            return value
        },
        set(val) {
            timer && clearTimeout(timer);
            if (value !== val) {
                timer = setTimeout(() => {
                    value = val;
                    trigger();
                    timer = null;
                }, delay)
            }
        }
    }))
}
```

![image-20201215143308194](https://gitee.com/wu_kang0718/image/raw/master//20201215143310027.png)

## toRef 方法

返回对象的 key 对应的值，返回的是一个具有 `value` 属性的对象

```typescript
export function toRef<T extends object, K extends keyof T>(
  object: T,
  key: K
): ToRef<T[K]> {
  return isRef(object[key])
    ? object[key]
    : (new ObjectRefImpl(object, key) as any)
}
```

### ObjectRefImpl 类

返回一个具有 `value` 属性的对象，这个对象不会收集依赖和触发依赖

```typescript
class ObjectRefImpl<T extends object, K extends keyof T> {
  public readonly __v_isRef = true

  constructor(private readonly _object: T, private readonly _key: K) {}

  get value() {
    return this._object[this._key]
  }

  set value(newVal) {
    this._object[this._key] = newVal
  }
}
```

示例：

```typescript
setup() {
   const b = Vue.ref(12)
   const text = {
       a: 12,
       b: Vue.ref(12)
   }
   console.log(Vue.toRef(text, "a"))
   console.log(Vue.toRef(text, "b"))
}
```



## toRefs 方法

接收一个对象，返回一个新的对象， 这个对象的每一个属性都是一个具有 `value` 属性的对象，这个方法只做一层代理，没有深度遍历，且不会收集依赖和触发依赖

```typescript
export function toRefs<T extends object>(object: T): ToRefs<T> {
  if (__DEV__ && !isProxy(object)) {
    console.warn(`toRefs() expects a reactive object but received a plain one.`)
  }
  const ret: any = isArray(object) ? new Array(object.length) : {}
  for (const key in object) {
    ret[key] = toRef(object, key)
  }
  return ret
}
```













