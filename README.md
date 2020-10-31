# markdownFile

保存一些工作和学习中的笔记



`npm install picgo -g`

`picgo install gitee-uploader`

`picgo-plugin-super-prefix`

#### pcigo 配置

```json
{
  "picBed": {
    "current": "gitee",
    "uploader": "gitee",
    "gitee": {
      "branch": "master", // 图片上传到这个分支
      "customPath": "",
      "customUrl": "",
      "path": "/", // 路径
      "repo": "wu_kang0718/image", // <用户名>/<仓库名称>
      "token": "6e16e1954c7f748c358ba0c5c1327918" // token 令牌
    }
  },
  "picgoPlugins": {
    "picgo-plugin-gitee-uploader": true,
    "picgo-plugin-super-prefix": true
  },
  "picgo-plugin-gitee-uploader": {
    "lastSync": "2020-10-31 02:22:35"
  },
  "picgo-plugin-super-prefix": {
    "fileFormat": "YYYYMMDDHHmmssSSS"
  }
}
```

#### typora 设置

选择 `Custom Command`

`picgo upload`