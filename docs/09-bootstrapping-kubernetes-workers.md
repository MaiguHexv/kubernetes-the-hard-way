# Kubernetes 工作节点

本实验将引导两个 Kubernetes 工作节点。以下组件会被安装：[runc](https://github.com/opencontainers/runc)、[容器网络插件](https://github.com/containernetworking/cni)、[containerd](https://github.com/containerd/containerd)、[kubelet](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet) 和 [kube-proxy](https://kubernetes.io/docs/concepts/cluster-administration/proxies)。

## 准备工作

本节的命令必须在 `jumpbox` 上运行。

将 Kubernetes 二进制文件和 systemd unit 文件复制到每个工作节点实例：

```bash
for HOST in node-0 node-1; do
  SUBNET=$(grep ${HOST} machines.txt | cut -d " " -f 4)
  sed "s|SUBNET|$SUBNET|g" \
    configs/10-bridge.conf > 10-bridge.conf

  sed "s|SUBNET|$SUBNET|g" \
    configs/kubelet-config.yaml > kubelet-config.yaml

  scp 10-bridge.conf kubelet-config.yaml \
  root@${HOST}:~/
done
```

```bash
for HOST in node-0 node-1; do
  scp \
    downloads/worker/* \
    downloads/client/kubectl \
    configs/99-loopback.conf \
    configs/containerd-config.toml \
    configs/kube-proxy-config.yaml \
    units/containerd.service \
    units/kubelet.service \
    units/kube-proxy.service \
    root@${HOST}:~/
done
```

```bash
for HOST in node-0 node-1; do
  scp \
    downloads/cni-plugins/* \
    root@${HOST}:~/cni-plugins/
done
```

下一节的命令必须在每个工作节点实例上运行：`node-0`、`node-1`。使用 `ssh` 命令登录工作节点实例，例如：

```bash
ssh root@node-0
```

## 部署 Kubernetes 工作节点

安装操作系统依赖：

```bash
{
  yum -y install socat conntrack ipset kmod iptables
}
```

> socat 二进制文件用于支持 `kubectl port-forward` 命令。

禁用 Swap

Kubernetes 对 swap 内存的支持有限，因为在使用 swap 时，很难为 Pod 的内存使用提供保障并进行核算。

检查 swap 是否已禁用：

```bash
swapon --show
```

如果输出为空，说明 swap 已禁用。如果 swap 已启用，运行以下命令立即禁用 swap：

```bash
swapoff -a
```

> 要确保重启后 swap 仍保持禁用，请查阅你所使用 Linux 发行版的文档。

创建安装目录：

```bash
mkdir -p \
  /etc/cni/net.d \
  /opt/cni/bin \
  /var/lib/kubelet \
  /var/lib/kube-proxy \
  /var/lib/kubernetes \
  /var/run/kubernetes
```

安装工作节点二进制文件：

```bash
{
  mv crictl kube-proxy kubelet runc \
    /usr/local/bin/
  mv containerd containerd-shim-runc-v2 containerd-stress /bin/
  mv cni-plugins/* /opt/cni/bin/
}
```

### 配置 CNI 网络

创建 `bridge` 网络配置文件：

```bash
mv 10-bridge.conf 99-loopback.conf /etc/cni/net.d/
```

为了确保穿越 CNI `bridge` 网络的流量由 `iptables` 处理，加载并配置 `br-netfilter` 内核模块：

```bash
{
  modprobe br-netfilter
  echo "br-netfilter" >> /etc/modules-load.d/modules.conf
}
```

```bash
{
  echo "net.bridge.bridge-nf-call-iptables = 1" \
    >> /etc/sysctl.d/kubernetes.conf
  echo "net.bridge.bridge-nf-call-ip6tables = 1" \
    >> /etc/sysctl.d/kubernetes.conf
  sysctl -p /etc/sysctl.d/kubernetes.conf
}
```

### 配置 containerd

安装 `containerd` 配置文件：

```bash
{
  mkdir -p /etc/containerd/
  mv containerd-config.toml /etc/containerd/config.toml
  mv containerd.service /etc/systemd/system/
}
```

### 配置 Kubelet

创建 `kubelet-config.yaml` 配置文件：

```bash
{
  mv kubelet-config.yaml /var/lib/kubelet/
  mv kubelet.service /etc/systemd/system/
}
```

### 配置 Kubernetes Proxy

```bash
{
  mv kube-proxy-config.yaml /var/lib/kube-proxy/
  mv kube-proxy.service /etc/systemd/system/
}
```

### 启动工作节点服务

```bash
{
  systemctl daemon-reload
  systemctl enable containerd kubelet kube-proxy
  systemctl start containerd kubelet kube-proxy
}
```

检查 kubelet 服务是否正在运行：

```bash
systemctl is-active kubelet
```

```text
active
```

在进入下一节之前，请务必在 `node-0` 和 `node-1` 这两个工作节点上完成本节的所有步骤。

## 验证

在 `jumpbox` 机器上运行以下命令。

列出已注册的 Kubernetes 节点：

```bash
ssh root@server \
  "kubectl get nodes \
  --kubeconfig admin.kubeconfig"
```

```
NAME     STATUS   ROLES    AGE    VERSION
node-0   Ready    <none>   1m     v1.32.3
node-1   Ready    <none>   10s    v1.32.3
```

下一节：[配置 kubectl 远程访问](10-configuring-kubectl.md)
