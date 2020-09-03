1. 在服务器 git 仓库中创建 hook 文件

    例：

    ```bash
    cd /home/gitrepo/nuxt-blog/hooks/
    cp  post-update.sample  post-update
    ```

2. 在服务器 clone 一份 git 中的项目

    - 生成服务器自己的公钥

        ```bash
        ssh-keygen -t rsa -C "<your_email@youremail.com>"
        ```

    - 添加到git中

        ```bash
         cd /root/.ssh/
         cat id_rsa.pub >> /home/git/.ssh/authorized_keys
        ```

    - clone 项目	

        ```bash
        cd /opt/project
        git clone git@<ip>:/home/gitrepo/<project-name>.git
        ```

3. 编写 git hook 文件，在新项目中完成自动化部署

