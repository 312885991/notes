##	JDBC 实现增删改查

### 一、创建相应的包，包结构如图所示

![](image/p1.png)

### 二、创建数据库 `yonyou` 

```mysql

create database yonyou;

create table students(
id int(3) primary key,
name varchar(20),
age int(3),
birthday date);

insert into students(id,name,age,birthday) values(1,'张三丰',20,'1998/2/3');
insert into students(id,name,age,birthday) values(2,'赵敏',21,'1998/3/3');
insert into students(id,name,age,birthday) values(3,'张无忌',20,'1997/5/3');
insert into students(id,name,age,birthday) values(4,'张三',21,'1997/5/3');
insert into students(id,name,age,birthday) values(5,'李四',22,'1997/5/3');
insert into students(id,name,age,birthday) values(6,'王五',25,'1997/5/3');

create table scores(
sid int(3),
cname varchar(20),
score numeric(4,1),
 foreign key(sid) references students(id)
);

insert into scores(sid,cname,score) values(1,'C++',95.5);
insert into scores(sid,cname,score) values(2,'C++',87.5);
insert into scores(sid,cname,score) values(3,'C++',0);
insert into scores(sid,cname,score) values(4,'C++',100.0);
insert into scores(sid,cname) values(5,'C++');
insert into scores(sid,cname) values(6,'C++');
```



### 三、编写 `JdbcUtils` 工具类，快速获取 `Connection连接`

- 编写的工具类如下：

```java
package cn.ecut.day02.utils;

import org.junit.Assert;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class JdbcUtils {
    /**
     * 单例设计模式
     */
    static private final JdbcUtils jdbcUtils=new JdbcUtils();

    /**
     * 构造器私有化
     */
    private JdbcUtils() {
    }

    /**
     * 提供对外获取实例的方法
     * @return
     */
    static public JdbcUtils getInstance(){
        return jdbcUtils;
    }

    /**
     * 建立连接
     * @param db 数据库名
     * @param username 用户名
     * @param password 密码
     * @return
     * @throws ClassNotFoundException
     * @throws SQLException
     */
    public Connection getConnection(String db,String username,String password) throws ClassNotFoundException, SQLException {
        String driver="com.mysql.cj.jdbc.Driver";
        String url="jdbc:mysql://localhost:3306/"+db+"?serverTimezone=Asia/Shanghai&useSSL=false";
        //加载驱动
        Class.forName(driver);
        //建立连接
        Connection connection = DriverManager.getConnection(url, username, password);
        //断言不为空
        Assert.assertNotNull(connection);
        //返回连接
        return connection;

    }
}
```



### 四、测试 是否能成功连接数据库

- 测试类如下：

```java
package cn.ecut.day02.test;

import cn.ecut.day02.utils.JdbcUtils;
import org.junit.Test;

import java.sql.Connection;
import java.sql.SQLException;

/**
 * @author 刘路生
 */
public class ConnectionTest {
    /**
     * 获取工具实例
     */
    private JdbcUtils jdbcUtils=JdbcUtils.getInstance();
    /**
     *获取连接
     */
    private Connection connection=jdbcUtils.getConnection("yonyou","root","123456");

    public ConnectionTest() throws SQLException, ClassNotFoundException {
    }

    @Test
    public void test01() throws SQLException {
        System.out.println(connection);
    }
}

```

- 连接成功

![](image/p2.png)



### 五、创建实体类

- 学生类

```java
package cn.ecut.day02.entity;

import java.sql.Date;
import java.util.List;

/**
 * 学生实体类
 */
public class Student {
    /**
     * 学生ID
     */
    private Integer id;
    /**
     * 学生姓名
     */
    private String name;
    /**
     * 学生年龄
     */
    private Integer age;
    /**
     * 学生生日
     */
    private Date birthday;

    @Override
    public String toString() {
        return "Student{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", age=" + age +
                ", birthday=" + birthday +
                '}';
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public Date getBirthday() {
        return birthday;
    }

    public void setBirthday(Date birthday) {
        this.birthday = birthday;
    }
}

```

