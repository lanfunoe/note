# day1    mall整合Spring Boot+MyBatis搭建基本骨架

[项目地址](https://github.com/macrozheng/mall-learning)

[今日内容](https://www.macrozheng.com/mall/architect/mall_arch_01.html#mysql%E6%95%B0%E6%8D%AE%E5%BA%93%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA)

## 数据库

### 搭建数据库-----导入sql；



### 创建逆向工程

1. idea项目搭建；

2. 添加项目依赖；

3. 修改Spring Boot配置文件；

4. Mybatis generator 配置文件

   文件名必须是：`generatorConfig.xml`

   [文件详解](https://blog.csdn.net/weixin_44368212/article/details/109283097)

5. CommentGenerator，properties文件的撰写

6. 运行Generator的main函数，生成model，mapper和xml文件；

## 代码编写

### CommonResult

   - 关于泛型的使用

> ```java
>      * 这个<T> T 可以传入任何类型的List
>      * 参数T
>      *     第一个 表示是泛型
>      *     第二个 表示返回的是T类型的数据
>      *     第三个 限制参数类型为T
>      * @param data
>      * @return
>      */
>     private <T> T getListFisrt(List<T> data) {
>         if (data == null || data.size() == 0) {
>             return null;
>         }
>         return data.get(0);
>     }
> 
> }
> ```

[引用文章](https://www.cnblogs.com/jpfss/p/9929108.html)

###  PmsBrandServiceImpl 

- insert 与 insertSelective
  > 比如User里面有三个字段:id，name，age，password
  > 但是我只设置了一个字段；
  >
  > User u=new user();
  > u.setName（"张三"）；
  > insertSelective（u）；
  >
  > 
  >
  > insertSelective执行对应的sql语句的时候，只插入对应的name字段；（主键是自动添加的，默认插入为空）
  >
  > insert into tb_user （id，name） value （null，"张三"）；
  >
  > 
  >
  > 而insert则是不论你设置多少个字段，统一都要添加一遍，不论你设置几个字段，即使是一个。
  >
  > User u=new user();
  > u.setName（"张三"）；
  > insertSelective（u）；
  >
  > insert into tb_user （id，name，age，password） value （null，"张三"，null，null）；

[引用文章](https://www.cnblogs.com/jayinnn/p/9561341.html)

- PageHelper.startPage(int pageNum, int pageSize)

  pageNum 当前页码， pageSize 每页大小

### PmsBrandController

- @Autowired与@Resource

> @Autowired注解是按照类型（byType）装配依赖对象，默认情况下它要求依赖对象必须存在，如果允许null值，可以设置它的required属性为false。
>
> @Resource默认按照ByName自动注入，由J2EE提供

[引用文章](https://www.cnblogs.com/think-in-java/p/5474740.html)

### CommonPage

- 在查询获取list集合之后，使用`PageInfo<T> pageInfo = new PageInfo<>(List<T> list, intnavigatePages)`获取分页相关数据

  list：分页之后的数据  

  navigatePages：导航分页的页码数

常用数据：

- pageNum：当前页的页码  
- pageSize：每页显示的条数  
- size：当前页显示的真实条数  
- total：总记录数  
- pages：总页数  
- prePage：上一页的页码  
- nextPage：下一页的页码
- isFirstPage/isLastPage：是否为第一页/最后一页  
- hasPreviousPage/hasNextPage：是否存在上一页/下一页  
- navigatePages：导航分页的页码数  
- navigatepageNums：导航分页的页码，\[1,2,3,4,5]
