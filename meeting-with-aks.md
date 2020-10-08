meeting with aks - 2019
===

###### tags: `tech workshop`

- aks
    - 為了工程需求，單獨升級 VM 的情形，是正確的行為嗎？
        - 不正確的，之後會有 multiple pool 去選擇不同 size 的 VM 並 scale
    - ACS 的定位？
        - 因為 k8s 在大戰中獲勝了，所以 AKS 以 ACS 為 foundation 發展
    - ACS and AKS 具體差異是什麼？
        - AKS master node 是交由 Azure 管理，ACS 則是自己管理
        - aks 是由 Azure 來 handle 關於 upgrade, scale node 這件事 (master node 交給 aks 管)
    - azurefile storageClass 太慢了，而且讀寫花費很高？
        - 非常不建議用 azurefile，可以考慮用 minio 之類的

- helm
    - chart 的版本控制
        - 這是個問題，還沒解決
    - helm go template 不太好用
        - 在 helm v3 似乎會改進這問題
        - https://github.com/helm/community/blob/master/helm-v3/000-helm-v3.md

- k8s
    - k8s 擅長處理 stateless，那 stateful 呢？
        - Rita 和 Alex 建議是把 stateful 的東西拆出 aks，交由 Azure service 管理

- pod
    - 升級 VM，有可能會進一步優化到裡面運行的 pod 嗎？
        - 沒問
    - high I/O 的 service 下，我們觀察到增加 pod 數量似乎不一定會優化 response time and 降低 failure rate，原因？
        - 沒問
    - k8s hpa 只接受 auto-scale according to CPU usage，how about high I/O usage
        - 寫 custom 的 plugin 進 cluster，改寫 hpa 觀測的東西
    - 因為 disk 被綁在某台機器上，如果在 k8s delete pod 的話沒辦法完美轉移到其他 node
        - 所以建議不要用 azuredisk 和 azurefile
    - umanaged disk 和 managed disk 遇到的雷
        - 再次申明，不建議綁 DB 在 cluster 
    - database cluster 如果要建立在 aks 上面有什麼建議嗎？
        - 不建議放進 cluster 裡面
        - -> 因為如果建一個 db cluster 進 k8s cluster 會擔心因為 db loading 太多影響到其他 service 
        - -> 因為上面的原因，會想要另外開 VM 來放 db cluster 來隔離 service 和 db
        - -> 做到上面的原因，就必須請 DBA 來管理 database，花費會因此變很貴
        - -> 那還不如用 cloud 管理的 db server


- ACI
    - 他可以告訴 k8s 我要多增加一個 node 但是那個 node 會被產生在 ACI 管理的 cluster，完全跟現在的 k8s cluster 隔離，就不會影響到現在的 service
    - 他可以加開任何你想要的類型的 node，可能是 GPU 或是更高級的 VM
    - 假設有 task 跑起來，會 trigger 想要的 node Ready，並用 toleration 的機制限定想要的 task (可能是 ML training task) 才能進這個 node，並且在 task 結束後會自動關掉這個 node
    - 缺點是跑起來至少需要 8 分鐘 (node pull image 的時間) + N 分鐘 (training task pull image 的時間)
- minio
    - 取代 Azurefile，https://github.com/minio/minio
- virtual kubelet

azure cli 通常走在 azure portal 前面，所以如果覺得應該要有的功能 portal 沒有，可以去 az-cli 找找