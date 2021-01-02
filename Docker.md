### 什么是Docker

Docker是为开发人员和系统管理员提供容器构建, 运行和共享的应用程序平台, 使用容器来部署应用程序称为容器化, 容器不是新的, 但是应用轻松部署应用程序的容器却是新的.

### Docker的使用

#### 镜像

```shell
# 搜索镜像
docker search tomcat
# 查看镜像列表
docker image ls
# 拉取镜像
docker pull tomcat
# 启动镜像(生成容器)
docker run -d --name first_tomcat -p 8080:8080 tomcat
# -d 表示后台运行 并返回容器ID
# -p 将容器的端口映射到宿主机的端口, 第一个8080表示映射到宿主机的端口, 第二个8080表示Tomcat容器的端口
```

#### 容器

```shell
# 查看容器列表
docker container ls
# 进入到容器中
docker exec -it containerid /bin/bash
# 容器内安装软件
apt-get update
apt-get install vim
# 退出容器
exit
# 查看所有容器
docker ps -all
# 查看运行中的容器
docker ps
# 容器的启动, 停止, 重启
docker stop containerID
docker start containerID
docker restart containerID
# 删除容器
docker rm container id
```

### Docker 默认仓库地址修改

* 登录阿里云 找到镜像加速器.

* 执行`vim /etc/docker/daemon.json`, 创建/修改文件, 并设置加速器地址

    ```json
    {
      "registry-mirrors": ["https://089rhvhq.mirror.aliyuncs.com"]
    }
    ```

* 生效并重启

     ```shell
    sudo systemctl daemon-reload // centOS 命令
    sudo systemctl restart docker
    ```

### 镜像与容器

一个软件包通过docker构建之后编程一个image, 这个image在docker Engine上运行就变成了容器.

### 镜像

docker的镜像实际上是由一层一层的文件系统组成, 这种层级的文件系统叫UnionFS(联合文件系统). 它是一种分层, 轻量级且高性能的文件系统, 他支持文件系统的修改作为一次提交来一层一层的叠加.所有镜像是只读的, 不可以修改, 运行生成容器后, 可以在容器内进行修改, 但是所有的修改相当于, 在镜像层上面添加一个可以写的层.所有修改只对此单个容器生效.

关于可写层(容器层)的具体说明

* 添加文件, 在容器中创建文件时, 新文件被添加到容器层中
* 读取文件, 在容器中读取一个文件时, Docker会从上往下一次在个镜像层中查找此文件, 一但找到, 立即将其复制到容器层, 然后打开并读取到内存
* 修改文件, 在容器中修改已存在的文件时, Docker会从上往下一次在各镜像层中查找文件, 一但找到, 立即将其复制到容器层, 然后修改
* 删除文件, 在容器中删除文件时, Docker也是从上往下一次在镜像层中查找此文件, 找到后, 会在容器中记录下次删除记录.

### Dokerfile

打包镜像的基础文件, 记录打包需要的内容

* FROM: 当前镜像的基础镜像, 及此镜像从哪个镜像基础上运行.

    > FROM Ubuntu:14.04

* RUN: 镜像编译时执行的命令 在build过程中, 每一个RUN命令都会新建一个镜像层

* ENV: 设置环境变量, 可以通过${key}的方式来引用, 并且可以通过docker run -e key=value来设置变量的值

* COPY/ADD: 将主机的文件复制到镜像, 如果目录不存在, 会自动创建所需要的目录, 注意只是赋值, 不会提取和解压

    >  COPY docker-entrypoint.sh /usr/local/bin/

    > ADD 与 COPY 作用类似, 只是ADD会对压缩文件解压

* WORKDIR: 指定镜像的工作目录, 如果不存在就创建, 只有的命令基于此目录工作

* CMD: 容器启动后默认执行的命令, 若有多个, 则以最后一个为准

    > CMD ["mysqld"]
    >
    > CMD mysqld

* ENTRYPOINT: 与CMD命令类似, 可以在docker运行的时候默认触发执行的命令

    > 与CMD不同的是:
    >
    > CMD命令, 可以在docker run执行的时候, 在命令行参数中指定运行的程序来覆盖dockerfile中允许的指令
    >
    > ENTRYPOINT不会覆盖执行指令, 而且命令中声明的数据会被当做参数传递给ENTRYPOINT指定的程序.

* EXPOSE: 指定镜像需要暴露的端口, 启动镜像时, 可以使用`-p`将该端口映射给宿主.

    > EXPOSE 3306

### 构建镜像

* 新建一个Dockerfile

``` dockerfile
## 指明当前要构建的镜像所继承的基础镜像 
FROM java:8 
## 设置工作目录 
WORKDIR /app 
## 把Hello.java拷贝到工作目录 
COPY Hello.java /app 
## docker镜像编译时出发的动作，它会在当前镜像上执行指定命令并形成一个新的层 
RUN ["javac","Hello.java"] 
## docker run执行时，会执行这个命令 
ENTRYPOINT ["java","Hello"]
```

* 编写一个Hello.java

    ```java
    public class Hello{
    	public static void main(String[] args){
    		System.out.println("Hello this is laven test progrm for DockerImage");
    		while(true){
    			// 永远执行的函数
    		}
    	}
    }
    ```

* 执行docker构建命令

    ```shell
    docker build -t laven:latest .
    # docker build [options] 
    # -t 表示设置镜像的名字及标签, 一般是name:tag
    # . 表示构建的目录
    ```

构建成功后可以通过`docker image` 查看构建后的镜像

运行`docker run laven: lastest`运行此镜像

运行后可以通过`docker exec -it containerId /bin/bash`进入到容器层

### 挂载Volume

将容器内的目录与本机目录做映射, 以保留响应修改

```shell
docker run -d --name first_tomcat -p 8080:8080 -v /tem/webapps:/usr/local/tomcat/webapps tomcat

// 可以在容器创建后建立挂载点
docker run -it -v ~/Documents/docker/:/datas centos
```

#### Mysql的默认挂载点

> https://github.com/docker-library/mysql/blob/master/5.7/Dockerfile
>
> mysql容器默认的容器内挂载目录: /var/lib/mysql

```shell
# 运行mysql
docker run -d --name mysql01 -e MYSQL_ROOT_PASSWORD=123456 mysql
# 查询所有的挂载
docker volume ls
# 查看具体volume的信息
docker volume inspect [volume_name]

```



