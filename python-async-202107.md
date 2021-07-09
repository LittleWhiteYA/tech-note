Python async - 2021/7
===

###### tags: `tech workshop`

# 什麼是 Coroutine?

- [【Python教學】淺談 Coroutine 協程使用方法](https://www.maxlist.xyz/2020/03/29/python-coroutine/)

# Python & Node.js async/await 差異

## 執行的時間點

node.js

```javascript=
const sleep = function (time) {
  return new Promise(function (resolve, reject) {
    setTimeout(function () {
      resolve();
    }, time);
  });
};

async function exec() {
  console.log('exec!');
  await sleep(2000);
}

async function go() {
  const start = Date.now();
  const c1 = exec();
  const c2 = exec();
  console.log(c1, c2);
  await c1;
  await c2;
  const end = Date.now();
  console.log(end - start, 'ms');
}

go();

// output:
// exec!
// exec!
// Promise { <pending> } Promise { <pending> }
// 2007 ms
```

python

```python=
import asyncio
import time


async def exec():
    print('exec!')
    await asyncio.sleep(2)


if __name__ == "__main__":

    async def go():
        start = time.time()
        c1 = exec()
        c2 = exec()
        print(c1, c2)
        await c1
        await c2
        end = time.time()

        print(f"{end - start} secs")

    asyncio.run(go())

# output:
# <coroutine object exec at 0x7ff900ccf4c0> <coroutine object exec at 0x7ff8fd854740>
# exec!
# exec!
# 4.003887891769409 secs
```

- 執行後發現 node.js 執行時間為 2s，而 python 執行時間為 4s，由此可見 node.js 的 Promise 在宣告的當下就會開始執行，而 python 的 Coroutine object 並沒有馬上執行


## Tasks

如何在 Python 上達到 Node.js Promise (宣告並立即執行) 的效果呢？
- 包成 Tasks

```python=
import asyncio
import time


async def exec():
    print('exec!')
    await asyncio.sleep(2)


if __name__ == "__main__":

    async def go():
        start = time.time()
        c1 = asyncio.create_task(exec())
        c2 = asyncio.create_task(exec())
        print(c1, c2)
        await c1
        await c2
        end = time.time()

        print(f"{end - start} secs")

    asyncio.run(go())
```

- 執行時間跟之前相比縮短為 2 秒

## await

python

- await 後面必須接一個 Coroutine 對象或是 awaitable 類型的對象

node.js

> 幫補充，如果是 function 就執行？？

## 其他功能

- timeout
- cancel

## coroutine, tasks, future 的差異

- https://segmentfault.com/a/1190000012631063


# 如何從 sync 改寫成 async？

[Adopting Python Asyncio in Large Scale Project (Instagram) – Jimmy Lai – PyCon Taiwan 2018](https://www.youtube.com/watch?v=ACgMTqX5Ee4&ab_channel=PyConTaiwan)

把 async function 轉成 sync function

```python=
def async_to_sync(coroutine):
    loop = asyncio.get_event_loop()

    return loop.run_until_complete(coroutine)
```

sync function 轉成 async function

```python=
async def sync_to_async(func):
    loop = asyncio.get_event_loop()
    future = loop.run_in_executor(None, func)

    return await future
```

- 慢慢從底層的 library 改寫

![](https://i.imgur.com/rt4Jujc.png)


## 參考

- [Python & Node.js 协程 async/await 的差异](https://zhuanlan.zhihu.com/p/103107567)