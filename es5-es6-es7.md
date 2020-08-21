ES5、ES6、ES7 - 2019/12/25
---

###### tags: `tech workshop`

---

![](https://cdn-images-1.medium.com/max/800/1*ko3KtcVSlzpe3RnTRgJaHw.jpeg =500x)

> from [ES7 Async Await 聖經](https://medium.com/@peterchang_82818/javascript-es7-async-await-%E6%95%99%E5%AD%B8-703473854f29-tutorial-example-703473854f29)

---

# Promise、async/await 簡單介紹

---

## ES5

- callback hell

```
doFirstThing(function(result1) {
  doSecondThing(result1, function(result2) {
    doThirdThing(result2, function(result3) {
      console.log(result3);
    })
  })
})

function doFirstThing(callback) {
  callback("喔~~~~~");
}

function doSecondThing(str, callback) {
  callback(str + "\n是誰住在深海的大鳳梨裡？");
}

function doThirdThing(str, callback) {
  callback(str + "\n海綿寶寶！");
}

/*
喔~~~~~
是誰住在深海的大鳳梨裡
海綿寶寶
*/
```

---

## 有了 Promise

```
doFirstThing()
.then(str => doSecondThing(str))
.then(str => doThirdThing(str))
.then(str => console.log(str));

function doFirstThing() {
  return Promise.resolve("喔~~~~~");
}

function doSecondThing(str) {
  return Promise.resolve(str + "\n是誰住在深海的大鳳梨裡？");
}

function doThirdThing(str) {
  return Promise.resolve(str + "\n海綿寶寶！");
}

/*
喔~~~~~
是誰住在深海的大鳳梨裡
海綿寶寶
*/
```

---

## Promise 的問題咧？

- 有多個 async function 就要一直 then, then, then...

```
doFirstThing()
.then(str => doSecondThing(str))
.then(str => doThirdThing(str))
.then(str => console.log(str));
```

- Promise resolve 只能傳一個參數，如果 doTenthThing 要拿 doFirstThing 的變數，就要包成 Object 才行
```
doFirstThing()
.then(str => ({ one: str, str: doSecondThing(str) })
.then(obj => ({ ...obj, str: doThirdThing(obj.str) })
...
.then(obj => doTenthThing(obj.one))
.then(str => console.log(str));
```

---

## 有了 async/await

- 幾乎不用 then 了
- 每個 function 的回傳結果只要當成普通變數就行
```
...
const one = await doFirstThing();
...
const ten = await doTenthThing(one);
...
```

---

# async/await Array forEach vs. map

---

## async/await with forEach

- sync forEach 寫法
```
const arr = [];

function cal(num) {
  return num * 2;
}

[1, 2, 3].forEach(num => {
  const n = cal(num);
  arr.push(n);
});                       

console.log(arr);
console.log('Done');

// [2, 4, 6]
// Done
```

---

- 改寫成 async forEach
```
async function cal(num) {
  return num * 2;
}

(async () => {
  const arr = [];
  [1, 2, 3].forEach(async num => {   
    const n = await cal(num);
    arr.push(n);
  });

  const res = await Promise.all(arr);

  console.log(res);
  console.log('Done');
})();

// []  !?!?!?!?
// Done
```

---

- forEach (pseudo code)
> [Array.prototype.forEach() - MDN](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach)
```
Array.prototype.forEach = function (callback) {  
  // this represents our array  
  for (let index = 0; index < this.length; index++) {  
    // We call the callback for each entry  
    // 這邊沒有等 !!!!!
    callback(this[index], index, this)  
  }  
}
```

---

## async/await with map
- sync map 寫法
```
function cal(num) {
  return num * 2;
}

const arr = [1, 2, 3].map(num => {
  const n = cal(num);
  return n;
});                       

console.log(arr);
console.log('Done');

// [2, 4, 6]
// Done
```

---

- async map
```
async function cal(num) {
  return num * 2;
}

(async () => {
  const arr = [1, 2, 3].map(async num => {
    const n = await cal(num);
    return n;                             
  });
  
  const res = await Promise.all(arr);
  
  console.log(res);
  console.log('Done');
})();


// [2, 4, 6]
// Done
```

> 參考 [async/await with forEach()](https://codeburst.io/javascript-async-await-with-foreach-b6ba62bbf404)

---

# Promise.all vs. pMap

---

## Promise.all

```
async function callAPI(num) {
  return num * 2;
}

const N = 10; // 這個 N 如果很大就慘ㄌ

(async () => {
  const arr = Array.from(Array(N).keys()).map(num => callAPI(num));
                                                                   
  const res = await Promise.all(arr);
  
  console.log(res);
  console.log('Done');
})();
```

---

## pMap
- https://github.com/sindresorhus/p-map
```
const pMap = require('p-map');                                         

async function callAPI(num) { 
  return num * 2;
}

const N = 10;

(async () => { 
  const arr = Array.from(Array(N).keys());

  // concurrency 可以設定最多同時等幾個 Promise
  const res = await pMap(arr, num => callAPI(num), { concurrency: 5 });

  console.log(res);
  console.log('Done');
})();
```

---

- more https://github.com/sindresorhus/promise-fun

---

## pMap 錯誤使用

- 巢狀 pMap
  - [understood](https://github.com/Yoctol/understood/commit/df2816f8c8146e368c2fcb6446909f08df6d2e9a)

---

# Promise 執行順序
```
async function tmp2() {    
  console.log('g');
}

async function tmp() {
  console.log('c');
  process.nextTick(() => {
    console.log('f');
  });
  await tmp2();
  console.log('e');
}

console.log('a');
tmp().then(() => {
  console.log('b');
});
console.log('d');
```

---

# lodash
https://lodash.com/docs/4.17.5

- Array
  1. _.difference
  2. _.intersection
  3. _.union
  4. _.flatten
  5. _.flattenDeep
- Object
  1. _.pick
- Other
  1. _.cloneDeep

[YOU MIGHT NOT NEED LODASH](https://youmightnotneed.com/lodash/)