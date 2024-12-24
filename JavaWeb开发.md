# JavaWeb 开发
## 前端web开发
### 1. web标准
三个组成部分：
	1. HTML：负责网页的结构（页面元素和内容）
	2. CSS：负责网页的表现（页面元素的外观、位置等页面样式，如：颜色、大小等。）
	3. JavaScript：负责网页的行为（交互效果）
### 2. HTML, CSS
1. HTML语言特点：
	- 不区分大小写
	- 属性可以是双引号，也可以是单引号
	- 语法松散
2. CSS引入方式
	- 行内样式（不推荐）
	- 内嵌样式：写在style标签中
	- 外联样式：写在一个单独的.css文件中（需要通过link标签在网站中引入）
3. CSS选择器
	- 元素选择器
	- id选择器
	- 类选择器
4. 页面布局：盒子模型
### 3. Javascript, Vue
1. js引入方式
	- 内部脚本（位于script标签之间）
	- 外部脚本（在外部文件中，不包括script标签）
2. js基础语法
	- 输出语句：
		1. window.alert() ：写入警告框
		2. document.write() ：写入html
		3. console.log()：写入控制台
	- 变量：
		1. var关键字用来声明变量。（var声明的变量可以全局使用，可以重复定义）
		2. JavaScript是一门弱类型语言，变量可以存储不同类型的值。
		3. let声明的变量是局部变量，不能重复定义。
		4. const 定义的变量是常量
	 - 类型转换
		 1. 字符串转换为数字，使用parseInt，如果不是数字得到NaN。
		 2. 字符串转换为布尔类型，除了空字符串均为true。
3. js函数
	定义：
	```javascript
	//定义方式一
	function functionName(参数1，参数2...)
	{
		//执行的代码
	}
	//定义方式二
	var add=function(参数1，参数2...)
	{
		//执行的代码
	}
	```
4. js对象
	- 基础对象
		1. Array：长度可变，类型可变。
		2. String：和Java的String类似
		3. JSON
			- 定义：`var 变量名 = '{"key1": value1, "key2": value2}'`
			- 作用：常用于数据传输
			
	- BOM对象：浏览器对象模型
		1. **Window**：浏览器窗口对象
		2. Navigator：浏览器对象
		3. Screen：屏幕对象
		4. History：历史记录对象
		5. **Location**：地址栏对象

	- DOM对象：文件对象模型
		1. Document
		2. Element
		3. Attribute
		4. Text
		5. Comment
5. js事件监听
	- 事件绑定
		1. 在标签属性中绑定事件。
		2. 通过document得到元素，再进行绑定。
	- 常见事件
		1. onclick
		2. onblur：失去焦点
		3. onfocus：得到焦点
		4. onload：加载完成
		5. onsubmit
		6. onkeydown
		7. onmouseover
		8. onmouseout
6. Vue概述：一套**前端框架**，免除DOM操作，简化书写。
7. Vue的常用指令
	- v-bind
	- v-model
	- v-on
	- v-if
	- v-show
	- v-for
8. Vue的生命周期
### 4. Ajax, Axios, ElementUI, Nginx
1. Ajax作用：对网页进行异步更新。
2. Axios：封装了Ajax的框架。
3. Element组件库
	- 常见组件
		1. button
		2. table
		3. pagination（分页）
		4. Dialog（对话框）
		5. form（表单）
4. Vue路由：URL中的hash（#）与组件的关系
5. Nginx：可以用来部署web项目

## 后端web开发
### 1. Maven
1. 概念：一款用于管理和构建Java项目的工具。
2. 作用：
	- 依赖管理：方便快捷地管理项目依赖的资源（jar包），避免版本冲突问题。
	- 统一项目结构
	- 标准化的项目构建流程

### 2. SpringBoot Web 基础篇
1. HTTP协议
	- 概述：超文本传输协议，规定了浏览器和服务器之间数据传输的规则。
	- 特点：
		1. 基于TCP协议：面向连接、安全
		2. 基于请求-响应模型：一次请求对应一次响应。
		3. HTTP协议是无状态的协议：对于事务处理没有记忆能力，每次请求-响应都是独立的。（缺点：多次请求不能共享数据	优点：速度快）
	- 请求协议：
		1. 请求行：请求方式、资源路径、协议
		2. 请求头（key : value）：Host、User-Agent、Accept（浏览器能够接收的资源类型）、Accept-Language、Accept-Encoding、Content-Type、Content-Length
		3. 请求体：POST请求特有，存放请求参数。
	- 响应协议：
		1. 响应行：协议、**状态码**、描述
		2. 响应头：
		3. 响应体：响应数据
	- 协议解析：建议使用Web服务器，例如Tomcat。
