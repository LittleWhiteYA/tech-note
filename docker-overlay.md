docker overlay - 2019/07/19
===

###### tags: `tech workshop`

# 目標

- 實做類似 k8s service 的效果，即便跨機器 (host)，也能透過 docker network 連到不同機器上的 container，而不需要 expose container port 等等的
- 在富邦正式環境會有 2 台機器的情形下或許幫的上忙

參考 [Kubernetes Service 概念詳解](https://tachingchen.com/tw/blog/kubernetes-service/)

# docker overlay

- https://docs.docker.com/network/
- **Overlay networks**: are best when you need containers running on different Docker hosts to communicate, or when multiple applications work together using swarm services.

- 使用 telnet (for TCP)、nc (for UDP) 等套件 debug 網路問題

- 主要確定:
    - 防火牆有沒有開通
    - consul 有沒有開
    - 其他 container 有沒有開
    - 網路 (可用 telnet、nc 測試) 有沒有通

參考:
- https://luppeng.wordpress.com/2016/05/03/setting-up-an-overlay-network-on-docker-without-swarm/
- https://luppeng.wordpress.com/2018/01/03/revisit-setting-up-an-overlay-network-on-docker-without--docker-swarm/
- https://ithelp.ithome.com.tw/articles/10193708

## key-value store - Consul

- https://www.ithome.com.tw/guest-post/99967


https://docs.docker.com/engine/reference/commandline/dockerd/