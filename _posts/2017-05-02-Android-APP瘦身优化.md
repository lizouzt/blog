---
layout:     post    
title:      "APP瘦身"    
subtitle:   "每一点的改进都是对用户体验的爱心"          
date:       2017-05-02            
author:     "sunlang"                      
comments:	true
header-img: "img/post-bg-01.jpg"
---
## 前言：
随着app版本的更新迭代，我们的apk文件越来越大。之前一个apk也就是2MB左右，到了现在，一个apk文件大小已经升到20MB＋。apk文件大小的爆炸增长主要因为app集成的基础业务越来越多，用户对app质量的期待越来越高。主要表现在以下几个方面：

1.基础业务越来越多，一个app要承当更多的功能。

2.Android设备多样化，开发者不得不做多套兼容方案。

3.开源类库越来越丰富，导致大量重复的业务在一个app里面出现。

4.用户对app的视觉要求越来越高，有的app可能会出现500KB甚至1MB的图片。

在大部分情况下，apk大小的增长主要是为了满足用户的需求和期待。app应该是一个轻量级的，在满足基础功能的前提下，尽量让apk的大小降到最低。如果一个用户花了10分钟才从市场get到你的app，那将是很糟糕的一件事情。所以，我们要对app进行瘦身。

## 一、Android应用编译及运行流程：

![ipv6.png](/blog/img/android_apk/android_apk.png)


## 二、APK的文件结构：
![ipv6.jpg](/blog/img/android_apk/apk_file.jpg)

各文件的介绍如下：

1. lib：lib目录下的子目录armeabi,x86存放的是项目用到的so文件。

2. classes.dex：classes.dex是java源码编译后生成的java字节码文件。

3. res：res目录存放资源文件。包括图片、字符串、raw文件夹下面的音频文件、各种xml文件等等。

4. assets：assets目录可以存放一些配置文件。

5. resources.arsc：编译后的二进制资源文件

6. META-INF：META-INF目录下存放的是签名信息，用来保证apk包的完整性和系统的安全。

7. AndroidManifest.xml：该文件是每个应用都必须定义和包含的，它描述了应用的名字、版本、权限、引用的库文件等信息。



## 三、APK的优化瘦身：
从apk产生的流程图中可以看出，APK中classes.dex、lib、资源文件是大头，APK瘦身主要就是优化这三个文件

### classes.dex
通过代码混淆，减小类名、方法名和变量的名长度；删掉不必要的jar包和代码实现该文件的优化；

### lib：
一个硬件设备对应一种架构（mips、arm或者x86），只保留与设备架构相关的库文件夹（主流的架构都是arm的，mips属于小众，默认也是支持arm的so的，但x86的不支持），这样可以大大降低lib文件夹的大小
###  资源文件：

* 手动lint检查，手动删除代码中没有引用到的资源，实际效果不等。在Android Studio中打开“Analyze” 然后选择"Inspect Code..."，范围选择整个项目，然后点击"OK"

![ipv6.png](/blog/img/android_apk/unused_res1.png) 

![ipv6.png](/blog/img/android_apk/unused_res2.png) 

* gradle脚本中开启shrinkResources,shrinkResources配合minifyEnabled使用效果更佳

![ipv6.png](/blog/img/android_apk/shrink_resources.png) 

* 一套图、一套布局，多套dimens.xml文件，在使用最小资源的情况下搞定多分辨率适配；目前项目中图片都放在xxhdpi文件夹中

* tinypng有损压缩


  <https://tinypng.com/>

Android打包本身会对png进行无损压缩，大家可以看看apk中的图片的大小实际上比你代码工程里的图片要小（针对没进行过无损压缩的那些png图）。
所以，纯粹的进行无损压缩并不会对apk的减小有任何效果。
现在大家主流的比较喜欢用的tinypng其实是有损压缩：

 ```
    [原文] TinyPNG uses smart lossy compression techniques to reduce the file size of your PNG files…
    [翻译] TinyPNG使用智能有损压缩技术，来减少PNG文件的大小…
```


tinypng是一个支持压缩png和jpg图片格式的网站，通过其独特的算法（通过一种叫“量化”的技术，把原本png文件的24位真彩色压缩为8位的索引演示，是一种矢量压缩方法，把颜色值用数值123等代替。）可以实现在无损压缩的情况下图片文件大小缩小到原来的30%-50%。



tinypng的缺点是在压缩某些带有过渡效果（带alpha值）的图片时，图片会失真，这种图片可以将png图片转换为下面介绍的webP格式，可以在保证图片质量的前提下大幅缩小图片的大小。

* png换成jpg

一些背景，启动页，宣传页的PNG图片比较大，这些图片图形比较复杂，如果转用有损JPG可能只有不到一半（当然是有损，不过通过设置压缩参数可以这种损失比较小到忽略）。
因为都是大图，所以这种方式能有效减小apk的大小。
这种情况下的apk的减小是不可估量的。


* jpg换成webp

如果png大图转成jpg还是很大，或者想压的更小，而尽量不降低画质，那么可以考虑一下webp。

WebP是谷歌研发出来的一种图片数据格式，它是一种支持有损压缩和无损压缩的图片文件格式，派生自图像编码格式 VP8。根据 Google 的测试，无损压缩后的 WebP 比 PNG 文件少了 45％ 的文件大小，即使这些 PNG 文件经过其他压缩工具压缩之后，WebP 还是可以减少 28％ 的文件大小。目前很多公司已经将webP技术运用到Android APP中，比如FaceBook、腾讯、淘宝。webP相比于png最明显的问题是加载稍慢，不过现在的智能设备硬件配置越来越高，这都不是事儿。

假如你打算在 App 中使用 WebP，除了 Android4.0 以上提供的原生支持外，其他版本以可以使用官方提供的解析库webp-android-backport编译成so使用。


```
    1. android 4.0+才原生支持webp, 所以4.0以下的设备将无法看到图片，但是也不会崩溃。
    2. 我们的应用就是支持4.0以上的设备，所以可以考虑
```


* 覆盖第三库里的大图

有些第三库里引用了一些大图但是实际上并不会被我们用到，就可以考虑用1×1的透明图片覆盖。有些用到的大图也可以新切一个小图代替，前提保证显示效果不变


* 使用tintcolor实现按钮反选效果：

通常按钮的正反旋图片我们都是通过提供一张按钮正常图片和一张按钮反选图片，然后通过selector实现，两张图片除了alpha值不一样外其它的内容都是重复的，在Android 5.0及以上的版本可以通过tintcolor实现只提供一张按钮的图片，在程序中实现按钮反选效果，前提是图片的内容一样，只是正反选按钮的颜色不一样。


* 用更小的库替代方案，比如只用到了谷歌统计，那么就不要把整个google play services都集成进来，只集成需要的部分。

* 定期清理废弃的代码，定期删除无用的逻辑和过期的业务功能模块，以及废弃的test代码。

* 业务模块采用插件化框架，代码动态从云端拉取，插件化，这是另外一个话题了，这里不赘述。

## 四、瘦身后的效果：

![ipv6.jpg](/blog/img/android_apk/1234.jpg)





 