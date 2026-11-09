# 生成数据加密配置和密钥

Kubernetes 存储着各种类型的数据，包括集群状态、应用配置和机密信息（Secrets）。Kubernetes 支持对集群静态数据进行[加密](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data)。

在本实验中，你将生成一个加密密钥和一个[加密配置](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#understanding-the-encryption-at-rest-configuration)，用于加密 Kubernetes 的 Secrets。

## 加密密钥

生成加密密钥：

```bash
export ENCRYPTION_KEY=$(head -c 32 /dev/urandom | base64)
```

## 加密配置文件

创建 `encryption-config.yaml` 加密配置文件：

```bash
envsubst < configs/encryption-config.yaml \
  > encryption-config.yaml
```

将 `encryption-config.yaml` 加密配置文件复制到每一台 controller 实例上：

```bash
scp encryption-config.yaml root@server:~/
```

下一节：[初始化 etcd 集群](07-bootstrapping-etcd.zh-CN.md)
