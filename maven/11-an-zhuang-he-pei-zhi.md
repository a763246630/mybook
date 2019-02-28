### 环境变量配置 

```
win

MAVEN_HOME : D:\apache-maven-3.3.9
MAVEN_HOME : E:\tools\maven\apache-maven-3.5.4
Path : %MAVEN_HOME%\bin 

linux

vi  /etc/profile   编辑系统配置文件
#set Maven environment
export MAVEN_HOME=/java/maven/apache-maven-3.5.4
export PATH=$MAVEN_HOME/bin:$PATH

source /etc/profile
mvn -v 查看maven版本
```

