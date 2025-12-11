---
title: Kubernetes node server 常見端口
menu:
  sidebar:
    name: Kubernetes node server 常見端口
    identifier: KubernetesPort
    weight: 20
    parent: devops
tags:
- Kubernetes
- devops
- container
---

# Kubernetes Node Server 常見端口及用途

## Control Plane (Master Node) 端口

Control Plane 節點運行管理 Kubernetes 集群所需的核心組件。

| 端口範圍 | 協議 | 組件 | 用途 | 備註 |
|---------|------|------|------|------|
| **6443** | TCP | kube-apiserver | Kubernetes API Server，所有 API 請求的入口 | 必須對所有節點開放 |
| **2379-2380** | TCP | etcd | etcd server 客戶端 API | 僅 Control Plane 節點間通信 |
| **10250** | TCP | kubelet | Kubelet API，由 API Server 調用 | 所有節點都需要 |
| **10251** | TCP | kube-scheduler | Scheduler 健康檢查端口 | 已棄用，改用 10259 (HTTPS) |
| **10252** | TCP | kube-controller-manager | Controller Manager 健康檢查端口 | 已棄用，改用 10257 (HTTPS) |
| **10255** | TCP | kubelet | 只讀 Kubelet API（未認證） | 建議禁用，安全風險 |
| **10257** | TCP | kube-controller-manager | Controller Manager 安全端口（HTTPS） | 新版本使用此端口 |
| **10259** | TCP | kube-scheduler | Scheduler 安全端口（HTTPS） | 新版本使用此端口 |

### 安全注意事項
- `6443` 是最關鍵的端口，應僅向授權用戶和節點開放
- etcd 端口 (`2379-2380`) 應僅限於 Control Plane 節點間通信
- 建議禁用 `10255` 端口，因為它提供未經身份驗證的訪問

---

## Worker Node 端口

Worker Node 運行實際的應用程序容器。

| 端口範圍 | 協議 | 組件 | 用途 | 備註 |
|---------|------|------|------|------|
| **10250** | TCP | kubelet | Kubelet API | 必須向 Control Plane 開放 |
| **10255** | TCP | kubelet | 只讀 Kubelet API（未認證） | 建議禁用 |
| **30000-32767** | TCP/UDP | NodePort Services | NodePort 服務端口範圍 | 可自定義範圍 |

### NodePort 說明
- NodePort 允許從集群外部訪問服務
- 默認範圍：`30000-32767`
- 可通過 API Server 的 `--service-node-port-range` 參數自定義

---

## 通用端口

這些端口在 Control Plane 和 Worker Node 上都可能使用。

| 端口範圍 | 協議 | 組件 | 用途 | 備註 |
|---------|------|------|------|------|
| **179** | TCP | Calico BGP | Calico 網絡插件 BGP 通信 | 僅在使用 Calico 時需要 |
| **4149** | TCP | Calico | Calico BGP 健康檢查 | 僅在使用 Calico 時需要 |
| **5473** | TCP | Calico Typha | Calico Typha 代理 | 大型集群推薦 |
| **9099** | TCP | Calico Felix | Calico Felix 健康檢查 | 僅在使用 Calico 時需要 |
| **10248** | TCP | kubelet | Kubelet 健康檢查端口 | localhost 訪問 |
| **10249** | TCP | kube-proxy | kube-proxy 健康檢查端口 | localhost 訪問 |
| **10256** | TCP | kube-proxy | kube-proxy 健康檢查（新版） | localhost 訪問 |

---

## 其他常用服務端口

這些是常見的 Kubernetes 生態系統組件使用的端口。

### Ingress Controller

| 端口 | 協議 | 組件 | 用途 |
|------|------|------|------|
| **80** | TCP | Ingress Controller | HTTP 流量 |
| **443** | TCP | Ingress Controller | HTTPS 流量 |
| **8080** | TCP | Ingress Controller | 管理/健康檢查端口 |
| **10254** | TCP | NGINX Ingress | 健康檢查端口 |

### Monitoring & Logging

| 端口 | 協議 | 組件 | 用途 |
|------|------|------|------|
| **9090** | TCP | Prometheus | Prometheus Server |
| **9100** | TCP | Node Exporter | 節點指標導出 |
| **9093** | TCP | Alertmanager | 告警管理 |
| **3000** | TCP | Grafana | 可視化儀表板 |
| **9200** | TCP | Elasticsearch | Elasticsearch API |
| **5601** | TCP | Kibana | Kibana Web UI |
| **24224** | TCP | Fluentd | 日誌收集 |

### Service Mesh (Istio)

