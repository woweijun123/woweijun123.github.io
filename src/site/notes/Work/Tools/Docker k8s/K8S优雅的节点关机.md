---
{"dg-publish":true,"permalink":"/Work/Tools/Docker k8s/K8S优雅的节点关机/","title":"K8S优雅的节点关机","tags":["flashcards"],"noteIcon":"","created":"2025-02-25T23:17:27.000+08:00","updated":"2026-03-16T22:51:09.038+08:00"}
---

在Kubernetes（K8S）中，进行优雅的节点关机维护是确保集群稳定性和可用性的关键步骤。以下是一个详细的指南，涵盖了从准备到恢复的全过程：
##### 1. 准备阶段
1.   **选择合适的时间** ：选择一个低负载时段进行维护，以减少对用户的影响。
2.   **通知团队** ：提前通知相关团队和用户，确保他们了解维护时间和可能的影响。
3.   **备份数据** ：在维护前确保所有重要数据都已备份，以防万一出现数据丢失或损坏。
##### 2. 标记节点为不可调度
1.  使用kubectl cordon命令：将节点标记为不可调度（cordon），以防新的Pod被调度到该节点。
```shell
kubectl cordon k8s-node1 k8s-node2
```
##### 3. 驱逐正在运行的Pod
1.  使用kubectl drain命令：驱逐节点上的Pod，这样可以安全地迁移工作负载。
```shell
kubectl drain k8s-node1 k8s-node2 --ignore-daemonsets --delete-local-data
```
-  `--ignore-daemonsets` ：忽略DaemonSet管理的Pod，因为DaemonSet会在每个节点上运行一个Pod，驱逐它们没有意义。
-  `--delete-local-data` ：允许删除具有本地数据的Pod，这通常用于StatefulSet管理的Pod，但请确保这些Pod的数据已经得到了妥善处理。
2.  检查Pods迁移状态：确保所有Pod都已经成功迁移到其他节点上。
```shell
kubectl get pods --all-namespaces -o wide
```
##### 四、停止Docker守护进程（可选）
1.  停止Docker服务：在节点上停止Docker守护进程（如果使用的是Docker作为容器运行时）。
```shell
ssh <node-ip> "sudo systemctl stop docker"
```

这一步是可选的，因为kubectl drain命令已经确保了Pod的迁移，停止Docker服务只是为了确保节点在关机时不会有新的容器被创建或运行。
##### 五、执行节点关机操作
1.  关机命令：根据所使用的操作系统和硬件，执行相应的关机命令来关闭节点。
```shell
ssh <node-ip> "sudo shutdown -h now"
```
或者
```shell
ssh <node-ip> "sudo poweroff"
```
##### 六、维护操作
1.   **硬件检查** ：检查节点的硬件状态，确保没有故障或性能瓶颈。
2.   **操作系统更新** ：确保节点的操作系统和软件包是最新的，应用必要的安全补丁。
3.   **Kubernetes组件升级** ：在需要时，升级kubelet和kube-proxy组件，以保持Kubernetes的最新版本。
##### 七、重新启动节点
1.   **启动节点** ：完成维护操作后，重新启动节点。
2.  检查节点状态：确保节点已经成功启动并连接到Kubernetes集群。
```shell
kubectl get nodes
```
##### 八、重新启用节点
1.  使用kubectl uncordon命令：将节点标记为可调度（uncordon），使其能够接收新的Pod。
```shell
kubectl uncordon <node-name>
```
##### 九、验证Pod状态
1.  检查Pod状态：确保Pod正常运行并且没有出现问题。
```shell
kubectl get pods --all-namespaces -o wide
```
##### 十、记录与更新
1.   **记录维护过程** ：详细记录维护步骤和遇到的问题，以便将来参考。
2.   **更新文档** ：确保系统文档和操作手册更新至最新状态，包含任何新的配置或变更。