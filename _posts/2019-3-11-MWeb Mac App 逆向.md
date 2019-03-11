---
layout:     post
title:      "MWeb Mac App 逆向"
subtitle:   "逆向 Mac App"
date:       2019-3-11
author:     "夜禹"
header-img: "img/in-post/1004/usephone.jpg"
description: "这里记录了逆向 Mac App 的简单做法"
category: 逆向 移动端 Mac App
header-mask: 0.3
catalog:    true
tags:
    - 逆向
---

## 背景
电脑里充斥了大量的 MarkDown 文件，之前都是编译 MacDown 的代码来用，但管理起大量的文件来就力不从心了，同时公司要求不能用其他公司的云产品，朋友推荐了 [MWeb](https://zh.mweb.im/)，据说是国产 App 难得好用的工具，简单用了下，还不错。

当然，这块 App 不是免费的，不算便宜，可以试用 14 天，

![](/img/in-post/0311/MWeb.png)

这里还是鼓励大家使用正版，逆向只为学习和玩。。。

## 分析
Run 起来 App 后，第一反应当然是根据弹框中显示的字符串定位逻辑，将二进制拖进 Hopper 中，

![](/img/in-post/0311/Hopper.png)

然后发现这些函数都是在 Appdelegate 中，且注意到弹框是在 App 启动后被拉起来，所以搜索 App 的生命周期函数，重点分析一下，就很快找到了线索，

![](/img/in-post/0311/AppDelegate-Function.png)

细细分析一下即可知道这里的逻辑是：
* 禁止 UI
* 判断是否有权限(这里采用 c 函数，意图增大逆向难度)
* 如果有权限则取消禁止 UI
* 如果没有权限则走试用等逻辑，这里集成了 DevMateKit 来处理

所以这里关键函数就是这个 sub_100246a60，

![](/img/in-post/0311/sub_100246a60.png)

所以只要 Hook 这个 c 函数，return 1 即可。

## 步骤
* 新建 MacOS 动态库
* Hook sub_100246a60，代码如下

```C
intptr_t g_slide;
static void _register_func_for_add_image(const struct mach_header *header, intptr_t slide) {
    Dl_info image_info;
    int result = dladdr(header, &image_info);
    if (result == 0) {
        NSLog(@"load mach_header failed");
        return;
    }
    
    //获取当前的可执行文件路径
    //    NSString *execName = [[[NSBundle mainBundle] infoDictionary] objectForKey:@"CFBundleExecutable"];

    NSString *execName = @"Contents/MacOS/MWeb";
    NSString *execPath = [[[NSBundle mainBundle] bundlePath] stringByAppendingFormat:@"/%@", execName];
    if (strcmp([execPath UTF8String], image_info.dli_fname) == 0) {
        g_slide = slide;
    }    
}

int (*orig_SubMethod)(char);

int hook_SubMethod(char arg) {
    return 1;
}
static void __attribute__((constructor)) initialize(void) {
//注册添加镜像回调    
_dyld_register_func_for_add_image(_register_func_for_add_image);
//通过 模块偏移前的基地址 + ASLR偏移量 找到函数真正的地址进行hook
    MSHookFunction((void *)(0x0000000100246a60 + g_slide), (void *)hook_SubMethod, (void **)&orig_SubMethod);
    
```
这里需要引入 libsubstitute.dylib 和 substrate.h
需要编译 [substitute](https://github.com/comex/substitute)，直接编译会报错 syscall 被弃用，可下载早一些的 SDK 如 [MacOSX10.11.sdk](https://github.com/phracker/MacOSX-SDKs)，然后放到/Applications/XCode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs下。
然后通过如下命令行编译:
```C
./configure --xcode-sdk macosx10.11 && make -j8
```
在out目录就可以看到生成的动态库。

将其重命名为libsubstitute.0.dylib 并拷贝到/usr/local/lib/

* 调试时可以在 Build Phases 中添加脚本，

```C
cd ${TARGET_BUILD_DIR}
export DYLD_INSERT_LIBRARIES=./libMWeb.dylib && /Applications/MWeb.app/Contents/MacOS/MWeb
```
Cmd + B 即可拉起 App，当 App 启动后，通过 Debug - Attach To Process 即可开始断点调试

* 当调试完成后，可参考 [打包](https://blog.nswebfrog.com/2018/02/09/make-injection-app-for-mac/) 中的方法将 App 打包使用，当然也可以对 App 直接用 [insert_dylib](https://github.com/Tyilo/insert_dylib) 进行动态库注入使用

## 参考文章
* https://blog.nswebfrog.com/2018/02/09/make-injection-app-for-mac/
* http://www.alonemonkey.com/2017/05/31/get-start-with-mac-reverse/
* https://blog.csdn.net/glt_code/article/details/83420589
* http://iosre.com/t/hook-ida-sub-xxx/720