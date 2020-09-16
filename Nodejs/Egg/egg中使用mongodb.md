## 安装依赖

```bash
npm install egg-mongoose --save
```

## 添加插件

> config/plugin.js	

```js
'use strict';

/** @type Egg.EggPlugin */
module.exports = {
  // had enabled by egg
  // static: {
  //   enable: true,
  // }
  mongoose: {
    enable: true,
    package: 'egg-mongoose'
  }
};
```

## 配置插件

> config/config.default.js

添加一下代码

```js
exports.mongoose = {
  client: {
    url: 'mongodb://39.96.170.240/blog',
    options: {}
  },
};
```

## 使用

再 `app` 目录下创建 `model` 目录，`model` 目录中使用 `mongoose`

> app/model/Type.js

```js
module.exports = app => {
    const mongoose = app.mongoose;
    console.log("aasas")
    const Schema = mongoose.Schema

    const TypeSchema = new Schema({
        name: String
    })

    return mongoose.model("Type", TypeSchema)
}
```