2. 请求响应
	- 请求
		1. postman：一款好用的接口测试工具
		2. 请求参数：用相同的参数名接收
			- 简单参数
			- 复杂参数
			- 数组参数
			- 集合参数
			- 日期参数：@DateTimeFormatter(pattern="xxxxxx")
			- Json参数：@RequestBody
			- 路径参数:：@PathVariable
	- 响应
		1. @responseController：用该注解修饰的方法直接返回响应，如果返回的是对象、集合会自动转换为Json对象。
		2. 返回值有规范，一般返回result对象，包含code、msg和data三部分。
3. 分层解耦
	- **三层架构**：
		1. 数据访问（DAO）	  
		2. 逻辑处理（Service）
		3. 接收请求、响应数据（Controller）
	- 控制反转（IOC）
		1. @Component：将当前类交给IOC容器管理，成为IOC容器中的bean。
		2. Component的衍生注解：Service、Repository和Controller。
		
	- 依赖注入（DI）
		1. @Autowired：运行时，IOC容器会提供该类型的bean对象，并赋值给该变量。
		2. @Primary
		3. @Autowired + @Qualified（"bean的名称"）
		4. @Resource（name = “bean的名称"）

### 3. MySQL
1. 数据库设计
	- SQL：一门操作**关系型数据库**的编程语言，定义操作所有关系型数据库的**统一标准**。
	- DDL：数据库设计语言，一般被图形化操作取代。
	- 多表设计
		1. 一对多：外键约束，推荐使用逻辑外键。
		2. 一对一：常用于单表拆分，将一张表的基础信息放在一张表，其他字段放在另一张表，提高操作效率。
		3. 多对多：建立第三张中间表，中间表至少包含两个外键，分别关联两方主键。
2. 数据库操作
	- **DQL：数据库查询语言**
		1. 关键字：
			- select 字段列表 from 表名
			- where 查询条件
			- group by 字段名 having 分组后的查询条件
			- order by 字段名（排序展示，降序加上DESC）
			- limit 起始索引，每页展示记录个数（分页查询）
		2. 聚合函数：
			- count(*)：统计非空值的个数。
			- min()
			- max()
			- sum()
		3. 多表查询：内连接、外连接
		4. 子查询：SQL语句中嵌套select语句。
			- 标量子查询：查询结果是单个值
			- 列子查询：查询结果是一列
			- 行子查询：查询结果是一行
			- 表子查询：查询结果是多行多列
	- DML：数据库操纵语言：
		1. 事务：一组操作的集合，操作要么同时成功，要么同时失败。
		2. 开启事务：start transaction / begin
		3. 提交事务：commit
		4. 回滚事务：rollback
3. 数据库优化
	- 索引：帮助数据库高效获取数据的数据结构。
		1. 数据库默认使用B+树索引结构
		2. 主键、唯一约束会自动创建索引。
### 4. SpringBoot Mybatis
1. Mybatis简介：是一款优秀的**持久层**框架，用于简化JDBC的开发。
2. 数据库连接池：是个容器，负责分配、管理数据库连接Connection。
3. lombok：是一个实用的Java类库，能通过注解的形式自动生成构造器、getter、setter、toString等方法。
例如：@Data、@NoArgsConstructor、@AllArgsConstructor
4. Java程序配置sql语句：
	- 使用注解：适合简单的sql语句
	- 使用xml映射文件：适合复杂的sql语句
5. XML映射文件规范：
	- XML映射文件与Mapper接口同包同名
	- XML映射文件的namespace属性与Mapper接口的全限定名一致。
	- XML映射文件的sql语句的id与接口的方法名一致，并保持返回值类型一致。
6. 动态SQL：随着用户的输入或外部条件的变化而变化的SQL语句。
	- if：用于判断条件是否成立。使用test属性进行条件判断，如果条件为真则拼接SQL。
	- foreach：用于批量操作
	- sql include：
### 5. SpringBoot Web 开发篇
1. 开发规范-Restful：表述性状态转换，是一种软件架构风格。
Rest风格：URL定位资源、HTTP动词描述操作。
2. 开发流程：
	- 查看页面原型，明确需求
	- 阅读接口文档
	- 思路分析
	- 接口开发
	- 接口测试
	- 前后端联调
