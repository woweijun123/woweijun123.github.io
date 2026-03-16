---
{"dg-publish":true,"permalink":"/Work/Tools/Docker k8s/K8S DevOps/","title":"K8S DevOps","tags":["flashcards"],"noteIcon":"","created":"2025-04-27T11:27:47.000+08:00","updated":"2026-03-16T23:00:02.068+08:00"}
---

# 相关资料
链接: https://pan.baidu.com/s/1CSQuqScsmXJkWz5IXKmGkw?pwd=ng42 提取码: ng42
# 概览图
![](https://weichengjun2.dpdns.org//i/2025/02/18/67b40052df9ae.png)
# 单体服务
## 基础环境
创建目录「以下操作均在该目录进行」
```shell
mkdir -p /opt/k8s/devops
```
### 命名空间初始化
```shell
# 创建命令空间
kubectl apply -f /opt/k8s/devops/kube-devops-namespace.yaml
```
## Gitlab
创建目录「以下操作均在该目录进行」
```shell
mkdir -p /opt/k8s/devops/gitlab
```
### 安装 Gitlab
```shell
# 下载安装包
wget https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7/gitlab-ce-15.9.1-ce.0.el7.x86_64.rpm
# 安装
rpm -i gitlab-ce-15.9.1-ce.0.el7.x86_64.rpm
# 编辑 /etc/gitlab/gitlab.rb 文件
# 修改 external_url 访问路径 http://<ip>:<port>
# 其他配置修改如下
gitlab_rails['time_zone'] = 'Asia/Shanghai'
puma['worker_processes'] = 2
sidekiq['concurrency'] = 8
postgresql['shared_buffers'] = "128MB"
postgresql['max_worker_processes'] = 4
prometheus_monitoring['enable'] = false

# 更新配置并重启
gitlab-ctl reconfigure
gitlab-ctl restart
```
### 页面配置
```shell
# 查看默认密码
cat /etc/gitlab/initial_root_password
登录后修改默认密码 > 右上角头像 > Perferences > Password
# 修改系统配置：点击左上角三横 > Admin
Settings > General > Account and limit > 取消 Gravatar enabled > Save changes
# 关闭用户注册功能
Settings > General > Sign-up restrictions > 取消 Sign-up enabled > Save changes
# 开启 webhook 外部访问
Settings > Network > Outbound requests > Allow requests to the local network from web hooks and services 勾选
# 设置语言为中文（全局）
Settings > Preferences > Localization > Default language > 选择简体中文 > Save changes
# 设置当前用户语言为中文
右上角用户头像 > Preferences > Localization > Language > 选择简体中文 > Save changes
```
### 配置 Secret
```shell
# 创建 gitlab 默认用户名密码 secret
echo root > ./username
echo asdfjkl; > password
kubectl create secret generic git-user-pass --from-file=./username --from-file=./password -n kube-devops 
```
### 卸载
```shell
# 停止服务
gitlab-ctl stop
# 卸载 rpm 软件（注意安装的软件版本是 ce 还是 ee）
rpm -e gitlab-ce
# 查看进程
ps -ef|grep gitlab 
# 干掉第一个 runsvdir -P /opt/gitlab/service log 进程

# 删除 gitlab 残余文件
find / -name *gitlab* | xargs rm -rf
find / -name gitlab | xargs rm -rf
```
## Harbor
创建目录「以下操作均在该目录进行」
```shell
mkdir -p /opt/harbor
```
### 安装 Harbor
[GitHub - goharbor/harbor: An open source trusted cloud native registry project that stores, signs, and scans content.](https://github.com/goharbor/harbor)
[Docker 企业级镜像仓库 Harbor 的搭建与维护 - 自由早晚乱余生 - 博客园](https://www.cnblogs.com/operationhome/p/10868498.html#%E4%B8%80%E4%BB%80%E4%B9%88%E6%98%AF-harbor)
#### 要求
- Docker 引擎版本 20.10.10-ce+ 或更高版本
- docker-compose （v1.18.0+） 或 docker compose v2 （docker-compose-plugin）
执行步骤
```shell
# 创建主目录、数据目录、日志目录
mkdir -p /opt/harbor/{main,data,logs}
# 切换到主目录
cd /opt/harbor/main
# 下载&解压
wget https://github.com/goharbor/harbor/releases/download/v2.12.2/harbor-offline-installer-v2.12.2.tgz
tar -xvf harbor-offline-installer-v2.12.2.tgz && rm -rf harbor-offline-installer-v2.12.2.tgz
# #### 复制 harbor.yml
cp harbor.yml.tmpl harbor.yml
# 修改harbor.yml，内容如下
hostname : 10.0.0.6 # 设置成你的外网ip/局域网ip
port : 8888# 设置访问端口
https：# 注释掉https的配置内容
harbor_admin_password：'asdfjkl;' # 默认不用更改，账号：damin  密码：Harbor12345
data_volume：/root/dnmp/data/harbor/data # 配置data目录（设置第3步创建的目录 /mnt/docker/harbor/data）
# 执行
bash prepare && bash install.sh
# 启动
docker-compose up -d
```
### 配置
#### 创建Secret
```shell
# 创建 harbor 访问账号密码「jenkins」
kubectl create secret docker-registry harbor-secret --docker-server=10.0.0.101:8888 --docker-username=admin --docker-password='asdfjkl;' -n kube-devops
# 创建 harbor 访问账号密码「k8s-cicd-dev环境」
kubectl create secret docker-registry harbor-secret --docker-server=10.0.0.101:8888 --docker-username=admin --docker-password='asdfjkl;' -n k8s-cicd-dev
# 创建 harbor 访问账号密码「k8s-cicd-master环境」
kubectl create secret docker-registry harbor-secret --docker-server=10.0.0.101:8888 --docker-username=admin --docker-password='asdfjkl;' -n k8s-cicd-master
```
#### 信任的非安全镜像仓库
```shell
# 每个node都编辑配置文件，添加如下内容
vim /etc/docker/daemon.json
{
  "registry-mirrors": ["https://j8f5idhu.mirror.aliyuncs.com"],
  "exec-opts": ["native.cgroupdriver=systemd"] # 指定了 Docker 使用的 cgroup 驱动程序
  "insecure-registries": ["10.0.0.101:8888"] # 添加信任的非安全镜像仓库
}
```
### 使用
```shell
# 登录到私有仓库
docker login 10.0.0.101:8888
# 标记镜像
docker tag nginx:1.17.1 10.0.0.101:8888/k8s/nginx:1.17.1
# 推送镜像
docker push 10.0.0.101:8888/k8s/nginx:1.17.1
# 拉取镜像
docker pull 10.0.0.101:8888/k8s/nginx:1.17.1
```
## SonarQube
创建目录「以下操作均在该目录进行」
```shell
mkdir -p /opt/k8s/devops/sonarqube
```
### 安装 SonarQube
执行命令
```shell
# 切换目录
cd /opt/k8s/devops/sonarqube
# 应用
kubectl apply -f .
# 卸载
kubectl delete -f .
# 查询po,svc
kubectl get po,svc -o wide -n kube-devops
NAME                                   READY   STATUS    RESTARTS        AGE     IP               NODE        NOMINATED NODE   READINESS GATES
pod/postgres-sonar-f644b4fbd-xf6s5     1/1     Running   0               15m     10.244.169.139   k8s-node2   <none>           <none>
pod/sonarqube-79fff8f874-f6crv         1/1     Running   0               7m55s   10.244.169.140   k8s-node2   <none>           <none>

NAME                      TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE    SELECTOR
service/postgres-sonar    NodePort   10.99.233.173   <none>        5432:32693/TCP   15m    app=postgres-sonar
service/sonarqube         NodePort   10.106.40.38    <none>        9000:30004/TCP   15m    app=sonarqube
```
### 登录网址
[Site Unreachable](http://10.0.0.101:30004/dashboard)
默认账号密码：admin/admin
### 生成服务 token
登录到 sonarqube 后台，点击头像 > MyAccount > Security > Generate Tokens > generate 生成 token 并复制
```shell
# 以 jenkins 生成 token
jenkins
sqa_ad5df8b4b8c579fdf55ce31cc72f4217eb716057
```
### 创建 Webhook 服务
点击顶部菜单栏的配置 > 配置（小三角） > 网络调用
```shell
Name：jenkins
# 配置jenkins的连接地址
URL：http://10.0.0.101:30001/sonarqube-webhook/
密码：无需密码
```
## Jenkins
[Jenkins的安装教程 - 禅道程序猿 - 博客园](https://www.cnblogs.com/zentao/p/18345233)
[Jenkins  中文网](https://www.jenkins.io/zh/)
创建目录「以下操作均在该目录进行」
```shell
mkdir -p /opt/k8s/devops/jenkins
```
### 构建带 maven 环境的 jenkins 镜像安装 Jenkins
```shell
# 进入目录
cd /opt/k8s/devops/jenkins
# 构建带 maven 环境的 jenkins 镜像
docker build -t 10.0.0.101:8888/library/jenkins-maven:2.462.3-lts-jdk11 jenkins-maven/
# 登录 harbor
docker login -uadmin 10.0.0.101:8888
# 推送镜像到 harbor
docker push 10.0.0.101:8888/library/jenkins-maven:2.462.3-lts-jdk11
# 安装
kubectl create -f manifests
# 卸载
kubectl delete -f manifests
# 查看po,svc
kubectl get po,svc -n kube-devops
# 查找PV
kubectl get pv,pvc -n kube-devops -o wide | grep jenkins |awk '{print $1}'
# 删除PV
kubectl delete pv pvc-0fd237dc-98a6-4f37-8cea-3fda8d21699e -n kube-devops
# 查看容器日志，获取默认密码
kubectl logs -f pod名称 -n kube-devops
# 默认密码文件位置「find / -name initialAdminPassword 2>/dev/null」
cat /var/jenkins_home/secrets/initialAdminPassword
# 修改为清华源镜像地址「修改文件 /var/jenkins_home/updates/default.json」
{
  "updateCenter": {
    "url": "https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json"  
  }  
}
# 容器启动后替换镜像
sed -i 's#updates.jenkins-ci.org/download#mirrors.tuna.tsinghua.edu.cn/jenkins#g' /var/jenkins_home/updates/default.json
sed -i 's#www.google.com#www.baidu.com#g' /var/jenkins_home/updates/default.json
```
### 安装插件
#### Build Authorization Token Root
构建授权 token
#### Gitlab
gitlab 配置插件
#### SonarQube Scanner
代码质量审查工具配置步骤
1. Dashboard > 系统管理 > 系统配置 > Add SonarQube
```shell
Name：sonarqube # 注意这个名字要在 Jenkinsfile 中用到
Server URL：http://sonarqube:9000
Server authentication token：创建 credentials 配置为从 sonarqube 中得到的 token
	也可以创建新的「Jenkins 凭据提供者: Jenkins」
	Domain：全局凭据
	类型选：Secret text
	ID填：sonarqube-token
	Secret填：sqa_ad5df8b4b8c579fdf55ce31cc72f4217eb716057 # 从sonarqube生成的token
```
2. 系统管理 > 全局工具配置 > SonarQube Scanner > Add SonarQube Scanner
```shell
Name：sonarqube-scanner
自动安装：取消勾选
SONAR_RUNNER_HOME：/usr/local/sonar-scanner-cli
```
#### Node and Label parameter
节点标签参数配置
#### Kubernetes
jenkins + k8s 环境配置
进入 Dashboard > 系统管理 > 节点和云管理 > Clouds > New cloud
```shell
配置 k8s 集群
名称：kubernetes
点击 Kubernetes Cloud details 继续配置
1.Kubernetes 地址：
	https://kubernetes.default # 如果 jenkins 是运行在 k8s 容器中，直接配置服务名即可
	# 如果 jenkins 部署在外部，那么则不仅要配置外部访问 ip 以及 apiserver 的端口（6443），还需要配置服务证书

2.禁用HTTPS证书检查

3.Jenkins 地址：
	http://jenkins-service:8080 # 如果部署在 k8s 集群内部
	http://192.168.113.120:32479 # 如果在外部：（换成你们自己的）
4.配置完成后保存即可
```
Jenkins 配置节点标签
进入 Dashboard > 节点列表 > master > Configure > 系统管理 > 节点管理 > 列表中 master 节点最右侧的齿轮按钮
```shell
配置从节点
	标签：maven

节点属性
	勾选环境变量
	键：MAVEN_OPTS

修改标签的值与项目中 Jenkinsfile 中 agent > kubernetes > label 的值相匹配
```
#### Config File Provider
用于加载外部配置文件，如 Maven 的 settings.xml 或者 k8s 的 kubeconfig 等
kubeconfig 文件 id
```shell
进入系统管理 > Managed files > Add a new Config
	Type：选择 Custom file 点击 next
	Name: kubeconfig # 随意命名
	Content: # 在 k8s master 节点执行 `cat ~/.kube/config` 查看文件内容，并将所有内容复制到Content
	保存并提交
复制保存后文件 id 到 Jenkinsfile 中的 KUBECONFIG_CREDENTIAL_ID 处
```
#### Git Parameter
git 参数插件，在进行项目参数化构建时使用
#### Extended Choice Parameter
扩展参数的插件，后续微服务流水线使用
### 创建访问凭证
路径：Dashboard > 系统管理 > 凭据 > 系统 > 全局凭据 (unrestricted) > Add Credentials
[创建访问凭证路径](http://10.0.0.101:30001/manage/credentials/store/system/domain/_/)
####  gitlab 
```
类型：用户密码
范围：全局
用户名：root
密码：asdfjkl;
ID：gitlab-user-pass
```
#### harbor
```
类型：用户密码
范围：全局
用户名：admin
密码：asdfjkl;
ID：harbor-user-pass
```
## 案例：SpringBoot 项目 CICD
### Jenkins 创建流水线项目
在首页点击 Create a Job 创建一个流水线风格的项目 `k8s-cicd-demo`
#### Jenkins 触发器
在 Jenkins 项目配置下找到构建触发器栏目
勾选 `Build when a change is pushed to GitLab. GitLab webhook URL: http://10.0.0.101:30001/project/k8s-cicd-demo`
URL `http://10.0.0.101:30001/project/k8s-cicd-demo`就是用于配置到 gitlab 项目 webhook 的地址
启用 Gitlab 构建触发器：
	Push Events：勾选，表示有任意推送到 git 仓库的操作都会触发构建
	Opend Merge Request Events：勾选，表示有请求合并时触发构建
点击高级 > Secret Token > Generate 按钮，生成 token `4b64827ab87e62807c5220d8ef3ffe02`
保存以上配置
#### GitLab 项目 Webhook 配置
进入 GitLab 项目设置界面 > Webhooks
URL: `http://10.0.0.101:30001/project/k8s-cicd-demo`
Secret 令牌(Jenkins生成的): `4b64827ab87e62807c5220d8ef3ffe02`
按照需求勾选触发来源：
	推送事件：表示收到新的推送代码就会触发
	标签推送事件：新标签推送才会触发
	评论：根据评论决定触发
	合并请求事件：创建、更新或合并请求触发
取消 SSL 验证
点击添加 webhook 按钮，添加后可以点击测试确认链接是否可以访问
进入项目点击侧边栏设置 > Webhooks 进入配置即可
URL：在 jenkins 创建 pipeline 项目后
#### Jenkins Pipeline 配置
```shell
定义：Pipeline script from SCM # 从远程仓库拉取 Jenkinsfile 配置
SCM：Git
Repositories：
	Repository URL：http://10.0.0.6:2000/my/k8s-cicd-demo.git # 仓库地址
	Credentials：root # 仓库访问的账号密码
	Branches to build：# 选择拉取哪个分支下的代码
		*/master
		*/dev
	脚本路径：Jenkinsfile # 脚本文件名称以及所在路径
```
### 项目构建
方式一：在 Jenkins 管理后台，进入项目中点击立即构建进行项目构建
方式二：在开发工具中修改代码，并将代码提交到远程仓库自动触发构建
# 微服务
创建目录「以下操作均在该目录进行」
```shell
mkdir -p /opt/k8s/devops/microservices
```
## 项目构建
### 命名空间初始化
```shell
# 创建
kubectl apply -f /opt/k8s/devops/microservices/basics/kube-devops-namespace.yaml
```
### 镜像仓库凭证
harbor-secret
```shell
# 创建 harbor 访问账号密码「jenkins」
kubectl create secret docker-registry harbor-secret --docker-server=10.0.0.101:8888 --docker-username=admin --docker-password='asdfjkl;' -n shop-dev
# 创建 harbor 访问账号密码「jenkins」
kubectl create secret docker-registry harbor-secret --docker-server=10.0.0.101:8888 --docker-username=admin --docker-password='asdfjkl;' -n shop-master
```
### 项目环境
#### Redis
```shell
# 进入目录
cd /opt/k8s/devops/microservices/basics/redis
# 安装
helm install redis . -n redis
# 卸载
helm uninstall redis -n redis
# 查看
kubectl get po,svc -n redis
NAME                   READY   STATUS    RESTARTS   AGE
pod/redis-master-0     1/1     Running   0          8m45s
pod/redis-replicas-0   1/1     Running   0          8m45s
pod/redis-replicas-1   1/1     Running   0          7m52s
pod/redis-replicas-2   1/1     Running   0          7m23s

NAME                     TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/redis-headless   ClusterIP   None            <none>        6379/TCP         8m46s
service/redis-master     NodePort    10.105.130.89   <none>        6379:30100/TCP   8m46s
service/redis-replicas   ClusterIP   10.101.241.61   <none>        6379/TCP         8m46s

# 在集群外部工具连接redis集群，并设置key=test value=riven
# 进入从节点
kubectl exec -it redis-replicas-0 -- /bin/bash
# 命令行登录redis
I have no name!@redis-replicas-0:/$ redis-cli -a redis123
# 在从节点能拿到数据
127.0.0.1:6379> get test
"riven"
```
>集群其他服务访问`redis`链接为：
>`redis-master.redis:6379`
>因为当前redis集群部署在命名空间redis中，所以需要加上`.redis`「有service后，服务之间直接用service_name访问，避免IP&端口变化等问题」
#### RocketMQ
```shell
# 进入目录
cd /opt/k8s/devops/microservices/basics/rocketmq
# 安装
helm install rocketmq -f examples/dev.yaml charts/rocketmq -n rocketmq
# 卸载
helm uninstall rocketmq -n rocketmq
# 查看po,svc
kubectl get po,svc -n rocketmq -o wide
NAME                                      READY   STATUS    RESTARTS   AGE     IP              NODE        NOMINATED NODE   READINESS GATES
pod/rocketmq-broker-master-0              1/1     Running   0          8m33s   10.244.36.171   k8s-node1   <none>           <none>
pod/rocketmq-broker-replica-id1-0         1/1     Running   0          8m33s   10.244.36.146   k8s-node1   <none>           <none>
pod/rocketmq-dashboard-6c66bcd997-qhbjp   1/1     Running   0          8m33s   10.244.36.149   k8s-node1   <none>           <none>
pod/rocketmq-nameserver-0                 1/1     Running   0          8m31s   10.244.36.165   k8s-node1   <none>           <none>

NAME                          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE     SELECTOR
service/rocketmq-dashboard    ClusterIP   10.98.95.15      <none>        8080/TCP   8m33s   app.kubernetes.io/instance=rocketmq,app.kubernetes.io/name=rocketmq,component=dashboard
service/rocketmq-nameserver   ClusterIP   10.105.235.204   <none>        9876/TCP   8m33s   app.kubernetes.io/instance=rocketmq,app.kubernetes.io/name=rocketmq,component=nameserver
# 访问
http://rocketmq.riven.cn/
```
>集群其他服务访问`rocketmq`链接为：
>`rocketmq-nameserver.rocketmq:9876`
#### MySQL
```shell
# 进入目录
cd /opt/k8s/devops/microservices/basics/mysql
# 应用
kubectl apply -f .
# 查看
kubectl get po,svc -n mysql -o wide
NAME          READY   STATUS    RESTARTS        AGE   IP              NODE        NOMINATED NODE   READINESS GATES
pod/mysql-0   2/2     Running   2 (9m33s ago)   14m   10.244.36.181   k8s-node1   <none>           <none>
pod/mysql-1   0/2     Pending   0               12m   <none>          <none>      <none>           <none>

NAME                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE   SELECTOR
service/mysql         ClusterIP   10.110.30.167   <none>        3306/TCP         14m   app=mysql
service/mysql-read    ClusterIP   10.104.112.12   <none>        3306/TCP         14m   app=mysql
service/mysql-write   NodePort    10.97.217.84    <none>        3306:30002/TCP   14m   app=mysql,statefulset.kubernetes.io/pod-name=mysql-0
# 用工具数据库
# 链接：10.0.0.101:30002
# 账号密码：root/123456
```
>集群其他服务访问`mysql`链接为：
>`mysql-write.mysql:3306`
#### Nacos
[Redirecting to: https://nacos.io/docs/latest/what-is-nacos/](https://nacos.io/docs)
```shell
# 进入目录
cd /opt/k8s/devops/microservices/basics/nacos
# 应用
kubectl apply -f .
# 卸载
kubectl delete -f .
# 查看
kubectl get po,svc -n nacos -o wide
NAME          READY   STATUS    RESTARTS   AGE   IP               NODE        NOMINATED NODE   READINESS GATES
pod/nacos-0   1/1     Running   0          61s   10.244.169.135   k8s-node2   <none>           <none>
pod/nacos-1   1/1     Running   0          54s   10.244.36.168    k8s-node1   <none>           <none>
pod/nacos-2   1/1     Running   0          47s   10.244.169.136   k8s-node2   <none>           <none>

NAME                     TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)                                                       AGE    SELECTOR
service/nacos-headless   NodePort   10.106.205.181   <none>        8848:32244/TCP,9848:30533/TCP,9849:30007/TCP,7848:32508/TCP   103s   app=nacos
# 访问: http://10.0.0.101:32244/nacos
# 账号密码: nacos/nacos
```
>集群其他服务访问`nacos`链接为：
>`nacos-headless.nacos:8848`
#### Seata
[Kubernetes部署 | Apache Seata](https://seata.apache.org/zh-cn/docs/ops/deploy-by-kubernetes)
```shell
# 进入目录
cd /opt/k8s/devops/microservices/basics/seata
# 应用
kubectl apply -f seata-server.yaml
# 卸载
kubectl delete -f seata-server.yaml
# 查看
kubectl get po,svc -n seata-server -o wide
NAME                                READY   STATUS   RESTARTS      AGE     IP              NODE        NOMINATED NODE   READINESS GATES
pod/seata-server-76786cddfb-7bfv4   0/1     Error    4 (57s ago)   2m18s   10.244.36.159   k8s-node1   <none>           <none>

NAME                     TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE     SELECTOR
service/seata-headless   ClusterIP   10.109.229.58   <none>        8091/TCP   2m19s   k8s-app=seata-server
```
>集群其他服务访问`seata`链接为：
>`seata-headless.seata-server:8091`
#### Nexus
```shell
# 进入目录
cd /opt/k8s/devops/microservices/basics/nexus
# 应用
kubectl apply -f nexus.yaml
# 查看
kubectl get po,svc -o wide -n nexus
NAME                         READY   STATUS    RESTARTS   AGE   IP              NODE        NOMINATED NODE   READINESS GATES
pod/nexus3-c6657554b-np7pk   1/1     Running   0          29m   10.244.36.173   k8s-node1   <none>           <none>

NAME             TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE   SELECTOR
service/nexus3   NodePort   10.100.98.49   <none>        8081:30081/TCP   29m   app=nexus3
```
>集群其他服务访问`seata`链接为：
>`nexus3.nexus:30081`

**创建代理仓库**「spring」
- 在 Nexus 3 中，导航到 "Repositories"（仓库）部分。
- 点击 "Create repository"（创建仓库）按钮。
- 选择 "maven2 (proxy)"（maven2 代理）类型的仓库。
- 填写仓库的配置信息：
    - **Name（名称）：** spring。
    - **Remote storage（远程存储）：** `https://maven.aliyun.com/repository/public/spring`。
    - 其他选项可以根据需要进行配置，通常默认设置即可。
- 点击 "Create repository"（创建仓库）保存配置。

**修改 maven-public group 仓库的顺序**
```shell
maven-releases
maven-snapshots
maven-central
spring
```
修改全局配置
路径：`/Users/weichengjun/.m2/settings.xml`
链接：`http://10.0.0.101:30081/repository/maven-public/`
```xml
...
<mirrors>
	<!-- 新增内容 -->
	<mirror>
		<id>releases</id>
		<name>nexus maven</name>
		<mirrorOf>*</mirrorOf>
		<url>http://10.0.0.101:30081/repository/maven-public/</url>
	</mirror>
</mirrors>
...

...
<repositories>
	<!-- 新增内容 -->
	<repository>
		<id>repository</id>
		<name>Nexus Repository</name>
		<url>http://10.0.0.101:30081/repository/maven-public/</url>
		<releases>
			<enabled>true</enabled>
		</releases>
		<snapshots>
			<enabled>true</enabled>
		</snapshots>
	</repository>
<repositories>
...
```
#### MongoDB
#### Elasticsearch

### 服务
API 网关
用户服务
商品服务
秒杀服务
前端服务
## Jenkins CICD
### 创建流水线项目
在首页点击 Create a Job 创建一个流水线风格的项目 `shop-flashsale`
#### Jenkins General 配置
>也可不配置，构建一次后会自动生成，只是为了熟悉配置
```shell
参数化构建过程
	Git 参数
		名称：BRANCH_NAME
		参数类型：分支
		默认值：origin/master
	Extended Choice Parameter
		Name：SERVICE
		Description：选择发布的微服务
			Basic Parameter Types：Check Boxes
				Number of Visible Items: 5
				Delimiter: ,
				Choose Source for Value
					Value: 'frontend-server,shop-parent/api-gateway,shop-parent/shop-uaa,shop-parent/shop-provider/product-server,shop-parent/shop-provider/flashsale-server'
				Choose Source for Default Value
					Default value: none
				Choose Source for Value Description
					Description: 前端页面,服务网关,认证服务,商品服务,秒杀服务
	选项 参数
		名称：DOCKERHUB NAMESPACE
		选项：# 参数换行分割
			snapshots
			releases
		描述：Harbor 命名空间
	选项 参数
		名称：REPLICAS
		选项：# 参数换行分割
			1
			3
			5
			7
		描述：副本数
	字符 参数
		名称：TAG_NAME
		选项：snapshots
		描述：标签名称，必须以 v 开头，例如：v1、v1.0.0
```
#### Jenkins Pipeline 配置
```shell
定义：Pipeline script from SCM # 从远程仓库拉取 Jenkinsfile 配置
SCM：Git
Repositories：
	Repository URL：http://10.0.0.6:2000/my/shop-flashsale.git # 仓库地址
	Credentials：root # 仓库访问的账号密码
	Branches to build：# 选择拉取哪个分支下的代码
		*/master
	脚本路径：Jenkinsfile # 脚本文件名称以及所在路径
```
# Kubesphere DevOps
创建目录「以下操作均在该目录进行」
```shell
mkdir -p /opt/k8s/kubesphere/
```
## 开启 DevOps
定制资源定义 > 搜索 `ClusterConfiguration` > `ks-installer` 右边三点点击编辑YAML
设置以下内容
```shell
devops:
	enabled: true
	jenkinsJavaOpts_MaxRAM: 2g
	jenkinsJavaOpts_Xms: 512m
	jenkinsJavaOpts_Xmx: 512m
	jenkinsMemoryLim: 2Gi
	jenkinsMemoryReq: 1500Mi
	jenkinsVolumeSize: 8Gi
```
## 集成 SonarQube
[将 SonarQube 集成到流水线](https://www.kubesphere.io/zh/docs/v3.3/devops-user-guide/how-to-integrate/sonarqube/)
```shell
# 切换目录
cd /opt/k8s/kubesphere/
# 安装
helm upgrade --install sonarqube sonarqube --repo https://charts.kubesphere.io/main -n kubesphere-devops-system  --create-namespace --set service.type=NodePort
# 卸载
helm uninstall sonarqube -n kubesphere-devops-system
# 查看 sonarqube
kubectl get po -n kubesphere-devops-system
# 获取 SonarQube 控制台地址
export NODE_PORT=$(kubectl get --namespace kubesphere-devops-system -o jsonpath="{.spec.ports[0].nodePort}" services sonarqube-sonarqube)
export NODE_IP=$(kubectl get nodes --namespace kubesphere-devops-system -o jsonpath="{.items[0].status.addresses[0].address}")
echo http://$NODE_IP:$NODE_PORT
# 执行上面命令得到访问地址
http://10.0.0.101:32696
```
#### 创建token
My Account > Security > Generate Tokens
```shell
Name: ks-jenkins
b401feab7842893b4e26f8529d803447eb41ac0e
```
#### 创建 Webhook 服务器
执行以下命令获取 SonarQube Webhook 的地址。
```shell
export NODE_PORT=$(kubectl get --namespace kubesphere-devops-system -o jsonpath="{.spec.ports[0].nodePort}" services devops-jenkins)
export NODE_IP=$(kubectl get nodes --namespace kubesphere-devops-system -o jsonpath="{.items[0].status.addresses[0].address}")
echo http://$NODE_IP:$NODE_PORT/sonarqube-webhook/
```
执行上面命令得到 webhook 地址
```shell
http://10.0.0.101:30180/sonarqube-webhook/
```
依次点击 Administration > Configuration > Webhooks > Create
```shell
Name: ks-jenkins
Url: http://10.0.0.101:30180/sonarqube-webhook/
```
#### 将 SonarQube 配置添加到 ks-installer
执行以下命令编辑 `ks-installer`。
```shell
kubectl edit cc -n kubesphere-system ks-installer
```
搜寻至 `devops`。添加字段 `sonarqube` 并在其下方指定 `externalSonarUrl` 和 `externalSonarToken`。
```yml
devops:
	...
    sonarqube:
      externalSonarToken: b401feab7842893b4e26f8529d803447eb41ac0e
      externalSonarUrl: 'http://10.0.0.101:30180'
```
完成操作后保存此文件。
#### 将 SonarQube 服务器添加至 Jenkins
```shell
# 执行以下命令获取 Jenkins 的地址
export NODE_PORT=$(kubectl get --namespace kubesphere-devops-system -o jsonpath="{.spec.ports[0].nodePort}" services devops-jenkins)
export NODE_IP=$(kubectl get nodes --namespace kubesphere-devops-system -o jsonpath="{.items[0].status.addresses[0].address}")
echo http://$NODE_IP:$NODE_PORT
```
执行上面命令得到 webhook 地址
```http
http://10.0.0.101:30180
```
请使用地址 `http://<Node IP>:30180` 访问 Jenkins。安装 KubeSphere 时，默认情况下也会安装 Jenkins 仪表板。此外，Jenkins 还配置有 KubeSphere LDAP，这意味着您可以直接使用 KubeSphere 帐户（例如 `admin/P@88w0rd`）登录 Jenkins。有关配置 Jenkins 的更多信息，请参见 [Jenkins 系统设置](https://www.kubesphere.io/zh/docs/v3.3/devops-user-guide/how-to-use/pipelines/jenkins-setting/)。
>备注
>取决于您的实例的部署位置，您可能需要设置必要的端口转发规则，并在您的安全组中放行端口 `30180`，以便访问 Jenkins。

点击左侧导航栏中的**系统管理**。向下翻页找到并点击**系统配置**。搜寻到 **SonarQube servers**，然后点击 **Add SonarQube**。
输入 **Name** 和 **Server URL** (`http://<Node IP>:<NodePort>`)。点击**添加**，选择 **Jenkins**，然后在弹出的对话框中用 SonarQube 管理员令牌创建凭证（如下方第二张截图所示）。创建凭证后，从 **Server authentication token** 旁边的下拉列表中选择该凭证。点击**应用**完成操作。
#### 将 sonarqubeURL 添加到 KubeSphere 控制台
指定 `sonarqubeURL`，以便可以直接从 KubeSphere 控制台访问 SonarQube。
1. 执行以下命令：
```shell
kubectl edit  cm -n kubesphere-system  ks-console-config
```
2. 搜寻到 `data.client.enableKubeConfig`，在下方添加 `devops` 字段并指定 `sonarqubeURL`。
```yml
client:
  enableKubeConfig: true
  devops: # 手动添加该字段。
    sonarqubeURL: http://10.0.0.101:31377 # SonarQube IP 地址。
```
3. 保存该文件。
#### 重启服务
```
kubectl -n kubesphere-devops-system rollout restart deploy devops-apiserver
kubectl -n kubesphere-system rollout restart deploy ks-console
```
## 更新 settings.xml
集群管理 > 配置 > 配置字典下搜索 `ks-devops-agent`
点进去后点击更多操作 > 编辑 YAML 添加如下内容并保存
```xml
      <servers>
        <server>
          <id>releases</id>
          <username>admin</username>
          <password>asdfjkl;</password>
        </server>
        <server>
          <id>snapshots</id>
          <username>admin</username>
          <password>asdfjkl;</password>
        </server>
      </servers>

      <mirrors>
      	<mirror>
      		<id>releases</id>
      		<name>nexus maven</name>
      		<mirrorOf>*</mirrorOf>
      		<url>http://10.0.0.101:30081/repository/maven-public/</url>
      	</mirror>
      </mirrors>
```
## 创建devops
### 创建企业空间
工作台 > 企业空间 > 创建
创建企业空间内容
```
名称: devops
```
### 创建项目和DevOps项目
点击进入devops企业空间
创建两个项目
- 项目：存放项目中的 Deployment，StatefulSet 等 Kubernetes 资源对象
- DevOps 项目：用于完成 CICD 流程，使用 Jenkins 进行构建。
## 单体服务
### 项目
ks-k8s-cicd-dev
### 创建凭证
[创建凭证](http://10.0.0.101:30880/devops/clusters/host/devops/ks-k8s-cicd-devopsp47cd/credentials)
#### harbor-secret
```shell
# 创建 harbor 访问账号密码「jenkins」
kubectl create secret docker-registry harbor-secret --docker-server=10.0.0.101:8888 --docker-username=admin --docker-password='asdfjkl;' -n ks-k8s-cicd-dev
```
####  gitlab 
```
类型：用户密码
ID：gitlab-user-pass
用户名：root
密码：asdfjkl;
```
#### harbor
```
类型：用户密码
ID：harbor-user-pass
用户名：admin
密码：asdfjkl;
```
#### kubeconfig-id
```
创建kubeconfg类型凭证，内容自动生成
```
### DevOps项目
ks-k8s-cicd-devops
### 创建流水线
点击`ks-k8s-cicd-devops`进去创建流水线`ks-k8s-cicd-pipeline` > 通过 `jenkinsfile` 创建
```groovy
pipeline {
  agent {
    node {
      label 'maven'
    }
  }
  stages {
    stage('clone code') {
      agent none
      steps {
        container('maven') {
          git(url: 'http://10.0.0.6:2000/my/k8s-cicd-demo.git', credentialsId: 'gitlab-user-pass', changelog: true, poll: false, branch: 'kubesphere')
        }
      }
    }

    stage('unit test') {
      agent none
      steps {
        container('maven') {
          sh 'mvn clean test'
        }

      }
    }

    stage('build & push') {
      agent none
      steps {
        container('maven') {
          sh 'mvn clean package -DskipTests'
          sh 'docker build -f Dockerfile -t $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:SNAPSHOT-$BUILD_NUMBER .'
          withCredentials([usernamePassword(credentialsId : 'harbor-user-pass' ,usernameVariable : 'DOCKER_USERNAME' ,passwordVariable : 'DOCKER_PASSWORD' ,)]) {
            sh '''
              echo "$DOCKER_PASSWORD" | docker login $REGISTRY -u "$DOCKER_USERNAME" --password-stdin
              docker push $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:SNAPSHOT-$BUILD_NUMBER
            '''
          }
        }
      }
    }

    stage('push latest') {
      agent none
      steps {
        container('maven') {
          sh 'docker tag $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:SNAPSHOT-$BUILD_NUMBER $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:latest'
          sh 'docker push $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:latest'
        }
      }
    }

    stage('deploy to dev') {
      agent none
      steps {
        container('maven') {
          input(message: '发布到开发环境?', ok: '确认发布', submitter: '')
          withCredentials([kubeconfigContent(credentialsId : 'kubeconfig-id' ,variable : 'KUBECONFIG_CONFIG' ,)]) {
            sh 'mkdir -p ~/.kube/'
            sh 'echo "$KUBECONFIG_CONFIG" > ~/.kube/config'
            sh '''
              sed -i "s#REGISTRY#${REGISTRY}#" deploy/cicd-demo-dev.yaml
              sed -i "s#DOCKERHUB_NAMESPACE#${DOCKERHUB_NAMESPACE}#" deploy/cicd-demo-dev.yaml
              sed -i "s#APP_NAME#${APP_NAME}#" deploy/cicd-demo-dev.yaml
              sed -i "s#BUILD_NUMBER#${BUILD_NUMBER}#" deploy/cicd-demo-dev.yaml
              kubectl apply -f deploy/cicd-demo-dev.yaml
            '''
          }

        }

      }
    }

    stage('push with tag') {
      agent none
      when {
        expression {
          return env.TAG_NAME =~ /^v.*$/
        }
      }
      steps {
        input(message: '发布带有标签的镜像?', ok: '确认发布', submitter: '')
        withCredentials([usernamePassword(credentialsId : 'gitlab-user-pass', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
          sh '''
            # 设置 Git 全局邮箱
            git config --global user.email "1054487195@qq.com"
            # 设置 Git 全局用户名
            git config --global user.name "riven"
            # 创建 Git 标签
            git tag -a ${TAG_NAME} -m "${TAG_NAME}"
            # 推送 Git 标签
            git push http://${GIT_USERNAME}:${GIT_PASSWORD}@${GIT_REPO_URL}/${GIT_GROUP}/k8s-cicd-demo.git --tags --ipv4
            # Docker 镜像打标签
            docker tag ${REGISTRY}/${DOCKERHUB_NAMESPACE}/${APP_NAME}:SNAPSHOT-${BUILD_NUMBER} ${REGISTRY}/${DOCKERHUB_NAMESPACE}/${APP_NAME}:${TAG_NAME}
            # 推送 Docker 镜像
            docker push ${REGISTRY}/${DOCKERHUB_NAMESPACE}/${APP_NAME}:${TAG_NAME}
          '''
        }
      }
    }

    stage('deploy to master') {
      agent none
      when {
        expression {
          return env.TAG_NAME =~ /^v.*$/ && env.GIT_BRANCH == 'kubesphere-master'
        }
      }
      steps {
        input(message: '发布到预发布环境?', ok: '确认发布', submitter: '')
        sh '''
          sed -i "s#REGISTRY#$REGISTRY#" deploy/cicd-demo.yaml
          sed -i "s#DOCKERHUB_NAMESPACE#$DOCKERHUB_NAMESPACE#" deploy/cicd-demo.yaml
          sed -i "s#APP_NAME#$APP_NAME#" deploy/cicd-demo.yaml
          sed -i "s#TAG_NAME#$TAG_NAME#" deploy/cicd-demo.yaml
          kubectl apply -f deploy/cicd-demo.yaml
        '''
      }
    }

  }
  environment {
    DOCKER_CREDENTIAL_ID = 'harbor-user-pass'
    GIT_REPO_URL = '10.0.0.6:2000'
    GIT_CREDENTIAL_ID = 'gitlab-user-pass'
    GIT_GROUP = 'my'
    KUBECONFIG_CREDENTIAL_ID = 'kubeconfig-id'
    REGISTRY = '10.0.0.6:8888'
    DOCKERHUB_NAMESPACE = 'k8s'
    APP_NAME = 'k8s-cicd-demo'
    TAG_NAME = "${params.TAG_NAME}"
    BRANCH_NAME = "${params.BRANCH_NAME}"
  }
  parameters {
    string(name: 'TAG_NAME', defaultValue: '', description: '部署版本：必须以 v 开头，例如：v1、v1.0.0')
    string(name: 'BRANCH_NAME', defaultValue: 'kubesphere', description: '请输入要构建的分支名称')
  }
}
```

## 微服务
### 项目
ks-shop-dev
### 创建凭证
[创建凭证](http://10.0.0.101:30880/devops/clusters/host/devops/ks-shop-dev-devopsf5mrd/credentials)
#### harbor-secret
```shell
# 创建 harbor 访问账号密码「jenkins」
kubectl create secret docker-registry harbor-secret --docker-server=10.0.0.101:8888 --docker-username=admin --docker-password='asdfjkl;' -n ks-shop-dev
```
####  gitlab 
```
类型：用户密码
ID：gitlab-user-pass
用户名：root
密码：asdfjkl;
```
#### harbor
```
类型：用户密码
ID：harbor-user-pass
用户名：admin
密码：asdfjkl;
```
#### kubeconfig-id
```
类型：kubeconfig
ID：kubeconfig-id
内容：自动会填充kubeconfig内容
```
### DevOps项目
ks-shop-dev-devops
### 创建流水线
点击`ks-shop-dev-devops`进去创建流水线`ks-shop-dev-pipeline` > 通过 `jenkinsfile` 创建
```groovy
pipeline {
    agent {
        // k8s 相关配置
        kubernetes {
            // 标签选择器：找到标签为 maven 的 jenkins 节点，进行执行脚本
            label 'maven'
        }
    }

    stages {
        stage('checkout scm') {
            agent none
            steps {
                git(url: "http://$GIT_REPO_URL/$GIT_GROUP/shop-flashsale.git", credentialsId: 'gitlab-user-pass', changelog: true, poll: false, branch: "$BRANCH_NAME")
            }
        }

        stage('unit test') {
          agent none
          steps {
            container('maven') {
                sh '''
                    cd ${SERVICE} && mvn clean test
                '''
            }
          }
        }

//         stage('sonarqube analysis') {
//           agent none
//           steps {
//             withCredentials([string(credentialsId : 'sonarqube' ,variable : 'SONAR_TOKEN' ,)]) {
//               withSonarQubeEnv('sonar') {
//                 container('maven') {
//                     sh '''
//                         service_name=${SERVICE#*/}
//                         service_name=${service_name#*/}
//
//                         cd ${SERVICE}
//                         mvn sonar:sonar -Dsonar.projectKey=${service_name}
//                         echo "mvn sonar:sonar -Dsonar.projectKey=${service_name}"
//                     '''
//                 }
//               }
//             }
//             timeout(time: 1, unit: 'HOURS') {
//               waitForQualityGate true
//             }
//           }
//         }

        stage('build & push') {
            agent none
            steps {
                withCredentials([usernamePassword(credentialsId : 'harbor-user-pass' ,passwordVariable : 'DOCKER_PASSWORD' ,usernameVariable : 'DOCKER_USERNAME' ,)]) {
                    container('maven') {
                        sh '''
                            cd ${SERVICE}
                            mvn clean package -DskipTests
                            cd ${WORKSPACE}
                            chmod -R 777 deploy/copy.sh && deploy/copy.sh
                        '''
                        sh '''
                            echo "${DOCKER_PASSWORD}" | docker login ${REGISTRY} -u "${DOCKER_USERNAME}" --password-stdin

                            service_name=${SERVICE#*/}
                            service_name=${service_name#*/}
                            cd deploy/${service_name}/build

                            docker build -f Dockerfile -t ${REGISTRY}/${DOCKERHUB_NAMESPACE}/${service_name}:SNAPSHOT-$BUILD_NUMBER .
                            docker push ${REGISTRY}/${DOCKERHUB_NAMESPACE}/${service_name}:SNAPSHOT-${BUILD_NUMBER}
                        '''
                    }
                }
            }
        }

        stage('push latest') {
            agent none
            steps {
                container('maven') {
                    sh '''
                        service_name=${SERVICE#*/}
                        service_name=${service_name#*/}
                        cd deploy/${service_name}/build

                        docker tag ${REGISTRY}/${DOCKERHUB_NAMESPACE}/${service_name}:SNAPSHOT-${BUILD_NUMBER} ${REGISTRY}/${DOCKERHUB_NAMESPACE}/${service_name}:latest
                        docker push ${REGISTRY}/${DOCKERHUB_NAMESPACE}/${service_name}:latest
                    '''
                }
            }
        }

        stage('deploy to dev') {
            agent none
            steps {
                container('maven') {
                    withCredentials([kubeconfigContent(credentialsId : "$KUBECONFIG_CREDENTIAL_ID" ,variable : 'ADMIN_KUBECONFIG' ,)]) {
                        sh '''
                        service_name=${SERVICE#*/}
                        service_name=${service_name#*/}
                        cd deploy/${service_name}

                        sed -i\'\' "s#REGISTRY#${REGISTRY}#" deployment.yaml
                        sed -i\'\' "s#DOCKERHUB_NAMESPACE#${DOCKERHUB_NAMESPACE}#" deployment.yaml
                        sed -i\'\' "s#APP_NAME#${service_name}#" deployment.yaml
                        sed -i\'\' "s#BUILD_NUMBER#${BUILD_NUMBER}#" deployment.yaml
                        sed -i\'\' "s#REPLICAS#${REPLICAS}#" deployment.yaml

                        mkdir ~/.kube
                        echo "$ADMIN_KUBECONFIG" > ~/.kube/config

                        if [ ${service_name} != "frontend-server" ]; then
                            kubectl create cm ${service_name}-yml --dry-run=\'client\' -oyaml --from-file=build/target/bootstrap.yml -n ks-shop-dev > ${service_name}-configmap.yml
                        fi

                        kubectl apply -f .'''
                    }
                }
            }
        }

        stage('push with tag') {
            agent none
            when {
                expression {
                    return params.TAG_NAME =~ /v.*/
                }
            }
            steps {
              input(id: 'release-image-with-tag', message: 'release image with tag?')
              container('maven') {
                  withCredentials([usernamePassword(credentialsId : 'gitlab-user-pass' ,passwordVariable : 'GIT_PASSWORD' ,usernameVariable : 'GIT_USERNAME' ,)]) {
                      sh 'git config --global user.email "1054487195@qq.com" '
                      sh 'git config --global user.name "riven" '
                      sh 'git tag -a ${TAG_NAME} -m "${TAG_NAME}" '
                      sh 'git push http://${GIT_USERNAME}:${GIT_PASSWORD}@${GIT_REPO_URL}/${GIT_ACCOUNT}/${APP_NAME}.git --tags --ipv4'
                  }

                  sh '''
                      service_name=${SERVICE#*/}
                      service_name=${service_name#*/}

                      docker tag ${REGISTRY}/${DOCKERHUB_NAMESPACE}/${service_name}:SNAPSHOT-${BUILD_NUMBER} ${REGISTRY}/${DOCKERHUB_NAMESPACE}/${service_name}:${TAG_NAME}
                      docker push ${REGISTRY}/${DOCKERHUB_NAMESPACE}/${service_name}:${TAG_NAME}
                  '''
              }
            }
        }

        stage('deploy to production') {
            agent none
            when {
                expression {
                    return params.TAG_NAME =~ /v.*/
                }
            }
            steps {
                input(id: 'deploy-to-production', message: 'deploy to production?')
                container('maven') {
                  withCredentials([kubeconfigContent(credentialsId : "$KUBECONFIG_CREDENTIAL_ID",variable : 'ADMIN_KUBECONFIG' ,)]) {
                      sh '''
                          service_name=${SERVICE#*/}
                          service_name=${service_name#*/}
                          cd deploy/${service_name}/prod

                          sed -i\'\' "s#REGISTRY#${REGISTRY}#" deployment.yaml
                          sed -i\'\' "s#DOCKERHUB_NAMESPACE#${DOCKERHUB_NAMESPACE}#" deployment.yaml
                          sed -i\'\' "s#APP_NAME#${service_name}#" deployment.yaml
                          sed -i\'\' "s#BUILD_NUMBER#${BUILD_NUMBER}#" deployment.yaml
                          sed -i\'\' "s#REPLICAS#${REPLICAS}#" deployment.yaml

                          mkdir ~/.kube
                          echo "$ADMIN_KUBECONFIG" > ~/.kube/config

                          if [ ${service_name} != "frontend-server" ]; then
                              kubectl create cm ${service_name}-yml --dry-run=\'client\' -oyaml --from-file=build/target/bootstrap.yml -n ks-shop-flashsale > ${service_name}-configmap.yml
                          fi
                          kubectl apply -f .
                      '''
                  }
                }
            }
        }
    }

    // 参数化构建
    // 在 Jenkins 构建作业中，点击 Build with Parameters 按钮会进入参数化构建页面
    // 通过将经常变动的参数，配置为参数，可以更方便灵活的实现项目构建
    parameters {
      choice(name: 'SERVICE', choices: ['frontend-server', 'shop-parent/api-gateway', 'shop-parent/shop-uaa', 'shop-parent/shop-provider/product-server', 'shop-parent/shop-provider/flashsale-server'], description: '请选择要部署的服务')
      choice(name: 'DOCKERHUB_NAMESPACE', choices: ['snapshots', 'releases'], description: '请选择部署到哪个镜像仓库')
      choice(name: 'REPLICAS', choices: ['1', '3', '5', '7'], description: '请选择构建后的副本数')
      string(name: 'BRANCH_NAME', defaultValue: 'kubesphere', description: '请输入要构建的分支名称')
      string(name: 'TAG_NAME', defaultValue: 'snapshot', description: '部署版本：必须以 v 开头，例如：v1、v1.0.0')
    }

    environment {
      APP_NAME = 'shop-flashsale' // 应用名称
      DOCKER_CREDENTIAL_ID = 'harbor-user-pass' // Harbor 访问凭证 id
      REGISTRY = '10.0.0.6:8888' // Harbor 仓库地址
      GIT_REPO_URL = '10.0.0.6:2000' // GIT 仓库地址
      GIT_CREDENTIAL_ID = 'gitlab-user-pass' // GIT 访问凭证
      GIT_GROUP = 'my' // GIT 分组
      SONAR_CREDENTIAL_ID = 'sonarqube-token' // SonarQube Token
      KUBECONFIG_CREDENTIAL_ID = 'kubeconfig-id' // kubeconfig 文件 id
    }
}
```