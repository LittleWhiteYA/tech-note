graphQL Cache - 2020/03/11
===

###### tags: `tech workshop`

# 目標

如何利用 cache 加快 seeker query 的時間

- reference
    - https://www.youtube.com/watch?v=4lqEXYHPmVk&list=PLpi1lPB6opQyraZSmwFre_FpL00_3nTzV
    - https://www.youtube.com/watch?v=CV3puKM_G14&list=PLpi1lPB6opQyraZSmwFre_FpL00_3nTzV


## Browser, CDN, Reverse Proxy, Server-side Cache

![](https://i.imgur.com/dcS0bte.png)


## Cache with GraphQL

### POST /graphql

![](https://i.imgur.com/TAFbnSh.png)

![](https://i.imgur.com/SX7SVCO.png)
 
- 其實 HTTP spec 沒有限制 POST method 不能 cache，只是在 cache 的地方 (E.g. nginx) 要設定好
 
 但是
 
 ![](https://i.imgur.com/54kGFil.png)

- 意思是說幾乎全部的 cache 預設都不支援 POST

### change POST to GET /graphql

![](https://i.imgur.com/nI86hR8.png)

![](https://i.imgur.com/lO6pdVh.png)

- 如何避免 URI 過長的問題？
    - ![](https://i.imgur.com/aYm8wCv.png)

    - persisted queries
    - https://github.com/apollographql/apollo-link-persisted-queries
    - ![](https://i.imgur.com/YnH3fyJ.png)
    - ![](https://i.imgur.com/yc7vYHS.png)

### graphQL cache 相比 REST 的限制

![](https://i.imgur.com/5elf1KC.png)

![](https://i.imgur.com/h6G00Qs.png)

- 在上面的 query，即便有共用 friends 的 name、birthday 欄位，依舊會被認為是不同的 query 而無法 cache

- 在彈性上做取捨

![](https://i.imgur.com/CMZgZPu.png)

![](https://i.imgur.com/bzDEtyH.png)

## Caching with Apollo Server

https://www.apollographql.com/docs/apollo-server/performance/caching/

https://github.com/apollographql/apollo-server/tree/master/packages/apollo-server-plugin-response-cache

demo

![](https://i.imgur.com/wVAavfy.png)



## Cache-Control

https://blog.techbridge.cc/2017/06/17/cache-introduction/

header 有了 Cache-Control 就能在上面不同的層級做 cache