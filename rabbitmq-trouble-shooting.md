rabbitmq 壞掉了 - 2020/03/16
===

###### tags: `tech workshop`

# 目標

- 假設今天有資料的 pod (postgresql, mongodb, rabbitmq) 壞掉了，知道要如何復原、除錯

## 以復原這次 rabbitmq 壞掉的情形舉例

### 觀察 logs
- 用 `kubectl logs <pod-name>` 觀察 pod logs

```

BOOT FAILED
===========

Error description:
    init:do_boot/3
    init:start_em/1
    rabbit:start_it/1 line 475
    rabbit:'-boot/0-fun-0-'/0 line 320
    rabbit_node_monitor:prepare_cluster_status_files/0 line 130
    rabbit_node_monitor:write_cluster_status/1 line 144
throw:{error,{could_not_write_file,"/var/lib/rabbitmq/mnesia/rabbit@rabbitmq-rabbitmq-ha-0.rabbitmq-rabbitmq-ha-discovery.default.svc.cluster.local/cluster_nodes.config",
                                   enospc}}
Log file(s) (may contain more information):
   <stdout>

2019-12-06 07:50:23.185 [error] <0.8.0> 
Error description:
    init:do_boot/3
    init:start_em/1
    rabbit:start_it/1 line 475
    rabbit:'-boot/0-fun-0-'/0 line 320
    rabbit_node_monitor:prepare_cluster_status_files/0 line 130
    rabbit_node_monitor:write_cluster_status/1 line 144
throw:{error,{could_not_write_file,"/var/lib/rabbitmq/mnesia/rabbit@rabbitmq-rabbitmq-ha-0.rabbitmq-rabbitmq-ha-discovery.default.svc.cluster.local/cluster_nodes.config",
                                   enospc}}
Log file(s) (may contain more information):
   <stdout>
{"init terminating in do_boot",{error,{could_not_write_file,"/var/lib/rabbitmq/mnesia/rabbit@rabbitmq-rabbitmq-ha-0.rabbitmq-rabbitmq-ha-discovery.default.svc.cluster.local/cluster_nodes.config",enospc}}}
init terminating in do_boot ({error,{could_not_write_file,/var/lib/rabbitmq/mnesia/rabbit@rabbitmq-rabbitmq-ha-0.rabbitmq-rabbitmq-ha-discovery.default.svc.cluster.local/cluster_nodes.config,enospc}})

Crash dump is being written to: /var/log/rabbitmq/erl_crash.dump...done
```

- `could_not_write_file`，可能可以猜測是 disk 的問題
- 如何確定是 disk 的問題？還是 rabbitmq 的問題？

### 換掉 pod image

- 把 pod image 換成一些基本的 image (e.g. ubuntu, busybox)
- 等 pod 正常啟動後，就能用 `kubectl exec -it <pod> bash` 的方式進去 pod 裡面確認 disk 問題

```
root@test-rabbitmq:/# df -h
Filesystem      Size  Used Avail Use% Mounted on
overlay         125G   59G   66G  48% /
tmpfs            64M     0   64M   0% /dev
tmpfs           6.9G     0  6.9G   0% /sys/fs/cgroup
/dev/sda1       125G   59G   66G  48% /etc/hosts
shm              64M     0   64M   0% /dev/shm
/dev/sdi        3.9G  3.8G     0 100% /var/lib/rabbitmq
tmpfs           6.9G   12K  6.9G   1% /run/secrets/kubernetes.io/serviceaccount
tmpfs           6.9G     0  6.9G   0% /proc/acpi
tmpfs           6.9G     0  6.9G   0% /proc/scsi
tmpfs           6.9G     0  6.9G   0% /sys/firmware
```

- 以這次的例子蠻明顯應該是 disk 容量滿了
- 如果是沒有太大掉資料的問題 ( 這次的 rabbbitmq 情形就是這樣)，那要如何快速復原？

### 關掉 pod、掛上一個新的 disk

- 把跟 pod 綁上的 pvc 砍掉，讓 pod 自動重新 create 一個新的 pv (disk)
- 暫時復原，但是資料全部變不見

### 要如何拿壞掉的 disk 來驗屍

