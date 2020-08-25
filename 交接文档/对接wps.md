## 项目地址

> svn://39.106.209.22/repository/repository/trunk/web/jtb-web-wps

## 项目目录

![项目目录](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20200825125537974.png)

- plugins    插件
    - wps wps 插件
        - oawps 发文系统使用的 wps 加载项
- src  业务代码 （结构同发文管理）

## 环境

#### 开发环境

> ​	开发环境需要集成 `wps` 开发环境，需要全局安装 `wpsjs` 模块
>
> ```bash
> npm install wpsjs -g
> ```

###### 运行

```bash
npm run dev
```

开发环境需要启动系统的开发环境和wps加载项的开发环境，**`npm run dev` 环境已经全部集成了**

- wps 开发项的启动方式

    在 `plugins/wps/oawps` 目录下，运行 `wpsjs debug`，**如果运行`npm run dev` 命令，已经集成了wps的启动命令，不需要开发人员手动启动 wps**

- 业务系统调用 wps 加载项的方法，封装在 `src/plugin/wps/wps-api.js` 文件中，使用 `wps-api.js`，必须引入 `wpsjsrpcsdk.js` 文件

    - 例：

        ![wps-api.js](https://raw.githubusercontent.com/wukang0718/mdImage/master/images/202008/25/130906-232759.png?token=AKCNZH63BHA5O6UGPHZGX6K7ISOTA)

- wps 加载项开发文档: 