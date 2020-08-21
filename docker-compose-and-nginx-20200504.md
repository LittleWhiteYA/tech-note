docker-compose and nginx - 2020/05/04
===

###### tags: `tech workshop`

# on-premises 的目標
- 把整個 yoctol 整套服務在富邦跑起來
- data 必須有備份機制
- 網路限制
    - 服務全部會被限制在內網
    - 影響到 minio (圖片)、DNS
- 架一個 fakebon 來模擬 on-premises 環境
- DMZ

## 選擇用 docker-compose 的原因
- 不能用 k8s
- 有考慮以 docker swarm 當作替代方案，但是複雜度太高XD

## docker-compose 的缺點
- accounts、kurator ... 必須拆開寫在不同 config 檔
- 沒有類似 k8s 的 secret (docker swarm 有)、configmap
- 沒有類似 helm 能管理更新版本
- 沒有 autoscale、autorecover 機制 (docker swarm 有)

# infra docker-compose branch
- https://github.com/Yoctol/infra/compare/docker-compose

### docker-compose prototype
- https://github.com/Yoctol/infra/commit/f98c021ed73b1738204f49b3cb336bdcf6f2320a

### nginx reverse proxy 設定
- https://www.cnblogs.com/TonyYPZhang/p/5423149.html
- ![](https://i.imgur.com/CqIhKSS.png)

- https://github.com/Yoctol/infra/commit/99048a14cf04f0d5b8cee43f7c301f032655746f
- https://github.com/Yoctol/infra/commit/26f8d9c62973ef14fde6189c8d488543b8f6fb2e


### 跑 script 一鍵打包 image
- https://github.com/Yoctol/infra/commit/a8bb1d1e8e178980fbd06ea8e83279218b5a4476

### 抽出環境變數
- https://github.com/Yoctol/infra/commit/c69975b2e7e67ec64bd62bb37ca3144dcde4568d
- https://github.com/Yoctol/infra/commit/87c526d1152a44b9fbf9aa87dd7bd3740ef44053

### nginx 上傳檔案大小
- https://github.com/Yoctol/infra/commit/29d2866e45af4f3bae149e1a5cb645f66248225e

### local file system rathar than docker volume
- https://github.com/Yoctol/infra/commit/50b3c8b77333d6bfa170c42f1ad3cb6ab61382b5

### DMZ nginx
- https://github.com/Yoctol/infra/commit/18efd92ebe4b49df965519f31a8dc6cd254c2f37
- ![](https://i.imgur.com/9pYfKlw.png)


### 透過 nginx，讓 pg master 掛掉的時候還能維持服務
- https://github.com/Yoctol/infra/commit/a44535b2f603612a287df684c753e3e07ab84d49
- https://severalnines.com/blog/nginx-database-load-balancer-mysql-or-mariadb-galera-cluster
- https://blog.51cto.com/legehappy/2104515

學到的東西



### reference
- https://blog.51cto.com/wangwei007/1103727
- https://severalnines.com/blog/nginx-database-load-balancer-mysql-or-mariadb-galera-cluster
- https://blog.51cto.com/legehappy/2104515
- https://www.cnblogs.com/felixzh/p/8707102.html
- https://www.jianshu.com/p/5caa48664da5
- https://medium.com/@joatmon08/using-containers-to-learn-nginx-reverse-proxy-6be8ac75a757