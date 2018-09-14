---
layout:     post
title:      "对Objective-C中block的分析(下) "
subtitle:   "block分析"
date:       2016-02-13
author:     "夜禹"
header-img: "img/post-bg-see-u-ali.jpg"
category: Block
header-mask: 0.3
catalog:    true
tags:
    - Record
---

> block大大简化了代码，在apple官方代码以及各种第三方开源库中广泛使用。<br />

主要有两大用处：
<p><li>参数</li></p>
通常作为最后一个参数用来call back，最常见的是网络中suceess和failure的callback。

<p><li>回调函数</li></p>
用来注册一个回调函数，在特定场景发生时调用。

通常我在用的时候总体而言作用上和delegate差不多，比如之前做的一个组件中，下拉的开始call back以及松手call back，既可以通过注册block，也可以通过注册delegate来实现，但是block比delegate要方便很多，比如在一个类中需要支持多个delegate相同的函数时，就需要在该class中添加判定各个delegate的不同，以及映射相应的函数，从而造成class的复杂和冗余，而使用block的话就可以很好的解决这个问题，并且调用函数和注册call back放在一起，review起来相对较容易。


block相当于js中的闭包，block可以与C,C++,OC在一起使用，可以作为参数等。
block本质上是一个oc对象，block所引用到的对象也是强引用，每个block的copy都会引用一份，所以block可以传给别的thread或runtime,而不会出现block使用的变量已经被释放了，在ARC中block都会被copy一份，block不会在stack上了。
可以将一个block引用传给一个任意类型的指针，或者将一个任意类型的指针传给block，但<strong>不能对block进行解引用操作</strong>，这样编译器不能计算出block的size。

在apple官方文档中虽然说明block也是一个对象，但是block与一般的对象不同，最主要的区别就是默认内存分配的位置不同，一般的对象的内存分配在堆上，而block默认分配在栈上。在MRC中，需要用Block_copy()和Block_release()来进行block的内存管理，一般的对象却是retain和release，更说明了block与一般对象不同，作了特殊处理。在ARC中，block默认会copy到堆上，也是防止Developer不小心错误使用分配在栈上的block导致程序的crash，现在用block类型的属性，通常都是copy。


由于用instrument没有检测出block的retain cycle，只有日后到工作中看能不能遇到场景。
本来想尝试在completion所在函数中将completion置为nil，这样存在堆上的block引用计数为0，会被释放，这样即便block中引入了self，也可以打破循环。(实测这样不可以)
避免retain cycle就是不要形成循环，形成循环后即使将变量置为nil也无效。
