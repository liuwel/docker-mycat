# 使用docker创建mycat mysql主从服务器

### 拉取 [github项目](https://github.com/liuwel/docker-mycat "github") 
配置文件已经全部写好 基本找下面流程走一遍就能直接用

注意：mycat 和 mysql使用的字符集编码全部是 <code>utf8mb4</code>

```shell
% cd ~ #切换到主目录
% git clone https://github.com/liuwel/docker-mycat.git
% tree docker-mycat
├ compose
│   ├ docker-compose.yml
│   ├ master
│   │   └ Dockerfile
│   ├ mycat
│   │   ├ Dockerfile
│   │   └ Mycat-server-1.6.5-release-20180122220033-linux.tar.gz
│   ├ s1
│   │   └ Dockerfile
│   └ s2
│       └ Dockerfile
├ config
│   ├ hosts
│   ├ mycat
│   │   ├ autopartition-long.txt
│   │   ├ auto-sharding-long.txt
│   │   ├ auto-sharding-rang-mod.txt
│   │   ├ cacheservice.properties
│   │   ├ ehcache.xml
│   │   ├ index_to_charset.properties
│   │   ├ log4j2.xml
│   │   ├ migrateTables.properties
│   │   ├ myid.properties
│   │   ├ partition-hash-int.txt
│   │   ├ partition-range-mod.txt
│   │   ├ rule.xml
│   │   ├ schema.xml
│   │   ├ sequence_conf.properties
│   │   ├ sequence_db_conf.properties
│   │   ├ sequence_distributed_conf.properties
│   │   ├ sequence_time_conf.properties
│   │   ├ server.xml
│   │   ├ sharding-by-enum.txt
│   │   ├ wrapper.conf
│   │   ├ zkconf
│   │   │   ├ autopartition-long.txt
│   │   │   ├ auto-sharding-long.txt
│   │   │   ├ auto-sharding-rang-mod.txt
│   │   │   ├ cacheservice.properties
│   │   │   ├ ehcache.xml
│   │   │   ├ index_to_charset.properties
│   │   │   ├ partition-hash-int.txt
│   │   │   ├ partition-range-mod.txt
│   │   │   ├ rule.xml
│   │   │   ├ schema.xml
│   │   │   ├ sequence_conf.properties
│   │   │   ├ sequence_db_conf.properties
│   │   │   ├ sequence_distributed_conf-mycat_fz_01.properties
│   │   │   ├ sequence_distributed_conf.properties
│   │   │   ├ sequence_time_conf-mycat_fz_01.properties
│   │   │   ├ sequence_time_conf.properties
│   │   │   ├ server-mycat_fz_01.xml
│   │   │   ├ server.xml
│   │   │   └ sharding-by-enum.txt
│   │   └ zkdownload
│   │       └ auto-sharding-long.txt
│   ├ mycat-logs
│   │   ├ mycat.log
│   │   ├ mycat.pid
│   │   └ wrapper.log
│   ├ mysql-master
│   │   ├ conf.d
│   │   │   ├ client.cnf
│   │   │   ├ docker.cnf
│   │   │   └ mysql.cnf
│   │   ├ my.cnf
│   │   ├ mysql.cnf
│   │   └ mysql.conf.d
│   │       └ mysqld.cnf
│   ├ mysql-s1
│   │   ├ conf.d
│   │   │   ├ client.cnf
│   │   │   ├ docker.cnf
│   │   │   └ mysql.cnf
│   │   ├ my.cnf
│   │   ├ mysql.cnf
│   │   └ mysql.conf.d
│   │       └ mysqld.cnf
│   └ mysql-s2
│       ├ conf.d
│       │   ├ client.cnf
│       │   ├ docker.cnf
│       │   └ mysql.cnf
│       ├ my.cnf
│       ├ mysql.cnf
│       └ mysql.conf.d
│           └ mysqld.cnf
└ README.md
19 directories, 69 files
```
#### mysql 主从服务器的配置已经写在config对应的目录中 
mysql-m1 : 主服务器 IP:172.18.0.2 

mysql-s1 : 从服务器slave1 IP:172.18.0.3

mysql-s2 : 从服务器slave2 IP:172.18.0.4

mycat    : Mycat服务器 IP:172.18.0.5  

### 修改hosts文件 添加解析
```shell
% sudo vi /etc/hosts
# docker-mycat m1:mysql-master主服务器 s1,s2：mysql-slave 从服务器
# mycat mycat中间件服务器
172.18.0.2      m1
172.18.0.3      s1
172.18.0.4      s2
172.18.0.5      mycat
127.0.0.1       local
```

