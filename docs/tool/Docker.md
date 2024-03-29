# Docker

## 安装

1. 卸载旧版本

   ```bash
   yum remove docker \
                     docker-client \
                     docker-client-latest \
                     docker-common \
                     docker-latest \
                     docker-latest-logrotate \
                     docker-logrotate \
                     docker-engine
   ```

2. 下载utils安装包

   ```bash
    yum install -y yum-utils
   ```

3. 设置镜像仓库地址

   ```bash
      yum-config-manager \
       --add-repo \
       http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
   ```

4. 更新软件包索引

   ```bash
   yum makecache fast
   ```

5. 安装docker相关的引擎

   注：docker-ce 社区版 ee企业版

   ```bash
   yum install docker-ce docker-ce-cli containerd.io
   ```

## 启动测试

1. 启动docker

   ```bash
   systemctl start docker
   ```

2. 查看docker的版本

   ```bash
   docker version
   ```

3. hello world测试

   ```bash
   docker run hello-world
   ```

4. 设置开机启动

   ```bash
   systemctl enable docker
   ```

   

## docker内服务安装实例
<a href="#tool/docker_service/elasticsearch.md" style="font-size: 20px; color: #4682B4; text-decoration: none">ElasticSearch</a>





## 常用命令

```bash
# 查看正在运行的容器
docker ps

# 查看所有镜像
docker images

# 停止容器运行
docker stop <容器id>

# 重启容器
docker restart <容器id>

# 查看docker版本
docker version

# 查看docker的系统信息,包括镜像和容器的数量
docker info

# 查看docker的帮助命令
docker --help

# 查看容器的实时日志
docker logs -f <容器的id或者name>

# 查看特定范围的日志（例如最近100行）
docker logs --tail 100 <容器的id或name>

# 查看特定时间戳之后的日志
docker logs --since "1 hour ago" <容器id或name>

# 保存日志到指定目录文件
docker logs <容器的id或name> > /home/project/projectOne/logs.txt
```



## 实例：构建一个jar包

1. 将jar包打包后，放到服务器目录中

2. 编写 **DockerFile** 文件

   1.  **DockerFile** 文件示例

      ```bash
      # 指定基础镜像为包含Java运行环境的镜像
      FROM openjdk:8-jdk
      # 将jar包添加到容器中
      ADD hualao-1.0.jar hualao-1.0.jar
      # 暴露应用程序运行的端口
      EXPOSE 9999
      # 设置容器启动时执行的命令
      CMD ["java", "-jar", "hualao-1.0.jar"]
      ```

   2.  **DockerFile** 写法说明。一般情况下，只使用部分命令

      ```bash
      # 基础镜镜像,—切从这里开始构建
      FROM
      # 镜像是谁写的，姓名+邮箱
      MAINTAINER
      # 镜像构建的时候需要运行的命令
      RUN
      # 将jar包添加到容器中
      ADD
      # 镜像的工作目录
      WORKDIR
      # 挂载的目录
      VOLUME
      # 暴露应用程序运行的端口
      EXPOSE
      # 设置容器启动时执行的命令,只有最后一个会生效，可被替代
      CMD
      # 设置容器启动时执行的命令,可以追加命令
      ENTRYPOINT
      # 当构建一个被继承DockerFile这个时候就会运行ONBUILD的指令。触发指令。
      ONBUILD
      # 类似ADD，将我们文件拷贝到镜像中
      COPY
      # 构建的时候设置的环境变量
      ENV
      ```

   

3. 构建镜像

   1. 直接在 **dockerfile** 文件所在目录中，运行以下命令

      ```bash
      docker build -t <自定义的镜像名称> .
      ```

   2. 在其他目录中，或者直接运行报错时，使用指定 **dockerfile目录** 的方式构建镜像

      ```bash
      docker build -f /home/project/DockerFile -t <自定义的镜像名称> .
      ```

4. 启动镜像

   将镜像指定端口进行运行

   ```
   docker run -d -p 80:8080 <镜像名称>
   ```

   **`-p`参数**：在运行`docker run`命令时，使用`-p`参数  `宿主机端口：容器端口`来映射端口。例如，应用程序在容器内部监听8080端口，希望在宿主机的80端口映射到它，可以使用`-p 80:8080`。



## 删除资源

#### 删除指定id的镜像

```bash
docker rmi <镜像id>
```

在删除镜像的时候，有可能会出现 **Error response from daemon: conflict: unable ...**  错误，原因是镜像有个依赖，需要先删除依赖再删除镜像

使用 `docker rm <依赖id>` 删除依赖，再使用 `docker rmi <镜像id>` 删除镜像

#### 查看磁盘占用资源

```bash
docker system df
```

#### 清除未使用的镜像

```bash
docker image prune
```

#### 清除没有标签的镜像

```bash
docker image prune -a
```

#### 清除未使用的容器

```bash
docker system prune
```