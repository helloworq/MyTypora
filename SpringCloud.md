# SpringCloud-openfeign实现微服务调用

​                                                                                             参考《Springcloud实战》

## 新建项目

新建一个多模块项目，如图为项目结构。

![](E:\DistCode\TyporaLoad\SpringCloud.assets\微信截图_20200802191248.png)

分别为服务消费者，服务提供者和服务注册中心。这和dubbo很像。

## 服务注册中心配置

### 启动类配置信息

```java
@EnableEurekaServer
@SpringBootApplication
public class FeignserverApplication {
    public static void main(String[] args) {
        SpringApplication.run(FeignserverApplication.class, args);
    }
}
```

### pom文件配置信息

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.2.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.myfeign</groupId>
    <artifactId>feignserver</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>feignserver</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Hoxton.SR6</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-loadbalancer</artifactId>
        </dependency>
        <!--服务注册中心-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
    </dependencies>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>

```

这里列出来的是核心的配置信息。

### application.properties配置信息

```properties
server.port=8081

eureka.instance.hostname=localhost
eureka.client.register-with-eureka=false
eureka.client.fetch-registry=false       #防止将自身注册进服务
eureka.client.service-url.defaultZone=http://localhost:8081/eureka     #注册中心地址，提供者和消费者都需要配置这个地址

```



## 消费者配置信息

### 启动类配置

```java
@EnableFeignClients(basePackages = "com.myfeign.feignconsumer.Controller")
@EnableDiscoveryClient      //允许将自己扫描成客户端
@SpringBootApplication
public class FeignconsumerApplication {
    public static void main(String[] args) {
        SpringApplication.run(FeignconsumerApplication.class, args);
    }
}
```

### pom文件配置信息

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.2.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.myfeign</groupId>
    <artifactId>feignconsumer</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>feignconsumer</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Hoxton.SR6</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!--feign组件-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        <!--负载中心完成服务消费任务-->
        <dependency>
                 <groupId>org.springframework.cloud</groupId>
                 <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
        </dependency>
        <!--eureka客户端承担服务发现功能-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>

```

### application.properties配置信息

```properties
server.port=5000

spring.application.name=openfeign                                     #服务名
eureka.client.service-url.defaultZone=http://localhost:8081/eureka    #注册中心地址
eureka.client.fetch-registry= true                                    #允许注册
```



## 提供者配置信息

### 启动类配置

```java
c
@SpringBootApplication
public class FeignproviderApplication {
    public static void main(String[] args) {
        SpringApplication.run(FeignproviderApplication.class, args);
    }
}
```

### pom文件配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.2.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.myfeign</groupId>
    <artifactId>feignprovider</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>feignprovider</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Hoxton.SR6</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
    
</project>

```

### application.properties配置

```properties
spring.application.name=feignService

server.port=8082
eureka.client.service-url.defaultZone=http://localhost:8081/eureka
```

提供者配置信息较为简单。

## 提供者服务类

```java
@RestController
public class OrderController {
    /**
     * GetMapping example with @RequestParam
     * @return userName
     */
    @GetMapping("/getUserName")
    public String getUserName(){
        return "dasdsa";
    }
}
```

## 消费者调用类

```java
@FeignClient(value = "feignService")    //指定在注册中心注册的服务名为自己提供服务
public interface DepartService { 
    
    @GetMapping("getUserName")          //必须和提供者的value一致
    public String getUserName();
}

@RestController
public class DepartController {

    @Autowired
    private DepartService departService;

    @GetMapping("getUserName")
    public void getUserName(){
        System.out.println(departService.getUserName());
    }
}
```

消费者调用类接口的两个@GetMapping映射器里的value必须和提供者提供的服务接口value一致，否则将无法调用。