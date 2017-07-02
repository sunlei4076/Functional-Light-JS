# Functional-Light JavaScript
# Appendix A: Transducing
# 附录 A：Transducing

Transducing is a more advanced technique than we've covered in this book. It extends many of the concepts from Chapter 8 on list operations.

Transducing 是我们这本书要讲到的更为高级的技术。它继承了第 8 章数组操作的许多思想。

I wouldn't necessarily call this topic strictly "Functional-Light", but more like a bonus on top. I've left this to an appendix because you might very well need to skip the discussion for now and come back to it once you feel fairly comfortable with -- and make sure you've practiced! -- the main book concepts.

我觉得这个技术并不是严格的 "Functional-Light"，它更像是一个额外的补足。我把这个技术留到附录来讲意味着你现在很可能并不需要关心它，当你确保你已经理解了整本书的主要内容而且比较熟练了，你可以再回头看看这一章节。

To be honest, even after teaching transducing many times, and writing this chapter, I am still trying to fully wrap my brain around this technique. So don't feel bad if it twists you up. Bookmark this appendix and come back when you're ready.

说实话，即使我已经教过 transducing 很多次了，在写这一章的时候，我仍然需要花很多脑力去理清楚这个技术。所以，如果你看这一章看的很疑惑也没必要感到沮丧。把这一章加个书签，等你觉得你的能力差不多足够时再回头看看。

Transducing means transforming with reduction.

Transducing 意味着减少。

I know that may sound like a jumble of words that confuses more than it clarifies. But let's take a look at how powerful it can be. I actually think it's one of the best illustrations of what you can do once you grasp the principles of Functional-Light Programming.

我知道这听起来很令人费解。但是让我们来看看它有多强大。实际上，我认为这是你掌握  Functional-Light 编程后可以做的最好的例证之一。

As with the rest of this book, my approach is to first explain *why*, then *how*, then finally boil it down to a simplified, repeatable *what*. That's often backwards of how many teach, but I think you'll learn the topic more deeply this way.

和这本书的其他部分一样，我的方法是先解释*为什么*使用这个技术，然后*如何*使用，最后归结为简单的这个技术到底是什么样的。这通常会有很多东西要学，但是我觉得用这种方式你会更深入的理解它。

## Why, First
## 首先，为什么

Let's start by extending a scenario we covered back in Chapter 3, testing words to see if they're short enough and/or long enough:

让我们从扩展我们在第 3 章中介绍的例子开始，测试单词是否足够短和/或足够长：

```js
function isLongEnough(str) {
	return str.length >= 5;
}

function isShortEnough(str) {
	return str.length <= 10;
}
```

In Chapter 3, we used these predicate functions to test a single word. Then in Chapter 8, we learned how to repeat such tests using list operations like `filter(..)`. For example:

在第 3 章中，我们使用这些断言函数来测试一个单词。然后在第 8 章中，我们学习了如何使用像 `filter(..)` 这样的数组操作来重复这些测试。例如:

```js
var words = [ "You", "have", "written", "something", "very", "interesting" ];

words
.filter( isLongEnough )
.filter( isShortEnough );
// ["written","something"]
```

It may not be obvious, but this pattern of separate adjacent list operations has some non-ideal characteristics. When we're dealing with only a single array of a small number of values, everything is fine. But if there were lots of values in the array, each `filter(..)` processing the list separately can slow down a bit more than we'd like.

这个例子可能并不明显，但是这种分开相邻数组操作的方式具有一些不理想的地方。当我们处理一个值比较少的数组时一切都还好。但是如果数组中有很多值，每个 `filter(..)` 分开处理数组的值会比我们预期的慢一点。

A similar performance problem arises when our arrays are async/lazy (aka observables), processing values over time in response to events (see Chapter 10). In this scenario, only a single value comes down the event stream at a time, so processing that discrete value with two separate `filter(..)`s function calls isn't really such a big deal.

当我们的数组是异步/懒惰（也称为可观察的），随着时间的推移响应事件处理（见第 10 章），会出现类似的性能问题。 在这种情况下，一次只有一个值会降低事件流，因此使用两个单独的 `filter(..)` 函数处理这些值并不是一件很大的事情。

But, what's not obvious is that each `filter(..)` method produces a separate observable. The overhead of pumping a value out of one observable into another can really add up. That's especially true since in these cases, it's not uncommon for thousands or millions of values to be processed; even such small overhead costs add up quickly.

但是，不太明显的是每个 `filter(..)` 方法都会产生一个单独的可观察值。 从一个可观察值中抽出一个值的开销真的可以加起来。 这是特别真实的，因为在这些情况下，处理数千或数百万的值并不罕见; 所以，即使是这么小的成本也会很快累计起来。

The other downside is readability, especially when we need to repeat the same series of operations against multiple lists (or observables). For example:

另一个缺点是可读性，特别是当我们需要对多个数组（或观察值）重复相同的操作时。例如:

```js
zip(
	list1.filter( isLongEnough ).filter( isShortEnough ),
	list2.filter( isLongEnough ).filter( isShortEnough ),
	list3.filter( isLongEnough ).filter( isShortEnough )
)
```

Repetitive, right?

显得很重复，对不对？

