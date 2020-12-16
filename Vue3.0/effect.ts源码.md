## effect 

**这个方法没有暴露给开发者，但是很重要**

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

收集依赖



## trigger

