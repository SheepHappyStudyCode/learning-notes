# Spring 学习笔记

## 1 控制反转 IOC

### 1.1 含义和作用

控制反转。将对象的创建和注入交给spring容器，让对象的创建和使用解耦。

### 1.2 基于xml文件的IOC

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">
    
    <!-- 通过bean标签注册 -->
    <bean id="userBean" class="com.wxw.pojo.UserBean" />

</beans>

```

较为繁琐，不太推荐。

### 1.3 基于注解的IOC

#### 1.3.1 配置类

用一个java类替代xml文件，加上`@configuration`注解，生成Bean对象的方法加上`@Bean`注解。

```java
package com.javacode2018.lesson001.demo20;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class ConfigBean {

    //bean名称为方法默认值：user1
    @Bean
    public User user1() {
        return new User();
    }

    //bean名称通过value指定了：user2Bean
    @Bean("user2Bean")
    public User user2() {
        return new User();
    }

    //bean名称为：user3Bean，2个别名：[user3BeanAlias1,user3BeanAlias2]
    @Bean({"user3Bean", "user3BeanAlias1", "user3BeanAlias2"})
    public User user3() {
        return new User();
    }

}
```

#### 1.3.2 扫包 + 注解

将Bean对象的完全创建交给spring。

实现方法：需要托管的对象加上`@Component`注解，然后扫描配置类所在的包。



## 2 面向切面编程 AOP

### 2.1 含义和作用

1. 是一种抽象化的面向对象编程。

2. 作用：**将业务代码和非业务代码解耦**。例如，打印日志的代码不用跟业务代码放在一起，而是写在aop里。

### 2.2 使用案例

#### 2.2.1 权限校验：

1. 定义切面类：

```java
/**
 * 权限校验 AOP
 *
 * @author yupi
 */
@Aspect
@Component
public class AuthInterceptor {

    @Resource
    private UserService userService;

    /**
     * 执行拦截
     *
     * @param joinPoint
     * @param authCheck
     * @return
     */
    @Around("@annotation(authCheck)")
    public Object doInterceptor(ProceedingJoinPoint joinPoint, AuthCheck authCheck) throws Throwable {
        List<String> anyRole = Arrays.stream(authCheck.anyRole()).filter(StringUtils::isNotBlank).collect(Collectors.toList());
        String mustRole = authCheck.mustRole();
        RequestAttributes requestAttributes = RequestContextHolder.currentRequestAttributes();
        HttpServletRequest request = ((ServletRequestAttributes) requestAttributes).getRequest();
        // 当前登录用户
        User user = userService.getLoginUser(request);
        // 拥有任意权限即通过
        if (CollectionUtils.isNotEmpty(anyRole)) {
            String userRole = user.getUserRole();
            if (!anyRole.contains(userRole)) {
                throw new BusinessException(ErrorCode.NO_AUTH_ERROR);
            }
        }
        // 必须有所有权限才通过
        if (StringUtils.isNotBlank(mustRole)) {
            String userRole = user.getUserRole();
            if (!mustRole.equals(userRole)) {
                throw new BusinessException(ErrorCode.NO_AUTH_ERROR);
            }
        }
        // 通过权限校验，放行
        return joinPoint.proceed();
    }
}
```

2. 切面表达式的相应注解：

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface AuthCheck {

    /**
     * 有任何一个角色
     *
     * @return
     */
    String[] anyRole() default "";

    /**
     * 必须有某个角色
     *
     * @return
     */
    String mustRole() default "";

}
```

#### 2.2.1 日志记录：

