## SpringSecurity

### Password Storage

> 格式：{id}encodedPassword

###  Password Encoding



##	Spring Security 常用配置详解

[引用](https://docs.spring.io/spring-security/reference/features/authentication/password-storage.html#authentication-password-storage-dpe)

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

也就是说用来记录账号，密码，角色信息。

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

也就是对角色的权限——所能访问的路径做出限制

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

