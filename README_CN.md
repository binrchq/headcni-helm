# HeadCNI Helm Chart 示例配置

<div align="center">

![HeadCNI Logo](https://img.shields.io/badge/HeadCNI-CNI%20Plugin-blue?style=for-the-badge&logo=kubernetes)
![Kubernetes](https://img.shields.io/badge/Kubernetes-1.20+-326CE5?style=for-the-badge&logo=kubernetes)
![Helm](https://img.shields.io/badge/Helm-3.0+-0F1689?style=for-the-badge&logo=helm)
![Tailscale](https://img.shields.io/badge/Tailscale-WireGuard-FF6B6B?style=for-the-badge&logo=tailscale)

**基于实际 Helm Chart 结构的完整部署配置示例**

[English](README.md) | 中文

</div>

---

## 📋 目录

- [🎯 项目简介](#-项目简介)
- [📁 文件说明](#-文件说明)
- [🚀 快速开始](#-快速开始)
- [🔧 配置详解](#-配置详解)
- [📊 监控与调试](#-监控与调试)
- [🐛 故障排除](#-故障排除)
- [🌟 最佳实践](#-最佳实践)
- [📚 相关资源](#-相关资源)

## 🎯 项目简介

HeadCNI 是一个创新的 Kubernetes CNI 插件，它巧妙地将 Headscale 和 Tailscale 的功能融为一体，为 Kubernetes 集群提供模块化、可扩展的网络解决方案。

### ✨ 核心特性

- 🔒 **零配置网络安全** - 自动发现和配置 Tailscale 网络
- ⚡ **高性能网络转发** - 基于 veth 对的高效网络架构
- 🛡️ **企业级安全** - 利用 Tailscale 的 WireGuard 加密技术
- 🚀 **简单部署** - 无需额外的 etcd 集群，开箱即用
- 📈 **监控友好** - 内置 Prometheus 指标收集
- 🎯 **智能 IPAM** - 支持多种 IP 分配策略
- 🔄 **动态管理** - 守护进程 + 插件架构

---

## 📁 文件说明

### 1. `headcni-k3s-allinone.yaml` 🐳
**用途**: k3s 集群的 HeadCNI 完整部署配置

**特点**: 
- 🎯 使用 k3s 默认 CIDR 范围 (Pod: `10.42.0.0/16`, Service: `10.43.0.0/16`)
- 🚫 禁用 MagicDNS 以避免与 k3s 内置 DNS 冲突
- ⚡ 优化了 k3s 环境下的网络配置
- 🔧 包含完整的 RBAC 权限配置

### 2. `headcni-k8s-allinone.yaml` ☸️
**用途**: 标准 Kubernetes 集群的 HeadCNI 完整部署配置

**特点**:
- 🎯 使用标准 k8s CIDR 范围 (Pod: `10.244.0.0/16`, Service: `10.96.0.0/12`)
- ✨ 启用 MagicDNS 以获得更好的 DNS 解析体验
- 🏭 包含完整的生产环境配置
- 📊 增强的监控和日志配置

---

## 🚀 快速开始

### 📋 前置条件

<div align="center">

| 组件 | 版本要求 | 说明 |
|------|----------|------|
| **Kubernetes** | 1.20+ | 支持 k3s 和标准 k8s |
| **Helm** | 3.0+ | 包管理工具 |
| **HeadScale** | 最新版本 | Tailscale 控制服务器 |
| **Linux 内核** | 4.19+ | 网络功能支持 |

</div>

### 🎯 部署步骤

#### 步骤 1: 更新配置
编辑相应的 YAML 文件，更新以下关键配置：

```yaml
config:
  headscale:
    url: "https://your-actual-headscale-server:8080"  # 🔗 实际服务器地址
    authKey: "your-actual-auth-key"                   # 🔑 实际认证密钥
```

#### 步骤 2: 部署到 k3s 集群
```bash
# 🚀 克隆项目
git clone <repository-url>
cd headcni-helm

# 🎯 使用 k3s 配置部署
helm install headcni-k3s . -f examples/headcni-k3s-allinone.yaml
```

#### 步骤 3: 部署到标准 Kubernetes 集群
```bash
# 🎯 使用 k8s 配置部署
helm install headcni-k8s . -f examples/headcni-k8s-allinone.yaml
```

---

## 🔧 配置详解

### 🎛️ 核心配置项

#### HeadScale 配置
| 配置项 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `url` | string | - | HeadScale 服务器地址 |
| `authKey` | string | - | 认证密钥（通过 Secret 管理） |
| `timeout` | string | "30s" | 连接超时时间 |
| `retries` | integer | 3 | 重试次数 |

#### Tailscale 配置
| 配置项 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `mode` | string | "daemon" | 运行模式（daemon 或 host） |
| `acceptDNS` | boolean | false | 是否接受 HeadScale DNS 设置 |
| `interfaceName` | string | "headcni01" | Tailscale 接口名称 |
| `tags` | array | - | 节点标签，用于访问控制 |

#### 网络配置
| 配置项 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `podCIDR` | string | - | 集群 Pod CIDR 范围 |
| `serviceCIDR` | string | - | 集群 Service CIDR 范围 |
| `mtu` | integer | 1280 | 网络接口 MTU 值 |
| `enableNetworkPolicy` | boolean | true | 是否启用网络策略 |

---

## 📊 监控与调试

### 🔍 检查部署状态

```bash
# 📊 查看 Pod 状态
kubectl get pods -n kube-system -l app=headcni

# 📝 查看日志
kubectl logs -n kube-system -l app=headcni

# 🎯 查看 DaemonSet 状态
kubectl get daemonset -n kube-system
```

### 🌐 检查网络配置

```bash
# 🔧 查看 CNI 配置
kubectl exec -n kube-system -c headcni-daemon <pod-name> -- cat /etc/cni/net.d/10-headcni.conflist

# 📡 查看 Tailscale 状态
kubectl exec -n kube-system -c headcni-daemon <pod-name> -- tailscale status
```

### 📈 访问监控指标

```bash
# 🔌 端口转发
kubectl port-forward -n kube-system svc/headcni-metrics 9001:9001

# 📊 访问指标页面
curl http://localhost:9001/metrics
```

---

## 🐛 故障排除

### ❗ 常见问题

#### 1. Pod 无法启动
**症状**: Pod 一直处于 Pending 或 CrashLoopBackOff 状态

**排查步骤**:
- 🔍 检查 HeadScale 服务器连通性
- 🔑 验证认证密钥是否正确
- 📝 查看 Pod 事件和日志
- 🌐 确认网络策略配置

#### 2. 网络不通
**症状**: Pod 间无法通信或无法访问外部服务

**排查步骤**:
- 🔧 检查 CNI 配置是否正确
- 📡 验证 Pod CIDR 配置
- 🌐 检查 Tailscale 接口状态
- 🔒 确认防火墙规则

#### 3. DNS 解析失败
**症状**: 域名解析超时或失败

**排查步骤**:
- ✨ 检查 MagicDNS 配置
- 🌐 验证 DNS 服务器设置
- 📄 检查 resolv.conf 配置
- 🔧 确认 DNS 策略设置

### 🛠️ 调试命令

```bash
# 🚀 进入 Pod 调试
kubectl exec -n kube-system -it <pod-name> -- /bin/sh

# 🌐 检查网络接口
ip addr show

# 🛣️ 检查路由表
ip route show

# 🏓 测试网络连通性
ping <target-ip>

# 📊 检查网络统计
netstat -i
```

---

## 🌟 最佳实践

### 🏗️ 架构设计

1. **网络规划**
   - 合理规划 Pod 和 Service CIDR 范围
   - 避免与现有网络地址冲突
   - 考虑未来扩展需求

2. **安全配置**
   - 启用网络策略控制流量
   - 配置适当的 RBAC 权限
   - 定期轮换认证密钥

3. **监控告警**
   - 配置 Prometheus 指标收集
   - 设置关键指标告警
   - 建立日志聚合系统

### 📈 性能优化

1. **资源管理**
   - 合理设置 CPU 和内存限制
   - 监控资源使用情况
   - 优化垃圾回收配置

2. **网络优化**
   - 调整 MTU 值优化传输效率
   - 配置连接池参数
   - 启用网络缓存

---

## 📚 相关资源

### 🔗 官方资源

- [HeadCNI 项目主页](https://github.com/binrc/headcni) 🏠
- [HeadScale 官方文档](https://github.com/juanfont/headscale) 📖
- [Tailscale 官方文档](https://tailscale.com/kb/) 📚
- [Kubernetes CNI 规范](https://github.com/containernetworking/cni) 📋

### 📖 学习资源

- [Kubernetes 网络模型详解](https://kubernetes.io/docs/concepts/services-networking/) 🌐
- [CNI 插件开发指南](https://github.com/containernetworking/cni/blob/main/SPEC.md) 🔧
- [Tailscale 网络原理](https://tailscale.com/blog/how-tailscale-works/) 🧠

### 🛠️ 工具推荐

- [Helm 包管理器](https://helm.sh/) 📦
- [k9s 集群管理工具](https://k9scli.io/) 🎮
- [Lens IDE](https://k8slens.dev/) 👁️

---

## 🤝 贡献指南

如果您发现配置问题或有改进建议，我们非常欢迎您的贡献：

### 🐛 报告问题
1. 在 GitHub 上创建 Issue
2. 详细描述问题和复现步骤
3. 提供相关的日志和配置信息

### 💡 贡献代码
1. Fork 项目仓库
2. 创建功能分支
3. 提交 Pull Request
4. 参与代码审查

### 💬 社区讨论
- 加入我们的 Discord 频道
- 参与 GitHub Discussions
- 关注项目更新动态

---

## 📄 许可证

本项目采用与 HeadCNI 项目相同的许可证。

<div align="center">

**Made with ❤️ by the HeadCNI Community**

[![GitHub stars](https://img.shields.io/github/stars/binrc/headcni?style=social)](https://github.com/binrc/headcni)
[![GitHub forks](https://img.shields.io/github/forks/binrc/headcni?style=social)](https://github.com/binrc/headcni)
[![GitHub issues](https://img.shields.io/github/issues/binrc/headcni)](https://github.com/binrc/headcni/issues)

</div> 