```java
@Aspect
@Component
@Slf4j
public class LogInterceptor {

    /**
     * 执行拦截
     */
    @Around("execution(* com.yupi.yuapi.controller.*.*(..))")
    public Object doInterceptor(ProceedingJoinPoint point) throws Throwable {
        // 计时
        StopWatch stopWatch = new StopWatch();
        stopWatch.start();
        // 获取请求路径
        RequestAttributes requestAttributes = RequestContextHolder.currentRequestAttributes();
        HttpServletRequest httpServletRequest = ((ServletRequestAttributes) requestAttributes).getRequest();
        // 生成请求唯一 id
        String requestId = UUID.randomUUID().toString();
        String url = httpServletRequest.getRequestURI();
        // 获取请求参数
        Object[] args = point.getArgs();
        String reqParam = "[" + StringUtils.join(args, ", ") + "]";
        // 输出请求日志
        log.info("request start，id: {}, path: {}, ip: {}, params: {}", requestId, url,
                httpServletRequest.getRemoteHost(), reqParam);
        // 执行原方法
        Object result = point.proceed();
        // 输出响应日志
        stopWatch.stop();
        long totalTimeMillis = stopWatch.getTotalTimeMillis();
        log.info("request end, id: {}, cost: {}ms", requestId, totalTimeMillis);
        return result;
    }
}
```

## Bean 的生命周期
1. 大概可以分为五个阶段：创建实例 --> 属性注入 --> 初始化 --> 使用 --> 销毁

