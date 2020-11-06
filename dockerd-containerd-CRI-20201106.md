dockerd, containerd, CRI - 2020/11/06
===

###### tags: `tech workshop`

主要參考這部影片 [SDN x Cloud Native Meetup - Webinar 邱牛上菜 #6 CRI&OCI](https://www.youtube.com/watch?v=5JhQOjSSnzQ&ab_channel=CloudNativeTaiwan)


# How docker works

![](https://i.imgur.com/3BMqcrU.jpg)

- OCI https://ithelp.ithome.com.tw/articles/10216215

## docker client call remote docker server

- docker client default 是用 unix socket 的方式跟 docker server 溝通
    - local: `/var/run/docker.sock`
- https://unix.stackexchange.com/questions/498862/how-is-forwarding-can-be-also-done-through-unix-sockets-done
- circleCI 也是使用 remote docker server

1. use ssh to port-forwarding remote server unix socket

```
ssh yoctol@40.65.169.166 -L ./local_docker.sock:/var/run/docker.sock
```

2. use docker client call docker server using above unix socket creating by ssh

```
docker -H unix://local_docker.sock ps
```

# How kubernetes works

- How k8s works (v1)

![](https://i.imgur.com/72qdMyy.png)

- How k8s works (v2)

![](https://i.imgur.com/9xOlfED.png)

- How k8s works (v3)

![](https://i.imgur.com/x405nUY.png)

- Performance (v1 v.s. v3)

![](https://i.imgur.com/t0qNQCW.png)

- How k8s works (v4)

![](https://i.imgur.com/yKcWR1v.png)

- How k8s works (v1 -> v4)

![](https://i.imgur.com/X23Gmpv.jpg)

# docker, podman, ctr, crictl

![](https://i.imgur.com/yCT1oCl.jpg)


### ctr

- 當 cluster 的 docker, dockerd 出現問題的時候，可以改用 ctr debug，確定問題出在哪一層

### podman

- https://github.com/containers/podman
- [Podman: Simply put: alias docker=podman](https://podman.io/whatis.html)