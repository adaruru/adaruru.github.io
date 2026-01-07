---
title: NFS
date: 2026-01-02T00:00:00+08:00
menu:
  sidebar:
    name: NFS
    identifier: NFS
    parent: devops
    weight: 20
tags:
- devops
- NFS
---

# NFS

1. Local-like File Access 類本地讀寫

   Network FileSystem 網路檔案系統，NFS 是一種讓多台電腦可以像操作本地硬碟一樣訪問遠端檔案的系統。

   是一種 RPC Service（Remote Procedure Call），當你用 NFS 讀寫檔案時，客戶端其實是在透過 RPC 向伺服器發送「打開檔案」「讀檔案」「寫檔案」這些遠端操作的請求，

2. Dynamic Port Allocation 動態端口

   容器配置不需要額外維護 port export，rpcbind 會自動協調服務所使用的 TCP/UDP port，並且同一個 network 內的容器可以直接訪問這些 port；若存在防火牆或跨主機通信時，則需要額外開通對應 port，配置相對複雜。

3. Server/Client 伺服器/客戶端

   因為是一種 RPC Service，所以會有兩種角色 Server、Client 

   就像 FileZilla 也是，你要遠端可以 FileZilla 連到，你需要在 target server 安裝 FileZilla server 或其他 ftp 類型 server on 著 listening ( 持續運行以監聽連線請求 )

   當 server 啟用時，可以讓服務跨 os、跨 server，同步讀取、寫入本地檔案。

```
┌─────────────────┐         網路          ┌─────────────────┐
│  NFS Server     │ <─────────────────>  │  NFS Client     │
│  (141)          │                      │  (142)          │
│                 │                      │                 │
│  /nfs (實體目錄) │ ─────匯出(export)──>  │  /nfs (掛載點)   │
│  chmod 777      │                      │  mount 後可存取  │
└─────────────────┘                      └─────────────────┘
```

## 安裝

### Windows 啟用 server/client

開啟或關閉 Windows 功能

![image-20251230163558311](/posts/devops/attach/NFS/image-20251230163558311.png) 

### Windows 啟動 Server/Client

```powershell
    # 安裝 NFS Client 功能
    Enable-WindowsOptionalFeature -FeatureName ServicesForNFS-ClientOnly -Online -All

    # 啟動 NFS Client 服務
    Start-Service -Name NfsClnt
    Set-Service -Name NfsClnt -StartupType Automatic

    # 啟動相關服務
    Start-Service -Name NfsRdr
    Set-Service -Name NfsRdr -StartupType Automatic

    # 檢查服務狀態
    Get-Service -Name NfsClnt, NfsRdr
    
    # 啟動 Client 掛載 Z:
    C:\Windows\System32\mount.exe -o anon \\192.168.168.101\nfs Z:
    # 啟動 Client 掛載 Y:
    C:\Windows\System32\mount.exe -o anon \\192.168.100.141\nfs Y:
    # 安全移除 Z: 掛載
    umount Z:
```

### Linux 安裝 server

```
apt install -y nfs-kernel-server nfs-common
```

設定腳本

bash /root/nfsServer.sh

```shell
echo "===== 執行腳本 $0 ====="
echo "完整路徑: $(readlink -f "$0")"
echo "檔名: $(basename "$0")"
echo "當前時間: $(date)"
echo "主機名稱: $(hostname)"

echo "Setting up NFS Server..."

# 安裝 NFS Server
if ! dpkg -l | grep -q nfs-kernel-server; then
	echo "Installing NFS packages..."
	apt update
	apt install -y nfs-kernel-server nfs-common
	echo "✓ NFS packages installed"
else
	echo "✓ NFS packages already installed"
fi

echo "建立 NFS 共享目錄 /nfs"
mkdir -p /nfs
echo "設定目錄權限為 777 (任何人可讀寫執行)..."
chmod -R 777 /nfs

# 配置 NFS 匯出
# all_squash 所有用戶都被映射為匿名用戶
echo "Configuring NFS exports..."
if ! grep -q "# NFS Server Shared Directory" /etc/exports; then
	cat >>/etc/exports <<'EOF'

# NFS Server Shared Directory
/nfs  192.168.100.0/24(rw,sync,no_subtree_check,no_root_squash) \
      192.168.168.0/24(rw,sync,no_subtree_check,no_root_squash)
EOF
	echo "✓ NFS export configured"
else
	echo "✓ NFS export already configured"
fi

# 應用 NFS 配置並啟動服務
echo "Starting NFS services..."
exportfs -ra
systemctl restart nfs-kernel-server
systemctl enable nfs-kernel-server
echo "✓ NFS Server started and enabled"

# 驗證 NFS 匯出
echo ""
echo "NFS Export Status:"
exportfs -v
echo ""
showmount -e localhost
echo "more option"
echo "
#1. 取消所有 export（會移除所有現有的 NFS 匯出）
exportfs -ua
# 2. 重新 export /etc/exports 內容
exportfs -ra
# 3. 檢查現有匯出
exportfs -v"

echo "===== 腳本執行完成 ====="
```