Wouldn't it be better (both for readability and performance) if we could combine the `isLongEnough(..)` predicate with the `isShortEnough(..)` predicate? You could do so manually:

如果我们可以将 `isLongEnough(..)` 断言与 `isShortEnough(..)` 断言组合在一起是不是会更好一点呢（可读性和性能）？ 您可以手动执行：

```js
function isCorrectLength(str) {
	return isLongEnough( str ) && isShortEnough( str );
}
```

But that's not the FP way!

但这不是 FP 的方式！

In Chapter 8, we talked about fusion -- composing adjacent mapping functions. Recall:

在第 8 章中，我们讨论了融合--组合相邻映射函数。回忆一下：

```js
words
.map(
	pipe( removeInvalidChars, upper, elide )
);
```

Unfortunately, combining adjacent predicate functions doesn't work as easily as combining adjacent mapping functions. To understand why, think about the "shape" of the predicate function -- a sort of academic way of describing the signature of inputs and output. It takes a single value in, and it returns a `true` or a `false`.

不幸的是，组合相邻断言函数并不像组合相邻映射函数那样容易。为什么呢？想想断言函数长什么“样子” -- 一种描述输入和输出签名的学术方式。 它接收一个单一的参数，返回一个 true 或false。

If you tried `isShortEnough(isLongEnough(str))`, it wouldn't work properly. `isLongEnough(..)` will return `true` / `false`, not the string value that `isShortEnough(..)` is expecting. Bummer.

如果你试着用 `isshortenough(islongenough(STR))`，这是行不通的。因为 `islongenough(..) ` 会返回 true 或者 false ，而不是返回 `isshortenough(..)` 所要的字符串类型的值。这可真倒霉。

A similar frustration exists trying to compose two adjacent reducer functions. The "shape" of a reducer is a function that receives two values as input, and returns a single combined value. The output of a reducer as a single value is not suitable for input to another reducer expecting two inputs.

试图组合两个相邻的 reducer 函数同样是行不通的。 reducer 函数接收两个值作为输入，并返回单个组合值。 reducer 函数的单一返回值也不能作为参数传到另一个需要两个输入的 reducer 函数中。

Moreover, the `reduce(..)` helper takes an optional `initialValue` input. Sometimes this can be omitted, but sometimes it has to be passed in. That even further complicates composition, since one reduction might need one `initialValue` and the other reduction might seem like it needs a different `initialValue`. How can we possibly do that if we only make one `reduce(..)` call with some sort of composed reducer?

此外，`reduce(..)` helper 可以接收一个可选的 `initialValue` 输入。 有时可以省略，但有时候它又必须被传入。这就让组合更复杂了，因为一个 `reduce(..)` 可能需要一个 `initialValue`，而另一个 `reduce(..)` 可能需要另一个  `initialValue`。 所以我们怎么可能只用某种组合的 reducer 来实现 `reduce(..)` 呢。

Consider a chain like this:

考虑像这样的链：

```js
words
.map( strUppercase )
.filter( isLongEnough )
.filter( isShortEnough )
.reduce( strConcat, "" );
// "WRITTENSOMETHING"
```

Can you envision a composition that includes all of these steps: `map(strUppercase)`, `filter(isLongEnough)`, `filter(isShortEnough)`, `reduce(strConcat)`? The shape of each operator is different, so they won't directly compose together. We need to bend their shapes a little bit to fit them together.

你能想出一个组合能够包含 `map(strUppercase)`， `filter(isLongEnough)`，`filter(isShortEnough)`， `reduce(strConcat)` 所有这些操作吗？每种操作的行为是不同的，所以不能直接组合在一起。我们需要把它们修改下让它们组合在一起。

Hopefully these observations have illustrated why simple fusion-style composition isn't up to the task. We need a more powerful technique, and transducing is that tool.

希望这些例子说明了为什么简单的组合不能胜任这项任务。我们需要一个更强大的技术，而 transducing 就是这个技术。

## How, Next

## 如何，下一步

Let's talk about how we might derive a composition of mappers, predicates and/or reducers.

让我们谈谈我们该如何得到一个能组合映射，断言和/或 reducers的框架。

Don't get too overwhelmed: you won't have to go through all these mental steps we're about to explore in your own programming. Once you understand and can recognize the problem transducing solves, you'll be able just jump straight to using a `transduce(..)` utility from a FP library and move on with the rest of your application!

别太紧张：你不必经历编程过程中所有的探索步骤。一旦你理解了 transducing 能解决的问题，你就可以直接使用 FP库中的 `transduce(..)` 工具继续你的应用程序的剩余部分！

Let's jump in.

让我们开始探索吧。

### Expressing Map/Filter As Reduce

## 表示map /筛选器为减少

The first trick we need to perform is expressing our `filter(..)` and `map(..)` calls as `reduce(..)` calls. Recall how we did that in Chapter 8:

我们要做的第一件事情就是将我们的 `filter(..)` 和 `map(..)`调用变为 `reduce(..)` 调用。回想一下我们在第 8 章是怎么做的：

