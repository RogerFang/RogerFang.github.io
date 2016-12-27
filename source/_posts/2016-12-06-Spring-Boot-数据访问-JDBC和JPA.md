---
title: 'Spring Boot(数据访问):JDBC和JPA'
date: 2016-12-06 13:01:56
categories:
- Spring Boot
tags:
- 数据访问
- JDBC
- JPA
---

* **JDBC的自动配置**
spring-boot-starter-data-jpa依赖于spring-boot-starter-jdbc，而Spring Boot对JDBC做了一些自动配置，源码放置在org.springframework.boot.autoconfigure.jdbc下，通过"spring.datasource"为前缀的属性自动配置DataSouce；Spring Boot自动开启了注解事务的支持(@EnableTransactionManagement)；还配置了一个jdbcTemplate。
* **JPA的自动配置**
Spring Boot默认JPA的实现者是Hibernate，使用JPA只需要添加spring-boot-starter-data-jpa，然后再定义DataSource、实体类和数据访问层，无需任何额外配置。


# 添加maven依赖
pom.xml文件
```xml
<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.4.1.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>

        <!--Spring Boot提供的测试依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>

        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.0.24</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

# Spring Boot配置文件
* application.properties

```
# 激活profile, application-{profile}.properties
spring.profiles.active=prod

# server
server.port=80

# spring-data-jpq
# update: 启动时会根据实体类生成数据表,当实体类属性变动时,表结构也会更新
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true

# log
logging.file=log/demo.log
logging.level.org.springframework.web=debug
```

* application-dev.properties 使用默认的数据源

```
# 数据库配置
# Spring Boot默认的数据源是：org.apache.tomcat.jdbc.pool.DataSource
#spring.datasouce.type=
spring.datasource.url=jdbc:mysql://localhost:3306/spring_boot_test
spring.datasource.username=root
spring.datasource.password=root
```

* application-prod.properties 配置使用Druid数据源

```
# 数据库配置
# Spring Boot默认的数据源是：org.apache.tomcat.jdbc.pool.DataSource
spring.datasouce.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.url=jdbc:mysql://localhost:3306/spring_boot_test
spring.datasource.username=root
spring.datasource.password=root

# 连接池设置
# 初始化大小，最小，最大
spring.datasouce.initialSize=10
spring.datasouce.minIdle=5
spring.datasouce.maxActive=20
# 配置获取连接超时时间
spring.datasouce.maxWait=60000
spring.datasouce.validationQuery=SELECT 1
#配置监控统计拦截的filters
spring.datasouce.filters=stat,wall
```
# 编写实体类
```java
package com.roger.model;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

@Entity
public class Product {

    @Id
    @GeneratedValue
    private Long id;
    private String name;
    private String description;
    private Double price;

	...setter/gettter

    @Override
    public String toString() {
        return "Product{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", description='" + description + '\'' +
                ", price=" + price +
                '}';
    }
}
```

# 填充初始数据
在resources目录下增加一个data.sql,程序运行时会自动往表里插入数据。
>Spring JDBC has a DataSource initializer feature.Spring Boot enables it by default and loads SQL from the standard locations **schema.sql** and **data.sql** (in the root of the classpath)

schema.sql: 自动用来初始化表结构
data.sql: 自动用来填充数据

```sql
INSERT INTO product (id, name, description, price) VALUES (1, "电脑", "电脑系列产品", 10000)
```

# DAO层: JDBC/JPA
* ProductDao

```java
package com.roger.dao.jdbc;

import com.roger.model.Product;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.BeanPropertyRowMapper;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.RowMapper;
import org.springframework.stereotype.Repository;

/**
 * 使用JDBC实现DAO层
 */
@Repository
public class ProductDao {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    public Product findById(long id){
        String sql = "select * from Product where id = ?";
        RowMapper<Product> rowMapper = new BeanPropertyRowMapper<>(Product.class);
        return jdbcTemplate.queryForObject(sql, rowMapper, id);
    }
}
```

* ProductRepository

```java
package com.roger.dao.jpa;

import com.roger.model.Product;
import org.springframework.data.jpa.repository.JpaRepository;

/**
 * 使用JPA实现DAO层
 * Created by Roger on 2016/12/6.
 */
public interface ProductRepository extends JpaRepository<Product, Long> {
    // 排序
    List<Product> findByName(String name, Sort sort);

    // 分页
    Page<Product> findByName(String name, Pageable pageable);
}
```

# Service层
```java
package com.roger.service;

