title: JavaScript模块管理器
date: 2016-07-10 15:47:41
tags:
- JavaScript
comments: true
categories:
- 模块化
---
# 模块化
时光回溯到2009年，CommonJS规范和NodeJS都还在襁褓之中，离Bower诞生还有三年时间，Ruby还统治着github，CoffeeScript在年末提交了第一个commit……
备受加载顺序，依赖关系折磨的前端开发，开始站起来试图解决日益复杂的前端开发的种种问题，RequireJS降临了。如果说NodeJS吹响了JS全栈革命的号角，那么同时发生的前端模块化革命便是RequireJS的历史使命。
五年过去了，RequireJS战胜了同级生LabJS，带起了中国小伙伴SeaJS。他完美地引领了前端模块化的革命，但今天看来，它有些过时了：它重浏览器端，轻打包编译，没有及时跟进包管理体系，almond没有成为标配而只是周边，配置晦涩……诞生太早的RequireJS，虽然一度成为了前端模块化的某种程度上的事实标准，但难掩其缺点。
为了解决这个问题，前端的模块管理器（package management）应运而生，五年间，NodeJS成为了服务端以及脚本工具的一代翘楚，NPM的成功让大家意识到一个集中式的依赖／包管理体系的重要性，Bower应运而生，还有试图将CMD和NPM包带到前端领域，统一前后端包格式的Browserify等等，大量的前端工具爆发式地出现，WebPack是其中的(又)一款模块打包工具。

