## 译：如何更新package以及更新规则

**译者注**：上一篇主要讲解了如何在npm上发布你的package，下面我们继续讲解如何更新以及一些更新规范。

### 如何语义化版本

在发布一个新的代码版本时，描述修改范围和程度是很重要的，这是因为当其他工程将该package作为依赖时，package的修改很有可能会引起该工程出错。于是，语义化版本（简称：semver）的提出，定义了一种标准去解决这样的问题。

#### 开发者而言

如果一些package项目需要发布出来供他人使用时，一般需要从`1.0.0`版本开始，尽管还是有一些npm上的项目没有遵循这样的规范。
发布之前，对于开发者而言，项目的更新版本需要按照如下标准进行：

| 代码状态 | 阶段 | 规则 | 例子 |
| :---: | :---: | :---: | :---: |
| 首发版本 | 新产品 | 以1.0.0作为初始版本 | 1.0.0 |
| bug修复，非feature级的小步改动 | 补丁发布 | 增长第三位数字 | 1.0.1 |
| 新的feature，并且不会影响已有的feature | 小版本发布 | 增长第二位数字 | 1.1.0 |
| 修改会破坏向后兼容性 | 重大发布 | 增长第一位数字 | 2.0.0 |

#### 使用者而言

作为使用者， 你可以在`package.json`文件中指定你的项目可以接受的更新版本。
例如，如果你的项目一开始引用该依赖的版本是`1.0.4`， 那么根据不同的版本更新阶段，你可以指定的依赖更新范围如下所示：

+ 补丁发布：1.0 、1.0.x 、~1.0.4
+ 小版本发布：1 、1.x 、^1.0.4
+ 重大发布：* 、x


### 如何更新package

#### 版本更新

当你对package做了修改，你可以通过以下方式更新版本：
```
npm version <update_type>
```
其中`<update_type>`是按照语义化版本规范得到的发布版本类型，包括：patch，minor，major，分别对应着补丁，小版本，以及重大版本。

同时，该命令会改变`package.json`文件中的	`version`	参数。

*注意：如果你的 git仓库与该 package进行了关联，那么这个命令也会在你的 git仓库里添加一条包含有更新版本号的 tag。*

完成版本更新后，运行`npm publish`即可完成package的更新。

**验证：** 访问`https://npmjs.com/package/<package>`	这种形式的URL，检查其版本是否有更新。

#### 更新 README 文件
只有在你的package有新版本发布时，才会更新`README`文件在package主页上显示的内容，所以你需要通过运行`npm version patch`和`npm publish`去更新文档在主页上的显示内容。


### 更多

访问这个URL [npm semver calculator](https://semver.npmjs.com/)，这个工具能帮你更好的了解npm的语义化规范是如何工作的。

更多关于`package.json`文件中的依赖版本管理，可以参考[working with package.json](https://docs.npmjs.com/getting-started/using-a-package.json#specifying-packages)。

对于使用其他方式标注发布版本，可以参考[dist-tag](https://docs.npmjs.com/cli/dist-tag)和[how they relate to semantic versioning](https://docs.npmjs.com/getting-started/using-tags)


**相关信息**
* 原文地址：
    * [How to Update a Package](https://docs.npmjs.com/getting-started/publishing-npm-packages#how-to-update-a-package)
    * [How to use Semantic Versioning](https://docs.npmjs.com/getting-started/semantic-versioning)
* 译文出自：TWNTF
* 译者：Yuqing Xia
* 时间：2018.08.14


