---
layout: post_ymy
title: 对Objective-C中block的分析(上)
date: '2016-02-06 14:51:00'
tags: [知识点浅析]
---

<blockquote>间间断断的学习iOS开发到现在已有一年半多了，之前一直用OneNote记录点点滴滴，时间久了，不免杂乱，故而在此对每个感兴趣的点进行分析与总结。</blockquote>

### block官方文档浅析
block的概念是在iOS4.0开始出现的，通过对apple官方文档的阅读做以下总结与分析：



1.block定义
block的声明和定义通过使用符号```^```来实现，example如下所示：
{% highlight objc %}
int multiplier = 7;
int (^myBlock)(int) = ^(int num) {
    return num * multiplier;
};
{% endhighlight %}
myBlock为block变量，返回值和参数都为int类型，
{% highlight objc %}
void (^blockReturningVoidWithVoidArgument)(void);
int (^blockReturningIntWithIntAndCharArguments)(int, char);
void (^arrayOfTenBlocksReturningVoidWithIntArgument[10])(int);

typedef float (^MyBlockType)(float, float);
 
MyBlockType myFirstBlock = // ... ;
MyBlockType mySecondBlock = // ... ;
{% endhighlight %}
block支持省略参数(...)，当block没有参数时，要向()中写入void。
<p><li>这个图是相应的一些注解</li></p>
![注解](/assets/images/blog_pic/2016-02-06-01.png)
block本质上其实也是个变量，先声明定义，然后使用，很像函数指针，只不过*换成了^

2.block涉及的变量
block拥有的比较特别的地方就是不仅可以使用传进来的形参，也可以使用相同scope中包含的变量。
block涉及的变量有以下几种：
<p><li>全局变量和静态变量</li></p>
block对于全局变量和静态变量的访问和其他函数等一致(全局变量和静态变量存储在全局区和静态区),可正常访问和修改。
<p><li>传入的参数</li></p>
访问跟function一样，参数都是传值。
<p><li>实例变量</li></p>
block对于实例变量的访问与对象内其他函数一致(实例变量存储在堆中)，可正常访问和修改。
以下为实例代码：
{% highlight objc %}
dispatch_async(queue, ^{
    // instanceVariable is used by reference, a strong reference is made to self
    doSomethingWithObject(instanceVariable);
}); // 对self强引用(造成引用循环)
 
 
id localVariable = instanceVariable;
dispatch_async(queue, ^{
    /*
      localVariable is used by value, a strong reference is made to localVariable
      (and not to self).
    */
    doSomethingWithObject(localVariable);
}); //获取实例变量的值(不造成引用循环)
{% endhighlight %}

<p><li>scope中<strong>未添加</strong>__block修饰的变量</li></p>
block在定义时，会capture相同scope内block<strong>之前</strong>已经定义好的其余变量，在block定义之后定义的local变量，block是访问不到的，如果变量未加__block修饰，则copy一份并转成const型，可在block内部使用，如果在block内部修改变量，则编译器会报错，如果在block定义之后和block调用之前修改了变量的值，不会影响block已经capture到的value。
<p><li>scope中<strong>添加了</strong>__block修饰的变量</li></p>
当scope内local变量添加了__block修饰时，则block对变量有个引用，变量在block内外均可修改，当block调用时，得到的该变量的值为该变量的当前值。添加__block的变量存储在一个scope内所有变量和block共享的一个storage钟。__block修饰的变量不能是可变长度数组，也不能是包含C99可变长度数组的结构。
总结local变量对于block而言就是两种：const，引用

以下为实例代码：
{% highlight objc %}
NSInteger CounterGlobal;
static NSInteger CounterStatic;
{
    NSInteger localCounter = 42;
    __block char localCharacter;
    NSString *str = @"ymy";
    void (^aBlock)(void) = ^(void) {
        ++CounterGlobal;
        ++CounterStatic;
        CounterGlobal = localCounter; // localCounter fixed at block creation
        localCharacter = 'a'; // sets localCharacter in enclosing scope
        localCounter = 1; //error，需添加__block修饰
        NSLog(@"str = %@", str);
        NSLog(@"test = %@", test); //error,不能访问到test
    };
    ++localCounter; // unseen by the block
    localCharacter = 'b';
    str = @"ymy_def";
    NSString *test = @"test";
    aBlock(); // execute the block，打印出来的str为ymy，CounterGlobal为42，CounterStatic为1
}
{% endhighlight %}

3.避免的情况
apple官方文档中说明的要避免的情况：
{% highlight objc %}
void dontDoThis() {
    void (^blockArray[3])(void);  // an array of 3 block references
 
    for (int i = 0; i < 3; ++i) {
        blockArray[i] = ^{ printf("hello, %d\n", i); };
        // WRONG: The block literal scope is the "for" loop.
    }
}
 
void dontDoThisEither() {
    void (^block)(void);
 
    int i = random():
    if (i > 1000) {
        block = ^{ printf("got i at: %d\n", i); };
        // WRONG: The block literal scope is the "then" clause.
    }
    // ...
}
{% endhighlight %}
块文本(即^{...})是代表block的本地堆栈数据结构的地址，因此本地堆栈数据结构的范围是闭合的复合语句。(我也不是太理解)
所以在参数为void的时候，尽量也带上(void)，不省略了。

