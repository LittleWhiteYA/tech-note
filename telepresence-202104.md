telepresence - 2021/4
===

###### tags: `tech workshop`

投影片介紹: https://docsend.com/view/h3zr885eiuqj94wi

## 換掉 cluster 上的 deployment

```bash=
$ telepresence --swap-deployment <deploy>:<container> \
  --expose 5000 \
  --run-shell
```

## 另外在 cluster 上開一個 pod 接到 local 環境

```bash=
$ telepresence --run-shell
```



## ref

- [Telepresence: 本地快速測試 Kubernetes 與 OpenShift 微服務](https://srcmesh.com/tw/blog/telepresence/)
- [Intro: Telepresence: Fast Local-to-Remote Development for Kubernetes - Daniel Bryant, Datawire](https://www.youtube.com/watch?v=9eyHSjbZwR8&ab_channel=CNCF%5BCloudNativeComputingFoundation%5D)
- [github](https://github.com/telepresenceio/telepresence)
- [官網](https://www.telepresence.io/)