AKS snapshots - 2020/05/22
===

###### tags: `tech workshop`

# 前情提要

我們現在的 AKS 上，對於有用到 mongodb 的服務如果要做備份都是使用 mongodump，但是 mongodump 常常會影響到 production 的服務

# 目標

- 在不影響服務的狀態下，做備份

## mongodump

- 除了 mongodump，有沒有其他的備份方式？
    - https://docs.mongodb.com/manual/core/backups/

- mongodump 的問題
    - Ops 的人可能不懂 mongodb
    - mongodump 會吃很多資源，進一步影響到正在跑的服務
    - 每次備份完讀寫進 Azure files 都花一筆錢

## 改成 snapshots

- snapshots
    - 其他公司目前了解下來都是用 snapshots 做備份
    - 備份速度上有巨大的差異
    - 災難還原很快，不用下載備份檔到 local，重新做 mongorestore
    - 對 production DB 的影響應該很小
    - 迭加 snapshots 版本上去更省錢、省空間
    - 缺點在於 AKS 對於 snapshots 支援不夠好，不能根據 snapshot 創出 pv 來使用，必須手動作這件事


## Azure 上的備份服務

- [Recovery Services vaults](https://docs.microsoft.com/zh-tw/azure/backup/backup-azure-recovery-services-vault-overview) - VM layer
- [Azure backup](https://docs.microsoft.com/zh-tw/azure/backup/backup-azure-vms-introduction) - VM layer
- [Azure snapshot](https://docs.microsoft.com/zh-tw/azure/virtual-machines/windows/snapshot-copy-managed-disk) - disk layer
- 只要在 Azure 上存在的服務，就可以透過 Azure shell 來做到自動化
    - 安全性問題
        - 做了 az login 的 pod 會擁有整個 Azure resource group 的權限
        - 可以透過 https://github.com/Azure/aad-pod-identity 稍微改善

### az login

- 透過[這篇](https://docs.microsoft.com/zh-tw/cli/azure/create-an-azure-service-principal-azure-cli)，可以拿到 appId、tenant、password，就可以使用 az login 了
- 可以用 `az ad sp credential list --id <appId>` 方式，得到相關資訊

## mongodb on AKS

- 那在 AKS 上面怎麼做？
    - k8s 的基礎是 container 而不是 VM
    - 這樣就只剩 Azure snapshot 可以用

- 問了 AKS support 後的回答跟上面的研究結論一致
    - Azure Disk 在 AKS 裡推薦使用的 backup 的方式是 az snapshot：https://docs.microsoft.com/en-us/azure/aks/azure-disks-dynamic-pv#back-up-a-persistent-volume
    - 您可以嘗試第三方的 Veloro：https://github.com/vmware-tanzu/velero
    - 1.17 版本以上的 kubernetes 還可以支持 azure disk snapshot 功能：https://github.com/kubernetes-sigs/azuredisk-csi-driver/tree/master/deploy/example/snapshot
 

## 實做在 AKS 上做 Azure snapshots 做備份、還原

### 備份

- 開一個 cronjob 會定期跑 `az snapshot create`，就會得到一個 Azure snapshot

### 拿 snapshot 還原 disk 資料

- [AKS 使用 snapshots](https://github.com/MicrosoftDocs/azure-docs/blob/master/articles/aks/azure-disks-dynamic-pv.md)
- 這邊有個問題是，AKS 不支援直接透過 snapshot create persistent volume

## k8s 1.17 支援

- https://kubernetes.io/blog/2019/12/09/kubernetes-1-17-feature-cis-volume-snapshot-beta/
- https://github.com/kubernetes-sigs/azuredisk-csi-driver/tree/master/deploy/example/snapshot

## 指令