3. 文件上传的存储方式
	- 本地存储
	- 第三方云服务（阿里云OSS）
4. 第三方服务的通用思路、
	- 准备工作
	- 按照官方SDK编写入门程序
	- 集成使用
5. 三种配置文件
	- xml配置文件：最臃肿
	- properties配置文件：较轻量，但结构不清晰
	- **yml配置文件：简洁，以数据为中心，最推荐**
6. 属性注入
	- @Value
	- @ConfigurationProperties
7. 登录校验
	- 会话跟踪方案：
		1. Cookie
		2. Session
		3. **令牌技术**
	- JWT令牌技术：登录成功后发放令牌，只有带着令牌才能访问服务器资源。
	
	- 过滤器Filter：作用将对资源的请求拦截下来，实现一些特殊的功能。例如，登录校验，统一编码处理，敏感字符处理等。
	- 拦截器Interceptor：在spring内部环境进行拦截。
8. 异常处理
全局异常处理器：@RestControllerAdvice、@ExceptionHandler。
1. 事务管理
	- @Transactional：在一个方法上加上该注解，该方法里的所有操作组成一个事务。
	- rollbackFor：默认只有出现RuntimeException才会进行事务回滚，如果出现任何异常都要回滚事务，在注解后面加上`rollbackFor = Exception.class`
	- 事务属性propagation
2. AOP基础：@Aspect
3. AOP核心概念：
	- 连接点（JointPoint）
	- 切入点（PointCut）
	- 通知（Advice）
	- 切面（Aspect）
	- 目标对象（Target）
4. AOP进阶
	- 通知类型
		1. @Around：环绕通知，此注解标注的通知方法在目标方法前后执行。
		2. @Before
		3. @After
		4. @AftereReturning
		5. @AfterThrowing
	- 通知顺序：
		1. 默认情况：和类名的字母排序有关，排序较小通知方法的在before先执行，在after后执行。
		2. 使用Order注解来自定义执行顺序，数字越小越优先执行。
	- 切入点表达式：决定项目中哪些方法需要插入通知
		1. execution表达式：`execution(访问修饰符？ 返回值类型 包名.类名.?方法名（方法参数） throws 异常?)`
	 通配符：*可通配任意一个字符，..可通配任意多个字符
		2. @annotation表达式：在需要通知的地方加上自定义注解，然后用`@annotation(注解全类名)`进行匹配
	- 连接点JointPoint：通过连接点可以得到方法执行的相关信息。
### 6. SpringBoot Web 进阶篇
1. 配置优先级：命令行参数 > Java系统属性 > properties > yml > yaml
2. Bean管理
	- 获取bean：先拿到IOC容器，再用getBean()方法拿到bean对象。
	- bean的作用域：singleton（默认是单例模式）、prototype(每使用一次Bean对象就创建一次）
3. 第三方Bean的配置：在配置类写一个方法，返回第三方的Bean对象，并用@Bean注解。Bean对象的对象名默认是方法名。
4. spring的自动配置：spring的启动类使用@EnableAutoConfiguration注解，该注解封装了@Import注解，@Import的括号里封装了ImportSelector的实现类，该实现类会从外部配置文件中读取各个依赖的自动配置类的全类名并返回。
5. 自定义starter：通常由起步模块和自动配置模块组成。起步模块负责引入各种依赖，包括自动配置模块，自动配置模块内部要定义配置类，并在配置信息写在`META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`文件中。
7. Maven高级技术：
	- 分模块设计与开发
	- 继承与聚合
		1. 父工程：只负责依赖管理，一般只有pom文件，文件内包含子工程需要用到的依赖。
		2. 版本锁定：父工程中使用dependencyManagement标签声明子工程依赖的版本号，并且在properties标签内定义版本号。
		3. 聚合：在parent工程中的modules标签指定聚合模块。
	- 私服
		1. 依赖查找顺序：本地仓库 --> 私服 -->中央仓库
		2. 使用私服的步骤：
			1. 设置私服的访问用户名/密码（在自己maven安装目录下的conf/settings.xml中的servers中配置）
			2. 设置私服依赖下载的仓库组地址（在自己maven安装目录下的conf/settings.xml中的mirrors、profiles中配置）
			3. IDEA的maven工程的pom文件中配置上传（发布）地址(直接在tlias-parent中配置发布地址)

