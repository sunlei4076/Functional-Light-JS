# JavaScript 轻量级函数式编程
# 附录 C： 函数式编程库

如果您已经从头到尾通读了此书，请停下来回顾一下从第 1 章到现在您的收货。相当漫长的一段旅程，不是吗？希望您已经收获了大量新知识，并将这些新知识应用在您自己的程序当中。

在本书即将完结时，我希望给您提供一些函数式编程库的快速指南。注意我并不会详细的展开每一个库，而是将你在结束“轻量级函数式编程”后进军真正的函数式编程时应该注意的东西快速梳理一下。

如果有可能，我建议你不要做重新造轮子这样的事情。如果你找到了一个能满足你需求的函数式编程库，那么用它就对了。只有在你实在找不到合适的库来应对你面临的问题时，才应该使用本书提供的工具库——或者自己造轮子。

## 目录

在本书第 1 章曾列出了一个函数式编程库的列表，现在我们来扩展这个列表。我们不会介绍所有的库（它们之中有许多功能重叠），但下面这些你应该有所关注：

* [Ramda](http://ramdajs.com): 通用函数式编程工具集
* [Sanctuary](https://github.com/sanctuary-js/sanctuary): Ramda 伴侣
* [lodash/fp](https://github.com/lodash/lodash/wiki/FP-Guide): 通用函数式编程工具集
* [functional.js](http://functionaljs.com/): 通用函数式编程工具集
* [Immutable](https://github.com/facebook/immutable-js): 不可变数据结构
* [Mori](https://github.com/swannodette/mori): (ClojureScript Inspired) 不可变数据结构
* [Seamless-Immutable](https://github.com/rtfeldman/seamless-immutable): 不可变数据助手
* [tranducers-js](https://github.com/cognitect-labs/transducers-js): 数据转换器
* [monet.js](https://github.com/cwmyers/monet.js): 单体类型

上面的列表只列出了所有函数式编程库的一小部分，并不是说没有在列表中列出的库就不好，也不是说列表中列出的就是最佳选择，总之这只是 JavaScript 函数式编程世界中的一瞥。您可以前往[这里](https://github.com/stoeffel/awesome-fp-js)查看更完整的函数式编程资源。

[Fantasy Land](https://github.com/fantasyland/fantasy-land)（又名 FL）是函数式编程世界中十分重要的学习资源之一，与其说它是一个库，不如说它是一本百科全书。

FL 不是一份为弱者准备的轻量级读物，而是一个完整而详细的 JavaScript 函数式编程路线图。为了尽可能提升互通性，FL 已经成为 JavaScript 函数式编程库遵循的实际标准。

FL 与“轻量级函数式编程”的概念相反，它广而深地覆盖了 JavaScript 函数式编程世界。也就是说，在结束了本书的冒险之后，FL 将会成为你接下来前进的方向。我建议您将其保存在收藏夹中，并在您使用本书的概念进行至少 6 个月的实战练习之后再回来。

## Ramda (0.23.0)

摘自 [Ramda 文档](http://ramdajs.com/):

> Ramda 的函数自动地被柯里化。
>
> Ramda 函数的参数经过优化，更便于柯里化。需要被操作的数据往往放在最后提供。

我认为合理的设计是 Ramda 的优势之一。值得注意的是，Ramda 的柯里化形式（似乎大多数的库都是这种形式）是我们在第 3 章中讨论过的“松散柯里化”。

第 3 章里面最后的例子——我们定义无值（point-free）工具函数 printIf()——可以在 Ramda 中这样实现：

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

* 使用 R.complement(..) 代替 not(..) 函数来创建 isShortEnough(..) 的含义对立函数 isLongEnough(..).

* 使用 R.flip(..) 代替 reverseArgs(..) 函数，值得一提的是，R.flip(..) 仅交换头两个参数，而 reverseArgs(..) 会将所有参数反向。在这种情景下，flip(..) 更加方便——我们不再需要使用 partialRight(..) 或其他方式进行再判断。

* R.partial(..) 所有的后续参数以单个数组的形式存在。

* 因为 Ramda 使用松散柯里化，因此我们不需要使用 R.uncurryN(..) 来反柯里化printIf(..) 以获得其参数。如果我们这样做了，就相当于 R.uncurryN(2, ..) 包裹 R.partial(..) 进行调用，完全没有必要。

Ramda 是一个受欢迎的、功能强大的库。如果你想要在你的代码中实践函数式编程，从 Ramda 开始是个不错的选择。

## Lodash/fp (4.17.4)

Lodash 是整个 JavaScript 生态系统中最受欢迎的库。Lodash 团队发布了一个“函数式编程友好”的版本—— ["lodash/fp"](https://github.com/lodash/lodash/wiki/FP-Guide).

在第 8 章中，我们讨论了合并独立列表操作（map(..), filter(..) 以及 reduce(..)）。使用 "lodash/fp" 时，你可以这样做：

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

与我们所熟知的 _. 命名空间不同，"lodash/fp" 以 fp. 作为其命名空间。我认为使用 fp. 能够很好的和 _. 进行区分，并且相比 _. 来说，fp. 对我的眼睛也“更友好”。

注意 fp.compose(..)（在 proper 中又名 _.flowRight(..)）接受一个填充函数的数组，而不是独立的函数作为参数。

lodash 拥有良好的稳定性、广泛的社区支持以及优秀的性能，是你探索函数式编程世界时坚实的后盾。

## Mori (0.3.2)

在第 6 章中，我们已经快速浏览了一下 Immutable.js 库，该库可能是最广为人知的不可变数据结构库了。

不过让我们再看一个流行的库：[Mori](https://github.com/swannodette/mori). Mori 设计了一套与众不同（从表面上看更像函数式编程）的 API：它使用独立的函数而不直接在值上操作。

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

上面的例子中，有些地方值得一提：

* 使用 vector 代替 list（正如你所假设），主要是因为文档说它的行为更像 JavaScript 中的数组。

* 不能像在操作原生 JavaScript 数组那样在任意位置设置值，在 vector 结构中，这将会抛出异常。因此我们必须使用 mori.into(..)，传入一个合适长度的数组来扩展 vector 的长度。在上例中，vector 有 43 个可用坑位（4 + 39），所以我们可以在最后一个坑位（索引为 43）上写入 "meaning of life" 这个值。

* 使用 mori.into(..) 创建一个较大的 vector，再用 mor.assoc(..) 根据这个 vector 创建另一个 vector 的做法听起来效率低下。但是，不可变数据结构的好处在于数据不会进行克隆，每次“改变”发生，新的数据结构只会追踪其与旧数据结构的不同之处。

Mori 大量受到 ClojureScript 启发。如果您有 ClojureScript 编程经验，那您应该对 Mori 的 API 感到非常熟悉。由于我没有这种编程经验，因此我感觉 Mori 中的方法名有点奇怪。

但相比于在数据上直接调用方法，我真的很喜欢调用独立方法这样的设计。Mori 还有一些自动返回原生 JavaScript 数组的方法，用起来非常方便。

## Summary

JavaScript 并不是一个设计来专门进行函数式编程的语言。但是其自身的很多特性（例如函数值，闭包等）都对函数式编程非常友好。本章提及的库将使你更方便的进行函数式编程。

有了本书中函数式编程概念的武装，相信你已经准备好开始探索现实世界的代码了。找一个优秀的函数式编程库来用，然后练习、练习、再练习。

恩……就是这样。我已经将我目前所知道的知识分享给你了。我在此正式认证您为“轻量级 JavaScript 函数式编程程序员”！好了，是时候结束我们“一起学习函数式编程”这个部分了，但我的学习之旅还将继续——我希望，你也是！
