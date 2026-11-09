# 创建计算资源

Kubernetes 需要一组机器来承载 Kubernetes 控制平面，以及最终运行容器的工作节点。在本次实验中，你将创建搭建 Kubernetes 集群所需的机器。

## 机器清单文件

本教程将借助一个文本文件来充当机器清单（machine database），用于保存搭建 Kubernetes 控制平面和 worker 节点时会用到的各种机器属性。以下格式定义了机器清单中的条目，每行一条：

```text
IPV4_ADDRESS FQDN HOSTNAME POD_SUBNET
```

每一列分别对应机器 IP 地址 `IPV4_ADDRESS`、域名 `FQDN`、主机名 `HOSTNAME` 以及 IP 子网 `POD_SUBNET`。Kubernetes 会为每个 `pod` 分配一个 IP 地址，而 `POD_SUBNET` 表示分配给集群中每台机器的、用于该用途的唯一 IP 地址段。

下面是一个示例机器清单，与创建本教程时所用的类似。请注意其中的 IP 地址已被隐去。只要每台机器之间以及每台机器与 `jumpbox` 之间可以互相访问，你的机器可以分配任意 IP 地址。

```bash
cat machines.txt
```

```text
XXX.XXX.XXX.XXX server.kubernetes.local server
XXX.XXX.XXX.XXX node-0.kubernetes.local node-0 10.200.0.0/24
XXX.XXX.XXX.XXX node-1.kubernetes.local node-1 10.200.1.0/24
```

现在轮到你了：参照上面的示例机器清单，为你要用来创建 Kubernetes 集群的三台机器创建一个 `machines.txt` 文件，填入它们的详细信息。

## 配置 SSH 访问

本教程将使用 SSH 来配置集群中的机器。请确认你可以通过 `root` 用户 SSH 登录机器清单中列出的每一台机器。你可能需要通过修改 sshd_config 文件并重启 SSH 服务，在每台节点上启用 root 用户的 SSH 访问。


### 生成并分发 SSH 密钥

在本节中，你将生成一对 SSH 密钥并分发到 `server`、`node-0` 和 `node-1` 这几台机器上，在本教程后续章节中，你将使用它们在这些机器上执行命令。请在 `jumpbox` 机器上运行以下命令。

生成新的 SSH 密钥：

```bash
ssh-keygen
```

```text
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /root/.ssh/id_rsa
Your public key has been saved in /root/.ssh/id_rsa.pub
```

将 SSH 公钥复制到每台机器：

```bash
while read IP FQDN HOST SUBNET; do
  ssh-copy-id root@${IP}
done < machines.txt
```

每台机器都添加完密钥后，验证 SSH 公钥访问是否可用：

```bash
while read IP FQDN HOST SUBNET; do
  ssh -n root@${IP} hostname
done < machines.txt
```

```text
server
node-0
node-1
```

## 主机名

在本节中，你将为主机名分配给 `server`、`node-0` 和 `node-1` 机器。当你从 `jumpbox` 向各台机器执行命令时会用到这些主机名。主机名在集群内部也扮演着重要角色：Kubernetes 客户端不会使用 IP 地址向 Kubernetes API server 发送命令，而是使用 `server` 这个主机名。此外，每台 worker 机器 `node-0` 和 `node-1` 在向给定的 Kubernetes 集群注册时也会用到主机名。

要为每台机器配置主机名，请在 `jumpbox` 上运行以下命令。

为 `machines.txt` 文件中列出的每台机器设置主机名：

```bash
while read IP FQDN HOST SUBNET; do
    CMD="sed -i 's/^127.0.1.1.*/127.0.1.1\t${FQDN} ${HOST}/' /etc/hosts"
    ssh -n root@${IP} "$CMD"
    ssh -n root@${IP} hostnamectl set-hostname ${HOST}
    ssh -n root@${IP} systemctl restart systemd-hostnamed
done < machines.txt
```

验证每台机器的主机名是否已设置：

```bash
while read IP FQDN HOST SUBNET; do
  ssh -n root@${IP} hostname --fqdn
done < machines.txt
```

```text
server.kubernetes.local
node-0.kubernetes.local
node-1.kubernetes.local
```

## 主机查找表

在本节中，你将生成一个 `hosts` 文件，它的内容会被追加到 `jumpbox` 的 `/etc/hosts` 文件，以及本教程所用三台集群成员机器的 `/etc/hosts` 文件中。这样每台机器都可以通过 `server`、`node-0` 或 `node-1` 这样的主机名被访问到。

创建一个新的 `hosts` 文件，并添加一行头部注释，用于标识即将添加的机器：

```bash
echo "" > hosts
echo "# Kubernetes The Hard Way" >> hosts
```

为 `machines.txt` 文件中的每台机器生成一行主机记录，并追加到 `hosts` 文件：

```bash
while read IP FQDN HOST SUBNET; do
    ENTRY="${IP} ${FQDN} ${HOST}"
    echo $ENTRY >> hosts
done < machines.txt
```

查看 `hosts` 文件中的主机记录：

```bash
cat hosts
```

```text

# Kubernetes The Hard Way
XXX.XXX.XXX.XXX server.kubernetes.local server
XXX.XXX.XXX.XXX node-0.kubernetes.local node-0
XXX.XXX.XXX.XXX node-1.kubernetes.local node-1
```

## 向本机添加 `/etc/hosts` 条目

在本节中，你将把 `hosts` 文件中的 DNS 条目追加到 `jumpbox` 机器本地的 `/etc/hosts` 文件中。

将 `hosts` 中的 DNS 条目追加到 `/etc/hosts`：

```bash
cat hosts >> /etc/hosts
```

验证 `/etc/hosts` 文件已更新：

```bash
cat /etc/hosts
```

```text
127.0.0.1       localhost
127.0.1.1       jumpbox

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

# Kubernetes The Hard Way
XXX.XXX.XXX.XXX server.kubernetes.local server
XXX.XXX.XXX.XXX node-0.kubernetes.local node-0
XXX.XXX.XXX.XXX node-1.kubernetes.local node-1
```

此时，你应该可以使用主机名 SSH 登录到 `machines.txt` 文件中列出的每一台机器。

```bash
for host in server node-0 node-1
   do ssh root@${host} hostname
done
```

```text
server
node-0
node-1
```

## 向远程机器添加 `/etc/hosts` 条目

在本节中，你将把 `hosts` 文件中的主机记录追加到 `machines.txt` 文本文件中列出的每台机器的 `/etc/hosts` 文件里。

将 `hosts` 文件复制到每台机器，并把内容追加到 `/etc/hosts`：

```bash
while read IP FQDN HOST SUBNET; do
  scp hosts root@${HOST}:~/
  ssh -n \
    root@${HOST} "cat hosts >> /etc/hosts"
done < machines.txt
```

至此，从 `jumpbox` 或 Kubernetes 集群中任意一台机器连接到其他机器时，都可以使用主机名了。你现在不必再使用 IP 地址，而是可以通过 `server`、`node-0` 或 `node-1` 这样的主机名连接到机器。

下一节：[创建 CA 并生成 TLS 证书](04-certificate-authority.md)
