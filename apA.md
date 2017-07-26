# JavaScript轻量级函数式编程
# 附录 A：Transducing

Transducing 是我们这本书要讲到的更为高级的技术。它继承了第 8 章数组操作的许多思想。

我不会把 Transducing 严格的称为“轻量级函数式编程”，它更像是一个顶级的技巧。我把这个技术留到附录来讲意味着你现在很可能并不需要关心它，当你确保你已经理解了整本书的主要内容而且比较熟练了，你可以再回头看看这一章节。

说实话，即使我已经教过 transducing 很多次了，在写这一章的时候，我仍然需要花很多脑力去理清楚这个技术。所以，如果你看这一章看的很疑惑也没必要感到沮丧。把这一章加个书签，等你觉得你差不多能理解时再回头看看。

Transducing 就是通过减少来转换。

我知道这听起来很令人费解。但是让我们来看看它有多强大。实际上，我认为这是你掌握了轻量级函数式编程后可以做的最好的例证之一。

和这本书的其他部分一样，我的方法是先解释*为什么*使用这个技术，然后*如何*使用，最后归结为简单的这个技术到底是什么样的。这通常会有多学很多东西，但是我觉得用这种方式你会更深入的理解它。

## 首先，为什么

让我们从扩展我们在第 3 章中介绍的例子开始，测试单词是否足够短和/或足够长：

```js
function isLongEnough(str) {
	return str.length >= 5;
}

function isShortEnough(str) {
	return str.length <= 10;
}
```

在第 3 章中，我们使用这些断言函数来测试一个单词。然后在第 8 章中，我们学习了如何使用像 `filter(..)` 这样的数组操作来重复这些测试。例如:

```js
var words = [ "You", "have", "written", "something", "very", "interesting" ];

words
.filter( isLongEnough )
.filter( isShortEnough );
// ["written","something"]
```

这个例子可能并不明显，但是这种分开操作相同数组的方式具有一些不理想的地方。当我们处理一个值比较少的数组时一切都还好。但是如果数组中有很多值，每个 `filter(..)` 分开处理数组的值会比我们预期的慢一点。

当我们的数组是异步/懒惰（也称为 observables），随着时间的推移响应事件处理（见第 10 章），会出现类似的性能问题。在这种情况下，一次事件只有一个值，因此使用两个单独的 `filter(..)` 函数处理这些值并不是一件很大的事情。

但是，不太明显的是每个 `filter(..)` 方法都会产生一个单独的 observable 值。从一个 observable 值中抽出一个值的开销真的可以加起来。这是特别真实的，因为在这些情况下，处理数千或数百万的值并不罕见; 所以，即使是这么小的成本也会很快累计起来。

另一个缺点是可读性，特别是当我们需要对多个数组（或观察值）重复相同的操作时。例如:

```js
zip(
	list1.filter( isLongEnough ).filter( isShortEnough ),
	list2.filter( isLongEnough ).filter( isShortEnough ),
	list3.filter( isLongEnough ).filter( isShortEnough )
)
```

显得很重复，对不对？

如果我们可以将 `isLongEnough(..)` 断言与 `isShortEnough(..)` 断言组合在一起是不是会更好一点呢（可读性和性能）？你可以手动执行：

```js
function isCorrectLength(str) {
	return isLongEnough( str ) && isShortEnough( str );
}
```

但这不是 FP 的方式！

在第 8 章中，我们讨论了融合 —— 组合相邻映射函数。回忆一下：

```js
words
.map(
	pipe( removeInvalidChars, upper, elide )
);
```

不幸的是，组合相邻断言函数并不像组合相邻映射函数那样容易。为什么呢？想想断言函数长什么“样子” —— 一种描述输入和输出的学术方式。它接收一个单一的参数，返回一个 true 或false。

如果你试着用 `isshortenough(islongenough(str))`，这是行不通的。因为 `islongenough(..) ` 会返回 true 或者 false ，而不是返回 `isshortenough(..)` 所要的字符串类型的值。这可真倒霉。

试图组合两个相邻的 reducer 函数同样是行不通的。reducer 函数接收两个值作为输入，并返回单个组合值。reducer 函数的单一返回值也不能作为参数传到另一个需要两个输入的 reducer 函数中。

