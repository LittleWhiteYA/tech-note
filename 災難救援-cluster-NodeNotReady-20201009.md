災難救援 cluster NodeNotReady - 2020/10/09
===

###### tags: `tech workshop`

# 時間流水帳

- 大概在 10/09 晚上 10 點的時候打開電腦，發現 understood 壞掉了

![](https://i.imgur.com/fHIpddu.png)

- 開始檢查 cluster 狀況
    - `$ kubectl get pod`

![](https://i.imgur.com/GVhNLcb.png)

- 因為看到了很多 crashLoopBackOff 和 Terminating，直覺想到可能是 node 的問題
    - `$ kubectl get node`

![](https://i.imgur.com/6CVikbH.png)

- 看到 NotReady，先執行 `$ kubectl describe node xxx` 觀察問題

![](https://i.imgur.com/zwyr5JK.png)

- 重點在後面那個 `Kubelet stopped posting node status`

![](https://i.imgur.com/fHIpddu.png)

- 沒看過的問題，依照以往經驗，先重開 VM 試試看吧

![](https://i.imgur.com/AHHMtOj.png)

- 嗯... 發現重開沒效... 死定了

![](https://i.imgur.com/STZCXBt.png)

- 追查過程發現有幾個地方有顯示出問題在幾點的時候發生的

    - Azure
![](https://i.imgur.com/mWW1u5n.png)

    - grafana (前提是 prometheus server 不是剛好開在掛的那台上)

![](https://i.imgur.com/eCCS29p.png)

- 想到：目前問題暫時無法解決，首要目標是先復原服務

- 這時遇到幾個問題
    1. 我們的 node 資源用量一直都很緊繃，剩下的 3 台機器沒辦法負荷原本 4 台機器的服務 (主要是 requests 整體要求佔比太高)
    2. 假設資源足夠了，原本掛在那台 node 上的服務 (e.g. database, server) 一直保持 Terminating，怎辦？

### 剩下的 3 台機器沒辦法負荷原本 4 台機器的服務

- 解法 1: 把一些服務的 pod 數量調降或是關掉，調降 pod 裡面的 requests，讓剩下 3 台機器可以容納全部服務的 requests
    - 如果沒有 Azure 權限的人可以操作，但需要大量的手動

- 解法 2: scale node 數量
    - 這次運用的方式，較簡單

### 原本掛在壞掉那台 node 上的服務 (e.g. database, server) 一直保持 Terminating，怎辦？

- 如果不是 database 的 pod 就直接砍掉，應該沒什麼其他疑慮
- 如果是 database 的 pod ..... 就必須砍掉 pod 後，手動把掛在 VM 上的 disk detach 
![](https://i.imgur.com/Ah5LoGd.png)
    - 如果沒有 detach，新開的 pod 會噴出類似的 error

![](https://i.imgur.com/HaNDrzZ.png)


# trace error

-> debug kubelet

- https://ithelp.ithome.com.tw/articles/10209357
- 發現 kubelet 打不開

-> 試試看 docker 好了

- docker ps 沒反應？？？
- 從這裡推測應該是 docker 的問題

-> 試試看 `sudo service docker start`

![](https://i.imgur.com/K0yXeB0.png)

- 看起來是 docker server 直接壞掉
- 關鍵字： `Result: start-limit-hit`

-> 試試看直接執行 `dockerd`

![](https://i.imgur.com/XRrsySF.png)

- 關鍵字：
```
prior storage driver overlay2 failed: lstat /var/lib/docker/overlay2/4c4dbc8a262d6c31b1912d6191ed43d06f9de17eb1a0a5a4fddf0192eccf8727-init: structure needs cleaning 
```
- 看起來找到原因了
- 後來再深入研究發現應該是 disk 某個地方壞掉了
    - https://unix.stackexchange.com/questions/330742/cannot-remove-file-structure-needs-cleaning

-> 修 disk

- 先用[這篇](https://blog.nillsf.com/index.php/2020/06/22/vm-broken-use-os-disk-swap-in-azure-to-fix-and-restore/)把產生壞掉的 disk，並綁在另外一台 VM 上開始修
- 用 fdisk + e2fsck 解決
- 不小心就好ㄌ

# 後續

## 今天的問題有辦法避免嗎？

- 好像沒辦法，因為是 kubelet 壞掉，在那台機器上的 pod 會卡在 terminating 狀態，而沒辦法移到其他 node 上

## 假設今天的問題再發生一次最好、最快的解決方法是啥？

- 前提：由於在 k8s 的架構下，每個 node 都是等價的
- 直接把 node 砍掉，然後通知 AKS 服務重啟新的 node (使用 azure cli `az resource update`)，或是透過更改 AKS nodepool 的 scale 數量來更新 AKS 的 node 數量
    - by Azure support team