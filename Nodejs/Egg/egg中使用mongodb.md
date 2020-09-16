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

改成一下:

配置参数卸载 `config` 中，不使用 `exports.mongoose` 的方式



```js
/* eslint valid-jsdoc: "off" */

'use strict';

/**
 * @param {Egg.EggAppInfo} appInfo app info
 */
module.exports = appInfo => {
  /**
   * built-in config
   * @type {Egg.EggAppConfig}
   **/
  const config = exports = {
    mongoose: {
      client: {
        url: 'mongodb://39.96.170.240/blog',
        options: {
          useNewUrlParser: true,
          useUnifiedTopology: true,
          useFindAndModify: false,
          useCreateIndex: true,
          autoReconnect: true
        }
      }
    },
    security: {
      csrf: {
        enable: false,
      }
    }
  };

  // use for cookie sign key, should change to your own and keep security
  config.keys = appInfo.name + '_1600059179764_5209';

  // add your middleware config here
  config.middleware = [];

  // add your user config here
  const userConfig = {
    // myAppName: 'egg',
  };

  return {
    ...config,
    ...userConfig,
  };
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

