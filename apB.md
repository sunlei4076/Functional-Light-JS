# JavaScript 轻量级函数式编程
# 附录 B: 谦虚的 Monad

首先，我坦白：在开始写以下内容之前我并不太了解 Monad 是什么。我为了确认一些事情而犯了很多错误。如果你不相信我，去看看 [这本书 Git 仓库](https://github.com/getify/Functional-Light-JS) 中关于本章的提交历史吧！

我囊括了本书中所有涉及 Monad 的话题。就像我写书的过程一样，每个开发者在学习 FP 的旅程中都会经历这个部分。

尽管其他 FP 著作差不多都把 Monad 作为开始，而我们却只对它做了简要说明，并基本以此结束本书。在轻量级函数式编程中我确实没有遇到太多需要仔细考虑 Monad 的问题，这就是本文比文章主体更有价值的原因。但是并不是说 Monad 是没用的或者是不普遍的 —— 恰恰相反，它很有用，也很流行。

JavaScript FP 界有一个小笑话，几乎每个人都不得不在他们的文章或者博客里写 Monad 是什么，把它拎出来写就像是一个仪式。在过去的几年里，人们把 Monad 描述为卷饼、洋葱和各种各样古怪的抽象概念。我肯定不会重蹈覆辙！

> 一个 Monad 仅仅是自函子 (endofunctor) 范畴中的一个 monoid

我们引用这句话来开场，所以把话题转到这个引言上面似乎是很合适的。可是才不会这样，我们不会讨论 Monad 、endofunctor 或者范畴论。这句引言不仅故弄玄虚而且华而不实。

我只希望通过我们的讨论，你不再害怕 Monad 这个术语或者这个概念了 —— 我曾经怕了很长一段时间 —— 并在看到该术语时知道它是什么。你可能，也只是可能，会正确地使用到它们。

## 类型

在 FP 中有一个巨大的兴趣领域：类型论，本书基本上完全远离了该领域。我不会深入到类型论，坦白的说，我没有深入的能力，即使干了也吃力不讨好。

但是我要说，Monad 基本上是一个值类型。

数字 `42` 有一个值类型（number），它带有我们依赖的特征和功能。字符串 `"42"` 可能看起来很像，但是在编程里它有不同的用途。

在面向对象编程中，当你有一组数据（甚至是一个单独的离散值），并且想要给它绑上一些行为，那么你将创建一个对象或者类来表示 "type"。接着实例就成了该类型的一员。这种做法通常被称为 “数据结构”。

我将会非常宽泛的使用数据结构这个概念，而且我断定，当我们在编程中为一个特定的值定义一组行为以及约束条件，并且将这些特征与值一起绑定在一个单一抽象概念上时，我们可能会觉得很有用。这样，当我们在编程中使用一个或多个这种值的时候，它们的行为会自然的出现，并且会使它们更方便的工作。方便的是，对你的代码的读者来说，是更有描述性和声明性的。

Monad 是一种数据结构。是一种类型。它是一组使处理某个值变得可预测的特定行为。

回顾第 8 章，我们谈到了函子（functor）：包括一个值和一个用来对构成函子的数据执行操作的类 map 实用函数。Monad 是一个包含一些额外行为的函子（functor）。

## 松散接口

实际上，Monad 并不是单一的数据类型，它更像是相关联的数据类型集合。它是一种根据不同值的需要而用不同方式实现的接口。每种实现都是一种不同类型的 Monad。

例如，你可能阅读 "Identity Monad"、"IO Monad"、"Maybe Monad"、"Either Monad" 或其他形形色色的字眼。他们中的每一个都有基本的 Monad 行为定义，但是它根据每个不同类型的 Monad 用例来继承或者重写交互行为。

可是它不仅仅是一个接口，因为它不只是使对象成为 Monad 的某些 API 方法的实现。对这些方法的交互的保障是必须的，是 monadic 的。这些众所周知的常量对于使用 Monad 提高可读性是至关重要的；另外，它是一个特殊的数据结构，读者必须全部阅读才能明白。

事实上，这些 Monad 方法的名字和真实接口授权的方式甚至没有一个统一的标准；Monad 更像是一个 loose interface。有些人称这些方法为 `bind(..)`,有些称它为 `chain(..)`，还有些称它为 `flatMap(..)`，等等。

所以，Monad 是一个对象数据结构，并且有充足的方法（几乎任何名称或排序），至少满足了 Monad 定义的主要行为需求。每一种 Monad 都基于最少数量的方法来进行不同的扩展。但是，因为它们在行为上都有重叠，所以一起使用两种不同的 Monad 仍然是直截了当和可控的。

从某种意义上说，Monad 更像是接口。

## Maybe

在 FP 中，像 Maybe 这样涵盖 Monad 是很普遍的。事实上，Maybe Monad 是另外两个更简单的 Monad 的搭配：Just 和 Nothing。

既然 Monad 是一个类型，你可能认为我们应该定义 `Maybe` 作为一个要被实例化的类。这虽然是一种有效的方法，但是它引入了 `this` 绑定的问题，所以在这里我不想讨论；相反，我打算使用一个简单的函数和对象的实现方式。

以下是 Maybe 的最简单的实现：

```js
var Maybe = { Just, Nothing, of/* aka: unit, pure */: Just };

function Just(val) {
	return { map, chain, ap, inspect };

	// *********************

	function map(fn) { return Just( fn( val ) ); }
	// 又叫：bind, flatMap
	function chain(fn) { return fn( val ); }
	function ap(anotherMonad) { return anotherMonad.map( val ); }

	function inspect() {
		return `Just(${ val })`;
	}
}

function Nothing() {
	return { map: Nothing, chain: Nothing, ap: Nothing, inspect };

	// *********************

	function inspect() {
		return "Nothing";
	}
}
```

**注意：** `inspect(..)` 方法只用于我们的示例中。从 Monad 的角度来说，它并没有任何意义。

如果现在大部分都没有意义的话，不要担心。我们将会更专注的说明我们可以用它做什么，而不是过多的深入 Monad 背后的设计细节和理论。

所有的 Monad 一样，任何含有 `Just(..)` 和 `Nothing()` 的 Monad 实例都有 `map(..)`、`chain(..)`（也叫 `bind(..)` 或者 `flatMap(..)`）和 `ap(..)` 方法。这些方法及其行为的目的在于提供多个 Monad 实例一起工作的标准化方法。你将会注意到，无论 `Just(..)` 实例拿到的是怎样的一个 `val` 值， `Just(..)` 实例都不会去改变它。所有的方法都会创建一个新的 Monad 实例而不是改变它。

Maybe 是这两个 Monad 的结合。如果一个值是非空的，它是 `Just(..)` 的实例；如果该值是空的，它则是 `Nothing()` 的实例。注意，这里由你的代码来决定 "空" 的意思，我们不做强制限制。下一节会详细介绍这一点。

但是 Monad 的价值在于不论我们有 `Just(..)` 实例还是 `Nothing()` 实例，我们使用的方法都是一样的。`Nothing()` 实例对所有的方法都有空操作定义。所以如果 Monad 实例出现在 Monad 操作中，它就会对 Monad 操作起短路作用。

Maybe 这个抽象概念的作用是隐式地封装了操作和无操作的二元性。

### 与众不同的 Maybe

JavaScript Maybe Monad 的许多实现都包含 `null` 和 `undefined` 的检查（通常在 `map(..)`中），如果是空的话，就跳过该 Monad 的特性行为。事实上，Maybe 被声称是有价值的，因为它自动地封装了空值检查得以在某种程度上短路了它的特性行为。

这是 Maybe 的典型说明：

```js
// 代替不稳定的 `console.log( someObj.something.else.entirely )`：

Maybe.of( someObj )
.map( prop( "something" ) )
.map( prop( "else" ) )
.map( prop( "entirely" ) )
.map( console.log );
```

换句话说，如果我们在链式操作中的任何一环得到一个 `null` 或者 `undefined` 值，Maybe 会智能的切换到空操作模式 —— 它现在是一个 `Nothing()` Monad 实例！ —— 把剩余的链式操作都停止掉。如果一些属性丢失或者是空的话，嵌套的属性访问能安全的抛出 JS 异常。这是非常酷的而且很实用。

但是，我们这样实现的 Maybe 不是一个纯 Monad。

Monad 的核心思想是，它必须对所有的值都是有效的，不能对值做任何检查 —— 甚至是空值检查。所以为了方便，这些其他的实现都是走的捷径。这是无关紧要的。但是当学习一些东西的时候，你应该先学习它的最纯粹的形式，然后再学习更复杂的规则。

我早期提供的 Maybe Monad 的实现不同于其他的 Maybe，就是它没有空置检查。另外，我们将 `Maybe` 作为 `Just(..)` 和 `Nothing()` 的非严格意义上的结合。

等一下，如果我们没有自动短路，那 Maybe 是怎么起作用的呢？！？这似乎就是它的全部意义。

不要担心，我们可以从外部提供简单的空值检查，Maybe Monad 其他的短路行为也还是可以很好的工作的。你可以在之前做一些 `someObj.something.else.entirely` 属性嵌套，但是我们可以做的更 “正确”：

```js
function isEmpty(val) {
	return val === null || val === undefined;
}

var safeProp = curry( function safeProp(prop,obj){
	if (isEmpty( obj[prop] )) return Maybe.Nothing();
	return Maybe.of( obj[prop] );
} );

Maybe.of( someObj )
.chain( safeProp( "something" ) )
.chain( safeProp( "else" ) )
.chain( safeProp( "entirely" ) )
.map( console.log );
```

我们设计了一个用于空值检查的 `safeProp(..)` 函数，并选择了 `Nothing()` Monad 实例。或者把值包装在 `Just(..)` 实例中（通过 `Maybe.of(..)`）。然后我们用 `chain(..)` 替代 `map(..)`，它知道如何 “展开” `safeProp(..)` 返回的 Monad。

当遇到空值的时候，我们得到了一连串相同的短路。只是我们把这个逻辑从 Maybe 中排除了。

不管返回哪种类型的 Monad，我们的 `map(..)` 和 `chain(..)` 方法都有不变且可预测的反馈，这就是 Monad，尤其是 Maybe Monad 的好处。这难道不酷吗？

## Humble

现在我们对 Maybe 和它的作用有了更多的了解，我将会在它上面加一些小的改动 —— 我将通过设计 Maybe + Humble Monad 来添加一些转折并且加一些诙谐的元素。从技术上来说，`Humble(..)` 并不是一个 Monad，而是一个产生 Maybe Monad 实例的工厂函数。

Humble 是一个使用 Maybe 来跟踪 `egoLevel` 数字状态的数据结构包装器。具体来说，`Humble(..)` 只有在他们自身的水平值足够低（少于 `42`）到被认为是 Humble 的时候才会执行生成的 Monad 实例；否则，它就是一个 `Nothing()` 空操作。这听起来真的和 Maybe 很像！

这是一个 Maybe + Humble Monad 工厂函数：

```js
function Humble(egoLevel) {
	// 接收任何大于等于 42 的数字
	return !(Number( egoLevel ) >= 42) ?
		Maybe.of( egoLevel ) :
		Maybe.Nothing();
}
```

你可能会注意到，这个工厂函数有点像 `safeProp(..)`，因为，它使用一个条件来决定是选择 Maybe 的 `Just(..)` 还是 `Nothing()`。

让我们来看一个基础用法的例子：

```js
var bob = Humble( 45 );
var alice = Humble( 39 );

bob.inspect();							// Nothing
alice.inspect();						// Just(39)
```

如果 Alice 赢得了一个大奖，现在是不是在为自己感到自豪呢？

```js
function winAward(ego) {
	return Humble( ego + 3 );
}

alice = alice.chain( winAward );
alice.inspect();						// Nothing
```

`Humble( 39 + 3 )` 创建了一个 `chain(..)` 返回的 `Nothing()` Monad 实例，所以现在 Alice 不再有 Humble 的资格了。

现在，我们来用一些 Monad ：

```js
var bob = Humble( 41 );
var alice = Humble( 39 );

var teamMembers = curry( function teamMembers(ego1,ego2){
	console.log( `Our humble team's egos: ${ego1} ${ego2}` );
} );

