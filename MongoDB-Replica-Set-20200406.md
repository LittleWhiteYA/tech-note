MongoDB Replica Set - 2020/04/06
===

###### tags: `tech workshop`


## 原理
https://docs.mongodb.com/manual/replication/
![](https://i.imgur.com/HBd3HjW.png)

正常情況對 Primary 節點進行讀寫，Secondary 會複製 Primary 的 oplog，並在自己的 data set 裡面再執行一次。

![](https://i.imgur.com/iTWU4j8.png)

這些節點彼此透過 Heartbeat (每隔一段時間試探節點是否存活的方式) 檢測狀態

![](https://i.imgur.com/Dj30mny.png)

如果當 primary 節點如果在預設 10 秒內無法和其它節點進行通信，這系統會自動從 secondary 節點投票中選取一個當主節點。

## 讓 Secondary 也能讀取資料

- Secondary 預設是不允許進行資料查詢的，因為**非同步複製**的關係，資料有可能在查詢時，還是舊的資料
- 如果能容忍這種非同步的錯誤，讓 Secondary 也能進行資料查詢，對系統來說，可達到讀寫分離，可以減輕 Primary 的 loading。

## Demo

- docker 跑起來
- replica set 設定

```
$ docker network create mongo-cluster666
```

透過 docker network，讓在 network 裡面的 mongoDB 彼此互相可見，並且在連其他 replicaSet 的時候可以根據 container name 直接 mapping hostname

```
$ docker run -d \
-p 30010:27017 \
--name mongo1 \
--net mongo-cluster666 \
mongo mongod --replSet mongo-set2
```

參考: https://gist.github.com/oleurud/d9629ef197d8dab682f9035f4bb29065

## rs
- rs.config 看設定檔
- rs.reconfig 修改
- rs.add 新增
- rs.remove 刪除

## 選舉

- 選擇主節點需要由**大多數 (一半以上的成員)** 決定
- 主節點只有在得到大多數支持時才能繼續作為主節點
- [WHY WE NEED ODD NUMBER NODES IN RS](http://dbversity.com/mongodb-why-we-need-odd-number-nodes-in-rs/)
![](https://i.imgur.com/gYD9mLg.png)

![](https://i.imgur.com/kHhUph0.png)

## 同步

- 根據操作日誌 oplog
![](https://i.imgur.com/bzvZ8oY.png)


## rollback

![](https://i.imgur.com/OOJQ1HP.png)

**左邊 primary** Op 126 還沒備份到**右邊 secondary** 的時候，中間網路斷掉了

![](https://i.imgur.com/Ra2l85O.png)

A、B 會從 oplog 中尋找左右兩邊共同的操作點，進行 rollback


## 如何透過 mongo connection string URI 做到讀寫分離

- https://docs.mongodb.com/manual/reference/connection-string/
- https://stackoverflow.com/questions/47998855/connect-to-mongodb-replica-set-running-inside-docker-with-java-windows
- MongoClient 如何解析 primary 和 secondary 的 DNS
    - https://github.com/mongodb/specifications/blob/master/source/server-discovery-and-monitoring/server-discovery-and-monitoring.rst#clients-use-the-hostnames-listed-in-the-replica-set-config-not-the-seed-list



## 參考
- [MongoDB Replica Set 高可用性架構搭建](https://blog.toright.com/posts/4508/mongodb-replica-set-%E9%AB%98%E5%8F%AF%E7%94%A8%E6%80%A7%E6%9E%B6%E6%A7%8B%E6%90%AD%E5%BB%BA.html)
- [使用Docker建立MongoDB Cluster](https://ithelp.ithome.com.tw/articles/10187117)