- 成绩类

```java
package cn.ecut.day02.entity;

public class Score {
    /**
     * 学生ID
     */
    private Integer sid;
    /**
     * 课程名称
     */
    private String cname;
    /**
     * 成绩
     */
    private Double score;

    @Override
    public String toString() {
        return "Score{" +
                "sid=" + sid +
                ", cname='" + cname + '\'' +
                ", score=" + score +
                '}';
    }

    public Integer getSid() {
        return sid;
    }

    public void setSid(Integer sid) {
        this.sid = sid;
    }

    public String getCname() {
        return cname;
    }

    public void setCname(String cname) {
        this.cname = cname;
    }

    public Double getScore() {
        return score;
    }

    public void setScore(Double score) {
        this.score = score;
    }
}

```

- 学生信息类

```java
package cn.ecut.day02.entity;

import java.sql.Date;
import java.util.List;

public class StudentInfo {
    /**
     * 学生ID
     */
    private Integer id;
    /**
     * 学生姓名
     */
    private String name;
    /**
     * 学生年龄
     */
    private Integer age;
    /**
     * 学生生日
     */
    private Date birthday;
    /**
     * 课程名
     */
    private String cname;
    /**
     * 学生成绩
     */
    private Double score;

    @Override
    public String toString() {
        return "StudentInfo{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", age=" + age +
                ", birthday=" + birthday +
                ", cname='" + cname + '\'' +
                ", score=" + ("NULL".equals(score)? "缺考" :score) +
                '}';
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public Date getBirthday() {
        return birthday;
    }

    public void setBirthday(Date birthday) {
        this.birthday = birthday;
    }

    public String getCname() {
        return cname;
    }

    public void setCname(String cname) {
        this.cname = cname;
    }

    public Double getScore() {
        return score;
    }

    public void setScore(Double score) {
        this.score = score;
    }
}

```

### 六、编写 数据持久层 `StudentDao`

- 代码如下：

```java
public class StudentDao {
    /**
     * 获取工具实例
     */
    private JdbcUtils jdbcUtils=JdbcUtils.getInstance();
    /**
     *获取连接
     */
    private Connection connection=jdbcUtils.getConnection("yonyou","root","123456");

    /**
     * 构造器
     * @throws SQLException
     * @throws ClassNotFoundException
     */
    public StudentDao() throws SQLException, ClassNotFoundException {
    }
}
```

####  1、查询函数

```java
/**
 * 根据学生ID查询该学生的具体信息
 * @param id 主键
 * @return 学生具体信息
 * @throws SQLException
 */
public List<StudentInfo> getStudentById(int id) throws SQLException {
    String sql="select id,name,age,birthday,cname,score from students inner join scores on students.id=scores.sid where id=?";
    ps = connection.prepareStatement(sql);
    ps.setInt(1,id);
    rs = ps.executeQuery();
    List<StudentInfo> list=new ArrayList<>();
    while(rs.next()){
        StudentInfo studentInfo=new StudentInfo();
        studentInfo.setId(rs.getInt("id"));
        studentInfo.setName(rs.getString("name"));
        studentInfo.setAge(rs.getInt("age"));
        studentInfo.setBirthday(rs.getDate("birthday"));
        studentInfo.setCname(rs.getString("cname"));
        studentInfo.setScore(rs.getDouble("score"));
        list.add(studentInfo);
    }
    //释放资源
    this.release(ps,rs,connection);
    return list;
}
```

####  2、插入函数

```java
/**
 * 插入学生信息
 * @param student
 * @throws SQLException
 */
public void insertStudent(Student student) throws SQLException {
    String sql="insert into students(id,name,age,birthday) values(?,?,?,?)";
    ps = connection.prepareStatement(sql);
    ps.setInt(1,student.getId());
    ps.setString(2,student.getName());
    ps.setInt(3,student.getAge());
    ps.setDate(4,student.getBirthday());
    ps.executeUpdate();
    //释放资源
    this.release(ps,rs,connection);
}
```

