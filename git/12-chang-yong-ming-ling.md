# 常用命令

- 在当前目录新建一个Git代码库

```
git init
```

- 新建一个目录，将其初始化为Git代码库

```
git init [project-name]
```

- 下载一个项目和它的整个代码历史

```
git clone git@gitlab.com:a763246630/springbootdemo.git 克隆git代码库
```

- Git的设置文件为.gitconfig，它可以在用户主目录下（全局配置），也可以在项目目录下（项目配置）。

- 显示当前的Git配置

```
git config --list
```

- 编辑Git配置文件

```
git config -e [--global]
```

- 设置提交代码时的用户信息

```
git config [--global] user.name "[name]"
```

```
git config [--global] user.email "[email address]"
```

-  更新远程代码库到本地,自动merge.

```
git pull
```

-  更新远程代码库到本地,不会自动merge

```
git fetch
```

- 将文件添加到git stage（与git关联）中

```
git add
```

- 提交修改代码到本地库

```
git commit -m 'comments here'
```

