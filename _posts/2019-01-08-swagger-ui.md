---
layout: post
title: 使用swagger自动生成文档
category : swagger
tagline: "Supporting tagline"
tags : [swagger]
---
{% include JB/setup %}
# **使用swagger自动生成文档**

---


后端的同学肯定会经常碰到这样的需求：这个接口开发完了，但是前端和测试不知道怎么联调和测试，你写一个文档给他们吧，这种要求实在是很细碎，有没有一种方法能自动生成文档从而减少沟通时间呢？
swagger是一个开源的组件，可以自动帮你生成接口文档，下面是具体的代码，
<!--break-->
```
@Configuration
@EnableSwagger2
public class SwaggerConfig {

    @Bean
    public Docket capitalApi() {

        List<ResponseMessage> responseMessageArrayList = Lists.newArrayList(
                new ResponseMessageBuilder()
                        .code(400)
                        .message("Bad Request")
                        .responseModel(new ModelRef(Response.class.getSimpleName()))
                        .build(),
                new ResponseMessageBuilder()
                        .code(500)
                        .message("Internal Server Error")
                        .responseModel(new ModelRef(Response.class.getSimpleName()))
                        .build()
        );
        return new Docket(DocumentationType.SWAGGER_2)
                .useDefaultResponseMessages(false)
                .globalResponseMessage(RequestMethod.GET, responseMessageArrayList)
                .globalResponseMessage(RequestMethod.POST, responseMessageArrayList)
                .globalResponseMessage(RequestMethod.PUT, responseMessageArrayList)
                .forCodeGeneration(true)
                .select()
                .apis(RequestHandlerSelectors.basePackage("path.of.controller"))
                .build()
                //下面这个视需求而定
                .globalOperationParameters(
                        Lists.newArrayList(new ParameterBuilder()
                                .name("accessToken")
                                .description("Authorization Token")
                                .modelRef(new ModelRef("string"))
                                .parameterType("header")
                                .required(false)
                                .build()));
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("API Infomation")
                .description("https://www.page.net/")
                .termsOfServiceUrl("https://www.page.net/")
                .version("1.0")
                .build();
    }
}
```

此外还有一些swagger的注解用来显示字段是否显示，是否必填，比如@required,@hidden等，需要自己去摸索了,项目运行起来后，swagger的访问地址为
http://{your-domain}/{application-name}/swagger-ui.html
此外，有的项目是有拦截器的，可能会拦截你的swagger请求，这时候需要在拦截器中配置，忽略来自swagger的请求，具体配置如下
```
List<String> notNeedLoginUrlList = new LinkedList<>();
notNeedLoginUrlList.add("/error");
notNeedLoginUrlList.add("/login");
notNeedLoginUrlList.add("/swagger-ui.html");
notNeedLoginUrlList.add("/v2/api-docs");
notNeedLoginUrlList.add("/configuration/ui");
notNeedLoginUrlList.add("/swagger-resources");
notNeedLoginUrlList.add("/configuration/security");
notNeedLoginUrlList.add("/webjars/**");
registry.addInterceptor(notNeedLoginUrlList);
```
问题解决,效果如下图所示
![](../image/swagger.png)
