﻿# Spring
## 1 核心容器
### 1.1 使用配置文件开发
#### 1.1.1 bean对象的构建方式
- 构造方法
- 静态方法
- 实例工厂
#### 1.1.2 依赖注入方式
- setter注入
- 构造器注入
### 1.2 使用注解开发
#### 1.2.1 定义bean对象
@Component
@Controller
@Service
@Repository
@ComponentScan
@Scope

#### 1.2.2 依赖注入
1. 常用注解：
  @Autowired @Qualifier
  @Value
  @PropertyResource

​		@Resource

1. 第三方bean的管理
  在独立配置类返回bean对象，并在方法上加上@Bean注解，再在另一个配置类@Import这个配置类。

## 2  整合
### 2.1 spring 整合 mybatis
### 2.2 spring 整合 Junit

## 3 AOP
### 3.1 基本概念
- AOP的作用：在不改变原始设计的基础上进行功能加强
- 连接点：程序执行时的任何位置
- 切入点：匹配连接点的式子
- 通知：在切入点执行的功能
- 通知类：定义通知的类
- 切面：描述通知和切入点的关系
### 3.2 核心概念
目标对象：被代理的对象
代理：由于原始对象无法完成最终的功能，需要对其进行功能加强，通过原始对象的代理对象实现
### 3.3 切入点表达式
### 3.4 通知类型
前置通知@Before
后置通知@After
环绕通知@Around ，依赖形参ProceedingJoinPoint
### 3.5 通知获取数据
得到方法参数：先拿到JoinPoint对象，再用getArgs()方法拿到参数。
得到返回值：在@AfterReturning的括号里加上returning = "接受返回值的变量名"
## 4 事务
### 4.1 事务角色
- 事务管理员：发起事务方，通常指spring业务层的方法
- 事务协调员：加入事务方，通常是数据层方法或是业务层方法
### 4.2 事务属性
`@Transaction(rollbackFor(异常类))`
### 4.3 事务的传播行为
开启一个新事务：propagation = REQUIRES_NEW 

# SpringMVC
SpringMVC是一种基于 Java 实现的MVC模型的轻量级web框架

# SpringBoot
## 1 yaml数据读取
1. @Value注入
2. 用 Environment 对象加载全部配置
3. 用**自定义对象**接收数据，加上@Component和@ConfigurationProperties注解
## 2 多环境开发
1. maven的环境配置的优先级比spring的环境配置高。
2. 可以使用命令行参数修改配置。

# MybatisPlus
## 1 概念
基于Mybatis框架基础上开发的增强型工具，旨在简化开发，提高效率。
## 2 使用
1. 在自定义mapper接口继承BaseMapper<实体类>，然后该接口就拥有常见的crud方法。
2. 分页查询：
	- 使用getPages方法，并将IPage对象传进去。
	- 写一个分页拦截的配置类。
3. 条件查询：使用LambdaQueryWrapper
4. 条件查询的null值判定，selectList()第一个参数判断是否为空
5. 查询投影：wrapper对象使用select方法，方法里填写要查询的字段
6. 分组查询：groupBy()
7. 字段映射和表名映射的问题及解决方法：
	- 属性名和字段名不匹配：`@TableField(value="字段名")`加在实体类对应的属性上
	- 属性名不在数据库中：`@TableField(exist = false)`
	- 密码等数据不能显示：`@TableField(select = false)`
	- 实体名和表名不匹配：`@TableName("表名")`
8. id生成策略：`@TableId(type=枚举名)`或者做全局配置
9. 逻辑删除：`@TableLogic(value = "xxx", delval = "xxx")`或者做全局配置
10. 乐观锁：@Version 
11. 代码生成器：AutoGenerator
