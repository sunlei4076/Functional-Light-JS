# JavaScript 轻量级函数式编程
# 第 8 章：列表操作

你是否还沉迷于上一节介绍的闭包／对象之中？欢迎回来！

> 如果你能做一些令人惊叹的事情，请持续保持下去。

本文之前已经简要的提及了一些实用函数：`map(..)`、`filter(..)` 和 `reduce(..)`，现在深入了解一下它们。在 Javascript 中，这些实用函数通常被用于 Array（即 “list” ）的原型上。因此可以很自然的将这些实用函数和数组或列表操作联系起来。

在讨论具体的数组方法之前，我们应该很清楚这些操作的作用。在这章中，弄明白**为何**有这些列表操作和这些操作**如何**工作同等重要。请保持头脑清晰，跟上节奏。

在本章内外，有大量常见且通俗易懂的列表操作的例子，它们描述一些细小的操作去处理一系列的值（如数组中的每一个值加倍）。这样通俗易懂。

但是不要停留在这些简单示例的表面，而错过了更深层次的点。通过对一系列任务建模来理解一些非常重要的函数式编程在列表操作中的价值 —— 一些些看起来不像列表的语句 —— 作为列表操作，而不是单独执行。

这不仅仅是编写许多简练代码的技巧。我们所要做的是，从命令式转变为声明式风格，使代码模式更容易辨认，从而可读性更好。

但这里有一些**更需要掌握的**东西。在命令式代码中，一组计算的中间结果都是通过赋值来存储。代码中依赖的命令模式越多，越难验证它们不是错误。比如，在逻辑上，值的意外改变，或隐藏的潜在原因／影响。

通过与／或链接组合列表操作，中间结果被隐式地跟踪，并在很大程度上避免了这些风险。

**注意：** 相比前面几章，为了代码片段更加简练，我们将采用 ES6 的箭头函数。尽管第 2 章中对于箭头函数的建议依旧普遍适用于编码中。

## 非函数式编程列表处理

作为本章讨论的快速预览，我想调用一些操作，这些操作看上去可以将 Javascript 数组和函数式编程列表操作相关联，但事实上并没有。我们不会在这里讨论这些，因为它们与一般的函数式编程最佳实践不一致：

* `forEach(..)`
* `some(..)`
* `every(..)`

`forEach(..)` 是遍历辅助函数，但是它被设计为带有副作用的函数来处理每次遍历；你或许已经猜测到了它为什么不是我们正在讨论的函数式编程列表操作！

`some(..)` 和 `every(..)` 鼓励使用纯函数（具体来说，就像 `filter(..)` 这样的谓词函数），但是它们不可避免地将列表化简为 `true` 或 `false` 的值，本质上就像搜索和匹配。这两个实用函数和我们期望采用函数式编程来组织代码相匹配，因此，这里我们将跳过它们。

## 映射

我们将采用最基础和最简单的操作 `map(..)` 来开启函数式编程列表操作的探索。

映射的作用就将一个值转换为另一个值。例如，如果你将 `2` 乘以 `3`，你将得到转换的结果 `6` 。需要重点注意的是，我们并不是在讨论映射转换是暗示就地转换或重新赋值，而是将一个值从一个地方映射到另一个新的地方。

换句话说

```js
var x = 2, y;

// 转换／投影
y = x * 3;

// 变换／重新赋值
x = x * 3;
```
如果我们定义了乘 `3` 这样的函数，这个函数充当映射（转换）的功能。

```js
var multipleBy3 = v => v * 3;

var x = 2, y;

// 转换／投影
y = multiplyBy3( x );
```
我们可以自然的将映射的概念从单个值扩展到值的集合。`map(..)` 操作将列表中所有的值转换为新列表中的列表项，如下图所示：

<p align="center">
	<img src="fig9.png" width="400">
</p>

实现 `map(..)` 的代码如下：

```js
function map(mapperFn,arr) {
	var newList = [];

	for (let idx = 0; idx < arr.length; idx++) {
		newList.push(
			mapperFn( arr[idx], idx, arr )
		);
	}

	return newList;
}
```
**注意：** `mapperFn, arr` 的参数顺序，乍一看像是在倒退。但是这种方式在函数式编程类库中非常常见。因为这样做，可以让这些实用函数更容易被组合。

`mapperFn(..)` 自然地将传入的列表项做映射／转换，并且也传入了 `idx` 和 `arr`。这样做，可以和内置的数组的 `map(..)` 保持一致。在某些情况下，这些额外的参数非常有用。

但是，在一些其他情况中，你只希望传递列表项到 `mapperFn(..)`。因为额外的参数可能会改变它的行为。在第三章的“共同目的（ All for one ）”中，我们介绍了 unary(..)，它限制函数仅仅接受一个参数，不论多少个参数被传入。

回顾第三章关于把 `parseInt()` 的参数数量限制为 1，从而使之成为可被安全使用的 `mapperFn()` 的例子：

```js
map( ["1","2","3"], unary( parseInt ) );
// [1,2,3]
```

Javascript 提供了内置的数组操作方法 `map(..)`，这个方法使得列表中的链式操作更为便利。

**注意：** Javascript 数组中的原型中定义的操作( `map(..)`、`filter(..)` 和 `reduce(..)` )的最后一个可选参数可以被用于绑定 “this” 到当前函数。我们在第二章中曾经讨论过“什么是 this？”，以及在函数式编程的最佳实践中应该避免使用 `this`。基于这个原因，在这章中的示例中，我们不采用 `this` 绑定功能。

除了明显的字符和数字操作外，你可以对列表中的这些值类型进行操作。我们可以采用 `map(..)` 方法来通过函数列表转换得到这些函数返回的值，示例代码如下:

```js
var one = () => 1;
var two = () => 2;
var three = () => 3;

[one,two,three].map( fn => fn() );
// [1,2,3]
```
我们也可以先将函数放在列表中，然后组合列表中的每一个函数，最后执行它们，代码如下：

```js
var increment = v => ++v;
var decrement = v => --v;
var square = v => v * v;

var double = v => v * 2;

[increment,decrement,square]
.map( fn => compose( fn, double ) )
.map( fn => fn( 3 ) );
// [7,5,36]
```

我们注意到关于 `map(..)` 的一些有趣的事情：我们通常假定列表是从左往右执行的，但 `map(..)` 没有这个概念，它确实不需要这个次序。每一个转换应该独立于其他的转换。

映射普遍适用于并行处理的场景中，尤其在处理大列表时可以提升性能。但是在 Javascript 中，我们并没有看到这样的场景。因为这里不需要你传入诸如 `mapperFn(..)` 这样的纯函数，即便你**应当这样做**。如果传入了非纯函数，JS 在不同的顺序中执行不同的方法，这将很快产生大问题。

尽管从理论上讲，单个映射操作是独立的，但 JS 需要假定它们不是。这是令人讨厌的。

### 同步 vs 异步

这篇文章中讨论的列表操作都是同步地操作一组已经存在的值组成的列表，`map(..)` 在这里被看作是急切的操作。但另外一种思考方式是将映射函数作为时间处理器，该处理器会在新元素加入到列表中时执行。

想象一下这样的场景：

```js
var newArr = arr.map();

arr.addEventListener( "value", multiplyBy3 );
```

现在，任何时候，当一个值加入到 `arr` 中的时候，`multiplyBy3(..)` 事件处理器（映射函数）将加入的值当参数执行，将转换后的结果加入到 `newArr`。

我们建议，数组以及在数组上应用的数组操作都是迫切的同步的，然而，这些相同的操作也可以应用在一直接受新值的“惰性列表”（即流）上。我们将在第 10 章中深入讨论它。

