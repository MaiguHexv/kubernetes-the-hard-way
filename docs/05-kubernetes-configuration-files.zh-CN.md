# 生成用于认证的 Kubernetes 配置文件

在本实验中，你将生成 [Kubernetes 客户端配置文件](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/)，通常称为 kubeconfig，用于配置 Kubernetes 客户端连接到 Kubernetes API Server 并完成身份认证。

## 客户端认证配置

在本节中，你将为 `kubelet` 和 `admin` 用户生成 kubeconfig 文件。

### kubelet 的 Kubernetes 配置文件

在为 Kubelet 生成 kubeconfig 文件时，必须使用与该 Kubelet 节点名称相匹配的客户端证书。这样才能确保 Kubelet 被 Kubernetes [Node Authorizer](https://kubernetes.io/docs/reference/access-authn-authz/node/) 正确授权。

> 以下命令必须在 [生成 TLS 证书](04-certificate-authority.md) 实验中生成 SSL 证书时所在的同一目录下运行。

为 `node-0` 和 `node-1` 工作节点生成 kubeconfig 文件：

```bash
for host in node-0 node-1; do
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://server.kubernetes.local:6443 \
    --kubeconfig=${host}.kubeconfig

  kubectl config set-credentials system:node:${host} \
    --client-certificate=${host}.crt \
    --client-key=${host}.key \
    --embed-certs=true \
    --kubeconfig=${host}.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:node:${host} \
    --kubeconfig=${host}.kubeconfig

  kubectl config use-context default \
    --kubeconfig=${host}.kubeconfig
done
```

输出结果：

```text
node-0.kubeconfig
node-1.kubeconfig
```

### kube-proxy 的 Kubernetes 配置文件

为 `kube-proxy` 服务生成 kubeconfig 文件：

```bash
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://server.kubernetes.local:6443 \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config set-credentials system:kube-proxy \
    --client-certificate=kube-proxy.crt \
    --client-key=kube-proxy.key \
    --embed-certs=true \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-proxy \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config use-context default \
    --kubeconfig=kube-proxy.kubeconfig
}
```

输出结果：

```text
kube-proxy.kubeconfig
```

### kube-controller-manager 的 Kubernetes 配置文件

为 `kube-controller-manager` 服务生成 kubeconfig 文件：

```bash
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://server.kubernetes.local:6443 \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config set-credentials system:kube-controller-manager \
    --client-certificate=kube-controller-manager.crt \
    --client-key=kube-controller-manager.key \
    --embed-certs=true \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-controller-manager \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config use-context default \
    --kubeconfig=kube-controller-manager.kubeconfig
}
```

输出结果：

```text
kube-controller-manager.kubeconfig
```


### kube-scheduler 的 Kubernetes 配置文件

为 `kube-scheduler` 服务生成 kubeconfig 文件：

```bash
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://server.kubernetes.local:6443 \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config set-credentials system:kube-scheduler \
    --client-certificate=kube-scheduler.crt \
    --client-key=kube-scheduler.key \
    --embed-certs=true \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-scheduler \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config use-context default \
    --kubeconfig=kube-scheduler.kubeconfig
}
```

输出结果：

```text
kube-scheduler.kubeconfig
```

### admin 的 Kubernetes 配置文件

为 `admin` 用户生成 kubeconfig 文件：

```bash
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=admin.kubeconfig

  kubectl config set-credentials admin \
    --client-certificate=admin.crt \
    --client-key=admin.key \
    --embed-certs=true \
    --kubeconfig=admin.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=admin \
    --kubeconfig=admin.kubeconfig

  kubectl config use-context default \
    --kubeconfig=admin.kubeconfig
}
```

输出结果：

```text
admin.kubeconfig
```

## 分发 Kubernetes 配置文件

将 `kubelet` 和 `kube-proxy` 的 kubeconfig 文件复制到 `node-0` 和 `node-1` 机器上：

```bash
for host in node-0 node-1; do
  ssh root@${host} "mkdir -p /var/lib/{kube-proxy,kubelet}"

  scp kube-proxy.kubeconfig \
    root@${host}:/var/lib/kube-proxy/kubeconfig

  scp ${host}.kubeconfig \
    root@${host}:/var/lib/kubelet/kubeconfig
done
```

将 `kube-controller-manager` 和 `kube-scheduler` 的 kubeconfig 文件复制到 `server` 机器上：

```bash
scp admin.kubeconfig \
  kube-controller-manager.kubeconfig \
  kube-scheduler.kubeconfig \
  root@server:~/
```

下一节：[生成数据加密配置和密钥](06-data-encryption-keys.zh-CN.md)
