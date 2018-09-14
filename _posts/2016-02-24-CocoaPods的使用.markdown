---
layout:     post
title:      "CocoaPos的使用 "
subtitle:   "CocoaPods"
date:       2016-02-24
author:     "夜禹"
header-img: "img/post-bg-see-u-ali.jpg"
description: "这几个月在使用Git的过程中学到了不少实用的小技能，小赵准备简单总结一下，来与大家分享。也欢迎大家在评论区不断补充~"
category: Record
header-mask: 0.3
catalog:    true
tags:
    - Record
---

> CocoaPods是iOS项目的依赖管理系统，有点类似于Java项目管理时使用的Maven。通过使用CocoaPods，可以很方便的添加、删除和更新iOS项目所依赖的第三方库或者私有库。其官方网站是[CocoaPods.org](https://cocoapods.org/)。

## 安装Ruby环境
CocoaPods是使用Ruby实现的，Ruby可以通过```gem```命令来安装，MAC中一般都已经自带了Ruby环境，如果没有请参考[Ruby 官方文档](https://www.ruby-lang.org/en/documentation/installation/)来安装Ruby环境。



如果```gem```比较老，可以用下面这个命令升级```gem```。
{% highlight bash %}
	sudo gem update
{% endhighlight %}

## 安装CocoaPods
在安装CocoaPods之前，首先要知道团队所使用的CocoaPods版本，自己安装时也一定要与团队所使用的CocoaPods版本对应，否则容易出现兼容问题，如果个人使用，就无所谓了。
假设目前团队所使用的CocoaPods版本是0.36，则可有使用以下命令安装0.36版本的CocoaPods：
{% highlight bash %}
sudo gem install cocoapods -v 0.36
pod setup
{% endhighlight %}
如果想查看当前所安装的CocoaPods 的版本，可以使用以下命令：
{% highlight bash %}
	pod --version
{% endhighlight %}
如果安装了不合适的CocoaPods版本，可以使用以下命令卸载CocoaPods：
{% highlight bash %}
	sudo gem uninstall cocoapods
{% endhighlight %}
然后选择需要卸载的版本。
由于默认的RubyGems使用的是亚马逊的云服务(墙你懂得...)，所以将默认的RubyGems替换为淘宝的RubyGems镜像，速度要快很多，具体信息可以参考[RubyGems 镜像 - 淘宝网](https://ruby.taobao.org)。
通过采用下面的命令将默认ruby源更换成淘宝源：
{% highlight bash %}
gem sources -a https://ruby.taobao.org/
gem sources --remove https://rubygems.org/
gem sources -l
{% endhighlight %}
注意，此处用的是https，因为目前已经全面从http迁移到https，所以以前的http已经无法正常使用了。

## 使用CocoaPods索引
通过下面的命令可以添加CocoaPods索引，可以参考《iOS开发进阶》。
{% highlight bash %}
pod repo remove master
pod repo add master https://github.com/CocoaPods/Specs.git
pod repo update
{% endhighlight %}

接下来我们就可以开始使用CocoaPods了。
首先要学会写Podfile，具体可有参考[官方链接](https://guides.cocoapods.org/using/the-podfile.html),需要注意的是，写Podfile时，如果使用的源全是开源第三方库，可以使用下面这样的写法：
{% highlight ruby %}
#Podfile
platform:ios, '8.0'

pod 'AFNetworking', '3.0.4'
pod 'JSPatch', '~> 0.1.3'
{% endhighlight %}
但通常在一个公司团队开发时，常常会用团队自己的私有源，这时候可以用下面这样的写法，当然自己要有访问私有源的权限。
{% highlight ruby %}
source "https://github.com/CocoaPods/Specs.git"
source "git@bitbucket.org:mingyu_ye/myprivatepodsymy.git"

platform:ios, '8.0'

pod 'MYNewPrivatePod', '0.0.1'
pod 'MYCocoaPodsTestYMY', '0.0.1'

pod 'JSPatch', '~> 0.1.3'
{% endhighlight %}
然后将编辑好的Podfile放到工程的根目录下，然后进到工程目录下，执行以下命令：
{% highlight bash %}
pod install
{% endhighlight %}
以后打开工程时都使用*.xcworkspave打开工程。
当修改了Podfile后有时采用pod install，有时是pod update，区别可参考[官方链接](https://guides.cocoapods.org/using/pod-install-vs-update.html),很多人都错误的在所有情况下都使用pod update。

当想查找某个库时，可以使用以下命令：
{% highlight ruby %}
pod search AFNetworking
{% endhighlight %}
这样就搜出了AFNetworking相关信息。
Podfile上可以添加一些类似插件的东西，可以在[此处](https://rubygems.org/gems/)进行搜索，然后在Podfile中加上类似以下代码，即可实现插件对应的效果，有一个写的不错的[插件地址](https://github.com/mahaiyannn?tab=repositories)。
{% highlight ruby %}
require 'cocoapods-check_latest'
require 'cocoapods-sorted-search'
{% endhighlight %}
如果在使用第三方库时找不到头文件，则将```Build Setting```中```User Header Search Paths```设置成```${SRCROOT} recursive```即可。

我们不仅要会使用外面的库，还要会自己建立库供别人使用，要建立自己的库，那么就需要会写spec文件。
写的方法可以参考[官方文档](https://guides.cocoapods.org/making/specs-and-specs-repo.html)。
所有的spec文件都存放在~/.cocoapods目录当中，有兴趣可以进入到这个目录看一下。

如果要从服务器更新本地第三方库索引，可以使用下面这个命令：
{% highlight ruby %}
pod repo update master
{% endhighlight %}

建立私有库可参考[此链接](https://nicolastinkl.gitbooks.io/just-for-life/content/ios_private_cocoapods.html)，建立公开库和私有库的区别仅在于库是否放在私有权限中。
主要步骤如下：<br />
1.在cocoaPods上建立一个public spec或建立一个私有spec repo。<br />
2.创建Pod所对应的工程文件。<br />
3.创建Pod所对应的spec文件并在本地测试spec文件是否可用。<br />
4.spec文件中tag与version保持一致，pod search时是根据tag大小。<br />
5.向public的repo或private的repo提交spec文件，可用trunk方法进行提交，可参考[此链接](https://guides.cocoapods.org/making/getting-setup-with-trunk.html)，trunk只能添加，删除或修改tag只能走pull request路线，将cocoapods spec给fork下来，然后修改后pull request。<br />
6.本文中引用public和private的spec，public放在github上，private放在bitbucket上。<br />
7.最后在工程中Podfile引用库。<br />

spec文件如下所示：
{% highlight ruby %}
Pod::Spec.new do |s|

  s.name         = "MYNewPrivatePod"
  s.version      = "0.0.1"
  s.summary      = "MYNewPrivatePod Source."
  s.description  = <<-DESC
                   MYNewPrivatePod Source description
                   DESC

  s.homepage     = "https://bitbucket.org/mingyu_ye/mynewprivatepod"

  s.license = {
    :type => 'Copyright',
    :text => <<-LICENSE
           yemingyu copyright
    LICENSE
  }

  s.author             = { "yemingyu" => "me@yemingyu.com" }

  s.platform     = :ios

  s.ios.deployment_target = "8.0"

  s.source       = { :git => "https://bitbucket.org/mingyu_ye/mynewprivatepod.git", :tag => s.version}

  s.source_files  = "MYNewPrivatePod", "MYNewPrivatePod/**/*.{h,m,mm}"
  
  s.resources = "MYNewPrivatePod/**/*.{png,xib,wav,plist}"
  s.requires_arc = true
  s.prefix_header_file = "MYNewPrivatePod/MYNewPrivatePod-Prefix.pch"
  #如果对pods有依赖，就添加FRAMEWORK_SEARCH_PATHS
  #s.xcconfig = { 'FRAMEWORK_SEARCH_PATHS' => "'$(PODS_ROOT)/JSPATCH' '$(PODS_ROOT)/ReactiveCocoa'", 'OTHER_CFLAGS' => '-Xclang -fobjc-runtime-has-weak' , 'CLANG_ENABLE_MODULES' => 'NO' }
  #依赖的framework
  #s.framework = ''
  # non_arc_files = '' 
  #s.exclude_files = non_arc_files

  #s.dependency "JSONKit", "~> 1.4"

end
{% endhighlight %}

最后说一下.gitignore文件,通常在团队合作时不将Pods文件夹传到git上，因为通常太大太慢。

## Tips
** 是所有子目录，包括子目录的子目录。<br />
* 只是一级子目录。<br />
多个repo里的最好不要重名，包括私有的和公有的，以防conflict。

撰写本博客时还参考的链接如下：
<ul>
<li><a href="http://studentdeng.github.io/blog/2013/09/13/cocoapods-tutorial/">Cocoapods 入门</a></li>
<li><a href="http://ishalou.com/blog/2012/10/16/how-to-create-a-cocoapods-spec-file/">如何编写一个CocoaPods的spec文件</a></li>
<li><a href="http://www.exiatian.com/cocoapods%E5%AE%89%E8%A3%85%E4%BD%BF%E7%94%A8%E5%8F%8A%E9%85%8D%E7%BD%AE%E7%A7%81%E6%9C%89%E5%BA%93/">CocoaPods安装使用及配置私有库</a></li>
<li><a href="http://www.cocoachina.com/ios/20150228/11206.html">使用Cocoapods创建私有podspec</a></li>
<li><a href="http://www.cnblogs.com/wengzilin/p/4742530.html">iOS：手把手教你发布代码到CocoaPods(Trunk方式)</a></li>
</ul>

源码面前没有秘密，什么时候看官方文档都是一个很好地选择。

本文中涉及的[引用Pod代码地址](https://github.com/yemingyu/MYTestGetPods)，[Pod工程所在地址](https://github.com/yemingyu/MYCocoaPodsTest)。

