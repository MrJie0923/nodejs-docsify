# 快速搭建Docker环境

## 安装Docker CE

Docker有两个分支版本：Docker CE和Docker EE，即社区版和企业版。本教程基于CentOS 7安装Docker CE。

1. 安装Docker的依赖库。

```
yum install -y yum-utils device-mapper-persistent-data lvm2
```

2. 添加Docker CE的软件源信息。

```
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

3. 安装Docker CE。

```
yum makecache fast
yum -y install docker-ce
```

4. 启动Docker服务。

```
systemctl start docker
```



## 配置阿里云镜像仓库（镜像加速）

Docker的默认官方远程仓库是[hub.docker.com](https://hub.docker.com/)，由于网络原因，下载一个Docker官方镜像可能会需要很长的时间，甚至下载失败。为此，阿里云容器镜像服务ACR提供了官方的镜像站点，从而加速官方镜像的下载。下面介绍如何使用阿里云镜像仓库。

1. 登录容器镜像服务控制台。

   a. 在页面左侧资源栏点击 **一键复制登录url**，打开浏览器隐身窗口（无痕模式）输入已复制的登录链接。

![img](https://img.alicdn.com/tfs/TB1vdlJG1L2gK0jSZFmXXc7iXXa-338-661.jpg)

b. 输入资源提供的 **子用户名称** 和 **子用户密码** ，点击 【登录】 ；然后搜索容器镜像，点击【容器镜像服务】登录控制台。

![img](https://img.alicdn.com/tfs/TB1HwFwXCRLWu4jSZKPXXb6BpXa-1208-456.jpg)

c. 登录成功页面如下。（若弹出开通服务窗口，关闭即可）

![img](https://img.alicdn.com/tfs/TB1FLk6HAY2gK0jSZFgXXc5OFXa-1569-605.png)

2. 单击【镜像中心】 > 【镜像加速器】，可以看到阿里云为您提供了一个专属的镜像加速地址。

![img](https://img.alicdn.com/tfs/TB1LYA8HrH1gK0jSZFwXXc7aXXa-912-722.png)



3. 配置Docker的自定义镜像仓库地址。请将下面命令中的镜像仓库地址`https://kqh8****.mirror.aliyuncs.com`替换为阿里云为您提供的专属镜像加速地址。

```
tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://kqh8****.mirror.aliyuncs.com"]
}
EOF
```

4. 重新加载服务配置文件。

```
systemctl daemon-reload
```

5. 重启Docker服务。

```
systemctl restart docker
```

## 使用Docker安装Nginx服务

1. 查看Docker镜像仓库中Nginx的可用版本。

```
docker search nginx
```

命令输出如下所示：

![img](https://img.alicdn.com/tfs/TB1zTRaHRr0gK0jSZFnXXbRRXXa-1090-592.png)



2. 拉取最新版的Nginx镜像。

```
docker pull nginx:latest
```

命令输出如下所示：

![img](https://img.alicdn.com/tfs/TB1zcA4HpP7gK0jSZFjXXc5aXXa-688-195.png)



3. 查看本地镜像。

```
docker images
```

命令输出如下所示：

![img](https://img.alicdn.com/tfs/TB1igZ2HpY7gK0jSZKzXXaikpXa-731-88.png)



4. 运行容器。

```
docker run --name nginx-test -p 8080:80 -d nginx
```

命令参数说明：

- --name nginx-test：容器名称。
- -p 8080:80： 端口进行映射，将本地8080端口映射到容器内部的80端口。
- -d nginx： 设置容器在后台一直运行。

命令输出如下所示：

![img](https://img.alicdn.com/tfs/TB1CDRaHRr0gK0jSZFnXXbRRXXa-560-69.png)



5. 在浏览器地址栏输入`http://:8080`访问Nginx服务