* 原文地址：https://medium.com/webpack/webpack-4-code-splitting-chunk-graph-and-the-splitchunks-optimization-be739a861366
* 译文出自：TWNTF
* 译者：wtzeng

# webpack 4：代码分割，chunk graph 以及 splitChunks 优化

webpack 4 对 chunk 做graph 了一些大幅度的改进并在 `CommonsChunkPlugin` 之后，新增加了一个分割 chunk 的优化方案。

让我们先来看看以前的 chunk graph 有哪些不足吧。

在以前的 chunk graph 中，chunks 之间被认为是以父子关系关联起来的，然后 chunk 内包含了 modules。

当一个 chunk 有父 chunk 时，如果它被加载了，则说明至少一个它的父 chunk 已经被加载了。这个信息可以用来优化加载过程，例如，当 chunk 中的某个 module 被其所有父 chunk 都引用时，这个 module 就可以从当前 chunk 中移除掉，因为在它的任意一个父 chunk 里这个 module 都已经被加载了。

在一个入口文件或者异步加载的文件中，都会引用一份 chunk 清单，这些 chunk 是被平行加载的。

这种 graph 关系让 chunk 分割变得困难。用 CommonsChunkPlugin 遇到的情况举例，modules 被从一个或多个 chunks 中移除出来，生成一个新的 chunk 文件，这个新的 chunk 需要被连接进 chunk graph 中，怎么做呢？当作原先 chunk 的父 chunk ？还是当作原先 chunk 的子 chunk 吗？CommonsChunkPlugin 把它作为原先 chunk 的父 chunk，但是技术上来看这是错的，并且对其它优化造成了负面影响（因为当作父 chunk 是不准确的）。

webpack 4 中的新 chunk graph 引入了一个新对象，ChunkGroup，每个 ChunkGroup 对象都包含了各自的 Chunks 引用的信息。

在一个入口文件或者异步加载文件中，一个单独的 ChunkGroup 被使用，这意味着所有被文件引用的 chunks 都是平行的关系了。一个 chunk 又可以被多个 ChunkGroups 引用。

在 chunks 之间，不再是父子关系，现在它们的关系都保存在了 ChunkGroups 中。

现在 chunks 的分割就很好表达了。新分割出来的 chunk 被添加到所有包含这个 chunk 的 ChunkGroups 中。这就不会再由于 chunks 间的父子关系产生负面影响了。

---

现在麻烦解决了，我们可以开始深入 chunk 分割了。我们可以分割出来任何 chunk，并不用冒着破坏 chunk graph 的风险。

CommonsChunkPlugin 有着许多问题：

* 它会分割多余 module 到 chunk 中
* 它处理异步 chunk 不够高效
* 它的配置使用很麻烦
* 它的实现方式不好理解

所以一个新的插件产生了：`SplitChunksPlugin`。

它通过计数模块的重复使用次数以及模块分类（例如 node_modules 中的模块），启发式地实现了自动标识需要被分割出来的模块，然后把它们分割成为新的 chunks。

这是一种范式的转变。`CommonsChunkPlugin` 的理念像是：“创建一个 chunk，然后把所有符合 `minChunks` 条件的模块都移到那个 chunk 中”。`SplitChunksPlugin` 的理念像是：“要做这么几件事，确保你实现了它们”。（命令式 vs 声明式）

`SplitChunksPlugin` 有着这些优点：

* 它不会分割多余的模块到 chunk 中（只要你没有强制通过 name 来合并 chunk）
* 它处理异步 chunk 也足够高效
* 它默认支持异步 chunk
* 它可以把 vendor  的 chunk 分割成多个
* 它更易于使用
* 它不用在 chunk graph 中用一些小花招
* 大部分行为都可以自动完成

---

以下是几个使用 SplitChunksPlugin 的例子。这些例子只展示插件的默认行为，如果额外配置的话可以表现出更多可能性。

注意1：你可以通过 `optimization.splitChunks` 项来配置它。这些例子展示了一些关于 chunks 的东西，插件默认只在异步 chunks 上工作，但在 `optimization.splitChunks.chunks` 为 `all` 时，它也会工作在初始的 chunks 上。

注意2：我们假设这里用到的每一个外部库都大于 30kb，因为插件的优化仅在包大小超过这个阈值时生效。

## Vendors

`chunk-a`: react, react-dom, 一些组件

`chunk-b`: react, react-dom, 一些其它组件

`chunk-c`: angular, 一些组件

`chunk-d`: angular, 一些其它组件

webpack 将会自动生成两个 vendors chunks，得到以下结果：

`vendors~chunk-a~chunk-b`: react, react-dom

`vendors~chunk-c~chunk-d`: angular

`chunk-a` 到 `chunk-d`: 只有 chunk 内独有的组件

## Vendors 重叠

`chunk-a`: react, react-dom, 一些组件

`chunk-b`: react, react-dom, lodash, 一些其它组件

`chunk-c`: react, react-dom, lodash, 一些其它组件

webpack 也会对此生成两个 vendors chunks，得到以下结果：

`vendors~chunk-a~chunk-b~chunk-c`: react, react-dom

`vendors~chunk-b~chunk-c`: lodash

`chunk-a` 到 `chunk-c`: 只有 chunk 内独有的组件

## 共享模块

`chunk-a`: vue, 一些组件, 一些共享组件

`chunk-b`: vue, 一些其它组件, 一些共享组件

`chunk-c`: vue, 一些其它组件, 一些共享组件

当共享组件大于 30kb，webpack 会生成 vendors chunk 以及一个 commons chunk，得到以下结果：

`vendors~chunk-a~chunk-b~chunk-c`: vue

`commons~chunk-a~chunk-b~chunk-c`: 一些共享组件

`chunk-a` 到 `chunk-c`: 只有 chunk 内独有的组件

当共享组件小于 30kb 时，webpack 倾向于在 `chunk-a` 到 `chunk-c` 之间重复打包共享组件。因为我们认为这种情况下，减小包大小带来的好处不足以抵消因为代码分割造成的多发网络请求带来的坏处。

## 多个共享模块

`chunk-a`: react, react-dom, 一些组件, 一些共享 react 组件

`chunk-b`: react, react-dom, angular, 一些其它组件

`chunk-c`: react, react-dom, angular, 一些组件, 一些共享 react 组件, 一些共享 angular 组件

`chunk-d`: angular, 一些其它组件, 一些共享 angular 组件

webpack 将会生成两个 vendors chunks 和两个 commons chunks，得到以下结果：

`vendors~chunk-a~chunk-b~chunk-c`: react, react-dom

`vendors~chunk-b~chunk-c~chunk-d`: angular

`commons~chunk-a~chunk-c`: 一些共享 react 组件

`commons~chunk-c~chunk-d`: 一些共享 angular 组件

`chunk-a` to `chunk-d`: 只有 chunk 内独有的组件

---

注意：由于生成的 chunk 名称包含了所有来源 chunk 的名称。建议在有持久缓存机制的生产环境上，不要包含 `[name]` 模板作为文件名，或者通过设置 `optimization.splitChunks.name: false` 来关闭文件名生成机制。否则当新的 chunk 包含之前打包过的 vendors 时，原来打包出来的 vendors chunk 缓存将会失效。