```js
function strUppercase(str) { return str.toUpperCase(); }
function strConcat(str1,str2) { return str1 + str2; }

function strUppercaseReducer(list,str) {
	list.push( strUppercase( str ) );
	return list;
}

function isLongEnoughReducer(list,str) {
	if (isLongEnough( str )) list.push( str );
	return list;
}

function isShortEnoughReducer(list,str) {
	if (isShortEnough( str )) list.push( str );
	return list;
}

words
.reduce( strUppercaseReducer, [] )
.reduce( isLongEnoughReducer, [] )
.reduce( isShortEnough, [] )
.reduce( strConcat, "" );
// "WRITTENSOMETHING"
```

That's a decent improvement. We now have four adjacent `reduce(..)` calls instead of a mixture of three different methods all with different shapes. We still can't just `compose(..)` those four reducers, however, because they accept two arguments instead of one.

这是一个不错的改进。我们现在有四个相邻的 `reduce(..)` 调用，而不是三种不同方法的混合。然而，我们仍然不能组合这四个 reducer，因为它们接受两个参数而不是一个参数。

In Chapter 8, we sort of cheated and used `list.push(..)` to mutate as a side effect rather than calling `list.concat(..)` to return a whole new array. Let's be a bit more formal for now:

在 8 章，我们偷了点懒使用了数组的 `push` 方法而不是 `concat(..)` 方法返回一个新数组，导致有副作用。现在让我们更正式一点：

```js
function strUppercaseReducer(list,str) {
	return list.concat( [strUppercase( str )] );
}

function isLongEnoughReducer(list,str) {
	if (isLongEnough( str )) return list.concat( [str] );
	return list;
}

function isShortEnoughReducer(list,str) {
	if (isShortEnough( str )) return list.concat( [str] );
	return list;
}
```

Later, we'll look at whether `concat(..)` is necessary here or not.

在后面我们会来头看看这里是否需要 `concat(..)`。

### Parameterizing The Reducers

## 参数化 Reducers

Both filter reducers are almost identical, except they use a different predicate function. Let's parameterize that so we get one utility that can define any filter-reducer:

除了使用不同的断言函数之外，两个 filter reducers 几乎相同。 让我们把这些 reducers 参数化得到一个可以定义任何 filter-reducer 的工具函数：

```js
function filterReducer(predicateFn) {
	return function reducer(list,val){
		if (predicateFn( val )) return list.concat( [val] );
		return list;
	};
}

var isLongEnoughReducer = filterReducer( isLongEnough );
var isShortEnoughReducer = filterReducer( isShortEnough );
```

Let's do the same parameterization of the `mapperFn(..)` for a utility to produce any map-reducer:

同样的，我们把 `mapperFn(..)` 也参数化来生成 map-reducer 函数：

```js
function mapReducer(mapperFn) {
	return function reducer(list,val){
		return list.concat( [mapperFn( val )] );
	};
}

var strToUppercaseReducer = mapReducer( strUppercase );
```

Our chain still looks the same:

我们的调用链看起来是一样的：

```js
words
.reduce( strUppercaseReducer, [] )
.reduce( isLongEnoughReducer, [] )
.reduce( isShortEnough, [] )
.reduce( strConcat, "" );
```

### Extracting Common Combination Logic

### 提取共用组合逻辑

Look very closely at the above `mapReducer(..)` and `filterReducer(..)` functions. Do you spot the common functionality shared in each?

仔细观察上面的 `mapReducer(..)` 和滤镜 `filterReducer(..)` 函数。你发现共享功能了吗？

This part:

这部分：

```js
return list.concat( .. );

// or
return list;
```

Let's define a helper for that common logic. But what shall we call it?

让我们为这个通用逻辑定义一个辅助函数。 但是我们叫什么呢？

```js
function WHATSITCALLED(list,val) {
	return list.concat( [val] );
}
```

If you examine what that `WHATSITCALLED(..)` function does, it takes two values (an array and another value) and it "combines" them by concatenating the value onto the end of the array, returning a new array. Very uncreatively, we could name this `listCombination(..)`:

`WHATSITCALLED(..)` 函数做了些什么呢，它接收两个参数（一个数组和另一个值），将值 concat 到数组的末尾返回一个新的数组。 所以这个 `WHATSITCALLED(..)` 不合适，我们可以叫它 `listCombination(..)` ：

```js
function listCombination(list,val) {
	return list.concat( [val] );
}
```

Let's now re-define our reducer helpers to use `listCombination(..)`:

我们现在用 `listCombination(..)` 来重新定义我们的 reducer 辅助函数：

```js
function mapReducer(mapperFn) {
	return function reducer(list,val){
		return listCombination( list, mapperFn( val ) );
	};
}

function filterReducer(predicateFn) {
	return function reducer(list,val){
		if (predicateFn( val )) return listCombination( list, val );
		return list;
	};
}
```

