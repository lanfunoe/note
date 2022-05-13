# mall整合Redis实现缓存功能

[今日内容](https://www.macrozheng.com/mall/architect/mall_arch_03.html#redis%E7%9A%84%E5%AE%89%E8%A3%85%E5%92%8C%E5%90%AF%E5%8A%A8)

Redis上篇直接安装在了服务器，相应部分跳过

## 整合依赖

```
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```



##  修改SpringBoot配置文件

```
 redis:
    host: {你的云服务器对应值} # Redis服务器地址
    database: 0 # Redis数据库索引（默认为0）
    port: 6379 # Redis服务器连接端口
    password: # Redis服务器连接密码（默认为空）
    jedis:
      pool:
        max-active: 8 # 连接池最大连接数（使用负值表示没有限制）
        max-wait: -1ms # 连接池最大阻塞等待时间（使用负值表示没有限制）
        max-idle: 8 # 连接池中的最大空闲连接
        min-idle: 0 # 连接池中的最小空闲连接
    timeout: 3000ms # 连接超时时间（毫秒）
```

### 关于opsForValue

建议看源码，描述只可意会不可言传

通过源码可知：

就是将StringRedisTemplate----spring的redis操作对象传给上层创建一个对象用以执行set，get等方法。

## 添加UmsMemberController

> 添加根据电话号码获取验证码的接口和校验验证码的接口

### 关于@Value

[引用文章](https://blog.csdn.net/woheniccc/article/details/79804600)

> 作用：该注解的作用是将我们配置文件的属性读出来，有**@Value(“${}”)**和**@Value(“#{}”)**两种方式

> 区别：① ${ property : default_value }
> ② #{ obj.property? :default_value }
> 第一个注入的是外部配置文件对应的property，第二个则是SpEL表达式对应的内容。 那个
> default_value，就是前面的值为空时的默认值。注意二者的不同，#{}里面那个**obj代表对象**。

此部分就是日常Redis操作，如果还记得Redis就基本没难度

# mall整合SpringSecurity和JWT实现认证和授权（一）

[数据来源](https://jwt.io/introduction)

 > JWT是JSON WEB TOKEN的缩写，它是基于 RFC 7519 标准定义的一种可以安全传输的的JSON对象，由于使用了数字签名，所以是可信任和安全的。

## JWT的组成

- JWT token的格式：header.payload.signature
### header
- header中用于存放签名的生成算法

>  such as HMAC SHA256 or RSA.

```json
{"alg": "HS512"}
```
### payload
- payload中用于存放用户名、token的生成时间和过期时间

  payload包含：claims

  > Claims are statements about an entity (typically, the user) and additional data. 
  >
  > Claims类型：*registered*, *public*, and *private* claims.

#### registered

预定义的不强制，但是推荐，提供一组useful, interoperable claims.

常见：**iss** (issuer), **exp** (expiration time), **sub** (subject), **aud** (audience)

#### Public claims
能被随意定义，但是为了避免冲突，应该在IANA JSON Web Token Registry中定义，或者定义为具有避免冲突namespace的url

#### Private claims
自定义的claims，能在使用他们的各方之间共享信息
```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```
### signature

- signature为以header和payload生成的签名，一旦header和payload被篡改，验证将失败

可以在该网站上获得解析结果：https://jwt.io/

```java
//secret为加密算法的密钥
String signature = HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)
```

## JWT实现认证和授权的原理

- 用户调用登录接口，登录成功后获取到JWT的token；
- 之后用户每次调用接口都在http的header中添加一个叫Authorization的头，值为JWT的token；
- 后台程序通过对Authorization头中信息的解码及数字签名校验来获取其中的用户信息，从而实现认证和授权

##  代码相关

```
compact()
根据JWT Compact Serialization规则构建JWT并将其序列化为紧凑的URL安全字符串。
```



### UserDetails

```
package org.springframework.security.core.userdetails;

import java.io.Serializable;
import java.util.Collection;
import org.springframework.security.core.GrantedAuthority;

public interface UserDetails extends Serializable {
    Collection<? extends GrantedAuthority> getAuthorities();

    String getPassword();

    String getUsername();

    boolean isAccountNonExpired();

    boolean isAccountNonLocked();

    boolean isCredentialsNonExpired();

    boolean isEnabled();
}

```

### Date

```
new Date 当前时间
a.before(b);   a<b?
```

> ps：没有springSecuity 基础，下面看官方文档等方式学习即springSecurity篇
