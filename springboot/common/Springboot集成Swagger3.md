# Springboot 集成 Swagger 3.0

`前后端分离的项目，接口文档的存在十分重要。与手动编写接口文档不同，swagger是一个自动生成接口文档的工具，在需求不断变更的环境下，手动编写文档的效率实在太低。与swagger2相比新版的swagger3配置更少，使用更加方便。`

## 一 、引入swagger3.0 

### 1.1 引入依赖

`引入swagger3的maven坐标，下面是swagger3的maven地址。`

https://mvnrepository.com/artifact/io.springfox/springfox-boot-starter

```xml
<!-- 引入Swagger3依赖 -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-boot-starter</artifactId>
    <version>3.0.0</version>
</dependency>
```

### 1.2 自定义配置文件

``` java
package com.example.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.oas.annotations.EnableOpenApi;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;

@Configuration
@EnableOpenApi
public class Swagger3 {

  //    http://localhost:8088/swagger-ui.html     原路径
  //    http://localhost:8088/doc.html     原路径

  /**
   * 配置swagger3核心配置 docket
   */
  @Bean
  public Docket createRestApi() {
    // 指定api类型为swagger2
    return new Docket(DocumentationType.OAS_30)
        // 用于定义api文档汇总信息
        .apiInfo(apiInfo())
        .enable(true)
        .select()
        // 指定controller包
        .apis(RequestHandlerSelectors.basePackage("com.example.controller"))
        // 所有controller
        .paths(PathSelectors.any())
        .build();
  }

  private ApiInfo apiInfo() {
    return new ApiInfoBuilder()
        // 文档页标题
        .title("foodie接口api")
        // 联系人信息
        .contact(new Contact("高建国", "https://www.baidu.com", "304590126@qq.com"))
        // 详细信息
        .description("接口文档说明信息")
        // 文档版本号
        .version("1.0.1")
        // 网站地址
        .termsOfServiceUrl("https://www.xxxxx.com")
        .build();
  }
}

```

### 1.3 编辑controller接口文档信息

```java
@Api(tags="用户管理")
@RestController
@RequestMapping("/passport")
public class PassportController {

  @Autowired
  private UsersService usersService;

  @GetMapping("/usernameIsExist")
  @ApiOperation(value = "用户是否存在", notes = "用户是否存在", httpMethod = "GET")
  public JSONResult usernameIsExist(@RequestParam String username) {
    if (StringUtils.isEmpty(username)) {
      return JSONResult.errorMsg("用户名不能为空！");
    }
    Boolean re = usersService.usernameIsExist(username);
    return re ? JSONResult.errorMsg("用户名已存在！") : JSONResult.ok();
  }
  }
```

附：Swagger2和Swagger3注解对比

| swagger2             | swagger3                                                     | 注解位置                           |
| -------------------- | ------------------------------------------------------------ | ---------------------------------- |
| `@Api`               | `@Tag`(name = “接口类描述”)                                  | Controller 类上                    |
| `@ApiOperation`      | `@Operation`(summary =“接口方法描述”)                        | Controller 方法上                  |
| `@ApiImplicitParams` | `@Parameters`                                                | Controller 方法上                  |
| `@ApiImplicitParam`  | `@Parameter`(description=“参数描述”)                         | Controller 方法上 `@Parameters` 里 |
| `@ApiParam`          | `@Parameter`(description=“参数描述”)                         | Controller 方法的参数上            |
| `@ApiIgnore`         | `@Parameter`(hidden = true)或 `@Operation`(hidden = true)或 `@Hidden` | -                                  |
| `@ApiModel`          | `@Schema`                                                    | DTO类上                            |
| `@ApiModelProperty`  | `@Schema`                                                    | DTO属性上                          |

## 二、引入第三方UI


此处可集成也可使用原生的ui，就是一个皮肤而已。

### 2.1 引入依赖

如果是swagger2版本就用对应的2版本，引用错的话会出问题。

```xml
    <!-- https://mvnrepository.com/artifact/com.github.xiaoymin/knife4j-spring-boot-starter -->
    <properties>
    <swagger.knife4j.version>3.0.2</swagger.knife4j.version>
    </properties>
    <dependency>
      <groupId>com.github.xiaoymin</groupId>
      <artifactId>knife4j-spring-boot-starter</artifactId>
      <version>${swagger.knife4j.version}</version>
    </dependency>
```

