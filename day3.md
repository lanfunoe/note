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

就是将StringRedisTemplate----spring的redis操作对象，传给上层创建一个对象用以执行set，get等方法。

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

### SecurityConfig

Spring Security下的[枚举](https://so.csdn.net/so/search?q=枚举&spm=1001.2101.3001.7020)SessionCreationPolicy,管理session的创建策略

ALWAYS

总是创建HttpSession

IF_REQUIRED

Spring Security只会在需要时创建一个HttpSession

NEVER

Spring Security不会创建HttpSession，但如果它已经存在，将可以使用HttpSession

STATELESS

Spring Security永远不会创建HttpSession，它不会使用HttpSession来获取SecurityContext

### JwtAuthenticationTokenFilter

- Authentication与UserDetails对比

​	前者是登录产生的对象，根据提交的数据产生，

​	UserDetails是从数据库（一般）读取数据产生的对象。

- 实例化UsernamePasswordAuthenticationToken之后调用了setDetails(request,authRequest)将请求的信息设到UsernamePasswordAuthenticationToken中去，包括ip、session等内容

  

### RestfulAccessDeniedHandler

实现AccessDeniedHandler

flush：缓存区清空

getWriter：返回一个PrintWriter，以供读写输出。

response.setContentType设置返回内容格式

 	   text/html           HTML
 	    text/plain          TXT
 	    text/xml             XML
 	    application/json     json字符串



```
compact()
根据JWT Compact Serialization规则构建JWT并将其序列化为紧凑的URL安全字符串。
```

### UmsAdminServiceImpl

- BeanUtils

工具类使用copyProperties对对象进行复制。

- ```
  passwordEncoder.matches(password, userDetails.getPassword()) 
  比较方法
  ```

- BadCredentialsException 密码错误类
- 



## Spring Security 常用配置详解

[引用](https://www.jianshu.com/p/77b4835b6e8e)

[引用](https://blog.csdn.net/MarcoAsensio/article/details/104573094)

[引用](https://www.cnblogs.com/woyujiezhen/p/13049979.html)

- 方法级别安全的配置：在调用方法的时候来进行验证和授权

实现：配置类上加@EnableGlobalMethodSecurity

```
prePostEnabled： 确定 前置注解
[@PreAuthorize,@PostAuthorize,..] 是否启用
securedEnabled： 确定安全注解 [@Secured] 是否启用
jsr250Enabled： 确定 JSR-250注解 [@RolesAllowed..]是否启用
```



在同一个应用程序中，可以启用多个类型的注解，但是只应该设置一个注解对于行为类的接口或者类

```
    // 下面不能设置两个注解，如果设置两个，只有其中一个生效
    // @PreAuthorize("hasAnyRole('user')")
  @Secured({ "ROLE_user", "ROLE_admin" })
  void deleteUser();
}
```

### securedEnabled = true

`@Secured`注解是用来定义业务方法的安全配置。在需要安全[角色/权限等]的方法上指定 @Secured，并且只有那些角色/权限的用户才可以调用该方法。

`@Secured`缺点（限制）就是不支持`Spring EL`表达式。不够灵活。并且指定的角色必须以`ROLE_`开头，不可省略。

```java
eg:
@Secured({"ROLE_user"})
    void updateUser(User user);
```

### prePostEnabled = true

**`@PreAuthorize`：** 进入方法之前验证授权。可以将登录用户的`roles`参数传到方法中验证。

```java
// 只能user角色可以访问
@PreAuthorize ("hasAnyRole('user')")
// user 角色或者 admin 角色都可访问
@PreAuthorize ("hasAnyRole('user') or hasAnyRole('admin')")
// 同时拥有 user 和 admin 角色才能访问
@PreAuthorize ("hasAnyRole('user') and hasAnyRole('admin')")
// 限制只能查询 id 小于 10 的用户
@PreAuthorize("#id < 10")
User findById(int id);

// 只能查询自己的信息
 @PreAuthorize("principal.username.equals(#username)")
User find(String username);

// 限制只能新增用户名称为abc的用户
@PreAuthorize("#user.name.equals('abc')")
void add(User user)
```

**`@PostAuthorize`：** 该注解使用不多，在方法执行后再进行权限验证。 适合验证带有返回值的权限。`Spring EL` 提供 返回对象能够在表达式语言中获取返回的对象`returnObject`

```java
// 查询到用户信息后，再验证用户名是否和登录用户名一致
@PostAuthorize("returnObject.name == authentication.name")
@GetMapping("/get-user")
public User getUser(String name){
    return userService.getUser(name);
}
// 验证返回的数是否是偶数
@PostAuthorize("returnObject % 2 == 0")
public Integer test(){
    // ...
    return id;
}
```

**`@PreFilter`：** 对集合类型的参数执行过滤，移除结果为`false`的元素

```java
// 指定过滤的参数，过滤偶数
@PreFilter(filterTarget="ids", value="filterObject%2==0")
public void delete(List<Integer> ids, List<String> username)
```

**`@PostFilter`：** 对集合类型的返回值进行过滤，移除结果为`false`的元素

```java
@PostFilter("filterObject.id%2==0")
public List<User> findAll(){
    ...
    return userList;
}
```

### 启用jsr250Enabled

jsr250Enabled注解比较简单，只有

- **`@DenyAll`：** 拒绝所有访问
- **`@RolesAllowed({"USER", "ADMIN"})`：** 该方法只要具有`"USER"`, `"ADMIN"`任意一种权限就可以访问。这里可以省略前缀`ROLE_`，实际的权限可能是`ROLE_ADMIN`
- **`@PermitAll`：** 允许所有访问

### configure(AuthenticationManagerBuilder)

 用于通过允许AuthenticationProvider容易地添加来建立认证机制。

也就是说用来记录账号，密码，角色信息，用于配置UserDetailsService及PasswordEncoder。

下方代码不从数据库读取，直接手动赋予

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
AuthenticationManagerBuilder allows 
    public void configure(AuthenticationManagerBuilder auth) {
        auth
            .inMemoryAuthentication()
            .withUser("user")
            .password("password")
            .roles("USER")
        .and()
            .withUser("admin")
            .password("password")
            .roles("ADMIN","USER");
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

### configure(HttpSecurity)

允许基于选择匹配在资源级配置基于网络的安全性。以下示例将以/ admin /开头的网址限制为具有ADMIN角色的用户，并声明任何其他网址需要成功验证。

也就是对角色的权限——所能访问的路径做出限制，用于配置需要拦截的url路径、jwt过滤器及出异常后的处理器；

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
protected void configure(HttpSecurity http) throws Exception {
    http
        .authorizeUrls()
        .antMatchers("/admin/**").hasRole("ADMIN")
        .anyRequest().authenticated()
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

### configure(WebSecurity)

用于影响全局安全性(配置资源，设置调试模式，通过实现自定义防火墙定义拒绝请求)的配置设置。

一般用于配置全局的某些通用事物，例如静态资源等

```
public void configure(WebSecurity web) throws Exception {
    web
        .ignoring()
        .antMatchers("/resources/**");
}
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
