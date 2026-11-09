# 初始化 etcd 集群

Kubernetes 组件本身是无状态的，它们将集群状态存储在 [etcd](https://github.com/etcd-io/etcd) 中。在本实验中，你将初始化一个单节点的 etcd 集群。

## 前提条件

将 `etcd` 二进制文件和 systemd unit 文件复制到 `server` 机器上：

```bash
scp \
  downloads/controller/etcd \
  downloads/client/etcdctl \
  units/etcd.service \
  root@server:~/
```

本实验中的命令必须在 `server` 机器上运行。使用 `ssh` 命令登录到 `server` 机器，例如：

```bash
ssh root@server
```

## 初始化 etcd 集群

### 安装 etcd 二进制文件

解压并安装 `etcd` 服务端和 `etcdctl` 命令行工具：

```bash
{
  mv etcd etcdctl /usr/local/bin/
}
```

### 配置 etcd 服务端

```bash
{
  mkdir -p /etc/etcd /var/lib/etcd
  chmod 700 /var/lib/etcd
  cp ca.crt kube-api-server.key kube-api-server.crt \
    /etc/etcd/
}
```

每个 etcd 成员在 etcd 集群中必须拥有唯一的名称。将 etcd 的名称设置为与当前计算实例的主机名一致：

创建 `etcd.service` systemd unit 文件：

```bash
mv etcd.service /etc/systemd/system/
```

### 启动 etcd 服务端

```bash
{
  systemctl daemon-reload
  systemctl enable etcd
  systemctl start etcd
}
```

## 验证

列出 etcd 集群的成员：

```bash
etcdctl member list
```

```text
6702b0a34e2cfd39, started, controller, http://127.0.0.1:2380, http://127.0.0.1:2379, false
```

下一节：[引导 Kubernetes 控制平面](08-bootstrapping-kubernetes-controllers.zh-CN.md)
