---
layout: "post"
title: "Spring MVC三种方式实现HTTP内容协商(Content Negotiation)"
date: "2017-05-11 10:23"
tags: spring Content-Negotiation http
---

![API Content Negotiation](images/2017/05/content-negotiation-REST-API-longevity-nordic-apis.png)

题图来自于：http://nordicapis.com/content-negotiation/

## 写在前面

Web Service或REST API设计中内容协商（Content Negotiation），简单来说就是根据客户端的支持内容格式情况来封装响应消息体，本文将介绍在Spring MVC项目实现三种常见的内容协商方法。通常情况下来说，我们有如下三种方法来去决定请求支持的媒体类型：

1. URL中使用后缀，例如 .xml/.json
2. URL使用查询参数，例如 ?format=json
3. HTTP头部使用Accept字段

在默认情况下，Spring的内容协商策略管理器（ContentNegotiationManager）会尝试使用这三种策略，如果以上三种策略都没有被启用的话，我们可以定义一个默认的内容类型。

## 内容协商策略

我们先来看看Spring MVC项目中的内容协商策略，假定我们项目要支持XML和JSON这两种媒体类型（或称为内容类型），那我们先需要在我们项目maven工程里面添加对应依赖，在此我们使用Jackson做XML和JSON的序列化和反序列化工作。

```xml
<dependency>
  <groupId>com.fasterxml.jackson.core</groupId>
  <artifactId>jackson-core</artifactId>
  <version>2.8.7</version>
</dependency>
<dependency>
  <groupId>com.fasterxml.jackson.core</groupId>
  <artifactId>jackson-databind</artifactId>
  <version>2.8.7</version>
</dependency>
<dependency>
  <groupId>com.fasterxml.jackson.dataformat</groupId>
  <artifactId>jackson-dataformat-xml</artifactId>
  <version>2.8.7</version>
</dependency>
```

接下来我们看看三种方法（策略）如何在Spring MVC项目中使用。

## 方法一：后缀

默认情况下，这种策略是未被开启，其实Spring框架可以通过检测URL后缀来去确定响应消息体的内容类型的，我们先看下Controller层的API实现代码：

```java
@RequestMapping(
  value = "/users/{id}",
  produces = { "application/json", "application/xml" },
  method = RequestMethod.GET)
public @ResponseBody User getUserById(@PathVariable long id) {
    return usersMap.get(id);
}
```

我们来尝试调用下API，URL里面使用json后缀，表示要API将资源表述为JSON内容类型。

```bash
curl http://localhost:8080/spirng-mvc-content-negotiation/api/users/1.json
```

响应消息体返回如下，返回了user对象的JSON封装:

```json
{
  id: 1,
  name: "Annie",
  email: "annie@sample.com",
  mobilePhone: "18600090009"
}
```

我们来尝试调用下API，URL里面使用xml后缀，表示要API将资源表述为XML内容类型。

```bash
curl http://localhost:8080/spirng-mvc-content-negotiation/api/users/1.xml
```

响应消息体返回如下，返回了user对象的XML封装:

```xml
<User>
  <id>1</id>
  <name>Annie</name>
  <email>annie@sample.com</email>
  <mobilePhone>18600090009</mobilePhone>
</User>
```

如果我们不指定任何的后缀，默认的内容类型会是怎样的呢？

```bash
curl http://localhost:8080/spirng-mvc-content-negotiation/api/users/1
```

现在默认是XML内容类型返回，我们接下来通过Java和XML文件配置两种方法来对这种以后缀作为内容协商方法的策略进行设置。

### 配置方式一：Java代码

file: src/main/java/io/junq/examples/spring/config/WebConfiguration.java

```java
@Override
public void configureContentNegotiation(final ContentNegotiationConfigurer configurer) {
  configurer.favorPathExtension(true)
    .favorParameter(false)
    .ignoreAcceptHeader(true)
    .useJaf(false)
    .defaultContentType(MediaType.APPLICATION_JSON);
}
```

WebConfiguration继承WebMvcConfigurerAdapter类，然后对于configureContentNegotiation方法进行覆盖，来进行默认内容类型的修改，我们来看看具体发生了什么：

* favorPathExtension(true) 使用后缀方式进行内容协商
* favorParameter(false) 禁用使用URL查询方式进行内容协商
* ignoreAcceptHeader(true) 忽略请求头部的Accept字段
* useJaf(false) 禁用JAF去解析内容类型
* defaultContentType(MediaType.APPLICATION_JSON) 设置默认响应消息体内容类型为JSON

### 配置方式二：XML文件

其实跟我们前面使用Java代码是一模一样的，只不过用了XML文件来实现:

```xml
<bean id="contentNegotiationManager"
  class="org.springframework.web.accept.ContentNegotiationManagerFactoryBean">
    <property name="favorPathExtension" value="true" />
    <property name="favorParameter" value="false"/>
    <property name="ignoreAcceptHeader" value="true" />
    <property name="defaultContentType" value="application/json" />
    <property name="useJaf" value="false" />
</bean>
```

## 方法二：查询参数

