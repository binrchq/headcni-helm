# HeadCNI Helm Chart Examples

Complete deployment configuration examples for HeadCNI based on the actual Helm Chart structure.

[中文](README_CN.md) | English

## Overview

HeadCNI is a Kubernetes CNI plugin that integrates Headscale and Tailscale functionality, providing a modular and extensible networking solution for Kubernetes clusters.

## Files

### `headcni-k3s-allinone.yaml`
- **Purpose**: Complete HeadCNI deployment for k3s clusters
- **Features**: 
  - Uses k3s default CIDR ranges (Pod: `10.42.0.0/16`, Service: `10.43.0.0/16`)
  - Disables MagicDNS to avoid conflicts with k3s built-in DNS
  - Optimized for k3s environment
  - Complete RBAC configuration

### `headcni-k8s-allinone.yaml`
- **Purpose**: Complete HeadCNI deployment for standard Kubernetes clusters
- **Features**:
  - Uses standard k8s CIDR ranges (Pod: `10.244.0.0/16`, Service: `10.96.0.0/12`)
  - Enables MagicDNS for better DNS resolution
  - Production-ready configuration
  - Enhanced monitoring and logging

## Quick Start

### Prerequisites
- Kubernetes 1.20+ (supports k3s and standard k8s)
- Helm 3.0+
- HeadScale server
- HeadScale API authentication key

### Deployment

1. **Update Configuration**
   ```yaml
   config:
     headscale:
       url: "https://your-headscale-server:8080"
       authKey: "your-auth-key"
   ```

2. **Deploy to k3s**
   ```bash
   helm install headcni-k3s ./headcni-helm -f examples/headcni-k3s-allinone.yaml
   ```

3. **Deploy to Kubernetes**
   ```bash
   helm install headcni-k8s ./headcni-helm -f examples/headcni-k8s-allinone.yaml
   ```

## Configuration

### Core Settings

| Section | Key | Description |
|---------|-----|-------------|
| **HeadScale** | `url` | HeadScale server address |
| | `authKey` | Authentication key |
| | `timeout` | Connection timeout (default: 30s) |
| | `retries` | Retry count (default: 3) |
| **Tailscale** | `mode` | Operation mode: daemon/host |
| | `acceptDNS` | Accept HeadScale DNS settings |
| | `interfaceName` | Tailscale interface name |
| | `tags` | Node tags for access control |
| **Network** | `podCIDR` | Cluster Pod CIDR range |
| | `serviceCIDR` | Cluster Service CIDR range |
| | `mtu` | Network interface MTU |
| | `enableNetworkPolicy` | Enable network policies |

## Verification

### Check Status
```bash
# Pod status
kubectl get pods -n kube-system -l app=headcni

# Logs
kubectl logs -n kube-system -l app=headcni

# DaemonSet status
kubectl get daemonset -n kube-system
```

### Network Configuration
```bash
# CNI config
kubectl exec -n kube-system -c headcni-daemon <pod-name> -- cat /etc/cni/net.d/10-headcni.conflist

# Tailscale status
kubectl exec -n kube-system -c headcni-daemon <pod-name> -- tailscale status
```

### Metrics
```bash
# Port forward
kubectl port-forward -n kube-system svc/headcni-metrics 9001:9001

# Access metrics
curl http://localhost:9001/metrics
```

## Troubleshooting

### Common Issues

**Pod won't start**
- Check HeadScale server connectivity
- Verify authentication key
- Check Pod events and logs

**Network issues**
- Verify CNI configuration
- Check Pod CIDR settings
- Confirm Tailscale interface status

**DNS problems**
- Check MagicDNS configuration
- Verify DNS server settings
- Review resolv.conf

### Debug Commands
```bash
# Enter Pod
kubectl exec -n kube-system -it <pod-name> -- /bin/sh

# Network interfaces
ip addr show

# Routing table
ip route show

# Connectivity test
ping <target-ip>
```

## Best Practices

### Security
- Enable network policies
- Configure appropriate RBAC permissions
- Rotate authentication keys regularly

### Performance
- Set reasonable resource limits
- Monitor resource usage
- Optimize MTU settings

### Monitoring
- Enable Prometheus metrics
- Set up alerts for key metrics
- Implement log aggregation

## Resources

- [HeadCNI Project](https://github.com/binrc/headcni)
- [HeadScale Documentation](https://github.com/juanfont/headscale)
- [Tailscale Documentation](https://tailscale.com/kb/)
- [Kubernetes CNI Spec](https://github.com/containernetworking/cni)

## Contributing

We welcome contributions! Please:
1. Report issues on GitHub
2. Submit pull requests
3. Join community discussions

---

*Made with ❤️ by the HeadCNI Community* 