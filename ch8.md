# JavaScript轻量级函数式编程
# 第 8 章: 列表操作

你是否还沉迷于上一节介绍的闭包 ／ 对象之中？ 欢迎回来！

> 如果你能做一些令人惊叹的事情， 请持续保持下去。

本文之前已经简要的提及了一些功能函数：`map(..)`, `filter(..)`, 和 `reduce(..)`，现在深入了解一下。在Javascript中，这些功能函数通常被用于Array（即“list”）的原型上。因此可以很自然的将这些功能函数和数组或列表操作联系起来。

曾经我们谈及数组的这些方法， 我们希望概念性地清楚这些操作的用途。在这章中，弄明白为何有这些列表操作和这些操作如何工作同等重要。请跟上文章的节奏。

在本章内外，有大量的插图通过细小的任务（如将数组中的每一个数组加倍）来描述这些列表操作。这种方式通俗易懂。

但是别停留在这些简单示例的表面，而错过了更深的知识。通过对一系列任务建模来理解一些非常重要的函数式编程在列表操作中的价值。- 一些看起来不像列表的语句 - 作为列表操作，而不是单独执行。

这不仅仅是编写许多简练代码的技巧。 我们所要做的是，从命令式转变为声明式风格，使代码模式更容易辨认，从而更容易阅读。

但**要抓住更重要的**事情。 在命令式代码中，一组计算的中间结果都是通过赋值来存储的。代码中依赖的命令模式越多，代码中存在的错误就越难被验证 - 在逻辑上，值的意外突变，或隐藏的潜在原因／影响。

通过链接且／或组合列表操作，中间结果被隐式地跟踪，并在很大程度上避免了这些风险。

**注意** 相比前面几张，为了代码片段更加简练，我们将采用ES6的箭头函数。然而第二章中对于箭头函数的建议依旧适用于一般编码中。

## 非函数式编程列表处理

作为本章讨论的快速预览，我想调用一些看上去可能和Javascript列表和函数式编程的列表操作相关联的操作，但这些操作不是。这些不会在这里讨论，因为它们与一般的函数式编程最佳实践不一致：

* `forEach(..)`
* `some(..)`
* `every(..)`

`forEach(..)` 是迭代器辅助函数, 但是它有被设计为每个函数调用的副作用; 你会许已经猜测到了它为什么不是我们正在讨论的函数式编程列表操作！

`some(..)` 和 `every(..)` 鼓励使用纯函数 (具体来说，就像`filter(..)`这样的谓词函数), 但是它们不可避免地将列表化简为`true` 或 `false`的值,本质上就像搜索和匹配. 这两个工具函数并不适合用函数式编程来建模, 因此，这里也不再讨论它们。

## 映射

我们将于最基础和最基本的操作`map(..)`来开启函数式编程列表操作的探索。

映射是将一个值转换为另一个值。例如，如果你将`2`乘以`3`，你讲得到转换的结果`6`。 需要重点注意的是，我们并不是在讨论映射转换，因为函数时编程中的映射暗示了“就地” 改变或重新赋值。而映射转换则是将一个值从一个地方映射到另一个新的地方。

换句话说

```js
var x = 2, y;

// 转换 / 投影
y = x * 3;

// 变换 / 重新赋值
x = x * 3;
```
如果我们定义了乘`3`这样的函数，这个函数扮演了映射（转换）的角色。

```js
var multipleBy3 = v => v * 3;

var x = 2, y;

// 转换 / 投影
y = multiplyBy3( x );
```
我们可以自然的将单个值扩展到值的集合。 map(..)操作将列表中的所有值转换为新列表，如下图所示：

<p align="center">
	<img src="fig9.png" width="400">
</p>

实现 `map(..)`的代码如下:

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
**注意** `mapperFn, arr`的参数顺序，初一看像是倒退。但是这种方式是函数式编程的常规做法。因为这样做，可以使得这些功能函数更容易被组合。

`mapperFn(..)`函数按照列表的每一项做映射／转换，在映射／转换时也可以按照`idx` 和 `arr`。 这样做，可以和内置的数组的`map(..)`保持一致。在某些情况下，这些额外的细小的信息非常有用。

但是，在其他场景中，使用`mapperFn(..)`函数的时候，只想传递列表项参数。因为额外的参数可能会改变它的行为。

在第三章的“共同目的（All for one）”中，我们介绍了仅限接受单个参数（不管传入了多少参数）的 `unary(..)`函数。

在`mapperFn(..)`函数中，传入第三章中的限制 `parseInt(..)`函数，可以被安全地反复调用， 代码如下：

```js
map( ["1","2","3"], unary( parseInt ) );
// [1,2,3]
```

Javascript 提供了内建的数组操作方法`map(..)`，这个方法是的列表中的链式操作更为便利。

**注意** Javascript数组中的原型中定义的 操作(`map(..)`, `filter(..)`, 和 `reduce(..)`)的最后一个可选参数可以被用于绑定“this”到当前函数。我们在第二章中曾经讨论过“什么是this”，以及在函数式编程的最佳实践中应该避免使用`this`。基于这个原因，在这章中的示例中，我们不采用`this`绑定。

除了明显的字符和数字操作外，你可以对列表中的这些值类型进行操作。我们可以采用`map(..)`方法来通过函数列表转换得到这些函数返回的值，示例代码如下:

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

我们注意到关于`map(..)`的一些有趣的事情：我们通常假定列表是从左往右执行的，但`map(..)`并不需要在意这些次序。每一个转换应该独立于其他的转换，彼此没有关联。

映射普遍使用于并行处理的场景中，尤其在处理大列表时可以提升性能。但是在Javascript中，我们并没有看到这样的场景。因为这里不需要你传入诸如`mapperFn(..)`这样的纯函数，即便你**应当这样做**。 在无序执行回调时，如果传入了非纯函数，这将会是一个灾难。

尽管从理论上讲，单独的映射操作时独立的，JS不得不假设它们不是独立的。这是令人讨厌的。

### 同步 vs 异步

