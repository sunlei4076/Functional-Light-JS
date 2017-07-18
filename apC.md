# JavaScript 轻量级函数式编程
# 附录 C：FP 函数库

如果您已经从头到尾通读了此书，请花一分钟的时间停下来回顾一下从第 1 章到现在的收获。相当漫长的一段旅程，不是吗？希望您已经收获了大量新知识，并为你的程序的功能性有深入的了解。

在本书即将完结时，我想给你提供一些关于使用官方 FP 函数库的快速指南。注意这并不是一个详细的文档，而是将你在结束“轻量级函数式编程”后进军真正的函数式编程时应该注意的东西快速梳理一下。

如果有可能，我建议你*不要*做重新造轮子这样的事情。如果你找到了一个能满足你需求的 FP 函数库，那么用它就对了。只有在你实在找不到合适的库来应对你面临的问题时，才应该使用本书提供的辅助实用函数——或者自己造轮子。

## 目录

在本书第 1 章曾列出了一个函数式编程库的列表，现在我们来扩展这个列表。我们不会涉及所有的库（它们之中有许多重叠），但下面这些你应该有所关注：

* [Ramda](http://ramdajs.com): 通用 FP 实用函数
* [Sanctuary](https://github.com/sanctuary-js/sanctuary): FP 类型 Ramda 伴侣
* [lodash/fp](https://github.com/lodash/lodash/wiki/FP-Guide): 通用 FP 实用函数
* [functional.js](http://functionaljs.com/): 通用 FP 实用函数
* [Immutable](https://github.com/facebook/immutable-js): 不可变数据结构
* [Mori](https://github.com/swannodette/mori): (ClojureScript Inspired) 不可变数据结构
* [Seamless-Immutable](https://github.com/rtfeldman/seamless-immutable): 不可变数据助手
* [tranducers-js](https://github.com/cognitect-labs/transducers-js): 数据转换器
* [monet.js](https://github.com/cwmyers/monet.js): Monad 类型

上面的列表只列出了所有函数式编程库的一小部分，并不是说没有在列表中列出的库就不好，也不是说列表中列出的就是最佳选择，总之这只是 JavaScript 函数式编程世界中的一瞥。您可以前往[这里](https://github.com/stoeffel/awesome-fp-js)查看更完整的函数式编程资源。

[Fantasy Land](https://github.com/fantasyland/fantasy-land)（又名 FL）是函数式编程世界中十分重要的学习资源之一，与其说它是一个库，不如说它是一本百科全书。

Fantasy Land 不是一份为初学者准备的轻量级读物，而是一个完整而详细的 JavaScript FP 路线图。为了尽可能提升互通性，FL 已经成为 JavaScript 函数式编程库遵循的实际标准。

Fantasy Land 与“轻量级函数式编程”的概念相反，它以火力全开的姿态进军 JavaScript 的 FP 世界。也就是说，当你的冒险超越本书时，FL 将会成为你接下来前进的方向。我建议您将其保存在收藏夹中，并在您使用本书的概念进行至少 6 个月的实战练习之后再回来。

## Ramda (0.23.0)

摘自 [Ramda 文档](http://ramdajs.com/):

> Ramda 函数自动地被柯里化。
>
> Ramda 函数的参数经过优化，更便于柯里化。需要被操作的数据往往放在最后提供。

我认为合理的设计是 Ramda 的优势之一。值得注意的是，Ramda 的柯里化形式（似乎大多数的库都是这种形式）是我们在第 3 章中讨论过的“松散柯里化”。

第 3 章的最后一个例子——我们定义无值（point-free）工具函数 `printIf()` ——可以在 Ramda 中这样实现：

```js
function output(msg) {
	console.log( msg );
}

function isShortEnough(str) {
	return str.length <= 5;
}

var isLongEnough = R.complement( isShortEnough );

var printIf = R.partial( R.flip( R.when ), [output] );

var msg1 = "Hello";
var msg2 = msg1 + " World";

printIf( isShortEnough, msg1 );			// Hello
printIf( isShortEnough, msg2 );

printIf( isLongEnough, msg1 );
printIf( isLongEnough, msg2 );			// Hello World
```

与我们在第 3 章中的实现相比有几处不同：

* 我们使用 `R.complement(..)` 而不是 `not(..)` 在 `isShortEnough(..)` 周围新建一个否定函数 `isLongEnough(..)`。

* 使用 `R.flip(..)` 而不是 `reverseArgs(..)` 函数，值得一提的是，`R.flip(..)` 仅交换头两个参数，而 `reverseArgs(..)` 会将所有参数反向。在这种情景下，`flip(..)` 更加方便，所以我们不再需要使用 `partialRight(..)` 或其他投机取巧的方式进行处理。

* `R.partial(..)` 所有的后续参数以单个数组的形式存在。

* 因为 Ramda 使用松散柯里化，因此我们不需要使用 `R.uncurryN(..)` 来获得一个包含所有参数的 `printIf(..)`。如果我们这样做了，就相当于使用 `R.uncurryN(2, ..)` 包裹 `R.partial(..)` 进行调用，这是完全没有必要的。

Ramda 是一个受欢迎的、功能强大的库。如果你想要在你的代码中实践 FP，从 Ramda 开始是个不错的选择。

## Lodash/fp (4.17.4)

Lodash 是整个 JS 生态系统中最受欢迎的库。Lodash 团队发布了一个“FP 友好”的 API 版本—— ["lodash/fp"](https://github.com/lodash/lodash/wiki/FP-Guide)。

在第 8 章中，我们讨论了合并独立列表操作（`map(..)`、`filter(..)` 以及 `reduce(..)`）。使用“lodash/fp”时，你可以这样做：

```js
var sum = (x,y) => x + y;
var double = x => x * 2;
var isOdd = x => x % 2 == 1;

fp.compose( [
	fp.reduce( sum )( 0 ),
	fp.map( double ),
	fp.filter( isOdd )
] )
( [1,2,3,4,5] );					// 18
```

与我们所熟知的 `_.` 命名空间前缀不同，“lodash/fp”将 `fp.` 定义为其命名空间前缀。我发现一个很有用的区别，就是 `.fp` 比 `_.` 更容易识别。

注意 `fp.compose(..)`（在 lodash proper 中又名 `_.flowRight(..)`）接受一个函数数组，而不是独立的函数作为参数。

lodash 拥有良好的稳定性、广泛的社区支持以及优秀的性能，是你探索 FP 世界时的坚实后盾。

## Mori (0.3.2)

在第 6 章中，我们已经快速浏览了一下 Immutable.js 库，该库可能是最广为人知的不可变数据结构库了。

让我们来看一下另一个流行的库：[Mori](https://github.com/swannodette/mori). Mori 设计了一套与众不同（从表面上看更像函数式编程）的 API：它使用独立的函数而不直接在值上操作。

```js
var state = mori.vector( 1, 2, 3, 4 );

var newState = mori.assoc(
	mori.into( state, Array.from( {length: 39} ) ),
	42,
	"meaning of life"
);

state === newState;						// false

mori.get( state, 2 );					// 3
mori.get( state, 42 );					// undefined

mori.get( newState, 2 );				// 3
mori.get( newState, 42 );				// "meaning of life"

mori.toJs( newState ).slice( 1, 3 );	// [2,3]
```

这是一个指出关于 Mori 的一些有趣的事情的例子：

* 使用 `vector` 而不是 `list`（正如你所假设），主要是因为文档说它的行为更像 JavaScript 中的数组。

* 不能像在操作原生 JavaScript 数组那样在任意位置设置值，在 vector 结构中，这将会抛出异常。因此我们必须使用 `mori.into(..)`，传入一个合适长度的数组来扩展 vector 的长度。在上例中，vector 有 43 个可用位置（4 + 39），所以我们可以在最后一个位置（索引为 43）上写入 `"meaning of life"` 这个值。

* 使用 `mori.into(..)` 创建一个较大的 vector，再用 `mor.assoc(..)` 根据这个 vector 创建另一个 vector 的做法听起来效率低下。但是，不可变数据结构的好处在于数据不会进行克隆，每次“改变”发生，新的数据结构只会追踪其与旧数据结构的不同之处。

Mori 受到 ClojureScript 极大的启发。如果您有 ClojureScript 编程经验，那您应该对 Mori 的 API 感到非常熟悉。由于我没有这种编程经验，因此我感觉 Mori 中的方法名有点奇怪。

但相比于在数据上直接调用方法，我真的很喜欢调用独立方法这样的设计。Mori 还有一些自动返回原生 JavaScript 数组的方法，用起来非常方便。

## 总结

JavaScript 不是作为 FP 语言来特别设计的。不过其自身的确拥有很多对 FP 非常友好基础语法（例如函数值，闭包等）。本章提及的库将使你更方便的进行函数式编程。

有了本书中函数式编程概念的武装，相信你已经准备好开始处理现实世界的代码了。找一个优秀的函数式编程库来用，然后练习、练习、再练习。

就是这样了。我已经将我目前所知道的知识分享给你了。我在此正式认证您为“JavaScript 轻量级函数式编程”程序员！好了，是时候结束我们一起学习 FP 这部分的“章节”了，但我的学习之旅还将继续。我希望，你也是！