#### 3、删除函数

```java
/**
 * 删除学生信息
 * @param id 主键
 * @throws SQLException
 */
public void deleteStudentById(Integer id) throws SQLException {
    String sql="delete from students where id=?";
    ps = connection.prepareStatement(sql);
    ps.setInt(1,id);
    ps.executeUpdate();
    //释放资源
    this.release(ps,rs,connection);
}
```

#### 4、更新函数

```java
/**
 * 根据ID修改学生的年纪
 * @param id 主键
 * @param age 需要修改的年龄
 * @throws SQLException
 */
public void updateStudentAgeById(Integer id, Integer age) throws SQLException {
    String sql="update students set age=? where id=?";
    ps = connection.prepareStatement(sql);
    ps.setInt(1,age);
    ps.setInt(2,id);
    ps.executeUpdate();
    //释放资源
    this.release(ps,rs,connection);
}
```

#### 5、释放资源函数

```java
/**
 * 释放资源
 * @param ps
 * @param rs
 * @param connection
 */
private void release(PreparedStatement ps,ResultSet rs,Connection connection){
    if(ps!=null){
        try {
            ps.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }finally {
            if(rs!=null){
                try{
                    rs.close();
                }catch (SQLException e){
                    e.printStackTrace();
                }finally {
                    if(connection!=null){
                        try{
                            connection.close();
                        }catch (SQLException e){
                            e.printStackTrace();
                        }
                    }
                }
            }
        }
    }
}
```



### 七、编写 服务层 `StudentService`

- 代码如下：

```java
package cn.ecut.day02.service;

import cn.ecut.day02.dao.StudentDao;
import cn.ecut.day02.entity.Student;
import cn.ecut.day02.entity.StudentInfo;

import java.sql.SQLException;
import java.util.List;

public class StudentService {
    /**
     * 实例化StudentDao对象
     */
    private StudentDao studentDao=new StudentDao();

    public StudentService() throws SQLException, ClassNotFoundException {
    }

    /**
     * 根据学生ID查询该学生的具体信息
     * @param id 主键
     * @return 学生具体信息
     * @throws SQLException
     */
    public List<StudentInfo> getStudentById(int id) throws SQLException {
        return studentDao.getStudentById(id);
    }

    /**
     * 插入学生信息
     * @param student
     * @throws SQLException
     */
    public void insertStudent(Student student) throws SQLException {
        studentDao.insertStudent(student);
    }

    /**
     * 删除学生信息
     * @param id 主键
     * @throws SQLException
     */
    public void deleteStudentById(Integer id) throws SQLException {
       studentDao.deleteStudentById(id);
    }

    /**
     * 根据ID修改学生的年纪
     * @param id 主键
     * @param age 需要修改的年龄
     * @throws SQLException
     */
    public void updateStudentAgeById(Integer id, Integer age) throws SQLException {
        studentDao.updateStudentAgeById(id,age);
    }
}

```

### 八、编写测试功能类

#### 1、测试查询

```java
/**
 * 测试查询
 */
@Test
public void testQuery() throws SQLException {
    List<StudentInfo> studentInfo = studentService.getStudentById(6);
    for(StudentInfo s:studentInfo){
        System.out.println(s);
    }
}
```

- 测试结果：

![](image/p3.png)

#### 2、测试插入

```java
/**
 * 测试插入
 */
@Test
public void testInsert() throws SQLException {
    Student student=new Student();
    student.setId(8);
    student.setName("刘路生");
    student.setAge(20);
    studentService.insertStudent(student);
}
```



#### 3、测试删除

```java
/**
 * 测试删除
 */
@Test
public void testDelete() throws SQLException {
    studentService.deleteStudentById(7);
}
```



#### 4、测试更新

```java
/**
 * 测试更新
 */
@Test
public void testUpdate() throws SQLException {
    studentService.updateStudentAgeById(2,23);
}
```



