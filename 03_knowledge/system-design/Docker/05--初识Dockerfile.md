# Dockerfile

Dockerfile是构建Docker镜像的脚本文件，由一系列指令构成。

通过docker build 命令构建镜像时，Dockerfile 中的指令由上到下依次执行，每条指令都会构建出一个镜像（就是镜像的分层）。因此指令越多，层次就越多，创建的镜像就越多，效率就越低。

所以在定义Dockerfile时，能在一个指令完成的动作就不要分为两条。

## 指令简介

- 指令不区分大小写，一般惯例是写为全大写
- 指令后至少会携带一个参数
- 注释以# 开头

```dockerfile
# FORM 指令，用于指定基础镜像。且必须是第一条指令；若省略了tag，则默认为latest
FORM <image>[:<tag>]
# MAINTAINER 指令的参数填写的一般是维护者的姓名和邮箱。此指令官方不建议使用，使用LABEL指令代替
MAINTAINER <name>
#LABEL 指令：以键值对的形式包含任意镜像的元数据信息，用来代替MAINTAINER指令。通过docker inspect 可查看 LABEL 和MAINTAINER 的内容
LABEL <key>=<value> <key>=<value> <key>=<value> ......
# ENV指令：指定环境变量，这些环境变量后续可以被RUN指令使用，容器运行后，也可以在容器中获取这些环境变量
ENV <key> <value>
ENV <key>=<value> <key>=<value> <key>=<value> ......
#WORKDIR指令：容器打开后默认进入的目录，一般在后续的RUN、CMD、ENTRYINT、ADD等指令中会引用该目录。
# 可以设置多个WORKDIR 指令，后续WORKDIR指令若使用相对路径，则会基于之前WORKDIR指令的路径。
# 使用docker run 运行容器时，加-w 参数可以覆盖 上面设置的工作目录。
WORKDIR path
# RUN <COMMAND>：这里的<COMMAND> 时shell命令，在docker build 过程中，使用shell 运行指定的command
RUN <COMMAND>
# 在docker build 过程中，会调用第一个参数“EXECUTABLE” 指定的程序运行，并使用后面的 参数作为运行参数
RUN ["EXECUTABLE","PARAM1","PARAM2"...]
# 在容器启动后，即执行完docker run 后立即执行“EXECUTABLE”指定的可执行文件，并将后面的参数作为运行参数
CMD ["EXECUTABLE","PARAM1","PARAM2"...]
# command表示shell 命令，在容器启动后立即运行指定的shell命令
CMD command param1 param2 ...
#提供给ENTRYPOINT 的默认参数
CMD ["PARAM1","[PARAM2]",...]
# 在容器启动过程中，即执行完docker run 时，会执行“EXECUTABLE”指定的可执行文件，并将后面的参数作为运行参数
ENTRYPOINT ["EXECUTABLE","PARAM1","PARAM2"...]
# command表示shell 命令，在容器启动过程中，会运行指定的shell命令
ENTRYPOINT command param1 param2 ...
#
EXPOSE <port> [<port>...]
#
ARG <varname>[=<default value>]
# 复制宿主机的文件（src）到容器中的指定目录（dest）
# src可以是绝对路径，也可以是相对路径（相对路径是相对于docker build 命令所指定的路径的）
# src可以是一个压缩文件，复制到容器后自动解压
# src也可以是一个url。这时ADD指令相当于wget命令
# src 最好不要是目录，不然会将目录中的所有内容复制到容器中
# dest 是一个绝对路径，其最后的路径必须要加上斜杠，不然会将最后的目录名称当作是文件名
ADD <src> <dest>
ADD ["<src>","<dest>"] #路径中存在空格时使用双引号引起来
# 功能和ADD相同，但是src不能是url，若src是压缩未见，复制到容器后不能自动解压
COPY
# 指定当前镜像的子镜像进行构建时要执行的指令
ONBUILD [INSTRUCTION]
# 在容器创建可以挂载的数据卷
VOLUME ["dir1","dir2",...]
```

## 指令用法（构建自己的HelloWorld镜像）

1. **scratch 镜像**
    
    scratch 镜像是一个空镜像，是所有镜像的Base Image。
    
    scratch 镜像只能在Dockerfile 中被继承，不能pull拉取，不能run，没有tag，并且他不会生成镜像中的文件系统层。
    
    scratch 是一个保留字，用户不能作为自己镜像的名称。
    
2. **安装编译器**
    
    ```sh
    # 由于要编写C语言。所以要安装C语言的编译器
    yum install -y gcc gcc-c++
    yum install -y glibc-static
    ```
    
3. **创建hello.c**
    
    在宿主机创建一个hello.c 文件
    
    ```c
    #includ<stdio.h>
    int main()
    {
        printf("hello my docker world\n");
        return 0;
    }
    ```
    
4. **编译测试hello.c**
    
    ```sh
    gcc --statuc -o hello hello.c
    ./hello
    ```
    
