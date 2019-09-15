---
title: \#Fullstack\# 数据库实训“活在华工”微信小程序 - 后端实现(spring boot + maven)
date: 2019-07-21 15:13:21
categories: Fullstack
tags:
- sprint boot
- maven
thumbnail: /images/dbpractice_backend.png
---

**数据库实训“活在华工”微信小程序 - 后端实现(spring boot + maven)**

<!-- more -->

## 目录结构

- *.mvn*：maven环境
- *.setting*：project设置
- .vscode：launch设置
- ***src*：源码文件**
- *target*：生成文件
- ***scripts*：数据库脚本**
- mvnw：maven源代码
- mvnw.md：maven控制台
- .classpath：Java class路径
- .factorypath：Java factory路径
- .project：Java project设置
- pom.xml：maven设置
- .gitignore：版本管理忽略文件
- README.md：说明文档
- LICENSE：开源证书

## .vscode

### launch.json

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "java",
      "name": "Debug (Launch) - DBPracticeApplication(demo)",
      "request": "launch",
      "cwd": "${workspaceFolder}",
      "console": "internalConsole",
      "stopOnEntry": false,
      "mainClass": "src.main.java.com.example.dbpractice.DBPracticeApplication",
      "args": ""
    },
    {
      "type": "java",
      "name": "Debug (Launch) - DBPracticeApplicationTests(demo)",
      "request": "launch",
      "cwd": "${workspaceFolder}",
      "console": "internalConsole",
      "stopOnEntry": false,
      "mainClass": "src.test.java.com.example.dbpractice.DBPracticeApplicationTests",
      "args": ""
    }
  ]
}
```

## scripts

### dbpractice.sql

```mysql
-- 使用UTF-8编码
SET NAMES utf8mb4;

-- 创建数据库`dbpractice`
CREATE DATABASE `dbpractice`;

-- 使用数据库`dbpractice`
USE `dbpractice`;


-- 检查是否已经存在相应数据表`activity`
DROP TABLE IF EXISTS `activity`;

-- 使用UTF-8编码
SET character_set_client = utf8mb4;

