## 服务管理

* 查询服务端口占用

```
lsof -i:端口号 或 netstat -tunlp|grep 端口号
```

* 杀进程

```
kill -9 [pid]
```

* 查看内存占用

```
free -h
```

![](/node_modules/img/free -h.png)



