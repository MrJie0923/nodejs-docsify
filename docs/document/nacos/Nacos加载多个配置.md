## 加载多个配置

通过之前的学习，我们已经知道Spring应用对Nacos中配置内容的对应关系是通过下面三个参数控制的：

- spring.cloud.nacos.config.prefix
- spring.cloud.nacos.config.file-extension
- spring.cloud.nacos.config.group

默认情况下，会加载`Data ID=${spring.application.name}.properties`，`Group=DEFAULT_GROUP`的配置。

假设现在有这样的一个需求：我们想要对所有应用的Actuator模块以及日志输出做统一的配置管理。所以，我们希望可以将Actuator模块的配置放在独立的配置文件`actuator.properties`文件中，而对于日志输出的配置放在独立的配置文件`log.properties`文件中。通过拆分这两类配置内容，希望可以做到配置的共享加载与统一管理。

这时候，我们只需要做以下两步，就可以实现这个需求：

**第一步**：在Nacos中创建`Data ID=actuator.properties`，`Group=DEFAULT_GROUP`和`Data ID=log.properties`，`Group=DEFAULT_GROUP`的配置内容。

[![img](http://blog.didispace.com/images/pasted-153.png)](http://blog.didispace.com/images/pasted-153.png)

**第二步**：在Spring Cloud应用中通过使用`spring.cloud.nacos.config.ext-config`参数来配置要加载的这两个配置内容，比如：

```
spring.cloud.nacos.config.ext-config[0].data-id=actuator.properties
spring.cloud.nacos.config.ext-config[0].group=DEFAULT_GROUP
spring.cloud.nacos.config.ext-config[0].refresh=true
spring.cloud.nacos.config.ext-config[1].data-id=log.properties
spring.cloud.nacos.config.ext-config[1].group=DEFAULT_GROUP
spring.cloud.nacos.config.ext-config[1].refresh=true
```

可以看到，`spring.cloud.nacos.config.ext-config`配置是一个数组List类型。每个配置中包含三个参数：`data-id`、`group`，`refresh`；前两个不做赘述，与Nacos中创建的配置相互对应，`refresh`参数控制这个配置文件中的内容时候支持自动刷新，默认情况下，只有默认加载的配置才会自动刷新，对于这些扩展的配置加载内容需要配置该设置时候才会实现自动刷新。

## 共享配置

通过上面加载多个配置的实现，实际上我们已经可以实现不同应用共享配置了。但是Nacos中还提供了另外一个便捷的配置方式，比如下面的设置与上面使用的配置内容是等价的：

```
spring.cloud.nacos.config.shared-dataids=actuator.properties,log.properties
spring.cloud.nacos.config.refreshable-dataids=actuator.properties,log.properties
```

- `spring.cloud.nacos.config.shared-dataids`参数用来配置多个共享配置的`Data Id`，多个的时候用用逗号分隔
- `spring.cloud.nacos.config.refreshable-dataids`参数用来定义哪些共享配置的`Data Id`在配置变化时，应用中可以动态刷新，多个`Data Id`之间用逗号隔开。如果没有明确配置，默认情况下所有共享配置都不支持动态刷新

## 配置加载的优先级

当我们加载多个配置的时候，如果存在相同的key时，我们需要深入了解配置加载的优先级关系。

在使用Nacos配置的时候，主要有以下三类配置：

- A: 通过`spring.cloud.nacos.config.shared-dataids`定义的共享配置
- B: 通过`spring.cloud.nacos.config.ext-config[n]`定义的加载配置
- C: 通过内部规则（`spring.cloud.nacos.config.prefix`、`spring.cloud.nacos.config.file-extension`、`spring.cloud.nacos.config.group`这几个参数）拼接出来的配置

要弄清楚这几个配置加载的顺序，我们从日志中也可以很清晰的看到，我们可以做一个简单的实验：

```
spring.cloud.nacos.config.ext-config[0].data-id=actuator.properties
spring.cloud.nacos.config.ext-config[0].group=DEFAULT_GROUP
spring.cloud.nacos.config.ext-config[0].refresh=true

spring.cloud.nacos.config.shared-dataids=log.properties
spring.cloud.nacos.config.refreshable-dataids=log.properties
```

根据上面的配置，应用分别会去加载三类不同的配置文件，启动应用的时候，将会在日志中看到如下输出：

```
2019-02-08 21:23:02.665  INFO 63804 --- [main] o.s.c.a.n.c.NacosPropertySourceBuilder   : Loading nacos data, dataId: 'log.properties', group: 'DEFAULT_GROUP'
2019-02-08 21:23:02.671  INFO 63804 --- [main] o.s.c.a.n.c.NacosPropertySourceBuilder   : Loading nacos data, dataId: 'actuator.properties', group: 'DEFAULT_GROUP'
2019-02-08 21:23:02.677  INFO 63804 --- [main] o.s.c.a.n.c.NacosPropertySourceBuilder   : Loading nacos data, dataId: 'alibaba-nacos-config-client.properties', group: 'DEFAULT_GROUP'
```

后面加载的配置会覆盖之前加载的配置，所以优先级关系是：`A < B < C`