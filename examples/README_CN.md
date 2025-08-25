# HeadCNI Helm Chart 示例配置

基于实际 Helm Chart 结构的 HeadCNI 完整部署配置示例。

English | [中文](README_CN.md)

## 概述

HeadCNI 是一个 Kubernetes CNI 插件，它集成了 Headscale 和 Tailscale 功能，为 Kubernetes 集群提供模块化、可扩展的网络解决方案。

## 文件说明

### `headcni-k3s-allinone.yaml`
- **用途**: k3s 集群的 HeadCNI 完整部署配置
- **特点**: 
  - 使用 k3s 默认 CIDR 范围 (Pod: `10.42.0.0/16`, Service: `10.43.0.0/16`)
  - 禁用 MagicDNS 以避免与 k3s 内置 DNS 冲突
  - 针对 k3s 环境优化
  - 完整的 RBAC 权限配置

### `headcni-k8s-allinone.yaml`
- **用途**: 标准 Kubernetes 集群的 HeadCNI 完整部署配置
- **特点**:
  - 使用标准 k8s CIDR 范围 (Pod: `10.244.0.0/16`, Service: `10.96.0.0/12`)
  - 启用 MagicDNS 获得更好的 DNS 解析体验
  - 生产就绪配置
  - 增强的监控和日志配置

## 快速开始

### 前置条件
- Kubernetes 1.20+ (支持 k3s 和标准 k8s)
- Helm 3.0+
- HeadScale 服务器
- HeadScale API 认证密钥

### 部署步骤

1. **更新配置**
   ```yaml
   config:
     headscale:
       url: "https://your-headscale-server:8080"
       authKey: "your-auth-key"
   ```

2. **部署到 k3s**
   ```bash
   helm install headcni-k3s ./headcni-helm -f examples/headcni-k3s-allinone.yaml
   ```

3. **部署到 Kubernetes**
   ```bash
   helm install headcni-k8s ./headcni-helm -f examples/headcni-k8s-allinone.yaml
   ```

## 配置说明

### 核心设置

| 配置项 | 键值 | 说明 |
|--------|------|------|
| **HeadScale** | `url` | HeadScale 服务器地址 |
| | `authKey` | 认证密钥 |
| | `timeout` | 连接超时 (默认: 30s) |
| | `retries` | 重试次数 (默认: 3) |
| **Tailscale** | `mode` | 运行模式: daemon/host |
| | `acceptDNS` | 接受 HeadScale DNS 设置 |
| | `interfaceName` | Tailscale 接口名称 |
| | `tags` | 节点标签，用于访问控制 |
| **网络** | `podCIDR` | 集群 Pod CIDR 范围 |
| | `serviceCIDR` | 集群 Service CIDR 范围 |
| | `mtu` | 网络接口 MTU |
| | `enableNetworkPolicy` | 启用网络策略 |

## 验证部署

### 检查状态
```bash
# Pod 状态
kubectl get pods -n kube-system -l app=headcni

# 日志
kubectl logs -n kube-system -l app=headcni

# DaemonSet 状态
kubectl get daemonset -n kube-system
```

### 网络配置
```bash
# CNI 配置
kubectl exec -n kube-system -c headcni-daemon <pod-name> -- cat /etc/cni/net.d/10-headcni.conflist

# Tailscale 状态
kubectl exec -n kube-system -c headcni-daemon <pod-name> -- tailscale status
```

### 监控指标
```bash
# 端口转发
kubectl port-forward -n kube-system svc/headcni-metrics 9001:9001

# 访问指标
curl http://localhost:9001/metrics
```

## 故障排除

### 常见问题

**Pod 无法启动**
- 检查 HeadScale 服务器连通性
- 验证认证密钥
- 查看 Pod 事件和日志

**网络问题**
- 验证 CNI 配置
- 检查 Pod CIDR 设置
- 确认 Tailscale 接口状态

**DNS 问题**
- 检查 MagicDNS 配置
- 验证 DNS 服务器设置
- 检查 resolv.conf

### 调试命令
```bash
# 进入 Pod
kubectl exec -n kube-system -it <pod-name> -- /bin/sh

# 网络接口
ip addr show

# 路由表
ip route show

# 连通性测试
ping <target-ip>
```

## 最佳实践

### 安全
- 启用网络策略
- 配置适当的 RBAC 权限
- 定期轮换认证密钥

### 性能
- 设置合理的资源限制
- 监控资源使用情况
- 优化 MTU 设置

### 监控
- 启用 Prometheus 指标
- 设置关键指标告警
- 实现日志聚合

## 相关资源

- [HeadCNI 项目](https://github.com/binrc/headcni)
- [HeadScale 文档](https://github.com/juanfont/headscale)
- [Tailscale 文档](https://tailscale.com/kb/)
- [Kubernetes CNI 规范](https://github.com/containernetworking/cni)

## 贡献

我们欢迎贡献！请：
1. 在 GitHub 上报告问题
2. 提交 Pull Request
3. 参与社区讨论

---

*由 HeadCNI 社区用心制作 ❤️* 