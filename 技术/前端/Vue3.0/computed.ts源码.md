> 源码位置： [https://github.com/vuejs/vue-next/blob/master/packages/reactivity/src/computed.ts](https://github.com/vuejs/vue-next/blob/master/packages/reactivity/src/computed.ts)

## computed

接收一个参数可以是一个 `function` 或者是一个对象，对象必须有 `get` 方法，可以设置 `set` 方法，返回一个有 `value` 属性的对象

```typescript
/**
 * 计算属性的方法 
 * @param getterOrOptions get 方法 或者 是一个对象，对象中包括 get 和 set 方法
 */
export function computed<T>(
  getterOrOptions: ComputedGetter<T> | WritableComputedOptions<T>
) {
  let getter: ComputedGetter<T>
  let setter: ComputedSetter<T>
  // 参数是一个函数的时候，就是 getter 函数
  if (isFunction(getterOrOptions)) {
    getter = getterOrOptions
    setter = __DEV__
      ? () => {
          console.warn('Write operation failed: computed value is readonly')
        }
      : NOOP
  } else {
    getter = getterOrOptions.get
    setter = getterOrOptions.set
  }

  // ComputedRefImpl 返回这个实例
  return new ComputedRefImpl(
    getter,
    setter,
    isFunction(getterOrOptions) || !getterOrOptions.set
  ) as any
}
```

### ComputedRefImpl 类

```typescript
/**
 * 一个 computed 实例
 * getter  方法
 * _setter  方法
 * isReadonly 是否可以修改
 */
class ComputedRefImpl<T> {
  private _value!: T
  private _dirty = true

  public readonly effect: ReactiveEffect<T>
  // 有一个 __v_isRef 属性 为 true ，在调用 isRef 的时候，返回 true
  public readonly __v_isRef = true;
  public readonly [ReactiveFlags.IS_READONLY]: boolean

  constructor(
    getter: ComputedGetter<T>,
    private readonly _setter: ComputedSetter<T>,
    isReadonly: boolean
  ) {
    // 调用 effect 方法，收集 getter 方法中的依赖
    this.effect = effect(getter, {
      lazy: true, // 方法不会立即执行
      scheduler: () => {
        // 只有这个 值 被使用过，才会重新计算
        if (!this._dirty) {
          this._dirty = true // 等下一次或者值的时候，才会调用
          trigger(toRaw(this), TriggerOpTypes.SET, 'value')
        }
      }
    })

    this[ReactiveFlags.IS_READONLY] = isReadonly
  }

  get value() {
    if (this._dirty) {
      this._value = this.effect() // 这个就是 getter 函数执行的结果
      this._dirty = false
    }
    // 收集依赖
    track(toRaw(this), TrackOpTypes.GET, 'value')
    return this._value
  }

  set value(newValue: T) {
    this._setter(newValue)
  }
}
```

示例：

```typescript
setup() {
    const a = Vue.ref(1)
    const b = Vue.computed(() => {
        console.log("computed");
        return a.value * 2
    })
    a.value = 2;
    a.value = 3;
    a.value = 4;
    console.log(b.value) // computed 8
    a.value = 5;
    a.value = 6;
    console.log(b.value) // computed 12
    console.log(Vue.isRef(b)) // true
}
```

