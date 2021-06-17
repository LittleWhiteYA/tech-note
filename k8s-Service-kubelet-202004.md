k8s Service, kubelet- 2020/4
===

###### tags: `tech workshop`

# kubelet

1. debug kubelet
進到 master or agent node, 執行
```
$ jouralctl -u kubelet 
```

2. restart kubelet service

```
$ systemctl restart kubelet
```
# 關於 Service

- part I - [What Is Service?](https://www.hwchiu.com/kubernetes-service-i.html)
- part II - [How to Implement Kubernetes Service - ClusterIP](https://www.hwchiu.com/kubernetes-service-ii.html)
- part III - [How to Implement Kubernetes Service - NodePort](https://www.hwchiu.com/kubernetes-service-iii.html)

> 以上會細到講解 k8s 裡面是怎麼透過 iptables 傳送封包的