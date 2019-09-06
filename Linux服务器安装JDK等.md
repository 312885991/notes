# Linux服务器上安装 JDK、MySQL、Tomcat



> 将 JDK MySQL Tomcat Redis 压缩包上传到 Linux 服务器上

<b style='color:red'>相关压缩包百度网盘下载地址：</b>[百度网盘](https://pan.baidu.com/s/1LFZzxuFsyVSoV_LbjYgXlA
)  提取码: **8h91**

![![](image/t1.png)](https://images.gitee.com/uploads/images/2019/0906/203042_3dd20b6b_4771190.png "t1.png")



> 利用 putty 工具连接到 远程服务器

![![](image/t2.png)](https://images.gitee.com/uploads/images/2019/0906/203052_fd067ee9_4771190.png "t2.png")

## 一、JDK 安装与配置

### 1.1、 解压

- `cd /usr/local/jdk`  转到压缩包目录

- `tar -xvf jdk1.8.tar.gz` 解压文件

  解压后的文件：

  ![![](image/t3.png)](https://images.gitee.com/uploads/images/2019/0906/203109_1a1b6f7f_4771190.png "t3.png")

### 1.2、配置环境变量

- `vi /etc/profile ` 进入文件中，配置环境变量 ，按i进行编辑 

  输入如下内容：

  ```xml
  #set java environment
  JAVA_HOME=/usr/local/jdk/jdk1.8.0_211
  CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
  PATH=$PATH:$JAVA_HOME/bin
  export JAVA_HOME CLASSPATH PATH
  ```

- 按 ESC 键  输入 :wq 回车保存

- `source /etc/profile`  使文件生效

###  1.3、查看

- `java -version`  查看是否配置成功?

  ![![](image/t4.png)](https://images.gitee.com/uploads/images/2019/0906/203119_8891ccda_4771190.png "t4.png")



## 二、MySQL 安装与配置

### 2.1、 解压

- `cd /usr/local/mysql`  转到压缩包目录

- `tar -xvf mysql-8.0.17.tar ` 解压文件

  解压后的文件：

  ![![](image/m1.png)](https://images.gitee.com/uploads/images/2019/0906/203129_0c71357c_4771190.png "m1.png")

### 2.2、安装与配置

- **安装 common**

  - `rpm -ivh mysql-community-common-8.0.17-1.el7.x86_64.rpm --nodeps --force `

- **安装 libs**

  - `rpm -ivh mysql-community-libs-8.0.17-1.el7.x86_64.rpm --nodeps --force`

- **安装 client**

  - `rpm -ivh mysql-community-client-8.0.17-1.el7.x86_64.rpm --nodeps --force`

- **安装 server**

  - `rpm -ivh mysql-community-server-8.0.17-1.el7.x86_64.rpm --nodeps --force` 

- 查看已安装的包

  `rpm -qa | grep mysql`

  ![![](image/m2.png)](https://images.gitee.com/uploads/images/2019/0906/203139_2562a6fe_4771190.png "m2.png")

- 初始化

  ```xml
  yum install -y libaio
  mysqld --initialize
  chown mysql:mysql /var/lib/mysql -R
  rm -rf /var/lib/mysql
  systemctl start mysqld.service
  systemctl enable mysqld
  ```

- 查看数据库密码

  ```
  cat /var/log/mysqld.log | grep password
  ```

  ![![](image/m3.png)](https://images.gitee.com/uploads/images/2019/0906/203148_8c549e65_4771190.png "m3.png")

-   登录 `mysql -u root -p`

  ![![](m4.png)](https://images.gitee.com/uploads/images/2019/0906/203157_7056576a_4771190.png "m4.png")

- 第一次登录之后 必须修改密码 `ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '123456';`

  ![![](image/m5.png)](https://images.gitee.com/uploads/images/2019/0906/203206_3bc1a19c_4771190.png "m5.png")

  - 说明是 所设置的密码不符合MySQL8.0默认的密码策略（刚开始设置的密码必须符合长度，且必须含有数字，小写或大写字母，特殊字符）

  - 先随便修改个符合规定的密码：`ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'Root_12root';`

  - 再查看 mysql 初始的密码策略 `SHOW VARIABLES LIKE 'validate_password%'; `

    ![![](image/m6.png)](https://images.gitee.com/uploads/images/2019/0906/203216_6d9e69e1_4771190.png "m6.png")

  - 修改密码验证策略

    ```
    set global validate_password.policy=0;
    set global validate_password.length=1;
    ```

  - 再进行修改密码 `ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '123456';`

    ![![](image/m7.png)](https://images.gitee.com/uploads/images/2019/0906/203226_e4f4c276_4771190.png "m7.png")

### 2.3、配置远程连接

-  登录 `mysql -u root -p`
- 使用 mysql 数据库  `use mysql;`
- 查看用户访问权限  ` select user,host from user;`

![![](image/m8.png)](https://images.gitee.com/uploads/images/2019/0906/203236_ec55a0f2_4771190.png "m8.png")

- 修改访问权限 `update user set host='%' where user='root';`

  ![![](image/m9.png)](https://images.gitee.com/uploads/images/2019/0906/203245_65e310f2_4771190.png "m9.png")

- 刷新权限  `flush privileges;`

### 2.4、测试连接



![![](image/m10.png)](https://images.gitee.com/uploads/images/2019/0906/203256_9ee84c8d_4771190.png "m10.png")

### 2.5、BUG

重启mysql服务时，出现以下情形：

![![](image/b1.png)](https://images.gitee.com/uploads/images/2019/0906/203304_442f0e74_4771190.png "b1.png")

解决方法:

- 出现这种错误，有很多种不同的情况，可以去百度
- 这里是由于 其他 SpringBoot项目 在使用 MySQL 数据库 ，所以先把这些应用关闭
- 直接杀死进程（根据端口） `sudo fuser -k -n tcp 端口号`
- 再启动 MySQL 服务

### 2.6、其他相关 MySQL 命令

- `service mysqld stop`  停止 MySQL 服务
- `service mysqld start ` 启动 MySQL 服务
- `service mysqld restart`  重启 MySQL 服务
- `service mysqld status`  查看 MySQL 服务的启动状态

## 三、Tomcat 安装

### 3.1、 解压

- `cd /usr/local/tomcat`  转到压缩包目录

- `yum install -y unzip zip` 安装 zip 和 unzip 命令

- `unzip tomcat9.zip` 解压文件

  解压后的文件：

  ![![](image/t5.png)](https://images.gitee.com/uploads/images/2019/0906/203322_6fb3511b_4771190.png "t5.png")

### 3.2、启动 Tomcat

- `cd /usr/local/tomcat/apache-tomcat-9.0.20/bin/`  转到 tomcat 的 bin目录

- `./startup.sh` 启动服务器

  - 发现报错 Permission denied (权限不够)

    ![![](image/t6.png)](https://images.gitee.com/uploads/images/2019/0906/203335_58d90193_4771190.png "t6.png")

  - 解决方法

    - 如果你是root登陆的话（不是的话，切换到root用户）
    - `chmod +x  *.sh`  对 *.sh 命令赋可执行的权限
    - 重新启动tomcat `cd /usr/local/tomcat/apache-tomcat-9.0.20/bin/`  `./startup.sh`

    ![![](image/t9.png)](https://images.gitee.com/uploads/images/2019/0906/203348_a65df9fa_4771190.png "t9.png")



###  3.3、访问 tomcat

- 访问 `47.98.165.28:8080`  查看是否可以成功访问

![![](image/t10.png)](https://images.gitee.com/uploads/images/2019/0906/203402_886fae5b_4771190.png "t10.png")

## 四、Redis 安装

### 4.1、 解压

- `cd /usr/local/redis`  转到压缩包目录

- `tar -xvf redis5.0.tar.gz` 解压文件

  解压后的文件：

  ![![](image/t11.png)](https://images.gitee.com/uploads/images/2019/0906/203415_92772efb_4771190.png "t11.png")

### 4.2、安装与配置

- `cd /usr/local/redis/redis-5.0.5` 进入目录中

- `make` 使用make命令编译

  - 编译完成后，会在src目录下生成6个可执行文件。分别是redis-server、redis-cli、redis-benchmark、redis-check-aof、redis-check-rdb、redis-sentinel

- 将译生成的可执行文件拷贝到/usr/local/bin目录下（这个后期可以直接使用命令）

  - `cd /usr/local/redis/redis-5.0.5/src`
  - `cp {redis-server,redis-cli,redis-benchmark,redis-check-aof,redis-check-rdb,redis-sentinel} /usr/local/bin`

- 进入 redis-5.0.5目录中 

  - `cd /usr/local/redis/redis-5.0.5`
  - `make install` 执行安装命令 

  ![![](image/t12.png)](https://images.gitee.com/uploads/images/2019/0906/203426_a480f3b7_4771190.png "t12.png")

- 执行基本配置 `./utils/install_server.sh`

  ![![](image/t13.png)](https://images.gitee.com/uploads/images/2019/0906/203435_e0a93162_4771190.png "t13.png")



  **<b style='color:red'>一阵回车就OK了</b>**



![![](image/t14.png)](https://images.gitee.com/uploads/images/2019/0906/203444_c0cf32da_4771190.png "t14.png")

- 查看开机启动列表

  - `chkconfig --list`

  ![![](image/t15.png)](https://images.gitee.com/uploads/images/2019/0906/203454_9aeaf132_4771190.png "t15.png")

  > 开启Redis服务操作通过/etc/init.d/redis_6379 start命令，也可通过（service redis_6379 start）
  >
  > 关闭Redis服务操作通过/etc/init.d/redis_6379 stop命令，也可通过（service redis_6379 stop）

- 配置 redis 相关信息

  - `cd /etc/redis`  转到该目录下

  - `vi 6379.conf` 编辑该配置文件

    - 在 bind 127.0.0.1 前 加 “#” 将其注释掉 (<b style='color:red'>这样才能实现远程访问连接</b>)

      ![![](image/t16.png)](https://images.gitee.com/uploads/images/2019/0906/203505_66332c3d_4771190.png "t16.png")

    - 默认为保护模式，把 protected-mode yes 改为 protected-mode no

      ![![](image/t17.png)](https://images.gitee.com/uploads/images/2019/0906/203514_37c292d9_4771190.png "t17.png")

    - 将 requirepass foobared前的“#”去掉，密码改为你想要设置的密码

      ![![](image/t18.png)](https://images.gitee.com/uploads/images/2019/0906/203524_ae4979a0_4771190.png "t18.png")

    - :wq 保存

- 重启 redis

  - `/etc/init.d/redis_6379 stop` 关闭redis
  - `/etc/init.d/redis_6379 start` 启动redis
 ### 4.3、BUG

- 关闭服务时，可能出现以下bug ，此时使用 `redis-cli -a 123456 shutdown` 命令解决

![![](image/bug.png)
](https://images.gitee.com/uploads/images/2019/0906/203535_e368b481_4771190.png "bug.png")
 ### 4.4、测试连接

![![](image/t19.png)](https://images.gitee.com/uploads/images/2019/0906/203544_6393066c_4771190.png "t19.png")

![![](image/t20.png)
](https://images.gitee.com/uploads/images/2019/0906/203553_20b33d46_4771190.png "t20.png")


## 五、SpringBoot 项目部署

### 5.1 将需要上传的项目打成 jar 包

![![](image/j1.png)](https://images.gitee.com/uploads/images/2019/0906/223945_aa5afc66_4771190.png "j1.png")

### 5.2、将 jar 包上传到 Linux 服务器上

![![](image/j2.png)](https://images.gitee.com/uploads/images/2019/0906/223954_904426c6_4771190.png "j2.png")

### 5.3、利用 putty 工具连接上 Linux 服务器

### 5.4、运行 jar 包（后台运行）

- `nohup java -jar eureka.jar >eureka-log.txt &`  后台运行 jar 包，并且将启动信息及后续日志 打印到 eureka-log.txt 文件中

- `nohup java -jar oauth.jar >oauth-log.txt &`

- `nohup java -jar zuul.jar >zuul-log.txt &`

- `nohup java -jar user-biz.jar >user-biz-log.txt &`

- `nohup java -jar app.jar >app-log.txt &`


![![](image/j3.png)](https://images.gitee.com/uploads/images/2019/0906/224003_4efb6a1a_4771190.png "j3.png")



### 5.5 其他相关 Linux 命令

- `netstat -ant`  查看服务器所有被占用端口
- `netstat -tunlp | grep 端口号 ` 验证某个端口号是否被占用
- `netstat -tunlp | grep 端口号 ` 查看所有监听端口号
-  `sudo fuser -k -n tcp 端口号`  根据 端口号 直接杀死进程
- `free -mh` 查看内存使用情况 
- `top`  查看 CPU 使用情况
- `ps -ef|grep java` 查看与 java 相关的进程（可以查看对应的进程号）
- `kill -9 进程号` 根据 进程号 直接结束进程