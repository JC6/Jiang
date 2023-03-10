# Docker

* 介绍
    * Docker 是一个开放源代码的开放平台软件，用于开发应用、交付应用和运行应用。Docker 容器与虚拟机类似，但二者在原理上不同。
    * 容器是应用程序层的抽象，将代码和依赖项打包在一起。多个容器可以在同一台机器上运行，并与其他容器共享操作系统内核，每个容器在用户空间中作为隔离进程运行。容器占用的空间比虚拟机少，可以处理更多的应用程序，需要更少的虚拟机和操作系统。
    * 虚拟机（VM）是将一台服务器变成多台服务器的物理硬件的抽象。虚拟机管理程序允许多个虚拟机在一台机器上运行。每个虚拟机都包含操作系统、应用程序、必要的二进制文件和库的完整副本。虚拟机的启动速度也很慢。

* Linux 上的 Docker 直接运行在 Linux 环境中，macOS 上的 Docker 运行在 Docker Desktop for Mac 提供的 Linux VM 环境中
    * 因此 Docker Desktop for Mac 需要始终处于打开状态，也不能直接安装无 GUI 的 Docker Engine，除非自行安装虚拟机

* docker build
    * 通过 Dockerfile 构建镜像
    * docker build -t 镜像名:tag Dockerfile所在目录
    * tag 控制版本，不填写默认 latest

* docker run
    * 直接运行一个 docker 容器
    * 适合运行 cli 工具，一次性使用，用完释放，如构建 Android，Flutter 项目的 apk 时所用的容器
    * 示例
      ```shell
      docker run \
         --rm \
         -v ~/.gradle:/root/.gradle \
         -v $PWD:/app \
         -w /app \
         android bash gradlew clean :app:assembleRelease
      ```
        * --rm，执行完删除容器，除通过 -v 映射到宿主机的文件，其他文件都会释放
        * -v，将宿主机的目录（或文件）和容器内的目录（或文件）建立映射关系，使得容器内可以访问宿主机的目录（或文件）
            * 用于映射执行代码时需映射到工作目录，或通过 -w 手动指定工作目录
        * -w，手动指定工作目录，相当于执行命令前 cd 到工作目录
        * android，镜像名，默认 tag 为 latest
        * bash gradlew clean :app:assembleRelease，容器运行后执行的命令

* docker compose
    * 多个 docker run 可以合并为一个 compose 方便部署
      ```shell
      docker compose up -d
      ```
    * compose 结构复杂后也可以另行拆分为多个 compose 文件
    * 同一个 compose 下的服务默认在同一个 docker network 里，内部不需要映射端口号，可以通过服务名 + 容器内端口号直接访问内部服务
    * 适合运行服务，如 Jenkins，SonarQube，Nexus 等

## 如何编写一个 Dockerfile（以 Flutter 为例）

1. 先构建一个 Android 镜像，既可用于构建 Android 项目，也可用作 Flutter 镜像的基础镜像
   ```dockerfile
   # 基础镜像根据项目需要，选择带有指定 Java 版本的 Ubuntu 系统
   FROM eclipse-temurin:11
   
   # 设置环境变量
   ENV ANDROID_HOME=/opt/android/sdk
   # RUN/COPY 尽量写在一条语句中，Docker 采用分层设计，每执行一句 RUN/COPY 会形成一个新的 layer，增加最终的镜像大小
   # 每个 layer 会产生一个 cache，在需要缓存的时候，可分多句编写，如 RUN pip install -r requirements.txt 等
   RUN set -eux; \
       apt-get update; \
       apt-get install -y unzip; \
       wget -O commandlinetools-linux.zip https://dl.google.com/android/repository/commandlinetools-linux-9477386_latest.zip; \
       unzip commandlinetools-linux.zip;\
       rm commandlinetools-linux.zip; \
       mkdir -p $ANDROID_HOME/cmdline-tools; \
       mv cmdline-tools $ANDROID_HOME/cmdline-tools/latest; \
       yes | $ANDROID_HOME/cmdline-tools/latest/bin/sdkmanager --licenses; \
       $ANDROID_HOME/cmdline-tools/latest/bin/sdkmanager "platforms;android-33"; \
       rm -rf /var/lib/apt/lists/*
   ```
2. 基于 Android 镜像构建一个 Flutter 镜像
   ```dockerfile
   FROM android
   
   ARG FLUTTER_VERSION
   
   WORKDIR /opt
   
   ENV PATH=$PATH:/opt/flutter/bin \
      PUB_HOSTED_URL=https://pub.flutter-io.cn \
      FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn
   
   RUN set -eux; \
      apt-get update; \
      apt-get install -y file git xz-utils zip; \
      wget https://storage.flutter-io.cn/flutter_infra_release/releases/stable/linux/flutter_linux_$FLUTTER_VERSION-stable.tar.xz; \
      tar xf flutter_linux_$FLUTTER_VERSION-stable.tar.xz; \
      rm flutter_linux_$FLUTTER_VERSION-stable.tar.xz; \
      chown -R root:root flutter; \
      flutter config --no-analytics; \
      rm -rf /var/lib/apt/lists/*
   ```
3. 编写 Dockerfile 的过程本质上和在 Linux 服务器上配置环境过程一致，比如构建 Flutter 镜像，只需阅读 Flutter
   官方文档中的[在 Linux 操作系统上安装和配置 Flutter 开发环境](https://flutter.cn/docs/get-started/install/linux)
   ，按配置流程编写 Dockerfile 即可

### 端口号

1. 自定义端口号避免使用小于1024的端口号，除非设计如此（如80，443等）
2. 避免与其他服务端口号冲突，自定义端口号前可使用相关命令查看主机上当前监听的所有端口号
   ```shell
   lsof -i -P | grep LISTEN
   ```

### uid, gid

* 整个系统(Docker 容器内的系统和宿主机)共享同一个内核，而内核只管理一套 uid 和 gid
* 在 macOS 上，Docker 容器内的系统和 Docker Desktop 提供的 Linux VM 共享同一个内核，真正的主机上的 macOS 不受影响
* 编写仅在 Linux 上使用的 Dockerfile，需要考虑容器默认使用的 root 用户是否对宿主机有影响

## 构建

1. 构建 Android 镜像
   ```shell
   docker build -t android .
   ```

2. 构建 Flutter 镜像
   ```shell
   docker build -t flutter --build-arg FLUTTER_VERSION=3.7.6 .
   ```

## 运行

1. Android
   ```shell
   docker run \
      --rm \
      -v ~/.gradle:/root/.gradle \
      -v $PWD:/app \
      -w /app \
      android bash gradlew clean :app:assembleRelease
   ```

2. Flutter
   ```shell
   docker run \
      --rm \
      -v $PWD:/app \
      -w /app \
      flutter flutter build apk
   ```
