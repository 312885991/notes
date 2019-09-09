### 一、MySQL8.0免安装版详情配置

####	1.MySQL8.0下载

- 官网下载地址  [MySQL8.0下载]([https://dev.mysql.com/downloads/mysql](https://dev.mysql.com/downloads/mysql/))  （官网下载速度慢的不行，这里可以去我的网盘进行下载）
- 网盘下载地址  [MySQL8.0下载](https://pan.baidu.com/s/1ibmYHTu1CPq8WWu8VBUByA)   提取码  `hiav`   

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

   ![![](mysql-01.png)](https://images.gitee.com/uploads/images/2019/0909/164445_0f377297_4771190.png "mysql-01.png")

5. 安装 `mysql8.0` 服务，并初始化

   1. 以`管理员权限打开` `cmd` 窗口

   2. 执行安装命令：

      ```ini
      mysqld --install MySQL --defaults-file="D:\databases\mysql-8.0.15-winx64\my.ini"
      ```
      - 如果MySQL服务已经存在，则使用 `sc delete mysql` 命令删除已存在的MySQL服务

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

      ![![](mysql-02.png)](https://images.gitee.com/uploads/images/2019/0909/164503_3a25e38c_4771190.png "mysql-02.png")

   2. 使用查找到的密码登录 `mysql8.0`：

      ```ini
      mysql -u root -p
      ```

      ![![](mysql-04.png)](https://images.gitee.com/uploads/images/2019/0909/164514_9f189582_4771190.png "mysql-04.png")

   3. 登陆成功后，修改密码：

      ```ini
      ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '你的密码';
      ```

      ![![](mysql-05.png)](https://images.gitee.com/uploads/images/2019/0909/164523_7d0b01dc_4771190.png "mysql-05.png")

   4. 修改成功

      ![![](mysql-06.png)](https://images.gitee.com/uploads/images/2019/0909/164531_bbd11dc7_4771190.png "mysql-06.png")
   
      

#### 4.MySQL8.0测试连接

   1. 打开 `Navicat` 应用

   2. 测试连接

      ![](https://images.gitee.com/uploads/images/2019/0909/164541_e6f63999_4771190.png "mysql-07.png")

      

   3. 连接成功， `MySQL8.0`  免安装版安装成功!

   