# Spring Boot (5.7.3) 应用初始化流程

## 创建 Spring Application 实例

```java
@SpringBootApplication
public class AuthorizationServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(AuthorizationServerApplication.class, args);
    }
}
```

![](assets/2022-10-03-17-55-16-image.png)

Spring Boot 应用程序启动。

![](assets/2022-10-03-17-56-42-image.png)

![](assets/2022-10-03-17-58-08-image.png)

![](assets/2022-10-04-09-27-38-image.png)

创建 SpringApplication 实例：

1. 将当前主启动类赋值给 primarySources。在这里指的是 AuthorizationServerApplication.class。

2. 推断 Web 应用程序类型为 Servlet 。

3. 通过 `getSpringFactoriesInstances()` 方法获取类路径下`META-INF/spring.factories` 文件，并从中查找关于 `org.springframework.boot.BootstrapRegistryInitializer` 的配置。
   
   举例说明：
   
   ![](assets/2022-10-04-10-30-34-image.png)
   
   ![](assets/2022-10-04-10-31-00-image.png)
   
   ![](assets/2022-10-04-10-31-43-image.png)

4. 与第三步同样的方式查找关于 `org.springframework.context.ApplicationContextInitializer` 的配置。
   
   举例说明：
   
   ![](assets/2022-10-04-10-46-27-image.png)

5. 与上述同样的方式查找关于 `org.springframework.context.ApplicationListener` 的配置。
   
   举例说明：
   
   ![](assets/2022-10-04-10-48-51-image.png)

6. 查找 main 方法。
   
   ![](assets/2022-10-04-10-56-33-image.png)

## getSpringFactoriesInstances() 方法说明

![](assets/2022-10-04-10-12-27-image.png)

![](assets/2022-10-04-10-16-36-image.png)

![](assets/2022-10-04-10-17-31-image.png)

![](assets/2022-10-04-10-18-19-image.png)

获取类路径下 `META-INF/spring.factories` 文件，从中加载所有配置类，并缓存到 `cache` 中。下次获取配置类优先从缓存中查找。

## 应用程序启动

![](assets/2022-10-04-16-37-21-image.png)

### DefaultBootstrapContext

创建 `DefaultBootstrapContext` 实例，并执行所有初始化器的初始化方法。

![](assets/2022-10-04-11-15-48-image.png)

![](assets/2022-10-04-11-17-56-image.png)

### SpringApplicationRunListeners

创建 `SpringApplicationRunListeners` 实例，并从类路径下 `META-INF/spring.factories` 文件中获取 `org.springframework.boot.SpringApplicationRunListener` 配置类。

![](assets/2022-10-04-11-21-29-image.png)

初始化 `SpringApplicationRunListener` 类型配置类，例如：EventPublishingRunListener 。

![](assets/2022-10-04-11-32-36-image.png)

### DefaultApplicationArguments

封装 main 方法传递进来的参数 `args` 。

### prepareEnvironment

创建 `ApplicationServletEnvironment` 实例和配置环境变量。

### configureIgnoreBeanInfo

配置 `spring.beaninfo.ingore` 环境变量。

![](assets/2022-10-04-17-13-04-image.png)

### printBanner

打印 Banner。

![](assets/2022-10-04-17-21-11-image.png)

![](assets/2022-10-04-17-22-15-image.png)

获取 Banner 文件。

![](assets/2022-10-04-17-23-40-image.png)

加载图片类型的 Banner。优先从环境变量 `spring.banner.image.location` 属性中指定的路径加载。如果路径为空，则从类路径下加载文件名为 banner 且扩展类型是 `.gif|.jpg|.png`的文件。如果上述都不满足，则返回 null 。

![](assets/2022-10-04-17-29-10-image.png)

加载文本类型的 Banner。优先从环境变量 `spring.banner.image.location` 属性中指定的路径加载。否则，从类路径下查找文件名为 ` banner.txt` 的文件。如果上述条件都不满足，则返回 null 。

![](assets/2022-10-04-17-33-50-image.png)

如果图片类型和文本类型的 Banner 都没有找到，则使用 Spring 提供的默认 Banner 。

![](assets/2022-10-04-17-39-51-image.png)

### createApplicationContext

根据 Web 应用程序类型，创建上下文环境（IOC）。

![](assets/2022-10-04-17-47-20-image.png)

![](assets/2022-10-04-18-06-11-image.png)

### prepareContext

上下文环境准备。

![](assets/2022-10-05-16-29-26-image.png)

### refreshContext

刷新上下文。

![](assets/2022-10-05-16-31-35-image.png)

![](assets/2022-10-05-16-32-03-image.png)

![](assets/2022-10-05-16-32-33-image.png)

![](assets/2022-10-05-16-33-13-image.png)

#### prepareBeanFactory

Bean 工厂配置。

![](assets/2022-10-05-16-49-26-image.png)

![](assets/2022-10-05-16-53-10-image.png)

#### invokeBeanFactoryPostProcessors

调用 Bean 工厂后置处理器。

#### registerBeanPostProcessors

注册 Bean 后置处理器。

#### initMessageSource

![](assets/2022-10-05-17-29-19-image.png)

#### initApplicationEventMulticaster

初始化 `ApplicationEventMulticaster` 。

![](assets/2022-10-05-17-30-41-image.png)

#### onRefresh

创建 Web 服务。

![](assets/2022-10-05-17-36-09-image.png)

#### registerListeners

注册事件监听器。

![](assets/2022-10-05-17-39-03-image.png)

#### finishBeanFactoryInitialization

创建 Bean 实例。

![](assets/2022-10-05-17-44-55-image.png)

![](assets/2022-10-05-17-48-52-image.png)

##### getBean

![](assets/2022-10-05-18-00-33-image.png)

![](assets/2022-10-05-18-01-51-image.png)

###### populateBean

属性值填充。

![](assets/2022-10-05-18-12-43-image.png)

###### initializeBean

初始化 Bean。

![](assets/2022-10-05-18-15-37-image.png)

applyBeanPostProcessorsBeforeInitialization。

![](assets/2022-10-05-18-16-46-image.png)

invokeInitMethods。

![](assets/2022-10-05-18-19-56-image.png)

![](assets/2022-10-05-18-23-24-image.png)

applyBeanPostProcessorsAfterInitialization。

![](assets/2022-10-05-18-24-10-image.png)
