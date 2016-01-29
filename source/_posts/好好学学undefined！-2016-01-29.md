title: 好好学学undefined！
date: 2016-01-29 09:48:56
tags:
- jQuery
- JavaScript
comments: true
categories:
- JavaScript
---
# undefined类型
undefined类型只有一个值，即特殊的undefined，我们称之为`字面值undefined`，undefined意为`未定义`。

`字面值undefined`是全局Global对象（window）的一个特殊属性，其值是未定义的。但 typeof window.undefined 返回"undefined" 。

我们可以通过下面的例子来验证undefined是否为全局Global对象（window）的属性:

    alert('undefined' in window);//输出：true   
    var anObj = {};   
    alert('undefined' in anObj); //输出：false 
    从中可以看出，undefined是window对象的一个属性，但却不是anObj对象的一个属性。
## `字面值undefined`的产生
`字面值undefined`产生的原因有5种：

    * 访问对象不存在的属性或方法
    * 声明了变量但从未赋值
    * 调用函数时，应该提供的参数没有提供，该参数等于undefined。
    * 方法没有返回值，默认返回undefined
    * 访问越界的数组。
    * void(expression) 形式的表达式。
    
    {% codeblock lang:JavaScript %}
    var v1,obj = {};      
      
    console.log(v1); //`字面值undefined`    
    console.log(obj.get); //`字面值undefined`
    
    typeof v1; // "undefined"    
    typeof v2; // 对未声明的变量使用typeof 也会输出 "undefined"。     
    typeof obj.get; // "undefined"
    
    var message1 = undefined;  //显示的设置为undefined
    typeof message1 //"undefined"
    
    function test(){}; 
    console.log(test()); //`字面值undefined`
    
    var arr = []; 
    console.log(arr[8]) //`字面值undefined`
    {% endcodeblock %}
        
<span style="color: red;">当我们在程序中使用`字面值undefined`时，实际上使用的是window对象的undefined属性，同样，当我们定义一个变量但未赋予其初始值，例如：`var aValue;`这时，JavaScript在预编译时会将其初始值设置为对window.undefined属性的引用，于是，当我们将一个变量或值与undefined比较时，实际上是与window对象的undefined属性比较。这个比较过程中，JavaScript会搜索window对象名叫"undefined"的属性，然后再比较两个操作数的引用指针是否相同。</span>

---
您可以通过将变量与`字面值undefined`进行比较确定变量是否存在，您也可以通过将变量的类型与字符串“undefined”进行比较确定其类型是否为 undefined类型。
以下示例演示了如何确定已声明的变量的 x：
        
    var x;
    
    // This method works.
    if (x == undefined) { //这种方式只能对已经声明的变量使用，对未声明的变量使用会报错。
        document.write("comparing x to undefined <br/>");
    }
    
    // This method doesn't work - you must check for the string "undefined".
    if (typeof(x) == undefined) {//未执行，因为typeof 方法返回的是字符串。
        document.write("comparing the type of x to undefined <br/>");
    }
    // This method does work. 
    if (typeof(x) == "undefined") {
        document.write("comparing the type of x to the string 'undefined'");
    }
    
    // Output: 
    // comparing x to undefined 
    // comparing the type of x to the string 'undefined'
## 提高访问`字面值undefined`的性能：
由于window对象的属性值是非常多的，在每一次与`字面值undefined`的比较中，搜索window对象的undefined属性都会花费时间。在需要频繁与undefined进行比较的函数中，这可能会是一个性能问题点。因此，在这种情况下，我们可以自行定义一个局部的undefined变量，来加快对undefined的比较速度。例如：

    function anyFunc() {
        var undefined; //自定义局部undefined变量
        if (x == undefined){} //作用域上的引用比较
        while (y != undefined){} //作用域上的引用比较
    };
其中，定义undefined局部变量时，其初始值会是对window.undefined属性值的引用。新定义的局部undefined变量存在与该函数的作用域上。
在随后的比较操作中，JavaScript代码的书写方式没有任何的改变，但比较速度却很快。因为作用域上的变量数量会远远少于window对象的属性，搜索变量的速度会极大提高。
这就是许多前端JS框架为什么常常要自己定义一个局部undefined变量的原因!!!
比如jQuery 源码：

    (function( window, undefined ) {
        /*
        * 
        * code
        * 
        * */
    })( window );
    