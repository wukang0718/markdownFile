# nodejs监听本地文件的修改，自动上传到github

[toc]

[代码地址：https://github.com/wukang0718/nodeServerUploadMarkdownFile](https://github.com/wukang0718/nodeServerUploadMarkdownFile)

> 

## 思路

1. 监听本地文件的修改
2. 上传到github
3. 设置脚本开机自启动

## 实现

新创建一个项目，执行`npm init`一直回车

在项目中创建一个`config.json`文件，存放github相关配置信息

```json
{
    "dirPath": "D:/dir/markdown文件",  // 要监听的本地文件目录
    "commitMessage": "使用nodejs自动提交的文件",   // 提交到github的message
    "repo": "<github.name>/<项目名称>", // github仓库地址
    "branch": "master", // 分支
    "remote": "origin" // 仓库
}
```



### 准备工作

#### 日志处理

> ​	使用`log4js`模块

- 安装

```bash
npm install log4js --save
```

- 使用

在项目目录中创建`log.js`

```javascript
const log4js = require('log4js');

log4js.configure({
    appenders: {
        file: {
            type: 'file',
            filename: 'logs/app.log',
            layout: {
                type: 'pattern',
                pattern: '%r %p - %m',
            }
        }
    },
    categories: {
        default: {
            appenders: ['file'],
            level: 'debug'
        }
    }
})

module.exports = log4js.getLogger()
```

#### 消息提示（可有可无，设置自启动后，消息提示不会再生效了）

> ​	使用`node-notifier`模块

- 安装

```bash
npm install node-notifier --save
```

- 使用

再项目目录中创建`toast.js`文件

```javascript
const notifier = require("node-notifier");
const path = require('path');

module.exports = function(message) {
    notifier.notify({
        title: "nodejsServer消息",
        message,
        icon: path.join(__dirname, "./assets/icon.png") // 提示消息的图标，可有可无
    })
}
```

### 上传到github

> ​	使用`simple-git`模块

- 安装

```bash
npm install simple-git --save
```

- 使用

在项目下创建`git-cmd.js`的文件

```javascript
const git = require('simple-git');
const toast = require("./toast"); // toast时一个消息提示插件，可有可无，所有的信息都会落在日志里,设置自启动后，消息提示不会生效
const { dirPath, branch, commitMessage, remote } = require("./config.json");
const logger = require('./log'); // 落日志

/**
 * 初始化git
 */
let gitEntity = git(dirPath)

/**
 * git提交，要先更新在提交
 */
function gitPush() {
    gitPull().then(() => {
        logger.info("开始提交到github");
        gitEntity
            .add('./*')
            .commit(commitMessage)
            .push([remote, branch])
            .then(() => {
                logger.info(`Push to ${branch} success`);
            })
            .catch(err => {
                logger.error(`Push to ${branch} error`);
                logger.error(err);
                toast(`Push to ${branch} error`);
            })

    })
}

/**
 * git更新
 */
function gitPull() {
    logger.info("开始从github更新");
    return gitEntity
        .pull(remote, branch)
        .then(() => {
            logger.info(`Pull to ${branch} success`);
        })
        .catch(err => {
            logger.error(`Pull to ${branch} error`);
            logger.error(err);
            toast(`Pull to ${branch} error`);
        });
}

/**
 * 延时执行函数, 延迟3秒
 */
const delayMethod = function (type) {
    let timer;
    return function () {
        clearTimeout(timer);
        timer = setTimeout(function () {
            gitCmd[type]();
            clearTimeout(timer);
        }, 3000)
    }
};

const gitCmd = {
    push: gitPush,
    pull: gitPull,
    delayInvoke: {
        push: delayMethod("push"),
        pull: delayMethod("pull")
    }
}


module.exports = function (type, immediately) {
    if (immediately) {
        return gitCmd[type]();
    } else {
        gitCmd.delayInvoke[type]();
    }
}
```

### 监听文件

> ​	使用`chokidar`模块

- 安装

```bash
npm install chokidar --save
```

- 使用

在项目下新建`watchFile.js`文件

```javascript
const chokidar = require('chokidar');
const invokeGit = require("./git-cmd");

/**
 * 监听文件夹
 * @param {*} dir 
 */

module.exports = function(dirPath) {
    chokidar.watch(dirPath, {
        ignored: /(\.git)|(\.idea)|(\.vscode)/
    }).on("all", ((event, path) => {
        // 文件修改同步到github仓库
        invokeGit("push")
    }))
}
```

### 启动项目

再项目目录中创建`app.js`文件

```javascript
const invokeGit = require("./git-cmd");
const toast = require("./toast");
const logger = require("./log");
const watch = require("./watchFile");
const { dirPath } = require("./config.json");

logger.info('服务启动成功');
toast("nodejsUploadMarkdown初始化成功");

// 程序加载 ---- 更新文件
invokeGit("pull", true).then(() => {
    watch(dirPath);
    logger.info("监听文件");
    toast("markdwon笔记初始化更新完成，开始监听文件自动同步");
});
```

### 设置开机自启动

使用`node-windows`模块

- 安装

```bash
npm install node-windows --save
```

- 使用

再项目中创建`server.js`文件

```javascript
const logger = require("./log");
const toast = require("./toast");
const path = require("path");
const Service = require("node-windows").Service;

const svc = new Service({
    name: "nodejsUploadMarkdownToGithub",
    description: "nodejs脚本自动上传文件到github",
    script: path.resolve(__dirname, "app.js"),
    wait: 1,
    grow: 0.25,
    maxRestarts: 40
});

svc.on("install", () => {
    logger.info("自启动程序安装成功");
    svc.start();
})

svc.on("uninstall", () => {
    logger.info("自启动程序卸载成功");
})

svc.on("alreadyinstalled ", () => {
    logger.warn("程序已经启动");
})

svc.on("error", (err) => {
    logger.error("自启动服务异常" + err);
    toast("自启动服务异常");
})

svc.on("start", () => {
    logger.info("自启动脚本，启动服务");
})

const args = process.argv.slice(2);
if (args[0] === "uninstall") {
    svc.uninstall();
} else {
    svc.install();
}
```

### 再package.json中配置启动命令

```json
{
    "scripts": {
        "start": "node server.js", // 启动服务
        "uninstall": "node server.js uninstall" // 卸载服务
     }
}
```

## 注意事项

- 要修改repo的remote 

```bash
git remote set-url remote-name https://<username>:<password>@github.com/<username>/<repo_name>.git
```

- 设置为repo用户名和邮箱 

```bash
git config  user.email "xxxx@xx.com"
git config  user.name "xxxx"
```

## 启动服务

```bash
npm start
```

## 遗留问题

- 一定要设置repo的remote  和 为repo单独设置用户名和邮箱，不明白为什么？
- 有时候会出现第一次安装没有运行启动脚本，卸载再安装就可以了，不明白为什么？

