title: 从命令式编程到函数式编程
date: 2019-01-05 23:35:33
tags: 
  - ramda 
  - functional programming
  - TDD
---

不要被题目骗了:D, 此文不讲概念(题目太大)，只和大家分享一个很简单的案例, 权当抛砖引玉。

## 背景

最近在做一个项目，对接的API要求对response body进行签名，验签规则此处不赘述了，有一点比较好玩的事要求对嵌套的字段进行stringify再进行验签处理，有点类似于lodash的[flatten](https://lodash.com/docs/4.17.11#flatten), 不过不是处理array而是json object, 可能说到这里有点费解，看代码便知。

>  TALK IS CHEAP, SHOW ME THE CODE!

<!-- more -->  

## 测试先行

上述的功能我要用一个叫`flatten`的函数实现，test code如下，

```js
const chai = require('chai');
chai.should();

const { flatten } = require('../lib/sign');  // the function I'm gonna build
describe('lib sign', () => {
  it('flatten', () => {
    const input = {
      foo: '1000',
      bar: 2,
      qux: {
        qux1: 'qux1',
        qux2: 22
      },
      sign: 'abcd',
      quz: [{ quz1: 'quz1value' }]
    };
    const output = flatten(input);
    output.should.be.eql({
      foo: '1000',
      bar: 2,
      qux: '{"qux1":"qux1","qux2":22}',
      quz: '[{"quz1":"quz1value"}]'
    });
  });
});
```

可以看出主要变化就2点，
1. `sign` 字段要被过滤掉
2. 如果不是primitive的值，要做stringify
太简单了是不是？是不是primitive用一个`isObject`的函数来判断，其他的for 加 if 就解决了，

```js
function isObject(obj) {
  return obj === Object(obj);
}
function flatten(obj) {
  delete obj.sign;
  Object.keys(obj).forEach((key) => {
    if (isObject(obj[key])) {
      obj[key] = JSON.stringify(obj[key])
    }
  })
  return obj
}
```
看! 测试也通过了，就这样结束了吗？

不。。。我不喜欢上面这个flatten方法，因为它不是immutable的，作为flatten的入参的这个object已被改的面目全非了, 比如在调用完flatten后你还想去找`sign`的值，对不起它已经不存在了。对immutable不太理解的同学可以移步这篇[博客](https://medium.freecodecamp.org/write-safer-and-cleaner-code-by-leveraging-the-power-of-immutability-7862df04b7b6)。

## 如何改进

因为有测试代码在保护着功能代码，我们可以大胆重构，比如，
```js
function flatten(obj) {
  const newObj = {}
  Object.keys(obj).forEach((key) => {
    if (key === 'sign') {
      return
    }
    if (isObject(obj[key])) {
      newObj[key] = JSON.stringify(obj[key])
    } else {
      newObj[key] = obj[key]
    }
  })
  return newObj
}
```

好一些了，可是代码看上去好像变复杂了，如果有了解过lodash的话好像可以写的跟简洁一些，比如
```js
const _ = require('lodash')
function flatten(obj) {
  const objWithoutSign = _.omit(obj, ['sign'])
  return _.mapValues(objWithoutSign, (val) => {
    return isObject(val) ? JSON.stringify(val) : val
  });
}
```

## 大功告成？

不，这个flatten不够好，它没有复用性，比如我不需要过滤`sign`字段，但是还要stringify怎么办？再拷贝一份`_.mapValues(..)`处理？

轮到[Ramda](https://ramdajs.com/) 登场，为什么要用Ramda此处还是不赘述了，官网有好几篇很好的文章，我这里只分享我是如何实现的。同样地，因为有测试代码，又可以放心重构, 此处思路要有所转变，以funtional programming的方式，我们不考虑有什么数据要处理，而是考虑有哪些处理要aggregate， 显然，
1. 要过滤某一个字段 => R.omit
2. key/value中的value要转变 => R.map
3. value怎么转变？ => stringify
```js
const R = require('ramda')
const stringify = (val) => {
  return isObject(val) ? JSON.stringify(val) : val
}
const flatten = R.compose(
  R.map(stringify),
  R.omit(['sign'])
);
```

可以看到，现在我们用`R.compase`定义了`flatten`，同时还有一个辅助的fucntion `stringify`, 
但是既然用了Function Programming的方式就尽量写的纯粹一些，`stringify`可以改写成

```js
const stringify = R.ifElse(isObject, JSON.stringify, R.identity);
```

## 百密一疏

智者千虑必有一疏，上述这个代码，测试是跑不过的。。。
`output`是长得这幅样子，你看出来哪里有问题了吗？ 
```
Object
  bar: function()
  foo: function()
  qux: function()
  quz: function()
```

## 都是Curry惹的祸

我们知道Ramda中所有的function都是curried化过的，详细解释移步[此处](https://ramdajs.com/docs/#curry), 所有的value都变成function, 看起来很像是partial application，即参数不够（~~没吃饱~~), 默默地再翻了一下MDN关于stringify的文档
> JSON.stringify(value[, replacer[, space]])

呃，忘了stringify 是三个参数（虽然不常用）, 此处需要[R.unary](https://ramdajs.com/docs/#unary) 来帮忙，因为Ramda的 compose/pipe都是需要function只支持一个参数的。

## Holy Grail版

```js
const R = require('ramda');
function isObject(obj) {
  return obj === Object(obj);
}
const stringify = R.ifElse(isObject, R.unary(JSON.stringify), R.identity);
const flatten = R.compose(
  R.map(stringify),
  R.omit(['sign'])
);
```

其实`isObject`也可以继续用Ramda改写，留给你当作业了 ：）
