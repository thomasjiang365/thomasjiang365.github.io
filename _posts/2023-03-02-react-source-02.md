---
title: 'React 18源码学习篇02 调试'
date: 2023-03-02
permalink: /posts/2023/03/react-source-02/
tags:
  - react
  - web
share: false
related: false
---

> 在对React整体架构有了基本认识的基础上，学习React源码最直接有效的方式就是亲自调试代码，在关键的代码调用处打断点、查看变量赋值以及函数调用栈等等。而常见的调试工具包括不限于**Chrome Sources**、 **VS Code**。根据React源码构建的位置，调试由易到难大致可以划分为三个思路：
>
> 1. 调试npm registry中的React构建产物
> 2. 调试本地构建的React产物
> 3. 调试React源码和项目代码共同构建的产物

## 调试npm registry中的React构建产物

通过`create-react-app`快速创建**react_debug**项目，创建完成的项目默认安装了所有依赖项，包括`react`和`react-dom`的最新版本。执行`start`命令后浏览器弹出初始页面，可在`Sources`选项卡中对`react.development.js`、`react-dom.development.js`和`scheduler.development.js`进行断点调试。

<img src="/images/blog/react-source/02/react_debug_01.png" alt="react_debug_01" style="zoom:100%;" />


### 操作步骤
1. 初始化**react_debug**项目并运行
```shell
npx create-react-app react_debug
npm run start
```
此时项目目录结构：
<img src="/images/blog/react-source/02/react_debug_structure_01.png" alt="react_debug_structure_01" style="zoom:100%;" />
2. 浏览器(以Chrome为例)自动弹出页面，打开调试面板并在需要调试的函数如`performUnitOfWork`处增加断点，如下：
<img src="/images/blog/react-source/02/react_debug_chrome_01.png" alt="react_debug_chrome_01" style="zoom:100%;" />

### 总结
这是最简单、省时的调试React源码的方法，几乎不需要额外的配置或其他手动修改源码的操作。在`Sources`选项卡中打开要调试的文件并全局搜索需要调试的函数进行断点基本可以满足追踪函数调用栈的需求。这种调试方式的缺点也很明显，1)不能修改源码调试，比如增加日志 2)无法定位到源码的具体文件。对于React源码的初学者推荐用这种方式进行调试，一来可以快速进入、学习源码的具体逻辑，二来降低”魔改“源码过程中产生大量报错产生的挫败感。

## 调试本地构建的React产物

在第一种调试方法中，`react`、`react-dom`是React官方构建并发布在`npm registry`的包，我们无法进行修改调试。而这个过程可以通过**本地发包**或者**链接**的方式进行替换，源码在修改后重新本地发包即可。
<img src="/images/blog/react-source/02/react_debug_02_fix.png" alt="react_debug_02_fix" style="zoom:100%;" />

### 操作步骤
在这里，我们通过`yalc`本地发包举例。
1. fork React的源码到自己的仓库方便修改，拉取到本地
```shell
git clone git@github.com:[yourname]/react.git
```
2. 进入react源码的目录并安装依赖
```shell
yarn install
```
3. 构建测试环境产物
```shell
yarn build-for-devtools-dev
```
此时react源码目录下新增`build`文件夹，如下<br>
<img src="/images/blog/react-source/02/react_build.png" alt="react_build" style="zoom:100%;" />

4. 切换到需要本地发包的包路径下执行本地发包
```shell
yalc publish
```
5. 切换到`react_debug`目录下，执行
```shell
yalc add react reac-dom
```
此时`package.json`会变为
```json
-- "react": "^18.2.0",
-- "react-dom": "^18.2.0",
++ "react": "file:.yalc/react",
++ "react-dom": "file:.yalc/react-dom",
```
`react_debug`目录变为
<img src="/images/blog/react-source/02/react_debug_structure_02.png" alt="react_debug_structure_02" style="zoom:100%;" /><br>
至此`react_debug`项目下的`react`和`react-dom`包都已切换为本地发布到`yalc`中的包。
如果想要修改源码比如在代码中增加日志，可以重新构建react源码，并推送更新到`react_debug`
6. 修改入口函数，增加打印
<img src="/images/blog/react-source/02/react_add_log.png" alt="react_add_log" style="zoom:100%;" />
7. 重新构建产物
```shell
yarn build-for-devtools-dev
```
8. 切换到需要本地发包的包路径下执行本地发包、推送
```shell
yalc push
```
9. 切换到`react_debug`目录下，删除依赖并重新安装后执行
```shell
yarn start
```
在浏览器中可以看到控制台已经成功打印我们添加的日志。
<img src="/images/blog/react-source/02/react_debug_console_02.png" alt="react_debug_console_02" style="zoom:100%;" />

### 总结

相较于第一种调试方法，本地发包的方式在步骤上显得更加繁琐也更加耗时，并且调试的仍然是`*.development.js` 无法与源码中的文件对应，好处则是可以灵活的修改源码。除了使用`yalc`，`npm link`也可以达到同样的目的，这里不再演示。

## 调试React源码和项目代码共同构建的产物

前两种方式的关注点更聚焦在react作为一个第三方库，与`react_debug`是互相独立的关系。而这种方式将react源码作为`react_debug`项目的一部分，与`react_debug`项目共同进行打包。在构建配置项中开启`sourcemap`选项后就可以在浏览器`Sources`选项卡下打开react源码中对应的文件。
<img src="/images/blog/react-source/02/react_debug_03.png" alt="react_debug_03" style="zoom:100%;" />

### 操作步骤

具体操作步骤可参考[^1]。(参考中react源码版本为18.1.0，本文选用18.2.0，在eslint某些选项有所不同，可根据报错信息自行关闭对应规则校验)

具体效果如图所示:
<img src="/images/blog/react-source/02/react_debug_chrome_03.png" alt="react_debug_chrome_03" style="zoom:100%;" />

### 总结

这种调试方法基本解决了源码无法修改以及无法直接定位到源码具体文件的问题。对于react源码有一定的认识后，推荐用这种方式进行调试，更灵活且构建速度更快。

## 全文总结

作为一个第三方库，调试react在本质上并没有和调试组件库、工具库有什么区别，关键在于我们用什么方式将需要调试的库引入到项目中。另外需要注意的是，我们项目中导入的第三方库大部分都是经过`rollup`、`webpack`等等工具构建过之后的产物，甚至经过混淆、压缩之后代码已经完全不可读。在`dev`模式下，我们可以看到，react源码的构建产物比如`react.development.js`、`react-dom.development.js`等等仍然具有较好的可读性，所以除了特别需要修改react源码的需要，推荐使用第一种调试方法。其他调试方法可参考[^2][^3]。

## 参考

[^1]: [React18 源码解析之搭建调试环境](https://www.xiabingbao.com/post/react/debug-react-source-rfkxi0.html)
[^2]: [全网最优雅的 React 源码调试方式](https://juejin.cn/post/7126501202866470949)
[^3]: [另辟蹊径搭建阅读React源码调试环境-支持所有React版本细分文件断点调试](https://terry-su.github.io/cn/debug-react-source-code-using-special-method)