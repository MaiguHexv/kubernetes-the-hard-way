# 配置 kubectl 远程访问

本实验将基于 `admin` 用户凭据，为 `kubectl` 命令行工具生成一个 kubeconfig 文件。

> 请在 `jumpbox` 机器上运行本实验中的命令。

## Admin 的 Kubernetes 配置文件

每个 kubeconfig 都需要连接一个 Kubernetes API Server。

根据之前实验中配置的 `/etc/hosts` DNS 记录，你应该能够 ping 通 `server.kubernetes.local`。

```bash
curl --cacert ca.crt \
  https://server.kubernetes.local:6443/version
```

```text
{
  "major": "1",
  "minor": "32",
  "gitVersion": "v1.32.3",
  "gitCommit": "32cc146f75aad04beaaa245a7157eb35063a9f99",
  "gitTreeState": "clean",
  "buildDate": "2025-03-11T19:52:21Z",
  "goVersion": "go1.23.6",
  "compiler": "gc",
  "platform": "linux/arm64"
}
```

生成一个用于以 `admin` 用户身份进行认证的 kubeconfig 文件：

```bash
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://server.kubernetes.local:6443

  kubectl config set-credentials admin \
    --client-certificate=admin.crt \
    --client-key=admin.key

  kubectl config set-context kubernetes-the-hard-way \
    --cluster=kubernetes-the-hard-way \
    --user=admin

  kubectl config use-context kubernetes-the-hard-way
}
```
执行上述命令后，会在 `kubectl` 命令行工具默认使用的位置 `~/.kube/config` 创建 kubeconfig 文件。这也意味着你无需指定配置文件即可运行 `kubectl` 命令。


## 验证

查看远程 Kubernetes 集群的版本：

```bash
kubectl version
```

```text
Client Version: v1.32.3
Kustomize Version: v5.5.0
Server Version: v1.32.3
```

列出远程 Kubernetes 集群中的节点：

```bash
kubectl get nodes
```

```
NAME     STATUS   ROLES    AGE    VERSION
node-0   Ready    <none>   10m   v1.32.3
node-1   Ready    <none>   10m   v1.32.3
```

下一节：[配置 Pod 网络路由](11-pod-network-routes.md)
