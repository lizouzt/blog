---
layout:     post    
title:      "有关线上监控"    
subtitle:   ""          
date:       2017-05-31            
author:     "王青"                      
comments:	true
header-img: "img/post-bg-02.jpg"
---
	

## 线上监控目的
* 尽早的发现线上问题
	* __当前__ 的异常情况。
	* __一段时间内__ 的异常情况。


	
## 各种各样的监控
机器、服务情况、业务等等

不同种类的监控一般实施人员会有区别

* 运维、云平台服务提供商
* 开发
* 测试

运维一般做业务无关的监控，主要是 __机器、容器__ 相关，同时也可能包括一些服务（进程）的 __死活__ 监控等等。

一个活着的服务是不是提供了 __可用的服务__ ，则一般由开发来做。（不绝对）

开发和测试在这方面 __并没有明确的界限__ ，一般会有一个分工，以免产生重复劳动。

具体需要做哪些监控，取决于 __实际的需求__。

* 机器
	* 存活、CPU、内存、硬盘、带宽等等
	
* 服务
	* 进程存活、数量正确，等等
	* 是否正常响应

* HTTP 接口（通过实时请求检查返回的 header 、 body）
	* 是否 __200__ 或符合预期的 __301__ / __302__ 等
	* __errno__ 是否未 0 或者符合预期的其他错误码
	* __header__ 中关键字段是否正确
		* Location
		* Set-cookie
		* Content-Type
		* 等
	* __body__ 中字段是否正确
		* 根据需要对比数据库或其他存储介质
	* 响应时间

* 页面UI（一般通过 __模拟浏览器__ 行为来校验 __dom或截图__）
	* header 部分同 HTTP 接口
	* 页面元素是否存在或者样式是否正确
	
* 日志（从各机器采集日志来进行统计分析）
	* 4xx/5xx 一段时间内数量
	* 请求量（ __激增__ 或 __暴跌__ ）
	* 平均响应时间
	* 业务日志报错情况（ __提前日志输出__ ）

* 业务
	* 日志和数据库（或其他存储介质）是否一致
	* 各个存储介质之间数据一致性
		* 如订单与商品
	* 某些业务量的 __激增__ 或 __暴跌__

* 竞品 / 第三方
	* 新上架的商品
	* 价格变化
	* 排行榜


## 主要的流程
* 采集
	* 接口请求、日志拉取等
* 校验 / 统计
* 检查是否需要报警
	* 单词/连续失败
	* 一段时间内失败次数
	* 数量激增或暴跌
	* 数据不一致

* 报警
	* 短信、邮件、微信等
* 存储
	* 作为长期分析使用

## 目前的工作
接口监控、日志监控、页面元素监控（待完成）

使用 nodejs 进行编写

### 目录结构（参看代码）

* ylwlib // 存放自写的基础库和配置
	* config.js // 配置文件，包括邮件、部分阈值等等
	* req.js	  // HTTP 请求封装
	* mail.js   // 发邮件
	* sms.js	  // 发短信
	* util.js   // 一些公共方法，校验之类的
* apimon // 接口监控
	* apimon.js		// 单个 case 文件执行用
	* runApiMon.js	// 所有 case 执行的管理入口
	* cases			// 目录，存放各种 case 文件
	* var 				// 运行中需要的数据文件存放
* logmon	// 日志监控
	* logmon.js		// 日志监控执行文件
	* logmonCases.js // 日志监控各阈值配置文件
* uimon	// UI 监控（待完成）

### 有关执行

* 目前在测试机上执行， 由 crontab 维护
* 日志监控每小时执行一次
* 接口监控每分钟执行一次
	* 目前是串行，后续可以根据情况改成并行
* 日志监控每次执行的时候，会进行 __git pull__ , 意味着日常只要提交 __git__ ，就能保证下次监控使用最新的 case 列表。不需要再去调整已经部署完毕的监控框架。

* 一些小逻辑
	* 考虑到执行时间暂时不可控（比如超过1分钟），都有 __lock__ 文件。
	* 同时为了避免某些异常导致 __lock__ 文件不被清除，都有 __过期失效__ 逻辑。
	* 有关接口监控的报警阈值，目前是两个控制逻辑
		* 连续报警（目前是连续2次）
		* 最近N次成功失败记录
			* 当次失败（未达到连续报警阈值）
			* 最近 __N__ 次记录中，失败率达到 __M%__ 的，认为不稳定，也会报警。

