### Nacos 服务安装

-----

#### 1.安装包下载

-----

* 您可以从 [最新稳定版本](https://github.com/alibaba/nacos/releases) 下载 `nacos-server-$version.zip` 包。

#### 2.启动服务器

-----

##### window下启动

解压后进入bin目录：

启动 `startup.cmd`

##### linux下启动

启动命令(standalone代表着单机模式运行，非集群模式):

`sh startup.sh -m standalone`

#### 3.服务注册与发现和配置管理

----

##### 服务注册

`curl -X POST 'http://127.0.0.1:8848/nacos/v1/ns/instance?serviceName=nacos.naming.serviceName&ip=20.18.7.10&port=8080'`

##### 服务发现

`curl -X GET 'http://127.0.0.1:8848/nacos/v1/ns/instance/list?serviceName=nacos.naming.serviceName'`

##### 发布配置

`curl -X POST "http://127.0.0.1:8848/nacos/v1/cs/configs?dataId=nacos.cfg.dataId&group=test&content=HelloWorld"`

##### 获取配置

`curl -X GET "http://127.0.0.1:8848/nacos/v1/cs/configs?dataId=nacos.cfg.dataId&group=test"`