import com.roger.dao.jdbc.ProductDao;
import com.roger.dao.jpa.ProductRepository;
import com.roger.model.Product;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class ProductService {

    @Autowired
    private ProductDao productDao;

    @Autowired
    private ProductRepository productRepository;

    public Product findByIdJdbc(long id){
        return productDao.findById(id);
    }

    public Product findByIdJpa(long id){
        return productRepository.findOne(id);
    }

    /**
     * 排序
     * 根据价格降序
     * @param name
     * @return
     */
    public List<Product> findByName(String name){
        return productRepository.findByName(name, new Sort(Sort.Direction.DESC, "price"));
    }

    /**
     * 分页
     * @param name
     * @param pageNumber
     * @param pageSize
     * @return
     */
    public Page<Product> pageByName(String name, int pageNumber, int pageSize){
        return productRepository.findByName(name, new PageRequest(pageNumber, pageSize));
    }
}
```

# 配置监控统计功能
## 方式一
使用了原生的servlet, filter方式，然后通过**在Application.java上添加注解@ServletComponentScan**进行启动扫描包的方式进行处理的。
* 配置servlet

```java
package com.roger.servlet;

import com.alibaba.druid.support.http.StatViewServlet;

import javax.servlet.annotation.WebInitParam;
import javax.servlet.annotation.WebServlet;

@WebServlet(urlPatterns = "/druid/*",
        initParams = {
                // IP白名单, 没有配置则允许所有访问
                // @WebInitParam(name = "allow", value = "192.168.1.72,192.168.1.73"),
                // IP黑名单, 存在时deny优先于allow
                // @WebInitParam(name = "deny", value = "192.168.1.71"),
                @WebInitParam(name = "loginUsername", value = "admin"),
                @WebInitParam(name = "loginPassword", value = "admin")
        })
public class DruidStatViewServlet extends StatViewServlet {
}
```

* 配置filter

```java
package com.roger.filter;

import com.alibaba.druid.support.http.WebStatFilter;

import javax.servlet.annotation.WebFilter;
import javax.servlet.annotation.WebInitParam;

@WebFilter(filterName = "druidWebStatFilter",
        urlPatterns = "/*",
        initParams = {
                // 配置忽略资源
                @WebInitParam(name = "exclusions", value = "*.js, *.css, *.jpg")
        })
public class DruidStatFilter extends WebStatFilter {
}
```

* Application.java上添加@ServletComponentScan注解

```java
package com.roger;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.servlet.ServletComponentScan;

@SpringBootApplication
// @ServletComponentScan是的spring能够扫描到自己编写的servlet和filter
@ServletComponentScan(basePackages = {"com.roger.servlet","com.roger.filter"})
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

>访问：http://localhost/druid/index.html

## 方式二
使用代码注册servlet, filter。
```java
package com.roger.config;

import com.alibaba.druid.support.http.StatViewServlet;
import com.alibaba.druid.support.http.WebStatFilter;
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.boot.web.servlet.ServletRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * druid监控配置方式二
 */
@Configuration
public class DruidConfig {

    /**
     * 注册一个StatViewServlet
     * @return
     */
    @Bean
    public ServletRegistrationBean DruidStatViewServlet2(){
        ServletRegistrationBean servletRegistrationBean = new ServletRegistrationBean(new StatViewServlet(), "/druid2/*");

        // 添加初始化参数: initParams

        // IP白名单
        // servletRegistrationBean.addInitParameter("allow", "");
        // IP黑名单
        // servletRegistrationBean.addInitParameter("deny", "");
        // 登录查看信息的账号密码
        servletRegistrationBean.addInitParameter("loginUsername", "admin");
        servletRegistrationBean.addInitParameter("loginPassword", "admin");
        // 是否能够重置数据
        // servletRegistrationBean.addInitParameter("resetEnabled", "false");
        return servletRegistrationBean;
    }

    /**
     * 注册一个StatFilter
     * @return
     */
    @Bean
    public FilterRegistrationBean druidStatFilter2(){
        FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean(new WebStatFilter());
        // 添加过滤规则
        filterRegistrationBean.addUrlPatterns("/*");
        // 添加忽略的格式信息
        filterRegistrationBean.addInitParameter("exclusions", "*.js,*.css,*.jpg");
        return filterRegistrationBean;
    }
}
```

>访问：http://localhost/druid2/index.html