### 映射 vs 遍历

有些人提倡在迭代的时候采用 `map(..)` 替代 `forEach(..)`，它本质上不会去触碰接受到的值，但仍有可能产生副作用：

```js
[1,2,3,4,5]
.map( function mapperFn(v){
	console.log( v );			// 副作用!
	return v;
} )
..
```
这种技术似乎非常有用的原因是 `map(..)` 返回数组，这样你可以在它之后继续链式执行更多的操作。而 `forEach(..)` 返回的的值是 `undefined`。然而，我认为你应当避免采用这种方式使用 `map(..)`，因为这里明显的以非函数式编程的方式使用核心的函数式编程操作，将引起巨大的困惑。

你应该听过一句老话，用合适的工具做合适的事，对吗？锤子敲钉子，螺丝刀拧螺丝等等。这里有些细微的不同：采用**恰当的方式**使用合适的工具。

锤子是挥动手敲的，如果你尝试采用嘴去钉钉子，效率会大打折扣。`map(..)` 是用来映射值的，而不是带来副作用。

### 一个词：函子

在这本书中，我们尽可能避免使用人为创造的函数式编程术语。我们有时候会使用官方术语，但在大多数时候，采用日常用语来描述更加通俗易懂。

这里我将被一个可能会引起恐慌的词：函子来短暂地打断这种通俗易懂的模式。这里之所以要讨论函子的原因是我们已经了解了它是干什么的，并且这个词在函数式编程文献中被大量使用。你不会被这个词吓到而带来副作用。

函子是采用运算函数有效用操作的值。

如果问题中的值是复合的，意味着它是由单个值组成，就像数组中的情况一样。例如，函子在每个单独的值上执行操作函数。函子实用函数创建的新值是所有单个操作函数执行的结果的组合。

这就是用 `map(..)` 来描述我们所看到东西的一种奇特方式。`map(..)` 函数采用关联值（数组）和映射函数（操作函数），并为数组中的每一个独立元素执行映射函数。最后，它返回由所有新映射值组成的新数组。

另一个例子：字符串函子是一个字符串加上一个实用函数，这个实用函数在字符串的所有字符上执行某些函数操作，返回包含处理过的字符的字符串。参考如下非常刻意的例子：

```js
function uppercaseLetter(c) {
	var code = c.charCodeAt( 0 );

	// 小写字母?
	if (code >= 97 && code <= 122) {
		// 转换为大写!
		code = code - 32;
	}

	return String.fromCharCode( code );
}

function stringMap(mapperFn,str) {
	return [...str].map( mapperFn ).join( "" );
}

stringMap( uppercaseLetter, "Hello World!" );
// 你好，世界!
```

`stringMap(..)` 允许字符串作为函子。你可以定义一个映射函数用于任何数据类型。只要实用函数满足这些规则，该数据结构就是一个函子。

## 过滤器

想象一下，我带着空篮子去逛食品杂货店的水果区。这里有很多水果（苹果、橙子和香蕉）。我真的很饿，因此我想要尽可能多的水果，但是我真的更喜欢圆形的水果（苹果和橙子）。因此我逐一筛选每一个水果，然后带着装满苹果和橙子的篮子离开。

我们将这个筛选的过程称为“过滤”。将这次购物描述为从空篮子开始，然后只**过滤**（挑选，包含）出苹果和橙子，或者从所有的水果中**过滤掉**（跳过，不包括）香蕉。你认为哪种方式更自然？

如果你在一锅水里面做意大利面条，然后将这锅面条倒入滤网（过滤）中，你是过滤了意大利面条，还是过滤掉了水？ 如果你将咖啡渣放入过滤器中，然后泡一杯咖啡，你是将咖啡过滤到了杯子里，还是说将咖啡渣过滤掉？

你有没有发现过滤的结果取决于你想要把什么保留在过滤器中，还是说用过滤器将其过滤出去？

那么在航空／酒店网站上如何指定过滤选项呢？你是按照你的标准过滤结果，还是将不符合标准的过滤掉？仔细想想，这个例子也许和前面有不相同的语意。

取决于你的想法，过滤是排除的或者保留的，这种概念上的融合，使其难以理解。

我认为最通常的理解过滤（在编程之外）是剔除掉不需要的成员。不幸的是，在程序中我们基本上将这个语意倒转为更像是过滤需要的成员。

列表的 `filter(..)` 操作采用一个函数确定每一项在新数组中是保留还是剔除。这个函数返回 `true` 将保留这一项，返回 `false` 将剔除这一项。这种返回 `true`/`false` 来做决定的函数有一个特别的称谓：谓词函数。

如果你认为 `true` 是积极的信号，`filter(..)` 的定义是你是“保留”一个值，而不是“抛弃”一个值。

如果 `filter(..)` 被用于剔除操作，你需要转动你的脑子，积极的返回 `false` 发出排除的信号，并且被动的返回 `true` 来让一个值通过过滤器。

这种语意上不匹配的原因是你会将这个函数命名为 `predicateFn(..)`，这对于代码的可读性有意义，我们很快会讨论这一点。

下图很形象的介绍了列表间的 `filter(..)` 操作：


<p align="center">
	<img src="fig10.png" width="400">
</p>

实现 `filter(..)` 的代码如下：

```js
function filter(predicateFn,arr) {
	var newList = [];

	for (let idx = 0; idx < arr.length; idx++) {
		if (predicateFn( arr[idx], idx, arr )) {
			newList.push( arr[idx] );
		}
	}

	return newList;
}
```

注意，就像之前的 `mapperFn(..)`，`predicateFn(..)` 不仅仅传入了值，还传入了 `idx` 和 `arr`。如果有必要，也可以采用 `unary(..)` 来限制它的形参。

正如 `map(..)`，`filter(..)` 也是 JS 数组内置支持的实用函数。

我们将谓词函数定义这样：

```js
var whatToCallIt = v => v % 2 == 1;
```

这个函数采用 `v % 2 == 1` 来返回 `true` 或 `false`。这里的效果是，值为奇数时返回 `true`，值为偶数时返回 `false`。这样，我们该如何命名这个函数？一个很自然的名字可能是：

```js
var isOdd = v => v % 2 == 1;
```

考虑一下如何在你的代码中使用 `isOdd(..)` 来做简单的值检查：


```js
var midIdx;

if (isOdd( list.length )) {
	midIdx = (list.length + 1) / 2;
}
else {
	midIdx = list.length / 2;
}
```

有感觉了，对吧？让我们采用内置的数组的 `filter(..)` 来对一组值做筛选：

```js
[1,2,3,4,5].filter( isOdd );
// [1,3,5]
```

如果让你描述 `[1,3,5]` 这个结果，你是说“我将偶数过滤掉了”，还是说“我做了奇数的筛选” ？我认为前者是更自然的描述。但后者的代码可读性更好。阅读代码几乎是逐字的，这样我们“过滤的每一个数字都是奇数”。

我个人觉得这语意混乱。对于经验丰富的开发者来说，这里毫无疑问有大量的先例。但是对于一个新手来说，这个逻辑表达看上去不采用双重否定不好表达，换句话说，采用双重否定来表达比较好。

为了便以理解，我们可以将这个函数从 `isOdd(..)` 重命名为 `isEven(..)`：

```js
var isEven = v => v % 2 == 1;

[1,2,3,4,5].filter( isEven );
// [1,3,5]
```
耶，但是这个函数名变得无意义，下面的示例中，传入的偶数，确返回了 `false`

```js
isEven( 2 );		// false
```

呸！

回顾在第 3 章中的 "No Points"，我们定义 `not(..)` 操作来反转谓词函数，代码如下：


