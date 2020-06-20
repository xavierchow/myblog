title: When ramdajs meets Promise
date: 2020-04-25 20:08:23
tags:
---
# Challenge<a id="sec-1"></a>

I use [ramdajs](https://ramdajs.com/) a lot as I like its conciseness and functional style, but recently I met a challenge with it with Promise. The scenario is deadly simple, I need to fetch some records from the DB and do some transformation and save them back.

Quite straightforward to use ramda for data transformation, isn't it?

```js
const ids = ['123', '456', '789']
const records = await DB.getByIDs(ids)
const updated = R.map(record => {
  const transfromed = transform(record)
  return DB.update(transfromed) // ops, update returns a Promise
} , records)
```
<!-- more -->  

Here the problem is that the updating is asynchronous operation, and I do need to wait for its result. so `async` is needed in the map function, then now R.map returns an array of Promise. So I have to use `Promise.all` to get the results of them.

```js
const ids = ['123', '456', '789']
const records = await DB.getByIDs(ids)
const updated = Promise.all(R.map(async record => {
  const transfromed = transform(record)
  return DB.update(transfromed)
} , records))
```

# So it doesn't look like a challenge<a id="sec-2"></a>

Exactly, the code snippet above doesn't seem to be problematic, but What I faced is that the `ids` was not an array of 3 items, it will be thousands or ten thousands IDs. So what? you might ask, I'm going to tell you I won't run this code against my DB as one of cons for Promise is that you have no control when it starts, i.e. Promise.all will issue 10K write operation hamming the DB at the same time.

One of approaches to solve it is to use [bluebird](https://github.com/petkaantonov/bluebird), it's a cool library and I do use it a lot, it provides a fine-grained control about Promise, you can use [map](http://bluebirdjs.com/docs/api/promise.map.html) with a `concurrency` to control the throughput of the executing Promises.

# This is a tiny script, I don't want to import bluebird<a id="sec-3"></a>

There should be another way, all I want is to make the Promise to be sequentially composed, why not use `for`?

```js
const ids = ['123', '456', '789']
const records = await DB.getByIDs(ids)
const updated = []
for(var i = 0; i< records.length ; i++) {
  const transfromed = transform(record)
  updated.push(await DB.update(transfromed))
}
```

It works but I really don't like it, it's a more imperative coding style than functional way and it's slow as the Promises run one by one.

# Back to use ramdajs<a id="sec-4"></a>

I recall there is a `pipeP` function from ramda but now it's deprecated and replaced with `pipeWith`. Indeed piping is a good way to do things in sequential, only thing makes me uncomfortable is that it needs to be fed with functions, but I have a bunch of data instead of functions, but there is a way.

```js
const ids = ['123', '456', '789']
const records = await DB.getByIDs(ids)
const funcs = R.map((record) => async(acc) => {
  acc = R.defaultTo([])(acc)
  return acc.push(await DB.update(tranform(record)))
})(records)
const updated = R.pipeWith(R.then)(funcs)()
```

Here the point is to build a bunch of functions and use R.then to run one after the previous one resolves. And as I still need the result of all updated I need a `accumulation` passing through the pipe functions. However, I'm not sure if pipeWith works with thousands of functions and &#x2026; the \`accumulation\` reminds me the `reduce` should work as well.

# Reduce version<a id="sec-5"></a>

```js
const ids = ['123', '456', '789']
const records = await DB.getByIDs(ids)
const rf = async (acc, item) => {
  const resloved = await acc
  return resolved.push(await DB.update(tranform(item)))
}
const updated = await R.reduce(rf, [], records)
```

Each step returns a promise, to reduce the promises, you can see `await` are applied to both acc and item transformation. Also bear in mind don't let DB.update run before `await acc`.

# Transducer version<a id="sec-6"></a>

```js
const ids = ['123', '456', '789']
const records = await DB.getByIDs(ids)
const rf = async (acc, item) => {
  const resolved = await acc
  return resolved.push(await item)
}
const updated = await R.transduce(R.map(R.pipe(tranform, DB.update)), rf, [], records)
```

Of course you could use transducer as well.

# Speed it up<a id="sec-7"></a>

If you are not satisfy with running Promise one by one, you could still use a bit `Prmose.all` to run by batch.

```js
const ids = ['1', '2', ... , '999']
const partitions = R.splitEvery(20)(ids);

const rf = async (acc, item) => {
  const resolved = await acc
  const current = await Promise.all(R.map(async item => {
    return DB.update(tranform(item));
  })(await DB.getByIDs(ids)));
  return resolved.push(current)
}
const updated = R.flatten(await R.reduce(rf, [], partitions))

```

That is all, Promise is not a pure but as you see there are still a few ways to combine it with functional ways.

