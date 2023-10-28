---
title: "简历"
url: "/resume/"

---
# 教育经历
2015年9月 - 2019年6月
非985 211本科 软件工程 
本科  GPA:3.31/5.0 (专业前8%)

# 工作技能
编程语言:  C, C++, Java, Kotlin, Python, Shell, SQL

技术栈:   Akka, Zookeeper, Linux, Protobuf, Redis, MySQL, Git,  Jenkins, Docker, SpringBoot 

英语：大学英语六级（CET-6）

# 工作经历
某上市游戏公司 - 上海 - 游戏服务器开发工程师   2020年12月 – 2023年5月
参与 一款线上SLG游戏 项目服务端研发工作。

某创业板上市软件公司 - 厦门 - Java服务端开发工程师    2019年5月 – 2020年11月
参与云开放平台的开发工作。

# 项目经历
## 一款线上SLG游戏 - 上海 - 游戏服务器开发   2020年12月 – 2023年5月
项目简介：
线上SLG游戏，目前已在Google Play上架且已上线近三年。目前拥有用户两百余万，曾单季流水破亿。

涉及技术：
Akka, Zookeeper, Mysql, Redis, Protobuf, Netty, Kotlin等
具体技术细节：
- 使用Akka Actor模型开发，通过Actor之间发送内部消息的方式实现无锁编程开发；
- 使用Akka Cluster集群实现Actor节点的动态分配和扩展；
- 使用Zookeeper存储服务器环境配置和策划配置表信息，实现配置热更新；
- 使用MySQL存储服务器的数据，数据存在kotlin内存对象中，每隔一段时间将变更部分更新至MySQL中；
- 使用Redis存储经常访问的key-value信息，如登录时查询玩家账号，聊天好友列表等；
- 使用Netty作为整个服务器的网关，解析客户端发来的，压缩过的ProtoBuf请求。

主要负责工作：
参与项目各个模块的开发，其中包括：沙盘大地图模块，战斗模块，玩家内城养成模块，运营活动模块等几乎SLG游戏中所有功能模块均有涉及。
参与策划案的评审，帮助策划设计策划表，并补全需求中遗漏的业务逻辑；
跟踪版本发布上线，及时倾听客服反馈，针对线上紧急Bug及时给出修复方案；
帮助测试人员调试测试服务器环境，解答业务逻辑，及时跟进线上Bug情况，并编写了快速创号等辅助工具供测试人员使用。

主要开发内容：
1. 大型跨服沙盘玩法两个，支持同沙盘100余名玩家激烈战斗；
2. 常驻玩法2个（竞技场、爬塔），运营活动15个
3. 战斗模块中，快速上手战斗模块开发，并协助策划设计和开发战斗内技能；
4. 运营活动15个，且针对运营活动，优化完善了活动框架，简化一部分开发流程；实现了全区全服统一活动数据的框架（之前都是本服的）；
5. 及时跟进修复线上Bug，并协助中台数仓同事拉取资源产销日志，扣除或补发资源给玩家。

项目摘要：
从入职以来就一直负责这款线上项目，我加入团队时，项目已经上线半年（2020年7月项目上线）。 入职后负责的内容从战斗到活动，再到外城沙盘，最后到最复杂的跨服玩法。
项目后期（2022年6月后）主要由我维护服务器，近半年内我一个人负责开发功能和维护线上环境，并及时回复线上客服反馈。

## 云开放平台 - 福建厦门 - 全栈开发 2019年5月 – 2020年11月
项目简介：
为了将集团各个子公司的各种能力（如身份证识别OCR，人脸识别）封装成接口，整合提供给外部使用，开发一个类似于科大讯飞的开放平台。由各个子公司在系统上提供接口服务。各个机构也可以在该平台上申请使用系统已提供的接口服务。
具体的业务流程为：
1. 机构在开放平台上提供服务，由平台审核之后即可被其他机构申请使用；
2. 机构在开放平台上申请服务，由服务提供方和平台都授权使用之后，才可以正式通过平台提供的接口调用服务。
涉及技术：
Vue、SpringBoot、Maven、Redis、MongoDB、Nginx、Mybatis
负责工作：
1. 参与设计整个系统的流程架构、数据库和接口规范；
2. 应用管理、服务管理、服务审核、服务授权、请求日志、机构注册等系统内各个模块的开发；
3. 参与开发外部接口的安全传输功能。使用OAuth 2.0协议标准生成access_token，并使用RSA与AES加密算法及一系列流程保证安全传输。
项目摘要：
1. 入职以来第一次经历从项目启动、需求分析再到开发的整个流程。了解到需求分析也是十分重要的一环，而数据库的设计需要保持一定的灵活性；
2. 学习到开放外部接口所需要的鉴权与加密传输的内容，了解OAuth 2.0的协议规范与加密算法的知识。



