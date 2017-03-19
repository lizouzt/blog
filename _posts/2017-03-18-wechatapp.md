---
layout:     post
title:      微信小程序基础介绍
subtitle:   微信小程序基础框架及开发一个简单的微信小程序
date:       2017-03-18 17:00:00
author:     "chenming"
header-img: "img/post-bg-03.jpg"
---
## 一个简单的小程序长什么样

<img src="/blog/img/wechatapp/blacklist.jpg" width="350" style="display:inline-block;"/>
<img src="/blog/img/wechatapp/blacklistdetail.jpg" width="350" style="margin-left:20px;display:inline-block;"/>
## 微信小程序基础框架
一个微信小程序由一个App实例和若干个Page实例组成，App实例为小程序应用，监听并处理小程序的生命周期函数、声明全局变量等，Pages则表示先程序的多个页面。一个完整的页面由.wxml, .wxss, .js, .json几个文件组成。
一个完整的小程序目录结构如下：
<img src="/blog/img/wechatapp/structure.jpg" width="350" style="display:inline-block;"/>

## 配置文件
app.json 应用的全局配置文件，app.json通常主要配置以下内容：
```json
{
  "pages":[
    "pages/index/index",
    "pages/blacklist/blacklist"
  ],
  "window":{
    "backgroundTextStyle":"light",
    "navigationBarBackgroundColor": "#fff",
    "navigationBarTitleText": "云莱坞",
    "navigationBarTextStyle":"#333"
  },
  "tabBar": {
    "list": [{
      "pagePath": "pages/index/index",
      "text": "首页",
      "iconPath": "",
      "selectedIconPath": ""
    }, {
      "pagePath": "pages/blacklist/blacklist",
      "text": "黑名单",
      "iconPath": "",
      "selectedIconPath": ""
    }]
  },
  "networkTimeout": {
    "request": 10000,
    "downloadFile": 10000
  },
  "debug": true
}
```
* pages: 页面路由，当前应用需要加载哪些页面，数组第一项为小程序的初始页面。
* window: 用于设置小程序的状态栏，导航条，标题以及窗口背景色。
* tabBar: 只能配置最少2个、最多5个 tab，tab按数组的顺序排序，可设置tabBar的名称和要展示的页面以及icon
* networkTimeout: 配置小程序请求超时时间
* debug: 调试模式开关

page.json 页面的全局配置文件

* 每一个小程序页面也可以使用.json文件来对本页面的窗口表现进行配置，页面中配置项会覆盖 app.json 的 window 中相同的配置项。
* enablePullDownRefresh: 是否开启下拉刷新
* disableScroll: 控制当前页面是否可以上下滚动

## App
``` App() ``` 函数用来注册一个小程序
> App() 必须在 app.js 中注册，且不能注册多个。

```  javascript
App({
  onLaunch: function () {
    //调用API从本地缓存中获取数据
    var logs = wx.getStorageSync('logs') || []
    logs.unshift(Date.now())
    wx.setStorageSync('logs', logs)
  },
  getUserInfo:function(cb){
    var that = this
    if(this.globalData.userInfo){
      typeof cb == "function" && cb(this.globalData.userInfo)
    }else{
      //调用登录接口
      wx.login({
        success: function () {
          wx.getUserInfo({
            success: function (res) {
              that.globalData.userInfo = res.userInfo
              typeof cb == "function" && cb(that.globalData.userInfo)
            }
          })
        }
      })
    }
  },
  globalData:{
    userInfo:null
  }
})
```

## Page
``` Page() ``` 函数注册一个页面
* 通常包含：
> 初始化数据
> 生命周期函数，如onLoad（页面加载），onShow（页面显示）等
> 页面相关事件处理函数，如onPullDownRefresh（下拉刷新），onShareAppMessage（用户分享）等

``` javascript
Page({
    data:{
        iplist:null
    },
    onLoad:function(){
        var self = this;
        wx.request({
          url: 'https://api.yunlaiwu.com/ip/iphotlist',
          header: {
            'content-type': 'application/json'
            },
    success: function(res) {
    console.log(res.data)
      self.setData({
          iplist:res.data.data
          });
        }
        })
    }
});
```
Page onLoad时url中传递的参数会被写入options中

``` javascript
wx.navigateTo({
  url: 'blacklistdetail?id=currtId'
});
Page({
  onLoad: function(options){
    console.log(options.id);
  }
});
```
* 事件处理函数

```
<view bindtap="viewTap"> click me </view>

```

```
Page({
  viewTap: function() {
    console.log('view tap')
  }
})

```
* setData
> ```setData()```函数用于将数据从逻辑层发送到视图层，同时改变对应的 this.data 的值,所以不要直接对this.data赋值

* 页面路由
在小程序中所有页面的路由全部由框架进行管理，小程序提供了两种方法，一种是调用api，如：wx.navigateTo，另一种是使用navigator组件，如：``` xml <navigator open-type="navigate"/>```


## WXML模版语言
* WXML是微信小程序的一套标签语言，包含数据绑定，条件渲染，列表渲染，模版，事件，引用等，不能使用html渲染，也不能外链跳转。

## WXSS

* 选择器目前只支持#id, .class, element以及::after, ::before
* 全局样式可以写在app.wxss中，页面样式写在对应的pages.wxss中，会覆盖app.wxss中相同选择器的样式。
* 尺寸单位使用rpx(responsive pixel)，可以根据屏幕宽度进行自适应。规定屏幕宽为750rpx。如在 iPhone6 上，屏幕宽度为375px，共有750个物理像素，则750rpx = 375px = 750物理像素，1rpx = 0.5px = 1物理像素。

<img src="/blog/img/wechatapp/rpx.jpg" width="750" style="display:inline-block;"/>

组件
* 微信小程序提供了常用的组件，包括视图，导航，媒体，地图，画布等等。

[组件文档](https://mp.weixin.qq.com/debug/wxadoc/dev/component/slider.html)

问题
* swiper以及fullpage滚动效果不佳。
* canvas，rpx存在部分机型的兼容性问题。
* 不可自定义组件，开发复杂的应用比较困难。
* 开发工具不完善。
* 不支持requestAnimationFrame。

## 连接
[简易教程](https://mp.weixin.qq.com/debug/wxadoc/dev/)
[微信小程序redux绑定](https://github.com/charleyw/wechat-weapp-redux)
