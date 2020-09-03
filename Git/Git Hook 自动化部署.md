1. 在服务器 git 仓库中创建 hook 文件

    例：

    ```bash
    cd /home/gitrepo/nuxt-blog/hooks/
    cp  post-update.sample  post-update
    ```

2. 在服务器 clone 一份 git 中的项目

    ```bash
    cd /opt/project
    git clone /home/gitrepo/<project-name>.git
    ```

3. 编写 git hook 文件，在新项目中完成自动化部署

    ```bash
    #!/bin/sh
    
    set -e
    
    cd /opt/project/nuxt-blog/
    
    echo "git 同步项目开始"
    
    unset GIT_DIR
    # 一定要unset GIT_DIR清除变量， 不然会引起remote: fatal: Not a git repository: ‘.’错误
    
    git pull origin master
    
    echo "git 执行自动化部署开始..."
    
    npm run generate
    
    echo "npm generate fulfilled"
    
    rm -rf /opt/blog-html/*
    
    mv dist/* /opt/blog-html/
    
    echo "successful"
    ```

## 错误

- ### remote: fatal: Not a git repository: '.'

    #### 解决：

    ​			**在 `post-update` 文件中 `unset GIT_DIR`**

    > 注意： 一定要unset GIT_DIR清除变量， 不然会引起remote: fatal: Not a git repository: ‘.’错误。