### Linux 安裝 client

```
apt install -y nfs-common
```

設定腳本，以輸入的第一個參數當作 Server IP，下面例子是在 142 安裝 client Listening 141 的 NFS Server

bash /root/nfsClientOn.sh 192.168.100.141

```shell
echo "===== 執行腳本 $0 ====="
echo "完整路徑: $(readlink -f "$0")"
echo "檔名: $(basename "$0")"
echo "當前時間: $(date)"
echo "主機名稱: $(hostname)"

echo "Setting up NFS Client..."
# 接收第一個參數作為 NFS Server IP，如果沒有提供則使用預設值
PRIMARY_IP="${1:-192.168.168.101}"
echo "NFS Server IP: ${PRIMARY_IP}"
NODE_IP=$(ip -4 addr show eth0 | grep -oP '(?<=inet\s)\d+(\.\d+){3}' | head -1)

# 安裝 NFS Client
if ! dpkg -l | grep -q nfs-common; then
	echo "Installing NFS client packages..."
	apt update
	apt install -y nfs-common
	echo "✓ NFS client packages installed"
else
	echo "✓ NFS client packages already installed"
fi

# 驗證可以看到 NFS Server 的匯出
echo ""
echo "Checking NFS Server exports from ${PRIMARY_IP}:"
showmount -e ${PRIMARY_IP}

# 建立掛載點
echo ""
echo "mkdir /nfs..."
mkdir -p /nfs

# 檢查是否已經掛載
if mountpoint -q /nfs; then
	echo "✓ /nfs is already mounted"
	umount /nfs
	echo "Unmounted existing /nfs to remount"
fi

# 掛載 NFS
echo ""
echo "Mounting NFS from ${PRIMARY_IP}:/nfs to /nfs..."
if mount -t nfs ${PRIMARY_IP}:/nfs /nfs; then
	echo "✓ NFS mount successful"

	# 測試讀寫
	echo "Testing read/write..."
	mkdir -p /nfs/data
	echo "test from node ${NODE_IP} at $(date)" >/nfs/data/test-${NODE_IP##*.}.txt
	cat /nfs/data/test-${NODE_IP##*.}.txt
	echo "✓ NFS read/write test passed"

	# 設定開機重新掛載
	echo "Configuring auto-mount on boot..."
	if ! grep -q "${PRIMARY_IP}:/nfs" /etc/fstab; then
		echo "${PRIMARY_IP}:/nfs  /nfs  nfs  defaults  0  0" >>/etc/fstab
		echo "✓ Added to /etc/fstab for auto-mount on boot"
	else
		echo "✓ Already configured in /etc/fstab"
	fi

	echo ""
	echo "Current mount status:"
	df -h | grep nfs
else
	echo "✗ NFS mount failed - check network connectivity and NFS Server status"
	exit 1
fi

echo "
#1. 取消所有 export（會移除所有現有的 NFS 匯出）
exportfs -ua
# 2. 重新 export /etc/exports 內容
exportfs -ra
# 3. 檢查現有匯出
exportfs -v"

echo "===== 腳本執行完成 ====="

```

### /etc/exports option

