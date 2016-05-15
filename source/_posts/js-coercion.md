title: Is empty array in JavaScript truthy or falsy?
tags: javascript
date: 2015-12-15 23:25:35
---

Today, one of my friends argued with me about the empty array `[]`, he think it's falsy but in my experience I persist it should be truthy.
<!-- more -->  
So, I demo it as follows:
```
var a = [];
if (a) console.log('empty array is truthy');
```
Or you can also use double logical NOT to cast it to boolean, it's the same.
```
var a = [];
if (!!a) console.log('empty array is truthy');
```
Looking the the evidence sentence shown in the screen, I think the argument is over till he shows me the following code,
```
console.log([] == false);
```
## Guess what?
The result is **true**. Woooooot? Well, I know there must be something wrong, escpecially the suspicous loose equality operation. I have to figure it out why it has such odd behaviour.

## Type Coertion
There are both awesome articles about the equality comparison algorithm in JavaScript world.
* http://bclary.com/2004/11/07/#a-11.9.3
* https://developer.mozilla.org/en-US/docs/Web/JavaScript/Equality_comparisons_and_sameness
![](/images/js-coercion-table.png)
--*table from https://developer.mozilla.org/en-US/docs/Web/JavaScript/Equality_comparisons_and_sameness*

The key point is applying `ToPrimitive([])` and `ToNumber(false)`, then compare them.

### What's ToPrimitive?
ToPrimitive is a internal operation which attempts to convert its object argument to a primitive value, by attempting to invoke varying sequences of `toString` and `valueOf` methods on the Object. The detail steps can be found here: http://bclary.com/2004/11/07/#a-9.1
So, firstly try `[].valueOf()` which returns itself, it's not primitive, then try to call `[].toString`, for `Array.toString`, according [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/toString), 
> For Array objects, the toString method joins the array and returns one string containing each array element separated by commas. 

This means we get the empty string `''`.

### What's the ToNumber?
Basically apply the `+` operation on the target, for boolean, 
>  1 if the argument is true. +0 if the argument is false.

for other types refer to :http://bclary.com/2004/11/07/#a-9.3

## Problem now is simplified 
Is `'' == 0` truthy or falsy? Again begin with applying `ToNumber` to emtpy string `''`, the rules is complicated [here](http://bclary.com/2004/11/07/#a-9.3.1) but for empty string it's simple as follows,
> A StringNumericLiteral that is empty or contains only white space is converted to +0.

So, `0 === 0`? Surely it's true.

## Conclusion
Do **NOT** use loose eqality operation, it's complex and confusing even for experienced JavaScript developers. Although the only exception is checking with null and undefined because either is equal to itself. i.e `foo != null` is equivalent to `foo !== null` && `foo !== undefined`. However, I don't recommend this way for less typing leads more confusion, the reader may even doubt whether the author forgot a `=` or not.

