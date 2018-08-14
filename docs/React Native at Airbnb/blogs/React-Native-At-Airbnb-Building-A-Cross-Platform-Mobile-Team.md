* 原文地址：https://medium.com/airbnb-engineering/building-a-cross-platform-mobile-team-3e1837b40a88
* 译文出自：TWNTF
* 译者：Yingjian Li


# 构建一个跨平台的移动端团队

> React Native 转型

_这是 Airbnb 关于 React Native 经验分享和移动端未来计划系列文章的第三篇._

除了数不尽的技术上的利弊, 我们还了解到 React Native 对工程团队的意义. 这远比向当前平台中添加一个新库或模式要复杂的多,我们在期间遭遇了许多组织上的挑战.与技术上的挑战不同,组织上的挑战更难发现,解决,恢复.谢天谢地,我们的 mobile 文化十分健全,但下面是在团队中使用 React Native 时需要注意的一些事项.

### 两极分化的 React Native

根据我们的经验, 工程师们关于 React Native 有着完全不同的看法,有人认为它是统一了 Android,iOS 和 web 的银弹,有人则完全反对在团队中使用 React Native.即使是在使用之后,情况也十分类似,有的团队认为它是极好的,而有的人却感到失望并重新使用 native.

### 追踪根本原因

在使用 React Native 时会有许多不可避免的 bug,改进和性能问题.但也还有许多不断变化的部分:

1.  React Native 自身变化十分迅速
2.  基础设施和功能的开发是同时进行的
3.  React Native 对所有人来说都是一门新的技术,需要大家一起学习
4.  我们的文档与调试指引有时会不一致,产生困惑

因此,想要找到问题的根本原始通常是很困难的.有时无法确定问题是由哪个团队引起的还是 React Native 自身导致的.

### React Native 仍然需要 native

一个常见的误解就是 React Native 可以使你完全不用写原生代码.然而事实并非如此,有时你仍然需要写原生代码.例如,文本在不同平台上的渲染会有细微的差别,键盘的处理也有所不同以及在 Android 上 Activity 的重新创建和旋转等. 高质量的 React Native 应用需要仔细地平衡所有平台. 而三个平台间专业知识的平衡又十分困难,就导致了很难产出一个高质量的应用.

### 跨平台调试

大多数的工程师都精通一到两个平台.同时精通 Android,iOS 和 React 的人少之又少.即使绝大部分的工作都是在成熟的 React Native 环境下完成的,有时也需要在深层的 native 层构建或调试.这样工程师们就会在一个自己从来没用过的平台中 debug.而且由于问题的根本原因很难被定位,工程师会无从下手.

### 招聘

即使我们采用了 React Native,我们的 mobile 团队也在不断发展壮大.但是在社区中,许多人开始把 Airbnb 和 React Native 关联了起来,而且有些甚至认为我们的 App 是纯的 React Native App,但事实并非如此.结果许多 Android 和 iOS 工程师会犹豫是否要选择 Airbnb,我们仍然在[招聘](https://www.airbnb.com/careers/departments/engineering)!

### 艰难的 Hybrid App

只用 React Native 或 native 开发应用是相对简单的.但是,一旦你选择了混合开发,你就会发现很多问题.如何划分团队?团队间如何协作?如何跨应用共享状态?如何保证测试?如何有效地跨平台调试?如何决定哪个平台使用新功能?如何雇佣和分配资源?这些都是不可避免而且很难解决的问题.

### 三种开发环境

如果想要高效地开发 React Native,必须要有稳定的,最新的 React Native, Android 和 iOS 环境.对于像 Airbnb 这样体量的公司来说,每个平台都需要大量的时间来配置,学习和更新.几个星期后回来通常意味着你需要花几个小时让一切恢复到最新状态.

### React Native 和 native 之间的平衡

在很多情况下,解决问题的最佳方案都需要横跨 React Native 和 native.例如,我们导航栏的实现大量的使用 Activities 和 ViewControllers,大部分的代码都是各自平台上的 native 代码.很多时候都不清楚究竟是该写 React Native 代码还是 native 代码.而工程师自然会选择他们熟悉的平台,这就会产生一些不理想的代码.

### 跨平台测试

我们发现工程师们会主要开发他们熟悉的平台,而且认为如果在他们测试的平台上是好的,在其他平台上也是一样.受益于 React Native 的特性,大多数情况下确实是这样的.但是有些时候会在 QA 环境或产品环境中出现问题.

### 团队划分

同时使用 React Native 和 native 的团队会同时面对技术问题和沟通问题.一旦把代码库按照 native 和 React Native 划分开,代码会变得碎片化.共享业务逻辑,模型,状态等变得更加困难而且工程师不会再有整个工作流程的开发经验.我们从最开始就清楚这一点,但考虑到如果能够更多的和 web 端协作,情况可能会有好转.一些团队之间开始在 web 和 mobile 之间分享资源和代码,但效果并不明显.

### 感观的迭代速度

使用 React Native 的一个主要目标就是提高开发速度.通常情况下,React Native 的功能是由单个工程师完成的.从 React Native 工程师的角度来看,如果他花费了超过 Android 或 iOS 上开发时间的 50%的时间,即使整体花费的时间更少,他们也会觉得更长.

### 公共资源和文档

Android 和 iOS 已有 10 年历史,而且有数百万的工程师贡献于学习资源,开源和在线帮助.我们利用如[CodePath](https://codepath.com/androidbootcamp)等资源帮助人们学习 Android 和 iOS.即使 React Native 拥有最大的跨平台社区之一,而且可以利用 React 的资源,但仍比 Android 和 iOS 小得多.再加上我们必须内部构建大部分基础设施,这意味着相对于 native,我们把有限的 React Native 资源过度地投资在教育和训练.

---

_这是 Airbnb 关于 React Native 经验分享和移动端未来计划系列文章的第三篇._

[第一篇: React Native at Airbnb]()

[第二篇: The Technology]()

[第三篇: Building a Cross-Platform Mobile Team]()

[第四篇: Making a Decision on React Native]()

[第五篇: What's Next for Mobile]()
