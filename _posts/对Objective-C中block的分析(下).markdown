---
layout: post_ymy
title: 对Objective-C中block的分析(下)
date: '2016-02-06 14:51:00'
tags: [知识点浅析]
---

block大大简化了代码，在apple官方代码以及各种第三方开源库中广泛使用，常常作为最后一个参数用来call back，最常见的是网络中suceess和failure的callback，通常我在用的时候总体而言和delegate的作用差不多，比如之前做的一个组件中，下拉的开始call back以及松手call back既可以通过注册block，也可以通过注册delegate来实现。

函数回调，参数

block相当于js中的闭包，block可以与C,C++,OC在一起使用，可以作为参数等。
block本质上是一个oc对象，block所引用到的对象也是强引用，每个block的copy都会引用一份，所以block可以传给别的thread或runtime,而不会出现block使用的变量已经被释放了，在ARC中block都会被copy一份，block不会在stack上了。
可以将一个block引用传给一个任意类型的指针，或者将一个任意类型的指针传给block，但<strong>不能对block进行解引用操作</strong>，这样编译器不能计算出block的size。
block的返回值是可以infer出来的，如果有多种返回值，有必要的话需要转换一下，如果参数列表为void，那么也可以省略(void)，直接^{}，但建议都加上(void)。

嵌套的block中，局部变量的值为最近的括号里的（试一下嵌套的效果，弄清楚access范围）
Their values are taken at the point of the block expression within the program.在block定义的位置获取local variable的value，后面就unseen了
When a block is copied, it creates strong references to object variables used within the block
block设置三种变量
不要用^{}这种模式
Blocks also support two other types of variable:
1. At function level are __block variables. These are mutable within the block (and the enclosing scope) and are preserved if any referencing block is copied to the heap.加了block那么在scope中都是可变的，block在调用时取variable的当前值，而不是普通的local variable在定义时去那一次值
const imports.



