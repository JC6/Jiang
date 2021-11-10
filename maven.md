# Maven 部署与使用
## 部署 Maven

1.安装 JDK 1.8
* 其他版本运行 Nexus 会报错：The version of the JVM must be 1.8.

2.下载 [nexus repository oss](https://www.sonatype.com/products/repository-oss)

3.启动 Nexus

```bash
./bin/nexus start
```

4.浏览器打开 http://127.0.0.1:8081 ，设置管理员用户名密码

## 上传 library

1. library 的 build.gradle 中添加 

```groovy
apply plugin: 'maven'

uploadArchives {
    repositories {
        mavenDeployer {
            repository(url: "http://127.0.0.1:8081/repository/maven-releases/") {
                authentication(userName: "username", password: "password")
            }

            pom.groupId = 'com.example.demo'
            pom.artifactId = 'demo'
            pom.version = '1.0.0'
        }
    }
}
```

2.编译后运行 uploadArchives 上传到 maven 仓库

## 使用 library

1.项目的 build.gradle 中添加

```groovy
allprojects {
  repositories {
    maven { url "http://127.0.0.1:8081/repository/maven-releases/" }
  }
}
```

2.app 的 build.gradle 中添加

```groovy
implementation 'com.example.demo:demo:1.0.0'
```
