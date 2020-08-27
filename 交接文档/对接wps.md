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

开发环境需要启动系统的开发环境和wps加载项的开发环境，`npm run dev`  是业务开发环境 `npm run devWps` 是运行 wps 开发环境

- wps 开发项的启动方式

    在 `plugins/wps/oawps` 目录下，运行 `wpsjs debug`，**如果运行`npm run devWps` 命令，已经集成了wps的启动命令，不需要开发人员手动启动 wps**

- 业务系统调用 wps 加载项的方法，封装在 `src/plugin/wps/wps-api.js` 文件中，使用 `wps-api.js`，必须引入 `wpsjsrpcsdk.js` 文件

    - 例：

        ![image-20200827134818513](https://raw.githubusercontent.com/wukang0718/mdImage/master/images/202008/27/134821-54892.png?token=AKCNZHY4S3FN6PZDDGIYEOS7I5EWG)

- wps 加载项开发文档: [wps开发文档](https://open.wps.cn/docs/office)

#### 生产环境

###### 运行

```bash
npm run build  #	外网环境打包
npm run build:inner   #	内网环境打包
```

内外网环境打包的区别就在于，wps 加载项发布的地址不同

- 外网测试地址：http://192.168.240.21:9092/javascript/plugin/wps/oawps/
- 内网测试地址（国产）：http://192.168.1.30:8012/javascript/plugin/wps/oawps/

**注：**

- wps 加载项目发布一定需要提供服务器地址和端口
- wps 加载项发布过程中会出现选择发布模式，选 **离线模式发布**
- **用户机器需要安装 wps 加载项，才能正常使用**
    - 用户机器访问：`[wps 发布地址]publish.html`
    - 例：
        - 外网用户安装：
            - http://192.168.240.21:9092/javascript/plugin/wps/oawps/publish.html

#### wps 加载项开发

> ​	开发文档地址：[https://open.wps.cn/docs/office](https://open.wps.cn/docs/office)

业务相关功能都封装在 `/plugins/wps/oawps/js/wps.js` 文件中，目前有 新建文档、打开服务器文档