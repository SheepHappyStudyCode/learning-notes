# MybatisPlus
## 1 快速入门
### 1.1 常见注解
1. `@TableName`：用来指定表名。
2. `@TableId`：用来指定表中的关键字段信息。
	- 该注解还可以定义主键的策略，例如`auto, input, assign_id`
3. `@TableField`：用来指定表中的普通字段信息。
	使用场景：
	- is开头并且是布尔类型的属性
	- 成员变量名与数据库关键字冲突
### 1.2 常见配置
大部分采用默认配置就行，不会时可以去查官网。
## 2 核心功能
### 2.1 条件构造器 
1. QueryWrapper和LambdaQueryWrapper通常用于构建select, delete, update的where条件部分。
2. UpdateWrapper和LambdaUpdateWrapper通常只有在set语句比较特殊时才使用。
3. 尽量使用LambdaWrapper避免硬编码。
### 2.2 自定义sql
 利用mp的wrapper来构建复杂的where条件，然后自己定义SQL语句中剩下的部分。
 ### 2.3 IService的Lambda方法
 1. 支持链式编程，功能非常强大。
 2. 调用userService的lambdaQuery()方法或者lambdaUpdate()方法进行链式编程。
 ## 3 扩展功能
 ### 3.1 代码生成
 ### 3.2 静态工具
 当业务需要同时查询多个表时，可以使用静态工具Db避免循环依赖，Db的方法与IService类似，只是需要多传一个表对应实体的class类。
 ### 3.3 逻辑删除
1.  在二维表加入一个deleted字段来表名该数据有没有被逻辑删除。
2. 不需要修改mp语句，只是要多加一些配置项。
### 3.4 枚举处理器
将Java中的枚举类型转换为数据库的字段

### 3.5 Json处理器
 使用mp自带的handler会报错，目前没有找到解决方案。

### 3.6 分页插件
1. 使用步骤:
	- 在配置类中注册MP的核心插件，同时添加分页插件。
	- 生成Page类，用page进行查询。

## 4 回头学习

1. `selectMaps()`方法：查询只需要返回**部分字段**时，可以将结果映射为一个map，map只存需要字段的信息。

案例：

```java
/**
 * 查询每个部门的平均薪资（返回Map）
 * sql： SELECT departmentId,AVG(salary) AS avg_salary FROM t_employee GROUP BY department_id;
 */
@Test
public void selectByQueryWrapper10ReturnMap(){
    QueryWrapper<Employee> queryWrapper=new QueryWrapper();
    // QueryWrapper<Employee> queryWrapper2=Wrappers.<Employee>query();
    queryWrapper
             .select("department_id","AVG(salary) AS avg_salary")
             .groupBy("department_id");
    List<Map<String, Object>> maps = employeeMapper.selectMaps(queryWrapper);
    System.out.println(maps);
}

```

2. 



   

