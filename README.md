# Functional-Light JavaScript (book)
# 轻量函数式编程

This book explores the core principles of functional programming (FP) as they are applied to JavaScript. But what makes this book different is that we approach these principles without drowning in all the heavy terminology. We look at a subset of FP foundational concepts that I call "Functional-Light Programming" (FLP) and apply it to JavaScript.

本书主要探索函数式编程(FP)的核心思想。在此过程中，作者不会执着于使用大量复杂的概念来进行诠释，这也是本书的特别之处。我们在 JavaScript 中应用的仅仅是一套基本的函数式编程概念的子集。我称之为“轻量函数式编程(FLP)”。

**Note:** Despite the word "Light" in the title, I do not consider or recommend this book as a "beginner", "easy", or "intro" book on the topic. This book is rigorous and full of gritty detail; it expects a solid foundation of JS knowledge before diving in. "Light" means limited in scope; instead of being more broad, this book goes much deeper into each topic than you typically find in other FP-JavaScript books.

**注释：** 题目中使用了“轻量”二字，然而这并不是一本“轻松的”“入门级”书籍。本书是严谨的，充斥着各种复杂的细节，适合拥有扎实 JS 知识基础的阅读者进行研读。“轻量”意味着范围的缩小。通常来说，关于函数式编程的 JavaScript 书籍都热衷于拓展阅读者的知识面，并企图覆盖更多的知识点。而本书则对于每一个话题都进行了深入的探究，尽管这种探究是小范围进行的。

Let's face it: unless you're already a member of the FP cool kids club (I'm not!), a statement like, "a monad is just a monoid in the category of endofunctors", just doesn't mean anything useful to us.

让我们面对这个事实：除非你已经是函数式编程高手中的一员（至少我不是！），否则类似“一个单元仅仅是自函子中的独异点”这类说法对我们来说毫无意义。

That's not to say the terms are meaning*less* or that FPrs are bad for using them. Once you graduate from Functional-Light, you'll maybe/hopefully want to study FP more formally, and you'll certainly have plenty of exposure to what they mean and why.

这并不是说，各种复杂繁琐的概念是*无意义*的，更不是说，函数式编程者滥用了它们。一旦你完全掌握了轻量的函数式编程内容，你将会／但愿会想要对函数式编程的各种概念进行更正式更系统的学习，并且你一定会对它们的意义和原因有更深入的理解。

But I want you to be able to apply some of the fundamentals of FP to your JavaScript *now*, because I believe it will help you write better, more *reason*able code.

但是我更想要让你能够*现在*就把一些函数式编程的基础运用到 JavaScript 编程过程中去，因为我相信这会帮助你写出更优秀的，更*符合逻辑*的代码。

**To read more about the motivations and perspective behind this book, check out the [Preface](preface.md).**

**更多关于本书背后的动机和各种观点讨论，请参看[前言](preface.md)。**

## Book
## 本书

[目录](toc.md)

* [引言](foreword.md) (by [Brian Lonsdorf aka "Prof Frisby"](https://twitter.com/DrBoolean))
* [前言](preface.md)
* [第一章：为何采用函数式编程？](ch1.md)
* [第二章：功能函数的基础](ch2.md)
* [第三章：管理函数输入](ch3.md)
* [第四章：创建函数](ch4.md)
* [第五章：减轻副作用](ch5.md)
* [第六章：值的不变性](ch6.md)
* [第七章：闭包 VS 对象](ch7.md)
* [第八章：列表操作](ch8.md)
* [第九章：递归](ch9.md)
* [第十章：异步功能](ch10.md)
* [第十一章：融会贯通](ch11.md)
* [附录 A ：转换](apA.md)
* [附录 B ：基础单元](apB.md)
* [附录 C ：函数式编程代码库](apC.md)

## Publishing

## 关于出版

I'm self-publishing this book, most likely digitally [on Leanpub](https://leanpub.com/fljs/). I'll also be trying to work out an option to sell print book copies, but that part is still uncertain.

本书主要在 [on Leanpub](https://leanpub.com/fljs/) 平台上以电子版本的形式进行出版。我也尝试出售本书的纸质版本，然而仍然没有确实的方案。

If you'd like to contribute financially towards the effort (or any of my other OSS work) aside from purchasing the books, I do have a [patreon](https://www.patreon.com/getify) that I would always appreciate your generosity towards.

如果除了购买本书以外，你想要对本书作一些物质上的捐赠，请在 [patreon](https://www.patreon.com/getify) 上进行操作。本书作者感谢你的慷慨解囊。

<a href="https://www.patreon.com/getify">[![patreon.png](https://s11.postimg.org/axpzguh77/patreon.png)](https://www.patreon.com/getify)</a>

## In-person Training

## 真人教学课程

The content for this book derives heavily from a training workshop I teach professionally (in both public and private-corporate workshop format) of the same name.

本书内容大多源自于我教授的一个同名课程（以公司举办的公开或内部研讨会这样的形式进行）。

If you like this content and would like to contact me regarding conducting training on this, or other various JS/HTML5/Node.js topics, please reach out to me through any of these channels listed here:

[http://getify.me](http://getify.me)

如果你喜欢本书的内容，并希望组织此类课程，或者组织关于其他 JS／HTML5/Node.js 课程，请通过以下方式联系我：
[http://getify.me](http://getify.me)

## Online Video Training

## 在线视频课程

I also have several JS training courses available in on-demand video format. I teach courses through [Frontend Masters](https://FrontendMasters.com), like my [Functional-Lite JS](https://frontendmasters.com/courses/functional-js-lite/) workshop. Some of those courses are also available on [PluralSight](https://www.pluralsight.com/search?q=kyle%20simpson&categories=all).

我还提供一些可以在线点播的 JS 培训课程。

## Contributions

## 关于贡献内容

Any contributions you make to this effort **are of course greatly appreciated**.

But **PLEASE** read the [Contributions Guidelines](CONTRIBUTING.md) carefully before submitting a PR.

**非常欢迎**对于本书的任何内容贡献。但是在提交 PR 之前*请务必*认真阅读 [Contributions Guidelines](CONTRIBUTING.md)。

## License & Copyright

## 版权

The materials herein are all (c) 2016-2017 Kyle Simpson.

本书所有的材料和内容都归属 (c) 2016-2017 Kyle Simpson 所有。

<a rel="license" href="http://creativecommons.org/licenses/by-nc-nd/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-nd/4.0/88x31.png" /></a><br />本书根据<a rel="license" href="http://creativecommons.org/licenses/by-nc-nd/4.0/">Creative Commons Attribution-NonCommercial-NoDerivs 4.0 Unported License</a> 进行授权许可.
