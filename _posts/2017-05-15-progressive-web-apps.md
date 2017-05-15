---
layout:     post
title:      "Progressive Web Apps (PWA) 渐进式网页应用程序"
subtitle:   ""
date:       2017-05-15
author:     "waka"
comments:	true
header-img: "/img/FE/pwa.png"
---
<style>
.site-heading h1{
  color: #000411;
}
</style>
# Progressive Web Apps (PWA) 渐进式网页应用程序

> 参考文章：

> [Progressive Web Apps - Web | Google Developers](https://developers.google.com/web/progressive-web-apps/)

> [你需要开发PWA应用吗？](https://sanwen8.cn/p/6aeBjXT.html)

> [改造你的网站，变身 PWA
](http://www.tui8.com/articles/news/108995.html)

> [我们真的需要网页版App吗?Google *PWA*的困局](https://www.baidu.com/link?url=O0GFwM4yABaS8NjM9uOm3S9sRRE-78WcHHZ7YEygm3ItmniD_c5JvCnAuxmX3u_Yara3TamCXDTaUdvMeGODjNGRW3I1HbA8sIfYeuIWuFW&wd=&eqid=fbb3152e000884910000000259187bfc)


![](http://upload-images.jianshu.io/upload_images/1828354-baafd5aa96ba1f50.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 一个问题

作为一个端工程师，他的意义是什么？这里的端包括各种终端，PC、Android、iOS等等等等..

我认为主要有两点：
1. 数据的展示
2. 用户的交互

而用户的交互这里最重要的就是用户体验。

现在是移动端的时代，在移动端，App体验要比网页好很多，那么它好在哪里？

App: 
- 离线使用
- 消息推送
- 沉浸式体验
- 访问设备的硬件能力

Web:
- 不需要从App Store下载
- 动态更新

它们彼此的优点正式对方的缺点，那么有没有一种技术可以把两者的优势结合起来呢？

# What? PWA是什么? 

渐进式网页应用（Progressive Web Apps，简称PWA）是一个新的概念，它弥合了网站（Website）和移动应用（Mobile App）之间的差异。它们能够确保离线功能的可用性，并且能够提升速度和性能”。

### 怎样理解渐进式(Progressive)？

搜索下PWA的历史就会发现、它第一次出现于Googler Alex Russell的博客文章《[Progressive Web Apps: Escaping Tabs Without Losing Our Soul](https://infrequently.org/2015/06/progressive-apps-escaping-tabs-without-losing-our-soul/)》中，其主要观点是：**Web的发展方向应该是“在保留灵魂的基础上渐进增强”，而非现在大行其道的Hybrid App方向。**

Progressive 主要有这两层含义：
> 1. 如果用户需要，网页可以渐进式地变成App，比如被**添加到主屏幕**、**全屏方式运行**、**离线工作**、**推送通知消息**等。
2. 但它仍是**Web**而非放到**App Store**里。简单的来说，就是不用下载，用一个url就能直接打开。

所有这些“使得Web更能与App匹敌”的特性都是以渐进的方式增强的，在比传统网页应用更好的同时也保证了降级兼容。

PWA要实现的目的，或者说要解决的痛点，就是利用一系列现代Web技术的组合，以在移动设备上提供最好的体验（媲美原生App）。

# Why? PWA的主要目标

- 改进用户体验
- 加强访问者的参与度
- 提高转化率

### PWA的优点

- 离线模式

- 给人的感觉是应用，但运行机制是网站

- 提高性能

- 能在设备上快速安装

- 推送通知 （push notifications)

- 不必提交到应用软件商店（App Store）

##### 离线模式

网站在某些情况下是有局限性的，在涉及到互联网连接的时候更是如此；没有网络连接时，网站即便能够显示出来，也不可能正常运行。而另一方面，移动应用通常是自包含的（self-contained），这方便用户离线浏览，从而显著增加了用户参与度和软件可用性。这也是App和网页最重要的区别。

通过保存访问者已访问过的信息来实现。这意味着任何时候，即使是没有连接网络的时候，访问者都可以访问渐进式网页应用已访问过的页面。

在没有网络连接的情况下，当用户浏览到先前未访问过的页面时，不是在浏览器中提示错误信息，而是可能显示一个定制的离线页面。该页面可能会显示品牌Logo和基本信息，有时甚至是更先进的功能，旨在吸引用户停留在该页面上。

很明显，这样做的好处在于增加了访客留在该网站上的可能性，而不是促使用户关闭浏览器，等有了网络连接再继续使用。

这已经成为移动应用大幅增长的主要原因之一。

因访问者在离线模式下也可以访问产品目录，这使得企业有大幅提高他们的客户留存率和参与度的可能。

##### 给人的感觉是应用，但运行机制是网站

渐进式网页应用的主要卖点在于外观和体验通常会类似于移动应用，让用户在熟悉的环境下操作，同时仍然具有动态数据和数据库访问的全部网站功能。

像网站一样，渐进式网页应用可以通过URL访问，因此可以通过搜索引擎进行索引。这意味着可以在搜索引擎，比如Google和Baidu上找到该页面。与所有内部数据只能局限于内部访问的移动应用相比，这是一个巨大的优势。

根据项目要求，渐进式网页应用可以设计成与现有的企业网站或移动应用完全相同，也可以有意设计成有所不同以便让用户感知他们正在浏览渐进式网页应用。甚至可以将渐进式网页应用无缝地集成到现有的网站/应用程序的结构和设计中。

在Google进行的同一项研究中，我们发现所有网站访问者中有11.5％接受并下载了相应的渐进式网页应用。这对任何类型的网站来说都是很高的转化率。

##### 提高性能

渐进式网页应用的速度要明显快得多，这要归功于底层技术能够缓存和提供文本、样式表、图片以及Web站点上的其他内容。

这得益于服务工作者（service worker），它们的运行独立于Web站点，只请求原始数据，而不涉及任何样式或布局信息。

显然，速度的提升可以改善用户体验和提高留存率。同时，很多报告显示优化性能也能显著的提高转化率，这可从销售角度来说增加了渐进式网页应用的价值。

![](http://upload-images.jianshu.io/upload_images/1828354-ba126c33552acf50.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

(Controlled 代表由 Service Worker 控制页面，Supported 代表默认浏览器缓存)

第一个表格显示的是桌面用户的加载时间。用户使用服务工作者加载网页的时间与使用浏览器加载缓存内容的时间相比减少了29％。

对移动设备而言，性能仍然有明显提高，虽然不及桌面应用，但加载时间还是减少了22％。

值得注意的是，在两种测试中的第三行都基于首次访问的数据，因此无论是否安装服务工作者，结果是一样的 。这是因为服务工作者只有在二次访问时才起作用。

##### 能在设备上快速安装

另外很有意思的一点是在于，当用户访问网站时，一些浏览器会自动提示用户安装渐进式网页应用。这是通过浏览器自身所实现的唤起行动（call to action）来实现的。这使得渐进式网页应用更可信，同时增值了它的权威性和可靠性。
与移动应用相比，用户安装渐进式网页应用时无需很长的下载时间。同时，用户不会被转到Google Play或App Store，而是直接将应用程序下载到他们的设备上。

这意味着渐进式网页应用就像移动应用一样，在手机和平板电脑上有自己的图标，但无需经历乏味和缓慢的应用商店提交过程。

![](http://upload-images.jianshu.io/upload_images/1828354-1d6cdfbe1c2d0f23.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

此外，安装了渐进式网页应用的用户还可以在其主屏幕上看到图标，这会在用户每次使用手机时提醒他品牌名称和产品。这可带来宝贵的品牌意识。

![从左至右: App、小程序、PWA](http://upload-images.jianshu.io/upload_images/1828354-70b43de67ca94139.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 推送通知

渐进式网页应用可选择实现各种设备特定的硬件功能，例如推送通知。软件发布商和开发人员可以完全控制如何实现这个功能，从而为通知新内容提供创新的解决方案。

直接显示在手机上的推送通知的读取次数要远远超过电子邮件形式的新闻信札以及社交媒体上的状态更新等。

但是如果用户安装许多应用程序和渐进式网页应用，通过推送通知发布最新产品、博客帖子（blog posts）、文章或其他相关信息， 可能会导致用户的通知区域杂乱无章。

![](http://upload-images.jianshu.io/upload_images/1828354-a1a687555134305b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

感谢2016年7月的Google研究。

在所有下载渐进式网页应用的用户中，将近60％都授予渐进式网页应用发布推送通知的权限， 不过还有36.3％的用户没有点开推送通知，或者由于渐进式网页应用的个人设置没有收到推送通知。

将此数字与有多少网站访问者从主页上下载渐进式网页应用的统计数据结合起来，我们可以估计大约6-7％的网站现有流量能够转换为接受推送通知的渐进式网页应用用户。

##### 不必提交应用软件商店

随着需遵守的监管点不断增加，在Google Play、Windows Phone Apps或Apple App Store发布应用程序可能是一个乏味和耗时的过程。

通过使用渐进式网页应用，开发人员无需等待批准就可以推送新的更新，并且能在传统移动应用目前无法实现的级别上进行定期更新。

用户重新运行渐进式网页应用时，系统会自动下载更新。并且，可以通过推送通知，让用户获知应用更新已下载。而且，这同样不是强制性的，软件发布商可以完全控制将什么内容和信息推送给用户。

##### 面临的困难

- 缺乏通用支持
  - 以下有些重要信息需要注意，主要是并非所有浏览器都支持渐进式网页应用。
Google Chrome和Opera这两个浏览器对服务工作者和渐进式网页应用给与了极大的支持。
  - 苹果的Safari浏览器目前仍然不提供渐进式网页应用支持，虽然有消息说他们会考虑，但迄今为止没有任何具体的内容发布。
  - 微软表示他们将在2016年7月之前在Edge上实施渐进式网页应用，但目前仍然没有关于这方面的消息。
  - 然而，即使不是所有的浏览器都支持渐进式网页应用，对不兼容浏览器的用户也不会造成任何问题，因为这些浏览器只是忽略了渐进式网页应用，依然能够像往常一样显示网站。

- 不列在应用商店目录中
  - 有些人可能会认为自己的渐进式网页应用没有列在应用商店中会降低曝光率，但通常情况并非如此。
  - 事实上，与移动应用相比，渐进式网页应用可以通过Google或其他搜索引擎上搜索到，这与网站类似，而与移动应用有所不同。这意味着数十亿的日常搜索可能最终导致搜索到渐进式网页应用。

- 有限的本地硬件支持
  - 与移动应用本地化设计不同，渐进式网页应用不能100%支持给定手机上的所有硬件功能。
  - 虽然渐进式网页应用支持普通访问的功能，如加速器（Accelerometer）、摄像机和麦克风，但有一些功能需要由本机的移动应用来完成。

# How？怎样开发？

开发一个 PWA 并不难。事实上，我们可以将现存的网站进行改进，使之成为PWA。

Google 引领了 [PWA 的一系列动作](https://developers.google.com/web/progressive-web-apps/) ，所以大多数教程都在说如何从零开始构建一个基于 Chrome，native-looking mobile app。然而并不是只有特殊的单页应用可以PWA化，也不需要一定遵循 material interface design guidelines。大多数网站都可以在数小时内实现 PWA 化。这包括WordPress站点或者静态站点。

将你的网站改进为一个 Progressive Web App 总共有三个必要步骤：

### 第一步：开启 HTTPS

由于一些显而易见的原因(Service Worker 权限相当大)，所以https的网络环境是非常必要的；本地调试时 Chrome 允许使用 localhost 在 HTTP 连接下测试PWA

### 第二步：创建一个 Web App Manifest

manifest 文件提供了一些我们网站的信息，例如 name，description 和需要在主屏使用的图标的图片，启动屏的图片等。

manifest文件是一个 JSON 格式的文件，位于你项目的根目录。它必须用 Content-Type: application/manifest+json 或者 Content-Type: application/json 这样的 HTTP 头来请求。这个文件可以被命名为任何名字，在示例代码中他被命名为 /manifest.json :
```
{
  "name"              : "PWA Website",
  "short_name"        : "PWA",
  "description"       : "An example PWA website",
  "start_url"         : "/",
  "display"           : "standalone",
  "orientation"       : "any",
  "background_color"  : "#ACE",
  "theme_color"       : "#ACE",
  "icons": [
    {
      "src"           : "/images/logo/logo072.png",
      "sizes"         : "72x72",
      "type"          : "image/png"
    },
    {
      "src"           : "/images/logo/logo152.png",
      "sizes"         : "152x152",
      "type"          : "image/png"
    },
    {
      "src"           : "/images/logo/logo192.png",
      "sizes"         : "192x192",
      "type"          : "image/png"
    },
    {
      "src"           : "/images/logo/logo256.png",
      "sizes"         : "256x256",
      "type"          : "image/png"
    },
    {
      "src"           : "/images/logo/logo512.png",
      "sizes"         : "512x512",
      "type"          : "image/png"
    }
  ]
}
```

在页面的 <head> 中引入：
```
<link rel="manifest" href="/manifest.json">
```

manifest 中主要属性有：

- *name* —— 网页显示给用户的完整名称

- *short_name* —— 当空间不足以显示全名时的网站缩写名称

- *description* —— 关于网站的详细描述

- *start_url* —— 网页的初始 相对 URL（比如 /
 ）

- *scope* —— 导航范围。比如， /app/
 的scope就限制 app 在这个文件夹里。

- *background-color* —— 启动屏和浏览器的背景颜色

- *theme_color* —— 网站的主题颜色，一般都与背景颜色相同，它可以影响网站的显示

- *display* —— 首选的显示方式： fullscreen, standalone (看起来像是native app)， minimal-ui(有简化的浏览器控制选项) 和 browser (常规的浏览器 tab)

- *icons* —— 定义了 src URL, sizes 和 type的图片对象数组。

MDN提供了完整的manifest属性列表: [Web App Manifest properties](https://developer.mozilla.org/en-US/docs/Web/Manifest)

### 第三步：创建一个 Service Worker

> [使用*Service* *Workers* - Web API 接口 | MDN](https://www.baidu.com/link?url=8O-BIE3Vxi0MIjjI9482zKZxKdoqnNkiSbM7z-c-ipguT75w5eDtD4munEEU0rumEW2upHDjMhOOvNaDxfKkV5xggRNgH0apuogLthxGMu2uly89PXGT23ndtDqAZjexNkADMYRl9WNxtaKX4X9INq&wd=&eqid=e18caed80009b9200000000259189497)

> 打开链接，打开编辑器

Service Worker 是拦截和响应你的网络请求的编程接口。这是一个位于你根目录的一个单独的 javascript 文件。

![](http://upload-images.jianshu.io/upload_images/1828354-66c4733774f9aac0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

你的 js 文件（在示例代码中是 /js/main.js ）可以检查是否支持 Service Worker，并且注册：
```
if ('serviceWorker' in navigator) {

  // register service worker
  navigator.serviceWorker.register('/service-worker.js');

}
```

如果你不需要离线功能，可以简单的创建一个空的 /service-worker.js 文件 —— 用户会被提示安装你的 app。

Service Worker 很复杂，你可以修改示例代码来达到自己的目的。这是一个标准的 web worker，浏览器用一个单独的线程来下载和执行它。它没有调用 DOM 和其他页面 api 的能力，但他可以拦截网络请求，包括页面切换，静态资源下载，ajax请求所引起的网络请求。

这就是需要 HTTPS 的最主要的原因。想象一下第三方代码可以拦截来自其他网站的 service worker， 将是一个灾难。

service worker 主要有三个事件： install ， activate 和 fetch 。

##### Install 事件

这个事件在app被安装时触发。它经常用来缓存必要的文件。缓存通过 [Cache API](https://developer.mozilla.org/en-US/docs/Web/API/Cache)来实现。

首先，我们来构造几个变量：

- 缓存名称（ CACHE ）和版本号（ version ）。你的应用可以有多个缓存但是只能引用一个。我们设置了版本号，这样当我们有重大更新时，我们可以更新缓存，而忽略旧的缓存。

- 一个离线页面的URL（ offlineURL ）。当离线时用户试图访问之前未缓存的页面时，这个页面会呈现给用户。

- 一个拥有离线功能的页面必要文件的数组（ installFilesEssential ）。这个数组应该包含静态资源，比如 CSS 和 JavaScript 文件，但我也把主页面（ / ）和图标文件写进去了。如果主页面可以多个URL访问，你应该把他们都写进去，如 /和 /index.html 。注意， offlineURL 也要被写入这个数组。

- 可选的，描述文件数组（ installFilesDesirable ）。这些文件都很会被下载，但如果下载失败不会中止安装。

```
// configuration
const
  version = '1.0.0',
  CACHE = version + '::PWAsite',
  offlineURL = '/offline/',
  installFilesEssential = [
    '/',
    '/manifest.json',
    '/css/styles.css',
    '/js/main.js',
    '/js/offlinepage.js',
    '/images/logo/logo152.png'
  ].concat(offlineURL),
  installFilesDesirable = [
    '/favicon.ico',
    '/images/logo/logo016.png',
    '/images/hero/power-pv.jpg',
    '/images/hero/power-lo.jpg',
    '/images/hero/power-hi.jpg'
  ];
```

installStaticFiles() 方法添加文件到缓存，这个方法用到了基于 promise的 [Cache API](https://developer.mozilla.org/en-US/docs/Web/API/Cache) 。当必要的文件都被缓存后才会生成返回值。
```
// install static assets
function installStaticFiles() {

  return caches.open(CACHE)
    .then(cache => {

      // cache desirable files
      cache.addAll(installFilesDesirable);

      // cache essential files
      return cache.addAll(installFilesEssential);

    });

}
```

最后，我们添加 install 的事件监听函数。 waitUntil 方法确保所有代码执行完毕后，service worker 才会执行 install。执行 installStaticFiles() 方法，然后执行 self.skipWaiting() 方法使service worker进入 active状态。
```
// application installation
self.addEventListener('install', event => {

  console.log('service worker: install');

  // cache core files
  event.waitUntil(
    installStaticFiles()
    .then(() => self.skipWaiting())
  );

});

```

##### Activate 事件

当 install完成后， service worker 进入active状态，这个事件立刻执行。你可能不需要实现这个事件监听，但是示例代码在这里删除老旧的无用缓存文件：

```
// clear old caches
function clearOldCaches() {

  return caches.keys()
    .then(keylist => {

      return Promise.all(
        keylist
          .filter(key => key !== CACHE)
          .map(key => caches.delete(key))
      );

    });

}

// application activated
self.addEventListener('activate', event => {

  console.log('service worker: activate');

    // delete old caches
  event.waitUntil(
    clearOldCaches()
    .then(() => self.clients.claim())
    );

});
```
注意，最后的 self.clients.claim() 方法设置本身为active的service worker。

##### Fetch 事件

当有网络请求时这个事件被触发。它调用 respondWith()方法来劫持 GET 请求并返回：

- 缓存中的一个静态资源。

- 如果 #1 失败了，就用 [Fetch API](https://developer.mozilla.org/en/docs/Web/API/Fetch_API) （这与 service worker 的fetch 事件没关系）去网络请求这个资源。然后将这个资源加入缓存。

- 如果 #1 和 #2 都失败了，那就返回一个适当的值。

```
// application fetch network data
self.addEventListener('fetch', event => {

  // abandon non-GET requests
  if (event.request.method !== 'GET') return;

  let url = event.request.url;

  event.respondWith(

    caches.open(CACHE)
      .then(cache => {

        return cache.match(event.request)
          .then(response => {

            if (response) {
              // return cached file
              console.log('cache fetch: ' + url);
              return response;
            }

            // make network request
            return fetch(event.request)
              .then(newreq => {

                console.log('network fetch: ' + url);
                if (newreq.ok) cache.put(event.request, newreq.clone());
                return newreq;

              })
              // app is offline
              .catch(() => offlineAsset(url));

          });

      })

  );

});
```
最后这个 offlineAsset(url) 方法通过几个辅助函数返回一个适当的值：
```
// is image URL?
let iExt = ['png', 'jpg', 'jpeg', 'gif', 'webp', 'bmp'].map(f => '.' + f);

function isImage(url) {

  return iExt.reduce((ret, ext) => ret || url.endsWith(ext), false);

}


// return offline asset
function offlineAsset(url) {

  if (isImage(url)) {

    // return image
    return new Response(
      '<svg role="img" viewBox="0 0 400 300" xmlns="http://www.w3.org/2000/svg"><title>offline</title><path d="M0 0h400v300H0z" fill="#eee" /><text x="200" y="150" text-anchor="middle" dominant-baseline="middle" font-family="sans-serif" font-size="50" fill="#ccc">offline</text></svg>',
      { headers: {
        'Content-Type': 'image/svg+xml',
        'Cache-Control': 'no-store'
      }}
    );

  }
  else {

    // return page
    return caches.match(offlineURL);

  }

}
```
offlineAsset() 方法检查是否是一个图片请求，如果是，那么返回一个带有 “offline” 字样的 SVG。如果不是，返回 offlineURL 页面。

开发者工具提供了查看 Service Worker 相关信息的选项：

![](http://upload-images.jianshu.io/upload_images/1828354-1a2404cbc12292eb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
在开发者工具的 *Cache Storage* 选项列出了所有当前域内的缓存和所包含的静态文件。当缓存更新的时候，你可以点击左下角的刷新按钮来更新缓存：

![](http://upload-images.jianshu.io/upload_images/1828354-4af5921cd43e8d31.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
不出意料， *Clear storage* 选项可以删除你的 service worker 和缓存：

![](http://upload-images.jianshu.io/upload_images/1828354-9505a2bd774f6de1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 开发工具

如果你觉得 javascript 调试困难，那么 service worker 也不会很好。Chrome的开发者工具的 *Application* 提供了一系列调试工具。

你应该打开 *隐身窗口* 来测试你的 app，这样在你关闭这个窗口之后缓存文件就不会保存下来。

最后， [Lighthouse extension for Chrome](https://chrome.google.com/webstore/detail/lighthouse/blipmdconlkpinefehnmjammfjpmpbjk) 提供了很多改进 PWA 的有用信息。

# PWA 陷阱

有几点需要注意：

### URL 隐藏

我们的示例代码隐藏了 URL 栏，我不推荐这种做法，除非你有一个单 url 应用，比如一个游戏。对于多数网站，manifest 选项 display: minimal-ui 或者 display: browser 是最好的选择。

### 缓存太多

你可以缓存你网站的所有页面和所有静态文件。这对于一个小网站是可行的，但这对于上千个页面的大型网站实际吗？没有人会对你网站的所有内容都感兴趣，而设备的内存容量将是一个限制。即使你像示例代码一样只缓存访问过的页面和文件，缓存大小也会增长的很快。

也许你需要注意：

- 只缓存重要的页面，类似主页，和最近的文章。

- 不要缓存图片，视频和其他大型文件

- 经常删除旧的缓存文件

- 提供一个缓存按钮给用户，让用户决定是否缓存

### 缓存刷新

在示例代码中，用户在请求网络前先检查该文件是否缓存。如果缓存，就使用缓存文件。这在离线情况下很棒，但也意味着在联网情况下，用户得到的可能不是最新数据。

静态文件，类似于图片和视频等，不会经常改变的资源，做长时间缓存没有很大的问题。你可以在HTTP 头里设置 Cache-Control
 来缓存文件使其缓存时间为一年（31,536,000 seconds）：
```
Cache-Control: max-age=31536000
```
页面，CSS和 script 文件会经常变化，所以你应该改设置一个很短的缓存时间比如 24 小时，并在联网时与服务端文件进行验证：
```
Cache-Control: must-revalidate, max-age=86400
```