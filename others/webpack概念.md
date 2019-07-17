#### 概念

**module**

提供比完整程序接触面(surface area)更小的离散功能块。精心编写的模块提供了可靠的抽象和封装界限，使得应用程序中每个模块都具有条理清楚的设计和明确的目的。

对于一份同逻辑的代码，当我们手写下一个一个的文件，它们无论是 ESM 还是 commonJS 或是 AMD，他们都是module 。

**chunk**

这是 webpack 特定的术语被用在内部来管理 building 过程。bundle 由 chunk 组成，其中有几种类型（例如，入口 chunk(entry chunk) 和子 chunk(child chunk)）。通常 chunk 会直接对应所输出的 bundle，但是有一些配置并不会产生一对一的关系。

当我们写的 module 源文件传到 webpack 进行打包时，webpack 会根据==文件引用关系==生成 **chunk** 文件，webpack 会对这个 chunk 文件进行一些操作；

**bundle**

由多个不同的模块生成，bundles 包含了早已经过加载和编译的最终源文件版本。

webpack 处理好 chunk 文件后，最后会输出 **bundle** 文件，这个 bundle 文件包含了经过加载和编译的最终源文件，所以它可以直接在浏览器中运行。

> 我们直接写出来的是 module，webpack 处理时是 chunk，最后生成浏览器可以直接运行的 bundle。

**bundle分离（bundle splitting）**

这个流程提供一个优化 build 的方法，允许 webpack 为应用程序生成多个 bundle。最终效果是，当其他某些 bundle 的改动时，彼此独立的另一些 bundle 都可以不受到影响，减少需要重新发布的代码量，因此由客户端重新下载并利用浏览器缓存。

**代码分离（code splitting）**

指将代码分离到每个 bundles/chunks 里面，你可以按需加载，而不是加载一个包含全部的 bundle。



**filename**

filename 是一个很常见的配置，就是对应于 `entry` 里面的输入文件，经过webpack 打包后输出文件的文件名。比如说经过下面的配置，生成出来的文件名为 `index.min.js`。

```js
{
    entry: {
        index: "../src/index.js"
    },
    output: {
        filename: "[name].min.js", // index.min.js
    }
}
```

**chunkFilename**

`chunkFilename` 指未被列在 `entry` 中，却又需要被打包出来的 `chunk` 文件的名称。一般来说，这个 `chunk` 文件指的就是要**懒加载**的代码。

```javascript
{
    entry: {
        index: "../src/index.js"
    },
    output: {
        filename: "[name].min.js",  // index.min.js
        chunkFilename: 'bundle.js', // bundle.js
    }
}
```



-----------------------------------------------------------------------------------------------------------------------------------------------------------

首先来个背景介绍，哈希一般是结合 CDN 缓存来使用的。如果文件内容改变的话，那么对应文件哈希值也会改变，对应的 HTML 引用的 URL 地址也会改变，触发 CDN 服务器从源服务器上拉取对应数据，进而更新本地缓存。

**hash**

hash 计算是跟整个项目的构建相关, 项目如果有一些变动， hash一定会变。这样会影响CDN和浏览器的缓存策略。

**chunkhash**

chunkhash 就是解决 hash 值相同的问题，它根据不同的入口文件(Entry)进行依赖文件解析、构建对应的 chunk，生成对应的哈希值。

**contenthash**

将根据资源的内容创建出唯一hash， 内容不变， hash就不变

因为使用 chunkhash 时 index.js  和  index.css  同为一个 chunk，如果  index.js 内容发生变化，但是 index.css 没有变化，打包后他们的 hash 都发生变化，这对 css 文件来说是一种浪费。

**hash值的区别**

hash：以项目为维度生成的hash值，项目全部文件都共用一个hash值 chunkhash： 以chunk为维度生成的hash值，不同入口生成不同的chunkhash值 contenthash： 根据资源内容生成的hash值 一般是用chunkhash，contenthash也有使用场合，比如在mini-css-extract-plugin插件配置使用。

**loader**

webpack 只能理解 JavaScript 和 JSON 文件。loader 让 webpack 能够去处理其他类型的文件，并将它们转换为有效模块。

