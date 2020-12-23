# Namespace, endpoint 最佳实践

如何在生产环境中正确使用namespace、endpoint

#### namespace

---

关于 namespace ，以下主要从 **namespace 的设计背景** 和 **namespace 的最佳实践** 两个方面来讨论。

#### namespace设计背景

namespace 的设计是 nacos 基于此做多环境以及多租户数据(**配置和服务**)隔离的。即：

* 从一个租户(用户)的角度来看，如果有多套不同的环境，那么这个时候可以根据指定的环境来创建不同的 namespce，以此来实现多环境的隔离。例如，你可能有日常，预发和生产三个不同的环境，那么使用一套 nacos 集群可以分别建以下三个不同的 namespace。如下图所示：

![](http://edas.oss-cn-hangzhou.aliyuncs.com/deshao/pictures/nacos_ingle_tenant_namespace.jpg)

* 从多个租户(用户)的角度来看，每个租户(用户)可能会有自己的 namespace,每个租户(用户)的配置数据以及注册的服务数据都会归属到自己的 namespace 下，以此来实现多租户间的数据隔离。例如超级管理员分配了三个租户，分别为张三、李四和王五。分配好了之后，各租户用自己的账户名和密码登录后，创建自己的命名空间。如下图所示：

![](http://edas.oss-cn-hangzhou.aliyuncs.com/deshao/pictures/nacos_multi_tenant_namespace.jpg)

#### namespace 的最佳实践

关于 namespace 的最佳实践，这部分主要包含有两个 Action：

- **如何来获取 namespace 的值**

  无论您是基于 Spring Cloud 或者 Dubbo 来使用 nacos，都会涉及到 namespace 的参数输入，那么这个时候 namespace 的值从哪里可以获取呢？

  1. 如果您在使用过程中没有感知到这个参数的输入，那么 nacos 统一会使用一个默认的 namespace 作为输入，nacos naming 会使用 **public** 作为默认的参数来初始化，nacos config 会使用一个**空字符串**作为默认的参数来初始化。

  2. 如果您需要自定义自己的 namespace，那么这个值该怎么来产生？

     1. 可以在 nacos 的控制台左边功能侧看到有一个 **命名空间** 的功能，点击就可以看到 **新建命名空间** 的按钮，那么这个时候就可以创建自己的命名空间了。创建成功之后，会生成一个**命名空间ID**，主要是用来避免**命名空间名称**有可能会出现重名的情况。因此当您在应用中需要配置指定的 namespace 时，**填入的是命名空间ID**。重要的事情说三遍：
        1. 当您在应用中需要配置指定的 namespace 时，**填入的是命名空间 ID**
        2. 当您在应用中需要配置指定的 namespace 时，**填入的是命名空间 ID**
        3. 当您在应用中需要配置指定的 namespace 时，**填入的是命名空间 ID**

     说明: namesace 为 **public** 是 nacos 的一个保留控件，如果您需要创建自己的 namespace，最好不要和 **public** 重名，以一个实际业务场景有具体语义的名字来命名，以免带来字面上不容易区分自己是哪一个 namespace。

- **namespace 参数初始化方式**

  nacos client 对 namespace 的初始化流程如下图所示:

![](http://edas.oss-cn-hangzhou.aliyuncs.com/deshao/nacos/nacos_namespace.jpg)

nacos client 对 namespace 的初始化，主要包含两部分：

* 用户态通过 nacos client 构造实例时通过 properties 参数传入的 namespace。

* 在云环境下(**阿里云下的 EDAS**)的 namespace 参数解析。

  可通过 **-Dnacos.use.cloud.namespace.parsing=true/false** 来控制是否需要在云环境自动解析 namespace 参数，默认为 **true**，是会自动解析，其目的就是方便用户上云时可以以零成本的方式平滑上云。如果用户在云上需要用自建的 nacos 下的 namespace，那这个时候只需将 **-Dnacos.use.cloud.namespace.parsing=false** 即可。

#### endpoint

---

关于 endpoint ，也主要从 **endpoint 的设计背景** 和 **endpoint 的参数初始化** 两个方面来讨论。

#### endpoint 的设计背景

当 nacos server 集群需要扩缩容时，客户端需要有一种能力能够及时感知到集群发生变化。及时感知到集群的变化是通过 endpoint 来实现的。也即客户端会定时的向 endpoint 发送请求来更新客户端内存中的集群列表。