```js
var isEven = not( isOdd );

isEven( 2 );		// true
```

但在前面定义的 `filter(..)` 方式中，无法使用**这个** `isEven(..)`，因为它的逻辑已经反转了。我们将以偶数结束，而不是奇数，我们需要这么做：

```js
[1,2,3,4,5].filter( not( isEven ) );
// [1,3,5]
```
这样完全违背了我们的初衷，所以我们不要这么做。这样，我们转一圈又回来了。

### 过滤掉 & 过滤

为了消除这些困惑，我们定义 `filterOut(..)` 函数来执行**过滤掉**那些值，而实际上其内部执行否定的谓词检查。这样，我们将已经定义的 `filter(..)` 设置别名为 `filterIn(..)`。

```js
var filterIn = filter;

function filterOut(predicateFn,arr) {
	return filterIn( not( predicateFn ), arr );
}
```

现在，我们可以在任意过滤操作中，使用语意化的过滤器，代码如下所示：

```js
isOdd( 3 );								// true
isEven( 2 );							// true

filterIn( isOdd, [1,2,3,4,5] );			// [1,3,5]
filterOut( isEven, [1,2,3,4,5] );		// [1,3,5]
```

我认为采用 `filterIn(..)` 和 `filterOut(..)`（在 Ramda 中称之为 `reject(..)` ）会让代码的可读性比仅仅采用 `filter(..)` 更好。


## Reduce

`map(..)` 和 `filter(..)` 都会产生新的数组，而第三种操作（`reduce(..)`）则是典型地将列表中的值合并（或减少）到单个值（非列表），比如数字或者字符串。本章后续会探讨如何采用高级的方式使用 `reduce(..)`。`reduce(..)` 是函数式编程中的最重要的实用函数之一。就像瑞士军刀一样，具有丰富的用途。

组合或缩减被抽象的定义为将两个值转换成一个值。有些函数式编程文献将其称为“折叠”，就像你将两个值合并到一个值。我认为这对于可视化是很有帮助的。

就像映射和过滤，合并的方式完全取决于你，一般取决于列表中值的类型。例如，数字通常采用算术计算合并，字符串采用拼接的方式合并，函数采用组合调用来合并。

有时候，缩减操作会指定一个 `initialValue`，然后将这个初始值和列表的第一个元素合并。然后逐一和列表中剩余的元素合并。如下图所示：

<p align="center">
	<img src="fig11.png" width="400">
</p>

你也可以去掉上述的 `initialValue`，直接将第一个列表元素当做 `initialValue`，然后和列表中的第二个元素合并，如下图所示：

<p align="center">
	<img src="fig12.png" width="400">
</p>

**警告：** 在 JavaScript 中，如果在缩减操作的列表中一个值都没有（在数组中，或没有指定 `initialValue` ），将会抛出异常。一个缩减操作的列表有可能为空的时候，需要小心采用不指定 `initialValue` 的方式。

传递给 `reduce(..)` 执行缩减操作的函数执行一般称为缩减器。缩减器和之前介绍的映射和谓词函数有不同的特征。缩减器主要接受当前的缩减结果和下一个值来做缩减操作。每一步缩减的当前结果通常称为累加器。

例如，对 5、10、15 采用初始值为 3 执行乘的缩减操作：

1. `3` * `5` = `15`
2. `15` * `10` = `150`
3. `150` * `15` = `2250`

在 JavaScript 中采用内置的 `reduce(..)` 方法来表达列表的缩减操作：

```js
[5,10,15].reduce( (product,v) => product * v, 3 );
// 2250
```

我们可以采用下面的方式实现 `reduce(..)`：

```js
function reduce(reducerFn,initialValue,arr) {
	var acc, startIdx;

	if (arguments.length == 3) {
		acc = initialValue;
		startIdx = 0;
	}
	else if (arr.length > 0) {
		acc = arr[0];
		startIdx = 1;
	}
	else {
		throw new Error( "Must provide at least one value." );
	}

	for (let idx = startIdx; idx < arr.length; idx++) {
		acc = reducerFn( acc, arr[idx], idx, arr );
	}

	return acc;
}
```

就像 `map(..)` 和 `filter(..)`，缩减函数也传递不常用的 `idx` 和 `arr` 形参，以防缩减操作需要。我不会经常用到它们，但我觉得保留它们是明智的。

在第 4 章中，我们讨论了 `compose(..)` 实用函数，和展示了用 `reduce(..)` 来实现的例子：

```js
function compose(...fns) {
	return function composed(result){
		return fns.reverse().reduce( function reducer(result,fn){
			return fn( result );
		}, result );
	};
}
```
基于不同的组合，为了说明 `reduce(..)`，可以认为缩减器将函数从左到右组合（就像 `pipe(..)` 做的事情）。在列表中这样使用：

```js
var pipeReducer = (composedFn,fn) => pipe( composedFn, fn );

var fn =
	[3,17,6,4]
	.map( v => n => v * n )
	.reduce( pipeReducer );

fn( 9 );			// 11016  (9 * 3 * 17 * 6 * 4)
fn( 10 );			// 12240  (10 * 3 * 17 * 6 * 4)
```
不幸的是，`pipeReducer(..)` 是非点自由的（见第 3 章中的“无形参”），但我们不能仅仅以缩减器本身来传递 `pipe(..)`，因为它是可变的；传递给 `reduce(..)` 额外的参数（`idx` 和 `arr`）会产生问题。

前面，我们讨论采用 `unary(..)` 来限制 `mapperFn(..)` 或 `predicateFn(..)` 仅采用一个参数。`binary(..)` 做了类似的事情，但在 `reducerFn(..)` 中限定两个参数：

```js
var binary =
	fn =>
		(arg1,arg2) =>
			fn( arg1, arg2 );
```

采用 `binary(..)`，相比之前的示例有一些简洁：

```js
var pipeReducer = binary( pipe );

var fn =
	[3,17,6,4]
	.map( v => n => v * n )
	.reduce( pipeReducer );

fn( 9 );			// 11016  (9 * 3 * 17 * 6 * 4)
fn( 10 );			// 12240  (10 * 3 * 17 * 6 * 4)
```

不像 `map(..)` 和 `filter(..)`，对传入数组的次序没有要求。`reduce(..)` 明确要采用从左到右的处理方式。如果你想从右到左缩减，JavaScript 提供了 `reduceRight(..)` 函数，它和 `reduce(..)` 的行为出了次序不一样外，其他都相同。

```js
var hyphenate = (str,char) => str + "-" + char;

["a","b","c"].reduce( hyphenate );
// "a-b-c"

["a","b","c"].reduceRight( hyphenate );
// "c-b-a"
```

`reduce(..)` 采用从左到右的方式工作，很自然的联想到组合函数中的 `pipe(..)`。`reduceRight(..)` 从右往左的方式能自然的执行 `compose(..)`。因此，我们重新采用 `reduceRight(..)` 实现 `compose(..)`：

```js
function compose(...fns) {
	return function composed(result){
		return fns.reduceRight( function reducer(result,fn){
			return fn( result );
		}, result );
	};
}
```

这样，我们不需要执行 `fns.reverse()`；我们只需要从另一个方向执行缩减操作！

### Map 也是 Reduce

`map(..)` 操作本质来说是迭代，因此，它也可以看作是（`reduce(..)`）操作。这个技巧是将 `reduce(..)` 的 `initialValue` 看成它自身的空数组。在这种情况下，缩减操作的结果是另一个列表！

```js
var double = v => v * 2;

[1,2,3,4,5].map( double );
// [2,4,6,8,10]

[1,2,3,4,5].reduce(
	(list,v) => (
		list.push( double( v ) ),
		list
	), []
);
// [2,4,6,8,10]
```