Our chain still looks the same (so we won't repeat it).

我们的调用链看起来还是一样的（这里就不重复写了）。

### Parameterizing The Combination

## 参数化组合

Our simple `listCombination(..)` utility is only one possible way that we might combine two values. Let's parameterize the use of it to make our reducers more generalized:

我们的 `listCombination(..)` 小工具只是组合两个值的一种方式。 让我们作为参考，让我们的 reducers 更加一般化：

```js
function mapReducer(mapperFn,combinationFn) {
	return function reducer(list,val){
		return combinationFn( list, mapperFn( val ) );
	};
}

function filterReducer(predicateFn,combinationFn) {
	return function reducer(list,val){
		if (predicateFn( val )) return combinationFn( list, val );
		return list;
	};
}
```

To use this form of our helpers:

使用这种形式的辅助函数：

```js
var strToUppercaseReducer = mapReducer( strUppercase, listCombination );
var isLongEnoughReducer = filterReducer( isLongEnough, listCombination );
var isShortEnoughReducer = filterReducer( isShortEnough, listCombination );
```

Defining these utilities to take two arguments instead of one is less convenient for composition, so let's use our `curry(..)` approach:

将这些实用程序定义为接收两个参数而不是一个参数不太方便，因此我们使用我们的 `curry(..)` （柯里化）方法：

```js
var curriedMapReducer = curry( function mapReducer(mapperFn,combinationFn){
	return function reducer(list,val){
		return combinationFn( list, mapperFn( val ) );
	};
} );

var curriedFilterReducer = curry( function filterReducer(predicateFn,combinationFn){
	return function reducer(list,val){
		if (predicateFn( val )) return combinationFn( list, val );
		return list;
	};
} );

var strToUppercaseReducer =
	curriedMapReducer( strUppercase )( listCombination );
var isLongEnoughReducer =
	curriedFilterReducer( isLongEnough )( listCombination );
var isShortEnoughReducer =
	curriedFilterReducer( isShortEnough )( listCombination );
```

That looks a bit more verbose, and probably doesn't seem very useful.

这看起来有点冗长而且可能不是很有用。

But this is actually necessary to get to the next step of our derivation. Remember, our ultimate goal here is to be able to `compose(..)` these reducers. We're almost there.

但这实际上是我们进行下一步推导的必要条件。 请记住，我们的最终目标是能够 `compose(..)` 这些 reducers。 我们快要完成了。

### Composing Curried

组合柯里化

This step is the trickiest of all to visualize. So read slowly and pay close attention here.

这一步是最棘手的。 所以请慢慢阅读。

Let's consider the curried functions from above, but without the `listCombination(..)` function having been passed in to each:

让我们看看没有将 `listCombination(..)` 传递给柯里化函数的样子：

```js
var x = curriedMapReducer( strUppercase );
var y = curriedFilterReducer( isLongEnough );
var z = curriedFilterReducer( isShortEnough );
```

Think about the shape of all three of these intermediate functions, `x(..)`, `y(..)`, and `z(..)`. Each one expects a single combination function, and produces a reducer function with it.

看看这三个中间函数 `x(..)`， `y(..)` 和 `z(..)` 的形状。每个函数都期望得到一个单一的组合函数并产生一个 reducer 函数。

Remember, if we wanted the independent reducers from all these, we could do:

记住，如果我们想要所有这些的独立的 reducer，我们可以这样做：

```js
var upperReducer = x( listCombination );
var longEnoughReducer = y( listCombination );
var shortEnoughReducer = z( listCombination );
```

But what would you get back if you called `y(z)`? Basically, what happens when passing `z` in as the `combinationFn(..)` for the `y(..)` call? That returned reducer function internally looks kinda like this:

但是，如果你调用 `y(z)`，会得到什么呢？当把 `z` 传递给 `y(..)` 调用，而不是 `combinationFn(..)` 时会发生什么呢？这个返回的 reducer 函数内部看起来像这样：

```js
function reducer(list,val) {
	if (isLongEnough( val )) return z( list, val );
	return list;
}
```

See the `z(..)` call inside? That should look wrong to you, because the `z(..)` function is supposed to receive only a single argument (a `combinationFn(..)`), not two arguments (`list` and `val`). The shapes don't match. That won't work.

看到 `z(..)` 里面的调用了吗？ 这看起来应该是错误的，因为 `z(..)` 函数应该只接收一个参数（`combinationFn(..)`），而不是两个参数（list 和 val）。 和要求不匹配。 不行。

Let's instead look at the composition `y(z(listCombination))`. We'll break that down into two separate steps:

我们来看看组合 `y(z(listCombination))`。 我们将把它分成两个不同的步骤：

```js
var shortEnoughReducer = z( listCombination );
var longAndShortEnoughReducer = y( shortEnoughReducer );
```

We create `shortEnoughReducer(..)`, then we pass *it* in as the `combinationFn(..)` to `y(..)`, producing `longAndShortEnoughReducer(..)`. Re-read that a few times until it clicks.

我们创建 `shortEnoughReducer(..)`，然后将它作为 `combinationFn(..)` 传递给 `y(..)`，生成 `longAndShortEnoughReducer(..)`。 多读几遍，直到理解。

Now consider: what do `shortEnoughReducer(..)` and `longAndShortEnoughReducer(..)` look like internally? Can you see them in your mind?

现在想想： `shortEnoughReducer(..)` 和 `longAndShortEnoughReducer(..)` 的内部构造时什么样的呢？你能想得到吗？

```js
// shortEnoughReducer, from z(..):
function reducer(list,val) {
	if (isShortEnough( val )) return listCombination( list, val );
	return list;
}

// longAndShortEnoughReducer, from y(..):
function reducer(list,val) {
	if (isLongEnough( val )) return shortEnoughReducer( list, val );
	return list;
}
```

Do you see how `shortEnoughReducer(..)` has taken the place of `listCombination(..)` inside `longAndShortEnoughReducer(..)`? Why does that work?

你看到 `shortEnoughReducer(..)` 替代了 `longAndShortEnoughReducer(..)` 里面 `listCombination(..)` 的位置了吗？ 为什么这样也能运行？

Because **the shape of a `reducer(..)` and the shape of `listCombination(..)` are the same.** In other words, a reducer can be used as a combination function for another reducer; that's how they compose! The `listCombination(..)` function makes the first reducer, then *that reducer* can be as the combination function to make the next reducer, and so on.

**因为 `reducer(..)` 的“形状”和 `listCombination(..)` 的形状是一样的。** 换句话说，reducer 可以用作另一个 reducer 的组合函数; 所以它们可以组合起来！  `listCombination(..)` 函数作为第一个 reducer 的组合函数，这个 reducer 又可以作为组合函数给下一个 reducer，以此类推。

Let's test out our `longAndShortEnoughReducer(..)` with a few different values:

我们用几个不同的值来测试我们的 `longAndShortEnoughReducer(..)` ：

```js
longAndShortEnoughReducer( [], "nope" );
// []

longAndShortEnoughReducer( [], "hello" );
// ["hello"]

longAndShortEnoughReducer( [], "hello world" );
// []
```

The `longAndShortEnoughReducer(..)` utility is filtering out both values that are not long enough and values that are not short enough, and it's doing both these filterings in the same step. It's a composed reducer!

`longAndShortEnoughReducer(..)` 会过滤出不够长且不够短的值，它在同一步骤中执行这两个过滤。 这是一个组合 reducer！

Take another moment to let that sink in. It still kinda blows my mind.

再来一遍，它仍然让我有点摸不着头脑。

Now, to bring `x(..)` (the uppercase reducer producer) into the composition:

现在，把 `x(..)` （生成大写 reducer 的产生器）加入组合：

```js
var longAndShortEnoughReducer = y( z( listCombination) );
var upperLongAndShortEnoughReducer = x( longAndShortEnoughReducer );
```

As the name `upperLongAndShortEnoughReducer(..)` implies, it does all three steps at once -- a mapping and two filters! What it kinda look likes internally:

正如 `upperLongAndShortEnoughReducer(..)` 名字所示，它同时执行所有三个步骤 - 一个映射和两个过滤器！它内部看起来是这样的：

```js
// upperLongAndShortEnoughReducer:
function reducer(list,val) {
	return longAndShortEnoughReducer( list, strUppercase( val ) );
}
```

A string `val` is passed in, uppercased by `strUppercase(..)` and then passed along to `longAndShortEnoughReducer(..)`. *That* function only conditionally adds this uppercased string to the `list` if it's both long enough and short enough. Otherwise, `list` will remain unchanged.

一个字符串类型的 `val` 被传入，由  `strUppercase(..)` 转换成大写，然后传递给  `longAndShortEnoughReducer(..)`。 该函数只有在 `val` 满足足够长且足够短的条件时才将它添加到数组中。 否则数组保持不变。

It took my brain weeks to fully understand the implications of that juggling. So don't worry if you need to stop here and re-read a few (dozen!) times to get it. Take your time.

我的脑子花了几个星期来充分了解这种杂耍似的操作。 所以别着急，如果你需要在这好好研究下，重新阅读个几（十几个）次。 慢慢来。

Now let's verify:

现在来验证一下：

```js
upperLongAndShortEnoughReducer( [], "nope" );
// []

upperLongAndShortEnoughReducer( [], "hello" );
// ["HELLO"]

upperLongAndShortEnoughReducer( [], "hello world" );
// []
```

This reducer is the composition of the map and both filters! That's amazing!

这个 reducer 成功的组合了和 map 和两个 reducer！ 太棒了！

Let's recap where we're at so far:

让我们回顾一下我们到目前为止：

```js
var x = curriedMapReducer( strUppercase );
var y = curriedFilterReducer( isLongEnough );
var z = curriedFilterReducer( isShortEnough );

var upperLongAndShortEnoughReducer = x( y( z( listCombination ) ) );

words.reduce( upperLongAndShortEnoughReducer, [] );
// ["WRITTEN","SOMETHING"]
```

That's pretty cool. But let's make it even better.

这已经很酷了，但时我们可以让它更好。

`x(y(z( .. )))` is a composition. Let's skip the intermediate `x` / `y` / `z` variable names, and just express that composition directly:

`x(y(z( .. )))`  是一个组合。 我们可以直接跳过中间的 `x` / `y` / `z` 变量名，直接这么表示该组合：

```js
var composition = compose(
	curriedMapReducer( strUppercase ),
	curriedFilterReducer( isLongEnough ),
	curriedFilterReducer( isShortEnough )
);

var upperLongAndShortEnoughReducer = composition( listCombination );

words.reduce( upperLongAndShortEnoughReducer, [] );
// ["WRITTEN","SOMETHING"]
```

Think about the flow of "data" in that composed function:

我们来考虑下该组合函数中“数据”的流动：

1. `listCombination(..)` flows in as the combination function to make the filter-reducer for `isShortEnough(..)`.

2. *That* resulting reducer function then flows in as the combination function to make the filter-reducer for `isLongEnough(..)`.

3. Finally, *that* resulting reducer function flows in as the combination function to make the map-reducer for `strUppercase(..)`.

   ​

4. `listCombination(..)` 作为组合函数传入构造 `isShortEnough(..)` 过滤器的 reducer。

5. 然后，所得到的 reducer 函数作为组合函数传入继续构造  `isShortEnough(..)` 过滤器的 reducer。

6. 最后，所得到的 reducer 函数作为组合函数传入构造  `strUppercase(..)`映射的reducer。

In the previous snippet, `composition(..)` is a composed function expecting a combination function to make a reducer; `composition(..)` has a special label: transducer. Providing the combination function to a transducer produces the composed reducer:

在前面的片段中，`composition(..)` 是一个组合函数，期望组合函数来形成一个 reducer; 而这个 `composition(..)` 有一个特殊的标签：transducer。 给transducer提供组合函数产生组合的reducer：

// TODO: fact-check if the transducer *produces* the reducer or *is* the reducer

// TODO：检查 transducer 是产生 reducer 还是它本身就是 reducer

```js
var transducer = compose(
	curriedMapReducer( strUppercase ),
	curriedFilterReducer( isLongEnough ),
	curriedFilterReducer( isShortEnough )
);

words
.reduce( transducer( listCombination ), [] );
// ["WRITTEN","SOMETHING"]
```

**Note:** We should make an observation about the `compose(..)` order in the previous two snippets, which may be confusing. Recall that in our original example chain, we `map(strUppercase)` and then `filter(isLongEnough)` and finally `filter(isShortEnough)`; those operations indeed happen in that order. But in Chapter 4, we learned that `compose(..)` typically has the effect of running its functions in reverse order of listing. So why don't we need to reverse the order *here* to get the same desired outcome? The abstraction of the `combinationFn(..)` from each reducer reverses the effective applied order of operations under the hood. So counter-intuitively, when composing a tranducer, you actually want to list them in desired order of execution!

注意：我们应该好好观察下前面两个片段中的 `compose(..)` 顺序，这地方有点难理解。 回想一下，在我们的原始示例中，我们先 `map(strUppercase)` 然后 `filter(isLongEnough)` ，最后 `filter(isShortEnough)`; 最后也确实是按照这个顺序。 但在第 4 章中，我们了解到，`compose(..)` 通常是以相反的顺序运行。 那么为什么我们不需要反转这里的顺序来获得同样的期望结果呢？ 来自每个 reducer 的 `combinationFn(..)` 的抽象反转了操作顺序。 所以和直觉相反，当组合一个 tranducer 时，你只需要按照实际的顺序组合就好！

#### List Combination: Pure vs Impure

### 列表组合：纯与不纯

As a quick aside, let's revisit our `listCombination(..)` combination function implementation:

我们再来看一下我们的 `listCombination(..)` 组合函数的实现：

```js
function listCombination(list,val) {
	return list.concat( [val] );
}
```

While this approach is pure, it has negative consequences for performance. First, it creates the `[..]` temporary array wrapped around `val`. Then, `concat(..)` creates a whole new array to append this temporary array onto. For each step in our composed reduction, that's a lot of arrays being created and thrown away, which is not only bad for CPU but also GC memory churn.

虽然这种方法是纯的，但它对效率有负面影响。 首先，它创建临时数组来包裹 `val`。 然后，`concat(..)` 方法创建一个全新的数组来连接这个临时数组。 每一步都会创建和销毁的很多数组，这不仅对 CPU 不利，也会造成 GC 内存的流失。

The better-performing, impure version:

下面是性能更好但是不纯的版本：

```js
function listCombination(list,val) {
	list.push( val );
	return list;
}
```

Thinking about `listCombination(..)` in isolation, there's no question it's impure and that's usually something we'd want to avoid. However, there's a bigger context we should consider.

单独的考虑下 `listCombination(..)` ，毫无疑问，这是不纯的，这通常是我们想要避免的。 但是，我们应该考虑一个更大的背景。

`listCombination(..)` is not a function we interact with at all. We don't directly use it anywhere in the program, instead we let the transducing process use it.

`listCombination(..)` 不是我们完全有交互的函数。 我们不直接在程序中的任何地方使用它，而是在 transducing 的过程中使用它。

Back in Chapter 5, we asserted that our goal with reducing side effects and defining pure functions was only that we expose pure functions to the API level of functions we'll use throughout our program. We observed that under the covers, inside a pure function, it can cheat for performance sake all it wants, as long as it doesn't violate the external contract of purity.

回到第 5 章，我们定义纯函数来减少副作用的目标只是限制在应用的 API 层级。对于底层实现，只要没有违反对外部是纯函数，就可以纯函数内为了性能而不纯。

`listCombination(..)` is more an internal implementation detail of the transducing -- in fact, it'll often be provided by the transducing library for you! -- rather than a top-level method you'd interact with on a normal basis throughout your program.

`listCombination(..)` 更多的是转换的内部实现细节。实际上，它通常由 transducing 库提供！ 而不是你的程序中进行交互的顶级方法。

Bottom line: I think it's perfectly acceptable, and advisable even, to use the performance-optimal impure version of `listCombination(..)`. Just make sure you document that it's impure with a code comment!

底线：我认为甚至使用  `listCombination(..)` 的性能最优但是不纯的版本也是完全可以接受的。 只要确保你记录它是不纯的代码注释！

### Alternate Combination

#### 替代组合

So far, this is what we've derived with transducing:

到目前为止，这是我们用转换所得到的：

```js
words
.reduce( transducer( listCombination ), [] )
.reduce( strConcat, "" );
// WRITTENSOMETHING
```

That's pretty good, but we have one final trick up our sleeve with transducing. And frankly, I think this part is what makes all this mental effort you've expended thus far, actually worth it.

这已经非常棒了，但是我们还藏着最后一个的技巧。 坦白来说，我认为这部分能够让你迄今为止付出的所有努力变得值得。

Can we somehow "compose" these two `reduce(..)` calls to get it down to just one `reduce(..)`? Unfortunately, we can't just add `strConcat(..)` into the `compose(..)` call; its shape is not correct for that kind of composition.

我们可以用某种方式实现只用一个  `reduce(..)`  来“组合”这两个 `reduce(..)` 吗？ 不幸的是，我们并不能将 `strConcat(..)` 添加到 `compose(..)` 调用中; 它的“形状”不适用于那个组合。

But let's look at these two functions side-by-side:

但是让我们来看下这两个功能：

```js
function strConcat(str1,str2) { return str1 + str2; }

function listCombination(list,val) { list.push( val ); return list; }
```

If you squint your eyes, you can almost see how these two functions are interchangable. They operate with different data types, but conceptually they do the same thing: combine two values into one.

如果你用心观察，可以看出这两个功能是如何互换的。 它们以不同的数据类型运行，但在概念上它们也是一样的：将两个值组合成一个。

In other words, `strConcat(..)` is a combination function!

换句话说， `strConcat(..)`  是一个组合函数！

That means that we can use *it* instead of `listCombination(..)` if our end goal is to get a string concatenation rather than a list:

这意味着如果我们的最终目标是获得字符串连接而不是数组，我们就可以用它代替 `listCombination(..)` ：

```js
words.reduce( transducer( strConcat ), "" );
// WRITTENSOMETHING
```

Boom! That's transducing for you. I won't actually drop the mic here, but just gently set it down...

Boom！ 这就是 transducing。

## What, Finally

## 最后

Take a deep breath. That was a lot to digest.

深吸一口气，确实有很多要消化。

Clearing our brains for a minute, let's turn our attention back to just using transducing in our applications without jumping through all those mental hoops to derive how it works.

放空我们的大脑，让我们把注意力转移到如何在我们的程序中使用转换，而不关心它的工作原理。

Recall the helpers we defined earlier; let's rename them for clarity:

回想起我们之前定义的辅助函数，为清楚起见，我们重新命名一下：

```js
var transduceMap = curry( function mapReducer(mapperFn,combinationFn){
	return function reducer(list,v){
		return combinationFn( list, mapperFn( v ) );
	};
} );

var transduceFilter = curry( function filterReducer(predicateFn,combinationFn){
	return function reducer(list,v){
		if (predicateFn( v )) return combinationFn( list, v );
		return list;
	};
} );
```

Also recall that we use them like this:

还记得我们这样使用它们：

```js
var transducer = compose(
	transduceMap( strUppercase ),
	transduceFilter( isLongEnough ),
	transduceFilter( isShortEnough )
);
```

`transducer(..)` still needs a combination function (like `listCombination(..)` or `strConcat(..)`) passed to it to produce a transduce-reducer function, which then can then be used (along with an initial value) in `reduce(..)`.

`transducer(..)` 仍然需要一个组合函数（如 `listCombination(..)` 或 `strConcat(..)`）来产生一个传递给  `reduce(..)` （连同初始值）的 transduce-reducer 函数。

But to express all these transducing steps more declaratively, let's make a `transduce(..)` utility that does these steps for us:

但是为了更好的表达所有这些转换步骤，我们来做一个 `transduce(..)` 工具来为我们做这些步骤：

```js
function transduce(transducer,combinationFn,initialValue,list) {
	var reducer = transducer( combinationFn );
	return list.reduce( reducer, initialValue );
}
```

Here's our running example, cleaned up:

这是我们的运行示例：

```js
var transducer = compose(
	transduceMap( strUppercase ),
	transduceFilter( isLongEnough ),
	transduceFilter( isShortEnough )
);

transduce( transducer, listCombination, [], words );
// ["WRITTEN","SOMETHING"]

transduce( transducer, strConcat, "", words );
// WRITTENSOMETHING
```

Not bad, huh!? See the `listCombination(..)` and `strConcat(..)` functions used interchangably as combination functions?

不错，嗯！ 看到 `listCombination(..)` 和 `strConcat(..)` 函数可以互换使用组合函数了吗？

### Transducers.js

Finally, let's illustrate our running example using the `transducers-js` library (https://github.com/cognitect-labs/transducers-js):

最后，我们来说明我们运行的例子，使用sensors-js库（https://github.com/cognitect-labs/transducers-js）：

```js
var transformer = transducers.comp(
	transducers.map( strUppercase ),
	transducers.filter( isLongEnough ),
	transducers.filter( isShortEnough )
);

transducers.transduce( transformer, listCombination, [], words );
// ["WRITTEN","SOMETHING"]

transducers.transduce( transformer, strConcat, "", words );
// WRITTENSOMETHING
```

Looks almost identical to above.

看起来几乎与上述相同。

**Note:** The above snippet uses `transformers.comp(..)` since the library provides it, but in this case our `compose(..)` from Chapter 4 would produce the same outcome. In other words, composition itself isn't a transducing-sensitive operation.

The composed function in this snippet is named `transformer` instead of `transducer`. That's because if we call `transformer(listCombination)` (or `transformer(strConcat)`), we won't get a straight up transduce-reducer function as earlier.

`transducers.map(..)` and `transducers.filter(..)` are special helpers that adapt regular predicate or mapper functions into functions that produce a special transform object (with the transducer function wrapped underneath); the library uses these transform objects for transducing. The extra capabilities of this transform object abstraction are beyond what we'll explore, so consult the library's documentation for more information.

Since calling `transformer(..)` produces a transform object and not a typical two-arity transduce-reducer function, the library also provides `toFn(..)` to adapt the transform object to be useable by native array `reduce(..)`:

注意：上面的代码段使用 `transformers.comp(..)` ，因为这个库提供这个 API，但在这种情况下，我们从第 4 章的 `compose(..)` 也将产生相同的结果。换句话说，组合本身不是 transducing 敏感的操作。

该片段中的组合函数被称为 `transformer` ，而不是 `transducer`。那是因为如果我们直接调用 `transformer(listCombination)`（或 `transformer(strConcat)`），那么我们不会像以前那样得到一个直观的 transduce-reducer 函数。

`transducers.map(..)` 和 `transducers.filter(..)` 是特殊的辅助函数，可以将常规的断言函数或映射函数转换成适用于产生特殊变换对象的函数（里面包含了 reducer 函数）;这个库使用这些变换对象进行转换。此转换对象抽象的额外功能超出了我们将要探索的内容，请参阅该库的文档以获取更多信息。

由于 `transformer(..)` 产生一个变换对象，而不是一个典型的 two-arity transduce-reducer 函数，该库还提供 `toFn(..)` 来使变换对象适应本地数组 `reduce(..)`：

```js
words.reduce(
	transducers.toFn( transformer, strConcat ),
	""
);
// WRITTENSOMETHING
```

`into(..)` is another provided helper that automatically selects a default combination function based on the type of empty/initial value specified:

`into(..)` 是另一个提供的辅助函数，它根据指定的空/初始值的类型自动选择默认的组合函数：

```js
transducers.into( [], transformer, words );
// ["WRITTEN","SOMETHING"]

transducers.into( "", transformer, words );
// WRITTENSOMETHING
```

When specifying an empty `[]` array, the `transduce(..)` called under the covers uses a default implementation of a function like our `listCombination(..)` helper. But when specifying an empty `""` string, something like our `strConcat(..)` is used. Cool!

当指定一个空 `[]` 数组时，被覆盖下的 `transduce(..)` 使用一个像我们的 `listCombination(..)` 辅助函数的函数的默认实现。 但是当指定一个空的“”字符串时，会使用像我们的 `strConcat(..)` 这样的东西。 这很酷！

As you can see, the `transducers-js` library makes transducing pretty straightforward. We can very effectively leverage the power of this technique without getting into the weeds of defining all those intermediate transducer-producing utilities ourselves.

如你所见， `transducers-js`  库使转换非常简单。 我们可以非常有效地利用这种技术的力量，而不用陷入自定义所有这些中间传感器的繁琐程序中。

## Summary

To transduce means to transform with a reduce. More specifically, a transducer is a composable reducer.

We use transducing to compose adjacent `map(..)`, `filter(..)`, and `reduce(..)` operations together. We accomplish this by first expressing `map(..)`s and `filter(..)`s as `reduce(..)`s, and then abstracting out the common combination operation to create unary reducer-producing functions that are easily composed.

Transducing primarily improves performance, which is especially obvious if used on a lazy sequence (async observable).

But more broadly, transducing is how we express a more declarative composition of functions that would otherwise not be directly composable. The result, if used appropriately as with all other techniques in this book, is clearer, more readable code! A single `reduce(..)` call with a transducer is easier to reason about than tracking multiple `reduce(..)` calls.

## 概要

transduce 意味着减少转化。 更具体点，transduer 是可组合的 reducer。

我们使用转换来组合相邻的`map(..)`，`filter(..)`，和 `reduce(..)` 操作。 我们首先将 `map(..)` 和`filter(..)` 表示为`reduce(..)`，然后抽象出常用的组合操作来创建一个容易组合的一致的 reducer 生成函数。

transducing 主要提高性能，如果在延迟序列（异步可观察）中使用，则这一点尤为明显。

但是更广泛地说，transducing 是我们如何表达更多的声明性组合的功能，否则这些函数将不能直接组合。 如果使用这个技术能像使用本书中的所有其他技术一样用的好处，代码就会显得更清晰，易读！ 使用 transducer 进行单次 `reduce(..)` 调用比追踪多个 `reduce(..)` 调用更容易理解。

