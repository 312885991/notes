

## Swagger2的简单使用

### 一、导入Swagger2所需要的依赖

```xml
<!--添加Swagger依赖 -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.8.0</version>
</dependency>
<!--添加Swagger-UI依赖 -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.8.0</version>
</dependency>
```

- 版本尽量保持一致

### 二、配置Swagger2

- 新建一个 `Swagger2Config`  类

```java
package com.yonyou.config;

import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.ResourceHandlerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurationSupport;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger.web.UiConfiguration;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@Configuration
@EnableSwagger2
public class Swagger2Config extends WebMvcConfigurationSupport {

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/swagger-ui.html")
                .addResourceLocations("classpath:/META-INF/resources/");
        registry.addResourceHandler("webjars/**")
                .addResourceLocations("classpath:/META-INF/resources/webjars/");
    }
    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
            	//扫描的controller包
                .apis(RequestHandlerSelectors.basePackage("com.yonyou.controller"))
                .apis(RequestHandlerSelectors.withClassAnnotation(Api.class))
                .apis(RequestHandlerSelectors.withMethodAnnotation(ApiOperation.class))
                .paths(PathSelectors.any())
                .build();
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("测试API")
                .description("测试系统接口文档说明")
                .contact(new Contact("刘路生", "", "312885991@qq.com"))
                .version("1.0")
                .build();
    }

    @Bean
    UiConfiguration uiConfig() {
        return new UiConfiguration(null, "list", "alpha", "schema",
                UiConfiguration.Constants.DEFAULT_SUBMIT_METHODS, false, true, 60000L);
    }
}
```



### 三、编写测试类 `FileController`

```java
@Api(value = "文件操作API", tags = "文件上传与下载")
@RestController
@RequestMapping("/file")
public class FileController {

    //文件存储的位置
    private final String location = "D:/upload/";


    @ApiOperation(value = "文件下载", notes = "文件下载")
    @GetMapping("/download")
    public Result downLoadFile(@ApiParam(value = "填写需要下载的文件") String fileName, HttpServletResponse response){
        File file = new File(location, fileName);
        if(file.exists()){
            response.setContentType("application/force-download");
            response.addHeader("Content-Disposition", "attachment;fileName=" + fileName);
            FileInputStream input = null;
            try{
                input = new FileInputStream(file);
                ServletOutputStream out = response.getOutputStream();
                int real;
                byte[] bytes = new byte[1024];
                while((real=input.read(bytes))!=-1){
                    out.write(bytes, 0, real);
                }
                return Result.success();
            }catch (Exception e){
                e.printStackTrace();
                return Result.error("文件下载异常");
            }
        }
        return Result.error("请求下载的文件为空");
    }


    @ApiOperation(value = "文件上传", notes= "文件上传")
    @PostMapping("/upload")
    public Result upLoadFile(@ApiParam(value = "选择需要上传的文件",required = true) MultipartFile file){
        if (file.isEmpty()){
            return Result.error("请不要上传空文件");
        }
        String fileName = file.getOriginalFilename();
        long size = file.getSize();
        //控制台打印文件信息
        System.out.println(fileName+"-->"+size);
        //拼接成新的文件名
        int index = fileName.lastIndexOf(".");
        String suffix = fileName.substring(index);
        String name = Date.valueOf(LocalDate.now())+suffix;
        //指明文件上传位置
        File dest = new File(location, name);
        //判断文件父目录是否存在
       if(!dest.getParentFile().exists()){
           dest.getParentFile().mkdir();
       }
        try {
            //写入文件
            file.transferTo(dest);
            return Result.success();
        } catch (IOException e) {
            e.printStackTrace();
            return Result.error("文件上传失败");
        }
    }
}
```

### 四、访问 `http://localhost:8080/swagger-ui.html` 地址查看

- 结果：

![](p1.png)



### 五、swagger2常用注解

- @Api：用在请求的类上，说明该类的作用

  ```xml-dtd
  @Api：用在请求的类上，说明该类的作用
      tags="说明该类的作用"
      value="该参数没什么意义，所以不需要配置"
  ```

- @ApiOperation：用在请求的方法上，说明方法的作用

  ```xml-dtd
  @ApiOperation："用在请求的方法上，说明方法的作用"
      value="说明方法的作用"
      notes="方法的备注说明"
  ```

  示例：

  ```xml-dtd
  @ApiOperation(value="用户注册",notes="手机号、密码都是必输项，年龄随边填，但必须是数字")
  ```

- @ApiParam: 用在请求的参数上，对参数进行说明

  ```xml-dtd
  @ApiParam:"用在请求的参数上，对参数进行说明"
  	value:"对参数进行说明"
  	required:"参数是否必传"
  ```

  示例：

  ```xml-dtd
  public Result upLoadFile(@ApiParam(value = "选择需要上传的文件",required = true) MultipartFile file)
  ```


- @ApiImplicitParams：用在请求的方法上，包含一组参数说明

  ```xml-dtd
  @ApiImplicitParams：用在请求的方法上，包含一组参数说明
      @ApiImplicitParam：用在 @ApiImplicitParams 注解中，指定一个请求参数的配置信息       
          name：参数名
          value：参数的汉字说明、解释
          required：参数是否必须传
          paramType：参数放在哪个地方
              · header --> 请求参数的获取：@RequestHeader
              · query --> 请求参数的获取：@RequestParam
              · path（用于restful接口）--> 请求参数的获取：@PathVariable
              · body（不常用）
              · form（不常用）    
          dataType：参数类型，默认String，其它值dataType="Integer"       
          defaultValue：参数的默认值
  ```

  示例：

  ```xml-dtd
  @ApiImplicitParams({
      @ApiImplicitParam(name="mobile",value="手机号",required=true,paramType="form"),
      @ApiImplicitParam(name="password",value="密码",required=true,paramType="form"),
      @ApiImplicitParam(name="age",value="年龄",required=true,paramType="form",dataType="Integer")
  })
  ```

- @ApiResponses：用于请求的方法上，表示一组响应

  ```xml-dtd
  @ApiResponses：用于请求的方法上，表示一组响应
      @ApiResponse：用在@ApiResponses中，一般用于表达一个错误的响应信息
          code：数字，例如400
          message：信息，例如"请求参数没填好"
          response：抛出异常的类
  ```

  示例：

  ```xml-dtd
  @ApiOperation(value = "select1请求",notes = "多个参数，多种的查询参数类型")
  @ApiResponses({
      @ApiResponse(code=400,message="请求参数没填好"),
      @ApiResponse(code=404,message="请求路径没有或页面跳转路径不对")
  })
  ```

- @ApiModel：用于响应类上，表示一个返回响应数据的信息

  ```xml-dtd
  @ApiModel：用于响应类上，表示一个返回响应数据的信息
              （这种一般用在post创建的时候，使用@RequestBody这样的场景，
              请求参数无法使用@ApiImplicitParam注解进行描述的时候）
      @ApiModelProperty：用在属性上，描述响应类的属性
  ```

  示例：

  ```xml-dtd
  import java.io.Serializable;
  
  @ApiModel(description= "返回响应数据")
  public class RestMessage implements Serializable{
  
      @ApiModelProperty(value = "是否成功")
      private boolean success=true;
      @ApiModelProperty(value = "返回对象")
      private Object data;
      @ApiModelProperty(value = "错误编号")
      private Integer errCode;
      @ApiModelProperty(value = "错误信息")
      private String message;
  
      /* getter/setter */
  }
  ```
