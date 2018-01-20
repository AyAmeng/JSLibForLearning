<div data-v-13f76525="" data-v-3f216172="" class="container entry-view">

# 理解Event的冒泡模型

<div data-v-13f76525="" data-v-3f216172="" class="post-info-mobile">

<div data-v-13f76525="" data-v-3f216172="" class="action create-at-action">2018 年 01 月 16 日</div>

[少木木](/user/5821e3d8bf22ec0068e631e2)</div>

<div data-v-13f76525="" data-v-3f216172="" class="post-content-container">

<div data-v-13f76525="" data-v-3f216172="" class="entry-content post-content juejin-image-viewer__container">

> 本文探索一下Event的冒泡过程和初学遇到的几个小bug

### DOM Event概述

Event接口是检测在DOM中的发生的所有事件，我们一直在用，而且从DOM的很早的版本就一直在用着。早期的网景(后来的火狐)和IE是各自为战，直到W3C一统江湖，DOM[版本](https://link.juejin.im?target=https%3A%2F%2Fwww.w3.org%2FDOM%2FDOMTRhttps%3A%2F%2Fwww.w3.org%2FDOM%2FDOMTR)一路发展而来，经历了DOM-0(洪荒时代)、DOM-1(只有两章核心内容)、DOM-2(划时代的一个版本，我们学的Event就在这个版本，而且目前的用的也是这个版本)、DOM-3、DOM-4(草案阶段)。

*   通过一个例子唤醒对Event的认识

```
//1、有一个js函数如下
function print(){
  console.log(1)
}

//2、在html的button里面点击触发上面的函数
<button id=button onclick="?">点我</button>
//问号处填可以填什么 A. print() B.print C.print.call()

//在js里面的onclick里面触发
button.onclick = ?
//问号处可以填什么 A. print() B.print C.print.call()

```

*   很明显第一个问号应该选`A` `C`，第二个问号应该选`B`
*   第一处在HTM中，点击事件要立刻执行代码，肯定选择带`()`的，而第二处在JS中，onclick是一个属性，不需要立刻执行，等用户点击了，浏览器再反应，不需要`()`。

既然`onclick`等on事件在JS中是一个属性，那么后面的就会覆盖前面的，所以DOM2里面引入了一个重要的`EventListener`，是一个队列。

### addEventListener

这是一个队列，[例子1](https://link.juejin.im?target=http%3A%2F%2Fjs.jirengu.com%2Frixodijifu%2F1%2Fedit%3Fhtml%2Cjs%2Coutput)，先进先出的特点，为后面的冒泡模型做准备。

```
function f(){
  console.log("eventListener不会覆盖")
}

button2.addEventListener('click', function(){
  console.log("eventListener不会覆盖1")
})
button2.addEventListener('click', f)
button2.removeEventListener('click', f)
button2.addEventListener('click', function(){
  console.log("eventListener不会覆盖3")
})

```

*   会打印出什么呢，答案是`eventListener不会覆盖1` `eventListener不会覆盖3`
*   所以说既然on可以一个打印出结果，就可以借助`remove`来实现`one`执行一次的操作

```
function f(){
  console.log("eventListener不会覆盖2")
  button2.removeEventListener('click', f)
}

button2.addEventListener('click', f)

```

只会[打印一次](https://link.juejin.im?target=http%3A%2F%2Fjs.jirengu.com%2Fyunidupexu%2F2%2Fedit%3Fhtml%2Cjs%2Coutput)，不会一直打印了，也就是`one`的原理。

*   具体的模型可以看[W3C](https://link.juejin.im?target=https%3A%2F%2Fwww.w3.org%2FTR%2FDOM-Level-3-Events%2F%23dom-event-architecture)

<figure>![W3C的模型](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="894" height="645"></svg>)

<figcaption></figcaption>

</figure>

### 冒泡模型

上面的官方文档中，我只研究一下捕获阶段(capture phase)和冒泡阶段(bubbling phase)。

*   什么是冒泡呢？我们先看一段代码

```
grand.addEventListener('click', function(){
  console.log('我是你爷爷')
})
dad.addEventListener('click', function(){
  console.log('我是你爸爸')
})

son.addEventListener('click', function(){
  console.log('我是你儿子')
})

```

*   这是三个`div`的事件，当你点击的时候，控制台打印必然会有顺序。那么应该是什么顺序呢，正常人的思维不外乎两种结果
    *   第一种：我是你的儿子 我是你爸爸 我是你爷爷
    *   第二种： 我是你爷爷 我是你爸爸 我是你儿子
    *   到底是那种呢，W3C说都行，看你代码咋写的了，上面的代码打印顺序是第一个中，也就是冒泡。

<figure>![冒泡排序](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="1280" height="533"></svg>)

<figcaption></figcaption>

</figure>

*   如果你想实现第二种打印方式，也就是捕获阶段，应该修改代码如下

```
grand.addEventListener('click', function(){
  console.log('我是你爷爷')
}, true)
dad.addEventListener('click', function(){
  console.log('我是你爸爸')
}, true)

son.addEventListener('click', function(){
  console.log('我是你儿子')
}, true)

```

*   也就是说`addEventListener`后面的参数决定了顺序，当你不写的时候是`undefined`，也就是`false`的意思。
*   复习一下五个`falsey`值
    *   `0` `NaN` `''` `null` `undefined` 除此之外都是`true`

<figure>![简单的图解](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="1043" height="693"></svg>)

<figcaption></figcaption>

</figure>

上图是简单的图解，注意优先运行为`true`的部分，再运行`false`的部分。

简单的实例====================>[demo](https://link.juejin.im?target=http%3A%2F%2Fjs.jirengu.com%2Fyunidupexu%2F5%2Fedit)

*   一个变式

```
grand.addEventListener('click', function(){
  console.log('我是你爷爷')
}, true)
dad.addEventListener('click', function(){
  console.log('我是你爸爸')
})

son.addEventListener('click', function(){
  console.log('我是你儿子')

```

*   上述代码应该是什么顺序呢

<figure>![变式](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="1280" height="531"></svg>)

<figcaption></figcaption>

</figure>

*   谁是`true`，先打印谁，都是`false`，继续按照冒泡顺序打印。

#### 一个奇葩的问题

```
son.addEventListener('click', function(){
  console.log('我是你儿子true')
}, true)

son.addEventListener('click', function(){
  console.log('我是你儿子false')
})

```

*   给同一个元素 `false` `true`，应该打印什么呢
*   答案是： 按照书写的顺序，谁在前面先打印谁。

<figure>![奇葩](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="1280" height="552"></svg>)

<figcaption></figcaption>

</figure>

#### 意想不到的Bug

`parent`是关键字不能使用，一不小心使用的话会出问题。

<figure>![意外bug](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="1280" height="530"></svg>)

<figcaption></figcaption>

</figure>

*   你用了关键字做变量，把鼠标点烂也看不到效果。

### 点击空白，对话框消失的案例

*   领导说有一个需求，点击某个按钮，弹出对话框，点击空白会消失。
*   你的第一个思路：先把div设为none，点击按钮的时候，再让这个`div`的display是block，点击其他地方变为none。
*   很好，你去实现一下吧。

#### 第一个bug

*   很快你会碰到了第一个bug

    *   第一个错误：监听错了对象

    <figure>![监听错了对象](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="1280" height="595"></svg>)

    <figcaption></figcaption>

    </figure>

正常来说，应该点击body控制台打印数字1，你点烂了你的罗技鼠标也没出来。为什么呢？

*   我们使用**border大法**，看看它到底在哪

<figure>![body的位置](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="1280" height="533"></svg>)

<figcaption></figcaption>

</figure>

使用了红色border之后，发现body的高度太矮了，点击不到啊。

*   你明白监听错对象了，那你就换了一个对象，监听文档呗，肯定没问题了。

#### 第二个bug

*   很好，你进入了第二个bug了

    *   第二个bug：你都能点击到，但是弹不出对话框了

    <figure>![第二个bug](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="1280" height="594"></svg>)

    <figcaption></figcaption>

    </figure>

根据图片 中的控制台可以发现，确实都点击到了，监听没问题，而且点击后，也是按照冒泡的顺序打印的结果。

*   那为什么没有对话框了呢

<figure>![正常的现象](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="1280" height="638"></svg>)

<figcaption></figcaption>

</figure>

注释掉出问题的代码后，上图是正常的点击出现对话框啊，说明问题就出在注释的代码上。

*   bug出现的原因就在于：默认冒泡的影响，当你点击的浮层那个`div`，之后，往 `body` `document`上冒泡，在`document`上立刻被杀死，display变为none，你做梦能看到 弹出框啊。

<figure>![阻止冒泡](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="750" height="351"></svg>)

<figcaption></figcaption>

</figure>

#### 修复第二个bug

我们既然知道了第二个bug产生的原因，那么我们阻止冒泡顺序

*   解决的方案，不让其往上冒泡，自己管理。

```
clickMe.addEventListener('click', function(){
  popover.style.display = 'block'
  console.log('点击浮层了') 
})

wrapper.addEventListener('click', function(e){
  e.stopPropagation()
})

document.addEventListener('click', function(){
  popover.style.display = 'none'
  console.log('点击文档了') 
})

```

*   但是随之而来的是一个关于内存占用的问题，现在你是只有一个popover，只有一个函数，等你有了很多个popover，如果按照这个写法会有很多个函数，所以不能这么写，采用下面的写法，节省内存。

```
$(clickMe).on('click', function(){
  $(popover).show()
  console.log('show')
  setTimeout(function(){
    console.log('one click')
    $(document).one('click', function(){
     console.log('我觉的他不会执行')
     $(popover).hide()            
    })
  },0)

})
// $(wrapper).on('click', function(e){
//   e.stopPropagation()
// })

$(document).on('click', function(){
  console.log('走到document啦')
})

```

*   只有点击的时候才用，设置settimeout是为了让他异步，不至于立刻隐藏，产生第一个bug。
*   注意一下，jQuery的 `show()` `hide()`

<figure>![jQuery节省内存](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="1280" height="600"></svg>)

<figcaption></figcaption>

</figure>

*   当你点击按钮，只会打印图中这两句话，另外两句只有再次点击才会打印。

<figure>![具体的分析](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="880" height="557"></svg>)

<figcaption></figcaption>

</figure>

JS版本的节省内存的版本==================>[节省内存](https://link.juejin.im?target=http%3A%2F%2Fjs.jirengu.com%2Fpepetixevo%2F1%2Fedit%3Fhtml%2Cjs%2Coutput)

jQuery版本的节省内存版本=================>[jQuery节省内存](https://link.juejin.im?target=http%3A%2F%2Fjs.jirengu.com%2Fzufemacina%2F1%2Fedit%3Fhtml%2Cjs%2Coutput)

#### 对话框小三角的制作

```
.popover{
  display: inline-block;
  border: 1px solid red;
  position: relative;
  padding: 10px;
  margin:10px;
}
.popover::before{
  position: absolute;
  content: '';
  top: 5px;
  right: 100%;
  border: 10px solid transparent;
  border-right-color:red;
}
.popover::after{
  content: '';
  border: 10px solid transparent;
  position: absolute;
  right: 100%;
  top: 5px;
  border-right-color: white;
  margin-right: -1px;
}

```

主要利用`boder-right-color`以及两个伪元素。

浮层三角的实例=============================>[demo](https://link.juejin.im?target=http%3A%2F%2Fjs.jirengu.com%2Fzugepaneyu%2F3%2Fedit%3Fhtml%2Ccss%2Coutput)

### 冒泡的直观体现

点击一下会有惊喜的[github.com/codevvvv9/b…](https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2Fcodevvvv9%2Fbubble%2Fblob%2Fmaster%2Fbubble.gif)

[冒个泡](https://link.juejin.im?target=http%3A%2F%2Fjs.jirengu.com%2Fgeqihuboqe%2F1%2Fedit%3Fhtml%2Ccss%2Cjs%2Coutput)

</div>

</div>

</div>