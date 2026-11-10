# Harbor

使用tar解压离线安装包

```sh
tar xzvf harbor-offline-installer-version.tgz
```

配置Harbor的HTTPS访问

首先生成证书颁发机构证书

1.生成CA证书私钥

```sh
openssl genrsa -out ca.key 4096
```

2.生成CA证书

```sh
openssl req -x509 -new -nodes -sha512 -days 3650 \
 -subj "/C=CN/ST=Beijing/L=Beijing/O=example/OU=Personal/CN=MyPersonal Root CA" \
 -key ca.key \
 -out ca.crt
```

生成服务器证书

以下步骤假设您的 Harbor 注册表的主机名是 `yourdomain.com`，并且其 DNS 记录指向您正在运行 Harbor 的主机

1.生成私钥

```sh
openssl genrsa -out yourdomain.com.key 4096
```

2.生成证书签名请求（CSR)

```sh
openssl req -sha512 -new \
    -subj "/C=CN/ST=Beijing/L=Beijing/O=example/OU=Personal/CN=yourdomain.com" \
    -key yourdomain.com.key \
    -out yourdomain.com.csr
```

3.生成x509v3扩展文件

```sh
cat > v3.ext <<-EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1=yourdomain.com
DNS.2=yourdomain
DNS.3=hostname
EOF
```

4.使用`v3.ext`文件为您的Harbor主机生成证书

```sh
openssl x509 -req -sha512 -days 3650 \
    -extfile v3.ext \
    -CA ca.crt -CAkey ca.key -CAcreateserial \
    -in yourdomain.com.csr \
    -out yourdomain.com.crt
```

向Harbor和Docker提供证书

生成 `ca.crt`、`yourdomain.com.crt`和 `yourdomain.com.key`文件后，您必须将它们提供给 Harbor 和 Docker，并重新配置 Harbor 以使用它们

1.将服务器证书和密钥复制到 Harbor 主机上的证书文件夹中

```sh
cp yourdomain.com.crt /data/cert/
cp yourdomain.com.key /data/cert/
```

2.将 `yourdomain.com.crt`转换为 `yourdomain.com.cert`，供 Docker 使用

```sh
openssl x509 -inform PEM -in yourdomain.com.crt -out yourdomain.com.cert
```

3.将服务器证书、密钥和 CA 文件复制到 Harbor 主机上的 Docker 证书文件夹中。您必须首先创建相应的文件夹

```sh
cp yourdomain.com.cert /etc/docker/certs.d/yourdomain.com/
cp yourdomain.com.key /etc/docker/certs.d/yourdomain.com/
cp ca.crt /etc/docker/certs.d/yourdomain.com/
```

4.重启Docker

```sh
systemctl restart docker
```

配置Harbor YML文件

运行安装脚本

```sh
sudo ./install.sh
```

登录验证

```sh
docker login yourdomain.com

Username: admin
Password:

WARNING! Your credentials are stored unencrypted in '/root/.docker/config.json'.
Configure a credential helper to remove this warning. See
https://docs.docker.com/go/credential-store/

Login Succeeded

```