-- 创建数据表`activity`
CREATE TABLE `activity` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `title` varchar(30) NOT NULL,
  `date` datetime DEFAULT NULL,
  `place` varchar(50) DEFAULT NULL,
  `desc` varchar(500) DEFAULT NULL,
  `sponsor` varchar(15) NOT NULL,
  `certified` tinyint(1) DEFAULT '0',
  PRIMARY KEY (`id`),
  KEY `sponsor` (`sponsor`),
  CONSTRAINT `activity_ibfk_1` FOREIGN KEY (`sponsor`) REFERENCES `user` (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- 向数据表`activity`插入数据
LOCK TABLES `activity` WRITE;
/* ... */;
UNLOCK TABLES;


-- 检查是否已经存在相应数据表`participant_activity`
DROP TABLE IF EXISTS `participant_activity`;

-- 使用UTF-8编码
SET character_set_client = utf8mb4;

-- 创建数据表`participant_activity`
CREATE TABLE `participant_activity` (
  `uid` varchar(15) NOT NULL,
  `aid` int(11) NOT NULL,
  PRIMARY KEY (`uid`,`aid`),
  KEY `aid` (`aid`),
  CONSTRAINT `participant_activity_ibfk_1` FOREIGN KEY (`uid`) REFERENCES `user` (`id`),
  CONSTRAINT `participant_activity_ibfk_2` FOREIGN KEY (`aid`) REFERENCES `activity` (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- 向数据表`participant_activity`插入数据
LOCK TABLES `participant_activity` WRITE;
/* ... */;
UNLOCK TABLES;


-- 检查是否已经存在相应数据表`user`
DROP TABLE IF EXISTS `user`;

-- 使用UTF-8编码
SET character_set_client = utf8mb4;

-- 创建数据表`user`
CREATE TABLE `user` (
  `id` varchar(15) NOT NULL,
  `name` varchar(10) NOT NULL,
  `major` varchar(50) DEFAULT NULL,
  `address` varchar(50) DEFAULT NULL,
  `phone` varchar(20) DEFAULT NULL,
  `email` varchar(20) DEFAULT NULL,
  `wechat` varchar(20) DEFAULT NULL,
  `qq` varchar(20) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- 向数据表`user`插入数据
LOCK TABLES `user` WRITE;
/* ... */
UNLOCK TABLES;
```

## src

### main.java.com.example.dbpractice.config配置层

#### dao

##### DataSourceConfiguration.java

```java
// Dao层配置之：配置DataSource
@Configuration
@MapperScan("com.example.dbpractice.dao")
public class DataSourceConfiguration {
    @Value("${jdbc.driver}")
    private String jdbcDriver;
    @Value("${jdbc.url}")
    private String jdbcUrl;
    @Value("${jdbc.username}")
    private String jdbcUser;
    @Value("${jdbc.password}")
    private String jdbcPassword;

    @Bean(name = "dataSource")
    public ComboPooledDataSource createDateSource() throws PropertyVetoException {
        ComboPooledDataSource dataSource=new ComboPooledDataSource();
        dataSource.setDriverClass(jdbcDriver);
        dataSource.setJdbcUrl(jdbcUrl);
        dataSource.setUser(jdbcUser);
        dataSource.setPassword(jdbcPassword);
        // 关闭连接后不自动commit
        dataSource.setAutoCommitOnClose(false);
        return dataSource;
    }
}
```

##### SessionFactoryConfiguration.java

```java
// Dao层配置之：配置SessionFactory
@Configuration
public class SessionFactoryConfiguration {
    @Value("${mybatis_config_file}")
    private String mybatisConfigFilePath;
    @Value("${mapper_path}")
    private String mapperPath;
    @Value("${entity_package}")
    private String entityPackage;
    @Autowired
    @Qualifier("dataSource")
    private DataSource dataSource;

    @Bean(name = "sqlSessionFactory")
    public SqlSessionFactoryBean createSqlSessionFactoryBean() throws IOException {
        SqlSessionFactoryBean sqlSessionFactoryBean=new SqlSessionFactoryBean();
   		// mybatis文件扫描路径
        sqlSessionFactoryBean.setConfigLocation(new ClassPathResource(mybatisConfigFilePath));
        
        // mapper文件扫描路径
        PathMatchingResourcePatternResolver resolver=new PathMatchingResourcePatternResolver();
        String packageSearchPath = PathMatchingResourcePatternResolver.CLASSPATH_ALL_URL_PREFIX + mapperPath;
        sqlSessionFactoryBean.setMapperLocations(resolver.getResources(packageSearchPath));
        
        // entity包扫描路径
        sqlSessionFactoryBean.setTypeAliasesPackage(entityPackage);
        return sqlSessionFactoryBean;
        
        // data源扫描路径
        sqlSessionFactoryBean.setDataSource(dataSource);
    }
}
```

#### service

##### TransactionManagementConfiguration.java

```java
// Service层配置之：配置TransactionManagement
@Configuration
@EnableTransactionManagement
public class TransactionManagementConfiguration implements TransactionManagementConfigurer {

    @Autowired
    private DataSource dateSource;
    @Override
    public PlatformTransactionManager annotationDrivenTransactionManager() {
        // 启动事务管理机制
        return new DataSourceTransactionManager(dateSource);
    }
}
```

### main.java.com.example.dbpractice.handler异常层

```java
@ControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(value = Exception.class)
    @ResponseBody    
    // 不是返回html而是返回错误信息
    private Map<String,Object> exceptionHandler (HttpServletRequest req, Exception e){
        Map<String,Object> modelMap = new HashMap<>();
        modelMap.put("success", false);
        modelMap.put("errMsg", e.getMessage());
        return modelMap;
    }
}
```

### main.java.com.example.dbpractice.entity实体层（表结构定义）

#### Acitivity.java

```java
package com.example.dbpractice.entity;

import java.util.Date;

public class Activity {
    // 活动id 自增
    private Integer id;
    // 活动标题
    private String title;
    // 活动时间
    private Date date;
    // 活动地点
    private String place;
    // 活动类别
    private String tag;
    // 活动描述 500以内
    private String desc;
    // 活动发起人 uid
    private String sponsor;
    // 活动是否官方认证
    private Boolean certified;

    // Constructor
    public Activity(Integer id, String title, Date date, String place,
                    String tag, String desc, String sponsor, Boolean certified) {
        this.id = id;
        this.title = title;
        this.date = date;
        this.place = place;
        this.tag = tag;
        this.desc = desc;
        this.sponsor = sponsor;
        this.certified = certified;
    }

    // Getters and setters
    // ...
}
```

#### Participant_Activity.java

```java
package com.example.dbpractice.entity;

public class Participant_Activity {
    // 参与者id
    private String p_uid;
    // 参与的活动id
    private Integer aid;

    // Constructor
    public Participant_Activity(String p_uid, Integer aid) {
        this.p_uid = p_uid;
        this.aid = aid;
    }
	
    // Getters and setters
    // ...
}
```

#### User.java

```java
package com.example.dbpractice.entity;

public class User {
    // 用户学号
    private String uid;
    // 用户姓名
    private String name;
    // 用户专业
    private String major;
    // 用户地址
    private String address;
    // 用户手机
    private String phone;
    // 用户邮箱
    private String email;
    // 用户微信
    private String wechat;
    // 用户QQ
    private String qq;

    // Constructor
    public User(String uid, String name, String address, String contact) {
        this.uid=uid;
        this.name=name;
        this.address=address;
        this.contact=contact;
    }

    // Getters and setters
    // ...
}
```

### main.java.com.example.dbpractice.controller控制层（前后端沟通）

#### DBController.java

```java
package com.example.dbpractice.controller;

import com.example.dbpractice.entity.Activity;
import com.example.dbpractice.entity.Participant_Activity;
import com.example.dbpractice.entity.User;
import com.example.dbpractice.service.DBService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import java.util.HashMap;
import java.util.List;
import java.util.Map;

@RestController
@RequestMapping(value = "/dbadmin")
public class DBController {
    @Autowired
    private DBService DBService;

    // 针对user表的可选操作
    // 返回用户列表
    @RequestMapping(value = "/listuser",method = RequestMethod.GET)
    private Map<String,Object> getUserList(){
        Map<String, Object> modelMap = new HashMap<>();
        List<User> userList = DBService.getUserList();
        modelMap.put("userList",userList);
        return modelMap;
    }

    // 返回uid指定的用户
    @RequestMapping(value = "/getuserbyid",method = RequestMethod.GET)
    private Map<String,Object> getUserById(String uid){
        Map<String, Object> modelMap = new HashMap<>();
        User user = DBService.getUserById(uid);
        modelMap.put("user",user);
        return modelMap;
    }

    // 新增用户
    @RequestMapping(value = "/adduser",method = RequestMethod.POST)
    private Map<String,Object> addUser(@RequestBody User userToAdd){
        Map<String, Object> modelMap = new HashMap<>();
        modelMap.put("success", DBService.addUser(userToAdd));
        return modelMap;
    }

    // 修改用户信息
    @RequestMapping(value = "/modifyuser",method = RequestMethod.POST)
    private Map<String,Object> modifyUser(@RequestBody User userToModify){
        Map<String, Object> modelMap = new HashMap<>();
        modelMap.put("success", DBService.modifyUser(userToModify));
        return modelMap;
    }

    // 删除用户
    @RequestMapping(value = "/deleteuser",method = RequestMethod.GET)
    private Map<String,Object> deleteUser(String uid){
        Map<String, Object> modelMap = new HashMap<>();
        modelMap.put("success", DBService.deleteUser(uid));
        return modelMap;
    }



    //针对activity表的可选操作
    //返回活动列表
    @RequestMapping(value = "/listactivity",method = RequestMethod.GET)
    private Map<String,Object> getActvityList(){
        Map<String, Object> modelMap = new HashMap<>();
        List<Activity> activityList = DBService.getActivityList();
        modelMap.put("activityList",activityList);
        return modelMap;
    }

    // 返回aid指定的活动
    @RequestMapping(value = "/getactivitybyid",method = RequestMethod.GET)
    private Map<String,Object> getActivityById(int uid){
        Map<String, Object> modelMap = new HashMap<>();
        Activity activity = DBService.getActivityById(uid);
        modelMap.put("activity",activity);
        return modelMap;
    }

    // 新增活动
    @RequestMapping(value = "/addactivity",method = RequestMethod.POST)
    // TODO: 修改RequestBody为RequestParam以通过小程序POST请求
    private Map<String,Object> addUser(@RequestParam Activity activityToAdd){
        Map<String, Object> modelMap = new HashMap<>();
        modelMap.put("success", DBService.addActivity(activityToAdd));
        return modelMap;
    }

    // 修改活动信息
    @RequestMapping(value = "/modifyactivity",method = RequestMethod.POST)
    private Map<String,Object> modifyUser(@RequestBody Activity activityToModify){
        Map<String, Object> modelMap = new HashMap<>();
        modelMap.put("success", DBService.modifyActivity(activityToModify));
        return modelMap;
    }

    // 删除活动
    @RequestMapping(value = "/deleteactivity",method = RequestMethod.GET)
    private Map<String,Object> deleteActivity(int uid){
        Map<String, Object> modelMap = new HashMap<>();
        modelMap.put("success", DBService.deleteActivity(uid));
        return modelMap;
    }



    // 针对participant_activity表的可选操作
    // 返回参与关系列表
    @RequestMapping(value = "/listparticipantactivity",method = RequestMethod.GET)
    private Map<String,Object> getParticipantActivityList(){
        Map<String, Object> modelMap = new HashMap<>();
        List<Participant_Activity> paList = DBService.getParticipantActivityList();
        modelMap.put("paList",paList);
        return modelMap;
    }

    // 返回uid指定的用户参与的活动
    @RequestMapping(value = "/getactivitybyparticipantid",method = RequestMethod.GET)
    private Map<String,Object> getActivityByParticipantId(String uid){
        Map<String, Object> modelMap = new HashMap<>();
        List<Activity> activityList = DBService.getActivityByParticipantId(uid);
        modelMap.put("activityList",activityList);
        return modelMap;
    }

    // 返回aid指定的活动的参与者
    @RequestMapping(value = "/getparticipantbyactivityid",method = RequestMethod.GET)
    private Map<String,Object> getParticipantByActivityId(int aid){
        Map<String, Object> modelMap = new HashMap<>();
        List<User> userList = DBService.getParticipantByActivityId(aid);
        modelMap.put("userList","userList");
        return modelMap;
    }

    // 新增参与关系
    @RequestMapping(value = "/addparticipantactivity",method = RequestMethod.POST)
    private Map<String,Object> addUser(@RequestBody Participant_Activity p_aToAdd){
        Map<String, Object> modelMap = new HashMap<>();
        modelMap.put("success", DBService.addParticipantActivity(p_aToAdd));
        return modelMap;
    }

    //删除参与关系
    @RequestMapping(value = "/deleteparticipantactivity",method = RequestMethod.GET)
    private Map<String,Object> deleteParticipantActivity(Participant_Activity p_aToDelete){
        Map<String, Object> modelMap = new HashMap<>();
        modelMap.put("success", DBService.deleteParticipantActivity(p_aToDelete));
        return modelMap;
    }
}
```

### main.java.com.example.dbpractice.handler服务层（表操作封装）

#### DBService.java

```java
package com.example.dbpractice.service;

import com.example.dbpractice.entity.Activity;
import com.example.dbpractice.entity.Participant_Activity;
import com.example.dbpractice.entity.User;

import java.util.List;

public interface DBService {
    // activity表 操作
    // 查询全部活动
    List<Activity> getActivityList();
    // 查询全部官方活动
    List<Activity> getCertifiedActivity();
    // 以aid查询活动
    Activity getActivityById(int aid);
    // 以sponsor的id查询活动
    List<Activity> getActivityBySponsorId(int s_uid);
    // 插入活动记录
    boolean addActivity(Activity activity);
    // 更新活动记录
    boolean modifyActivity(Activity activity);
    // 删除活动记录
    boolean deleteActivity(int aid);

    // participant_activity表 操作
    // 返回所有
    List<Participant_Activity> getParticipantActivityList();
    // 以活动id查询参与者
    List<User> getParticipantByActivityId(int aid);
    // 以参与者id查询活动
    List<Activity> getActivityByParticipantId(String p_uid);
    // 新增参与关系
    boolean addParticipantActivity(Participant_Activity p_a);
    // 删除参与关系
    boolean deleteParticipantActivity(Participant_Activity p_a);
    
    // user表 操作
    // 返回所有
    List<User> getUserList();
    // 以id查询用户
    User getUserById(String uid);
    // 新增用户
    boolean addUser(User user);  //返回的int是数据库操作受影响的行数
    // 更新用户信息
    boolean modifyUser(User user);
    // 删除用户
    boolean deleteUser(String uid);
}
```

#### impl

##### DBServiceImpl.java

```java
package com.example.dbpractice.service.impl;

import com.example.dbpractice.dao.ActivityDao;
import com.example.dbpractice.dao.Participant_ActivityDao;
import com.example.dbpractice.dao.UserDao;
import com.example.dbpractice.entity.Activity;
import com.example.dbpractice.entity.Participant_Activity;
import com.example.dbpractice.entity.User;
import com.example.dbpractice.service.DBService;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class DBServiceImpl implements DBService {
    @Autowired
    private UserDao userDao;
    @Autowired
    private ActivityDao activityDao;
    @Autowired
    private Participant_ActivityDao p_aDao;
    
    // activity表操作的实现
    // ...

    // participant_activity表操作的实现
    // ...
    
    // user表操作的实现
    // ...
}
```

### main.java.com.example.dbpractice.dao接口层（表操作实现）

#### ActivityDao.java

```java
package com.example.dbpractice.dao;

import com.example.dbpractice.entity.Activity;

import java.util.List;

public interface ActivityDao {
    // 新增活动记录
    int insertActivity(Activity activity);
    
    // 删除活动记录
    int deleteActivity(int aid);
    
    // 查询全部活动记录
    List<Activity> queryActivity();
    // 查询官方活动记录
    List<Activity> queryCertifiedActivity();
    // 以activity的id查询活动记录
    Activity queryActivityByActivityId(int aid);
    // 以sponsor的id查询活动记录
    List<Activity> queryActivityBySponsorId(int sid);
    // TODO: 通过其他字段查询活动记录
    // ...

    // 更新活动记录
    int updateActivity(Activity activity);    
}
```

#### Participant_ActivityDao.java

```java
package com.example.dbpractice.dao;

import com.example.dbpractice.entity.Activity;
import com.example.dbpractice.entity.Participant_Activity;
import com.example.dbpractice.entity.User;

import java.util.List;

public interface Participant_ActivityDao {
    // 新增参与关系
    int insertParticipantActivity(Participant_Activity p_a);
    
    // 删除参与关系
    int deleteParticipantActivity(Participant_Activity p_a);
    
    // 查询参与关系
    List<Participant_Activity> queryParticipantActivity();
    // 以参与者的id+活动的id查询参与关系
    Participant_Activity queryOneParticipantActivity(Participant_Activity p_a);
    // 以活动的id查询参与者
    List<User> queryParticipantByActivityId(int aid);
    // 以参与者的id查询活动
    List<Activity> queryActivityByParticipantId(String pid);
    
    // 更新参与关系
    // ...
}
```

#### UserDao.java

```java
package com.example.dbpractice.dao;

import com.example.dbpractice.entity.User;

import java.util.List;

public interface UserDao {
    // 新增用户信息
    int insertUser(User user);
    
    //删除用户信息
    int deleteUser(String uid);
    
    // 查询用户信息
    List<User> queryUser();
    // 以用户的id查询用户信息
    User queryUserById(String uid);
    
    //更新用户信息
    int updateUser(User user);
}
```

### main.resources.Mapper映射层（表操作映射）

#### ActivityDao.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.dbpractice.dao.ActivityDao">
    <select id="queryActivity" resultType="com.example.dbpractice.entity.Activity">
        SELECT *
        FROM `activity`
    </select>
    <select id="queryCertifiedActivity" resultType="com.example.dbpractice.entity.Activity">
        SELECT *
        FROM `activity`
        WHERE `certified`=TRUE
    </select>
    <select id="queryActivityById"  resultType="com.example.dbpractice.entity.Activity">
        SELECT *
        FROM `activity`
        WHERE `id`=#{aid}
    </select>
    <select id="queryActivityBySponsorId" resultType="com.example.dbpractice.entity.Activity">
        SELECT `aid`
        FROM `activity`
        WHERE `sponsor`=#{s_uid}
    </select>
    <insert id="insertActivity" parameterType="com.example.dbpractice.entity.Activity"
            useGeneratedKeys="true" keyColumn="id" keyProperty="id">
        INSERT INTO `activity`
        VALUES (#{title},#{date},#{place},#{tag},#{desc},#{sponsor},#{certified})
    </insert>
    <update id="updateActivity" parameterType="com.example.dbpractice.entity.Activity">
        UPDATE `activity`
        <set>
            <if test="title!=null">`title`=#{title}</if>
            <if test="date!=null">`date`=#{date}</if>
            <if test="place!=null">`place`=#{place}</if>
            <if test="tag!=null">`tag`=#{tag}</if>
            <if test="desc!=null">`desc`=#{desc}</if>
            <if test="sponsor!=null">`sponsor`=#{sponsor}</if>
            <if test="certified!=null">`certified`=#{certified}</if>
        </set>
        WHERE `id`=#{id}
    </update>
    <delete id="deleteActivity">
        DELETE FROM `activity`
        WHERE `id`=#{aid}
    </delete>
</mapper>
```

#### Participant_ActivityDao.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.dbpractice.dao.Participant_ActivityDao">
    <select id="queryParticipantActivity" resultType="com.example.dbpractice.entity.Participant_Activity">
        SELECT *
        FROM `participant_activity`
    </select>
    <select id="queryParticipantByActivityId" resultType="com.example.dbpractice.entity.User">
        SELECT `uid`
        FROM `participant_activity`
        WHERE `aid`=#{aid}
    </select>
    <select id="queryActivityByParticipantId" resultType="com.example.dbpractice.entity.Activity">
        SELECT `aid`
        FROM `participant_activity`
        WHERE `uid`=#{p_uid}
    </select>
    <select id="queryOneParticipantActivity" resultType="com.example.dbpractice.entity.Participant_Activity">
        SELECT *
        FROM `participant_activity`
        WHERE `uid`=#{uid} AND `aid`=#{aid}
    </select>
    <insert id="insertParticipantActivity" parameterType="com.example.dbpractice.entity.Participant_Activity">
        INSERT INTO `participant_activity`
        VALUES (#{p_uid},#{aid})
    </insert>
    <delete id="deleteParticipantActivity" parameterType="com.example.dbpractice.entity.Participant_Activity">
        DELETE FROM `participant_activity`
        WHERE `uid`=#{p_uid} AND `aid`=#{aid}
    </delete>
</mapper>
```

#### UserDao.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.dbpractice.dao.UserDao">
    <select id="queryUser" resultType="com.example.dbpractice.entity.User" >
        SELECT `id`,`name`,`address`,`contact`
        FROM `user`
    </select>
    <select id="queryUserById" resultType="com.example.dbpractice.entity.User">
        SELECT `id`,`name`,`address`,`contact`
        FROM `user`
        WHERE `id`=#{uid}
    </select>
    <insert id="insertUser" parameterType="com.example.dbpractice.entity.User">
        INSERT INTO `user`(`id`,`name`,`address`,`contact`)
        VALUES(#{uid},#{name},#{address},#{contact})
    </insert>
    <update id="updateUser" parameterType="com.example.dbpractice.entity.User">
        update `user`
        <set>
            <if test="name!=null">`name`=#{name},</if>
            <if test="address!=null">`address`=#{address},</if>
            <if test="contact!=null">`contact`=#{contact}</if>
        </set>
        where `id`=#{uid}
    </update>
    <delete id="deleteUser">
        DELETE FROM `user`
        WHERE `id`=#{uid}
    </delete>
</mapper>
```

#### application.properties（project配置）

```properties
# server
server.port=8888
server.servlet.context-path=/dbpractice

# jbbc
jdbc.driver=com.mysql.cj.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/dbpractice?useUnicode=true&characterEncoding=utf-8&serverTimezone=UTC&useSSL=false
jdbc.username=root
jdbc.password=password

# mybatis
mybatis_config_file=mybatis-config.xml
mapper_path=/Mapper/*.xml
entity_package=com.example.dbpractice.entity
```

#### mybatis-config.xml（mybatis配置）

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

    <settings>
        <!-- Globally enables or disables any caches configured in any mapper under this configuration -->
        <setting name="cacheEnabled" value="false"/>
        <!-- Sets the number of seconds the driver will wait for a response from the database -->
        <setting name="defaultStatementTimeout" value="5"/>
        <!-- Enables automatic mapping from classic database column names A_COLUMN to camel case classic Java property names aColumn -->
        <setting name="mapUnderscoreToCamelCase" value="true"/>
        <!-- Allows JDBC support for generated keys. A compatible driver is required.
        This setting forces generated keys to be used if set to true,
         as some drivers deny compatibility but still work -->
        <setting name="useGeneratedKeys" value="true"/>
        <!-- Allows to support column references -->
        <setting name="useColumnLabel" value="true"/>
    </settings>

    <!-- Continue editing here -->

</configuration>
```

### test

// 单元测试

// ...



