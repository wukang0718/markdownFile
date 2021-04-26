- `promise` 构造函数
    - 参数
        - fn： 函数，函数应该有两个参数：resolve、reject

- promise 有三个状态

    - pending // 初始化状态没有执行 resolve 或者是 reject
    - fulfilled // 执行resolve之后的promise 的状态
    - rejected  //  执行reject 之后的promise 的状态

    promise 的状态只能修改一次

- then / catch 都会返回一个新的 promise 实例

- then 函数的返回值不可以是之前的 promise 实例

- then / catch 回调函数只会被调用一次