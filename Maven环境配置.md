

## 1、约定



- 分类存放
  - X:/applications 目录存放 开发工具，比如 IDEA 、Eclipse 、Typora
  - X:/Environments 目录存放开发环境，比如  JDK、MinGW-w64、cmder
  - X:/IdeaProjects 目录存放 IDEA 工程
  - X:/workspaces 目录存放 Eclipse 的工作空间
  - X:/Repository 目录用作 apache-maven 的仓库
  - X:/Repository/Synchronization 目录用作 apache-maven 从 中央仓库同步时使用的本地仓库位置
    这个目录，也是在 apache-maven 的 conf/settings.xml 中所配置的 localRepository 对应的目录
  - X:/Repository/ThirdParty 目录 用于存放不存在于 maven 中央仓库的 jar 文件 ( 比如 ojdbc.jar )



> 注意，这里的  X  盘可以是硬盘上的 C盘、D盘等。



- 编码
  - Eclipse 所有的文本文件采用 UTF-8 编码 ( Text file encoding )
  - 换行方式采用 Unix 方式 （ New text file line delimiter ）



## 2、maven



### 2.1、下载



从 [apache-maven 官网](<http://maven.apache.org/>) 下载 最新版 maven ，在官网的下载页面中:

- Binary zip archive ( apache-maven-3.6.0-**bin**.zip  ) 表示已经编译好的，可以直接使用的压缩包
- Source zip archive ( apache-maven-3.6.0-**src**.zip ) 表示 源码包



### 2.2、解压



根据 约定，应该将 apache-maven-3.6.0-**bin**.zip 解压到 X:/applications 目录下。



### 2.3、配置



根据约定，在  X:/applications/apache-maven-3.6.0 目录中找到 `conf` 目录，使用 EditPlus 、 EmEditor、Notepad++ 等工具打开 其中的 settings.xml 文件。



#### 2.3.1、替换



将 conf/settings.xml 文件中的所有内容，替换为以下内容:

```xml
<?xml version="1.0" encoding="UTF-8"?>

<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
    
  <!-- localRepository
   | The path to the local repository maven will use to store artifacts.
   |
   | Default: ${user.home}/.m2/repository
   -->
  <localRepository>/path/to/local/repo</localRepository>
  
  <pluginGroups></pluginGroups>

  <proxies></proxies>

  <servers></servers>

  <mirrors>
   
  </mirrors>
 
  <profiles></profiles>

</settings>
```



#### 2.3.2、修改本地仓库

建议根据约定，将 conf/settings.xml 文件中的本地仓库位置修改为如下形式:

```xml
<localRepository>D:/Repository/Synchronization</localRepository>
```



#### 2.3.3、修改 mirror 



```xml
   <mirrors>
	<!-- 由阿里云提供的Maven中央仓库镜像 -->
	<mirror>
		<id>nexus-aliyun</id>
		<name>Nexus aliyun</name>
		<url>http://maven.aliyun.com/nexus/content/groups/public/</url>
		<mirrorOf>*</mirrorOf>
	</mirror> 

	<!-- 由 Apache 提供的仓库 -->
	<mirror>
		<id>apache-repo</id>  
		<name>apache Repository</name>  
		<url>https://repository.apache.org/content/groups/public/</url> 
		<mirrorOf>apache-repo</mirrorOf> <!-- 注意，这里必须写 apache-repo -->
	</mirror> 
  </mirrors>
```



#### 2.3.4、最后的 settings.xml



```xml
<?xml version="1.0" encoding="UTF-8"?>


<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
    
  <!-- localRepository
   | The path to the local repository maven will use to store artifacts.
   |
   | Default: ${user.home}/.m2/repository
   -->
  <localRepository>D:/Repository/Synchronization</localRepository>
  
  <pluginGroups></pluginGroups>

  <proxies></proxies>

  <servers></servers>

  <mirrors>
	<!-- 由阿里云提供的Maven中央仓库镜像 -->
	<mirror>
		<id>nexus-aliyun</id>
		<name>Nexus aliyun</name>
		<url>http://maven.aliyun.com/nexus/content/groups/public/</url>
		<mirrorOf>*</mirrorOf>
	</mirror> 

	<!-- 由 Apache 提供的仓库 -->
	<mirror>
		<id>apache-repo</id>  
		<name>apache Repository</name>  
		<url>https://repository.apache.org/content/groups/public/</url> 
		<mirrorOf>apache-repo</mirrorOf> <!-- 注意，这里必须写 apache-repo -->
	</mirror> 
  </mirrors>
 
  <profiles></profiles>

</settings>

```



## 3、IDEA



### 3.1、配置 maven



#### 3.1.1、 打开设置界面

在 以下界面中点击右下角  "Configure" 右侧的 三角形，随后点击 "Settings" 选项即可打开设置界面。

![idea01.png](https://images.gitee.com/uploads/images/2019/0906/191601_32c227f7_4771190.png "idea01.png")


#### 3.1.2、 设置软件主题和字体



在 "Settings for New Projects" 窗口中选择  "Appearance & Behavior" 下面的 "Appearance"。

![![](idea02.png)
](https://images.gitee.com/uploads/images/2019/0906/191507_ec83750f_4771190.png "idea02.png")


> 在右侧的  "Theme" 选项中可以通过 下拉列表来切换 主题。



> 勾选 "Use custom font " 后，可以设置 "字体" 和 字体尺寸



#### 3.1.3、在 IDEA 中配置 maven



在 "Settings for New Projects" 窗口中选择 "**Build , Execution , Deployment**" 下方的 "**Build Tools**" 下方的 "**Maven**"，如下图所示:

![![](idea03.png)](https://images.gitee.com/uploads/images/2019/0906/191624_2d0af42a_4771190.png "idea03.png")



>  在右侧 "**Maven home directory**" 同一行最右侧的 `...` 按钮上点击，并选择到 apache-maven 的主目录。
>
> 所谓的 apache-maven 主目录，就是在解压 maven 时，apache-maven 的解压目录。



> 在右侧 "**User settings file**"同一行最右侧，勾选 "Override" 复选框，再在复选框之前的 输入框中点击 "文件夹" 图标，选择 apache-maven 的配置文件( 根据约定，应该是 X:\applications\apache-maven-3.6.0\conf\settings.xml )



修改后的界面如下图所示:

![![](idea04.png)](https://images.gitee.com/uploads/images/2019/0906/191634_1c48d25b_4771190.png "idea04.png")



最后点击整个界面的右下角的 "Apply" 按钮，再点击 "OK" 按钮。



### 3.2、创建 maven-webapp



#### 3.2.1、开启 "New Project" 界面

在 欢迎界面 ( 下图 ) 中点击 "Create New Project" 后即可弹出 "New Project" 界面。

![![](idea01.png)](https://images.gitee.com/uploads/images/2019/0906/191643_f309f57c_4771190.png "idea01.png")



 "New Project" 界面:

![![](idea05.png)](https://images.gitee.com/uploads/images/2019/0906/191652_5237982f_4771190.png "idea05.png")



#### 3.2.2、选择 maven 原型



##### 3.2.2.1、选择 maven

在 "New Project" 界面左侧列表中选择 `maven` :

![![](idea06.png)](https://images.gitee.com/uploads/images/2019/0906/191701_eeda6937_4771190.png "idea06.png")



##### 3.2.2.2、指定 Project SDK

因为是首次使用 IDEA 开发 Java 工程(包括 maven 工程)，因此这里的 "Project SDK"是尚未配置的，因此需要配置 SDK。

在 正上方的 "Project SDK" 同一行右侧点击`New...` 按钮，选择 JDK 的按照目录即可。

选择之后 "Project SDK" 就是相应的 JDK 。



##### 3.2.2.3、在 archetype 列表中选择 maven-webapp

在 "New Project" 界面中勾选 "**Create from archetype**" ，

然后选择 "**org.apache.maven.archetypes:maven-archetype-webapp**" ，如下图所示:

![![idea07](idea07.png)](https://images.gitee.com/uploads/images/2019/0906/191711_4a817412_4771190.png "idea07.png")

随后点击 "**Next**" 按钮，进入下一步 ( 如下图所示 )。

![![](idea08.png)](https://images.gitee.com/uploads/images/2019/0906/191720_ffcbbc92_4771190.png "idea08.png")

在这里，分别输入 GroupId 、ArtifactId 

- GroupId  :  cn.edu.ecut
- ArtifactId : hello

直接点击  "**Next**" 按钮，进入下一步 ( 如下图所示 )。

![![](idea09.png)](https://images.gitee.com/uploads/images/2019/0906/191728_f620e13e_4771190.png "idea09.png")

继续点击 "**Next**"，进入下一步( 如下图所示 )

![![](idea10.png)](https://images.gitee.com/uploads/images/2019/0906/191734_6851a857_4771190.png "idea10.png")

这里，根据约定，应该将 "**Project location**" 更改为 **X:\IdeaProjects\hello** ，修改的界面后如下图所示:

![![](idea11.png)](https://images.gitee.com/uploads/images/2019/0906/191752_1d48dca0_4771190.png "idea11.png")

最后，点击 "**Finish**" 即可开始创建工程。



在工程构建过程中，IDEA软件界面右下角可能提示:

![![](idea12.png)](https://images.gitee.com/uploads/images/2019/0906/191742_1268e73e_4771190.png "idea12.png")



点选 "**Enable Auto-Import**"即可。



随后进入热烈的项目构建过程中，最后在软件界面下方看到 "**BUILD SUCCESS**" 即表示项目构建成功。

![![](idea13.png)](https://images.gitee.com/uploads/images/2019/0906/191800_4163f85b_4771190.png "idea13.png")





### 3.3、配置 maven-webapp

打开 当前工程下的 pom.xml 文件，其中:

```xml
  <groupId>cn.edu.ecut</groupId>
  <artifactId>hello</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>war</packaging>
```

是当前工程最基本的信息。`<packaging>` 用于指定项目的大包方式。



#### 3.3.1、修改 properties 部分



替换 pom.xml 中 `<properties>` 中原来的内容为以下内容:

```xml
  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <!-- 根据自己的 Project SDK 来确定版本 -->
    <maven.compiler.source>12</maven.compiler.source>
    <maven.compiler.target>12</maven.compiler.target>
    <maven.compiler.compilerVersion>12</maven.compiler.compilerVersion>
  </properties>
```



#### 3.3.2、修改 dependencies 部分

替换 pom.xml 中 `<dependencies>` 中原来的内容为以下内容:

```xml
  <dependencies>

    <!-- 添加对 Servlet 的支持 -->
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>javax.servlet-api</artifactId>
      <version>4.0.1</version>
      <scope>provided</scope>
    </dependency>

    <!-- MySQL 驱动 -->
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>5.1.45</version>
    </dependency>

  </dependencies>
```



#### 3.3.3、修改 build 部分

替换 pom.xml 中 `<build>` 中原来的内容为以下内容:

```xml
  <build>
    <finalName>${project.artifactId}</finalName>
    <plugins>

      <!-- Maven Compiler Plugin -->
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.8.0</version>
        <configuration>
          <source>${maven.compiler.source}</source>
          <target>${maven.compiler.target}</target>
          <compilerVersion>${maven.compiler.compilerVersion}</compilerVersion>
          <encoding>${project.build.sourceEncoding}</encoding>
        </configuration>
      </plugin>

      <!-- Tomcat Maven Plugin -->
      <plugin>
        <groupId>org.apache.tomcat.maven</groupId>
        <artifactId>tomcat7-maven-plugin</artifactId>
        <version>2.2</version>
        <configuration>
          <port>8080</port>
          <path>/</path>
          <uriEncoding>UTF-8</uriEncoding>
          <useBodyEncodingForURI>true</useBodyEncodingForURI>
        </configuration>
      </plugin>

    </plugins>

  </build>
```



#### 3.3.4、追加 repositories 配置

在 pom.xml 文件内部的  `</project>` 添加以下内容:

```xml
  <repositories>
    <repository>
      <id>nexus-aliyun</id>
      <name>Nexus aliyun</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
      <releases>
        <enabled>true</enabled>
      </releases>
      <snapshots>
        <enabled>true</enabled>
      </snapshots>
    </repository>
    <repository>
      <id>apache-repo</id>
      <name>apache Repository</name>
      <url>https://repository.apache.org/content/groups/public/</url>
      <releases>
        <enabled>true</enabled>
      </releases>
      <snapshots>
        <enabled>true</enabled>
      </snapshots>
    </repository>
  </repositories>
```







