# Spring Security 5.7.3

## 前言

[Spring Security](https://docs.spring.io/spring-security/reference/index.html) 是一个针对常见攻击提供身份验证、授权和保护的框架。凭借对保护命令式和反应式应用程序的一流支持，它是保护基于Spring的应用程序的事实标准。

## Servlet 应用程序

### 快速开始

#### 更新依赖

```groovy
dependencies {
    implement "org.springframework.boot:spring-boot-starter-security:5.7.3"
}
```

#### Spring Boot Auto Configuration

Spring Boot 自动配置：

+ 启用 Spring Security 的默认配置，该配置将创建一个 servlet`Filter`名为`springSecurityFilterChain`的 bean 。这个 bean 负责应用程序中的所有安全性（保护应用程序 URL 、验证提交的用户名和密码、重定向到登录表单等）。

+ 创建一个 `UserDetailsService` bean，用户名为 user 并随机生成密码输出到控制台。

+ 使用 Servlet 容器为每个请求注册一个名为 `springSecurityFilterChain ` 的 bean 的 `Filter` 。

Spring Boot 的配置不多，但它做了很多。功能概述如下：

+ 与应用程序的任何交互都需要经过身份验证的用户。

+ 为您生成默认登录表单。

+ 基于表单的身份认证。

+ 使用 BCrypt 算法存储密码。

+ 允许用户登出。

+ [CSRF](https://developer.mozilla.org/zh-CN/docs/Glossary/CSRF) 攻击防范。

+ [Session Fixation](https://en.wikipedia.org/wiki/Session_fixation) 攻击防范。

+ 安全 Header 集成：
  
  + [HTTP Strict Transport Security](https://en.wikipedia.org/wiki/HTTP_Strict_Transport_Security) 用于安全请求。
  
  + [X-Content-Type-Options](https://msdn.microsoft.com/en-us/library/ie/gg622941(v=vs.85).aspx) 集成。
  
  + 缓存控制（稍后可以被应用程序覆盖，以允许缓存静态资源）。
  
  + [X-XSS-Protection](https://msdn.microsoft.com/en-us/library/dd565647(v=vs.85).aspx) 集成。
  
  + X-Frame-Options 集成有助于方式点击劫持 [Clickjacking](https://en.wikipedia.org/wiki/Clickjacking)。

+ 与以下 Servlet API 方法集成：
  
  + [`HttpServletRequest#getRemoteUser()`](https://docs.oracle.com/javaee/6/api/javax/servlet/http/HttpServletRequest.html#getRemoteUser())
  
  + [`HttpServletRequest.html#getUserPrincipal()`](https://docs.oracle.com/javaee/6/api/javax/servlet/http/HttpServletRequest.html#getUserPrincipal())
  
  + [`HttpServletRequest.html#isUserInRole(java.lang.String)`](https://docs.oracle.com/javaee/6/api/javax/servlet/http/HttpServletRequest.html#isUserInRole(java.lang.String))
  
  + [`HttpServletRequest.html#login(java.lang.String, java.lang.String)`](https://docs.oracle.com/javaee/6/api/javax/servlet/http/HttpServletRequest.html#login(java.lang.String,%20java.lang.String))
  
  + [`HttpServletRequest.html#logout()`](https://docs.oracle.com/javaee/6/api/javax/servlet/http/HttpServletRequest.html#logout())

### 架构

本节讨论基于 Servlet 的应用程序中 Spring Security 的高级架构。

#### Filter 回顾

Spring Security 的 Servlet 支持基于 Servlet `Filter` ，因此首先了解 `Filter` 的作用是有帮助的。下图显示了单个 HTTP 请求的处理程序的典型分层。

![](assets/7308d87c177fd354a8e262f6ba7a3252be8f3a25.png)

客户端向应用程序发送一个请求，容器创建一个 `FilterChain `，其中包含 `Filter` 和 `Servlet `，应该根据请求 URI 的路径处理 `HttpServletRequest `。在 SpringMVC 应用程序中，Servlet 是  `DispatcherServlet `的实例。最多一个 Servlet 可以处理单个 `HttpServletRequest `和 `HttpServletResponse `。但是，可以使用多个 `Filter` ：

+ 防止调用下游 `Filter` 或 `Servlet` 。在这种情况下，`Filter` 通常会写入 `HttpServletResponse`。

+ 修改下游 `Filter` 和 `Servlet `使用的 `HttpServletRequest`或 `HttpServletResponse `。

`Filter` 的力量来自传递给它的 `FilterChain` 。

示例 1. FilterChain 用法示例：

```java
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) {
    // do something before the rest of the application
    chain.doFilter(request, response); // invoke the rest of the application
    // do something after the rest of the application
}
```

由于 `Filter` 只影响下游 `Filter` 和 `Servlet` ，因此每个 `Filter` 的调用顺序极其重要。

#### DelegatingFilterProxy

Spring 提供了一个名为 `DelegatingFilterProxy `的 `Filter` 实现，它允许在 Servlet Container 的生命周期和 Spring 的 `ApplicationContext `之间架起桥梁。Servlet Container 允许使用自己的标准注册 `Filter` ，但它不知道 Spring 定义的 Bean 。`DelegatingFilterProxy` 可以通过标准 Servlet Container 机制注册，但将所有工作委派给实现 `Filter` 的 Spring Bean 。

下面是 `DelegatingFilterProxy `如何装配`Filter` 和 `FilterChain `的图片。

![](assets/7df26be86b25ac808f67250e481ca099f9adf675.png)

`DelegatingFilterProxy `从 `ApplicationContext ` 中查找 Bean Filter0，然后调用 Bean Filter0 。 `DelegatingFilterProxy ` 的伪代码如下所示。

示例 2. `DelegatingFilterProxy` 伪代码

```java
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) {
    // Lazily get Filter that was registered as a Spring Bean
    // For the example in DelegatingFilterProxy
delegate
 is an instance of Bean Filter0
    Filter delegate = getFilterBean(someBeanName);
    // delegate work to the Spring Bean
    delegate.doFilter(request, response);
}
```

`DelegatingFilterProxy` 的另一个好处是，它允许延迟查找 `Filter `bean 实例。这很重要，因为容器需要在启动之前注册 `Filter `实例。然而，Spring 通常使用 `ContextLoaderListener `来加载 Spring Beans ，直到 `Filter `实例需要注册后才能加载。

#### FilterChainProxy

Spring Security 的 Servlet 支持包含在 FilterChainProxy 中。`FilterChainProxy `是 Spring Security 提供的一种特殊 `Filter` ，允许通过 `SecurityFilterChain `委托给许多 `Filter` 实例。由于 `FilterChainProxy `是一个Bean，因此它通常包装在 `DelegatingFilterProxy `中。

![](assets/be1fd1a34987cbfbeb8c3af3a5e5df411245641d.png)
