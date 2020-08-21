node cluster - 2019/07/19
===

###### tags: `tech workshop`

node cluster v.s. pm2
https://medium.com/the-node-js-collection/switching-from-cluster-module-to-pm2-rabbitmq-in-node-js-d0cce5eb96f4


# node cluster 架構
![](https://i.imgur.com/9P6heTW.png)

- Pros:
    - Scales according to number of CPU cores available on your machine.
    - Easy to manage as there is no dependency on any other module/service.
    - Easy to implement process communication.
- Cons:
    - You take a hit on performance of the app if there are too many messages.
    - The implementation doesn’t appear to be the best for managing communication of dedicated workers.
    - You need to manage the process state by yourself (no automation).
    - You can’t start/stop/restart (sms/email/notif) workers without affecting app because they are coupled.

# pm2 + RabbitMQ

- or container + RabbitMQ

![](https://i.imgur.com/LK2SvAZ.png)

- Pros:
    - PM2 manages all processes.
    - It’s easy to start/stop/restart any worker.
    - Workers remain independent of each other.
    - The Node.js process is free of communication messages.
- Cons:
    - RabbitMQ is not that easy to implement in production.
    - Adds dependency of PM2 and RabbitMQ modules.
    - Need to manage/monitor RabbitMQ server along with PM2.