在这篇文章中，因为列表中的值都已存在，讨论的列表操作都是运行在同步模式下。`map(..)`函数在这里立即执行列表操作。但是映射还有另一种用法是将其当作事件处理函数使用，在迭代执行列表中的每一个元素触发执行该函数。

想象一下这样的场景：

```js
var newArr = arr.map();

arr.addEventListener( "value", multiplyBy3 );
```

现在，每一个值加入到`arr`中的时候，`multiplyBy3(..)`事件处理函数会被执行——映射函数——将加入的值当映射函数的参数执行，将执行结果加入到`newArr`。

我们所暗示的是数组，以及在数组上执行的操作，这些都是同步执行，然后这些相同的操作也可以作用在“延迟列表”（即流）模型上。流会一直接受它的值，我们将在第十章中深入讨论它。

### 映射 vs Eaching

有些人提倡在迭代的时候采用`map(..)`替代`forEach(..)`，本质上结束到的值在传递的时候没有修改，但随后会产生一些副作用：

```js
[1,2,3,4,5]
.map( function mapperFn(v){
	console.log( v );			// 副作用!
	return v;
} )
..
```
这种技术似乎又用的原因是`map(..)`返回数组，这样你可以在它之后继续执行更多的操作。而`forEach(..)`执行后返回的的值是`undefined`。然后，我认为你应当避免这样使用 However,`map(..)`。因为这是一种明显用非函数式编程的方式操作核心的函数式编程操作，这样容易引起混乱。

你应该听过一句老话，用合适的工具做合适的事———锤子配钉子，螺丝刀配螺丝等等，对吗？这里有些细微的不同：指的是采用**恰当的方式**使用合适的工具。

锤子是挥动手敲的，如果你尝试采用嘴去钉钉子，效率会大打折扣。`map(..)`是用来映射值的，而不是带来副作用。

### 一个词: 函子

在这本书中，我们尽可能避免使用人为创造的函数式编程术语。我们有时候会使用官方术语，但在大多数时候，采用日常用语来描述更加通俗易懂。这里之所以要讨论函子的原因是我们已经了解了它的作用，并且这个词在函数式编程文献中被大量使用。你不会被这个词吓到而带来副作用。

函子是采用运算函数有效用操作的值。

如果问题中的值时复合的，意味着它是由单个值组成，就像数组中的情况一样。例如，函子在每个值上执行操作符函数。函子创建的值是有所有单独操作函数返回的结果。

这就是用`map(..)`来描述我们所看到东西的一种奇特方式。`map(..)`函数采用关联值（数组）和映射函数（操作函数），并为数组中的每个元素单独地执行映射函数。最后，它返回包含所有新映射值的新数组

另一个例子：字符串函子是一个字符串加上一个功能函数，这个功能函数在字符串的所有字符上执行某些函数操作，返回包含处理过的字符的字符串。参考如下特意写的例子：

```js
function uppercaseLetter(c) {
	var code = c.charCodeAt( 0 );

	// lowercase letter?
	if (code >= 97 && code <= 122) {
		// uppercase it!
		code = code - 32;
	}

	return String.fromCharCode( code );
}

function stringMap(mapperFn,str) {
	return [...str].map( mapperFn ).join( "" );
}

stringMap( uppercaseLetter, "Hello World!" );
// HELLO WORLD!
```

`stringMap(..)`允许字符串作为它的函子。 你可以定义为任何数据结构定义一个映射函数。只要功能函数满足这些规则，这个数据结构就是一个函子。

## 过滤器

想象一下，我带着空篮子去逛食品杂货店的水果区。这里有很多水果（苹果，橙子和香蕉）。我真的很饿，因此我想要尽可能多的水果, 但是我真的更喜欢圆形的水果（苹果和香蕉）. 因此我逐一筛选每一个水果，然后带着装满苹果和橙子的篮子离开.

我们将这个筛选的过程称为“过滤”。将这次购物描述为从空篮子开始，然后只过滤（挑选，包含）出苹果和橙子，或者从所有的水果中过滤掉（跳过，不包括）香蕉。这种描述不是更自然吗？

如果你在一锅水里面做意大利面条，然后将这锅面条倒入滤网（过滤）中，你是不死过滤了意大利面条，或者过滤掉了水？ 如果你将咖啡渣放入过滤取中，然后泡一杯咖啡，你是不是将咖啡过滤到了杯子里，或将咖啡渣过滤掉？

你有没有发现过滤的结果取决于你想要把什么保留在过滤器中，还是将其通过过滤器？

那么在航空／酒店网站上如何指定过滤选项呢？你是按照你的标准过滤结果，还是将不符合标准的过滤掉？ 仔细想想，这个例子也许和前面有不相同的语意。

取决于你的想法，过滤是排除的或者包含的，这种概念上的融合，使其难以理解。

我认为最通常的理解过滤（在编程之外）过滤是剔除掉不需要的成员。不幸的是，在程序中我们基本上将这个语意转变为更像是过滤需要的成员。

列表的`filter(..)`操作采用一个函数确定每一项在新数组中是保留还是剔除。这个函数返回`true`将保留这一项，返回`false`将剔除这一项。这样返回`true` / `false`来做决定的函数，有一个特别的称谓：断言函数。

如果你认为`true`是积极的信号，`filter(..)`的定义是，你说“保留”一个值，而不是“抛弃”一个值。

如果`filter(..)`被用语剔除操作，你需要转动你的脑子，积极的返回`false`发出排除的信号，并且被动的返回`true`来让一个值通过过滤器。

这种语意上不匹配的原因是你会将这个函数命名为`predicateFn(..)`，这对于代码的可读性有意义，我们很快会讨论这一点。

下图很形象的介绍了列表间的`filter(..)`操作：


<p align="center">
	<img src="fig10.png" width="400">
</p>

实现 `filter(..)` 的代码如下:

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

注意，就像之前的`mapperFn(..)`， `predicateFn(..)`不仅仅传入了值，还传入了`idx` 和 `arr` 。如果有必要，采用 `unary(..)` 来限制它的参数。