### 有关接口监控报警文案

* 邮件，会另附上 URL 地址
* 短信不含 URL 地址
	* IP详情页（奶爸宣言） get_Chrome_producer: network FAILED
	* IP详情页（奶爸宣言） get_Chrome_producer: objId校验 FAILED
	* __URL描述__ + __get/post__ + __UA__ + __user[nologin/nopower/author/producer]__ + __检查项描述__ + FAILED
	* network 表示网络问题，比如自身网络失败，或者域名不可解析
	* 其他项基本都是 case 中配置的
* 所以写 case 的时候，需要用简单文案表达清楚 __监控的事项__


### 日志监控配置简单说明

直接参考文件 __logmonCases.js__


### 接口监控 Case 说明
部署路径
> apimon/cases

目前是按照 yunlaiwu.com 的前缀 m/user/api/www 来区分的目录，实际不影响任何监控功能。
目录可以按照需要来部署。

文件名举例
> sns\_user\_info.js

最好按照 url 的路径来取名，方面查阅。 一定要 **.js** 结尾，同时 符合 js 语法。

case 编写举例

```
'use strict';

/**
 * <必填> list cases 本身是个 list， 每个 item 根据 不同 情况， 如不同 请求参数区分。
 */
var cases = [
    {
        // <必填> string url:  url 地址， 可以是接口或者页面。
        url: 'https://api.yunlaiwu.com/sns/user/info?uid=57b4213a165abd0065c7ad54',

        // <选填> object qs:  querystring， 可以直接放 url 里
        // qs: {uid: '57b4213a165abd0065c7ad54'}, 

        // <选填> object form: 给 post 用的提交参数。
        // form: 

        // <选填> list<string> ua: 元素为 ylwlib/config.js 里的 userAgentList 的 key 。不填时，默认执行 ['Chrome']
        ua:  ['Chrome', 'iOS'],
        
        // <选填> list<string> method: 元素为  get 或 post ，  不填时，默认执行 ['get']
        method: ['get'],

        // <选填> list<string> method: 元素为  ylwlib/userlist.online 里 的 key， 未登录情况填 'nologin'。不填时，默认执行 ['nologin']
        user: ['nologin', 'producer'], // 元素为 

        // <必填> string desc: 描述，报警时描述使用。
        desc: '用户信息接口', 

        // <选填> bool followRedirect: 遇到 302 等跳转是否跟随跳转，默认是 true。 为 false 时， 302 将不会继续请求 location。
        // followRedirect: true,

        // <选填> object check: 需要检查的项。 不填时，默认校验 statusCode 是否为 200。
        check: {
            // <选填> string statusCode: 返回码， 一般是 200， 有些情况下可能是 302， 或其他。
            statusCode: 200,
            // <选填> string errnoKey: 接口里用来标示 errno 的 key，一般是 errno，某些时候可能 code，不填则不校验。（页面没必要填）
            errnoKey: 'errno',

            // <选填> list<object> funcs: 自定义的校验方法列表。
            funcs: [
                {
                    // <必填> string desc: 描述，报警时描述使用。
                    desc: 'objId校验',

                    // <必填> function func: 自定义的校验方法。 入参固定为 ({headers, body}) 用法简单如下，headers 里的参数可以自行打印查看。
                    // 请务必在成功时返回 true， 失败是返回 false， 否则可能影响监控。 
                    func: function ({headers, body}) {
                        try {
                            // 转成 对象
                            var body = body;

                            if (typeof body != 'object') {
                                body = JSON.parse(body);
                            }

                            if (body.data.objectId == '57b4213a165abd0065c7ad54') {
                                return true;
                            }
                        } catch (e) {
                            return false;
                        }
                        return false;
                    }
                },
            ]
        }
    },
];

// <必填> 通用写法，勿改动，请照抄
module.exports = {
    cases: cases,
}
```

日常增加新的 case 之后， 可以本地调试。

有关一些本地调试相关的事情在 README 上有说明
https://github.com/yunlaiwu/ylwautotest

## 后续工作
* UI监控的加入
	* 使用 phantomjs 作为组件
	* 目前主逻辑能跑通，具体的监控逻辑尚未完成
* 相关同学也可以尝试对新接口、页面进行自己的补充
	* 如有必要（ __请确保代码无误__ ），也可以对框架本身代码进行维护

* 谢谢 ^_^
	
	