4.关于block的内存管理
block在非ARC中有三种，ARC中有两种，现在已经完全进入ARC时代，了解这几种情况对于我们理解apple代码以及代码的不断演进很有帮助。

<p><li>__NSGlobalBlock__</li></p>
当block内部没用引用任何外部变量时，block为__NSGlobalBlock__，copy之后依然为__NSGlobalBlock__。
实例代码如下：
{% highlight objc %}
    MYCompletion test = ^(void){
        return 1234;
    };
    MYCompletion testCopy = [test copy];

    (lldb) po test
    <__NSGlobalBlock__: 0x10a53a150>
    
    (lldb) po testCopy
    <__NSGlobalBlock__: 0x10a53a150>
{% endhighlight %}

<p><li>__NSMallocBlock__</li></p>
在ARC中，当block内部引用了外部变量时，block为__NSMallocBlock__，copy之后依然为__NSMallocBlock__。
实例代码如下：
{% highlight objc %}
    int a = 10;
    MYCompletion testMalloc = ^(void){
        return a;
    };
    MYCompletion testMallocCopy = [testMalloc copy];

    (lldb) po testMalloc
    <__NSMallocBlock__: 0x7fe6584113e0>
    
    (lldb) po testMallocCopy
    <__NSMallocBlock__: 0x7fe6584113e0>
{% endhighlight %}
在ARC中，block默认都经过了copy，放在heap上。

<p><li>__NSStackBlock__</li></p>
在非ARC中，testMalloc为__NSStackBlock__,testMallocCopy才为__NSMallocBlock__。apple之所以将ARC中block默认位置放在heap上，是为了防止block因位于栈区，而在后面调用时栈被释放造成crash。
实例代码如下：
{% highlight objc %}    
    MYCompletion func()
    {
        int a = 10;
        MYCompletion result = ^(void){
            return a;
        };
        return result;
    }
    MYCompletion testStack = func();
    int c = testStack();

    (lldb) po testStack
    <__NSStackBlock__: 0x7fff52eaf988>
{% endhighlight %}
上述代码中加入result是为了骗过编译器，否则编译不过，执行testStack()可能会出现BAD_ACCESS错误，但实测我的未出现。

5.关于block引用的变量的内存情况
local变量在block copy之前在栈中，在copy之后，捕获到的变量中未加__block修饰的变量都copy一份到heap上，添加了__block修饰的变量都移动到heap上。
实例代码如下：
{% highlight objc %}
    int a = 123;
    __block int b = 123;
    NSLog(@"%@", @"=== block copy前");
    NSLog(@"&a = %p, &b = %p", &a, &b);
    
    void(^block)() = ^{
        NSLog(@"%@", @"=== Block");
        NSLog(@"&a = %p, &b = %p", &a, &b);
        NSLog(@"a = %d, b = %d", a, b = 456);
    };
    block = [block copy];
    block();
    
    NSLog(@"%@", @"=== block copy后");
    NSLog(@"&a = %p, &b = %p", &a, &b);
    NSLog(@"a = %d, b = %d", a, b);
    
    [block release];
{% endhighlight %}

6.block的线程安全问题
如果block是用在多线程中的，通常在属性中添加atomic属性，但依旧不够，因为在调用时block也可能会被修改，从而导致执行block时出现问题。
实例代码如下：
{% highlight objc %}
typedef void (^MYBlockType)(int);
@property (atomic, copy) MYBlockType testBlock;

//ARC
if (self.testBlock)
{
    //这个时候即便if通过，此处的self.testBlock
    //atomic只能够确保block执行的原子性，并不能保证线程安全
    self.testBlock(123);
}

MyBlockType block = self.testBlock;
//block现在没有别的线程可以修改
if (block)
{
    block(123);
}

//非ARC，非ARC加上retain和release
MyBlockType block = [self.testBlock retain];
if (block)
{
    block(123);
}
[block release];
{% endhighlight %}

7.block的引用循环问题
使用block引入的最大的问题就是引用循环问题，不仅是block，而是apple中内存管理一个很大的问题就是引用循环，环越大越难发现，这点Android就可以处理，算是apple有所取舍吧，毕竟引用计数相对垃圾回收感觉效率上还是会好一些。
在ARC下用__weak来处理，在非ARC下用__block来处理。
实例代码如下：
{% highlight objc %}
//ARC
    __weak typeof(self) weakSelf = self;
    self.testBlock = ^(void)
    {
        __strong typeof(self) strongSelef = weakSelf; //防止self被释放
        //使用weakSelf访问self成员
        [strongSelef anotherFunc];
        return 0;
    };

//非ARC
    __block typeof(self) weakSelf = self;
    self.testBlock = ^(int paramInt)
    {
        //使用weakSelf访问self成员
        [weakSelf anotherFunc];
    };
{% endhighlight %}




