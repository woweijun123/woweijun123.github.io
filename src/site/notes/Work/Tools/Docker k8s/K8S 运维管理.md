---
{"dg-publish":true,"permalink":"/Work/Tools/Docker k8s/K8S 运维管理/","title":"K8S 运维管理","tags":["flashcards"],"noteIcon":"","created":"2026-03-16T22:46:07.000+08:00","updated":"2026-03-16T22:46:07.000+08:00"}
---

# Helm 包管理器
创建目录「以下操作均在该目录进行」
```shell
mkdir -p /opt/k8s/helm/
```
## Helm 简介
[Artifact Hub](https://artifacthub.io/)
Helm 是一种 **Kubernetes 的包管理器**，它允许您打包、配置和部署 Kubernetes 应用程序。Helm 使用称为 Charts 的打包格式，可以将复杂的 Kubernetes 应用程序定义为一个易于管理的单元。

Helm 是查找、分享和使用软件构件 Kubernetes 的最优方式。  
Helm 管理名为 chart 的 Kubernetes 包的工具。Helm 可以做以下的事情：
- 从头开始创建新的 chart
- 将 chart 打包成归档(tgz)文件
- 与存储 chart 的仓库进行交互
- 在现有的 Kubernetes 集群中安装和卸载 chart
- 管理与 Helm 一起安装的 chart 的发布周期
## Helm 架构
### 主要概念
- **Chart:** Helm 的打包格式，包含部署 Kubernetes 应用程序所需的所有资源定义。
- **Release:** Chart 的运行实例，表示在 Kubernetes 集群中部署的一个应用程序。
- **Repository:** Helm chart 的仓库，用于存储和共享 Charts。
- **Values:** 用于自定义 Chart 部署的配置值。
### Helm 的优势
- **简化部署：** Helm 可以将复杂的 Kubernetes 应用程序部署简化为一条命令。
- **版本控制：** Helm 可以跟踪应用程序的版本，并支持回滚到以前的版本。
- **配置管理：** Helm 允许您使用 Values 文件自定义应用程序的配置。
- **共享和重用：** Helm Charts 可以轻松地共享和重用，促进 Kubernetes 应用程序的生态系统。
### 组件
#### Helm 客户端
**Helm 客户端** 是终端用户的命令行客户端。负责以下内容：
- 本地 chart 开发
- 管理仓库
- 管理发布
- 与 Helm 库建立接口
    - 发送安装的 chart
    - 发送升级或卸载现有发布的请求
#### Helm 库
**Helm 库** 提供执行所有 Helm 操作的逻辑。与 Kubernetes API 服务交互并提供以下功能：
- 结合 chart 和配置来构建版本
- 将 chart 安装到 Kubernetes 中，并提供后续发布对象
- 与 Kubernetes 交互升级和卸载 chart

独立的 Helm 库封装了 Helm 逻辑以便不同的客户端可以使用它。
## 安装 Helm
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
## Helm 的常用命令
```shell
## 添加 Helm 仓库
helm repo add stable https://charts.helm.sh/stable  # 添加名为 stable 的仓库，URL 为 https://charts.helm.sh/stable
helm repo add bitnami https://charts.bitnami.com/bitnami #添加bitnami仓库

## 列出已添加的 Helm 仓库
helm repo list  # 列出所有已添加的 Helm 仓库
# 删除名为 stable 的仓库
helm repo remove stable
## 更新 Helm 仓库
helm repo update  # 更新所有已添加的 Helm 仓库
## 在仓库中搜索 chart
helm search repo nginx  # 在仓库中搜索包含 "nginx" 的 chart
## 下载 chart 到本地
helm pull stable/nginx  # 从 stable 仓库下载 nginx chart 到本地
## 安装 chart
helm install my-nginx stable/nginx  # 安装 stable 仓库中的 nginx chart，并命名为 my-nginx
#helm install my-nginx ./nginx #安装当前目录的nginx chart
## 列出已安装的 chart
helm list  # 列出所有已安装的 chart
## 升级 chart
helm upgrade my-nginx stable/nginx  # 升级名为 my-nginx 的 chart 到最新版本
## 卸载 chart
helm uninstall my-nginx  # 卸载名为 my-nginx 的 chart
## 查看 chart 的 values.yaml 文件
helm show values stable/nginx  # 显示 stable 仓库中 nginx chart 的 values.yaml 文件内容
## 创建一个新的 chart
helm create my-chart  # 创建名为 my-chart 的新 chart 项目
## 打包 chart
helm package my-chart  # 将 my-chart 打包成 .tgz 文件
## 本地渲染 chart 模板
helm template my-nginx stable/nginx  # 渲染 stable 仓库中 nginx chart 的模板，并命名为 my-nginx
## 获取release的values
helm get values my-nginx #获取my-nginx这个release的values
## 获取release的历史记录
helm history my-nginx #获取my-nginx这个release的历史记录
## 管理 chart 依赖
helm dependency update my-chart # 更新 my-chart 的依赖
helm dependency build my-chart # 构建 my-chart 的依赖
## 检查 chart 配置是否有误
helm lint my-chart # 检查 my-chart 的配置是否有误
## 回滚 release 到历史版本
helm rollback my-nginx 1 # 回滚名为 my-nginx 的 release 到第一个历史版本
```
## chart 详解
### 目录结构
```
mychart
├── Chart.yaml
├── charts # 该目录保存其他依赖的 chart（子 chart）
├── templates # chart 配置模板，用于渲染最终的 Kubernetes YAML 文件
│   ├── NOTES.txt # 用户运行 helm install 时候的提示信息
│   ├── _helpers.tpl # 用于创建模板时的帮助类
│   ├── deployment.yaml # Kubernetes deployment 配置
│   ├── ingress.yaml # Kubernetes ingress 配置
│   ├── service.yaml # Kubernetes service 配置
│   ├── serviceaccount.yaml # Kubernetes serviceaccount 配置
│   └── tests
│       └── test-connection.yaml
└── values.yaml # 定义 chart 模板中的自定义配置的默认值，可以在执行 helm install 或 helm update 的时候覆盖
```
### Redis chart 实践
#### 修改 helm 源
```shell
# 查看默认仓库
helm repo list

# 添加仓库
helm repo add bitnami https://charts.bitnami.com/bitnami
# 阿里云
helm repo add aliyun https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
helm repo add azure http://mirror.azure.cn/kubernetes/charts
```
#### 搜索 redis chart
```shell
# 搜索 redis chart
helm search repo redis

# 查看安装说明
helm show readme azure/redis
```
#### 修改配置安装
```shell
# 创建redis目录，并切换
mkdir -p /opt/k8s/helm/redis && cd /opt/k8s/helm/redis
# 先将 chart 拉到本地
helm pull azure/redis

# 解压后，修改 values.yaml 中的参数
tar -xvf redis-10.5.7.tgz

# 修改values.yaml文件，内容如下
...
storageClass: nfs-storage # 修改
redis:  
  password: redis # 修改
cluster:  
  enabled: false # 修改
...
修改实例存储大小 persistence.size 为需要的大小
修改 service.nodePorts.redis 向外暴露端口，范围 <30000-32767>

# 安装
helm install redis ./redis
# 卸载
helm uninstall redis
```
#### 查看安装情况
```shell
# 查看 helm 安装列表
helm list

# 查看 redis 命名空间下所有对象信息
kubectl get all
```
#### 升级与回滚
```shell
# 要想升级 chart 可以修改本地的 chart 配置并执行：
helm upgrade [RELEASE] [CHART] [flags]
helm upgrade redis ./redis
# 使用 helm ls 的命令查看当前运行的 chart 的 release 版本，并使用下面的命令回滚到历史版
helm ls
NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
redis   default         2               2025-02-17 17:39:56.253158943 +0800 CST deployed        redis-10.5.7    5.0.7

# 回滚到历史版本
helm rollback <RELEASE> [REVISION] [flags]
# 查看历史
helm history redis
# 回退到上一版本
helm rollback redis
# 回退到指定版本
helm rollback redis 3
```
#### helm 卸载 redis
```shell
helm delete redis
```

# k8s 集群监控
## 监控方案
### Heapster
Heapster 是容器集群监控和性能分析工具，天然的支持Kubernetes 和 CoreOS。  
Kubernetes 有个出名的监控 agent---cAdvisor。在每个kubernetes Node 上都会运行 cAdvisor，它会收集本机以及容器的监控数据(cpu,memory,filesystem,network,uptime)。  
在较新的版本中，K8S 已经将 cAdvisor 功能集成到 kubelet 组件中。每个 Node 节点可以直接进行 web 访问。

### Weave Scope
[Weave Scope](https://www.weave.works/oss/scope/) 可以监控 kubernetes 集群中的一系列资源的状态、资源使用情况、应用拓扑、scale、还可以直接通过浏览器进入容器内部调试等，其提供的功能包括：
- 交互式拓扑界面
- 图形模式和表格模式
- 过滤功能
- 搜索功能
- 实时度量
- 容器排错
- 插件扩展

### Prometheus
Prometheus 是一套开源的监控系统、报警、时间序列的集合，最初由 SoundCloud 开发，后来随着越来越多公司的使用，于是便独立成开源项目。自此以后，许多公司和组织都采用了 Prometheus 作为监控告警工具。

## Prometheus 监控 k8s
创建目录「以下操作均在该目录进行」
```shell
mkdir -p /opt/k8s/kube-prometheus/
```
### kube-prometheus
#### 替换国内镜像
```shell
sed -i 's/quay.io/quay.mirrors.ustc.edu.cn/g' prometheusOperator-deployment.yaml
sed -i 's/quay.io/quay.mirrors.ustc.edu.cn/g' prometheus-prometheus.yaml
sed -i 's/quay.io/quay.mirrors.ustc.edu.cn/g' alertmanager-alertmanager.yaml
sed -i 's/quay.io/quay.mirrors.ustc.edu.cn/g' kubeStateMetrics-deployment.yaml
sed -i 's/k8s.gcr.io/lank8s.cn/g' kubeStateMetrics-deployment.yaml
sed -i 's/quay.io/quay.mirrors.ustc.edu.cn/g' nodeExporter-daemonset.yaml
sed -i 's/quay.io/quay.mirrors.ustc.edu.cn/g' prometheusAdapter-deployment.yaml
sed -i 's/k8s.gcr.io/lank8s.cn/g' prometheusAdapter-deployment.yaml

# 查看是否还有国外镜像
grep -r "image: " *
```
#### 修改访问入口
```shell
分别修改 prometheus、alertmanager、grafana 中的 spec.type 为 NodePort 方便测试访问
```
#### 安装
```shell
# 使用服务器端应用（SSA）(生产环境推荐添加参数 --server-side)将 `manifests/setup` 目录下的所有 Kubernetes 资源清单应用到 Kubernetes 集群。
kubectl apply --server-side -f manifests/setup
# 等待 `monitoring` 命名空间中的所有 `CustomResourceDefinition` 资源达到 `Established` 状态。
kubectl wait \
--for condition=Established \
--all CustomResourceDefinition \
--namespace=monitoring
kubectl apply -f manifests/
```
#### 配置 Ingress
```shell
# 通过域名访问（没有域名可以在主机配置 hosts）
10.0.0.101 grafana.wolfcode.cn
10.0.0.101 prometheus.wolfcode.cn
10.0.0.101 alertmanager.wolfcode.cn

# 创建 prometheus-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  namespace: monitoring
  name: prometheus-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: grafana.wolfcode.cn  # 访问 Grafana 域名
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: grafana
            port:
              number: 3000
  - host: prometheus.wolfcode.cn  # 访问 Prometheus 域名
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: prometheus-k8s 
            port:
              number: 9090
  - host: alertmanager.wolfcode.cn  # 访问 alertmanager 域名
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: alertmanager-main
            port:
              number: 9093

# 创建 ingress
kubectl apply -f prometheus-ingress.yaml
```
#### 卸载
```shell
kubectl delete --ignore-not-found=true -f /opt/k8s/kube-prometheus/manifests -f /opt/k8s/kube-prometheus/manifests/setup
```

# ELK 日志管理
![image 20250218113627|1300](https://weichengjun2.dpdns.org//i/2025/02/18/67b40052df9ae.png)
## ELK 组成
### Elasticsearch
ES 作为一个搜索型文档数据库，拥有优秀的搜索能力，以及提供了丰富的 REST API 让我们可以轻松的调用接口。
### Filebeat
Filebeat 是一款轻量的数据收集工具
### Logstash
通过 Logstash 同样可以进行日志收集，但是若每一个节点都需要收集时，部署 Logstash 有点过重，因此这里主要用到 Logstash 的数据清洗能力，收集交给 Filebeat 去实现
### Kibana
Kibana 是一款基于 ES 的可视化操作界面工具，利用 Kibana 可以实现非常方便的 ES 可视化操作
## 集成 ELK
创建目录「以下操作均在该目录进行」
```shell
mkdir -p /opt/k8s/elk/
```
### 部署 es 搜索服务
创建`es.yaml`
```yml
---
apiVersion: v1
kind: Service # 定义资源类型为 Service（服务）
metadata:
  name: elasticsearch-logging # Service 的名称
  namespace: kube-logging # Service 所在的命名空间
  labels:
    k8s-app: elasticsearch-logging # 标签，用于选择 Pod
    kubernetes.io/cluster-service: "true" # Kubernetes 集群服务标签
    addonmanager.kubernetes.io/mode: Reconcile # Addon 管理器模式
    kubernetes.io/name: "Elasticsearch" # Kubernetes 名称标签
spec:
  ports:
    - port: 9200 # Service 暴露的端口
      protocol: TCP # 协议类型
      targetPort: db # Pod 上的目标端口，对应容器端口的名称，或者端口号
  selector:
    k8s-app: elasticsearch-logging # 选择具有此标签的 Pod
---
# RBAC authn and authz （RBAC 认证和授权）
apiVersion: v1
kind: ServiceAccount # 定义资源类型为 ServiceAccount（服务账号）
metadata:
  name: elasticsearch-logging # ServiceAccount 的名称
  namespace: kube-logging # ServiceAccount 所在的命名空间
  labels:
    k8s-app: elasticsearch-logging # 标签
    kubernetes.io/cluster-service: "true" # Kubernetes 集群服务标签
    addonmanager.kubernetes.io/mode: Reconcile # Addon 管理器模式
---
kind: ClusterRole # 定义资源类型为 ClusterRole（集群角色）
apiVersion: rbac.authorization.k8s.io/v1 # API 版本
metadata:
  name: elasticsearch-logging # ClusterRole 的名称
  labels:
    k8s-app: elasticsearch-logging # 标签
    kubernetes.io/cluster-service: "true" # Kubernetes 集群服务标签
    addonmanager.kubernetes.io/mode: Reconcile # Addon 管理器模式
rules:
  - apiGroups:
      - "" # 核心 API 组
    resources:
      - "services" # 授权的资源类型：Services
      - "namespaces" # 授权的资源类型：Namespaces
      - "endpoints" # 授权的资源类型：Endpoints
    verbs:
      - "get" # 允许的操作：get（获取）
---
kind: ClusterRoleBinding # 定义资源类型为 ClusterRoleBinding（集群角色绑定）
apiVersion: rbac.authorization.k8s.io/v1 # API 版本
metadata:
  namespace: kube-logging # ClusterRoleBinding 所在的命名空间
  name: elasticsearch-logging # ClusterRoleBinding 的名称
  labels:
    k8s-app: elasticsearch-logging # 标签
    kubernetes.io/cluster-service: "true" # Kubernetes 集群服务标签
    addonmanager.kubernetes.io/mode: Reconcile # Addon 管理器模式
subjects:
  - kind: ServiceAccount # 授权对象类型：ServiceAccount
    name: elasticsearch-logging # ServiceAccount 的名称
    namespace: kube-logging # ServiceAccount 所在的命名空间
    apiGroup: "" # API 组
roleRef:
  kind: ClusterRole # 引用的 ClusterRole 类型
  name: elasticsearch-logging # 引用的 ClusterRole 名称
  apiGroup: rbac.authorization.k8s.io # API 组
---
# Elasticsearch deployment itself （Elasticsearch 部署本身）
apiVersion: apps/v1 # API 版本
kind: StatefulSet # 使用 StatefulSet 创建 Pod（有状态副本集）
metadata:
  name: elasticsearch-logging # Pod 名称，StatefulSet 创建的 Pod 有序号和顺序
  namespace: kube-logging # 命名空间
  labels:
    k8s-app: elasticsearch-logging # 标签
    kubernetes.io/cluster-service: "true" # Kubernetes 集群服务标签
    addonmanager.kubernetes.io/mode: Reconcile # Addon 管理器模式
    srv: srv-elasticsearch # 自定义标签
spec:
  serviceName: elasticsearch-logging # 与 Service 相关联，确保使用以下 DNS 地址访问 StatefulSet 中的每个 Pod (es-cluster-[0,1,2].elasticsearch.elk.svc.cluster.local)
  replicas: 1 # 副本数量，单节点
  selector:
    matchLabels:
      k8s-app: elasticsearch-logging # 和 Pod 模板配置的 labels 相匹配
  template:
    metadata:
      labels:
        k8s-app: elasticsearch-logging # 标签
        kubernetes.io/cluster-service: "true" # Kubernetes 集群服务标签
    spec:
      serviceAccountName: elasticsearch-logging # 使用 ServiceAccount
      containers:
        - image: docker.io/library/elasticsearch:7.9.3 # 使用的 Elasticsearch 镜像
          name: elasticsearch-logging # 容器名称
          resources:
            limits:
              cpu: 1000m # CPU 限制
              memory: 2Gi # 内存限制
            requests:
              cpu: 100m # CPU 请求
              memory: 500Mi # 内存请求
          ports:
            - containerPort: 9200 # 容器端口
              name: db # 端口名称
              protocol: TCP # 协议
            - containerPort: 9300 # 容器端口
              name: transport # 端口名称
              protocol: TCP # 协议
          volumeMounts:
            - name: elasticsearch-logging # 卷挂载名称
              mountPath: /usr/share/elasticsearch/data/ # 挂载路径
          env:
            - name: "NAMESPACE" # 环境变量：命名空间
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace # 从 Pod 元数据中获取命名空间
            - name: "discovery.type" # 环境变量：定义单节点类型
              value: "single-node" # 单节点
            - name: ES_JAVA_OPTS # 环境变量：设置 Java 内存参数
              value: "-Xms512m -Xmx2g" # 内存参数
      volumes:
        - name: elasticsearch-logging # 卷名称
          hostPath:
            path: /data/es/ # 主机路径，将数据存储在主机节点上
      nodeSelector: # 节点选择器，如果需要匹配落盘节点可以添加 nodeSelect
        es: data # 选择具有 es: data 标签的节点
      tolerations:
        - effect: NoSchedule # 污点容忍
          operator: Exists # 存在即可
      initContainers: # 初始化容器，在主容器启动前运行
        - name: elasticsearch-logging-init # 初始化容器名称
          image: alpine:3.6 # 使用的镜像
          command: [ "/sbin/sysctl", "-w", "vm.max_map_count=262144" ] # 设置内核参数
          securityContext:
            privileged: true # 运行特权容器
        - name: increase-fd-ulimit # 初始化容器名称
          image: busybox # 使用的镜像
          imagePullPolicy: IfNotPresent # 如果本地不存在镜像，则拉取
          command: [ "sh", "-c", "ulimit -n 65536" ] # 修改文件描述符最大数量
          securityContext:
            privileged: true # 运行特权容器
        - name: elasticsearch-volume-init # 初始化容器名称，es 数据落盘初始化，加上 777 权限
          image: alpine:3.6 # 使用的镜像
          command:
            - chmod
            - -R
            - "777"
            - /usr/share/elasticsearch/data/ # 修改权限
          volumeMounts:
            - name: elasticsearch-logging # 卷挂载名称
              mountPath: /usr/share/elasticsearch/data/ # 挂载路径
```

创建`namespace.yaml`
```yml
apiVersion: v1
kind: Namespace
metadata:
  name: kube-logging
```
执行命令
```shell
# 需要提前给 es 落盘节点打上标签 
kubectl label node k8s-node1 es=data
# 创建命名空间
kubectl create -f namespace.yaml
# 创建服务
kubectl create -f es.yaml
# 查看 pod 启用情况
kubectl get pod -n kube-logging
```
### 部署 logstash 数据清洗
创建 `logstash.yaml`
```shell
---
apiVersion: v1
kind: Service # 定义 Kubernetes Service 资源
metadata:
  name: logstash # Service 名称
  namespace: kube-logging # Service 所在的命名空间
spec:
  ports:
    - port: 5044 # Service 暴露的端口
      targetPort: beats # Pod 上的目标端口名称，对应容器端口名称
  selector:
    type: logstash # 通过标签选择 Pod
  clusterIP: None # 使用 Headless Service，不分配 ClusterIP
---
apiVersion: apps/v1
kind: Deployment # 定义 Kubernetes Deployment 资源
metadata:
  name: logstash # Deployment 名称
  namespace: kube-logging # Deployment 所在的命名空间
spec:
  selector:
    matchLabels:
      type: logstash # 通过标签选择 Pod
  template:
    metadata:
      labels:
        type: logstash # Pod 标签
        srv: srv-logstash # Pod 标签
    spec:
      containers:
        - image: docker.io/kubeimages/logstash:7.9.3 # 使用的 Logstash 镜像，支持 arm64 和 amd64 架构
          name: logstash # 容器名称
          ports:
            - containerPort: 5044 # 容器端口
              name: beats # 端口名称
          command:
            - logstash # 容器启动命令
            - '-f' # 指定配置文件
            - '/etc/logstash_c/logstash.conf' # 配置文件路径
          env:
            - name: "XPACK_MONITORING_ELASTICSEARCH_HOSTS" # 环境变量，指定 Elasticsearch 主机
              value: "http://elasticsearch-logging:9200" # Elasticsearch Service 地址
          volumeMounts:
            - name: config-volume # 挂载 ConfigMap 卷，用于 logstash.conf 配置文件
              mountPath: /etc/logstash_c/ # 挂载路径
            - name: config-yml-volume # 挂载 ConfigMap 卷，用于 logstash.yml 配置文件
              mountPath: /usr/share/logstash/config/ # 挂载路径
            - name: timezone # 挂载主机时区文件，保证 Logstash 使用正确时区
              mountPath: /etc/localtime # 挂载路径
          resources: # 资源限制和请求，避免资源抢占
            limits:
              cpu: 1000m # CPU 限制
              memory: 2048Mi # 内存限制
            requests:
              cpu: 512m # CPU 请求
              memory: 512Mi # 内存请求
      volumes:
        - name: config-volume # ConfigMap 卷，用于 logstash.conf 配置文件
          configMap:
            name: logstash-conf # ConfigMap 名称
            items:
              - key: logstash.conf # ConfigMap 中的键
                path: logstash.conf # 挂载到容器内的路径
        - name: timezone # 主机时区卷
          hostPath:
            path: /etc/localtime # 主机路径
        - name: config-yml-volume # ConfigMap 卷，用于 logstash.yml 配置文件
          configMap:
            name: logstash-yml # ConfigMap 名称
            items:
              - key: logstash.yml # ConfigMap 中的键
                path: logstash.yml # 挂载到容器内的路径
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: logstash-conf
  namespace: kube-logging
  labels:
    type: logstash
data:
  logstash.conf: |-
    input {
      beats { 
        port => 5044 
      } 
    } 
    filter {
      # 处理 ingress 日志 
      if [kubernetes][container][name] == "nginx-ingress-controller" {
        json {
          source => "message" 
          target => "ingress_log" 
        }
        if [ingress_log][requesttime] { 
          mutate { 
            convert => ["[ingress_log][requesttime]", "float"] 
          }
        }
        if [ingress_log][upstremtime] { 
          mutate { 
            convert => ["[ingress_log][upstremtime]", "float"] 
          }
        } 
        if [ingress_log][status] { 
          mutate { 
            convert => ["[ingress_log][status]", "float"] 
          }
        }
        if  [ingress_log][httphost] and [ingress_log][uri] {
          mutate { 
            add_field => {"[ingress_log][entry]" => "%{[ingress_log][httphost]}%{[ingress_log][uri]}"} 
          } 
          mutate { 
            split => ["[ingress_log][entry]","/"] 
          } 
          if [ingress_log][entry][1] { 
            mutate { 
              add_field => {"[ingress_log][entrypoint]" => "%{[ingress_log][entry][0]}/%{[ingress_log][entry][1]}"} 
              remove_field => "[ingress_log][entry]" 
            }
          } else { 
            mutate { 
              add_field => {"[ingress_log][entrypoint]" => "%{[ingress_log][entry][0]}/"} 
              remove_field => "[ingress_log][entry]" 
            }
          }
        }
      }
      # 处理以srv进行开头的业务服务日志 
      if [kubernetes][container][name] =~ /^srv*/ { 
        json { 
          source => "message" 
          target => "tmp" 
        } 
        if [kubernetes][namespace] == "kube-logging" { 
          drop{} 
        } 
        if [tmp][level] { 
          mutate{ 
            add_field => {"[applog][level]" => "%{[tmp][level]}"} 
          } 
          if [applog][level] == "debug"{ 
            drop{} 
          } 
        } 
        if [tmp][msg] { 
          mutate { 
            add_field => {"[applog][msg]" => "%{[tmp][msg]}"} 
          } 
        } 
        if [tmp][func] { 
          mutate { 
            add_field => {"[applog][func]" => "%{[tmp][func]}"} 
          } 
        } 
        if [tmp][cost]{ 
          if "ms" in [tmp][cost] { 
            mutate { 
              split => ["[tmp][cost]","m"] 
              add_field => {"[applog][cost]" => "%{[tmp][cost][0]}"} 
              convert => ["[applog][cost]", "float"] 
            } 
          } else { 
            mutate { 
              add_field => {"[applog][cost]" => "%{[tmp][cost]}"} 
            }
          }
        }
        if [tmp][method] { 
          mutate { 
            add_field => {"[applog][method]" => "%{[tmp][method]}"} 
          }
        }
        if [tmp][request_url] { 
          mutate { 
            add_field => {"[applog][request_url]" => "%{[tmp][request_url]}"} 
          } 
        }
        if [tmp][meta._id] { 
          mutate { 
            add_field => {"[applog][traceId]" => "%{[tmp][meta._id]}"} 
          } 
        } 
        if [tmp][project] { 
          mutate { 
            add_field => {"[applog][project]" => "%{[tmp][project]}"} 
          }
        }
        if [tmp][time] { 
          mutate { 
            add_field => {"[applog][time]" => "%{[tmp][time]}"} 
          }
        }
        if [tmp][status] { 
          mutate { 
            add_field => {"[applog][status]" => "%{[tmp][status]}"} 
            convert => ["[applog][status]", "float"] 
          }
        }
      }
      mutate { 
        rename => ["kubernetes", "k8s"] 
        remove_field => "beat" 
        remove_field => "tmp" 
        remove_field => "[k8s][labels][app]" 
      }
    }
    output { 
      elasticsearch { 
        hosts => ["http://elasticsearch-logging:9200"] 
        codec => json 
        index => "logstash-%{+YYYY.MM.dd}" #索引名称以logstash+日志进行每日新建 
      }
    }
---
apiVersion: v1
kind: ConfigMap # 定义 Kubernetes ConfigMap 资源
metadata:
  name: logstash-yml # ConfigMap 名称
  namespace: kube-logging # ConfigMap 所在的命名空间
  labels:
    type: logstash # ConfigMap 标签
data:
  logstash.yml: |- # Logstash 配置文件内容
    http.host: "0.0.0.0" # 设置 Logstash HTTP API 监听的 IP 地址，0.0.0.0 表示监听所有网络接口
    xpack.monitoring.elasticsearch.hosts: http://elasticsearch-logging:9200 # 设置 Logstash 监控数据发送到的 Elasticsearch 主机地址，指向 Elasticsearch Service
```
运行命令
```shell
# 创建
kubectl apply -f logstash.yaml
```
### 部署 filebeat 数据采集
创建 `filebeat.yaml`
```shell
---
apiVersion: v1
kind: ConfigMap # 定义 ConfigMap 资源，用于存储 Filebeat 配置文件
metadata:
  name: filebeat-config # ConfigMap 名称：filebeat-config
  namespace: kube-logging # 命名空间：kube-logging
  labels:
    k8s-app: filebeat # 标签：k8s-app=filebeat
data:
  filebeat.yml: |- # Filebeat 配置文件内容
    filebeat.inputs:
      - type: container # 输入类型：container（容器日志）
        enable: true # 启用该输入
        paths:
          - /var/log/containers/*.log # Filebeat 采集挂载到 Pod 中的日志目录
        processors:
          - add_kubernetes_metadata: # 添加 Kubernetes 元数据，用于后续数据清洗
              host: ${NODE_NAME} # 主机名，从环境变量 NODE_NAME 获取
              matchers:
                - logs_path:
                    logs_path: "/var/log/containers/" # 匹配日志路径
    #output.kafka:  # 如果日志量较大，ES 中的日志有延迟，可以选择在 Filebeat 和 Logstash 中间加入 Kafka
    #  hosts: ["kafka-log-01:9092", "kafka-log-02:9092", "kafka-log-03:9092"] 
    # topic: 'topic-test-log' 
    #  version: 2.0.0 
    output.logstash: # 因为还需要部署 Logstash 进行数据清洗，因此 Filebeat 将数据推送到 Logstash
      hosts: [ "logstash:5044" ] # Logstash 主机地址和端口
      enabled: true # 启用 Logstash 输出
---
apiVersion: v1
kind: ServiceAccount # 定义 ServiceAccount 资源，用于 Filebeat Pod 的身份验证
metadata:
  name: filebeat # ServiceAccount 名称：filebeat
  namespace: kube-logging # 命名空间：kube-logging
  labels:
    k8s-app: filebeat # 标签：k8s-app=filebeat
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole # 定义 ClusterRole 资源，授予 Filebeat Pod 必要权限
metadata:
  name: filebeat # ClusterRole 名称：filebeat
  labels:
    k8s-app: filebeat # 标签：k8s-app=filebeat
rules:
  - apiGroups: [ "" ] # 核心 API 组
    resources:
      - namespaces # 资源类型：namespaces
      - pods # 资源类型：pods
    verbs: [ "get", "watch", "list" ] # 允许的操作：get, watch, list
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding # 定义 ClusterRoleBinding 资源，将 ClusterRole 绑定到 ServiceAccount
metadata:
  name: filebeat # ClusterRoleBinding 名称：filebeat
subjects:
  - kind: ServiceAccount # 授权对象类型：ServiceAccount
    name: filebeat # ServiceAccount 名称：filebeat
    namespace: kube-logging # 命名空间：kube-logging
roleRef:
  kind: ClusterRole # 引用的 ClusterRole 类型
  name: filebeat # 引用的 ClusterRole 名称：filebeat
  apiGroup: rbac.authorization.k8s.io # API 组：rbac.authorization.k8s.io
---
apiVersion: apps/v1
kind: DaemonSet # 定义 DaemonSet 资源，确保每个节点运行一个 Filebeat Pod
metadata:
  name: filebeat # DaemonSet 名称：filebeat
  namespace: kube-logging # 命名空间：kube-logging
  labels:
    k8s-app: filebeat # 标签：k8s-app=filebeat
spec:
  selector:
    matchLabels:
      k8s-app: filebeat # 选择具有 k8s-app=filebeat 标签的 Pod
  template:
    metadata:
      labels:
        k8s-app: filebeat # Pod 标签：k8s-app=filebeat
    spec:
      serviceAccountName: filebeat # 使用 ServiceAccount：filebeat
      terminationGracePeriodSeconds: 30 # Pod 终止宽限期：30 秒
      containers:
        - name: filebeat # 容器名称：filebeat
          image: docker.io/kubeimages/filebeat:7.9.3 # 使用的 Filebeat 镜像，支持 arm64 和 amd64 架构
          args: [
            "-c", "/etc/filebeat.yml", # 指定 Filebeat 配置文件路径
            "-e","-httpprof","0.0.0.0:6060" # 启用 Filebeat HTTP profiler
          ]
          #ports:
          #  - containerPort: 6060
          #    hostPort: 6068
          env:
            - name: NODE_NAME # 环境变量：节点名称
              valueFrom:
                fieldRef:
                  fieldPath: spec.nodeName # 从 Pod 规范中获取节点名称
            - name: ELASTICSEARCH_HOST # 环境变量：Elasticsearch 主机
              value: elasticsearch-logging # Elasticsearch Service 名称
            - name: ELASTICSEARCH_PORT # 环境变量：Elasticsearch 端口
              value: "9200" # Elasticsearch 端口
          securityContext:
            runAsUser: 0 # 以 root 用户运行 Filebeat
            # 如果使用 Red Hat OpenShift，取消注释以下行：
            #privileged: true
          resources:
            limits:
              memory: 1000Mi # 内存限制：1000Mi
              cpu: 1000m # CPU 限制：1000m
            requests:
              memory: 100Mi # 内存请求：100Mi
              cpu: 100m # CPU 请求：100m
          volumeMounts:
            - name: config # 挂载 Filebeat 配置文件
              mountPath: /etc/filebeat.yml # 挂载路径
              readOnly: true # 只读挂载
              subPath: filebeat.yml # 子路径：filebeat.yml
            - name: data # 持久化 Filebeat 数据到宿主机
              mountPath: /usr/share/filebeat/data # 挂载路径
            - name: varlibdockercontainers # 挂载宿主机上的源日志目录到 Filebeat 容器中，如果未修改 Docker 或 Containerd 的运行时标准日志落盘路径，可以将 mountPath 改为 /var/lib
              mountPath: /var/lib # 挂载路径
              readOnly: true # 只读挂载
            - name: varlog # 挂载宿主机上的 /var/log/pods 和 /var/log/containers 的软链接到 Filebeat 容器中
              mountPath: /var/log/ # 挂载路径
              readOnly: true # 只读挂载
            - name: timezone #挂载时区文件
              mountPath: /etc/localtime #挂载路径
      volumes:
        - name: config # Filebeat 配置文件卷
          configMap:
            defaultMode: 0600 # 默认权限：0600
            name: filebeat-config # ConfigMap 名称：filebeat-config
        - name: varlibdockercontainers # 宿主机上的源日志目录卷，如果未修改 Docker 或 Containerd 的运行时标准日志落盘路径，可以将 path 改为 /var/lib
          hostPath:
            path: /var/lib # 宿主机路径
        - name: varlog # 宿主机上的 /var/log/ 目录卷
          hostPath:
            path: /var/log/ # 宿主机路径
        # data folder stores a registry of read status for all files, so we don't send everything again on a Filebeat pod restart
        - name: inputs #输入文件
          configMap:
            defaultMode: 0600
            name: filebeat-inputs
        - name: data # Filebeat 数据卷，用于存储已读取文件的注册信息，避免 Filebeat Pod 重启时重复发送日志
          hostPath:
            path: /data/filebeat-data # 宿主机路径
            type: DirectoryOrCreate # 如果目录不存在，则创建
        - name: timezone #时区文件
          hostPath:
            path: /etc/localtime
      tolerations: # 加入容忍，能够调度到每一个节点
        - effect: NoExecute # 污点效果：NoExecute
          key: dedicated # 污点键：dedicated
          operator: Equal # 操作符：Equal
          value: gpu # 污点值：gpu
        - effect: NoSchedule # 污点效果：NoSchedule
          operator: Exists # 操作符：Exists
```
运行命令
```shell
# 创建
kubectl apply -f filebeat.yaml
```
### 部署 kibana 可视化界面
创建`kibana.yaml`
此处有配置 kibana 访问域名，如果没有域名则需要在本机配置 hosts
`10.0.0.102 kibana.wolfcode.cn`
```shell
---
apiVersion: v1
kind: ConfigMap # 定义 ConfigMap 资源，用于存储 Kibana 配置文件
metadata:
  namespace: kube-logging # 命名空间：kube-logging
  name: kibana-config # ConfigMap 名称：kibana-config
  labels:
    k8s-app: kibana # 标签：k8s-app=kibana
data:
  kibana.yml: |- # Kibana 配置文件内容
    server.name: kibana # Kibana 服务器名称：kibana
    server.host: "0" # Kibana 服务器监听地址：0（监听所有接口）
    i18n.locale: zh-CN # 设置默认语言为中文
    elasticsearch:
      hosts: ${ELASTICSEARCH_HOSTS} # Elasticsearch 集群连接地址，由于都在 Kubernetes 部署且在同一命名空间下，可以直接使用 Service 名称连接
---
apiVersion: v1
kind: Service # 定义 Service 资源，用于暴露 Kibana 服务
metadata:
  name: kibana # Service 名称：kibana
  namespace: kube-logging # 命名空间：kube-logging
  labels:
    k8s-app: kibana # 标签：k8s-app=kibana
    kubernetes.io/cluster-service: "true" # Kubernetes 集群服务标签
    addonmanager.kubernetes.io/mode: Reconcile # Addon 管理器模式
    kubernetes.io/name: "Kibana" # Kubernetes 名称标签
    srv: srv-kibana # 自定义标签：srv-kibana
spec:
  type: NodePort # Service 类型：NodePort，通过节点端口暴露服务
  ports:
    - port: 5601 # Service 端口：5601
      protocol: TCP # 协议：TCP
      targetPort: ui # Pod 目标端口：ui（对应容器端口名称）
  selector:
    k8s-app: kibana # 选择具有 k8s-app=kibana 标签的 Pod
---
apiVersion: apps/v1
kind: Deployment # 定义 Deployment 资源，用于部署 Kibana Pod
metadata:
  name: kibana # Deployment 名称：kibana
  namespace: kube-logging # 命名空间：kube-logging
  labels:
    k8s-app: kibana # 标签：k8s-app=kibana
    kubernetes.io/cluster-service: "true" # Kubernetes 集群服务标签
    addonmanager.kubernetes.io/mode: Reconcile # Addon 管理器模式
    srv: srv-kibana # 自定义标签：srv-kibana
spec:
  replicas: 1 # 副本数量：1
  selector:
    matchLabels:
      k8s-app: kibana # 选择具有 k8s-app=kibana 标签的 Pod
  template:
    metadata:
      labels:
        k8s-app: kibana # Pod 标签：k8s-app=kibana
    spec:
      containers:
        - name: kibana # 容器名称：kibana
          image: docker.io/kubeimages/kibana:7.9.3 # 使用的 Kibana 镜像，支持 arm64 和 amd64 架构
          resources:
            # 初始化时需要更多 CPU，因此使用 burstable 类
            limits:
              cpu: 1000m # CPU 限制：1000m
            requests:
              cpu: 100m # CPU 请求：100m
          env:
            - name: ELASTICSEARCH_HOSTS # 环境变量：Elasticsearch 主机地址
              value: http://elasticsearch-logging:9200 # Elasticsearch Service 地址
          ports:
            - containerPort: 5601 # 容器端口：5601
              name: ui # 端口名称：ui
              protocol: TCP # 协议：TCP
          volumeMounts:
            - name: config # 挂载 Kibana 配置文件
              mountPath: /usr/share/kibana/config/kibana.yml # 挂载路径
              readOnly: true # 只读挂载
              subPath: kibana.yml # 子路径：kibana.yml
      volumes:
        - name: config # Kibana 配置文件卷
          configMap:
            name: kibana-config # ConfigMap 名称：kibana-config
---
apiVersion: networking.k8s.io/v1
kind: Ingress # 定义 Ingress 资源，用于通过域名访问 Kibana
metadata:
  name: kibana # Ingress 名称：kibana
  namespace: kube-logging # 命名空间：kube-logging
spec:
  ingressClassName: nginx # 使用的 Ingress 控制器：nginx
  rules:
    - host: kibana.wolfcode.cn # 域名：kibana.wolfcode.cn
      http:
        paths:
          - path: / # 路径：/
            pathType: Prefix # 路径类型：Prefix（前缀匹配）
            backend:
              service:
                name: kibana # 后端 Service 名称：kibana
                port:
                  number: 5601 # 后端 Service 端口：5601
```
运行命令
```shell
# 创建
kubectl apply -f kibana.yaml
# 查看svc
kubectl get svc -n kube-logging -o wide
NAME                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE   SELECTOR
elasticsearch-logging   ClusterIP   10.106.218.50   <none>        9200/TCP         71m   k8s-app=elasticsearch-logging
kibana                  NodePort    10.110.253.35   <none>        5601:31068/TCP   25m   k8s-app=kibana
logstash                ClusterIP   None            <none>        5044/TCP         36m   type=logstash
```
### Kibana 配置
#### 避免日志无限增长并占用过多磁盘空间
在 Kibana 中设置索引生命周期管理（ILM）策略
**1. 导航到 Stack Management（堆栈管理）**
- 打开 Kibana 界面。
- 在左侧导航栏中，找到并点击 "Stack Management"（堆栈管理）图标（通常是一个齿轮或设置图标）。
**2. 进入 Index Lifecycle Policies（索引生命周期策略）**
- 在 Stack Management 页面中，找到 "Data"（数据）部分。
- 点击 "Index Lifecycle Policies"（索引生命周期策略）。
**3. 创建新的 ILM 策略**
- 在 Index Lifecycle Policies 页面中，点击 "Create policy"（创建策略）按钮。
- 在弹出的对话框中，输入策略名称 "logstash-history-ilm-policy"。
**4. 配置策略阶段**
- **热阶段（Hot Phase）：**
    - 默认情况下，热阶段是启用的。
    - 由于您需要关闭热阶段，所以直接跳过。
- **删除阶段（Delete Phase）：**
    - 找到 "Delete phase"（删除阶段），并将其切换为启用状态。
    - 在 "Minimum age"（最小保留时间）字段中，输入 "7d"（7 天），表示日志将保留 7 天后被删除。
**5. 保存策略**
- 检查您的配置，确保策略名称和保留时间设置正确。
- 点击页面底部的 "Save policy"（保存策略）按钮。
**6. 应用 ILM 策略到索引模板**
- 找到索引模板，并且编辑。
- 在索引模板的生命周期策略中，选择刚刚创建好的生命周期策略。
#### 方便在 discover 中查看日志
1. 打开Kibana 界面->Stack Management 可以看到采集到的日志
2. 打开 索引模式->创建索引模式 按钮
3. 索引模式名称 里面配置 logstash-* ，点击下一步
4. 时间字段 选择 @timestamp，点击 创建索引模式 按钮

由于部署的单节点，产生副本后索引状态会变成 yellow，打开 dev tools，取消所有索引的副本数
```http
PUT _all/_settings
{
    "number_of_replicas": 0
}
```

为了标准化日志中的 map 类型，以及解决链接索引生命周期策略，我们需要修改默认模板
```http
PUT _template/logstash 
{ 
    "order": 1, 
    "index_patterns": [ 
      "logstash-*" 
    ], 
    "settings": { 
      "index": { 
      "lifecycle" : { 
          "name" : "logstash-history-ilm-policy" 
        }, 
        "number_of_shards": "2", 
        "refresh_interval": "5s", 
        "number_of_replicas" : "0" 
      } 
    }, 
    "mappings": { 
        "properties": { 
          "@timestamp": { 
            "type": "date" 
          }, 
          "applog": { 
            "dynamic": true, 
            "properties": { 
              "cost": { 
                "type": "float" 
              }, 
              "func": { 
                "type": "keyword" 
              }, 
              "method": { 
                "type": "keyword" 
              } 
            } 
          }, 
          "k8s": { 
            "dynamic": true, 
            "properties": { 
              "namespace": { 
                "type": "keyword" 
              }, 
              "container": { 
                "dynamic": true, 
                "properties": { 
                  "name": { 
                    "type": "keyword" 
                  } 
                } 
              }, 
              "labels": { 
                "dynamic": true, 
                "properties": { 
                  "srv": { 
                    "type": "keyword" 
                  } 
                } 
              } 
            } 
          }, 
          "geoip": { 
            "dynamic": true, 
            "properties": { 
              "ip": { 
                "type": "ip" 
              }, 
              "latitude": { 
                "type": "float" 
              }, 
              "location": { 
                "type": "geo_point" 
              }, 
              "longitude": { 
                "type": "float" 
              } 
            } 
          } 
      } 
    }, 
    "aliases": {} 
  } 
```
最后即可通过 discover 进行搜索了