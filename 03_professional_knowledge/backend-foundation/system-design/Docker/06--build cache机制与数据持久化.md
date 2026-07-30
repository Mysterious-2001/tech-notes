# build cache

为了理解镜像构建过程中的build cache机制，需要先搭建一个测试环境。

## 搭建测试环境

1. 在/root/cache 目录下新建hello.log文件
    
2. 在/root/cache 目录下新建Dockerfile文件，内容如下：
    
    ```dockerfile
    FROM centos:7
    LABEL auth="TOM"
    COPY hello.log /var/log
    RUN yum -y install vim
    CMD /bin/bash
    ```
    
3. 第一次构建镜像
    
    ```sh
    docker build -t test:1.0
    ```
    

## 镜像的生成过程

Docker镜像的构建过程，大量应用了镜像间的父子关系。即下层镜像作为上层镜像的父镜像，下层镜像是上层镜像的输入。上层镜像是在下层镜像的基础上变化而来。

上面的镜像构建过程具体如下：

1. **FROM centos：7**
    
    该语句不会产生新的镜像层，他使用指定的镜像作为基础镜像。
    
2. **LABEL auth="Tom"**
    
    LABEL指令仅修改上一步提取出的镜像json文件内容，在json中添加LABEL auth=“TOM”，不更新镜像文件系统。
    
    会生成一个新的镜像层，只不过该镜像层中只记录json文件内容变化。
    
3. **COPY hello.log /var/log**
    
    生成新的镜像层文件系统。产生了新的镜像层。
    
4. **RUN yum -y install vim**
    
    RUN指令本身不会改变镜像层文件系统大小，但是RUN 的是yum install,会下载并安装一个工具，也就改变了镜像层文件系统大小，所以会产生新的镜像层。
    
5. **CMD /bin/bash**
    
    CMD 或ENTRYPOINT 不会改变镜像层文件系统大小，但是会将指令写入到json文件中，改变json文件大小。所以json文件的镜像id也变了，产生了新的镜像层。
    

## 修改Dockerfile后重新构建

1. **修改Dockerfile**
    
    ```dockerfile
    FROM centos:7
    LABEL auth="TOM"
    COPY hello.log /var/log
    RUN yum -y install vim
    CMD /bin/bash
    EXPOSE 9000
    ```
    
2. **构建新镜像**
    
    此时构建过程中发现很多using cache，这就是使用了**build cache**。
    

