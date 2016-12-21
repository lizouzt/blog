---
layout:     post
title:      "NodeJS在cluster模式下内存超限重启"
subtitle:   "没有PM2的情况下怎么办"
date:       2016-12-21 13:00:00
author:     "stefan"
comments:	true
header-img: "img/post-bg-06.jpg"
---

## 为何要进行内存超限重启

在代码层面，我们有很多种方式来防止NodeJS在执行的过程中出现内存泄露。但是当业务逻辑复杂的情况下，很难做到完全无内存泄露。这时候，一个偷懒的办法就是要让服务进程在超过一定内存阈值的时候重启。

直接杀死进程的方式是不推荐的，因为会影响正在处理中的 request。在 cluster 模式下，node对每一个 fork 出的子进程提供了优雅的退出方式：

```javascript
cluster.worker.disconnect()
```

他会保证子进程退出前处理完所有正在进行中的 request。

所以，我们可以在内存超出一定阈值的时候，调用`disconnect`方法。

## 如何检测子进程内存占用量

使用`usage`模块可以检测进程内存占用量（PS，在 node 0.12 版本下未能安装，在 4.x 版本下可以）：

usage.lookup(process.pid, function(err, result) {

	if (result === null || result === undefined) {
        console.log("memory check fail");
        return;
    }

    //Bytes
    console.info(parseInt(result.memory));

});

## 在内存超限时重启

在内存超过一定阈值的情况下重启子进程的 cluster 模式下的进程管理的代码如下：

```javascript

//in memory.js

var cluster = require('cluster');
var usage = require('usage');
var os = require('os');

var CPU_COUNT = process.env.CPU_COUNT;
var CHECK_INTERVAL = process.env.CHECK_INTERVAL;

var cpuCount = CPU_COUNT || os.cpus().length;
var checkInterval = CHECK_INTERVAL || 5000;

module.exports = {
    run: function(bytes, runFunc, cleanFunc) {

        if (cluster.isMaster) {
            for (var i = 0; i < cpuCount; i++) {
                cluster.fork();
            }
            //注意点B
            cluster.on('disconnect', function(worker) {
                console.log('' + worker.id + ' disconnect, restart now');
                worker.isSuicide = true;
                cluster.fork();
            });
            cluster.on('exit', function(worker) {
                if (worker.isSuicide) {
                    console.info('process exit by kill');
                } else {
                    console.info('process exit by accident');
                    cluster.fork();
                }
                console.info('process exit');
            });
        } else {

            runFunc && runFunc();

            var checkTimer = setInterval(function() {

                usage.lookup(process.pid, function(err, result) {

                    if (result === null || result === undefined) {
                        console.log("memory check fail");
                        return;
                    }
                    if (parseInt(result.memory) > bytes) {
                        console.log("memory exceed, start to kill");

                        //注意点A
                        var killtimer = setTimeout(function() {
                            console.info("process down!")
                            process.exit(1);
                        }, 5000);
                        killtimer.unref();

                        cleanFunc && cleanFunc();

                        try {
                            if (['disconnected', 'dead'].indexOf(cluster.workder.state) < 0) {
                                cluster.worker.disconnect();
                            }
                        } catch (err) {};

                        clearInterval(checkTimer);
                    }
                });

            }, checkInterval);
        }

    }
}
```

代码里有两个注意点：

### 注意点A

其实这里是参考了 domain 的文档：[https://nodejs.org/api/domain.html](https://nodejs.org/api/domain.html)，我们这里设置若干秒后将进程推出。

但是我们发现，这里调用了 `killtimer.unref()`，这里是为了防止定时器的存在阻止程序退出（PS：国内几乎所有的书都在说 unref 的作用是阻止回调调用，其实不然，[https://cnodejs.org/topic/570924d294b38dcb3c09a7a0](https://cnodejs.org/topic/570924d294b38dcb3c09a7a0)）。

### 注意点B

`disconnect`调用时，我们就要 fork 出新的子进程，但是有些情况下，进程会意外退出。在意外退出时，我们也要 fork 新的子进程补位，这时候就要区分到底是意外退出还是程序退出，否则就会 fork 冗余的子进程。

cluster 的官方文档有说明可以通过`worker.suicide`和`worker.exitedAfterDisconnect`来判断进程是否是意外退出，但遗憾的是，node 的某些中间版本因为 bug （比如 4.x）失去了对这两个flag的支持，所以这里我们通过自己设置标志位`isSuicide`来判断是否意外退出。

## 使用方法

直接调用 export 出的 run 方法，设置超限阈值，设置子进程服务，设置重启回调：

```javascript
var memory = require('./memory');

memory.run(400000000, function() {
	require('./server.js');
}, function() {
	console.info('clean now!');
});
```

这里要注意的是，内存的阈值要低于**总内存/进程数**，要给重启时 fork 的新进程（因为那时候的老进程还未退出）以及系统上的其他服务留有内存空间。

最后一句话：重启大法虽然好，但是也要防止内存泄露。
