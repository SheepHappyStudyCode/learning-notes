﻿# JavaSE学习过程中的笔记
## 环境安装阶段
1. 安装jdk
步骤：1. 搜索Oracle官网，点击产品，选择较老版本的JDK进行下载(最新版本的JDK的稳定性不好，容易出bug）2.安装时的路径**不能有中文和空格**。
2. 安装IDE
下载IDEA,有社区版和最终版，如果用到JavaEE的开发就下载最终版。

	
## 新建项目阶段
1.  Java项目结构：Java项目分为四个层级：project->module->package->class
2. package命名规范：一般是企业域名倒着写 ，例如黑马网站域名为:www.itheima.com，package命名为com.itheima.hello
3. 导入模块：导入模块要选择iml文件

![导入模块](/imgs/2023-10-31/3FAmZAYB2TDALT1C.png)

## Java基础学习
### 1. Java的数据类型
- **基本数据类型**
![输入图片说明](/imgs/2023-11-16/NO0MIfC3SGMobBVD.png)
- **引用数据类型**
		1. String类型
		2. Scanner
### 2. 数据类型转换
- 表达式的自动类型转换：![输入图片说明](/imgs/2023-11-16/jDDJBWZnuGgKWDGw.png)
即使是两个byte类型相加，结果也为int类型
- 强制类型转换

### 3. Scanner的使用：
1. 导包。
2. 创建Scanner对象。
3. 调用Scanner对象的方法进行输入。
	
### 4. API学习
1. String
String有两种构建方式：1、双引号 2、new构造器  
两者的区别为：前者生成字符串常量，存放在堆内存的常量区，常量区的一个字符串可以给多个对象；后者生成一个新的字符串。

5. ArrayList
集合和泛型都**不支持基本类型**，只支持引用数据类型。
集合存储的是每个对象在堆内存中的地址。

## Java提高学习
### 1. **static**关键字
- 用static修饰的成员是类成员，可以通过类调用。
- static的用途：创建工具类，工具类中的都为类方法，方便调用。  p.s.工具类一般无法创建对象，在构造函数前加private修饰。
- static的应用：静态代码块
格式：static {}
特点：类加载时自动执行，由于类只加载一次，静态代码块也只执行一次。
 作用：完成类的初始化。例如，对类变量赋初值。
- 实例代码块
格式：{}
特点：在创建对象时，执行实例代码块，并且是在构造器之前执行。
作用：完成对对象的初始化
- static的应用：单例设计模式
    写法（饿汉式单例写法）：
	 1. 构造器私有
	 2. 定义一个类变量记住类的一个对象。
	 3. 定义一个类方法，返回该对象。

     作用：确保一个类只有一个对象。
### 2. 继承
- 写法：
```java
public class A extends B{
}
// A继承B
```
- 权限修饰符：public private protected 缺省
![输入图片说明](/imgs/2024-01-06/2gE2BVsjwbxLDxjr.png)
- Java**不支持多继承**，一个类只能继承一个基类。
- Object类：是Java所有类的祖宗。
- 方法重写：子类重写父类的方法
重写小技巧：加Override注解 
` @Override `
注意事项：私有方法、静态方法不能被重写
应用：重写 **toString()** 函数，这样打印输出时可以直接打印对象的内容。（可以用generator自动生成）
- 子类强行访问父类的成员，使用==super==关键字。
- 在子类的构造器中，用this(...)能够调用其他的兄弟构造器。
### 3. 多态
- 好处：在多态形式下，右边的对象是**解耦合**的，便于维护和扩展。
- 强制类型转换：有继承关系的对象能够==强制类型转换==。
```java
//Student Teacher 是 People 的派生类
People p = new Student(); //子类对象给父类对象引用
if( p  instanceof Student) //如果p引用的是Student类
{
	Student s = (Student) p;
}
else
{
	Teacher t = (Teacher) p;
```
- final关键字: 
	1. 修饰类：该类不能被继承。
	2. 修饰方法：该方法不能被重写。
	3. 修饰变量：该变量只能被赋值一次。（该变量即为**常量**）
- 抽象类
写法：用abstract关键字修饰类或方法。
抽象方法应用：模板方法设计模式。
### 4. 接口（ interface )
- 定义接口：
``` Java
 public interface 接口名
{
	成员变量（常量）
	成员方法 （抽象方法）
}
```
- 特点：没有构造器，不能创建对象。
- 实现接口：

``` Java
修饰符 class 实现类 implements 接口名1，接口名2
{
	//必须重写完接口的所有抽象方法，否则也是一个抽象类
}
```
- 接口的**好处**：
	1. 弥补了单继承的不足，一个类可以实现多个接口。
	2. 让程序可以**面向接口编程**，这样程序员可以灵活地切换各种业务的实现。
	
- jdk8后的接口**新特性**：
	1. 默认方法：default关键字修饰，成员方法可以有函数体。
	2. 私有方法
	3. 静态方法
- 接口的多继承：一个接口可以继承多个接口。
注意事项：
	1. 一个接口继承多个接口，若接口存在方法签名冲突，不支持多继承。
	2. 一个类实现多个接口，若多个接口存在方法 签名冲突，不支持多实现。
	3. 一个子类继承了父类，又实现了接口，父类中和接口中有同名的默认方法，实现类会优先使用父类的。
	4. 一个类实现类多个接口，多个接口有同名的默认方法，可以不冲突，重写该默认方法就行。
### 5. 内部类
 1. 成员内部类：用法和普通的成员变量，成员方法类似。
 2. 静态内部类：用法和静态成员方法类似。
 3. 静态内部类（鸡肋）
 4. **匿名内部类**：可以方便地创建一个**子类对象**，常作为**参数传递**给方法。
 ```Java
 //Animal是一个基类对象，需要重写cry方法
 Animal a = new Animal() {
 @Override
 public void cry()
	 {
	 }
 }
 ```
### 6. 枚举
-  写法：
 ```Java
 修饰符 enum 枚举名
 {
	  枚举对象的名称（枚举对象是常量）
	  成员变量、方法......
}
```  
- 特点：
	1. 构造器一定私有。
	2. 枚举会提供一些其他的API接口。
- 枚举的应用：
	1. 单例设计模式。
	2.  做信息标志和分类。
### 7.泛型
1. 泛型类
```Java
public class MyArray<E>
{
}
```
2. 泛型接口
3. 泛型方法
```Java
修饰符 <类型变量1，类型变量2...> 返回值类型 函数名（参数)
{}
```
4. 泛型的注意事项
- 泛型只是工作在编译阶段，一旦编译成class文件，就不存咋泛型了。
- 泛型不支持基本数据类型，只能支持对象类型。
### 8. 常用API
1. Object类
常用方法:
	```Java
	protected Object clone() //返回一个对象的克隆

	boolean equals(Object obj) //判断两个对象的内容是否相等

	String toString() //返回一个对象的字符串表示
	```
2. Objects类（工具类）
	- equals()方法        
	 优点：更安全，空对象也能判断。
3. 包装类：把基本数据类型包装成对象。
 作用：
	- 让集合和泛型正常使用基本类型数据。
	 - 将字符串转换为数字。（使用 **valueOf()** 函数）
4. 字符串相关的API
	1. StringBuilder:  对字符串的操作效率比较高。
	常用方法：
		- ` StringBuilder append(任意数据类型) `
		- ` StringBuilder reverse()`
		- `String toString()`
		
		效率比String高的原因：
		StringBuilder操作的是**可变字符串**。
	5. StringBuffer ：用法和StringBuilder一样，但StringBuffer是线程安全的。
	6. StringJoiner：字符串拼接比较简洁。
5. Math：工具类
6. System：工具类
	常用方法：
	1. `exit()`方法：退出虚拟机。
	2. `currentTimeMillis()`方法：返回系统时间
7. Runtime：代表程序运行的环境。
常用方法：
	1. getRuntime()：通过该类方法拿到单例对象。
	2. availableProcessors()：返回虚拟机能够使用的处理器数。
	3. totalMemory()：返回虚拟机的内存总量。
	4. freeMemory()：返回虚拟机空闲的内存总量。
	5. exec("路径")：启动某个程序，返回该程序的对象，用destroy()来退出程序。
8. **BigDecimal**：解决小数运算失真的问题
方法：将小数转换成字符串再封装成BigDecimal对象。
建议使用`valueOf()`函数
9. JDK8之前传统的时间API（不推荐）
	 1. Date
		 - 无参构造器：拿到系统当前时间的对象。  
		 - getTme()：拿到时间毫秒值。
		 - setTime()：把时间毫秒值转换成日期对象。
	2. SimpleDateFormat
		作用：
		1. 格式化输出时间。（创建对象时生成格式，然后将Date对象格式化）
		2. 解析字符串时间成为时间对象。
	3. Calendar
		方法：
		 1. `public static Calendar getInstance()`：得到日历对象。
		 2. `public int get(int field)`：得到日历中的某个信息。
		 3. `public final Date getTime()`：得到日期对象。
		 4. `public long getTimeMillis()`：得到时间毫秒值。
		 5. `public void set(int filed, int value)`：修改日期中的某个信息。
		 6. `public void add(int field, int amount)`：为某个信息增加/减少指定的值。
10. JDK8之后的时间API（推荐使用）
	- 传统时间API存在的问题
		1. 设计不合理，使用不方便，很多被淘汰了。
		2. 都是可变对象，修改后会丢失最开始的时间信息。
		3. 线程不安全。
		4. 不能精确到纳秒，只能精确到毫秒。
	- 本机时间的API
		1. LocalDate：代表本地日期（年，月，日，星期）
		2. LocalTime：代表本地时间（时，分，秒，纳秒）
		3. **LocalDateTime**：代表本期日期、时间（年，月，日，星期,时，分，秒，纳秒）
	- 时区时间的API
		1. ZoneId：时区对象。
		2. ZoneDateTime：时区时间。
	- Instant：时间戳
			作用：记录代码的执行时间。
			
	- DateTimeFormatter：格式化器，用于时间的格式化、解析。(线程安全）
	- 其他补充
		1. Period：用于计算两个LocalDate对象相差的年数、月数和天数。
		2. Duration：用于计算两个时间对象的天数、小时数、分数、秒数、纳秒数；支持LocalTime、LocalDateTime、Instant...
11. Arrays类
	方法：
	1. 数组的复制、扩容等批量处理操作。
	2. 给对象排序。 （实现Comparable接口 或者 创建Comparator比较器接口的匿名内部对象）
12. **Lambda表达式**
	- 作用：简化匿名内部类的代码写法。
	- 注意：Lambda表达式不能简化所有匿名内部类的写法，只能简化**函数式接口**的匿名内部类。
	- 写法：（参数列表） -> 重写的函数体
	- Lambda表达式的简略写法：
		1. *参数类型*可以简化不写。
		2. 如果只有一个参数，参数类型可以省略，括号也可以省略。
		3. 如果方法体代码只有一行，可以省略大括号不写，同时要省略分号。此时，如果这行代码时return语句，也必须去掉return不写。
13. 方法引用（标识符：`::`）
	- 静态方法引用
		写法：类名::静态方法
	- 实例方法引用
		写法：对象名::实例方法
	- 特定类型的方法引用
		写法：类型::方法
	- 构造器引用（不常用）
		写法：类名::new
### 9.正则表达式
1. 作用：
	- 校验数据格式的合法性。
	- 查找内容。
2. 注意：在正则表达式中，一个\要用两个\\表示。例如，表示数字要用`\\d`。
3. 查找步骤：
	- 定义爬取规则
	- 把正则表达式封装成一个pattern对象 `Pattern pattern = Pattern.compile(regex)`
	- 通过pattern对象去获取查找内容的匹配器对象。`Matcher matcher =  pattern.matcher(data)`
	- 定义一个循环爬取信息。
		```Java
		while(matcher.find()
		{
			String rs = matcher.group();
		}
		```
4. 通过正则表达式替换内容
	使用`repalceAll() `方法
### 10.异常
1. 异常的分类：
	- 运行时异常
	- 编译时异常
2. 异常处理语句
	- try{} catch{} 语句
	- 在方法上使用throws关键字，可以将方法内部的异常抛到上层。
3. 异常处理方式
	- 捕获异常，记录异常并响应合适的信息给用户。
	- 捕获异常，尝试重新修复。
### 11.集合进阶（一）
1. Collection<E>集合体系
	主要分为List<E>  和 Set<E>
	- List系列集合：添加的元素是有序、可重复、有索引。
	- Set系列集合：添加的元素是无序、不重复、无索引。
2. 常用方法：add(E e), clear(), isEmpty(), size(), contains(Object obj), remove(E e), toArray(()
3. Collection的遍历方式
	- 迭代器
	- 增强for循环：for（数据类型 变量名：集合） 
	- 利用Lambda表达式简化书写。（`容器.forEach(System.out :: println`)
4. List集合：ArrayList 和 LinkedList
5. Set集合
	- 分类：
		1. HashSet：无序、不重复、无索引。
		2. LinkedHashSet：**有序**、不重复、无索引。
		3. TreeSet：**排序**、不重复、无索引。
	- 方法：与Collection基本相同。
	- 底层原理
		1. HashSet
			是一种哈希表，基于数组，链表和红黑树实现。
		2. LinkedHashSet
			结构与HashSet类似，只是每个元素都额外多了一个双链表机制记录它前后元素的位置。
		3. TreeSet
			基于**红黑树**实现排序。
			注意：
				- 对于数值类型，按照数值大小进行排序。
				- 对于字符串类型，按照字符编号进行排序。
				- 对于自定义数据类型，无法直接排序，需要指定排序规则。
	- 注意事项：并发修改异常。
		解决方法：使用迭代器自己的删除方法。
6. 可变参数：
	- 作用：可以接收多个数据，放到一个数组中。
	- 注意事项：
		1. 一个形参列表中，只能有一个可变参数。
		2. 可变参数必须放在形参列表的最后面。
7. Collections：用于操作Collection
	方法：addAll(), shuffle(), sort()
### 12.集合进阶（二）
1. Map集合：双列集合
	分类：
		1. HashMap：无序
		2. LinkedHashMap：有序
		3. TreeMap：排序
2. 常用方法：
`
size(), clear(), isEmpty(), get(Object key), remove(Object key), containsKey(Object key), containsValue(Object value), Ste<K> keySet(), Collection<V> values(), putAll()
`
3. Map集合的遍历方式
	- 键找值
	- 键值对（难度较大）
	- Lambda
4. Set是基于Map实现的，底层一样，都是哈希表，红黑树。
5. 补充知识：集合的嵌套
### 13. Stream流
1. 使用步骤
	1. 通过数据源（数组，集合）获得Stream流。
	2. 调用Stream流的各种方法对数据进行处理、计算。
	3. 获取处理的结果，遍历、统计、收集到一个新集合返回。
2. 常用方法
	- 获取Stream流：
		1. stream()：得到Stream流，适用于collection集合
		2. stream(T[] array)：Arrays类的静态方法，获取数组的Stream流
		3. of(T...values)：Stream类的静态方法，接收数据变为Stream流
	- 中间方法：
		1. filter()：过滤
		5. sorted()
		6. limit()：得到前几个数据。
		7. skip()：跳过前几个数据。
		8. map()：将每个数据加工为新的数据。
		9. distinct()： 去重
		10. 从concat()：将两个流合并。
	- 终结方法： 
		1. collect(Collector collector)：把流处理的结果收集到一个指定的集合中。 p.s.流只能收集一次。
		例：`.collect(Collectors.toSet())` `.collect(Collectors.toMap(a -> a.getName(), a -> a.getHeight))`
		2. toArray()
		3. forEach()

## Java高级学习
### 1. IO流（一）
1. File类：可以代表文件或文件夹。
2. 字符编码
	- Ascii编码：占一个字节，存储英文字符和符号
	- GBK编码：英文占一个字节，以0开头，中文占两个字节，以1开头
	- **UTF-8编码**：可变字符集，英文占一个字节，中文占三个字节，最适合开发人员使用。
3. IO流
	- 分类：
		1. 字节输入流：InputStream
		2. 字节输出流：OutputStream
		3. 字符输入流：Reader
		4. 字符输出流：Writer
		p.s. *字节流* 便于操作图片，视频，音频等文件；*字符流* 便于操作文本文件。
	- InputStream
		1. 是抽象类，它的一个实现类是FileInputStream。
		2. read()方法：每次读取一个字符。
			缺点：
			-  读取性能差。
			- 读取汉字会乱码
		3. 流使用完一定要关闭，释放系统资源。
		方法：使用`try-catch-finally`语句（JDK7之前使用） 或者 `try-with-resource`语句(更推荐）
		4. read(byte[] buffer)：每次读取多个字节。
		5. 注意事项：字符输出流写出数据后，必须刷新流`flush()`或者关闭流，否则输出不生效。  
### 2. IO流（二）
1. 缓冲流：
	- 字节缓冲流：自带8kb的缓冲区
	- 字符缓冲流：自带8k字符的缓冲区
2. 转换流：解决不同编码时，字符流读取文本内容乱码的问题。
	- InputStreamReader
	- OutputSreamWriter
3. **打印流**：输出方便且高效
	- PrintStream
	- PrintWriter
	应用：输出语句的重定向。System.out的本质是一个打印流对象，可以通过`System.setOut()`方法进行修改。
4. 数据流：可以将特定类型的数据写入或读出
	- DataOutputStream
	- DataInputStream
5. 序列化流
	- ObjectInputStream
		p.s.序列化必须实现Serializable接口
	- ObjectOutputStream
6. io框架：第三方库
	- Commons-io框架
### 3. 特殊文件
1. Properties属性文件：用于储存键值对
使用Properties集合进行读写
2. XML文件
	1. xml的作用：是一种数据格式，可以存储复杂的数据结构和数据关系。
	2. 应用场景：经常作为系统的配置文件，或者作为一种特殊的数据结构，在网络中进行传输。 
	3. 解析XML文件：使用dom4j框架，自上而下解析。
	4. 输出XML文件：不建议使用dom4j，推荐直接把程序中的数据拼接成XML格式，然后用IO流写出去。
	5. 补充知识：约束XML文件的编写
### 4. 日志技术
1. 作用：
	1. 可以将系统执行的信息，方便地记录到指定位置。
	2. 可以随时以开关的形式控制日志的启停，无需修改源代码。
2. 日志技术的技术体系：
	1. 日志框架：Logback
	2. 日志接口：SLF4J
3. 使用步骤：
	1. 导入jar包
	2. 向src文件夹导入logback的xml文件（核心配置文件）
	3. 创建Logback提供的Logger对象，然后用Logger对象调用其提供的方法就可以记录系统的日志信息。
4. **Logback的日志级别**
	- 常见日志级别（由低到高）：trace, debug, info, warn, error
	- 设置日志级别：`<root level = "XXX">`：XXX可写all,debug,info...
### 5. 多线程
1. 线程（Thread）：是程序的一个执行流程
2. 多线程：从软硬件上实现的多条执行流程的技术
3. 线程的创建方式：
	1. 继承Thread类
		- 优点：编码简单
		- 缺点：线程类已经继承Thread，无法继承其他类，不利于功能的扩展。
		- 注意事项：
			1. 启动线程必须调用start方法，不是调用run方法。
			2. 不要把主线程任务放在启动子线程之前。
	2. 实现Runnable接口
		- 优点：任务类只是实现接口，可以继续继承其他类、其他接口，扩展性强。
		- 缺点：需要多创建一个Runnable对象
		- 技巧：利用Lambda表达式简化书写
	3. 利用Callable接口、FutureTask类来实现
		优点：可以获得线程执行的结果
4. 线程安全
	 - 定义：多个线程，同时操作同一个共享资源时，可能会出现业务安全问题。
	 - 线程同步的方式：
		 1. 同步代码块	`synchronized(同步锁){访问共享资源的核心代码}`
			 - 作用：把访问共享资源的核心代码块上锁。
			 - 原理：每次只允许一个线程加锁后进入，执行完毕后自动解锁， 其他线程才可以进来执行。
			 - 注意事项：对于当前同步执行的线程来说，同步锁必须是同一个对象，否则会出BUG。
			 - 锁对象使用规范
				 1. 实例方法建议使用 **this** 作为锁对象
				 2. 静态方法建议以   **类名.class**   作为锁对象
		 2. 同步方法
		 3. Lock锁
			 注意事项：
				 1. 锁对象最好用final修饰。
				 2. 即使出现异常，unlock()也要能执行。
5. 线程通信
	- 多个线程共同操作共享资源时，线程间通过某种方式相互告知自己的状态，以相互协调，并避免无效的资源争夺。
6. 线程池：一个可以复用线程的技术。
	- 创建线程池的方式（ExecutorService）
		1. ThreadPoolExecutor构造器（7个参数）（更推荐）
			- 执行Runnable任务，execute()
			- 执行Callable任务，submit()
		2. Executors工具类 （不推荐）
			注意事项：大型并发系统环境中使用Executors如果不注意可能会出现系统风险。
	- 核心线程数量配置
		1. 计算密集型的任务，核心线程数量 = CPU核数 + 1
		2. IO密集型的任务，核心线程数量 = CPU核数 + 2
7. 其它细节知识：
 - 进程：正在运行的程序（软件）就是一个独立的进程。线程是属于进程的，一个进程中可以同时运行很多个线程。**进程中的多个线程是并发和并行的。**
 - 并发：CPU同时处理线程的数量有限，为了保证所有线程都能执行，CPU会轮询为系统的每个线程服务，由于CPU切换的速度很快，给我们的感觉是这些线程在同时进行。
 - 并行：在同一个时刻上，同时有多个线程在被CPU调度执行。
 - 线程的六种生命状态：
	 1. New 
	 2. Runnable 
	 3. Terminated 
	 4. Blocked （锁阻塞）
	 5. Waiting （无限等待）
	 6. Timed Waiting（计时等待）
8. 乐观锁：多线程可以同时访问公共资源，要出现线程安全问题时才开始控制。
### 6. 网络编程
1. 基本的通信架构：
	- CS架构（Client客户端／Server服务端）
	- **BS架构**（Browser浏览器／Server服务端）
2. 网络通信的**三要素**
	- IP地址：上网设备的唯一标识

		Java中的**InetAddress**代表IP地址。
	- 端口号：标记正在计算机设备上运行的应用程序，被规定为一个16位的二进制，范围是0~65535。
	
		分类：周知端口（0 ~ 1003）、**注册端口**（1024 ~ 49151）、动态端口。   p.s.同一个设备中端口号不能重复。
	- 协议：
		1. UDP协议：用户数据报协议
			- 特点：无连接、不可靠通信，但通信效率高。
			- 应用：语音通话、视频直播
		2. TCP协议：传输控制协议
			- 特点：面向连接、可靠通信，但通信效率相对不高。
			- 步骤：
				1. 三次握手建立可靠连接。
				2. 四次挥手断开连接。
			- 应用：网页、文件下载，支付。
3. UDP通信的实现`DatagramSocket  DatagramPacket`
4. TCP通信的实现 `Socket ServerSocket`
### 7. Java高级技术
1. 单元测试：针对最小的功能单元（方法），编写测试代码对其进行正确性测试。
	Junit单元测试框架
	- 优点
		1. 可以灵活地编写测试代码。
		2. 自动生成测试报告。
	- 步骤：
		1. 导入Junit框架的jar包。
		2. 定义测试类，编写测试方法（公共、无参、无返回值）
		3. **测试方法必须声明@Test注解**
		4. 开始测试
	- 注解：
		1. Before, After：用于修饰实例方法，在每个测试方法前/后执行。
		2. BeforeClass, AfterClass：在所有测试方法前/后只执行一次。
2. 反射（Reflection）：加载类，并允许以编程的方式解剖类中的各种成分（成员变量、方法、构造器等）
作用：
	1. 基本作用：可以得到一个类的全部成分然后操作。
	2. 可以破坏封装性。
	3. **最重要的用途：适合做Java的框架，基本上，主流的框架都会基于反射设计出一些通用的功能。**
3. 注解：Java的一种特殊标记，让其他程序根据注解信息来决定怎么执行程序。
	- 自定义注解
		```java
		public @interface 注解名 {  
	    属性类型   属性名();  //下面是举例
	    boolean bbb() default  true;  
	    String[] ccc();  
			}
		```
	- 元注解：修饰注解的注解
		1. @Target：声明被修饰的注解只能在哪些位置使用。
		2. @Retention：声明注解的保留周期。
	- 注解的解析：判断类上，方法上、成员变量上是否存在注解，并将注解里的内容给解析出来。
	- 应用场景：做框架。
4. 动态代理设计模式
