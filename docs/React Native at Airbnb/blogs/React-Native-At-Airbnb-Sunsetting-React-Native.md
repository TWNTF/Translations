---
layout: single
comments: false
title: React Native at Airbnb: Sunsetting React Native
---

# 弃用 React Native

> 由于大量的技术和组织问题,我们将弃用 React Native,并且今后将使用 Native 进行开发.

_这是我们介绍 React Native 经验和 Airbnb mobile 未来计划[系列博客](https://medium.com/airbnb-engineering/react-native-at-airbnb-f95aa460be1c)的第四篇_.

尽管已经有许多团队正在使用或计划使用 React Native 进行开发, 但是使用 React Native 却无法打到我们最初的目标.此外,我们还遇到了许多无法克服的[技术](https://medium.com/airbnb-engineering/react-native-at-airbnb-the-technology-dafd0b43838)和[组织](https://medium.com/airbnb-engineering/building-a-cross-platform-mobile-team-3e1837b40a88)方面的问题,使得 React Native 的开发工作变得更加困难.

因此,我们将弃用 React Native,并使用 Native 开发继续前进.

## 无法达到目标

### 开发速度

如果 React Native 如预期一样工作, 开发将会十分迅速. 但是大量的[技术](https://medium.com/airbnb-engineering/react-native-at-airbnb-the-technology-dafd0b43838)和[组织](https://medium.com/airbnb-engineering/building-a-cross-platform-mobile-team-3e1837b40a88)方面的问题,是许多项目产生了非预期的推迟.

### 代码质量

最近, 随着 React Native 的不断成熟以及积累的经验不断增多,我们能够完成一些曾经不确定是否可行的事情.我们实现了`共享元素转换(shared element transitions)`,`视差特效(parallax)`以及掉帧的性能优化等. 然而,某些技术问题(如初始化和异步渲染等)导致某些功能很难实现,内外部资源的匮乏也增加了开发的难度.

### 一次编码到处运行

虽然 React Native 的特性几乎都是跨平台的,我们的 app 中也只有小部分使用 React Ntive.此外,还需要大量的桥接基础设施来确保产品工程师能够高效地工作.结果导致我们需要维护三个平台的代码.我们也许可以在 mobile 端和 web 端可以共享代码或 npm 包,但却没有找到恰当的实现方式.

### 提升开发体验

React Native 的开发体验参差不齐.在某些方面(如构建时间)表现的很好.而在其他方面(如调试)则表现的很糟糕.具体细节的介绍在本系列的第二部分中.

## 弃用计划

由于 React Native 无法帮助我们达到一些特定的目标,我们决定弃用 React Native.我们目前正在与团队一起制定健全的过渡计划.我们已经停止了所有的 React Native 新功能的开发,并且计划在今年年底之前把最常使用的页面过渡到 native.这也受益于事先计划好的重新设计.我们的 native 基础设施团队将会持续支持 React Native 直到 2018 年.在 2019 年,我们将逐渐减少对 React Native 的支持,开始减少 React Native 的开销,如启动时的 runtime 初始化.

在 Airbnb,我们拥抱开源.我们积极地使用和贡献全世界许多的开源项目,而且也开源了我们一部分的 React Native 工作.由于我们不再会使用 React Native,我们也无法继续维护我们的 React Native 仓库和社区.考虑到社区,我们将会吧我们 React Native 的开源工作迁移到[react-native-community](https://github.com/react-native-community),我们已经开始着手[react-native-maps](https://github.com/react-community/react-native-maps)的迁移工作了,后续还会继续迁移[react-navigation](https://github.com/airbnb/native-navigation)和[lottie-react-native](https://github.com/airbnb/lottie-react-native/).

## React Native 并非一无是处

尽管 React Native 无法满足我们的需求,但是使用过 React Native 的工程师通常都认为体验不错.在这些工程师中:

* 60%表示鹅妹子嘤
* 20%认为还不错
* 15%稍有不满
* 5%强烈不满

63%的工程表示如果有机会的话还会选择 React Native,74%会考虑在新项目中使用 React Native.值得注意的是，这些结果存在固有的选择偏差，因为它只选择使用过 React Native 的人进行调查。

这些工程师写了 80000 行产品代码,220 个页面以及 40000 行 javascript 基础设施.作为参考,在每个 native 平台上,我们拥有大概 10 倍的代码量以及 4 倍的页面.

## React Native 正在逐渐成熟

这一系列文章说明了我们目前的 React Native 经历.但是 Facebook 和广大的 React Native 社区正专注于使 React Native 能大规模地应用到混合 App 中.React Native 正以最快速度发展.去年一共有超过 2500 个提交,而且 Facebook[刚刚宣布](https://facebook.github.io/react-native/blog/2018/06/14/state-of-react-native-2018)他们正在解决我们直面的技术问题.即使我们不在使用 React Native,我们仍会关注这部分工作.

## 写在最后

我们把 React Native 集成到了快速大型 App 中,并且持续快速的开发.我们遇到的许多问题是由我们采用的混合方案引起的.我们的规模允许我们承担并且解决这些问题,但小公司可能没有时间来解决这些问题.使 React Native 与 native 无缝连接是有可能的但是十分具有挑战性.每一个使用 React Native 的公司都会有自己独特的经历(团队构成,App,产品需求和 React Native 成熟度等).
当把迭代速度,质量,开发体验等一系列因素综合起来考虑 React Native 是否达到或超过了我们的目标和预期时,很多时候我们都感觉像是在颠覆移动开发的边缘.即使这种经历是振奋人心的,当我们在平衡好处和痛点,以及目前团队的需求和资源时,我们认为我们不再需要 React Native.

是否使用一个进平台是一个重大的决定,而且完全取决于你的团队情况.我们的经历和退出的原因也许并不适用于你的团队.事实上,[许多公司](https://instagram-engineering.com/react-native-at-instagram-dd828a9a90c7)还在继续成功地使用 React Native,而且 React Native 对许多其他公司而言可能仍然是最佳选择.

尽管我们从来没有停止过 native 的研发,弃用 React Native 释放了许多资源,可以使 native 发展的更好.下一篇将介绍我们 native 的新计划.

_这是 Airbnb 关于 React Native 经验分享和移动端未来计划系列文章的第四篇._

[第一篇: React Native at Airbnb]()

[第二篇: The Technology]()

[第三篇: Building a Cross-Platform Mobile Team]()

[第四篇: Making a Decision on React Native]()

[第五篇: What's Next for Mobile]()
