# Carnation（康乃馨）[![Release](https://jitpack.io/v/carnation-security/carnation-spring-boot-starter.svg)](https://jitpack.io/#carnation-security/carnation-spring-boot-starter)

## 介绍

> 康乃馨权限控制框架，名称由来，康乃馨的花语象征着安全，故取名为康乃馨;
>
> 康乃馨是一个非常轻量级的权限认证框架， 由于平时基本都是基于SpringMVC在开发Web应用，所以康乃馨也是基于SpringMVC开发的，使用的是过滤器和AOP拦截来实现的，所以该框架并不是一款通用的权限框架， 同时，框架在保存用户信息时默认使用的redis，因为现在项目基本都有这个嘛。

## 权限认证流程

> 框架主要由**认证器**和**授权器**组成。认证器的主要工作是对用户进行登录认证，由于不同系统的认证逻辑各不相同，所以具体的认证逻辑由开发人员自己完成，在开发人员完成验证用户合法性之后，向认证器提供认证成功后的用户信息（`UserInfo`对象），认证器会自动完成后续认证，最后返回一个`Token`对象（包含`token`和`refreshToken`）；之后访问需要认证的接口时， 授权器会自动校验请求是否携带token以及token的合法性，再通过AOP来验证用户是否具有权限

## 使用方法

> 由于框架是基于SpringMVC的，所以框架已经主动引入了以下相关依赖，使用者可以无需再引入

```groovy
dependencies {
    apiElements 'org.springframework.boot:spring-boot-starter-web'
    apiElements 'org.springframework.boot:spring-boot-starter-aop'
    apiElements 'org.springframework.boot:spring-boot-starter-data-redis' 
    // redis 所所需要的连接池已经引入， 同样不需要再引入
}
```

### 引入Carnation(最新版[![Release](https://jitpack.io/v/carnation-security/carnation-spring-boot-starter.svg)](https://jitpack.io/#carnation-security/carnation-spring-boot-starter))

```groovy
repositories {
    maven { url 'https://jitpack.io' }
    // maven { url 'https://www.jitpack.io' } 如上面无法使用尝试这个
}

dependencies {
    // 版本请以最新的为准
    implementation 'com.github.carnation-security:carnation-spring-boot-starter:1.0.5'
}
```

> 框架提供了默认配置, 可零配置启动, 默认使用字符串作为角色和权限的认证， 如果要使用枚举类作为角色和权限，需要自定义枚举类型的角色，权限和用户信息（三个实现对应的接口），除此之外还要定义对应的注解， 最后将这些全部告诉框架（在注册器哪里可以设置）

## 关联项目

>Carnation框架 [![Release](https://jitpack.io/v/carnation-security/carnation-core.svg)](https://jitpack.io/#carnation-security/carnation-core): [Carnation-Core](https://github.com/carnation-security/carnation-core)
>
>自动配置 [![Release](https://jitpack.io/v/carnation-security/carnation-spring-boot-autoconfigure.svg)](https://jitpack.io/#carnation-security/carnation-spring-boot-autoconfigure): [Carnation-Spring-Boot-Autoconfigure](https://github.com/carnation-security/carnation-spring-boot-autoconfigure)
>
>使用案例：[Carnation-Spring-Boot-Starter-Test](https://github.com/carnation-security/carnation-spring-boot-starter-test)

## 框架的灵活程度

> 框架是非常灵活的，几乎每个地方都可以自定义，具体可以自定义的有

```java
public class CarnationConfiguration {

    /**
     * 框架属性
     */
    private ICarnationProperties carnationProperties;

    /**
     * 授权成功处理器
     */
    private AuthSuccessHandler authSuccessHandler;

    /**
     * 授权失败处理器
     */
    private AuthFailureHandler authFailureHandler;

    /**
     * 认证token处理器
     */
    private AuthenticateTokenHandler authenticateTokenHandler;

    /**
     * 授权token处理器
     */
    private AuthorizeTokenHandler authorizeTokenHandler;

    /**
     * 角色检查器
     */
    private RoleChecker roleChecker;

    /**
     * 全局角色检查注解
     */
    private Class<? extends Annotation> roleCheckAnnotation;

    /**
     * 权限检查器
     */
    private PermissionChecker permissionChecker;

    /**
     * 全局权限检查注解
     */
    private Class<? extends Annotation> permissionCheckAnnotation;

    /**
     * 用户信息的实际类型
     */
    private Class<? extends UserInfo<?, ?>> userInfoType;

    /**
     * 刷新token信息获取器
     */
    private RefreshTokenInfoGetter refreshTokenInfoGetter;

    /**
     * token生成器
     */
    private TokenGenerator tokenGenerator;

    /**
     * 排除的url
     */
    private ExcludeUrlPatterns excludeUrlPatterns;

    /**
     * 跨域配置
     */
    private CorsConfigurationSource corsConfigurationSource;
}
```

> 可配置的属性

```java
@ConfigurationProperties("carnation")
public class CarnationProperties implements ICarnationProperties {

    /**
     * token过期时间
     */
    private Integer tokenExpireIn = 24 * 3600;

    /**
     * 刷新token过期时间
     */
    private Integer refreshTokenExpireIn = 30 * 24 * 3600;

    /**
     * token过期时间单位
     */
    private TimeUnit timeUnit = TimeUnit.SECONDS;

    /**
     * token前缀
     */
    private String tokenPrefix = "carnation.token.";

    /**
     * refresh_token前缀
     */
    private String refreshTokenPrefix = "carnation.refresh_token.";

    /**
     * token名
     */
    private String tokenName = "Authorization";

    /**
     * token的获取位置
     */
    private TokenPosition tokenPosition = TokenPosition.HEADER;

    /**
     * 是否开启刷新token
     */
    private Boolean enableRefreshToken = true;

    /**
     * 开启跨域(开启跨域后拦截器会被注册在第一个)
     */
    private boolean enableCors = false;

    /**
     * 拦截器的注册顺序
     */
    private Integer filterOrder = Ordered.LOWEST_PRECEDENCE;
}
```



## 注意事项

> 1. 在自定义用户信息时请一定要保留一个空参构造器， 框架使用的是jackson进行用户信息序列化， 没有空参构造会报错
> 2. 由于框架可以使用字符串和枚举类型进行权限认证，所以在加注解之前，需要在Contoller类上或方法上增加@CarnationAuth`注解，不然权限认证不会生效
