* 原文地址：https://webpack.js.org/plugins/split-chunks-plugin/
* 译文出自：TWNTF
* 译者：wtzeng

# SplitChunksPlugin

最初，chunks（以及在它们中引入的模块）通过父子关系在 webpack graph 中相关联。`CommonsChunkPlugin` 被用于避免它们之间的共同依赖被重复打包，但是想做更多的优化就不可能了。

从 webpack v4 开始，`CommonsChunkPlugin` 被移除，转而支持 `optimization.splitChunks`。

## 默认行为

开箱即用的 `SplitChunksPlugin` 对大多数用户来说，都会很顺手。

默认情况下它只会影响按需加载的 chunks，因为改变初始的 chunks 的话会影响 HTML 文件中引入项目文件的 script 标签。

在满足以下这些条件时，webpack 会自动分割 chunks：

* 新 chunk 是在多个 chunk 间共享的或者是来自 `node_modules` 文件夹
* 新 chunk 将会大于 30kb（在 minify 和 gzip 压缩之前）
* 按需加载的 chunks 并行请求小于5个时
* 初始化加载时的并行请求小于3个时

当最后两个条件不能满足时，webpack 倾向于打包出更大的 chunks 而不是分割。

## 配置

webpack 为开发者提供了一系列的选项来控制 `SplitChunksPlugin` 插件的功能。

> 默认的配置已经尽可能满足了大多数 web 性能优化的最佳实践，但是对你自己的项目来说，优化策略可能有些不同。如果你想改变默认的配置，你应该精确测量你的变化造成的影响以确保你真正从中获益。

### `optimization.splitChunks`

这个配置对象代表了 `splitChunksPlugin` 的默认行为：

```Javascript
module.exports = {
  //...
  optimization: {
    splitChunks: {
      chunks: 'async',
      minSize: 30000,
      maxSize: 0,
      minChunks: 1,
      maxAsyncRequests: 5,
      maxInitialRequests: 3,
      automaticNameDelimiter: '~',
      name: true,
      cacheGroups: {
        vendors: {
          test: /[\\/]node_modules[\\/]/,
          priority: -10
        },
        default: {
          minChunks: 2,
          priority: -20,
          reuseExistingChunk: true
        }
      }
    }
  }
};
```

#### `splitChunks.automaticNameDelimiter`

`string`

插件默认会用来源和 chunk 名称命名生成的 chunk（例如 `vendors~main.js`）。这个选项让你可以指定生成 chunk 的文件名中的分隔符。

#### `splitChunks.chunks`

`function` `string`

这个选项指明了插件将作用在哪些 chunks 上。当值为一个字符串时，有效的值包括 `all`, `async` 以及 `initial`。值为 `all` 时将有强大的效果，因为这意味着即使是在异步加载的 chunks 和非异步加载的 chunks 之间共享的模块都将被分割出来。

```Javascript
module.exports = {
  //...
  optimization: {
    splitChunks: {
      // include all types of chunks
      chunks: 'all'
    }
  }
};
```

此外，你可以提供一个函数来实现更多的控制。这个函数的返回值需要指明插件是否作用在每一个 chunk 上。

```Javascript
module.exports = {
  //...
  optimization: {
    splitChunks: {
      chunks (chunk) {
        // exclude `my-excluded-chunk`
        return chunk.name !== 'my-excluded-chunk';
      }
    }
  }
};
```

