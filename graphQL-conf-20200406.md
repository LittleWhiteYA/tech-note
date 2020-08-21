graphQL conf. - 2020/04/06
===

###### tags: `tech workshop`

# 目標

- graphql resolver info 可以拿來幹嘛？
- graphql cache 進階


## graphql resolver info

[What About the Database? How to Write Efficient Queries for GraphQL Resolvers](https://www.youtube.com/watch?v=XIlPRypoNOg&list=PLpi1lPB6opQyraZSmwFre_FpL00_3nTzV)

![](https://i.imgur.com/eGZEldY.png)

### Scoped Selections

![](https://i.imgur.com/7SQp2wk.png)

![](https://i.imgur.com/c27blNy.png)

![](https://i.imgur.com/c913hhx.png)

- 只 select 我們需要的欄位就好


![](https://i.imgur.com/Zsh48Rt.png)

GraphQL AST
- https://medium.com/@adamhannigan81/understanding-the-graphql-ast-f7f7b8e62aa4
- https://blog.smartive.ch/advanced-graphql-patterns-embrace-the-ast-4929647c5bd3

![](https://i.imgur.com/BqTAqWv.png)

https://github.com/Yoctol/kurator/blob/8c183541b516564b115e1a6d6c518805b8c93ac6/server/src/schema/kuratorSchema/project/project.js#L630-L635

- kurator 其實有實做

```js
    productionFacebookPage: async (project, args, ctx, info) => {
      ...
      const allFields = graphqlFields(
        info,
        {},
        { excludedFields: ['__typename'] }
      );
      const fieldsKeyList = Object.keys(allFields);
      ...
    }
```

![](https://i.imgur.com/jjHndry.png)

### 2, 3, 4

2 - 概念跟 1 很像，只是取出 sub-selections
3 - 在大部分專案已經有實做
4 - https://github.com/stems/graphql-depth-limit


## graphql cache

- 最近實作的 cache PR

![](https://i.imgur.com/nFCRmZq.png)


https://github.com/Yoctol/kurator/pull/6226

```js 

export class CacheDirective extends SchemaDirectiveVisitor {
  visitFieldDefinition(field) {
    const { resolve = defaultFieldResolver } = field;

    // eslint-disable-next-line no-param-reassign
    field.resolve = async (obj, args, ctx, info) => {
      // ttl set 10 secs by default
      const { ttl = 10 } = this.args;

      const cacheKey = {
        userId: ctx.session.userId,
        fieldName: info.fieldName,
        path: info.path,
        variableValues: info.variableValues,
      };

      const cacheKeyString = generateCacheKeyString(cacheKey);

      const cache = await createCache();

      function cacheSetInBackground(key, value, _ttl) {
        return cache.set(key, JSON.stringify(value), { ttl: _ttl });
      }

      const resultFromCache = await cache.get(cacheKeyString);

      let result;
      if (resultFromCache) {
        result = JSON.parse(resultFromCache);

        // return result from cache, then refresh cache in background
        Promise.resolve(resolve(obj, args, ctx, info)).then(res =>
          cacheSetInBackground(cacheKeyString, res, ttl)
        );
      } else {
        result = await resolve(obj, args, ctx, info);

        cacheSetInBackground(cacheKeyString, result, ttl);
      }

      return result;
    };
  }
}
```

但是以上的作法還是無法避免下面這張圖的問題
[上次 tech-workshop 有提到](https://hackmd.io/C2yzLJJETLq3lNYM_cHzGw)

![](https://i.imgur.com/jbbglvr.png)



### 如何解決當不同 query 卻要拿到相同 fields 的情形？

[Intuit API: Caching at the Edge](https://www.youtube.com/watch?v=zhhxEr5mrWA&list=PLpi1lPB6opQyraZSmwFre_FpL00_3nTzV)

![](https://i.imgur.com/HKDhpsl.png)

![](https://i.imgur.com/nihDwVw.png)

![](https://i.imgur.com/nds7Tpm.png)

![](https://i.imgur.com/b4KV5JV.png)


#### 以下跟我們現在 kurator cache directive 的作法類似
![](https://i.imgur.com/2tdxelb.png)

![](https://i.imgur.com/Aw2nS99.png)

#### 更多層的 query

![](https://i.imgur.com/QhijUlD.png)

![](https://i.imgur.com/7lx3v62.png)

![](https://i.imgur.com/am5rMXZ.png)

- 有了 secondary key 的設計，就可以讓不同 query 可以直接從 cache 裡拿到相同欄位的值

#### comparing queries across requests

![](https://i.imgur.com/SMMsyHO.png)

- 左邊 query 要的欄位其實是右邊 query 的 super set，所以可以直接從 cache 裡拿出值來

## 結束