**注意：** 我们欺骗了这个缩减器，并允许采用 `list.push(..)` 去改变传入的列表所带来的副作用。一般来说，这并不是一个好主意，但我们清楚创建和传入 `[]` 列表，这样就不那么危险了。创建一个新的列表，并将 val 合并到这个列表的最后面。这样更有条理，并且性能开销较小。我们将在附录 A 中讨论这种欺骗。

通过 `reduce(..)` 实现 `map(..)`，并不是表面上的明显的步骤，甚至是一种改善。然而，这种能力对于理解更高级的技术是至关重要的，如在附录 A 中的“转换”。

### Filter 也是 Reduce

就像通过 `reduce(..)` 实现 `map(..)` 一样，也可以使用它实现 `filter(..)`：

```js
var isOdd = v => v % 2 == 1;

[1,2,3,4,5].filter( isOdd );
// [1,3,5]

[1,2,3,4,5].reduce(
	(list,v) => (
		isOdd( v ) ? list.push( v ) : undefined,
		list
	), []
);
// [1,3,5]
```

**注意：** 这里有更加不纯的缩减器欺骗。不采用 `list.push(..)`，我们也可以采用 `list.concat(..)` 并返回合并后的新列表。我们将在附录 A 中继续介绍这个欺骗。

## 高级列表操作

现在，我们对这些基础的列表操作 `map(..)`、`filter(..)` 和 `reduce(..)` 感到比较舒服。让我们看看一些更复杂的操作，这些操作在某些场合下很有用。这些常用的实用函数存在于许多函数式编程的类库中。

### 去重

筛选列表中的元素，仅仅保留唯一的值。基于 `indexOf(..)` 函数查找（它采用 `===` 严格等于表达式）：

```js
var unique =
	arr =>
		arr.filter(
			(v,idx) =>
				arr.indexOf( v ) == idx
		);
```

实现的原理是，当从左往右筛选元素时，列表项的 `idx` 位置和 `indexOf(..)` 找到的位置相等时，表明该列表项第一次出现，在这种情况下，将列表项加入到新数组中。


另一种实现 `unique(..)` 的方式是遍历 `arr`，当列表项不能在新列表中找到时，将其插入到新的列表中。这样可以采用 `reduce(..)` 来实现：

```js
var unique =
	arr =>
		arr.reduce(
			(list,v) =>
				list.indexOf( v ) == -1 ?
					( list.push( v ), list ) : list
		, [] );
```

**注意：** 这里还有很多其他的方式实现这个去重算法，比如循环，并且其中不少还更高效，实现方式更聪明。然而，这两种方式的优点是，它们使用了内建的列表操作，它们能更方便的和其他列表操作链式／组合调用。我们会在本章的后面进一步讨论这些。

`unique(..)` 令人满意地产生去重后的新列表：

```js
unique( [1,4,7,1,3,1,7,9,2,6,4,0,5,3] );
// [1, 4, 7, 3, 9, 2, 6, 0, 5]
```

### 扁平化

大多数时候，你看到的数组的列表项不是扁平的，很多时候，数组嵌套了数组，例如：

```js
[ [1, 2, 3], 4, 5, [6, [7, 8]] ]
```

如果你想将其转化成下面的形式:

```js
[ 1, 2, 3, 4, 5, 6, 7, 8 ]
```

我们寻找的这个操作通常称为 `flatten(..)`。它可以采用如同瑞士军刀般的 `reduce(..)` 实现：

```js
var flatten =
	arr =>
		arr.reduce(
			(list,v) =>
				list.concat( Array.isArray( v ) ? flatten( v ) : v )
		, [] );
```

**注意：** 这种处理嵌套列表的实现方式依赖于递归，我们将在后面的章节中进一步讨论。

在嵌套数组（任意嵌套层次）中使用 `flatten(..)`：

```js
flatten( [[0,1],2,3,[4,[5,6,7],[8,[9,[10,[11,12],13]]]]] );
// [0,1,2,3,4,5,6,7,8,9,10,11,12,13]
```

也许你会限制递归的层次到指定的层次。我们可以通过增加额外的 `depth` 形参来实现：

```js
var flatten =
	(arr,depth = Infinity) =>
		arr.reduce(
			(list,v) =>
				list.concat(
					depth > 0 ?
						(depth > 1 && Array.isArray( v ) ?
							flatten( v, depth - 1 ) :
							v
						) :
						[v]
				)
		, [] );
```

不同层级扁平化的结果如下所示：

```js
flatten( [[0,1],2,3,[4,[5,6,7],[8,[9,[10,[11,12],13]]]]], 0 );
// [[0,1],2,3,[4,[5,6,7],[8,[9,[10,[11,12],13]]]]]

flatten( [[0,1],2,3,[4,[5,6,7],[8,[9,[10,[11,12],13]]]]], 1 );
// [0,1,2,3,4,[5,6,7],[8,[9,[10,[11,12],13]]]]

flatten( [[0,1],2,3,[4,[5,6,7],[8,[9,[10,[11,12],13]]]]], 2 );
// [0,1,2,3,4,5,6,7,8,[9,[10,[11,12],13]]]

flatten( [[0,1],2,3,[4,[5,6,7],[8,[9,[10,[11,12],13]]]]], 3 );
// [0,1,2,3,4,5,6,7,8,9,[10,[11,12],13]]

flatten( [[0,1],2,3,[4,[5,6,7],[8,[9,[10,[11,12],13]]]]], 4 );
// [0,1,2,3,4,5,6,7,8,9,10,[11,12],13]

flatten( [[0,1],2,3,[4,[5,6,7],[8,[9,[10,[11,12],13]]]]], 5 );
// [0,1,2,3,4,5,6,7,8,9,10,11,12,13]
```

#### 映射，然后扁平化

`flatten(..)` 的常用用法之一是当你映射一组元素列表，并且将每一项值从原来的值转换为数组。例如：

```js
var firstNames = [
	{ name: "Jonathan", variations: [ "John", "Jon", "Jonny" ] },
	{ name: "Stephanie", variations: [ "Steph", "Stephy" ] },
	{ name: "Frederick", variations: [ "Fred", "Freddy" ] }
];

firstNames
.map( entry => [entry.name].concat( entry.variations ) );
// [ ["Jonathan","John","Jon","Jonny"], ["Stephanie","Steph","Stephy"],
//   ["Frederick","Fred","Freddy"] ]
```

返回的值是二维数组，这样也许给处理带来一些不便。如果我们想得到所有名字的一维数组，我们可以对这个结果执行 `flatten(..)`：

```js
flatten(
	firstNames
	.map( entry => [entry.name].concat( entry.variations ) )
);
// ["Jonathan","John","Jon","Jonny","Stephanie","Steph","Stephy","Frederick",
//  "Fred","Freddy"]
```
除了稍显啰嗦之外，将 `map(..)` 和 `flatten(..)` 采用独立的步骤的最主要的缺陷是关于性能方面。它会处理列表两次。

函数式编程的类库中，通常会定义一个 `flatMap(..)`（通常命名为 `chain(..)`）函数。这个函数将映射和之后的扁平化的操作组合起来。为了连贯性和组合（通过闭包）的简易性，`flatMap(..)` / `chain(..)` 实用函数的形參 `mapperFn, arr` 顺序通常和我们之前看到的独立的 `map(..)`、`filter(..)`和 `reduce(..)` 一致。

```js
flatMap( entry => [entry.name].concat( entry.variations ), firstNames );
// ["Jonathan","John","Jon","Jonny","Stephanie","Steph","Stephy","Frederick",
//  "Fred","Freddy"]
```