此外，`reduce(..)` 辅助函数可以接收一个可选的 `initialValue` 输入。有时可以省略，但有时候它又必须被传入。这就让组合更复杂了，因为一个 `reduce(..)` 可能需要一个 `initialValue`，而另一个 `reduce(..)` 可能需要另一个 `initialValue`。所以我们怎么可能只用某种组合的 reducer 来实现 `reduce(..)` 呢。

考虑像这样的链：

```js
words
.map( strUppercase )
.filter( isLongEnough )
.filter( isShortEnough )
.reduce( strConcat, "" );
// "WRITTENSOMETHING"
```

你能想出一个组合能够包含 `map(strUppercase)`， `filter(isLongEnough)`，`filter(isShortEnough)`， `reduce(strConcat)` 所有这些操作吗？每种操作的行为是不同的，所以不能直接组合在一起。我们需要把它们修改下让它们组合在一起。

希望这些例子说明了为什么简单的组合不能胜任这项任务。我们需要一个更强大的技术，而 transducing 就是这个技术。

## 如何，下一步

让我们谈谈我们该如何得到一个能组合映射，断言和/或 reducers 的框架。

别太紧张：你不必经历编程过程中所有的探索步骤。一旦你理解了 transducing 能解决的问题，你就可以直接使用 FP 库中的 `transduce(..)` 工具继续你的应用程序的剩余部分！

让我们开始探索吧。

### 把 Map/Filter 表示为 Reduce

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

这是一个不错的改进。我们现在有四个相邻的 `reduce(..)` 调用，而不是三种不同方法的混合。然而，我们仍然不能 `compose(..)` 这四个 reducer，因为它们接受两个参数而不是一个参数。

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

在后面我们会来头看看这里是否需要 `concat(..)`。

### 参数化 Reducers

除了使用不同的断言函数之外，两个 filter reducers 几乎相同。让我们把这些 reducers 参数化得到一个可以定义任何 filter-reducer 的工具函数：

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

同样的，我们把 `mapperFn(..)` 也参数化来生成 map-reducer 函数：

```js
function mapReducer(mapperFn) {
	return function reducer(list,val){
		return list.concat( [mapperFn( val )] );
	};
}

var strToUppercaseReducer = mapReducer( strUppercase );
```

我们的调用链看起来是一样的：

```js
words
.reduce( strUppercaseReducer, [] )
.reduce( isLongEnoughReducer, [] )
.reduce( isShortEnough, [] )
.reduce( strConcat, "" );
```

### 提取共用组合逻辑

仔细观察上面的 `mapReducer(..)` 和 `filterReducer(..)` 函数。你发现共享功能了吗？

这部分：

```js
return list.concat( .. );

// 或者
return list;
```

让我们为这个通用逻辑定义一个辅助函数。但是我们叫它什么呢？

```js
function WHATSITCALLED(list,val) {
	return list.concat( [val] );
}
```

`WHATSITCALLED(..)` 函数做了些什么呢，它接收两个参数（一个数组和另一个值），将值 concat 到数组的末尾返回一个新的数组。所以这个 `WHATSITCALLED(..)` 名字不合适，我们可以叫它 `listCombination(..)` ：

```js
function listCombination(list,val) {
	return list.concat( [val] );
}
```

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

我们的调用链看起来还是一样的（这里就不重复写了）。

### 参数化组合

我们的 `listCombination(..)` 小工具只是组合两个值的一种方式。让我们将它的用途参数化，以使我们的 reducers 更加通用：

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

使用这种形式的辅助函数：

```js
var strToUppercaseReducer = mapReducer( strUppercase, listCombination );
var isLongEnoughReducer = filterReducer( isLongEnough, listCombination );
var isShortEnoughReducer = filterReducer( isShortEnough, listCombination );
```

将这些实用函数定义为接收两个参数而不是一个参数不太方便组合，因此我们使用我们的 `curry(..)` （柯里化）方法：

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

这看起来有点冗长而且可能不是很有用。

但这实际上是我们进行下一步推导的必要条件。请记住，我们的最终目标是能够 `compose(..)` 这些 reducers。我们快要完成了。

### 组合柯里化

这一步是最棘手的。所以请慢慢的用心的阅读。

