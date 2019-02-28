windows

```
计算机→属性→高级系统设置→高级→环境变量

系统变量→新建 JAVA_HOME 变量.变量值填写jdk的安装目录（本人是 E:\Java\jdk1.7.0)

E:\tools\jdk\jdk1.8.0_171

系统变量→寻找 Path 变量→编辑  在变量值最后输入 %JAVA_HOME%\bin;%JAVA_HOME%\jre\bin;
变量名: CLASSPATH
变量值: .;%JAVA_HOME%\lib\tools.jar;%JAVA_HOME%\lib\dt.jar;(1.5后不需要配置次变量)
```

linux

```
wget https://download.oracle.com/otn-pub/java/jdk/8u201-b09/42970487e3af4f5aa5bca3f542482c60/jdk-8u201-linux-x64.tar.gz

我们需要安装的有jdk1.7，安装包去官网下载就行了

安装jdk
首先，解压JDK程序包：

tar  -zxf  jdk-7u71-linux-x64.tar.gz –C

如果是rpm包

rpm –ivh  jdk-7u71-linux-x64.tar.gz

然后重命名文件夹

mv jdk1.7.0_71/ jdk/

最后配置环境变量

vi  ~/.bashrc

在文件末尾添加如下配置：

export  JAVA_HOME=/opt/jdk

export  PATH=$PATH:$JAVA_HOME

·用文本编辑器打开/etc/profile 
·在profile文件末尾加入： 
export JAVA_HOME=/java/jdk/jdk1.8.0_172
export PATH=$JAVA_HOME/bin:$PATH 
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
```