| 選項             | 說明                                                    |
| ---------------- | ------------------------------------------------------- |
| rw               | 允許讀寫 (read-write)                                   |
| ro               | 只允許讀取 (read-only)                                  |
| sync             | 同步寫入，數據立即寫到硬碟（較安全但慢）                |
| async            | 異步寫入，數據先存記憶體（較快但可能遺失）              |
| no_subtree_check | 不檢查子目錄，提升效能                                  |
| no_root_squash   | **Client 的 root 用戶 = Server 的 root 用戶**           |
| root_squash      | Client 的 root 用戶被映射為 nobody（預設）              |
| all_squash       | 所有用戶都被映射為 nobody（會導致其他 User 無法寫入！） |
| anonuid=1000     | 匿名用戶的 UID                                          |
| anongid=1000     | 匿名用戶的 GID                                          |

### 常用指令：

```bash
#1. 取消所有 export（會移除所有現有的 NFS 匯出）
exportfs -ua
# 2. 重新 export /etc/exports 內容
exportfs -ra
# 3. 檢查現有匯出
exportfs -v"

# Server 查看 NFS 日誌
journalctl -u nfs-kernel-server -f

# Server 確認 exports 設定
nano /etc/exports

# Client 查看掛載詳細信息
mount -v -t nfs 192.168.100.141:/nfs /nfs
# 查看 NFS 統計
nfsstat
# 測試網路連接
showmount -e 192.168.100.141

# client 確認目錄權限
ls -ld /nfs

# client 確認是否有掛載
mount | grep nfs
# 應該顯示：192.168.100.141:/nfs on /nfs type nfs ...

# Client 測試寫入權限
touch /nfs/test.txt
mkdir /nfs/testdir
```

## 權限問題解析

### 權限的三層檢查

```
Client 寫入文件
    ↓
1. NFS 選項檢查 (/etc/exports)
    ├─ rw/ro：是否允許寫入？
    ├─ no_root_squash：Client 的 root 保持 root 身份？
    └─ all_squash：是否把所有人都變成 nobody？
    ↓
2. 網路傳輸到 Server
    ↓
3. Server 本地文件系統權限檢查
    └─ chmod 777 /nfs：所有人可讀寫執行
```

1. 文件 owner 變成 nobody
   - 使用了 `all_squash`
2. Permission denied
   - Server 的 `/nfs` 權限不足:`chmod 777 /nfs`
   - 使用了 `root_squash` 且用 root 寫入: 改用 `no_root_squash`
   - 使用了 `all_squash`: ` 改用 `no_root_squash`

## NFS 使用限制

性能受網路延遲影響大、動態 port 管理複雜、防火牆與容器環境不友好、缺乏分散式鎖與一致性保證，並不適合作為應用程式的共享狀態存儲，僅適合存放靜態檔案或可容忍延遲的共享資料。

### Redis 共寫

或其他 in-memory cache / DB。

- Redis 的持久化

  持久化行為依賴本地一致性與 fsync

  RDB/AOF 寫磁碟時假設底層檔案系統能確保一致性與原子性

  NFS 上可能會增加資料不一致或丟失風險

- 網路文件系統在強一致性、cache flush 行為上與本地磁碟不同，可能造成 fsync 不如預期即時落盤或 metadata 操作不一致。
- Redis 持久化設計是針對本地檔案系統最佳化（fallocate/rename/fork+BGREWRITE），這些行為在 NFS 下可能不是原子或有延遲問題。
- 需要同步還是要回歸官方架構 replica 或 sentinal 等官方推薦作法。

### 避免高頻存取

NFS 適合共享檔案或備份，但必須明確區分「資料共用」與「資料持久化」用途

若要多主共寫，需搭配分布式鎖或專用分布式檔案系統（如 Ceph、GlusterFS）

不適合任何 DB 共寫

### 有條件的 log 共寫

- NFS 在 cache 與 metadata 刷新上會有短暫延遲。

NFS 適合低至中頻 append-only log 共寫，但對高頻、高可靠、嚴格順序要求的 log，不建議直接使用。

最佳實務是將 NFS 作為備份或聚合存儲，而非主要即時 log 寫入位置。

### 在 K8s 不可使用

- Pod IP 會變
- rpcbind 記錄的是 IP:port
- 動態 port 無法寫 NetworkPolicy
- NFS lock 會因 Pod restart 直接失效
- Stateful + Stateless Pod 混用是反模式

NFS 是為裸機與固定網路時代設計的共享狀態系統，而不是為 Stateless、Container、Cloud-Native 而生，是故 Kubernetes 官方不推薦 NFS 作為應用層共享狀態，