5. **创建Dockerfile**
    
    在当前目录下新建Dockerfile （vim Dockerfile ，文件名就叫Dockerfile ），内容如下：
    
    ```dockerfile
    FROM scratch
    ADD hello /
    CMD ["/hello"]
    ```
    
6. **构建镜像**
    
    ```sh
    docker build -t hello-my-world .
    ```
    
    - -t :指定要生成镜像的`<repositort>和<tag>`。若省略tag，则默认latest
    - 最后的【.】：是一个宿主机的URL路径。构建镜像时会从该路径中查找Dockerfile文件。
    
    此时执行docker images 可以看到自己创建的镜像。
    
    然后可以运行这个镜像了：docker run hello-my-world
    
7. **为经常重新打标签**
    
    当进行过出新版本了，上个版本的镜像标签就不能是tag了。
    
    docker tag 命令 可以对镜像重打标签。重打标签实际是复制了一份镜像，将新镜像指定新的tag。新镜像的IMageID和Digest 和原镜像的相同。
    
    ```sh
    docker tag hello-my-world hello-my-world:1.0
    ```
    

## CMD与ENTRYPOINT

这两个指令都是用于指定容器启动时要执行的命令，无论哪个指令，每个Dockerfile中只能有一个CMD/ENTRYPOINT 指令，**多个指令只会执行最后一个。**

不同点如下：

CMD指定的是容器启动时默认的命令，即docker run 若指定了要运行的命令，则Dockerfile 中的CMD指定的命令时不会执行的。

而ENTRYPOINT 指定的是容器启动时一定会执行的命令。

### CMD 例子

1. 创建Dockerfile，在dfs目录中新建文件Dockerfile2,其内容如下：
    
    ```dockerfile
    FROM centos:7
    CMD cal
    ```
    
    然后构建镜像
    
    ```sh
    docker build -f ./Dockerfile2 -t mycal .
    ```
    
    -f :指定构建时要使用的Dockerfile文件（当文件名称不是默认的Dockerfile时才需要指定）
    
    然后运行就会显示当前月份的日历
    
    ```sh
    docker run -it mycal
    ```
    
    此时若在docker run 时指定要执行的命令，则Dockerfile中CMD 指定的命令就不执行了
    

