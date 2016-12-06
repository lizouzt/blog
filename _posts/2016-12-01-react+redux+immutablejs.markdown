---
layout:     post
title:      "如何用React+Redux+ImmutableJS进行SPA开发"
subtitle:   "如何用React+Redux+ImmutableJS进行SPA开发"
date:       2016-12-01 13:00:00
author:     "stefan"
header-img: "img/post-bg-05.jpg"
---

## 综述

使用`React + Redux + Immutable.js`进行富客户端的开发是云莱坞在2016-2017年进行前端研发的基本技术栈，但不意味着在任何时候对它进行滥用，应该至少满足如下三个条件：

> 1. 数据集合较庞大、数据关联性较强
> 2. 业务流程较复杂
> 3. 绝大多数子业务可被抽象为复用的视图或者组件

因此，比如`一个简单的静态页面`、`一个虽然数据项很多但是仅仅为纯表单的页面`都不满足上述条件，因此不建议使用该架构。

至于为什么使用 React，主要有如下三个原因：

> 1. React是一个生态圈健壮的以解决组件化开发为目标的前端框架，文档全面；
> 2. 由于virtual DOM的存在，使得平台兼容性强，后续公司可基于改方案落地RN到客户端研发；
> 3. React支持服务端同构渲染（虽说Vue也支持吧），方便后续做基于SSR的RTT优化。

至于为什么使用 Redux，而不使用 Flux 或者 ReFlux，原因在于 Redux 单一数据集合以及绑定策略节省了大量的前端代码，并且对数据进行集中维护。结合单向数据流的概念，强迫组件更加存粹。

而对 Immutable.js 的使用，则得益于他本身“不可变数据”的特性，以及本身包含的巨多语法糖。

本文不是单纯的技术介绍和教程，更多的还是对公司内部基于这套技术体系的编码指导，尽量不讲 How，而是谈 What 和 Why。

本文从如下几个方面进行拆解：

* React + Redux 如何进行集成开发，以及注意的点
* Immutable.js 如何整合进开发流程
* 性能优化
* 调试工具和部署方案

## React + Redux 如何进行集成开发

## Immutable.js 如何整合进开发流程

一张图解释 Immutable 如何使用 Structural Sharing（结构共享，即如果对象树中一个节点发生变化，只修改这个节点和受它影响的父节点，其它节点则进行共享）来避免 deepCopy 把所有节点都复制一遍带来的性能损耗：

<img src="http://img.alicdn.com/tps/i2/TB1zzi_KXXXXXctXFXXbrb8OVXX-613-575.gif" />

至于 Immutable 的好处，之于实际的项目，在我看来，并不是什么神乎其技的 time travel（不过在基于 DraftJS 做编辑器的时候这个特性很有用），而是在于以下两点（对于 Immutable.js 带来的应用的状态可预见的好处就不赘述了）：

**第一点**，丰富的语法糖，有了ImmutableJS，代码中就不会再出现如下的东东:

	```javascript

	//为了在不污染原对象的前提下增加新的KV
	var state = Object.assign({}, state, {
		key: value
	});

	//为了在不污染原数组的前提下插入新元素
	var state = [
		...state.slice(0, index),
		insertData,
		...state.slice(index + 1)
	];

	```

有时候，为了保证`reducer`在处理 state 的时候不会改变传入的 state，就要写大量的上述代码。这种感觉就和吃屎一样，但是有了 Immutable.js，你就可以不用吃屎了：

	```javascript

	var state = state.set('key', value);

	var state = state.splice(index, 1, value);

	```

世界一下子清净了。

**第二点**，性能的提升。由于 immutable 内部使用了 Trie 数据结构来存储，只要两个对象的 `hashCode` 相等，值就是一样的。这样的算法避免了深度遍历比较，性能非常好。这对我们在进行具体渲染过程中的性能优化非常有用。

既然这么好，Immutable.js 如何同现有的 React + Redux 技术方案进行集成呢？好在有`redux-immutable`（事实上还有一个叫做`redux-immutablejs`的库也能实现二者的集成）这么一个库，在它的帮助下，可以实现 Immutable.js 的植入。在介绍集成方案之前，我们有必要先划分数据使用的边界，就是在那些地方使用 Javascript 原生数据结构（简称为JSD），哪些地方使用 immutable，如下图所示：



## 调试工具和部署方案