正如`map(..)`，`filter(..)`也是JS数组内置支持的功能函数。

我们将谓词函数定义这样：

```js
var whatToCallIt = v => v % 2 == 1;
```

这个函数采用`v % 2 == 1`来返回`true`或`false`. 这里的效果是，值为偶数时返回`true`，值为奇数时返回`false`。 这样，我们该如何命名这个函数？一个很自然的名字可能是：

```js
var isOdd = v => v % 2 == 1;
```

考虑一下如何在你的代码中使用`isOdd(..)`来做简单的值检查：


```js
var midIdx;

if (isOdd( list.length )) {
	midIdx = (list.length + 1) / 2;
}
else {
	midIdx = list.length / 2;
}
```

有感觉了，对吧？让我们采用内置的数组操作`filter(..)`来对一组值做筛选：

```js
[1,2,3,4,5].filter( isOdd );
// [1,3,5]
```

如果让你描述`[1,3,5]`这个结果，你是说“我将偶去掉了”，还是说“我将奇数挑选出来了” ？我认为前者是更自然的描述。但代码可读性好的是后者。阅读代码几乎是逐字的，我们“挑选的每一个数字都是奇数”。

我个人觉得这语意混乱。 对于经验丰富的开发者，这里毫无疑问有大量的先例。但是对于一个新手来说，这个逻辑表达看上去不采用双重否定不好表达，换句话说，采用双重否定来表达。

为了便以理解，我们可以将这个函数从`isOdd(..)`重命名为`isEven(..)`：

```js
var isEven = v => v % 2 == 1;

[1,2,3,4,5].filter( isEven );
// [1,3,5]
```
耶，但是这个函数名变得无意义，下面的示例中，传入的偶数，确返回了`false`

```js
isEven( 2 );		// false
```

呸！

回顾在第三章中的"No Points"， 我们定义`not(..)`操作来反转谓词函数，代码如下：


```js
var isEven = not( isOdd );

isEven( 2 );		// true
```

但在前面定义的`filter(..)`方式中，无法使用**这个**`isEven(..)`，因为它的逻辑已经反转了。我们将以偶数结束，而不是奇数，我们需要这么做：

```js
[1,2,3,4,5].filter( not( isEven ) );
// [1,3,5]
```
这样完全违背了我们的初衷，所以我们不要这么做。这样，我们只是在兜圈子。

### 过滤掉 & 过滤

为了消除这些困惑，我们定义`filterOut(..)`函数来执行**过滤掉**那些值，而实际上其内部执行否定的谓词检查。这样，我们将已经定义的`filter(..)`设置别名为`filterIn(..)`。

```js
var filterIn = filter;

function filterOut(predicateFn,arr) {
	return filterIn( not( predicateFn ), arr );
}
```

现在，我们可以可以在任意过滤操作中，使用语意化的过滤器，代码如下所示：

```js
isOdd( 3 );								// true
isEven( 2 );							// true

filterIn( isOdd, [1,2,3,4,5] );			// [1,3,5]
filterOut( isEven, [1,2,3,4,5] );		// [1,3,5]
```

我认为采用`filterIn(..)`和`filterOut(..)`（在Ramda中称之为`reject(..)`）会让代码的可读性比仅仅采用`filter(..)`更好。


## Reduce

`map(..)`和`filter(..)`产生新的数组，而第三种操作（`reduce(..)`）典型地将列表中的值合并（或减少）到单个值（非列表），比如数字或者字符串。本章后续会探讨如何采用高级的方式使用`reduce(..)`。`reduce(..)`是函数式编程中的最重要的工具之一。就像瑞士军刀一样，具有丰富的用途。

组合或缩减被抽象的定义为将两个值转换成一个值。有些函数式编程将其称为“折叠”，就像你将两个值合并到一个值。我认为这对于可视化是很有帮助的。

就像映射和过滤，合并到方式完全取决于你，一般取决于列表中值的类型。例如，数字通常采用算术计算合并，字符串才有拼接的方式合并，函数才有组合调用来合并。

有时候，缩减操作会指定一个`initialValue`，然后将这个初始值和列表的第一个元素合并。然后逐一和列表中剩余的元素合并。如下图所示：

<p align="center">
	<img src="fig11.png" width="400">
</p>

你也可以去掉上述的`initialValue`，直接将第一个列表元素当做`initialValue`，然后和列表中的第二个元素合并，如下图所示：

<p align="center">
	<img src="fig12.png" width="400">
</p>

**警告：** 在JavaScript中，如果在缩减操作的列表中没有一个值（在数组中，或没有指定`initialValue`），将会抛出异常。如果一个缩减操作的列表可能为空的时候，需要小心采用不指定`initialValue`的方式。

传递给`reduce(..)`执行缩减操作的函数执行一般称为缩减器。缩减器和之前介绍的映射和谓词函数有不同的特征。缩减器主要接受当前的缩减结果和下一个值来做缩减操作。每一步缩减结果通常称为累加器。

例如，对5，10，15采用初始值为3执行乘的缩减操作：

1. `3` * `5` = `15`
2. `15` * `10` = `150`
3. `150` * `15` = `2250`

在JavaScript直接采用内建于列表的`reduce(..)`方法执行缩减操作：

```js
[5,10,15].reduce( (product,v) => product * v, 3 );
// 2250
```

我们可以采用下面的方式实现`reduce(..)`：

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

就像`map(..)`和`filter(..)`， 缩减函数也传递`idx`和`arr`参数，以备缩减操作使用。我不经常使用这些参数，但我觉得保留这些参数是不错的。

在第四章中，我们讨论了`compose(..)`工具和`reduce(..)`的实现：

```js
function compose(...fns) {
	return function composed(result){
		return fns.reverse().reduce( function reducer(result,fn){
			return fn( result );
		}, result );
	};
}
```
基于不同的组合，未了说明`reduce(..)`，可以认为缩减器将函数从做到右组合（就像`pipe(..)`做到的事情）。在列表中这样使用：

