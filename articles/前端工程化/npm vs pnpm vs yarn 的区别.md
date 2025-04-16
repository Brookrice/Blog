# npm vs pnpm vs yarn 的区别

## npm - 先锋

很多人认为 npm 是 node package manager 的缩写，其实不是，而且 npm 根本也不是任何短语的缩写,它的前身其实是名为 pm（pkgmakeinst） 的 bash 工具，它可以在各种平台上安装各种东西。
硬要说缩写的话，也应该是 node pm 或者 new pm。

## 嵌套的 node_modules 结构

npm 在早期采用的是嵌套的 node_modules 结构，直接依赖会平铺在 node_modules 下，子依赖嵌套在直接依赖的 node_modules 中。

比如项目依赖了A 和 C，而 A 和 C 依赖了不同版本的 B@1.0 和 B@2.0，node_modules 结构如下：

```js
node_modules
├── A@1.0.0
│   └── node_modules
│       └── B@1.0.0
└── C@1.0.0
    └── node_modules
        └── B@2.0.0
```
如果 D 也依赖 B@1.0，会生成如下的嵌套结构：

```js
node_modules
├── A@1.0.0
│   └── node_modules
│       └── B@1.0.0
├── C@1.0.0
│   └── node_modules
│       └── B@2.0.0
└── D@1.0.0
    └── node_modules
        └── B@1.0.0
```
可以看到同版本的 B 分别被 A 和 D 安装了两次。

## 依赖地狱 Dependency Hell

在真实场景下，依赖增多，冗余的包也变多，node_modules 最终会堪比黑洞，很快就能把磁盘占满。而且依赖嵌套的深度也会十分可怕，这个就是依赖地狱。

## 扁平的 node_modules 结构

为了将嵌套的依赖尽量打平，避免过深的依赖树和包冗余，npm v3 将子依赖「提升」(hoist)，采用扁平的 node_modules 结构，子依赖会尽量平铺安装在主依赖项所在的目录中。

```js
node_modules
├── A@1.0.0
├── B@1.0.0
└── C@1.0.0
    └── node_modules
        └── B@2.0.0
```
可以看到 A 的子依赖的 B@1.0 不再放在 A 的 node_modules 下了，而是与 A 同层级。
而 C 依赖的 B@2.0 因为版本号原因还是嵌套在 C 的 node_modules 下。
这样不会造成大量包的重复安装，依赖的层级也不会太深，解决了依赖地狱问题，但也形成了新的问题。

## 幽灵依赖 Phantom dependencies
幽灵依赖是指在 package.json 中未定义的依赖，但项目中依然可以正确地被引用到。
比如上方的示例其实我们只安装了 A 和 C：
```js
{
  "dependencies": {
    "A": "^1.0.0",
    "C": "^1.0.0"
  }
}
```

由于 B 在安装时被提升到了和 A 同样的层级，所以在项目中引用 B 还是能正常工作的。
幽灵依赖是由依赖的声明丢失造成的，如果某天某个版本的 A 依赖不再依赖 B 或者 B 的版本发生了变化，那么就会造成依赖缺失或兼容性问题。

## 不确定性 Non-Determinism
不确定性是指：同样的 package.json 文件，install 依赖后可能不会得到同样的 node_modules 目录结构。
还是之前的例子，A 依赖 B@1.0，C 依赖 B@2.0，依赖安装后究竟应该提升 B 的 1.0 还是 2.0。

```js
node_modules
├── A@1.0.0
├── B@1.0.0
└── C@1.0.0
    └── node_modules
        └── B@2.0.0
```
```js
node_modules
├── A@1.0.0
│   └── node_modules
│       └── B@1.0.0
├── B@2.0.0
└── C@1.0.0
```
取决于用户的安装顺序。

如果有 package.json 变更，本地需要删除 node_modules 重新 install，否则可能会导致生产环境与开发环境 node_modules 结构不同，代码无法正常运行。

## 依赖分身 Doppelgangers

假设继续再安装依赖 B@1.0 的 D 模块和依赖 @B2.0 的 E 模块，此时：
 - A 和 D 依赖 B@1.0
 - C 和 E 依赖 B@2.0
  
以下是提升 B@1.0 的 node_modules 结构：

```js
node_modules
├── A@1.0.0
├── B@1.0.0
├── D@1.0.0
├── C@1.0.0
│   └── node_modules
│       └── B@2.0.0
└── E@1.0.0
    └── node_modules
        └── B@2.0.0
```
可以看到 B@2.0 会被安装两次，实际上无论提升 B@1.0 还是 B@2.0，都会存在重复版本的 B 被安装，这两个重复安装的 B 就叫 doppelgangers。
而且虽然看起来模块 C 和 E 都依赖 B@2.0，但其实引用的不是同一个 B，假设 B 在导出之前做了一些缓存或者副作用，那么使用者的项目就会因此而出错。