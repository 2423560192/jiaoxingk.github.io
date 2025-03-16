# 从零开始：手把手教你在CentOS 7 上安装 Docker

> 大家好，我是 **云辰星**，一名热爱钻研的 Python 全栈开发者，技术栈覆盖 Web 开发、爬虫、小程序开发等多个领域。
>
> 在 Web 开发中，我常使用 **Django** 和 **Flask** 打造高效应用，从后端逻辑到前端交互都能搞定。
> 爬虫是我的一大强项，擅长用 Python 撸数据，无论是抓取还是清洗处理，都游刃有余。
>
> 小程序开发也难不倒我，从 0 到 1，从开发到上线，我能独立完成全流程。
>
> 除此之外，我喜欢写 **自动化脚本**，把重复工作交给代码，效率拉满。
>
> 算法方面，我刷题不少，LeetCode 上的经典面试题信手拈来，逻辑清晰不在话下。
>
> 偶尔还玩玩 **逆向工程**，分析网站和小程的加密逻辑，破解点小难题也挺有意思。
>
> 这是我的技术小天地，欢迎一起交流代码、分享经验！

## 前言：Docker 是什么？

如果你是刚接触云计算或容器化的小白，可能听过“Docker”这个词，但不太清楚它到底是干嘛的。

简单来说，Docker 就像一个超级便携的行李箱，它能把你的应用程序、依赖库、配置文件一股脑儿打包起来，不管你把它丢到哪台服务器上（Windows、Linux 还是 Mac），都能保证它跑得起来，不用担心“在我电脑上明明好好的，怎么到你那儿就崩了”的尴尬。

专业点讲，Docker 是一个开源的容器化平台，基于 Linux 内核的容器技术（比如 cgroups 和 namespaces），让开发者可以轻松构建、部署和管理应用程序。相比传统的虚拟机，Docker 更轻量，启动速度快到飞起，几秒钟就能搞定一个容器。

而且，它还有一个强大的生态系统，比如 Docker Hub，里面有海量的镜像（预打包的软件包），直接拉下来就能用。

好了，废话不多说，今天我们就来聊聊怎么在 CentOS 7 上把 Docker 装起来，顺便配置一下加速器，让它跑得更顺畅！

## 一、安装前的准备

在动手之前，咱们得先检查一下“装备”齐不齐。Docker 对系统有最低要求：

- **操作系统**：CentOS 7，64 位架构。
- **内核版本**：3.10 或更高。

CentOS 7 默认就满足这些条件，所以一般不用担心。不过，为了确认，你可以跑个命令检查一下：

```bash
uname -r
```

如果输出的版本号 ≥ 3.10（比如 `3.10.0-1160.el7.x86_64`），那就没问题。

另外，确保你的服务器或虚拟机能联网，毕竟安装 Docker 需要从网上下载包。如果网络不通，那就得考虑离线安装了（这篇先不展开，后面有需要再说）。

## 二、安装 Docker CE

Docker 有两个版本：社区版（CE）和企业版（EE）。我们用的是免费的 Docker CE，功能够用，适合个人学习和小型项目。下面是安装步骤：

### 1. 装点“前置装备”

Docker 需要一些依赖包，比如 `yum-utils`（管理 Yum 仓库的工具）和存储驱动相关的 `device-mapper-persistent-data`、`lvm2`。一条命令搞定：

```bash
yum install -y yum-utils device-mapper-persistent-data lvm2 --skip-broken
```

`--skip-broken` 是为了跳过可能的依赖冲突，省点麻烦。

### 2. 配置阿里云镜像源

Docker 官方的仓库在国外，下载速度慢得像乌龟爬。咱们换成国内的阿里云镜像源，速度嗖嗖的。运行：

```bash
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

然后更新一下 Yum 缓存，相当于刷新“购物清单”：

```bash
yum makecache fast
```

### 3. 正式安装 Docker CE

现在可以安装 Docker 了，一句话搞定：

```bash
yum install -y docker-ce
```

安装完后，Docker 服务会自动启动。如果没启动，别急，后面有启动方法。

## 三、启动 Docker

### 1. 关闭防火墙

Docker 会用到各种端口，手动配防火墙规则有点麻烦。如果是开发环境，干脆直接关掉防火墙：

```bash
systemctl stop firewalld
systemctl disable firewalld
```

**生产环境别这么干！** 要根据需要配置端口规则，比如开放 2375（Docker 默认端口）。

### 2. 启动 Docker 服务

启动 Docker：

```bash
systemctl start docker
```

设置开机自启：

```bash
systemctl enable docker
```

### 3. 检查是否成功

跑个命令看看 Docker 能不能用：

```bash
docker images
```

如果输出是空的（没镜像）或者列出镜像列表，说明 OK。如果报错，比如：

```text
Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
```

说明服务没启动，重跑 `systemctl start docker`。

再检查一下版本：

```bash
docker -v
```
![请添加图片描述](https://i-blog.csdnimg.cn/direct/c088ce6dbd7149399faa3ffa99d9f72c.png)



出现版本就算成功啦~

### 小插曲：启动失败怎么办？

如果启动报错（比如 `Job for docker.service failed`），可能是配置文件有问题。试试这个：

```bash
cd /etc/docker/
mv daemon.json daemon.conf
systemctl restart docker
```

然后再检查状态：`systemctl status docker`。

## 四、加速下载：配置镜像加速器

Docker Hub 的默认仓库在国外，拉镜像非常慢。由于阿里云的镜像只支持他们旗下的具有公网的服务器，所以可以用以下镜像。

### 1. 镜像加速

编辑（或创建）配置文件：

```bash
vim /etc/docker/daemon.json
```

按 `i` 进入编辑模式，粘贴：

```json
{
    "registry-mirrors": [
   "https://docker.m.daocloud.io",
   "https://docker.imgdb.de",
   "https://docker-0.unsee.tech",
   "https://docker.hlmirror.com",
   "https://cjie.eu.org"
    ]
}
```

保存退出（`Esc -> :wq`），重启 Docker：

```bash
sudo systemctl daemon-reload 
sudo systemctl restart docker
```


## 五、验证一下

试着拉个小镜像测试加速效果：

```bash
docker pull mysql
```



如果速度飞快，恭喜你，Docker 安装和加速都搞定了！

## 总结

这篇文章带你从零开始，在 CentOS 7 上装好了 Docker CE，还顺手配了镜像加速器。整个过程其实不复杂，核心就三步：装依赖、加源、启动服务。接下来，你可以试试用 Docker 跑个 mysql，或者玩玩容器化部署，体验一下“打包即运行”的快感。