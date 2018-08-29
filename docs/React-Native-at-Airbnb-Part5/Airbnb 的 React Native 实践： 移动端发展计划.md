* 原文地址：https://medium.com/airbnb-engineering/whats-next-for-mobile-at-airbnb-5e71618576ab
* 译文出自：TWNTF
* 译者：Yingjian Li


# Airbnb 的 React Native 实践： 移动端发展计划

> 整装待发,重回原生

_这是我们介绍 React Native 经验和 Airbnb mobile 未来计划[系列博客](https://medium.com/airbnb-engineering/react-native-at-airbnb-f95aa460be1c)的第五篇_

## 激动人心的时刻即将来临

虽然在使用 React Native 开发,但是我们也一直在积累原生开发的经验.现在我们已经一系列正在进行中或已完成的项目.其中,部分项目是受到 React Native 经验和最佳实践的启发而完成的.

### 服务器端渲染

尽管不再使用 React Native, 我们仍然十分重视只写一次产品代码的价值. 我们仍然高度依赖我们的统一设计语言系统, 使许多 Android 和 iOS 的页面看起来完全一样.

许多团队已经尝试了强大的服务器端渲染的框架,并且达成一致.在这些框架中,服务器将描述组件渲染,页面配置和相关操作的信息发送给移动设备.不同的移动平台获取到数据后使用 DLS 组件渲染原生页面或者整个流程.

大规模服务器端渲染也存在着一些挑战. 下面是我们正在解决的一些问题:

* 安全升级组件定义的同时维护向下兼容性.
* 跨平台共享组件的类型定义.
* 在运行时的事件响应,如点击按钮或用户输入
* 多个 JSON 驱动页面间跳转的同时,维护内部状态.
* 构建时没有实现的自定义组件的渲染.我们正在尝试使用[Lona](https://github.com/airbnb/Lona/)解决.

服务器端渲染框架使我们能够更方便的进行试验以及热更新,为我们提供了巨大的价值.

### Epoxy 组件

在 2016 年,我们开源了 Android 的[Epoxy](https://github.com/airbnb/epoxy).Epoxy 可以实现简单的异构`RecyclerViews`, `UICollectionViews`和`UITableviews`. 如今大多数的新页面都使用 Epoxy. 这使得我们可以将每个页面分解成独立的组件,进而实现懒加载. 而且在 iOS 和 Android 上均可使用.
iOS 端示例代码:

```swift
BasicRow.epoxyModel(
  content: BasicRow.Content(
    titleText: "Settings",
    subtitleText: "Optional subtitle"),
  style: .standard,
  dataID: "settings",
  selectionHandler: { [weak self] _, _, _ in
    self?.navigate(to: .settings)
  })
```

在 Android 上,我们利用[DSLs in Kotlin](https://kotlinlang.org/docs/reference/type-safe-builders.html),使得组件的实现变得简单而且类型安全:

```kotlin
basicRow {
 id("settings")
 title(R.string.settings)
 subtitleText(R.string.settings_subtitle)
 onClickListener { navigateTo(SETTINGS) }
}
```

### Epoxy 中的 diffing

在 React 中,[render](https://reactjs.org/tutorial/tutorial.html#what-is-react)会返回一个组件的列表.React 性能的关键是组件仅仅是真实 DOM 的数据模型.对组件树进行 diff,然后只更新改动部分.我们在 Epoxy 中引入了相似的概念.在 Epoxy 中,在[buildModels](https://reactjs.org/tutorial/tutorial.html#what-is-react)中声明整个页面的模型,再加上 Kotlin DSL,使得它在概念上与 React 十分相似,示例代码:

```kotlin
override fun EpoxyController.buildModels() {
  header {
    id("marquee")
    title(R.string.edit_profile)
  }
  inputRow {
    id("first name")
    title(R.string.first_name)
    text(firstName)
    onChange {
      firstName = it
      requestModelBuild()
    }
  }
  // 其余的模型放在这里
}
```

每当你的数据发生变化,调用`requestModelBuild()`将会调用最佳的`RecyclerView`来重新渲染你的页面.

iOS 端的示例代码:

```swift
override func itemModel(forDataID dataID: DemoDataID) -> EpoxyableModel? {
  switch dataID {
  case .header:
    return DocumentMarquee.epoxyModel(
      content: DocumentMarquee.Content(titleText: "Edit Profile"),
      style: .standard,
      dataID: DemoDataID.header)
  case .inputRow:
    return InputRow.epoxyModel(
      content: InputRow.Content(
        titleText: "First name",
        inputText: firstName)
      style: .standard,
      dataID: DemoDataID.inputRow,
      behaviorSetter: { [weak self] view, content, dataID in
        view.textDidChangeBlock = { _, inputText in
          self?.firstName = inputText
          self?.rebuildItemModel(forDataID: .inputRow)
        }
      })
  }
}
```

### 一个全新的 Android 产品框架(MvRx)

在近期开发中,最令人兴奋的是一个正在开发的新框架,我们内部叫做 MvRx.MvRx 结合了`Epoxy`,[`Jetpack`](https://developer.android.com/jetpack/),[`RxJava`](https://github.com/ReactiveX/RxJava)和`Kotlin`的优点,以及一些 React 中的规则,使得构建新页面变得更加简单,更加流畅.它是由常见的开发模式以及 React 中的优点开发的,是一个"固定"([opinionated](https://stackoverflow.com/questions/802050/what-is-opinionated-software))但灵活的框架.它也是线程安全的，几乎所有东西都不在主线程运行，这使得滚动和动画感觉流畅.

到目前为止,它已经应用到了大量的页面中,并且几乎完全不需要处理生命周期.我们目前正在一些 Android 产品上进行试验,并且如果一切顺利的话会将他开源.下面是创建一个发出网络请求的功能页面所需的全部代码:

```kotlin
data class SimpleDemoState(val listing: Async<Listing> = Uninitialized)

class SimpleDemoViewModel(override val initialState: SimpleDemoState) : MvRxViewModel<SimpleDemoState>() {
    init {
        fetchListing()
    }

    private fun fetchListing() {
        // This automatically fires off a request and maps its response to Async<Listing>
        // which is a sealed class and can be: Unitialized, Loading, Success, and Fail.
        // No need for separate success and failure handlers!
        // This request is also lifecycle-aware. It will survive configuration changes and
        // will never be delivered after onStop.
        ListingRequest.forListingId(12345L).execute { copy(listing = it) }
    }
}

class SimpleDemoFragment : MvRxFragment() {
    // This will automatically subscribe to the ViewModel state and rebuild the epoxy models
    // any time anything changes. Similar to how React's render method runs for every change of
    // props or state.
    private val viewModel by fragmentViewModel(SimpleDemoViewModel::class)

    override fun EpoxyController.buildModels() {
        val (state) = withState(viewModel)
        if (state.listing is Loading) {
            loader()
            return
        }
        // These Epoxy models are not the views themself so calling buildModels is cheap. RecyclerView
        // diffing will be automaticaly done and only the models that changed will re-render.
        documentMarquee {
            title(state.listing().name)
        }
        // Put the rest of your Epoxy models here...
    }

    override fun EpoxyController.buildFooter() = fixedActionFooter {
        val (state) = withState(viewModel)
        buttonLoading(state is Loading)
        buttonText(state.listing().price)
        buttonOnClickListener { _ -> }
    }
}
```

MvRx 具有简单的构造,用于处理 Fragment 参数,跨进程重启时`savedInstanceState`的持久化,TTI 追踪以及一些其他的功能.

我们也正在开发一个相似的 iOS 框架,该框架目前正处于早期测试阶段.

尽管我们已经对目前所取得的进展十分满意了,还是希望很快可以听到更多相关的消息.

### 迭代速度

当从 React Native 切换回原生时,迭代速度是显而易见的问题.测试改动的时间从原本只需要 1~2 秒,变成可能需要等待将近 15 分钟是不可接受的.幸运的是,我们能够从一些方面缓解这一现象.

我们在 Android 和 iOS 上构建的基础设施能够根据功能模块编译启动器和部分 app.

在 Android 端使用了[gradle 产品风格](https://developer.android.com/studio/build/build-variants#product-flavors), 我们的 gradle 模块如下图所示:

![](images/gradle%20modules.png)

新的间接层使得工程师可以 app 的切片上进行开发.再配合[Intellij module unloadinig](https://blog.jetbrains.com/idea/2017/06/intellij-idea-2017-2-eap-introduces-unloaded-modules/),可以在 Macbook Pro 上动态提高构建和 IDE 的性能.

我们已经构建了用于创建新的测试 flavor 的脚本,并且在短短几个越能,我们已经完成了超过 20 个.使用这些新风格可以使开发构建提速 2.5 倍,并且超过 5 分钟的比例下降了 15 倍.

作为参考,这是用于动态生成具有根依赖性模块产品风格的[gradle 代码块](https://gist.github.com/gpeal/d68e4fc1357ef9d126f25afd9ab4eee2).

相似地,iOS 上的模块如下图所示:

![](images/iOS%20modules.png)

同样系统的构建速度会提升 3-8 倍.

## 结论

很高兴能够成为一家不怕尝试新技术但又努力保持高质量,高速度和开发体验的公司.最后,React Native 是发布功能,为我们提供了新的移动开发思路的重要工具.如果这听起来像是您想参与的旅程，[请告诉我们](https://www.airbnb.com/careers/departments/engineering)！

_这是 Airbnb 关于 React Native 经验分享和移动端未来计划系列文章的第五篇._

[第一篇: React Native at Airbnb](../React-Native-at-Airbnb-Part1/Airbnb%20的%20React%20Native%20实践：%20概述.md)

[第二篇: The Technology](../React-Native-at-Airbnb-Part2/Airbnb%20的%20React%20Native%20实践：%20技术细节.md)

[第三篇: Building a Cross-Platform Mobile Team](../React-Native-at-Airbnb-Part3/Airbnb%20的%20React%20Native%20实践：%20构建一个跨平台的移动端团队.md)

[第四篇: Making a Decision on React Native](../React-Native-at-Airbnb-Part4/Airbnb%20的%20React%20Native%20实践：%20弃用%20React%20Native.md)

[第五篇: What's Next for Mobile](Airbnb 的 React Native 实践： 移动端发展计划.md)

