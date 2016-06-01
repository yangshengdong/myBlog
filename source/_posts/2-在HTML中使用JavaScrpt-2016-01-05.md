title: 第二章 在HTML中使用JavaScrpt
date: 2016-01-05 14:25:13
tags:
- JavaScript
comments: true
categories:
- 《JS高程3-笔记》
---
## script元素

`<script src="demo.js"></script>`尽管`<script>` 标签内没有内容，结束的 `</script>` 标签也是必需的。外部文件一般扩展名为.js，但这不是强制的，不写.js扩展名一样可以运行。
<!--more-->
* type 这个属性不是必须的，默认值是 “text /javascript”，表示的是编写代码使用的脚本语言的内容类型（也称为MIME 类型）。服务器在传送js文件时使用的MIME类型，通常是application/x-javascript，但在type中设置这个值可能会导致脚本被忽略，考虑到约定俗成和最大浏览器兼容性，目前type属性的值依旧还是text/javascript。
*  async  只适用外部引入脚本。
*  defer 只适用外部引入脚本。
*  language 已废弃。
*  src

>1. 如果通过&lt;script&gt;&lt;/script&gt;向页面写入一段可以执行的js代码？。
>
        <script>
            document.write('&lt;script&gt;alert(0)&lt;/script&gt;');//alert(0); 不执行
            document.write('<script>alert(4)</scr'+'ipt>'); //正常弹窗
            document.write('<script>alert(2)<\/script>'); //正常弹窗
            document.write('<script>alert(3)</script>'); //报错
        </script>

>2. 执行顺序：
>无论如何包含代码，只要不存在defer 和async 属性，浏览器都会按照`<script>`元素在页面中出现的先后顺序对它们依次进行解析。换句话说，在第一个`<script>`元素包含的代码解析完成后，第二个`<script>`包含的代码才会被解析，然后才是第三个、第四个……

## 标签的位置

    <!DOCTYPE html>
    <html>
        <head>
            <title>Example HTML Page</title>
        </head>
        <body>
            <!-- 这里放内容 -->
            <script type="text/javascript" src="example1.js"></script>
            <script type="text/javascript" src="example2.js"></script>
        </body>
    </html>
## 延迟脚本 defer

defer这个属性的用途是表明脚本在执行时不会影响页面的构造。也就是说，脚本会被延迟到整个页面都解析完毕(`/HTML`)后再运行。因此，在`<script>`元素中设置defer 属性，相当于告诉浏览器立即下载，但延迟执行。
>**注意：**
>多个延迟脚本并不一定会按照顺序执行，因此最好只包含一个延迟脚本。
>**浏览器支持情况：**
IE4、Firefox 3.5、Safari 5 和Chrome ，其他浏览器会忽略这个属性。为此，把延迟脚本放在页面底部仍然是最佳选择。
>**defer执行的脚本都会在DOMContentLoaded之前就执行，defer会阻塞页面加载**

## 异步脚本 async

HTML5 为`<script>`元素定义了async 属性。指定async 属性的目的是不让页面等待两个脚本下载和执行，从而异步加载页面其他内容。为此，建议异步脚本不要在加载期间修改DOM。
>**注意：**
>多个延迟脚本并不一定会按照顺序执行，因此最好只包含一个延迟脚本。
>**浏览器支持情况：**
- Firefox 3.6、Safari 5 和Chrome。
- ie系列，async没有任何效果
- 在chrome下，只有外联脚本，且是在body中引用的，才能生效.
- **异步执行的表现是，在DOMContentLoaded事件之后，window.loaded事件之前，所以，这个属性处理阻塞的问题是可行的**

## Defer和async的区别

先来试个一句话解释仨，当浏览器碰到 script 脚本的时候：
+ `<script src="script.js"></script>` 没有 defer 或 async，浏览器会立即加载并执行指定的脚本，“立即”指的是在渲染该 script 标签之下的文档元素之前，也就是说不等待后续载入的文档元素，读到就加载并执行。
+ `<script async src="script.js"></script>` 有 async，加载和渲染后续文档元素的过程将和 script.js 的加载与执行并行进行（异步）。
+ `<script defer src="myscript.js"></script>` 有 defer，加载后续文档元素的过程将和 script.js 的加载并行进行（异步），但是 script.js 的执行要在所有元素解析完成之后，DOMContentLoaded 事件触发之前完成。