<!--more-->
require.js一类的模块管理器的问题在于各种参数设置过于繁琐，不容易学习，很难完全掌握。而且，实际应用中，往往还需要在服务器端，将所有模块合并后，再统一加载，这多出了很多工作量。
* seajs / require : 是一种在线"编译" 模块的方案，相当于在页面上加载一个 CMD/AMD 解释器。这样浏览器就认识了 define、exports、module 这些东西。也就实现了模块化。
* browserify / webpack / Duo : 是一个预编译模块的方案，相比于上面 ，这个方案更加智能。没用过browserify，这里以webpack为例。首先，它是预编译的，不需要在浏览器中加载解释器。另外，你在本地直接写JS，不管是 AMD / CMD / ES6 风格的模块化，它都能认识，并且编译成浏览器认识的JS。
## Bower
[Bower](https://bower.io/)的主要作用是，为模块的安装、升级和删除，提供一种统一的、可维护的管理模式。
Bower是一个客户端技术的软件包管理器，它可用于搜索、安装和卸载如JavaScript、HTML、CSS之类的网络资源。其他一些建立在Bower基础之上的开发工具，如YeoMan和Grunt，这个会在以后的文章中介绍。
首先，安装Bower。
```
npm install -g bower
```
然后，使用`bower install`命令安装各种模块。下面是一些例子。
```
# 模块的名称
$ bower install jquery
# github用户名/项目名
$ bower install jquery/jquery
# git代码仓库地址
$ bower install git://github.com/user/package.git
# 模块网址
$ bower install http://example.com/script.js
```
所谓"安装"，就是将该模块（以及其依赖的模块）下载到当前目录的bower_components子目录中。下载后，就可以直接插入网页。
```
<script src="/bower_componets/jquery/dist/jquery.min.js">
```
`bower update`命令用于更新模块。如果不给出模块的名称，则更新所有模块。
```
bower update jquery
```
`bower uninstall`命令用于卸载模块。
```
bower uninstall jquery
```
注意，默认情况下，会连所依赖的模块一起卸载。比如，如果卸载jquery-ui，会连jquery一起卸载，除非还有别的模块依赖jquery。

(bower简明入门教程)[https://segmentfault.com/a/1190000000349555]

##  Duo
(Duo)[http://duojs.org/]是在Component的基础上开发的，语法和配置文件基本通用，并且借鉴了Browserify和Go语言的一些特点，相当地强大和好用。
首先，安装Duo。
```
npm install -g duo
```
然后，编写一个本地文件index.js。
```
var uid = require('matthewmueller/uid');
var fmt = require('yields/fmt');
　　
var msg = fmt('Your unique ID is %s!', uid());
window.alert(msg);
```
上面代码加载了uid和fmt两个模块，采用Component的"github用户名/项目名"格式。
接着，编译最终的脚本文件。
```
duo index.js > build.js
```
编译后的文件可以直接插入网页。
```
<script src="build.js"></script>
```
Duo不仅可以编译JavaScript，还可以编译CSS。
```
@import 'necolas/normalize.css';
@import './layout/layout.css';
　　
body {
  color: teal;
  background: url('./background-image.jpg');
}
```
编译时，Duo自动将normalize.css和layout.css，与当前样式表合并成同一个文件。
```
duo index.css > build.css
```
编译后，插入网页即可。
```
<link rel="stylesheet" href="build.css">
```
[Duo js 一个非常酷的前端打包工具](http://www.ifeenan.com/nodejs/2014-08-24-Duo%20JS%20%E4%B8%80%E4%B8%AA%E9%9D%9E%E5%B8%B8%E9%85%B7%E7%9A%84%E5%89%8D%E7%AB%AF%E6%89%93%E5%8C%85%E5%B7%A5%E5%85%B7/)

## Browserify
Browserify本身不是模块管理器，只是让服务器端的`CommonJS`格式的模块可以运行在浏览器端。这意味着通过它，我们可以使用`Node.js的npm模块管理器`。所以，实际上，它等于间接为浏览器提供了npm的功能。
首先，安装Browserify。
```
npm install -g browserify
```
然后，编写一个服务器端脚本。
```
var uniq = require('uniq');
var nums = [ 5, 2, 1, 3, 2, 5, 4, 2, 0, 1 ];
console.log(uniq(nums));
```
上面代码中uniq模块是CommonJS格式，无法在浏览器中运行。这时，Browserify就登场了，将上面代码编译为浏览器脚本。
```
browserify robot.js > bundle.js
```
生成的bundle.js可以直接插入网页。
<script src="bundle.js"></script>
Browserify编译的时候，会将脚本所依赖的模块一起编译进去。这意味着，它可以将多个模块合并成一个文件。

## webpack
web开发中常用到的静态资源主要有JavaScript、CSS、图片、Jade等文件，webpack中将静态资源文件称之为模块。
webpack是一个module bundler(模块打包工具)，其可以兼容多种js书写规范，且可以处理模块间的依赖关系，具有更强大的js模块化的功能。Webpack对它们进行统一的管理以及打包发布，其官方主页用下面这张图来说明Webpack的作用：
{% asset_img webpack.png %}
### webpack 好处：
0. 模块来源广泛，支持包括npm/bower等等的各种主流模块安装／依赖解决方案
1. 对 CommonJS 、 AMD 、ES6的语法做了兼容
2. 对js、css、图片等资源文件都支持打包
3. 串联式模块加载器以及插件机制，让其具有更好的灵活性和扩展性，例如提供对CoffeeScript、ES6的支持
4. 有独立的配置文件webpack.config.js
5. 可以将代码切割成不同的chunk，实现按需加载，降低了初始化时间
6. 支持 SourceUrls 和 SourceMaps，易于调试
7. 具有强大的Plugin接口，大多是内部插件，使用起来比较灵活
8. webpack 使用异步 IO 并具有多级缓存。这使得 webpack 很快且在增量编译上更加快

### 有了gulp和webpack，还需要bower吗？
webpack是一个比browserify功能更强大的模块加载器。既然是模块加载器，当然就包括对各种各样模块的加载，包括SASS/LESS/CoffeeScript/png/jpg等，以及webpack对于node_module模块加载已经非常完善了。
那么，为什么还需要bower呢？由于前端开发很多第三方模块并非都以标准npm包形式存在，有一些非主流，或者各种原因没放到npm上的包，可以在bower找到。
基于这个原因，使用webpack时候，凭着能用npm就用（依赖加载更加方便，功能更加强大），不能用的时候使用bower声明第三方模块依赖，然后使用js/css加载方式进行加载。
值得一提的是，webpack官方也提供非常便利的方式加载bower模块（模块的主要文件，被声明在bower.json main属性里面）,通过配置后就可以很方便地沿用require来加载bower模块。

### 为什么很多人喜欢gulp+webpack，而不直接使用webpack？
一个例子，webpack没有雪碧图功能，可能还有其他，但是这是我遇到的，需要配合gulp完成的

[详解前端模块化工具-Webpack](https://segmentfault.com/a/1190000003970448)
[gulp与webpack的迷思](https://segmentfault.com/a/1190000004249679)

> 阮一峰：http://www.ruanyifeng.com/blog/2014/09/package-management.html