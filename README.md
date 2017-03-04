## 高可用架构 PCC 性能挑战赛说明 V0.5

[TOC]

#### 实现类似 facebook 中的 like 功能，需要：

* 可以对一个对象（一条feed、文章、或者url）进行 like 操作，禁止 like 两次，第二次 like 返回错误码
* 有 isLike 接口，返回参数指定的对象有没有被当前用户 like 过
* 需要看到一个对象的 like 计数
* 可以看到一个对象的 like 用户列表（类似 QQ 空间）；
* 上述列表加分项：Like优先显示我的好友列表(social list)。
* 数据量：每天新增的 like 对象数为 1 千万，每秒 like 计数器查询量为 30 万次 / 秒。


#### 数据量

- 用户数量级1亿，好友数量级1~1万，单个对象的Like数量1-100万
- 提供比赛数据集（纯文本格式），需要参赛人员自行导入到自己数据库

#### 技术选型：

- 建议用关系数据库持久化，关系数据库可自行部署在主机上
- 技术栈及编程语言不限制
- 一共3台云主机，不使用云数据库
- 部署方式不限
- 分布式或单体应用不限

#### 评选方法

- 选手需要完成规定的 HTTP API
- 评委使用统一的压测工具进行压测（自动适配 HTTP Keep-Alive）
- 压测单台机器的 API，不能使用 Load balance 聚合多台机的 API 来作为比赛成绩，也就是说主要业务逻辑应在被压测的机器完成，但是缓存、存储或者调用的微服务可以部署在其他服务器
- 选手需要一定程度证明架构的线性扩展能力，比如压测3台服务器，应该可以得到压测1台x3的结果。
- 选取 QPS 最高的前 5 人进入决赛，通过评委对架构打分决出最终成绩

#### 奖励

性能挑战赛设置一二三等奖各一名。


#### 评分标准

- 性能分数 60 + 架构设计 40
- 取所有评委平均分

#### 评分展示环节

1. 有架构设计文档或者方便评委理解的展示材料；
2. RESTful 接口，压测数据；
3. 最好有优缺点分析，说明权衡点；

#### 接口及返回数据格式定义

**server_ip**/pcc?action=like|is_like|count|list&oid=xxx&uid=xxx

返回结果

##### action=like

```{
"oid":1,
"uid":1,
"like_list":[{"1":"nickname"},{"2","Jerry"}]
 }
```

like_list 返回当前对象的赞用户uid列表，只返回前 20 个用户即可

##### action=is_like

```{
"oid":1,
"uid":2,
"is_like":1
}
```

##### action=count

```{
"oid":1,
"count":1024
}
```

##### action=list&cursor=xxx&page_size=xxx&is_friend=1|0

page_size: 返回的列表长度[uint8]
is_friend: 是否仅返回只是好友的uid列表
cursor: 起始位置[uint64]，取上次返回结果的next_cursor

```{
"oid":1,
"like_list":[{"1":"nickname"},{"2","Jerry"}],
"next_cursor":1234
}
```

##### 错误码
业务层面出现错误，实现者也需要返回HTTP 200，在返回结果body里面输出error code
error code 由实现方自行定义

如

```{
"error_code":501,
"error_message":"object already been liked.",
"oid":1,
"uid":1
}
```

#### 测试数据集格式定义

测试数据下载：http://139.198.14.93:8080/data/sample/ （非压测数据）

##### 用户数据格式

uid为uint64，1亿条

```uid,nickname
1,Tom
2,Jerry
```

##### 用户好友数据格式

uid, friend_id为uint64，只存在双向好友关系，1亿个用户*1000,90% 300个以下正态分布，8%长尾分布300-1000,2% 1000-10000

```uid,friend_id
1,2
```

##### 对象Like列表数据格式

oid,uid为uint64，2亿个objects, 每个1-100w

```oid,like_uids
101:[1,2,3,4,5]
```