![无标题](https://img2024.cnblogs.com/blog/3388489/202606/3388489-20260601160334744-1626738280.png)

查看这两个镜像的 docker history，发现除了新加的指令EXPOSE镜像层外，其他层完全相同。这就是test:2.0在构建时复用了test:1.0的镜像层。

![无2标题](https://img2024.cnblogs.com/blog/3388489/202606/3388489-20260601160341429-346880722.png)

## build cache 机制

构建镜像时，当发现新构建的镜像层与本地已存在的镜像层重复时，默认会复用本地的镜像层，这种机制称为build cache。

**该机制加快了镜像的构建过程，节省空间。**

docker build cache 不是占用内存的cache，而是磁盘中对镜像层的检索、复用机制，无论是关闭Docker引擎，还是重启Docker 宿主机，只要本地有镜像层，就会复用。

## build cache 失效情况

1. **Dockerfile 文件发生变化**
    
    当Dockerfile文件中某个指令内容发生变化，那么变化的这个指令层开始所有的镜像层cache全部失效。
    
    镜像关系本质上是一种树状关系，只要上层节点变了，那么下层节点也就全部变化了。
    
2. **ADD或COPY指令内容变化**
    
    Dockerfile文件内容没变化，但是ADD或COPY指令所复制的文件内容变化了，也会导致build cache失效。
    
3. **RUN指令外部依赖变化**
    
    Dockerfile文件内容没变化，但是RUN命令外部依赖变了，比如要安装 的VIM软件源变了（版本变化、下载地址变化等。）
    
4. **指定不使用build cache**
    
    通过指定--no-cache 选择不使用build cache
    
    ```sh
    docker build --no-cache -t test:11.0
    ```
    

# 数据持久化

在容器层的UnionFS（联合文件系统）中对文件/目录的任何修改，在容器丢失或被删除后，这些修改将全部丢失。若要保存这些修改，一般有两种方式：

- 定制镜像持久化：将这个修改过的容器生成一个新镜像。
- 数据卷持久化：将这些修改通过数据卷同步到宿主机。

## 定制镜像持久化

1. 以分离模式运行tomcat容器
    
    ```sh
    docker run --name mytom -dp 8081:8080 tomcat:10.0
    ```
    
    此时浏览器无法访问到tomcat页面，因为webapps目录是空的。
    
2. 进入容器后删除目录
    
    ```sh
    docker exec -it mytom /bin/bash
    # 删除webapps
    rm -rf webapps
    # 将webapps.list 重命名为webapps
    mv webapps.list webapps
    ```
    
    此时浏览器就能访问了。
    
3. 容器生成镜像
    
    ```sh
    docker commit -m="modify webapps" -a="jerry" mytom tomcat10:own
    ```
    

此时的新镜像就永久保存了我们的修改。这种方式不适合变更频繁的场景。

## 数据卷持久化

DOcker提供了三种实时同步（宿主机与容器间数据同步）方式：

- 数据卷
- Bind mounts（绑定挂载）
- tmpfs（临时文件系统）

### 数据卷介绍

数据卷是**宿主机**中的一个特殊的文件/目录,这个文件/目录与容器中的另一个文件/目录进行了关联，在任何一端对文件/目录进行写操作，另一端都会同时改变。

宿主机中的这个文件/目录称为数据卷；容器中的这个关联文件/目录称为数据卷在该容器中的挂载点。

数据卷完全独立于容器的生命周期，属于宿主机文件系统，当容器被删除时，不会删除其挂载的数据卷。

### 数据卷特性

- 数据卷在容器启动时初始化，如果容器启动后容器本身包含了数据，那么数据在容器启动后直接出现在数据卷中，反之依然
- 可以对数据卷或挂载点的内容直接修改，修改后另一端可立即生效
- 数据卷一直存在，即使容器被删除
- 数据卷可以在容器之间共享和复用

### 创建读写数据卷

读写数据卷指 容器对挂载点有读写权限。

数据卷是在启动容器时指定，语法格式为：

```sh
docker run -it -v /宿主机目录绝对路径:/容器内目录绝对路径 镜像
```

如果指定的目录不存在会自动创建。比如：

```sh
# 创建数据卷
docker run --name myubuntu -it -v /root/host_mount:/opt/uc_mount ubuntu:latest /bin/bash
# 在宿主机数据卷创建一个新文件,在容器中就可以看到了
echo "host created file" > hello.log
```

即使容器停止，数据仍会保存，当容器重启后，再进入容器就可以查看。

```sh
# 退出容器
exit
# 重启容器
docker start myubuntu
# 进入容器
docker exec -it myubuntu /bin/bash
# 查看挂载点
cat /opt/uc_mount/hello.log
```

查看数据卷详情命令：docker inspect 【容器】

![无3标题](https://img2024.cnblogs.com/blog/3388489/202606/3388489-20260601160355818-1813631241.png)

### 创建只读数据卷

只读数据卷指的是容器对挂载点只读，宿主机对数据卷始终是读写。

这样是为了防止容器在运行过程中对文件进行修改。命令如下：

```sh
docker run -it -v /宿主机目录绝对路径:/容器内目录绝对路径:ro 镜像
# 实例如下：
# 创建数据卷
docker run --name myubuntu -it -v /root/host_mount:/opt/uc_mount:ro ubuntu:latest /bin/bash
```

### 数据卷共享

当一个容器和另一个容器使用相同的数据卷时，这两个容器就实现了“数据卷共享”。

当一个容器C启动时创建并挂载了数据卷，而其他容器也要共享这个数据卷，那么这些容器只需要在docker run时通过--volumes-from[容器C] 选项 就能实现数据卷共享。此时容器C称为数据卷容器。

加入数据卷容器的名称为myubuntu。那么另一个容器创建如下：

```sh
docker run --name myubuntu2 --volumes-from myubuntu -it ubuntu:latest /bin/bash
```

此时就完成了两个容器的数据卷共享。

# Dockerfile 持久化

Dockerfile 持久化是通过VOLUME指令指定数据卷的方式实现持久化。

VOLUME 指令可以在容器中创建 数据卷的挂载点。其参数可以是字符串数组，或者使用空格隔开。

比如：VOLUME ["/var/www","/etc/apache"] 或 VOLUME /var/www /etc/apache

## 实现持久化

1. 创建一个Dockerfile,这个容器有2个挂载点：/opt/xxx 和 /opt/ooo
    
    ```dockerfile
    FROM centos:7
    VOLUME /opt/xxx /opt/ooo
    CMD /bin/bash
    ```
    
2. 然后构建镜像
    
    ```sh
    docker build -t volscon .
    ```
    
3. 运行镜像
    
    ```sh
    docker run --name myvols -it volscon
    ```
    
4. 查看数据卷详情
    

![无标4题](https://img2024.cnblogs.com/blog/3388489/202606/3388489-20260601160405936-2078485803.png)

## 两种数据卷持久化方式区别：

docker run -v 与Dockerfile 的 VOLUME 指令方式指定数据卷的区别：

- docker run -v 针对已存在的容器；而VOLUME 指令针对准备创建的镜像。
- docker run -v 可以指定宿主机的数据卷目录；而VOLUME 指令无法指定数据卷目录。