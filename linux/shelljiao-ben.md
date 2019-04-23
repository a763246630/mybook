### 基本语法

#!/bin/sh是指此脚本使用/bin/sh来解释执行，#!是特殊的表示符，其后面跟的是此解释此脚本的shell的路径。或者 #!/bin/bash   

**/bin/sh**与**/bin/bash**虽然大体上没什么区别，但仍存在不同的标准。标记为**#!/bin/sh**的脚本不应使用任何**POSIX**没有规定的特性 (如**let**等命令, 但**#!/bin/bash**可以)。Debian曾经采用**/bin/bash**更改**/bin/dash**，目的使用更少的磁盘空间、提供较少的功能、获取更快的速度。但是后来经过shell脚本测试存在运行问题。因为原先在**bash shell**下可以运行的shell script (shell 脚本)，在**/bin/sh**下还是会出现一些意想不到的问题，不是100%的兼用

使用**man sh**命令和**man bash**命令去观察，可以发现**sh**本身就是**dash**，也就更好的说明集成Debian系统之后的更改。

$ cat /etc/shells可以查看系统支持的shell格式
其实第一句的#!是对脚本的解释器程序路径，脚本的内容是由解释器解释的，我们可以用各种各样的解释器来写对应的脚本。

```
kill `lsof -t -i:4505` 根据端口杀进程
 

chmod u+x test.sh
 

chmod +77
```