幼稚的采用独立的两步来实现 `flatMap(..)`：

```js
var flatMap =
	(mapperFn,arr) =>
		flatten( arr.map( mapperFn ), 1 );
```

**注意：** 我们将扁平化的层级指定为 `1`，因为通常 `flatMap(..)` 的定义是扁平化第一级。

尽管这种实现方式依旧会处理列表两次，带来了不好的性能。但我们可以将这些操作采用 `reduce(..)` 手动合并：

```js
var flatMap =
	(mapperFn,arr) =>
		arr.reduce(
			(list,v) =>
				list.concat( mapperFn( v ) )
		, [] );
```

现在 `flatMap(..)` 方法带来了便利性和性能。有时你可能需要其他操作，比如和 `filter(..)` 混合使用。这样的话，将 `map(..)` 和 `flatten(..)` 独立开来始终更加合适。

### Zip

到目前为止，我们介绍的列表操作都是操作单个列表。但是在某些情况下，需要操作多个列表。有一个闻名的操作：交替选择两个输入的列表中的值，并将得到的值组成子列表。这个操作被称之为 `zip(..)`：

```js
zip( [1,3,5,7,9], [2,4,6,8,10] );
// [ [1,2], [3,4], [5,6], [7,8], [9,10] ]
```

选择值 `1` 和 `2` 到子列表 `[1,2]`，然后选择 `3` 和 `4` 到子列表 `[3,4]`，然后逐一选择。`zip(..)` 被定义为将两个列表中的值挑选出来。如果两个列表的的元素的个数不一致，这个选择会持续到较短的数组末尾时结束，另一个数组中多余的元素会被忽略。

一种 `zip(..)` 的实现：

```js
function zip(arr1,arr2) {
	var zipped = [];
	arr1 = arr1.slice();
	arr2 = arr2.slice();

	while (arr1.length > 0 && arr2.length > 0) {
		zipped.push( [ arr1.shift(), arr2.shift() ] );
	}

	return zipped;
}
```
采用 `arr1.slice()` 和 `arr2.slice()` 可以确保 `zip(..)` 是纯的，不会因为接受到到数组引用造成副作用。

**注意：** 这个实现明显存在一些非函数式编程的思想。这里有一个命令式的 `while` 循环并且采用 `shift()` 和 `push(..)` 改变列表。在本书前面，我认为在纯函数中使用非纯的行为（通常是为了性能）是有道理的，只要其产生的副作用完全包含在这个函数内部。这种实现是安全纯净的。

### 合并

采用插入每个列表中的值的方式合并两个列表，如下所示：

```js
mergeLists( [1,3,5,7,9], [2,4,6,8,10] );
// [1,2,3,4,5,6,7,8,9,10]
```

它可能不是那么明显，但其结果看上去和采用 `flatten(..)` 和 `zip(..)` 组合相似，代码如下:

```js
zip( [1,3,5,7,9], [2,4,6,8,10] );
// [ [1,2], [3,4], [5,6], [7,8], [9,10] ]

flatten( [ [1,2], [3,4], [5,6], [7,8], [9,10] ] );
// [1,2,3,4,5,6,7,8,9,10]

// 组合后：
flatten( zip( [1,3,5,7,9], [2,4,6,8,10] ) );
// [1,2,3,4,5,6,7,8,9,10]
```

回顾 `zip(..)`，他选择较短列表的最后一个值，忽视掉剩余的值； 而合并两个数组会很自然地保留这些额外的列表值。并且 `flatten(..)` 采用递归处理嵌套列表，但你可能只期望较浅地合并列表，保留嵌套的子列表。

这样，让我们定义一个更符合我们期望的 `mergeLists(..)`：

```js
function mergeLists(arr1,arr2) {
	var merged = [];
	arr1 = arr1.slice();
	arr2 = arr2.slice();

	while (arr1.length > 0 || arr2.length > 0) {
		if (arr1.length > 0) {
			merged.push( arr1.shift() );
		}
		if (arr2.length > 0) {
			merged.push( arr2.shift() );
		}
	}

	return merged;
}
```

**注意：** 许多函数式编程类库并不会定义 `mergeLists(..)`，反而会定义 `merge(..)` 方法来合并两个对象的属性。这种 `merge(..)` 返回的结果和我们的 `mergeLists(..)` 不同。

另外，这里有一些选择采用缩减器实现合并列表的方法：

```js
// 来自 @rwaldron
var mergeReducer =
	(merged,v,idx) =>
		(merged.splice( idx * 2, 0, v ), merged);


// 来自 @WebReflection
var mergeReducer =
	(merged,v,idx) =>
		merged
			.slice( 0, idx * 2 )
			.concat( v, merged.slice( idx * 2 ) );
```

采用 `mergeReducer(..)`：

```js
[1,3,5,7,9]
.reduce( mergeReducer, [2,4,6,8,10] );
// [1,2,3,4,5,6,7,8,9,10]
```

**提示**：我们将在本章后面使用 `mergeReducer(..)` 这个技巧。

## 方法 vs 独立

对于函数式编程者来说，普遍感到失望的原因是 Javascript 采用统一的策略处理实用函数，但其中的一些也被作为独立函数提供了出来。想想在前面的章节中的介绍的大量的函数式编程实用程序，以及另一些实用函数是数组的原型方法，就像在这章中看到的那些。

当你想合并多个操作的时候，这个问题的痛苦程度更加明显：


```js
[1,2,3,4,5]
.filter( isOdd )
.map( double )
.reduce( sum, 0 );					// 18

//  采用独立的方法.

reduce(
	map(
		filter( [1,2,3,4,5], isOdd ),
		double
	),
	sum,
	0
);									// 18
```

两种方式的 API 实现了同样的功能。但它们的风格完全不同。很多函数式编程者更倾向采用后面的方式，但是前者在 Javascript 中毫无疑问的更常见。后者特别地让人不待见之处是采用嵌套调用。人们更偏爱链式调用 —— 通常称为流畅的API风格，这种风格被 jQuery 和一些工具采用 —— 这种风格紧凑／简洁，并且可以采用声明式的自上而下的顺序阅读。

这种独立风格的手动合并的视觉顺序既不是严格的从左到右（自上而下），也不是严格的从右到左，而是从里往外。

从右往左（自下而上）这两种风格自动组成规范的阅读顺序。因此为了探索这些风格隐藏的差异，让我们特别的检查组合。他看上去应当简洁，但这两种情况都有点尴尬。

### 链式组合方法

这些数组方法接收绝对的 `this` 形参，因此尽管从外表上看，它们不能被当作一元运算看待，这会使组合更加尴尬。为了应对这些，我首先需要一个 `partial(..)` 版本的 `this`：

```js
var partialThis =
	(fn,...presetArgs) =>
		// 故意采用 function 来为了 this 绑定
		function partiallyApplied(...laterArgs){
			return fn.apply( this, [...presetArgs, ...laterArgs] );
		};
```
我们也需要一个特殊的 `compose(..)`，它在上下文链中调用每一个部分应用的方法。它的输入值（即绝对的 this）由前一步传入：

```js
var composeChainedMethods =
	(...fns) =>
		result =>
			fns.reduceRight(
				(result,fn) =>
					fn.call( result )
				, result
			);
```

一起使用这两个 `this` 实用函数：

```js
composeChainedMethods(
   partialThis( Array.prototype.reduce, sum, 0 ),
   partialThis( Array.prototype.map, double ),
   partialThis( Array.prototype.filter, isOdd )
)
( [1,2,3,4,5] );					// 18
```