```js
var pipeReducer = (composedFn,fn) => pipe( composedFn, fn );

var fn =
	[3,17,6,4]
	.map( v => n => v * n )
	.reduce( pipeReducer );

fn( 9 );			// 11016  (9 * 3 * 17 * 6 * 4)
fn( 10 );			// 12240  (10 * 3 * 17 * 6 * 4)
```
不幸的是，`pipeReducer(..)`是非点自由（见第三章中的“无点”），但我们不能仅仅最为缩减器本身传递`pipe(..)`，因为它是可变的；传递给`reduce(..)`额外的参数（`idx`和`arr`）会产生问题。

前面，我们讨论采用`unary(..)`来限制`mapperFn(..)`或`predicateFn(..)`仅采用一个参数。`binary(..)`做了类似的事情，但在`reducerFn(..)`中限制了两个参数：

```js
var binary =
	fn =>
		(arg1,arg2) =>
			fn( arg1, arg2 );
```

采用`binary(..)`，相比之前的示例有一些简洁：

```js
var pipeReducer = binary( pipe );

var fn =
	[3,17,6,4]
	.map( v => n => v * n )
	.reduce( pipeReducer );

fn( 9 );			// 11016  (9 * 3 * 17 * 6 * 4)
fn( 10 );			// 12240  (10 * 3 * 17 * 6 * 4)
```

不像`map(..)`和`filter(..)`，传入数组的次序没有什么要求。`reduce(..)`确定的采用从左到右的处理方式。如果你想从右到左缩减，JavaScript提供了另一个和`reduce(..)`相似的函数：`reduceRight(..)`

```js
var hyphenate = (str,char) => str + "-" + char;

["a","b","c"].reduce( hyphenate );
// "a-b-c"

["a","b","c"].reduceRight( hyphenate );
// "c-b-a"
```

`reduce(..)`采用从左到右的方式工作，很自然的联想到组合函数中的`pipe(..)`。`reduceRight(..)`从右往左的方式能自然的执行`compose(..)`。 因此，我们重新采用`reduceRight(..)`实现`compose(..)` :

```js
function compose(...fns) {
	return function composed(result){
		return fns.reduceRight( function reducer(result,fn){
			return fn( result );
		}, result );
	};
}
```

这样，我们不需要执行`fns.reverse()`; 我们只需要从另一个方向执行缩减操作！

### 映射是缩减

`map(..)`操作本质来说是迭代，因此，它也可以用缩减（`reduce(..)`）来实现。实现的技巧是将`reduce(..)`的`initialValue`看成它自身的空数组。在这种情况下，缩减操作的结果是另一个列表。

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

**注意** 我们这个缩减器来作弊，并允许采用`list.push(..)`去改变传入的列表带来的副作用。一般来说，这并不是一个好主意，但我们清楚创建和传入`[]`列表，这样就不那么危险了。创建一个新的列表，并将val合并到这个列表的最后面。这样更有条理，并且性能开销较小。我们将在附录A中讨论这种作弊。

通过`reduce(..)`实现`map(..)`，并不是表面上的明显的步骤，甚至是一种改善。然而，这种能力对于理解更高级的技术是至关重要的，如在附录A中的“转换”。


### 过滤器是缩减

就像通过`reduce(..)`实现`map(..)`一样，也可以使用它实现`filter(..)`：

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

**注意** 这里有更加不纯的缩减器作弊。不采用`list.push(..)`，我们也可以采用`list.concat(..)`并返回合并后的新列表。我们将在附录A中继续介绍这个欺骗。


## 高级列表操作

现在，我们对这些基础的列表操作`map(..)`, `filter(..)`,和`reduce(..)`感到比较舒服。让我们看看一些更复杂的操作，这些操作在某些场合下很有用。这些常用的功能函数存在于许多函数式编程的类库中。


### 去重

筛选列表中的元素，仅仅保留唯一的值。基于`indexOf(..)`函数查找（这里采用`===`严格等于表达式）：

```js
var unique =
	arr =>
		arr.filter(
			(v,idx) =>
				arr.indexOf( v ) == idx
		);
```

实现的原理是，当从左往右筛选元素时，列表项的`idx`位置和`indexOf(..)`找到的位置相等时，表明该列表项第一次出现，在这种情况下，将列表项加入到新数组中。


另一种实现`unique(..)`的方式是遍历`arr`，当列表项不能在新列表中找到时，将其插入到新的列表中。这样可以采用`reduce(..)`来实现：

```js
var unique =
	arr =>
		arr.reduce(
			(list,v) =>
				list.indexOf( v ) == -1 ?
					( list.push( v ), list ) : list
		, [] );
```

**注意** 这里还有很多其他的方式实现这个去重算法，比如循环，并且其中不少还更高效，实现方式更聪明。然而，这两种方式的优点是，它们使用了内建的列表操作，它们能更方便的和其他列表操作链式调用／组合调用。我们会在本章的后面进一步讨论这些。

`unique(..)`  产生去重的新的列表

```js
unique( [1,4,7,1,3,1,7,9,2,6,4,0,5,3] );
// [1, 4, 7, 3, 9, 2, 6, 0, 5]
```

### 扁平化

大多数时候，你看到的数组的列表项不是扁平的，很多时候，数组嵌套了数组，例如：

```js
[ [1, 2, 3], 4, 5, [6, [7, 8]] ]
```

如果你想将其转化成下来的形式:

```js
[ 1, 2, 3, 4, 5, 6, 7, 8 ]
```

我们寻找的这个操作通常称为`flatten(..)`。它可以采用如同瑞士军刀般的`reduce(..)`实现：

```js
var flatten =
	arr =>
		arr.reduce(
			(list,v) =>
				list.concat( Array.isArray( v ) ? flatten( v ) : v )
		, [] );
```

**注意** 这种处理嵌套列表的实现方式依赖于递归，我们将会在后面的章节中进一步讨论。

在嵌套数组（任意嵌套层次）中使用`flatten(..)`：

```js
flatten( [[0,1],2,3,[4,[5,6,7],[8,[9,[10,[11,12],13]]]]] );
// [0,1,2,3,4,5,6,7,8,9,10,11,12,13]
```

