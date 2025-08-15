---
title: OAuth2是如何进行授权登录的？
date: 2025-08-15 17:32:18
tags:  [项目实录,OAuth2]
---
在各个知名软件中都能看到像微信登录、qq登录这种第三方登录模式，大部分所实现的都是基于OAuth2。举一个最生动的例子，我现在要玩王者荣耀，并且我是微信区，在登录微信时后台做了哪些操作？ 

以下是OAuth2进的一些理论知识以及我在项目中的一些实践应用。（第三方微信登录属于后面所说的第四种-授权码模式）

<!-- more -->

## OAuth2理论知识

OAuth2专门面向互联网的复杂权限验证，解决三方交互过程中，达成可信的数据交互。

以下都是以`登录百度时的微信认证登录`为例。

### 重要角色：

**1.资源拥有者：即用户，拥有微信账号**

**2.授权服务器（认证服务器）：【相当于示例中的微信】用于服务提供者对资源拥有者【①】的身份进行认证，对访问资源授权。认证成功会给客户端发令牌，作为访问资源服务器的凭证。**

**3.资源服务器：【相当于百度】存储资源的服务器**

**需要向授权服务器申请access-token**

### 四种授权模式：

#### 客户端模式

直接客户端给认证服务器身份【三方都很信任，适用于内部系统】

向认证服务器传的参数包括客户端id，客户端密钥，【相当于用来表示用于百度，其实是应用注册时的标识】grant_type：授权模式



#### 密码模式

将1客户端模式中传入的参数grant_type换为password，还需补充两个信息：用户名和密码。

【用于资源拥有者和授权服务器对客户端完全信任，可用于一些内部开发的客户端】

#### 简化模式：

不再是直接发送一个请求来获得key

首先在请求中带有客户端id、重定向地址，接着请求进uaa服务器进行认证之后会自己跳转到资源服务器页面，从根本上解决了别人会窃取信息，即信息泄露的问题

#### 授权码模式【微信的标准实现】

相对于第三种模式来说：仍然需要去访问一个地址，带有参数：客户端id、response_type=code、重定向地址

接着返回的不是**key**，而是**code**。（该code是传到后端的，然后再向认证服务器的接口传参：与一相似，grant_type为authorization_code，code为刚才的code，重定向地址）

### 个人理解

 **以微信登录为例，用户首先请求登录第三方应用，在认证时该应用会携带参数（该应用在微信注册时的AppId、以及重定向地址、response_type=code（表示请求授权码））跳转到微信OAuth2授权登录接口，然后用户进行扫码（这个就相当于输入密码加用户授权）接着微信会重定向到刚才的应用并携带授权码code，此时第三方服务器再通过授权码code加上appId、appsecret来获取access_token,【但它只会在服务器端使用，不会暴露给前端】此时再将access_token交给第三方应用来获取用户信息。**

### 更准确的描述（简化版）

第三方应用跳转到微信授权页（带 appid + redirect_uri）

用户扫码/确认授权 → 微信回调 redirect_uri?code=xxx&state=yyy

第三方后端用 appid + appsecret + code 向微信换 access_token

第三方后端用 access_token 获取用户信息并登录

## 项目中实现

### 项目实现步骤

#### **搭建认证服务**

##### **引入三个自定义依赖，在制品库中已经定义好。**

##### **2.修改配置文件**

修改模块application配置：

##### **3.生成RSA证书：oauth2自述文档**

##### **4.书写配置类**

##### **5.数据层接口与实现**

使用代码工具生成user、role、menu表映射代码

##### **6.服务层接口与实现**

代码模块：**配置类:**

Oauth2Config: 启用 OAuth2 功能。

MyBatisInit 和 RedisInit: 分别初始化 MyBatis 和 Redis 相关配置。

**实体类:**

User, Role, Menu: 分别表示用户、角色和菜单实体。

**Mapper接口:**

UserMapper, RoleMapper, MenuMapper: 分别用于操作用户、角色和菜单数据。

**Service层:**

IUserService, IRoleService, IMenuService: 定义了用户、角色和菜单的基本服务接口。

UserServiceImpl, RoleServiceImpl, MenuServiceImpl: 实现了基本的服务接口。

LoadUserDetailServiceImpl: 实现了用户详情加载逻辑，用于认证。

JwtTokenEnhancerDataServiceImpl: 用于增强 JWT Token 的信息。

ResourcesServiceImpl: 实现了资源与角色的缓存初始化。

Controller层:

ResourcesCacheController: 提供重新加载资源缓存的接口。

### 模块总结

这个模块基于 Spring Security + OAuth2，完成了用户认证（用户名密码）、授权（角色-资源映射），并通过 JWT 实现无状态认证，Redis 加速权限校验。

核心流程是用户登录时由 LoadUserDetailServiceImpl 加载用户信息，BCrypt 校验密码，权限初始化通过 ResourcesServiceImpl 把路径-角色映射放 Redis，访问资源时解析 Token 并比对角色权限。

### 一些问题

#### OAuth2 是什么？它和 Spring Security 的关系是什么？

- OAuth2 是一种授权协议，常用于第三方授权登录。

- Spring Security 是一个安全框架，提供认证（Authentication）和授权（Authorization）能力。

- Spring Security OAuth2 是基于 Spring Security 的 OAuth2 实现，模块是认证服务的核心。

#### JWT 是什么？为什么用它？

JWT（JSON Web Token）是一种无状态认证方式，包含用户信息、签名，可放在请求头里。

优点：无状态（不需要 Session）、跨服务使用方便，适合微服务。

#### Redis 在这里的作用是什么？

存储资源和角色的映射关系，用于快速权限判断。

也可作为缓存，减少数据库查询。

#### BCrypt 为什么用于加密密码？

BCrypt 有加盐，抗暴力破解，安全性比 MD5/SHA1 高。

#### RSA 证书的作用是什么？

用于 JWT 签名和验证，保证 Token 不被篡改。

### 项目整体结构以及认证流程

#### 用户认证流程（可以画成时序图）

用户访问登录接口，携带 username 和 password。

LoadUserDetailServiceImpl 从数据库查询用户信息和角色列表。

Spring Security 内部用 BCryptPasswordEncoder 校验密码。

认证成功后，生成 JWT Token，通过 JwtTokenEnhancerDataServiceImpl 把额外信息（如用户ID）放进去。

Token 返回给前端，后续访问带着 Authorization: Bearer <token>

#### 权限控制流程

ResourcesServiceImpl 初始化菜单路径和角色映射，存入 Redis。

用户访问受保护资源时，Spring Security 拦截请求。

读取 Token → 解析角色 → 判断角色是否匹配 Redis 缓存的资源权限。

如果有权限放行，否则返回 403。