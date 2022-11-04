# day2    mall整合Swagger-UI实现在线API文档

[今日内容](https://www.macrozheng.com/mall/architect/mall_arch_03.html#redis%E7%9A%84%E5%AE%89%E8%A3%85%E5%92%8C%E5%90%AF%E5%8A%A8)

## [Swagger](https://so.csdn.net/so/search?q=Swagger&spm=1001.2101.3001.7020) UI

> 参考文章如下

[详情见此文章]([Swagger UI简介_真香号的博客-CSDN博客_swagger ui](https://blog.csdn.net/zhanshixiang/article/details/104605292))

[第一篇模糊的地方见此文章](https://blog.csdn.net/zhanggonglalala/article/details/98070986)

[配置相关](https://blog.csdn.net/qq_45017999/article/details/107999533)

 ###   组件

 - Swagger 2

 - UI 显示页面

 ### 配置类

   

```
/**
 * Swagger2API文档的配置
 */
@Configuration
@EnableSwagger2
public class Swagger2Config {
    @Bean
    public Docket createRestApi(){
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                //为当前包下controller生成API文档
                .apis(RequestHandlerSelectors.basePackage("com.macro.mall.tiny.controller"))
                //为有@Api注解的Controller生成API文档
//                .apis(RequestHandlerSelectors.withClassAnnotation(Api.class))
                //为有@ApiOperation注解的方法生成API文档
//                .apis(RequestHandlerSelectors.withMethodAnnotation(ApiOperation.class))
                .paths(PathSelectors.any())
                .build();
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("SwaggerUI演示")
                .description("mall-tiny")
                .contact("macro")
                .version("1.0")
                .build();
    }
}
```

- apiInfo：api基本信息的配置，信息会在api文档上显示，可有选择的填充，比如配置文档名称、项目版本号等
- select：返回 ApiSelectorBuilder 对象，通过对象调用 build()可以创建 Docket 对象


- apis：使用什么样的方式来扫描接口，扫描扫描注释的接口

>
> ​	RequestHandlerSelectors 配置swagger扫描接口的方式
>
> 1. basePackage() 指定要扫描哪些包any() 全部都扫描
> 2. none() 全部不扫描
> 3. withClassAnnotation() 扫描类上的注解 参数是一个注解的反射对象
> 4. withMethodAnnotation() 
>
- path：可以设置满足什么样规则的 url 被生成接口文档。可以使用正则表达式进行匹配，PathSelectors下有四种方法

  ![](resource\142858.png)

### swagger api 注解



- @Api： 用于类，标识这个类是swagger的资源

- @ApiOperation： 用于方法，描述 Controller类中的 method接口

- @ApiParam： 用于参数，单个参数描述。name一般与参数名一致，value是解释，required是是否参数必须

  

### 参数注解----图解

  

  

![image-20220504144342340](resource\image-20220504144342340.png)

## Redis

无Redis，Linux基础，请看Redis篇

> 关于day2内容时间的说明：
>
> 先行学习Redis，Linux内容， 后面的内容留待后日（也算进day2， 但是会在版本中注明"day2    2ed"
>
> -------------------------------------------------------------------------------------------------
