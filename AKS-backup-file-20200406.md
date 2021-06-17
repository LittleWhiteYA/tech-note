AKS backup file - 2020/04/06
===

###### tags: `tech workshop`

手把手，帶你走 

pv = persistentVolume
pvc = persistenVolumeClaim

> pv: 就是硬碟，k8s 看到 pvc 就會開一個 pv
> pvc: 就是「我要硬碟」這件事情

# 先找到 pvc 對應到 pv

- `kubectl get pvc`

```
...
page-bindings-mongodb           Bound     pvc-dea55502-8808-11e8-9583-000d3aa1273b   8Gi        RWO            db             11d
poc-calvin-mongodb              Bound     pvc-58aa4134-04a3-11e8-b495-000d3aa1d0a1   8Gi        RWO            db             179d
understood-backup-pgbackup      Bound     pvc-dccc93f0-de5b-11e7-9746-000d3aa1273b   4Gi        RWX            files          227d
...
```

- 中間顯示的 volume 就是 pv name
- `kubectl describe pv xxx`

# 拿到 DB 硬碟

## 舊版 ACS
- 觀察 DiskURI，找出藏在裡面的 storage account name
- https://p3494433440.blob.core.windows.net // p3494433440 就是 storage account name
- 去 Azure 找到 storage account(綠色 icon, 非 classic) -> 找到存在裡面的 DB 硬碟

## 新版 AKS
- 拿到 DiskName
- 去 Azure 找到 disk (磁碟) -> 找到 diskName

# 從 portal UI 去 mount 上面找到的硬碟到某台 VM 上某個地方

- 在 Azure 上找到 VM
- 新增 data disk，選你要 mount 的 DB disk
- 要記得 save

# 拿到 key

- 現在 acs 的 cluster key 放在 deployer 機器上
- 進到 deployer || 創一個帳號密碼，然後從 master VM 進去每一個 node

- `ssh -i xxx_rsa yoctol@10.240.0.x`

# ssh 進 VM

- use `lsblk`, `mount`, `df -h`
- lsblk 會看到被掛上的 disk
- 這時候仔細找，會發現一個沒有掛載路徑的 disk
- 94 他
- 看到被掛上的 disk 後，把他掛到 VM 上的某個路徑
    - `sudo mount -t ext4 <source> <des>`
    - [參考](https://askubuntu.com/questions/177825/how-to-mount-an-external-hdd)

# 用被掛上的資料夾當 base，跑一個 DB container

- `docker run -p 27017:27017 -v /<mount path>/data:/data/db --name mongo-test mongo:3.6`，[參考](https://stackoverflow.com/questions/35400740/how-to-set-docker-mongo-data-volume)
- dump database and restore to new container