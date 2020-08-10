#	git自动上传

##	思路

- 开机启动一个nodejs服务器，
- 启动成功后，先从配置的git仓库中同步一次代码
- 保存文件后，请求一个nodejs接口，通知nodejs调用git命令同步文件
- 同步过程：（使用延迟加载将多次请求合并成一次）
    - 先更新文件
    - 在提交



##	实现

- node端
  - simple-git库

```javascript
const git = require('simple-git')
const path = '上传的路径'
const commitMessage = '提交时的说明'
const repo = 'https://github.com/[github名字]/[github目录].git'
 
git(path)
  .init()
  .add('./*')
  .commit(commitMessage)
  .addRemote('origin', repo)
  .push(['-f', 'origin', 'master'], () => {
    console.log("Push to master success");
})
```