也许你会限制递归的层次到指定的层次。我们可以通过增加额外的`depth`参数来实现：

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

`flatten(..)`的常用用法之一是当你映射一组元素列表，并且将每一项值从原来的值转换为数组。例如：

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

返回的值是二维数组，这样也许给处理带来一些不便。如果我们想得到所有名字的一维数组，我们可以对这个结果执行`flatten(..)`：

```js
flatten(
	firstNames
	.map( entry => [entry.name].concat( entry.variations ) )
);
// ["Jonathan","John","Jon","Jonny","Stephanie","Steph","Stephy","Frederick",
//  "Fred","Freddy"]
```
除了稍显啰嗦之外，将`map(..)`和`flatten(..)`采用独立的步骤的最主要的缺陷是关于性能方面。它会处理列表两次。

函数式编程的类库中，通常会定义一个`flatMap(..)`（经常命名为`chain(..)`）函数。这个函数将映射然后扁平化的操作组合起来。 For consistency and ease of composition (via currying), the `flatMap(..)` / `chain(..)` utility typically matches the `mapperFn, arr` parameter order that we saw earlier with the standalone `map(..)`, `filter(..)`, and `reduce(..)` utilities.

```js
flatMap( entry => [entry.name].concat( entry.variations ), firstNames );
// ["Jonathan","John","Jon","Jonny","Stephanie","Steph","Stephy","Frederick",
//  "Fred","Freddy"]
```

天真的采用独立的两步来实现`flatMap(..)`：

```js
var flatMap =
	(mapperFn,arr) =>
		flatten( arr.map( mapperFn ), 1 );
```

**注意：** 我们采用`1`来指定扁平化的层级，因为通常`flatMap(..)`的定义是扁平化第一级。

尽管这种实现方式依旧会处理列表两次，这带来了不好的性能。我们可以将这些操作采用`reduce(..)`手动合并：

```js
var flatMap =
	(mapperFn,arr) =>
		arr.reduce(
			(list,v) =>
				list.concat( mapperFn( v ) )
		, [] );
```

现在通过`flatMap(..)`带来了便利性和性能，有时你可能需要其他操作，如和`filter(..)`混合使用。如果是这样的话，将`map(..)`和`flatten(..)`独立开来始终是更合适的。

### Zip

到目前为止，我们介绍的列表操作都是操作单个列表。 但是在某些情况下，需要操作多个列表。一个闻名的操作是交替选择两个输入的列表中的值，并将得到的值组成子列表。这个操作称之为`zip(..)`:

```js
zip( [1,3,5,7,9], [2,4,6,8,10] );
// [ [1,2], [3,4], [5,6], [7,8], [9,10] ]
```

选择值`1`和`2`到子列表`[1,2]`，然后选择`3`和`4`到列表`[3,4]`，然后逐一选择。`zip(..)`被定义为将两个列表中的值挑选出来。如果两个列表的的元素的个数不一致，那么选择的列表的长度以较短的列表为准，多的那个列表中的元素将被忽略掉。

一种`zip(..)`的实现:

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
采用`arr1.slice()`和`arr2.slice()`可以确保`zip(..)`是纯的，不会因得到引用而造成的副作用。

**注意** 这里存在一些明显的非函数式编程的东西在这个实现中。有必要在通过`while`的列表的循环和变换中采用`shift()`和`push(..)`。在本书前面，我认为在纯函数中使用非纯的行为（通常是为了性能）是明智的，只要其产生的副作用完全包含在这个函数内部。这种实现是安全纯净的。

### 合并

采用插入每个列表中的值的方式合并两个列表，如下所示：

```js
mergeLists( [1,3,5,7,9], [2,4,6,8,10] );
// [1,2,3,4,5,6,7,8,9,10]
```

它可能不是那么明显，但其结果看上去和采用`flatten(..)`和`zip(..)`组合相似，代码如下:

```js
zip( [1,3,5,7,9], [2,4,6,8,10] );
// [ [1,2], [3,4], [5,6], [7,8], [9,10] ]

flatten( [ [1,2], [3,4], [5,6], [7,8], [9,10] ] );
// [1,2,3,4,5,6,7,8,9,10]

// composed:
flatten( zip( [1,3,5,7,9], [2,4,6,8,10] ) );
// [1,2,3,4,5,6,7,8,9,10]
```

然后，回顾`zip(..)`，它在两个列表中选择值，选择的列表项的长度仅仅到较短列表的长度，会忽略掉较长列表中的剩余部分。合并两个数组一般会保留这些额外的列表项。并且`flatten(..)`同构递归处理嵌套列表，但是你可能只期望合并列表的时候较浅的扁平化，保留嵌套的列表。

这样，我们按照更符合我们预期的定义`mergeLists(..)`如下：

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

**注意** 许多函数式编程类库并不会定义`mergeLists(..)`，反而会定义`merge(..)`方法来合并两个对象的属性。这样的`merge(..)`的结果和我们的`mergeLists(..)`不同。

另外，这里有几个选择采用缩减器实现合并列表：

```js
// via @rwaldron
var mergeReducer =
	(merged,v,idx) =>
		(merged.splice( idx * 2, 0, v ), merged);


// via @WebReflection
var mergeReducer =
	(merged,v,idx) =>
		merged
			.slice( 0, idx * 2 )
			.concat( v, merged.slice( idx * 2 ) );
```

采用 `mergeReducer(..)`执行合并:

```js
[1,3,5,7,9]
.reduce( mergeReducer, [2,4,6,8,10] );
// [1,2,3,4,5,6,7,8,9,10]
```

**提示**：我们将在本章后面采用`mergeReducer(..)`这个技巧。

## 方法 vs 独立


A common source of frustration for FPers in JavaScript is unifying their strategy for working with utilities when some of them are provided as standalone functions -- think about the various FP utilities we've derived in previous chapters -- and others are methods of the array prototype -- like the ones we've seen in this chapter.

The pain of this problem becomes more evident when you consider combining multiple operations:

