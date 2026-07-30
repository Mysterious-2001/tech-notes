# 安装MySQL

## MySQL官网安装

1. 拉取镜像：docker pull mysql:5.7
    
2. 以分离模式启动MySQL容器
    
    ```sh
    docker run --name mysql -dp 3306:3306 -e MYSQL_ROOT_PASSWORD=111 mysql:5.7
    ```
    
    MYSQL_ROOT_PASSWORD=111: 指定访问密码为111
    
3. 以交互方式进入容器
    
    ```sh
    docker exec -it mysql /bin/bash
    ```
    
4. 在容器内，使用mysql客户端连接mysql服务
    
    ```sh
    mysql -uroot -p
    ```
    
    然后就可以执行sql了。
    
    但是，**这种方式会存在两个问题：字符编码问题和数据安全问题！**
    
5. 当使用navicat连接数据库后，手动插入一条包含中文的记录，会报错，就是字符编码不匹配。
    
    此时需要在容器系统的/etc/mysql/conf.d中新建一个my.cnf文件，在其中指定字符编码！
    
6. 新建的数据库、表都存放在容器系统中的/var/lib/mysql目录中。
    
    运行错误日志存在容器系统的/var/log/mysql 目录中。
    
    此时如果容器被删除，则数据全部丢失！
    

## MySQL生产安装

为了保证数据的安全性，在生产环境下安装MySQL容器，在启动时使用数据卷来持久化数据。

1. 启动MySQL容器
    
    ```sh
    docker run --name mysql -e MYSQL_ROOT_PASSWORD=111 -v /root/mysql/log:/var/log/mysql -v /root/mysql/data:/var/lib/mysql -v /root/mysql/conf:/etc/mysql/conf.d -dp 3306:3306 mysql:5.7
    ```
    
    这里指定了3个数据卷。
    
2. 新建my.cnf
    
    在宿主机的/root/mysql/conf 中新建my.cnf，然后输入以下内容
    
    ```sql
    [client]
    default_character_set=utf8
    
    [mysql]
    default_character_set=utf8
    
    [mysqld]
    character_set_server=utf8
    ```
    
3. 重启MySQL容器（因为修改了配置，所以要重启）
    
    ```sh
    docker restart mysql
    ```
    
4. 进入容器并连接MySQL
    
    ```sh
    docker exec -it mysql /bin/bash
    mysql -uroot -p
    ```
    

## MySQL集群安装（一主一从）

### Master 的安装与配置

1. 启动master容器
    
    ```sh
    docker run --name mysql_master -e MYSQL_ROOT_PASSWORD=111 -v /root/mysql_master/log:/var/log/mysql -v /root/mysql_master/data:/var/lib/mysql -v /root/mysql_master/conf:/etc/mysql/conf.d -dp 3316:3306 mysql:5.7
    ```
    
2. 新建my.cnf
    
    ```sql
    [client]
    default_character_set=utf8
    
    [mysql]
    default_character_set=utf8
    
    [mysqld]
    character_set_server=utf8
    
    server_id=01
    binlog-ignore-db=mysql
    log-bin=master-log-bin 
    binlog_cache_size=1M 
    binlog_format=mixed 
    expire_logs_days=7 
    slave_skip_errors=1062
    ```
    
3. 重启master容器
    
    ```sh
    docker restart mysql_master
    ```
    
4. 进入容器并连接MySQL，创建用户并授权
    
    ```sh
    docker exec -it mysql_master /bin/bash
    mysql -uroot -p
    #'slave'@'%'：用户名 slave，% 代表任意 IP 都能连接（从库机器随便哪台都能连主库）
    create user 'slave'@'%' identified by '123456'
    # 赋主从同步权限
    #replication slave：必备，从库拉取 binlog、主从同步权限
    #replication client：查看 master 状态、binlog 信息权限
    #*.*：全库生效
    grant replication slave,replication client on *.* to 'slave'@'%'
    ```
    
    ### Slave 的安装与配置
    
    1. 打开新的会话窗口，然后启动slave容器
        
        ```sh
        docker run --name mysql_slave -e MYSQL_ROOT_PASSWORD=111 -v /root/mysql_slave/log:/var/log/mysql -v /root/mysql_slave/data:/var/lib/mysql -v /root/mysql_slave/conf:/etc/mysql/conf.d -dp 3326:3306 mysql:5.7
        ```
        
    2. 新建my.cnf
        
        ```sql
        [client] 
        default_character_set=utf8 
        [mysql] 
        default_character_set=utf8 
        [mysqld] 
        character_set_server=utf8 
        
        server_id=02 
        binlog-ignore-db=mysql 
        log-bin=slave-log-bin 
        binlog_cache_size=1M 
        binlog_format=mixed 
        expire_logs_days=7 
        slave_skip_errors=1062 
        relay_log=relay-log-bin 
        log_slave_updates=1 
        read_only=1 
        ```
        
    3. 重启slave容器
        
        ```sh
        docker restart mysql_slave
        ```
        
    
    ### 配置主从复制
    
    4. 查看master状态
        
        在master数据库中执行show master status 命令，查看二进制日志文件名及要开始的位置。
        

