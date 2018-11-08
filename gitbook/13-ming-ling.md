## gitbook相关命令

* 下载初始化文件和配置

```
gitbook init
```

- 安装配置插件
```
gitbook install
```
-  构建book
```
gitbook build
```
- 构建并启动项目默认端口4000 lrport 指定监控端口 port 启动端口
```
gitbook serve --lrport 32928 --port 4000
```
- 解析和打印关于book0的调试信息	级别调试，信息，警告，错误，禁用，  默认为信息
```
gitbook parse
```
- 显示GitBook 帮助信息

```
gitbook -h
```

- 显示GitBook可用命令

```
gitbook help
```

- 列出本地已安装的gitbook版本

```
gitbook ls
```

- 列出当前活动的gitbook版本

```
gitbook current
```

- 列出远程可以下载安装的gitbook版本

```
gitbook ls-remote
```

- 下载安装某个gitbook版本

```
gitbook fetch [version]
```

- 卸载某个gitbook版本

```
gitbook uninstall [version]
```

- 更新到最新的gitbook版本

```
gitbook update
```