### docker-compose.yml配置文件
```shell
% cd ~/docker-mycat/compose
% cat docker-compose.yml
```
```yml
version: '2'
services:
  m1:
    build: ./mysql_m1
    container_name: m1
    volumes:
      - ../config/mysql-master/:/etc/mysql/:ro
      - /etc/localtime:/etc/localtime:ro
      - ../config/hosts:/etc/hosts:ro
    ports:
      - "3309:3306"
    networks:
      mysql:
        ipv4_address: 172.18.0.2
    ulimits:
      nproc: 65535
    hostname: m1
    mem_limit: 512m
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: m1test
  s1:
      build: ./mysql_s1
      container_name: s1
      volumes:
        - ../config/mysql-s1/:/etc/mysql/:ro
        - /etc/localtime:/etc/localtime:ro
        - ../config/hosts:/etc/hosts:ro
      ports:
        - "3307:3306"
      networks:
        mysql:
          ipv4_address: 172.18.0.3
      links:
        - m1
      ulimits:
        nproc: 65535
      hostname: s1
      mem_limit: 512m
      restart: always
      environment:
        MYSQL_ROOT_PASSWORD: s1test
  s2:
    build: ./mysql_s2
    container_name: s2
    volumes:
      - ../config/mysql-s2/:/etc/mysql/:ro
      - /etc/localtime:/etc/localtime:ro
      - ../config/hosts:/etc/hosts:ro
    ports:
      - "3308:3306"
    links:
      - m1
    networks:
      mysql:
        ipv4_address: 172.18.0.4
    ulimits:
      nproc: 65535
    hostname: s2
    mem_limit: 512m
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: s2test
  mycat:
    build: ./mycat
    container_name: mycat
    volumes:
      - ../config/mycat/:/mycat/conf/:ro
      - ../config/mycat-logs/:/mycat/logs/:rw
      - /etc/localtime:/etc/localtime:ro
      - ../config/hosts:/etc/hosts:ro
    ports:
      - "8066:8066"
      - "9066:9066"
    links:
      - m1
      - s1
      - s2
    networks:
      mysql:
        ipv4_address: 172.18.0.5
    ulimits:
      nproc: 65535
    hostname: mycat
    mem_limit: 512m
    restart: always
networks:
  mysql:
    driver: bridge
    ipam:
      driver: default
      config:
      - subnet: 172.18.0.0/24
        gateway: 172.18.0.1
```
### Build 镜像
```shell
% sudo docker-compose build m1 s1 s2
Building m1
Step 1/4 : FROM mysql:5.7.17
 ---> 9546ca122d3a
 ...
Successfully built cffffead5570
Successfully tagged compose_s2:latest
```
### 运行 docker mysql主从数据库 (mysql数据库密码在yml文件里面)
```shell
% sudo docker-compose up -d m1 s1 s2  
Creating m1                             
Creating s2                             
Creating s1                             
```
### mysql主从配置
#### 配置m1主服务器
```shell
sudo docker exec -it m1 /bin/bash
root@m1:/# mysql -uroot -pm1test
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.7.17-log MySQL Community Server (GPL)

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
mysql>
```
已经进入m1主服务器mysql 命令行 

创建用于主从复制的用户repl
```shell
mysql> create user repl;
```
给repl用户授予slave的权限
```shell
mysql> GRANT REPLICATION SLAVE ON *.* TO 'repl'@'172.18.0.%' IDENTIFIED BY 'repl';
Query OK, 0 rows affected, 1 warning (0.00 sec)
```
锁表
```shell
mysql> FLUSH TABLES WITH READ LOCK;
Query OK, 0 rows affected (0.00 sec)
```
查看binlog状态 记录File 和 Position 状态稍后从库配置的时候会用
```shell
mysql>  show master status;
+-------------------+----------+--------------+------------------+-------------------+
| File              | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+-------------------+----------+--------------+------------------+-------------------+
| master-bin.000003 |      644 |              |                  |                   |
+-------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
```
#### 配置从库s1 s2
进入s1 shell
```shell
% sudo docker exec -it s1 /bin/bash
root@s1:/# mysql -uroot -ps1test
mysql> change master to master_host='m1',master_port=3306,master_user='repl',master_password='repl',master_log_file='master-bin.000003',master_log_pos=644;
Query OK, 0 rows affected, 2 warnings (0.05 sec)
mysql> start slave;
Query OK, 0 rows affected (0.00 sec)
```
进入s2 shell
```shell
sudo docker exec -it s2 /bin/bash                                                            
root@s2:/# mysql -uroot -ps2test
mysql> change master to master_host='m1',master_port=3306,master_user='repl',master_password='repl',master_log_file='master-bin.000003',master_log_pos=644;
Query OK, 0 rows affected, 2 warnings (0.03 sec)

mysql> start slave;
Query OK, 0 rows affected (0.00 sec)
```
### mysql主从配置完成 现在测试一下
登陆主数据库 创建masterdb数据库 (这个数据库名在稍后的mycat里面会用到)
```shell
% mysql -uroot -pm1test -hm1
MySQL [(none)]> create database masterdb;
Query OK, 1 row affected (0.01 sec)
```
进入从库看看数据库是否创建
```shell
% mysql -uroot -ps1test -hs1
MySQL [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| masterdb           |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)
```
可以看到从库也已经创建成功了 到这里msyql的主从已经配置完成了

