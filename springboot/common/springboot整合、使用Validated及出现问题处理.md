# Springboot 整合、使用Validated和不生效问题处理

## 一、名词和技术介绍
### 1.1  JSR规范提案
```markdown
JSR：Java Specification Requests的缩写，意思是Java 规范提案。
    是指向JCP(Java Community Process)提出新增一个标准化技术规范的正式请求。
任何人都可以提交JSR，以向Java平台增添新的API和服务，JSR已成为Java界的一个重要标准。
本文介绍的Bean Validation 就是出自JSR303，JSR349，以及JSR380 规范提案。
该规范从JSR 303 发展到 JSR 380，目前最新规范是Bean Validation 2.0。
需要注意的是，规范提案只是提供了规范，并没有提供具体的实现。
具体实现框架有默认的javax.validation.api，以及hibernate-validator。
目前绝大多使用hibernate-validator。
```
### 1.2 javax.validation.api
```markdown
Java 在2009年的 JAVAEE 6 中发布了 JSR303以及javax下的validation包内容。

这项工作的主要目标是为java应用程序开发人员提供基于java对象的约束（constraints）声明和对约束的验证工具（validator），
以及约束元数据存储库和查询API，以及默认实现。

Java8开始，Java EE改名为Jakarta EE，注意javax.validation相关的包移动到了jakarta.validation的包下。
所以大家看不同的版本的时候，会发现以前的版本包在javax.validation包下，java 8之后在jakarta.validation。
```

### 1.3 hibernate-validator
```markdown
Hibernate-Validator框架是另外一个针对Bean Validation 规范的实现，
它提供了 JSR 380 规范中所有内置 constraint 的实现，除此之外还有一些附加的 constraint。
（注意：此处的Hibernate 不是 Hibernate ORM没有任何关系，hibernate-validator是Hibernate 基金会下的项目之一）
hibernate-validator的介绍地址：https://docs.jboss.org/hibernate/stable/validator/reference/en-US/html_single/
```

## 二、Bean Validation 定义的约束注解

```markdown
下面列出常用约束，每个约束都有参数 message，groups 和 payload。这是 Bean Validation 规范的要求。
请注意，这里没有区分哪些是默认注解，哪些是hibernate-validator的注解，因为我们实际开发中，会同时用两个包。
```
### 2.1 2个标识注解

1. **@Valid**（规范、常用）

   ```markdown
   标记用于验证级联的属性、方法参数或方法返回类型。
   在验证属性、方法参数或方法返回类型时，将验证在对象及其属性上定义的约束。
   ```

   

2. **@Validated**（spring扩展提供）

``` markdown
spring 提供的扩展注解，可以添加在类、方法参数、普通方法上，表示它们需要进行约束校验，可以方便的用于分组校验。
其中，message 是提示消息，groups 可以根据情况来分组。当然，也可以不使用注解，而直接使用Validator.validate()来进行分组验证。
```

### 2.2 约束注解

- **@AssertFalse**
  检查元素是否为 false，支持数据类型：boolean
- **@AssertTrue**
  检查元素是否为 true，支持数据类型：boolean
- **@DecimalMax(value=, inclusive=)**
  inclusive：boolean，默认 true，表示是否包含，是否等于
  value：当 inclusive=false 时，检查带注解的值是否小于指定的最大值。当 inclusive=true 检查该值是否小于或等于指定的最大值。参数值是根据 bigdecimal 字符串表示的最大值。
  支持数据类型：BigDecimal、BigInteger、CharSequence、（byte、short、int、long 和其封装类）
- **@DecimalMin(value=, inclusive=)**
  支持数据类型：BigDecimal、BigInteger、CharSequence、（byte、short、int、long 和其封装类）
  inclusive：boolean，默认 true，表示是否包含，是否等于
  value：
  当 inclusive=false 时，检查带注解的值是否大于指定的最大值。当 inclusive=true 检查该值是否大于或等于指定的最大值。参数值是根据 bigdecimal 字符串表示的最小值。
- **@Digits(integer=, fraction=)**
  检查值是否为最多包含 integer 位整数和 fraction 位小数的数字
  支持的数据类型：
  BigDecimal, BigInteger, CharSequence, byte, short, int, long 、原生类型的封装类、任何 Number 子类。
- **@Email**
  检查指定的字符序列是否为有效的电子邮件地址。可选参数 regexp 和 flags 允许指定电子邮件必须匹配的附加正则表达式（包括正则表达式标志）。
  支持的数据类型：CharSequence
