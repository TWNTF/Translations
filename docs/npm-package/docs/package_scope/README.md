## 译：Package 的域名

以下文档需要 npm 版本不低于 2。

域（scopes）常用于对相关联的 package 进行分组，并创建命名空间去管理，就像是为这些 package 创建了一个域。更多关于 npm 域的定义可以查看[npm scopes](https://docs.npmjs.com/misc/scope)。

如果一个 package 的包名以`@`符号开头，就说明这是一个被界定域名的 package ，称为`scoped package`，这个 scope 定义在`@`和`/`中间。例如，

```
@scope/project-name
```

每一个npm用户都能拥有自己的 scope ，例如，

```
@username/project-name
```

npm组织也会拥有组织的 scope ，例如，
```
@orgname/project-name
```

更多关于 npm 域的信息可以参考[CLI documentation](https://docs.npmjs.com/misc/scope#publishing-public-scoped-packages-to-the-public-npm-registry)。

### 初始化 scoped package

创建一个 scoped package ，你可以简单通过命名包名实现，例如，

```json
{
	"name": "@username/project-name"
}
```

如果你使用`npm init`进行初始化，那么你可以通过以下命令设置域名，例如，

```
npm init --scope=username
```

如果你需要多次使用同一个域名，你可与将其作为参数添加到`.npmrc`文件中，例如，

```
npm config set scope username
```

### 发布 scoped package

默认情况下，scoped package 是私有的，如果想要发布私有的包，你需要是付费用户。
但是，发布公有的 scoped package 是免费的，在发布公有 scoped package 时需要设置命令中的参数，后续发布也需要保留这个参数，如下，

```
npm publish --access=public
```

### 使用 scoped package

引用 scoped package 时，需要将 scope 添加到引用的包名中，例如在`package.json`文件中，
```json
{
  "dependencies": {
    "@username/project-name": "^1.0.0"
  }
}
```
添加以及引用，如下，

```
npm install @username/project-name --save
var projectName = require("@username/project-name")
```

更多的关于私有 scoped package 的信息，可以访问[private module](https://docs.npmjs.com/private-modules/intro)


**相关信息**
* 原文地址：[How to Work with Scoped Packages](https://docs.npmjs.com/getting-started/scoped-packages)
* 译文出自：TWNTF
* 译者：Yuqing Xia
* 时间：2018.08.16