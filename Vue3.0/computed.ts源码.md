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

