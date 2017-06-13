# 轻量级函数式编程
# 第一章：为什么使用函数式编程？

> Functional programmer: (noun) One who names variables "x", names functions "f", and names code patterns "zygohistomorphic prepromorphism"
>
> James Iry ‏@jamesiry 5/13/15
>
> https://twitter.com/jamesiry/status/598547781515485184

函数式编程（以下简称FP）不是一个新的概念，这个概念几乎贯穿了整个编程史。我不确定这么说是否合理，但是很确定的一点是：直到最近几年，FD才成为整个开发界的主流观念。所以我觉得FD更像学者的领域。

然而一切都在变。无论是语言，还是库和框架，都对FP的兴趣空前高涨。你也很可能在读相关内容，因为你终于意识到FP是不容忽视的东西。或者你跟我一样，已经很多次尝试着去学FP，但却很难理解所有的术语或数学符号。

无论你出于何目的翻阅本书，欢迎加入我们！

## 置信度

我有一个非常简单的前提，这是我作为软件开发老师（JavaScript）中所做的一切的基础：你不能信任的代码是你不明白的代码。此外，对你不信任或不明白的代码，那么你就会对这样的代码不自信，并且会认为它不适合开发你的业务。当代码运行的时候也只能祈求好运。

信任是什么意思？信任代表你通过读代码而不只是运行，你能证实你理解这段代码能干什么事，而不只是停留在它可能是干什么。也许我们不应该总倾向于通过运行测试程序，从而来验证我们的程序的正确性。我并不是说测试不好，而是说我们应该期待去能够很好地了解代码，这样我们在运行测试代码之前就会知道它肯定能跑通。

通过读代码能对我们的程序有更多信心，我相信构成FP基础的技术，是本着心态设计的。 理解FP并在程序中用心实践的人，通过FP已经被证实的原则，能够写出他们可读性高和可验证的代码，来达到他们想要的目的。

我希望您能通过理解轻量级函数式编程的原则，对您编写的代码更有信心，并且能在之后的路上越走越好。

## 交流渠道

函数式编程为何重要？为了回答这个问题，我们先退一万步，来讨论一下编程本身的重要性。

我认为代码不是构成电脑的说明，这么说你可能感到很奇怪。事实上，代码能指示电脑如何运行就是一个意外中的惊喜。

作为与他人交流的一种手段，我深信代码的作用非常重要。

根据以往经验你可能知道，有时候花很多时间写代码其实是为了搞明白现有的代码。我们的大部分时间其实都是在维护别人的代码（或自己的老代码），只有少部分时间是在敲新代码。

你知道研究过这个话题的研究人员说给出了怎样的数据吗？70％ 的时间我们花费维护代码只是花在阅读并理解代码。 也难怪全球程序员的平均代码行数是5行。 我们一天花了7个半小时才读懂代码，然后找出哪5行代码奏效。

我想我们应该更多的关注一下代码的可读性。可能的话，不妨多花点时间在可读性上。顺便提一句，可读性并不只是说代码量少，对代码的熟悉程度会影响代码的可读性（这一点也是被证实过的）。

因此，如果我们要花费更多的时间来关注编写代码的可读性和可理解性，所以FP就是这样一个非常方便的模式。FP的原则是完善的，经过了深入的研究和审查，并且可被证明和验证。

如果我们使用FP原则，我相信我们将创建更容易理解的代码。一旦我们知道这些原则，它们将在代码中被识别和熟悉，这意味着当我们读取一段代码时，我们将花费更少的时间来进行定位。我们的重点将在于如何组建所有已知的乐高片段，而不是这些乐高片段意味着什么。

FP是编写可读代码的最有效工具之一(可能还有其他)。这就是为什么FP如此重要。

### 可读性曲线

很重要的是，我花了一年时间来讲述一种多年来让我感到困惑和沮丧的现象，在写这本书时特别尖锐。

这也可能是许多开发人员倾向于碰到的东西。亲爱的读者，当你读这篇文章的时候，你可能会发现自己也在这条船上。但是振作起来，如果坚持下去，陡峭的学习曲线总会过去。

<p align="center">
	<img src="fig17.png" width="600">