让我们看看没有将 `listCombination(..)` 传递给柯里化函数的样子：

```js
var x = curriedMapReducer( strUppercase );
var y = curriedFilterReducer( isLongEnough );
var z = curriedFilterReducer( isShortEnough );
```

看看这三个中间函数 `x(..)`， `y(..)` 和 `z(..)`。每个函数都期望得到一个单一的组合函数并产生一个 reducer 函数。

记住，如果我们想要所有这些的独立的 reducer，我们可以这样做：

```js
var upperReducer = x( listCombination );
var longEnoughReducer = y( listCombination );
var shortEnoughReducer = z( listCombination );
```

但是，如果你调用 `y(z)`，会得到什么呢？当把 `z` 传递给 `y(..)` 调用，而不是 `combinationFn(..)` 时会发生什么呢？这个返回的 reducer 函数内部看起来像这样：

```js
function reducer(list,val) {
	if (isLongEnough( val )) return z( list, val );
	return list;
}
```

看到 `z(..)` 里面的调用了吗？ 这看起来应该是错误的，因为 `z(..)` 函数应该只接收一个参数（`combinationFn(..)`），而不是两个参数（list 和 val）。这和要求不匹配。不行。

我们来看看组合 `y(z(listCombination))`。我们将把它分成两个不同的步骤：

```js
var shortEnoughReducer = z( listCombination );
var longAndShortEnoughReducer = y( shortEnoughReducer );
```

我们创建 `shortEnoughReducer(..)`，然后将它作为 `combinationFn(..)` 传递给 `y(..)`，生成 `longAndShortEnoughReducer(..)`。多读几遍，直到理解。

现在想想： `shortEnoughReducer(..)` 和 `longAndShortEnoughReducer(..)` 的内部构造是什么样的呢？你能想得到吗？

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

你看到 `shortEnoughReducer(..)` 替代了 `longAndShortEnoughReducer(..)` 里面 `listCombination(..)` 的位置了吗？ 为什么这样也能运行？

**因为 `reducer(..)` 的“形状”和 `listCombination(..)` 的形状是一样的。** 换句话说，reducer 可以用作另一个 reducer 的组合函数; 它们就是这样组合起来的！  `listCombination(..)` 函数作为第一个 reducer 的组合函数，这个 reducer 又可以作为组合函数给下一个 reducer，以此类推。

我们用几个不同的值来测试我们的 `longAndShortEnoughReducer(..)` ：

```js
longAndShortEnoughReducer( [], "nope" );
// []

longAndShortEnoughReducer( [], "hello" );
// ["hello"]

longAndShortEnoughReducer( [], "hello world" );
// []
```

`longAndShortEnoughReducer(..)` 会过滤出不够长且不够短的值，它在同一步骤中执行这两个过滤。这是一个组合 reducer！

再花点时间消化下。

现在，把 `x(..)` （生成大写 reducer 的产生器）加入组合：

```js
var longAndShortEnoughReducer = y( z( listCombination) );
var upperLongAndShortEnoughReducer = x( longAndShortEnoughReducer );
```

正如 `upperLongAndShortEnoughReducer(..)` 名字所示，它同时执行所有三个步骤 - 一个映射和两个过滤器！它内部看起来是这样的：

```js
// upperLongAndShortEnoughReducer:
function reducer(list,val) {
	return longAndShortEnoughReducer( list, strUppercase( val ) );
}
```

一个字符串类型的 `val` 被传入，由 `strUppercase(..)` 转换成大写，然后传递给  `longAndShortEnoughReducer(..)`。该函数只有在 `val` 满足足够长且足够短的条件时才将它添加到数组中。否则数组保持不变。

我的脑子花了几个星期来充分了解这种杂耍似的操作。所以别着急，如果你需要在这好好研究下，重新阅读个几（十几个）次。慢慢来。

现在来验证一下：

```js
upperLongAndShortEnoughReducer( [], "nope" );
// []

upperLongAndShortEnoughReducer( [], "hello" );
// ["HELLO"]

upperLongAndShortEnoughReducer( [], "hello world" );
// []
```

这个 reducer 成功的组合了和 map 和两个 filter，太棒了！

让我们回顾一下我们到目前为止所做的事情：

