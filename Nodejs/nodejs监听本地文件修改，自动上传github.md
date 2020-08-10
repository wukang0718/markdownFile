# nodejs监听本地文件的修改，自动上传到github

## 思路

1. 监听本地文件的修改
2. 上传到github
3. 设置脚本开机自启动

## 实现

新创建一个项目，执行`npm init`一直回车

在项目中创建一个`config.json`文件，存放github相关配置信息

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
const toast = require("./toast");
const { dirPath, branch, commitMessage, remote } = require("./config.json");
const logger = require('./log');

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



