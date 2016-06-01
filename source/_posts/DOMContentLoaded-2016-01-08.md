title: onload vs DOMContentLoaded
date: 2016-01-08 10:27:11
tags:
- jQuery
- JavaScript
comments: true
categories:
- JavaScript
---
* `$(document).ready(function () { });`
* `$(function () { });`

以上两行代码的目的和效果都一样———待DOM加载完成之后，执行传入的function函数。

这是我们在页面初始化时经常使用的监听方案，那么他的实际的执行关系时什么样的呢？
在原生js中是什么样的一种表现？
<!--more-->
## 定义
+ onload:
当onload事件触发的时候，页面上的所有dom，样式表，脚本，图片，flash，iframe都已经加载完成了。
+ DOMContentLoaded:
当DOMContentLoaded事件触发时，仅当dom加载完成，不包括样式表，图片，flash，iframe

光看定义，一目了然，哪个比较适合作为我们判断的标准：图片啊什么的，我们完全可以不用等。

在某些Gecko和Webkit引擎版本的浏览器里面，包括IE8在内，会同时发起多个http的请求并行加载样式表和脚步，但是脚本会等样式表加载完成之后才会被执行，甚至样式表加载之前页面都不会渲染。opera不会，样式表未加载好就可以执行js。

{% asset_img onLoadVSDomContentLoaded.png %}

### 兼容方案
#### ie8及以下兼容处理方案
ie的一般处理方案 --- `onreadystatechange` 事件。
html加载过程中会有一个document.readyState状态
五种状态：
+ 0（未初始化）：还没有send
+ 1 loading（载入）：正在发送请求
+ 2 loaded（载入完成）：执行完成，已经接收到全部响应内容
+ 3 interactive（交互）： 正在解析响应内容
+ 4 complete（完成）： 响应内容解析完成，客户端可以用了。
*complete事件和window.onload事件是同时的。*

这就是要监听页面的readystatechange事件，当事件为interactive或者complete时就可以开始做js的事情了。但是如果我们注册 ready 函数的时间点太晚了，这时页面已经加载完成，而我们才注册自己的 ready 函数，那就用不着上面的层层检查了，直接看看当前页面的 readyState 就可以了，如果已经是 complete ，那就可以直接执行我们准备注册的 ready 函数了。不过 ChrisS 报告了一个很特别的错误情况，我们需要延迟一下执行。

> setTimeout 经常被用来做网页上的定时器，允许为它指定一个毫秒数作为间隔执行的时间。当被启动的程序需要在非常短的时间内运行，我们就会给它指定一个很小的时间数，或者需要马上执行的话，我们甚至把这个毫秒数设置为0，但事实上，setTimeout有一个最小执行时间，当指定的时间小于该时间时，浏览器会用最小允许的时间作为setTimeout的时间间隔，也就是说即使我们把setTimeout的毫秒数设置为0，被调用的程序也没有马上启动。这个最小的时间间隔是多少呢？这和浏览器及操作系统有关。在John Resig的新书《Javascript忍者的秘密》一书中提到。
>> Browsers all have a 10ms minimum delay on OSX and a(approximately) 15ms delay on Windows.（在苹果机上的最小时间间隔是10毫秒，在Windows系统上的最小时间间隔大约是15毫秒），另外，MDC中关于setTimeout的介绍中也提到，Firefox中定义的最小时间间隔（DOM_MIN_TIMEOUT_VALUE）是10毫秒，HTML5定义的最小时间间隔是4毫秒。

既然规范都是这样写的，那看来使用setTimeout是没办法再把这个最小时间间隔缩短了。这样，通过设置为 1, 我们可以让程序在浏览器支持的最小时间间隔之后执行了。

```javascript
    if (document.readyState === "complete") {
        // 延迟 1 毫秒之后，执行 ready 函数
        setTimeout(jQuery.ready, 1);
    }
```

#### doScroll 检测法
但是当页面中带有iframe时，这个readyState状态会挂起一直等待，等待页面的iframe也加载完毕之后再处理，这个过程是我们不想要得，那就有另外一种处理方案。
> MSDN 关于 JScript 的一个方法有段不起眼的话，当页面 DOM 未加载完成时，调用 doScroll 方法时，会产生异常。那么我们反过来用，如果不异常，那么就是页面DOM加载完毕了！Diego Perini 在 2007 年的时候，报告了一种检测 IE 是否加载完成的方式，使用 doScroll 方法调用。详细的说明见这里。原理是对于 IE 在非 iframe 内时，只有不断地通过能否执行 doScroll 判断 DOM 是否加载完毕。在本例中每间隔 50 毫秒尝试去执行 doScroll，注意，由于页面没有加载完成的时候，调用 doScroll 会导致异常，所以使用了 try -catch 来捕获异常。

