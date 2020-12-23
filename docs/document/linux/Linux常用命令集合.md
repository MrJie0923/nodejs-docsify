### Linux操作命令解释

----

![img](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicadHiavYnpTGmqVecvTr8c6MEZ5JwMADzAZBJDr96jQYKHT49ZutA737ZKNCuvxKkVwHhL3ibJfJkFicw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![img](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicadHiavYnpTGmqVecvTr8c6MEr7M5ibOnH7SAzLbprMicCsz1fX8xESoYzVLow8456J4pwoSRfueHl25g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![img](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicadHiavYnpTGmqVecvTr8c6MEdOibI4fAffOmeZ7bRbC999Ybe16OXVia4K5p8qxCXbNFBMcdrYd5qA8A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

#### 基本命令

| 执行命令              | 含义                                  |
| :-------------------- | :------------------------------------ |
| cd ~                  | 切换到登录用户的主目录即/home/用户名  |
| cd /                  | 进入根目录                            |
| cd /home/lj           | 将/home/LJ作为当前的目录              |
| cd ..                 | 返回到上一层目录                      |
| cd -                  | 回到上次所在的目录                    |
| cd ../../             | 去上上层目录                          |
| ls                    | 查看当前目录                          |
| ls -la                | 查看当前目录的文件信息 包含了隐藏文件 |
| pwd                   | 查看当前目录的绝对路径                |
| cp /目录/1.txt /目录/ | 复制                                  |
| rm                    | 删除                                  |
| q！                   | 不保存文件退出                        |
| wq！                  | 保存退出                              |
| hostname              | 查看当前主机名                        |
| ifconfig              | 查看网卡相关信息                      |
| firewall-cmd --state  | centos7查看卡其关闭防火墙状态         |



#### touch命令参数

-a  或--time=atime或--time=access或--time=use 只更改存取时间。

-c  或--no-create 不建立任何文档。

-d 使用指定的日期时间，而非现在的时间。

-f 此参数将忽略不予处理，仅负责解决BSD版本touch指令的兼容性问题。

-m  或--time=mtime或--time=modify 只更改变动时间。

-r 把指定文档或目录的日期时间，统统设成和参考文档或目录的日期时间相同。

-t 使用指定的日期时间，而非现在的时间

#### rm命令参数

-f, --force  忽略不存在的文件，从不给出提示。

-i, --interactive 进行交互式删除

-r, -R, --recursive  指示rm将参数中列出的全部目录和子目录均递归地删除。

-d, --dir 删除空目录

#### mv命令参数

-b ：若需覆盖文件，则覆盖前先行备份。

-f ：force 强制的意思，如果目标文件已经存在，不会询问而直接覆盖；

-i ：若目标文件 (destination) 已经存在时，就会询问是否覆盖

-n：不要覆盖现有文件。（-n选项将覆盖以前的任何-f或-i选项。）

-u ：若目标文件已经存在，且 source 比较新，才会更新(update)

#### CP命令参数

-a, --archive  等于-dR --preserve=all

--backup[=CONTROL  为每个已存在的目标文件创建备份

-b     类似--backup 但不接受参数

--copy-contents   在递归处理是复制特殊文件内容

-d     等于--no-dereference --preserve=links

-f, --force   如果目标文件无法打开则将其移除并重试(当 -n 选项

存在时则不需再选此项)

-i, --interactive   覆盖前询问(使前面的 -n 选项失效)

-H     跟随源文件中的命令行符号链接

-l, --link    链接文件而不复制

-L, --dereference  总是跟随符号链接

-n, --no-clobber  不要覆盖已存在的文件(使前面的 -i 选项失效)

-P, --no-dereference  不跟随源文件中的符号链接

-p     等于--preserve=模式,所有权,时间戳

--preserve[=属性列表  保持指定的属性(默认：模式,所有权,时间戳)，如果

可能保持附加属性：环境、链接、xattr 等

-R, -r, --recursive 复制目录及目录内的所有项目

#### 用户管理

* 添加用户
  * useradd -d /home/jeffrey -m jeffrey
* 更改密码
  * passwd jy



#### 安装软件方法

* yum安装方法

  - 查看是否存在yum

  ```
  rpm -qa | grep yum
  ```

  - 没有则安装

  ```
  rpm -ivh yum-*.noarch.rpm
  ```

  - 自定义yum的配置。我们可以通过打开/etc/yum.repos.d/Centos-Base.repo进行源的配置

  > YUM有哪些特点呢

  - 安装方便
  - 可以同时配置多个源
  - 配置文件简单明了

  > 推荐个不错的yum源

  - EPEL

  是一个针对红帽企业版Linux及衍生发行版的一个高质量附加软件包项目。网址：http://fedoraproject.org/wiki/EPEL/zh-cn

  - RPMForge

  > 这是一个第三方软件仓库，被centos社区认为是一个最安全最稳定的一个软件仓库