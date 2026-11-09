# 准备工作

本节将介绍跟随本教程所需满足的机器要求。

## 虚拟机或物理机

本教程需要四（4）台运行 Rocky Linux 的 ARM64 或 AMD64 架构的虚拟机或物理机。下表列出了这四台机器及其 CPU、内存和存储要求。

| 名称     | 描述                   | CPU | 内存   | 存储  |
|---------|------------------------|-----|-------|------|
| jumpbox | 管理主机                | 1   | 512MB | 10GB |
| server  | Kubernetes 服务端       | 1   | 2GB   | 20GB |
| node-0  | Kubernetes 工作节点     | 1   | 2GB   | 20GB |
| node-1  | Kubernetes 工作节点     | 1   | 2GB   | 20GB |

如何准备这些机器由你自行决定，唯一的要求是每台机器都满足上述系统要求，包括机器规格和操作系统版本。四台机器准备好之后，通过查看 `/etc/os-release` 文件来验证操作系统要求：

```bash
cat /etc/os-release
```

你应该会看到类似下面的输出：

```text
NAME="Rocky Linux"
VERSION="10.2 (Red Quartz)"
RELEASE_TYPE="stable"
ID="rocky"
ID_LIKE="rhel centos fedora"
VERSION_ID="10.2"
PLATFORM_ID="platform:el10"
PRETTY_NAME="Rocky Linux 10.2 (Red Quartz)"
ANSI_COLOR="0;32"
LOGO="fedora-logo-icon"
CPE_NAME="cpe:/o:rocky:rocky:10::baseos"
HOME_URL="https://rockylinux.org/"
VENDOR_NAME="RESF"
VENDOR_URL="https://resf.org/"
BUG_REPORT_URL="https://bugs.rockylinux.org/"
SUPPORT_END="2035-05-31"
ROCKY_SUPPORT_PRODUCT="Rocky-Linux-10"
ROCKY_SUPPORT_PRODUCT_VERSION="10.2"
REDHAT_SUPPORT_PRODUCT="Rocky Linux"
REDHAT_SUPPORT_PRODUCT_VERSION="10.2"
```

下一节：[搭建 jumpbox](02-jumpbox.md)