| 端口 | 協議 | 組件 | 用途 |
|------|------|------|------|
| **15000** | TCP | Envoy Admin | Envoy 管理接口 |
| **15001** | TCP | Envoy Outbound | Envoy 出站流量 |
| **15006** | TCP | Envoy Inbound | Envoy 入站流量 |
| **15008** | TCP | HBONE | HBONE mTLS 隧道 |
| **15010** | TCP | Pilot | XDS 和 CA 服務 |
| **15012** | TCP | Pilot | XDS 和 CA 服務（TLS） |
| **15017** | TCP | Webhook | Admission Webhook |
| **15020** | TCP | Merged Metrics | 合併指標 |
| **15021** | TCP | Health Checks | 健康檢查 |
| **15090** | TCP | Envoy Prometheus | Envoy 指標 |

### Container Runtime

| 端口 | 協議 | 組件 | 用途 |
|------|------|------|------|
| **2375** | TCP | Docker | Docker API（未加密，不建議） |
| **2376** | TCP | Docker | Docker API（TLS） |
| **/var/run/docker.sock** | Unix Socket | Docker | Docker 本地 socket |
| **/run/containerd/containerd.sock** | Unix Socket | containerd | containerd socket |

### DNS

| 端口 | 協議 | 組件 | 用途 |
|------|------|------|------|
| **53** | TCP/UDP | CoreDNS | DNS 服務 |
| **9153** | TCP | CoreDNS | Metrics 端口 |

---

## 防火牆配置建議

### Control Plane 節點

**入站規則：**
```bash
# 允許 API Server
6443/tcp (from all nodes and authorized users)

# 允許 etcd
2379-2380/tcp (from control plane nodes only)

# 允許 kubelet
10250/tcp (from control plane nodes)

# 允許 scheduler 和 controller manager
10257/tcp, 10259/tcp (localhost or control plane nodes)
```

### Worker 節點

**入站規則：**
```bash
# 允許 kubelet
10250/tcp (from control plane nodes)

# 允許 NodePort 服務
30000-32767/tcp (from external or as needed)

# 允許 CNI 插件（例如 Calico）
179/tcp (from all nodes)
4789/udp (VXLAN, if using overlay network)
```

### 通用規則

**所有節點應允許：**
```bash
# 允許節點間通信（根據 CNI 需求）
All traffic between cluster nodes (optional, depends on CNI)

# 允許 SSH 管理
22/tcp (from management network)
```

---

## 端口檢查命令

### 檢查端口是否監聽

**Linux:**
```bash
# 使用 netstat
netstat -tuln | grep <port>

# 使用 ss
ss -tuln | grep <port>

# 使用 lsof
lsof -i :<port>
```

**Windows:**
```powershell
# 使用 netstat
netstat -ano | findstr :<port>

# 使用 PowerShell
Get-NetTCPConnection -LocalPort <port>
```

### 測試端口連接性

```bash
# 使用 telnet
telnet <host> <port>

# 使用 nc (netcat)
nc -zv <host> <port>

# 使用 curl (僅 HTTP/HTTPS)
curl -v https://<host>:<port>
```

### 檢查 Kubernetes 組件狀態

```bash
# 檢查所有組件
kubectl get componentstatuses

# 檢查節點狀態
kubectl get nodes

# 檢查所有 Pods
kubectl get pods --all-namespaces

# 檢查服務端口
kubectl get svc --all-namespaces
```

---

## 故障排除

### API Server 無法訪問 (6443)

1. 檢查 API Server 是否運行
```bash
systemctl status kube-apiserver
kubectl cluster-info
```

2. 檢查防火牆規則
```bash
sudo iptables -L -n | grep 6443
```

3. 檢查證書有效性
```bash
openssl s_client -connect <api-server>:6443
```

### etcd 連接問題 (2379-2380)

```bash
# 檢查 etcd 健康狀態
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  endpoint health
```

### Kubelet 問題 (10250)

```bash
# 檢查 kubelet 狀態
systemctl status kubelet

# 檢查 kubelet 日誌
journalctl -u kubelet -f

# 測試 kubelet API
curl -k https://localhost:10250/healthz
```

---

## 安全最佳實踐

1. **最小權限原則**
   - 僅開放必要的端口
   - 使用網絡策略限制 Pod 間通信

2. **加密通信**
   - 所有組件間使用 TLS 加密
   - 定期輪換證書

3. **防火牆配置**
   - 使用防火牆限制端口訪問
   - Control Plane 端口不應公開暴露到互聯網

4. **禁用不安全端口**
   - 禁用 kubelet 的只讀端口 (10255)
   - 不使用 Docker 未加密端口 (2375)

5. **監控和審計**
   - 監控異常端口訪問
   - 啟用 API Server 審計日誌

---

## 參考資源

- [Kubernetes 官方文檔 - 端口和協議](https://kubernetes.io/docs/reference/networking/ports-and-protocols/)
- [Kubernetes 安全最佳實踐](https://kubernetes.io/docs/concepts/security/security-checklist/)
- [CNI 插件文檔](https://www.cni.dev/)

---