```js
[1,2,3,4,5]
.filter( isOdd )
.map( double )
.reduce( sum, 0 );					// 18

// vs.

reduce(
	map(
		filter( [1,2,3,4,5], isOdd ),
		double
	),
	sum,
	0
);									// 18
```

Both API styles accomplish the same task, but they have very different ergonomics. Many FPers will prefer the latter to the former, but the former is unquestionably more common in JavaScript. One thing specifically that's disliked about the latter is the nesting of the calls. The preference for the method chain style -- typically called a fluent API style, as in jQuery and other tools -- is that it's compact/concise and it reads in declarative top-down order.

The visual order for that manual composition of the standalone style is neither strictly left-to-right (top-to-bottom) nor right-to-left (bottom-to-top); it's inner-to-outer, which harms the readability.

Automatic composition normalizes the reading order as right-to-left (bottom-to-top) for both styles. So, to explore the implications of the style differences, let's examine composition specifically; it seems like it should be straightforward, but it's a little awkward in both cases.

### Composing Method Chains

The array methods receive the implicit `this` argument, so despite their appearance, they can't be treated as unary; that makes composition more awkward. To cope, we'll first need a `this`-aware version of `partial(..)`:

```js
var partialThis =
	(fn,...presetArgs) =>
		// intentionally `function` to allow `this`-binding
		function partiallyApplied(...laterArgs){
			return fn.apply( this, [...presetArgs, ...laterArgs] );
		};
```

We'll also need a version of `compose(..)` that calls each of the partially applied methods in the context of the chain -- the input value it's being "passed" (via implicit `this`) from the previous step:

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

And using these two `this`-aware utilities together:

```js
composeChainedMethods(
   partialThis( Array.prototype.reduce, sum, 0 ),
   partialThis( Array.prototype.map, double ),
   partialThis( Array.prototype.filter, isOdd )
)
( [1,2,3,4,5] );					// 18
```

**Note:** The three `Array.prototype.XXX`-style references are grabbing references to the built-in `Array.prototype.*` methods so that we can reuse them with our own arrays.

### Composing Standalone Utilities

Standalone `compose(..)`-style composition of these utilities doesn't need all the `this` contortions, which is its most favorable argument. For example, we could define standalones as:

```js
var filter = (arr,predicateFn) => arr.filter( predicateFn );

var map = (arr,mapperFn) => arr.map( mapperFn );

var reduce = (arr,reducerFn,initialValue) =>
	arr.reduce( reducerFn, initialValue );
```

But this particular standalone style suffers from its own awkwardness; the cascading array context is the first argument rather than the last, so we have to use right-partial application to compose them:

```js
compose(
	partialRight( reduce, sum, 0 ),
	partialRight( map, double ),
	partialRight( filter, isOdd )
)
( [1,2,3,4,5] );					// 18
```

That's why FP libraries typically define `filter(..)`, `map(..)`, and `reduce(..)` to alternately receive the array last instead of first. They also typically automatically curry the utilities:

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

Working with the utilities defined in this way, the composition flow is a bit nicer:

```js
compose(
	reduce( sum )( 0 ),
	map( double ),
	filter( isOdd )
)
( [1,2,3,4,5] );					// 18
```

The cleanliness of this approach is in part why FPers prefer the standalone utility style instead of instance methods. But your mileage may vary.

### Adapting Methods To Standalones

In the previous definition of `filter(..)` / `map(..)` / `reduce(..)`, you might have spotted the common pattern across all three: they all dispatch to the corresponding native array method. So, can we generate these standalone adaptations with a utility? Yes! Let's make a utility called `unboundMethod(..)` to do just that:

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

And to use this utility:

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

**Note:** `unboundMethod(..)` is called `invoker(..)` in Ramda.

### Adapting Standalones To Methods

If you prefer to work with only array methods (fluent chain style), you have two choices. You can:

1. Extend the built-in `Array.prototype` with additional methods.
2. Adapt a standalone utility to work as a reducer function and pass it to the `reduce(..)` instance method.

**Don't do (1).** It's never a good idea to extend built-in natives like `Array.prototype` -- unless you define a subclass of `Array`, but that's beyond our discussion scope here. In an effort to discourage bad practices, we won't go any further into this approach.

Let's **focus on (2)** instead. To illustrate this point, we'll convert the recursive `flatten(..)` standalone utility from earlier:

```js
var flatten =
	arr =>
		arr.reduce(
			(list,v) =>
				list.concat( Array.isArray( v ) ? flatten( v ) : v )
		, [] );
```

Let's pull out the inner `reducer(..)` function as the standalone utility (and adapt it to work without the outer `flatten(..)`):

```js
// intentionally a function to allow recursion by name
function flattenReducer(list,v) {
	return list.concat(
		Array.isArray( v ) ? v.reduce( flattenReducer, [] ) : v
	);
}
```

Now, we can use this utility in an array method chain via `reduce(..)`:

```js
[ [1, 2, 3], 4, 5, [6, [7, 8]] ]
.reduce( flattenReducer, [] )
// ..
```

## Looking For Lists

So far, most of the examples have been rather trivial, based on simple lists of numbers or strings. Let's now talk about where list operations can start to shine: modeling an imperative series of statements declaratively.

Consider this base example:

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

First, let's observe that the five variable declarations and the running series of `if` conditionals guarding the function calls are effectively one big composition of these six calls `getCurrentSession()`, `getSessionId(..)`, `lookupUser(..)`, `getUserId(..)`, `lookupOrders(..)`, and `processOrders(..)`. Ideally, we'd like to get rid of all these variable declarations and imperative conditionals.

Unfortunately, the `compose(..)` / `pipe(..)` utilties we explored in Chapter 4 don't by themselves offer a convenient way to express the `!= null` conditionals in the composition. Let's define a utility to help:

```js
var guard =
	fn =>
		arg =>
			arg != null ? fn( arg ) : arg;
```

This `guard(..)` utility lets us map the five conditional-guarded functions:

```js
[ getSessionId, lookupUser, getUserId, lookupOrders, processOrders ]
.map( guard )
```

