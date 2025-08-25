# HeadCNI Helm Chart Example Configurations

<div align="center">

![HeadCNI Logo](https://img.shields.io/badge/HeadCNI-CNI%20Plugin-blue?style=for-the-badge&logo=kubernetes)
![Kubernetes](https://img.shields.io/badge/Kubernetes-1.20+-326CE5?style=for-the-badge&logo=kubernetes)
![Helm](https://img.shields.io/badge/Helm-3.0+-0F1689?style=for-the-badge&logo=helm)
![Tailscale](https://img.shields.io/badge/Tailscale-WireGuard-FF6B6B?style=for-the-badge&logo=tailscale)

**Complete deployment configuration examples based on actual Helm Chart structure**

English | [中文](README_CN.md)

</div>

---

## 📋 Table of Contents

- [🎯 Project Overview](#-project-overview)
- [📁 File Descriptions](#-file-descriptions)
- [🚀 Quick Start](#-quick-start)
- [🔧 Configuration Guide](#-configuration-guide)
- [📊 Monitoring & Debugging](#-monitoring--debugging)
- [🐛 Troubleshooting](#-troubleshooting)
- [🌟 Best Practices](#-best-practices)
- [📚 Resources](#-resources)

## 🎯 Project Overview

HeadCNI is an innovative Kubernetes CNI plugin that seamlessly integrates Headscale and Tailscale functionality, providing a modular and extensible networking solution for Kubernetes clusters.

### ✨ Core Features

- 🔒 **Zero-Configuration Network Security** - Automatic discovery and configuration of Tailscale networks
- ⚡ **High-Performance Network Forwarding** - Efficient network architecture based on veth pairs
- 🛡️ **Enterprise-Grade Security** - Leverages Tailscale's WireGuard encryption technology
- 🚀 **Simple Deployment** - No additional etcd cluster required, ready to use out of the box
- 📈 **Monitoring Friendly** - Built-in Prometheus metrics collection
- 🎯 **Intelligent IPAM** - Supports multiple IP allocation strategies
- 🔄 **Dynamic Management** - Daemon + Plugin architecture

---

## 📁 File Descriptions

### 1. `headcni-k3s-allinone.yaml` 🐳
**Purpose**: Complete HeadCNI deployment configuration for k3s clusters

**Features**: 
- 🎯 Uses k3s default CIDR ranges (Pod: `10.42.0.0/16`, Service: `10.43.0.0/16`)
- 🚫 Disables MagicDNS to avoid conflicts with k3s built-in DNS
- ⚡ Optimized network configuration for k3s environment
- 🔧 Includes complete RBAC permission configuration

### 2. `headcni-k8s-allinone.yaml` ☸️
**Purpose**: Complete HeadCNI deployment configuration for standard Kubernetes clusters

**Features**:
- 🎯 Uses standard k8s CIDR ranges (Pod: `10.244.0.0/16`, Service: `10.96.0.0/12`)
- ✨ Enables MagicDNS for better DNS resolution experience
- 🏭 Includes complete production environment configuration
- 📊 Enhanced monitoring and logging configuration

---

## 🚀 Quick Start

### 📋 Prerequisites

<div align="center">

| Component | Version Requirement | Description |
|-----------|-------------------|-------------|
| **Kubernetes** | 1.20+ | Supports k3s and standard k8s |
| **Helm** | 3.0+ | Package manager |
| **HeadScale** | Latest version | Tailscale control server |
| **Linux Kernel** | 4.19+ | Network functionality support |

</div>

### 🎯 Deployment Steps

#### Step 1: Update Configuration
Edit the corresponding YAML file and update the following key configurations:

```yaml
config:
  headscale:
    url: "https://your-actual-headscale-server:8080"  # 🔗 Actual server address
    authKey: "your-actual-auth-key"                   # 🔑 Actual authentication key
```

#### Step 2: Deploy to k3s Cluster
```bash
# 🚀 Clone the project
git clone <repository-url>
cd headcni-helm

# 🎯 Deploy using k3s configuration
helm install headcni-k3s . -f examples/headcni-k3s-allinone.yaml
```

#### Step 3: Deploy to Standard Kubernetes Cluster
```bash
# 🎯 Deploy using k8s configuration
helm install headcni-k8s . -f examples/headcni-k8s-allinone.yaml
```

---

## 🔧 Configuration Guide

### 🎛️ Core Configuration Items

#### HeadScale Configuration
| Config Item | Type | Default | Description |
|-------------|------|---------|-------------|
| `url` | string | - | HeadScale server address |
| `authKey` | string | - | Authentication key (managed via Secret) |
| `timeout` | string | "30s" | Connection timeout |
| `retries` | integer | 3 | Retry count |

#### Tailscale Configuration
| Config Item | Type | Default | Description |
|-------------|------|---------|-------------|
| `mode` | string | "daemon" | Operation mode (daemon or host) |
| `acceptDNS` | boolean | false | Whether to accept HeadScale DNS settings |
| `interfaceName` | string | "headcni01" | Tailscale interface name |
| `tags` | array | - | Node tags for access control |

#### Network Configuration
| Config Item | Type | Default | Description |
|-------------|------|---------|-------------|
| `podCIDR` | string | - | Cluster Pod CIDR range |
| `serviceCIDR` | string | - | Cluster Service CIDR range |
| `mtu` | integer | 1280 | Network interface MTU value |
| `enableNetworkPolicy` | boolean | true | Whether to enable network policies |

---

## 📊 Monitoring & Debugging

### 🔍 Check Deployment Status

```bash
# 📊 View Pod status
kubectl get pods -n kube-system -l app=headcni

# 📝 View logs
kubectl logs -n kube-system -l app=headcni

# 🎯 View DaemonSet status
kubectl get daemonset -n kube-system
```

### 🌐 Check Network Configuration

```bash
# 🔧 View CNI configuration
kubectl exec -n kube-system -c headcni-daemon <pod-name> -- cat /etc/cni/net.d/10-headcni.conflist

# 📡 View Tailscale status
kubectl exec -n kube-system -c headcni-daemon <pod-name> -- tailscale status
```

### 📈 Access Monitoring Metrics

```bash
# 🔌 Port forward
kubectl port-forward -n kube-system svc/headcni-metrics 9001:9001

# 📊 Access metrics page
curl http://localhost:9001/metrics
```

---

## 🐛 Troubleshooting

### ❗ Common Issues

#### 1. Pod Cannot Start
**Symptoms**: Pod remains in Pending or CrashLoopBackOff state

**Troubleshooting Steps**:
- 🔍 Check HeadScale server connectivity
- 🔑 Verify authentication key is correct
- 📝 Check Pod events and logs
- 🌐 Confirm network policy configuration

#### 2. Network Not Working
**Symptoms**: Pods cannot communicate or access external services

**Troubleshooting Steps**:
- 🔧 Check if CNI configuration is correct
- 📡 Verify Pod CIDR configuration
- 🌐 Check Tailscale interface status
- 🔒 Confirm firewall rules

#### 3. DNS Resolution Failed
**Symptoms**: Domain resolution timeout or failure

**Troubleshooting Steps**:
- ✨ Check MagicDNS configuration
- 🌐 Verify DNS server settings
- 📄 Check resolv.conf configuration
- 🔧 Confirm DNS policy settings

### 🛠️ Debug Commands

```bash
# 🚀 Enter Pod for debugging
kubectl exec -n kube-system -it <pod-name> -- /bin/sh

# 🌐 Check network interfaces
ip addr show

# 🛣️ Check routing table
ip route show

# 🏓 Test network connectivity
ping <target-ip>

# 📊 Check network statistics
netstat -i
```

---

## 🌟 Best Practices

### 🏗️ Architecture Design

1. **Network Planning**
   - Reasonably plan Pod and Service CIDR ranges
   - Avoid conflicts with existing network addresses
   - Consider future expansion needs

2. **Security Configuration**
   - Enable network policies to control traffic
   - Configure appropriate RBAC permissions
   - Regularly rotate authentication keys

3. **Monitoring & Alerting**
   - Configure Prometheus metrics collection
   - Set up key metrics alerts
   - Establish log aggregation system

### 📈 Performance Optimization

1. **Resource Management**
   - Reasonably set CPU and memory limits
   - Monitor resource usage
   - Optimize garbage collection configuration

2. **Network Optimization**
   - Adjust MTU values for optimal transmission efficiency
   - Configure connection pool parameters
   - Enable network caching

---

## 📚 Resources

### 🔗 Official Resources

- [HeadCNI Project Homepage](https://github.com/binrc/headcni) 🏠
- [HeadScale Official Documentation](https://github.com/juanfont/headscale) 📖
- [Tailscale Official Documentation](https://tailscale.com/kb/) 📚
- [Kubernetes CNI Specification](https://github.com/containernetworking/cni) 📋

### 📖 Learning Resources

- [Kubernetes Network Model Explained](https://kubernetes.io/docs/concepts/services-networking/) 🌐
- [CNI Plugin Development Guide](https://github.com/containernetworking/cni/blob/main/SPEC.md) 🔧
- [Tailscale Network Principles](https://tailscale.com/blog/how-tailscale-works/) 🧠

### 🛠️ Tool Recommendations

- [Helm Package Manager](https://helm.sh/) 📦
- [k9s Cluster Management Tool](https://k9scli.io/) 🎮
- [Lens IDE](https://k8slens.dev/) 👁️

---

## 🤝 Contributing

We warmly welcome your contributions if you find configuration issues or have improvement suggestions:

### 🐛 Report Issues
1. Create an Issue on GitHub
2. Describe the problem and reproduction steps in detail
3. Provide relevant logs and configuration information

### 💡 Contribute Code
1. Fork the project repository
2. Create a feature branch
3. Submit a Pull Request
4. Participate in code review

### 💬 Community Discussion
- Join our Discord channel
- Participate in GitHub Discussions
- Follow project update dynamics

---

## 📄 License

This project uses the same license as the HeadCNI project.

<div align="center">

**Made with ❤️ by the HeadCNI Community**

[![GitHub stars](https://img.shields.io/github/stars/binrc/headcni?style=social)](https://github.com/binrc/headcni)
[![GitHub forks](https://img.shields.io/github/forks/binrc/headcni?style=social)](https://github.com/binrc/headcni)
[![GitHub issues](https://img.shields.io/github/issues/binrc/headcni)](https://github.com/binrc/headcni/issues)

</div> 