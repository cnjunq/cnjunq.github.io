---
layout: "post"
title: "[教程]在Spring REST API中使用Swagger2进行文档管理"
date: "2017-03-21 22:17"
---

## 写在前面

使用RESTful API作为Web服务对外提供服务的入口，基本上已经成为了标准，在提供REST API的同时，如何进行
API文档管理是一个较为麻烦的事情，作为开发人员我们都了解API文档的重要性，但总是嫌其编写的麻烦，Swagger的
出现很好地帮我们解决文档编写的事情，开发人员可以采取自己喜欢的姿势进行API文档编写，例如使用Swagger-Editor
进行API的提前设计，进而使用Swagger-Codegen来生成多种语言的客户端代码，使用Swagger-UI来进行API文档
的直观查看，当然你也可以在代码中直接进行API文档的编写，本教程将介绍如何在Spring REST API项目中，进行
Swagger2的集成，这样便可以很方便地进行API文档编写了。

在本篇教程中将使用Springfox的Swagger2来集成，关于 Swagger2 和 Springfox 的介绍如下：

* [Swagger 2][fcfb529e]
* [Springfox][effd621e]



## 项目配置

本教程将假设你已经有了一个Spring REST服务，如果没有话，可以参考Spring官方项目示例：

[Building a RESTful Web Service][32062182]

## 添加Maven依赖

在你的Spring REST项目中添加maven依赖，打开项目的pom.xml，添加如下内容：

```
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.6.1</version>
</dependency>
```


## 在项目中集成Swagger2

### 集成Swagger2

首先创建一个SwaggerConfiguration类，如下：

```
@Configuration
@EnableSwagger2
public class SwaggerConfiguration {                                    
    @Bean
    public Docket api() {
        return new Docket(DocumentationType.SWAGGER_2)  
          .select()                                  
          .apis(RequestHandlerSelectors.any())              
          .paths(PathSelectors.any())                          
          .build();                                           
    }
}
```

通过使用`@EnableSwagger2`注解来开启Swagger2，在整个类中最重要的便是`Docket`的Bean，`Docket`定义了
Swagger的版本、需要暴露的API等信息，我们来看看`Docket`生成后都做了些什么工作：

1. 创建一个`Docket`对象
2. 调用select()方法，生成`ApiSelectorBuilder`对象实例，该对象负责定义外漏的API入口
3. 通过使用`RequestHandlerSelectors`和`PathSelectors`来提供Predicate，在此我们使用any()方法，将所有API都通过Swagger进行文档管理

如果你的项目是一个Spring Boot项目，上面的配置已经OK了，可以开始进行API文档的编写了，但如果你的项目
不是Spring Boot项目，只是Spring项目的话，需要稍微多做些工作。

### 非Spring Boot项目中，配置Resource Handler

由于没有使用Spring Boot，这样就不能自动配置Resource Handler，需要自己进行下配置，因为我们接下来要使用
Swagger UI，它有些静态资源需要加载，新建一个类，继承WebMvcConfigurerAdapter并使用@EnableWebMvc注解，然后添加如下代码，如果你已经有了一个这样的类，可以进行将以下代码添加至该类：

```
@Override
public void addResourceHandlers(ResourceHandlerRegistry registry) {
    registry.addResourceHandler("swagger-ui.html")
      .addResourceLocations("classpath:/META-INF/resources/");

    registry.addResourceHandler("/webjars/**")
      .addResourceLocations("classpath:/META-INF/resources/webjars/");
}
```

### 体验成果

假设我是使用Spring REST官方示例项目的话，打开浏览器，输入：

```
http://localhost:8080/v2/api-docs
```

你应该会看到JSON格式的API文档，如下图所示：

![Swagger2 API Docs](images/2017/03/swagger2-api-docs.png)

这样的JSON文件，读起来是很不方便的，接下来我们用Swagger UI来进行查看。

## Swagger UI

Swagger UI提供了Web页面以方便开发人员查看API文档等。

### 集成Swagger UI

在项目pom.xml中引入：

```
<dependency>
			<groupId>io.springfox</groupId>
			<artifactId>springfox-swagger-ui</artifactId>
			<version>2.6.1</version>
</dependency>
```

在此运行示例项目，在浏览器中输入：

```
http://localhost:8080/swagger-ui.html
```

可以看到如下Web页面：

![Swagger UI](images/2017/03/swagger-ui.png)


## 写在后面

到此你已经成功地在项目中集成了Swagger2，接下来可以使用Springfox的注解进行API文档编写了，详细阅读：

http://springfox.github.io/springfox/docs/current/

[fcfb529e]: http://swagger.io/ "Swagger 2"
[effd621e]: https://github.com/springfox/springfox "Springfox"
[32062182]: https://spring.io/guides/gs/rest-service/ "Building a RESTful Web Service"
