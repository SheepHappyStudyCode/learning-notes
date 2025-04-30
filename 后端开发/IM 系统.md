# IM 系统学习

## 参考资料

1. [IM开发技术分享：浅谈IM系统中离线消息、历史消息的最佳实践-腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/1984666)
2. [即时通讯网 - 即时通讯开发者社区!](http://www.52im.net/)

## 什么是即时通信

类似于 QQ、微信这样的应用，发送方发送的消息接收方能立即收到，需要保证消息的**及时性、可靠性、一致性和扩展性**。

## 为什么要学习 IM 技术

即时通信产品通常都有非常好的用户体验，即使不是通信产品，很有应用都需要即时通信能力，例如在线游戏、直播带货等

## IM 技术的难点

1. 如何保证消息的**及时、可靠、一致、安全和扩展**
  - 及时：使用**长连接**和高效的通信协议
  - 可靠：消息确认机制
  - 一致：接收方拉取消息时得判断本地消息和数据库消息的 id，避免消息重复
  - 安全：使用加密算法、TLS/SSL 协议
  - 扩展：IM 系统架构如何支持十万、百万连接
2. 如何兼容多种类型的客户端：IM 产品通常都有多种客户端，例如移动端（安卓/IOS）、小程序、web 端、PC 端，不同客户端能够使用的通信协议和通信条件是不同的，例如移动端 APP 能够直接发起系统调用，实现 socket 连接，不过需要解决弱网环境下的连接问题，还要考虑省电和后台进程等情况、Web 端支持的通信协议少，且只能使用浏览器提供的服务
3. 如何处理在线消息、离线消息、历史消息的**存储和转发**，如何实现消息漫游
4. 如何处理多种类型的消息：文本、图片、视频，如何实现在线语音通话和在线视频通话

## 开源的解决方案

1. J-IM：基于 tio 的 IM 系统
2. OpenIM：前微信技术专家使用 go 技术栈写的 IM 系统
3. RocketMQ：消息队列也能堪称一个简单 IM 系统，能够实现消息的及时转发，也能存储离线消息。

## 如何实现一个 IM 系统

1. 熟悉常用的网络通信协议，熟练使用至少一个网络编程框架如 Netty
2. 熟悉多种数据库，实现不同消息类型的存储。例如，OpenIM 使用 MongoDB 存储离线消息，使用 MySQL 存储历史消息。
3. 消息如何序列化，使用什么样的通信协议
3. 如何设计出一个合理的架构，便于扩展和管理

## 自己动手实现一个 IM 系统

1. 使用 Netty 实现 NIO 服务器实现长连接，需要实现消息确认、心跳机制、认证机制
2. 使用 Netty 实现客户端
3. 需要支持在线消息、离线消息和历史消息
4. 需要支持文本、图片、语音、视频等消息类型
5. 需要支持单聊、群聊等多种会话类型
6. 序列化协议：protobuf

## 如何保证消息的有序性

需要保证一个序列号，序列号在单个会话唯一且连续

## 数据表设计

### 单聊消息表

```sql
CREATE TABLE `private_messages` (
  `id` BIGINT UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '消息唯一ID',
  `msg_id` VARCHAR(64) NOT NULL COMMENT '消息业务ID，用于客户端标识',
  `sender_id` BIGINT UNSIGNED NOT NULL COMMENT '发送者用户ID',
  `receiver_id` BIGINT UNSIGNED NOT NULL COMMENT '接收者用户ID',
  `content_type` TINYINT UNSIGNED NOT NULL DEFAULT 0 COMMENT '消息类型：0文本，1图片，2语音，3视频，4文件，5位置等',
  `content` TEXT COMMENT '消息内容，文本类型直接存储，其他类型存储资源地址',
  `extra` JSON DEFAULT NULL COMMENT '扩展字段，存储额外信息',
  `status` TINYINT UNSIGNED NOT NULL DEFAULT 0 COMMENT '消息状态：0未读，1已读，2撤回，3删除等',
  `send_time` BIGINT UNSIGNED NOT NULL COMMENT '发送时间戳(毫秒)',
  `read_time` BIGINT UNSIGNED DEFAULT NULL COMMENT '已读时间戳(毫秒)',
  `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `updated_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `idx_msg_id` (`msg_id`),
  KEY `idx_sender_receiver_time` (`sender_id`, `receiver_id`, `send_time`),
  KEY `idx_receiver_sender_time` (`receiver_id`, `sender_id`, `send_time`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='单聊消息表';

// 优化查询历史消息的设计
CREATE TABLE `private_messages` (
  `id` BIGINT UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '消息唯一ID',
  `msg_id` VARCHAR(64) NOT NULL COMMENT '消息业务ID，用于客户端标识',
  `small_user_id` BIGINT UNSIGNED NOT NULL COMMENT '较小用户ID',
  `large_user_id` BIGINT UNSIGNED NOT NULL COMMENT '较大用户ID',
  `is_small_sender` TINYINT(1) NOT NULL COMMENT '较小用户ID是否为发送者：1是，0否',
  `content_type` TINYINT UNSIGNED NOT NULL DEFAULT 0 COMMENT '消息类型：0文本，1图片，2语音，3视频，4文件，5位置等',
  `content` TEXT COMMENT '消息内容，文本类型直接存储，其他类型存储资源地址',
  `extra` JSON DEFAULT NULL COMMENT '扩展字段，存储额外信息',
  `status` TINYINT UNSIGNED NOT NULL DEFAULT 0 COMMENT '消息状态：0未读，1已读，2撤回，3删除等',
  `send_time` BIGINT UNSIGNED NOT NULL COMMENT '发送时间戳(毫秒)',
  `read_time` BIGINT UNSIGNED DEFAULT NULL COMMENT '已读时间戳(毫秒)',
  `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `updated_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `idx_msg_id` (`msg_id`),
  KEY `idx_conversation_time` (`small_user_id`, `large_user_id`, `send_time`),
  KEY `idx_small_user_id` (`small_user_id`, `send_time`),
  KEY `idx_large_user_id` (`large_user_id`, `send_time`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='单聊消息表';
```