bob.map( teamMembers ).ap( alice );
// Humble 队列：41 39
```

由于 `teamMembers(..)` 是柯里化的，`bob.map(..)` 的调用传入了 `bob` 自身的级别（`41`），并且创建了一个被其余的方法包装的 Monad 实例。在 **这个** Monad 中调用的 `ap(alice)` 调用了 `alice.map(..)`，并且传递给来自 Monad 的函数。这样做的效果是，Monad 的值已经提供给了 `teamMembers(..)` 函数，并且把显示的结果给打印了出来。

然而，如果一个 Monad 或者两个 Monad 实际上是 `Nothing()` 实例（因为它们本身的水平值太高了）：

```js
var frank = Humble( 45 );

bob.map( teamMembers ).ap( frank );

frank.map( teamMembers ).ap( bob );
```

`teamMembers(..)` 永远不会被调用（也没有信息被打印出来），因为，`frank` 是一个 `Nothing()` 实例。这就是 Maybe monad 的作用，我们的 `Humble(..)` 工厂函数允许我们根据自身的水平来选择。赞！

### Humility

再来一个例子来说明 Maybe + Humble 数据结构的行为：

```js
function introduction() {
	console.log( "I'm just a learner like you! :)" );
}

var egoChange = curry( function egoChange(amount,concept,egoLevel) {
	console.log( `${amount > 0 ? "Learned" : "Shared"} ${concept}.` );
	return Humble( egoLevel + amount );
} );

