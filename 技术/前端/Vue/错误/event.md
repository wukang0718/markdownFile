## [Vue warn]: Error in v-on handler: "TypeError: Cannot read property 'change' of undefined"	

#### 原因

​	ts class 定义 中使用了 箭头函数绑定方法，使用箭头函数会在 函数定义是 定义好 this 指向，且不能修改

#### 解决

​	使用普通函数的定义方式声明方法

