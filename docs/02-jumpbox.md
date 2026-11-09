# 搭建 jumpbox

在本次实验中，你会将四台机器中的一台配置为 `jumpbox`。在整个教程中，你都会在这台机器上执行命令。虽然我们使用一台专用机器是为了保证环境一致，但这些命令几乎可以在任何机器上运行，包括你运行 macOS 或 Linux 的个人工作站。

可以把 `jumpbox` 看作你的管理机器，在从零开始搭建 Kubernetes 集群的过程中，它就是你的大本营。在开始之前，我们需要安装几个命令行工具，并克隆 Kubernetes The Hard Way 的 git 仓库，其中包含了一些额外的配置文件，这些文件将在本教程后续配置各类 Kubernetes 组件时用到。

登录 `jumpbox`：

```bash
ssh root@jumpbox
```

所有命令都将以 `root` 用户身份执行。这样做是为了方便，同时也能减少搭建整个环境所需的命令数量。

### 安装命令行工具

现在你已以 `root` 用户身份登录到 `jumpbox` 机器，接下来安装本教程中执行各类任务所需的命令行工具。

```bash
{
  yum -y install wget curl vim openssl git
}
```

### 同步 GitHub 仓库

接下来下载本教程的一份副本，其中包含用于从零开始构建 Kubernetes 集群的配置文件和模板。使用 `git` 命令克隆 Kubernetes The Hard Way 的 git 仓库：

```bash
git clone --depth 1 https://github.com/MaiguHexv/kubernetes-the-hard-way.git
```

切换至 `kubernetes-the-hard-way` 目录：

```bash
cd kubernetes-the-hard-way
```

这将是本教程后续章节的工作目录。如果之后你迷路了，可以在 `jumpbox` 上运行 `pwd` 命令，确认自己处在正确的目录中：

```bash
pwd
```

```text
/root/kubernetes-the-hard-way
```

### 下载二进制文件

在本节中，你将下载各类 Kubernetes 组件的二进制文件。这些二进制文件会被存放在 `jumpbox` 上的 `downloads` 目录中，这样可以避免为 Kubernetes 集群中的每台机器重复下载，从而减少完成本教程所需的网络带宽。

需要下载的二进制文件列表保存在 `downloads-amd64.txt` 或 `downloads-arm64.txt` 文件中，具体取决于你的硬件架构。你可以用 `cat` 命令查看：

```bash
cat downloads-$(uname -m | sed -e 's/x86_64/amd64/' -e 's/aarch64/arm64/').txt
```

使用 `wget` 命令将二进制文件下载到名为 `downloads` 的目录中：

```bash
wget -q --show-progress \
  --https-only \
  --timestamping \
  -P downloads \
  -i downloads-$(uname -m | sed -e 's/x86_64/amd64/' -e 's/aarch64/arm64/').txt
```

由于国内限速，你可以在Chrome中下载好之后再传到jumpbox机器上。下载完成后，可以用 `ls` 命令列出它们：

```bash
ls -oh downloads
```

从发布归档中解压出各组件二进制文件，并整理到 `downloads` 目录下的相应子目录中。

```bash
{
  ARCH=$(uname -m | sed -e 's/x86_64/amd64/' -e 's/aarch64/arm64/')
  mkdir -p downloads/{client,cni-plugins,controller,worker}
  tar -xvf downloads/crictl-v1.32.0-linux-${ARCH}.tar.gz \
    -C downloads/worker/
  tar -xvf downloads/containerd-2.1.0-beta.0-linux-${ARCH}.tar.gz \
    --strip-components 1 \
    -C downloads/worker/
  tar -xvf downloads/cni-plugins-linux-${ARCH}-v1.6.2.tgz \
    -C downloads/cni-plugins/
  tar -xvf downloads/etcd-v3.6.0-rc.3-linux-${ARCH}.tar.gz \
    -C downloads/ \
    --strip-components 1 \
    etcd-v3.6.0-rc.3-linux-${ARCH}/etcdctl \
    etcd-v3.6.0-rc.3-linux-${ARCH}/etcd
  mv downloads/{etcdctl,kubectl} downloads/client/
  mv downloads/{etcd,kube-apiserver,kube-controller-manager,kube-scheduler} \
    downloads/controller/
  mv downloads/{kubelet,kube-proxy} downloads/worker/
  mv downloads/runc.${ARCH} downloads/worker/runc
}
```

```bash
rm -rf downloads/*gz
```

为这些二进制文件添加可执行权限。

```bash
{
  chmod +x downloads/{client,cni-plugins,controller,worker}/*
}
```

### 安装 kubectl

在本节中，你将在 `jumpbox` 机器上安装 `kubectl`，它是 Kubernetes 官方的客户端命令行工具。在本教程后面完成集群搭建之后，`kubectl` 将用于与 Kubernetes 控制平面交互。

使用 `chmod` 命令为 `kubectl` 二进制文件添加可执行权限，并将它移动到 `/usr/local/bin/` 目录：

```bash
{
  cp downloads/client/kubectl /usr/local/bin/
}
```

至此 `kubectl` 已安装完成，可以通过运行 `kubectl` 命令来验证：

```bash
kubectl version --client
```

```text
Client Version: v1.32.3
Kustomize Version: v5.5.0
```

到此为止，`jumpbox` 上已经配置好了完成本教程各项实验所需的全部命令行工具和实用程序。

下一节：[创建计算资源](03-compute-resources.md)
