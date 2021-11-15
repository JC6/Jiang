## 在 CentOS 7 上安装 Nexus Repository OSS

1.安装 JDK 1.8

```bash
yum install java-1.8.0-openjdk-devel.x86_64
```

其他版本运行 Nexus 会报错：The version of the JVM must be 1.8.

2.下载 [nexus repository oss](https://www.sonatype.com/products/repository-oss)

```bash
wget https://download.sonatype.com/nexus/3/latest-unix.tar.gz
```

或拷贝本机文件

```bash
scp -P port latest-unix.tar.gz remote_username@remote_ip:remote_folder
```

3.解压

```bash
tar -xvzf latest-unix.tar.gz
```

4.启动

```bash
./nexus-3/bin/nexus start
```

5.登录 http://remote_ip:8081/ ，设置用户名密码