```js
var x = curriedMapReducer( strUppercase );
var y = curriedFilterReducer( isLongEnough );
var z = curriedFilterReducer( isShortEnough );

var upperLongAndShortEnoughReducer = x( y( z( listCombination ) ) );

words.reduce( upperLongAndShortEnoughReducer, [] );
// ["WRITTEN","SOMETHING"]
```

这已经很酷了，但是我们可以让它更好。

`x(y(z( .. )))`  是一个组合。我们可以直接跳过中间的 `x` / `y` / `z` 变量名，直接这么表示该组合：

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

我们来考虑下该组合函数中“数据”的流动：

1. `listCombination(..)` 作为组合函数传入，构造 `isShortEnough(..)` 过滤器的 reducer。

2. 然后，所得到的 reducer 函数作为组合函数传入，继续构造 `isShortEnough(..)` 过滤器的 reducer。

3. 最后，所得到的 reducer 函数作为组合函数传入，构造 `strUppercase(..)` 映射的 reducer。

在前面的片段中，`composition(..)` 是一个组合函数，期望组合函数来形成一个 reducer；而这个 `composition(..)` 有一个特殊的标签：transducer。给 transducer 提供组合函数产生组合的 reducer：

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

**注意**：我们应该好好观察下前面两个片段中的 `compose(..)` 顺序，这地方有点难理解。回想一下，在我们的原始示例中，我们先 `map(strUppercase)` 然后 `filter(isLongEnough)` ，最后 `filter(isShortEnough)`；这些操作实际上也确实按照这个顺序执行的。但在第 4 章中，我们了解到，`compose(..)` 通常是以相反的顺序运行。那么为什么我们不需要反转这里的顺序来获得同样的期望结果呢？来自每个 reducer 的 `combinationFn(..)` 的抽象反转了操作顺序。所以和直觉相反，当组合一个 tranducer 时，你只需要按照实际的顺序组合就好！

#### List Combination: Pure vs Impure

### 列表组合：纯与不纯

我们再来看一下我们的 `listCombination(..)` 组合函数的实现：

```js
function listCombination(list,val) {
	return list.concat( [val] );
}
```

虽然这种方法是纯的，但它对性能有负面影响。首先，它创建临时数组来包裹 `val`。然后，`concat(..)` 方法创建一个全新的数组来连接这个临时数组。每一步都会创建和销毁的很多数组，这不仅对 CPU 不利，也会造成 GC 内存的流失。

下面是性能更好但是不纯的版本：

```js
function listCombination(list,val) {
	list.push( val );
	return list;
}
```

单独的考虑下 `listCombination(..)` ，毫无疑问，这是不纯的，这通常是我们想要避免的。但是，我们应该考虑一个更大的背景。

`listCombination(..)` 不是我们完全有交互的函数。我们不直接在程序中的任何地方使用它，而只是在 transducing 的过程中使用它。

回到第 5 章，我们定义纯函数来减少副作用的目标只是限制在应用的 API 层级。对于底层实现，只要没有违反对外部是纯函数，就可以在函数内为了性能而变得不纯。

`listCombination(..)` 更多的是转换的内部实现细节。实际上，它通常由 transducing 库提供！而不是你的程序中进行交互的顶层方法。

底线：我认为甚至使用  `listCombination(..)` 的性能最优但是不纯的版本也是完全可以接受的。只要确保你用代码注释记录下它不纯即可！

### 可选的组合

到目前为止，这是我们用转换所得到的：

```js
words
.reduce( transducer( listCombination ), [] )
.reduce( strConcat, "" );
// 写点什么
```

这已经非常棒了，但是我们还藏着最后一个的技巧。坦白来说，我认为这部分能够让你迄今为止付出的所有努力变得值得。

我们可以用某种方式实现只用一个 `reduce(..)` 来“组合”这两个 `reduce(..)` 吗？ 不幸的是，我们并不能将 `strConcat(..)` 添加到 `compose(..)` 调用中; 它的“形状”不适用于那个组合。

但是让我们来看下这两个功能：

```js
function strConcat(str1,str2) { return str1 + str2; }

function listCombination(list,val) { list.push( val ); return list; }
```

如果你用心观察，可以看出这两个功能是如何互换的。它们以不同的数据类型运行，但在概念上它们也是一样的：将两个值组合成一个。

