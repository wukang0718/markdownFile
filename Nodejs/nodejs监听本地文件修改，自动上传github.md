# nodejs监听本地文件的修改，自动上传到github

## 思路

1. 监听本地文件的修改
2. 上传到github
3. 设置脚本开机自启动

## 实现

新创建一个项目，执行`npm init`一直回车

在项目中创建一个`config.json`文件，存放github相关配置信息

```json
{
    "dirPath": "D:/武康/markdown文件",  // 要监听的本地文件目录
    "commitMessage": "公司电脑设置nodejs脚本自动提交",   // 提交到github的message
    "repo": "wukang0718/markdownFile", // github仓库地址
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

在项目目录中创建`log`



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
const toast = require("./toast"); // toast时一个消息提示插件，可有可无，所有的信息都会落在日志里
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



