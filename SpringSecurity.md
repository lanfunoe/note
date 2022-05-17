# SpringSecurity

[学习资料：官方文档](https://docs.spring.io/spring-security/reference/features/authentication/password-storage.html#authentication-password-storage-dpe)

## Password Storage

### DelegatingPasswordEncoder

委托指定算法对密码进行编码

> 格式：{id}encodedPassword

### Password Encoding

idForEncode:决定使用哪种编码器编码密码。

### Password Matching

- 通过id匹配PasswordEncoder，如果出现

   id is not mapped (including a null id) 将会报错：IllegalArgumentException

 此行为可由

 ```java
 DelegatingPasswordEncoder.setDefaultPasswordEncoderForMatches(PasswordEncoder)
 ```

自定义

- 此方式不可恢复明文

### 单用户

  ```java
  User user = User.withDefaultPasswordEncoder()
    .username("user")
    .password("password")
    .roles("user")
    .build();
  System.out.println(user.getPassword());
  // {bcrypt}$2a$10$dXJ3SW6G7P50lGmMkkmwe.20cQQubK3.HZWzG3YB1tlRy.fqvM/BG
  ```

### 多用户

```java
UserBuilder users = User.withDefaultPasswordEncoder();
User user = users
  .username("user")
  .password("password")
  .roles("USER")
  .build();
User admin = users
  .username("admin")
  .password("password")
  .roles("USER","ADMIN")
  .build()
```

虽然哈希了存储的密码，但是密码任然暴露存储和源码中，因此，是不安全的。  

>  PS：各种编码器细节略

### Password Storage Configuration   

Spring Security uses [DelegatingPasswordEncoder](https://docs.spring.io/spring-security/reference/features/authentication/password-storage.html#authentication-password-storage-dpe) by default. However, this can be customized by exposing a `PasswordEncoder` as a Spring bean.   

```java
@Bean
public static PasswordEncoder passwordEncoder() {
    return NoOpPasswordEncoder.getInstance();
}
```

### Change Password Configuration 

​    

```java
http
    .passwordManagement(Customizer.withDefaults())
```

Then, when a password manager navigates to `/.well-known/change-password` then Spring Security will redirect your endpoint, `/change-password`

Or, if your endpoint is something other than `/change-password`, you can also specify that like so:

 

```java
http
    .passwordManagement((management) -> management
        .changePasswordPage("/update-password")
    )
```

 With the above configuration, when a password manager navigates to /.well-known/change-password, then Spring Security will redirect to /update-password.   

## CSRF

> Cross Site Request Forgery (CSRF)

此部分先行略过

## Servlet Applications

### Getting Start

#### Spring Boot automatically:

- 打开默认配置，生成了servlet过滤器，此过滤器是一个名为springSecurityFilterChain的bean，这个bean在你的应用中负全责例如：protecting the application URLs, validating submitted username and passwords, redirecting to the log in form, and so on

- 生成一个username为user，密码随机的bean，bean名称为：UserDetailsService，密码记录在console

- 每个请求都用一个名为SpringsecurityFilterchain的bean注册过滤器。

#### springboot的功能：

- 用户与应用程序的任何交互需要认证

- 为您生成默认登录表单

- 让用户名为user和密码为密码在控制台的用户使用基于表格的身份验证进行身份验证

- 使用bcrypt保护密码存储

- 让用户注销

- CSRF攻击预防

- 会话固定保护

- 安全标头集成

  ```
  HTTP Strict Transport Security for secure requests
  
  X-Content-Type-Options integration
  
  Cache Control (can be overridden later by your application to allow caching of your static resources)
  
  X-XSS-Protection integration
  
  X-Frame-Options integration to help prevent Clickjacking
  ```

  

- 与以下servlet API方法集成：

  ```
  HttpServletRequest#getRemoteUser()
  
  HttpServletRequest.html#getUserPrincipal()
  
  HttpServletRequest.html#isUserInRole(java.lang.String)
  
  HttpServletRequest.html#login(java.lang.String, java.lang.String)
  
  HttpServletRequest.html#logout()
  ```

### Architecture

#### DelegatingFilterProxy

- allows bridging between the Servlet container’s lifecycle and Spring’s ApplicationContext.

  它能通过一般的servlet 容器机制注册，但是能够将工作交付对应的bean完成。

  

  ![delegatingfilterproxy](resource\delegatingfilterproxy.png)

  `DelegatingFilterProxy` looks up *Bean Filter0* from the `ApplicationContext` and then invokes *Bean Filter0*. 

- 另一个优点：能够延迟filter bean实例的装载，一般的容器需要在容器启动前完成filter实例的注册，但是，Spring通常使用ContextLoaderListener来加载bean，这个过程需要注册过滤器实例之后才能完成。

#### FilterChainProxy

![ ](resource\filterchainproxy.png)

Spring Security对Servlet支持包含在FilterChainProxy中。FilterChainProxy是Spring Security提供的特殊filter，允许通过SecurityFilterChain委派许多filter实例。由于FilterChainProxy是一个Bean，因此通常将其包裹在中 [DelegatingFilterProxy](https://docs.spring.io/spring-security/reference/servlet/architecture.html#servlet-delegatingfilterproxy).

#### SecurityFilterChain

![](resource\securityfilterchain.png)

FilterChainProxy使用SecurityFilterchain来确定为此请求应调用的 Spring Security Filter。

SecurityFilterChain被FilterChainProxy注册而不是 DelegatingFilterProxy。

FilterChainProxy为直接在Servlet容器或DeLegatingFilterProxy上注册提供了许多优势

1. 给所有的springsecurity的servl支持提供了一个准确的开始点，因此，debug的时候在这设置断点是一个很好的选择

2. 由于FilterChainProxy是SpringSecurity使用的核心，因此可以执行未被视为可选的任务，例如，它清除了SecurityContext，以避免内存泄漏。它还应用Spring Security的HTTPFireWall来保护应用程序免受某些类型的攻击。

3. 它在确定应该何时调用SecurityFilterChain提供了更大的灵活性，在servlet容器中，仅根据URL调用过滤器。但是，FilterChainProxy可以通过利用RequestMatcher接口来确定基于HTTPServletRequest中任何内容的调用。

4. 可以使用FilterChainProxy来确定应该使用哪个SecurityFilterchain。这允许为应用程序的不同位置提供完全独立的配置。

   ![](D:\Everything\note\resource\multi-securityfilterchain.png)

   FilterChainProxy decides which SecurityFilterChain should be used.

   仅第一个匹配的SecurityFilterChain会被调用，如果一个/api/messages/这样的URL被请求，它将首先匹配SecurityFilterChain0的/api/**的模式，所以仅仅SecurityFilterChain0被调用，虽然SecurityFilterChain n也匹配，如果没有匹配上的，剩余的都交由n号SecurityFilterChain。

   > 请注意，SecurityFilterChain0仅配置了三个实例。但是，SecurityFilterChainn配置了四个。重要的是要注意，每个SecurityFilterchain都可以唯一并隔离配置。实际上，如果应用程序希望Spring Security忽略某些请求，可以配置0个实例。

#### Security Filters

  过滤器顺序是重要的， 但是通常不必要知道这个顺序。

#### Handling Security Exceptions

略

### Authentication

#### Authentication Architecture

##### SecurityContextHolder

![](resource/securitycontextholder.png)

The SecurityContextHolder is where Spring Security stores the details of who is authenticated.Spring Security不在意其内部细节，它包含的值就是当前经过认证的用户。

最简单的方式去表面一个用户是经过认证的就是直接设置SecurityContextHolder。

```java
SecurityContext context = SecurityContextHolder.createEmptyContext();
Authentication authentication =
    new TestingAuthenticationToken("username", "password", "ROLE_USER");
context.setAuthentication(authentication);

SecurityContextHolder.setContext(context);
```

If you wish to obtain information about the authenticated principal, you can do so by accessing the SecurityContextHolder.

```java
SecurityContext context = SecurityContextHolder.getContext();
Authentication authentication = context.getAuthentication();
String username = authentication.getName();
Object principal = authentication.getPrincipal();
Collection<? extends GrantedAuthority> authorities = authentication.getAuthorities();
```

默认情况下，SecurityContexTholder使用threadLocal来存储这些详细信息，这意味着SecurityContext始终可用于同一线程中的方法，即使SecurityContext并没有显式的作为参数传入这些方法。如果注意在执行完 present principal的请求后清理线程，使用threadLocal是安全的。SpringSecurity的FilterChainProxy确保了SecurityContext被清除。

某些方式用ThreadLocal是不合适的，例如：

a Swing client might want all threads in a Java Virtual Machine to use the same security context

SecurityContexTholder可以使用启动策略配置，以指定您希望如何存储上下文，对于独立应用程序，可使用SecurityContextHolder.MODE_GLOBAL策略，其他应用程序可能希望让安全线程产生的线程也采用相同的安全身份，此时应该使用SecurityContextHolder.MODE_INHERITABLETHREADLOCAL。

改变默认SecurityContextHolder.MODE_THREADLOCAL的两种方式：

1.设置系统property。

2.调用静态方法SecurityContextHolder

##### SecurityContext

The SecurityContext is obtained from the SecurityContextHolder. The SecurityContext contains an Authentication object.

##### Authentication

功能：

1.是AuthenticationManager的输入，用户提交认证的证书，When used in this scenario, `isAuthenticated()` returns `false`

2.Represents the currently authenticated user. The current Authentication can be obtained from the SecurityContext.

组成：

- `principal` - 标识用户，使用用户名/密码进行身份验证时，这通常是UserDetails的实例.

- `credentials` - often a password. In many cases this will be cleared after the user is authenticated to ensure it is not leaked.

- `authorities` - GrantedAuthority是授予用户的高级权限，例如：roles，scopes。

##### GrantedAuthority

GrantedAuthoritys are high level permissions the user is granted. A few examples are roles or scopes.

GrantedAuthoritys can be obtained from the Authentication.getAuthorities() method.

返回值类型：Collection

当使用基于账号密码的认证时，GrantedAuthority 通常被UserDetailsService导入。

授予权利的设置通常是应用级别的，不能为单个对象设置，但是可以使用项目的域对象安全功能解决此需求。

##### AuthenticationManager

AuthenticationManager是定义Spring Security过滤器如何执行执行身份验证的API。

控制器（即Spring Security的过滤器调用AuthenticationManager在SecurityContexTholder上设置返回的身份验证

the most common implementation is ProviderManager.



##### ProviderManager

`ProviderManager` delegates to a `List` of AuthenticationProviders.Each `AuthenticationProvider` has an opportunity to indicate that authentication should be successful, fail, or indicate it cannot make a decision and allow a downstream `AuthenticationProvider` to decide.

![](resource/providermanager.png)

 If none of the configured AuthenticationProviders can authenticate, then authentication will fail with a ProviderNotFoundException which is a special AuthenticationException that indicates the ProviderManager was not configured to support the type of Authentication that was passed into it.

 each AuthenticationProvider knows how to perform a specific type of authentication. For example, one AuthenticationProvider might be able to validate a username/password, while another might be able to authenticate a SAML assertion.

This allows each AuthenticationProvider to do a very specific type of authentication, while supporting multiple types of authentication and only exposing a single AuthenticationManager bean.

ProvidManager还允许配置可选的Parent AuthenticationManager,The parent can be any type of AuthenticationManager, but it is often an instance of ProviderManager.

![](resource/providermanager-parent.png)

multiple ProviderManager instances might share the same parent AuthenticationManager

![](resource/providermanagers-parent.png)

默认情况下，ProvidManager将尝试从成功的身份验证请求返回的身份验证对象中清除任何敏感的凭据信息This prevents information like passwords being retained longer than necessary in the `HttpSession`。This may cause issues when you are using a cache of user objects, for example, to improve performance in a stateless application：如果Authentication包含一个指向缓存中对象的指针，例如UserDetails实例，这个操作就会删除他的验证信息，从而导致不可能再次验证cache的值。An obvious solution is to make a copy of the object first, either in the cache implementation or in the `AuthenticationProvider` which creates the returned `Authentication` object.

You need to take this into account if you are using a cache。

另外，您可以禁用ProvidManager上的ErasecredentialSafterAuthentication属性

##### AuthenticationProvider

Multiple AuthenticationProviders can be injected into ProviderManager. Each AuthenticationProvider performs a specific type of authentication. For example, DaoAuthenticationProvider supports username/password based authentication while JwtAuthenticationProvider supports authenticating a JWT token.

##### Request Credentials with AuthenticationEntryPoint

“认证入口点”

功能：

AuthenticationEntryPoint is used to send an HTTP response that requests credentials from a client.

应用场景：

Sometimes a client will proactively include credentials such as a username/password to request a resource. In these cases, Spring Security does not need to provide an HTTP response that requests credentials from the client since they are already included.

In other cases, a client will make an unauthenticated request to a resource that they are not authorized to access. In this case, an implementation of AuthenticationEntryPoint is used to request credentials from the client. The AuthenticationEntryPoint implementation might perform a redirect to a log in page, respond with an WWW-Authenticate header, etc.

##### AbstractAuthenticationProcessingFilter

功能：

AbstractAuthenticationProcessingFilter is used as a base Filter for authenticating a user’s credentials. Before the credentials can be authenticated, Spring Security typically requests the credentials using AuthenticationEntryPoint.Next, the AbstractAuthenticationProcessingFilter can authenticate any authentication requests that are submitted to it.

![](resource/abstractauthenticationprocessingfilter.png)

1. When the user submits their credentials, the AbstractAuthenticationProcessingFilter creates an Authentication from the HttpServletRequest to be authenticated. The type of Authentication created depends on the subclass of AbstractAuthenticationProcessingFilter. For example, UsernamePasswordAuthenticationFilter creates a UsernamePasswordAuthenticationToken from a username and password that are submitted in the HttpServletRequest.

2.  Next, the Authentication is passed into the AuthenticationManager to be authenticated.

3. If authentication fails, then Failure

   The SecurityContextHolder is cleared out.

   RememberMeServices.loginFail is invoked. If remember me is not configured, this is a no-op.

   AuthenticationFailureHandler is invoked.

4.  If authentication is successful, then Success.

   SessionAuthenticationStrategy is notified of a new log in.

   The Authentication is set on the SecurityContextHolder. Later the SecurityContextPersistenceFilter saves the SecurityContext to the HttpSession.

   RememberMeServices.loginSuccess is invoked. If remember me is not configured, this is a no-op(空操作).

   ApplicationEventPublisher publishes an InteractiveAuthenticationSuccessEvent.

   AuthenticationSuccessHandler is invoked.

#### Username/Password

##### Reading the Username & Password

Spring Security provides the following built in mechanisms for reading a username and password from the HttpServletRequest:

Section Summary

 Form

 Basic

 Digest

###### Form

Form Login

Spring Security provides support for username and password being provided through an html form. This section provides details on how form based authentication works within Spring Security.

将用户重定向到表单上的日志:

![](resource/loginurlauthenticationentrypoint.png)

1. First, a user makes an unauthenticated request to the resource `/private` for which it is not authorized.

2. Spring Security’s FilterSecurityInterceptor indicates that the unauthenticated request is Denied by throwing an AccessDeniedException.

3. Since the user is not authenticated, ExceptionTranslationFilter initiates Start Authentication and sends a redirect to the log in page with the configured AuthenticationEntryPoint. In most cases the AuthenticationEntryPoint is an instance of LoginUrlAuthenticationEntryPoint.

4. 然后，浏览器将请求将其重定向到的页面登录页面。

5. 渲染登录页面

   ![](resource/usernamepasswordauthenticationfilter.png)

   1. 登录时UsernamePasswordAuthenticationFilter从HttpServletRequest中抽离用户名和密码生成UsernamePasswordAuthenticationToken，其类型为Authentication

   2. Next, the UsernamePasswordAuthenticationToken is passed into the AuthenticationManager to be authenticated. The details of what AuthenticationManager looks like depend on how the user information is stored.

   3. If authentication fails, then *Failure*

      - The [SecurityContextHolder](https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html#servlet-authentication-securitycontextholder) is cleared out.
      - `RememberMeServices.loginFail` is invoked. If remember me is not configured, this is a no-op.
      - `AuthenticationFailureHandler` is invoked.

   4.  If authentication is successful, then *Success*.

      - `SessionAuthenticationStrategy`收到新登录的通知
      - The [Authentication](https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html#servlet-authentication-authentication) is set on the [SecurityContextHolder](https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html#servlet-authentication-securitycontextholder).
      - `RememberMeServices.loginSuccess` is invoked. If remember me is not configured, this is a no-op.
      - `ApplicationEventPublisher` publishes an `InteractiveAuthenticationSuccessEvent`.
      - The `AuthenticationSuccessHandler` is invoked. 通常，这是一个SimpleRauthenticationsuccesshandler，当我们重定向到登录页面时，它将重定向到由ExceptionTranslationFilter 保存的请求。

      SpringSecurity默认登录表单是可用的，但是一旦提供了任何的servlet based configuration，必须提供基于表单的登录

       A minimal, explicit Java configuration can be found below:

      ```java
      protected void configure(HttpSecurity http) {
      	http
      		// ...
      		.formLogin(withDefaults());
      }
      ```

In this configuration Spring Security will render a default log in page。

自定义：

```java
protected void configure(HttpSecurity http) throws Exception {
	http
		// ...
		.formLogin(form -> form
			.loginPage("/login")
			.permitAll()
		);
}
```

###### Basic

Basic Authentication

First, we see the WWW-Authenticate header is sent back to an unauthenticated client.

![](resource/basicauthenticationentrypoint.png)

1. First, a user makes an unauthenticated request to the resource `/private` for which it is not authorized.

2.  Spring Security’s FilterSecurityInterceptor indicates that the unauthenticated request is Denied by throwing an AccessDeniedException.

3. 启动身份验证， The configured AuthenticationEntryPoint is an instance of BasicAuthenticationEntryPoint which sends a WWW-Authenticate header. The RequestCache is typically a NullRequestCache that does not save the request since the client is capable of replaying the requests it originally requested

   When a client receives the WWW-Authenticate header it knows it should retry with a username and password. Below is the flow for the username and password being processed.

   ![](resource/basicauthenticationfilter.png)

1. When the user submits their username and password, the BasicAuthenticationFilter creates a UsernamePasswordAuthenticationToken which is a type of Authentication by extracting the username and password from the HttpServletRequest.

2.  Next, the UsernamePasswordAuthenticationToken is passed into the AuthenticationManager to be authenticated. The details of what AuthenticationManager looks like depend on how the user information is stored.

3. If authentication fails, then Failure

   The SecurityContextHolder is cleared out.

   RememberMeServices.loginFail is invoked. If remember me is not configured, this is a no-op.

   AuthenticationEntryPoint is invoked to trigger the WWW-Authenticate to be sent again.

4. The `BasicAuthenticationFilter` invokes `FilterChain.doFilter(request,response)` to continue with the rest of the application logic.

   Spring Security’s HTTP Basic Authentication support in is enabled by default. However, as soon as any servlet based configuration is provided, HTTP Basic must be explicitly provided.

   ```java
   protected void configure(HttpSecurity http) {
   	http
   		// ...
   		.httpBasic(withDefaults());
   }
   ```

###### Digest Authentication(摘要认证)

由DigestAuthenticationFilter提供

提醒：

> You should not use Digest Authentication in modern applications because it is not considered secure. The most obvious problem is that you must store your passwords in plaintext, encrypted, or an MD5 format. All of these storage formats are considered insecure. Instead, you should store credentials using a one way adaptive password hash (i.e. bCrypt, PBKDF2, SCrypt, etc) which is not supported by Digest Authentication.

摘要式身份验证尝试解决基本身份验证的许多弱点，特别是通过确保凭据永远不会以明文形式通过网络发送。许多浏览器都支持摘要式身份验证。

摘要式身份验证的核心是“nonce”。这是服务器生成的值。Spring Security的nonce采用以下格式：

示例 1.摘要语法

```txt
base64(expirationTime + ":" + md5Hex(expirationTime + ":" + key))
expirationTime:   The date and time when the nonce expires, expressed in milliseconds
key:              A private key to prevent modification of the nonce token
```

The following provides an example of configuring Digest Authentication with Java Configuration:`NoOpPasswordEncoder`

Example 2. Digest Authentication

```java
@Autowired
UserDetailsService userDetailsService;

DigestAuthenticationEntryPoint entryPoint() {
	DigestAuthenticationEntryPoint result = new DigestAuthenticationEntryPoint();
	result.setRealmName("My App Relam");
	result.setKey("3028472b-da34-4501-bfd8-a355c42bdf92");
}

DigestAuthenticationFilter digestAuthenticationFilter() {
	DigestAuthenticationFilter result = new DigestAuthenticationFilter();
	result.setUserDetailsService(userDetailsService);
	result.setAuthenticationEntryPoint(entryPoint());
}

protected void configure(HttpSecurity http) throws Exception {
	http
		// ...
		.exceptionHandling(e -> e.authenticationEntryPoint(authenticationEntryPoint()))
		.addFilterBefore(digestFilter());
}
```

##### Password Storage

reading a username and password的每一个方法都支持以下支持的存储引擎：

- Simple Storage with [In-Memory Authentication](https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/in-memory.html#servlet-authentication-inmemory)
- Relational Databases with [JDBC Authentication](https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/jdbc.html#servlet-authentication-jdbc)
- Custom data stores with [UserDetailsService](https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/user-details-service.html#servlet-authentication-userdetailsservice)
- LDAP storage with [LDAP Authentication](https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/ldap.html#servlet-authentication-ldap)（轻量级目录访问协议）

###### IN memory(基于内存)

Spring Security’s `InMemoryUserDetailsManager` implements `UserDetailsService` to provide support for username/password based `authentication` that is stored in memory.`InMemoryUserDetailsManager` provides management of `UserDetails` by implementing the `UserDetailsManager` interface.`UserDetail`s based `authentication` is used by Spring Security when it is configured to accept a username/password for authentication.

Example 1. InMemoryUserDetailsManager Java Configuration

```java
@Bean
public UserDetailsService users() {
	UserDetails user = User.builder()
		.username("user")
		.password("{bcrypt}$2a$10$GRLdNijSQMUvl/au9ofL.eDwmoohzzS7.rmNSJZ.0FxO/BTk76klW")
		.roles("USER")
		.build();
	UserDetails admin = User.builder()
		.username("admin")
		.password("{bcrypt}$2a$10$GRLdNijSQMUvl/au9ofL.eDwmoohzzS7.rmNSJZ.0FxO/BTk76klW")
		.roles("USER", "ADMIN")
		.build();
	return new InMemoryUserDetailsManager(user, admin);
}
```

上面的样本以安全的格式存储密码，但有很多不足之处。

In the sample below we leverage [User.withDefaultPasswordEncoder](https://docs.spring.io/spring-security/reference/features/authentication/password-storage.html#authentication-password-storage-dep-getting-started) to ensure that the password stored in memory is protected. However, it does not protect against obtaining the password by decompiling the source code，因此以下不适合工业环境.

Example 2. InMemoryUserDetailsManager with User.withDefaultPasswordEncoder

Java

```java
@Bean
public UserDetailsService users() {
	// The builder will ensure the passwords are encoded before saving in memory
	UserBuilder users = User.withDefaultPasswordEncoder();
	UserDetails user = users
		.username("user")
		.password("password")
		.roles("USER")
		.build();
	UserDetails admin = users
		.username("admin")
		.password("password")
		.roles("USER", "ADMIN")
		.build();
	return new InMemoryUserDetailsManager(user, admin);
}
```

###### JDBC Authentication

`JdbcDaoImpl` implements [UserDetailsService](https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/user-details-service.html#servlet-authentication-userdetailsservice) to provide support for username/password based authentication 

`JdbcUserDetailsManager` extends `JdbcDaoImpl` to provide management of `UserDetails` through the `UserDetailsManager` interface.

- 默认使用场景：

Spring Security提供了基于JDBC的身份验证的默认查询。本节提供了与默认查询一起使用的相应默认模式。您将需要调整模式，以将任何自定义匹配到您使用的查询和选择的数据库。

- 数据库示例（先行跳过

Example 1. Default User Schema

```sql
create table users(
	username varchar_ignorecase(50) not null primary key,
	password varchar_ignorecase(500) not null,
	enabled boolean not null
);

create table authorities (
	username varchar_ignorecase(50) not null,
	authority varchar_ignorecase(50) not null,
	constraint fk_authorities_users foreign key(username) references users(username)
);
create unique index ix_auth_username on authorities (username,authority);SQLCopied!
```

Oracle is a popular database choice, but requires a slightly different schema. You can find the default Oracle Schema for users below.

Example 2. Default User Schema for Oracle Databases

```sql
CREATE TABLE USERS (
    USERNAME NVARCHAR2(128) PRIMARY KEY,
    PASSWORD NVARCHAR2(128) NOT NULL,
    ENABLED CHAR(1) CHECK (ENABLED IN ('Y','N') ) NOT NULL
);


CREATE TABLE AUTHORITIES (
    USERNAME NVARCHAR2(128) NOT NULL,
    AUTHORITY NVARCHAR2(128) NOT NULL
);
ALTER TABLE AUTHORITIES ADD CONSTRAINT AUTHORITIES_UNIQUE UNIQUE (USERNAME, AUTHORITY);
ALTER TABLE AUTHORITIES ADD CONSTRAINT AUTHORITIES_FK1 FOREIGN KEY (USERNAME) REFERENCES USERS (USERNAME) ENABLE;SQLCopied!
```



If your application is leveraging groups, you will need to provide the groups schema. The default schema for groups can be found below.

Example 3. Default Group Schema

```sql
create table groups (
	id bigint generated by default as identity(start with 0) primary key,
	group_name varchar_ignorecase(50) not null
);

create table group_authorities (
	group_id bigint not null,
	authority varchar(50) not null,
	constraint fk_group_authorities_group foreign key(group_id) references groups(id)
);

create table group_members (
	id bigint generated by default as identity(start with 0) primary key,
	username varchar(50) not null,
	group_id bigint not null,
	constraint fk_group_members_group foreign key(group_id) references groups(id)
);
```

- Setting up a DataSource：

   Before we configure `JdbcUserDetailsManager`, we must create a `DataSource`. In our example, we will setup an [embedded DataSource](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/data-access.html#jdbc-embedded-database-support) that is initialized with the [default user schema](https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/jdbc.html#servlet-authentication-jdbc-schema).    

   Example 4. Embedded Data Source

```java
@Bean
DataSource dataSource() {
	return new EmbeddedDatabaseBuilder()
		.setType(H2)
		.addScript(JdbcDaoImpl.DEFAULT_USER_SCHEMA_DDL_LOCATION)
		.build();
}JAVACopied!
```

In a production environment, you will want to ensure you setup a connection to an external database.  

-  JdbcUserDetailsManager Bean

Example 5. JdbcUserDetailsManager

```java
@Bean
UserDetailsManager users(DataSource dataSource) {
	UserDetails user = User.builder()
		.username("user")
		.password("{bcrypt}$2a$10$GRLdNijSQMUvl/au9ofL.eDwmoohzzS7.rmNSJZ.0FxO/BTk76klW")
		.roles("USER")
		.build();
	UserDetails admin = User.builder()
		.username("admin")
		.password("{bcrypt}$2a$10$GRLdNijSQMUvl/au9ofL.eDwmoohzzS7.rmNSJZ.0FxO/BTk76klW")
		.roles("USER", "ADMIN")
		.build();
	JdbcUserDetailsManager users = new JdbcUserDetailsManager(dataSource);
	users.createUser(user);
	users.createUser(admin);
	return users;
}
```

###### UserDetails

`UserDetail`s is returned by the `UserDetailsService`

The `DaoAuthenticationProvider` validates the `UserDetails` ，然后返回具有 principal的`Authentication`。the `UserDetails` returned by the configured `UserDetailsService`

###### UserDetailsService

DaoAuthenticationProvider使用UserDetailsservice来检索用户名，密码和其他属性，进行用户名和密码的验证。  

> This is only used if the AuthenticationManagerBuilder has not been populated and no AuthenticationProviderBean is defined.                

Example 1. Custom UserDetailsService Bean

```java
@Bean
CustomUserDetailsService customUserDetailsService() {
	return new CustomUserDetailsService();
}
```

###### PasswordEncoder

Spring Security’s servlet support storing passwords securely by integrating with `PasswordEncoder`

Customizing the `PasswordEncoder` implementation used by Spring Security can be done by exposing a PasswordEncoder Bean.

###### DaoAuthenticationProvider

 [`DaoAuthenticationProvider`](https://docs.spring.io/spring-security/site/docs/5.7.0/api/org/springframework/security/authentication/dao/DaoAuthenticationProvider.html) is an [`AuthenticationProvider`](https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html#servlet-authentication-authenticationprovider) implementation that                                                                                                    leverages a [`UserDetailsService`](https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/user-details-service.html#servlet-authentication-userdetailsservice) and [`PasswordEncoder`](https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/password-encoder.html#servlet-authentication-password-storage) to authenticate a username and password.   

- 流程图：

![](resource/daoauthenticationprovider.png)

1.   The authentication `Filter`passes a `UsernamePasswordAuthenticationToken` to the `AuthenticationManager` which is implemented by [`ProviderManager`](https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html#servlet-authentication-providermanager)
2.  The `ProviderManager` is configured to use an [AuthenticationProvider](https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html#servlet-authentication-authenticationprovider) of type `DaoAuthenticationProvider`.
3. `DaoAuthenticationProvider` then uses the [`PasswordEncoder`](https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/password-encoder.html#servlet-authentication-password-storage) to validate the password on the `UserDetails` returned in the previous step.
4. `DaoAuthenticationProvider` then uses the [`PasswordEncoder`](https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/password-encoder.html#servlet-authentication-password-storage)验证上一步返回的UserDetails中的密码
5. When authentication is successful, the [`Authentication`](https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html#servlet-authentication-authentication) that is returned is of type `UsernamePasswordAuthenticationToken` and has a principal that is the `UserDetails` returned by the configured `UserDetailsService`. Ultimately, the returned `UsernamePasswordAuthenticationToken` will be set on the [`SecurityContextHolder`](https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html#servlet-authentication-securitycontextholder) by the authentication `Filter`.

###### LDAP Authentication

略

> 其余的先行略过，项目所需基本满足。

### Java Configuration

 The configuration creates a Servlet Filter known as the `springSecurityFilterChain` which is responsible for all the security (protecting the application URLs, validating submitted username and passwords, redirecting to the log in form, etc) within your application.

You can find the most basic example of a Spring Security Java Configuration below:

```java
import org.springframework.beans.factory.annotation.Autowired;

import org.springframework.context.annotation.*;
import org.springframework.security.config.annotation.authentication.builders.*;
import org.springframework.security.config.annotation.web.configuration.*;

@EnableWebSecurity
public class WebSecurityConfig {

	@Bean
	public UserDetailsService userDetailsService() {
		InMemoryUserDetailsManager manager = new InMemoryUserDetailsManager();
		manager.createUser(User.withDefaultPasswordEncoder()
                           .username("user")
                           .password("password")
                           .roles("USER")
                           .build());
		return manager;
	}
}
```

- Allow the user with the **Username** *user* and the **Password** *password* to authenticate with form based authentication
- Allow the user to logout

#### AbstractSecurityWebApplicationInitializer

The next step is to register the `springSecurityFilterChain` with the war. 

- [AbstractSecurityWebApplicationInitializer without Existing Spring](https://docs.spring.io/spring-security/reference/servlet/configuration/java.html#abstractsecuritywebapplicationinitializer-without-existing-spring) - Use these instructions if you are not using Spring already
- [AbstractSecurityWebApplicationInitializer with Spring MVC](https://docs.spring.io/spring-security/reference/servlet/configuration/java.html#abstractsecuritywebapplicationinitializer-with-spring-mvc) - Use these instructions if you are already using Spring

#### AbstractSecurityWebApplicationInitializer without Existing Spring

```java
import org.springframework.security.web.context.*;

public class SecurityWebApplicationInitializer
	extends AbstractSecurityWebApplicationInitializer {

	public SecurityWebApplicationInitializer() {
		super(WebSecurityConfig.class);
	}
}
```

The `SecurityWebApplicationInitializer` will do the following things:

- Automatically register the springSecurityFilterChain Filter for every URL in your application
- Add a ContextLoaderListener that loads the [WebSecurityConfig](https://docs.spring.io/spring-security/reference/servlet/configuration/java.html#jc-hello-wsca).

#### AbstractSecurityWebApplicationInitializer with Spring MVC

If we were using Spring elsewhere in our application we probably already had a `WebApplicationInitializer` that is loading our Spring Configuration. If we use the previous configuration we would get an error. Instead, we should register Spring Security with the existing `ApplicationContext`. For example,

```java
import org.springframework.security.web.context.*;

public class SecurityWebApplicationInitializer
	extends AbstractSecurityWebApplicationInitializer {

}
```

This would simply only register the springSecurityFilterChain Filter for every URL in your application. After that we would ensure that `WebSecurityConfig` was loaded in our existing ApplicationInitializer. For example, if we were using Spring MVC it would be added in the `getRootConfigClasses()`

```java
public class MvcWebApplicationInitializer extends
		AbstractAnnotationConfigDispatcherServletInitializer {

	@Override
	protected Class<?>[] getRootConfigClasses() {
		return new Class[] { WebSecurityConfig.class };
	}

	// ... other overrides ...
}
```

#### HttpSecurity

How does Spring Security know that we want to require all users to be authenticated? How does Spring Security know we want to support form based authentication? 

Actually, there is a bean that is being invoked behind the scenes called `SecurityFilterChain`

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
	http
		.authorizeRequests(authorize -> authorize
			.anyRequest().authenticated()
		)
		.formLogin(withDefaults())
		.httpBasic(withDefaults());
	return http.build();
}
```

The default configuration above:

- Ensures that any request to our application requires the user to be authenticated
- Allows users to authenticate with form based login
- Allows users to authenticate with HTTP Basic authentication

#### Multiple HttpSecurity

the following is an example of having a different configuration for URL’s that start with `/api/`.

```java
@EnableWebSecurity
public class MultiHttpSecurityConfig {
	@Bean                                                             
	public UserDetailsService userDetailsService() throws Exception {
		// ensure the passwords are encoded properly
		UserBuilder users = User.withDefaultPasswordEncoder();
		InMemoryUserDetailsManager manager = new InMemoryUserDetailsManager();
		manager.createUser(users.username("user").password("password").roles("USER").build());
		manager.createUser(users.username("admin").password("password").roles("USER","ADMIN").build());
		return manager;
	}

	@Bean
	@Order(1)                                                        
	public SecurityFilterChain apiFilterChain(HttpSecurity http) throws Exception {
		http
			.antMatcher("/api/**")                                   
			.authorizeHttpRequests(authorize -> authorize
				.anyRequest().hasRole("ADMIN")
			)
			.httpBasic(withDefaults());
		return http.build();
	}

	@Bean                                                            
	public SecurityFilterChain formLoginFilterChain(HttpSecurity http) throws Exception {
		http
			.authorizeHttpRequests(authorize -> authorize
				.anyRequest().authenticated()
			)
			.formLogin(withDefaults());
		return http.build();
	}
}
```

####  Custom DSLs

You can provide your own custom DSLs in Spring Security. For example, you might have something that looks like this:

```java
public class MyCustomDsl extends AbstractHttpConfigurer<MyCustomDsl, HttpSecurity> {
	private boolean flag;

	@Override
	public void init(HttpSecurity http) throws Exception {
		// any method that adds another configurer
		// must be done in the init method
		http.csrf().disable();
	}

	@Override
	public void configure(HttpSecurity http) throws Exception {
		ApplicationContext context = http.getSharedObject(ApplicationContext.class);

		// here we lookup from the ApplicationContext. You can also just create a new instance.
		MyFilter myFilter = context.getBean(MyFilter.class);
		myFilter.setFlag(flag);
		http.addFilterBefore(myFilter, UsernamePasswordAuthenticationFilter.class);
	}

	public MyCustomDsl flag(boolean value) {
		this.flag = value;
		return this;
	}

	public static MyCustomDsl customDsl() {
		return new MyCustomDsl();
	}
}
```

> This is actually how methods like `HttpSecurity.authorizeRequests()` are implemented.

The custom DSL can then be used like this:

```java
@EnableWebSecurity
public class Config {
	@Bean
	public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
		http
			.apply(customDsl())
				.flag(true)
				.and()
			...;
		return http.build();
	}
}
```

The code is invoked in the following order:

- Code in `Config`s configure method is invoked
- Code in `MyCustomDsl`s init method is invoked
- Code in `MyCustomDsl`s configure method is invoked

If you want, you can add `MyCustomDsl` to `HttpSecurity` by default by using `SpringFactories`. For example, you would create a resource on the classpath named `META-INF/spring.factories` with the following contents:

META-INF/spring.factories

```
org.springframework.security.config.annotation.web.configurers.AbstractHttpConfigurer = sample.MyCustomDsl
```

Users wishing to disable the default can do so explicitly.

```java
@EnableWebSecurity
public class Config {
	@Bean
	public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
		http
			.apply(customDsl()).disable()
			...;
		return http.build();
	}
}
```

#### Post Processing Configured Objects

Spring Security introduces the concept of an `ObjectPostProcessor` which can be used to modify or replace many of the Object instances created by the Java Configuration. For example, if you wanted to configure the `filterSecurityPublishAuthorizationSuccess` property on `FilterSecurityInterceptor` you could use the following:

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
	http
		.authorizeRequests(authorize -> authorize
			.anyRequest().authenticated()
			.withObjectPostProcessor(new ObjectPostProcessor<FilterSecurityInterceptor>() {
				public <O extends FilterSecurityInterceptor> O postProcess(
						O fsi) {
					fsi.setPublishAuthorizationSuccess(true);
					return fsi;
				}
			})
		);
	return http.build();
}
```

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

