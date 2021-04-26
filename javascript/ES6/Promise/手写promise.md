- `promise` 构造函数
    - 参数
        - fn： 函数，函数应该有两个参数：resolve、reject

- promise 有三个状态

    - pending // 初始化状态没有执行 resolve 或者是 reject
    - fulfilled // 执行resolve之后的promise 的状态
    - rejected  //  执行reject 之后的promise 的状态

    promise 的装