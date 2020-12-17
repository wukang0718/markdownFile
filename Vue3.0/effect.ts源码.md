> 源码位置： [https://github.com/vuejs/vue-next/blob/master/packages/reactivity/src/effect.ts](https://github.com/vuejs/vue-next/blob/master/packages/reactivity/src/effect.ts)
>
> **这个文件的方法暴露在 "@vue/reactivity"**

## effect 

执行需要收集依赖的函数，又有的依赖收集的调用，都要通过这个函数执行

```typescript
/**
 * 执行需要收集依赖的函数，又有的依赖收集的调用，都要通过这个函数执行
 * @param fn Function 在这个函数里收集依赖
 * @param options 
 */
export function effect<T = any>(
  fn: () => T,
  options: ReactiveEffectOptions = EMPTY_OBJ
): ReactiveEffect<T> {
  if (isEffect(fn)) {
    fn = fn.raw // 获取到函数的原始值
  }
  // 获取到的是一个 加了 effect 标识的函数，并且在这个函数中处理了依赖收集需要的参数
  const effect = createReactiveEffect(fn, options)
  if (!options.lazy) {
    // 没有配置懒执行
    effect()
  }
  return effect
}
```

### createReactiveEffect

执行一个需要收集依赖的函数，返回一个 `effect` 类型的函数

```typescript
/**
 * 执行一个需要收集依赖的函数，返回一个 effect 类型的函数
 * @param fn Function
 * @param options 配置参数
 */
function createReactiveEffect<T = any>(
  fn: () => T,
  options: ReactiveEffectOptions
): ReactiveEffect<T> {
  // 这个函数不会立即执行，是否执行，在 effect 函数中控制
  // 用户在 options 中配置了 lazy：true 可以让这个函数 在 effect 中不执行
  const effect = function reactiveEffect(): unknown {
    // 调用了 stop 后会停止这个函数的依赖处理部分的继续执行
    if (!effect.active) {
      return options.scheduler ? undefined : fn()
    }
    // 保证函数不会重复执行
    if (!effectStack.includes(effect)) {
      cleanup(effect) // 清除 函数的依赖相关
      try {
        enableTracking() // 启动依赖收集
        effectStack.push(effect) // 把当前函数 推到 effect 栈中
        activeEffect = effect // 这个就是依赖收集的时候，收集到的和依赖相关的函数
        return fn() // fn 执行结束了，也就结束了依赖收集
      } finally {
        effectStack.pop() // 函数执行完，出栈
        resetTracking() // 恢复依赖收集，在 fn 中可能执行了 pauseTracking 方法，停止了依赖收集
        activeEffect = effectStack[effectStack.length - 1] // 下一次需要被依赖收集的函数
      }
    }
  } as ReactiveEffect
  effect.id = uid++
  effect.allowRecurse = !!options.allowRecurse
  effect._isEffect = true
  effect.active = true
  effect.raw = fn
  effect.deps = []
  effect.options = options
  return effect
}
```



## pauseTracking

暂停依赖收集

```typescript
/**
 * 暂停收集依赖
 */
export function pauseTracking() {
  trackStack.push(shouldTrack)
  shouldTrack = false // shouldTrack 会停止依赖收集
}
```



## enableTracking

启动依赖收集

```typescript
/**
 * 启动依赖收集
 */
export function enableTracking() {
  trackStack.push(shouldTrack)
  shouldTrack = true
}
```



## resetTracking

重置依赖收集，把依赖收集的状态恢复到上一次，默认是 `true`

```typescript
/**
 * 重置依赖收集
 */
export function resetTracking() {
  const last = trackStack.pop()
  shouldTrack = last === undefined ? true : last
}
```



## track

**收集依赖**

```typescript
/**
 * 收集依赖
 * @param target 收集依赖的依赖源
 * @param type 依赖的类型，可以是get、has、iterate
 * @param key 依赖的哪个属性
 */
export function track(target: object, type: TrackOpTypes, key: unknown) {
  // shouldTrack 标识是否收集依赖，可以调用 pauseTracking 暂停收集依赖
  // activeEffect 渲染时是 当前组件的渲染任务
  // 在执行 effect 或者 watchEffect 时，调用了 ref.value(或者其他会获取依赖的方法)，就是当前 effect（或者其他方法）传递的函数
  if (!shouldTrack || activeEffect === undefined) {
    return
  }
  // targetMap 是 WeakMap 类型，在 依赖的源对象被垃圾回收后，会自动删除这个key
  // depsMap 是对 target 收集的依赖
  // depsMap 是一个 Map类型
  let depsMap = targetMap.get(target)
  if (!depsMap) {
    targetMap.set(target, (depsMap = new Map()))
  }
  // dep 是 set 类型
  let dep = depsMap.get(key)
  if (!dep) {
    // 这里收集需要触发的依赖，使用set类型，保证没有重复的依赖
    /**
     * targetMap(WeakMap): {
     *  target(Map): {
     *    key(Set): []
     *  }
     * }
     */
    depsMap.set(key, (dep = new Set()))
  }
  // 在这里把依赖收集起来
  if (!dep.has(activeEffect)) {
    dep.add(activeEffect)
    activeEffect.deps.push(dep)
    if (__DEV__ && activeEffect.options.onTrack) {
      // onTrack 只在开发的时候可以有效，可以用来追踪依赖收集
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



## trigger

**触发依赖，effect 的 `scheduler` 配置也是在这里被执行的**

```typescript
/**
 * 触发依赖
 * @param target 触发依赖的数据源
 * @param type 触发依赖的类型可以是 set/add/delete/clear
 * @param key 触发依赖的 key，对应着收集依赖的key
 * @param newValue 新值
 * @param oldValue 旧值
 * @param oldTarget 旧的数据源
 */
export function trigger(
  target: object,
  type: TriggerOpTypes,
  key?: unknown,
  newValue?: unknown,
  oldValue?: unknown,
  oldTarget?: Map<unknown, unknown> | Set<unknown>
) {
  // depsMap 获取到 target 对应的 Map 依赖源
  const depsMap = targetMap.get(target)
  // 没有需要触发的依赖
  if (!depsMap) {
    // never been tracked
    return
  }

  const effects = new Set<ReactiveEffect>()
  const add = (effectsToAdd: Set<ReactiveEffect> | undefined) => {
    if (effectsToAdd) {
      effectsToAdd.forEach(effect => {
        // allowRecurse 设置为 true 会导致函数被多次触发
        if (effect !== activeEffect || effect.allowRecurse) {
          // 把依赖添加到 effects 中
          effects.add(effect)
        }
      })
    }
  }

  if (type === TriggerOpTypes.CLEAR) {
    // collection being cleared
    // trigger all effects for target
    // 调用了 .clear 方法，触发 target 的所有依赖
    depsMap.forEach(add)
  } else if (key === 'length' && isArray(target)) {
    // 改变了 数组的 length
    // 要触发 被删除的 元素的 依赖 和 length 的依赖
    depsMap.forEach((dep, key) => {
      if (key === 'length' || key >= (newValue as number)) {
        add(dep)
      }
    })
  } else {
    // schedule runs for SET | ADD | DELETE
    if (key !== void 0) {
      // depsMap.get(key) 是一个 Set
      // 触发 target 上 相关这个 key 的依赖，最普通的情况
      add(depsMap.get(key))
    }

    // also run for iteration key on ADD | DELETE | Map.SET
    switch (type) {
      case TriggerOpTypes.ADD:
        // 添加事件触发
        if (!isArray(target)) {
          // 数组的遍历要触发
          add(depsMap.get(ITERATE_KEY))
          if (isMap(target)) {
            // map 的编辑需要触发
            add(depsMap.get(MAP_KEY_ITERATE_KEY))
          }
        } else if (isIntegerKey(key)) {
          // 数组添加一项
          // new index added to array -> length changes
          // 数组长度变化，需要触发 length 的相关依赖
          add(depsMap.get('length'))
        }
        break
      case TriggerOpTypes.DELETE:
        // 删除事件触发
        if (!isArray(target)) {
          // 数组遍历事件触发
          add(depsMap.get(ITERATE_KEY))
          if (isMap(target)) {
            // map 遍历事件触发
            add(depsMap.get(MAP_KEY_ITERATE_KEY))
          }
        }
        break
      case TriggerOpTypes.SET:
        if (isMap(target)) {
          // 触发 map 的遍历
          add(depsMap.get(ITERATE_KEY))
        }
        break
    }
  }

  const run = (effect: ReactiveEffect) => {
    // onTrigger 开发环境可以用
    if (__DEV__ && effect.options.onTrigger) {
      effect.options.onTrigger({
        effect,
        target,
        key,
        type,
        newValue,
        oldValue,
        oldTarget
      })
    }
    if (effect.options.scheduler) {
      // 如果 effect 中设置了 scheduler，调用这个方法，自定义是否需要触发依赖
      // effect 和 watchEffect 方法可以设置，其他方法（watch）??? TODO
      effect.options.scheduler(effect)
    } else {
      effect()
    }
  }
  // 调用所有依赖添加的函数
  effects.forEach(run)
}
```

