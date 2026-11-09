# 准备工作

本节将介绍跟随本教程所需满足的机器要求。

## 虚拟机或物理机

本教程需要四（4）台运行 Debian 12（bookworm）的 ARM64 或 AMD64 架构的虚拟机或物理机。下表列出了这四台机器及其 CPU、内存和存储要求。

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
PRETTY_NAME="Debian GNU/Linux 12 (bookworm)"
NAME="Debian GNU/Linux"
VERSION_ID="12"
VERSION="12 (bookworm)"
VERSION_CODENAME=bookworm
ID=debian
```

下一节：[搭建 jumpbox](02-jumpbox.zh-CN.md)