- **@Max(value=)**
  检查值是否小于或等于指定的最大值
  支持的数据类型：
  BigDecimal, BigInteger, byte, short, int, long, 原生类型的封装类, CharSequence 的任意子类（字符序列表示的数字）, Number 的任意子类, javax.money.MonetaryAmount 的任意子类
- **@Min(value=)**
  检查值是否大于或等于指定的最大值
  支持的数据类型：
  BigDecimal, BigInteger, byte, short, int, long, 原生类型的封装类, CharSequence 的任意子类（字符序列表示的数字）, Number 的任意子类, javax.money.MonetaryAmount 的任意子类
- **@NotBlank**
  检查字符序列是否为空，以及去空格后的长度是否大于 0。与 @NotEmpty 的不同之处在于，此约束只能应用于字符序列。
  支持数据类型：CharSequence
- **@NotNull**
  检查值是否不为 null
  支持数据类型：任何类型
- @NotEmpty
  检查元素是否为 null 或 空
  支持数据类型：CharSequence, Collection, Map, arrays

- @Size(min=, max=)
  检查元素个数是否在 min（含）和 max（含）之间
  支持数据类型：CharSequence，Collection，Map, arrays

- @Negative
  检查元素是否严格为负数。零值被认为无效。
  支持数据类型：
  BigDecimal, BigInteger, byte, short, int, long, 原生类型的封装类, CharSequence 的任意子类（字符序列表示的数字）, Number 的任意子类, javax.money.MonetaryAmount 的任意子类

- **@NegativeOrZero**
  检查元素是否为负或零。
  支持数据类型：
  BigDecimal, BigInteger, byte, short, int, long, 原生类型的封装类, CharSequence 的任意子类（字符序列表示的数字）, Number 的任意子类, javax.money.MonetaryAmount 的任意子类
- **@Positive**
  检查元素是否严格为正。零值被视为无效。
  支持数据类型：
  BigDecimal, BigInteger, byte, short, int, long, 原生类型的封装类, CharSequence 的任意子类（字符序列表示的数字）, Number 的任意子类, javax.money.MonetaryAmount 的任意子类
- **@PositiveOrZero**
  检查元素是否为正或零。
  支持数据类型：
  BigDecimal, BigInteger, byte, short, int, long, 原生类型的封装类, CharSequence 的任意子类（字符序列表示的数字）, Number 的任意子类, javax.money.MonetaryAmount 的任意子类
- **@Null**
  检查值是否为 null
  支持数据类型：任何类型
- **@Future**
  检查日期是否在未来
  支持的数据类型：
  java.util.Date, java.util.Calendar, java.time.Instant, java.time.LocalDate, java.time.LocalDateTime, java.time.LocalTime, java.time.MonthDay, java.time.OffsetDateTime, java.time.OffsetTime, java.time.Year, java.time.YearMonth, java.time.ZonedDateTime, java.time.chrono.HijrahDate, java.time.chrono.JapaneseDate, java.time.chrono.MinguoDate, java.time.chrono.ThaiBuddhistDate
  如果 Joda Time API 在类路径中，ReadablePartial 和ReadableInstant 的任何实现类
- **@FutureOrPresent**
  检查日期是现在或将来
  支持数据类型：同@Future
- **@Past**
  检查日期是否在过去
  支持数据类型：同@Future
- **@PastOrPresent**
  检查日期是否在过去或现在
  支持数据类型：同@Future
- **@Pattern(regex=, flags=)**
  根据给定的 flag 匹配，检查字符串是否与正则表达式 regex 匹配
  支持数据类型：CharSequence

-----

以下是hibernate-Validator 附加注解:

- **@Range(min=, max=) **
  被注释的元素必须在合适的范围内。
- **@Length(min=, max=)：**
  被注释的字符串的大小必须在指定的范围内。
- **@URL(protocol=,host=,port=,regexp=,flags=) **
  被注释的字符串必须是一个有效的 URL 。
- **@SafeHtml **
  判断提交的 HTML 是否安全。例如说，不能包含 javascript 脚本等等。


## 三 、应用案例

### 3.1  jar包引入

注：因为**springboot2.3.0**以上版本默认去除了**validation**的依赖，所以要添加**validation**的start

```xml
    <!-- 参数校验框架 -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>
```



### 3.2 Object设定