![image-20241219231748602](https://gitee.com/SheeepHappy/blog-pic/raw/master/20250409105159449.png)

2. 在生成 Bean 对象的时候，spring 暴露出了很多扩展点，所以生命周期可以扩展出更多的阶段。

![image-20241219231930392](https://gitee.com/SheeepHappy/blog-pic/raw/master/20250409105159451.png)


## Spring 如何解决循环依赖
关键在于提前暴露未创建完毕的Bean

### Bean 对象的三级缓存

1. 一级缓存（singletonObjects)：存储已创建完毕的单例 Bean
2. 二级缓存（earlySingletonObjects）：存储所有只完成实例化，未进行属性注入和初始化的 Bean 对象
3. 三级缓存（singletonFactories）：存储能建立这个 Bean 的一个工厂，通过工厂能获取到这个 Bean，延迟 Bean 的生成，工厂生成的 Bean 会塞入二级缓存。

### 解决过程（A 和 B 互相依赖）

#### 1. 创建 `A` 时：

- Spring 实例化 `A`，但还没有完成依赖注入（`B` 还没有注入）。
- Spring 发现 `A` 依赖 `B`，于是尝试创建 `B`。

#### 2. 创建 `B` 时：

- Spring 实例化 `B`，但还没有完成依赖注入（`A` 还没有注入）。
- Spring 发现 `B` 依赖 `A`，于是尝试获取 `A`。

#### 3. 获取 `A` 时：

- Spring 发现 `A` 还在初始化过程中，并且已经有一个占位符（部分初始化的 `A`）存在于 `earlySingletonObjects` 中。因此，Spring 会将这个占位符注入到 `B` 中。

#### 4. 完成初始化：

- Spring 完成 `B` 的初始化，并将其放入 `singletonObjects` 中。
- 之后，Spring 会再次完成 `A` 的依赖注入，将 `B` 注入到 `A` 中，并将 `A` 放入 `singletonObjects` 中。

#### 5. 最终 Bean 状态：

- `A` 和 `B` 都在容器中，且依赖关系已经正确注入。



## 开发者如何解决循环依赖

### 1. 尽量使用 setter 注入或者字段注入

构造器注入可能会使两个相互依赖的 Bean 对象无法实例化



### 2. 通过 `@Lazy` 注解解决

`@Lazy` 注解可以**延迟注入某个依赖**，直到需要的时候才会进行注入。这样可以打破循环依赖的问题，特别是在构造器注入时。



### 3. 分解成更小的模块

有时候循环依赖是因为设计不当导致的，可以通过引入中间层、接口、抽象类来拆解依赖，减少直接的相互依赖。



### 4. **使用 `@PostConstruct` 或 `@Bean` 配置**

对于某些特殊情况，可以利用 `@PostConstruct` 来在 Bean 初始化后进行依赖的设置，或者使用 `@Bean` 注解手动配置依赖关系。

**示例：**

```
java复制代码@Service
public class A {
    private B b;

    @PostConstruct
    public void init() {
        this.b = new B(this);
    }
}

@Service
public class B {
    private A a;

    public B(A a) {
        this.a = a;
    }
}
```



## Spring 如何实现动态代理

Spring 的动态代理主要有两种实现方式：**JDK 动态代理** 和 **CGLIB 动态代理**。这两种方式的选择取决于目标对象的类型以及代理的需求。

### 1. JDK 动态代理

#### 代码示例：

```java
public interface UserService {
    void addUser();
    void deleteUser();
}

public class UserServiceImpl implements UserService {
    @Override
    public void addUser() {
        System.out.println("Adding user");
    }

    @Override
    public void deleteUser() {
        System.out.println("Deleting user");
    }
}

public class DynamicProxy implements InvocationHandler {
    private Object target;

    public DynamicProxy(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("Before method " + method.getName());
        Object result = method.invoke(target, args);
        System.out.println("After method " + method.getName());
        return result;
    }
}

public class Test {
    public static void main(String[] args) {
        UserService userService = new UserServiceImpl();
        DynamicProxy dynamicProxy = new DynamicProxy(userService);

        UserService proxy = (UserService) Proxy.newProxyInstance(
            userService.getClass().getClassLoader(),
            userService.getClass().getInterfaces(),
            dynamicProxy
        );

        proxy.addUser();
        proxy.deleteUser();
    }
}

```

**优点：**

1. 代理对象必须实现接口，符合 Java 的面向接口编程思想。

2. 可以很方便地为接口方法添加增强逻辑。

**缺点：**

被代理类必须实现接口，不能代理普通类

### 2. **CGLIB 动态代理**

CGLIB（Code Generation Library）是一个**基于继承**的动态代理库，Spring 在没有接口的情况下，会使用 CGLIB 来生成代理类。CGLIB 动态代理是通过生成目标类的子类来实现代理的，代理类会覆盖父类的方法，来实现方法增强。

#### 代码示例

```java
public class UserService {
    public void addUser() {
        System.out.println("Adding user");
    }

    public void deleteUser() {
        System.out.println("Deleting user");
    }
}

public class MethodInterceptorImpl implements MethodInterceptor {
    @Override
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
        System.out.println("Before method " + method.getName());
        Object result = proxy.invokeSuper(obj, args);  // 调用父类方法
        System.out.println("After method " + method.getName());
        return result;
    }
}

public class Test {
    public static void main(String[] args) {
        UserService userService = new UserService();

        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(UserService.class);
        enhancer.setCallback(new MethodInterceptorImpl());

        UserService proxy = (UserService) enhancer.create();
        proxy.addUser();
        proxy.deleteUser();
    }
}

```

**优点：**

- 不要求目标类实现接口，可以直接对普通类进行代理。
- 灵活性较高，能够代理任何类。

**缺点**：

- 由于使用继承的方式，因此无法代理 `final` 类和 `final` 方法。
- 生成的代理类会比 JDK 动态代理更大，性能上也可能稍微逊色。



## BeanFactory、FactoryBean 和 ObjectFactory

1. BeanFactory：Spring 用于**创建 Bean 对象的底层容器接口**，负责创建 Bean 对象，可以延迟 Bean 对象的生成，解决一些循环依赖。
2. FactoryBean ：Spring 给开发者提供的一个接口，用于**自定义复杂的 Bean 对象**。
3. ObjectFactory：Spring 框架中的一个更为通用的接口（函数式接口），只提供一种创建对象的方式，主要用于延迟 Bean 对象的生成。

## Spring 使用了哪些设计模式

**工厂模式**： BeanFactory和ApplicationContext作为Spring的核心容器，负责创建和管理Bean对象。

**单例模式**：Spring默认的Bean作用域就是单例，确保一个类只有一个实例。

**代理模式**：Spring AOP的实现基础，通过JDK动态代理或CGLIB实现横切关注点的分离。

**模板方法**模式：如JdbcTemplate、HibernateTemplate等，定义操作的骨架，具体实现由子类完成。

观察者模式：Spring的事件机制，如ApplicationEvent和ApplicationListener。

适配器模式：Spring MVC中的HandlerAdapter，允许多种控制器形式。

装饰器模式：在Spring中的各种Wrapper类。

**策略模式**：如Resource接口的不同实现策略（ClassPathResource等）。
