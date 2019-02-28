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

打包跳过test目录的三种方法

mvn package -Dmaven.test.skip=true  跳过test

mvn package -DskipTests

也可以在pom.xml文件中修改

<plugin>  
    <groupId>org.apache.maven.plugin</groupId>  
    <artifactId>maven-compiler-plugin</artifactId>  
    <version>2.1</version>  
    <configuration>  
        <skip>true</skip>  
    </configuration>  
</plugin>  
<plugin>  
    <groupId>org.apache.maven.plugins</groupId>  
    <artifactId>maven-surefire-plugin</artifactId>  
    <version>2.5</version>  
    <configuration>  
        <skip>true</skip>  
    </configuration>  
</plugin> 



将jar上传到maven库
在nexus私库setting  Repositorie的Repositories新建 或 用自带的


    <distributionManagement>
        <!--pom.xml这里<id> 和 settings.xml 配置 <id> 对应  -->
        <repository>
            <id>maven-releases</id>
            <url>http://192.168.8.70:8081/repository/maven-releases/</url>
        </repository>
        <snapshotRepository>
            <id>maven-snapshots</id>
            <url>http://192.168.8.70:8081/repository/maven-snapshots/</url>
        </snapshotRepository>
    </distributionManagement>


​	
maven上传快照到私库 jar带时间戳 https://www.cnblogs.com/qq27271609/p/5497815.html
	
maven 打包时强制更新本地包 
mvn clean install -e -U