接下来是mycat的配置其实在 ~/config/mycat 里面已经配置好了直接就可以用了

看下schama.xml配置文件  
```shell 
% cat ~/config/mycat/schema.xml
```

```xml
<?xml version="1.0"?>                                                                                          
<!DOCTYPE mycat:schema SYSTEM "schema.dtd">
<mycat:schema xmlns:mycat="http://io.mycat/"> 
        <schema name="masterdb" checkSQLschema="false" sqlMaxLimit="100" dataNode="masterDN" />                
        <dataNode name="masterDN" dataHost="masterDH" database="masterdb" />    
        <dataHost name="masterDH" maxCon="1000" minCon="10" balance="1"           
                          writeType="0" dbType="mysql" dbDriver="native" switchType="-1" slaveThreshold="100"> 
                <heartbeat>select user()</heartbeat>                   
                <writeHost host="m1" url="172.18.0.2:3306" user="root" password="m1test">                      
                        <readHost host="s1" url="172.18.0.3:3306" user="root" password="s1test" />             
                        <readHost host="s2" url="172.18.0.4:3306" user="root" password="s2test" />             
                </writeHost>
        </dataHost>  
</mycat:schema>               
```
server.xml 配置文件 
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- - - Licensed under the Apache License, Version 2.0 (the "License");
        - you may not use this file except in compliance with the License. - You
        may obtain a copy of the License at - - http://www.apache.org/licenses/LICENSE-2.0
        - - Unless required by applicable law or agreed to in writing, software -
        distributed under the License is distributed on an "AS IS" BASIS, - WITHOUT
        WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. - See the
        License for the specific language governing permissions and - limitations
        under the License. -->
<!DOCTYPE mycat:server SYSTEM "server.dtd">
<mycat:server xmlns:mycat="http://io.mycat/">
        <system>
        <property name="charset">utf8mb4</property>
        <property name="useSqlStat">0</property>  <!-- 1为开启实时统计、0为关闭 -->
        <property name="useGlobleTableCheck">0</property>  <!-- 1为开启全加班一致性检测、0为关闭 -->

                <property name="sequnceHandlerType">2</property>
      <!--  <property name="useCompression">1</property>--> <!--1为开启mysql压缩协议-->
        <!--  <property name="fakeMySQLVersion">5.6.20</property>--> <!--设置模拟的MySQL版本号-->
        <!-- <property name="processorBufferChunk">40960</property> -->
        <!--
        <property name="processors">1</property>
        <property name="processorExecutor">32</property>
         -->
                <!--默认为type 0: DirectByteBufferPool | type 1 ByteBufferArena-->
                <property name="processorBufferPoolType">0</property>
                <!--默认是65535 64K 用于sql解析时最大文本长度 -->
                <!--<property name="maxStringLiteralLength">65535</property>-->
                <!--<property name="sequnceHandlerType">0</property>-->
                <!--<property name="backSocketNoDelay">1</property>-->
                <!--<property name="frontSocketNoDelay">1</property>-->
                <!--<property name="processorExecutor">16</property>-->
                <!--
                        <property name="serverPort">8066</property> <property name="managerPort">9066</property>
                        <property name="idleTimeout">300000</property> <property name="bindIp">0.0.0.0</property>
                        <property name="frontWriteQueueSize">4096</property> <property name="processors">32</property> -->
                <!--分布式事务开关，0为不过滤分布式事务，1为过滤分布式事务（如果分布式事务内只涉及全局表，则不过滤），2为不