换句话说， `strConcat(..)`  是一个组合函数！

这意味着如果我们的最终目标是获得字符串连接而不是数组，我们就可以用它代替 `listCombination(..)` ：

```js
words.reduce( transducer( strConcat ), "" );
// 写点什么
```

Boom！ 这就是 transducing。

## 最后

深吸一口气，确实有很多要消化。

放空我们的大脑，让我们把注意力转移到如何在我们的程序中使用转换，而不是关心它的工作原理。

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

还记得我们这样使用它们：

```js
var transducer = compose(
	transduceMap( strUppercase ),
	transduceFilter( isLongEnough ),
	transduceFilter( isShortEnough )
);
```

`transducer(..)` 仍然需要一个组合函数（如 `listCombination(..)` 或 `strConcat(..)`）来产生一个传递给  `reduce(..)` （连同初始值）的 transduce-reducer 函数。

但是为了更好的表达所有这些转换步骤，我们来做一个 `transduce(..)` 工具来为我们做这些步骤：

```js
function transduce(transducer,combinationFn,initialValue,list) {
	var reducer = transducer( combinationFn );
	return list.reduce( reducer, initialValue );
}
```

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
// 写点什么
```

不错，嗯！ 看到 `listCombination(..)` 和 `strConcat(..)` 函数可以互换使用组合函数了吗？

### Transducers.js

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

看起来几乎与上述相同。

**注意：**上面的代码段使用 `transformers.comp(..)` ，因为这个库提供这个 API，但在这种情况下，我们从第 4 章的 `compose(..)` 也将产生相同的结果。换句话说，组合本身不是 transducing 敏感的操作。

该片段中的组合函数被称为 `transformer` ，而不是 `transducer`。那是因为如果我们直接调用 `transformer(listCombination)`（或 `transformer(strConcat)`），那么我们不会像以前那样得到一个直观的 transduce-reducer 函数。

`transducers.map(..)` 和 `transducers.filter(..)` 是特殊的辅助函数，可以将常规的断言函数或映射函数转换成适用于产生特殊变换对象的函数（里面包含了 reducer 函数）；这个库使用这些变换对象进行转换。此转换对象抽象的额外功能超出了我们将要探索的内容，请参阅该库的文档以获取更多信息。

由于 `transformer(..)` 产生一个变换对象，而不是一个典型的二元 transduce-reducer 函数，该库还提供 `toFn(..)` 来使变换对象适应本地数组的 `reduce(..)` 方法：

```js
words.reduce(
	transducers.toFn( transformer, strConcat ),
	""
);
// WRITTENSOMETHING
```

`into(..)` 是另一个提供的辅助函数，它根据指定的空/初始值的类型自动选择默认的组合函数：

```js
transducers.into( [], transformer, words );
// ["WRITTEN","SOMETHING"]

transducers.into( "", transformer, words );
// WRITTENSOMETHING
```

当指定一个空 `[]` 数组时，内部的 `transduce(..)` 使用一个默认的函数实现，这个函数就像我们的 `listCombination(..)`。但是当指定一个空的“”字符串时，会使用像我们的 `strConcat(..)` 这样的东西。这很酷！

如你所见， `transducers-js`  库使转换非常简单。我们可以非常有效地利用这种技术的力量，而不至于陷入定义所有这些中间转换器生产工具的繁琐过程中去。

## 总结

Transduce 就是通过减少来转换。更具体点，transduer 是可组合的 reducer。

我们使用转换来组合相邻的`map(..)`、`filter(..)` 和 `reduce(..)` 操作。我们首先将 `map(..)` 和 `filter(..)` 表示为 `reduce(..)`，然后抽象出常用的组合操作来创建一个容易组合的一致的 reducer 生成函数。

transducing 主要提高性能，如果在延迟序列（异步 observables）中使用，则这一点尤为明显。

但是更广泛地说，transducing 是我们针对那些不能被直接组合的函数，使用的一种更具声明式风格的方法。否则这些函数将不能直接组合。如果使用这个技术能像使用本书中的所有其他技术一样用到好处，代码就会显得更清晰，易读！ 使用 transducer 进行单次 `reduce(..)` 调用比追踪多个 `reduce(..)` 调用更容易理解。

