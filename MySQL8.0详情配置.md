### 一、MySQL8.0免安装版详情配置

####	1.MySQL8.0下载

- 官网下载地址  [MySQL8.0下载]([https://dev.mysql.com/downloads/mysql](https://dev.mysql.com/downloads/mysql/))  （官网下载速度慢的不行，这里可以去我的网盘进行下载）
- 网盘下载地址  [MySQL8.0下载](https://pan.baidu.com/s/1cIlvhPpThSSEuNcwmU63yQ )  提取码  `hiav`   

#### 2.MySQL8.0解压

1. 将下载好的 `MySQL8.0压缩包` 放置在自己的某个空闲磁盘中，这里我自己放置在 `D:\databases` 下
2. 将 `mysql-8.0.15-winx64.zip` 解压在当前文件夹下，得到 `mysql-8.0.15-winx64` 文件夹

#### 3.MySQL8.0配置

1. 打开 `mysql-8.0.15-winx64` 文件夹，在当前文件夹下创建 `my.ini` 文件

2. 再在 `mysql-8.0.15-winx64` 文件夹下创建 `data` 文件夹 (如已有，则清空)

3. 配置 `my.ini` 文件，内容如下

   ```ini
   [mysqld]
   # 绑定IPv4
   bind-address=0.0.0.0
   # 设置mysql的安装目录,即你解压缩安装包的位置
   basedir=D:\databases\mysql-8.0.15-winx64
   # 设置mysql数据库的数据的存放目录
   datadir=D:\databases\mysql-8.0.15-winx64\data
   # 设置端口号
   port=3306
   # 允许最大连接数
   max_connections=200
   # 开启查询缓存
   explicit_defaults_for_timestamp=true
   # 创建表使用的默认存储引擎
   default-storage-engine=INNODB
   # 设置服务端的默认字符集
   character-set-server=utf8
   [mysql]  
   # 设置mysql客户端默认字符集  
   default-character-set=utf8
   ```

4. 配置环境变量

   ![](https://oscimg.oschina.net/oscnet/2373cad5eed458f4df8673aa41b79868ca9.jpg)

5. 安装 `mysql8.0` 服务，并初始化

   1. 以`管理员权限打开` `cmd` 窗口

   2. 执行安装命令：

      ```ini
      mysqld --install MySQL --defaults-file="D:\databases\mysql-8.0.15-winx64\my.ini"
      ```

   3. 执行初始化命令：

      ```ini
      mysqld --initialize
      ```

6. 启动 `mysql8.0` 服务，执行以下命令：

   ```ini
   net start mysql
   ```

7. 修改 `root` 用户密码

   1. 在 `D:\databases\mysql-8.0.15-winx64\data` 目录下找到后缀为 `.err` 文件，在该文件中查找刚才系统为 `root` 用户生成的随机密码，本人的如下：

      ![](https://oscimg.oschina.net/oscnet/66b5bb93d208bfda80426c820e2c778a678.jpg)

   2. 使用查找到的密码登录 `mysql8.0`：

      ```ini
      mysql -u root -p
      ```

      ![](https://oscimg.oschina.net/oscnet/479882e2e469c00781b32161ea648e4d153.jpg)

   3. 登陆成功后，修改密码：

      ```ini
      ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '你的密码';
      ```

      ![](https://oscimg.oschina.net/oscnet/7eb266b48299e272d67184f51158e397dd9.jpg)

   4. 修改成功

      ![](https://oscimg.oschina.net/oscnet/46c2d0b8e1dd48afbe9326653f6f5a5ae4d.jpg)


#### 4.MySQL8.0测试连接

   1. 打开 `Navicat` 应用

   2. 测试连接

      ![](https://oscimg.oschina.net/oscnet/6841b76e5209c3cb4153924c4b3c7e08fd9.jpg)

   3. 连接成功， `MySQL8.0`  免安装版安装成功!

   

