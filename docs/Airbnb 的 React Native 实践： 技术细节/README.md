* 原文地址：https://medium.com/airbnb-engineering/react-native-at-airbnb-the-technology-dafd0b43838
* 译文出自：TWNTF
* 译者：Yingjian Li


# Airbnb 的 React Native 实践： 技术细节

> 一些技术上的细节

_这是 Airbnb 关于 React Native 经验分享和移动端未来计划系列文章的第二篇._

在 Android, iOS, web 和跨平台框架中,React Native 是一个全新的,快速发展的平台. 在两年后,我们可以肯定地说 React Native 在很多方面都是革命性的.React Native 对移动开发时一种[范式转移](https://en.wikipedia.org/wiki/Paradigm_shift),我们从中收获了许多,但是随之而来的还有一些明显的痛点.

## React Native 的优点

### 跨平台

React Native 最主要的优点就是代码可以同时在 Android 和 iOS 上运行.大多数用 React Native 完成的功能可以达到 95%-100%的代码共享率,只有 0.2%的文件需要指定平台的(_.android.js/ _.ios.js).

### 统一的设计语言系统(DLS)

我们开发了一种跨平台的设计语言[DLS](https://airbnb.design/building-a-visual-language/). 每个组件都有 Android, iOS, React Native 和 web 版本.拥有一个统一的设计语言对于开发跨平台功能是十分必要的,因为这样可以保证不同平台间的设计,组件名和页面的一致性. 然而,我们仍然可以根据平台特征对不同平台做出更合理的的决策. 例如, 我们在 Android 上使用了原生的[Toolbar](https://developer.android.com/reference/android/support/v7/widget/Toolbar),在 iOS 上使用了[UINavigationBar](https://developer.apple.com/documentation/uikit/uinavigationbar), 并且由于不符合 Android 平台设计规范的原因,我们在 Android 上隐藏了[扩展指示器](https://developer.apple.com/ios/human-interface-guidelines/views/tables/).

我们选择重写组件,而不是封装原生组件.因为分离特定平台的 API 是一种更可靠的方式, 这样可以减少 Android 工程师和 iOS 工程师再不熟悉 React Native 情况下的维护成本. 然而,这样确实会导致同一组件在原生平台和 React Native 之间的碎片不同步.

### React

React 被大家喜爱是有[原因](https://insights.stackoverflow.com/survey/2018/#technology-most-loved-dreaded-and-wanted-frameworks-libraries-and-tools)的,不但操作简单,而且功能强大,扩展性强.React 中我们最喜欢的是:

* **组件**: React 中的组件通过`props`和`state`加强了关注分离,这使得 React 具有可扩展性.
* **简化的生命周期**: 众所周知, Android 和 iOS(稍好于 Android)上的生命周期是十分复杂的. React 组件跟本地解决了这个问题,这也使得学习 React Native 比学习 Android 和 iOS 简单得多.
* **声明式**: React 的声明式特性使我们能同步 UI 与 state.

### 迭代速度

当我们在开发 React Native 时, 利用[热加载](https://facebook.github.io/react-native/blog/2016/03/24/introducing-hot-reloading.html)特性,只需要 1-2 秒就可以在 Android 和 iOS 上测试我们的改动. 尽管构建性能在原生 App 中拥有最高的优先级,但从来没有达到过 React Native 中的迭代速度.最好情况下,原生的编译时间需要 15 秒.但是完整的构建需要长达 20 分钟.

### 投资基础设施

我们做了大量的与原生基础设施集成的工作.所有的核心部分(如网络,国际化,实验,共享元素转换,设备信息,账号信息)和一些其他部分都被封装在了单独的 React Native 的 API 中.因为我们希望把现有的 Android 和 iOS 的 API 封装成具有规范性和一致性的模块暴露给 React,所以这部分桥接工作会更复杂一点.虽然随着快速迭代和新基础设施的开发,需要不断地更新这些桥接中间层，但基础设施团队的投入使产品工作变得更加容易.

如果不在基础设施上大量投资的话,React Native 会降低开发体验和用户体验.因此,我们认为引入 React Native 到一个成熟的 App 中,需要不断地有效地投入.

### 性能

React Native 最大的隐患就是它的性能.然而我们在实践中这几乎算不上是问题.大多数的 React Native 页面和原生一样流畅. 通常情况下,我们都是从单一维度来考虑性能问题.经常会有移动开发工程师看到 JS 代码会有"比 Java 慢"的感觉.然而在很多情况下,将业务逻辑和布局抽出主线程会提高渲染的性能.

我们遇到的性能问题大多是由过多的渲染引起的,[`shouldComponentUpdate`](https://reactjs.org/docs/react-component.html#shouldcomponentupdate),[`removeClippedSubviews`](https://facebook.github.io/react-native/docs/view.html#removeclippedsubviews)和更好的使用`Redux`可以帮助我们缓和性能问题.

然而,初始化和第一次的渲染时间使得 React Native 在启动画面,深度链接时的性能较差,而且在页面间导航的 TTI(Time to iteract)会较长.此外,页面掉帧也很难 debug,因为 React Native 和原生视图间的转换是由[Yoga](https://github.com/facebook/yoga)负责的.

### Redux

我们使用[Redux](https://redux.js.org/)来高效地管理`state`,避免 UI 的 state 不同步,方便页面间共享数据.然而,Redux 中的模板代码却一直被人诟病,而且学习曲线较难.虽然我们提供了一些通用模板的生成器,但是在使用 React Native 时, 它仍是最有挑战性和最容易产生混淆的部分.不过值得注意的是,这些问题并不是 React Native 特有的.

### 原生支持

由于 React Native 中的所有内容都能桥接到原生代码,因此最后我们实现了许多最开始不确定能否实现的功能,如:

1.  _共享元素变换(Shared element transitions)_: 我们构建了一个支持原生 Android 和 iOS 上共享元素代码的`<SharedElement>`组件,甚至在 React Native 页面间也通用.
2.  _Lottie_:我们在 Android 和 iOS 上封装现有的库,使 Lottie 支持 React Native.
3.  原生网络栈: 在 Android 和 iOS 上,React Native 使用了我们现有的原生网络栈和缓存.
4.  其他核心基础设施:像网络一样,我们还封装了其他现有原生基础设施,如国际化,实验等,以便无缝地在 React Native 上开展工作.

### 静态分析

我们本可以利用在 web 端使用`ESLint`的大量经验,但我们 Aibnb 中第一个使用`prettier`的平台.我们发现它可以有效地减少 PR 上的小问题以及琐碎的争论.`prettier`现在已经开始被使用到 web 基础设施团队中了.

我们还使用分析来衡量渲染次数和性能,以找出具有性能问题的高优先级页面.

相较于我们的 web 基础设施,React Native 小而新,因此适合试验许多新想法. 我们在 React Native 上创建的许多工具和想法已经推广到 web 端了.

### 动画

感谢 React Native [Animated](https://facebook.github.io/react-native/docs/animated.html)库, 我们才能实现无抖动的动画,甚至是交互驱动的动画,如滚动视差.

### JS/React 开源

因为 React 确实在运行`React`和`Javascript`, 我们可以利用大量 javacript 的工程,如`redux`,`reselect`,`jest`等.

### Flexbox

React Native 通过[Yoga](https://github.com/facebook/yoga)处理布局.Yoga 是一个通过[flexbox](https://www.w3schools.com/css/css3_flexbox.asp)API 处理布局计算的跨平台 C++库.我们在早期使用 Yoga 时,遇到了缺乏宽高比等不足,但已经在后续的更新中解决了.此外,有趣的教程(如[flexboxfroggy](https://flexboxfroggy.com/))使得入门变得更加的有趣.

### 与 web 端合作

在后面使用 React Native 时,我们开始一次性构建 web,iOS 和 android,因为 web 端也同样使用`Redux`, 我们发现许多代码不需要更改就可以在 web 和原生平台间跨平台分享.

## 不足

### React Native 还不够成熟

React Native 远不如 Android 和 iOS 成熟.它是一门新技术,但野心勃勃,发展极其迅速.尽管 React Native 在大多数场合下都表现得十分出色,但也在某些地方表现的并不成熟,而且使得一些原生中的小功能变得十分复杂.不幸的是,很难预测什么时候会遇到这种问题,而且需要多长时间来解决这些问题(也许几个小时,也许几天).

### 维护 React Native 的 fork

由于 React Native 的不成熟,很多时候我们需要修补 React Native 的源码.除了贡献给 React Native 外, 我们还必须[维护一个 fork](https://github.com/airbnb/react-native/commits/0.46-canary)来快速的合并改动,升级我们的版本.在两年内,我们不得不在 React Native 添加大概 50 多条提交.这使得升级 React Native 的过程变得十分痛苦.

### javascript 工具

Javascript 是一门无类型语言,缺少安全性既不易扩展,又成为了那些习惯了类型语言,对 React Native 感兴趣的的移动工程师们争论的焦点. 我们尝试采用[`flow`](https://flow.org/),但是晦涩难懂的错误信息使得开发体验大幅度下降.我们还尝试了[`TypeScript`](http://www.typescriptlang.org/),但在与现有基础设施(如[`babel`](https://babeljs.io/)和[`metro bundler`](https://github.com/facebook/metro))的集成时却出现了问题.尽管如此,我们在 web 端会继续使用`TypeScript`.

### 重构

JavaScript 无类型的一个副作用就是重构会变得十分困难,而且容易出错.重命名 props 时,尤其像`onClick`这种常见 props 或者传到多个组件中的 props,想要准确地对他们进行重命名简直就是噩梦.更糟的情况是, 重构可能会破坏生产环境而不是编译过程,而且很难添加适当的静态分析规则.

### JavaScript 核心环境不一致

React Native 一个微妙而棘手的问题是由[JavaScript 核心环境](https://facebook.github.io/react-native/docs/javascript-environment.html)引起的.下面是我们得出的结论:

* iOS 将 JavaScript 核心环境[开箱即用](https://developer.apple.com/documentation/javascriptcore),因此 iOS 具有一致性而且不会出现问题.
* Android 中没有 JavaScript 核心环境,所以 React Native 需要自己打包,但默认的是[老版本](https://github.com/facebook/react-native/issues/10245). 因此我们必须打包一个[新版本](https://github.com/react-community/jsc-android-buildscripts).
* 在 Debug 时,React Native 依赖于强大 Chrome 开发者工具.然而在 debug 时,所有的 JavaScript 都运行在 Chrome 的 V8 引擎当中.这在 99.9%的情况下都不会出现问题.但是,我们发现`toLocaleString`在 iOS 和 debug 模式下的 Android 正常运行.事实证明 Android 的 JSC 中不包含该方法,它会默默的失败,除非你是在 debug 模式下.如果没有类似的经验,debug 类似问题将会十分痛苦.

### React Native 开源库

学习一个平台是十分困难而且耗时的.大多数人只熟悉一到两个平台.包含原生桥接的 React Native 库(如地图,视频等)需要三个平台的知识.我们发现大多数的 React Native 开源项目的作者都只精通一到两个平台.这会导致 Android 和 iOS 的不一致性或意想不到的 bug.

在 Android 上,许多的 React Native 库还需要使用`node_modules`的相对路径, 而不是像社区期望的那样利用 maven 发布工件.

### 并行的基础设施

我们已经在 Android 和 iOS 的原生基础设施上累计了许多年的经验.然而对于 React Native 我们是从零开始,因此我们需要完成许多与已有基础设施的桥接工作.这意味着许多时候一个产品工程师会需要一个还不存在的功能.这时他们要么在他们不熟悉的平台中开发范围外的功能,或者被阻塞,直到桥接工作的完成.

### 崩溃监控

我们使用[`Bugsnag`](https://www.bugsnag.com/)监控 Android 和 iOS 上的崩溃记录.尽管我们能使`Bugsnag`在所有平台上正常工作,但相比于其他平台,他并不十分可靠而且需要更多工作量.因为 React Native 是一门相当新的技术,而且工业中使用较少,我们不得不构建大量的基础设施(如内部上传 source map), 还必须使`Bugsnag`能够过滤出那些发生在 React Native 上的崩溃信息.

由于 React Native 中大量自定义基础设施的存在,我们有时遇到一些无法正常收到崩溃报告或 source map 没有正确上传的问题.

最后,调试 React Native 崩溃通常是十分困难的,如果 bug 涉及 React Native 和原生代码,调用堆栈不会从 React Native 跳到 native.

### 原生桥接

React Native 有一个[`桥接 API`](https://facebook.github.io/react-native/docs/communication-ios.html)可以在 React Native 和 native 间通信. 你需要写大量繁琐的代码来使它正常工作.首先它需要正确的设置所有的三种开发环境.我们还遇到过许多类型问题,例如整数类型通常会被封装成字符串,直到它通过桥接器时,我们才注意到这个问题.更严重的情况下, iOS 会默默失败而 Android 会崩溃.从 2017 年年底开始,我们开始研究自动生成 TypeScript 定义的桥接代码,但为时已晚.

### 初始化时间

在 React Native 第一次渲染之前,我们必须初始化运行时,不幸的是,像我们这种大小的 App,即使在高端机上也需要花费几秒的时间.因此,React Native 几乎不可能用来做启动页面. 我们通过在 app 启动时初始化的方式,减少 React Native 的初次渲染时间.

### 初始渲染时间

与原生页面不同,渲染 React Native 至少需要经历主线程->js->yoga 布局线程->主线程的过程,才能有足够的信息完成第一次的页面渲染.我们观察平均初始渲染时间,iOS 需要 280ms,Android 需要 440ms.在 Android 上,我们使用通常被用在共享元素变化的[`postponeEnterTransition`](https://developer.android.com/reference/android/app/Activity.html#postponeEnterTransition%28%29)API,在渲染完成之后延迟展示页面.iOS 上,我们在加速 React Native 的导航栏配置时遇到了问题.因此我们为所有的 React Native 页面转换加了 50ms 的人造延迟, 避免配置完成后导航栏的闪烁.

### App 大小

React Native 对 app 的大小有着不可忽视的影响.在 Android 上 React Native(Java + JS + Yoga 等原生库 native + JavaScript 运行时)的总大小为 8mb/ABI. 加上 apk 中的 x86 和 arm(32 位),大小会达到将近 12mb.

### 64 位

由于[这个](https://github.com/facebook/react-native/issues/2814)问题,我们仍然无法在 Android 上使用 64 位的 APK.

### 手势

我们避免在需要复杂手势的页面上使用 React Native,因为 Android 和 iOS 的触屏子系统不同,所以 React Native 社区很难得到一个统一的 API.然而,随着[`react-native-gesture-handler`](https://github.com/kmagiera/react-native-gesture-handler)1.0 的发布,这部分工作也将持续地取得进展.

### 长列表

React Native 已经在这方面取得了一定的进展([`FlatList`](https://facebook.github.io/react-native/docs/flatlist.html)),但是还远远达不到 Android[`RecyclerView`](https://developer.android.com/guide/topics/ui/layout/recyclerview)和 iOS[`UICollectionView`](https://developer.apple.com/documentation/uikit/uicollectionview)的成熟度和灵活性.由于线程处理的问题,许多问题很难被解决.由于适配器数据无法同步访问,因此在快速滑动时可能会出现页面因异步渲染而闪烁的情况.文本也无法同步的测量,因此 iOS 上无法根据预先计算好的高度进行特定的优化.

### 升级 React Native

尽管大多数情况下,升级 React Native 版本都是微不足道的,但有时却是十分痛苦的.特别是因为使用了 React 16 的 alpha 和 beta 版本,React Native 0.43(2017.4)到 React Native 0.49(2017.10)间的版本几乎都是不可用的.这是一个很大的问题,因为大多数 web 端的 React 库都不支持预发布的 React 版本.在 2017 年年中,关于这次升级依赖关系的争吵对 React Native 的基础设施造成了巨大的伤害.

### Accessibility

2017 年我们对 Accessibility 进行了一次改革,投入了大量的资源以确保残疾人也能够使用 Airbnb 根据自身需求预定房源. 然而 React Native 的可访问性 API 中存在着许多问题.为了实现最简单的可访问栏,我们不得不维护自己的 React Native [fork](https://github.com/airbnb/react-native/commits/0.46-canary). 对于这些情况,可能在 Android 和 iOS 上只需要一行的修改,但我们需要花费几天的时间来考虑如何把它添加到 React Native 中, cherry pick,填到 React Native 的 issue 中,并在接下来的几个星期里持续关注该 issue.

### 令人厌烦的崩溃

我们曾经不得不解决一些非常奇怪的,很难修复的崩溃.例如,我们现在遇到的这个`@ReactProp`的[问题](https://issuetracker.google.com/issues/37045084),在任何设备上都无法重现,即使与崩溃设备完全相同的软硬件也不行.

### Android 上跨进程的 SavedInstanceState

Android 会频繁地清理后台进程, 但可以[同步地把 state 存储到 bundle 中](https://developer.android.com/topic/libraries/architecture/saving-states#use_onsaveinstancestate_as_backup_to_handle_system_initiated_process_death).但是对于 React Native 来说, 所有的 state 都在 js 线程中,无法完成同步存储.即便使用 redux 存储 state 也不行.因为它包含了序列化和非序列化的数据以及其他符合 savedInstanceState bundle 的数据,可能会导致产品环境的崩溃.

---

_这是 Airbnb 关于 React Native 经验分享和移动端未来计划系列文章的第二篇._

[第一篇: React Native at Airbnb](../Airbnb%20的%20React%20Native%20实践：%20概述/README.md)

[第二篇: The Technology](../Airbnb%20的%20React%20Native%20实践：%20技术细节/README.md)

[第三篇: Building a Cross-Platform Mobile Team](../Airbnb%20的%20React%20Native%20实践：%20构建一个跨平台的移动端团队/README.md)

[第四篇: Making a Decision on React Native](../Airbnb%20的%20React%20Native%20实践：%20弃用%20React%20Native/README.md)

[第五篇: What's Next for Mobile](../Airbnb%20的%20React%20Native%20实践：%20移动端发展计划/README.md)

