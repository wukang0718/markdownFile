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



## pauseTracking

## enableTracking

## resetTracking

## track

## trigger