The result of this mapping is an array of functions that are ready to compose (actually, pipe, in this listed order). We could spread this array to `pipe(..)`, but since we're already doing list operations, let's do it with a `reduce(..)`, using the session value from `getCurrentSession()` as the initial value:

```js
.reduce(
	(result,nextFn) => nextFn( result )
	, getCurrentSession()
)
```

Next, let's observe that `getSessionId(..)` and `getUserId(..)` can be expressed as a mapping from the respective values `"sessId"` and `"uId"`:

```js
[ "sessId", "uId" ].map( propName => partial( prop, propName ) )
```

But to use these, we'll need to interleave them with the other three functions (`lookupUser(..)`, `lookupOrders(..)`, and `processOrders(..)`) to get the array of five functions to guard / compose as discussed above.

To do the interleaving, we can model this as list merging. Recall `mergeReducer(..)` from earlier in the chapter:

```js
var mergeReducer =
	(merged,v,idx) =>
		(merged.splice( idx * 2, 0, v ), merged);
```

We can use `reduce(..)` (our swiss army knife, remember!?) to "insert" `lookupUser(..)` in the array between the generated `getSessionId(..)` and `getUserId(..)` functions, by merging two lists:

```js
.reduce( mergeReducer, [ lookupUser ] )
```

Then we'll concatenate `lookupOrders(..)` and `processOrders(..)` onto the end of the running functions array:

```js
.concat( lookupOrders, processOrders )
```

To review, the generated list of five functions is expressed as:

```js
[ "sessId", "uId" ].map( propName => partial( prop, propName ) )
.reduce( mergeReducer, [ lookupUser ] )
.concat( lookupOrders, processOrders )
```

Finally, to put it all together, take this list of functions and tack on the guarding and composition from earlier:

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

Gone are all the imperative variable declarations and conditionals, and in their place we have clean and declarative list operations chained together.

If this version is harder for you read right now than the original, don't worry. The original is unquestionably the imperative form you're probably more familiar with. Part of your evolution to become a functional programmer is to develop a recognition of FP patterns such as list operations. Over time, these will jump out of the code more readily as your sense of code readability shifts to declarative style.

Before we leave this topic, let's take a reality check: the example here is heavily contrived. Not all code segments will be straightforwardly modeled as list operations. The pragmatic take-away is to develop the instinct to look for these opportunities, but not get too hung up on code acrobatics; some improvement is better than none. Always step back and ask if you're **improving or harming** code readability.

## Fusion

As you roll FP list operations into more of your thinking about code, you'll likely start seeing very quickly chains that combine behavior like:

```js
..
.filter(..)
.map(..)
.reduce(..);
```

And more often than not, you're also probably going to end up with chains with multiple adjacent instances of each operation, like:

```js
someList
.filter(..)
.filter(..)
.map(..)
.map(..)
.map(..)
.reduce(..);
```

The good news is the chain-style is declarative and it's easy to read the specific steps that will happen, in order. The downside is that each of these operations loops over the entire list, meaning performance can suffer unnecessarily, especially if the list is longer.

With the alternate standalone style, you might see code like this:

```js
map(
	fn3,
	map(
		fn2,
		map( fn1, someList )
	)
);
```

With this style, the operations are listed from bottom-to-top, and we still loop over the list 3 times.

Fusion deals with combining adjacent operators to reduce the number of times the list is iterated over. We'll focus here on collapsing adjacent `map(..)`s as it's the most straightforward to explain.

Imagine this scenario:

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

Think about each value that goes through this flow of transformations. The first value in the `words` list starts out as `"Mr."`, becomes `"Mr"`, then `"MR"`, and then passes through `elide(..)` unchanged. Another piece of data flows: `"responsible"` -> `"responsible"` -> `"RESPONSIBLE"` -> `"RESPONS..."`.

In other words, you could think of these data transformations like this:

```js
elide( upper( removeInvalidChars( "Mr." ) ) );
// "MR"

elide( upper( removeInvalidChars( "responsible" ) ) );
// "RESPONS..."
```

Did you catch the point? We can express the three separate steps of the adjacent `map(..)` calls as a composition of the transformers, since they are all unary functions and each returns the value that's suitable as input to the next. We can fuse the mapper functions using `compose(..)`, and then pass the composed function to a single `map(..)` call:

```js
words
.map(
	compose( elide, upper, removeInvalidChars )
);
// ["MR","JONES","ISNT","RESPONS...","FOR","THIS","DISASTER"]
```

This is another case where `pipe(..)` can be a more convenient form of composition, for its ordering readability:

```js
words
.map(
	pipe( removeInvalidChars, upper, elide )
);
// ["MR","JONES","ISNT","RESPONS...","FOR","THIS","DISASTER"]
```

What about fusing two or more `filter(..)` predicate functions? Typically treated as unary functions, they seem suitable for composition. But the wrinkle is they each return a different kind of value (`boolean`) than the next one would want as input. Fusing adjacent `reduce(..)` calls is also possible, but reducers are not unary so that's a bit more challenging; we need more sophisticated tricks to pull this kind of fusion off. We'll cover these advanced techniques in Appendix A "Transducing".

## Beyond Lists

So far we've been discussing operations in the context of the list (array) data structure; it's by far the most common scenario you encounter them. But in a more general sense, these operations can be performed against any collection of values.

Just as we said earlier that array's `map(..)` adapts a single-value operation to all its values, any data structure can provide a `map(..)` operation to do the same. Likewise, it can implement `filter(..)`, `reduce(..)`, or any other operation that makes sense for working with the data structure's values.

The important part to maintain in the spirit of FP is that these operators must behave according to value immutability, meaning that they must return a new data structure rather than mutating the existing one.

Let's illustrate with a well-known data structure: the binary tree. A binary tree is a node (just an object!) that has two references to other nodes (themselves binary trees), typically referred to as *left* and *right* child trees. Each node in the tree holds one value of the overall data structure.

<p align="center">
	<img src="fig7.png" width="250">
</p>

For ease of illustration, we'll make our binary tree a binary search tree (BST). However, the operations we'll identify work the same for any regular non-BST binary tree.

