
$\color{red}{更多精彩文章请关注本人公众号}$
![yunqing.jpg](https://upload-images.jianshu.io/upload_images/8020666-7e97ce57ac7a2143.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#一、为什么使用Swagger2
当下很多公司都采取前后端分离的开发模式，前端和后端的工作由不同的工程师完成。在这种开发模式下，维持一份及时更新且完整的 Rest API 文档将会极大的提高我们的工作效率。传统意义上的文档都是后端开发人员手动编写的，相信大家也都知道这种方式很难保证文档的及时性，这种文档久而久之也就会失去其参考意义，反而还会加大我们的沟通成本。而 Swagger 给我们提供了一个全新的维护 API 文档的方式，下面我们就来了解一下它的优点：

1、代码变，文档变。只需要少量的注解，Swagger 就可以根据代码自动生成 API 文档，很好的保证了文档的时效性。

2、跨语言性，支持 40 多种语言。

3、Swagger UI 呈现出来的是一份可交互式的 API 文档，我们可以直接在文档页面尝试 API 的调用，省去了准备复杂的调用参数的过程。

4、还可以将文档规范导入相关的工具（例如 Postman、SoapUI）, 这些工具将会为我们自动地创建自动化测试。

#二、配置使用Swagger2

##1、在pom.xml加入swagger2相关依赖
```
<!--swagger2的依赖-->
<dependency>
   <groupId>io.springfox</groupId>
   <artifactId>springfox-swagger2</artifactId>
   <version>2.9.2</version>
</dependency>
<dependency>
   <groupId>io.springfox</groupId>
   <artifactId>springfox-swagger-ui</artifactId>
   <version>2.9.2</version>
</dependency>
```


##2、新建Swagger2Config配置类


```
import io.swagger.annotations.Api;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.ResourceHandlerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurationSupport;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.*;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spi.service.contexts.SecurityContext;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

import java.util.ArrayList;
import java.util.List;

/**
 * @author yunqing
 * @date 2019/12/17 10:23
 */
@Configuration
@EnableSwagger2
public class Swagger2Config extends WebMvcConfigurationSupport {
    @Bean
    public Docket createRestApi(){
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                //为当前包下controller生成API文档
//                .apis(RequestHandlerSelectors.basePackage("com.troila"))
                //为有@Api注解的Controller生成API文档
//                .apis(RequestHandlerSelectors.withClassAnnotation(Api.class))
                //为有@ApiOperation注解的方法生成API文档
//                .apis(RequestHandlerSelectors.withMethodAnnotation(ApiOperation.class))
                //为任何接口生成API文档
                .apis(RequestHandlerSelectors.any())
                .paths(PathSelectors.any())
                .build();
                //添加登录认证
                /*.securitySchemes(securitySchemes())
                .securityContexts(securityContexts());*/
    }

    private ApiInfo apiInfo() {
        Contact contact = new Contact("yunqing", "", "yunqing****@gmail.com");
        return new ApiInfoBuilder()
                .title("SwaggerUI演示")
                .description("接口文档，描述词省略200字")
                .contact(contact)
                .version("1.0")
                .build();
    }

    /**
     * 配置swagger2的静态资源路径
     * @param registry
     */
    @Override
    protected void addResourceHandlers(ResourceHandlerRegistry registry) {
        // 解决静态资源无法访问
        registry.addResourceHandler("/**")
                .addResourceLocations("classpath:/static/");
        // 解决swagger无法访问
        registry.addResourceHandler("/swagger-ui.html")
                .addResourceLocations("classpath:/META-INF/resources/");
        // 解决swagger的js文件无法访问
        registry.addResourceHandler("/webjars/**")
                .addResourceLocations("classpath:/META-INF/resources/webjars/");
    }


    /**
     * 给API文档接口添加安全认证
     */
    /*private List<ApiKey> securitySchemes() {
        List<ApiKey> apiKeys = new ArrayList<>();
        apiKeys.add(new ApiKey("Authorization", "Authorization", "header"));
        return apiKeys;
    }

    private List<SecurityContext> securityContexts() {
        List<SecurityContext> securityContexts = new ArrayList<>();
        securityContexts.add(SecurityContext.builder()
                .securityReferences(defaultAuth())
                .forPaths(PathSelectors.regex("^(?!auth).*$")).build());
        return securityContexts;
    }

    private List<SecurityReference> defaultAuth() {
        AuthorizationScope authorizationScope = new AuthorizationScope("global", "accessEverything");
        AuthorizationScope[] authorizationScopes = new AuthorizationScope[1];
        authorizationScopes[0] = authorizationScope;
        List<SecurityReference> securityReferences = new ArrayList<>();
        securityReferences.add(new SecurityReference("Authorization", authorizationScopes));
        return securityReferences;
    }*/
}
```

##3、在shiro配置类中放行swagger2相关资源
```
//swagger2免拦截
filterChainDefinitionMap.put("/swagger-ui.html**", "anon");
filterChainDefinitionMap.put("/v2/api-docs", "anon");
filterChainDefinitionMap.put("/swagger-resources/**", "anon");
filterChainDefinitionMap.put("/webjars/**", "anon");
```


##4、配置为哪部分接口生成API文档
>主要是在Swagger2Config配置类中进行createRestApi()方法中进行配置，下面提供四种配置：

①为任何接口生成API文档，这种方式不必在接口方法上加任何注解，方便的同时也会因为没有添加任何注解所以生成的API文档也没有注释，可读性不高。
```
@Bean
    public Docket createRestApi(){
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                //为任何接口生成API文档
                .apis(RequestHandlerSelectors.any())
                .paths(PathSelectors.any())
                .build();
    }
```

②为当前配置的包下controller生成API文档
```
.apis(RequestHandlerSelectors.basePackage("com.troila"))

```
③为有@Api注解的Controller生成API文档
```
.apis(RequestHandlerSelectors.withClassAnnotation(Api.class))
```
④为有@ApiOperation注解的方法生成API文档
```
.apis(RequestHandlerSelectors.withMethodAnnotation(ApiOperation.class))
```
#三、Swagger2注解详解
##1、@Api ：请求类的说明
```
@Api：放在请求的类上，与 @Controller 并列，说明类的作用，如用户模块，订单类等。
	tags="说明该类的作用"
	value="该参数没什么意义，所以不需要配置"
```
###举例：
```
@Api(tags = "账户相关模块")
@RestController
@RequestMapping("/api/account")
public class AccountController {
	//TODO
}
```
##2、@ApiOperation：方法的说明
```
@ApiOperation："用在请求的方法上，说明方法的作用"
	value="说明方法的作用"
	notes="方法的备注说明"
```
###举例：
```
@ApiOperation(value = "修改密码", notes = "方法的备注说明，如果有可以写在这里")
@PostMapping("/changepass")
public AjaxResult changePassword(@AutosetParam SessionInfo sessionInfo,
        @RequestBody @Valid PasswordModel passwordModel) {
	//TODO
}
```
##3、@ApiImplicitParams、@ApiImplicitParam：方法参数的说明
```
@ApiImplicitParams：用在请求的方法上，包含一组参数说明
	@ApiImplicitParam：对单个参数的说明	    
	    name：参数名
	    value：参数的汉字说明、解释
	    required：参数是否必须传
	    paramType：参数放在哪个地方
	        · header --> 请求参数的获取：@RequestHeader
	        · query --> 请求参数的获取：@RequestParam
	        · path（用于restful接口）--> 请求参数的获取：@PathVariable
	        · body（请求体）-->  @RequestBody User user
	        · form（普通表单提交）	   
	    dataType：参数类型，默认String，其它值dataType="int"	   
	    defaultValue：参数的默认值
```
###举例：
```
@ApiOperation(value="用户登录",notes="随边说点啥")
@ApiImplicitParams({
        @ApiImplicitParam(name="mobile",value="手机号",required=true,paramType="form"),
        @ApiImplicitParam(name="password",value="密码",required=true,paramType="form"),
        @ApiImplicitParam(name="age",value="年龄",required=true,paramType="form",dataType="Integer")
})
@PostMapping("/login")
public AjaxResult login(@RequestParam String mobile, @RequestParam String password,
                        @RequestParam Integer age){
    //TODO
    return AjaxResult.OK();
}
```

###单个参数举例
```
@ApiOperation("根据部门Id删除")
@ApiImplicitParam(name="depId",value="部门id",required=true,paramType="query")
@GetMapping("/delete")
public AjaxResult delete(String depId) {
	//TODO
}
```
 ![image.png](https://upload-images.jianshu.io/upload_images/8020666-bb5f917023bd2e82.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
##4、@ApiResponses、@ApiResponse：方法返回值的说明
```
@ApiResponses：方法返回对象的说明
	@ApiResponse：每个参数的说明
	    code：数字，例如400
	    message：信息，例如"请求参数没填好"
	    response：抛出异常的类
```
###举例：
```
@ApiOperation(value = "修改密码", notes = "方法的备注说明，如果有可以写在这里")
@ApiResponses({
        @ApiResponse(code = 400, message = "请求参数没填好"),
        @ApiResponse(code = 404, message = "请求路径找不到")
})
@PostMapping("/changepass")
public AjaxResult changePassword(@AutosetParam SessionInfo sessionInfo,
        @RequestBody @Valid PasswordModel passwordModel) {
	//TODO
}
```
##5、@ApiModel：用于JavaBean上面，表示一个JavaBean
```
@ApiModel：用于JavaBean的类上面，表示此 JavaBean 整体的信息
	（这种一般用在post创建的时候，使用 @RequestBody 这样的场景，请求参数无法使用 @ApiImplicitParam 注解进行描述的时候 ）	
```
##6. @ApiModelProperty：用在JavaBean的属性上面，说明属性的含义
###@ApiModel和 @ApiModelProperty举例：
```
@ApiModel("修改密码所需参数封装类")
public class PasswordModel
{
    @ApiModelProperty("账户Id")
    private String accountId;
//TODO
}
```



##总结：API文档浏览地址
>配置好Swagger2并适当添加注解后，启动SpringBoot应用，

>访问http://localhost:8080/swagger-ui.html 即可浏览API文档。

>另外，我们需要为了API文档的可读性，适当的使用以上几种注解就可以。
 
![image.png](https://upload-images.jianshu.io/upload_images/8020666-eb1ba5c2d0f38325.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#四、把Swagger2的API接口导入Postman
1、访问http://localhost:8080/swagger-ui.html 文档的首页，复制下面这个地址
![image.png](https://upload-images.jianshu.io/upload_images/8020666-6c7121c38be02908.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 

2.打开postman-->import-->import Form Link

 ![image.png](https://upload-images.jianshu.io/upload_images/8020666-5d82a035e85fc471.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/8020666-d78e8fa2e0ffc971.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