var learn = egoChange( 3 );

var learner = Humble( 35 );

learner
.chain( learn( "closures" ) )
.chain( learn( "side effects" ) )
.chain( learn( "recursion" ) )
.chain( learn( "map/reduce" ) )
.map( introduction );
// 学习闭包
// 学习副作用
// 歇息递归
```

不幸的是，学习过程看起来已经缩短了。我发现学习一大堆东西而不和别人分享，会使自我太膨胀，这对你的技术是不利的。

让我们尝试一个更好的方法：

```js
var share = egoChange( -2 );

learner
.chain( learn( "closures" ) )
.chain( share( "closures" ) )
.chain( learn( "side effects" ) )
.chain( share( "side effects" ) )
.chain( learn( "recursion" ) )
.chain( share( "recursion" ) )
.chain( learn( "map/reduce" ) )
.chain( share( "map/reduce" ) )
.map( introduction );
// 学习闭包
// 分享闭包
// 学习副作用
// 分享副作用
// 学习递归
// 分享递归
// 学习 map/reduce
// 分享 map/reduce
// 我只是一个像你一样的学习者 :)
```

在学习中分享。是学习更多并且能够学的更好的最佳方法。

## 摘要

说了这么多，那什么是 Monad ？

Monad 是一个值类型，一个接口，一个有封装行为的对象数据结构。

但是这些定义中没有一个是有用的。这里尝试做一个更好的解释：Monad 是一个用更具有声明式的方式围绕一个值来组织行为的方法。

和这本书中的其他部分一样，他们在有用的地方使用 Monad，但是不要因为每个人都在 FP 中讨论他们而使用他们。Monad 不是万金油，但是他们确实提供了一些有用的实用函数。
