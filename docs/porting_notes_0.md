# AWTK WEB版移植笔记-序

## 一、WEB版本对AWTK意义

WEB版本对AWTK是很重要的，主要原因有：

* 让用C语言开发的AWTK应用程序，在不需要修改源码的情况下，能在浏览器中运行。这样做的好主要在于，可以很方便的向客户展示项目。你只需分享一个链接，客户就可以在浏览器中打开，并看到实际的运行效果，这是一种非常棒的体验。

* 把AWTK编译成一个JS库，你可以用JS开发AWTK应用程序，并在浏览器中运行。AWTK-JS让AWTK支持用JS来开发AWTK应用程序，并在嵌入式系统中运行，但不能在浏览器中运行。而AWTK-WEB则是让AWTK支持用JS来开发AWTK应用程序，并且能够在浏览器中运行，这打通了嵌入式和WEB之间的壁垒。

* 开发各种小程序也是AWTK的目标之一。而小程序无一例外都是用javascript开发的，WEB版本移植好了，支持小程序开发也就非常容易了，所以我们需要先迈过这个坎。

## 二、如何实现WEB版本的移植

要将用C语言开发的AWTK移植到WEB上，就得将C语言编译成Javascript或者WebAssembly，[emscripten](https://emscripten.org)为这个编译提供切实可行的途径，emscripten是一个伟大的工具，经过不少大型项目的验证，已经非常成熟了，可行性是没有问题的。

但是可行和实际能否搞定是有差距的，写个RTOS内核是可行的，但是实际成功的并不多。能用和好用也是有差距的，甚至是成功与失败的差别，GUI引擎很多，但是90%以上的都只能算是demo。AWTK的WEB不但要能用，而且要好用才行，所以整个移植过程就变得有趣了。

我把在移植过程遇到的问题，面临的各种选择做个笔记，以供有需要的朋友参考。

## 三、WEB版本的目标及策略

简单粗暴的将AWTK编译成WEB版本，中间遇到的问题和要做的工作可能少很多(当然也许有些问题更难解决)。但那只能让AWTK的WEB版本可用，离好用还有不小差距。为了让AWTK WEB好用，在移植之初就定了以下目标：

### 1.体积要小。

WEB版本要从网上加载资源，体积大小关系APP的加载时间，对用户体验造成直接影响。为了减小JS的体积，我们做了如下选择：

* 不使用SDL作为移植层。使用SDL作为移植层，就得加入SDL、navavg、stbimage和stbfont等一大堆东西，这会让代码量加倍，而且字体只能使用APP自己的字体，资源体积就更没法容忍了。
    
* 缺省字体使用浏览器自带的字体。字体文件很大，动辄就是几M甚至十几M。缺省字体使用浏览器自带的字体，可以大大降低资源的体积。而特殊字体通常很小，仍然采用APP自己的字体。
    
* 图片采用独立资源。这样可以减小显示第一个界面前所加载的资源的大小，等到第一个界面出来之后，可以显示加载资源的进度，这样大大增强用户体验。

### 2.速度要快。

* UI和Style数据采用二进制常量，编译到代码中，以减少访问网络的次数，这些数据不大，对代码加载时间产生的影响可以忽略。

* 图片解码使用浏览器本身的功能。这样解码速度会更快，也避免使用
stbimage解码库，减小代码体积。

* 字体解码使用浏览器本身的功能。这样解码速度会更快，也避免使用stbfont解码库，减小代码体积。

* 窗口动画采用WebGL贴图，可以提高窗口动画的效果。

* 启用脏矩形算法。不变不画，有变只画变的部分。
    
### 3.功耗与性能兼顾。

* 窗口动画采用WebGL贴图，正常绘制采用Canvas 2D接口。
    
### 4.最大化兼容性。
    
* 少用或不用第三方库。比如SDL有WEB版本的移植，但是移植到各种小程序可能就有问题，修改第三方的库是非常麻烦的事情，后期维护和升级也是一个难题。
    
* 隔离浏览器特定的功能。事件和输入法，各个小程序处理方式不同，Canvas接口有小的差异，必须把它们隔离开来。
    
* 保守的使用浏览器提供的功能，避免移植到其它平台时遇到麻烦。
    
* 自动适配支持WebAssembly和不支持WebAssembly的平台。WebAssembly很快很小，但是部分浏览器和小程序并不支持，必须自动检查，并加载不同的程序。

    


