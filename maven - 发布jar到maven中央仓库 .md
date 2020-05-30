# Gtihub
1、注册登录GitHub
```
https://github.com/chaozhouzhang
```
2、创建项目
```
https://github.com/chaozhouzhang/common
```
3、克隆到本地
```
https://github.com/chaozhouzhang/common.git
```

# Sonatype
1、注册登录sonatype
```
https://issues.sonatype.org/secure/Dashboard.jspa
```
2、创建issue
```
Project：Community Support - Open Source Project Repository Hosting (OSSRH)
Issue Type：New Project
Summary：项目描述
Group Id：com.github.chaozhouzhang
Project URL：https://github.com/chaozhouzhang/common
SCM url：https://github.com/chaozhouzhang/common.git
```
3、等待issue的状态变为resolved
```
https://issues.sonatype.org/browse/OSSRH-44307
```
```
Status:RESOLVED
```

# Intellij Idea
1、使用Intellij Idea 在从GitHub克隆的项目目录中创建项目，并开发源码：
```
File -> New Project -> Maven -> Next -> GroupId -> ArtifactId -> Next -> Next：
```
2、在pom.xml中配置maven
```
    <parent>
        <groupId>org.sonatype.oss</groupId>
        <artifactId>oss-parent</artifactId>
        <version>7</version>
    </parent>
    <distributionManagement>
        <repository>
            <id>releases</id>
            <name>releases Repository</name>
            <url>https://oss.sonatype.org/content/repositories/releases</url>
        </repository>
        <snapshotRepository>
            <id>snapshots</id>
            <name>snapshots Repository</name>
            <url>https://oss.sonatype.org/content/repositories/snapshots</url>
        </snapshotRepository>
    </distributionManagement>
```

3、打开/Users/zhangchaozhou/.m2/settings.xml，添加配置
```
    <servers>
        <server>
            <id>releases</id>
            <username>chaozhouzhang</username>
            <password>Teochew_121518</password>
        </server>
        <server>
            <id>snapshots</id>
            <username>chaozhouzhang</username>
            <password>Teochew_121518</password>
        </server>
    </servers>
```
4、在idea的终端中创建密钥，并记下其中的gpg.passphrase
```
gpg --gen-key
```
5、获取pub值
```
gpg --list-keys
```
6、将pub值上传到服务器
```
gpg --keyserver hkp://keyserver.ubuntu.com:11371 --send-keys pub 此处填写pub值
```
7、上传到maven中央仓库
```
mvn clean deploy -P sonatype-oss-release -Darguments="gpg.passphrase=此处填写之前记下的gpg.passphrase"
```
# Sonatype
打开后登录
```
https://oss.sonatype.org/#stagingRepositories
```
搜索你的jar包关键字，获得maven加载方式
```
<dependency>
  <groupId>com.github.chaozhouzhang</groupId>
  <artifactId>common</artifactId>
  <version>1.0-SNAPSHOT</version>
</dependency>
```

