title: Less is more when it comes to code
date: 2022-10-30 22:57:24
tags:
---

Recently I have been trying to learn Haskell, one of the famous functional programming languages. When I practiced with some exercises, I found an interesting thing that the [Excercise here](https://www.seas.upenn.edu/~cis1940/spring13/hw/03-rec-poly.pdf) requires you to write the code as little as possible.


Well, it is not necessary to pursue the most minimalistic line of code in most cases, but in general, a smaller code base is easier to understand and maintain over time.

Then I was curious how shorter I could get it done with ... Haskell.

<!-- more  -->

## Description of problem

Given a list with numbers from 0 to 9(inclusive), generate a vertical histogram to stdout displaying how many of each number were in the list. e.g.

```
given:  [1,4,5,4,6,6,3,4,2,4,9]

display:

    *
    *
    * *
 ******  *
==========
0123456789

```


## Before getting your hands dirty

The problem as mentioned above doesn't seem to be so difficult at first glance, it's all about counting the times of showing up in the list and rendering the result with ASCII character(asterisk).
Essentially I broke it into two sub-problems,
1. count the times of the number 
2. display a vertical histogram


I also had the plan to not only address it with Haskell but also find a solution with an imperative paradigm(that would be JavaScript as it's one of my most familiar language). It would be a typical `declarative vs imperative` comparison, then.


## Imperative paradigm(JavaScript version)

It's relatively straightforward; the first loop gathers the times of appearance for each number in a hash.
The keys of the hash are the numbers from 0 to 9 and the values are the corresponding appeared times.

The rendering part is a bit tricky as I have to draw the diagram up-down because it's stdout output. To know how many rows I have to draw, I used the `max` to get the number of rows from the values of hash.
The inner `for loop` (with `k` as its variable) prints 10 characters for 0~9, either asterisk or blank depending on if the value in the hash exceeds the position of current row.


*Side Note:* As the `console.log` brings a newline for each output, I have to build a `print` that prints a character without a newline.

```js
function histogram(arr) {
  const hash = {};
  for (let i = 0; i < arr.length; i++) {
    const cur = arr[i];
    hash[cur] = (hash[cur] || 0) + 1;
  }
  // console.log(hash);
  //   output: { '1': 1, '2': 1, '3': 1, '4': 4, '5': 1, '6': 2, '9': 1 }
  const rows = Math.max(...Object.values(hash));
  for (let j = rows; j > 0; j--) {
    for (let k = 0; k < 10; k++) {
      print(hash[k] >= j ? '*' : ' ');
    }
    print('\n');
  }

  console.log('==========');
  console.log('0123456789');
}

function print(arg) {
  process.stdout.write(arg);
}

histogram([1,4,5,4,6,6,3,4,2,4,9]);


```

## Declarative paradigm(Haskell version)

I think if the question asks for a horizontal histogram, it would be much easier as each line in the stdout represents the quantity for a number. That inspiration just reminds me that a histogram is actually a 2-D matrix and if I could flip it...
That's called [transpose](https://en.wikipedia.org/wiki/Transpose) actually in linear algebra; Haskell happens to have that built-in function!
Buckle up and let me elaborate on the solution.

Let us fiddle with a similar but simplified example first, e.g., a list from 1 ~ 3: `[1, 2, 2, 3, 3, 2]`, 
To get the matrix (1) eventually, we can try to build (2) first, don't worry about the `_`; it's just a  representation of a blank in the matrix for better visibility.

```
(1)
_ * _
_ * *
* * *
1 2 3


(2)
1 * _ _     [ [1, 0, 0]
2 * * *       [2, 2, 2]
3 * * _       [3, 3, 0] ]

```
If we replace the asterisks with the corresponding nubmers, (2) would be a two-dimension list `[[1, 0, 0], [2, 2, 2], [3, 3, 0]]`, so if you know about Haskell, the conversion from the original list to the 2-dimension list needs a function with a signature of `[a] -> [[a]]`,
that ... could be done with a `map`.
```
map f [1, 2, 3]

```

Where is the list `[1, 2, 3]` from? It is the left side label in diagram (2) that could be exacted with a `unique` or `dedup` function from the source list; in Hanskell it has such a function called `nub`.

```haskell
nub ['a', 'a', 'b', 'c', 'b']
-- output: ['a', 'b', 'c']

```
Then what should be the `f` like? 
To get the target matrix `[[1, 0, 0], [2, 2, 2], [3, 3, 0]]`, we actually can ignore the zero in the inner list, so the result we try to get is just `[[1], [2,2,2], [3,3]]`, you probably have realized it is... just a grouping with same numbers, we can use `filter` to achieve it.

```haskell
matrix x = map (\n -> filter (== n) x) $ nub x

```
So now we have a list `[[1], [2,2,2], [3,3]]` that represents the matrix (2), let's use `transpose` to flip it to:
`[[1,2,3], [2,3], [2]]`; if you have confusions about this step, you can revisit the [awesome animation here](https://en.wikipedia.org/wiki/Transpose).
Check the transposed list; it's exactly what we are looking for to guide us in drawing the asterisk!
![img](https://user-images.githubusercontent.com/2748884/199040330-71c21c00-9c70-42b7-aa35-04f0772f7b37.png)

Bear in mind we have to draw the rows top-down, so a reverse of the list is needed.
```haskell
matrix x = map (\n -> filter (== n) x) $ nub x
matrix_to_draw x = reverse . transpose $ matrix x
```

The final step, we need a `drawRow` function that ingests a list and produces a line of asterisk interpolated with blank correspondingly.
```haskell

drawRow s = map (\n -> if n `elem` s then '*' else ' ') [0 .. 9]
```

![img](https://user-images.githubusercontent.com/2748884/199044570-f0b01b34-0143-4d5d-baf6-5967fe19c999.png)



The remaining stuff is just `unlines` the list into a string and calls `putStr` to display it.
Actually, I can merge the following 3 functions into one line of code, but for the sake of readability I still prefer to have small functions with meaningful names.
```haskell
-- final code 
matrix x = map (\n -> filter (== n) x) $ nub x
drawRow s = map (\n -> if n `elem` s then '*' else ' ') [0 .. 9]
histogram x =
  putStr
    $  (unlines . (map drawRow) . reverse . transpose $ matrix x)
    ++ "==========\n0123456789\n"

```

## Wrap-up

Comparing the JavaScript version that is in an imperative style, we see how concise the functional programming language can be, and it's also more readable if you give the proper names to the functions.

Moreover, less is more can be considered from two other aspects,
- Less code is easier to read and more robust as there is less room for errors.
- Less code makes you think more about abstraction and reuse.

*Disclaimer*: This is not about the battle of languages, I'm not saying Haskell is better than any other language; in fact, JavaScript is able to write in a functional way with some help from libraries.
If you prefer reading or using JavaScript, here is the translated final code with Ramda package.

```js
const arr = [1, 4, 5, 4, 6, 6, 3, 4, 2, 4, 9];
const R = require('ramda');

function matrix(arr) {
  return R.map((n) => R.filter(R.identical(n), arr), R.uniq(arr));
}
function drawRow(list) {
  const arr0to9 = R.times(R.identity, 10);
  return R.join(
    '',
    R.map((x) => (R.includes(x, list) ? '*' : ' '), arr0to9)
  );
}
function histogram(arr) {
  const r = R.compose(R.reverse, R.transpose)(matrix(arr));
  // console.log(r);
  R.map(console.log, R.map(drawRow, r));
  console.log('==========');
  console.log('0123456789');
}

histogram(arr);
```
