back pressure - 2020/08/21
===

###### tags: `tech workshop`

# 為什麼會看到 back pressure？

- 主要是研究 analysis flask 跟 asyncio 的時候注意到的


# flask 作者怎麼看待 asyncio？

[Flask 作者 Armin Ronacher：我不觉得有异步压力](https://zhuanlan.zhihu.com/p/102307133)

- 实际上，承认失败（你超负载了）比假装可运作并持续保持缓冲状态要好，因为到了某个时候，它只会令情况变得更糟


# 什麼是 back pressure？

[I Love Lucy](https://www.youtube.com/watch?v=K3axU2b0dDk)
![](https://i.imgur.com/rRBbBWI.png)



[Backpressure explained — the resisted flow of data through software](https://medium.com/@jayphelps/backpressure-explained-the-flow-of-data-through-software-2350b3e77ce7)

![](https://i.imgur.com/jYZ7czW.png)


[如何形象的描述反应式编程中的背压(Backpressure)机制？](https://www.zhihu.com/question/49618581/answer/117107570)

[数据流中的积压问题](https://nodejs.org/zh-cn/docs/guides/backpressuring-in-streams/)

# nodejs example

[Using Transform Streams To Manage Backpressure For Asynchronous Tasks In Node.js](https://www.bennadel.com/blog/3236-using-transform-streams-to-manage-backpressure-for-asynchronous-tasks-in-node-js.htm)