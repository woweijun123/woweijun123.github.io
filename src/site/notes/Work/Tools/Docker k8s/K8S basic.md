---
{"dg-publish":true,"permalink":"/Work/Tools/Docker k8s/K8S basic/","title":"K8S basic","tags":["flashcards"],"noteIcon":"","created":"2026-02-04T09:48:16.000+08:00","updated":"2026-03-16T22:50:33.167+08:00"}
---

 https://pan.baidu.com/s/15MCgIFW9vrOdNycBReEpig?pwd=3dnm 提取码: 3dnm
# 1. Kubernetes介绍
## 1.1 应用部署方式演变
在部署应用程序的方式上，主要经历了三个时代：
-  **传统部署** ：互联网早期，会直接将应用程序部署在物理机上
> 优点：简单，不需要其它技术的参与
> 缺点：不能为应用程序定义资源使用边界，很难合理地分配计算资源，而且程序之间容易产生影响

-  **虚拟化部署** ：可以在一台物理机上运行多个虚拟机，每个虚拟机都是独立的一个环境
> 优点：程序环境不会相互产生影响，提供了一定程度的安全性
> 缺点：增加了操作系统，浪费了部分资源

-  **容器化部署** ：与虚拟化类似，但是共享了操作系统
> 优点：可以保证每个容器拥有自己的文件系统、CPU、内存、进程空间等运行应用程序所需要的资源都被容器包装，并和底层基础架构解耦容器化的应用程序可以跨云服务商、跨Linux操作系统发行版进行部署
![image-20200505183738289](https://weichengjun2.dpdns.org//i/2025/02/15/67b0097c5908e.png) 

容器化部署方式给带来很多的便利，但是也会出现一些问题，比如说：
- 一个容器故障停机了，怎么样让另外一个容器立刻启动去替补停机的容器
- 当并发访问量变大的时候，怎么样做到横向扩展容器数量

这些容器管理的问题统称为 **容器编排** 问题，为了解决这些容器编排问题，就产生了一些容器编排的软件：
-  **Swarm** ：Docker自己的容器编排工具
	Docker Swarm 是一个由 Docker 开发的调度框架。由 Docker 自身开发的好处之一就是标准 Docker API 的使用，Swarm 由多个代理（Agent）组成，把这些代理称之为节点（Node）。这些节点就是主机，这些主机在启动 Docker Daemon 的时候就会打开相应的端口，以此支持 Docker 远程 API。这些机器会根据 Swarm 调度器分配给它们的任务，拉取和运行不同的镜像。
-  **Mesos** ：Apache的一个资源统一管控的工具，需要和Marathon结合使用
	Mesos 是一个分布式调度系统内核，早于 Docker 产生，Mesos 作为资源管理器，从 DC/OS (数据中心操作系统)的角度提供资源视图。主/从结构工作模式，主节点分配任务，并用从节点上的 Executor 负责执行，通过 Zookeeper 给主节点提供服务注册、服务发现功能。通过 Framework Marathon 提供容器调度的能力。
-  **Kubernetes** ：Google开源的的容器编排工具
	Kubernetes 是基于 Google 在过去十五年来大量生产环境中运行工作负载的经验。Kubernetes 的实现参考了 Google 内部的资源调度框架，但并不是 Borg 的内部容器编排系统的开源，而是借鉴 Google 从运行 Borg 获得的经验教训，形成了 Kubernetes 项目。
	它使用 Label 和 Pod 的概念来将容器划分为逻辑单元。Pods 是同地协作（co-located）容器的集合，这些容器被共同部署和调度，形成了一个服务，这是 Kubernetes 和其他两个框架的主要区别。相比于基于相似度的容器调度方式（就像 Swarm 和Mesos），这个方法简化了对集群的管理。

![image-20200524150339551](https://weichengjun2.dpdns.org//i/2025/02/15/67b0097c9b0a4.png) 
## 1.2 kubernetes简介
![image-20200406232838722](https://weichengjun2.dpdns.org//i/2025/02/15/67b0097ccf389.png) 
kubernetes，是一个全新的基于容器技术的分布式架构领先方案，是谷歌严格保密十几年的秘密武器----Borg系统的一个开源版本，于2014年9月发布第一个版本，2015年7月发布第一个正式版本。

kubernetes的本质是 **一组服务器集群** ，它可以在集群的每个节点上运行特定的程序，来对节点中的容器进行管理。目的是实现资源管理的自动化，主要提供了如下的主要功能：
- **高可用性** ：一旦某一个容器崩溃，能够在1秒中左右迅速启动新的容器
- **弹性伸缩** ：可以根据需要，自动对集群中正在运行的容器数量进行调整
- **服务发现与负载均衡** ：服务可以通过自动发现的形式找到它所依赖的服务，如果一个服务起动了多个容器，能够自动实现请求的负载均衡
- **滚动更新与回滚** ：无停机更新应用，新发布的程序版本有问题可以立即回退到原来的版本
- **存储编排** ：可以根据容器自身的需求自动创建存储卷
- **批量管理：** 可以一次性部署和管理多个容器。
- **配置与密钥管理**：安全地管理应用程序配置和敏感信息，无需重新构建镜像。
## 1.3 kubernetes组件
一个kubernetes集群主要是由 **控制节点(Master)** 、**工作节点(Node)** 构成，每个节点上都会安装不同的组件。
![image-20200406184656917](https://weichengjun2.dpdns.org//i/2025/02/15/67b0097d4922d.png) 
### Master：集群的控制平面，负责集群的决策 ( 管理 )
 **ApiServer**  : 资源操作的唯一入口，接收用户输入的命令，提供认证、授权、API注册和发现等机制
>亦称作: kube-apiserver  
>API 服务器是 Kubernetes [控制平面](https://kubernetes.io/zh-cn/docs/reference/glossary/?all=true#term-control-plane)的组件， 该组件负责公开了 Kubernetes API，负责处理接受请求的工作。 API 服务器是 Kubernetes 控制平面的前端。
>Kubernetes API 服务器的主要实现是 [kube-apiserver](https://kubernetes.io/zh-cn/docs/reference/command-line-tools-reference/kube-apiserver/)。 `kube-apiserver` 设计上考虑了水平扩缩，也就是说，它可通过部署多个实例来进行扩缩。 你可以运行 `kube-apiserver` 的多个实例，并在这些实例之间平衡流>量。

 **Scheduler**  : 负责集群资源调度，按照预定的调度策略将Pod调度到相应的node节点上
>kube-scheduler 是控制平面的组件， 负责监视新创建的、未指定运行节点（node）的 Pods， 并选择节点来让 Pod 在上面运行。
>调度决策考虑的因素包括单个 Pod 及 Pods 集合的资源需求、软硬件及策略约束、 亲和性及反亲和性规范、数据位置、工作负载间的干扰及最后时限。
 
 **ControllerManager**  : 负责维护集群的状态，比如程序部署安排、故障检测、自动扩展、滚动更新等 
>[kube-controller-manager](https://kubernetes.io/zh-cn/docs/reference/command-line-tools-reference/kube-controller-manager/) 是[控制平面](https://kubernetes.io/zh-cn/docs/reference/glossary/?all=true#term-control-plane)的组件， 负责运行[控制器](https://kubernetes.io/zh-cn/docs/concepts/architecture/controller/)进程。
>从逻辑上讲， 每个[控制器](https://kubernetes.io/zh-cn/docs/concepts/architecture/controller/)都是一个单独的进程， 但是为了降低复杂性，它们都被编译到同一个可执行文件，并在同一个进程中运行。
>这些控制器包括：
> 	节点控制器（Node Controller）：负责在节点出现故障时进行通知和响应
> 	任务控制器（Job Controller）：监测代表一次性任务的 Job 对象，然后创建 Pods 来运行这些任务直至完成
> 	端点分片控制器（EndpointSlice controller）：填充端点分片（EndpointSlice）对象（以提供 Service 和 Pod 之间的链接）。
> 	服务账号控制器（ServiceAccount controller）：为新的命名空间创建默认的服务账号（ServiceAccount）。

**cloud-controller-manager**：
>一个 Kubernetes [控制平面](https://kubernetes.io/zh-cn/docs/reference/glossary/?all=true#term-control-plane)组件， 嵌入了特定于云平台的控制逻辑。 云控制器管理器（Cloud Controller Manager）允许将你的集群连接到云提供商的 API 之上， 并将与该云平台交互的组件同与你的集群交互的组件分离开来。
>通过分离 Kubernetes 和底层云基础设置之间的互操作性逻辑， `cloud-controller-manager` 组件使云提供商能够以不同于 Kubernetes 主项目的步调发布新特征。
>
>cloud-controller-manager 仅运行特定于云平台的控制器。 因此如果你在自己的环境中运行 Kubernetes，或者在本地计算机中运行学习环境， 所部署的集群不需要有云控制器管理器。  
>与 kube-controller-manager 类似，cloud-controller-manager 将若干逻辑上独立的控制回路组合到同一个可执行文件中， 供你以同一进程的方式运行。 你可以对其执行水平扩容（运行不止一个副本）以提升性能或者增强容错能力。

 **Etcd**  ：负责存储集群中各种资源对象的信息
>一致且高可用的**键值存储**，用作 **Kubernetes** 所有**集群数据**的**后台数据库**。
>如果你的 Kubernetes 集群使用 etcd 作为其后台数据库， 请确保你针对这些数据有一份 [备份](https://kubernetes.io/zh-cn/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster)计划。
>你可以在官方[文档](https://etcd.io/docs/)中找到有关 etcd 的深入知识。

### Node：集群的数据平面，负责为容器提供运行环境 ( 干活 )
**Kubelet**  : 负责维护容器的生命周期，即通过控制docker，来创建、更新、销毁容器 
>`kubelet` 会在集群中每个[节点（node）](https://kubernetes.io/zh-cn/docs/concepts/architecture/nodes/)上运行。 它保证[容器（containers）](https://kubernetes.io/zh-cn/docs/concepts/containers/)都运行在 [Pod](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/) 中。
>[kubelet](https://kubernetes.io/zh-cn/docs/reference/command-line-tools-reference/kubelet/) 接收一组通过各类机制提供给它的 PodSpec，确保这些 PodSpec 中描述的容器处于运行状态且健康。 kubelet 不会管理不是由 Kubernetes 创建的容器。

**KubeProxy**  : 负责提供集群内部的服务发现和负载均衡 
>[kube-proxy](https://kubernetes.io/zh-cn/docs/reference/command-line-tools-reference/kube-proxy/) 是集群中每个[节点（node）](https://kubernetes.io/zh-cn/docs/concepts/architecture/nodes/)上所运行的网络代理， 实现 Kubernetes [服务（Service）](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/) 概念的一部分。
>kube-proxy 维护节点上的一些网络规则， 这些网络规则会允许从集群内部或外部的网络会话与 Pod 进行网络通信。
>如果操作系统提供了可用的数据包过滤层，则 kube-proxy 会通过它来实现网络规则。 否则，kube-proxy 仅做流量转发。

**Container Runtime**：容器运行时
>这个基础组件使 Kubernetes 能够有效运行容器。 它负责管理 Kubernetes 环境中容器的执行和生命周期。
>Kubernetes 支持许多容器运行环境，例如 [containerd](https://containerd.io/docs/)、 [CRI-O](https://cri-o.io/#what-is-cri-o) 以及 [Kubernetes CRI (容器运行环境接口)](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-node/container-runtime-interface.md) 的其他任何实现。

**Docker**  : 负责节点上容器的各种操作
>Docker（这里特指 Docker Engine）是一种可以提供操作系统级别虚拟化 （也称作[容器](https://kubernetes.io/zh-cn/docs/concepts/containers/)）的软件技术。
>Docker 使用了 Linux 内核中的资源隔离特性（如 cgroup 和内核命名空间）以及支持联合文件系统（如 OverlayFS 和其他）， 允许多个相互独立的“容器”一起运行在同一 Linux 实例上，从而避免启动和维护虚拟机（Virtual Machines；VM）的开销。

下面，以部署一个nginx服务来说明kubernetes系统各个组件调用关系：
1.  首先要明确，一旦kubernetes环境启动之后，master和node都会将自身的信息存储到**etcd数据库**中
2.  一个nginx服务的安装请求会首先被发送到master节点的ApiServer组件
3.  ApiServer组件会调用scheduler组件来决定到底应该把这个服务安装到哪个node节点上在此时，它会从etcd中读取各个node节点的信息，然后按照一定的算法进行选择，并将结果告知ApiServer
4.  ApiServer调用controller-manager去调度Node节点安装nginx服务
5.  kubelet接收到指令后，会通知docker，然后由docker来启动一个nginx的pod
	pod是kubernetes的最小操作单元，容器必须跑在pod中至此，
6.  一个nginx服务就运行了，如果需要访问nginx，就需要通过kube-proxy来对pod产生访问的代理

这样，外界用户就可以访问集群中的nginx服务了
## 1.4 kubernetes概念
 **Master** ：集群控制节点，每个集群需要至少一个master节点负责集群的管控
 **Node** ：工作负载节点，由master分配容器到这些node工作节点上，然后node节点上的docker负责容器的运行
 **Pod** ：kubernetes的最小控制单元，容器都是运行在pod中的，一个pod中可以有1个或者多个容器
 **Controller** ：控制器，通过它来实现对pod的管理，比如启动pod、停止pod、伸缩pod的数量等等
 **Service** ：pod对外服务的统一入口，下面可以维护者同一类的多个pod
 **Label** ：标签，用于对pod进行分类，同一类pod会拥有相同的标签
 **NameSpace** ：命名空间，用来隔离pod的运行环境
# 2. kubernetes集群环境搭建
## 2.1 前置知识点
目前生产部署Kubernetes 集群主要有两种方式：
**kubeadm** 
Kubeadm 是一个K8s 部署工具，提供kubeadm init 和kubeadm join，用于快速部署Kubernetes 集群。
官方地址： [https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm/](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm/) 
 **二进制包** 
从github 下载发行版的二进制包，手动部署每个组件，组成Kubernetes 集群。
Kubeadm 降低部署门槛，但屏蔽了很多细节，遇到问题很难排查。如果想更容易可控，推荐使用二进制包部署Kubernetes 集群，虽然手动部署麻烦点，期间可以学习很多工作原理，也利于后期维护。
![image-20200404094800622](https://weichengjun2.dpdns.org//i/2025/02/15/67b0097daa55f.png) 
## 2.2 kubeadm 部署方式介绍
kubeadm 是官方社区推出的一个用于快速部署kubernetes 集群的工具，这个工具能通过两条指令完成一个kubernetes 集群的部署：
- 创建一个Master 节点kubeadm init
- 将Node 节点加入到当前集群中`$ kubeadm join <Master 节点的IP 和端口>`
## 2.3 安装要求
在开始之前，部署Kubernetes 集群机器需要满足以下几个条件：
- 一台或多台机器，操作系统CentOS7.x-86_x64
- 硬件配置：2GB 或更多RAM，2 个CPU 或更多CPU，硬盘30GB 或更多
- 集群中所有机器之间网络互通
- 可以访问外网，需要拉取镜像
- 禁止swap 分区
## 2.4 最终目标
- 在所有节点上安装Docker 和kubeadm
- 部署Kubernetes Master
- 部署容器网络插件
- 部署Kubernetes Node，将节点加入Kubernetes 集群中
- 部署Dashboard Web 页面，可视化查看Kubernetes 资源
## 2.5 准备环境
![image-20210609000002940](https://weichengjun2.dpdns.org//i/2025/02/15/67b0097e068c1.png) 

| 角色       | IP地址        | 组件                             |
| -------- | ----------- | ------------------------------ |
| master01 | 10.0.0.101  | docker，kubectl，kubeadm，kubelet |
| node01   | 10.0.0.102 | docker，kubectl，kubeadm，kubelet |
| node02   | 10.0.0.103 | docker，kubectl，kubeadm，kubelet |
## 2.6 环境初始化
### 2.6.1 检查操作系统的版本
```shell
# 此方式下安装kubernetes集群要求Centos版本要在7.5或之上
cat /etc/redhat-release
Centos Linux 7.5.1804 (Core)
```
### 2.6.2 主机名解析
为了方便集群节点间的直接调用，在这个配置一下主机名解析，企业中推荐使用内部DNS服务器
```shell
# 主机名成解析 编辑三台服务器的/etc/hosts文件，添加下面内容
vim /etc/hosts
10.0.0.101 k8s-master
10.0.0.102 k8s-node1
10.0.0.103 k8s-node2
```
分别在3个节点服务器上设置主机名
```shell
# 主节点
hostnamectl set-hostname k8s-master
# 节点1
hostnamectl set-hostname k8s-node1
# 节点2
hostnamectl set-hostname k8s-node2
```
### 2.6.3 时间同步
kubernetes要求集群中的节点时间必须精确一直，这里使用chronyd服务从网络同步时间
企业中建议配置内部的会见同步服务器
```shell
# 启动chronyd服务
systemctl start chronyd
systemctl enable chronyd
date
```
### 2.6.4  禁用iptable和firewalld服务
kubernetes和docker 在运行的中会产生大量的iptables规则，为了不让系统规则跟它们混淆，直接关闭系统的规则
```shell
# 1 关闭firewalld服务
systemctl stop firewalld
systemctl disable firewalld
# 2 关闭iptables服务
systemctl stop iptables
systemctl disable iptables
```
### 2.6.5 禁用selinux
selinux是linux系统下的一个安全服务，如果不关闭它，在安装集群中会产生各种各样的奇葩问题
```shell
# 编辑 /etc/selinux/config 文件，修改SELINUX的值为disable
# 永久
sed -i 's/enforcing/disabled/' /etc/selinux/config
# 临时
setenforce 0
```
### 2.6.6 禁用swap分区
swap分区指的是虚拟内存分区，它的作用是物理内存使用完，之后将磁盘空间虚拟成内存来使用，启用swap设备会对系统的性能产生非常负面的影响，因此kubernetes要求每个节点都要禁用swap设备，但是如果因为某些原因确实不能关闭swap分区，就需要在集群安装过程中通过明确的参数进行配置说明
**注意修改完毕之后需要重启linux服务**
```shell
# 编辑分区配置文件/etc/fstab，注释掉swap分区一行
# 临时
swapoff -a
# 永久
sed -ri 's/.*swap.*/#&/' /etc/fstab
```
### 2.6.7 修改linux的内核参数
```shell
# 修改linux的内核采纳数，添加网桥过滤和地址转发功能
# 编辑/etc/sysctl.d/kubernetes.conf文件，添加如下配置：
vim /etc/sysctl.d/kubernetes.conf
# 允许 IPv6 数据包通过桥接网络时，被 ip6tables 规则过滤。
net.bridge.bridge-nf-call-ip6tables = 1
# 解释：当数据包通过 Linux 网桥（bridge）时，如果启用了此选项，IPv6 数据包将被传递给 ip6tables 进行过滤。
#      这对于在 Kubernetes 集群中使用网络策略（Network Policies）来控制 IPv6 流量非常重要。

# 允许 IPv4 数据包通过桥接网络时，被 iptables 规则过滤。
net.bridge.bridge-nf-call-iptables = 1
# 解释：与上面的 IPv6 设置类似，此选项允许 IPv4 数据包在通过 Linux 网桥时被 iptables 规则过滤。
#      这对于在 Kubernetes 集群中使用网络策略（Network Policies）来控制 IPv4 流量至关重要。

# 启用 IPv4 数据包转发。
net.ipv4.ip_forward = 1
# 解释：此选项启用 Linux 内核的 IPv4 数据包转发功能。
#      在 Kubernetes 集群中，节点需要启用数据包转发，以便 Pod 之间的网络通信能够正常进行。
#      这允许节点充当路由器，将数据包从一个网络接口转发到另一个网络接口。

# 重新加载配置
sysctl -p
# 加载网桥过滤模块
modprobe br_netfilter
# 查看网桥过滤模块是否加载成功
lsmod | grep br_netfilter
```
### 2.6.8 配置ipvs功能
在Kubernetes中Service有两种带来模型，一种是基于iptables的，一种是基于ipvs的两者比较的话，ipvs的性能明显要高一些，但是如果要使用它，需要手动载入ipvs模块
```shell
# 1.安装ipset和ipvsadm
yum install ipset ipvsadm -y
# 2.添加需要加载的模块写入脚本文件
cat <<EOF > /etc/sysconfig/modules/ipvs.modules
#!/bin/bash
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4
EOF
# 3.为脚本添加执行权限
chmod +x /etc/sysconfig/modules/ipvs.modules
# 4.执行脚本文件
/bin/bash /etc/sysconfig/modules/ipvs.modules
# 5.查看对应的模块是否加载成功
lsmod | grep -e ip_vs -e nf_conntrack_ipv4
```
### 2.6.9 安装docker
```shell
# 1、切换镜像源
wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo

# 2、查看当前镜像源中支持的docker版本
yum list docker-ce --showduplicates

# 3、安装特定版本的docker-ce
# 必须制定--setopt=obsoletes=0，否则yum会自动安装更高版本
yum install --setopt=obsoletes=0 docker-ce-18.06.3.ce-3.el7 -y

# 4、添加一个配置文件
#Docker 在默认情况下使用Vgroup Driver为cgroupfs，而Kubernetes推荐使用systemd来替代cgroupfs
mkdir /etc/docker
tee /etc/docker/daemon.json <<EOF
{
	"exec-opts": ["native.cgroupdriver=systemd"],
	"registry-mirrors": ["https://kn0t2bca.mirror.aliyuncs.com"]
}
EOF

# 5、启动dokcer
systemctl restart docker
systemctl enable docker
```
### 2.6.10 安装Kubernetes组件
```shell
# 1、由于kubernetes的镜像在国外，速度比较慢，这里切换成国内的镜像源，编辑/etc/yum.repos.d/kubernetes.repo,添加下面的配置
[kubernetes]
# 仓库的友好名称，用于描述仓库的作用，这里也是 Kubernetes
name=Kubernetes
# 仓库的 URL，指向阿里云 Kubernetes YUM 仓库的地址，适用于 CentOS/RHEL 7 x86_64 系统
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
# 启用仓库，1 表示启用，0 表示禁用
enabled=1
# 关闭 GPG 签名检查，0 表示关闭，1 表示启用（不推荐在生产环境关闭）
gpgcheck=0
# 关闭仓库GPG签名检查，0表示关闭，1表示启用(不推荐在生产环境关闭)
repo_gpgcheck=0
# GPG 密钥 URL，用于验证软件包的签名，这里指定了两个密钥的 URL
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg

# 2、安装kubeadm、kubelet和kubectl
yum install -y kubelet-1.23.6 kubeadm-1.23.6 kubectl-1.23.6

# 3、配置kubelet的cgroup
# 编辑/etc/sysconfig/kubelet, 添加下面的配置
KUBELET_CGROUP_ARGS="--cgroup-driver=systemd"
# 解释：
#   此变量配置 kubelet 的 cgroup 驱动程序为 systemd。
#   cgroup 是一种 Linux 内核特性，用于限制、记录和隔离进程组（进程集合）使用的资源（CPU、内存、磁盘 I/O 等）。
#   systemd 是一种 Linux 系统和服务管理器，它提供了一种管理 cgroup 的方式。
#   将 kubelet 的 cgroup 驱动程序设置为 systemd，可以确保 kubelet 与 Docker 或 containerd 等容器运行时使用相同的 cgroup 驱动程序，从而避免资源管理上的冲突。
#   建议：在大多数现代 Linux 发行版中，systemd 是默认的 cgroup 驱动程序，因此建议使用此设置。
KUBE_PROXY_MODE="ipvs"
# 解释：
#   此变量配置 kube-proxy 的运行模式为 ipvs。
#   kube-proxy 是 Kubernetes 集群中的一个组件，负责实现 Kubernetes Service 的负载均衡。
#   ipvs（IP Virtual Server）是一种 Linux 内核提供的负载均衡技术，它比 iptables 模式具有更高的性能和更好的可扩展性。
#   将 kube-proxy 设置为 ipvs 模式，可以提高 Kubernetes Service 的性能和可扩展性。
#   建议：在大多数生产环境中，建议使用 ipvs 模式，因为它比 iptables 模式更高效。

# 4、设置kubelet开机自启
systemctl enable kubelet
```
### 2.6.11 集群初始化
#### master节点上执行
```shell
# 创建集群
kubeadm init \  
      --apiserver-advertise-address=10.0.0.101 \  
      --image-repository registry.aliyuncs.com/google_containers \  
      --kubernetes-version v1.23.6 \  
      --service-cidr=10.96.0.0/12 \  
      --pod-network-cidr=10.244.0.0/16

# 创建用户目录下的 .kube 目录，用于存放 Kubernetes 配置文件。
mkdir -p $HOME/.kube
# 将 Kubernetes 管理员配置文件复制到用户目录下的配置文件中，以便 kubectl 可以访问集群。
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
# 更改配置文件的所有者和所属组为当前用户，确保用户具有读取和修改权限。
chown $(id -u):$(id -g) $HOME/.kube/config

# 部署 CNI 网络插件
mkdir -p /opt/k8s && cd /opt/k8s
# 下载 calico 配置文件，可能会网络超时
curl -O https://docs.projectcalico.org/manifests/calico.yaml
curl -O https://calico-v3-25.netlify.app/archive/v3.25/manifests/calico.yaml
# 修改 calico.yaml 文件中的 CALICO_IPV4POOL_CIDR 配置，修改为与初始化的 cidr(10.244.0.0/16) 相同
vim calico.yaml
# 删除镜像 docker.io/ 前缀，避免下载过慢导致失败
sed -i 's#docker.io/##g' calico.yaml
# 安装并配置Calico作为你的Kubernetes集群的CNI（容器网络接口）插件
kubectl apply -f calico.yaml
```
#### node节点上执行
```shell
kubeadm join 10.0.0.101:6443 --token awk15p.t6bamck54w69u4s8 \
    --discovery-token-ca-cert-hash sha256:a94fa09562466d32d29523ab6cff122186f1127599fa4dcd5fa0152694f17117
```
#### 如果初始化的 token 不小心清空了，可以通过如下命令获取或者重新申请
```shell
# 如果 token 已经过期，就重新申请  
kubeadm token create
# token 没有过期可以通过如下命令获取  
kubeadm token list
# 获取 --discovery-token-ca-cert-hash 值，得到值后需要在前面拼接上 sha256:
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | \
openssl dgst -sha256 -hex | sed 's/^.* //'
```
#### 在master上查看节点信息
```shell
kubectl get nodes
NAME    STATUS   ROLES     AGE   VERSION
k8s-master  NotReady  master   6m    v1.23.6
k8s-node1   NotReady   <none>  22s   v1.23.6
k8s-node2   NotReady   <none>  19s   v1.23.6
```
#### 在任意节点使用 kubectl  
```shell
# 1. 将 master 节点中 /etc/kubernetes/admin.conf 拷贝到需要运行的服务器的 /etc/kubernetes 目录中
scp /etc/kubernetes/admin.conf root@k8s-node1:/etc/kubernetes
scp /etc/kubernetes/admin.conf root@k8s-node2:/etc/kubernetes

# 2. 在对应的服务器上配置环境变量
echo "export KUBECONFIG=/etc/kubernetes/admin.conf" >> ~/.bash_profile
source ~/.bash_profile
```
### 2.6.12 使用kubeadm reset重置集群「可以不执行这一步」
```shell
# 在master节点之外的节点进行操作
kubeadm reset
systemctl stop kubelet
systemctl stop docker
rm -rf /var/lib/cni/
rm -rf /var/lib/kubelet/*
rm -rf /etc/cni/
ifconfig cni0 down
ifconfig flannel.1 down
ifconfig docker0 down
ip link delete cni0
ip link delete flannel.1
```
### 2.6.13 重启kubelet和docker
```shell
# 重启kubelet
systemctl restart kubelet
# 重启docker
systemctl restart docker
```
重启完毕，集群的状态已经是Ready
```shell
# 查询节点情况
kubectl get nodes        
NAME         STATUS   ROLES                  AGE   VERSION
k8s-master   Ready    control-plane,master   49d   v1.23.6
k8s-node1    Ready    <none>                 49d   v1.23.6
k8s-node2    Ready    <none>                 15d   v1.23.6
```
### 2.6.14 kubeadm中的命令
```shell
# 生成 新的token
kubeadm token create --print-join-command
```
# 3. 资源管理
## 3.1 资源管理介绍
在kubernetes中，所有的内容都抽象为资源，用户需要通过操作资源来管理kubernetes。
> kubernetes的本质上就是一个集群系统，用户可以在集群中部署各种服务，所谓的部署服务，其实就是在kubernetes集群中运行一个个的容器，并将指定的程序跑在容器中。
> kubernetes的最小管理单元是pod而不是容器，所以只能将容器放在 `Pod` 中，而kubernetes一般也不会直接管理Pod，而是通过 `Pod控制器` 来管理Pod的。
> Pod可以提供服务之后，就要考虑如何访问Pod中服务，kubernetes提供了 `Service` 资源实现这个功能。
> 当然，如果Pod中程序的数据需要持久化，kubernetes还提供了各种 `存储` 系统。

![image-20200406225334627](https://weichengjun2.dpdns.org//i/2025/02/15/67b009803094d.png) 
> 学习kubernetes的核心，就是学习如何对集群上的 `Pod、Pod控制器、Service、存储` 等各种资源进行操作
## 3.2 资源级别
### 1. 工作负载（Workloads）
- **Pod (命名空间级别):**
    - Kubernetes 中最小的可部署单元，包含一个或多个容器。
- **ReplicaSet (命名空间级别):**
    - 确保指定数量的 Pod 副本始终运行。
- **Deployment (命名空间级别):**
    - 用于管理 ReplicaSet 和 Pod 的声明式更新。
- **StatefulSet (命名空间级别):**
    - 用于管理有状态应用，提供稳定的网络标识和持久化存储。
- **DaemonSet (命名空间级别):**
    - 确保每个节点上都运行一个 Pod 副本，常用于日志收集或监控。
- **Job (命名空间级别):**
    - 用于运行一次性任务，任务完成后 Pod 自动终止。
- **CronJob (命名空间级别):**
    - 基于时间调度的 Job，类似于 Linux 的 cron。
### 2. 服务发现和负载均衡（Service Discovery & Load Balancing）
- **Service (命名空间级别):**
    - 为 Pod 提供稳定的网络访问，实现服务发现和负载均衡。
- **Ingress (命名空间级别):**
    - 管理集群外部访问集群内部服务的 HTTP 和 HTTPS 路由。
- **EndpointSlice (命名空间级别):**
    - EndpointSlice 为网络终端节点提供了一种可伸缩的替代方案。
### 3. 配置和存储（Config & Storage）
- **ConfigMap (命名空间级别):**
    - 用于存储非敏感的配置数据，供 Pod 使用。
- **Secret (命名空间级别):**
    - 用于存储敏感信息，如密码、令牌等。
- **PersistentVolume (集群级别):**
    - 集群中可用的持久化存储。
- **PersistentVolumeClaim (命名空间级别):**
    - Pod 请求持久化存储的声明。
- **StorageClass (集群级别):**
    - 用于动态 provision PersistentVolume 的模板。
### 4. 权限控制（RBAC）
- **Role (命名空间级别):**
    - 定义命名空间内的权限规则。
- **ClusterRole (集群级别):**
    - 定义集群范围内的权限规则。
- **RoleBinding (命名空间级别):**
    - 将 Role 绑定到用户、组或 ServiceAccount。
- **ClusterRoleBinding (集群级别):**
    - 将 ClusterRole 绑定到用户、组或 ServiceAccount。
- **ServiceAccount (命名空间级别):**
    - 为Pod中运行的进程提供身份验证。
### 5. 集群管理（Cluster Management）
- **Node (集群级别):**
    - Kubernetes 集群中的工作机器。
- **Namespace (集群级别):**
    - 用于逻辑隔离集群资源的虚拟集群。
- **CustomResourceDefinition (集群级别):**
    - 用于扩展 Kubernetes API，定义自定义资源。
### 6. 元数据资源（Meta data resources）
- **HorizontalPodAutoscaler (命名空间级别):**
    - 根据 CPU 利用率或其他指标自动调整 Pod 副本数量。
- **LimitRange (命名空间级别):**
    - 限制命名空间内资源的资源使用量。
- **ResourceQuota (命名空间级别):**
    - 限制命名空间内资源的总使用量。

## 3.2 YAML语言介绍

YAML是一个类似 XML、JSON 的标记性语言。它强调以 **数据** 为中心，并不是以标识语言为重点。因而YAML本身的定义比较简单，号称"一种人性化的数据格式语言"。
```yml
<riven>
    <age>15</age>
    <address>Beijing</address>
</riven>
```

```yaml
riven:
  age: 15
  address: Beijing
```

YAML的语法比较简单，主要有下面几个：
- 大小写敏感
- 使用缩进表示层级关系
- 缩进不允许使用tab，只允许空格( 低版本限制 )
- 缩进的空格数不重要，只要相同层级的元素左对齐即可
- '#'表示注释

YAML支持以下几种数据类型：
- 纯量：单个的、不可再分的值
- 对象：键值对的集合，又称为映射（mapping）/ 哈希（hash） / 字典（dictionary）
- 数组：一组按次序排列的值，又称为序列（sequence） / 列表（list）
```yml
# 纯量, 就是指的一个简单的值，字符串、布尔值、整数、浮点数、Null、时间、日期
# 1 布尔类型
c1: true (或者True)
# 2 整型
c2: 234
# 3 浮点型
c3: 3.14
# 4 null类型 
c4: ~  # 使用~表示null
# 5 日期类型
c5: 2018-02-17    # 日期必须使用ISO 8601格式，即yyyy-MM-dd
# 6 时间类型
c6: 2018-02-17T15:02:31+08:00  # 时间使用ISO 8601格式，时间和日期之间使用T连接，最后使用+代表时区
# 7 字符串类型
c7: riven     # 简单写法，直接写值 , 如果字符串中间有特殊字符，必须使用双引号或者单引号包裹 
c8: line1
    line2     # 字符串过多的情况可以拆成多行，每一行会被转化成一个空格
```

```yml
# 对象
# 形式一(推荐):
riven:
  age: 15
  address: Beijing
# 形式二(了解):
riven: {age: 15,address: Beijing}
```

```yml
# 数组
# 形式一(推荐):
address:
  - 顺义
  - 昌平
# 形式二(了解):
address: [顺义,昌平]
```

> 小提示：
> 1 书写yaml切记 `:`(冒号) 后面要加一个空格
> 2 如果需要将多段yaml配置放在一个文件中，中间要使用 `---` 分隔
> 3 下面是一个yaml转json的网站，可以通过它验证yaml是否书写正确
>  [https://www.json2yaml.com/convert-yaml-to-json](https://www.json2yaml.com/convert-yaml-to-json) 

## 3.3 资源管理方式
- 命令式对象管理：直接使用命令去操作kubernetes资源
```shell
kubectl run svc-pod --image=nginx:1.17.1 --port=80
```
- 命令式对象配置：通过命令配置和配置文件去操作kubernetes资源
```shell
kubectl create/patch -f svc-pod.yaml
```
- 声明式对象配置：通过apply命令和配置文件去操作kubernetes资源
```shell
kubectl apply -f svc-pod.yaml
```

|类型|操作对象|适用环境|优点|缺点|
|--|----|----|--|--|
|命令式对象管理|对象|测试|简单|只能操作活动对象，无法审计、跟踪|
|命令式对象配置|文件|开发|可以审计、跟踪|项目大时，配置文件多，操作麻烦|
|声明式对象配置|目录|开发|支持目录操作|意外情况下难以调试|
### 3.3.1 命令式对象管理
 **kubectl命令** 
kubectl是kubernetes集群的命令行工具，通过它能够对集群本身进行管理，并能够在集群上进行容器化应用的安装部署。kubectl命令的语法如下：
```shell
kubectl [command] [type] [name] [flags]
```
 **comand** ：指定要对资源执行的操作，例如create、get、delete
 **type** ：指定资源类型，比如deployment、pod、service
 **name** ：指定资源的名称，名称大小写敏感
 **flags** ：指定额外的可选参数
```shell
# 查看所有pod
kubectl get pod 

# 查看某个pod
kubectl get pod pod_name

# 查看某个pod,以yaml格式展示结果
kubectl get pod pod_name -o yaml
```

 **资源类型** 
kubernetes中所有的内容都抽象为资源，可以通过下面的命令进行查看:
```shell
kubectl api-resources
```
经常使用的资源有下面这些：

|资源分类|资源名称|缩写|资源作用|
|----|----|--|----|
|集群级别资源|nodes|no|集群组成部分|
|namespaces|ns|隔离Pod||
|pod资源|pods|po|装载容器|
|pod资源控制器|replicationcontrollers|rc|控制pod资源|
||replicasets|rs|控制pod资源|
||deployments|deploy|控制pod资源|
||daemonsets|ds|控制pod资源|
||jobs||控制pod资源|
||cronjobs|cj|控制pod资源|
||horizontalpodautoscalers|hpa|控制pod资源|
||statefulsets|sts|控制pod资源|
|服务发现资源|services|svc|统一pod对外接口|
||ingress|ing|统一pod对外接口|
|存储资源|volumeattachments||存储|
||persistentvolumes|pv|存储|
||persistentvolumeclaims|pvc|存储|
|配置资源|configmaps|cm|配置|
||secrets||配置|
 **操作** 
kubernetes允许对资源进行多种操作，可以通过–help查看详细的操作命令
```shell
kubectl --help
```
经常使用的操作有下面这些：

| 命令分类  | 命令           | 翻译               | 命令作用                 |
| ----- | ------------ | ---------------- | -------------------- |
| 基本命令  | create       | 创建               | 创建一个资源               |
|       | edit         | 编辑               | 编辑一个资源               |
|       | get          | 获取               | 获取一个资源               |
|       | patch        | 更新               | 更新一个资源               |
|       | delete       | 删除               | 删除一个资源               |
|       | explain      | 解释               | 展示资源文档               |
| 运行和调试 | run          | 运行               | 在集群中运行一个指定的镜像        |
|       | expose       | 暴露               | 暴露资源为Service         |
|       | describe     | 描述               | 显示资源内部信息             |
|       | logs         | 日志输出容器在 pod 中的日志 | 输出容器在 pod 中的日志       |
|       | attach       | 缠绕进入运行中的容器       | 进入运行中的容器             |
|       | exec         | 执行容器中的一个命令       | 执行容器中的一个命令           |
|       | cp           | 复制               | 在Pod内外复制文件           |
|       | rollout      | 首次展示             | 管理资源的发布              |
|       | scale        | 规模               | 扩(缩)容Pod的数量          |
|       | autoscale    | 自动调整             | 自动调整Pod的数量           |
| 高级命令  | apply        | rc               | 通过文件对资源进行配置          |
|       | label        | 标签               | 更新资源上的标签             |
| 其他命令  | cluster-info | 集群信息             | 显示集群信息               |
|       | version      | 版本               | 显示当前Server和Client的版本 |
下面以一个namespace / pod的创建和删除简单演示下命令的使用：
```shell
# 创建一个namespace
kubectl create namespace dev
namespace/dev created

# 获取namespace
kubectl get ns
NAME              STATUS   AGE
default           Active   21h
dev               Active   21s
kube-node-lease   Active   21h
kube-public       Active   21h
kube-system       Active   21h

# 在此namespace下创建并运行一个nginx的Pod
kubectl run pod --image=nginx:1.17.1 
kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
deployment.apps/pod created

# 查看新创建的pod
kubectl get pod 
NAME  READY   STATUS    RESTARTS   AGE
pod   1/1     Running   0          21s

# 删除指定的pod
kubectl delete pod pod-864f9875b9-pcw7x
pod "pod" deleted

# 删除指定的namespace
kubectl delete ns dev
namespace "dev" deleted
```

### 3.3.2 命令式对象配置

命令式对象配置就是使用命令配合配置文件一起来操作kubernetes资源。
1） 创建一个nginxpod.yaml，内容如下：
```yml
apiVersion: v1
kind: Namespace
metadata:
  name: dev

---

apiVersion: v1
kind: Pod
metadata:
  name: nginxpod
spec:
  containers:
  - name: nginx-containers
    image: nginx:1.17.1
```
2）执行create命令，创建资源：
```shell
kubectl create -f nginxpod.yaml
namespace/dev created
pod/nginxpod created
```
此时发现创建了两个资源对象，分别是namespace和pod
3）执行get命令，查看资源：
```shell
 kubectl get -f nginxpod.yaml
NAME            STATUS   AGE
namespace/dev   Active   18s

NAME            READY   STATUS    RESTARTS   AGE
pod/nginxpod    1/1     Running   0          17s
```
这样就显示了两个资源对象的信息
4）执行delete命令，删除资源：
```shell
kubectl delete -f nginxpod.yaml
namespace "dev" deleted
pod "nginxpod" deleted
```
此时发现两个资源对象被删除了
>总结: 命令式对象配置的方式操作资源，可以简单的认为：命令  +  yaml配置文件（里面是命令需要的各种参数）

### 3.3.3 声明式对象配置

声明式对象配置跟命令式对象配置很相似，但是它只有一个命令apply。
```shell
# 首先执行一次kubectl apply -f yaml文件，发现创建了资源
kubectl apply -f nginxpod.yaml
namespace/dev created
pod/nginxpod created

# 再次执行一次kubectl apply -f yaml文件，发现说资源没有变动
kubectl apply -f nginxpod.yaml
namespace/dev unchanged
pod/nginxpod unchanged
```
>总结:  
    其实声明式对象配置就是使用apply描述一个资源最终的状态（在yaml中定义状态）  
    使用apply操作资源：  
        如果资源不存在，就创建，相当于 kubectl create  
        如果资源已存在，就更新，相当于 kubectl patch

> 扩展：kubectl可以在node节点上运行吗 ?

kubectl的运行是需要进行配置的，它的配置文件是$HOME/.kube，如果想要在node节点运行此命令，需要将master上的.kube文件复制到node节点上，即在master节点上执行下面操作：
```shell
scp  -r  HOME/.kube   k8s-node1: HOME/
```

> 使用推荐: 三种方式应该怎么用 ?

查询资源 使用命令式对象管理 `kubectl get(describe) 资源名称`
删除资源 使用命令式对象配置 `kubectl delete -f XXX.yaml`
创建/更新资源 使用声明式对象配置 `kubectl apply -f XXX.yaml`
# 4. 实战入门
本章节将介绍如何在kubernetes集群中部署一个nginx服务，并且能够对其进行访问。
## 4.1 Namespace
Namespace是kubernetes系统中的一种非常重要资源，它的主要作用是用来实现 **多套环境的资源隔离** 或者 **多租户的资源隔离** 。
默认情况下，kubernetes集群中的所有的Pod都是可以相互访问的。但是在实际中，可能不想让两个Pod之间进行互相的访问，那此时就可以将两个Pod划分到不同的namespace下。kubernetes通过将集群内部的资源分配到不同的Namespace中，可以形成逻辑上的"组"，以方便不同的组的资源进行隔离使用和管理。
可以通过kubernetes的授权机制，将不同的namespace交给不同租户进行管理，这样就实现了多租户的资源隔离。
此时还能结合kubernetes的资源配额机制，限定不同租户能占用的资源，例如CPU使用量、内存使用量等等，来实现租户可用资源的管理。
![image-20200407100850484](https://weichengjun2.dpdns.org//i/2025/02/15/67b0098096a70.png) 
kubernetes在集群启动之后，会默认创建几个namespace
```shell
kubectl get namespace
NAME              STATUS   AGE
default           Active   45h     #  所有未指定Namespace的对象都会被分配在default命名空间
kube-node-lease   Active   45h     #  集群节点之间的心跳维护，v1.13开始引入
kube-public       Active   45h     #  此命名空间下的资源可以被所有人访问（包括未认证用户）
kube-system       Active   45h     #  所有由Kubernetes系统创建的资源都处于这个命名空间
```
下面来看namespace资源的具体操作：
### 4.1.1  **查看** 
```shell
# 1 查看所有的ns  命令：kubectl get ns
kubectl get ns
NAME              STATUS   AGE
default           Active   45h
kube-node-lease   Active   45h
kube-public       Active   45h     
kube-system       Active   45h     

# 2 查看指定的ns   命令：kubectl get ns ns名称
kubectl get ns default
NAME      STATUS   AGE
default   Active   45h

# 3 指定输出格式  命令：kubectl get ns ns名称  -o 格式参数
# kubernetes支持的格式有很多，比较常见的是wide、json、yaml
kubectl get ns default -o yaml
apiVersion: v1 # Kubernetes API 版本
kind: Namespace # Kubernetes 资源类型，这里是 Namespace（命名空间）
metadata: # 元数据，描述资源的属性
  creationTimestamp: "2021-05-08T04:44:16Z" # 创建时间戳
  name: default # 命名空间名称，这里是 default（默认）命名空间
  resourceVersion: "151" # 资源版本，用于乐观锁
  selfLink: /api/v1/namespaces/default # 资源的自链接
  uid: 7405f73a-e486-43d4-9db6-145f1409f090 # 资源的唯一 ID (UUID)
spec: # 规范，描述资源的期望状态
  finalizers: # 终结器，用于控制资源的删除行为
  - kubernetes # Kubernetes 终结器，防止命名空间被意外删除
status: # 状态，描述资源的当前状态
  phase: Active # 命名空间的当前状态，这里是 Active（活跃）
  
# 4 查看ns详情  命令：kubectl describe ns ns名称
kubectl describe ns default
Name:         default
Labels:       <none>
Annotations:  <none>
Status:       Active  # Active 命名空间正在使用中  Terminating 正在删除命名空间

# ResourceQuota 针对namespace做的资源限制
# LimitRange针对namespace中的每个组件做的资源限制
No resource quota.
No LimitRange resource.
```
### 4.1.2  **创建** 
```shell
# 创建namespace
kubectl create ns dev
namespace/dev created
```
### 4.1.3  **删除** 
```shell
# 删除namespace
kubectl delete ns dev
namespace "dev" deleted
```
### 4.1.4  **配置方式** 
首先准备一个yaml文件：`ns-dev.yaml`
```yml
apiVersion: v1  # 指定使用的 API 版本为 v1
kind: Namespace  # 表明此资源配置创建的是一个命名空间对象
metadata:  # 元数据部分，包含资源的各种属性
  name: dev  # 定义命名空间的名称，这里是 "dev"
```
然后就可以执行对应的创建和删除命令了：
```shell
# 创建
kubectl create -f ns-dev.yaml
# 删除
kubectl delete -f ns-dev.yaml
```
## 4.2 Pod
Pod是kubernetes集群进行管理的最小单元，程序要运行必须部署在容器中，而容器必须存在于Pod中。
Pod可以认为是容器的封装，一个Pod中可以存在一个或者多个容器。
![image-20200407121501907](https://weichengjun2.dpdns.org//i/2025/02/15/67b00980d46fc.png) 
kubernetes在集群启动之后，集群中的各个组件也都是以Pod方式运行的。可以通过下面命令查看：
```shell
kubectl get pod -n kube-system
NAMESPACE     NAME                             READY   STATUS    RESTARTS   AGE
kube-system   coredns-6955765f44-68g6v         1/1     Running   0          2d1h
kube-system   coredns-6955765f44-cs5r8         1/1     Running   0          2d1h
kube-system   etcd-master                      1/1     Running   0          2d1h
kube-system   kube-apiserver-master            1/1     Running   0          2d1h
kube-system   kube-controller-manager-master   1/1     Running   0          2d1h
kube-system   kube-flannel-ds-amd64-47r25      1/1     Running   0          2d1h
kube-system   kube-flannel-ds-amd64-ls5lh      1/1     Running   0          2d1h
kube-system   kube-proxy-685tk                 1/1     Running   0          2d1h
kube-system   kube-proxy-87spt                 1/1     Running   0          2d1h
kube-system   kube-scheduler-master            1/1     Running   0          2d1h
```
### 4.2.1 创建并运行
kubernetes没有提供单独运行Pod的命令，都是通过Pod控制器来实现的
```shell
# 命令格式： kubectl run (pod控制器名称) [参数] 
# --image  指定Pod的镜像
# --port   指定端口
# --namespace  指定namespace
kubectl run nginx --image=nginx:1.17.1 --port=80 --namespace dev 
deployment.apps/nginx created
```
### 4.2.2 查看pod信息
```shell
# 查看Pod基本信息
kubectl get pods 
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          43s

# 查看Pod的详细信息
kubectl describe pod nginx 
Name:         nginx
Namespace:    dev
Priority:     0
Node:         k8s-node1/192.168.5.4
Start Time:   Wed, 08 May 2021 09:29:24 +0800
Labels:       pod-template-hash=5ff7956ff6
              run=nginx
Annotations:  <none>
Status:       Running
IP:           10.244.1.23
IPs:
  IP:           10.244.1.23
Controlled By:  ReplicaSet/nginx
Containers:
  nginx:
    Container ID:   docker://4c62b8c0648d2512380f4ffa5da2c99d16e05634979973449c98e9b829f6253c
    Image:          nginx:1.17.1
    Image ID:       docker-pullable://nginx@sha256:485b610fefec7ff6c463ced9623314a04ed67e3945b9c08d7e53a47f6d108dc7
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Wed, 08 May 2021 09:30:01 +0800
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-hwvvw (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-hwvvw:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-hwvvw
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age        From               Message
  ----    ------     ----       ----               -------
  Normal  Scheduled  <unknown>  default-scheduler  Successfully assigned dev/nginx-5ff7956ff6-fg2db to k8s-node1
  Normal  Pulling    4m11s      kubelet, k8s-node1     Pulling image "nginx:1.17.1"
  Normal  Pulled     3m36s      kubelet, k8s-node1     Successfully pulled image "nginx:1.17.1"
  Normal  Created    3m36s      kubelet, k8s-node1     Created container nginx
  Normal  Started    3m36s      kubelet, k8s-node1     Started container nginx
```
### 4.2.3 访问Pod
```shell
# 获取podIP
kubectl get pods -o wide
```
创建service `svc-nodeport.yaml`
```yaml
apiVersion: v1  # 指定使用的 API 版本为 v1
kind: Service  # 表明此资源配置创建的是一个服务对象 (Service)
metadata:  # 元数据部分，包含资源的各种属性
  name: svc-nodeport  # 定义服务的名称为 "svc-nodeport"
spec:  # 规格部分，定义服务的具体配置
  selector:  # 标签选择器，用于选择与此服务关联的 Pod
    app: svc-pod  # 选择标签为 app=nginx-pod 的 Pod
  type: NodePort  # 指定服务类型为 NodePort，允许外部通过节点 IP 和指定端口访问服务
  ports:  # 定义服务的端口配置
  - port: 80  # 服务的虚拟端口，集群内部使用该端口访问服务
    nodePort: 30022  # 指定绑定到节点的端口 (默认取值范围是：30000-32767)，如果不指定，会自动分配
    targetPort: 80  # 目标端口，即 Pod 内部容器监听的端口
```
执行命令
```shell
# 创建service
kubectl create -f svc-nodeport.yaml
# 查看service
kubectl get svc -o wide
NAME               TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)       SELECTOR
svc-nodeport   NodePort   10.105.64.191   <none>        80:30022/TCP  app=nginx-pod
```
接下来可以通过电脑主机的浏览器去访问集群中任意一个nodeip的30022端口，即可访问到pod
### 4.2.4 删除指定Pod
```shell
# 删除指定Pod
kubectl delete pod nginx 
pod "nginx" deleted

# 此时，显示删除Pod成功，但是再查询，发现又新产生了一个 
kubectl get pods 
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          21s

# 这是因为当前Pod是由Pod控制器创建的，控制器会监控Pod状况，一旦发现Pod死亡，会立即重建
# 此时要想删除Pod，必须删除Pod控制器

# 先来查询一下当前namespace下的Pod控制器
kubectl get deploy -n  dev
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   1/1     1            1           9m7s

# 接下来，删除此PodPod控制器
kubectl delete deploy nginx 
deployment.apps "nginx" deleted

# 稍等片刻，再查询Pod，发现Pod被删除了
kubectl get pods 
No resources found in dev namespace.
```
### 4.2.5 配置操作
创建一个`pod-nginx.yaml`，内容如下：
```yml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - image: nginx:1.17.1
    name: pod
    ports:
    - name: nginx-port
      containerPort: 80
      protocol: TCP
```
然后就可以执行对应的创建和删除命令了：
```shell
# 创建
kubectl create -f pod-nginx.yaml
# 删除
kubectl delete -f pod-nginx.yaml
```
## 4.3 Label
Label是kubernetes系统中的一个重要概念。它的作用就是在资源上添加标识，用来对它们进行区分和选择。
特点：
- 一个Label会以**key/value键值对**的形式附加到各种对象上，如Node、Pod、Service等等
- 一个资源对象可以定义**任意数量**的Label ，同一个Label也可以被添加到任意数量的资源对象上去
- Label通常在资源对象定义时确定，当然也可以在对象创建后动态添加或者删除

可以通过Label实现资源的多维度分组，以便灵活、方便地进行资源分配、调度、配置、部署等管理工作。
常用示例：
```shell
版本标签：“version”:“release”, “version”:“stable”…
环境标签：“environment”:“dev”，“environment”:“test”，“environment”:“pro”
架构标签：“tier”:“frontend”，“tier”:“backend”
```

标签的选择使用 `Label Selector` 查询和筛选拥有某些标签的资源对象
两种Label Selector：
- 基于等式的Label Selector
	- name = slave: 选择所有包含Label中key="name"且value="slave"的对象
	- env != production: 选择所有包括Label中的key="env"且value不等于"production"的对象
- 基于集合的Label Selector
	- name in (master, slave): 选择所有包含Label中的key="name"且value="master"或"slave"的对象
	- name not in (frontend): 选择所有包含Label中的key="name"且value不等于"frontend"的对象

标签的选择条件可以使用多个，此时将多个Label Selector进行组合，使用逗号","进行分隔即可。例如：
```shell
name=slave，env!=production
name not in (frontend)，env!=production
```
### 4.3.1 命令方式
```shell
# 新增标签
kubectl label pod nginx-pod version=1.0 
pod/nginx-pod labeled

# 删除标签
kubectl label pod nginx-pod version-
# 删除多个标签
kubectl label pods my-pod version- environment-
# 删除所有标签
kubectl label pods my-pod --all

# 更新标签
kubectl label pod nginx-pod version=2.0 --overwrite
pod/nginx-pod labeled

# 查看指定pod标签
kubectl get pod nginx-pod --show-labels
NAME        READY   STATUS    RESTARTS   AGE   LABELS
nginx-pod   1/1     Running   0          10m   version=2.0

# 查询所有节点的标签
kubectl get no --show-labels

# 筛选标签
kubectl get pod -l version=2.0 --show-labels
NAME        READY   STATUS    RESTARTS   AGE   LABELS
nginx-pod   1/1     Running   0          17m   version=2.0

kubectl get pod -l version!=2.0 --show-labels
No resources found in dev namespace.
```
### 4.3.2 配置方式
```yml
apiVersion: v1  # 指定使用的 API 版本为 v1
kind: Pod  # 表明此资源配置创建的是一个 Pod 对象
metadata:  # 元数据部分，包含资源的各种属性
  name: nginx  # 定义 Pod 的名称为 "nginx"
  labels:  # 标签部分，用于标识和选择 Pod
    version: "3.0"  # 标签键值对，版本为 "3.0"
    env: "test"  # 标签键值对，环境为 "test"
spec:  # 规格部分，定义 Pod 的具体配置
  containers:  # 定义 Pod 中运行的容器列表
  - image: nginx:1.17.1  # 使用的镜像为 nginx 的最新版本 (latest)
    name: pod  # 容器的名称为 "pod"
    ports:  # 定义容器暴露的端口列表
    - name: nginx-port  # 端口的名称为 "nginx-port"
      containerPort: 80  # 容器内部监听的端口为 80
      protocol: TCP  # 协议类型为 TCP
```
然后就可以执行对应的更新命令了
```shell
kubectl apply -f pod-nginx.yaml
```

## 4.4 Deployment
在kubernetes中，Pod是最小的控制单元，但是kubernetes很少直接控制Pod，一般都是通过Pod控制器来完成的。
Pod控制器用于pod的管理，确保pod资源符合预期的状态，当pod的资源出现故障时，会尝试进行重启或重建pod。
在kubernetes中Pod控制器的种类有很多，本章节只介绍一种：Deployment。
![image-20200408193950807](https://weichengjun2.dpdns.org//i/2025/02/15/67b0098124434.png) 
### 4.4.1 命令操作
```shell
# 命令格式: kubectl create deployment 名称  [参数] 
# --image  指定pod的镜像
# --port   指定端口
# --replicas  指定创建pod数量
# --namespace  指定namespace
kubectl run nginx --image=nginx:1.17.1 --port=80 --replicas=3
deployment.apps/nginx created

# 查看创建的Pod
kubectl get pods 
NAME                     READY   STATUS    RESTARTS   AGE
nginx-5ff7956ff6-6k8cb   1/1     Running   0          19s
nginx-5ff7956ff6-jxfjt   1/1     Running   0          19s
nginx-5ff7956ff6-v6jqw   1/1     Running   0          19s

# 查看deployment的信息
kubectl get deploy 
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   3/3     3            3           2m42s

# UP-TO-DATE：成功升级的副本数量
# AVAILABLE：可用副本的数量
kubectl get deploy -o wide
NAME    READY UP-TO-DATE  AVAILABLE   AGE     CONTAINERS   IMAGES              SELECTOR
nginx   3/3     3         3           2m51s   nginx        nginx:1.17.1        run=nginx

# 查看deployment的详细信息
kubectl describe deploy nginx 
Name:                   nginx
Namespace:              dev
CreationTimestamp:      Wed, 08 May 2021 11:14:14 +0800
Labels:                 run=nginx
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               run=nginx
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% maxSurge
Pod Template:
  Labels:  run=nginx
  Containers:
   nginx:
    Image:        nginx:1.17.1
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-5ff7956ff6 (3/3 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  5m43s  deployment-controller  Scaled up replicaset nginx-5ff7956ff6 to 3
  
# 删除 
kubectl delete deploy nginx 
deployment.apps "nginx" deleted
```
### 4.4.2 配置操作
创建一个`deploy-nginx.yaml`，内容如下：
```yml
apiVersion: apps/v1  # 指定使用的 API 版本为 apps/v1，适用于 Deployment 资源
kind: Deployment  # 表明此资源配置创建的是一个 Deployment 对象
metadata:  # 元数据部分，包含资源的各种属性
  name: nginx  # 定义 Deployment 的名称为 "nginx"
spec:  # 规格部分，定义 Deployment 的具体配置
  replicas: 3  # 设置副本数量为 3，即运行 3 个 Pod 副本
  selector:  # 选择器部分，用于选择由该 Deployment 管理的 Pod
    matchLabels:  # 匹配标签，选择具有指定标签的 Pod
      run: nginx  # 标签键值对，选择标签为 run=nginx 的 Pod
  template:  # Pod 模板，定义了每个 Pod 的具体配置
    metadata:  # Pod 的元数据部分
      labels:  # 标签部分，用于标识和选择 Pod
        run: nginx  # 标签键值对，标签为 run=nginx
    spec:  # Pod 的规格部分
      containers:  # 定义 Pod 中运行的容器列表
      - image: nginx:1.17.1  # 使用的镜像为 nginx 的最新版本 (latest)
        name: nginx  # 容器的名称为 "nginx"
        ports:  # 定义容器暴露的端口列表
        - containerPort: 80  # 容器内部监听的端口为 80
          protocol: TCP  # 协议类型为 TCP
```
然后就可以执行对应的创建和删除命令了：
```shell
# 创建
kubectl create -f deploy-nginx.yaml
# 删除
kubectl delete -f deploy-nginx.yaml
```
## 4.5 Service
通过上节课的学习，已经能够利用Deployment来创建一组Pod来提供具有高可用性的服务。
虽然每个Pod都会分配一个单独的Pod IP，然而却存在如下两问题：
- Pod IP 会随着Pod的重建产生变化
- Pod IP 仅仅是集群内可见的虚拟IP，外部无法访问

这样对于访问这个服务带来了难度。因此，kubernetes设计了Service来解决这个问题。
Service可以看作是一组同类Pod **对外的访问接口** 。借助Service，应用可以方便地实现服务发现和负载均衡。
![image-20200408194716912](https://weichengjun2.dpdns.org//i/2025/02/15/67b0098169a71.png) 
### 4.5.1 创建集群内部可访问的Service
```shell
# 暴露Service
kubectl expose deploy nginx --name=svc-nginx1 --type=ClusterIP --port=80 --target-port=80 
service/svc-nginx1 exposed

# 查看service
kubectl get svc svc-nginx1 -o wide
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE     SELECTOR
svc-nginx1   ClusterIP   10.109.179.231   <none>        80/TCP    3m51s   run=nginx

# 这里产生了一个CLUSTER-IP，这就是service的IP，在Service的生命周期中，这个地址是不会变动的
# 可以通过这个IP访问当前service对应的POD
curl 10.109.179.231:80
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
</head>
<body>
<h1>Welcome to nginx!</h1>
.......
</body>
</html>
```
### 4.5.2 创建集群外部也可访问的Service
```shell
# 上面创建的Service的type类型为ClusterIP，这个ip地址只用集群内部可访问
# 如果需要创建外部也可以访问的Service，需要修改type为NodePort
kubectl expose deploy nginx --name=svc-nginx2 --type=NodePort --port=80 --target-port=80 
service/svc-nginx2 exposed

# 此时查看，会发现出现了NodePort类型的Service，而且有一对Port（80:31928/TC）
kubectl get svc  svc-nginx2  -o wide
NAME          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE    SELECTOR
svc-nginx2    NodePort    10.100.94.0      <none>        80:31928/TCP   9s     run=nginx

# 接下来就可以通过集群外的主机访问 节点IP:31928访问服务了
# 例如在的电脑主机上通过浏览器访问下面的地址
http://10.0.0.101:31928/
```
### 4.5.3 删除Service
```shell
kubectl delete svc svc-nginx-1 
service "svc-nginx-1" deleted
```
### 4.5.4 配置方式
创建一个svc-nginx.yaml，内容如下：
```yml
apiVersion: v1
kind: Service
metadata:
  name: svc-nginx
spec:
  clusterIP: 10.109.179.231 # 固定svc的内网ip
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    run: nginx
  type: ClusterIP
```
然后就可以执行对应的创建和删除命令了：
创建：`kubectl create -f svc-nginx.yaml`
删除：`kubectl delete -f svc-nginx.yaml`
>  **小结** 至此，已经掌握了Namespace、Pod、Deployment、Service资源的基本操作，有了这些操作，就可以在kubernetes集群中实现一个服务的简单部署和访问了，但是如果想要更好的使用kubernetes，就需要深入学习这几种资源的细节和原理。
# 5. Pod详解
## 5.1 Pod介绍
### 5.1.1 Pod结构
![image-20200407121501907](https://weichengjun2.dpdns.org//i/2025/02/15/67b00981a987b.png) 
每个Pod中都可以包含一个或者多个容器，这些容器可以分为两类：
- 用户程序所在的容器，数量可多可少
- Pause容器，这是每个Pod都会有的一个 **根容器** ，它的作用有两个：
	- 可以以它为依据，评估整个Pod的健康状态
	- 可以在根容器上设置Ip地址，其它容器都用此Ip（Pod IP），以实现Pod内部的网路通信
		这里是Pod内部的通讯，Pod的之间的通讯采用虚拟二层网络技术来实现，我们当前环境用的是Flannel
### 5.1.2 Pod定义
资源清单文件
```yml
apiVersion: v1 # API 版本：指定 Kubernetes API 的版本，这里是 v1，是 Pod 的稳定版本
kind: Pod # Kubernetes 资源类型：Pod，Kubernetes 中最小的调度单元，可以包含一个或多个容器
metadata: # 元数据：描述 Pod 的信息，例如名称、命名空间、标签等
  name: my-pod # 必选，Pod 的名称：my-pod，必须符合命名规范，在同一命名空间下唯一
  namespace: default # Pod 所属的命名空间：default，默认为 default 命名空间。命名空间是 Kubernetes 中用于隔离资源的逻辑单元
  labels: # 自定义标签列表：用于组织和选择 Pod
    app: my-app # 示例标签：app=my-app，可以根据实际情况添加更多标签，方便管理和选择 Pod
spec: # 必选，Pod 规格：描述 Pod 的期望状态，例如包含哪些容器、使用哪些存储卷等
  containers: # 必选，Pod 中容器列表：Pod 可以包含一个或多个容器
    - name: my-container # 必选，容器名称：my-container，在一个 Pod 中必须唯一
      image: nginx:1.17.1 # 必选，容器的镜像名称：nginx:1.17.1，指定容器使用的镜像
      imagePullPolicy: IfNotPresent # 获取镜像的策略：IfNotPresent，如果本地不存在该镜像，则从镜像仓库拉取。其他可选值：Always（总是拉取）、Never（从不拉取）
      command: ["/bin/sh", "-c", "echo Hello Kubernetes!"] # 指定容器启动时执行命令，忽略 Docker 镜像中定义的 `ENTRYPOINT`，如果不指定，使用镜像打包时使用的启动命令。
      args: [] # 定义传递给 `command` 的参数列表，或在 `command` 为空时传递给 Docker 镜像的 `ENTRYPOINT`。
      workingDir: /app # 容器的工作目录：/app，指定容器内的工作目录
      volumeMounts: # 挂载到容器内部的存储卷配置：用于将存储卷挂载到容器内的指定路径
        - name: my-volume # 引用 Pod 定义的共享存储卷的名称：my-volume，需与 volumes[] 部分定义的卷名一致
          mountPath: /data # 存储卷在容器内 mount 的绝对路径：/data，将存储卷挂载到容器内的 /data 目录
          readOnly: false # 是否为只读模式：false，表示存储卷可以读写，默认为 false
      ports: # 需要暴露的端口号列表：用于将容器内的端口暴露到外部
        - name: http # 端口的名称：http
          containerPort: 80 # 容器需要监听的端口号：80，容器内部的端口
          hostPort: 8080 # 容器所在主机需要监听的端口号：8080，宿主机上的端口
          protocol: TCP # 端口协议：TCP，支持 TCP 和 UDP，默认为 TCP
      env: # 容器运行前需设置的环境变量列表：用于在容器启动前设置环境变量
        - name: MY_ENV_VAR # 环境变量名称：MY_ENV_VAR
          value: my-value # 环境变量的值：my-value
      resources: # 资源限制和请求的设置：用于限制容器使用的资源，并设置资源请求
        limits: # 资源限制的设置：限制容器可以使用的最大资源
          cpu: "1" # CPU 的限制：1 核
          memory: "512Mi" # 内存限制：512MiB
        requests: # 资源请求的设置：容器启动时请求的最小资源
          cpu: "0.5" # CPU 请求：0.5 核
          memory: "256Mi" # 内存请求：256MiB
      lifecycle: # 生命周期钩子：在容器生命周期的不同阶段执行的脚本
        postStart: # 容器启动后立即执行此钩子：用于在容器启动后执行一些初始化操作
          exec: # 使用 exec 方式执行命令
            command: ["/bin/sh", "-c", "echo Container started!"] # 启动后执行的命令
        preStop: # 容器终止前执行此钩子：用于在容器终止前执行一些清理操作
          exec: # 使用 exec 方式执行命令
            command: ["/bin/sh", "-c", "echo Container stopping!"] # 停止前执行的命令
      livenessProbe: # 对 Pod 内各容器健康检查的设置：用于检查容器是否健康，不健康则重启容器
        httpGet: # 使用 HTTPGet 方式进行健康检查
          path: /healthz # 健康检查的路径
          port: 80 # 健康检查的端口
        initialDelaySeconds: 15 # 容器启动完成后首次探测的时间：15 秒
        timeoutSeconds: 5 # 对容器健康检查探测等待响应的超时时间：5 秒
        periodSeconds: 10 # 对容器监控检查的定期探测时间设置：10 秒
        successThreshold: 1 # 连续几次探测成功才认为是健康
        failureThreshold: 3 # 连续几次探测失败才认为是不健康
      securityContext: # 安全上下文设置
        privileged: false # 是否以特权模式运行容器：false，不以特权模式运行
  restartPolicy: Always # Pod 的重启策略：Always，Pod 总是重启。其他可选值：Never（从不重启）、OnFailure（失败时重启）
  terminationGracePeriodSeconds: 30 # 终止宽限期：Pod 终止前等待的时间，单位为秒，用于容器优雅退出，默认为 30 秒。给容器内的进程时间来完成清理工作，例如保存数据、关闭连接等。
  nodeName: my-node # 设置 NodeName 表示将该 Pod 调度到指定名称的 node 节点上：my-node
  nodeSelector: # 设置 NodeSelector 表示将该 Pod 调度到包含这个 label 的 node 上
    diskType: ssd # 节点需要包含标签 diskType=ssd
  imagePullSecrets: # Pull 镜像时使用的 secret 名称：用于拉取私有镜像
    - name: my-registry-secret # 镜像仓库的 secret 名称
  hostNetwork: false # 是否使用主机网络模式：false，不使用主机网络模式。如果设置为 true，表示使用宿主机网络
  volumes: # 在该 pod 上定义共享存储卷列表：用于定义 Pod 中使用的存储卷
    - name: my-volume # 共享存储卷名称：my-volume
      emptyDir: {} # 类型为 emptyDir 的存储卷：emptyDir，与 Pod 同生命周期的一个临时目录。为空值表示使用 emptyDir 存储卷
      # hostPath: # 类型为 hostPath 的存储卷：hostPath，表示挂载 Pod 所在宿主机的目录
      #   path: /data # Pod 所在宿主机的目录，将被用于容器中 mount 的目录
      # secret: # 类型为 secret 的存储卷：secret，挂载集群与定义的 secret 对象到容器内部
      #   secretName: my-secret
      #   items:
      #     - key: my-key
      #       path: /etc/my-config
      # configMap: # 类型为 configMap 的存储卷：configMap，挂载预定义的 configMap 对象到容器内部
      #   name: my-configmap
      #   items:
      #     - key: my-key
      #       path: /etc/my-config
```

```shell
#小提示：
#   在这里，可通过一个命令来查看每种资源的可配置项
#   kubectl explain 资源类型         查看某种资源可以配置的一级属性
#   kubectl explain 资源类型.属性     查看属性的子属性
kubectl explain pod
KIND:     Pod
VERSION:  v1
FIELDS:
   apiVersion   <string>
   kind <string>
   metadata     <Object>
   spec <Object>
   status       <Object>

kubectl explain pod.metadata
KIND:     Pod
VERSION:  v1
RESOURCE: metadata <Object>
FIELDS:
   annotations  <map[string]string>
   clusterName  <string>
   creationTimestamp    <string>
   deletionGracePeriodSeconds   <integer>
   deletionTimestamp    <string>
   finalizers   <[]string>
   generateName <string>
   generation   <integer>
   labels       <map[string]string>
   managedFields        <[]Object>
   name <string>
   namespace    <string>
   ownerReferences      <[]Object>
   resourceVersion      <string>
   selfLink     <string>
   uid  <string>

```
## 5.2 Pod配置

本小节主要来研究 `pod.spec.containers` 属性，这也是pod配置中最为关键的一项配置。
### 5.2.1 基本配置

创建`pod-base.yaml`文件，内容如下：
```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-base
  labels:
    user: riven
spec:
  containers:
    - name: nginx
      image: nginx:1.17.1
    - name: busybox
      image: busybox:1.28
```
上面定义了一个比较简单Pod的配置，里面有两个容器：
- nginx：用1.17.1版本的nginx镜像创建，（nginx是一个轻量级web容器）
- busybox：用1.30版本的busybox镜像创建，（busybox是一个小巧的linux命令集合）
```shell
# 创建Pod
kubectl apply -f pod-base.yaml
pod/pod-base created

# 查看Pod状况
# READY 1/2 : 表示当前Pod中有2个容器，其中1个准备就绪，1个未就绪
# RESTARTS  : 重启次数，因为有1个容器故障了，Pod一直在重启试图恢复它
kubectl get pod
NAME       READY   STATUS    RESTARTS   AGE
pod-base   1/2     Running   4          95s

# 可以通过describe查看内部的详情
# 此时已经运行起来了一个基本的Pod，虽然它暂时有问题
kubectl describe pod pod-base
```
### 5.2.2 镜像拉取
创建`pod-imagepullpolicy.yaml`文件，内容如下：
```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-imagepullpolicy
spec:
  containers:
    - name: nginx
      image: nginx:1.17.1
      imagePullPolicy: Never # 用于设置镜像拉取策略
    - name: busybox
      image: busybox:1.28
```
imagePullPolicy，用于设置镜像拉取策略，kubernetes支持配置三种拉取策略：
- Always：总是从远程仓库拉取镜像（一直远程下载）
- IfNotPresent：本地有则使用本地镜像，本地没有则从远程仓库拉取镜像（本地有就本地 本地没远程下载）
- Never：只使用本地镜像，从不去远程仓库拉取，本地没有就报错 （一直使用本地）

> 默认值说明：
> 如果镜像tag为具体版本号， 默认策略是：IfNotPresent
> 如果镜像tag为：latest（最终版本） ，默认策略是always

```shell
# 创建Pod
kubectl create -f pod-imagepullpolicy.yaml
pod/pod-imagepullpolicy created

# 查看Pod详情
# 此时明显可以看到nginx镜像有一步Pulling image "nginx:1.17.1"的过程
kubectl describe pod pod-imagepullpolicy 
......
Events:
  Type     Reason     Age               From               Message
  ----     ------     ----              ----               -------
  Normal   Scheduled  <unknown>         default-scheduler  Successfully assigned dev/pod-imagePullPolicy to k8s-node1
  Normal   Pulling    32s               kubelet, k8s-node1     Pulling image "nginx:1.17.1"
  Normal   Pulled     26s               kubelet, k8s-node1     Successfully pulled image "nginx:1.17.1"
  Normal   Created    26s               kubelet, k8s-node1     Created container nginx
  Normal   Started    25s               kubelet, k8s-node1     Started container nginx
  Normal   Pulled     7s (x3 over 25s)  kubelet, k8s-node1     Container image "busybox:1.28" already present on machine
  Normal   Created    7s (x3 over 25s)  kubelet, k8s-node1     Created container busybox
  Normal   Started    7s (x3 over 25s)  kubelet, k8s-node1     Started container busybox

```
### 5.2.3 启动命令
在前面的案例中，一直有一个问题没有解决，就是的busybox容器一直没有成功运行，那么到底是什么原因导致这个容器的故障呢？

原来busybox并不是一个程序，而是类似于一个工具类的集合，kubernetes集群启动管理后，它会自动关闭。解决方法就是让其一直在运行，这就用到了command配置。

创建`pod-command.yaml`文件，内容如下：
```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-command
spec:
  containers:
    - name: nginx
      image: nginx:1.17.1
    - name: busybox
      image: busybox:1.28
      command: [ "/bin/sh","-c","touch /tmp/hello.txt;while true;do /bin/echo $(date +%T) >> /tmp/hello.txt; sleep 3; done;" ]
```
command，用于在pod中的容器初始化完毕之后运行一个命令。

> 稍微解释下上面命令的意思：
> “/bin/sh”,“-c”, 使用sh执行命令
> touch /tmp/hello.txt; 创建一个/tmp/hello.txt 文件
> while true;do /bin/echo $(date +%T) >> /tmp/hello.txt; sleep 3; done; 每隔3秒向文件中写入当前时间

```shell
# 创建Pod
kubectl create  -f pod-command.yaml
pod/pod-command created

# 查看Pod状态
# 此时发现两个pod都正常运行了
kubectl get pods pod-command
NAME          READY   STATUS   RESTARTS   AGE
pod-command   2/2     Runing   0          2s

# 进入pod中的busybox容器，查看文件内容
# 补充一个命令: kubectl exec  pod名称 -n 命名空间 -it -c 容器名称 /bin/sh  在容器内部执行命令
# 使用这个命令就可以进入某个容器的内部，然后进行相关操作了
# 比如，可以查看txt文件的内容
kubectl exec pod-command -it -c busybox -- tail -f /tmp/hello.txt
14:44:19
14:44:22
14:44:25
```

>特别说明：
    通过上面发现command已经可以完成启动命令和传递参数的功能，为什么这里还要提供一个args选项，用于传递参数呢?这其实跟docker有点关系，kubernetes中的command、args两项其实是实现覆盖Dockerfile中ENTRYPOINT的功能。
	 1.如果command和args均没有写，那么用Dockerfile的配置。
	 2.如果command写了，但args没有写，那么Dockerfile默认的配置会被忽略，执行输入的command
	 3.如果command没写，但args写了，那么Dockerfile中配置的ENTRYPOINT的命令会被执行，使用当前args的参数
	 4.如果command和args都写了，那么Dockerfile的配置被忽略，执行command并追加上args参数
### 5.2.4 环境变量
创建`pod-env.yaml`文件，内容如下：
```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-env
spec:
  containers:
    - name: busybox
      image: busybox:1.28
      command: [ "/bin/sh","-c","while true;do /bin/echo $(date +%T);sleep 60; done;" ]
      env: # 设置环境变量列表
        - name: "username"
          value: "admin"
        - name: "password"
          value: "123456"
```

env，环境变量，用于在pod中的容器设置环境变量。
```shell
# 创建Pod
kubectl create -f pod-env.yaml
pod/pod-env created

# 进入容器，输出环境变量
kubectl exec pod-env -c busybox -it /bin/sh
# echo $username
admin
# echo $password
123456
```

这种方式不是很推荐，推荐将这些配置单独存储在配置文件中，这种方式将在后面介绍。
### 5.2.5 端口设置
本小节来介绍容器的端口设置，也就是containers的ports选项。
首先看下ports支持的子选项：
```shell
kubectl explain pod.spec.containers.ports
KIND:     Pod
VERSION:  v1
RESOURCE: ports <[]Object>
FIELDS:
   name         <string>  # 端口名称，如果指定，必须保证name在pod中是唯一的		
   containerPort<integer> # 容器要监听的端口(0<x<65536)
   hostPort     <integer> # 容器要在主机上公开的端口，如果设置，主机上只能运行容器的一个副本(一般省略) 
   hostIP       <string>  # 要将外部端口绑定到的主机IP(一般省略)
   protocol     <string>  # 端口协议。必须是UDP、TCP或SCTP。默认为“TCP”。
```
接下来，编写一个测试案例，创建`pod-ports.yaml`
```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-ports
spec:
  containers:
    - name: nginx
      image: nginx:1.17.1
      ports: # 设置容器暴露的端口列表
        - name: nginx-port
          containerPort: 80
          protocol: TCP
```

```yml
# 创建Pod
kubectl create -f pod-ports.yaml
pod/pod-ports created

# 查看pod
# 在下面可以明显看到配置信息
kubectl get pod pod-ports -o yaml
......
spec:
  containers:
  - image: nginx:1.17.1
    imagePullPolicy: IfNotPresent
    name: nginx
    ports:
    - containerPort: 80
      name: nginx-port
      protocol: TCP
......

```
访问容器中的程序需要使用的是 `Podip:containerPort` 
### 5.2.6 资源配额
容器中的程序要运行，肯定是要占用一定资源的，比如cpu和内存等，如果不对某个容器的资源做限制，那么它就可能吃掉大量资源，导致其它容器无法运行。针对这种情况，kubernetes提供了对内存和cpu的资源进行配额的机制，这种机制主要通过resources选项实现，他有两个子选项：
- limits：用于限制运行时容器的最大占用资源，当容器占用资源超过limits时会被终止，并进行重启
- requests ：用于设置容器需要的最小资源，如果环境资源不够，容器将无法启动

可以通过上面两个选项设置资源的上下限。
接下来，编写一个测试案例，创建pod-resources.yaml
```yml
apiVersion: v1  # API 版本，这里使用的是 v1
kind: Pod       # 定义这是一个 Pod 资源
metadata:
  name: pod-resources  # Pod 的名称为 pod-resources
spec:
  containers:
    - name: nginx  # 容器的名称为 nginx
      image: nginx:1.17.1  # 使用的镜像为 nginx，版本为 1.17.1
      resources: # 定义容器的资源请求和限制
        requests: # 定义容器启动时所需的最小资源量
          cpu: "1"  # 请求 1 核 CPU
          memory: "10Mi"  # 请求 10 MiB 内存
        limits: # 定义容器可以使用的最大资源量
          cpu: "2"  # CPU 使用上限为 2 核
          memory: "1Gi"  # 内存使用上限为 1 GiB
```
在这对cpu和memory的单位做一个说明：
- cpu：core数，可以为整数或小数
- memory： 内存大小，可以使用Gi、Mi、G、M等形式
```shell
# 运行Pod
kubectl create  -f pod-resources.yaml
pod/pod-resources created

# 查看发现pod运行正常
kubectl get pod pod-resources 
NAME            READY   STATUS    RESTARTS   AGE  
pod-resources   1/1     Running   0          39s   

# 接下来，停止Pod
kubectl delete  -f pod-resources.yaml
pod "pod-resources" deleted

# 编辑pod，修改resources.requests.memory的值为1Gi
vim pod-resources.yaml
# 删除之前的pod
kubectl delete -f pod-resources.yaml
# 再次启动pod
kubectl create  -f pod-resources.yaml
pod/pod-resources created

# 查看Pod状态，发现Pod启动失败
kubectl get pod pod-resources -o wide
NAME            READY   STATUS    RESTARTS   AGE          
pod-resources   0/1     Pending   0          20s    

# 查看pod详情会发现，如下提示
kubectl describe pod pod-resources 
......
Warning  FailedScheduling  35s   default-scheduler  0/3 nodes are available: 1 node(s) had taint {node-role.kubernetes.io/master: }, that the pod didn't tolerate, 2 Insufficient memory.(内存不足)

```
## 5.3 Pod生命周期

我们一般将pod对象从创建至终的这段时间范围称为pod的生命周期，它主要包含下面的过程：
- pod创建过程
- 运行初始化容器（init container）过程
- 运行主容器（main container）
	- 容器启动后钩子（post start）、容器终止前钩子（pre stop）
	- 容器的存活性探测（liveness probe）、就绪性探测（readiness probe）

- pod终止过程
![image-20200412111402706](https://weichengjun2.dpdns.org//i/2025/02/15/67b00981f1602.png) 
在整个生命周期中，Pod会出现5种 **状态** （ **相位** ），分别如下：
- 挂起（Pending）：apiserver已经创建了pod资源对象，但它尚未被调度完成或者仍处于下载镜像的过程中
- 运行中（Running）：pod已经被调度至某节点，并且所有容器都已经被kubelet创建完成
- 成功（Succeeded）：pod中的所有容器都已经成功终止并且不会被重启
- 失败（Failed）：所有容器都已经终止，但至少有一个容器终止失败，即容器返回了非0值的退出状态
- 未知（Unknown）：apiserver无法正常获取到pod对象的状态信息，通常由网络通信失败所导致
### 5.3.1 创建和终止
 **pod的创建过程** 
1.  用户通过kubectl或其他api客户端提交需要创建的pod信息给apiServer
2.  apiServer开始生成pod对象的信息，并将信息存入etcd，然后返回确认信息至客户端
3.  apiServer开始反映etcd中的pod对象的变化，其它组件使用watch机制来跟踪检查apiServer上的变动
4.  scheduler发现有新的pod对象要创建，开始为Pod分配主机并将结果信息更新至apiServer
5.  node节点上的kubelet发现有pod调度过来，尝试调用docker启动容器，并将结果回送至apiServer
6.  apiServer将接收到的pod状态信息存入etcd中
![image-20200406184656917](https://weichengjun2.dpdns.org//i/2025/02/15/67b009826cff8.png) 

 **pod的终止过程** 
1.  用户向apiServer发送删除pod对象的命令
2.  apiServcer中的pod对象信息会随着时间的推移而更新，在宽限期内（默认30s，可以设置terminationGracePeriodSeconds=40s进行控制），pod被视为dead
3.  将pod标记为terminating状态
4.  kubelet在监控到pod对象转为terminating状态的同时启动pod关闭过程
5.  端点控制器监控到pod对象的关闭行为时将其从所有匹配到此端点的service资源的端点列表中移除
6.  如果当前pod对象定义了preStop钩子处理器，则在其标记为terminating后即会以同步的方式启动执行
7.  pod对象中的容器进程收到停止信号
8.  宽限期结束后，若pod中还存在仍在运行的进程，那么pod对象会收到立即终止的信号
9.  kubelet请求apiServer将此pod资源的宽限期设置为0从而完成删除操作，此时pod对于用户已不可见
### 5.3.2 初始化容器
初始化容器是在pod的主容器启动之前要运行的容器，主要是做一些主容器的前置工作，它具有两大特征：
1.  初始化容器必须运行完成直至结束，若某初始化容器运行失败，那么kubernetes需要重启它直到成功完成
2.  初始化容器必须按照定义的顺序执行，当且仅当前一个成功之后，后面的一个才能运行

初始化容器有很多的应用场景，下面列出的是最常见的几个：
- 提供主容器镜像中不具备的工具程序或自定义代码
- 初始化容器要先于应用容器串行启动并运行完成，因此可用于延后应用容器的启动直至其依赖的条件得到满足

>相对于 postStart 来说，首先 InitController 能够保证一定在 EntryPoint 之前执行，而 postStart 不能，其次 postStart 更适合去执行一些命令操作，而 InitController 实际就是一个容器，可以在其他基础容器环境下执行更复杂的初始化功能。  

接下来做一个案例，模拟下面这个需求：
假设要以主容器来运行nginx，但是要求在运行nginx之前先要能够连接上mysql和redis所在服务器
为了简化测试，事先规定好mysql `(10.0.0.102)` 和redis `(10.0.0.103)` 服务器的地址
创建`pod-initcontainer.yaml`，内容如下：
```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-initcontainer
spec:
  containers:
  - name: main-container
    image: nginx:1.17.1
    ports: 
    - name: nginx-port
      containerPort: 80
  initContainers:
  - name: test-mysql
    image: busybox:1.28
    command: ['sh', '-c', 'until ping 10.0.0.102 -c 1 ; do echo waiting for mysql...; sleep 2; done;']
  - name: test-redis
    image: busybox:1.28
    command: ['sh', '-c', 'until ping 10.0.0.103 -c 1 ; do echo waiting for reids...; sleep 2; done;']
```
执行命令
```shell
# 创建pod
kubectl create -f pod-initcontainer.yaml
pod/pod-initcontainer created

# 查看pod状态
# 发现pod卡在启动第一个初始化容器过程中，后面的容器不会运行
kubectl describe pod pod-initcontainer 
........
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  49s   default-scheduler  Successfully assigned dev/pod-initcontainer to k8s-node1
  Normal  Pulled     48s   kubelet, k8s-node1     Container image "busybox:1.28" already present on machine
  Normal  Created    48s   kubelet, k8s-node1     Created container test-mysql
  Normal  Started    48s   kubelet, k8s-node1     Started container test-mysql

# 动态查看pod
kubectl get pods pod-initcontainer -w
NAME                             READY   STATUS     RESTARTS   AGE
pod-initcontainer                0/1     Init:0/2   0          15s
pod-initcontainer                0/1     Init:1/2   0          52s
pod-initcontainer                0/1     Init:1/2   0          53s
pod-initcontainer                0/1     PodInitializing   0          89s
pod-initcontainer                1/1     Running           0          90s

# 接下来新开一个shell，为当前服务器新增两个ip，观察pod的变化
ifconfig ens33:1 10.0.0.102 netmask 255.255.255.0 up
ifconfig ens33:2 10.0.0.103 netmask 255.255.255.0 up
```
### 5.3.3 钩子函数
钩子函数能够感知自身生命周期中的事件，并在相应的时刻到来时运行用户指定的程序代码。
kubernetes在主容器的启动之后和停止之前提供了两个钩子函数：
- post start：容器创建之后执行，如果失败了会重启容器
- pre stop ：容器终止之前执行，执行完成之后容器将成功终止，在其完成之前会阻塞删除容器的操作

钩子处理器支持使用下面三种方式定义动作：
- `exec`命令：在容器内执行一次命令
```yml
……
  lifecycle:
    postStart: 
      exec:
        command:
        - cat
        - /tmp/healthy
……
```
- TCPSocket：在当前容器尝试访问指定的socket
```yml
……      
  lifecycle:
    postStart:
      tcpSocket:
        port: 8080
……
```
- HTTPGet：在当前容器中向某url发起http请求
```yml
……
  lifecycle:
    postStart:
      httpGet:
        path: / #URI地址
        port: 80 #端口号
        host: 10.0.0.101 #主机地址
        scheme: HTTP #支持的协议，http或者https
……
```

接下来，以exec方式为例，演示下钩子函数的使用，创建pod-hook-exec.yaml文件，内容如下：
```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-hook-exec
spec:
  containers:
  - name: main-container
    image: nginx:1.17.1
    ports:
    - name: nginx-port
      containerPort: 80
    lifecycle:
      postStart: 
        exec: # 在容器启动的时候执行一个命令，修改掉nginx的默认首页内容
          command: ["/bin/sh", "-c", "echo postStart... > /usr/share/nginx/html/index.html"]
      preStop:
        exec: # 在容器停止之前停止nginx服务
          command: ["/usr/sbin/nginx","-s","quit"]
```
执行命令
```shell
# 创建pod
kubectl create -f pod-hook-exec.yaml
pod/pod-hook-exec created

# 查看pod
kubectl get pods pod-hook-exec -o wide
NAME           READY   STATUS     RESTARTS   AGE    IP            NODE    
pod-hook-exec  1/1     Running    0          29s    10.244.2.48   k8s-node2   

# 访问pod
curl 10.244.2.48
postStart...
```
### 5.3.4 容器探测
容器探测用于检测容器中的应用实例是否正常工作，是保障业务可用性的一种传统机制。如果经过探测，实例的状态不符合预期，那么kubernetes就会把该问题实例" 摘除 "，不承担业务流量。kubernetes提供了三种探针来实现容器探测，分别是：
- startupProbe：启动探针，k8s 1.16 版本新增的探针，用于判断应用程序是否已经启动了。当配置了 startupProbe 后，会先禁用其他探针，直到 startupProbe 成功后，其他探针才会继续。  
	>作用：由于有时候不能准确预估应用一定是多长时间启动成功，因此配置另外两种方式不方便配置初始化时长来检测，而配置了 statupProbe 后，只有在应用启动成功了，才会执行另外两种探针，可以更加方便的结合使用另外两种探针使用。
- livenessProbe：存活探针，用于检测应用实例当前是否处于正常运行状态，如果不是，k8s会重启容器
	> livenessProbe 决定是否重启容器。
- readinessProbe：就绪探针，用于检测应用实例当前是否可以接收请求，如果不能，k8s不会转发流量
	>readinessProbe 决定是否将请求转发给容器。

上面两种探针目前均支持三种探测方式：
- Exec命令：在容器内执行一次命令，如果命令执行的退出码为0，则认为程序正常，否则不正常
```yml
……
  livenessProbe:
    exec:
      command:
      - cat
      - /tmp/healthy
……
```
- TCPSocket：将会尝试访问一个用户容器的端口，如果能够建立这条连接，则认为程序正常，否则不正常
```yml
……      
  livenessProbe:
    tcpSocket:
      port: 8080
……
```
- HTTPGet：调用容器内Web应用的URL，如果返回的状态码在200和399之间，则认为程序正常，否则不正常
```yml
……
  livenessProbe:
    httpGet:
      path: / #URI地址
      port: 80 #端口号
      host: 127.0.0.1 #主机地址
      scheme: HTTP #支持的协议，http或者https
……
```
创建pod-probe.yaml
```yml
apiVersion: v1 # api 文档版本
kind: Pod # 资源对象类型，也可以配置为像Deployment、StatefulSet这样的资源对象类型
metadata: # 资源对象的元数据
  name: pod-probe # 资源对象的名称
spec: # 资源对象的配置
  containers: # 对于Pod中的容器描述
    - name: nginx # 容器的名称
      image: nginx:1.17.1 # 容器的镜像
      ports: # 容器的端口
        - name: http # 端口的名称
          containerPort: 80 # 容器的端口号
      workingDir: /usr/share/nginx/html # 容器的工作目录
      startupProbe: # 启动探针
        exec: # exec探针
          command: [ "sh", "-c", "sleep 3s; echo success > /initialized" ] # 执行命令
        #        httpGet: # httpGet探针
        #          path: /index.html # 请求路径
        #          port: 80 # 请求端口
        #        tcpSocket: # tcpSocket探针
        #          port: 80 # 请求端口
        initialDelaySeconds: 10 # 容器启动后等待多少秒执行第一次探测
        periodSeconds: 10 # 执行探测的频率。默认是10秒，最小1秒
        timeoutSeconds: 5 # 探测超时时间。默认1秒，最小1秒
        failureThreshold: 5 # 失败阈值，超过5次失败，则认为容器不健康。 默认是3，最小值是1
        successThreshold: 1 # 成功阈值，超过1次成功，则认为容器健康
      livenessProbe: # 存活探针
        httpGet: # httpGet探针
          path: started.html # 请求路径
          port: 80 # 请求端口
        initialDelaySeconds: 10 # 容器启动后等待多少秒执行第一次探测
        periodSeconds: 10 # 执行探测的频率。默认是10秒，最小1秒
        timeoutSeconds: 5 # 探测超时时间。默认1秒，最小1秒
        failureThreshold: 5 # 失败阈值，超过5次失败，则认为容器不健康。 默认是3，最小值是1
        successThreshold: 1 # 成功阈值，超过1次成功，则认为容器健康
      readinessProbe: # 就绪探针
        httpGet:
          path: started.html
          port: 80
        initialDelaySeconds: 10 # 容器启动后等待多少秒执行第一次探测
        periodSeconds: 10 # 执行探测的频率。默认是10秒，最小1秒
        timeoutSeconds: 5 # 探测超时时间。默认1秒，最小1秒
        failureThreshold: 5 # 失败阈值，超过5次失败，则认为容器不健康。 默认是3，最小值是1
        successThreshold: 1 # 成功阈值，超过1次成功，则认为容器健康
```
创建pod，观察效果
```shell
# 创建Pod
kubectl create -f pod-probe.yaml
pod/pod-probe created

# 查看Pod详情
kubectl describe pods pod-probe
......
Events:
  Type     Reason     Age               From               Message
  ----     ------     ----              ----               -------
  Normal   Scheduled  47s               default-scheduler  Successfully assigned default/pod-probe to k8s-node2
  Normal   Pulled     45s               kubelet            Container image "nginx:1.17.1" already present on machine
  Normal   Created    44s               kubelet            Created container nginx
  Normal   Started    44s               kubelet            Started container nginx
  Warning  Unhealthy  7s (x3 over 23s)  kubelet            Readiness probe failed: HTTP probe failed with statuscode: 404
  Warning  Unhealthy  7s (x2 over 17s)  kubelet            Liveness probe failed: HTTP probe failed with statuscode: 404
  
# 观察上面的信息就会发现nginx容器启动之后就进行了健康检查
# 检查失败之后，容器被kill掉，然后尝试进行重启（这是重启策略的作用，后面讲解）
# 稍等一会之后，再观察pod信息，就可以看到RESTARTS不再是0，而是一直增长
kubectl get pods pod-probe
NAME        READY   STATUS    RESTARTS   AGE
pod-probe   0/1     Running   0          62s

# 接下来创建started.html文件并且从本地拷贝文件到容器里 /usr/share/nginx/html/started.html
echo 'started' >> started.html
kubectl cp started.html pod-probe:/usr/share/nginx/html
# 再试，结果就正常了......
kubectl get po pod-probe        
NAME        READY   STATUS    RESTARTS   AGE
pod-probe   1/1     Running   0          2m21s
```

至此，已经使用liveness Probe演示了三种探测方式，但是查看livenessProbe的子属性，会发现除了这三种方式，还有一些其他的配置，在这里一并解释下：
```shell
kubectl explain pod.spec.containers.livenessProbe
FIELDS:
   exec <Object>  
   tcpSocket    <Object>
   httpGet      <Object>
   initialDelaySeconds  <integer>  # 容器启动后等待多少秒执行第一次探测
   timeoutSeconds       <integer>  # 探测超时时间。默认1秒，最小1秒
   periodSeconds        <integer>  # 执行探测的频率。默认是10秒，最小1秒
   failureThreshold     <integer>  # 连续探测失败多少次才被认定为失败。默认是3。最小值是1
   successThreshold     <integer>  # 连续探测成功多少次才被认定为成功。默认是1
```
### 5.3.5 重启策略
在上一节中，一旦容器探测出现了问题，kubernetes就会对容器所在的Pod进行重启，其实这是由pod的重启策略决定的，pod的重启策略有 3 种，分别如下：
- Always ：容器失效时，自动重启该容器，这也是默认值。
- OnFailure ： 容器终止运行且退出码不为0时重启
- Never ： 不论状态为何，都不重启该容器

重启策略适用于pod对象中的所有容器，首次需要重启的容器，将在其需要时立即进行重启，随后再次需要重启的操作将由kubelet延迟一段时间后进行，且反复的重启操作的延迟时长以此为10s、20s、40s、80s、160s和300s，300s是最大延迟时长。
创建`pod-restartpolicy.yaml`
```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-restartpolicy
spec:
  containers:
  - name: nginx
    image: nginx:1.17.1
    ports:
    - name: nginx-port
      containerPort: 80
    livenessProbe:
      httpGet:
        scheme: HTTP
        port: 80
        path: /hello
  restartPolicy: Never # 设置重启策略为Never
```
运行Pod测试
```shell
# 创建Pod
kubectl create -f pod-restartpolicy.yaml
pod/pod-restartpolicy created

# 查看Pod详情，发现nginx容器失败
kubectl describe pods pod-restartpolicy
......
  Warning  Unhealthy  15s (x3 over 35s)  kubelet, k8s-node1     Liveness probe failed: HTTP probe failed with statuscode: 404
  Normal   Killing    15s                kubelet, k8s-node1     Container nginx failed liveness probe
  
# 多等一会，再观察pod的重启次数，发现一直是0，并未重启   
kubectl get pods pod-restartpolicy
NAME                   READY   STATUS    RESTARTS   AGE
pod-restartpolicy      0/1     Running   0          5min42s
```
## 5.4 Pod调度

在默认情况下，一个Pod在哪个Node节点上运行，是由Scheduler组件采用相应的算法计算出来的，这个过程是不受人工控制的。但是在实际使用中，这并不满足的需求，因为很多情况下，我们想控制某些Pod到达某些节点上，那么应该怎么做呢？这就要求了解kubernetes对Pod的调度规则，kubernetes提供了四大类调度方式：
- 自动调度：运行在哪个节点上完全由Scheduler经过一系列的算法计算得出
- 定向调度：NodeName、NodeSelector
- 亲和性调度：NodeAffinity、PodAffinity、PodAntiAffinity
- 污点（容忍）调度：Taints、Toleration
### 5.4.1 定向调度

定向调度，指的是利用在pod上声明nodeName或者nodeSelector，以此将Pod调度到期望的node节点上。注意，这里的调度是强制的，这就意味着即使要调度的目标Node不存在，也会向上面进行调度，只不过pod运行失败而已。
#### NodeName
NodeName用于强制约束将Pod调度到指定的Name的Node节点上。这种方式，其实是直接跳过Scheduler的调度逻辑，直接将Pod调度到指定名称的节点。
接下来，实验一下：创建`pod-nodename.yaml`
```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-nodename  # Pod 的名称，用于标识这个 Pod
spec:
  containers:
    - name: nginx       # 容器的名称
      image: nginx:1.17.1  # 容器使用的镜像，这里使用的是 Nginx 1.17.1 版本
  nodeName: k8s-node1   # 指定这个 Pod 应该调度到名为 "k8s-node1" 的节点上
```
执行命令
```shell
# 创建Pod
kubectl create -f pod-nodename.yaml
pod/pod-nodename created

# 查看Pod调度到NODE属性，确实是调度到了k8s-node1节点上
kubectl get po pod-nodename -o wide
NAME           READY   STATUS    RESTARTS   AGE   IP             NODE        NOMINATED NODE   READINESS GATES
pod-nodename   1/1     Running   0          24s   10.244.36.80   k8s-node1   <none>           <none>

# 接下来，删除pod，修改nodeName的值为k8s-node3（并没有k8s-node3节点），然后创建pod
kubectl delete -f pod-nodename.yaml
vim pod-nodename.yaml
kubectl create -f pod-nodename.yaml
pod/pod-nodename created

# 再次查看，发现已经向Node3节点调度，但是由于不存在k8s-node3节点，所以pod无法正常运行
kubectl get po pod-nodename -o wide
NAME           READY   STATUS    RESTARTS   AGE   IP       NODE        NOMINATED NODE   READINESS GATES
pod-nodename   0/1     Pending   0          20s   <none>   k8s-node3   <none>           <none>
```
#### NodeSelector
NodeSelector用于将pod调度到添加了指定标签的node节点上。它是通过kubernetes的label-selector机制实现的，也就是说，在pod创建之前，会由scheduler使用MatchNodeSelector调度策略进行label匹配，找出目标node，然后将pod调度到目标节点，该匹配规则是强制约束。

接下来，实验一下：
1. 首先分别为node节点添加标签
```shell
kubectl label nodes k8s-node1 env=pro
kubectl label nodes k8s-node2 env=test
```
2. 创建一个`pod-nodeselector.yaml`文件，并使用它创建Pod
```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-nodeselector  # Pod 的名称，用于标识这个 Pod
spec:
  containers:
    - name: nginx       # 容器的名称
      image: nginx:1.17.1  # 容器使用的镜像，这里使用的是 Nginx 1.17.1 版本
  nodeSelector:
    env: pro         # 节点选择器，指定 Pod 应该调度到具有 "env=pro" 标签的节点上
```
运行命令
```shell
# 创建Pod
kubectl create -f pod-nodeselector.yaml
pod/pod-nodeselector created

# 查看Pod调度到NODE属性，确实是调度到了node1节点上
kubectl get pods pod-nodeselector -o wide
NAME               READY   STATUS    RESTARTS   AGE     IP          NODE    ......
pod-nodeselector   1/1     Running   0          47s   10.244.1.87   node1   ......

# 接下来，删除pod，修改nodeSelector的值为env: xxxx（不存在打有此标签的节点）
kubectl delete -f pod-nodeselector.yaml
pod "pod-nodeselector" deleted

vim pod-nodeselector.yaml
kubectl create -f pod-nodeselector.yaml
pod/pod-nodeselector created

# 再次查看，发现pod无法正常运行,Node的值为none
kubectl get pods -o wide
NAME               READY   STATUS    RESTARTS   AGE     IP       NODE    
pod-nodeselector   0/1     Pending   0          2m20s   <none>   <none>

# 查看详情,发现node selector匹配失败的提示
kubectl describe pods pod-nodeselector
.......
Events:
  Type     Reason            Age        From               Message
  ----     ------            ----       ----               -------
  Warning  FailedScheduling  <unknown>  default-scheduler  0/3 nodes are available: 3 node(s) didn't match node selector.

```
### 5.4.2 亲和性调度
上一节，介绍了两种定向调度的方式，使用起来非常方便，但是也有一定的问题，那就是如果没有满足条件的Node，那么Pod将不会被运行，即使在集群中还有可用Node列表也不行，这就限制了它的使用场景。

基于上面的问题，kubernetes还提供了一种亲和性调度（Affinity）。它在NodeSelector的基础之上的进行了扩展，可以通过配置的形式，实现优先选择满足条件的Node进行调度，如果没有，也可以调度到不满足条件的节点上，使调度更加灵活。

Affinity主要分为三类：
- nodeAffinity(node亲和性）: 以node为目标，解决pod可以调度到哪些node的问题
- podAffinity(pod亲和性) : 以pod为目标，解决pod可以和哪些已存在的pod部署在同一个拓扑域中的问题
- podAntiAffinity(pod反亲和性) : 以pod为目标，解决pod不能和哪些已存在pod部署在同一个拓扑域中的问题

> 关于亲和性(反亲和性)使用场景的说明： 
> **亲和性** ：如果两个应用频繁交互，那就有必要利用亲和性让两个应用的尽可能的靠近，这样可以减少因网络通信而带来的性能损耗。 
> **反亲和性** ：当应用的采用多副本部署时，有必要采用反亲和性让各个应用实例打散分布在各个node上，这样可以提高服务的高可用性。
#### NodeAffinity
##### `NodeAffinity` 的可配置项
```yml
pod.spec.affinity.nodeAffinity
  requiredDuringSchedulingIgnoredDuringExecution  Node节点必须满足指定的所有规则才可以，相当于硬限制
    nodeSelectorTerms  节点选择列表
      matchFields   按节点字段列出的节点选择器要求列表
      matchExpressions   按节点标签列出的节点选择器要求列表(推荐)
        key    键
        values 值
        operat or 关系符 支持Exists, DoesNotExist, In, NotIn, Gt, Lt
  preferredDuringSchedulingIgnoredDuringExecution 优先调度到满足指定的规则的Node，相当于软限制 (倾向)
    preference   一个节点选择器项，与相应的权重相关联
      matchFields   按节点字段列出的节点选择器要求列表
      matchExpressions   按节点标签列出的节点选择器要求列表(推荐)
        key    键
        values 值
        operator 关系符 支持In, NotIn, Exists, DoesNotExist, Gt, Lt
	weight 倾向权重，在范围1-100。
```
关系符的使用说明:
```yml
- matchExpressions:
  - key: env              # 匹配存在标签的key为env的节点
    operator: Exists
  - key: env              # 匹配标签的key为env,且value是"xxx"或"yyy"的节点
    operator: In
    values: ["xxx","yyy"]
  - key: env              # 匹配标签的key为env,且value大于"xxx"的节点
    operator: Gt
    values: "xxx"
```
##### `requiredDuringSchedulingIgnoredDuringExecution`  示例
创建`pod-nodeaffinity-required.yaml`
```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-nodeaffinity-required  # Pod 的名称，用于标识这个 Pod
spec:
  containers:
    - name: nginx       # 容器的名称
      image: nginx:1.17.1  # 容器使用的镜像，这里使用的是 Nginx 1.17.1 版本
  affinity: # 亲和性设置，用于控制 Pod 的调度位置
    nodeAffinity: # 节点亲和性，用于控制 Pod 应该调度到哪些节点上
      requiredDuringSchedulingIgnoredDuringExecution: # 硬性要求，Pod 在调度时必须满足，运行时可以忽略
        nodeSelectorTerms:
          - matchExpressions: # 匹配表达式，用于匹配节点的标签
              - key: env  # 匹配节点的 "env" 标签
                operator: In  # 操作符，表示匹配的值必须在指定列表中
                values: [ "xxx","yyy" ]  # 匹配的值，即 "env" 标签的值必须是 "xxx" 或 "yyy"
```

```shell
# 创建pod
kubectl create -f pod-nodeaffinity-required.yaml
pod/pod-nodeaffinity-required created

# 查看pod状态 （运行失败）
kubectl get pods pod-nodeaffinity-required -o wide
NAME                        READY   STATUS    RESTARTS   AGE   IP       NODE    ...... 
pod-nodeaffinity-required   0/1     Pending   0          16s   <none>   <none>  ......

# 查看Pod的详情
# 发现调度失败，提示node选择失败
kubectl describe pod pod-nodeaffinity-required
......
  Warning  FailedScheduling  <unknown>  default-scheduler  0/3 nodes are available: 3 node(s) didn't match node selector.
  Warning  FailedScheduling  <unknown>  default-scheduler  0/3 nodes are available: 3 node(s) didn't match node selector.

# 接下来，停止pod
kubectl delete -f pod-nodeaffinity-required.yaml
pod "pod-nodeaffinity-required" deleted

# 修改文件，将values: ["xxx","yyy"]------> ["pro","yyy"]
vim pod-nodeaffinity-required.yaml

# 再次启动
kubectl create -f pod-nodeaffinity-required.yaml
pod/pod-nodeaffinity-required created

# 此时查看，发现调度成功，已经将pod调度到了node1上
kubectl get pods pod-nodeaffinity-required -o wide
NAME                        READY   STATUS    RESTARTS   AGE   IP            NODE  ...... 
pod-nodeaffinity-required   1/1     Running   0          11s   10.244.1.89   node1 ......
```
##### `preferredDuringSchedulingIgnoredDuringExecution` 示例
创建`pod-nodeaffinity-preferred.yaml`
```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-nodeaffinity-preferred  # Pod 的名称，用于标识这个 Pod
spec:
  containers:
    - name: nginx       # 容器的名称
      image: nginx:1.17.1  # 容器使用的镜像，这里使用的是 Nginx 1.17.1 版本
  affinity: # 亲和性设置，用于控制 Pod 的调度位置
    nodeAffinity: # 节点亲和性，用于控制 Pod 应该调度到哪些节点上
      preferredDuringSchedulingIgnoredDuringExecution: # 软性要求，优先考虑满足条件的节点，但不是硬性要求
        - weight: 1  # 权重，表示这个偏好项的重要性，数值越大，越优先
          preference:
            matchExpressions: # 匹配表达式，用于匹配节点的标签
              - key: env  # 匹配节点的 "env" 标签
                operator: In  # 操作符，表示匹配的值必须在指定列表中
                values: [ "xxx","yyy" ]  # 匹配的值，即 "env" 标签的值必须是 "xxx" 或 "yyy"
```

```shell
# 创建pod
kubectl create -f pod-nodeaffinity-preferred.yaml
pod/pod-nodeaffinity-preferred created

# 查看pod状态 （运行成功）
kubectl get pod pod-nodeaffinity-preferred -o wide
NAME                         READY   STATUS    RESTARTS   AGE   IP               NODE        NOMINATED NODE   READINESS GATES
pod-nodeaffinity-preferred   1/1     Running   0          14s   10.244.169.179   k8s-node2   <none>           <none>
```
##### NodeAffinity规则设置的注意事项
1. 如果同时定义了nodeSelector和nodeAffinity，那么必须两个条件都得到满足，Pod才能运行在指定的Node上
2. 如果nodeAffinity指定了多个nodeSelectorTerms，那么只需要其中一个能够匹配成功即可
```yml
...
- matchExpressions:
  - key: env
	operator: In
	values: [ "pro" ]
- matchExpressions:
  - key: env
	operator: In
	values: [ "pro2" ]
...
```
3. 如果一个nodeSelectorTerms中有多个matchExpressions ，则一个节点必须满足所有的才能匹配成功
```yml
...
nodeSelectorTerms:
  - matchExpressions:
	  - key: env
		operator: In
		values: [ "pro" ]
	  - key: env
		operator: In
		values: [ "pro2" ]
...
```
4. 如果一个pod所在的Node在Pod运行期间其标签发生了改变，不再符合该Pod的节点亲和性需求，则系统将忽略此变化
#### TopologyKey
用于指定节点亲和性（Node Affinity）和反亲和性（Node Anti-Affinity）规则的作用域(维度)。
常用有以下值：
```yml
topologyKey: kubernetes.io/hostname # 以节点为单位进行划分，每个节点都是一个独立的拓扑域。
topologyKey: beta.kubernetes.io/os # 以节点操作系统类型为单位进行划分，例如 Linux 节点和 Windows 节点。
topologyKey: topology.kubernetes.io/zone # 以可用区（Zone）为单位进行划分，例如 `us-east-1a` 和 `us-east-1b`。
```
假设我们有以下几个节点：
- node1: 标签 `disk-type=ssd, zone=us-east-1a, os=linux`
- node2: 标签 `disk-type=hdd, zone=us-east-1a, os=linux`
- node3: 标签 `disk-type=ssd, zone=us-east-1b, os=linux`
- node4: 标签 `disk-type=ssd, zone=us-east-1a, os=windows`
##### `kubernetes.io/hostname`
如果我们使用 `kubernetes.io/hostname` 作为 `topologyKey`，那么每个节点都是一个独立的拓扑域。
例如，我们可以使用节点反亲和性来避免将同一个 Pod 的多个副本调度到同一个节点上：
```yml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        topologyKey: kubernetes.io/hostname
  containers:
  - name: my-container
    image: nginx
```
这个配置表示，对于同一个 Pod，尽量不要将多个副本调度到同一个节点上。
##### `beta.kubernetes.io/os`
如果我们使用 `beta.kubernetes.io/os` 作为 `topologyKey`，那么所有具有相同操作系统类型的节点 будут 被视为同一个拓扑域。例如，我们可以使用节点亲和性来将 Pod 调度到 Linux 节点上：
```yml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: beta.kubernetes.io/os
            operator: In
            values:
            - linux
  containers:
  - name: my-container
    image: nginx
```
这个配置表示，Pod 必须调度到操作系统类型为 Linux 的节点上。
##### `topology.kubernetes.io/zone`
如果我们使用 `topology.kubernetes.io/zone` 作为 `topologyKey`，那么同一个可用区内的节点会被视为同一个拓扑域。例如，我们可以使用节点反亲和性来避免将同一个 Pod 的多个副本调度到同一个可用区内，以提高应用的可用性：
```yml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        topologyKey: topology.kubernetes.io/zone
  containers:
  - name: my-container
    image: nginx
```
这个配置表示，对于同一个 Pod，尽量不要将多个副本调度到同一个可用区内。
#### PodAffinity
PodAffinity主要实现以运行的Pod为参照，实现让新创建的Pod跟参照pod在一个区域的功能。
##### `PodAffinity` 的可配置项
```yml
pod.spec.affinity.podAffinity # 定义 Pod 亲和性规则
  requiredDuringSchedulingIgnoredDuringExecution # 硬性约束，调度时必须满足亲和性规则
    namespaces # 指定参照 Pod 的命名空间列表，可选，默认为当前 Pod 所在命名空间
    topologyKey # 指定调度作用域，例如 kubernetes.io/hostname（节点级别）或 topology.kubernetes.io/zone（区域级别）
    labelSelector # 标签选择器，用于匹配要亲和的 Pod
      matchExpressions # 匹配表达式列表，推荐使用，支持更灵活的匹配规则
        - key # 标签键
          operator # 关系运算符，支持 In, NotIn, Exists, DoesNotExist
          values # 标签值列表
      matchLabels # 匹配标签列表，等同于多个 matchExpressions 的 AND 关系，用于简单匹配
  preferredDuringSchedulingIgnoredDuringExecution # 软性约束，尽量满足亲和性规则
    - weight # 倾向权重，范围 1-100，数字越大权重越高，优先级越高
      podAffinityTerm # 亲和性选项
        namespaces # 指定参照 Pod 的命名空间列表，可选
        topologyKey # 指定调度作用域
        labelSelector # 标签选择器
          matchExpressions # 匹配表达式列表
            - key # 标签键
              operator # 关系运算符
              values # 标签值列表
          matchLabels # 匹配标签列表
```
##### `requiredDuringSchedulingIgnoredDuringExecution` 示例
1）首先创建一个参照Pod，`pod-podaffinity-target.yaml`：
```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-podaffinity-target  # Pod 的名称，用于标识这个 Pod
  labels:
    podenv: pro  # 设置标签，用于后续的 Pod 亲和性匹配
spec:
  containers:
    - name: nginx  # 容器的名称
      image: nginx:1.17.1  # 容器使用的镜像，这里使用的是 Nginx 1.17.1 版本
  nodeName: k8s-node1  # 将 Pod 直接调度到节点 k8s-node1 上
```
执行命令
```shell
# 启动目标pod
kubectl create -f pod-podaffinity-target.yaml
pod/pod-podaffinity-target created

# 查看pod状况
kubectl get pods  pod-podaffinity-target -o wide
NAME                     READY   STATUS    RESTARTS   AGE   IP             NODE        NOMINATED NODE   READINESS GATES
pod-podaffinity-target   1/1     Running   0          4s    10.244.36.85   k8s-node1   <none>           <none>
```

2）创建`pod-podaffinity-required.yaml`，内容如下：
```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-podaffinity-required  # Pod 的名称，用于标识这个 Pod
spec:
  containers:
    - name: nginx       # 容器的名称
      image: nginx:1.17.1  # 容器使用的镜像，这里使用的是 Nginx 1.17.1 版本
  affinity: # 亲和性设置，用于控制 Pod 的调度位置
    podAffinity: # Pod 亲和性，用于控制 Pod 应该和哪些 Pod 在同一个节点上
      requiredDuringSchedulingIgnoredDuringExecution: # 硬性要求，Pod 在调度时必须满足这个条件
        - labelSelector: # 标签选择器，用于匹配其他 Pod 的标签
            matchExpressions: # 匹配表达式，用于匹配其他 Pod 的标签
              - key: podenv  # 匹配其他 Pod 的 "podenv" 标签
                operator: In  # 操作符，表示匹配的值必须在指定列表中
                values: [ "xxx","yyy" ]  # 匹配的值，即 "podenv" 标签的值必须是 "xxx" 或 "yyy"
          topologyKey: kubernetes.io/hostname  # 拓扑键，表示在同一个节点上匹配
```

上面配置表达的意思是：新Pod必须要与拥有标签env=xxx或者env=yyy的pod在同一Node上，显然现在没有这样pod，接下来，运行测试一下。
```shell
# 启动pod
kubectl create -f pod-podaffinity-required.yaml
pod/pod-podaffinity-required created

# 查看pod状态，发现未运行
kubectl get po pod-podaffinity-required
NAME                       READY   STATUS    RESTARTS   AGE
pod-podaffinity-required   0/1     Pending   0          9s

# 查看详细信息
kubectl describe po pod-podaffinity-required
......
Events:
  Type     Reason            Age        From               Message
  ----     ------            ----       ----               -------
  Warning  FailedScheduling  <unknown>  default-scheduler  0/3 nodes are available: 2 node(s) didn't match pod affinity rules, 1 node(s) had taints that the pod didn't tolerate.

# 接下来修改  values: ["xxx","yyy"]----->values:["pro","yyy"]
# 意思是：新Pod必须要与拥有标签env=xxx或者env=yyy的pod在同一Node上
vim pod-podaffinity-required.yaml

# 然后重新创建pod，查看效果
kubectl delete -f  pod-podaffinity-required.yaml
kubectl create -f pod-podaffinity-required.yaml

# 发现此时Pod运行正常
kubectl get po pod-podaffinity-required -o wide
NAME                       READY   STATUS    RESTARTS   AGE   IP             NODE        NOMINATED NODE   READINESS GATES
pod-podaffinity-required   1/1     Running   0          5s    10.244.36.83   k8s-node1   <none>           <none>
```
##### `preferredDuringSchedulingIgnoredDuringExecution` 示例
 
#### PodAntiAffinity
##### `PodAntiAffinity` 的可配置项
```yml
pod.spec.affinity.podAntiAffinity # 定义 Pod 反亲和性规则
  requiredDuringSchedulingIgnoredDuringExecution # 硬性约束，调度时必须满足反亲和性规则
    namespaces # 指定参照 Pod 的命名空间列表，可选，默认为当前 Pod 所在命名空间
    topologyKey # 指定调度作用域，例如 kubernetes.io/hostname（节点级别）或 topology.kubernetes.io/zone（区域级别）
    labelSelector # 标签选择器，用于匹配要避免的 Pod
      matchExpressions # 匹配表达式列表，推荐使用，支持更灵活的匹配规则
        - key # 标签键
          operator # 关系运算符，支持 In, NotIn, Exists, DoesNotExist
          values # 标签值列表
      matchLabels # 匹配标签列表，等同于多个 matchExpressions 的 AND 关系，用于简单匹配
  preferredDuringSchedulingIgnoredDuringExecution # 软性约束，尽量满足反亲和性规则
    - weight # 倾向权重，范围 1-100，数字越大权重越高，优先级越高
      podAffinityTerm # 反亲和性选项
        namespaces # 指定参照 Pod 的命名空间列表，可选
        topologyKey # 指定调度作用域
        labelSelector # 标签选择器
          matchExpressions # 匹配表达式列表
            - key # 标签键
              operator # 关系运算符
              values # 标签值列表
          matchLabels # 匹配标签列表
```
##### `requiredDuringSchedulingIgnoredDuringExecution` 示例
PodAntiAffinity主要实现以运行的Pod为参照，让新创建的Pod跟参照pod不在一个区域中的功能。
它的配置方式和选项跟PodAffinty是一样的，这里不再做详细解释，直接做一个测试案例。
1. 继续使用上个案例中目标pod
```shell
kubectl get pods -o wide --show-labels
NAME                     READY   STATUS    RESTARTS   AGE     IP            NODE    LABELS
pod-podaffinity-required 1/1     Running   0          3m29s   10.244.1.38   k8s-node1   <none>     
pod-podaffinity-target   1/1     Running   0          9m25s   10.244.1.37   k8s-node1   podenv=pro
```
2. 创建`pod-podantiaffinity-required.yaml`，内容如下：
```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-podantiaffinity-required  # Pod 的名称，用于标识这个 Pod
spec:
  containers:
    - name: nginx       # 容器的名称
      image: nginx:1.17.1  # 容器使用的镜像，这里使用的是 Nginx 1.17.1 版本
  affinity: # 亲和性设置，用于控制 Pod 的调度位置
    podAntiAffinity: # Pod 反亲和性，用于控制 Pod 不应该和哪些 Pod 在同一个节点上
      requiredDuringSchedulingIgnoredDuringExecution: # 硬性要求，Pod 在调度时必须满足这个条件
        - labelSelector: # 标签选择器，用于匹配其他 Pod 的标签
            matchExpressions: # 匹配表达式，用于匹配其他 Pod 的标签
              - key: podenv  # 匹配其他 Pod 的 "podenv" 标签
                operator: In  # 操作符，表示匹配的值必须在指定列表中
                values: [ "pro" ]  # 匹配的值，即 "podenv" 标签的值必须是 "pro"
          topologyKey: kubernetes.io/hostname  # 拓扑键，表示在同一个节点上匹配
```

上面配置表达的意思是：新Pod必须要与拥有标签env=pro的pod不在同一Node上，运行测试一下。
```shell
# 创建pod
kubectl create -f pod-podantiaffinity-required.yaml
pod/pod-podantiaffinity-required created

# 查看pod
# 发现调度到了k8s-node2上
kubectl get pods pod-podantiaffinity-required -o wide
NAME                           READY   STATUS    RESTARTS   AGE   IP               NODE        NOMINATED NODE   READINESS GATES
pod-podantiaffinity-required   1/1     Running   0          8s    10.244.169.175   k8s-node2   <none>           <none>
```
##### `preferredDuringSchedulingIgnoredDuringExecution` 示例

### 5.4.3 污点和容忍

#### 污点（Taints）
前面的调度方式都是站在Pod的角度上，通过在Pod上添加属性，来确定Pod是否要调度到指定的Node上，其实我们也可以站在Node的角度上，通过在Node上添加 **污点** 属性，来决定是否允许Pod调度过来。
Node被设置上污点之后就和Pod之间存在了一种相斥的关系，进而拒绝Pod调度进来，甚至可以将已经存在的Pod驱逐出去。
污点的格式为： `key=value:effect` , key和value是污点的标签，effect描述污点的作用，支持如下三个选项：
- PreferNoSchedule：kubernetes将尽量避免把Pod调度到具有该污点的Node上，除非没有其他节点可调度
- NoSchedule：kubernetes将不会把Pod调度到具有该污点的Node上，但不会影响当前Node上已存在的Pod
- NoExecute：kubernetes将不会把Pod调度到具有该污点的Node上，同时也会将Node上已存在的Pod驱离
![image-20200605021606508](https://weichengjun2.dpdns.org//i/2025/02/15/67b00982cbfdb.png) 
使用kubectl设置和去除污点的命令示例如下：
```shell
# 设置污点
kubectl taint nodes k8s-node1 key=value:effect
# 去除污点
kubectl taint nodes k8s-node1 key:effect-
# 去除所有污点
kubectl taint nodes k8s-node1 key-
# 查询所有节点的污点
kubectl get nodes -o=custom-columns=NAME:.metadata.name,TAINTS:.spec.taints
```
接下来，演示下污点的效果：
1.  准备节点k8s-node1（为了演示效果更加明显，暂时停止k8s-node2节点）
2.  为k8s-node1节点设置一个污点:  `tag=riven:PreferNoSchedule` ；然后创建pod1( pod1 可以 )
3.  修改为k8s-node1节点设置一个污点:  `tag=riven:NoSchedule` ；然后创建pod2( pod1 正常 pod2 失败 )
4.  修改为k8s-node1节点设置一个污点:  `tag=riven:NoExecute` ；然后创建pod3 ( 3个pod都失败 )
```shell
# 停止节点 k8s-node2
kubectl cordon k8s-node2
# 恢复节点 k8s-node2
kubectl uncordon k8s-node2

# 为k8s-node1设置污点(PreferNoSchedule)
kubectl taint nodes k8s-node1 tag=riven:PreferNoSchedule

# 创建pod1
kubectl run taint1 --image=nginx:1.17.1
kubectl get pods -o wide
NAME                      READY   STATUS    RESTARTS   AGE     IP           NODE   
taint1-7665f7fd85-574h4   1/1     Running   0          2m24s   10.244.1.59   k8s-node1    

# 为k8s-node1设置污点(取消PreferNoSchedule，设置NoSchedule)
kubectl taint nodes k8s-node1 tag:PreferNoSchedule-
kubectl taint nodes k8s-node1 tag=riven:NoSchedule

# 创建pod2
kubectl run taint2 --image=nginx:1.17.1
kubectl get pods taint2 -o wide
NAME                      READY   STATUS    RESTARTS   AGE     IP            NODE
taint1-7665f7fd85-574h4   1/1     Running   0          2m24s   10.244.1.59   k8s-node1 
taint2-544694789-6zmlf    0/1     Pending   0          21s     <none>        <none>   

# 为k8s-node1设置污点(取消NoSchedule，设置NoExecute)
kubectl taint nodes k8s-node1 tag:NoSchedule-
kubectl taint nodes k8s-node1 tag=riven:NoExecute

# 创建pod3
kubectl run taint3 --image=nginx:1.17.1
kubectl get pods -o wide
NAME                      READY   STATUS    RESTARTS   AGE   IP       NODE     NOMINATED 
taint1-7665f7fd85-htkmp   0/1     Pending   0          35s   <none>   <none>   <none>    
taint2-544694789-bn7wb    0/1     Pending   0          35s   <none>   <none>   <none>     
taint3-6d78dbd749-tktkq   0/1     Pending   0          6s    <none>   <none>   <none>     
```
>小提示：使用kubeadm搭建的集群，默认就会给master节点添加一个污点标记,所以pod就不会调度到master节点上.

#### 容忍（Toleration）
上面介绍了污点的作用，我们可以在node上添加污点用于拒绝pod调度上来，但是如果就是想将一个pod调度到一个有污点的node上去，这时候应该怎么做呢？这就要使用到 **容忍** 。

![image-20200514095913741](https://weichengjun2.dpdns.org//i/2025/02/15/67b009830b465.png) 
> 污点就是拒绝，容忍就是忽略，Node通过污点拒绝pod调度上去，Pod通过容忍忽略拒绝

下面先通过一个案例看下效果：
1.  上一小节，已经在k8s-node1节点上打上了 `NoExecute` 的污点，此时pod是调度不上去的
2.  本小节，可以通过给pod添加容忍，然后将其调度上去

创建`pod-toleration.yaml`,内容如下
```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-toleration  # Pod 的名称，用于唯一标识这个 Pod
spec:
  containers:
    - name: nginx  # 容器的名称
      image: nginx:1.17.1  # 容器使用的镜像，这里使用 Nginx 1.17.1 版本
  tolerations: # 容忍列表，指定 Pod 可以容忍的污点
    - key: "tag"  # 要容忍的污点的键，必须与节点上的污点键完全匹配
      operator: "Equal"  # 操作符，指定容忍度与污点匹配的方式，这里表示必须完全相等
      value: "riven"  # 容忍的污点的值，必须与节点上的污点值完全匹配
      effect: "NoExecute"  # 容忍的效果，表示即使节点上有 "tag=riven:NoExecute" 的污点，这个 Pod 也可以被调度到该节点上，并且不会被驱逐
```

```shell
# 查询 k8s-node1 污点
kubectl get node k8s-node1 -o=jsonpath='{.spec.taints[*]}'
NAME        TAINTS
k8s-node1   [map[effect:NoExecute key:tag value:riven]]

# 创建未添加容忍的pod
kubectl create -f pod-toleration.yaml
# 查询添加容忍之前的pod
kubectl get pods -o wide
NAME                               READY   STATUS    RESTARTS   AGE     IP               NODE        NOMINATED NODE   READINESS GATES
pod-toleration                     0/1     Pending   0          7s     <none>           <none>      <none>           <none>

# 删除未添加容忍的pod
kubectl delete -f pod-toleration.yaml
# 给pod添加容忍
vim pod-toleration.yaml
# 创建添加容忍之后的pod
kubectl create -f pod-toleration.yaml
# 添加容忍之后的pod
kubectl get pods -o wide
NAME                               READY   STATUS    RESTARTS   AGE     IP               NODE        NOMINATED NODE   READINESS GATES
pod-toleration                     1/1     Running   0          4m37s   10.244.36.88     k8s-node1   <none>           <none>
```
下面看一下容忍的详细配置:
```shell
kubectl explain pod.spec.tolerations
......
FIELDS:
   key       # 对应着要容忍的污点的键，空意味着匹配所有的键
   value     # 对应着要容忍的污点的值
   operator  # key-value的运算符，支持Equal和Exists（默认）
   effect    # 对应污点的effect，空意味着匹配所有影响
   tolerationSeconds   # 容忍时间, 当effect为NoExecute时生效，表示pod在Node上的停留时间
```
# 6. Pod控制器详解
## 6.1 Pod控制器介绍

Pod是kubernetes的最小管理单元，在kubernetes中，按照pod的创建方式可以将其分为两类：
- 自主式pod：kubernetes直接创建出来的Pod，这种pod删除后就没有了，也不会重建
- 控制器创建的pod：kubernetes通过控制器创建的pod，这种pod删除了之后还会自动重建

>  `什么是Pod控制器` Pod控制器是管理pod的中间层，使用Pod控制器之后，只需要告诉Pod控制器，想要多少个什么样的Pod就可以了，它会创建出满足条件的Pod并确保每一个Pod资源处于用户期望的目标状态。如果Pod资源在运行中出现故障，它会基于指定策略重新编排Pod。

在kubernetes中，有很多类型的pod控制器，每种都有自己的适合的场景，常见的有下面这些：
- ReplicationController：比较原始的pod控制器，已经被废弃，由ReplicaSet替代
- ReplicaSet：保证副本数量一直维持在期望值，并支持pod数量扩缩容，镜像版本升级
- Deployment：通过控制ReplicaSet来控制Pod，并支持滚动升级、回退版本
- StatefulSet：管理有状态应用
- Horizontal Pod Autoscaler：可以根据集群负载自动水平调整Pod的数量，实现削峰填谷
- DaemonSet：在集群中的指定Node上运行且仅运行一个副本，一般用于守护进程类的任务
- Job：它创建出来的pod只要完成任务就立即退出，不需要重启或重建，用于执行一次性任务
- Cronjob：它创建的Pod负责周期性任务控制，不需要持续后台运行
## 6.2 ReplicaSet(RS)
ReplicaSet的主要作用是 **保证一定数量的pod正常运行** ，它会持续监听这些Pod的运行状态，一旦Pod发生故障，就会重启或重建。同时它还支持对pod数量的扩缩容和镜像版本的升降级。
![img](https://weichengjun2.dpdns.org//i/2025/02/15/67b0098348d51.png) 
资源清单文件
```yml
apiVersion: apps/v1 # 版本号
kind: ReplicaSet # 类型       
metadata: # 元数据
  name: replicaset # rs名称 
  namespace: default # 所属命名空间 
  labels: # 标签
    controller: rs
spec: # 详情描述
  replicas: 3 # 副本数量
  selector: # 选择器，通过它指定该控制器管理哪些pod
    matchLabels: # Labels匹配规则
      app: nginx-pod
    matchExpressions: # Expressions匹配规则
      - { key: app, operator: In, values: [ nginx-pod ] }
  template: # 模板，当副本数量不足时，会根据下面的模板创建pod副本
    metadata:
      labels:
        app: nginx-pod
    spec:
      containers:
        - name: nginx
          image: nginx:1.17.1
          ports:
            - containerPort: 80
```
在这里面，需要新了解的配置项就是 `spec` 下面几个选项：
- replicas：指定副本数量，其实就是当前rs创建出来的pod的数量，默认为1
- selector：选择器，它的作用是建立pod控制器和pod之间的关联关系，采用的Label Selector机制
	- 在pod模板上定义label，在控制器上定义选择器，就可以表明当前控制器能管理哪些pod了
- template：模板，就是当前控制器创建pod所使用的模板板，里面其实就是前一章学过的pod的定义
### 创建ReplicaSet
创建`replicaset.yaml`文件，内容如下：
```yml
apiVersion: apps/v1 # API 版本：指定 Kubernetes API 的版本，这里是 apps/v1，用于 ReplicaSet
kind: ReplicaSet # Kubernetes 资源类型：ReplicaSet，用于管理一组 Pod 的副本
metadata: # 元数据：描述 ReplicaSet 的信息
  name: replicaset # ReplicaSet 的名称：replicaset，必须是唯一的
spec: # ReplicaSet 规格：描述 ReplicaSet 的期望状态
  replicas: 3 # 副本数量：期望运行的 Pod 副本数，这里设置为 3
  selector: # 选择器：用于匹配该 ReplicaSet 管理的 Pod
    matchLabels: # 匹配标签：使用标签进行匹配
      app: nginx-pod # 匹配标签：app=nginx-pod 的 Pod
  template: # Pod 模板：定义 Pod 的规格，ReplicaSet 会根据此模板创建 Pod
    metadata: # Pod 元数据
      labels: # Pod 标签：必须与 ReplicaSet 的 selector 匹配
        app: nginx-pod # Pod 标签：app=nginx-pod
    spec: # Pod 规格
      containers: # 容器列表：Pod 中包含的容器
        - name: nginx # 容器名称：nginx
          image: nginx:1.17.1 # 容器镜像：使用的镜像为 nginx:1.17.1
```
命令执行
```shell
# 创建rs
kubectl create -f replicaset.yaml
replicaset.apps/replicaset created

# 查看rs
# DESIRED:期望副本数量  
# CURRENT:当前副本数量  
# READY:已经准备好提供服务的副本数量
kubectl get rs replicaset -o wide
NAME          DESIRED   CURRENT READY AGE   CONTAINERS   IMAGES             SELECTOR
replicaset 3         3       3     22s   nginx        nginx:1.17.1       app=nginx-pod

# 查看当前控制器创建出来的pod
# 这里发现控制器创建出来的pod的名称是在控制器名称后面拼接了-xxxxx随机码
kubectl get pod 
NAME                          READY   STATUS    RESTARTS   AGE
replicaset-6vmvt   1/1     Running   0          54s
replicaset-fmb8f   1/1     Running   0          54s
replicaset-snrk2   1/1     Running   0          54s

```
### 扩缩容
```shell
 # 编辑rs的副本数量，修改spec:replicas: 6即可
kubectl edit rs replicaset
replicaset.apps/replicaset edited

# 查看pod
kubectl get pods 
NAME                          READY   STATUS    RESTARTS   AGE
replicaset-6vmvt   1/1     Running   0          114m
replicaset-cftnp   1/1     Running   0          10s
replicaset-fjlm6   1/1     Running   0          10s
replicaset-fmb8f   1/1     Running   0          114m
replicaset-s2whj   1/1     Running   0          10s
replicaset-snrk2   1/1     Running   0          114m

# 当然也可以直接使用命令实现
# 使用scale命令实现扩缩容， 后面--replicas=n直接指定目标数量即可
kubectl scale rs replicaset --replicas=2 
replicaset.apps/replicaset scaled

# 命令运行完毕，立即查看，发现已经有4个开始准备退出了
kubectl get pods 
NAME                       READY   STATUS        RESTARTS   AGE
replicaset-6vmvt   0/1     Terminating   0          118m
replicaset-cftnp   0/1     Terminating   0          4m17s
replicaset-fjlm6   0/1     Terminating   0          4m17s
replicaset-fmb8f   1/1     Running       0          118m
replicaset-s2whj   0/1     Terminating   0          4m17s
replicaset-snrk2   1/1     Running       0          118m

#稍等片刻，就只剩下2个了
kubectl get pods 
NAME                       READY   STATUS    RESTARTS   AGE
replicaset-fmb8f   1/1     Running   0          119m
replicaset-snrk2   1/1     Running   0          119m
```
### 镜像更新
```shell
 # 编辑rs的容器镜像 - image: nginx:1.17.2
kubectl edit rs replicaset
replicaset.apps/replicaset edited

# 再次查看，发现镜像版本已经变更了
kubectl get rs -o wide
NAME                DESIRED  CURRENT   READY   AGE    CONTAINERS   IMAGES        ...
replicaset       2        2         2       140m   nginx         nginx:1.17.2  ...

# 同样的道理，也可以使用命令完成这个工作
# kubectl set image rs rs名称 容器=镜像版本 -n namespace
kubectl set image rs replicaset nginx=nginx:1.17.1  
replicaset.apps/replicaset image updated

# 再次查看，发现镜像版本已经变更了
kubectl get rs -o wide
NAME                 DESIRED  CURRENT   READY   AGE    CONTAINERS   IMAGES            ...
replicaset        2        2         2       145m   nginx        nginx:1.17.1 ... 
```
### 删除ReplicaSet
```shell
# 使用kubectl delete命令会删除此RS以及它管理的Pod
# 在kubernetes删除RS前，会将RS的replicasclear调整为0，等待所有的Pod被删除后，在执行RS对象的删除
kubectl delete rs replicaset 
replicaset.apps "replicaset" deleted
kubectl get pod -o wide
No resources found in dev namespace.

# 如果希望仅仅删除RS对象（保留Pod），可以使用kubectl delete命令时添加--cascade=false选项（不推荐）。
kubectl delete rs replicaset --cascade=false
replicaset.apps "replicaset" deleted
kubectl get pods 
NAME                  READY   STATUS    RESTARTS   AGE
replicaset-cl82j   1/1     Running   0          75s
replicaset-dslhb   1/1     Running   0          75s

# 也可以使用yaml直接删除(推荐)
kubectl delete -f replicaset.yaml
replicaset.apps "replicaset" deleted
```
## 6.3 Deployment(Deploy)
Deployment 用于管理运行一个应用负载的一组 Pod，通常适用于**无状态**的服务。
kubernetes在V1.2版本引入了Deployment控制器。
值得一提的是，这种控制器并不直接管理pod，而是通过管理ReplicaSet来间接管理Pod
即：Deployment管理ReplicaSet，ReplicaSet管理Pod。所以Deployment比ReplicaSet功能更加强大。
![img](https://weichengjun2.dpdns.org//i/2025/02/15/67b0098373b91.png) 
Deployment主要功能有下面几个：
- 支持ReplicaSet的所有功能
- 支持发布的停止、继续
- 支持滚动升级和回滚版本

资源清单文件
```yml
apiVersion: apps/v1 # 版本号
kind: Deployment # 类型
metadata: # 元数据
  name: # rs名称
  namespace: # 所属命名空间
  labels: #标签
    controller: deploy
spec: # 详情描述
  replicas: 3 # 副本数量
  revisionHistoryLimit: 3 # 保留的版本数：Deployment 历史版本保留数量，用于回滚，这里保留 10 个版本，超出则删除最旧的版本
  paused: false # 暂停部署，默认是false
  progressDeadlineSeconds: 600 # 部署超时时间（s），默认是600
  strategy: # 更新策略，支持Recreate和RollingUpdate两种更新策略
  type: Recreate # 更新策略：Recreate，先删除旧的 Pod，再创建新的 Pod。这意味着在更新期间，可能会有一段时间服务不可用。
#    type: RollingUpdate # 滚动更新策略  
#    rollingUpdate: # 滚动更新  
#      maxSurge: 25% # 最大额外可以存在的副本数，可以为百分比，也可以为整数（例如，如果 replicas=4，则最多会创建 1 个额外的 Pod，向上取整。控制更新速度。）  
#      maxUnavailable: 25% # 最大不可用状态的 Pod 的最大值，可以为百分比，也可以为整数（例如，如果 replicas=4，则最多允许 1 个 Pod 不可用，向上取整。控制可用性。）
  selector: # 选择器，通过它指定该控制器管理哪些pod
    matchLabels: # Labels匹配规则
      app: nginx-pod
    matchExpressions: # Expressions匹配规则
      - { key: app, operator: In, values: [ nginx-pod ] }
  template: # 模板，当副本数量不足时，会根据下面的模板创建pod副本
    metadata:
      labels:
        app: nginx-pod
    spec:
      containers:
        - name: nginx
          image: nginx:1.17.1
          ports:
            - containerPort: 80
```
### 创建deployment
创建`deployment.yaml`，内容如下：
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-pod
  template:
    metadata:
      labels:
        app: nginx-pod
    spec:
      containers:
        - name: nginx
          image: nginx:1.17.1
```
执行命令
```shell
# 创建deployment
kubectl create -f deployment.yaml --record=true
deployment.apps/deployment created

# 查看deployment
# UP-TO-DATE 最新版本的pod的数量
# AVAILABLE  当前可用的pod的数量
kubectl get deploy deployment
NAME            READY   UP-TO-DATE   AVAILABLE   AGE
deployment   3/3     3            3           15s

# 查看rs
# 发现rs的名称是在原来deployment的名字后面添加了一个10位数的随机串
kubectl get rs
NAME                       DESIRED   CURRENT   READY   AGE
deployment-6696798b78   3         3         3       23s

# 查看pod
kubectl get pods
NAME                             READY   STATUS    RESTARTS   AGE
deployment-6696798b78-d2c8n   1/1     Running   0          107s
deployment-6696798b78-smpvp   1/1     Running   0          107s
deployment-6696798b78-wvjd8   1/1     Running   0          107s

```
### 扩缩容
```shell
# 变更副本数量为5个
kubectl scale deploy deployment --replicas=5  
deployment.apps/deployment scaled

# 查看deployment
kubectl get deploy deployment 
NAME            READY   UP-TO-DATE   AVAILABLE   AGE
deployment   5/5     5            5           2m

# 查看pod
 kubectl get pods 
NAME                             READY   STATUS    RESTARTS   AGE
deployment-6696798b78-d2c8n   1/1     Running   0          4m19s
deployment-6696798b78-jxmdq   1/1     Running   0          94s
deployment-6696798b78-mktqv   1/1     Running   0          93s
deployment-6696798b78-smpvp   1/1     Running   0          4m19s
deployment-6696798b78-wvjd8   1/1     Running   0          4m19s

# 编辑deployment的副本数量，修改spec:replicas: 4即可
kubectl edit deploy deployment 
deployment.apps/deployment edited

# 查看pod
kubectl get pods 
NAME                             READY   STATUS    RESTARTS   AGE
deployment-6696798b78-d2c8n   1/1     Running   0          5m23s
deployment-6696798b78-jxmdq   1/1     Running   0          2m38s
deployment-6696798b78-smpvp   1/1     Running   0          5m23s
deployment-6696798b78-wvjd8   1/1     Running   0          5m23s

```
### 更新策略「镜像更新 」
deployment支持两种更新策略: `重建更新` 和 `滚动更新` ,可以通过 `strategy` 指定策略类型,支持两个属性:
```yml
strategy：指定新的Pod替换旧的Pod的策略， 支持两个属性：
  type：指定策略类型，支持两种策略
    Recreate：在创建出新的Pod之前会先杀掉所有已存在的Pod
    RollingUpdate：滚动更新，就是杀死一部分，就启动一部分，在更新过程中，存在两个版本Pod
  rollingUpdate：当type为RollingUpdate时生效，用于为RollingUpdate设置参数，支持两个属性：
    maxSurge： 用来指定在升级过程中可以超过期望的Pod的最大数量，默认为25%。
    maxUnavailable：用来指定在升级过程中不可用Pod的最大数量，默认为25%。
```
#### 重建更新
1.  编辑`deployment.yaml`,在spec节点下添加更新策略
```yml
spec:
  strategy: # 策略
    type: Recreate # 更新策略：Recreate，先删除旧的 Pod，再创建新的 Pod。这意味着在更新期间，可能会有一段时间服务不可用。
```
2.  更新进行验证
```shell
# 更新
kubectl apply -f deployment.yaml
# 变更镜像
kubectl set image deployment deployment nginx=nginx:1.17.2 
deployment.apps/deployment image updated

# 观察升级过程
kubectl get pods -w
NAME                             READY   STATUS    RESTARTS   AGE
deployment-5d89bdfbf9-65qcw   1/1     Running   0          31s
deployment-5d89bdfbf9-w5nzv   1/1     Running   0          31s
deployment-5d89bdfbf9-xpt7w   1/1     Running   0          31s

deployment-5d89bdfbf9-xpt7w   1/1     Terminating   0          41s
deployment-5d89bdfbf9-65qcw   1/1     Terminating   0          41s
deployment-5d89bdfbf9-w5nzv   1/1     Terminating   0          41s

deployment-675d469f8b-grn8z   0/1     Pending       0          0s
deployment-675d469f8b-hbl4v   0/1     Pending       0          0s
deployment-675d469f8b-67nz2   0/1     Pending       0          0s

deployment-675d469f8b-grn8z   0/1     ContainerCreating   0          0s
deployment-675d469f8b-hbl4v   0/1     ContainerCreating   0          0s
deployment-675d469f8b-67nz2   0/1     ContainerCreating   0          0s

deployment-675d469f8b-grn8z   1/1     Running             0          1s
deployment-675d469f8b-67nz2   1/1     Running             0          1s
deployment-675d469f8b-hbl4v   1/1     Running             0          2s

```
#### 滚动更新
1.  编辑`deployment.yaml`,在spec节点下添加更新策略
```yml
spec: # Deployment 规格：描述 Deployment 的期望状态
  strategy: # 策略：定义如何更新 Pod
    type: RollingUpdate # 更新策略：滚动更新，使用滚动更新策略
    rollingUpdate: # 滚动更新配置：滚动更新时的配置
	  maxSurge: 1 # 最大额外可以存在的副本数，可以为百分比，也可以为整数（例如，如果 replicas=4，则最多会创建 1 个额外的 Pod，向上取整。控制更新速度。）
      maxUnavailable: 1 # 最大不可用状态的 Pod 的最大值，可以为百分比，也可以为整数（例如，如果 replicas=4，则最多允许 1 个 Pod 不可用，向上取整。控制可用性。）
```
2.  更新进行验证
```shell
# 更新
kubectl apply -f deployment.yaml
# 变更镜像
kubectl set image deployment deployment nginx=nginx:1.17.3
deployment.apps/deployment image updated

# 观察升级过程
kubectl get pods -w
NAME                           READY   STATUS    RESTARTS   AGE
deployment-c848d767-8rbzt   1/1     Running   0          31m
deployment-c848d767-h4p68   1/1     Running   0          31m
deployment-c848d767-hlmz4   1/1     Running   0          31m
deployment-c848d767-rrqcn   1/1     Running   0          31m

deployment-966bf7f44-226rx   0/1     Pending             0          0s
deployment-966bf7f44-226rx   0/1     ContainerCreating   0          0s
deployment-966bf7f44-226rx   1/1     Running             0          1s
deployment-c848d767-h4p68    0/1     Terminating         0          34m

deployment-966bf7f44-cnd44   0/1     Pending             0          0s
deployment-966bf7f44-cnd44   0/1     ContainerCreating   0          0s
deployment-966bf7f44-cnd44   1/1     Running             0          2s
deployment-c848d767-hlmz4    0/1     Terminating         0          34m

deployment-966bf7f44-px48p   0/1     Pending             0          0s
deployment-966bf7f44-px48p   0/1     ContainerCreating   0          0s
deployment-966bf7f44-px48p   1/1     Running             0          0s
deployment-c848d767-8rbzt    0/1     Terminating         0          34m

deployment-966bf7f44-dkmqp   0/1     Pending             0          0s
deployment-966bf7f44-dkmqp   0/1     ContainerCreating   0          0s
deployment-966bf7f44-dkmqp   1/1     Running             0          2s
deployment-c848d767-rrqcn    0/1     Terminating         0          34m

# 至此，新版本的pod创建完毕，就版本的pod销毁完毕
# 中间过程是滚动进行的，也就是边销毁边创建
```

滚动更新的过程：
![img](https://weichengjun2.dpdns.org//i/2025/02/15/67b00983b5643.png) 
镜像更新中rs的变化
```shell
# 查看rs,发现原来的rs的依旧存在，只是pod数量变为了0，而后又新产生了一个rs，pod数量为4
# 其实这就是deployment能够进行版本回退的奥妙所在，后面会详细解释
kubectl get rs 
NAME                       DESIRED   CURRENT   READY   AGE
deployment-6696798b78   0         0         0       7m37s
deployment-6696798b11   0         0         0       5m37s
deployment-c848d76789   4         4         4       72s
```
### 版本回退
deployment支持版本升级过程中的暂停、继续功能以及版本回退等诸多功能，下面具体来看.
kubectl rollout： 版本升级相关功能，支持下面的选项：
- status 显示当前升级状态
- history 显示 升级历史记录
- pause 暂停版本升级过程
- resume 继续已经暂停的版本升级过程
- restart 重启版本升级过程
- undo 回滚到上一级版本（可以使用–to-revision回滚到指定版本）
```shell
# 查看当前升级版本的状态
kubectl rollout status deploy deployment
deployment "deployment" successfully rolled out

# 查看升级历史记录
kubectl rollout history deploy deployment
deployment.apps/deployment
REVISION  CHANGE-CAUSE
1         kubectl create --filename=deployment.yaml --record=true
2         kubectl create --filename=deployment.yaml --record=true
3         kubectl create --filename=deployment.yaml --record=true
# 可以发现有三次版本记录，说明完成过两次升级

# 版本回滚
# 这里直接使用--to-revision=1回滚到了1版本， 如果省略这个选项，就是回退到上个版本，就是2版本
kubectl rollout undo deployment deployment --to-revision=1 
deployment.apps/deployment rolled back

# 查看发现，通过nginx镜像版本可以发现到了第一版
kubectl get deploy -o wide
NAME            READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES         
deployment   4/4     4            4           74m   nginx        nginx:1.17.1   

# 查看rs，发现第一个rs中有4个pod运行，后面两个版本的rs中pod为运行
# 其实deployment之所以可是实现版本的回滚，就是通过记录下历史rs来实现的，
# 一旦想回滚到哪个版本，只需要将当前版本pod数量降为0，然后将回滚版本的pod提升为目标数量就可以了
kubectl get rs
NAME                       DESIRED   CURRENT   READY   AGE
deployment-6696798b78   4         4         4       78m
deployment-966bf7f44    0         0         0       37m
deployment-c848d767     0         0         0       71m
```
### 金丝雀发布
Deployment控制器支持控制更新过程中的控制，如“暂停(pause)”或“继续(resume)”更新操作。

比如有一批新的Pod资源创建完成后立即暂停更新过程，此时，仅存在一部分新版本的应用，主体部分还是旧的版本。然后，再筛选一小部分的用户请求路由到新版本的Pod应用，继续观察能否稳定地按期望的方式运行。确定没问题之后再继续完成余下的Pod资源滚动更新，否则立即回滚更新操作。这就是所谓的金丝雀发布。
```shell
# 更新deployment的版本，并配置暂停deployment
kubectl set image deploy deployment nginx=nginx:1.17.4 && kubectl rollout pause deployment deployment  
deployment.apps/deployment image updated
deployment.apps/deployment paused

# 观察更新状态
kubectl rollout status deploy deployment
Waiting for deployment "deployment" rollout to finish: 2 out of 4 new replicas have been updated...

# 监控更新的过程，可以看到已经新增了一个资源，但是并未按照预期的状态去删除一个旧的资源，就是因为使用了pause暂停命令
kubectl get rs -o wide
NAME                       DESIRED   CURRENT   READY   AGE     CONTAINERS   IMAGES         
deployment-5d89bdfbf9   3         3         3       19m     nginx        nginx:1.17.1   
deployment-675d469f8b   0         0         0       14m     nginx        nginx:1.17.2   
deployment-6c9f56fcfb   2         2         2       3m16s   nginx        nginx:1.17.4   
kubectl get pods 
NAME                             READY   STATUS    RESTARTS   AGE
deployment-5d89bdfbf9-rj8sq   1/1     Running   0          7m33s
deployment-5d89bdfbf9-ttwgg   1/1     Running   0          7m35s
deployment-5d89bdfbf9-v4wvc   1/1     Running   0          7m34s
deployment-6c9f56fcfb-996rt   1/1     Running   0          3m31s
deployment-6c9f56fcfb-j2gtj   1/1     Running   0          3m31s

# 确保更新的pod没问题了，继续更新
kubectl rollout resume deploy deployment
deployment.apps/deployment resumed

# 查看最后的更新情况
kubectl get rs -o wide
NAME                       DESIRED   CURRENT   READY   AGE     CONTAINERS   IMAGES         
deployment-5d89bdfbf9   0         0         0       21m     nginx        nginx:1.17.1   
deployment-675d469f8b   0         0         0       16m     nginx        nginx:1.17.2   
deployment-6c9f56fcfb   4         4         4       5m11s   nginx        nginx:1.17.4   
# 查看pod
kubectl get pods 
NAME                             READY   STATUS    RESTARTS   AGE
deployment-6c9f56fcfb-7bfwh   1/1     Running   0          37s
deployment-6c9f56fcfb-996rt   1/1     Running   0          5m27s
deployment-6c9f56fcfb-j2gtj   1/1     Running   0          5m27s
deployment-6c9f56fcfb-rf84v   1/1     Running   0          37s

```
 **删除Deployment** 
```shell
# 删除deployment，其下的rs和pod也将被删除
kubectl delete -f deployment.yaml
deployment.apps "deployment" deleted
```
## 6.4 StatefulSet(STS)
StatefulSet 运行一组 Pod，并为每个 Pod 保留一个稳定的标识，通常适用于**有状态**的服务。这可用于管理需要持久化存储或稳定、唯一网络标识的应用。
StatefulSet 用来管理某 [Pod](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/ "Pod 表示你的集群上一组正在运行的容器。") 集合的部署和扩缩， 并为这些 Pod 提供持久存储和持久标识符。

和 [Deployment](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/deployment/ "管理集群上的多副本应用。") 类似， StatefulSet 管理基于相同容器规约的一组 Pod。
但和 Deployment 不同的是， StatefulSet 为它们的每个 Pod 维护了一个有粘性的 ID。
这些 Pod 是基于相同的规约来创建的， 但是不能相互替换：无论怎么调度，**每个 Pod 都有一个永久不变的 ID**。

如果希望使用**存储卷为工作负载提供持久存储**，可以使用 StatefulSet 作为解决方案的一部分。 
尽管 StatefulSet 中的单个 Pod 仍可能出现故障， 但持久的 Pod 标识符使得将现有卷与替换已失败 Pod 的新 Pod 相匹配变得更加容易。

### 主要特点
- 稳定的、唯一的网络标识符：
	Pod 重新调度后其 PodName 和 HostName 不变，基于 Headless Service（即没有 Cluster IP 的 Service）来实现
- 稳定的持久化存储：
	Pod 重新调度后还是能访问到相同的持久化数据，**每个 Pod 提供独立的持久化存储**，基于 PVC 来实现
- 有序的、优雅的部署和扩缩：
	Pod 是有顺序的，在部署或者扩展的时候要依据定义的顺序依次进行（即从 0..N-1，在下一个Pod 运行之前所有之前的 Pod 必须都是 Running 和 Ready 状态），基于 init containers 来实现
- 有序收缩，有序删除
	有序收缩，有序删除（即从 N-1..0）

### 限制
- 给定 Pod 的存储必须由 [PersistentVolume Provisioner](https://kubernetes.io/zh-cn/docs/concepts/storage/dynamic-provisioning/) （[例子在这里](https://github.com/kubernetes/examples/tree/master/staging/persistent-volume-provisioning/README.md)） 基于所请求的 **storage class** 来制备，或者由管理员预先制备。
- 删除或者扩缩 StatefulSet 并**不会**删除它关联的存储卷。 这样做是为了保证数据安全，它通常比自动清除 StatefulSet 所有相关的资源更有价值。
- StatefulSet 当前需要[无头服务](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/#headless-services)来负责 Pod 的网络标识。你需要负责创建此服务。
- 当删除一个 StatefulSet 时，该 StatefulSet 不提供任何终止 Pod 的保证。 为了实现 StatefulSet 中的 Pod 可以有序且体面地终止，可以在删除之前将 StatefulSet 缩容到 0。
- 在默认 [Pod 管理策略](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/statefulset/#pod-management-policies)(`OrderedReady`) 时使用[滚动更新](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/statefulset/#rolling-updates)， 可能进入需要[人工干预](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/statefulset/#forced-rollback)才能修复的损坏状态。

### 注意事项
- kubernetes v1.5 版本以上才支持
- 所有Pod的Volume必须使用PersistentVolume或者是管理员事先创建好
- 为了保证数据安全，删除StatefulSet时不会删除Volume
- StatefulSet 需要一个 Headless Service 来定义 DNS domain，需要在 StatefulSet 之前创建好

### Pod 的 DNS格式
StatefulSet 中每个 Pod 的 DNS 格式为 `statefulSetName-{0..N-1}.serviceName.namespace.svc.cluster.local`
- serviceName 为 Headless Service 的名字
- 0..N-1 为 Pod 所在的序号，从 0 开始到 N-1
- statefulSetName 为 StatefulSet 的名字
- namespace 为服务所在的 namespace，Headless Servic 和 StatefulSet 必须在相同的 namespace
- .cluster.local 为 Cluster Domain
### 组成
Headless Service
	用于定义网络标志（DNS domain）
	Domain Name Server：域名服务
	将域名与 ip 绑定映射关系
	服务名 => 访问路径（域名） => ip
volumeClaimTemplate
	用于创建 PersistentVolumes

资源清单文件
```yml
apiVersion: apps/v1 # API 版本号，apps/v1 是常用的版本
kind: StatefulSet # 资源类型，这里是 StatefulSet
metadata: # 元数据，描述 StatefulSet 的信息
  name: web # StatefulSet 的名称，在同一命名空间中必须唯一
  namespace: default # 所属命名空间，默认为 default
  labels: # 标签，用于组织和选择 StatefulSet
    app: nginx # 自定义标签，可以设置多个
spec: # StatefulSet 的规格，描述 StatefulSet 的期望状态
  replicas: 3 # 期望的副本数量，即 Pod 的数量
  serviceName: "nginx" # Headless Service 的名称，用于为每个 Pod 提供 DNS 记录
  updateStrategy: # 更新策略
    type: RollingUpdate # 滚动更新策略，逐个更新 Pod
    #    type: OnDelete # 删除时更新策略，手动删除 Pod 才会更新
    rollingUpdate:
      partition: 0 # 分区值，用于控制更新的 Pod 范围，0 表示更新所有 Pod
  minReadySeconds: 10 # Pod 就绪的最小时间，防止 Pod 未完全启动就提供服务
  selector: # 选择器，用于选择属于该 StatefulSet 的 Pod
    matchLabels: # 匹配标签
      app: nginx # 必须与 template.metadata.labels 匹配
  template: # Pod 模板，用于创建 Pod
    metadata:
      labels: # Pod 的标签
        app: nginx # 必须与 spec.selector.matchLabels 匹配
    spec: # Pod 的规格
      containers: # 容器列表
        - name: nginx # 容器名称
          image: nginx:1.17.1 # 使用的镜像
          ports: # 端口列表
            - containerPort: 80 # 容器监听的端口
          volumeMounts: # 卷挂载列表
            - name: www # 卷的名称，与 volumeClaimTemplates 中定义的名称对应
              mountPath: /usr/share/nginx/html # 容器内挂载的路径
  volumeClaimTemplates: # 存储卷声明模板，用于为每个 Pod 创建持久化存储卷
    - metadata:
        name: www # 存储卷的名称
      spec:
        accessModes: [ "ReadWriteOnce" ] # 访问模式，ReadWriteOnce 表示只能被一个 Pod 挂载
        storageClassName: "my-storage-class" # 存储类名称，用于动态创建存储卷
        resources: # 资源请求和限制
          requests: # 资源请求
            storage: 1Gi # 请求的存储大小
```
### 创建StatefulSet
创建`statefulset.yaml`，内容如下：
```yml
---
apiVersion: v1 # API 版本：指定 Kubernetes API 的版本
kind: Service # 资源类型：Service（服务），用于暴露应用程序
metadata: # 元数据：资源的描述信息
  name: statefulset-service # Service 的名称：statefulset-service
  labels: # 标签：用于组织和选择 Kubernetes 对象
    app: nginx # 应用标签：标识该 Service 属于哪个应用
spec: # 规格：Service 的期望状态
  ports: # 端口：Service 暴露的端口
    - port: 80 # Service 端口：外部访问 Service 的端口
      targetPort: 80 # 目标端口：Pod 上的容器端口，流量将被转发到此端口。即使是 Headless Service，也建议显式指定。
      name: web # 端口名称：方便引用
  clusterIP: None # 集群IP：设置为 None 表示创建一个 Headless Service，不分配 Cluster IP。Headless Service 用于每个 Pod 拥有一个单独的 DNS 记录。
  selector: # 选择器：用于匹配该 Service 对应的 Pod，流量会转发到匹配的 Pod
    app: nginx # 应用标签：匹配标签 app=nginx 的 Pod
---
apiVersion: apps/v1 # API 版本：指定 Kubernetes API 的版本，apps/v1 适用于 StatefulSet
kind: StatefulSet # 资源类型：StatefulSet（有状态集合），用于管理有状态应用
metadata: # 元数据
  name: statefulset # StatefulSet 的名称：statefulset，必须在namespace中唯一
spec: # 规格
  selector: # 选择器：用于匹配该 StatefulSet 管理的 Pod，必须与 Pod 模板中的 labels 匹配。
    matchLabels: # 匹配标签
      app: nginx # 应用标签：匹配标签 app=nginx 的 Pod
  serviceName: "statefulset-service" # 服务名称：与 Headless Service 的名称匹配，用于创建 Pod 的 DNS 记录。StatefulSet 会为每个 Pod 创建一个 DNS 记录：<pod-name>.<service-name>.<namespace>.svc.<cluster-domain>
  updateStrategy: # 更新策略：滚动更新（RollingUpdate）和删除时更新（OnDelete）
    type: RollingUpdate # 使用滚动更新策略（这是默认策略，StatefulSet 会按顺序逐个更新 Pod，从序号最大的 Pod 开始，直到所有 Pod 都更新完成。）
    rollingUpdate:
      partition: 2 # 设置分区，用于控制更新的 Pod 数量（例如，设置为 2 表示只更新序号大于等于 2 的 Pod。）
      maxUnavailable: 1 # 设置最大不可用 Pod 数量
#  updateStrategy: # 更新策略：滚动更新（RollingUpdate）和删除时更新（OnDelete）
#    type: OnDelete # 使用删除时更新策略（只有当 Pod 被手动删除时，才会触发更新。）
  replicas: 3 # 副本数量：期望运行的 Pod 副本数，这里设置为 2
  minReadySeconds: 10 # Pod就绪的最小时间，防止Pod未完全启动就提供服务，默认为0
  template: # Pod 模板：定义 Pod 的规格
    metadata: # Pod 元数据
      labels: # Pod 标签：必须与 StatefulSet 的 selector 匹配
        app: nginx # 应用标签
    spec: # Pod 规格
      containers: # 容器列表
        - name: nginx # 容器名称
          image: nginx:1.17.1 # 容器镜像
          ports: # 容器端口
            - containerPort: 80 # 容器端口：容器内部监听的端口
              name: web # 端口名称
#          volumeMounts: # 卷挂载：将持久卷挂载到容器内部
#            - name: www # 卷名称：与 volumeClaimTemplates 中定义的卷名称一致
#              mountPath: /usr/share/nginx/html # 挂载路径：容器内部的挂载点
#  volumeClaimTemplates: # 卷声明模板：用于为每个 Pod 创建持久卷
#    - metadata: # 卷元数据
#        name: www # 卷名称：用于在 Pod 中挂载
#        annotations: # 注解：用于添加额外的元数据
#          volume.alpha.kubernetes.io/storage-class: anything # 存储类：指定使用的存储类，这里使用了 alpha 版本的注解，通常应该使用 storageClassName 字段来指定存储类。
#      spec: # 卷规格
#        accessModes: [ "ReadWriteOnce" ] # 访问模式：ReadWriteOnce 表示该卷只能被单个节点上的单个 Pod 以读写模式挂载
##        storageClassName: "my-storage-class" # 存储类名称，用于动态创建存储卷，需要预先创建
#        resources: # 资源请求
#          requests: # 请求的资源量
#            storage: 1Gi # 存储空间：请求 1Gi 的存储空间
```
执行命令
```shell
# 创建
kubectl create -f statefulset.yaml
# 查看 service 和 statefulset => sts
kubectl get service statefulset-service
kubectl get statefulset statefulset
# 查看 PVC 信息
kubectl get pvc
# 查看创建的 pod，这些 pod 是有序的
kubectl get pods -l app=nginx

# 查看这些 pod 的 dns
# 运行一个 pod，基础镜像为 busybox 工具包，利用里面的 nslookup 可以看到 dns 信息
kubectl run -it --image busybox:1.28.4 dns-test --restart=Never -- /bin/sh
# nslookup工具查询POD的DNS
nslookup statefulset-0.statefulset-service
Name:      statefulset-0.statefulset-service
Address 1: 10.244.36.81 statefulset-0.statefulset-service.default.svc.cluster.local

# 删除 StatefulSet 和 Headless Service「级联删除：删除 statefulset 时会同时删除 pods」
kubectl delete statefulset statefulset
# 非级联删除：删除 statefulset 时不会删除 pods，删除 sts 后，pods 就没人管了，此时再删除 pod 不会重建的
kubectl deelte sts statefulset --cascade=false
# 删除 service
kubectl delete service statefulset-service

# StatefulSet删除后PVC还会保留着，数据不再使用的话也需要删除
kubectl delete pvc www-statefulset-0 www-statefulset-1
```
### 扩缩容
```shell
# 扩容
kubectl scale statefulset statefulset --replicas=5
# 监控更新的过程，可以看到pod按照序号升序扩容
kubectl describe sts statefulset
Events:
  Type    Reason            Age   From                    Message
  ----    ------            ----  ----                    -------
  Normal  SuccessfulCreate  33m   statefulset-controller  create Pod statefulset-0 in StatefulSet statefulset successful
  Normal  SuccessfulCreate  33m   statefulset-controller  create Pod statefulset-1 in StatefulSet statefulset successful
  Normal  SuccessfulCreate  32m   statefulset-controller  create Pod statefulset-2 in StatefulSet statefulset successful
  Normal  SuccessfulCreate  69s   statefulset-controller  create Pod statefulset-3 in StatefulSet statefulset successful
  Normal  SuccessfulCreate  49s   statefulset-controller  create Pod statefulset-4 in StatefulSet statefulset successful

# 缩容
kubectl patch statefulset statefulset -p '{"spec":{"replicas":3}}'
# 监控更新的过程，可以看到pod按照序号降序缩容
kubectl describe sts statefulset
Events:
  Type    Reason            Age    From                    Message
  ----    ------            ----   ----                    -------
  Normal  SuccessfulCreate  34m    statefulset-controller  create Pod statefulset-0 in StatefulSet statefulset successful
  Normal  SuccessfulCreate  34m    statefulset-controller  create Pod statefulset-1 in StatefulSet statefulset successful
  Normal  SuccessfulCreate  34m    statefulset-controller  create Pod statefulset-2 in StatefulSet statefulset successful
  Normal  SuccessfulCreate  2m23s  statefulset-controller  create Pod statefulset-3 in StatefulSet statefulset successful
  Normal  SuccessfulCreate  2m3s   statefulset-controller  create Pod statefulset-4 in StatefulSet statefulset successful
  Normal  SuccessfulDelete  2s     statefulset-controller  delete Pod statefulset-4 in StatefulSet statefulset successful
  Normal  SuccessfulDelete  0s     statefulset-controller  delete Pod statefulset-3 in StatefulSet statefulset successful
```

### 更新策略「镜像更新」
#### Deployment 和 StatefulSet 更新策略的区别
| 特性/更新策略 | Deployment                                                   | StatefulSet                          |
|---------|--------------------------------------------------------------|--------------------------------------|
| 更新策略类型  | 滚动更新（Rolling Update）、重建（Recreate）                            | 滚动更新（Rolling Update）和删除时更新（OnDelete） |
| 默认更新策略  | 滚动更新                                                         | 删除时更新                                |
| 更新顺序    | 无序，多个 Pod 并行更新（Rolling Update）；先删除所有旧 Pod，再创建新 Pod（Recreate） | 有序，Pod 逐个更新，从序号大的 Pod 开始             |
| Pod 名称  | 不固定，Pod 被删除后重建，名称可能发生变化                                      | 固定，每个 Pod 有稳定的名称，如 web-0, web-1      |
| 存储      | 通常与 PVC 配合使用，但 Pod 和 PVC 之间没有强关联                             | 每个 Pod 都有唯一的 PVC，Pod 被删除后重建，PVC 仍然存在 |
| 适用场景    | 无状态应用，如 Web 服务器                                              | 有状态应用，如数据库、分布式缓存                     |
#### 滚动更新
1.  编辑`statefulset.yaml`,在spec节点下添加更新策略
```yml
spec:
  updateStrategy: # 更新策略：滚动更新（RollingUpdate）和删除时更新（OnDelete）  
  type: RollingUpdate # 使用滚动更新策略（这是默认策略，StatefulSet 会按顺序逐个更新 Pod，从序号最大的 Pod 开始，直到所有 Pod 都更新完成。）  
  rollingUpdate:  
    partition: 0 # 设置分区，用于控制更新的 Pod 数量（例如，设置为 0 表示只更新序号大于等于 0 的 Pod。）
```
2.  更新进行验证
```shell
# 更新
kubectl apply -f statefulset.yaml
# 通过命令更新pod的image版本
kubectl set image sts statefulset nginx=nginx:1.17.2
# 查看更新过程，可以看到是按照pod序号从高到低
kubectl describe sts statefulset
Events:
  Type    Reason            Age                  From                    Message
  ----    ------            ----                 ----                    -------
  Normal  SuccessfulDelete  52s                  statefulset-controller  delete Pod statefulset-2 in StatefulSet statefulset successful
  Normal  SuccessfulCreate  51s (x2 over 7m50s)  statefulset-controller  create Pod statefulset-2 in StatefulSet statefulset successful
  Normal  SuccessfulDelete  31s                  statefulset-controller  delete Pod statefulset-1 in StatefulSet statefulset successful
  Normal  SuccessfulCreate  29s (x2 over 8m10s)  statefulset-controller  create Pod statefulset-1 in StatefulSet statefulset successful
  Normal  SuccessfulDelete  9s                   statefulset-controller  delete Pod statefulset-0 in StatefulSet statefulset successful
  Normal  SuccessfulCreate  8s (x2 over 8m30s)   statefulset-controller  create Pod statefulset-0 in StatefulSet statefulset successful
# 通过patch命令更新image版本
kubectl patch sts statefulset --type='json' -p='[{"op": "replace", "path": "/spec/template/spec/containers/0/image", "value":"nginx:1.17.2"}]'
```
#### 删除更新
1.  编辑`statefulset.yaml`,在spec节点下添加更新策略
```yml
spec:
  updateStrategy: # 更新策略：滚动更新（RollingUpdate）和删除时更新（OnDelete）
    type: OnDelete # 使用删除时更新策略（只有当 Pod 被手动删除时，才会触发更新。）
```
2.  更新进行验证
```shell
# 更新
kubectl apply -f statefulset.yaml
# 通过命令更新pod的image版本
kubectl set image sts statefulset nginx=nginx:1.17.2
# 查看更新过程，没有发生改变
kubectl describe sts statefulset
Events:
  Type    Reason            Age   From                    Message
  ----    ------            ----  ----                    -------
  Normal  SuccessfulCreate  55s   statefulset-controller  create Pod statefulset-0 in StatefulSet statefulset successful
  Normal  SuccessfulCreate  35s   statefulset-controller  create Pod statefulset-1 in StatefulSet statefulset successful
  Normal  SuccessfulCreate  15s   statefulset-controller  create Pod statefulset-2 in StatefulSet statefulset successful
# 查看pod statefulset-0 的镜像也没有改变
kubectl describe po statefulset-0
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  2m4s  default-scheduler  Successfully assigned default/statefulset-0 to k8s-node1
  Normal  Pulled     2m3s  kubelet            Container image "nginx:1.17.1" already present on machine
  Normal  Created    2m3s  kubelet            Created container nginx
  Normal  Started    2m3s  kubelet            Started container nginx

# 删除pod statefulset-0
kubectl delete po statefulset-0
# 查看pod statefulset-0 的镜像成功改变
kubectl describe po statefulset-0
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  11s   default-scheduler  Successfully assigned default/statefulset-0 to k8s-node1
  Normal  Pulled     10s   kubelet            Container image "nginx:1.17.2" already present on machine
  Normal  Created    10s   kubelet            Created container nginx
  Normal  Started    10s   kubelet            Started container nginx
```
### 金丝雀发布
利用滚动更新中的 partition 属性，可以实现简易的灰度发布的效果  
例如我们有 3 个 pod，如果当前 partition 设置为 2，那么此时滚动更新时，只会更新那些 序号 >= 2 的 pod  
利用该机制，我们可以通过控制 partition 的值，来决定只更新其中一部分 pod，确认没有问题后再主键增大更新的 pod 数量，最终实现全部 pod 更新
1.  编辑`statefulset.yaml`,在spec节点下添加更新策略,将partition改为2,实现只更新一个pod
```yml
spec:
  updateStrategy: # 更新策略：滚动更新（RollingUpdate）和删除时更新（OnDelete）
    type: RollingUpdate # 使用滚动更新策略（这是默认策略，StatefulSet 会按顺序逐个更新 Pod，从序号最大的 Pod 开始，直到所有 Pod 都更新完成。）
    rollingUpdate:
      partition: 2 # 设置分区，用于控制更新的 Pod 数量（例如，设置为 0 表示只更新序号大于等于 0 的 Pod。）
```
2.  更新进行验证
```shell
# 更新
kubectl apply -f statefulset.yaml
# 通过命令更新pod的image版本
kubectl set image sts statefulset nginx=nginx:1.17.2
# 查看更新过程，pod：statefulset-2 镜像进行了变更
kubectl describe sts statefulset
Events:
  Type    Reason            Age               From                    Message
  ----    ------            ----              ----                    -------
  Normal  SuccessfulCreate  85s               statefulset-controller  create Pod statefulset-0 in StatefulSet statefulset successful
  Normal  SuccessfulCreate  65s               statefulset-controller  create Pod statefulset-1 in StatefulSet statefulset successful
  Normal  SuccessfulDelete  3s                statefulset-controller  delete Pod statefulset-2 in StatefulSet statefulset successful
  Normal  SuccessfulCreate  2s (x2 over 45s)  statefulset-controller  create Pod statefulset-2 in StatefulSet statefulset successful
# 查看pod的变化，也只有pod：statefulset-2镜像进行了变更
kubectl describe po statefulset-2
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  58s   default-scheduler  Successfully assigned default/statefulset-2 to k8s-node1
  Normal  Pulled     57s   kubelet            Container image "nginx:1.17.2" already present on machine
  Normal  Created    57s   kubelet            Created container nginx
  Normal  Started    57s   kubelet            Started container nginx
```
## 6.5 Horizontal Pod Autoscaler(HPA)
[Pod 水平自动扩缩 | Kubernetes](https://kubernetes.io/zh-cn/docs/tasks/run-application/horizontal-pod-autoscale/)
在前面的课程中，我们已经可以实现通过手工执行 `kubectl scale` 命令实现Pod扩容或缩容，但是这显然不符合Kubernetes的定位目标–自动化、智能化。 
Kubernetes期望可以实现通过**监测Pod的使用情况**，实现pod数量的**自动调整**，于是就产生了Horizontal Pod Autoscaler（HPA）这种控制器。
HPA可以获取每个Pod利用率，然后和HPA中定义的指标进行对比，同时计算出需要伸缩的具体值，最后实现Pod的数量的调整。
其实HPA与之前的Deployment一样，也属于一种Kubernetes资源对象，它通过追踪分析RC控制的所有目标Pod的负载变化情况，来确定是否需要针对性地调整目标Pod的副本数，这是HPA的实现原理。

通常用于 Deployment，不适用于无法扩/缩容的对象，如 DaemonSet
控制管理器每隔15s（可以通过--horizontal-pod-autoscaler-sync-period修改）查询metrics的资源使用情况
![img](https://weichengjun2.dpdns.org//i/2025/02/15/67b0098400533.png) 

资源清单文件
[HorizontalPodAutoscaler 演练 | Kubernetes](https://kubernetes.io/zh-cn/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/)
```yml
apiVersion: autoscaling/v2 # API 版本
kind: HorizontalPodAutoscaler # 资源类型，这里是 HorizontalPodAutoscaler
metadata:
  name: hpa # HPA 的名称，在同一命名空间中必须唯一
spec:
  minReplicas: 1 # 最小 Pod 数量，HPA 会确保至少有 1 个 Pod 运行
  maxReplicas: 5 # 最大 Pod 数量，HPA 会根据指标的变化自动扩容，但不会超过 5 个 Pod
  scaleTargetRef: # 指定要控制的伸缩目标
    apiVersion: apps/v1 # 目标 API 版本，Deployment 使用 apps/v1
    kind: Deployment # 目标资源类型，这里是 Deployment
    name: hpa-deployment # 目标 Deployment 的名称
  metrics: # 指标列表，HPA 会根据这些指标的变化自动调整副本数量
    - type: Resource # 资源指标，例如 CPU、内存
      resource:
        name: cpu # 指标名称，这里是 CPU
        target:
          type: Utilization # 目标类型，Utilization 表示目标 CPU 使用率百分比
          averageUtilization: 3 # 目标 CPU 使用率百分比，当 Pod 的平均 CPU 使用率超过 3% 时，HPA 会触发扩容
    - type: Resource # 资源指标，例如 CPU、内存
      resource:
        name: memory # 指标名称，这里是内存
        target:
          type: AverageValue # 目标类型，AverageValue 表示目标平均内存使用量
          averageValue: "200Mi" # 目标平均内存使用量，当 Pod 的平均内存使用量超过 200Mi 时，HPA 会触发扩容
```
接下来，我们来做一个实验
### 安装metrics-server
metrics-server可以用来收集集群中的资源使用情况
```shell
# 下载 metrics-server 组件配置文件  
wget https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml -O hpa-metrics-server-components.yaml  
# 修改镜像地址为国内的地址  
sed -i 's/k8s.gcr.io\/metrics-server/registry.cn-hangzhou.aliyuncs.com\/google_containers/g' hpa-metrics-server-components.yaml
# 修改容器的 tls 配置，不验证 tls，在 containers 的 args 参数中增加 --kubelet-insecure-tls 参数(它的作用是禁用 kubelet 和 API 服务器之间的 TLS 证书校验。)
# 安装组件
kubectl apply -f hpa-metrics-server-components.yaml
# 查看 pod 状态
kubectl get pods --all-namespaces | grep metrics
```
创建 `hpa-metrics-server-components.yaml`
```yml
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    k8s-app: metrics-server
    rbac.authorization.k8s.io/aggregate-to-admin: "true"
    rbac.authorization.k8s.io/aggregate-to-edit: "true"
    rbac.authorization.k8s.io/aggregate-to-view: "true"
  name: system:aggregated-metrics-reader
rules:
- apiGroups:
  - metrics.k8s.io
  resources:
  - pods
  - nodes
  verbs:
  - get
  - list
  - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    k8s-app: metrics-server
  name: system:metrics-server
rules:
- apiGroups:
  - ""
  resources:
  - nodes/metrics
  verbs:
  - get
- apiGroups:
  - ""
  resources:
  - pods
  - nodes
  verbs:
  - get
  - list
  - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server-auth-reader
  namespace: kube-system
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: extension-apiserver-authentication-reader
subjects:
- kind: ServiceAccount
  name: metrics-server
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server:system:auth-delegator
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:auth-delegator
subjects:
- kind: ServiceAccount
  name: metrics-server
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    k8s-app: metrics-server
  name: system:metrics-server
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:metrics-server
subjects:
- kind: ServiceAccount
  name: metrics-server
  namespace: kube-system
---
apiVersion: v1
kind: Service
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server
  namespace: kube-system
spec:
  ports:
  - name: https
    port: 443
    protocol: TCP
    targetPort: https
  selector:
    k8s-app: metrics-server
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server
  namespace: kube-system
spec:
  selector:
    matchLabels:
      k8s-app: metrics-server
  strategy:
    rollingUpdate:
      maxUnavailable: 0
  template:
    metadata:
      labels:
        k8s-app: metrics-server
    spec:
      containers:
      - args:
        - --cert-dir=/tmp
        - --secure-port=10250
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --metric-resolution=15s
        - --kubelet-insecure-tls
        image: registry.k8s.io/metrics-server/metrics-server:v0.7.2
        imagePullPolicy: IfNotPresent
        livenessProbe:
          failureThreshold: 3
          httpGet:
            path: /livez
            port: https
            scheme: HTTPS
          periodSeconds: 10
        name: metrics-server
        ports:
        - containerPort: 10250
          name: https
          protocol: TCP
        readinessProbe:
          failureThreshold: 3
          httpGet:
            path: /readyz
            port: https
            scheme: HTTPS
          initialDelaySeconds: 20
          periodSeconds: 10
        resources:
          requests:
            cpu: 100m
            memory: 200Mi
        securityContext:
          allowPrivilegeEscalation: false
          capabilities:
            drop:
            - ALL
          readOnlyRootFilesystem: true
          runAsNonRoot: true
          runAsUser: 1000
          seccompProfile:
            type: RuntimeDefault
        volumeMounts:
        - mountPath: /tmp
          name: tmp-dir
      nodeSelector:
        kubernetes.io/os: linux
      priorityClassName: system-cluster-critical
      serviceAccountName: metrics-server
      volumes:
      - emptyDir: {}
        name: tmp-dir
---
apiVersion: apiregistration.k8s.io/v1
kind: APIService
metadata:
  labels:
    k8s-app: metrics-server
  name: v1beta1.metrics.k8s.io
spec:
  group: metrics.k8s.io
  groupPriorityMinimum: 100
  insecureSkipTLSVerify: true
  service:
    name: metrics-server
    namespace: kube-system
  version: v1beta1
  versionPriority: 100

```
### 准备Service、Deployment
创建`hpa-deployment.yaml`文件，内容如下：
```yml
apiVersion: v1 # API 版本，v1 是核心 API 组
kind: Service # Kubernetes 资源类型，这里是 Service
metadata: # 元数据，描述 Service 的信息
  name: hpa-service # Service 的名称，在同一命名空间中必须唯一
  labels: # 标签，用于组织和选择 Kubernetes 对象
    app: hpa # 应用标签，标识该 Service 属于哪个应用
spec: # Service 的规格，描述 Service 的期望状态
  type: NodePort # Service 类型，这里是 NodePort。NodePort 会在每个 Node 上打开一个静态端口，并将流量转发到 Service 的 Pod。
  ports: # 端口列表
    - port: 80 # Service 暴露的端口，外部访问 Service 的端口
      targetPort: 80 # Pod 监听的端口，流量转发到 Pod 的哪个端口
      name: hpa # 端口名称，方便引用
  selector: # 选择器，用于选择属于该 Service 的 Pod
    app: hpa-deployment # 选择所有具有标签 app: hpa-deployment 的 Pod
---
apiVersion: apps/v1 # API 版本号，指定 Kubernetes API 版本，这里是 apps/v1，用于 Deployment
kind: Deployment # Kubernetes 资源类型，这里是 Deployment，用于创建和管理一组 Pod 的副本
metadata: # 元数据，描述 Deployment 的信息
  name: hpa-deployment # Deployment 的名称，必须是唯一的
spec: # Deployment 规格，描述 Deployment 的期望状态
  strategy: # 策略，定义如何更新 Pod
    type: RollingUpdate # 滚动更新策略，表示使用滚动更新的方式来更新 Pod。
    rollingUpdate: # 滚动更新策略：以滚动方式更新 Pod，逐步替换旧版本。
      maxSurge: 25% # 最大额外副本数：滚动更新过程中，最多可以比期望副本数多出的 Pod 数量，这里设置为 25%。
      maxUnavailable: 25% # 最大不可用副本数：滚动更新过程中，最多可以有多少个 Pod 处于不可用状态，这里设置为 25%。
  replicas: 1 # 副本数量，期望运行的 Pod 副本数，这里设置为 1
  selector: # 选择器，用于匹配该 Deployment 管理的 Pod
    matchLabels: # 匹配标签，使用标签进行匹配
      app: hpa-deployment # 匹配标签，app=hpa-deployment 的 Pod
  template: # Pod 模板，定义每次创建的 Pod 的具体配置
    metadata: # Pod 元数据
      labels: # Pod 标签，必须与 Deployment 的 selector 匹配
        app: hpa-deployment # Pod 标签，app=hpa-deployment
    spec: # Pod 规格
      containers: # 容器列表，Pod 中包含的容器
        - name: nginx # 容器名称，nginx
          image: nginx:1.17.1 # 容器镜像，使用的镜像为 nginx:1.17.1
          resources: # 资源配额，用于设置容器的资源限制和请求
            limits: # 限制资源（上限），容器可以使用的最大资源量
              cpu: "1" # CPU 限制，单位是 core 数，表示最多使用 1 个 CPU core
            requests: # 请求资源（下限），容器启动时请求的最小资源量
              cpu: "100m" # CPU 请求，单位是 millicore，100m 表示 0.1 个 CPU core
```
执行命令
```shell
# 创建deployment
kubectl create -f hpa-deployment.yaml
# 查看
kubectl get deployment,pod,svc -o wide
NAME                             READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS   IMAGES         SELECTOR
deployment.apps/hpa-deployment   1/1     1            1           9m28s   nginx        nginx:1.17.1   app=hpa-deployment

NAME                                  READY   STATUS    RESTARTS   AGE     IP             NODE        NOMINATED NODE   READINESS GATES
pod/hpa-deployment-86b6f47f6b-lnnhk   1/1     Running   0          9m28s   10.244.36.80   k8s-node1   <none>           <none>

NAME                  TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE     SELECTOR
service/hpa-service   NodePort    10.97.183.56   <none>        80:30332/TCP   9m28s   app=hpa-deployment
service/kubernetes    ClusterIP   10.96.0.1      <none>        443/TCP        26h     <none>
```
### 部署HPA
实现 cpu 或内存的监控，首先有个前提条件是该对象必须配置了 resources.requests.cpu 或 resources.requests.memory 才可以，可以配置当 cpu/memory 达到上述配置的百分比后进行扩容或缩容
方式一：命令创建
```shell
# 创建一个 HPA：先准备一个好一个有做资源限制的 deployment
kubectl autoscale deploy hpa-deployment --cpu-percent=20 --min=2 --max=5
```
方式二（**推荐**）：创建`hpa.yaml`文件，内容如下
```yml
apiVersion: autoscaling/v1 # API 版本，autoscaling/v1 是常用的版本
kind: HorizontalPodAutoscaler # 资源类型，这里是 HorizontalPodAutoscaler
metadata:
  name: hpa # HPA 的名称，在同一命名空间中必须唯一
spec:
  minReplicas: 1 # 最小 Pod 数量，HPA 会确保至少有 1 个 Pod 运行
  maxReplicas: 5 # 最大 Pod 数量，HPA 会根据指标的变化自动扩容，但不会超过 5 个 Pod
  targetCPUUtilizationPercentage: 3 # 目标 CPU 使用率百分比，当 Pod 的平均 CPU 使用率超过 3% 时，HPA 会触发扩容
  scaleTargetRef: # 指定要控制的伸缩目标
    apiVersion: apps/v1 # 目标 API 版本，Deployment 使用 apps/v1
    kind: Deployment # 目标资源类型，这里是 Deployment
    name: hpa-deployment # 目标 Deployment 的名称
```
执行命令
```shell
# 创建hpa
kubectl create -f hpa.yaml
horizontalpodautoscaler.autoscaling/hpa created

# 查看hpa
kubectl get hpa
NAME   REFERENCE                   TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
hpa    Deployment/hpa-deployment   0%/3%     1         5        1          7s
```
### 测试
在任意节点循环请求进行压测，然后通过控制台查看hpa和pod的变化
```
while true; do wget -q -O- http://10.97.183.56 > /dev/null ; done
```
hpa变化
```shell
kubectl get hpa -w
NAME   REFERENCE                   TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
hpa    Deployment/hpa-deployment   0%/3%     1         5         1          12s
hpa    Deployment/hpa-deployment   25%/3%    1         5         1          55s
hpa    Deployment/hpa-deployment   64%/3%    1         5         4          70s
hpa    Deployment/hpa-deployment   19%/3%    1         5         5          85s
hpa    Deployment/hpa-deployment   13%/3%    1         5         5          100s
hpa    Deployment/hpa-deployment   0%/3%     1         5         5          115s
hpa    Deployment/hpa-deployment   0%/3%     1         5         5          6m41s
hpa    Deployment/hpa-deployment   0%/3%     1         5         1          6m56s
```
deployment变化
```shell
kubectl get deployment -w
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
hpa-deployment   1/1     1            1           14s
hpa-deployment   1/4     1            1           114s
hpa-deployment   1/4     1            1           114s
hpa-deployment   1/4     1            1           114s
hpa-deployment   1/4     4            1           114s
hpa-deployment   2/4     4            2           117s
hpa-deployment   3/4     4            3           117s
hpa-deployment   4/4     4            4           117s
hpa-deployment   4/5     4            4           2m9s
hpa-deployment   4/5     4            4           2m9s
hpa-deployment   4/5     4            4           2m9s
hpa-deployment   4/5     5            4           2m9s
hpa-deployment   5/5     5            5           2m11s
hpa-deployment   5/1     5            5           7m40s
hpa-deployment   5/1     5            5           7m40s
hpa-deployment   1/1     1            1           7m40s

```
pod变化
```shell
kubectl get pods -w
NAME                              READY   STATUS              RESTARTS   AGE
hpa-deployment-7df9756ccc-bh8dr   1/1     Running             0          11m
hpa-deployment-7df9756ccc-cpgrv   0/1     Pending             0          0s
hpa-deployment-7df9756ccc-8zhwk   0/1     Pending             0          0s
hpa-deployment-7df9756ccc-cpgrv   0/1     ContainerCreating   0          0s
hpa-deployment-7df9756ccc-8zhwk   0/1     ContainerCreating   0          0s
hpa-deployment-7df9756ccc-g56qb   0/1     Pending             0          0s
hpa-deployment-7df9756ccc-fgst7   0/1     Pending             0          0s
hpa-deployment-7df9756ccc-g56qb   0/1     ContainerCreating   0          0s
hpa-deployment-7df9756ccc-fgst7   0/1     ContainerCreating   0          0s
hpa-deployment-7df9756ccc-8zhwk   1/1     Running             0          19s
hpa-deployment-7df9756ccc-cpgrv   1/1     Running             0          47s
hpa-deployment-7df9756ccc-g56qb   1/1     Running             0          48s
hpa-deployment-7df9756ccc-fgst7   1/1     Running             0          66s
hpa-deployment-7df9756ccc-fgst7   1/1     Terminating         0          6m50s
hpa-deployment-7df9756ccc-8zhwk   1/1     Terminating         0          7m5s
hpa-deployment-7df9756ccc-cpgrv   1/1     Terminating         0          7m5s
hpa-deployment-7df9756ccc-g56qb   1/1     Terminating         0          6m50s
```
## 6.6 DaemonSet(DS)
DaemonSet类型的控制器可以保证在集群中的每一台（或指定）节点上都运行一个副本。一般适用于日志收集、节点监控等场景。也就是说，如果一个Pod提供的功能是节点级别的（每个节点都需要且只需要一个），那么这类Pod就适合使用DaemonSet类型的控制器创建。
![img](https://weichengjun2.dpdns.org//i/2025/02/15/67b009844389e.png) 
DaemonSet控制器的特点：
- 每当向集群中添加一个节点时，指定的 Pod 副本也将添加到该节点上
- 当节点从集群中移除时，Pod 也就被垃圾回收了

资源清单文件
```yml
apiVersion: apps/v1 # 版本号
kind: DaemonSet # 类型       
metadata: # 元数据
  name: daemonset # rs名称 
  namespace: default # 所属命名空间 
  labels: #标签
    controller: daemonset
spec: # 详情描述
  revisionHistoryLimit: 3 # 保留历史版本
  updateStrategy: # 更新策略
    type: RollingUpdate # 滚动更新策略
    rollingUpdate: # 滚动更新
      maxUnavailable: 1 # 最大不可用状态的 Pod 的最大值，可以为百分比，也可以为整数
  selector: # 选择器，通过它指定该控制器管理哪些pod
    matchLabels: # Labels匹配规则
      app: nginx-pod
    matchExpressions: # Expressions匹配规则
      - { key: app, operator: In, values: [ nginx-pod ] }
  template: # 模板，当副本数量不足时，会根据下面的模板创建pod副本
    metadata:
      labels:
        app: nginx-pod
    spec:
      containers:
        - name: nginx
          image: nginx:1.17.1
          ports:
            - containerPort: 80
```
创建`daemonset.yaml`，内容如下：
```yml
apiVersion: apps/v1 # API版本：指定 Kubernetes API 的版本，apps/v1 适用于 DaemonSet 等应用资源
kind: DaemonSet # 资源类型：DaemonSet（守护进程集），确保每个节点上运行一个 Pod 副本
metadata: # 元数据：资源的描述信息
  name: daemonset # DaemonSet 的名称：daemonset
spec: # 规格：DaemonSet 的期望状态
  selector: # 选择器：用于匹配该 DaemonSet 管理的 Pod，必须与 Pod 模板中的 labels 匹配。
    matchLabels: # 匹配标签：使用标签进行匹配
      app: logging # 应用标签：匹配标签 app=logging 的 Pod
  template: # Pod 模板：定义 Pod 的规格，DaemonSet 会根据此模板在每个节点上创建 Pod
    metadata: # Pod 元数据
      labels: # Pod 标签：必须与 DaemonSet 的 selector 匹配，用于 DaemonSet 管理 Pod
        app: logging # 应用标签：标识该 Pod 属于哪个应用
      # name: fluentd # Pod 的名称：通常不需要在这里指定，由控制器自动生成，删除此行
    spec: # Pod 规格
      containers: # 容器列表：Pod 中包含的容器
        - name: fluentd-es # 容器名称：fluentd-es
          image: agilestacks/fluentd-elasticsearch:v1.3.0 # 容器镜像：使用的镜像为 agilestacks/fluentd-elasticsearch:v1.3.0
          env: # 环境变量：设置容器内的环境变量
            - name: FLUENTD_ARGS # 环境变量名称：FLUENTD_ARGS
              value: -qq # 环境变量值：-qq（静默模式）
          volumeMounts: # 卷挂载：将持久卷或主机路径挂载到容器内部
            - name: containers # 卷名称：containers
              mountPath: /var/lib/docker/containers # 挂载路径：容器内部的挂载点，用于访问 Docker 容器日志
            - name: varlog # 卷名称：varlog
              mountPath: /var/log # 挂载路径：容器内部的挂载点，用于访问系统日志
      volumes: # 卷列表：定义 Pod 中使用的卷
        - hostPath: # 主机路径卷：将主机上的目录挂载到 Pod 中
            path: /var/lib/docker/containers # 主机路径：/var/lib/docker/containers
          name: containers # 卷名称：containers
        - hostPath: # 主机路径卷
            path: /var/log # 主机路径：/var/log
          name: varlog # 卷名称：varlog
```
执行命令
```shell
# 创建
kubectl create -f daemonset.yaml
daemonset.apps/daemonset created

# 查看
kubectl get ds -o wide
NAME        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE    CONTAINERS   IMAGES                                     SELECTOR
daemonset   2         2         2       2            2           <none>          110s   fluentd-es   agilestacks/fluentd-elasticsearch:v1.3.0   app=logging

# 查看pod,发现在每个Node上都运行一个pod
kubectl get pods -o wide
NAME                               READY   STATUS    RESTARTS      AGE   IP               NODE        NOMINATED NODE   READINESS GATES
daemonset-2qmgr                    1/1     Running   0             77s   10.244.36.74     k8s-node1   <none>           <none>
daemonset-b2pdw                    1/1     Running   0             77s   10.244.169.182   k8s-node2   <none>           <none>

# 删除daemonset
kubectl delete -f daemonset.yaml
daemonset.apps "daemonset" deleted
```

### 可以设置node和pod的亲和性来设置ds需要到哪些容器部署

### 更新策略「镜像更新 」
不建议使用 RollingUpdate，建议使用 OnDelete 模式，这样避免频繁更新 ds

## 6.7 Job
主要用于负责 **批量处理(一次要处理指定数量任务) 短暂的一次性(每个任务仅运行一次就结束)** 任务。Job特点如下：
- 当Job创建的pod执行成功结束时，Job将记录成功结束的pod数量
- 当成功结束的pod达到指定的数量时，Job将完成执行
![img](https://weichengjun2.dpdns.org//i/2025/02/15/67b0098478395.png) 
资源清单文件
```yml
apiVersion: batch/v1 # 版本号
kind: Job # 类型       
metadata: # 元数据
  name: # rs名称 
  namespace: # 所属命名空间 
  labels: #标签
    controller: job
spec: # 详情描述
  completions: 1 # 指定job需要成功运行Pods的次数。默认值: 1
  parallelism: 1 # 指定job在任一时刻应该并发运行Pods的数量。默认值: 1
  activeDeadlineSeconds: 30 # 指定job可运行的时间期限，超过时间还未结束，系统将会尝试进行终止。
  backoffLimit: 6 # 指定job失败后进行重试的次数。默认是6
  manualSelector: true # 是否可以使用selector选择器选择pod，默认是false
  selector: # 选择器，通过它指定该控制器管理哪些pod
    matchLabels: # Labels匹配规则
      app: counter-pod
    matchExpressions: # Expressions匹配规则
      - { key: app, operator: In, values: [ counter-pod ] }
  template: # 模板，当副本数量不足时，会根据下面的模板创建pod副本
    metadata:
      labels:
        app: counter-pod
    spec:
      restartPolicy: Never # 重启策略只能设置为Never或者OnFailure
      containers:
        - name: counter
          image: busybox:1.28
          command: [ "bin/sh","-c","for i in 9 8 7 6 5 4 3 2 1; do echo $i;sleep 2;done" ]
```
关于重启策略设置的说明：
    如果指定为OnFailure，则job会在pod出现故障时重启容器，而不是创建pod，failed次数不变
    如果指定为Never，则job会在pod出现故障时创建新的pod，并且故障pod不会消失，也不会重启，failed次数加1
    如果指定为Always的话，就意味着一直重启，意味着job任务会重复去执行了，当然不对，所以不能设置为Always

创建`job.yaml`，内容如下：
```yml
apiVersion: batch/v1 # API 版本：指定 Kubernetes API 的版本，这里是 batch/v1，用于 Job
kind: Job # Kubernetes 资源类型：Job，用于创建一次性任务
metadata: # 元数据：描述 Job 的信息
  name: job # Job 的名称：job，必须是唯一的
spec: # Job 规格：描述 Job 的期望状态
  manualSelector: true # 手动选择器：true，允许手动选择 Pod。通常用于与自定义控制器或脚本集成，手动管理 Pod 的选择。
  selector: # 选择器：用于选择 Job 的 Pod
    matchLabels: # 匹配标签：使用标签进行匹配
      app: counter-pod # 匹配标签：app=counter-pod 的 Pod
  template: # Pod 模板：定义 Pod 的规格，Job 会根据此模板创建 Pod
    metadata: # Pod 元数据
      labels: # Pod 标签：必须与 Job 的 selector 匹配
        app: counter-pod # Pod 标签：app=counter-pod
    spec: # Pod 规格
      restartPolicy: Never # 重启策略：Never，Pod 运行结束后不重启。对于一次性任务，这是通常的选择。
      containers: # 容器列表：Pod 中包含的容器
        - name: counter # 容器名称：counter
          image: busybox:1.28 # 容器镜像：使用的镜像为 busybox:1.28
          command: [ "bin/sh","-c","for i in 9 8 7 6 5 4 3 2 1; do echo $i;sleep 3;done" ] # 容器启动命令：一个 shell 脚本，循环打印数字 9 到 1，每次间隔 3 秒
```
执行命令
```shell
# 创建job
kubectl create -f job.yaml
job.batch/job created

# 查看job
kubectl get job -o wide -w
NAME     COMPLETIONS   DURATION   AGE   CONTAINERS   IMAGES         SELECTOR
job   0/1           21s        21s   counter      busybox:1.28   app=counter-pod
job   1/1           31s        79s   counter      busybox:1.28   app=counter-pod

# 通过观察pod状态可以看到，pod在运行完毕任务后，就会变成Completed状态
kubectl get pods -w
NAME           READY   STATUS     RESTARTS      AGE
job-rxg96   1/1     Running     0            29s
job-rxg96   0/1     Completed   0            33s

# 接下来，调整下pod运行的总数量和并行数量 即：在spec下设置下面两个选项
completions: 6 # 指定job需要成功运行Pods的次数为6
parallelism: 3 # 指定job并发运行Pods的数量为3
#  然后重新运行job，观察效果，此时会发现，job会每次运行3个pod，总共执行了6个pod
kubectl get pods -w
NAME           READY   STATUS    RESTARTS   AGE
job-684ft   1/1     Running   0          5s
job-jhj49   1/1     Running   0          5s
job-pfcvh   1/1     Running   0          5s
job-684ft   0/1     Completed   0          11s
job-v7rhr   0/1     Pending     0          0s
job-v7rhr   0/1     Pending     0          0s
job-v7rhr   0/1     ContainerCreating   0          0s
job-jhj49   0/1     Completed           0          11s
job-fhwf7   0/1     Pending             0          0s
job-fhwf7   0/1     Pending             0          0s
job-pfcvh   0/1     Completed           0          11s
job-5vg2j   0/1     Pending             0          0s
job-fhwf7   0/1     ContainerCreating   0          0s
job-5vg2j   0/1     Pending             0          0s
job-5vg2j   0/1     ContainerCreating   0          0s
job-fhwf7   1/1     Running             0          2s
job-v7rhr   1/1     Running             0          2s
job-5vg2j   1/1     Running             0          3s
job-fhwf7   0/1     Completed           0          12s
job-v7rhr   0/1     Completed           0          12s
job-5vg2j   0/1     Completed           0          12s

# 删除job
kubectl delete -f job.yaml
job.batch "job" deleted
```
## 6.8 CronJob(CJ)

CronJob控制器以 Job控制器资源为其管控对象，并借助它管理pod资源对象，Job控制器定义的作业任务在其控制器资源创建之后便会立即执行，但CronJob可以以类似于Linux操作系统的周期性任务作业计划的方式控制其运行 **时间点** 及 **重复运行** 的方式。也就是说， **CronJob可以在特定的时间点(反复的)去运行job任务** 。

![img](https://weichengjun2.dpdns.org//i/2025/02/15/67b00984b9c28.png) 
资源清单文件
```yml
apiVersion: batch/v1beta1 # API版本号：指定Kubernetes API的版本，这里是batch/v1beta1，用于CronJob
kind: CronJob # Kubernetes资源类型：CronJob，用于创建定时任务
metadata: # 元数据：描述CronJob的信息
  name: my-cronjob # CronJob名称：my-cronjob，必须是唯一的
  namespace: default # CronJob所属命名空间：default，默认为default命名空间
  labels: # 标签：用于组织和选择Kubernetes对象
    controller: cronjob # 标签：controller=cronjob
spec: # CronJob规格：描述CronJob的期望状态
  schedule: "*/5 * * * *" # Cron表达式：定义任务的调度时间，这里是每隔5分钟执行一次
  concurrencyPolicy: Allow # 并发策略：Allow，允许并发执行，其他选项有：Forbid（禁止并发）、Replace（替换并发）
  failedJobHistoryLimit: 1 # 失败任务历史记录限制：1，保留1个失败任务的历史记录
  successfulJobHistoryLimit: 3 # 成功任务历史记录限制：3，保留3个成功任务的历史记录
  startingDeadlineSeconds: 600 # 启动截止时间：600秒，如果在这个时间内Job没有被调度，则视为失败
  jobTemplate: # Job模板：定义Job的模板，CronJob会根据这个模板创建Job
    metadata: # Job元数据
    spec: # Job规格
      completions: 1 # 完成次数：1，Job需要完成的次数
      parallelism: 1 # 并行度：1，同时运行的Pod数量
      activeDeadlineSeconds: 30 # 活跃截止时间：30秒，Job运行超过30秒则被终止
      backoffLimit: 6 # 重试限制：6，Job失败后最多重试6次
      manualSelector: true # 手动选择器：true，允许手动选择Pod
      selector: # 选择器：用于选择Job的Pod
        matchLabels: # 匹配标签
          app: counter-pod # 匹配标签：app=counter-pod
        matchExpressions: # 匹配表达式
          - { key: app, operator: In, values: [ counter-pod ] } # 匹配表达式：app in (counter-pod)
      template: # Pod模板：定义Pod的规格
        metadata: # Pod元数据
          labels: # Pod标签
            app: counter-pod # Pod标签：app=counter-pod
        spec: # Pod规格
          restartPolicy: Never # 重启策略：Never，Pod运行结束后不重启
          containers: # 容器列表
            - name: counter # 容器名称：counter
              image: busybox:1.28 # 容器镜像：busybox:1.28
              command: [ "bin/sh","-c","for i in 9 8 7 6 5 4 3 2 1; do echo $i;sleep 20;done" ] # 容器启动命令：一个shell脚本，循环打印数字9到1，每次间隔20秒
```

```
需要重点解释的几个选项：
schedule: cron表达式，用于指定任务的执行时间
    *    *      *    *     *
    <分钟> <小时> <日> <月份> <星期>

    分钟 值从 0 到 59.
    小时 值从 0 到 23.
    日 值从 1 到 31.
    月 值从 1 到 12.
    星期 值从 0 到 6, 0 代表星期日
    多个时间可以用逗号隔开； 范围可以用连字符给出；*可以作为通配符； /表示每...
concurrencyPolicy:
    Allow:   允许Jobs并发运行(默认)
    Forbid:  禁止并发运行，如果上一次运行尚未完成，则跳过下一次运行
    Replace: 替换，取消当前正在运行的作业并用新作业替换它
```
创建`cronjob.yaml`，内容如下：
```yml
apiVersion: batch/v1 # API 版本号，指定 Kubernetes API 版本，这里是 batch/v1，用于 CronJob
kind: CronJob # Kubernetes 资源类型，这里是 CronJob，用于创建定时任务
metadata: # 元数据，描述 CronJob 的信息
  name: cronjob # CronJob 的名称，必须是唯一的
  labels: # 标签，用于组织和选择 Kubernetes 对象
    controller: cronjob # 标签，controller=cronjob
spec: # CronJob 规格，描述 CronJob 的期望状态
  concurrencyPolicy: Allow # 并发执行策略，允许多个任务同时运行。其他选项有：
  # - Forbid：禁止并发运行，如果上一个任务还没完成，则不会启动新的任务
  # - Replace：替换并发，如果上一个任务还没完成，则会终止正在运行的任务，并启动新的任务
  failedJobsHistoryLimit: 1 # 保留最近一次失败的任务的历史记录
  successfulJobsHistoryLimit: 3 # 保留最近三次成功的任务的历史记录
  suspend: false # 是否暂停任务，false 表示运行，true 表示暂停
  # startingDeadlineSeconds: 30  # 如果任务因故错过调度时间，则在指定秒数内启动任务。注释掉表示不启用此功能
  schedule: "* * * * *" # 调度表达式，表示每分钟执行一次。这是一个 Cron 表达式，具体含义如下：
  # * * * * *
  # ┬ ┬ ┬ ┬ ┬
  # │ │ │ │ └─ Day of Week (0 - 6) (Sunday = 0 or 7)
  # │ │ │ └─── Month (1 - 12)
  # │ │ └───── Day of Month (1 - 31)
  # │ └─────── Hour (0 - 23)
  # └───────── Minute (0 - 59)
  jobTemplate: # Job 模板，定义每次执行的任务
    metadata:
    spec:
      template: # Pod 模板，定义任务 Pod 的具体配置
        spec:
          restartPolicy: Never # 重启策略，Never 表示任务 Pod 运行结束后不重启
          containers: # 容器列表
            - name: counter # 容器名称
              image: busybox:1.28 # 容器镜像
              command: [ "bin/sh","-c","for i in 9 8 7 6 5 4 3 2 1; do echo $i;sleep 3;done" ] # 容器启动命令
              # 这里是一个 shell 脚本，循环打印 9 到 1 这几个数字，每个数字打印间隔 3 秒
```
执行命令
```shell
# 创建cronjob
kubectl create -f cronjob.yaml
cronjob.batch/cronjob created

# 查看cronjob
kubectl get cronjobs
NAME         SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
cronjob   * * * * *   False     0        <none>          6s

# 查看job
kubectl get jobs
NAME                    COMPLETIONS   DURATION   AGE
cronjob-1592587800   1/1           28s        3m26s
cronjob-1592587860   1/1           28s        2m26s
cronjob-1592587920   1/1           28s        86s

# 查看pod
kubectl get pods
cronjob-1592587800-x4tsm   0/1     Completed   0          2m24s
cronjob-1592587860-r5gv4   0/1     Completed   0          84s
cronjob-1592587920-9dxxq   1/1     Running     0          24s

# 查看执行内容
kubectl logs -f cronjob-1592587800-x4tsm

# 删除cronjob
kubectl  delete -f cronjob.yaml
cronjob.batch "cronjob" deleted
```
# 7. Service详解
创建目录「以下操作均在该目录进行」
```shell
mkdir -p /opt/k8s/service
```
## 7.1 Service介绍
在kubernetes中，pod是应用程序的载体，我们可以通过pod的ip来访问应用程序，但是pod的ip地址不是固定的，这也就意味着不方便直接采用pod的ip对服务进行访问。
为了解决这个问题，kubernetes提供了Service资源，Service会对提供同一个服务的多个pod进行聚合，并且提供一个统一的入口地址。通过访问Service的入口地址就能访问到后面的pod服务。
![img](https://weichengjun2.dpdns.org//i/2025/02/15/67b009852b3bd.png) 
Service在很多情况下只是一个概念，真正起作用的其实是kube-proxy服务进程，每个Node节点上都运行着一个kube-proxy服务进程。当创建Service的时候会通过api-server向etcd写入创建的service的信息，而kube-proxy会基于监听的机制发现这种Service的变动，然后 **它会将最新的Service信息转换成对应的访问规则** 。
![img](https://weichengjun2.dpdns.org//i/2025/02/15/67b0098572c04.png) 
10.97.97.97:80 是service提供的访问入口，当访问这个入口的时候，可以发现后面有三个pod的服务在等待调用，kube-proxy会基于 **rr（轮询）** 的策略，将请求分发到其中一个pod上去
**这个规则会同时在集群内的所有节点上都生成，所以在任何一个节点上，都可以访问。**
```shell
ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  10.97.97.97:80 rr
  -> 10.244.1.39:80               Masq    1      0          0
  -> 10.244.1.40:80               Masq    1      0          0
  -> 10.244.2.33:80               Masq    1      0          0
```
### kube-proxy目前支持三种工作模式:
[虚拟 IP 和服务代理 | Kubernetes](https://kubernetes.io/zh-cn/docs/reference/networking/virtual-ips/#proxy-mode-nftables)
#### 7.1.1 userspace 模式
userspace模式下，kube-proxy会为每一个Service**创建一个监听端口**，发向Cluster IP的请求被Iptables规则重定向到kube-proxy监听的端口上，
kube-proxy根据LB算法选择一个提供服务的Pod并和其建立链接，以将请求转发到Pod上。  该模式下，kube-proxy充当了一个四层负责均衡器的角色。由于kube-proxy运行在userspace中，在进行**转发处理时会增加内核和用户空间之间的数据拷贝**，虽然**稳定，但是效率低**。
![img](https://weichengjun2.dpdns.org//i/2025/02/15/67b00985cbb1b.png) 
#### 7.1.2 iptables 模式
iptables模式下，kube-proxy为service后端的**每个Pod创建对应的iptables规则**，直接将发向Cluster IP的请求重定向到一个Pod IP。  该模式下kube-proxy不承担四层负责均衡器的角色，只负责创建iptables规则。
该模式的优点是较userspace模式**效率高**，但不能提供灵活的LB策略，当后端Pod不可用时也无法进行重试。
![image.png](https://weichengjun2.dpdns.org/i/2025/01/27/67977215dbce5.png)

#### 7.1.3 ipvs 模式
ipvs模式和iptables类似，kube-proxy监控Pod的变化并创建相应的ipvs规则。**ipvs相对iptables转发效率更高**。除此以外，ipvs支持更多的LB算法。
![image.png](https://weichengjun2.dpdns.org/i/2025/01/16/6788bbb868d25.png)
此模式**必须安装ipvs内核模块**，否则会降级为iptables
{ #9dbe5d}

```shell
# 开启ipvs
kubectl edit cm kube-proxy -n kube-system
# 修改mode: "ipvs"
kubectl delete pod -l k8s-app=kube-proxy -n kube-system
# 查看服务调度信息
ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  10.97.97.97:80 rr
  -> 10.244.1.39:80               Masq    1      0          0
  -> 10.244.1.40:80               Masq    1      0          0
  -> 10.244.2.33:80               Masq    1      0          0
```
## 7.2 Service类型
Service 的类型主要有四种
### 1. ClusterIP (默认类型)
- 特点：
    - 这是 Kubernetes 中默认的 Service 类型。
    - 它会为 Service 在集群内部提供一个虚拟 IP（Cluster IP），只能在集群**内部访问**。
    - 通过内部 DNS 解析，集群内部的 Pod 可以通过 Service 的名称访问到后端 Pod。
- 用途：
    - 适用于集群内部的服务通信，例如微服务之间的调用。
### 2. NodePort
- 特点：
    - 在**每个 Node 节点**上开放一个静态端口，将流量路由到 Service。
    - 可以通过 `<NodeIP>:<NodePort>` 的方式从集群**外部访问** Service。
    - NodePort 范围默认为 30000-32767。
- 用途：
    - 适用于需要从集群外部访问 Service 的场景，例如开发测试环境。
### 3. LoadBalancer
- 特点：
    - 在支持外部负载均衡器的云平台上，会自动创建一个外部负载均衡器。
    - 外部流量通过负载均衡器路由到 Service。
    - 通常与云平台集成，例如 AWS ELB、GCP Load Balancer。
- 用途：
    - 适用于生产环境，需要将 Service 暴露给**公网访问**的场景。
### 4. ExternalName
- 特点：
    - 将 Service 映射到一个**外部的域名**（CNAME 记录）。
    - 不创建 Cluster IP，而是通过 DNS 解析将 Service 名称解析为外部域名。
- 用途：
    - 适用于需要将集群内部的服务映射到外部服务的场景，例如访问外部数据库。

资源清单文件
```yml
kind: Service  # 资源类型
apiVersion: v1  # 资源版本
metadata: # 元数据
  name: service # 资源名称
spec: # 描述
  selector: # 标签选择器，用于确定当前service代理哪些pod
    app: nginx
  type: # Service类型，指定service的访问方式
  clusterIP:  # 虚拟服务的ip地址
  sessionAffinity: # session亲和性，支持ClientIP、None两个选项
  ports: # 端口信息
    - protocol: TCP 
      port: 3017  # service端口
      targetPort: 5003 # pod端口
      nodePort: 31122 # 主机端口
```
## 7.3 Service使用
### 7.3.1 实验环境准备
在使用service之前，首先利用Deployment创建出3个pod，注意要为pod设置 `app=svc-pod` 的标签
创建`svc-deployment.yaml`
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: svc-deployment
spec:
  replicas: 3 # 3个pod副本
  selector:
    matchLabels:
      app: svc-pod
  template:
    metadata:
      labels:
        app: svc-pod # 注意要为pod设置 `app=svc-pod` 的标签
    spec:
      containers:
        - name: svc-container
          image: nginx:1.17.1
          ports:
            - containerPort: 80
```
执行命令
```shell
# 切换目录
cd /opt/k8s/service
# 创建
kubectl create -f svc-deployment.yaml
# 查看pod详情
kubectl get pods -o wide --show-labels
NAME                             READY   STATUS     IP            NODE     LABELS
deployment-66cb59b984-8p84h   1/1     Running    10.244.1.39   k8s-node1    app=svc-pod
deployment-66cb59b984-vx8vx   1/1     Running    10.244.2.33   k8s-node2    app=svc-pod
deployment-66cb59b984-wnncx   1/1     Running    10.244.1.40   k8s-node1    app=svc-pod

# 为了方便后面的测试，修改下三台nginx的index.html页面（三台修改的IP地址不一致）
kubectl exec -it deployment-66cb59b984-8p84h -- /bin/sh
echo "10.244.1.39" > /usr/share/nginx/html/index.html
kubectl exec -it deployment-66cb59b984-wnncx -- /bin/sh
echo "10.244.1.40" > /usr/share/nginx/html/index.html
kubectl exec -it deployment-66cb59b984-vx8vx -- /bin/sh
echo "10.244.1.33" > /usr/share/nginx/html/index.html

# 修改完毕之后，访问测试
curl 10.244.1.39
10.244.1.39
curl 10.244.2.33
10.244.2.33
curl 10.244.1.40
10.244.1.40
```
### 7.3.2 ClusterIP类型的Service
创建`svc-clusterip.yaml`文件
```yml
apiVersion: v1 # API 版本，v1 是核心 API 组
kind: Service # Kubernetes 资源类型，这里是 Service
metadata: # 元数据，描述 Service 的信息
  name: svc-clusterip # Service 的名称，在同一命名空间中必须唯一
spec: # Service 的规格，描述 Service 的期望状态
  selector: # 选择器，用于选择属于该 Service 的 Pod
    app: svc-pod # 选择所有具有标签 app: svc-pod 的 Pod
  clusterIP: 10.97.97.97 # Service 的 ClusterIP 地址，如果不指定，Kubernetes 会自动分配一个
  type: ClusterIP # Service 类型，这里是 ClusterIP。ClusterIP 只能在集群内部访问
  ports: # 端口列表
    - port: 80 # Service 暴露的端口，集群内部访问 Service 的端口
      targetPort: 80 # Pod 监听的端口，流量转发到 Pod 的哪个端口
```
执行命令
```shell
# 创建service
kubectl create -f svc-clusterip.yaml
# 查看service
kubectl get svc -o wide
NAME                TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE   SELECTOR
svc-clusterip   ClusterIP   10.97.97.97   <none>        80/TCP    13s   app=svc-pod

# 查看service的详细信息
# 在这里有一个Endpoints列表，里面就是当前service可以负载到的服务入口
kubectl describe svc svc-clusterip
Name:              svc-clusterip
Namespace:         dev
Labels:            <none>
Annotations:       <none>
Selector:          app=svc-pod
Type:              ClusterIP
IP:                10.97.97.97
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         10.244.1.39:80,10.244.1.40:80,10.244.2.33:80
Session Affinity:  None
Events:            <none>

# 查看ipvs的映射规则
ipvsadm -Ln
TCP  10.97.97.97:80 rr
  -> 10.244.1.39:80               Masq    1      0          0
  -> 10.244.1.40:80               Masq    1      0          0
  -> 10.244.2.33:80               Masq    1      0          0

# 访问10.97.97.97:80观察效果
curl 10.97.97.97:80
10.244.2.33


# 创建工具pod通过 service name 进行访问（推荐）
kubectl run -it --image busybox:1.28.4 dns-test --restart=Never -- /bin/sh
wget http://svc-nodeport

# 默认在当前 namespace 中访问，如果需要跨 namespace 访问 pod，则在 service name 后面加上 .<namespace> 即可  
curl http://svc-nodeport.default
```
### 7.3.3 Endpoint
Endpoint是kubernetes中的一个资源对象，存储在etcd中，用来记录一个service对应的所有pod的访问地址，它是根据service配置文件中selector描述产生的。
一个Service由一组Pod组成，这些Pod通过Endpoints暴露出来， **Endpoints是实现实际服务的端点集合** 。换句话说，**service和pod之间的联系是通过endpoints实现的**。
![image-20200509191917069](https://weichengjun2.dpdns.org//i/2025/02/15/67b0098700a0d.png) 
资源清单文件
```yml
apiVersion: v1 # API 版本，通常为 v1
kind: Endpoints # 资源类型，必须为 Endpoints
metadata: # 元数据
  name: my-service # Endpoints 对象的名称，通常与关联的 Service 名称相同
  namespace: default # Endpoints 对象所属的命名空间，通常与关联的 Service 命名空间相同
  labels: # 标签，用于标识和选择 Endpoints 对象
    app: my-app # 示例标签，可以根据实际情况添加更多标签
  annotations: # 注解，用于存储非标识性元数据
    description: "Endpoints for my-service" # 示例注解，描述 Endpoints 对象的作用
subsets: # 后端 Pod 的地址和端口信息列表
  - addresses: # 后端 Pod 的地址列表
      - ip: 10.244.1.2 # Pod 的 IP 地址
        hostname: my-pod-1.example.com # Pod 的主机名（可选）
        nodeName: my-node-1 # Pod 所在的节点名称（可选）
        targetRef: # 指向 Pod 对象的引用（可选）
          kind: Pod # 引用的资源类型，必须为 Pod
          name: my-pod-1 # Pod 的名称
          namespace: default # Pod 所在的命名空间
          uid: "12345678-1234-1234-1234-1234567890ab" # Pod 的 UID
      - ip: 10.244.2.3 # Pod 的 IP 地址
        hostname: my-pod-2.example.com # Pod 的主机名（可选）
        nodeName: my-node-2 # Pod 所在的节点名称（可选）
        targetRef: # 指向 Pod 对象的引用（可选）
          kind: Pod # 引用的资源类型，必须为 Pod
          name: my-pod-2 # Pod 的名称
          namespace: default # Pod 所在的命名空间
          uid: "abcdef01-1234-5678-90ab-1234567890cd" # Pod 的 UID
    ports: # 端口信息列表
      - name: http # 端口名称（可选）
        port: 8080 # 端口号
        protocol: TCP # 协议，通常为 TCP 或 UDP
      - name: https # 端口名称（可选）
        port: 8443 # 端口号
        protocol: TCP # 协议，通常为 TCP 或 UDP
```
#### Service 和 Endpoints 两种关联方式
##### 1. 通过 Service 的 `selector` 字段自动关联（推荐）
这是最常见和推荐的方式。当 Service 的 `spec.selector` 字段不为空时，Kubernetes 会**自动创建**和维护与该 Service 同名的 Endpoints 对象。
- **Service 的 `selector` 字段：** Service 的 `selector` 字段是一个标签选择器，用于匹配具有特定标签的 Pod。
- **Endpoints 对象的自动生成：** Kubernetes 控制器会定期检查集群中的 Pod，并根据 Service 的 `selector` 字段筛选出匹配的 Pod。然后，它会自动创建一个与 Service 同名的 Endpoints 对象，并将匹配的 Pod 的 IP 地址和端口信息添加到 Endpoints 对象的 `subsets` 字段中。
- **关联方式：** Service 通过 `selector` 字段定义的标签选择器与 Endpoints 对象中 `subsets.addresses` 字段引用的 Pod 对象建立关联。
示例
```yml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-app # 选择具有标签 app=my-app 的 Pod
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```
##### 2. 通过手动创建 Endpoints 对象关联
当 Service 的 `spec.selector` 字段为空时，Kubernetes **不会自动创建 Endpoints** 对象，此时需要手动创建 Endpoints 对象，并将 Service 与 Endpoints 对象关联起来。
- **Service 的 `selector` 字段为空：** 当 Service 的 `spec.selector` 字段为空时，表示该 Service 不会自动选择后端 Pod。
- **手动创建 Endpoints 对象：** 需要手动创建一个与 Service 同名的 Endpoints 对象，并在 Endpoints 对象的 `subsets` 字段中指定后端 Pod 的 IP 地址和端口信息。
- **关联方式：** Service **通过名称**与手动创建的 Endpoints 对象建立关联。
**示例：**
```yml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
---
apiVersion: v1
kind: Endpoints
metadata:
  name: my-service # Endpoints 对象与 Service 名称相同
subsets:
  - addresses:
      - ip: 10.244.1.2 # 手动指定的 Pod 的 IP 地址
    ports:
      - port: 8080 # 手动指定的 Pod 的端口
```
#### 负载分发策略
对Service的访问被分发到了后端的Pod上去，目前kubernetes提供了两种负载分发策略：
- 如果不定义，默认使用kube-proxy的策略，比如随机、轮询
- 基于客户端地址的会话保持模式，即来自同一个客户端发起的所有请求都会转发到固定的一个Pod上
	- 此模式可以使在spec中添加 `sessionAffinity:ClientIP` 选项
```shell
# 查看ipvs的映射规则【rr 轮询】
ipvsadm -Ln
TCP  10.97.97.97:80 rr
  -> 10.244.1.39:80               Masq    1      0          0
  -> 10.244.1.40:80               Masq    1      0          0
  -> 10.244.2.33:80               Masq    1      0          0

# 循环访问测试
while true;do curl 10.97.97.97:80; sleep 5; done;
10.244.1.40
10.244.1.39
10.244.2.33
10.244.1.40
10.244.1.39
10.244.2.33
```
1.  编辑`svc-clusterip.yaml`,在spec节点下添加分发策略
```yml
spec: # Service 的规格，描述 Service 的期望状态  
  sessionAffinity: ClientIP # 会话亲和度，默认值是 None，可选值有 None、ClientIP
```
2.  更新进行验证
```shell
# 更新
kubectl apply -f svc-clusterip.yaml
# 查看ipvs规则【persistent 代表持久】
ipvsadm -Ln
TCP  10.97.97.97:80 rr persistent 10800
  -> 10.244.1.39:80               Masq    1      0          0
  -> 10.244.1.40:80               Masq    1      0          0
  -> 10.244.2.33:80               Masq    1      0          0

# 循环访问测试
while true;do curl 10.97.97.97; sleep 5; done;
10.244.2.33
10.244.2.33
10.244.2.33
  
# 删除service
kubectl delete -f svc-clusterip.yaml
```
### 7.3.4 HeadLiness类型的Service
在某些场景中，开发人员可能不想使用Service提供的负载均衡功能，而希望自己来控制负载均衡策略，针对这种情况，kubernetes提供了HeadLiness Service，这类Service不会分配Cluster IP，如果想要访问service，只能通过service的域名进行查询。
创建`svc-headliness.yaml`
```yml
apiVersion: v1
kind: Service
metadata:
  name: svc-headliness
spec:
  selector:
    controller: svc-pod
  type: ClusterIP
  clusterIP: None # 将clusterIP设置为None，即可创建headliness Service
  ports:
    - port: 80
      targetPort: 80
```
执行命令
```shell
# 创建service
kubectl create -f svc-headliness.yaml

# 获取service， 发现CLUSTER-IP未分配
kubectl get svc svc-headliness -o wide
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE   SELECTOR
svc-headliness   ClusterIP   None         <none>        80/TCP    11s   app=svc-pod

# 查看service详情
kubectl describe svc svc-headliness
Name:              svc-headliness
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          app=svc-pod
Type:              ClusterIP
IP:                None
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         10.244.1.39:80,10.244.1.40:80,10.244.2.33:80
Session Affinity:  None
Events:            <none>

# 查看域名的解析情况
kubectl exec -it deployment-66cb59b984-8p84h /bin/sh
# cat /etc/resolv.conf
nameserver 10.96.0.10
search dev.svc.cluster.local svc.cluster.local cluster.local

dig @10.96.0.10 svc-headliness.dev.svc.cluster.local
svc-headliness.dev.svc.cluster.local. 30 IN A 10.244.1.40
svc-headliness.dev.svc.cluster.local. 30 IN A 10.244.1.39
svc-headliness.dev.svc.cluster.local. 30 IN A 10.244.2.33
```
### 7.3.5 NodePort类型的Service
在之前的样例中，创建的Service的ip地址只有集群内部才可以访问，如果希望将Service暴露给集群外部使用，那么就要使用到另外一种类型的Service，称为NodePort类型。NodePort的工作原理其实就是 将service的**端口映射到每个Node**的一个端口上 ，然后就可以通过 `NodeIp:NodePort` 来访问service了。
![img](https://weichengjun2.dpdns.org//i/2025/02/15/67b0098742332.png) 
创建`svc-nodeport.yaml`
```yml
apiVersion: v1
kind: Service
metadata:
  name: svc-nodeport
spec:
  selector:
    app: svc-pod
  type: NodePort # service类型
  ports:
    - port: 80
      nodePort: 30002 # 指定绑定的node的端口(默认的取值范围是：30000-32767), 如果不指定，会默认分配
      targetPort: 80 # 容器的端口
```
执行命令
```shell
# 创建service
kubectl create -f svc-nodeport.yaml

# 查看service
kubectl get svc -o wide
NAME               TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)       SELECTOR
svc-nodeport   NodePort   10.105.64.191   <none>        80:30002/TCP  app=svc-pod
# 用集群外的机器「如：宿主机」访问集群中任意节点IP（10.0.0.101）的30002端口，即可访问到pod
curl 10.0.0.101:30002
```
### 7.3.6 LoadBalancer类型的Service
LoadBalancer和NodePort很相似，目的都是向外部暴露一个端口，区别在于LoadBalancer会在集群的外部再来做一个负载均衡设备，而这个设备需要外部环境支持的，外部服务发送到这个设备上的请求，会被设备负载之后转发到集群中。
使用云服务商（阿里云、腾讯云等）提供的负载均衡器服务
![img](https://weichengjun2.dpdns.org//i/2025/02/15/67b009878b744.png) 
### 7.3.7 ExternalName类型的Service
ExternalName类型的Service用于引入集群外部的服务，它通过 `externalName` 属性指定外部一个服务的地址，然后在集群内部访问此service就可以访问到外部的服务了。
![img](https://weichengjun2.dpdns.org//i/2025/02/15/67b00987c11fc.png) 
#### 创建ExternalName
创建`svc-externalname.yaml`，内容如下：
```yml
apiVersion: v1 # API 版本号，指定 Kubernetes API 版本，这里是 v1，用于 Service 和 Endpoints
kind: Service # Kubernetes 资源类型，这里是 Service，用于将外部流量路由到后端服务
metadata: # 元数据，描述 Service 的信息
  labels: # 标签，用于对 Service 进行分类和选择
    app: svc-externalname # 应用名称，用于标识该服务
  name: svc-externalname # Service 的名称，必须是唯一的
spec: # Service 规格，描述 Service 的期望状态
  type: ExternalName # Service 类型，ExternalName 表示将 Service 映射到一个外部的域名
  externalName: www.wolfcode.cn # 外部域名，需要访问的外部域名。也可以改成IP地址。
--- # YAML 文档分隔符，表示以下是另一个 Kubernetes 资源对象的定义
apiVersion: v1 # API 版本号，指定 Kubernetes API 版本，这里是 v1，用于 Service 和 Endpoints
kind: Endpoints # Kubernetes 资源类型，这里是 Endpoints，用于手动指定 Service 的后端服务地址
metadata: # 元数据，描述 Endpoints 的信息
  labels: # 标签，用于将该 Endpoints 对象与其他资源关联起来
    app: svc-externalname # 应用名称，与 Service 的标签保持一致，用于选择后端服务
  name: svc-externalname # Endpoints 的名称，与 Service 的名称保持一致
subsets: # 后端服务地址列表
  - addresses: # 后端服务的 IP 地址列表
      - ip: 120.78.159.117 # 后端服务的 IP 地址
    ports: # 端口信息，与 Service 的端口信息保持一致
      - name: web # 端口名称，通常与 Service 中定义的端口名称一致
        port: 80 # 端口号
        protocol: TCP # 协议，通常为 TCP
```
执行命令
```shell
# 创建service、endpoints
kubectl create -f svc-externalname.yaml

# 查看svc、ep
kubectl get svc,ep -o wide
NAME                       TYPE           CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE    SELECTOR
service/kubernetes         ClusterIP      10.96.0.1    <none>        443/TCP   9m3s   <none>
service/svc-externalname   ExternalName   <none>       qq.com        <none>    12s    <none>

NAME                            ENDPOINTS           AGE
endpoints/fuseim.pri-ifs        <none>              19d
endpoints/kubernetes            10.0.0.101:6443     9m2s
endpoints/svc-externalname-ep   123.150.76.218:80   12s

# 查看ep详情
kubectl describe ep svc-externalname-ep
Name:         svc-externalname-ep
Namespace:    default
Labels:       app=svc-externalname-ep
Annotations:  <none>
Subsets:
  Addresses:          123.150.76.218
  NotReadyAddresses:  <none>
  Ports:
    Name  Port  Protocol
    ----  ----  --------
    web   80    TCP

Events:  <none>

# 进入工具容器通过 service name 进行访问（推荐）
kubectl run -it --image busybox:1.28.4 dns-test --restart=Never -- wget http://svc-externalname
index.html           100% |**************************************************************************************| 72518   0:00:00 ETA

# 强制删除工具容器
kubectl delete po dns-test --force

# 域名解析
dig @10.96.0.10 svc-externalname.dev.svc.cluster.local
svc-externalname.dev.svc.cluster.local. 30 IN CNAME www.baidu.com.
www.baidu.com.          30      IN      CNAME   www.a.shifen.com.
www.a.shifen.com.       30      IN      A       39.156.66.18
www.a.shifen.com.       30      IN      A       39.156.66.14
```
## 7.4 Ingress介绍
[Ingress | Kubernetes](https://kubernetes.io/zh-cn/docs/concepts/services-networking/ingress/)
在前面课程中已经提到，Service对集群之外暴露服务的主要方式有两种：NotePort和LoadBalancer，但是这两种方式，都有一定的缺点：
- NodePort方式的缺点是会占用很多集群机器的端口，那么当集群服务变多的时候，这个缺点就愈发明显
- LB方式的缺点是每个service需要一个LB，浪费、麻烦，并且需要kubernetes之外设备的支持
基于这种现状，kubernetes提供了Ingress资源对象，Ingress只需要一个NodePort或者一个LB就可以满足暴露多个Service的需求。工作机制大致如下图表示：
![img](https://weichengjun2.dpdns.org//i/2025/02/15/67b0098814d70.png) 
实际上，Ingress相当于一个7层的负载均衡器，是kubernetes对反向代理的一个抽象，它的工作原理类似于Nginx，可以理解成在 **Ingress里建立诸多映射规则，Ingress Controller通过监听这些配置规则并转化成Nginx的反向代理配置 , 然后对外部提供服务** 。
在这里有两个核心概念：
- ingress：kubernetes中的一个对象，作用是定义请求如何转发到service的规则
- ingress controller：具体实现反向代理及负载均衡的程序，对ingress定义的规则进行解析，根据配置的规则来实现请求转发，实现方式有很多，比如Nginx, Contour, Haproxy等等

Ingress（以Nginx为例）的工作原理如下：
1.  用户编写Ingress规则，说明哪个域名对应kubernetes集群中的哪个Service
2.  Ingress控制器动态感知Ingress服务规则的变化，然后生成一段对应的Nginx反向代理配置
3.  Ingress控制器会将生成的Nginx配置写入到一个运行着的Nginx服务中，并动态更新
4.  到此为止，其实真正在工作的就是一个Nginx了，内部配置了用户定义的请求转发规则
![img](https://weichengjun2.dpdns.org//i/2025/02/15/67b0098852a25.png) 
## 7.5 Ingress使用
### 7.5.1 环境准备 搭建ingress环境
[https://kubernetes.github.io/ingress-nginx/deploy/#using-helm](https://kubernetes.github.io/ingress-nginx/deploy/#using-helm)
#### 安装 Helm
```shell
# 创建文件夹
mkdir -p /opt/k8s/helm && cd /opt/k8s/helm
# 下载二进制文件
wget https://get.helm.sh/helm-v3.2.3-linux-amd64.tar.gz -O linux-amd64.tar.gz
# 解压
tar -zxvf linux-amd64.tar.gz
# 将目录下的 helm 程序拷贝到 usr/local/bin/helm
cp -a linux-amd64/helm /usr/local/bin/helm
# 添加阿里云 helm 仓库

# 验证安装的helm
helm version
```
#### 添加 helm 仓库
```shell
# 添加仓库
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
# 查看仓库列表
helm repo list
# 搜索 ingress-nginx
helm search repo ingress-nginx
```
#### 下载安装包
```shell
# 下载安装包
helm pull ingress-nginx/ingress-nginx
# 将下载好的安装包解压
tar xf ingress-nginx-4.12.0.tgz
# 解压后，进入解压完成的目录
cd ingress-nginx
# 修改 values.yaml
vim values.yaml
# 修改内容如下
...
controller:
  name: controller
  enableAnnotationValidations: true
  image:
    chroot: false
    # registry: registry.k8s.io # 改为 registry.cn-hangzhou.aliyuncs.com
    image: ingress-nginx/controller # 改为 google_containers/nginx-ingress-controller
    tag: "v1.12.0" # 改为 v1.3.0
    digest: sha256:e6b8de175acda6ca913891f0f727bca4527e797d52688cbe9fec9040d6f6b6fa # 注释掉
    digestChroot: sha256:87c88e1c38a6c8d4483c8f70b69e2cca49853bb3ec3124b9b1be648edf139af3 # 注释掉
...

...
# registry: registry.k8s.io # 改为 registry.cn-hangzhou.aliyuncs.com
image: ingress-nginx/kube-webhook-certgen # 改为 google_containers/kube-webhook-certgen
tag: v1.5.0 # 改为 v1.3.0
digest: sha256:aaafd456bda110628b2d4ca6296f38731a3aaf0bf7581efae824a41c770a8fc4 # 注释掉
...

...
kind: Deployment # 改为 DaemonSet
nodeSelector: 
  ingress: "true" # 增加选择器，如果 node 上有 ingress=true 就部署
...

# 在生产环境中，建议使用 `LoadBalancer` 或 `NodePort` 类型的 Service 来暴露 Ingress 控制器，而不是依赖于 `hostNetwork: true`。
# 让 Nginx Ingress 控制器 Pod 直接使用宿主机的网络，Pod 的 IP 就是宿主机的 IP, 因此，可以直接用宿主机 IP 访问 Pod(简化了端口暴露，但也带来了潜在的冲突和安全风险。)
hostNetwork: false # 改为 true
# Pod 优先使用 Kubernetes 集群 DNS 解析域名
dnsPolicy: ClusterFirst # 改为 ClusterFirstWithHostNet

将 admissionWebhooks.enabled 修改为 false
将 service 中的 type 由 LoadBalancer 修改为 ClusterIP，如果服务器是云平台才用 LoadBalancer
```
#### 创建 NameSpace
```shell
# 为 ingress 专门创建一个 namespace
kubectl create ns ingress-nginx
```
#### 安装 ingress
```shell
# 为需要部署 ingress 的节点上加标签「后续访问k8s-node1，进行测试」
kubectl label node k8s-node1 ingress=true
# 安装 ingress-nginx
cd /opt/k8s/helm/ingress-nginx && helm install ingress-nginx .
```
#### 查看 ingress
```shell
# 查看ingress-nginx
kubectl get pod -n ingress-nginx -o wide
NAME                             READY   STATUS    RESTARTS       AGE   IP           NODE        NOMINATED NODE   READINESS GATES
ingress-nginx-controller-zst7s   1/1     Running   54 (19h ago)   14d   10.0.0.102   k8s-node1   <none>           <none>

# 查看service
kubectl get svc -n ingress-nginx
NAME                       TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
ingress-nginx-controller   ClusterIP   10.100.66.57   <none>        80/TCP,443/TCP   35d
```
#### 特别注释
即使 `ingress-nginx-controller` Service 的类型是 `ClusterIP`，内网电脑也能正确使用ingress解析域名的关键在于
**1. `hostNetwork: true`（共享宿主机网络）：**
- Pod 直接使用宿主机的网络，Pod 的 IP 就是宿主机的 IP，因此，可以直接用宿主机 IP 访问 Pod。
**2. `dnsPolicy: ClusterFirstWithHostNet`（优先使用集群 DNS）：**
- Pod 优先使用 Kubernetes 集群 DNS 解析域名。
- 如果集群 DNS 解析失败，则使用宿主机 DNS。
- 确保 Pod 在共享宿主机网络时，也能正确解析集群内外域名。
### 7.5.2 准备service和pod
为了后面的实验比较方便，创建如下图所示的模型
![image.png|300](https://weichengjun2.dpdns.org//i/2025/02/16/67b0eac54a794.png)
创建`ingress-nginx.yaml`
```yml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx-pod
  clusterIP: None
  type: ClusterIP
  ports:
    - port: 80
      targetPort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-pod
  template:
    metadata:
      labels:
        app: nginx-pod
    spec:
      containers:
        - name: nginx
          image: nginx:1.17.1
          ports:
            - containerPort: 80
```
执行命令
```shell
# 切换目录
cd /opt/k8s/service/ingress
# 创建
kubectl create -f ingress-nginx.yaml

# 查看
kubectl get svc
NAME             TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
nginx-service    ClusterIP   None         <none>        80/TCP     48s
```
### 7.5.3 Http代理
创建`ingress-http.yaml`
```yml
apiVersion: networking.k8s.io/v1
kind: Ingress # 资源类型为 Ingress
metadata:
  name: ingress-http
  annotations:
    kubernetes.io/ingress.class: "nginx" # 指定 ingress 的类型为 nginx
    nginx.ingress.kubernetes.io/rewrite-target: / # 默认重定向到根路径
spec:
  rules: # ingress 规则配置，可以配置多个
    - host: nginx.com # 域名配置，可以使用通配符 *
      http:
        paths: # 相当于 nginx 的 location 配置，可以配置多个
          - pathType: Prefix  # 路径匹配类型，前缀匹配，即以该路径为前缀的请求都会被转发到后端服务。例如：/api匹配/api/v1、/api/v2等
#         - pathType: Exact   # 精确匹配，请求路径必须与该路径完全一致。例如：/exact只匹配/exact
#         - pathType: ImplementationSpecific  # string: 由Ingress控制器实现的自定义匹配规则，具体规则由Ingress控制器决定
            backend:
              service:
                name: nginx-service # 代理到哪个 service
                port:
                  number: 80 # service 的端口
            path: /api # 等价于 nginx 中的 location 的路径前缀匹配
          - pathType: Exact  # 精确匹配，请求路径必须与该路径完全一致。例如：/Exec只匹配/Exec
            backend:
              service:
                name: nginx-service # 代理到哪个 service
                port:
                  number: 80 # service 的端口
            path: /
    - host: nginx2.com # 域名配置，可以使用通配符 *
      http:
        paths: # 相当于 nginx 的 location 配置，可以配置多个
          - pathType: Prefix  # 路径匹配类型，前缀匹配，即以该路径为前缀的请求都会被转发到后端服务。例如：/api匹配/api/v1、/api/v2等
            backend:
              service:
                name: nginx-service # 代理到哪个 service
                port:
                  number: 80 # service 的端口
            path: /test
```
执行命令
```shell
# 创建
kubectl create -f ingress-http.yaml

# 查看
kubectl get ing ingress-http
NAME           CLASS    HOSTS                  ADDRESS   PORTS   AGE
ingress-http   <none>   nginx.com,nginx2.com             80      17s

# 查看详情
kubectl describe ing ingress-http
...
Rules:
  Host        Path  Backends
  ----        ----  --------
  nginx.com   
              /api   nginx-service:80 (10.244.169.138:80,10.244.169.145:80,10.244.36.75:80)
              /      nginx-service:80 (10.244.169.138:80,10.244.169.145:80,10.244.36.75:80)
  nginx2.com  
              /test   nginx-service:80 (10.244.169.138:80,10.244.169.145:80,10.244.36.75:80)
...

# 接下来,在本地电脑上配置host文件,解析上面的两个域名到10.0.0.102(k8s-node1)上「因为ingress最开始通过标签选择器绑定到了k8s-node1」
vim /etc/hosts
10.0.0.102 nginx.com
10.0.0.102 nginx2.com
# 然后,就可以分别访问 http://nginx.com/api 和 http://nginx2.com/test 查看效果了
curl http://nginx.com/api
curl http://nginx2.com/test
```
### 7.5.4 Https代理
创建证书
```shell
# 生成证书
openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/C=CN/ST=BJ/L=BJ/O=nginx/CN=itheima.com"

# 创建密钥
kubectl create secret tls tls-secret --key tls.key --cert tls.crt
```
创建`ingress-https.yaml`
```yml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-https
spec:
  tls:
    - hosts:
      - nginx.itheima.com
      - tomcat.itheima.com
      secretName: tls-secret # 指定秘钥
  rules:
  - host: nginx.itheima.com
    http:
      paths:
      - path: /
        backend:
          serviceName: nginx-service
          servicePort: 80
  - host: tomcat.itheima.com
    http:
      paths:
      - path: /
        backend:
          serviceName: tomcat-service
          servicePort: 8080
```

```shell
# 创建
kubectl create -f ingress-https.yaml
ingress.extensions/ingress-https created

# 查看
kubectl get ing ingress-https 
NAME            HOSTS                                  ADDRESS         PORTS     AGE
ingress-https   nginx.itheima.com,tomcat.itheima.com   10.104.184.38   80, 443   2m42s

# 查看详情
kubectl describe ing ingress-https 
...
TLS:
  tls-secret terminates nginx.itheima.com,tomcat.itheima.com
Rules:
Host              Path Backends
----              ---- --------
nginx.itheima.com  /  nginx-service:80 (10.244.1.97:80,10.244.1.98:80,10.244.2.119:80)
tomcat.itheima.com /  tomcat-service:8080(10.244.1.99:8080,10.244.2.117:8080,10.244.2.120:8080)
...

# 下面可以通过浏览器访问https://nginx.itheima.com:31335 和 https://tomcat.itheima.com:31335来查看了
```
# 8. 数据存储

在前面已经提到，容器的生命周期可能很短，会被频繁地创建和销毁。那么容器在销毁时，保存在容器中的数据也会被清除。这种结果对用户来说，在某些情况下是不乐意看到的。为了持久化保存容器的数据，kubernetes引入了Volume的概念。

Volume是Pod中能够被多个容器访问的共享目录，它被定义在Pod上，然后被一个Pod里的多个容器挂载到具体的文件目录下，kubernetes通过Volume实现同一个Pod中不同容器之间的数据共享以及数据的持久化存储。Volume的生命容器不与Pod中单个容器的生命周期相关，当容器终止或者重启时，Volume中的数据也不会丢失。

kubernetes的Volume支持多种类型，比较常见的有下面几个：
- 简单存储：EmptyDir、HostPath、NFS
- 高级存储：PV、PVC
- 配置存储：ConfigMap、Secret
## 8.1 基本存储
创建目录「以下操作均在该目录进行」
```shell
mkdir -p /opt/k8s/volume/
```
### 8.1.1 EmptyDir
EmptyDir是最基础的Volume类型，一个EmptyDir就是Host上的一个空目录。
EmptyDir是在Pod被分配到Node时创建的，它的初始内容为空，并且无须指定宿主机上对应的目录文件，因为kubernetes会自动分配一个目录，当Pod销毁时， EmptyDir中的数据也会被永久删除。 EmptyDir用途如下：
- 临时空间，例如用于某些应用程序运行时所需的临时目录，且无须永久保留
- 一个容器需要从另一个容器中获取数据的目录（多容器共享目录）

接下来，通过两个容器之间文件共享的案例来使用一下EmptyDir。
在一个Pod中准备两个容器nginx和busybox，然后声明一个Volume分别挂载到两个容器的目录中，然后nginx容器负责向Volume中写日志，busybox中通过命令将日志内容读到控制台。
![img](https://weichengjun2.dpdns.org//i/2025/02/15/67b00988d3824.png) 
创建一个`volume-emptydir.yaml`
```yml
apiVersion: v1  # Kubernetes API 版本，指定为 v1
kind: Pod  # 资源类型为 Pod，表示一个 Pod
metadata:
  name: volume-emptydir  # Pod 的名称
spec:
  containers: # 定义 Pod 中的容器
    - name: nginx  # 容器名称
      image: nginx:1.17.1  # 使用 nginx:1.17.1 镜像
      ports: # 暴露端口
        - containerPort: 80  # 容器内部端口为 80
      volumeMounts: # 挂载卷
        - name: logs-volume  # 挂载的卷名称
          mountPath: /var/log/nginx  # 挂载到容器内的路径
    - name: busybox  # 另一个容器，用于读取日志
      image: busybox:1.28  # 使用 busybox 镜像
      command: [ "/bin/sh","-c","tail -f /logs/access.log" ]  # 容器启动命令，用于持续读取日志
      volumeMounts:
        - name: logs-volume
          mountPath: /logs
  volumes: # 定义卷
    - name: logs-volume
      emptyDir: { }  # 定义一个空目录卷
```
执行命令
```shell
# 创建Pod
kubectl create -f volume-emptydir.yaml
pod/volume-emptydir created

# 查看pod
kubectl get pods volume-emptydir -o wide
NAME                  READY   STATUS    RESTARTS   AGE      IP       NODE   ...... 
volume-emptydir       2/2     Running   0          97s   10.42.2.9   k8s-node1  ......

# 通过podIp访问nginx
curl 10.42.2.9
......

# 通过kubectl logs命令查看指定容器的标准输出
kubectl logs -f volume-emptydir -c busybox
10.42.1.0 - - [27/Jun/2021:15:08:54 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.29.0" "-"
```
### 8.1.2 HostPath
上节课提到，EmptyDir中数据不会被持久化，它会随着Pod的结束而销毁，如果想简单的将数据持久化到主机中，可以选择HostPath。
HostPath就是将Node主机中一个实际目录挂在到Pod中，以供容器使用，这样的设计就可以保证Pod销毁了，但是数据依据可以存在于Node主机上。
![img](https://weichengjun2.dpdns.org//i/2025/02/15/67b00989259f7.png) 
创建一个volume-hostpath.yaml：
```yml
apiVersion: v1  # Kubernetes API 版本，指定为 v1
kind: Pod  # 资源类型为 Pod，表示一个 Pod
metadata:
  name: volume-hostpath  # Pod 的名称
spec:
  containers: # 定义 Pod 中的容器
    - name: nginx  # 容器名称
      image: nginx:1.17.1  # 使用 nginx:1.17.1 镜像
      ports: # 暴露容器的端口
        - containerPort: 80  # 容器内部的端口为 80
      volumeMounts: # 挂载卷
        - name: logs-volume  # 挂载的卷名称
          mountPath: /var/log/nginx  # 挂载到容器内的路径
    - name: busybox  # 另一个容器，用于读取日志
      image: busybox:1.28  # 使用 busybox 镜像
      command: [ "/bin/sh","-c","tail -f /logs/access.log" ]  # 容器启动命令，用于持续读取日志
      volumeMounts:
        - name: logs-volume
          mountPath: /logs
  volumes: # 定义卷
    - name: logs-volume
      hostPath:
        path: /root/logs  # 主机上的路径
        type: DirectoryOrCreate  # 如果目录不存在，则创建该目录后再挂载
```
关于type的值的一点说明：
- DirectoryOrCreate 目录存在就使用，不存在就先创建后使用「权限755」
- Directory   目录必须存在
- FileOrCreate  文件存在就使用，不存在就先创建后使用「权限644」
- File 文件必须存在 
- Socket  unix套接字必须存在
- CharDevice  字符设备必须存在
- BlockDevice 块设备必须存在

执行命令
```shell
# 创建Pod
kubectl create -f volume-hostpath.yaml
pod/volume-hostpath created

# 查看Pod
kubectl get pods volume-hostpath -o wide
NAME                  READY   STATUS    RESTARTS   AGE   IP             NODE   ......
pod-volume-hostpath   2/2     Running   0          16s   10.42.2.10     k8s-node1  ......

# 访问nginx
curl 10.42.2.10

# 监听日志
kubectl logs -f volume-hostpath -c busybox

# 接下来就可以去host的/root/logs目录下查看存储的文件了
###  注意: 下面的操作需要到Pod所在的节点运行（案例中是k8s-node1）
ls /root/logs/
access.log  error.log

# 同样的道理，如果在此目录下创建一个文件，到容器中也是可以看到的
```
### 8.1.3 NFS
HostPath可以解决数据持久化的问题，但是一旦Node节点故障了，Pod如果转移到了别的节点，又会出现问题了，此时需要准备单独的网络存储系统，比较常用的用NFS、CIFS。
NFS是一个网络文件存储系统，可以搭建一台NFS服务器，然后将Pod中的存储直接连接到NFS系统上，这样的话，无论Pod在节点上怎么转移，只要Node跟NFS的对接没问题，数据就可以成功访问。
![img](https://weichengjun2.dpdns.org//i/2025/02/15/67b0098966f07.png) 
1）首先要准备nfs的服务器，这里为了简单，直接是master节点做nfs服务器
```shell
# 在centos上安装nfs服务
yum install nfs-utils -y

# 准备一个共享目录
mkdir /root/data/nfs -pv

# 将共享目录以读写权限暴露给10.0.0.0/24网段中的所有主机
vim /etc/exports
/root/data/nfs     10.0.0.0/24(rw,no_root_squash)

# 立即使新的配置生效
exportfs -arv
# 重载 NFS 服务器的配置文件，但不重启服务
systemctl reload nfs-server
# 重启并设置 NFS 开机自启
systemctl restart nfs && systemctl enable nfs
```
2）接下来，要在的每个node节点上都安装下nfs，这样的目的是为了node节点可以驱动nfs设备
```shell
# 在node上安装nfs服务，注意不需要启动
yum install nfs-utils -y
```
3）接下来，就可以编写pod的配置文件了，创建volume-nfs.yaml
```yml
apiVersion: v1  # Kubernetes API 版本，指定为 v1
kind: Pod  # 资源类型为 Pod，表示一个 Pod
metadata:
  name: volume-nfs  # Pod 的名称
spec:
  containers: # 定义 Pod 中的容器
    - name: nginx  # 容器名称
      image: nginx:1.17.1  # 使用 nginx:1.17.1 镜像
      ports: # 暴露容器的端口
        - containerPort: 80  # 容器内部的端口为 80
      volumeMounts: # 挂载卷
        - name: logs-volume  # 挂载的卷名称
          mountPath: /var/log/nginx  # 挂载到容器内的路径
    - name: busybox  # 另一个容器，用于读取日志
      image: busybox:1.28  # 使用 busybox 镜像
      command: [ "/bin/sh","-c","tail -f /logs/access.log" ]  # 容器启动命令，用于持续读取日志
      volumeMounts:
        - name: logs-volume
          mountPath: /logs
  volumes: # 定义卷
    - name: logs-volume
      nfs:
        server: 10.0.0.101  # NFS 服务器的 IP 地址
        path: /root/data/nfs  # NFS 服务器上共享的目录
        #readOnly: true # 是否只读
```
4）最后，运行下pod，观察结果
```shell
# 创建pod
kubectl create -f volume-nfs.yaml
pod/volume-nfs created

# 查看pod
kubectl get pods volume-nfs 
NAME                  READY   STATUS    RESTARTS   AGE
volume-nfs        2/2     Running   0          2m9s

# 查看nfs服务器上的共享目录，发现已经有文件了
ls /root/data/nfs
access.log  error.log
```
## 8.2 高级存储
创建目录「以下操作均在该目录进行」
```shell
mkdir -p /opt/k8s/volume/pv/
```
前面已经学习了使用NFS提供存储，此时就要求用户会搭建NFS系统，并且会在yaml配置nfs。由于kubernetes支持的存储系统有很多，要求客户全都掌握，显然不现实。为了能够屏蔽底层存储实现的细节，方便用户使用， kubernetes引入PV和PVC两种资源对象。
- PV（Persistent Volume）是持久化卷的意思，是对底层的共享存储的一种抽象。一般情况下PV由kubernetes管理员进行创建和配置，它与底层具体的共享存储技术有关，并通过插件完成与共享存储的对接。
- PVC（Persistent Volume Claim）是持久卷声明的意思，是用户对于存储需求的一种声明。换句话说，PVC其实就是用户向kubernetes系统发出的一种资源需求申请。
![img](https://weichengjun2.dpdns.org//i/2025/02/15/67b00989d6de5.png) 
使用了PV和PVC之后，工作可以得到进一步的细分：
- 存储：存储工程师维护
- PV： kubernetes管理员维护
- PVC：kubernetes用户维护
### 8.2.1 PV
PV是存储资源的抽象
资源清单文件
```yml
apiVersion: v1  
kind: PersistentVolume
metadata:
  name: pv2
spec:
  nfs: # 存储类型，与底层真正存储对应
  capacity:  # 存储能力，目前只支持存储空间的设置
    storage: 2Gi
  accessModes:  # 访问模式
  storageClassName: # 存储类别
  persistentVolumeReclaimPolicy: # 回收策略
```
PV 的关键配置参数说明：
-  **存储类型** 
	底层实际存储的类型，kubernetes支持多种存储类型，每种存储类型的配置都有所差异
-  **存储能力（capacity）** 
	目前只支持存储空间的设置( storage=1Gi )，不过未来可能会加入IOPS、吞吐量等指标的配置
-  **访问模式（accessModes）** 
	用于描述用户应用对存储资源的访问权限，访问权限包括下面几种方式：
	- ReadWriteOnce（RWO）：读写权限，但是只能被单个节点挂载
	- ReadOnlyMany（ROX）： 只读权限，可以被多个节点挂载
	- ReadWriteMany（RWX）：读写权限，可以被多个节点挂载
	 `需要注意的是，底层不同的存储类型可能支持的访问模式不同` 
-  **回收策略（persistentVolumeReclaimPolicy）** 
	当PV不再被使用了之后，对其的处理方式。目前支持三种策略：
	- Retain （保留） 保留数据，需要管理员手工清理数据
		1. 删除 PersistentVolume 对象。与之相关的、位于外部基础设施中的存储资产 （例如 AWS EBS、GCE PD、Azure Disk 或 Cinder 卷）在 PV 删除之后仍然存在。
		2. 根据情况，手动清除所关联的存储资产上的数据。
		3. 手动删除所关联的存储资产。
	- ~~Recycle（回收） 清除 PV 中的数据，效果相当于执行 `rm -rf /thevolume/*`~~
		** ycle` 已被废弃。取而代之的建议方案是使用动态制备。
	- Delete （删除） 与 PV 相连的后端存储完成 volume 的删除操作，当然这常见于云服务商的存储服务
	 `需要注意的是，底层不同的存储类型可能支持的回收策略不同` 
-  **存储类别**
	 PV可以通过storageClassName参数指定一个存储类别
	- 具有特定类别的PV只能与请求了该类别的PVC进行绑定
	- 未设定类别的PV则只能与不请求任何类别的PVC进行绑定
-  **状态（status）** 
	一个 PV 的生命周期中，可能会处于4中不同的阶段：
	- Available（可用）： 表示可用状态，还未被任何 PVC 绑定
	- Bound（已绑定）： 表示 PV 已经被 PVC 绑定
	- Released（已释放）： 表示 PVC 被删除，但是资源还未被集群重新声明
	- Failed（失败）： 表示该 PV 的自动回收失败

 **实验** 
使用NFS作为存储，来演示PV的使用，创建3个PV，对应NFS中的3个暴露的路径。
1.  准备NFS环境
```shell
# 创建目录
mkdir /root/data/{pv1,pv2,pv3} -pv

# 暴露服务
more /etc/exports
/root/data/pv1 10.0.0.0/24(rw,no_root_squash)
/root/data/pv2 10.0.0.0/24(rw,no_root_squash)
/root/data/pv3 10.0.0.0/24(rw,no_root_squash)

# 应用新的导出配置
exportfs -arv
# 重载 NFS 服务器的配置文件，但不重启服务
systemctl reload nfs-server
```
2.  创建`pv.yaml`
```yml
apiVersion: v1  # API版本，这里是v1
kind: PersistentVolume  # 资源类型，这里是持久卷(PersistentVolume)
metadata: # 元数据
  name: pv1  # 持久卷的名字，这里是pv1
spec: # 规格描述
  capacity: # 容量属性
    storage: 1Gi  # 存储大小，这里是1GiB
  volumeMode: Filesystem # 存储类型为文件系统
  accessModes: # 访问模式
    # 注意：一个PV/PVC只能指定一种或多种访问模式，具体取决于底层存储插件的支持情况。
#    - ReadWriteOnce  # (RWO) 支持被单个节点以读写方式挂载。这是最常见的模式，表示卷可以被单个节点挂载进行读写操作。
#    - ReadOnlyMany   # (ROX) 支持被多个节点以只读方式挂载。表示该卷可以同时被多个节点挂载，但所有挂载都是只读的。
    - ReadWriteMany  # (RWX) 支持被多个节点以读写方式挂载。表示该卷可以同时被多个节点挂载，并且每个挂载都可以进行读写操作。
  storageClassName: slow # 创建 PV 的存储类名，需要与 PVC 的相同
  persistentVolumeReclaimPolicy: Retain  # 回收策略，Retain表示保留数据
  nfs: # NFS配置
    path: /root/data/pv1  # NFS共享目录路径，这里是/root/data/pv1
    server: 10.0.0.101  # NFS服务器地址，这里是10.0.0.101
---
apiVersion: v1  # API版本，这里是v1
kind: PersistentVolume  # 资源类型，这里是持久卷(PersistentVolume)
metadata: # 元数据
  name: pv2  # 持久卷的名字，这里是pv2
spec: # 规格描述
  capacity: # 容量属性
    storage: 2Gi  # 存储大小，这里是2GiB
  volumeMode: Filesystem # 存储类型为文件系统
  accessModes: # 访问模式
    - ReadWriteMany  # 支持多个节点同时读写访问
  storageClassName: slow # 创建 PV 的存储类名，需要与 PVC 的相同
  persistentVolumeReclaimPolicy: Retain  # 回收策略，Retain表示保留数据
  nfs: # NFS配置
    path: /root/data/pv2  # NFS共享目录路径，这里是/root/data/pv2
    server: 10.0.0.101  # NFS服务器地址，这里是10.0.0.101
---
apiVersion: v1  # API版本，这里是v1
kind: PersistentVolume  # 资源类型，这里是持久卷(PersistentVolume)
metadata: # 元数据
  name: pv3  # 持久卷的名字，这里是pv3
spec: # 规格描述
  capacity: # 容量属性
    storage: 3Gi  # 存储大小，这里是3GiB
  volumeMode: Filesystem # 存储类型为文件系统
  accessModes: # 访问模式
    - ReadWriteMany  # 支持多个节点同时读写访问
  storageClassName: slow # 创建 PV 的存储类名，需要与 PVC 的相同
  persistentVolumeReclaimPolicy: Retain  # 回收策略，Retain表示保留数据
  nfs: # NFS配置
    path: /root/data/pv3  # NFS共享目录路径，这里是/root/data/pv3
    server: 10.0.0.101  # NFS服务器地址，这里是10.0.0.101
```
执行命令
```shell
# 创建 pv
kubectl create -f pv.yaml
persistentvolume/pv1 created
persistentvolume/pv2 created
persistentvolume/pv3 created

# 查看pv
kubectl get pv -o wide
NAME   CAPACITY   ACCESS MODES  RECLAIM POLICY  STATUS      AGE   VOLUMEMODE
pv1    1Gi        RWX            Retain        Available    10s   Filesystem
pv2    2Gi        RWX            Retain        Available    10s   Filesystem
pv3    3Gi        RWX            Retain        Available    9s    Filesystem

# 删除指定名称的pv
kubectl get pv -n nexus | grep nexus | awk '{print $1}' | xargs kubectl delete pv -n nexus
```
### 8.2.2 PVC
PVC是资源的申请，用来声明对存储空间、访问模式、存储类别需求信息。
资源清单文件
```yml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc
spec:
  accessModes: # 访问模式
  selector: # 采用标签对PV选择
  storageClassName: # 存储类别
  resources: # 请求空间
    requests:
      storage: 5Gi
```
PVC 的关键配置参数说明：
-  **访问模式（accessModes）**
	用于描述用户应用对存储资源的访问权限，访问权限包括下面几种方式：
	- ReadWriteOnce（RWO）：读写权限，但是只能被单个节点挂载
	- ReadOnlyMany（ROX）： 只读权限，可以被多个节点挂载
	- ReadWriteMany（RWX）：读写权限，可以被多个节点挂载
>注意：底层不同的存储类型可能支持的访问模式不同

用于描述用户应用对存储资源的访问权限
-  **选择条件（selector）** 通过Label Selector的设置，可使PVC对于系统中己存在的PV进行筛选
-  **存储类别（storageClassName）** PVC在定义时可以设定需要的后端存储的类别，只有设置了该class的pv才能被系统选出
-  **资源请求（Resources ）** 描述对存储资源的请求

>PVC与PV绑定的注意事项：
	1.`accessModes`访问模式 必须一致
	2.`volumeModel` 卷模型 必须一致
	3.`storageClassName` 存储类名称 必须一致
	4.requests.storage PVC资源 <= PV资源

 **实验** 
1.  创建pvc.yaml，申请pv
```yml
apiVersion: v1  # API版本，这里是v1
kind: PersistentVolumeClaim  # 资源类型，这里是持久卷声明(PersistentVolumeClaim)
metadata: # 元数据
  name: pvc1  # PVC的名字，这里是pvc1
spec: # 规格描述
  accessModes: # 访问模式，指定此PVC的访问方式
    - ReadWriteMany  # 支持多个节点同时读写访问
  volumeMode: Filesystem # 存储类型为文件系统
  storageClassName: slow # 创建 PVC 的存储类名，需要与 PV 的相同
  resources: # 资源需求
    requests: # 请求的具体资源量
      storage: 1Gi  # 请求的存储空间大小，这里是1GiB
#  selector: # 使用选择器选择对应的 pv
#    matchLabels:
#      release: "stable"
#    matchExpressions:
#      - {key: environment, operator: In, values: [dev]}
---
apiVersion: v1  # API版本，这里是v1
kind: PersistentVolumeClaim  # 资源类型，这里是持久卷声明(PersistentVolumeClaim)
metadata: # 元数据
  name: pvc2  # PVC的名字，这里是pvc2
spec: # 规格描述
  accessModes: # 访问模式，指定此PVC的访问方式
    - ReadWriteMany  # 支持多个节点同时读写访问
  volumeMode: Filesystem # 存储类型为文件系统
  storageClassName: slow # 创建 PVC 的存储类名，需要与 PV 的相同
  resources: # 资源需求
    requests: # 请求的具体资源量
      storage: 1Gi  # 请求的存储空间大小，这里是1GiB
---
apiVersion: v1  # API版本，这里是v1
kind: PersistentVolumeClaim  # 资源类型，这里是持久卷声明(PersistentVolumeClaim)
metadata: # 元数据
  name: pvc3  # PVC的名字，这里是pvc3
spec: # 规格描述
  accessModes: # 访问模式，指定此PVC的访问方式
    - ReadWriteMany  # 支持多个节点同时读写访问
  volumeMode: Filesystem # 存储类型为文件系统
  storageClassName: slow # 创建 PVC 的存储类名，需要与 PV 的相同
  resources: # 资源需求
    requests: # 请求的具体资源量
      storage: 1Gi  # 请求的存储空间大小，这里是1GiB
```
执行命令
```shell
# 创建pvc
kubectl create -f pvc.yaml
persistentvolumeclaim/pvc1 created
persistentvolumeclaim/pvc2 created
persistentvolumeclaim/pvc3 created

# 查看pvc
kubectl get pvc  -o wide
NAME   STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE   VOLUMEMODE
pvc1   Bound    pv1      1Gi        RWX                           15s   Filesystem
pvc2   Bound    pv2      2Gi        RWX                           15s   Filesystem
pvc3   Bound    pv3      3Gi        RWX                           15s   Filesystem

# 查看pv
kubectl get pv -o wide
NAME  CAPACITY ACCESS MODES  RECLAIM POLICY  STATUS    CLAIM       AGE     VOLUMEMODE
pv1    1Gi        RWx        Retain          Bound    dev/pvc1    3h37m    Filesystem
pv2    2Gi        RWX        Retain          Bound    dev/pvc2    3h37m    Filesystem
pv3    3Gi        RWX        Retain          Bound    dev/pvc3    3h37m    Filesystem   

```
2.  创建pod-pv.yaml, 使用pv
```yml
apiVersion: v1  # API版本，这里是v1
kind: Pod  # 资源类型，这里是Pod
metadata: # 元数据
  name: pod1  # Pod的名字，这里是pod1
spec: # 规格描述
  containers: # 容器定义
    - name: busybox  # 容器的名字，这里是busybox
      image: busybox:1.28  # 使用的镜像及其标签，这里是busybox 1.30版本
      command: [ "/bin/sh","-c","while true;do echo pod1 >> /root/out.txt; sleep 10; done;" ]  # 容器启动命令，这里是在/root/out.txt文件中不断追加"pod1"字符串，每10秒一次
      volumeMounts: # 卷挂载配置
        - name: volume  # 指定要挂载的卷的名字，这里是volume
          mountPath: /root/  # 在容器内的挂载点，这里是/root/
  volumes: # 卷定义
    - name: volume  # 卷的名字，与上面container中的volumeMounts中的name对应
      persistentVolumeClaim: # 使用持久卷声明(PVC)作为卷
        claimName: pvc1  # 引用的PVC的名字，这里是pvc1
        readOnly: false  # 是否以只读方式挂载，false表示可读写
---
apiVersion: v1  # API版本，这里是v1
kind: Pod  # 资源类型，这里是Pod
metadata: # 元数据
  name: pod2  # Pod的名字，这里是pod2
spec: # 规格描述
  containers: # 容器定义
    - name: busybox  # 容器的名字，这里是busybox
      image: busybox:1.28  # 使用的镜像及其标签，这里是busybox 1.30版本
      command: [ "/bin/sh","-c","while true;do echo pod2 >> /root/out.txt; sleep 10; done;" ]  # 容器启动命令，这里是在/root/out.txt文件中不断追加"pod2"字符串，每10秒一次
      volumeMounts: # 卷挂载配置
        - name: volume  # 指定要挂载的卷的名字，这里是volume
          mountPath: /root/  # 在容器内的挂载点，这里是/root/
  volumes: # 卷定义
    - name: volume  # 卷的名字，与上面container中的volumeMounts中的name对应
      persistentVolumeClaim: # 使用持久卷声明(PVC)作为卷
        claimName: pvc2  # 引用的PVC的名字，这里是pvc2
        readOnly: false  # 是否以只读方式挂载，false表示可读写
```
执行命令
```shell
# 创建pod
kubectl create -f pod-pv.yaml
pod/pod1 created
pod/pod2 created

# 查看pod
kubectl get pods -o wide
NAME   READY   STATUS    RESTARTS   AGE   IP            NODE   
pod1   1/1     Running   0          14s   10.244.1.69   k8s-node1   
pod2   1/1     Running   0          14s   10.244.1.70   k8s-node1  

# 查看pvc
kubectl get pvc -o wide
NAME   STATUS   VOLUME   CAPACITY   ACCESS MODES      AGE   VOLUMEMODE
pvc1   Bound    pv1      1Gi        RWX               94m   Filesystem
pvc2   Bound    pv2      2Gi        RWX               94m   Filesystem
pvc3   Bound    pv3      3Gi        RWX               94m   Filesystem

# 查看pv
kubectl get pv -o wide
NAME   CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM       AGE     VOLUMEMODE
pv1    1Gi        RWX            Retain           Bound    dev/pvc1    5h11m   Filesystem
pv2    2Gi        RWX            Retain           Bound    dev/pvc2    5h11m   Filesystem
pv3    3Gi        RWX            Retain           Bound    dev/pvc3    5h11m   Filesystem

# 查看nfs中的文件存储
more /root/data/pv1/out.txt
k8s-node1
k8s-node1
more /root/data/pv2/out.txt
k8s-node2
k8s-node2
```
### 8.2.3 生命周期
PVC和PV是一对一的，PV和PVC之间的相互作用遵循以下生命周期：
-  **资源供应** ：管理员手动创建底层存储和PV
-  **资源绑定** ：用户创建PVC，kubernetes负责根据PVC的声明去寻找PV，并绑定在用户定义好PVC之后，系统将根据PVC对存储资源的请求在已存在的PV中选择一个满足条件的
	- 一旦找到，就将该PV与用户定义的PVC进行绑定，用户的应用就可以使用这个PVC了
	- 如果找不到，PVC则会无限期处于Pending状态，直到等到系统管理员创建了一个符合其要求的PV
	- PV一旦绑定到某个PVC上，就会被这个PVC独占，不能再与其他PVC进行绑定了
-  **资源使用** ：用户可在pod中像volume一样使用pvc
	Pod使用Volume的定义，将PVC挂载到容器内的某个路径进行使用。
-  **资源释放** ：用户删除pvc来释放pv
	当存储资源使用完毕后，用户可以删除PVC，与该PVC绑定的PV将会被标记为“已释放”，但还不能立刻与其他PVC进行绑定。通过之前PVC写入的数据可能还被留在存储设备上，只有在清除之后该PV才能再次使用。
-  **资源回收** ：kubernetes根据pv设置的回收策略进行资源的回收
	对于PV，管理员可以设定回收策略，用于设置与之绑定的PVC释放资源之后如何处理遗留数据的问题。只有PV的存储空间完成回收，才能供新的PVC绑定和使用

![img](https://weichengjun2.dpdns.org//i/2025/02/15/67b0098a4beda.png) 

### 8.2.4 StorageClass
[k8s StorageClass 存储类 - misakivv - 博客园](https://www.cnblogs.com/misakivv/p/18430267#%E4%B8%89%E5%AE%9E%E4%BE%8B----%E4%BD%BF%E7%94%A8-nfs-%E7%B1%BB%E5%9E%8B%E7%9A%84-storageclass-%E5%8A%A8%E6%80%81%E5%88%9B%E5%BB%BA-pv)
StorageClass 是 Kubernetes 中用来定义不同类型存储的模板，它告诉 Kubernetes 如何为你的应用提供存储。
#### 概述
**集群级别资源**，StorageClass 是 Kubernetes 中的一种资源对象，它定义了创建 Persistent Volume (PV) 的策略和方法。
StorageClass 主要用于实现 PV 的动态供应，这意味着当用户创建了一个 Persistent Volume Claim (PVC) 时，Kubernetes 会根据所指定的 StorageClass 自动创建一个符合要求的 PV 并将其绑定到 PVC 上

`StorageClass` 作为对存储资源的抽象定义，对用户设置的 `PVC` 申请屏蔽后端存储的细节，一方面减少了用户对于存储资源细节的关注，另一方面减轻了管理员手工管理 `PV` 的工作，由系统自动完成 `PV` 的创建和绑定，实现动态的资源供应。基于 `StorageClass` 的动态资源供应模式将逐步成为云平台的标准存储管理模式。

**作用**
Kubernetes根据用户提交的PVC请求,找到对应的StorageClass, 然后调用该StorageClass声明的存储驱动`Provisioner`,创建出需要的PV供存储使用。
>Provisioner 是一个存储驱动

K8S有两种存储资源的供应模式：静态模式和动态模式，资源供应的最终目的就是将适合的PV与PVC绑定：
**静态模式：** 管理员预先创建许多各种各样的PV，等待PVC申请使用。
**动态模式：** 管理员无须预先创建PV，而是通过StorageClass自动完成PV的创建以及与PVC的绑定。

资源清单文件
```yml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  # 存储类的名称，用户在 PVC 中引用
  name: example-storage-class
  storageclass.kubernetes.io/is-default-class: "true"  # 设置为默认的 StorageClass
# 动态制备器的名称，需要与已安装的制备器匹配
provisioner: kubernetes.io/aws-ebs # 例子中使用的是 AWS EBS 制备器
# reclaimPolicy 定义了当 PVC 被删除时，PV 应该如何处理
reclaimPolicy: Delete # 可选值为 Retain 或 Delete
# 允许卷扩展
allowVolumeExpansion: true # 可选值为 true 或 false
# 定义了卷绑定到 Pod 的模式
volumeBindingMode: Immediate # 可选值为 Immediate 或 WaitForFirstConsumer
# 定义了存储系统需要的参数，这些参数会传递给制备器
parameters:
  type: gp2 # 存储类型，根据制备器和存储系统的要求设置
  iopsPerGB: "10" # 存储 IOPS 性能，某些存储系统可能需要这个参数
  minimumIOPS: "1000" # 存储的最小 IOPS 值，某些存储系统可能需要这个参数
  maximumIOPS: "20000" # 存储的最大 IOPS 值，某些存储系统可能需要这个参数
  encrypted: "true" # 存储的加密选项，某些存储系统可能支持加密
  availabilityZone: "us-east-1a" # 存储的区域，对于跨区域存储系统可能需要这个参数
  performance: "high" # 存储的性能等级，某些存储系统可能提供不同的性能等级
# 定义了挂载选项，这些选项会在 PV 被挂载到节点时使用
mountOptions:
  - debug # 例子，具体值根据需求设置，可能包括 "debug", "defaults", "ro" 等
  # - other-option # 可以添加额外的挂载选项
# 允许的拓扑约束，定义了存储可以被哪些节点访问
allowedTopologies:
  - matchLabelExpressions:
      - key: topology.kubernetes.io/region
        values:
          - us-east-1
      - key: topology.kubernetes.io/zone
        values:
          - us-east-1a
```

![image.png](https://weichengjun2.dpdns.org//i/2025/02/15/67b0098ad450a.png)

#### 8.2.4.1 provisioner
provisioner 指定了用于动态创建 PersistentVolume (PV) 的制备器（Provisioner）。制备器是一个插件，它负责在后端存储系统中根据 PersistentVolumeClaim (PVC) 的请求来创建存储资源，不同的存储插件支持不同的存储后端和服务提供商。当 PVC 被创建并且与之关联的 StorageClass 指定了一个制备器时，Kubernetes 会调用这个制备器来自动创建相应的 PV。

| 卷插件                  | 内置制备器 | 配置示例                                                                                                  |
| -------------------- | ----- | ----------------------------------------------------------------------------------------------------- |
| AWSElasticBlockStore | √     | [AWS EBS](https://kubernetes.io/zh-cn/docs/concepts/storage/storage-classes/#aws-ebs)                 |
| AzureFile            | √     | [Azure文件](https://kubernetes.io/zh-cn/docs/concepts/storage/storage-classes/#azure-file)（已弃用）         |
| AzureDisk            | √     | [Azure Disk](https://github.com/kubernetes-sigs/azuredisk-csi-driver)                                 |
| CephFS               | -     | -                                                                                                     |
| Cinder               | √     | [Open Stack Cinder](https://docs.openstack.org/cinder/latest/)                                        |
| FC                   | -     | -                                                                                                     |
| FlexVolume           | -     | -                                                                                                     |
| GCEPersistentDisk    | √     | [gcePD](https://docs.okd.io/3.11/install_config/persistent_storage/persistent_storage_gce.html)       |
| Glusterfs            | √     | [GlusterFS](https://docs.gluster.org/en/latest/)                                                      |
| iSCSI                | -     | -                                                                                                     |
| Local                | -     | [Local](https://kubernetes.io/zh-cn/docs/concepts/storage/storage-classes/#local)                     |
| NFS                  | -     | [NFS](https://kubernetes.io/zh-cn/docs/concepts/storage/storage-classes/#nfs)                         |
| PortworxVolume       | √     | [portworx-volume](https://kubernetes.io/zh-cn/docs/concepts/storage/storage-classes/#portworx-volume) |
| RBD                  | √     | [ceph-rbd](https://kubernetes.io/zh-cn/docs/concepts/storage/storage-classes/#ceph-rbd)               |
| VsphereVolume        | √     | [Vsphere](https://kubernetes.io/zh-cn/docs/concepts/storage/storage-classes/#vsphere)                 |
##### 8.2.4.1.1、内置制备器
Kubernetes 内置支持的 Provisioner 的命名都以 "kubernetes.io/" 开头
- kubernetes.io/aws-ebs ：用于在 AWS 上创建 Elastic Block Store (EBS) 卷。
- kubernetes.io/azure-disk ：用于在 Azure 上创建磁盘。
- kubernetes.io/gce-pd ：用于在 Google Cloud Platform (GCP) 上创建持久磁盘。
- kubernetes.io/cinder ：用于在 OpenStack 上创建 Cinder 卷。
##### 8.2.4.1.2、第三方制备器
为了符合 StorageClass 的用法，自定义 Provisioner 需要符合存储卷的开发规范，外部存储供应商的作者对代码、提供方式、运行方式、存储插件（包括Flex）等具有完全的自由控制权。

代码仓库 [kubernetes-sigs/sig-storage-lib-external-provisioner](https://github.com/kubernetes-sigs/sig-storage-lib-external-provisioner) 包含一个用于为外部制备器编写功能实现的类库。
你可以访问代码仓库 [kubernetes-sigs/sig-storage-lib-external-provisioner](https://github.com/kubernetes-sigs/sig-storage-lib-external-provisioner) 了解外部驱动列表。

例如，对NFS类型，Kubernetes没有提供内部的Provisioner，但可以使用外部的Provisioner。也有许多第三方存储提供商自行提供外部的Provisioner。

#### 8.2.4.2 reclaimPolicy（回收策略）
通过动态资源供应模式创建的PV将继承在StorageClass资源对象上设置的回收策略，配置字段名称为“reclaimPolicy“，可以设置的选项包括Delete（删除）和 Retain(保留）。
- 如果StorageClass没有指定reclaimPolicy，则默认值为Delete。
- 对于管理员手工创建的仍被StorageClass管理的PV，将使用创建PV时设置的资源回收策略。
#### 8.2.4.3 allowVolumeExpansion（允许卷扩展）
PV 可以被配置为允许扩容，当 StorageClass 资源对象的 allowVolumeExpansion字段被设置为true时，将允许用户通过编辑PVC的存储空间自动完成PV的扩容。

下表描述了支持存储扩容的Volume类型和要求的Kubernetes最低版本：

| 支持存储扩容的 Volume 类型     | Kubernetes 最低版本 |
|-----------------------|-----------------|
| gcePersistentDisk     | 1.11            |
| awsElasticBlock Store | 1.11            |
| Cinder                | 1.11            |
| glusterfs             | 1.11            |
| RBD                   | 1.11            |
| Azure File            | 1.11            |
| Azure Disk            | 1.11            |
| Portworx              | 1.13            |
| FlexVolume            | 1.14(Alpha)     |
| CSI                   | 1.16(Beta)      |
>此功能仅可用于扩容卷，不能用于缩小卷。
#### 8.2.4.4 mountOptions（挂载选项）
通过StorageClass资源对象的mountOptions字段，系统将为动态创建的PV设置挂载选项。
>并不是所有 PV类型都支持挂载选项，如果 PV不支持但 StorageClass 设置了该字段，则 PV将会创建失败。另外，系统不会对挂载选项进行验证，如果设置了错误的选项，则容器在挂载存储时将直接失败。
#### 8.2.4.5  volumeBindingMode（卷绑定模式）
StorageClass 资源对象的 volumeBindingMode 字段设置用于控制何时将 PVC 与动态创建的 PV 绑定。
目前支持的绑定模式包括： Immediate 和 WaitForFirstConsumer。
##### 8.2.4.5.1、Immediate
存储绑定模式的默认值为 Immediate，表示当一个PersistentVolumeClaim (PVC)创建出来时，就动态创建PV并进行PVC与PV的绑定操作。
需要注意的是，对于拓扑受限 (Topology-limited) 或无法从全部Node访问的后端存储，将在不了解Pod调度需求的情况下完成PV的绑定操作，这可能会导致某些Pod无法完成调度。

##### 8.2.4.5.2、WaitForFirstConsumer
WaitForFirstConsumer绑定模式表示PVC与PV的绑定操作延迟到第一个使用 PVC的Pod创建出来时再进行。

系统将根据Pod的调度需求，在Pod所在的Node上创建PV，这些调度需求可以通过以下条件（不限于）进行设置:
- Pod对资源的需求
- Node Selector
- Pod亲和性和反亲和性设置
- Taint和Toleration设置
目前支持 WaitForFirstConsumer 绑定模式的存储卷包括：
- AWSElasticBlockStore
- AzureDisk
- GCEPersistentDisk.
另外,有些存储插件通过预先创建好的PV绑定支持WaitForFirstConsumer模式，比如:
- AWSElasticBlockStore
- AzureDisk
- GCEPersistentDisk
- Local
#### 8.2.4.6  allowedTopologies（允许的拓扑约束）
在使用WaitForFirstConsumer模式的环境中，如果仍然希望基于特定拓扑信息（Topology）进行PV绑定操作，则在StorageClass的定义中还可以通过 allowedTopologies字段进行设置。

#### 8.2.4.7  parameters（存储参数）
后端存储资源提供者的参数设置，不同的 Provisioner 可能提供不同的参数设置。某些参数可以不显示设定，Provisioner 将使用其默认值。
目前 StorageClass 资源对象支持设置的存储参数最多为 512个，全部 key 和 value 所占的空间不能超过 256KiB。
[AWS EBS 存储参数(AWSElasticBlockStore)](https://github.com/kubernetes-sigs/aws-ebs-csi-driver/blob/master/docs/parameters.md)

| Parameters                   | Values                                             | Default | Description                                                                                                                                                                                                                                                                                                                                                                                    |
| ---------------------------- | -------------------------------------------------- | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| "csi.storage.k8s.io/fstype"  | xfs, ext3, ext4                                    | ext4    | File system type that will be formatted during volume creation. This parameter is case sensitive!                                                                                                                                                                                                                                                                                              |
| "type"                       | io1, io2, gp2, gp3, sc1, st1, standard, sbp1, sbg1 | gp3*    | EBS volume type.                                                                                                                                                                                                                                                                                                                                                                               |
| "iopsPerGB"                  |                                                    |         | I/O operations per second per GiB. Can be specified for IO1, IO2, and GP3 volumes.                                                                                                                                                                                                                                                                                                             |
| "allowAutoIOPSPerGBIncrease" | true, false                                        | false   | When `"true"`, the CSI driver increases IOPS for a volume when `iopsPerGB * <volume size>` is too low to fit into IOPS range supported by AWS. This allows dynamic provisioning to always succeed, even when user specifies too small PVC capacity or `iopsPerGB` value. On the other hand, it may introduce additional costs, as such volumes have higher IOPS than requested in `iopsPerGB`. |
| "iops"                       |                                                    |         | I/O operations per second. Can be specified for IO1, IO2, and GP3 volumes.                                                                                                                                                                                                                                                                                                                     |
| "throughput"                 |                                                    | 125     | Throughput in MiB/s. Only effective when gp3 volume type is specified. If empty, it will set to 125MiB/s as documented [here](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volume-types.html).                                                                                                                                                                                      |
| "encrypted"                  | true, false                                        | false   | Whether the volume should be encrypted or not. Valid values are "true" or "false".                                                                                                                                                                                                                                                                                                             |
| "blockExpress"               | true, false                                        | false   | Enables the creation of [io2 Block Express volumes](https://aws.amazon.com/ebs/provisioned-iops/#Introducing_io2_Block_Express) by increasing the IOPS limit for io2 volumes to 256000. Volumes created with more than 64000 IOPS will fail to mount on instances that do not support io2 Block Express.                                                                                       |
| "kmsKeyId"                   |                                                    |         | The full ARN of the key to use when encrypting the volume. If not specified, AWS will use the default KMS key for the region the volume is in. This will be an auto-generated key called `/aws/ebs` if not changed.                                                                                                                                                                            |
| "blockSize"                  |                                                    |         | The block size to use when formatting the underlying filesystem. Only supported on linux nodes and with fstype `ext3`, `ext4`, or `xfs`.                                                                                                                                                                                                                                                       |
| "inodeSize"                  |                                                    |         | The inode size to use when formatting the underlying filesystem. Only supported on linux nodes and with fstype `ext3`, `ext4`, or `xfs`.                                                                                                                                                                                                                                                       |
| "bytesPerInode"              |                                                    |         | The `bytes-per-inode` to use when formatting the underlying filesystem. Only supported on linux nodes and with fstype `ext3`, `ext4`.                                                                                                                                                                                                                                                          |
| "numberOfInodes"             |                                                    |         | The `number-of-inodes` to use when formatting the underlying filesystem. Only supported on linux nodes and with fstype `ext3`, `ext4`.                                                                                                                                                                                                                                                         |
| "ext4BigAlloc"               | true, false                                        | false   | Changes the `ext4` filesystem to use clustered block allocation by enabling the `bigalloc` formatting option. Warning: `bigalloc` may not be fully supported with your node's Linux kernel. Please see our [FAQ](https://github.com/kubernetes-sigs/aws-ebs-csi-driver/blob/master/docs/faq.md).                                                                                               |
| "ext4ClusterSize"            |                                                    |         | The cluster size to use when formatting an `ext4` filesystem when the `bigalloc` feature is enabled. Note: The `ext4BigAlloc` parameter must be set to true. See our [FAQ](https://github.com/kubernetes-sigs/aws-ebs-csi-driver/blob/master/docs/faq.md).                                                                                                                                     |
- **type** (必需)
    - 定义 EBS 卷的存储类型（默认值gp3）。例如：
        - gp2：通用目的 SSD
        - io1：提供高 IOPS 的 SSD
        - io2：适用于需要大量 IOPS 的应用程序的新一代 SSD
        - st1：通过 HDD 存储优化的卷
        - sc1：通过冷 HDD 存储优化的卷
- **iopsPerGB** (可选，仅当 type 为 io1 或 io2 时有效)
    - 定义每个 GiB 提供的 IOPS 数量。
    - 例如，如果 type 为 io1 并且 iopsPerGB 设置为 10，则 100 GiB 的卷将提供 1000 IOPS。
- **iops** (可选，仅当 type 为 io1 时有效)
    - 直接定义卷的 IOPS 总数。例如，iops: "1000" 表示卷将提供 1000 IOPS。
- **throughput** (可选，仅当 type 为 st1 时有效)
    - 定义卷的吞吐量，以 MiB/s 为单位。例如，throughput: "500" 表示卷的吞吐量为 500 MiB/s。
- **encrypted** (可选)
    - 布尔值，指示是否应该加密 EBS 卷。例如，encrypted: "true"。
- **kmsKeyId** (可选)
    - 指定用于加密 EBS 卷的 KMS 密钥 ID。例如，kmsKeyId: "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab"。
- **fsType** (可选)
    - 定义文件系统类型。例如，fsType: "ext4"。
- **volumeSize** (可选)
    - 定义请求的卷大小（以 GiB 为单位）。例如，volumeSize: "100" 表示请求 100 GiB 的卷。
- **availabilityZone** (可选)
    - 定义 EBS 卷应该创建在哪个可用区。例如，availabilityZone: "us-west-2a"。
- **multiAttachEnabled** (可选)
    - 布尔值，指示是否启用多附加。例如，multiAttachEnabled: "true" 允许卷同时附加到多个实例。
- **snapshotId** (可选)
    - 定义用于创建 EBS 卷的快照 ID。例如，snapshotId: "snap-0123456789abcdef0"。
- **tags** (可选)
    - 定义一组键值对，用于标记 EBS 卷。例如：
```yml
tags:
  - key: "project"
    value: "myproject"
  - key: "owner"
    value: "myteam"
```
#### 8.2.4.8 设置默认的 StorageClass（storageclass.kubernetes.io/is-default-class）
在创建 SC 的 YAML 文件时，需要在 metadata 部分添加一个注解，以标记该 SC 为默认。
```shell
# 将存在的 SC 设置默认
kubectl patch storageclass <sc-name> -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
# 将存在的 SC 取消默认
kubectl patch storageclass <sc-name> -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"false"}}}'
```
>如果你在集群中的多个 StorageClass 上将 storageclass.kubernetes.io/is-default-class 注解设置为 true，
>然后创建一个未设置 storageClassName 的 PersistentVolumeClaim (PVC)， Kubernetes 将使用**最近创建的默认 StorageClass**。
#### 8.2.4.9 实验 
##### 自动创建PV
###### **配置NFS**
```shell
# master 节点创建共享目录
mkdir -p /root/data/sc
# 添加目录权限
echo "/root/data/sc 10.0.0.0/24(rw,no_root_squash)" > /etc/exports
# 应用新的导出配置
exportfs -arv
# 重载 NFS 服务器的配置文件，但不重启服务
systemctl reload nfs-server
```
###### **RBAC 权限**
`nfs-rbac.yaml`
```yml
apiVersion: v1  # API版本，指向v1版本的core API组
kind: ServiceAccount  # 资源类型，这里是ServiceAccount
metadata: # 元数据
  name: nfs-client-provisioner  # ServiceAccount的名字，这里是nfs-client-provisioner
---
kind: ClusterRole  # 资源类型，这里是ClusterRole
apiVersion: rbac.authorization.k8s.io/v1  # API版本，指向rbac.authorization.k8s.io组的v1版本API
metadata: # 元数据
  name: nfs-client-provisioner-runner  # ClusterRole的名字，这里是nfs-client-provisioner-runner
rules: # 规则列表
  - apiGroups: [ "" ]  # API组，空字符串表示核心API组
    resources: [ "persistentvolumes" ]  # 资源类型，这里是PersistentVolumes
    verbs: [ "get", "list", "watch", "create", "delete" ]  # 允许的操作，包括获取、列出、监视、创建和删除
  - apiGroups: [ "" ]
    resources: [ "persistentvolumeclaims" ]  # 资源类型，这里是PersistentVolumeClaims
    verbs: [ "get", "list", "watch", "update" ]  # 允许的操作，包括获取、列出、监视和更新
  - apiGroups: [ "storage.k8s.io" ]  # API组，这里是storage.k8s.io
    resources: [ "storageclasses" ]  # 资源类型，这里是StorageClasses
    verbs: [ "get", "list", "watch" ]  # 允许的操作，包括获取、列出和监视
  - apiGroups: [ "" ]
    resources: [ "events" ]  # 资源类型，这里是Events
    verbs: [ "create", "update", "patch" ]  # 允许的操作，包括创建、更新和补丁
---
kind: ClusterRoleBinding  # 资源类型，这里是ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1  # API版本，指向rbac.authorization.k8s.io组的v1版本API
metadata: # 元数据
  name: run-nfs-client-provisioner  # ClusterRoleBinding的名字，这里是run-nfs-client-provisioner
subjects: # 主体列表
  - kind: ServiceAccount  # 主体类型，这里是ServiceAccount
    name: nfs-client-provisioner  # 主体名字，这里是nfs-client-provisioner
roleRef: # 角色引用
  kind: ClusterRole  # 角色类型，这里是ClusterRole
  name: nfs-client-provisioner-runner  # 角色名字，这里是nfs-client-provisioner-runner
  apiGroup: rbac.authorization.k8s.io  # API组，这里是rbac.authorization.k8s.io
---
kind: Role  # 资源类型，这里是Role
apiVersion: rbac.authorization.k8s.io/v1  # API版本，指向rbac.authorization.k8s.io组的v1版本API
metadata: # 元数据
  name: leader-locking-nfs-client-provisioner  # Role的名字，这里是leader-locking-nfs-client-provisioner
rules: # 规则列表
  - apiGroups: [ "" ]  # API组，空字符串表示核心API组
    resources: [ "endpoints" ]  # 资源类型，这里是Endpoints
    verbs: [ "get", "list", "watch", "create", "update", "patch" ]  # 允许的操作，包括获取、列出、监视、创建、更新和补丁
---
kind: RoleBinding  # 资源类型，这里是RoleBinding
apiVersion: rbac.authorization.k8s.io/v1  # API版本，指向rbac.authorization.k8s.io组的v1版本API
metadata: # 元数据
  name: leader-locking-nfs-client-provisioner  # RoleBinding的名字，这里是leader-locking-nfs-client-provisioner
subjects: # 主体列表
  - kind: ServiceAccount  # 主体类型，这里是ServiceAccount
    name: nfs-client-provisioner  # 主体名字，这里是nfs-client-provisioner
roleRef: # 角色引用
  kind: Role  # 角色类型，这里是Role
  name: leader-locking-nfs-client-provisioner  # 角色名字，这里是leader-locking-nfs-client-provisioner
  apiGroup: rbac.authorization.k8s.io  # API组，这里是rbac.authorization.k8s.io
```
###### **存储类**
`nfs-storage.yaml`
```yml
apiVersion: storage.k8s.io/v1  # API组版本，这里指向storage.k8s.io组的v1版本API
kind: StorageClass  # 资源类型，这里是StorageClass
metadata: # 元数据
  name: nfs-storage  # StorageClass的名字，这里是managed-nfs-storage
provisioner: fuseim.pri/ifs  # 存储驱动
parameters: # 参数配置
  archiveOnDelete: "false"  # 删除时是否归档。false表示不归档，删除PVC时会删除对应的数据；true表示归档，在删除时重命名旧路径而不是直接删除数据
reclaimPolicy: Retain  # 回收策略。默认是Delete，这意味着当PVC被删除时，相关的PV也会被自动删除。设置为Retain则在PVC被删除后保留PV和其上的数据
volumeBindingMode: Immediate  # 卷绑定模式。默认值为Immediate，表示一旦创建了PVC就立即尝试绑定到一个合适的PV。某些存储提供者（如azuredisk和AWS EBS）支持WaitForFirstConsumer模式，该模式会在第一个Pod使用此卷之前延迟PV的绑定和配置
```
###### **创建 provisioner**
`nfs-provisioner.yaml`
>部署 NFS Client Provisioner，这是一个 Kubernetes 外部存储插件，用于动态创建 NFS PV。
```yml
apiVersion: apps/v1  # API版本，指向apps组的v1版本API
kind: Deployment  # 资源类型，这里是Deployment
metadata: # 元数据
  name: nfs-provisioner  # Deployment的名字，这里是nfs-provisioner
  labels: # 标签
    app: nfs-provisioner  # 给此Deployment打上标签app=nfs-provisioner
spec: # 规格描述
  replicas: 1  # 副本数，这里设置为1，意味着将运行一个实例
  strategy: # 更新策略
    type: Recreate  # 使用Recreate策略，当更新时会先删除旧Pod再创建新Pod
  selector: # 标签选择器，用于匹配Pod模板中的标签
    matchLabels: # 匹配具有以下标签的Pod
      app: nfs-provisioner  # 必须包含标签app=nfs-provisioner
  template: # Pod模板
    metadata: # Pod元数据
      labels: # Pod标签
        app: nfs-provisioner  # 给生成的Pod打上标签app=nfs-provisioner
    spec: # Pod规格
      serviceAccountName: nfs-client-provisioner  # 指定使用的Service Account
      containers: # 容器列表
        - name: nfs-provisioner  # 容器名字，这里是nfs-provisioner
          image: registry.cn-beijing.aliyuncs.com/mydlq/nfs-subdir-external-provisioner:v4.0.0  # 使用的镜像及其标签
          volumeMounts: # 卷挂载配置
            - name: nfs-sc  # 卷的名字
              mountPath: /persistentvolumes  # 在容器内的挂载点
          env: # 环境变量
            - name: PROVISIONER_NAME  # 制备器名称环境变量
              value: fuseim.pri/ifs  # 设置制备器名称为
            - name: NFS_SERVER  # NFS服务器地址环境变量
              value: 10.0.0.101  # 设置NFS服务器地址
            - name: NFS_PATH  # NFS路径环境变量
              value: /root/data/sc  # 设置NFS共享目录路径
      volumes: # 卷配置
        - name: nfs-sc  # 卷的名字
          nfs: # 使用NFS类型的卷
            server: 10.0.0.101  # NFS服务器地址
            path: /root/data/sc  # NFS共享目录路径
```
###### **创建 PVC**
`nfs-pvc.yaml`
```yml
apiVersion: v1  # API版本，指向v1版本的core API组
kind: PersistentVolumeClaim  # 资源类型，这里是PersistentVolumeClaim
metadata: # 元数据
  name: nfs-pvc  # PVC的名字，这里是nfs-auto-pv
spec: # 规格描述
  storageClassName: nfs-storage  # 存储类名称，指定了使用哪个StorageClass来动态制备PV。在这个例子中是managed-nfs-storage
  accessModes: # 访问模式，指定此PVC的访问方式
    - ReadWriteOnce  # 支持被单个节点以读写方式挂载。这是最常见的模式，表示卷可以被单个节点挂载进行读写操作。
  resources: # 资源需求
    requests: # 请求的具体资源量
      storage: 10Mi  # 请求的存储空间大小，这里是300MiB
```
###### **执行命令观察PV是否动态创建**
```shell
# 创建 rbac
kubectl apply -f /opt/k8s/volume/pv/nfs-rbac.yaml
# 创建 sc
kubectl apply -f /opt/k8s/volume/pv/nfs-storage.yaml
# 创建 deploy「provisioner」
kubectl apply -f /opt/k8s/volume/pv/nfs-provisioner.yaml
# 创建 pvc
kubectl apply -f /opt/k8s/volume/pv/nfs-pvc.yaml

# 查询 pv,pvc,sc,deploy,po
kubectl get pv,pvc,sc,deploy,po

# 强制删除 pv,pvc,sc,deploy,sc,po
# --grace-period=0 删除的期限为 0 秒
# --force 强制删除资源
# --all 删除指定资源类型的所有实例
kubectl delete pv,pvc,sc,deploy,sc,sts,po --all --force --grace-period=0
```
###### **多个 Pod使用一个PVC**
`nfs-pod.yml`
```yml
apiVersion: v1
kind: Pod
metadata:
  name: nfs-pod1  # Pod的名字
spec:
  containers:
    - name: container1  # 容器的名字
      image: nginx:1.17.1  # 使用的镜像及其标签
      volumeMounts:  # 卷挂载配置
        - name: shared-data  # 指定要挂载的卷的名字
          mountPath: /usr/share/nginx/html  # 在容器内的挂载点
  volumes:  # 卷定义
    - name: shared-data  # 卷的名字
      persistentVolumeClaim:  # 使用PersistentVolumeClaim来提供存储
        claimName: nfs-pvc  # 引用的PVC的名字
---
apiVersion: v1
kind: Pod
metadata:
  name: nfs-pod2  # Pod的名字
spec:
  containers:
    - name: container2  # 容器的名字
      image: nginx:1.17.1  # 使用的镜像及其标签
      volumeMounts:  # 卷挂载配置
        - name: shared-data  # 指定要挂载的卷的名字
          mountPath: /usr/share/nginx/html  # 在容器内的挂载点
  volumes:  # 卷定义
    - name: shared-data  # 卷的名字
      persistentVolumeClaim:  # 使用PersistentVolumeClaim来提供存储
        claimName: nfs-pvc  # 引用的PVC的名字
```
执行命令
```shell
kubectl create -f nfs-pod.yaml

# Pod1 写入数据到共享存储
kubectl exec -it nfs-pod1 -- /bin/bash
echo "hello from nfs-pod1" > /usr/share/nginx/html/index.html
exit

# Pod2 从共享存储读取数据
kubectl exec -it nfs-pod2 -- cat /usr/share/nginx/html/index.html
hello from nfs-pod1
```
###### **PV 目录创建的流程**
- **创建 PVC：**
    - 当你创建 PVC 时，Kubernetes 会根据 StorageClass 自动创建 PV。
- **Provisioner 创建 PV：**
    - Provisioner 会在 NFS 服务器上的共享目录下创建一个新目录，目录名称包含了 PVC 的名称和 UUID。
- **挂载到 Pod：**
    - 创建的 PV 会被挂载到 Pod 中指定的路径。
###### **PV 目录的结构**
```
/<共享目录>/<命名空间>-<PVC名称>-<PV名称>
```
- **共享目录：**
    - 这是你在 NFS 服务器上创建并共享出去的目录，比如 nfs 服务端的 /root/data/sc
- **命名空间：**
    - 这是 Kubernetes 中的一个逻辑分组，用于隔离不同的应用程序和服务。PVC 所属的命名空间名称。
- **PVC 名称：**
    - 这是在 Kubernetes 中创建的 PersistentVolumeClaim 的名称。
- **PV 名称：**
    - 这是根据 PVC 动态创建出来的 PersistentVolume 的名称，通常是一个带有 UUID 的字符串。

exempli gratia
```shell
# 查询共享目录结构
ls -l /root/data/sc
drwxrwxrwx 2 root root 24 1月  27 11:57 default-nfs-pvc-pvc-cf2d5ca8-484f-4770-aa6e-f7cecc9101ff

# 查询 pvc 名称
kubectl get pvc nfs-pvc -o custom-columns='PVC-NAMESPACE:.metadata.namespace,PVC-NAME:.metadata.name'
PVC-NAMESPACE   PVC-NAME
default         nfs-pvc
# 查询 pv 名称
kubectl get pv pvc-cf2d5ca8-484f-4770-aa6e-f7cecc9101ff -o custom-columns='PV-NAME:.metadata.name'
PV-NAME
pvc-cf2d5ca8-484f-4770-aa6e-f7cecc9101ff
```
##### 动态创建 pvc 和 pv
`nfs-sts.yaml`
```yml
apiVersion: v1 # API 版本：指定 Kubernetes API 的版本
kind: Service # 资源类型：Service（服务），用于暴露应用程序
metadata: # 元数据：资源的描述信息
  name: nginx-headless # Service 的名称：nginx-headless
  labels: # 标签：用于组织和选择 Kubernetes 对象
    app: nginx # 应用标签：标识该 Service 属于哪个应用
spec: # 规格：Service 的期望状态
  type: NodePort # 服务类型：NodePort 表示将 Service 暴露为 NodePort，外部可以访问该 Service 的 NodePort
  ports: # 端口：Service 暴露的端口
    - name: web # 端口名称：方便引用
      port: 80 # Service 端口：外部访问 Service 的端口
      protocol: TCP
  selector: # 选择器：用于匹配该 Service 对应的 Pod，流量会转发到匹配的 Pod
    app: nginx # 应用标签：匹配标签 app=nginx-pvc 的 Pod
---
apiVersion: apps/v1 # API 版本：指定 Kubernetes API 的版本，apps/v1 适用于 StatefulSet
kind: StatefulSet # 资源类型：StatefulSet（有状态集合），用于管理有状态应用
metadata: # 元数据
  name: nginx-sts # StatefulSet 的名称：nginx-sts
spec: # 规格
  serviceName: "nginx-headless" # 服务名称：与 Headless Service 的名称匹配，用于创建 Pod 的 DNS 记录。StatefulSet 会为每个 Pod 创建一个 DNS 记录：<pod-name>.<service-name>.<namespace>.svc.<cluster-domain>
  replicas: 2 # 副本数量：期望运行的 Pod 副本数，这里设置为 2
  selector: # 选择器：用于匹配该 StatefulSet 管理的 Pod，必须与 Pod 模板中的 labels 匹配。
    matchLabels: # 匹配标签
      app: nginx # 应用标签：匹配标签 app=nginx-pvc 的 Pod
  template: # Pod 模板：定义 Pod 的规格
    metadata: # Pod 元数据
      labels: # Pod 标签：必须与 StatefulSet 的 selector 匹配
        app: nginx # 应用标签
    spec: # Pod 规格
      containers: # 容器列表
        - name: nginx # 容器名称
          imagePullPolicy: IfNotPresent # 镜像拉取策略：如果本地没有镜像，则从远程仓库拉取镜像
          image: nginx:1.17.1 # 容器镜像
          volumeMounts: # 卷挂载：将持久卷挂载到容器内部
            - name: www # 卷名称：与 volumeClaimTemplates 中定义的卷名称一致
              mountPath: /usr/share/nginx/html # 挂载路径：容器内部的挂载点
  volumeClaimTemplates: # 卷声明模板：用于为每个 Pod 创建持久卷
    - metadata: # 卷元数据
        name: www # 卷名称：用于在 Pod 中挂载
      spec: # 卷规格
        storageClassName: nfs-storage # 存储类名：与 PVC 的存储类名一致
        accessModes:
          - ReadWriteMany # 访问模式：ReadWriteMany 表示多个 Pod 可以同时读写该卷
        resources: # 资源请求
          requests: # 请求的资源量
            storage: 100Mi # 存储空间：请求 100Mi 的存储空间
```
执行命令
```shell
# 动态创建 pvc,pv
kubectl apply -f nfs-sts.yaml
```
观察到 PVC 与 PV 已经动态创建并相互绑定了
```shell
kubectl get pvc -l 'app=nginx'
NAME              STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
www-nginx-sts-0   Bound    pvc-55623a82-9a27-4f93-bce3-7ce315c901fa   100Mi      RWX            nfs-storage    13m
www-nginx-sts-1   Bound    pvc-f0ff063e-08cc-4fd6-885f-8cc6f65e6b94   100Mi      RWX            nfs-storage    11m
```
每个 Pod 通过绑定 PVC 获取独立的 PV
```shell
kubectl get pod nginx-sts-0 -o custom-columns='PVC-NAME:.spec.volumes[*].persistentVolumeClaim.claimName'
PVC-NAME
www-nginx-sts-0

kubectl get pod nginx-sts-1 -o custom-columns='PVC-NAME:.spec.volumes[*].persistentVolumeClaim.claimName'
PVC-NAME
www-nginx-sts-1
```
## 8.3 配置存储
创建目录「以下操作均在该目录进行」
```shell
mkdir -p /opt/k8s/volume/config/
```
### 8.3.1 ConfigMap
ConfigMap是一种比较特殊的存储卷，它的主要作用是用来存储配置信息的。
```shell
# 查看示例
kubectl create configmap -h
Examples:
# 创建一个名为 my-config 的 ConfigMap，内容来自 path/to/bar 目录下的所有文件。
# 文件名将作为 ConfigMap 中的键，文件内容作为 ConfigMap 中的值。
kubectl create configmap my-config --from-file=path/to/bar
# 创建一个名为 my-config 的 ConfigMap，内容来自指定的文件，并使用指定的键名。
# key1 对应 /path/to/bar/file1.txt 的内容，key2 对应 /path/to/bar/file2.txt 的内容。
kubectl create configmap my-config --from-file=key1=/path/to/bar/file1.txt --from-file=key2=/path/to/bar/file2.txt
# 创建一个名为 my-config 的 ConfigMap，直接指定键值对。
# key1 的值为 config1，key2 的值为 config2。
kubectl create configmap my-config --from-literal=key1=config1 --from-literal=key2=config2
# 创建一个名为 my-config 的 ConfigMap，内容来自环境变量文件。
# 文件中的每一行 key=value 将作为 ConfigMap 中的一个键值对。
kubectl create configmap my-config --from-env-file=path/to/foo.env --from-env-file=path/to/bar.env
# 创建名为 my-config 的 ConfigMap, 使用字面值键值对 key1=value1, 模拟运行并以 YAML 格式输出
kubectl create configmap my-config --from-literal=key1=value1 --dry-run=client -o yaml
# 从 cm.yaml 文件创建 ConfigMap my-config，模拟运行，以 YAML 格式输出到 test-cm.yaml 文件
kubectl create configmap my-config --from-file=cm.yaml --dry-run=client -o yaml > test-cm.yaml
```
#### 8.3.1.1 通过yaml创建
创建 `configmap.yaml`，内容如下：
```yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: configmap
data:
  info: |
    username:admin
    password:123456
  address: |
    address:beijing
```
接下来，使用此配置文件创建configmap
```shell
# 创建configmap
kubectl create -f configmap.yaml
configmap/configmap created

# 查看configmap详情
kubectl describe cm configmap 
Name:         configmap
Namespace:    dev
Labels:       <none>
Annotations:  <none>

Data
====
address:
----
address:beijing

info:
----
username:admin
password:123456


BinaryData
====

Events:  <none>
```

通过挂载卷的方式添加配置：创建一个`pod-configmap.yaml`，将上面创建的configmap挂载进去
```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-configmap  # Pod 名称
spec:
  containers:
    - name: nginx  # 容器名称
      image: nginx:1.17.1  # 容器镜像
      volumeMounts:  # 挂载卷
        - name: config  # 卷名称
          mountPath: /configmap/config  # 挂载路径
  volumes:  # 定义卷
    - name: config  # 卷名称
      configMap:
        name: configmap  # ConfigMap 名称
        items: # 配置文件映射, 若不写则默认映射所有
          - key: address  # ConfigMap 中的键名
            path: address  # 挂载到容器内的路径
```
使用此配置文件创建pod
```shell
# 创建pod
kubectl create -f pod-configmap.yaml
pod/pod-configmap created

# 查看pod
kubectl get pod pod-configmap 
NAME            READY   STATUS    RESTARTS   AGE
pod-configmap   1/1     Running   0          6s

# 进入容器，使用cat打印info文件内容
kubectl exec -it pod-configmap /bin/sh -- ls /configmap/config
address
# 可以看到映射已经成功，每个configmap都映射成了一个目录
# key--->文件     value---->文件中的内容
# 此时如果更新configmap的内容, 容器中的值也会动态更新
```
#### 8.3.1.2 通过目录创建
在目录 `/opt/k8s/volume/config/test` 创建以下两个文件
`db.properties`
```properties
username=root
password=admin
```
`redis.properties`
```properties
host: 127.0.0.1
port: 6379
```
然后执行命令
```shell
# 以目录 `/opt/k8s/volume/config/test` 创建configmap
kubectl create cm test-dir-config --from-file=/opt/k8s/volume/config/test

# 查看configmap详情
kubectl describe cm test-dir-config 
Name:         test-dir-config
Namespace:    dev
Labels:       <none>
Annotations:  <none>

Data
====
db.properties:
----
username=root
password=admin
redis.properties:
----
host: 127.0.0.1
port: 6379


BinaryData
====

Events:  <none>
```
#### 8.3.1.2 通过文件创建
```shell
# 以文件 `/opt/k8s/volume/config/application.yaml` 创建configmap
kubectl create cm spring-boot-test-yaml --from-file=/opt/k8s/volume/config/application.yaml 

# 查看configmap详情
kubectl describe cm spring-boot-test-yaml 
Name:         spring-boot-test-yaml
Namespace:    dev
Labels:       <none>
Annotations:  <none>

Data
====
application.yaml:
----
spring:
  application:
    name: test-app
server:
  port: 8080

BinaryData
====

Events:  <none>

# 以文件 `/opt/k8s/volume/config/application.yaml` 创建configmap，并设置别名app.yml
kubectl create cm spring-boot-test-aliases-yaml --from-file=app.yml=/opt/k8s/volume/config/application.yaml 
# 查看configmap详情
kubectl describe cm spring-boot-test-aliases 
Name:         spring-boot-test-aliases-yaml
Namespace:    dev
Labels:       <none>
Annotations:  <none>

Data
====
app.yml:
----
spring:
  application:
    name: test-app
server:
  port: 8080

BinaryData
====

Events:  <none>
```
#### 8.3.1.3 通过kv创建
```shell
# 以 kv 创建configmap
kubectl create cm kv-config --from-literal=username=root --from-literal=password=123456 

# 查看configmap详情「内容无名称」
kubectl describe cm kv-config 
Name:         kv-config
Namespace:    dev
Labels:       <none>
Annotations:  <none>

Data
====
password:
----
123456
username:
----
root

BinaryData
====

Events:  <none>
```

创建一个`pod-test-env.yaml`
- 通过ENV的方式添加配置：
- 挂载目录方式
```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-test-env  # Pod 名称
spec:
  restartPolicy: Never  # 重启策略，容器退出后不重启
  containers:
    - name: env-test  # 容器名称
      image: alpine  # 容器镜像
      command: [ "/bin/sh", "-c", "env;sleep 3600" ]  # 容器启动命令，打印环境变量并休眠3600秒
      imagePullPolicy: IfNotPresent  # 镜像拉取策略，仅在本地不存在时拉取
      env: # 环境变量
        - name: username  # 环境变量名称：username
          valueFrom:
            configMapKeyRef:
              name: kv-config  # ConfigMap 名称
              key: username  # ConfigMap 中的键名
        - name: password  # 环境变量名称：APP
          valueFrom:
            configMapKeyRef:
              name: kv-config  # ConfigMap 名称
              key: password  # ConfigMap 中的键名
```
使用此配置文件创建pod
```shell
# 创建pod
kubectl create -f pod-test-env.yaml
# 查看日志「是否传递进去环境变量」
kubectl logs -f pod-test-env 
KUBERNETES_SERVICE_PORT=443
KUBERNETES_PORT=tcp://10.96.0.1:443
HOSTNAME=pod-test-env
SHLVL=1
username=root
HOME=/root
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
KUBERNETES_PORT_443_TCP_PORT=443
password=123456
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_SERVICE_HOST=10.96.0.1
PWD=/
```
### 8.3.2 Secret
在kubernetes中，还存在一种和ConfigMap非常类似的对象，称为Secret对象。它主要用于存储敏感信息，例如密码、秘钥、证书等等。
** 注意：对特殊字符使用单引号描述 **
#### 8.3.2.1 通过yaml创建
1.  首先使用base64对数据进行编码
```shell
echo -n 'admin' | base64 # 准备username
YWRtaW4=
echo -n '123456' | base64 # 准备password
MTIzNDU2
```
2.  接下来编写`secret.yaml`，并创建Secret
```yml
apiVersion: v1
kind: Secret
metadata:
  name: secret
type: Opaque
data:
  username: YWRtaW4=
  password: MTIzNDU2
```
执行命令
```shell
# 创建secret
kubectl create -f secret.yaml
secret/secret created

# 查看secret详情
kubectl describe secret secret 
Name:         secret
Namespace:    dev
Labels:       <none>
Annotations:  <none>
Type:  Opaque
Data
====
password:  6 bytes
username:  5 bytes
```
3.  创建pod-secret.yaml，将上面创建的secret挂载进去：
```yml
apiVersion: v1
kind: Pod # 资源类型为 Pod
metadata:
  name: pod-secret # Pod 的名称，这里是 pod-secret
spec:
  imagePullSecrets: # 拉取镜像的密钥
    - name: harbor-secret # 密钥名称
  containers:
    - name: nginx # 容器的名称，这里是 nginx
      image: nginx:1.17.1 # 容器使用的镜像，这里是 nginx:1.17.1
      env:
      - name: TEST_PASSWORD # 环境变量的名称，这里是 TEST_PASSWORD
        valueFrom:
          secretKeyRef:
            name: secret # 要引用的 Secret 的名称，这里是 secret
            key: password # 要引用的 Secret 中的键，这里是 password
      volumeMounts: # 挂载卷
        - name: config # 卷的名称，这里是 config
          mountPath: /secret/config # 卷在容器内的挂载路径，这里是 /secret/config
  volumes: # 定义卷
    - name: config # 卷的名称，这里是 config
      secret:
        secretName: secret # 要挂载的 Secret 的名称，这里是 secret
```
执行命令
```shell
# 创建pod
kubectl create -f pod-secret.yaml
pod/pod-secret created

# 查看pod
kubectl get pod pod-secret 
NAME            READY   STATUS    RESTARTS   AGE
pod-secret      1/1     Running   0          2m28s

# 进入容器，查看secret信息，发现已经自动解码了
kubectl exec -it pod-secret /bin/sh
# ls /secret/config/
password  username
# more /secret/config/username
admin
# more /secret/config/password
123456
```
#### 8.3.2.2 通过kv创建
```shell
# 创建secret
kubectl create secret generic kv-secret --from-literal=username=root --from-literal=password='123456'
```
### 8.3.3 SubPath
当使用 ConfigMap 或 Secret 挂载目录时，SubPath 可以精确指定要覆盖的文件，而不是整个目录。
创建 `configmap`
```shell
kubectl create cm subpath-cm --from-file /opt/k8s/volume/config/nginx.conf 
```
创建 `pod-subpath.yaml`
```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-subpath  # Pod 的名称
spec:
  containers:
  - name: nginx  # 容器的名称
    image: nginx:1.17.1  # 容器镜像
    command: [ "bin/sh", "-c", "nginx daemon off;sleep 3600" ]  # 容器启动命令，这里让 nginx 在后台运行并休眠
    volumeMounts:  # 挂载卷
    - name: nginx-conf  # 卷的名称
      mountPath: /etc/nginx/nginx.conf  # 卷在容器内的挂载路径
      subPath: etc/nginx/nginx.conf  # SubPath，指定 ConfigMap 中的具体文件「该值与 volumes 中 items.path 的值相同」
  volumes:  # 定义卷
  - name: nginx-conf
    configMap:
      name: subpath-cm  # ConfigMap 的名称
      items:
      - key: nginx.conf  # ConfigMap 中的键名
        path: etc/nginx/nginx.conf  # ConfigMap 中的键对应的值将被写入到这个路径下「必须为相对路径」
```
运行命令
```shell
# 创建pod
kubectl create -f pod-subpath.yaml
# 查看详情
kubectl exec -it pod-subpath /bin/sh -- ls -al /etc/nginx

total 36
drwxr-xr-x 3 root root  177 Jul 17  2019 .
drwxr-xr-x 1 root root   66 Jan 26 02:33 ..
drwxr-xr-x 2 root root   26 Jul 17  2019 conf.d
-rw-r--r-- 1 root root 1007 Jun 25  2019 fastcgi_params
-rw-r--r-- 1 root root 2837 Jun 25  2019 koi-utf
-rw-r--r-- 1 root root 2223 Jun 25  2019 koi-win
-rw-r--r-- 1 root root 5231 Jun 25  2019 mime.types
lrwxrwxrwx 1 root root   22 Jun 25  2019 modules -> /usr/lib/nginx/modules
-rw-r--r-- 1 root root  646 Jan 26 02:33 nginx.conf
-rw-r--r-- 1 root root  636 Jun 25  2019 scgi_params
-rw-r--r-- 1 root root  664 Jun 25  2019 uwsgi_params
-rw-r--r-- 1 root root 3610 Jun 25  2019 win-utf
```
**注意事项:**
- **SubPath 的路径:** SubPath 指定的路径是相对于 ConfigMap 或 Secret 的根目录的。
- **文件不存在:** 如果 ConfigMap 或 Secret 中不存在指定的文件，则会报错。
- **目录结构:** 如果 SubPath 指定的是一个目录，则会将 ConfigMap 或 Secret 中对应目录下的所有内容挂载到容器中。
- **权限:** 确保容器内的进程具有读取挂载文件的权限。
### 8.3.4 配置的热更新
我们通常会将项目的配置文件作为 configmap 然后挂载到 pod，那么如果更新 configmap 中的配置，会不会更新到 pod 中呢？

**ConfigMap 更新与 Pod 的关系**
**默认方式（直接挂载）：**
更新 ConfigMap 后，一般情况下会自动更新到 Pod 中。
更新周期：取决于 Kubernetes 的配置和缓存机制，通常是更新后的一段时间内生效。

**使用 subPath 的方式：**
更新 ConfigMap 后，**不会**自动更新到 Pod 中。
subPath 的作用是精确控制挂载，但会限制自动更新。

**使用变量的方式：**
如果 Pod 中的某个变量是从 ConfigMap 或 Secret 中获取的，那么更新 ConfigMap 或 Secret 后，变量的值**不会**自动更新。

**解决 subPath 更新问题的方法**
**取消 subPath：**
1. 将 ConfigMap 挂载到一个不存在的目录。
2. 利用软链接将挂载的文件链接到目标位置。
**处理目标位置已有文件的情况：**
在 Pod 的 postStart 阶段执行删除命令，删除目标位置的原有文件，再创建软链接。
#### 通过 edit 命令修改 configmap
```shell
# 修改configmap，将 address 由 beijing 改为 chengdu
kubectl edit cm configmap 
# 查询configmap
kubectl describe cm configmap 
Name:         configmap
Namespace:    dev
Labels:       <none>
Annotations:  <none>

Data
====
address:
----
address:chengdu

info:
----
username:admin
password:123456


BinaryData
====

Events:  <none>
# 查询pod里面的配置详情
kubectl exec -it pod-configmap /bin/sh -- cat /configmap/config/address
address:chengdu # 可以看到配置已经由 address:beijing 改为 address:chengdu
```
#### 通过 replace 替换
由于 configmap 我们创建通常都是基于文件创建，并不会编写 yaml 配置文件，因此修改时我们也是直接修改配置文件，而 replace 是没有 --from-file 参数的，因此无法实现基于源配置文件的替换，此时我们可以利用下方的命令实现
  
该命令的重点在于 --dry-run 参数，该参数的意思打印 yaml 文件，但不会将该文件发送给 apiserver，再结合 -oyaml 输出 yaml 文件就可以得到一个配置好但是没有发给 apiserver 的文件，然后再结合 replace 监听控制台输出得到 yaml 数据即可实现替换
```shell
kubectl create cm test-dir-config --from-file=/opt/k8s/volume/config/test/db.properties --dry-run=client -o yaml | kubectl replace -f-
```
### 8.3.5 不可变的 Secret 和 ConfigMap
对于一些敏感服务的配置文件，在线上有时是不允许修改的，此时在配置 configmap 时可以设置 immutable: true 来禁止修改
创建 `configmap-immutable.yaml`
```yml
apiVersion: v1  # Kubernetes API 版本
kind: ConfigMap  # 资源类型：配置映射（ConfigMap）
metadata:
  name: configmap-immutable  # 配置映射的名称
data:
  info: |  # 键名为 "info"，值为多行字符串
    username:admin
    password:123456
  address: |  # 键名为 "address"，值为多行字符串
    address:beijing
immutable: true  # 设置该配置映射为不可变，创建后无法修改
```
运行命令
```shell
# 创建 configmap
kubectl create -f configmap-immutable.yaml 
# 修改 configmap，当 immutable: true 时，字段是不可变的
kubectl edit cm configmap-immutable 
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
# configmaps "configmap-immutable" was not valid:
# * data: Forbidden: field is immutable when `immutable` is set
#
apiVersion: v1
data:
  address: |
    address:beijing
  info: |
    username:admin3
    password:123456
immutable: true
kind: ConfigMap
metadata:
  creationTimestamp: "2025-01-26T03:19:11Z"
  name: configmap-immutable
  resourceVersion: "277501"
  uid: 026be16b-dc93-46b6-b328-b300ce9c4238
```
# 9. 安全认证

## 9.1 访问控制概述
创建目录「以下操作均在该目录进行」
```shell
mkdir -p /opt/k8s/auth/
```
Kubernetes作为一个分布式集群的管理工具，保证集群的安全性是其一个重要的任务。所谓的安全性其实就是保证对Kubernetes的各种 **客户端** 进行 **认证和鉴权** 操作。
### 客户端 
![img](https://weichengjun2.dpdns.org//i/2025/02/17/67b2dd3497fb4.png) 
在Kubernetes集群中，客户端通常有两类：
- **Service Account(服务账户)** ：kubernetes管理的账号，用于为Pod中的服务进程在访问Kubernetes时提供身份标识。
- **User Account(普通账户)** ：一般是独立于kubernetes之外的其他服务管理的用户账号。
#### 区别
普通账户是假定被外部或独立服务管理的，由管理员分配 keys，用户像使用 Keystone 或 google 账号一样，被存储在包含 usernames 和 passwords 的 list 的文件里。  
_需要注意：在 Kubernetes 中不能通过 API 调用将普通用户添加到集群中_。
- 普通帐户是针对（人）用户的，服务账户针对 Pod 进程。
- 普通帐户是全局性。在集群所有namespaces中，名称具有惟一性。
- 通常，群集的普通帐户可以与企业数据库同步，新的普通帐户创建需要特殊权限。服务账户创建目的是更轻量化，允许集群用户为特定任务创建服务账户。
- 普通帐户和服务账户的审核注意事项不同。
- 对于复杂系统的配置包，可以包括对该系统的各种组件的服务账户的定义。
#### Service Account 自动化
##### Service Account Admission Controller
通过 [Admission Controller](http://docs.kubernetes.org.cn/144.html) 插件来实现对 pod 修改，它是 apiserver 的一部分。创建或更新 pod 时会同步进行修改 pod。当插件处于激活状态（在大多数发行版中都默认情况）创建或修改 pod 时，会按以下操作执行：
1. 如果 pod 没有设置 ServiceAccount，则将 ServiceAccount 设置为 default。
2. 确保 pod 引用的 ServiceAccount 存在，否则将会拒绝请求。
3. 如果 pod 不包含任何 ImagePullSecrets，则将ServiceAccount 的 ImagePullSecrets 会添加到 pod 中。
4. 为包含 API 访问的 Token 的 pod 添加了一个 volume。
5. 把 volumeSource 添加到安装在 pod 的每个容器中，挂载在 /var/run/secrets/kubernetes.io/serviceaccount。
##### Token Controller
TokenController 作为 controller-manager 的一部分运行。异步行为:
- 观察 serviceAccount 的创建，并创建一个相应的 Secret 来允许 API 访问。
- 观察 serviceAccount 的删除，并删除所有相应的ServiceAccountToken Secret
- 观察 secret 添加，并确保关联的 ServiceAccount 存在，并在需要时向 secret 中添加一个 Token。
- 观察 secret 删除，并在需要时对应 ServiceAccount 的关联
##### Service Account Controller
Service Account Controller 在 namespaces 里管理ServiceAccount，并确保每个有效的 namespaces 中都存在一个名为 “default” 的 ServiceAccount。
执行命令
```shell
# 查看 service account (sa), default是默认创建的
kubectl get serviceaccount
NAME                     SECRETS   AGE
default                  1         27d
nfs-client-provisioner   1         21d

# 查看 service account (kube-system 系统命名空间下)
kubectl get serviceaccount -n kube-system
NAME                                 SECRETS   AGE
attachdetach-controller              1         27d
bootstrap-signer                     1         27d
calico-kube-controllers              1         27d
calico-node                          1         27d
certificate-controller               1         27d
clusterrole-aggregation-controller   1         27d
coredns                              1         27d
cronjob-controller                   1         27d
daemon-set-controller                1         27d
default                              1         27d
deployment-controller                1         27d
disruption-controller                1         27d
endpoint-controller                  1         27d
endpointslice-controller             1         27d
endpointslicemirroring-controller    1         27d
ephemeral-volume-controller          1         27d
expand-controller                    1         27d
generic-garbage-collector            1         27d
horizontal-pod-autoscaler            1         27d
job-controller                       1         27d
kube-proxy                           1         27d
namespace-controller                 1         27d
nfs-client-provisioner               1         21d
nfs-subdir-external-provisioner      1         21d
node-controller                      1         27d
persistent-volume-binder             1         27d
pod-garbage-collector                1         27d
pv-protection-controller             1         27d
pvc-protection-controller            1         27d
replicaset-controller                1         27d
replication-controller               1         27d
resourcequota-controller             1         27d
root-ca-cert-publisher               1         27d
service-account-controller           1         27d
service-controller                   1         27d
statefulset-controller               1         27d
token-cleaner                        1         27d
ttl-after-finished-controller        1         27d
ttl-controller                       1         27d
```
### 认证、授权与准入控制 
ApiServer是访问及管理资源对象的唯一入口。任何一个请求访问ApiServer，都要经过下面三个流程：
- Authentication（认证）：身份鉴别，只有正确的账号才能够通过认证
- Authorization（授权）： 判断用户是否有权限对访问的资源执行特定的动作
- Admission Control（准入控制）：用于补充授权机制以实现更加精细的访问控制功能。
![img](https://weichengjun2.dpdns.org//i/2025/02/15/67b0098b673f8.png) 
## 9.2 认证管理
Kubernetes集群安全的最关键点在于如何识别并认证客户端身份，它提供了3种客户端身份认证方式：
- HTTP Base认证：通过用户名+密码的方式认证
    这种认证方式是把“用户名:密码”用BASE64算法进行编码后的字符串放在HTTP请求中的Header Authorization域里发送给服务端。服务端收到后进行解码，获取用户名及密码，然后进行用户身份认证的过程。
- HTTP Token认证：通过一个Token来识别合法用户
    这种认证方式是用一个很长的难以被模仿的字符串--Token来表明客户身份的一种方式。每个Token对应一个用户名，当客户端发起API调用请求时，需要在HTTP Header里放入Token，API Server接到Token后会跟服务器中保存的token进行比对，然后进行用户身份认证的过程。
- HTTPS证书认证：基于CA根证书签名的双向数字证书认证方式
    这种认证方式是安全性最高的一种方式，但是同时也是操作起来最麻烦的一种方式。
![img](https://weichengjun2.dpdns.org//i/2025/02/15/67b0098bab8c6.png) 
 **HTTPS认证大体分为3个过程：** 
1.  证书申请和下发
	  HTTPS通信双方的服务器向CA机构申请证书，CA机构下发根证书、服务端证书及私钥给申请者
2.  客户端和服务端的双向认证
	  1. 客户端向服务器端发起请求，服务端下发自己的证书给客户端，
	     客户端接收到证书后，通过私钥解密证书，在证书中获得服务端的公钥，
	     客户端利用服务器端的公钥认证证书中的信息，如果一致，则认可这个服务器
	  2. 客户端发送自己的证书给服务器端，服务端接收到证书后，通过私钥解密证书，
	     在证书中获得客户端的公钥，并用该公钥认证证书信息，确认客户端是否合法
3.  服务器端和客户端进行通信
	  服务器端和客户端协商好加密方案后，客户端会产生一个随机的秘钥并加密，然后发送到服务器端。
	  服务器端接收这个秘钥后，双方接下来通信的所有内容都通过该随机秘钥加密
> 注意: Kubernetes允许同时配置多种认证方式，只要其中任意一个方式认证通过即可
## 9.3 授权管理
授权发生在认证成功之后，通过认证就可以知道请求用户是谁， 然后Kubernetes会根据事先定义的授权策略来决定用户是否有权限访问，这个过程就称为授权。
每个发送到ApiServer的请求都带上了用户和资源的信息：比如发送请求的用户、请求的路径、请求的动作等，授权就是根据这些信息和授权策略进行比较，如果符合策略，则认为授权通过，否则会返回错误。

API Server目前支持以下几种授权策略：
- AlwaysDeny：表示拒绝所有请求，一般用于测试
- AlwaysAllow：允许接收所有请求，相当于集群不需要授权流程（Kubernetes默认的策略）
- ABAC：基于属性的访问控制，表示使用用户配置的授权规则对用户请求进行匹配和控制
- Webhook：通过调用外部REST服务对用户进行授权
- Node：是一种专用模式，用于对kubelet发出的请求进行访问控制
- RBAC：基于角色的访问控制（kubeadm安装方式下的默认选项）

RBAC(Role-Based Access Control) 基于角色的访问控制，主要是在描述一件事情： **给哪些对象授予了哪些权限** 
其中涉及到了下面几个概念：
- 对象：User、Groups、ServiceAccount
- 角色：代表着一组定义在资源上的可操作动作(权限)的集合
- 绑定：将定义好的角色跟用户绑定在一起
![img](https://weichengjun2.dpdns.org//i/2025/02/15/67b0098be7de3.png) 
RBAC引入了4个顶级资源对象：
- Role、ClusterRole：角色，用于指定一组权限
- RoleBinding、ClusterRoleBinding：角色绑定，用于将角色（权限）赋予给对象

### Role、ClusterRole 
一个角色就是一组权限的集合，这里的权限都是许可形式的（白名单）。
```yml
# Role只能对命名空间内的资源进行授权，需要指定nameapce，会包含一组权限，没有拒绝规则，只是附加允许
kind: Role
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: authorization-role
rules:
- apiGroups: [""]  # 支持的API组列表,"" 空字符串，表示核心API群
  resources: ["pods"] # 支持的资源对象列表
  verbs: ["get", "watch", "list"] # 允许的对资源对象的操作方法列表
---
# ClusterRole可以对集群范围内资源、跨namespaces的范围资源、非资源类型进行授权
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
 name: authorization-clusterrole
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```
需要详细说明的是，rules中的参数：
- apiGroups: 支持的API组列表
```shell
"","apps", "autoscaling", "batch"
```
- resources：支持的资源对象列表
```shell
"services", "endpoints", "pods","secrets","configmaps","crontabs","deployments","jobs",
"nodes","rolebindings","clusterroles","daemonsets","replicasets","statefulsets",
"horizontalpodautoscalers","replicationcontrollers","cronjobs"
```
- verbs：对资源对象的操作方法列表
```shell
"get", "list", "watch", "create", "update", "patch", "delete", "exec"
```
常用命令
```shell
# 查看已有的角色信息
kubectl get role -n ingress-nginx -oyaml
# 查看某个集群角色的信息
kubectl get clusterrole view -oyaml
```
### RoleBinding、ClusterRoleBinding 
角色绑定用来把一个角色绑定到一个目标对象上，绑定目标可以是User、Group或者ServiceAccount。
```yml
# RoleBinding可以将同一namespace中的subject绑定到某个Role下，则此subject即具有该Role定义的权限
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: authorization-role-binding
subjects:
- kind: User
  name: riven
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: authorization-role
  apiGroup: rbac.authorization.k8s.io
---
# ClusterRoleBinding在整个集群级别和所有namespaces将特定的subject与ClusterRole绑定，授予权限
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
 name: authorization-clusterrole-binding
subjects:
- kind: User
  name: riven
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: authorization-clusterrole
  apiGroup: rbac.authorization.k8s.io
```
常用命令
```shell
# 查看 rolebinding 信息
kubectl get rolebinding --all-namespaces
  
# 查看指定 rolebinding 的配置信息
kubectl get rolebinding <role_binding_name> --all-namespaces -oyaml
```
### RoleBinding引用ClusterRole进行授权 
RoleBinding可以引用ClusterRole，对属于同一命名空间内ClusterRole定义的资源主体进行授权。
>一种很常用的做法就是，集群管理员为集群范围预定义好一组角色（ClusterRole），然后在多个命名空间中重复使用这些ClusterRole。
>这样可以大幅提高授权管理工作效率，也使得各个命名空间下的基础性授权规则与使用体验保持一致。
```yml
# 虽然authorization-clusterrole是一个集群角色，但是因为使用了RoleBinding
# 所以riven只能读取default命名空间中的资源
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: authorization-role-binding-ns
subjects:
- kind: User
  name: riven
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: authorization-clusterrole
  apiGroup: rbac.authorization.k8s.io
```
### 实战：创建一个只能管理default空间下Pods资源的账号 
1.  创建账号
```shell
# 1) 创建证书
cd /etc/kubernetes/pki/
(umask 077;openssl genrsa -out devman.key 2048)

# 2) 用apiserver的证书去签署
# 2-1) 签名申请，申请的用户是devman,组是devgroup
openssl req -new -key devman.key -out devman.csr -subj "/CN=devman/O=devgroup"
# 2-2) 签署证书
openssl x509 -req -in devman.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out devman.crt -days 3650

# 3) 设置集群、用户、上下文信息
kubectl config set-cluster kubernetes --embed-certs=true --certificate-authority=/etc/kubernetes/pki/ca.crt --server=https://10.0.0.101:6443
kubectl config set-credentials devman --embed-certs=true --client-certificate=/etc/kubernetes/pki/devman.crt --client-key=/etc/kubernetes/pki/devman.key
kubectl config set-context devman@kubernetes --cluster=kubernetes --user=devman

# 切换账户到devman
kubectl config use-context devman@kubernetes

# 查看dev下pod，发现没有权限
kubectl get pods
Error from server (Forbidden): pods is forbidden: User "devman" cannot list resource "pods" in API group "" in the namespace "default"

# 切换到admin账户
kubectl config use-context kubernetes-admin@kubernetes
```
2. 创建文件`role-rolebinding.yaml`
```yml
# 创建 Role（角色）和 RoleBinding（角色绑定），为 devman 用户授权
kind: Role # 定义资源类型为 Role（角色）
apiVersion: rbac.authorization.k8s.io/v1 # 指定 API 版本
metadata:
  name: dev-role # Role 的名称，这里命名为 dev-role
rules: # 定义权限规则
  - apiGroups: [ "" ] # 指定 API 组，"" 表示核心 API 组（Kubernetes 内置资源）
    resources: [ "pods" ] # 指定授权的资源类型为 pods（Pod 资源）
    verbs: [ "get", "watch", "list", "create"] # 指定允许的操作，包括 get（获取）、watch（监视）、list（列出）和 create（创建）
--- # YAML 文档分隔符，用于分隔多个资源定义
kind: RoleBinding # 定义资源类型为 RoleBinding（角色绑定）
apiVersion: rbac.authorization.k8s.io/v1 # 指定 API 版本
metadata:
  name: authorization-role-binding # RoleBinding 的名称，这里命名为 authorization-role-binding
subjects: # 指定被授权的对象
  - kind: User # 指定授权对象类型为用户
    name: devman # 指定被授权的用户名，这里是 devman
    apiGroup: rbac.authorization.k8s.io # 指定 API 组
roleRef: # 引用要绑定的 Role 资源
  kind: Role # 指定引用的资源类型为 Role
  name: dev-role # 指定引用的 Role 名称，这里是 dev-role
  apiGroup: rbac.authorization.k8s.io # 指定 API 组
```
执行命令
```shell
kubectl create -f role-rolebinding.yaml
role.rbac.authorization.k8s.io/dev-role created
rolebinding.rbac.authorization.k8s.io/authorization-role-binding created
```
3.  切换账户，再次验证
```shell
# 切换账户到devman
kubectl config use-context devman@kubernetes

# 再次查看
kubectl get pods
NAME                                 READY   STATUS             RESTARTS   AGE
nginx-deployment-66cb59b984-8wp2k    1/1     Running            0          4d1h
nginx-deployment-66cb59b984-dc46j    1/1     Running            0          4d1h
nginx-deployment-66cb59b984-thfck    1/1     Running            0          4d1h

# 为了不影响后面的学习,切回admin账户
kubectl config use-context kubernetes-admin@kubernetes
```
## 9.4 准入控制
通过了前面的认证和授权之后，还需要经过准入控制处理通过之后，apiserver才会处理这个请求。
准入控制是一个可配置的控制器列表，可以通过在Api-Server上通过命令行设置选择执行哪些准入控制器：
```shell
--admission-control=NamespaceLifecycle,LimitRanger,ServiceAccount,PersistentVolumeLabel,DefaultStorageClass,ResourceQuota,DefaultTolerationSeconds
```
只有当所有的准入控制器都检查通过之后，apiserver才执行该请求，否则返回拒绝。
当前可配置的Admission Control准入控制如下：
- AlwaysAdmit：允许所有请求
- AlwaysDeny：禁止所有请求，一般用于测试
- AlwaysPullImages：在启动容器之前总去下载镜像
- DenyExecOnPrivileged：它会拦截所有想在Privileged Container上执行命令的请求
- ImagePolicyWebhook：这个插件将允许后端的一个Webhook程序来完成admission controller的功能。
- Service Account：实现ServiceAccount实现了自动化
- SecurityContextDeny：这个插件将使用SecurityContext的Pod中的定义全部失效
- ResourceQuota：用于资源配额管理目的，观察所有请求，确保在namespace上的配额不会超标
- LimitRanger：用于资源限制管理，作用于namespace上，确保对Pod进行资源限制
- InitialResources：为未设置资源请求与限制的Pod，根据其镜像的历史资源的使用情况进行设置
- NamespaceLifecycle：如果尝试在一个不存在的namespace中创建资源对象，则该创建请求将被拒绝。当删除一个namespace时，系统将会删除该namespace中所有对象。
- DefaultStorageClass：为了实现共享存储的动态供应，为未指定StorageClass或PV的PVC尝试匹配默认的StorageClass，尽可能减少用户在申请PVC时所需了解的后端存储细节
- DefaultTolerationSeconds：这个插件为那些没有设置forgiveness tolerations并具有notready:NoExecute和unreachable:NoExecute两种taints的Pod设置默认的“容忍”时间，为5min
- PodSecurityPolicy：这个插件用于在创建或修改Pod时决定是否根据Pod的security context和可用的PodSecurityPolicy对Pod的安全策略进行控制

# 10.Kubernetes 可视化界面
## 10.1 Kubernetes Dashboard
之前在kubernetes中完成的所有操作都是通过命令行工具kubectl完成的。其实，为了提供更丰富的用户体验，kubernetes还开发了一个基于web的用户界面（Dashboard）。用户可以使用Dashboard部署容器化的应用，还可以监控应用的状态，执行故障排查以及管理kubernetes中各种资源。
创建目录「以下操作均在该目录进行」
```shell
mkdir -p /opt/k8s/dashboard/
```
### 下载yaml，并运行Dashboard
```yml
# 下载官方部署配置文件
wget https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml

# 修改属性
kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  type: NodePort # 新增
  ports:
    - port: 443
      targetPort: 8443
      nodePort: 30009 # 新增
  selector:
    k8s-app: kubernetes-dashboard

# 创建资源
kubectl apply -f recommended.yaml
# 查看资源是否已经就绪
kubectl get all -n kubernetes-dashboard -o wide
# 查看 pod
kubectl get po -n kubernetes-dashboard -o wide
NAME                                        READY   STATUS    RESTARTS        AGE   IP               NODE         NOMINATED NODE   READINESS GATES
dashboard-metrics-scraper-79459f84f-5mfvx   1/1     Running   3 (3h12m ago)   24h   10.244.235.219   k8s-master   <none>           <none>
kubernetes-dashboard-76dc96b85f-lh78d       1/1     Running   5 (3h10m ago)   24h   10.244.169.163   k8s-node2    <none>           <none>
# 查看 svc
kubectl get svc -n kubernetes-dashboard
NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)         AGE
dashboard-metrics-scraper   ClusterIP   10.111.26.22    <none>        8000/TCP        24h
kubernetes-dashboard        NodePort    10.96.132.131   <none>        443:30009/TCP   24h
```
### 配置所有权限账号
创建`dashboard-admin.yaml`
```yml
apiVersion: v1 # API 版本：v1
kind: ServiceAccount # 资源类型：ServiceAccount（服务账号）
metadata:
  labels:
    k8s-app: kubernetes-dashboard # 标签：k8s-app=kubernetes-dashboard
  name: dashboard-admin # ServiceAccount 名称：dashboard-admin
  namespace: kubernetes-dashboard # 命名空间：kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1 # API 版本：rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding # 资源类型：ClusterRoleBinding（集群角色绑定）
metadata:
  name: dashboard-admin-cluster-role # ClusterRoleBinding 名称：dashboard-admin-cluster-role
roleRef:
  apiGroup: rbac.authorization.k8s.io # 引用的 API 组：rbac.authorization.k8s.io
  kind: ClusterRole # 引用的资源类型：ClusterRole（集群角色）
  name: cluster-admin # 引用的 ClusterRole 名称：cluster-admin（集群管理员角色）
subjects:
  - kind: ServiceAccount # 授权对象类型：ServiceAccount（服务账号）
    name: dashboard-admin # ServiceAccount 名称：dashboard-admin
    namespace: kubernetes-dashboard # ServiceAccount 所在的命名空间：kubernetes-dashboard
```
执行命令
```shell
# 创建资源
kubectl apply -f dashboard-admin.yaml
# 查看账号信息
kubectl describe serviceaccount dashboard-admin -n kubernetes-dashboard
# 获取账号的 token 登录 dashboard
kubectl describe secrets dashboard-admin-token-5crbd -n kubernetes-dashboard
```
### 通过浏览器访问Dashboard的UI
在登录页面上输入上面的token
![image-20200520144548997](https://weichengjun2.dpdns.org//i/2025/02/15/67b0098c39bda.png) 

出现下面的页面代表成功
![image-20200520144959353](https://weichengjun2.dpdns.org//i/2025/02/15/67b0098cb505a.png) 
### 使用DashBoard
本章节以Deployment为例演示DashBoard的使用
#### 查看 
选择指定的命名空间 `dev` ，然后点击 `Deployments` ，查看dev空间下的所有deployment
![img](https://weichengjun2.dpdns.org//i/2025/02/15/67b0098d2a97f.png) 
#### 扩缩容 
在 `Deployment` 上点击 `规模` ，然后指定 `目标副本数量` ，点击确定
![img](https://weichengjun2.dpdns.org//i/2025/02/15/67b0098db056a.png) 
#### 编辑 
在 `Deployment` 上点击 `编辑` ，然后修改 `yaml文件` ，点击确定
![image-20200520163253644](https://weichengjun2.dpdns.org//i/2025/02/15/67b0098e06438.png) 
#### 查看Pod 
点击 `Pods` , 查看pods列表
![img](https://weichengjun2.dpdns.org//i/2025/02/15/67b0098e80beb.png) 
#### 操作Pod 
选中某个Pod，可以对其执行日志（logs）、进入执行（exec）、编辑、删除操作
> Dashboard提供了kubectl的绝大部分功能，这里不再一一演示

## 10.2 kubesphere
[在 Kubernetes 上最小化安装 KubeSphere](https://kubesphere.io/zh/docs/v3.3/quick-start/minimal-kubesphere-on-k8s/)
创建目录「以下操作均在该目录进行」
```shell
mkdir -p /opt/k8s/kubesphere/
```
### 本地存储动态 PVC
常见第三方开源存储系统
- [Ceph CSI](https://github.com/ceph/ceph-csi)
- [GlusterFS](https://docs.gluster.org/en/latest/)
- [OpenEBS](https://openebs.io/)
- [Longhorn](https://longhorn.io/)
总结
- 如果需要大规模、统一的存储解决方案，Ceph CSI是最佳选择。
- 如果需要简单的分布式文件系统，GlusterFS是一个不错的选择。
- 如果需要在Kubernetes环境中提供灵活的存储方案，OpenEBS是一个很好的选择。
- 如果需要在Kubernetes环境中，轻量级的块存储，Longhorn是一个很好的选择。
```shell
# 在所有节点安装 iSCSI 协议客户端（OpenEBS 需要该协议提供存储支持）
yum install iscsi-initiator-utils -y
# 设置开机启动
systemctl enable --now iscsid
# 启动服务
systemctl start iscsid
# 查看服务状态
systemctl status iscsid
# 安装 OpenEBS
kubectl apply -f https://openebs.github.io/charts/openebs-operator.yaml
# 查看状态（下载镜像可能需要一些时间）
watch -n 1 kubectl get all -n openebs
```
创建`default-storage-class.yaml`
```yml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: default
  annotations:
    cas.openebs.io/config: |
      - name: StorageType
        value: "hostpath"
      - name: BasePath
        value: "/var/openebs/local/"
    kubectl.kubernetes.io/last-applied-configuration: >
      {"apiVersion":"storage.k8s.io/v1","kind":"StorageClass","metadata":{"annotations":{"cas.openebs.io/config":"-
      name: StorageType\n  value: \"hostpath\"\n- name: BasePath\n  value:
      \"/var/openebs/local/\"\n","openebs.io/cas-type":"local","storageclass.beta.kubernetes.io/is-default-class":"true","storageclass.kubesphere.io/supported-access-modes":"[\"ReadWriteOnce\"]"},"name":"local"},"provisioner":"openebs.io/local","reclaimPolicy":"Delete","volumeBindingMode":"WaitForFirstConsumer"}
    openebs.io/cas-type: local
    storageclass.beta.kubernetes.io/is-default-class: 'true'
    storageclass.kubesphere.io/supported-access-modes: '["ReadWriteOnce"]'
provisioner: openebs.io/local
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
```
执行命令
```shell
# 在主节点创建本地 storage class
kubectl apply -f /opt/k8s/kubesphere/default-storage-class.yaml
```
### 安装
```shell
# 切换目录
cd /opt/k8s/kubesphere
# 下载
wget https://github.com/kubesphere/ks-installer/releases/download/v3.3.1/kubesphere-installer.yaml
wget https://github.com/kubesphere/ks-installer/releases/download/v3.3.1/cluster-configuration.yaml

# 安装
kubectl apply -f kubesphere-installer.yaml
kubectl apply -f cluster-configuration.yaml
# 查看 pod 是否启动
kubectl get po,svc -n kubesphere-system -o wide
# pod 是否启动完成后，检查安装日志
kubectl logs -n kubesphere-system $(kubectl get pod -n kubesphere-system -l 'app in (ks-install, ks-installer)' -o jsonpath='{.items[0].metadata.name}') -f
# 查看端口
kubectl get svc/ks-console -n kubesphere-system
# 卸载
sh kubesphere-delete.sh
```
### 访问
[KubeSphere](http://10.0.0.101:30880/dashboard)
默认端口是 30880，如果是云服务商，或开启了防火墙，记得要开放该端口
登录控制台访问，账号密码：`admin/P@88w0rd`

### 启用可插拔组件
[KubeSphere DevOps 系统](https://kubesphere.io/zh/docs/v3.3/pluggable-components/devops/)
[启用可插拔组件](https://kubesphere.io/zh/docs/v3.3/pluggable-components/)
## 10.3 Rancher
[多云混合云集群 | 容器云平台PaaS | 企业级Kubernetes 解决方案 | Rancher](https://www.rancher.cn/)
## 10.4 Kuboard
[Kuboard\_Kubernetes教程\_K8S安装\_管理界面](https://www.kuboard.cn/)