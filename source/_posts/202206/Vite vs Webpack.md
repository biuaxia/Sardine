title: Vite vs Webpack
date: 2022-06-06 10:28:00
toc: false
index_img: https://puep.qpic.cn/coral/Q3auHgzwzM4fgQ41VTF2rF1YCFdibQ9cyYktsNibicoKDA3PQPcYXddjA/0
category:
- 前端
tags:
- 项目
- 文件
- 用户
- 服务
- 解决
- 出现
- 打印
---

## Vite 为什么快？

大家在使用 `webpack` 去进行去进行项目构建时，应该都遇到过这样一种情况。

> 假设我们的项目中有 A、B 两个页面。
>
> 其中 A 页面是项目首页，里面的代码一切正常。
>
> B页面是一个需要经过跳转才会进入的页面，里面存在一些错误。比如：我导入一个不存在的文件 `a.js` 然后打印 `a`
> 
> 当我们去构建这个项目时，明明我们从来都没有进入过 B 页面，但是此时 webpack 依然会给我们抛出一个对应的错误 `Can't resolve './a.js' in xxx`

出现这个问题的原因就是因为 `webpack` 的构建机制导致的。

`webpack` 在开发时构建时，默认会去 **抓取并构建你的整个应用，然后才能提供服务** ，这就导致你的项目中，存在的任何一个错误（哪怕这个错误是在用户从来都没有进入过的页面中出现的），它依然会影响到你的整个项目构建。

也正是因为这个原因，**当你的项目越大时，构建的时间就会越长** ，你的项目启动速度也就会越慢。

---

但是对于 `vite` 而言，我们进行同样的测试，会发现一个很有意思的事情。

> 同样的 `Can't resolve './a.js' in xxx` 错误，在我们没有进入到 `B` 页面的时候，它是不会出现的，只有当我们进入了 `B` 页面，才会突然出现这样的一个错误。

而之所以会这样的原因就是因为： **`vite` 不会在一开始就构建你的整个项目** ，而是会将应用中的模块区分为 **依赖** 和 **源码（项目代码）** 两部分，对于 **源码** 部分，它会根据 **路由来拆分** 代码模块，只会去构建一开始就必须要构建的内容。

同时 `vite` 以 [原生 ESM](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules) 的方式为浏览器提供源码，让浏览器接管了 **打包** 的部分工作。

因为这样的一个机制，无论你的项目有多大，它只会构建一开始必须要构建的内容，这就让 `vite` 在构建时的速度大大提升了。

这也是 `vite` 为什么会快的一个核心原因。

## Vite 这种机制会存在问题吗？

如果大家对 `ESM` 的构建机制有了解的话，那么应该可以发现一个问题。

那就是 **`vite` 既然以 [原生 ESM](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules) 的方式为浏览器提供源码，让浏览器接管了打包的部分工作** ，那么假如我们的项目中存在 `commonJS` 的内容怎么办？是不是就意味着无法解析呢？

**是的！**

在 `vite` 的早期版本中，确实存在这个问题，这个问题导致的最核心的麻烦就是很多的 **依赖** 无法使用。

比如 `axios` 因为 `axios` 中使用了很多的 `commonJS` 规范，这就让 `vite` 无法解析对应的内容（对应的 [issue](https://github.com/axios/axios/issues/1879)），从而会抛出一个错误，关于这个问题曾经也在 [vite的issues](https://github.com/vitejs/vite/issues/162) 中进行过激烈的讨论。

尤大也对此进行了发言：

![](https://puep.qpic.cn/coral/Q3auHgzwzM4fgQ41VTF2rF1YCFdibQ9cyYktsNibicoKDA3PQPcYXddjA/0)

那么这样的问题最终是怎么解决的呢？

## 官方如何解决的这种问题？

因为这个问题非常的严重，所以针对于这个问题，`vite` 在后期提供了 [依赖预构建](https://cn.vitejs.dev/guide/dep-pre-bundling.html) 的功能，其中一个非常重要的目的就是为了解决 **CommonJS 和 UMD 兼容性** 问题。目前 `vite` 会先将 `CommonJS 或 UMD 发布的依赖项转换为 ESM` 之后，再重新进行编译。这也可以理解为  **速度对业务的一个妥协** 。

