# Mybatis

## 快速入门

[官方文档](https://mybatis.org/mybatis-3/zh/getting-started.html)

spring-boot 与 mybatis 整合: [MyBatis-Spring-Boot-Starter ](http://mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure/index.html)

使用 spring-boot 可以简化大量工作. 以下介绍 spring boot 方案.

使用 mybatis 的步骤:

1. pom.xml 中添加 mybatis-spring-boot-starter
2. pom.xml 中添加数据库驱动依赖
3. 配置文件中添加数据库连接信息
4. 创建数据库和表
5. 创建实体Bean
6. 编写 mybatis 的配置文件,  Mapper xml
7. 在主启动类加入 @MapperScan("{package-name}"), 可以不用在每个 Mapper 类上加 @Mapper 注解
8. 编写 Controller
9. 启动 SpringBoo 引导类

官方还是推荐使用 xml 来映射语句(使用注解来映射简单语句会使代码显得更加简洁，但对于稍微复杂一点的语句，Java 注解不仅力不从心，还会让你本就复杂的 SQL 语句更加混乱不堪。因此，如果你需要做一些很复杂的操作，最好用 XML 来映射语句。).

## 关键步骤

### 添加依赖

下面例子中先使用 [spring initializr](https://start.spring.io/)添加了 mybatis 依赖. 然后手动添加 sqlite-jdbc, druid, lombok 依赖, 整合 durid

```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.1.3</version>
</dependency>
<!-- https://mvnrepository.com/artifact/org.xerial/sqlite-jdbc -->
<dependency>
    <groupId>org.xerial</groupId>
    <artifactId>sqlite-jdbc</artifactId>
    <version>3.34.0</version>
</dependency>

<!-- https://mvnrepository.com/artifact/com.alibaba/druid -->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.2.4</version>
</dependency>

<!-- https://mvnrepository.com/artifact/org.projectlombok/lombok -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.18</version>
    <scope>provided</scope>
</dependency>
<!-- https://mvnrepository.com/artifact/com.alibaba/druid-spring-boot-starter -->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.2.4</version>
</dependency>


```

### 数据源配置

application.yml. 以 sqlite3 为例.

```yml
spring:
    datasource:
    	driver-class-name: org.sqlite.JDBC
        url: jdbc:sqlite:src\main\resources\db\optools.db
        username:
        password:
        

```

使用 druid .

*注意*: 

1. sqlite不支持spring.datasource.filters的wall，请去掉
2. spring.datasource.type=com.alibaba.druid.pool.DruidDataSource这一行以下的连接池的配置无法被druid加载，需要自己将配置设置到druid里，按照官网配置也不行。

```yml

server:
    port: 8080
    
spring:
  datasource:
    # 数据库访问配置, 使用druid数据源(默认数据源是HikariDataSource)
     type: com.alibaba.druid.pool.DruidDataSource
     driver-class-name: org.sqlite.JDBC
     url: jdbc:sqlite:src\main\resources\db\optools.db
     username: 
     password: 
     #链接池配置
     druid:
       # 连接池配置：大小，最小，最大
       initial-size: 5
       min-idle: 5
       max-active: 20
       
       # 连接等待超时时间
       max-wait: 30000
       
       # 配置检测可以关闭的空闲连接，间隔时间
       time-between-eviction-runs-millis: 60000
       
       # 配置连接在池中的最小生存时间
       min-evictable-idle-time-millis: 300000
       # 检测连接是否有，有效得select语句
       validation-query: select '1'
       # 申请连接的时候检测，如果空闲时间大于time-between-eviction-runs-millis，执行validationQuery检测连接是否有效，建议配置为true，不影响性能，并且保证安全性。
       test-while-idle: true
       # 申请连接时执行validationQuery检测连接是否有效，建议设置为false，不然会会降低性能
       test-on-borrow: false
       # 归还连接时执行validationQuery检测连接是否有效，建议设置为false，不然会会降低性能
       test-on-return: false
       
       # 是否缓存preparedStatement，也就是PSCache  官方建议MySQL下建议关闭   个人建议如果想用SQL防火墙 建议打开
       # 打开PSCache，并且指定每个连接上PSCache的大小
       # sqlite 需要关闭. 否则,报错.
       #pool-prepared-statements: true
       #max-open-prepared-statements: 20
       #max-pool-prepared-statement-per-connection-size: 20
       
       # 配置监控统计拦截的filters, 去掉后监控界面sql无法统计, 'wall'用于防火墙防御sql注入，stat监控统计,logback日志
       # sqlite不支持spring.datasource.filters的wall，请去掉    
       #filters: stat,wall
       # Spring监控AOP切入点，如x.y.z.service.*,配置多个英文逗号分隔
       #aop-patterns: com.springboot.servie.*
       # lowSqlMillis用来配置SQL慢的标准，执行时间超过slowSqlMillis的就是慢
       connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=500
       
       # WebStatFilter监控配置
       web-stat-filter:
         enabled: true
         # 添加过滤规则：那些访问拦截统计
         url-pattern: /*
         # 忽略过滤的格式：哪些不拦截，不统计
         exclusions: '*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*'
       
       # StatViewServlet配置（Druid监控后台的Servlet映射配置，因为SpringBoot项目没有web.xml所在在这里使用配置文件设置） 
       stat-view-servlet:
         enabled: true 
         # 配置Servlet的访问路径：访问路径为/druid/**时，跳转到StatViewServlet，会自动转到Druid监控后台
         url-pattern: /druid/*
         # 是否能够重置数据
         reset-enable: false
         # 设置监控后台的访问账户及密码
         login-username: xsge
         login-password: xsge
         # IP白名单：允许哪些主机访问，默认为“”任何主机
         # allow: 127.0.0.1
         # IP黑名单：禁止IP访问，（共同存在时，deny优先于allow）
         # deny: 192.168.1.218
       
       # 配置StatFilter
       filter: 
         stat: 
           log-slow-sql: true
mybatis:
    configuration:
        use-generated-keys: true
        map-underscore-to-camel-case: true

```

druid 配置类 [官方文档](https://github.com/alibaba/druid/tree/master/druid-spring-boot-starter)

```java
package cn.com.fmsh.optools.config;

import javax.sql.DataSource;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;

import com.alibaba.druid.spring.boot.autoconfigure.DruidDataSourceBuilder;

@Configuration
public class DruidConfiguration {

	private static final Logger logger = LoggerFactory.getLogger(DruidConfiguration.class);

	@Bean
	@Primary
	@ConfigurationProperties(prefix = "spring.datasource")
	public DataSource dataSource() {
		return DruidDataSourceBuilder.create().build();
	}

	/*
	 * @Bean public FilterRegistrationBean filterRegistrationBean() {
	 * FilterRegistrationBean filterRegistrationBean = new
	 * FilterRegistrationBean(new WebStatFilter());
	 * filterRegistrationBean.addUrlPatterns("/*");
	 * filterRegistrationBean.addInitParameter("exclusions",
	 * "*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*"); return
	 * filterRegistrationBean; }
	 */
}

```



### 创建数据库和表

以 sqlite3 为例. sqlite3 中创建表, 自增字段需要同时指定 INTEGER PRIMARY KEY AUTOINCREMENT

```sql
-- tbl_user definition

CREATE TABLE "tbl_user" (
	"id" INTEGER PRIMARY KEY AUTOINCREMENT,
	"name" VARCHAR(32) NOT NULL,
	"password" VARCHAR(64) NOT NULL,
	"email" VARCHAR(64) NOT NULL,
	"phone" VARCHAR(20) NOT NULL
);

CREATE UNIQUE INDEX tbl_user_email_IDX ON tbl_user (email);
```

### 创建实体 Bean

```java
// 使用了 lombok 简化代码
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
	private Integer id;
	private String name;
	private String password;
	private String email;
	private String phone;

}

```

### 配置 mybatis, 创建 Mapper

一般 mapper xml , mybatis 配置和 Mapper 类在同一个目录内

简单的 mapper xml 如下:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="org.mybatis.example.BlogMapper">
  <select id="selectBlog" resultType="Blog">
    select * from Blog where id = #{id}
  </select>
</mapper>
```

说明: namespace 是包名, select 标签表示 select 语句. id 表示唯一标识, mybatis 使用该 id 来使用映射语句. 