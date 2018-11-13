## 用户管理

- 使用root权限执行

```
su 
```

- 切换到指定用户

```
su - username
```

- 增加用户

```
useradd -d /usr/username -m username 或 adduser  username
```

- 为用户增加密码

```
passwd username
```

- 删除用户

```
userdel username
```

- 赋予root权限

```
方法一：修改 /etc/sudoers 文件，找到下面一行，把前面的注释（#）去掉

## Allows people in group wheel to run all commands
%wheel    ALL=(ALL)    ALL

然后修改用户，使其属于root组（wheel），命令如下：
#usermod -g root username

修改完毕，现在可以用tommy帐号登录，然后用命令 su – ，即可获得root权限进行操作。

方法二：修改 /etc/sudoers 文件，找到下面一行，在root下面添加一行，如下所示：

## Allow root to run any commands anywhere
root    ALL=(ALL)     ALL
tommy   ALL=(ALL)     ALL

修改完毕，现在可以用tommy帐号登录，然后用命令 sudo – ，即可获得root权限进行操作。

方法三：修改 /etc/passwd 文件，找到如下行，把用户ID修改为 0 ，如下所示：

tommy:x:0:33:tommy:/data/webroot:/bin/bash
```

- 切换到指定用户

```
su - username
```

- 将用户添加进工作组

```
usermod -G groupname username

语义:
-c: 加上备注文字，备注文字保存在passwd的备注栏中。 
-d：指定用户登入时的主目录，替换系统默认值/home/<用户名> 
-D：变更预设值。 
-e：指定账号的失效日期，日期格式为MM/DD/YY，例如06/30/12。缺省表示永久有效。 
-f：指定在密码过期后多少天即关闭该账号。如果为0账号立即被停用；如果为-1则账号一直可用。默认值为-1. 
-g：指定用户所属的群组。值可以使组名也可以是GID。用户组必须已经存在的，期默认值为100，即users。 
-G：指定用户所属的附加群组。 
-m：自动建立用户的登入目录。 
-M：不要自动建立用户的登入目录。 
-n：取消建立以用户名称为名的群组。 
-r：建立系统账号。 
-s：指定用户登入后所使用的shell。默认值为/bin/bash。 
-u：指定用户ID号。该值在系统中必须是唯一的。0~499默认是保留给系统用户账号使用的，所以该值必须大于499。
--------------------- 
```

- 新建工作组

```
groupadd groupname
```

-  修改组


```
修改GID groupmod -g newgid groupname
```

```
修改组名 groupmod -n newgroupname groupname
```

-  删除组


```
groupdel GROUPNAME
```

-  给组加密码 

```
gpasswd GROUPNAME
```