</p>

我们将在下一章更深入的讨论这个问题。但是一些命令语句是已经用过的，像 if 语句和 for 循环这样的语句。这些语句旨在精确地指导计算机“如何”完成一件事情。声明式代码，以及我们将努力学习遵循FP原则编写的代码，是更专注于描述产出“什么”代码。

还有个残酷的问题摆在眼前，这个问题花费了我在这本书上的很多时间：我需要花费更多的精力和更多的代码来提高代码的可读性，尽量减少或消除你可能编写错误的大部分地方。

如果你希望用FP重构过的代码能够立刻变得更美观、优雅、智能和简洁的话，这个有点不太现实，这个变化是需要一个过程的。

FP的构思是很奇特的，代码应该如何基于它进行构造，使得数据流更加明显，并能让读者很快理解你的思想。这种努力是非常值得的，然而过程很艰辛，你可能需要花很多时间基于FP来调整代码直到代码可读性变得好一些。

另外，我的经验是，将命令式代码转换为更偏向于声明式的FP之前，大约需要做六次尝试。对我来说，编写FP的代码更像是一个过程，而不是从一个范例到另一个范例的二进制转换。

我也会经常对写过的代码进行再读。就是说，写完一段代码，过几个小时或一天再看就会有不一样的感觉。通常，重写的代码是比较混乱不堪，所以需要反复调整。

函数式编程并没有让我在艺术家的画布上实现一个优雅的笔触，让观众敬畏。相反，它是一个艰苦的，详细的，有时是阴险的通过一个被忽视的领域杂草的破解。 函数式编程的过程并没有让我在艺术的画布上笔下生辉，让观众拍岸叫好。相反，编程的过程很艰辛，且历历在目，感觉像坐在一辆不知去向的马车穿过一片荒芜的灌木丛林。

我不是试图打压你的激情，我真的希望你也可以在这个道路上披荆斩棘。过后我终于看到可读性曲线向上延伸，所有付出都是值得的，我相信你也将会有这样的感受。

## Take

We're going to approach FP from the ground up, and uncover the basic foundational principles that I believe formal FPers would admit are the scaffolding for everything they do. But for the most part we'll stay arms length away from most of the intimidating terminology or mathematical notation that can so easily frustrate learners.

I believe it's less important what you call something and more important that you understand what it is and how it works. That's not to say there's no importance to shared terminology -- it undoubtedly eases communication among seasoned professionals. But for the learner, I've found it can be somewhat distracting.

So I hope this book can focus more on the base concepts and less on the fancy terminology. That's not to say there won't be terminology; there definitely will be. But don't get too wrapped up in the fancier words. Look beyond them to the ideas. That's what this book is trying to be about.

I call the less formal practice herein "Functional-Light Programming" because I think where the formalism of true FP suffers is that it can be quite overwhelming if you're not already accustomed to formal thought. I'm not just guessing; this is my own personal story. Even after teaching FP and writing this book, I can still say that the formalism of terms and notation in FP is very, very diffcult for me to process. I've tried, and tried, and I can't seem to get through much of it.

I know many FPers who believe that the formalism itself helps learning. But I think there's clearly a cliff where that only becomes true once you reach a certain comfort with the formalism. If you happen to already have a math background or even some flavors of CS experience, this may come more naturally to you. But some of us don't, and no matter how hard we try, the formalism keeps getting in the way.

So this book introduces the concepts that I believe FP is built on, but comes at it by giving you a boost to climb up the cliff wall rather than throwing you straight at it to figure out how to climb as you go.

## YAGNI

If you've been around programming for very long, chances are you've heard the phrase "YAGNI" before: "You Ain't Gonna Need It". This principle primarily comes from extreme programming, and stresses the high risk and cost of building a feature before it's needed.

Sometimes we guess we'll need a feature in the future, build it now believing it'll be easier to do as we build other stuff, then realize we guessed wrong and the feature wasn't needed, or needed to be quite different. Other times we guess right, but build a feature too early, and suck up time from the features that are genuinely needed now; we incur an opportunity cost in diluting our energy.

