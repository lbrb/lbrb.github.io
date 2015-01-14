---
layout: post
title: Jquery queue
category: 技术
tags: Jquery
keywords: jquery, queue
---

h3. 利用Jquery的queue完成复杂的动画效果

简单的动画效果我们可以使用下面的方法

    $('#object').hide('slow').show('slow');

但是复杂的动画效果，比如多个元素多种类型的效果，用上面的方法就很难实现了，这里使用queue来实现复杂的动画效果。

    $('document').queue([fun1, fun2, fun3]);//fun1,fun2,fun3为方法，里面可以写任何代码，动画或增删改元素都可以
    function fun1(){
        $("#a").hide('slow', function(){$('document').dequeue();})//方法执行完毕后，执行队列中的下一个方法。
    }
    ...//定义需要执行的方法

参考[jQuery动画高级用法](http://www.cnblogs.com/hh54188/archive/2011/04/09/1996469.html)