**Note:** A binary search tree is a general binary tree with a special constraint on the relationship of values in the tree to each other. Each value of nodes on the left side of a tree is less than the value of the node at the root of that tree, which in turn is less than each value of nodes in the right side of the tree. The notion of "less than" is relative to the kind of data stored; it can be numerical for numbers, lexicographic for strings, etc. BSTs are useful because they make searching for a value in the tree straightforward and more efficient, using a recursive binary search algorithm.

To make a binary tree node object, let's use this factory function:

```js
var BinaryTree =
	(value,parent,left,right) => ({ value, parent, left, right });
```

For convenience, we make each node store the `left` and `right` child trees as well as a reference to its own `parent` node.

Let's now define a BST of names of common produce (fruits, vegetables):

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

In this particular tree structure, `banana` is the root node; this tree could have been set up with nodes in different locations, but still had a BST with the same traversal.

Our tree looks like:

<p align="center">
	<img src="fig8.png" width="450">
</p>

There are multiple ways to traverse a binary tree to process its values. If it's a BST (our's is!) and we do an *in-order* traversal -- always visit the left child tree first, then the node itself, then the right child tree -- we'll visit the values in ascending (sorted) order.

Since you can't just easily `console.log(..)` a binary tree like you can with an array, let's first define a convenience method, mostly to use for printing. `forEach(..)` will visit the nodes of a binary tree in the same manner as an array:

```js
// in-order traversal
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

**Note:** Working with binary trees lends itself most naturally to recursive processing. Our `forEach(..)` utility recursively calls itself to process both the left and right child trees. We'll cover recursion in more detail in a later chapter, where we'll cover recursion in that chapter on recursion.

Recall `forEach(..)` was described at the beginning of this chapter as only being useful for side effects, which is not very typically desired in FP. In this case, we'll use `forEach(..)` only for the side effect of I/O, so it's perfectly reasonable as a helper.

Use `forEach(..)` to print out values from the tree:

```js
BinaryTree.forEach( node => console.log( node.value ), banana );
// apple apricot avocado banana cantelope cherry cucumber grape

// visit only the `cherry`-rooted subtree
BinaryTree.forEach( node => console.log( node.value ), cherry );
// cantelope cherry cucumber grape
```

To operate on our binary tree data structure using FP patterns, let's start by defining a `map(..)`:

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

You might have assumed we'd `map(..)` only the node `value` properties, but in general we might actually want to map the tree nodes themselves. So, the `mapperFn(..)` is passed the whole node being visited, and it expects to receive a new `BinaryTree(..)` node back, with the transformation applied. If you just return the same node, this operation will mutate your tree and quite possibly cause unexpected results!

Let's map our tree to a list of produce with all uppercase names:

```js
var BANANA = BinaryTree.map(
	node => BinaryTree( node.value.toUpperCase() ),
	banana
);

BinaryTree.forEach( node => console.log( node.value ), BANANA );
// APPLE APRICOT AVOCADO BANANA CANTELOPE CHERRY CUCUMBER GRAPE
```

`BANANA` is a different tree (with all different nodes) than `banana`, just like calling `map(..)` on an array returns a new array. Just like arrays of other objects/arrays, if `node.value` itself references some object/array, you'll also need to handle manually copying it in the mapper function if you want deeper immutability.

How about `reduce(..)`? Same basic process: do an in-order traversal of the tree nodes. One usage would be to `reduce(..)` our tree to an array of its values, which would be useful in further adapting other typical list operations. Or we can `reduce(..)` our tree to a string concatenation of all its produce names.

We'll mimic the behavior of the array `reduce(..)`, which makes passing the `initialValue` argument optional. This algorithm is a little trickier, but still manageable:

```js
BinaryTree.reduce = function reduce(reducerFn,initialValue,node){
	if (arguments.length < 3) {
		// shift the parameters since `initialValue` was omitted
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

Let's use `reduce(..)` to make our shopping list (an array):

```js
BinaryTree.reduce(
	(result,node) => result.concat( node.value ),
	[],
	banana
);
// ["apple","apricot","avocado","banana","cantelope"
//   "cherry","cucumber","grape"]
```

Finally, let's consider `filter(..)` for our tree. This algorithm is trickiest so far because it effectively (not actually) involves removing nodes from the tree, which requires handling several corner cases. Don't get intimiated by the implementation, though. Just skip over it for now, if you prefer, and focus on how we use it instead.

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

The majority of this code listing is dedicated to handling the shifting of a node's parent/child references if it's "removed" (filtered out) of the duplicated tree structure.

As an example to illustrate using `filter(..)`, let's narrow our produce tree down to only vegetables:

```js
var vegetables = [ "asparagus", "avocado", "brocolli", "carrot",
	"celery", "corn", "cucumber", "lettuce", "potato", "squash",
	"zucchini" ];

var whatToBuy = BinaryTree.filter(
	// filter the produce list only for vegetables
	node => vegetables.indexOf( node.value ) != -1,
	banana
);

// shopping list
BinaryTree.reduce(
	(result,node) => result.concat( node.value ),
	[],
	whatToBuy
);
// ["avocado","cucumber"]
```

You will likely use most of the list operations from this chapter in the context of simple arrays. But now we've seen that the concepts apply to whatever data structures and operations you might need. That's a powerful expression of how FP can be widely applied to many different application scenarios!

## Summary

Three common and powerful list operations:

* `map(..)`: transforms values as it projects them to a new list.
* `filter(..)`: selects or excludes values as it projects them to a new list.
* `reduce(..)`: combines values in a list to produce some other (usually but not always non-list) value.

Other more advanced operations that can be very useful in processing lists: `unique(..)`, `flatten(..)`, and `merge(..)`.

Fusion uses function composition techniques to consolidate multiple adjacent `map(..)` calls. This is mostly a performance optimization, but it also improves the declarative nature of your list operations.

Lists are typically visualized as arrays, but can be generalized as any data structure that represents/produces an ordered collection of values. As such, all these "list operations" are actually "data structure operations".
