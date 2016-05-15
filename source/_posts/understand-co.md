title: Why use tj/co
tags:
  - javascript
  - flow-control
date: 2016-05-15 15:26:05
---

大神[tj](https://github.com/tj)的[co](https://github.com/tj/co)已经超过5千star了，之前公司的小伙伴有在用这个模块，那时我还没有开始用es6，对于genenrator也是一知半解，匆匆扫了一眼，说实话没看懂，当时手头也比较忙也就这样先放着了。

转眼在项目中正式用es6也有半年多了，准备回过头来再学一下co，翻了一下github上的README, 什么鬼？还是没搞懂。。。主要问题是examples只讲了**how**，而没有讲**why**，而这正是我所在意的，我为什么要用这个模块，有什么好处？
<!-- more -->  


于是在[Medium](https://medium.com/)上翻到了tj的这篇博客[Callbacks vs Coroutines](https://medium.com/@tjholowaychuk/callbacks-vs-coroutines-174f1fe66127#.9fw59pduu)，
看完之后不禁拍案，原来动机如此，co想解决的事原来在我学习generator的时候我也尝试过！

短话长说（反了？），我们知道es6 generator function可以重复进入，程序运行到yield语句时，控制权转移到function外部，调next的时候，控制权回来。然而generator内的代码看上去是同步顺序执行的，
这不正是一个能完美的将异步调用转成类似同步执行的特性吗？（没错，对于天生异步的node.js，人们一直致力于将它拧巴地或优雅地转换成人类习惯的同步方式中)

上代码：
```js
function* main() {
  var result = yield myrequest("http://some.url");
  var res = JSON.parse(result);
  console.log(res.msg);
}

function myrequest(url) {
  makeAjaxCall(url, function(response){
    gen.next(response);
  });
}

var gen = main();
gen.next();

function makeAjaxCall(url, callback) {
  require('request')('http://some.url', function(err, res, body) {
    callback(body);
  });
}
```
可以看出主逻辑（main）是同步的而且很清晰，但是异步并没有平白消失，只是它们被移到了别的地方（makeAjaxCall），但是我们保证了主控制流是同步的，你可以说这是语法糖也好，但是promise也不是类似的思路么？
事实上co也是如此，就如同上面一样利用generator的特性来扭转控制流，当然比我上面的例子更优雅更强大（强壮）。

上面的代码用co重写后是这样的：
```js
const promiseAjax = () => {
  return require('request-promise').get('http://some.url');
};

co(function* () {
  var result =  yield promiseAjax();
  var res = JSON.parse(result);
  return res;
}).then(r => {
  console.log(r);
})
```

所以在面对callback hell的时候除了[aysnc](https://github.com/caolan/async), promise外又多了一个好办法。

下一步我要好好读一下co的源码，我建议你也这样做，或者至少你应该看一下这篇博客[Callbacks vs Coroutines](https://medium.com/@tjholowaychuk/callbacks-vs-coroutines-174f1fe66127#.9fw59pduu)，我相信会让你受益匪浅的。

