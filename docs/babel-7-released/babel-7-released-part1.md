## 译：Babel 7 发布了（Part 1）

在过去的两年里，Babel 7 经历了 4K 次的提交，超过 50 次预版本的发布，当然在这个过程中我们也收获了很多帮助，在这里我们灰常激动的宣布 Bebel 7 的正式发布了。这次 Babel 7 的发布，距离上次 Babel 6 的发布已经过去了大概 3 年的时间！在 Babel 7 发布的第一周里，肯定会存在一些需要动态调整的部分，希望大家能够理解。Babel 7 作为一次重大的发布版本，我们让它的运行速度更快，创建了新的一个升级工具，引入了 JS configs 文件，可以使用 “overrides” 进行配置，更多关于
size/minification 的选项，对于 JSX Fragment，TypeScript的支持，新的 proposals，还有很多很多！

如果你觉得我们的 Babel 做得还不错，你可以在 [Open Collective](https://opencollective.com/babel) 或者 [Patreon](https://www.patreon.com/henryzhu) 上对 Babel 及本人进行打赏，同时欢迎你或者你所在的公司团队能加入我们，一起为 Babel 贡献一份力。在此非常感谢 JS 社区在这个项目上的集体所有权！


### 它正在发生着

软件永远都不会是完美的，我们现在已经准备好输出一些东西，并且它们已经在产品环境下使用一段时间了。现在，`@babel/core`已经达到了每月 510 万的下载量了，这正是因为它使用在很多工具中，如[Next.js](https://zeit.co/blog/next6)，[vue-cli 3.0](https://medium.com/the-vue-point/vue-cli-3-0-is-here-c42bebe28fbb)，[eact Native 0.56](https://facebook.github.io/react-native/blog/2018/07/04/releasing-react-native-056)，甚至有[Wordpress's frontend](https://github.com/Automattic/wp-calypso)。

### Babel 的角色定位

在正式开始讲解前，我想先介绍一下近年来，Babel 在 JavaScript 生态系统中的扮演的角色。
最初的问题是，与服务器语言不同， JavaScript 无法保证每一个用户对其具有相同的支持能力，这是因为用户使用的浏览器对 JavaScript 的支持程度不同（尤其是旧版本的Internet Explorer）。如果开发人员想要使用新的语法（如，`class A{}`），对于使用旧浏览器的用户将会因为`SyntaxError`而显示白屏。

Babel 是一种让开发人员能使用最新的 JavaScript 语法，同时不用担心后向兼容性的方法。例如，程序员们无需考虑将`class A{}`转换成`var A = function A() {}`。

由于 Babel 能对 JavaScript 进行转换， 它也可以运用到新功能的实现中：因此它成为了帮助 TC39（指定 JavaScript 语言的技术委员会）获取更多关于 JavaScript 提议的反馈的桥梁，让社区对于未来语言有更多的发言权。

Babel 是当今 JavaScript 开发的基础，Github 目前有超过 130 万个依赖仓库，每月在 npm 上有 1700 万次下载，同时还有数百个使用者，其中包括许多主流框架（React, Vue, Ember, Polymer）和公司（Facebook, Netflix, Airbnb）。它已成为JavaScript开发的基础，许多人甚至不知道它正在被使用。即使你自己没有直接使用它，很可能在你的依赖中正在使用着 Babel。

感谢那些维护人员以及能够提供资助的个人或团体，这里就不翻译了^^，下面我们开始讲解 Babel 7 到底发生了那些改变。

### 重大改变

>我们正在将这些重大改变整理到我们的[迁移文档](http://babeljs.io/docs/en/next/v7-migration)中，很多改变会通过[babel-upgrade](https://github.com/babel/babel-upgrade)这个新的工具自动完成，剩下的改变也将会在未来加入到 `babel-upgrade` 中。

重大改变如下：
* 不再支持不在维护中的 node 版本，包括：0.10，0.12，4，5（[详情](http://babeljs.io/blog/2017/09/12/planning-for-7.0#drop-support-for-unmaintained-node-versions-010-012-5-4315-https-githubcom-babel-babel-issues-4315)）。
* 使用 scoped package 方式命名，使用`@babel`命名空间（[详情](http://babeljs.io/blog/2017/12/27/nearing-the-7.0-release#renames-scoped-packages-babel-x)）。这样以便于将官方 package 与非官方 package 进行区分，因此`babel-core`就变成了`@babel/core`。
* 移除（并停止发布）所有与年份相关的 presets（`preset-es2015` 等）([详情](http://babeljs.io/blog/2017/12/27/nearing-the-7.0-release#deprecated-yearly-presets-eg-babel-preset-es20xx))。`@babel/preset-env`会取代这些规则集，这是因为`@babel/preset-env`包括所有 yearly presets 的内容，同时它还能针对不能浏览器进行适配。
* 放弃了 [stage](https://tc39.github.io/process-document/) presets（`@babel/preset-stage-0` 等），选择支持多个单个 proposal，当然我们会默认移除`@babel/polyfill`中的 proposals（[详情](https://github.com/babel/babel/pull/8440)），预知更多详情，请参考[博文](https://babeljs.io/blog/2018/07/27/removing-babels-stage-presets)。
* 一些 package 已经重命名了：所有 TC39 proposal 插件的命名中的 `-transform`都会变为`-proposal`（[详情](http://babeljs.io/blog/2017/12/27/nearing-the-7.0-release#renames-proposal)），例如 `@babel/plugin-transform-class-properties`将会变为`@babel/plugin-proposal-class-properties`。
* 对于一些需要面向用户的 package（如`babel-loader`，`@babel/cli` 等），会在 `@babel/core`中引入 peerDependency（[详情](http://babeljs.io/blog/2017/12/27/nearing-the-7.0-release#peer-dependencies-integrations)）。

### babel-upgrade

[babel-upgrade](https://github.com/babel/babel-upgrade)，是我们团队开发的新工具，旨在自动处理升级过程中改变：目前在`package.json`和`.babelrc`的配置中存在依赖。
我们推荐在 git 项目里面直接运行`npx babel-upgrade`，或者你可以通过`npm install -g babel-upgrade`的方式，进行全局安装。
如果你想要同时更新文件，你可以在命令行里添加`--write`和`--install`。
```bash
npx babel-upgrade --write --install
```
请考虑通过提出问题或提PR的形式来帮助每个人过渡到Babel 7！我们未来的愿景是，希望将此工具用于所有未来的重大变革，并为需要更新的项目提供自动更新功能，如同 bot 一般。

### JavaScript Config Files

Babel 7 正在引入`babel.config.js`，该文件并不是必须的，也不是`.babelrc`的替代品，但是在一些特定的场景下还是很有作用的。
`*.js`形式的配置文件在 JavaScript 的生态系统中还是比较常用的，像 ESLint 和 Webpack 都分别使用`.eslintrc.js`和`webpack.config.js`作为配置文件。
下面的例子为，在`babel.config.js`中配置：仅在“产品”环境中使用插件进行编译（当然，你也可以在` .babelrc` 文件中，通过设置 env 配置项来达到相同的目的）。
```javascript
var env = process.env.NODE_ENV;
module.exports = {
  plugins: [
    env === "production" && "babel-plugin-that-is-cool"
  ].filter(Boolean)
};
```
Babel 对于 `babel.config.js`与`.babelrc`在配置解析方式上是不同的。Babel  对`babel.config.js` 对，它始终解析该配置文件；对于`.babelrc`，Babel 会针对每个文件都做一次向上查找的操作，直至找到配置文件为止。这使得`babel.config.js`能使用到下文即将提到的新特性：`override` 。

### 使用 `override`进行选择性配置

最近， 我发表了一篇关于发布 ES2015+ packages 以及编译配置想法的[文章](https://babeljs.io/blog/2018/06/26/on-consuming-and-publishing-es2015+-packages)。

这里面有部分[节选](https://babeljs.io/blog/2018/06/26/on-consuming-and-publishing-es2015+-packages#selective-compilation-with-overrides)是介绍在 Babel 配置中使用一个新的关键字`override`，它允许你分别为每一个部分进行配置。

```javascript
module.exports = {
  presets: [
    // defeault config...
  ],
  overrides: [{
    test: ["./node_modules"],
    presets: [
      // config for node_modules
    ],
  }, {
    test: ["./tests"],
    presets: [
      // config for tests
    ],
  }]
};
```
这样就实现在一个项目当中，可以针对测试、客户端以及服务端代码使用不同的编译配置，如此一来你就很好的避免了为每个文件夹都创建一个新的` .babelrc` 配置文件。

### 速度 

Babel 本身的运行速度就很[快](https://twitter.com/left_pad/status/927554660508028929)，所以它在构建上应该花费更少的时间！为了优化代码，我们做了很多的改变，也接受了[V8](https://twitter.com/v8js) 团队的[补丁](https://twitter.com/rauchg/status/924349334346276864)，我们很高兴与许多其他优秀的JavaScript工具一起成为[Web工具基准](https://github.com/v8/web-tooling-benchmark)测试的一部分。

#### 编译 Output 配置项

Babel 已经对 [preset 和 plugin 配置项](https://babeljs.io/docs/en/next/plugins#plugin-options)有了一段时间的支持了。你可以将需要的插件名称以数组的形式放在一起，然后把配置项传给对应的插件。
```diff
{
  "plugins": [
-   "pluginA",
+   ["pluginA", {
+     // options here
+   }],
  ]
}
```
对于某些插件的 `loose`配置项，我们做了一些改变，同时也为其他插件新增了一些配置项！注意：在使用这些新增的配置项时，可能会发生一些不兼容的情况，你需要明确知道是在做什么，因为在未编译的情况下，这可能会导致一些问题。所以在未编译的情况下，请直接使用 JS 的原生语法。如果可以的话，最好在库里面就用这样的形式。

* 对于 classes，`class A {}`这种形式经过 Babel 7 编译后不再包括 `classCallCheck` helper。
```javascript
class A {}
```
```diff
var A = function A() {
-  _classCallCheck(this, A);
};
```
* 当使用`for-of`只循环一个数组时，那么可以使用这个新的配置项`["transform-for-of", { "assumeArray": true }]`。
```javascript
let elm;
for (elm of array) {
  console.log(elm);
}
```
```javascript
let elm;

for (let _i = 0, _array = array; _i < _array.length; _i++) {
  elm = _array[_i];
  console.log(elm);
}
```

* 在使用`preset-env`时，`transform-typeof-symbol`插件的 `loose`模式将不复存在[#6831](https://github.com/babel/babel/pull/6831)。

我们发现很多库使用了一些新的配置项，所以我们决定默认继续做。请注意，以上默认行为是尽可能兼容规范的，因此不用 Babel 或使用`preset-env`是可以无缝衔接的，而允许较小的编译输出只是为了节省字节（每个项目都可以进行权衡）。当然，我们计划开发更好的文档和工具，以使其配置更容易。

#### 支持 "Pure" 注释

在提交[#6209](https://github.com/babel/babel/pull/6209)之后，Babel 在编译 ES6 类之后会带有 /*#__PURE__*/ 注释，这样做是为了给压缩工具提供一些线索，以便于去掉dead code（注：指逻辑上永远不会执行的代码），比如 Uglify、babel-minify。这些注释也会用于其它的 helper 函数。
```javascript
class C {
  m() {}
}
```
```javascript
var C =
/*#__PURE__*/
function () {
  // ...
}();
```
对于这部分，如果你在压缩和优化方面存在更多更好的想法，望请告知！

### 语法

#### 支持 [TC39 Proposals](https://github.com/tc39/proposals)

在这里我需要重申一下，我们已经[移除了 Stage persets](https://babeljs.io/blog/2018/07/27/removing-babels-stage-presets)，转而希望用户能明确选择需要添加的 proposals (这些 proposals < Stage 4)。

以下是 Babel 支持的一些新语法（提示：以下功能列表的内容是可能增加/删除/延迟的），并且有一些语法已经被 Babel 7 原生支持。

* ES2018: Object Rest Spread (var a = { b, ...c })
* ES2018 (new): Unicode Property Regex
* ES2018 (new): JSON Superset
* ES2015 (new): new.target
* Stage 3 (new): Class Private Instance Fields (class A { #b = 2 })
* Stage 3 (WIP): Static Class Fields, Private Static Methods (class A { static #a() {} })
* Stage 3 (new): Optional Catch Binding try { throw 0 } catch { do() }
* Stage 3 (new): BigInt (syntax only)
* Stage 3: Dynamic Import (import("a"))
* Stage 2 (new): import.meta (syntax only) (import.meta.url)
* Stage 2 (new): Numeric Seperators (1_000)
* Stage 2 (new): function.sent
* Stage 2: export-namespace-from (export * as ns from 'mod'), split from export-extensions
* Stage 2: Decorators. Check below for an update on our progress!
* Stage 1: export-default-from (export v from 'mod'), split from export-extensions
* Stage 1 (new): Optional Chaining (a?.b)
* Stage 1 (new): Logical Assignment Operators (a &&= b; a ||= b)
* Stage 1 (new): Nullish Coalescing Operator (a ?? b)
* Stage 1 (new): Pipeline Operator (a |> b)
* Stage 1 (new): Throw Expressions (() => throw new Error("a"))

> 要能跟踪所有 proposals 是很困难的，因此我们尝试在[babel/proposals](https://github.com/babel/proposals/)中进行追踪。


#### 支持 TypeScript（`@babel/preset-typescript`）

我们与 [TypeScript](https://github.com/Microsoft/TypeScript)团队合作，让 Babel 能使用`@babel/preset-typescript`解析并转换 typescript 的 type 语法，这与之前我们使用`@babel/preset-flow`处理 [Flow](https://flow.org/)的方法类似。

>更多相关细节请移步 TypeScript 团队所写的[文章](For more details check out this post from the TypeScript team!)。

使用`@babel/preset-typescript`之前 (使用 type 语法):
```typescript
interface Person {
  firstName: string;
  lastName: string;
}

function greeter(person : Person) {
  return "Hello, " + person.firstName + " " + person.lastName;
}
```
使用`@babel/preset-typescript`之后 (移除 types语法)：
```javascript
function greeter(person) {
  return "Hello, " + person.firstName + " " + person.lastName;
}
```
Flow 和 TypeScript 都是能使 JavaScript 用户能利用渐进类型的工具，很高兴 Babel 能同时支持它们。我们计划继续分别与 FB 和 Microsoft 团队密切合作，以保持兼容性并支持其新特性。

>这样的集成还很新，因此可能无法完全支持某些语法，欢迎[提交错误也可以提交 PR](https://github.com/babel/babel/labels/area:%20typescript?page=2&q=is%3Aopen+label%3A%22area%3A+typescript%22) 来帮助我们做得更好。

#### 支持JSX Fragment（<>）

早在[React Blog](https://reactjs.org/blog/2017/11/28/react-v16.2.0-fragment-support.html)就提过，Babel 从 v7.0.0-beta.31 就开始支持 JSX Fragment 了。
```JSX
render() {
  return (
    <>
      <ChildA />
      <ChildB />
    </>
  );
}

// output 👇

render() {
  return React.createElement(
    React.Fragment,
    null,
    React.createElement(ChildA, null),
    React.createElement(ChildB, null)
  );
}
```

### Babel Helpers 的改变

> [babel-upgrade PR](https://github.com/babel/babel-upgrade/pull/71) 正在进行时

之前的`@babel/runtime`已经被拆成`@babel/runtime`和`@babel/runtime-corejs2`[（PR）](https://github.com/babel/babel/pull/8266)两部分了，`@babel/runtime`只保留了 Babel's helper 函数，`@babel/runtime-corejs2`保留了 polyfill 中的所有函数（如：`Symbol`，`Promise`）。

Bable 7 可能会在代码中注入一些可以复用的函数，这些被复用的函数我们称之为“helper functions”，这是因为它们可以在模块之间共享。

下面举一个关于编译`class`的例子（不启用 `loose` 模式）：

根据规范，为了调用 class，你是需要通过`new Person()`来完成的，但是如果该 class 被编译成了函数，理论上来讲你是可以直接调用`Person()`，而无需通过 `new`来创建。对于这种情况，我们会在函数里增加运行时检查机制。
```javascript
class Person {}
```
```javascript
function _classCallCheck(instance, Constructor) { if (!(instance instanceof Constructor)) { throw new TypeError("Cannot call a class as a function"); } }

var Person = function Person() {
  _classCallCheck(this, Person);
};
```

通过`@babel/plugin-transform-runtime`和`@babel/runtime`（作为依赖项）, Babel 7 可以将 helper 函数提取出来，然后只需要通过使用 `require`这个模块化函数，使得编译后的 output 更小，如下所示：
```javascript
var _classCallCheck = require("@babel/runtime/helpers/classCallCheck");

var Person = function Person() {
  _classCallCheck(this, Person);
};
```

对于`external-helpers`和`rollup-plugin-babel`也可以通过同样的方式实现，我们正在研究是否能自动执行此操作，在不久的将来请留意在 Babel’s helpers 上面的相关文章。

**相关信息**
* 原文地址：[Babel 7 Released](https://babeljs.io/blog/2018/08/27/7.0.0) 
* 译文出自：TWNTF
* 译者：Yuqing Xia
* 时间：2018.09.13