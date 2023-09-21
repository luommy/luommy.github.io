---
title: Nacos相关记录
date: '2023-09-20 15:39:51'
updated: '2023-09-21 09:12:17'
excerpt: Nacos配置与使用；......
tags:
  - Nacos
permalink: /post/nacos-configuration-and-use-zm1qbo.html
comments: true
toc: true
---
# Nacos配置与使用

> Nacos1.X与2.X有差异，目前基本使用2.X版本，也是推荐的版本
>
> Nacos初次尝试....

‍

‍

## 权限认证

🔒开启权限认证：

> 注意
>
> * Nacos是一个内部微服务组件，需要在可信的内部网络中运行，不可暴露在公网环境，防止带来安全风险。
> * Nacos提供简单的鉴权实现，为防止业务错用的弱鉴权体系，不是防止恶意攻击的强鉴权体系。
> * 如果运行在不可信的网络环境或者有强鉴权诉求，请参考官方简单实现做进行[自定义插件开发](https://nacos.io/zh-cn/docs/v2/plugin/auth-plugin.html)。

修改[nacos配置](https://so.csdn.net/so/search?q=nacos%E9%85%8D%E7%BD%AE&spm=1001.2101.3001.7020)文件

——这个时候再访问nacos控制台页面，则会直接报错。

因此，还需要再设置两个属性（数值可随便填）

```properties
nacos.core.auth.server.identity.key=authKey
nacos.core.auth.server.identity.value=nacosSecurty
```

这两个属性是auth的白名单，用于标识来自其他服务器的请求。

添加好这两个属性时页面就能正常访问了。

---

还需要再其他服务的配置文件中加上如下配置，这也就是服务注册的权限

（修改代码方式）注意：密码不要有特殊符号不然会报错

```properties
spring.cloud.nacos.username=naco
spring.cloud.nacos.password=nacos
```

🤡

**此外还需要配置：**

|NACOS_AUTH_TOKEN|token|默认:SecretKey012345678901234567890123456789012345678901234567890123456789|
| ------------------| -------| ----------------------------------------------------------------------------|

‍
