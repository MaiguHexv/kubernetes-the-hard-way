# 创建 CA 并生成 TLS 证书

在本次实验中，你将使用 openssl 创建一套 [PKI 基础设施](https://en.wikipedia.org/wiki/Public_key_infrastructure)，用它来初始化一个证书颁发机构（Certificate Authority），并为以下组件生成 TLS 证书：kube-apiserver、kube-controller-manager、kube-scheduler、kubelet 和 kube-proxy。本节中的命令应在 `jumpbox` 上执行。

## 证书颁发机构

在本节中，你将创建一套证书颁发机构，用于为其他 Kubernetes 组件生成额外的 TLS 证书。使用 `openssl` 搭建 CA 并生成证书可能比较耗时，尤其是第一次操作的时候。为简化本次实验，我准备了一个 openssl 配置文件 `ca.conf`，它定义了为每个 Kubernetes 组件生成证书所需的全部细节。

花点时间看一下 `ca.conf` 配置文件：

```bash
cat ca.conf
```

你不需要理解 `ca.conf` 文件中的所有内容也能完成本教程，但可以把它当作学习 `openssl` 以及从高层面上理解证书管理相关配置的起点。

每个证书颁发机构都始于一个私钥和根证书。在本节中我们将创建一个自签名的证书颁发机构。虽然本教程只需要这么多，但在真实的生产环境中不应这样做。

生成 CA 配置文件、证书和私钥：

```bash
{
  openssl genrsa -out ca.key 4096
  openssl req -x509 -new -sha512 -noenc \
    -key ca.key -days 3653 \
    -config ca.conf \
    -out ca.crt
}
```

结果：

```txt
ca.crt ca.key
```

## 创建客户端和服务端证书

在本节中，你将分别为每个 Kubernetes 组件生成客户端和服务端证书，并为 Kubernetes 的 `admin` 用户生成一个客户端证书。

生成证书和私钥：

```bash
certs=(
  "admin" "node-0" "node-1"
  "kube-proxy" "kube-scheduler"
  "kube-controller-manager"
  "kube-api-server"
  "service-accounts"
)
```

```bash
for i in ${certs[*]}; do
  openssl genrsa -out "${i}.key" 4096

  openssl req -new -key "${i}.key" -sha256 \
    -config "ca.conf" -section ${i} \
    -out "${i}.csr"

  openssl x509 -req -days 3653 -in "${i}.csr" \
    -copy_extensions copyall \
    -sha256 -CA "ca.crt" \
    -CAkey "ca.key" \
    -CAcreateserial \
    -out "${i}.crt"
done
```

运行上述命令后，会为每个 Kubernetes 组件生成一个私钥、一个证书请求文件以及一个已签名的 SSL 证书。你可以用下面的命令列出生成的文件：

```bash
ls -1 *.crt *.key *.csr
```

## 分发客户端和服务端证书

在本节中，你会将各类证书复制到每台机器上，并放在各 Kubernetes 组件查找自己证书对的路径下。在真实环境中，这些证书应当被视为一组敏感机密，因为 Kubernetes 组件会把它们作为凭据，用于彼此之间的身份认证。

将相应的证书和私钥复制到 `node-0` 和 `node-1` 机器：

```bash
for host in node-0 node-1; do
  ssh root@${host} mkdir /var/lib/kubelet/

  scp ca.crt root@${host}:/var/lib/kubelet/

  scp ${host}.crt \
    root@${host}:/var/lib/kubelet/kubelet.crt

  scp ${host}.key \
    root@${host}:/var/lib/kubelet/kubelet.key
done
```

将相应的证书和私钥复制到 `server` 机器：

```bash
scp \
  ca.key ca.crt \
  kube-api-server.key kube-api-server.crt \
  service-accounts.key service-accounts.crt \
  root@server:~/
```

> `kube-proxy`、`kube-controller-manager`、`kube-scheduler` 和 `kubelet` 的客户端证书将在下一个实验中用于生成客户端身份认证配置文件。

下一节：[生成用于身份认证的 Kubernetes 配置文件](05-kubernetes-configuration-files.md)
