### 微服务注册-配置中心-Nacos简介

##### 什么是Nacos？

学习地址：http://blog.didispace.com/open-nacos-server-1-0-0/

------

服务（Service）是 Nacos 世界的一等公民。Nacos 支持几乎所有主流类型的“服务”的发现、配置和管理：

[Kubernetes Service](https://kubernetes.io/docs/concepts/services-networking/service/)

[gRPC](https://grpc.io/docs/guides/concepts.html#service-definition) & [Dubbo RPC Service](https://dubbo.incubator.apache.org/)

[Spring Cloud RESTful Service](https://spring.io/understanding/REST)

Nacos关键特性包括：

* 服务发现及管理

  > 动态服务发现对以服务为中心的（例如微服务和云原生）应用架构方式非常关键。Nacos支持DNS-Based和RPC-Based（Dubbo、gRPC）模式的服务发现。Nacos也提供实时健康检查，以防止将请求发往不健康的主机或服务实例。借助Nacos，您可以更容易地为您的服务实现断路器。

* 动态配置服务

  > 动态配置服务让您能够以中心化、外部化和动态化的方式管理所有环境的配置。动态配置消除了配置变更时重新部署应用和服务的需要。配置中心化管理让实现无状态服务更简单，也让按需弹性扩展服务更容易。

* 动态DNS服务

  > 通过支持权重路由，动态DNS服务能让您轻松实现中间层负载均衡、更灵活的路由策略、流量控制以及简单数据中心内网的简单DNS解析服务。动态DNS服务还能让您更容易地实现以DNS协议为基础的服务发现，以消除耦合到厂商私有服务发现API上的风险。

* 服务及其元数据管理

  > Nacos 能让您从微服务平台建设的视角管理数据中心的所有服务及元数据，包括管理服务的描述、生命周期、服务的静态依赖分析、服务的健康状态、服务的流量管理、路由及安全策略、服务的 SLA 以及最首要的 metrics 统计数据。

### Nacos地图

-----

![a](https://nacos.io/img/nacosMap.jpg)

### Nacos 生态图

-----

![](https://cdn.nlark.com/lark/0/2018/png/11189/1533045871534-e64b8031-008c-4dfc-b6e8-12a597a003fb.png)



### 基本架构及概念

----

![](https://cdn.nlark.com/yuque/0/2019/jpeg/338441/1561217892717-1418fb9b-7faa-4324-87b9-f1740329f564.jpeg)

#### 服务

服务是指一个或一组软件功能（例如特定信息的检索或一组操作的执行），其目的是不同的客户端可以为不同的目的重用（例如通过跨进程的网络调用）。Nacos 支持主流的服务生态，如 Kubernetes Service、gRPC|Dubbo RPC Service 或者 Spring Cloud RESTful Service.

#### 服务注册中心（Service register）

服务注册中心，它是服务，其实例及元数据的数据库。服务实例在启动时注册到服务注册表，并在关闭时注销。服务和路由器的客户端查询服务注册表以查找服务的可用实例。服务注册中心可能会调用服务实例的健康检查 API 来验证它是否能够处理请求。

#### 服务元数据(Service MetaData)

服务元数据是指包括服务端点(endpoints)、服务标签、服务版本号、服务实例权重、路由规则、安全策略等描述服务的数据

#### 服务提供方(Service Provider)

提供可以调用服务的应用程序

#### 服务消费方(Service Consumer)

发起对某个服务调用的应用程序

#### 配置(Configuration)

在系统开发过程中通常会将一些需要变更的参数、变量等从代码中分离出来独立管理，以独立的配置文件的形式存在。目的是让静态的系统工件或者交付物（如 WAR，JAR 包等）更好地和实际的物理运行环境进行适配。配置管理一般包含在系统部署的过程中，由系统管理员或者运维人员完成这个步骤。配置变更是调整系统运行时的行为的有效手段之一。

#### 配置管理(Configuration Management)

在数据中心中，系统中所有配置的编辑、存储、分发、变更管理、历史版本管理、变更审计等所有与配置相关的活动统称为配置管理。

#### 名字服务(Naming Service)

提供分布式系统中所有对象(Object)、实体(Entity)的“名字”到关联的元数据之间的映射管理服务，例如 ServiceName -> Endpoints Info, Distributed Lock Name -> Lock Owner/Status Info, DNS Domain Name -> IP List, 服务发现和 DNS 就是名字服务的2大场景。

#### 配置服务 (Configuration Service)

在服务或者应用运行过程中，提供动态配置或者元数据以及配置管理的服务提供者。

## 逻辑架构及其组件介绍

![](https://cdn.nlark.com/yuque/0/2019/png/338441/1561217775318-6e408805-18bb-4242-b4e9-83c5b929b469.png)

* 服务管理：实现服务CRUD、域名CURD,服务健康状态检查，服务权重管理等功能
* 配置管理：实现配置CRUD、版本管理、灰度发布管理....
* 元数据管理：提供元数据CRUD和打标能力
* 插件机制：实现三个模块可分可合能力，实现扩展点SPI机制
* 事件机制：实现异步化事件通知，sdk数据变化异步通知等逻辑
* 日志模块：管理日志分类，日志级别，日志可移植性（尤其避免冲突），日志格式，异常码+帮助文档
* 回调机制：sdk通知数据，通过统一的模式回调用户处理。接口和数据结构需要具备可扩展性
* 寻址模式：解决ip，域名，nameserver、广播等多种寻址模式，需要可扩展
* 推送通道：解决server与存储、server间、server与sdk间推送性能问题
* 容量管理：管理每个租户，分组下的容量，防止存储被写爆，影响服务可用性
* 流量管理：按照租户，分组等多个维度对请求频率，长链接个数，报文大小，请求流控进行控制
* 缓存机制：容灾目录，本地缓存，server缓存机制。容灾目录使用需要工具
* 启动模式：按照单机模式，配置模式，服务模式，dns模式，或者all模式，启动不同的程序+UI
* 一致性协议：解决不同数据，不同一致性要求情况下，不同一致性机制
* 存储模块：解决数据持久化、非持久化存储，解决数据分片问题
* Nameserver：解决namespace到clusterid的路由问题，解决用户环境与nacos物理环境映射问题
* CMDB：解决元数据存储，与三方cmdb系统对接问题，解决应用，人，资源关系
* Metrics：暴露标准metrics数据，方便与三方监控系统打通
* Trace：暴露标准trace，方便与SLA系统打通，日志白平化，推送轨迹等能力，并且可以和计量计费系统打通
* 接入管理：相当于阿里云开通服务，分配身份、容量、权限过程
* 用户管理：解决用户管理，登录，sso等问题
* 权限管理：解决身份识别，访问控制，角色管理等问题
* 审计系统：扩展接口方便与不同公司审计系统打通
* 通知系统：核心数据变更，或者操作，方便通过SMS系统打通，通知到对应人数据变更
* OpenAPI：暴露标准Rest风格HTTP接口，简单易用，方便多语言集成
* Console：易用控制台，做服务管理、配置管理等操作
* SDK：多语言sdk
* Agent：dns-f类似模式，或者与mesh等方案集成
* CLI：命令行对产品进行轻量化管理，像git一样好用

### 领域模型

----

#### 数据模型

Nacos数据模型key由三元组唯一确定，NameSpace默认是空串，公共命名空间，分组默认是

DEFAULT_GROUP

![](https://cdn.nlark.com/yuque/0/2019/jpeg/338441/1561217857314-95ab332c-acfb-40b2-957a-aae26c2b5d71.jpeg)

#### 服务领域模型

![](https://cdn.nlark.com/yuque/0/2019/jpeg/338441/1561217924697-ba504a35-129f-4fc6-b0df-1130b995375a.jpeg)



#### 配置领域模型

![](https://cdn.nlark.com/yuque/0/2019/jpeg/338441/1561217958896-4465757f-f588-4797-9c90-a76e604fabb4.jpeg)

## 类视图

#### Nacos-SDK 类视图

![](https://cdn.nlark.com/yuque/0/2019/jpeg/338441/1561218077514-bfa98d03-88a1-43b9-b014-1491406e3db7.jpeg)

# Nacos功能和需求列表

#### 服务发现

代码地址：https://github.com/alibaba/nacos/tree/develop/naming

|                 描述                 | 主要开发者  | 状态 | 排期  |
| :----------------------------------: | ----------- | ---- | ----- |
|            服务注册与发现            | nkorange    | 稳定 | 0.1.0 |
|  健康检查（服务端探测、客户端心跳）  | xuanyin     | 稳定 | 0.1.0 |
| 路由策略（权重、保护阈值、就近访问） | wangjianwei | 稳定 | 0.1.0 |

#### 配置管理

代码地址：https://github.com/alibaba/nacos/tree/develop/config



|                  描述                  | 主要开发者 | 状态   | 排期  |
| :------------------------------------: | ---------- | ------ | ----- |
| 配置管理（发布、修改、查询、监听配置） | yanlinly   | 稳定   | 0.1.0 |
|                灰度配置                | yanlinly   | 稳定   | 1.1.0 |
|                加密配置                |            | 不支持 |       |

#### 元数据管理

代码地址：https://github.com/alibaba/nacos/tree/develop/cmdb

|      描述      | 主要开发者 | 状态 | 排期  |
| :------------: | ---------- | ---- | ----- |
| 对接第三方CMDB | nkorange   | beta | 0.7.0 |

#### 地址服务器

代码地址：https://github.com/alibaba/nacos/tree/develop/address

|     描述      | 主要开发者 | 状态 | 排期  |
| :-----------: | ---------- | ---- | ----- |
| 支持Nacos寻址 | pbting     | beta | 1.1.0 |

#### Nacos内核