YAGNI challenges us to remember: even if it's counter intuitive in a situation, we often should postpone building something until it's presently needed. We tend to exaggerate our mental estimates of the future refactoring cost of adding it later when it is needed. Odds are, it won't be as hard to do later as we might assume.

As it applies to functional programming, I would give this admonition: there will be plenty of interesting and compelling patterns discussed in this text, but just because you find some pattern exciting to apply, it may not necessarily be appropriate to do so in a given part of your code.

This is where I will differ from many more formal FPers: just because you *can* FP something doesn't mean you *should* FP it. Moreover, there are many ways to slice a problem, and even though you may have learned a more sophisticated approach that is more "future-proof" to maintenance and extensibility, a simpler FP pattern might be more than sufficient in that spot.

Generally, I'd recommend to seek balance in what you code, and to be conservative in your application of FP concepts as you get the hang of things. Default to the YAGNI principle in deciding if a certain pattern or abstraction will help that part of the code be more readable or if it's just introducing more clever sophistication that isn't (yet) warranted.

> Reminder, any extensibility point that’s never used isn’t just wasted effort, it’s likely to also get in your way as well
>
> Jeremy D. Miller @jeremydmiller 2/20/15
>
> https://twitter.com/jeremydmiller/status/568797862441586688

Remember, every single line of code you write has a reader cost associated with it. That reader may be another team member, or even your future self. Neither of those readers will be impressed with overly clever unnecessary sophistication to show off your FP agility.

The best code is the code that is most readable in the future because it strikes exactly the right balance between what it can/should be (idealism) and what it must be (pragmatism).

## Resources

I have drawn on a great many different resources to be able to compose this text. I believe you too may benefit from them, so I wanted to take a moment to point them out.

### Books

Some FP/JavaScript books that you should definitely read:

* [Professor Frisby's Mostly Adequate Guide to Functional Programming](https://drboolean.gitbooks.io/mostly-adequate-guide/content/ch1.html) by [Brian Lonsdorf](https://twitter.com/drboolean)
* [JavaScript Allongé](https://leanpub.com/javascript-allonge) by [Reg Braithwaite](https://twitter.com/raganwald)
* [Functional JavaScript](http://shop.oreilly.com/product/0636920028857.do) by [Michael Fogus](https://twitter.com/fogus)

### Blogs/Sites

Some other authors and content you should check out:

* [Fun Fun Function Videos](https://www.youtube.com/watch?v=BMUiFMZr7vk) by [Mattias P Johansson](https://twitter.com/mpjme)
* [Awesome FP JS](https://github.com/stoeffel/awesome-fp-js)
* [Kris Jenkins](http://blog.jenkster.com/2015/12/what-is-functional-programming.html)
* [Eric Elliott](https://medium.com/@_ericelliott)
* [James A Forbes](https://james-forbes.com/)
* [James Longster](https://github.com/jlongster)
* [André Staltz](http://staltz.com/)
* [Functional Programming Jargon](https://github.com/hemanth/functional-programming-jargon#functional-programming-jargon)
* [Functional Programming Exercises](https://github.com/InceptionCode/Functional-Programming-Exercises)

### Libraries

The code snippets in this book do not use libraries. Each operation that we discover, we'll derive how to implement it in standalone, plain ol' JavaScript. However, as you begin to build more of your real code with FP, you'll quickly want a library to provide optimized and highly reliable versions of these commonly accepted utilities.

By the way, you'll want to make sure you check the documentation for the library functions you use to make sure you know how they work. There will be a lot of similarities in many of them to the code we build on in this text, but there will undoubtedly be some differences, even between popular libraries.

Here are a few popular FP libraries for JavaScript that are a great place to start your exploration with:

* [Ramda](http://ramdajs.com)
* [lodash/fp](https://github.com/lodash/lodash/wiki/FP-Guide)
* [functional.js](http://functionaljs.com/)
* [Immutable.js](https://github.com/facebook/immutable-js)

Appendix C illustrates some of these libraries using various examples from the text.

## Summary

This is Functional-Light JavaScript. The goal is to learn to communicate with our code but not suffocate under mountains of notation or terminology to get there. I hope this book jumpstarts your journey!
