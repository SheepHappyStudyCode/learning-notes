# MongoDB
## 1 MongoDB相关概念
### 1.1 业务应用场景
 1. 数据操作“三高”需求：
	- **高并发读写**
	- 海量数据的高效率存储和访问的需求
	- 对数据库的**高可扩展性**和**高可用性**的需求
2. 什么时候选择MongoDB
	- 应用**不需要事务**及复杂join支持
	- 新应用，**需求会变**，数据模型无法确定，想快速迭代开发
	- 应用需要2000以上的读写QPS
	- 应用需要大量的**地理位置查询**、文本查询
### 1.2 MongoDB简介
1. 简介：是一个开源、高性能、无模式的**文档型数据库**。
## 2 基本常用命令
### 2.1 数据库操作
1. 使用或创建数据库：`use 数据库`
	- 当数据库中有数据时才会持久化
2. 展示数据库：`show dbs`
3. 删除数据库：`db.dropDatabase()`
### 2.2 集合操作
1. 集合的显示创建（了解）：`db.createCollection('集合的名字')`
2. 删除集合：`db.集合名.drop()`
### 2.3 文档基本CRUD
1. 插入文档：`db.集合名.insert(json格式信息)`
2. 插入多条文档：`db.集合名.insertMany(json数组)`
3. 查询：`db.集合名.find()`
	- 可带json参数来指定查询条件
	- 只需要一个结果用`findOne()`
	- 投影查询：`find(query, projection)`
4. 修改文档：`db.集合名.update(query, update, option)`
	- 局部修改：`{$set:修改的键值对}`
	- 批量修改：option里加上`{multi:true}`
	- 增长：`inc`
5. 删除文档：`db.集合名.remove(条件)`
### 2.4 分页查询
1. 统计查询：`...count()`
	- 括号里可加条件
2. 分页查询：`...find().skip(num).limit(num)`
3. 排序查询：`...find().sort({键：0 或者 1})`
	- 1是升序，2是降序
### 2.5 其他查询
1. 正则的复杂查询
	- 正则表达式是js的写法
2. 比较查询：`find(field : {$gt:value})`
3. 包含查询：`find(field:{$in:[value数组]})`
4. 条件连接查询：
	- `$and:[{},{},{}]`
	- `$or:[{},{},{}]`
## 3 索引
### 3.1 索引类型
1. 单字段索引
2. 复合索引
	- 例：`{userid:1, score:-1}`
3. 其他索引
	- 地理空间索引
	- 文本索引
	- 哈希索引
### 3.2 索引管理
1. 查看索引：`db.collection.getIndexes()`
2. 建立索引：`...createIndex(key, option)`
3. 移除索引：`...dropIndex(index)`
4. 删除所有索引（除_id）：`...dropIndexes()`
5. 分析查询效率：`...find().explain()`

# MongoDB集群和安全
## 1 副本集-Replica Sets
### 1.1 
