title: 'ECMAScrip中的对象存取器:getter和setter'
date: 2016-01-21 17:34:17
tags:
- JavaScript
comments: true
---
显然这是一个无关IE（高级IE除外）的话题，尽管如此，有兴趣的同学还是一起来认识一下ECMAScript5标准中getter和setter的实现。在一个对象中，操作其中的属性或方法，通常运用最多的就是读（引用）和写了，譬如说o.get，这就是一个读的操作，而o.set = 1则是一个写的操作。事实上在除ie外最新主流浏览器的实现中，任何一个对象的键值都可以被getter和setter方法所取代，这被称之为“存取器属性”。
<!--more-->
毫无疑问，getter负责查询值，它不带任何参数，setter则负责设置键值，值是以参数的形式传递，在他的函数体中，一切的return都是无效的。和普通属性不同的是，存储器属性在只声明了get或set时，对于读和写是两者不可兼得的，当它只拥有了getter方法，那么它仅仅只读，同样的，当它只有setter方法，那么您读到的永远都是undefined。如何声明对象存储器属性呢？ 最快捷的途径就是利用对象字面量的语法来写了，请看下述一段代码：

    var oo = {
        name : '贤心',
        get sex(){
            return 'man';
        }
    };
    //显然这是不允许的，因为贤心并不希望外界去改变他是男性的事实，所以对于sex只设置了只读功能
    oo.sex = 'woman';//在严格模式下报错。
    console.log(oo.sex); //结果依然是man
有意思的是，这颠覆了我们以往的理解，就是在方法定义时并未用function关键字。事实上这里的get或set，你可以理解为两种不同状态下的function：包容的一面（写），安全的一面（读），当一种整体被肢解为不同的形态，意味着我们可能不再需要在表现形式上遵循传统，所以我们并没有使用冒号将键和值分开。那么，继续上面的例子。你将如何在存储器属性的基础上变得读写兼备呢，也许下面的一段会给你带来答案：

    var oo = {
        name : '贤心',
        get sex(){
            if(this.sexx){
                return this.sexx;
            }else{
                return 'man';
            }
        }, set sex(val){
            this.sexx = val;
        }
    };
    //噢，他如此包容，乃至于人们改变他的性别，他也接受
    oo.sex = 'woman';
    console.log(oo.sex); //结果woman

或许你会觉得这是多此一举的，因为我们完全可以忽视get和set，直接让sex方法具备两种权限。 但之所以我们将get和set单独拿出来，是为了更加清晰地理解ECMAScript5对javascript对象键值操作中，一个更为严谨的诠释。 当然，在IE污染的中国，新型的主流技术总是显得格格不入，在实际的项目开发中，也许你永远不会用到get和set，但谁又能保证以后不会呢……

摘自：[贤心博客](http://sentsin.com/web/20.html)
