---
title: 解决SpringBoot项目用Maven打成war包后Tomcat部署报错404
description: '解决将Springboot项目用Maven打成war包后部署到Tomcat无明显报错访问却404的问题'
publishDate: 2025-11-11 20:36:52
tags:
  - java
  - spinrgboot
  - tomcat
---

今天我把我之前写的SpringBoot项目打包部署到Tomcat时发现报错404，但是Tomcat又无明显报错信息。

经过一番尝试过后找到了解决方案。

## 问题描述

将项目部署到Tomcat时，若项目没有问题，应该能在Tomcat catalina.out 日志中发现SringBoot的启动画面，如下：

![SpringBoot启动画面](https://cos.arshe.cn/Troubleshooting-the-404-not-found-error-when-deploying-a-Maven-packaged-WAR-file-of-a-SpringBoot-application-to-Tomcat/Snipaste_2025-11-11_20-57-51.png)

如果发现SpringBoot项目没有启动，而访问网页Tomcat报错404，那大概率跟我遇到了一个问题。

---

## Tomcat版本与SpringBoot版本不匹配

以我的为例，我的SpringBoot版本为3.5.3，但是如果尝试用Tomcat 10以下的版本去部署就会出现以下警告：

```plaintext
WARNING [main] org.apache.tomcat.util.descriptor.web.WebXml.setVersion Unknown version string [5.0]. Default version will be used.
WARNING [main] org.apache.tomcat.util.descriptor.web.WebXml.setVersion Unknown version string [6.0]. Default version will be used.
```

意思就是要部署的web应用需要的Servlet版本为5.0或6.0，但是当前Tomcat的Servlet版本不足。

如Tomcat 9.0.112最高只支持4.0，自然无法启动项目，现在只需要升级Tomcat版本即可。

想了解具体版本对应信息，可自行百度。

---

## SpringBoot应用未触发初始化

SpringBoot部署到外部的Tomcat时，启动类必须继承`SpringBootServletInitializer`并覆盖`configure`方法

```java
@SpringBootApplication
public class DemoApplication extends SpringBootServletInitializer {
    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
        return application.sources(DemoApplication.class);
    }

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

原因在于不这样做，Tomcat 无法识别这是一个 Spring Boot 应用，只会把它当作普通的 Web 包（无 Spring 初始化逻辑）。

下面来讲解这其中的原理

---

### 内置Tomcat模式与外置Tomcat的区别

当直接在idea中启动SpringBoot时，细心的你会发现它也使用了Tomcat。在`spring-boot-starter-web`依赖中包含了`spring-boot-starter-tomcat`依赖里面内置了Tomcat。

当SpringBoot应用启动时，启动类的`main`方法就是启动的入口。它会先初始化 Spring 容器（加载配置、创建 Bean、扫描接口等），再启动内置 Tomcat，最后将 Spring 应用 “挂载” 到内置 Tomcat 上，整个流程由 Spring Boot 自己控制。

但当你把应用打包成 WAR 包 部署到 外部 Tomcat 时，外部 Tomcat 的启动逻辑是 “固定的”：它只认识 “符合 Servlet 规范的 Web 应用”，会按 Servlet 容器的流程启动（如加载 web.xml、初始化 Servlet 上下文等），完全不会主动去执行你应用中的 `main` 方法。

---

### 关键矛盾点：外部 Tomcat 不认识 Spring Boot 的 main 方法

这时`ServletContainerInitializer`接口的作用就出来了。

外部 Tomcat 启动时，会遵循 Java EE 的 Servlet 3.0+ 规范，在部署 Web 应用时自动检测并调用 “实现了 `ServletContainerInitializer` 接口的类”（这是 Servlet 容器留给开发者的 “钩子”，用于自定义初始化逻辑）。

而 `SpringBootServletInitializer` 就是 Spring Boot 对这个 “钩子” 的封装，它的工作流程如下：

1. **外部 Tomcat 启动**：部署 WAR 包时，检测到 Spring 提供的 `SpringServletContainerInitializer`（实现了 `ServletContainerInitializer` 接口）。
2. **触发 Spring 初始化钩子**：`SpringServletContainerInitializer` 会扫描 WAR 包中 “继承了 `SpringBootServletInitializer` 的类”（也就是你的启动类）。
3. **调用 `configure` 方法**：外部 Tomcat 会主动调用你启动类中覆盖的 `configure` 方法，这个方法的作用是 “告诉 Spring：我的启动类是哪个？需要加载哪些配置？”。
4. **初始化 Spring 容器**：通过 `configure` 方法指定的启动类，Spring 会像执行 main 方法一样，初始化 Spring 容器（加载 @Controller、@Service 等 Bean，扫描接口路由等）。
5. **挂载到外部 Tomcat**：Spring 容器初始化完成后，会将所有接口、Servlet 挂载到外部 Tomcat 上，此时外部 Tomcat 才能识别并转发请求到你的 Spring 应用。

---

### 为什么必须 “继承 + 覆盖 configure 方法”？缺一不可

- **必须继承 `SpringBootServletInitializer`**：只有继承了这个类，你的启动类才会被 `SpringServletContainerInitializer` 扫描到，外部 Tomcat 才知道 “这个类是 Spring 应用的入口”。如果不继承，外部 Tomcat 找不到 Spring 入口，自然不会初始化 Spring 容器。
- **必须覆盖 `configure` 方法**：`configure` 方法的核心是通过 `application.sources(DemoApplication.class)` 告诉 Spring “当前应用的启动类是哪个”。如果不覆盖这个方法，Spring 不知道要加载哪个类的配置（比如 @SpringBootApplication 注解、接口路由等），同样无法完成初始化。