然后从实用角度来说呢，首先把所有脚本都丢到 `</body>` 之前是最佳实践，因为对于旧浏览器来说这是唯一的优化选择，此法可保证非脚本的其他一切元素能够以最快的速度得到加载和解析。
接着，我们来看一张图咯：

{% asset_img deferAsync.png %}

**此图告诉我们以下几个要点：**
defer 和 async 在网络读取（下载）这块儿是一样的，都是异步的（相较于 HTML 解析）。它俩的差别在于脚本下载完之后何时执行，显然 defer 是最接近我们对于应用脚本加载和执行的要求的，关于 defer，此图未尽之处在于它是按照加载顺序执行脚本的，这一点要善加利用。async 则是一个乱序执行的主，反正对它来说脚本的加载和执行是紧紧挨着的，所以不管你声明的顺序如何，只要它加载完了就会立刻执行仔细想想，async 对于应用脚本的用处不大，因为它完全不考虑依赖（哪怕是最低级的顺序执行），不过它对于那些可以不依赖任何脚本或不被任何脚本依赖的脚本来说却是非常合适的，最典型的例子：Google Analytics。

### Defer async常规表现（一些高级浏览器）

#### herder
1. header中行内脚本执行顺序不受defer async影响，顺序执行，会阻塞DOMContentLoaded。
2. header中引用外部脚本，添加defer async后，浏览器表现情况不统一，async的可能先执行，所以引用外部脚本并不适合加在header中，也不适合添加defer async标示。
 
#### body
1. body中图片加载会阻塞window.loaded,不会阻塞DOMContentLoaded。
2. body中行内脚本执行顺序不受defer async影响，顺序执行，阻塞DOMContentLoaded。
3. body中引用外部脚本，defer async表现正常，外部脚本应该加在body中，body结束标签上面。

#### ajax
1. 无论是header还是body中，行内脚本执行的ajax还是外部脚本执行的ajax，都对页面加载没有影响。

#### 总结
1. defer执行的脚本都会在DOMContentLoaded之前就执行，defer会阻塞页面加载。
2. async脚本都会在loaded之前执行，它会阻塞window.loaded。
3. DOMContentLoaded在window.loaded之前执行，阻塞DOMContentLoaded也就会阻塞window.loaded
4. document ready在DOMContentLoaded之前执行，说明document ready是监听DOMContentLoaded完成的
 
### IE

1. IE支持defer属性,不支持async属性，从IE9及以上支持onload,支持DOMContentLoaded。
2. IE6，7支持行内脚本defer属性， 从表现上来看IE6,7,8,9都支持行内脚本的defer，所以我们在ie6,7,8,9中观察到的现象是，行内的先执行async,再执行没加defer async标记的，defer的延迟执行了。
3. 同时我们又发现IE6,7脚本中ajax影响了页面加载，影响document ready,IE8及以上版本不受影响。
4. 到了IE8以上，表现和webkit内核浏览器基本相似了。

**不是动态添加的脚本，都会阻塞页DOMContentLoaded，动态添加的脚本应该在document ready后加载，但是也会阻塞loaded**
>不懂DOMContentLoaded的点[这里](http://www.yangshengdonghome.com/2016/01/08/DOMContentLoaded/)。

## 嵌入代码与外部文件

* 推荐通过`<script>`标签引入外部js文件。
* 浏览器能够根据具体的设置缓存链接的所有外部JavaScript 文件。也就是说，如果有两个页面都使用同一个文件，那么这个文件只需下载一次。因此，最终结果就是能够加快页面加载的
速度。

## 文档模式 `<!DOCTYPE *>`

E5.5 引入了文档模式的概念，而这个概念是通过使用文档类型（doctype）切换实现的。
1. 混杂模式（quirks mode）
2. 标准模式（standards mode）
3. 准标准模式（almost standards mode）

如果在文档开始处没有发现文档类型声明，则所有浏览器都会默认开启混杂模式。
>现在所有的HTML文档都推荐使用HTML5规定的`<!DOCTYPE html>`

## noscript

包含在`<noscript></noscript>`元素中的内容只有在下列情况下才会显示出来：
* 浏览器不支持脚本；
* 浏览器支持脚本，但脚本被禁用。