过滤分布式事务,但是记录分布式事务日志-->
                <property name="handleDistributedTransactions">0</property>
                        <!--
                        off heap for merge/order/group/limit      1开启   0关闭
                -->
                <property name="useOffHeapForMerge">1</property>
                <!--
                        单位为m
                -->
                <property name="memoryPageSize">1m</property>
                <!--
                        单位为k
                -->
                <property name="spillsFileBufferSize">1k</property>
                <property name="useStreamOutput">0</property>
                <!--
                        单位为m
                -->
                <property name="systemReserveMemorySize">384m</property>
                <!--是否采用zookeeper协调切换  -->
                <property name="useZKSwitch">true</property>
        </system>

        <!-- 全局SQL防火墙设置
        <firewall>
           <whitehost>
              <host host="172.18.0.2" user="root"/>
              <host host="172.18.0.3" user="root"/>
                                <host host="172.18.0.4" user="root"/>
           </whitehost>
       <blacklist check="false">
       </blacklist>
        </firewall>-->
        <user name="root">
                <property name="password">youpassword</property>
                <property name="schemas">masterdb</property>

                <!-- 表级 DML 权限设置 -->
                <!--
                <privileges check="false">
                        <schema name="TESTDB" dml="0110" >
                                <table name="tb01" dml="0000"></table>
                                <table name="tb02" dml="1111"></table>
                        </schema>
                </privileges>
                 -->
        </user>
</mycat:server>
```
### 启动mycat
```shell
% cd ~/docker-mycat/compose
% sudo docker-compose up -d mycat
```

### 整体测试
```shell
% mysql -uroot -p -P8066 -hlocal
```
```shell
MySQL \[(none)\]> show databases;
+----------+
| DATABASE |
+----------+
| masterdb |
+----------+
1 row in set (0.00 sec)
```

### 测试数据
```shell
MySQL [(none)]> use masterdb                                                         
Database changed                                                                     
MySQL [masterdb]> CREATE TABLE `test_table` (                                        
    ->   `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '自增主键',                        
    ->   `title` varchar(255) DEFAULT NULL COMMENT '标题',                             
    ->   `content` text COMMENT '内容',                                                
    ->   PRIMARY KEY (`id`)                                                          
    -> ) ENGINE=InnoDB COMMENT='测试表'                                                 
    -> ;                                                                             
Query OK, 0 rows affected (0.03 sec)                                                 
```
```shell                                                                                   
MySQL [masterdb]> show tables;                                                       
+--------------------+                                                               
| Tables_in_masterdb |                                                               
+--------------------+                                                               
| test_table         |                                                               
+--------------------+                                                               
1 row in set (0.00 sec)                                                              
```
```shell                                                                              
MySQL [masterdb]> INSERT INTO `test_table` VALUES ('1', '测试标题1', '测试内容1');           
Query OK, 1 row affected (0.01 sec)                                           
                                                                                     
MySQL [masterdb]> INSERT INTO `test_table` VALUES ('2', '测试标题2', '测试内容2');           
Query OK, 1 row affected (0.01 sec)                                    
                                                                                     
MySQL [masterdb]> INSERT INTO `test_table` VALUES ('3', '测试标题3', '测试内容3');           
Query OK, 1 row affected (0.01 sec)                                              
                                                                                     
MySQL [masterdb]> INSERT INTO `test_table` VALUES ('4', '测试标题4', '测试内容4');           
Query OK, 1 row affected (0.01 sec)                                         
                                                                                     
MySQL [masterdb]> INSERT INTO `test_table` VALUES ('5', '测试标题5', '测试内容5');           
Query OK, 1 row affected (0.01 sec)                                               
                                                                                     
MySQL [masterdb]> INSERT INTO `test_table` VALUES ('6', '测试标题6', '测试内容6');           
Query OK, 1 row affected (0.01 sec)                                                  
```
```shell
MySQL [masterdb]> select * from test_table;                                          
+----+---------------+---------------+                                               
| id | title         | content       |                                               
+----+---------------+---------------+                                               
|  1 | 测试标题1     | 测试内容1     |                                                       
|  2 | 测试标题2     | 测试内容2     |                                                       
|  3 | 测试标题3     | 测试内容3     |                                                       
|  4 | 测试标题4     | 测试内容4     |                                                       
|  5 | 测试标题5     | 测试内容5     |                                                       
|  6 | 测试标题6     | 测试内容6     |                                                       
+----+---------------+---------------+                                               
6 rows in set (0.01 sec)
```