**注意：** 那三个 `Array.prototype.XXX` 采用了内置的 `Array.prototype.*` 方法，这样我们可以在数组中重复使用它们。

### 独立组合实用函数

独立的 `compose(..)`，组合这些功能函数的风格不需要所有的这些广泛令人喜欢的 `this` 参数。例如，我们可以独立的定义成这样：

```js
var filter = (arr,predicateFn) => arr.filter( predicateFn );

var map = (arr,mapperFn) => arr.map( mapperFn );

var reduce = (arr,reducerFn,initialValue) =>
	arr.reduce( reducerFn, initialValue );
```
但是，这种特别的独立风格给自身带来了不便。层级的数组上下文是第一个形参，而不是最后一个。因此我们需要采用右偏应用（right-partial application）来组合它们。

```js
compose(
	partialRight( reduce, sum, 0 ),
	partialRight( map, double ),
	partialRight( filter, isOdd )
)
( [1,2,3,4,5] );					// 18
```

这就是为何函数式编程类库通常定义 `filter(..)`、`map(..)` 和 `reduce(..)` 交替采用最后一个形参接收数组，而不是第一个。它们通常自动地柯理化实用函数：

```js
var filter = curry(
	(predicateFn,arr) =>
		arr.filter( predicateFn )
);

var map = curry(
	(mapperFn,arr) =>
		arr.map( mapperFn )
);

var reduce = curry(
	(reducerFn,initialValue,arr) =>
		arr.reduce( reducerFn, initialValue );
```

采用这种方式定义实用函数，组合流程会显得更加友好：

```js
compose(
	reduce( sum )( 0 ),
	map( double ),
	filter( isOdd )
)
( [1,2,3,4,5] );					// 18
```

这种很整洁的实现方式，就是函数式编程者喜欢独立的实用程序风格，而不是实例方法的原因。但这种情况因人而异。

### 方法适配独立

在前面的 `filter(..)` / `map(..)` / `reduce(..)` 的定义中，你可能发现了这三个方法的共同点：它们都派发到相对应的原生数组方法。因此，我们能采用实用函数生成这些独立适配函数吗？当然可以，让我们定义 `unboundMethod(..)` 来做这些：

```js
var unboundMethod =
	(methodName,argCount = 2) =>
		curry(
			(...args) => {
				var obj = args.pop();
				return obj[methodName]( ...args );
			},
			argCount
		);
```

使用这个实用函数：

```js
var filter = unboundMethod( "filter", 2 );
var map = unboundMethod( "map", 2 );
var reduce = unboundMethod( "reduce", 3 );

compose(
	reduce( sum )( 0 ),
	map( double ),
	filter( isOdd )
)
( [1,2,3,4,5] );					// 18
```

**注意:** `unboundMethod(..)` 在 Ramda 中称之为 `invoker(..)`。

### 独立函数适配为方法

如果你喜欢仅仅使用数组方法（流畅的链式风格），你有两个选择：

1. 采用额外的方法扩展内建的 `Array.prototype`
2. 把独立实用函数适配成一个缩减函数，并且将其传递给 `reduce(..)` 实例方法。

**不要采用第一种** 扩展诸如 `Array.prototype` 的原生方法从来不是一个好主意，除非定义一个 `Array` 的子类。但是这超出了这里的讨论范围。为了不鼓励这种不好的习惯，我们不会进一步去探讨这种方式。

让我们**关注第二种**。为了说明这点，我们将前面定义的递归实现的 `flatten(..)` 转换为独立实用函数：

```js
var flatten =
	arr =>
		arr.reduce(
			(list,v) =>
				list.concat( Array.isArray( v ) ? flatten( v ) : v )
		, [] );
```
让我们将里面的 `reducer(..)` 函数抽取成独立的实用函数（并且调整它，让其独立于外部的 `flatten(..)` 运行）：

```js
// 刻意使用具名函数用于递归中的调用
function flattenReducer(list,v) {
	return list.concat(
		Array.isArray( v ) ? v.reduce( flattenReducer, [] ) : v
	);
}
```

现在，我们可以在数组方法链中通过 `reduce(..)` 调用这个实用函数：

```js
[ [1, 2, 3], 4, 5, [6, [7, 8]] ]
.reduce( flattenReducer, [] )
// ..
```

## 查寻列表

到此为止，大部分示例有点无聊，它们基于一列数字或者字符串，让我们讨论一些有亮点的列表操作：声明式地建模一些命令式语句。

看看这个基本例子：

```js
var getSessionId = partial( prop, "sessId" );
var getUserId = partial( prop, "uId" );

var session, sessionId, user, userId, orders;

session = getCurrentSession();
if (session != null) sessionId = getSessionId( session );
if (sessionId != null) user = lookupUser( sessionId );
if (user != null) userId = getUserId( user );
if (userId != null) orders = lookupOrders( userId );
if (orders != null) processOrders( orders );
```

首先，我们可以注意到声明和运行前的一系列 If 语句确保了由 `getCurrentSession()`、`getSessionId(..)`、`lookupUser(..)`、`getUserId(..)`、`lookupOrders(..)` 和 `processOrders(..)` 这六个函数组合调用时的有效。理想地，我们期望摆脱这些变量定义和命令式的条件。

不幸的是，在第 4 章中讨论的 `compose(..)`／`pipe(..)` 实用函数并没有提供给一个便捷的方式来表达在这个组合中的 `!= null` 条件。让我们定义一个实用函数来解决这个问题：

```js
var guard =
	fn =>
		arg =>
			arg != null ? fn( arg ) : arg;
```

这个 `guard(..)` 实用函数让我们映射这五个条件确保函数：

```js
[ getSessionId, lookupUser, getUserId, lookupOrders, processOrders ]
.map( guard )
```

这个映射的结果是组合的函数数组（事实上，这是个有列表顺序的管道）。我们可以展开这个数组到 `pipe(..)`，但由于我们已经做列表操作，让我们采用 `reduce(..)` 来处理。采用 `getCurrentSession()` 返回的会话值作为初始值：

```js
.reduce(
	(result,nextFn) => nextFn( result )
	, getCurrentSession()
)
```

接下来，我们观察到 `getSessionId(..)` 和 `getUserId(..)` 可以看成对应的 `"sessId"` 和 `"uId"` 的映射：

```js
[ "sessId", "uId" ].map( propName => partial( prop, propName ) )
```

但是为了使用这些，我们需要将另外三个函数（`lookupUser(..)`、`lookupOrders(..)` 和 `processOrders(..)`）插入进来，用来获取上面讨论的那五个守护／组合函数。

为了实现插入，我们采用列表合并来模拟这些。回顾本章前面介绍的 `mergeReducer(..)`：

```js
var mergeReducer =
	(merged,v,idx) =>
		(merged.splice( idx * 2, 0, v ), merged);
```

我们可以采用 `reduce(..)`（我们的瑞士军刀，还记得吗？）在生成的 `getSessionId(..)` 和 `getUserId(..)` 函数之间的数组中“插入” `lookupUser(..)`，通过合并这两个列表：

```js
.reduce( mergeReducer, [ lookupUser ] )
```

然后我们将 `lookupOrders(..)` 和 `processOrders(..)` 加入到正在执行的函数数组末尾：

```js
.concat( lookupOrders, processOrders )
```

总结下，生成的五个函数组成的列表表达为：

```js
[ "sessId", "uId" ].map( propName => partial( prop, propName ) )
.reduce( mergeReducer, [ lookupUser ] )
.concat( lookupOrders, processOrders )
```

最后，将所有函数合并到一起，将这些函数数组添加到之前的守护和组合上：