![无标题](https://img2024.cnblogs.com/blog/3388489/202606/3388489-20260603150003231-1484910833.png)

2. slave指定master
    
    在slave数据库中运行change master to 命令指定master
    
    ```sql
    change master to master_host='192.168.220.128',master_user='slave',master_password='123456',master_port=3316,mster_log_file='master-log-bin.000002',master_log_pos=154,master_connect_retry=30,master_retry_count=3;
    ```
    
3. 此时slave 与master同步复制还没有开始，需要手动开启
    
    在slave数据库中执行 start slave； 开启slave的数据同步。
    

此时主从复制就配置完成了！master中写操作，slave中就能同步读取数据了。

# Redis单机版安装

1. 拉取·Redis
    
    ```sh
    docker pull redis:7.0
    ```
    
2. 复制一份redis 核心配置文件redis.conf到宿主机目录/root/redis
    
3. 修改redis.conf
    
    ```sh
    # 解除IP绑定
    # bind 127.0.0.1 -::1
    # 关闭保护模式，否则只能本机访问自己
    protected-mode no
    # 关闭守护模式，因为docker本身就是以分离模式运行，如果Redis再以该模式运行，则Redis无法启动
    daemonize no
    # 指定RDB、AOF持久化目录，后面会将这个目录作为挂载点
    dir /data
    ```
    
4. 启动容器
    
    ```sh
    docker run --name myredis -v /root/redis/redis.conf:/etc/redis/redis.conf -v /root/redis/data:/data -dp 6379:6379 
    redis:7.0 
    redis-server /etc/redis/redis.conf
    ```
    
    这里指定两个数据卷，一个是配置文件，一个是目录。
    
    注意：后面运行的命令是redis-server ,且加载的配置文件为挂载点目录/etc/redis 的 redis.conf
    
5. 进入容器连接Redis
    
    ```sh
    # 进入容器
    docker exec -it myredis /bin/bash
    # 连接redis
    redis-cli
    ```
    

# Redis一主两从集群搭建

master端口：6381.slave端口：6392、6383

1. 复制三分redis.conf
    
    仍在/root/redis 目录中完成配置，复制redis.conf并重命名为redis1.conf,并在文件最后添加如下配置，对外保留redis的IP和端口。注意：该IP是宿主机的IP，端口是当前redis对外的端口号。
    
    ```sh
    slave-announce-ip 192.168.220.128
    slave-announce-port 6381
    ```
    
    同理，再复制并修改redis2.conf
    
    ```sh
    slave-announce-ip 192.168.220.128
    slave-announce-port 6382
    ```
    
    同理，再复制并修改redis3.conf
    
    ```sh
    slave-announce-ip 192.168.220.128
    slave-announce-port 6383
    ```
    
2. 启动master
    
    ```sh
    ocker run --name myredis-1 -v /root/redis/redis1.conf:/etc/redis/redis.conf -v /root/redis/data/6381:/data -dp 6381:6379 redis:7.0 redis-server /etc/redis/redis.conf 
    ```
    
3. 启动slave
    
    ```sh
    # 指出slaveof 于谁
    docker run --name myredis-2 -v /root/redis/redis2.conf:/etc/redis/redis.conf -v /root/redis/data/6382:/data -dp 6382:6379 redis:7.0 redis-server /etc/redis/redis.conf --slaveof 192.168.220.128 6381 
    ```
    
    ```sh
    # 指出slaveof 于谁
    docker run --name myredis-3 -v /root/redis/redis3.conf:/etc/redis/redis.conf -v /root/redis/data/6383:/data -dp 6383:6379 redis:7.0 redis-server /etc/redis/redis.conf --slaveof 192.168.220.128 6381 
    ```
    
4. 查看关系
    
    通过info replication 查看他们之间的主从关系已经建立。
    
    ```sh
    docker exec -it myredis-1 redis-cli info replication
    docker exec -it myredis-2 redis-cli info replication
    docker exec -it myredis-3 redis-cli info replication
    ```
    

此时主从集群搭建完成。

# Redis高可用集群搭建

主从集群仍采用前面的，这里搭建三个Sentinel 节点的集群，端口号分别为：26381、26382、26383.

1. 复制三份sentinel.conf
    
    复制sentinel.conf为sentinel1.conf，修改两处
    
    ```sh
    # 指定要监视的master及 <quorum>
    sentinel monitor mymaster 192.168.220.128 6381 2
    #指定对外ip和端口号。ip为docker宿主机的ip，端口号为对外暴露的端口号。
    sentinel announce-ip 192.168.220.128
    sentinel announce-port 25381
    ```
    
    同理，再复制并修改sentinel2.conf
    
    ```sh
    # 指定要监视的master及 <quorum>
    sentinel monitor mymaster 192.168.220.128 6381 2
    #指定对外ip和端口号。ip为docker宿主机的ip，端口号为对外暴露的端口号。
    sentinel announce-ip 192.168.220.128
    sentinel announce-port 26382
    ```
    
    同理，再复制并修改sentinel3.conf
    
    ```sh
    # 指定要监视的master及 <quorum>
    sentinel monitor mymaster 192.168.220.128 6381 2
    #指定对外ip和端口号。ip为docker宿主机的ip，端口号为对外暴露的端口号。
    sentinel announce-ip 192.168.220.128
    sentinel announce-port 26383
    ```
    
2. 启动sentinel
    
    ```sh
    docker run --name mysentinel-1 -v /root/redis/sentinel1.conf:/etc/redis/sentinel.conf -dp 26381:26379 redis:7.0 redis-sentinel /etc/redis/sentinel.conf 
    ```
    
    ```sh
    docker run --name mysentinel-2 -v /root/redis/sentinel2.conf:/etc/redis/sentinel.conf -dp 26382:26379 redis:7.0 redis-sentinel /etc/redis/sentinel.conf 
    ```
    
    ```sh
    docker run --name mysentinel-3 -v /root/redis/sentinel3.conf:/etc/redis/sentinel.conf -dp 26383:26379 redis:7.0 redis-sentinel /etc/redis/sentinel.conf 
    ```
    
3. 关系查看
    
    ```sh
    docker exec -it mysentinel-1 redis-cli -h 192.168.220.128 -p 26381 info sentinel
    ```
    

此时高可用集群就搭建成功了。

# Redis分布式系统搭建

Redis集群的每个节点保存的数据相同。

而Redis分布式系统的节点中保存的数据是不同的。当有数据写入请求时，系统采用虚拟槽分区算法将数据写入到相应节点。

下面搭建一个三主三从的Redis分布式系统

![无1标题](https://img2024.cnblogs.com/blog/3388489/202606/3388489-20260603150019969-962009637.png)

1. 创建/root/cluster目录，将redis.conf 复制到这里
    
2. 复制六份redis.conf
    
    将下面的两个配置的注释去掉。
    
    ```sh
    # 开启cluster功能
    cluster-enabled yes
    #指定需要的配置文件名称
    cluster-config-file nodes-6379.conf
    ```
    
    注意：这6份配置文件内容完全相同。
    
3. 启动6个容器
    
    ```sh
    docker run --name myredis-1 --network host -v /root/cluster/redis1.conf:/etc/redis/redis.conf -v /root/cluster/data/6381:/data -d redis:7.0 redis-server /etc/redis/redis.conf --port 6381 
    
    docker run --name myredis-2 --network host -v /root/cluster/redis2.conf:/etc/redis/redis.conf -v /root/cluster/data/6382:/data -d redis:7.0 redis-server /etc/redis/redis.conf --port 6382 
    
    docker run --name myredis-3 --network host -v /root/cluster/redis3.conf:/etc/redis/redis.conf -v /root/cluster/data/6383:/data -d redis:7.0 redis-server /etc/redis/redis.conf --port 6383 
    
    docker run --name myredis-4 --network host -v /root/cluster/redis4.conf:/etc/redis/redis.conf -v /root/cluster/data/6384:/data -d redis:7.0 redis-server /etc/redis/redis.conf --port 6384 
    
    docker run --name myredis-5 --network host -v /root/cluster/redis5.conf:/etc/redis/redis.conf -v /root/cluster/data/6385:/data -d redis:7.0 redis-server /etc/redis/redis.conf --port 6385 
    
    docker run --name myredis-6 --network host -v /root/cluster/redis6.conf:/etc/redis/redis.conf -v /root/cluster/data/6386:/data -d redis:7.0 redis-server /etc/redis/redis.conf --port 6386 
    ```
    
4. 创建系统
    
    6个节点启动后，他们仍是6个独立的Redis，通过redis-cli --cluster create 命令将6个节点创建为一个分布式系统。
    
    -- cluster replicas 1 指定每个master带有一个slave。
    
    ```sh
    docker exec -it myredis-1 redis-cli --cluster create --cluster-replicas 1 192.168.192.101:6381 192.168.192.101:6382 192.168.192.101:6383 192.168.192.101:6384 192.168.192.101:6385 192.168.192.101:6386
    ```
    
    再输入yes即可完成创建：
    

![无标2题](https://img2024.cnblogs.com/blog/3388489/202606/3388489-20260603150029490-646370276.png)

3. 查看节点信息
    
    通过cluster nodes 命令查看个节点的关系及连接情况，只要每个节点给出connected。就说明分布式系统创建成功！
    

![无3标题](https://img2024.cnblogs.com/blog/3388489/202606/3388489-20260603150037045-758399095.png)