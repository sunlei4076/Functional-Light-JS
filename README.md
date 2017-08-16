# JavaScript 轻量级函数式编程

[![沪江Web前端团队](https://cdn.rawgit.com/Hujiang-FE/icons/fff32467/logo.svg)](https://github.com/hujiang-fe)

> 参与者（排名不分先后）：[aximario](https://github.com/aximario)、[blueken](https://github.com/blueken)、[brucecham](https://github.com/brucecham)、[cfanlife](https://github.com/cfanlife)、[dail](https://github.com/dail)、[kyoko-df](https://github.com/kyoko-df)、[l3ve](https://github.com/l3ve)、[lilins](https://github.com/lilins)、[LittlePineapple](https://github.com/LittlePineapple)、[MatildaJin](https://github.com/MatildaJin)、[冬青](https://github.com/miaodongqing)、[pobusama](https://github.com/pobusama)、[Cherry](https://github.com/sunshine940326)、[萝卜](https://github.com/torrac12)、[vavd317](https://github.com/vavd317)、[vivaxy](https://github.com/vivaxy)、[萌萌](https://github.com/yanyixin)、[zhouyao](https://github.com/zhouyao)

> 关于译者：这是一个流淌着沪江血液的纯粹工程：认真，是 HTML 最坚实的梁柱；分享，是 CSS  里最闪耀的一瞥；总结，是 JavaScript 中最严谨的逻辑。经过捶打磨练，成就了本书的中文版。本书包含了函数式编程之精髓，希望可以帮助大家在函数式编程的学习道路上走的更顺畅。比心。

本书主要探索函数式编程<sup>[\[1\]](#note1)</sup>(FP)的核心思想。在此过程中，作者不会执着于使用大量复杂的概念来进行诠释，这也是本书的特别之处。我们在 JavaScript 中应用的仅仅是一套基本的函数式编程概念的子集。我称之为“轻量级函数式编程(FLP)”。

**注释：** 题目中使用了“轻量”二字，然而这并不是一本“轻松的”“入门级”书籍。本书是严谨的，充斥着各种复杂的细节，适合拥有扎实 JS 知识基础的阅读者进行研读。“轻量”意味着范围缩小。通常来说，关于函数式编程的 JavaScript 书籍都热衷于拓展阅读者的知识面，并企图覆盖更多的知识点。而本书则对于每一个话题都进行了深入的探究，尽管这种探究是小范围进行的。

让我们面对这个事实：除非你已经是函数式编程高手中的一员（至少我不是！），否则类似“一个单子仅仅是自函子中的幺半群”这类说法对我们来说毫无意义。

这并不是说，各种复杂繁琐的概念是**无意义**的，更不是说，函数式编程者滥用了它们。一旦你完全掌握了轻量的函数式编程内容，你将会／但愿会想要对函数式编程的各种概念进行更正式更系统的学习，并且你一定会对它们的意义和原因有更深入的理解。

但是我更想要让你能够**现在**就把一些函数式编程的基础运用到 JavaScript 编程过程中去，因为我相信这会帮助你写出更优秀的，更**符合逻辑**的代码。

**更多关于本书背后的动机和各种观点讨论，请参看[前言](preface.md)。**

## 本书

[目录](toc.md)

* [引言](foreword.md) (by [Brian Lonsdorf aka "Prof Frisby"](https://twitter.com/DrBoolean))
* [前言](preface.md)
* [第一章：为什么使用函数式编程？](ch1.md)
* [第二章：函数基础](ch2.md)
* [第三章：管理函数的输入](ch3.md)
* [第四章：组合函数](ch4.md)
* [第五章：减少副作用](ch5.md)
* [第六章：值的不可变性](ch6.md)
* [第七章：闭包 vs 对象](ch7.md)
* [第八章：列表操作](ch8.md)
* [第九章：递归](ch9.md)
* [第十章：异步的函数式](ch10.md)
* [第十一章：融会贯通](ch11.md)
* [附录 A ：Transducing](apA.md)
* [附录 B ：谦虚的 Monad](apB.md)
* [附录 C ：函数式编程函数库](apC.md)

## 关于出版

本书主要在 [on Leanpub](https://leanpub.com/fljs/) 平台上以电子版本的形式进行出版。我也尝试出售本书的纸质版本，但没有确定的方案。

除了购买本书以外，如果你想要对本书作一些物质上的捐赠，请在 [patreon](https://www.patreon.com/getify) 上进行操作。本书作者感谢你的慷慨解囊。

<a href="https://www.patreon.com/getify">[![patreon.png](https://s11.postimg.org/axpzguh77/patreon.png)](https://www.patreon.com/getify)</a>

## 真人教学课程

本书内容大多源自于我教授的一个同名课程（以公司举办的公开或内部研讨会这样的形式进行）。

[http://getify.me](http://getify.me)

如果你喜欢本书的内容，并希望组织此类课程，或者组织关于其他 JS／HTML5／Node.js 课程，请通过以下方式联系我：
[http://getify.me](http://getify.me)

## 在线视频课程

我还提供一些可以在线点播的 JS 培训课程。我在 [Frontend Masters](https://FrontendMasters.com) 上开办课程，例如我的 [Functional-Lite JS](https://frontendmasters.com/courses/functional-js-lite/) 研讨会。还有一些课程发布在 [PluralSight](https://www.pluralsight.com/search?q=kyle%20simpson&categories=all) 上。

## Contributions

## 关于内容贡献

**非常欢迎**对于本书的任何内容贡献。但是在提交 PR 之前**请务必**认真阅读 [Contributions Guidelines](CONTRIBUTING.md)。

## License & Copyright

## 版权

本书所有的材料和内容都归属 (c) 2016-2017 Kyle Simpson 所有。

<a rel="license" href="http://creativecommons.org/licenses/by-nc-nd/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-nd/4.0/88x31.png" /></a><br />本书根据<a rel="license" href="http://creativecommons.org/licenses/by-nc-nd/4.0/">Creative Commons Attribution-NonCommercial-NoDerivs 4.0 Unported License</a> 进行授权许可.

1. <a name="note1"></a > FP，本书统称为函数式编程。