我们在上面介绍了使用后缀策略来进行内容协商，我们现在来使用URL查询参数的方式进行内容协商，我们先看看我们修改后果的成果。

```bash
curl http://localhost:8080/spirng-mvc-content-negotiation/api/users/1?format=json
```

API响应消息体为：

```json
{
  id: 1,
  name: "Annie",
  email: "annie@sample.com",
  mobilePhone: "18600090009"
}
```

尝试使用XML内容类型：

```bash
curl http://localhost:8080/spirng-mvc-content-negotiation/api/users/1?format=xml
```

API响应消息体为：

```xml
<User>
  <id>1</id>
  <name>Annie</name>
  <email>annie@sample.com</email>
  <mobilePhone>18600090009</mobilePhone>
</User>
```

我们现在来看看是如何进行配置的，跟前面一样，我们的配置方式有两种：Java代码和XML文件。

### 配置方式一：Java代码

```java
public void configureContentNegotiation(final ContentNegotiationConfigurer configurer) {
  configurer
    .favorPathExtension(false)
    .favorParameter(true)
    .parameterName("format")
    .ignoreAcceptHeader(true)
    .useJaf(false)
    .defaultContentType(MediaType.APPLICATION_JSON)
    .mediaType("xml", MediaType.APPLICATION_XML)
    .mediaType("json", MediaType.APPLICATION_JSON);
}
```

跟方法一的配置很是相似有几个地方不太一样，需要注意下：

* favorPathExtension(false) 禁用后缀策略
* favorParameter(true) 启用查询参数策略
* parameterName("format") 内容类型查询参数为format
* mediaType("xml", MediaType.APPLICATION_XML) 设定不同参数值所对应的内容类型

### 配置方式二：XML文件

下面是XML文件配置，跟Java代码基本一致。

```xml
<bean id="contentNegotiationManager"
  class="org.springframework.web.accept.ContentNegotiationManagerFactoryBean">
    <property name="favorPathExtension" value="false" />
    <property name="favorParameter" value="true"/>
    <property name="parameterName" value="format"/>
    <property name="ignoreAcceptHeader" value="true" />
    <property name="defaultContentType" value="application/json" />
    <property name="useJaf" value="false" />

    <property name="mediaTypes">
        <map>
            <entry key="json" value="application/json" />
            <entry key="xml" value="application/xml" />
        </map>
    </property>
</bean>
```

我们也可以同时启用后缀和URL查询参数的两种策略。

```java
@Override
public void configureContentNegotiation(final ContentNegotiationConfigurer configurer) {
    configurer
      .favorPathExtension(true)
      .favorParameter(true)
      .parameterName("format")
      .ignoreAcceptHeader(true)
      .useJaf(false)
      .defaultContentType(MediaType.APPLICATION_JSON)
      .mediaType("xml", MediaType.APPLICATION_XML)
      .mediaType("json", MediaType.APPLICATION_JSON);
}
```

这种情况下，Spring会先使用后缀策略，如果没有后缀的话，将使用URL查询参数的策略，如果两种方式都没有找到内容类型协商的话，将返回默认内容类型。

## 方法三：头部Accept字段

如果在请求头部Accept字段被启用的话，Spring MVC将在接收到请求会提取出来头部Accept字段，然后来决定内容类型。我们在尝试这个策略时，需要先将ignoreAcceptHeader(false)，以便可以接受Accept字段，同时我们要禁用上面说到的两种策略，这样的话，确保我们内容协商是由请求头部Accept字段提供的。

### 配置方式一：Java代码

```java
@Override
public void configureContentNegotiation(final ContentNegotiationConfigurer configurer) {
  configurer
    .favorPathExtension(false)
    .favorParameter(false)
    .ignoreAcceptHeader(false)
    .useJaf(false)
    .defaultContentType(MediaType.APPLICATION_JSON);
}
```

接下来，你可以基于三种情况来访问访问User API，看看响应消息体内容类型是否正确？

```bash
curl http://localhost:8080/spirng-mvc-content-negotiation/api/users/1
```

```bash
curl -H "Accept:application/json" http://localhost:8080/spirng-mvc-content-negotiation/api/users/1
```

```bash
curl -H "Accept:application/xml" http://localhost:8080/spirng-mvc-content-negotiation/api/users/1
```

### 配置方式二：XML文件

```xml
<bean id="contentNegotiationManager"
  class="org.springframework.web.accept.ContentNegotiationManagerFactoryBean">
    <property name="favorPathExtension" value="false" />
    <property name="favorParameter" value="false"/>
    <property name="ignoreAcceptHeader" value="false" />
    <property name="defaultContentType" value="application/json" />
    <property name="useJaf" value="false" />
</bean>
```

## 回顾总结

API一般会被多种客户端进行使用，如何使得各个平台的不同客户端都得到自己期望的响应消息体内容类型，内容协商便显得十分重要，我们介绍了在Spring MVC项目的三种方法来实现内容协商，希望大家能够在实际项目中做好内容协商。

[示例代码地址: https://github.com/cnjunq/spring-mvc-content-negotiation](https://github.com/cnjunq/spring-mvc-content-negotiation)
