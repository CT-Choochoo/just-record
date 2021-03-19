# SpringBoot 整合 Log4j

## 一、项目环境

1. **springboot 2.4.3**

2. **mybatis-plus-boot-starter 3.4.2** 

## 二、pom依赖处理

### 2.1 去除原生的spring-boot-starter-logging

```xml
 <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter</artifactId>
      <exclusions>
        <exclusion>
          <artifactId>spring-boot-starter-logging</artifactId>
          <groupId>org.springframework.boot</groupId>
        </exclusion>
      </exclusions>
    </dependency>
```

### 2.2 引入slf4j-api

```xml
<!--引入日志依赖 抽象层 与 实现层-->
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-api</artifactId>
      <version>1.7.30</version>
    </dependency>
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-log4j12</artifactId>
      <version>1.7.30</version>
    </dependency>
```

## 三 、日志配置文件

配置日志文件

### 3.1在resources目录下创建Log4j.properties的日志配置文件

``` properties
# 设置日志级别喝输出方式
log4j.rootLogger=DEBUG,stdout,file
log4j.additivity.org.apache=true
# 控制台输出配置
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.threshold=INFO
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss} %-5p %c{1}:%L - %m%n
# 文件输出配置
log4j.appender.file=org.apache.log4j.DailyRollingFileAppender
log4j.appender.file.layout=org.apache.log4j.PatternLayout
log4j.appender.file.DatePattern='.'yyyy-MM-dd-HH-mm
log4j.appender.file.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss} %-5p %c{1}:%L - %m%n
log4j.appender.file.Threshold=INFO
log4j.appender.file.append=true
log4j.appender.file.File=/workspaces/logs/foodie-api/foodie.log

```

### 3.2 mybatis 、mybatisPlus的sql打印配置

log-impl: 日志的实现方式选择**apach.ibatis**的**StdOutImpl**

```yaml
mybatis-plus:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```