- 這邊以牙醫通的 pv 為例，被拔掉 pvc 的 pv 狀態會從 Bound -> Released

```
$ k get pv pvc-c5a58d98-f843-11e8-893a-aaf5b52f6567
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS     CLAIM                           STORAGECLASS   REASON   AGE
pvc-c5a58d98-f843-11e8-893a-aaf5b52f6567   8Gi        RWO            Retain           Released   default/dentaltw-prod-mongodb   db                      366d
```

- 把 pvc 的相關資訊從 pv config 上拿掉，status 就會從 Released -> Available

```diff
$ k get pv pvc-c5a58d98-f843-11e8-893a-aaf5b52f6567 -o yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  annotations:
    pv.kubernetes.io/bound-by-controller: "yes"
    pv.kubernetes.io/provisioned-by: kubernetes.io/azure-disk
    volumehelper.VolumeDynamicallyCreatedByKey: azure-disk-dynamic-provisioner
  creationTimestamp: "2018-12-05T04:11:13Z"
  finalizers:
  - kubernetes.io/pv-protection
  name: pvc-c5a58d98-f843-11e8-893a-aaf5b52f6567
  resourceVersion: "52690936"
  selfLink: /api/v1/persistentvolumes/pvc-c5a58d98-f843-11e8-893a-aaf5b52f6567
  uid: cdbcb40c-f843-11e8-893a-aaf5b52f6567
spec:
  accessModes:
  - ReadWriteOnce
  azureDisk:
    cachingMode: None
    diskName: kubernetes-dynamic-pvc-c5a58d98-f843-11e8-893a-aaf5b52f6567
    diskURI: /subscriptions/3f9f53cd-7cee-491f-b25d-f4944b7e3767/resourceGroups/MC_yoctol-sea-aks-group_yoctol-sea-1_southeastasia/providers/Microsoft.Compute/disks/kubernetes-dynamic-pvc-c5a58d98-f843-11e8-893a-aaf5b52f6567
    fsType: ""
    kind: Managed
    readOnly: false
  capacity:
    storage: 8Gi
-  claimRef:
-    apiVersion: v1
-    kind: PersistentVolumeClaim
-    name: dentaltw-prod-mongodb
-    namespace: default
-    resourceVersion: "2472351"
-    uid: c5a58d98-f843-11e8-893a-aaf5b52f6567
  persistentVolumeReclaimPolicy: Retain
  storageClassName: db
  volumeMode: Filesystem
status:
  phase: Released

```

- create new pvc with existed pv

```
apiVersion: v1                                        
kind: PersistentVolumeClaim
metadata:
  name: rabbitmq-old
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 4Gi
  storageClassName: db
  volumeName: <pv name>
```

- create new pod with new custom pvc

```
apiVersion: v1
kind: Pod
metadata:
  name: test-rabbitmq
spec:
  hostAliases:
  - ip: "127.0.0.1"
    hostnames:
    - rabbitmq-rabbitmq-ha-0.rabbitmq-rabbitmq-ha-discovery.default.svc.cluster.local
  containers:
  - name: rabbitmq-container
    image: rabbitmq:3.7.12-alpine
    # image: ubuntu
    # tty: true
    volumeMounts:
    - mountPath: /var/lib/rabbitmq
      name: data
    env:
    - name: RABBITMQ_USE_LONGNAME
      value: "true"
    - name: RABBITMQ_NODENAME
      value: rabbit@rabbitmq-rabbitmq-ha-0.rabbitmq-rabbitmq-ha-discovery.default.svc.cluster.local
    - name: RABBITMQ_ERLANG_COOKIE
      valueFrom:
        secretKeyRef:
          key: rabbitmq-erlang-cookie
          name: rabbitmq-rabbitmq-ha
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: rabbitmq-old                                                                  
```

- 也可以把 k8s 上的資料 cp 到 local 模擬環境

### 如果是有資料不能掉的問題，那要怎麼辦？

- 我還不知道ㄏㄏ
- 需要監控所有的 disk usage 
- 需要有人幫忙


### AKS resize pvc size
https://github.com/kubernetes/kubernetes/issues/68427