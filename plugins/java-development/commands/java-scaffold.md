# Java 项目脚手架

使用 Maven、Spring Boot 3.x、MyBatis 和 MySQL 创建 Java 项目。

## 参数

$ARGUMENTS

## 项目结构

```
project-name/
├── pom.xml
├── README.md
├── .gitignore
├── src/
│   ├── main/
│   │   ├── java/com/example/project/
│   │   │   ├── ProjectApplication.java
│   │   │   ├── config/
│   │   │   │   └── MyBatisConfig.java
│   │   │   ├── controller/
│   │   │   │   └── UserController.java
│   │   │   ├── service/
│   │   │   │   ├── UserService.java
│   │   │   │   └── impl/
│   │   │   │       └── UserServiceImpl.java
│   │   │   ├── mapper/
│   │   │   │   └── UserMapper.java
│   │   │   ├── model/
│   │   │   │   └── User.java
│   │   │   └── dto/
│   │   │       ├── UserRequest.java
│   │   │       └── UserResponse.java
│   │   └── resources/
│   │       ├── application.yml
│   │       ├── mapper/
│   │       │   └── UserMapper.xml
│   │       └── db/migration/
│   │           └── V1__Create_user_table.sql
│   └── test/
│       └── java/com/example/project/
│           ├── ProjectApplicationTests.java
│           └── controller/
│               └── UserControllerTest.java
└── docker-compose.yml
```

## Maven 配置

**pom.xml**:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version>
        <relativePath/>
    </parent>

    <groupId>com.example</groupId>
    <artifactId>project-name</artifactId>
    <version>1.0.0</version>

    <properties>
        <java.version>21</java.version>
        <mybatis.version>3.0.3</mybatis.version>
    </properties>

    <dependencies>
        <!-- Spring Boot Starters -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- MyBatis -->
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>${mybatis.version}</version>
        </dependency>

        <!-- MySQL -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>

        <!-- Lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

        <!-- Validation -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>

        <!-- Test -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <!-- MyBatis Test -->
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter-test</artifactId>
            <version>${mybatis.version}</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

## 核心代码

**ProjectApplication.java**:
```java
package com.example.project;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ProjectApplication {
    public static void main(String[] args) {
        SpringApplication.run(ProjectApplication.class, args);
    }
}
```

**User.java**:
```java
package com.example.project.model;

import lombok.Data;
import java.time.LocalDateTime;

@Data
public class User {
    private Long id;
    private String username;
    private String email;
    private LocalDateTime createdAt;
}
```

**UserMapper.java**:
```java
package com.example.project.mapper;

import com.example.project.model.User;
import org.apache.ibatis.annotations.*;
import java.util.List;

@Mapper
public interface UserMapper {

    @Select("SELECT * FROM users")
    List<User> findAll();

    @Select("SELECT * FROM users WHERE id = #{id}")
    User findById(Long id);

    @Insert("INSERT INTO users(username, email, created_at) VALUES(#{username}, #{email}, #{createdAt})")
    @Options(useGeneratedKeys = true, keyProperty = "id")
    int insert(User user);

    @Update("UPDATE users SET username=#{username}, email=#{email} WHERE id=#{id}")
    int update(User user);

    @Delete("DELETE FROM users WHERE id = #{id}")
    int deleteById(Long id);
}
```

**UserController.java**:
```java
package com.example.project.controller;

import com.example.project.dto.UserRequest;
import com.example.project.dto.UserResponse;
import com.example.project.service.UserService;
import lombok.RequiredArgsConstructor;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import jakarta.validation.Valid;
import java.util.List;

@RestController
@RequestMapping("/api/users")
@RequiredArgsConstructor
public class UserController {

    private final UserService userService;

    @GetMapping
    public List<UserResponse> getAll() {
        return userService.getAllUsers();
    }

    @GetMapping("/{id}")
    public UserResponse getById(@PathVariable Long id) {
        return userService.getUserById(id);
    }

    @PostMapping
    public UserResponse create(@Valid @RequestBody UserRequest request) {
        return userService.createUser(request);
    }

    @PutMapping("/{id}")
    public UserResponse update(@PathVariable Long id, @Valid @RequestBody UserRequest request) {
        return userService.updateUser(id, request);
    }

    @DeleteMapping("/{id}")
    public void delete(@PathVariable Long id) {
        userService.deleteUser(id);
    }
}
```

## 配置文件

**application.yml**:
```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/project_db?useSSL=false&serverTimezone=UTC
    username: root
    password: password
    driver-class-name: com.mysql.cj.jdbc.Driver

mybatis:
  mapper-locations: classpath:mapper/*.xml
  type-aliases-package: com.example.project.model
  configuration:
    map-underscore-to-camel-case: true

server:
  port: 8080
```

**UserMapper.xml**:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.project.mapper.UserMapper">

    <resultMap id="UserResultMap" type="com.example.project.model.User">
        <id property="id" column="id"/>
        <result property="username" column="username"/>
        <result property="email" column="email"/>
        <result property="createdAt" column="created_at"/>
    </resultMap>

    <select id="findAll" resultMap="UserResultMap">
        SELECT id, username, email, created_at FROM users
    </select>

</mapper>
```

## Docker Compose

**docker-compose.yml**:
```yaml
version: '3.8'
services:
  mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: password
      MYSQL_DATABASE: project_db
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql

  app:
    build: .
    ports:
      - "8080:8080"
    depends_on:
      - mysql
    environment:
      SPRING_DATASOURCE_URL: jdbc:mysql://mysql:3306/project_db?useSSL=false&serverTimezone=UTC

volumes:
  mysql_data:
```

## 创建命令

```bash
# 创建 Maven 项目
mvn archetype:generate -DgroupId=com.example -DartifactId=project-name -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false

# 运行项目
mvn spring-boot:run

# 构建项目
mvn clean package
```

## 测试示例

```java
@SpringBootTest
class UserMapperTest {

    @Autowired
    private UserMapper userMapper;

    @Test
    void shouldInsertUser() {
        User user = new User();
        user.setUsername("test");
        user.setEmail("test@example.com");

        int result = userMapper.insert(user);
        assertThat(result).isEqualTo(1);
        assertThat(user.getId()).isNotNull();
    }
}
```