> 你可以结合 [html-webpack-plugin](https://webpack.js.org/plugins/html-webpack-plugin/) 一起使用。它可以帮你把所有生成的 vendor chunks 插入到页面中。

#### `splitChunks.maxAsyncRequests`

`number`

按需加载的并行请求的最大数量。

#### `splitChunks.maxInitialRequests`

`number`

一个入口产生的并行请求的最大数量。

#### `splitChunks.minChunks`

`number`

模块在被分割出来之前，最少被几个 chunks 共享。

#### `splitChunks.minSize`

`number`

分割出来的 chunk 的最小文件大小。

#### `splitChunks.maxSize`

`number`

`maxSize`（包括全局的 `optimization.splitChunks.maxSize` 和每个 cache group 的 `optimization.splitChunks.cacheGroups[x].maxSize` 以及 fallback cache group `optimization.splitChunks.fallbackCacheGroup.maxSize`）告诉 webpack 把大于 `maxSize` 的 chunks 分割出来，分割出来的 chunks 大小至少为 `minSize`。相关的算法是确定的，任何对模块的改变只会影响自身 chunk 相关的打包行为，不会影响其它分割出来的 chunk。所以它对持久化缓存很有用。当模块被分割出来之前大于 `maxSize` 或者小于 `minSize`，这些模块就不会被分割出来。

当 chunk 被分割出来时， 插件会根据一开始的 chunk 来得到一个分割出来的 chunk 的文件名。根据 `optimization.splitChunks.hidePathInfo` 的值，插件还会在文件名中加入一个与初始模块的名称或 hash 有关的 key。

`maxSize` 选项意图用于 HTTP/2 和持久化缓存中。为了更好的缓存策略，它增加了网络请求的数量。它同样可以减小文件的大小以便更快地完成 rebuild 过程。

> `maxSize` 比 `maxInitialRequest/maxAsyncRequests` 有更高的优先级。选项之间的优先级是 `maxInitialRequest/maxAsyncRequests < maxSize < minSize`。

#### `splitChunks.name`

`boolean: true` `function` `string`

分割出来的 chunk 的名称。值为 `true` 时，插件会根据所在的 chunks 和 cache group 的 key 自动生成一个文件名。当值为 `srting` 或者 `function` 时，你可以使用自定义的文件名。如果文件名恰好与入口文件名相同，入口文件会被移除。

```Javascript
module.exports = {
  //...
  optimization: {
    splitChunks: {
      name (module) {
        // generate a chunk name...
        return; //...
      }
    }
  }
};
```

> 如果为分割出来的不同 chunks 分配了相同的文件名，所有的 vendor 模块都会被打包进一个共享的 chunk 中，尽管这种做法是不被推荐的，因为它会导致超出需要的代码被加载。

#### `splitChunks.cacheGroups`

Cache groups can inherit and/or override any options from splitChunks.*; but test, priority and reuseExistingChunk can only be configured on cache group level. To disable any of the default cache groups, set them to false.

Cache groups 可以继承或者覆盖掉 `splitChunks.*` 中的任何配置，但是 `test`, `priority` 以及 `reuseExistingChunk` 选项只能在 cache group 级别配置。如果想禁用任何默认的 cache group，设置它们为 `false`：

```Javascript
module.exports = {
  //...
  optimization: {
    splitChunks: {
      cacheGroups: {
        default: false
      }
    }
  }
};
```

##### `splitChunks.cacheGroups.priority`

`number`

A module can belong to multiple cache groups. The optimization will prefer the cache group with a higher priority. The default groups have a negative priority to allow custom groups to take higher priority (default value is 0 for custom groups).

一个模块可以属于多个 cache groups，插件会为自定义的 cache group 采用更高的优先级。默认的 groups `priority` 值为负，以便自定义的 groups 优先级更高（自定义 groups 的 `priority` 默认值为 `0`）。

##### `splitChunks.cacheGroups.{cacheGroup}.reuseExistingChunk`

`boolean`

值为 `true` 时，如果当前欲分离的 chunk 中所用的模块和另一个主要 bundle 中的模块相同，那么它会直接复用那个 bundle，而不是新生成一个 chunk。这只会影响最终生成的 chunk 的文件名。（注：这里的翻译不是原文，而是根据这个 [issue](https://github.com/webpack/webpack.js.org/issues/2122) 结合原文总结而成）

```Javascript
module.exports = {
  //...
  optimization: {
    splitChunks: {
      cacheGroups: {
        vendors: {
          reuseExistingChunk: true
        }
      }
    }
  }
};
```

##### `splitChunks.cacheGroups.test`

`function` `RegExp` `string`

控制哪些模块将被打包进这个 cache group 中。省略的话它将会选择所有的模块。它可以匹配模块的绝对路径或者 chunk 名称。当 chunk 的名称匹配时，所有在 chunk 中的模块都将被选中。

## 示例

### 默认：示例 1

```Javascript
// index.js

import('./a'); // dynamic import
```

```Javascript
// a.js
import 'react';

//...
```

结果：一个独立的包含 `react` 的 chunk 将会被创造出来。在 import 被调用时，这个 chunk 将会和 `./a` chunk 一起并行加载。

原因：

* 条件1：这个 chunk 包含了 `node_modules` 中的模块
* 条件2：`react` 大于 30kb
* 条件3：import 调用时产生的并行请求为 2
* 条件4：不影响页面初始化时的请求

这么做的理由是什么呢？`react` 不像你的业务代码会经常变动。通过把它移到一个单独的 chunk 中，这个 chunk 可以独立于你的应用代码被缓存（假设你使用的是 chunkhash，并启用了 records, Cache-Control 等持久化缓存措施）。

### 默认：示例 2

```Javascript
// entry.js

// dynamic imports
import('./a');
import('./b');
```

```Javascript
// a.js
import './helpers'; // helpers is 40kb in size

//...
```

```Javascript
// b.js
import './helpers';
import './more-helpers'; // more-helpers is also 40kb in size

//...
```

结果：一个单独的 chunk 会被创建出来，它包含 `./helpers` 以及其所有依赖。在 import 被调用时，这个 chunk 会和初始的 chunks 被一起加载。

原因：

* 条件1：这个 chunk 被两个 import 调用
* 条件2：`helpers` 大于 30kb
* 条件3：import 调用时产生的并行请求为 2（译者注：没理解为什么不是3，经实验结果也是3）
* 条件4：不影响页面初始化时的请求

把 `helpers` 的内容放到每一个 chunk 里会导致相关代码被下载两次。通过分割 chunk，这些代码只用下载一次即可。我们也为此多发出了一个网络请求，这是一种权衡之下的结果，这也是为什么我们有一个 30kb 最小文件大小的限制（chunk 过小的时候，不值得为此多发一个网络请求）。

### 分割 chunks：示例 1

生成一个 `commons` chunk，包含所有入口点共享的代码。

**webpack.config.js**

```Javascript
module.exports = {
  //...
  optimization: {
    splitChunks: {
      cacheGroups: {
        commons: {
          name: 'commons',
          chunks: 'initial',
          minChunks: 2
        }
      }
    }
  }
};
```

> 这个配置会让你打包的初始化文件变得更大，当有模块不是立即执行时，建议你使用动态加载的方式引入。

### 分割 chunks：示例 2

生成一个 `vendors` chunk，包含应用中所有来自 `node_modules` 的代码。

**webpack.config.js**

```Javascript
module.exports = {
  //...
  optimization: {
    splitChunks: {
      cacheGroups: {
        commons: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          chunks: 'all'
        }
      }
    }
  }
};
```

> 这可能会生成一个包含所有外部包的更大的 chunk。建议你只把核心框架和工具打包进来，之后根据实际情况动态加载剩下的依赖。

### 分割 chunks：示例 3

生成一个 `custom vendor` chunks，包含 `node_modules` 中匹配 `RegExp` 的包。

**webpack.config.js**

```Javascript
module.exports = {
  //...
  optimization: {
    splitChunks: {
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/](react|react-dom)[\\/]/,
          name: 'vendor',
          chunks: 'all',
        }
      }
    }
  }
};
```

This will result in splitting react and react-dom into a separate chunk. If you're not sure what packages have been included in a chunk you may refer to Bundle Analysis section for details.

> 这会把 `react` 和 `react-dom` 分割到一个单独的 chunk 中。如果你不确定什么包被包括进了这个 chunk 中，你可以参考[打包分析](https://webpack.js.org/guides/code-splitting/#bundle-analysis)章节以获取更多细节。

## 更多资料

* [webpack 自动去重算法示例](https://github.com/webpack/webpack/blob/master/examples/many-pages/README.md)
* [webpack 4：代码分割，chunk graph 以及 splitChunks 优化](https://medium.com/webpack/webpack-4-code-splitting-chunk-graph-and-the-splitchunks-optimization-be739a861366)