```javascript
    (function doScrollCheck() {
        if (!jQuery.isReady) {  
            try {
                // Use the trick by Diego Perini
                // http://javascript.nwbox.com/IEContentLoaded/
                top.doScroll("left");
            } catch (e) {
                return setTimeout(doScrollCheck, 50);
            }   
            // and execute any waiting functions
            jQuery.ready();
        }
    })();
```

#### jQuery的实现  

```javascript
    //全局方法
    DOMContentLoaded = function() {
        if ( document.addEventListener ) {
            document.removeEventListener( "DOMContentLoaded", DOMContentLoaded, false );
            jQuery.ready();
        } else if ( document.readyState === "complete" ) {
            // we're here because readyState === "complete" in oldIE
            // which is good enough for us to call the dom ready!
            document.detachEvent( "onreadystatechange", DOMContentLoaded );
            jQuery.ready();
        }
    }

    //入口 jquery实例调用
    ready: function( fn ) {
        // Add the callback
        jQuery.ready.promise().done( fn );

        return this;
    }

    jQuery.ready.promise = function( obj ) {
        if ( !readyList ) {

            readyList = jQuery.Deferred();

            // Catch cases where $(document).ready() is called after the browser event has already occurred.
            // we once tried to use readyState "interactive" here, but it caused issues like the one
            // discovered by ChrisS here: http://bugs.jquery.com/ticket/12282#comment:15
            // 当页面加载完了，直接调用ready方法
            if ( document.readyState === "complete" ) {
                // Handle it asynchronously to allow scripts the opportunity to delay ready
                setTimeout( jQuery.ready, 1 );

            // Standards-based browsers support DOMContentLoaded
            } else if ( document.addEventListener ) {
                // Use the handy event callback
                document.addEventListener( "DOMContentLoaded", DOMContentLoaded, false );

                // A fallback to window.onload, that will always work
                window.addEventListener( "load", jQuery.ready, false );

            // If IE event model is used
            } else {
                // Ensure firing before onload, maybe late but safe also for iframes
                document.attachEvent( "onreadystatechange", DOMContentLoaded );

                // A fallback to window.onload, that will always work
                window.attachEvent( "onload", jQuery.ready );

                // If IE and not a frame
                // continually check to see if the document is ready
                var top = false;

                try {
                    top = window.frameElement == null && document.documentElement;
                } catch(e) {}

                if ( top && top.doScroll ) {
                    (function doScrollCheck() {
                        if ( !jQuery.isReady ) {

                            try {
                                // Use the trick by Diego Perini
                                // http://javascript.nwbox.com/IEContentLoaded/
                                top.doScroll("left");
                            } catch(e) {
                                return setTimeout( doScrollCheck, 50 );
                            }

                            // and execute any waiting functions
                            jQuery.ready();
                        }
                    })();
                }
            }
        }
        return readyList.promise( obj );
    };

    jQuery.extend({
        // 表示ready方法是否正在执行，若正在执行，则将isReady设置为true
        isReady: false,

        // ready方法执行前需要等待的次数
        readyWait: 1,

        // hold或者释放ready方法，若参数为true则readyWait++，否则执行ready，传入参数为true
        holdReady: function( hold ) {
            if ( hold ) {
                jQuery.readyWait++;
            } else {
                jQuery.ready( true );
            }
        },

        // 当DOM加载完毕时开始执行ready
        ready: function( wait ) {

            // 若传入的参数为true，则--readyWait；否则判断isReady，即ready是否正在执行  
            if ( wait === true ? --jQuery.readyWait : jQuery.isReady ) {
                return;
            }

            // Remember that the DOM is ready
            jQuery.isReady = true;

            // 若readyWait-1后还是大于0，则返回，不执行ready。
            if ( wait !== true && --jQuery.readyWait > 0 ) {
                return;
            }

            // If there are functions bound, to execute
            readyList.resolveWith( document, [ jQuery ] );

            // 触发ready方法，然后解除绑定的ready方法。
            if ( jQuery.fn.triggerHandler ) {
                jQuery( document ).triggerHandler( "ready" );
                jQuery( document ).off( "ready" );
            }
        }
    });
```
根据以上代码可见，最终DOMContented事件执行的，其实是jQUery.ready()这个工具函数。
（注意，jquery.ready()和jquery(document).raedy()不一样！！，前者是工具函数，后者是实例函数。）
这里是通过定义一个DOMContentLoaded函数作为桥梁来执行jquery.ready()函数的，这样做的目的就是为了及时的remove掉document的DOMContentLoaded事件的引用。
{% asset_img zongjie.png %}
推荐好文:  [何控制jquery的ready事件](http://www.xiabingbao.com/jquery/2015/06/27/jquery-holdready/)