```js
[ "sessId", "uId" ].map( propName => partial( prop, propName ) )
.reduce( mergeReducer, [ lookupUser ] )
.concat( lookupOrders, processOrders )
.map( guard )
.reduce(
	(result,nextFn) => nextFn( result )
	, getCurrentSession()
);
```

所有必要的变量声明和条件一去不复返了，取而代之的是采用整洁和声明式的列表操作链接在一起。

如果你觉得现在的这个版本比之前要难，不要担心。毫无疑问的，前面的命令式的形式，你可能更加熟悉。进化为函数式编程者的一步就是开发一些具有函数式编程风格的代码，比如这些列表操作。随着时间推移，我们跳出这些代码，当你切换到声明式风格时更容易感受到代码的可读性。

在离开这个话题之前，让我们做一个真实的检查：这里的示例过于造作。不是所有的代码片段被简单的采用列表操作模拟。务实的获取方式是本能的寻找这些机会，而不是过于追求代码的技巧；一些改进比没有强。经常退一步，并且问自己，是**提升了还是损害了**代码的可读性。

## 融合

当你更多的考虑在代码中使用函数式列表操作，你可能会很快地开始看到链式组合行为，如：

```js
..
.filter(..)
.map(..)
.reduce(..);
```

往往，你可能会把多个相邻的操作用链式来调用，比如：

```js
someList
.filter(..)
.filter(..)
.map(..)
.map(..)
.map(..)
.reduce(..);
```

好消息是，链式风格是声明式的，并且很容易看出详尽的执行步骤和顺序。它的不足之处在于每一个列表操作都需要循环整个列表，意味着不必要的性能损失，特别是在列表非常长的时候。

采用交替独立的风格，你可能看到的代码如下：

```js
map(
	fn3,
	map(
		fn2,
		map( fn1, someList )
	)
);
```

采用这种风格，这些操作自下而上列出，这依然会循环数组三遍。

融合处理了合并相邻的操作，这样可以减少列表的迭代次数。这里我们关注于合并相邻的 `map(..)`，这很容易解释。

想象一下这样的场景：

```js
var removeInvalidChars = str => str.replace( /[^\w]*/g, "" );

var upper = str => str.toUpperCase();

var elide = str =>
	str.length > 10 ?
		str.substr( 0, 7 ) + "..." :
		str;

var words = "Mr. Jones isn't responsible for this disaster!"
	.split( /\s/ );

words;
// ["Mr.","Jones","isn't","responsible","for","this","disaster!"]

words
.map( removeInvalidChars )
.map( upper )
.map( elide );
// ["MR","JONES","ISNT","RESPONS...","FOR","THIS","DISASTER"]
```

注意在这个转换流程中的每一个值。在 `words` 列表中的第一个值，开始为 `"Mr."`，变为 `"Mr"`，然后为 `"MR"`，然后通过 `elide(..)` 不变。另一个数据流为：`"responsible"` -> `"responsible"` -> `"RESPONSIBLE"` -> `"RESPONS..."`。

换句话说，你可以将这些数据转换看成这样：

```js
elide( upper( removeInvalidChars( "Mr." ) ) );
// "MR"

elide( upper( removeInvalidChars( "responsible" ) ) );
// "RESPONS..."
```

你抓住重点了吗？我们可以将那三个独立的相邻的 `map(..)` 调用步骤看成一个转换组合。因为它们都是一元函数，并且每一个返回值都是下一个点输入值。我们可以采用 `compose(..)` 执行映射功能，并将这个组合函数传入到单个 `map(..)` 中调用：

```js
words
.map(
	compose( elide, upper, removeInvalidChars )
);
// ["MR","JONES","ISNT","RESPONS...","FOR","THIS","DISASTER"]
```

这是另一个 `pipe(..)` 能更便利的方式处理组合的场景，这样可读性很有条理：

```js
words
.map(
	pipe( removeInvalidChars, upper, elide )
);
// ["MR","JONES","ISNT","RESPONS...","FOR","THIS","DISASTER"]
```

如何融合两个以上的 `filter(..)` 谓词函数呢？通常视为一元函数，它们似乎适合组合。但是有个小问题，每一个函数返回了不同类型的值（`boolean`），这些返回值并不是下一个函数需要的输入参数。融合相邻的 `reduce(..)` 调用也是可能的，但缩减器并不是一元的，这也会带来不小的挑战。我们需要更复杂的技巧来实现这些融合。我们将在附录 A 的“转换”中讨论这些高级方法。

## 列表之外

到目前为止，我们讨论的操作都是在列表（数组）数据结构中，这是迄今为止你遇到的最常见的场景。但是更普遍的意义是，这些操作可以在任一集合执行。

就像我们之前说过，数组的 `map(..)` 方法对数组中的每一个值做单值操作，任何数据结构都可以采用 `map(..)` 操作做类似的事情。同样的，也可以实现 `filter(..)`，`reduce(..)` 和其他能工作于这些数据结构的值的操作。

函数式编程精神中重要的部分是这些操作必须依赖值的不变性，意味着它们必须返回一个新的值，而不是改变存在的值。

让我们描述那个广为人知的数据结构：二叉树。二叉树指的是一个节点（只有一个对象！）有两个字节点（这些字节点也是二叉树），这两个字节点通常称之为**左**和**右**子树。树中的每个节点包含总体数据结构的值。

<p align="center">
	<img src="fig7.png" width="250">
</p>

在这个插图中，我们将我们的二叉树描述为二叉搜索树（BST）。然而，树的操作和其他非二叉搜索树没有区别。

**注意：** 二叉搜索树是特定的二叉树，该树中的节点值彼此之间存在特定的约束关系。每个树中的左子节点的值小于根节点的值，跟子节点的值也小于右子节点的值。这里“小于”的概念是相对于树中存储数据的类型。它可以是数字的数值，也可以是字符串在词典中的顺序，等等。二叉搜索树的价值在于在处理在树中搜索一个值非常高效便捷，采用一个递归的二叉搜索算法。

让我们采用这个工厂函数创建二叉树对象：

```js
var BinaryTree =
	(value,parent,left,right) => ({ value, parent, left, right });
```

为了方便，我们在每个Node中不仅仅保存了 `left` 和 `right` 子树节点，也保存了其自身的 `parent` 节点引用。

现在，我们将一些常见的产品名（水果，蔬菜）定义为二叉搜索树：

```js
var banana = BinaryTree( "banana" );
var apple = banana.left = BinaryTree( "apple", banana );
var cherry = banana.right = BinaryTree( "cherry", banana );
var apricot = apple.right = BinaryTree( "apricot", apple );
var avocado = apricot.right = BinaryTree( "avocado", apricot );
var cantelope = cherry.left = BinaryTree( "cantelope", cherry );
var cucumber = cherry.right = BinaryTree( "cucumber", cherry );
var grape = cucumber.right = BinaryTree( "grape", cucumber );
```

在这个树形结构中，`banana` 是根节点，这棵树可能采用不同的方式创建节点，但其依旧可以采用二叉搜索树一样的方式访问。

这棵树如下图所示：

<p align="center">
	<img src="fig8.png" width="450">
</p>

这里有多种方式来遍历一颗二叉树来处理它的值。如果这棵树是二叉搜索树，我们还可以有序的遍历它。通过先访问左侧子节点，然后自身节点，最后右侧子节点，这样我们可以得到升序排列的值。

现在，你不能仅仅通过像在数组中用 `console.log(..)` 打印出二叉树。我们先定义一个便利的方法，主要用来打印。定义的 `forEach(..)` 方法能像和数组一样的方式来访问二叉树：

