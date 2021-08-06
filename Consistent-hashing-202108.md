Consistent hashing - 2021/8
===

###### tags: `tech workshop`

# 前言

## 情境 1：一般 cache

假如你有三個 Node {A, B, C}，然後有 { 1, 2, 3, 4, 5, 6 } 這六筆資料，一般而言我們會很直覺想把他們平均打散在 cache server 上面，所以會採用 hash function 為 `hash(key) % 3` 的方式選擇 Node，資料放置的地方就會如下

```
Node A: {1, 4}
Node B: {2, 5}
Node C: {3, 6}
```

### 增加節點

假如我現在想增加一個 Node，根據我們之前的演算法，hash function 就會變成 `hash(key) % 4`，資料放置的地方就會如下

```
Node A: {1, 5}
Node B: {2, 6}
Node C: {3}
Node D: {4}
```

以上面的例子，會使得資料 { 4, 5, 6 } 的 cache 失效

### 刪除節點

假如我現在想刪除一個 Node，根據我們之前的演算法，hash function 就會變成 value % 2，資料放置的地方就會如下

```
Node A: {1, 3, 5}
Node B: {2, 4, 6}
```

以上面的例子，會使得資料 { 3, 4, 5, 6 } 的 cache 失效

## 情境 2： data sharding

假設一個 database cluster 的資料被隨機分散到不同的 data node，E.g. 以時間為 sharding key，將資料分散儲存在不同 data node
- 當你要搜尋的資料是分散在不同 data node 上的時候，速度方面會比起資料都在同台 data node 上要來的慢許多
- 當一台 data node 因為某種原因壞掉的話，影響的層級會更廣

# 解決方法 - Consistent Hashing Algorithm

我們先想像有一個 hash values 的空間，圍成一個圓 (Consistent Hash Ring)

![](https://i.imgur.com/QQI2SEb.png)


假設有 4 個 object data (m1 - m4)，經過 hash 後對應到圓上的位置

![](https://i.imgur.com/dNfeEDa.png)

假設現在有 3 台機器 (t1 - t3)，也經過 hash (拿機器 IP 或唯一名稱為輸入) 後對應到圓形上的位置

![](https://i.imgur.com/75LewT5.png)

將資料按照順時針，儲存到遇到的機器上

![](https://i.imgur.com/bNNMoAZ.png)


## 增加機器數

增加機器 t4 後，資料儲存位置的改變

![](https://i.imgur.com/AI3X3Fs.png)


## 刪除機器

刪除機器 t1, t4 後，資料位置的變動

![](https://i.imgur.com/pfwSlGK.png)


## 存在的問題

當機器分佈的位置不平均的時候，會造成負載不平衡的情形

![](https://i.imgur.com/4nWCtmU.png)

### 解決方法 - 虛擬節點

當 Consistent Hash Ring 中的 indexes 數量變多，分佈不均勻的機會就會變小，負載不平衡的情形也會改善

![](https://i.imgur.com/N90xzpz.png)

## 時間 / 空間複雜度

n = node 數量

- 時間複雜度 (query & add/remove node)： worst case O(n)，因為有可能要繞一整個環才找得到 node
    - 如果用 binary search 的方式找節點可以優化到 O(log n)
- 空間複雜度： O(n)，要存全部節點的 index

# Jump Consistent Hashing

而 Google 在 2014 年發表出來的 Jump Consistent Hash 則給了更好的方案，O(1) 的空間使用率以及 O(logN) 的時間複雜度 (query only)，而且非常平均的打散：「A Fast, Minimal Memory, Consistent Hash Algorithm」。

## 時間 / 空間複雜度

n = node 數量

- 時間複雜度： O(log n)
- 空間複雜度： O(1)


# 參考

- [Consistent Hashing Algorithm: 應用情境、原理與實作範例](https://medium.com/@chyeh/consistent-hashing-algorithm-%E6%87%89%E7%94%A8%E6%83%85%E5%A2%83-%E5%8E%9F%E7%90%86%E8%88%87%E5%AF%A6%E4%BD%9C%E7%AF%84%E4%BE%8B-41fd16ad334a)
- [Consistent Hashing](https://kkc.github.io/2016/08/02/Consistent-Hashing/)
- [一致性哈希算法（consistent hashing）](https://zhuanlan.zhihu.com/p/129049724)
- [Google 提出的 Jump Consistent Hash](https://blog.gslin.org/archives/2016/07/26/6688/google-%E6%8F%90%E5%87%BA%E7%9A%84-jump-consistent-hash/https://blog.gslin.org/archives/2016/07/26/6688/google-%E6%8F%90%E5%87%BA%E7%9A%84-jump-consistent-hash/)
- [Consistent Hashing & Jump Consistent Hashing](https://ithelp.ithome.com.tw/articles/10215047)
- [Data Partitioning - Distributed Hash Table and Consistent Hashing - CHORD(上)](https://ithelp.ithome.com.tw/articles/10226592)