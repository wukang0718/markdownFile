1. 调试界面新建 `launch.json`

2. 修改配置内容

    ```json
    {
        // 使用 IntelliSense 了解相关属性。 
        // 悬停以查看现有属性的描述。
        // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
        "version": "0.2.0",
        "configurations": [
            {
                "name": "Launch Egg",
                "type": "node",
                "request": "launch",
                "cwd": "${workspaceRoot}",
                "runtimeExecutable": "npm",
                "windows": {
                    "runtimeExecutable": "npm.cmd"
                },
                // 启动我们的 egg-bin debug 并默认是 brk
                "runtimeArgs": [
                    "run",
                    "debug",
                    "--",
                    "--inspect-brk"
                ],
                // 日志输出到 Terminal，否则启动期的日志看不到
                "console": "integratedTerminal",
                "protocol": "auto",
                // 进程重启后自动 attach
                "restart": true,
                // 因为无需再 proxy，故改回原来的 9229 端口
                "port": 9229,
                // 自动 attach 子进程
                "autoAttachChildProcesses": true
            }
        ]
    }
    ```

    