![无标题](https://img2024.cnblogs.com/blog/3388489/202606/3388489-20260601125331900-1720297483.png)

此时也无法为CMD指定的命令设置选项

![无2标题](https://img2024.cnblogs.com/blog/3388489/202606/3388489-20260601125339457-1167841769.png)

如果说Dockerfile 的内容如下（CMD指令的另一种形式），也是一样的效果，

```dockerfile
FROM centos:7
CMD ["/bin/bash","-c","cal"]
```

### ENTRYPOINT例子

1.创建Dockerfile，名字为Dockerfile4，其内容如下：

```dockerfile
FROM centos:7
ENTRYPOINT cal
```

然后构建并运行，结果显示本月的日历

如果docker run 时指定了执行命令，ENTRYPOINT的指令依然会执行，不会被覆盖

![无标3题](https://img2024.cnblogs.com/blog/3388489/202606/3388489-20260601125347040-1334197598.png)

此时，即使在docker run中添加命令选项，也是无效的。

如果Dockerfile 的内容如下，那么在docker run中添加命令选项 是有效的。

```dockerfile
FROM centos:7
ENTRYPOINT ["cal"]
```

![屏幕4截图_29-5-2026_162930_](https://img2024.cnblogs.com/blog/3388489/202606/3388489-20260601125355792-85829496.jpg)

### 总结

- CMD指定命令：启动命令docker run 中不能添加参数【arg】。
    
    因为Dockerfile 中的CMD可以被替代，如果启动的镜像 后仍有内容，对于docker daemon来说 ，会认为是一个命令，如果有两个或两个以上，后面才会认为是参数。
    
- ENTRYPOINT指令：启动命令docker run 中可以添加参数【arg】。
    
    因为ENTRYPOINT 不能被替代，如果启动的镜像 后仍有内容，对于docker daemon来说，其只能是【arg】。
    

不过，docker daemon 对于ENTRYPOINT 指定的【command】和【“EXECUTABLE”】的处理方式不同：

- 【command】指定的是shell：daemon会直接运行。而不会与docker run 后的【arg】拼接，
- 【“EXECUTABLE”】指定的命令，daemon会先与docker run 后的【arg】拼接，然后运行拼接后的结果。

**结论：无论CMD还是ENTRYPOINT ，使用[“EXECUTABLE”]的方式的通用性会更强。**

## ADD与COPY指令

创建一个Dockerfile，其内容如下：

```dockerfile
FROM centos:7
WORKDIR /opt
ADD zookeeper.tar.gz /opt/add
COPY zookeeper.tar.gz /opt/copy
CMD /bin/bash
```

构建并运行镜像后会发现：

**ADD指令添加的是解压后的目录**

**COPY指令添加的是未解压的。**

## ARG指令

该指令用于定义一个变量，**该变量会在镜像构建时使用，而不是容器启动时**。

创建一个Dockerfile，其内容如下：

```dockerfile
FROM centos:7
ARG name=TOM
RUN echo $name
```

RUN指令用于指定在docker build时要执行的内容

### 使用ARG默认值构建

```sh
docker build -t myargs:1.0 .
```

构建时没给变量name赋予新值，所以name还是=TOM

### 使用ARG指定值构建

```sh
docker build -t myargs:1.0 --build-arg name=jerry .
```

构建时给变量name赋予新值，所以name=jerry

## ONBUILD指令

ONBUILD指令 只对**当前镜像的子镜像进行构建时有效**。

比如下面实现的是：父镜像中没有wget命令，但是子镜像会增加。

- 父镜像的Dockerfile
    
    ```dockerfile
    FROM centos:7
    ENV WORKPATH /usr/local
    WORKDIR $WORKPATH
    ONBUILD RUN yum -y install wget
    CMD /bin/bash
    ```
    
    当前镜像及其子镜像的工作目录都是/usr/local.
    
    子镜像在进行docker build 时会运行RUN的安装命令。
    
    构建父镜像
    
    ```sh
    docker build -t parent:1.0
    ```
    
- 子镜像的Dockerfile
    
    ```dockerfile
    FROM parent:1.0
    ```
    
    构建并运行子镜像后，就可以使用wget指令了。
    

## 构建新镜像的方式总结

- docker build
- docker commit
- docker import （注意：docker load并没有构建出新镜像，其与原镜像是同一个镜像）
- docker compose
- docker hub 中完成Automated Builds

# 将Springboot项目部署到Docker

## 准备应用

新建一个名称为hello-docker的简单SpringBoot项目。

1. POM文件内容如下：
    
    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    	<modelVersion>4.0.0</modelVersion>
    	<parent>
    		<groupId>org.springframework.boot</groupId>
    		<artifactId>spring-boot-starter-parent</artifactId>
    		<version>4.0.6</version>
    		<relativePath/> <!-- lookup parent from repository -->
    	</parent>
    	<groupId>com.ali</groupId>
    	<artifactId>hello-docker</artifactId>
    	<version>0.0.1-SNAPSHOT</version>
    	<name>hello-docker</name>
    	<description>hello-docker</description>
    	<url/>
    	<licenses>
    		<license/>
    	</licenses>
    	<developers>
    		<developer/>
    	</developers>
    	<scm>
    		<connection/>
    		<developerConnection/>
    		<tag/>
    		<url/>
    	</scm>
    	<properties>
    		<java.version>17</java.version>
    	</properties>
    	<dependencies>
    		<dependency>
    			<groupId>org.springframework.boot</groupId>
    			<artifactId>spring-boot-starter-web</artifactId>
    		</dependency>
    
    		<dependency>
    			<groupId>org.springframework.boot</groupId>
    			<artifactId>spring-boot-starter-test</artifactId>
    			<scope>test</scope>
    		</dependency>
    	</dependencies>
    
    	<build>
    		<plugins>
    			<plugin>
    				<groupId>org.springframework.boot</groupId>
    				<artifactId>spring-boot-maven-plugin</artifactId>
    			</plugin>
    		</plugins>
    	</build>
    
    </project>
    
    ```
    
    其配置文件application.yml内容如下：
    
    ```yaml
    logging:
      pattern:
        console: level-%-5level - %msg%n
    ```
    
    Controller类如下：
    
    ```java
    package com.ali.hellodocker.controller;
    
    import org.springframework.web.bind.annotation.GetMapping;
    import org.springframework.web.bind.annotation.RestController;
    
    @RestController
    public class SomeController {
    
        @GetMapping("/hello")
        public String hello() {
            System.out.println("================hello========================");
            return "Hello Docker!";
        }
    }
    
    ```
    
    然后打成jar包
    

## 发布应用

1. 在宿主机上创建一个专门的目录，用来存放jar包和Dockerfile 等内容。
    
    ```sh
    mkdir /root/hello-docker
    ```
    
    然后将jar包上传到这个目录
    
2. 创建Dockerfile
    
    ```dockerfile
    FROM swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/openjdk:17
    MAINTAINER zhangsan zs@163.com
    LABEL version="1.0" description="my own app"
    COPY hello-docker-0.0.1-SNAPSHOT.jar hd.jar
    ENTRYPOINT ["java","-jar","hd.jar"]
    EXPOSE 9000
    ```
    
3. 然后构建并运行镜像
    
    ```sh
    docker build -t hello-docker:1.0 .
    docker run --name myhd -dp 9000:8080 hello-docker:1.0
    ```
    
4. 这时就可以通过ip:9000/hello访问了
    

![无标4题](https://img2024.cnblogs.com/blog/3388489/202606/3388489-20260601125415204-337342941.png)

5. 可以通过docker log命令查看输出日志

![无标5题](https://img2024.cnblogs.com/blog/3388489/202606/3388489-20260601125422266-1763323786.png)