```js
// 顺序遍历
BinaryTree.forEach = function forEach(visitFn,node){
	if (node) {
		if (node.left) {
			forEach( visitFn, node.left );
		}

		visitFn( node );

		if (node.right) {
			forEach( visitFn, node.right );
		}
	}
};
```

**注意：** 采用递归处理二叉树更自然。我们的 `forEach(..)` 实用函数采用递归调用自身来处理左右字节点。我们将在后续的章节章深入讨论递归。

回顾在本章开头描述的 `forEach(..)`，它存在有用的副作用，通常函数式编程期望有这个副作用。在这种情况下，我们仅仅在 I／O 的副作用下使用 `forEach(..)`，因此它是完美的理想的辅助函数。

采用 `forEach(..)` 打印那个二叉树中的值：

```js
BinaryTree.forEach( node => console.log( node.value ), banana );
// apple apricot avocado banana cantelope cherry cucumber grape

// 仅访问根节点为 `cherry` 的子树
BinaryTree.forEach( node => console.log( node.value ), cherry );
// cantelope cherry cucumber grape
```

为了采用函数式编程的方式操作我们定义的那个二叉树，我们定义一个 `map(..)` 函数：

```js
BinaryTree.map = function map(mapperFn,node){
	if (node) {
		let newNode = mapperFn( node );
		newNode.parent = node.parent;
		newNode.left = node.left ?
			map( mapperFn, node.left ) : undefined;
		newNode.right = node.right ?
			map( mapperFn, node.right ): undefined;

		if (newNode.left) {
			newNode.left.parent = newNode;
		}
		if (newNode.right) {
			newNode.right.parent = newNode;
		}

		return newNode;
	}
};
```

你可能会认为采用 `map(..)` 仅仅处理节点的 `value` 属性，但通常情况下，我们可能需要映射树的节点本身。因此，`mapperFn(..)` 传入整个访问的节点，在应用了转换之后，它期待返回一个全新的 `BinaryTree(..)` 节点回来。如果你返回了同样的节点，这个操作会改变你的树，并且很可能会引起意想不到的结果！

让我们映射我们的那个树，得到一列大写产品名：

```js
var BANANA = BinaryTree.map(
	node => BinaryTree( node.value.toUpperCase() ),
	banana
);

BinaryTree.forEach( node => console.log( node.value ), BANANA );
// APPLE APRICOT AVOCADO BANANA CANTELOPE CHERRY CUCUMBER GRAPE
```

`BANANA` 和 `banana` 是一个不同的树（所有的节点都不同），就像在列表中执行 `map(..)` 返回一个新的数组。就像其他对象／数组的数组，如果 `node.value` 本身是某个对象／数组的引用，如果你想做深层次的转换，那么你就需要在映射函数中手动的对它做深拷贝。

如何处理 `reduce(..)`？相同的基本处理过程：有序遍历树的节点的方式。一种可能的用法是 `reduce(..)` 我们的树得到它的值的数组。这对将来适配其他典型的列表操作很有帮助。或者，我们可以 `reduce(..)` 我们的树，得到一个合并了它所有产品名的字符串。

我们模仿数组中 `reduce(..)` 的行为，它接受那个可选的 `initialValue` 参数。该算法有一点难度，但依旧可控：

```js
BinaryTree.reduce = function reduce(reducerFn,initialValue,node){
	if (arguments.length < 3) {
		// 移动参数，直到 `initialValue` 被删除
		node = initialValue;
	}

	if (node) {
		let result;

		if (arguments.length < 3) {
			if (node.left) {
				result = reduce( reducerFn, node.left );
			}
			else {
				return node.right ?
					reduce( reducerFn, node, node.right ) :
					node;
			}
		}
		else {
			result = node.left ?
				reduce( reducerFn, initialValue, node.left ) :
				initialValue;
		}

		result = reducerFn( result, node );
		result = node.right ?
			reduce( reducerFn, result, node.right ) : result;
		return result;
	}

	return initialValue;
};
```

让我们采用 `reduce(..)` 产生一个购物单（一个数组）：

```js
BinaryTree.reduce(
	(result,node) => result.concat( node.value ),
	[],
	banana
);
// ["apple","apricot","avocado","banana","cantelope"
//   "cherry","cucumber","grape"]
```

最后，让我们考虑在树中用 `filter(..)`。这个算法迄今为止最棘手，因为它有效（实际上没有）影响从树上删除节点，这需要处理几个问题。不要被这种实现吓到。如果你喜欢，现在跳过它，关注我们如何使用它而不是实现。

```js
BinaryTree.filter = function filter(predicateFn,node){
	if (node) {
		let newNode;
		let newLeft = node.left ?
			filter( predicateFn, node.left ) : undefined;
		let newRight = node.right ?
			filter( predicateFn, node.right ) : undefined;

		if (predicateFn( node )) {
			newNode = BinaryTree(
				node.value,
				node.parent,
				newLeft,
				newRight
			);
			if (newLeft) {
				newLeft.parent = newNode;
			}
			if (newRight) {
				newRight.parent = newNode;
			}
		}
		else {
			if (newLeft) {
				if (newRight) {
					newNode = BinaryTree(
						undefined,
						node.parent,
						newLeft,
						newRight
					);
					newLeft.parent = newRight.parent = newNode;

					if (newRight.left) {
						let minRightNode = newRight;
						while (minRightNode.left) {
							minRightNode = minRightNode.left;
						}

						newNode.value = minRightNode.value;

						if (minRightNode.right) {
							minRightNode.parent.left =
								minRightNode.right;
							minRightNode.right.parent =
								minRightNode.parent;
						}
						else {
							minRightNode.parent.left = undefined;
						}

						minRightNode.right =
							minRightNode.parent = undefined;
					}
					else {
						newNode.value = newRight.value;
						newNode.right = newRight.right;
						if (newRight.right) {
							newRight.right.parent = newNode;
						}
					}
				}
				else {
					return newLeft;
				}
			}
			else {
				return newRight;
			}
		}

		return newNode;
	}
};
```

这段代码的大部分是为了专门处理当存在重复的树形结构中的节点被“删除”（过滤掉）的时候，移动节点的父／子引用。

作为一个描述使用 `filter(..)` 的例子，让我们产生仅仅包含蔬菜的树：

```js
var vegetables = [ "asparagus", "avocado", "brocolli", "carrot",
	"celery", "corn", "cucumber", "lettuce", "potato", "squash",
	"zucchini" ];

var whatToBuy = BinaryTree.filter(
	// 将蔬菜从农产品清单中过滤出来
	node => vegetables.indexOf( node.value ) != -1,
	banana
);

// 购物清单
BinaryTree.reduce(
	(result,node) => result.concat( node.value ),
	[],
	whatToBuy
);
// ["avocado","cucumber"]
```

你会在简单列表中使用本章大多数的列表操作。但现在你发现这个概念适用于你可能需要的任何数据结构和操作。函数式编程可以广泛应用在许多不同的场景，这是非常强大的！

## 总结

三个强大通用的列表操作：

* `map(..)`: 转换列表项的值到新列表。
* `filter(..)`: 选择或过滤掉列表项的值到新数组。
* `reduce(..)`: 合并列表中的值，并且产生一个其他的值（经常但不总是非列表的值）。

其他一些非常有用的处理列表的高级操作：`unique(..)`、`flatten(..)` 和 `merge(..)`。

融合采用函数组合技术来合并多个相邻的 `map(..)`调用。这是常见的性能优化方式，并且它也使得列表操作更加自然。

列表通常以数组展现，但它也可以作为任何数据结构表达／产生一个有序的值集合。因此，所有这些“列表操作”都是“数据结构操作”。