```java

@ApiModel
public class UserVO implements Serializable {


  private static final long serialVersionUID = 2016247254418250644L;

  /**
   * 用户名
   */
  @ApiModelProperty(value = "用户名", name = "username", example = "xiIj")
  @NotBlank(message = "用户名不能为空!",groups = {RegisterGroup.class})
  private String username;

  /**
   * 密码
   */
  @ApiModelProperty(value = "密码", name = "password", example = "123456")
  @Length(min=6,message = "密码长度不能少于6位！")
  @NotBlank(message = "密码不能为空!",groups = {RegisterGroup.class})
  private String password;
  @ApiModelProperty(value = "确认密码", name = "confirmPassword", example = "123456")
  @NotBlank(message = "请确认密码!",groups = {RegisterGroup.class})
  private String confirmPassword;


  public String getUsername() {
    return username;
  }

  public void setUsername(String username) {
    this.username = username;
  }

  public String getPassword() {
    return password;
  }

  public void setPassword(String password) {
    this.password = password;
  }

  @Override
  public String toString() {
    return new StringJoiner(", ", UserVO.class.getSimpleName() + "[", "]")
        .add("username='" + username + "'")
        .add("password='" + password + "'")
        .add("confirmPassword='" + confirmPassword + "'")
        .toString();
  }

  public String getConfirmPassword() {
    return confirmPassword;
  }

  public void setConfirmPassword(String confirmPassword) {
    this.confirmPassword = confirmPassword;
  }
}
```

### 3.3 controller/调用层设置

```java
// RegisterGroup.class 是定义的自定义校验接口类型，用于多个接口使用同一entity校验时，区分各自的限定条件
  @PostMapping("/register")
  public JSONResult add(
      @RequestBody @Validated(RegisterGroup.class) UserVO userVO, HttpServletRequest request,
      HttpServletResponse response)
      throws Exception {
      
		// ......
  }
}
```

### 3.4 自定义异常信息

```
使用全局异常处理来处理校验逻辑的思路很简单，首先我们需要通过@ControllerAdvice注解定义一个全局异常的处理类，然后自定义一个校验异常，当我们在Controller中校验失败时，直接抛出该异常，这样就能达到校验失败返回错误信息的目的。
```



```java
package com.example.exception;

import com.example.utils.JSONResult;
import java.util.List;
import org.apache.commons.lang3.exception.ExceptionUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.validation.BindException;
import org.springframework.validation.BindingResult;
import org.springframework.validation.FieldError;
import org.springframework.validation.ObjectError;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;

/**
 * 全局异常处理
 */
@ControllerAdvice
public class GlobalExceptionHandler {

  protected Logger logger = LoggerFactory.getLogger(getClass());

  /**
   * 数据校验异常
   */
  @ExceptionHandler({BindException.class})
  @ResponseBody
  public JSONResult objectParamExceptionHandle(BindException e) {
    logger.error(e.getMessage(), e);
    BindingResult result = e.getBindingResult();
    if (result.hasErrors()) {
      List<ObjectError> errors = result.getAllErrors();
      ObjectError objectError = errors.stream().findFirst().get();
      return JSONResult.errorMsg(objectError.getDefaultMessage());
    }
    return JSONResult.errorMsg(this.getErrorMsg(e));
  }


  @ExceptionHandler({MethodArgumentNotValidException.class})
  @ResponseBody
  public JSONResult paramExceptionHandle(MethodArgumentNotValidException e) {
    BindingResult result = e.getBindingResult();
    FieldError error = result.getFieldError();
    return JSONResult.errorMsg(error.getDefaultMessage());
  }

  /**
   * 全局异常
   */
  @ExceptionHandler(Exception.class)
  @ResponseBody
  public JSONResult defaultExceptionHandler(Exception e) {
    logger.error(e.getMessage(), e);
    return JSONResult.errorMsg(ExceptionUtils.getMessage(e));
  }

  private String getErrorMsg(Exception e) {
    return this.getErrorMsg(e, 0, 20);
  }

  private String getErrorMsg(Exception e, int currentCount, int maxCount) {
    try {
      if (null != e) {
        if (null != e.getMessage()) {
          return e.getMessage();
        }

        if (currentCount < maxCount) {
          return this.getErrorMsg(e, currentCount + 1, maxCount);
        }
      }
    } catch (Exception var5) {
      logger.error(var5.getMessage(), var5);
      return "失败,请稍候再试！";
    }

    logger.error("错误：", e);
    return "失败,请稍候再试！";
  }
}

```

