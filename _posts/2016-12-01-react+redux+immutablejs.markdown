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
> 2. 由于virtual DOM的存在，使得平台兼容性强，后续公司可基于该方案落地RN到客户端研发；
> 3. React支持服务端同构渲染（虽说Vue也支持吧），方便后续做基于SSR的RTT优化。

至于为什么使用 Redux，而不使用 Flux 或者 ReFlux，原因在于 Redux 单一数据集合以及绑定策略节省了大量的前端代码，并且对数据进行集中维护。结合单向数据流的概念，强迫组件更加纯粹。

而对 Immutable.js 的使用，则得益于他本身“不可变数据”的特性，以及本身包含的巨多语法糖。

本文不是单纯的技术介绍和教程，更多的还是对公司内部基于这套技术体系的编码指导，尽量不讲 How，而是谈 What 和 Why。

本文从如下几个方面进行拆解：

* React + Redux 如何进行集成开发，以及注意的点
* Immutable.js 如何整合进开发流程
* 性能优化
* 调试工具和部署方案

## 目录

[React + Redux 如何进行集成开发](#1)

* [到底该在什么地方进行 state 的初始化](#11)
* [如何捕获异步 action 的回调](#12)
* [Redux store vs React state](#13)
* [应该在什么位置获取异步数据](#14)

[Immutable.js 如何整合进开发流程](#2)

* [为啥要使用 immutable](#21)
* [使用 immutable 的边界性问题](#22)
* [如何集成 immutable 到流程中](#23)

[性能优化](#3)

[调试工具和部署方案](#4)

## <a name="1"></a>React + Redux 如何进行集成开发

其实 Redux 的<a href="http://redux.js.org/docs/basics/UsageWithReact.html">官方文档</a>已经将整体的集成方案讲的很明白了，本文只是针对一些“为什么这么做”、“该怎么做”做一些最佳实践的补充。这个子章节也会不断的扩展的：

### <a name="11"></a>到底该在什么地方进行 state 的初始化

这个问题回答得比较好的是 stackoverflow 的<a href="http://stackoverflow.com/questions/33749759/read-stores-initial-state-in-redux-reducer/33791942#33791942">这个帖子</a>：

> TL;DR
>
> Without combineReducers() or similar manual code, initialState always wins over state = ... in the reducer because the state passed to the reducer is initialState and is not undefined, so the ES6 argument syntax doesn't apply.
>
> With combineReducers() the behavior is more nuanced. Those reducers whose state is specified in initialState will receive that state. Other reducers will receive undefined and because of that will fall back to the state = ... default argument they specify.
>
> In general, initialState wins over the state specified by the reducer. This lets reducers specify initial data that makes sense to them as default arguments, but also allows loading existing data (fully or partially) when you're hydrating the store from some persistent storage or the server.

### <a name="12"></a>如何捕获异步 action 的回调

在使用 thunk 中间件处理异步 action 的时候，标准的做法：

```javascript

export function asyncAction(item) {
	return (dispatch, getState) => {
		dispatch(requestItem(item));
		return fetch("api.some_url/items/item")
			.then(response => response.json())
			.then(json => {
				if (json.success) {
					dispatch(receivePostsSuccess(reddit, json));
				} else {
					dispatch(receivePostsFail(reddit, json));
				}
			});
		}
	};
}

```

如果想在异步回调结束后触发一些操作，比如系统通知tip，我们允许在局部针对返回的 promise 进行进一步处理：

```javascript
this.props.dispatch(asyncAction(item)).then(onSuccess, onFailure);
```

但是，如果你的通知不是局部的，而是可以抽象为一个通用的通知系统，建议通过增加一个针对于通知系统的子 store 来解决这个问题，为他配置单独的 action 和 reducer。

### <a name="13"></a>Redux store vs React state

试想一种情况，我们做一个 todo 应用，需要展示 todo 的列表，也要对单独的 todo 进行分别编辑，那么 store 和 state 的设计无非就是如下两种：

<img src="/blog/img/react+redux+immutablejs/1.png" style="display: block; margin: 10px auto; width: 800px; height: auto;" />

<img src="/blog/img/react+redux+immutablejs/2.png" style="display: block; margin: 10px auto; width: 800px; height: auto;" />

前面这种，将单独的 todo 的状态放在了 store 里，而后面这种是放在了 todo 组件的局部的 state 里。

一般来讲推荐后面这种，原因有二：

1. todo 组件是复用的，如果放在 store 里，在 SPA 的应用中，每次进入都要**重置状态**；
2. 用作局部的 state，使得维护起来更加方便，因为在这个 case 里，对 todo 的临时操作**并不影响全局**。

在提交每个 todo 的时候，肯定要派发 store 中的 action（因为提交后整个 todolist 都会全局性的改变）。如果用前一种方案，todo 组件必然和 store 绑定在了一起，所以可以在 todo 组件内部去 dispatch action。而对于后者，则应该是通过 todolist 传入的回调方法去添加或者更新 todo 信息。

总结一下就是，使用前一种方式就不要画蛇添足通过父子组件传递回调的方式去派发事件，因为 todo 组件已经不是独立的组件了，已经耦合到大的系统里了，没必要增加复杂度了。而使用后一种方式就不要直接派发事件给 store，因为后一种方式的子组件本来设计思路就是解耦的。

[https://github.com/reactjs/redux/issues/1287](https://github.com/reactjs/redux/issues/1287)里的高票回答得很好：

> Use React for ephemeral state that doesn't matter to the app globally and doesn't mutate in complex ways. For example, a toggle in some UI element, a form input state. Use Redux for state that matters globally or is mutated in complex ways. For example, cached users, or a post draft.
>
> Sometimes you'll want to move from Redux state to React state (when storing something in Redux gets awkward) or the other way around (when more components need to have access to some state that used to be local).
>
> The rule of thumb is: do whatever is less awkward.

简而言之就是，如果这个状态不影响其他部分的业务逻辑并且足够简单，就使用 react state。

### <a name="14"></a>应该在什么位置获取异步数据

问：是不是所有的异步数据都要在 store 的 action 中获取？

答案是：否！前面那个小节已经讲过了局部 state 和全局 store 的关系，如何选择完全看数据是否和全局有关联。推此即彼，异步数据的获取也同样。如果获取到的数据是用来更新全局状态，那么当然要放在 store 的 action里。否则，直接在组件内部调用封装好的异步请求去改变局部的 state 就可以了。

## <a name="2"></a>Immutable.js 如何整合进开发流程

本章节分为三点：`为啥要使用 immutable`、`使用 immutable 的边界性问题`、`如何集成 immutable 到流程中`。

### <a name="21"></a>为啥要使用 immutable

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

### <a name="22"></a>使用 immutable 的边界性问题

既然这么好，Immutable.js 如何同现有的 React + Redux 技术方案进行集成呢？好在有`redux-immutable`（事实上还有一个叫做`redux-immutablejs`的库也能实现二者的集成）这么一个库，在它的帮助下，可以实现 Immutable.js 的植入。在介绍集成方案之前，我们有必要先划分数据使用的边界，就是在哪些地方使用 Javascript 原生数据结构（简称为JSD），哪些地方使用 immutable，哪些地方需要做二者的转化，如下图所示：

<img src="/blog/img/react+redux+immutablejs/flow.png" style="display: block; margin: 10px auto; width: 800px; height: auto;" />

* 在 React 视图里，props 其实就来自于 Redux 维护的全局的 state 的，所以 props 中的每一项一定是 immutable 的。
* 在 React 视图里，组件自己维护的局部 state **如果是用来提交到 store 的**， 必须为 immutable 的，否则不强制。
* 从视图层向同步和异步 action 发送的数据（A/B），必须是 immutable 的。
* Action 提交给 reducer 的数据（C/D），必须是 immutable 的。
* reducer 处理后所得 state (E)当然一定是 immutable 的。

这样似乎看起来，所有地方都是 immutable 的，但是其实异步 action 和服务器的交互当然是 JSD （F）。换句话说，我们要求，除了向服务端发送数据请求的时候，其他位置，不允许出现`toJS`的代码。而接收到服务端的数据后，在流转入全局 state 之前，统一转化为 immutable 数据。

为什么要做这种统一呢？是因为：

* 避免 JSD 和 immutable 的混用导致出错；
* 统一 JSD 和 immutable 的转化路径。比如你在局部 state 里不使用 immutable，但是这些局部的 state 很可能被用来通过异步 action 提交的服务端，这样在数据流里就会同时存在 JSD 和 immutable，这对于代码的维护性是一种灾难。

有的人说，我觉得成本太大，我只想在 reducer 里局部使用 immutable，于是代码变成了这样：

```javascript

export default function(state, action) {
	state = state.fromJS(state);
	....
	return state.toJS();
}

```

这样看起来只是让 immutable 侵入到 reducer 中，其实却是得不偿失的。因为`toJS`和`fromJS`会消耗大量的性能。

### <a name="23"></a>如何集成 immutable 到流程中

按照 Redux 的工作流，我们从创建 store 开始。Redux 的 createStore 可以传递多个参数，前两个是： reducers 和 initialState。

reducers 我们用 `redux-immutable` 提供的 combineReducers 来处理，他可以将 immutable 类型的全局 state 进行分而治之：

```javascript

const rootReducer = combineReducers({
	routing: routingReducer,
	a: immutableReducer,
	b: immutableReducer
});

```

当然 initialState 需要是 immutable 的：

```javascirpt

const initialState = Immutable.Map();
const store = createStore(rootReducer, initialState);

```

如果你不传递 initialState，`redux-immutable`也会帮助你在 store 初始化的时候，通过每个子 reducer 的初始值来构建一个全局 Map 作为全局 state。当然，这要求你的每个子 reducer 的默认初始值是 immutable的。

接下来，你会发现，`react-router-redux`的接入也要改造，因为 routerReducer 是不兼容 immutable 的，所以你必须自定义 reducer：

```javascript

import Immutable from 'immutable';
import {
	LOCATION_CHANGE
} from 'react-router-redux';

const initialState = Immutable.fromJS({
	locationBeforeTransitions: null
});

export default (state = initialState, action) => {
	if (action.type === LOCATION_CHANGE) {
		return state.set('locationBeforeTransitions', action.payload);
	}
 	return state;
};

```

除此之外，还要让`react-router-redux`能够访问到挂载在全局 state 上的路由信息：

```

import {
	browserHistory
} from 'react-router';
import {
	syncHistoryWithStore
} from 'react-router-redux';

const history = syncHistoryWithStore(browserHistory, store, {
	selectLocationState (state) {
		return state.get('routing').toObject();
	}
});

```

处理好 store 创建、reducer 集成、路由控制，接下来改处理 connect 链接，因为 connect 本身只支持 plain Object，所以需要将数据转成 connect 能支持的格式。但这并不意味着你要这么干：

```javascript

@connect(state => state.toJS())

```

这种传递方式本质上和上面提及的只在 reducer 里使用 immutable 是一样一样的，会带来巨大的性能开销。正确的方式是，将绑定到 props 的 state 转化为属性为 immutable 的 Object 对象：

```javascript

@connect(state => state.toObject())

```

当然，以上的例子是整个转化过去，你也可以按需绑定对应组件所关心的 state。

细心的人可能会发现，在使用 immutable 维护全局的 state 的情况下，组件 props 的校验也需要与时俱进，使用 immutable 类型校验，这就需要我们 import 专门针对 immutable 类型进行校验的库：`react-immutable-proptypes`，使用方法基本上和普通的 PropTypes 一致：

```javscript

propTypes: {
    oldListTypeChecker: React.PropTypes.instanceOf(Immutable.List),
    anotherWay: ImmutablePropTypes.list,
    requiredList: ImmutablePropTypes.list.isRequired,
    mapsToo: ImmutablePropTypes.map,
    evenIterable: ImmutablePropTypes.iterable
}

```

与此同时，产生defaultProps的地方应该为：

```javascript

fromJS({
	prop1: xxx,
	prop2: xxx,
	prop3: xxx
}).toObject();

```

## <a name="3"></a>性能优化

[https://github.com/lcxfs1991/blog/issues/8](https://github.com/lcxfs1991/blog/issues/8)这篇文章写的很好了。其实关键点就在拆包和 shouldComponentUpate。

使用他提供的 decorator 来减少 render 的触发。

## <a name="4"></a>调试工具和部署方案

调试当然用的就是`react-devtools`和`redux-devtools`。

`react-devtools`是一个 Browser Extension，主要用于针对组件级别的 Props 和 state 跟踪：

<img src="https://github.com/facebook/react-devtools/raw/master/images/devtools-full.gif" />

`redux-devtools`则同时支持 Browser Extension 和用户自定义调试，集成略微复杂。如果用 Browser Extension 需要在 store 定义时注入代码，如果纯自定义，要考虑的东西就更多了，参看文章[https://github.com/gaearon/redux-devtools/blob/master/docs/Walkthrough.md](https://github.com/gaearon/redux-devtools/blob/master/docs/Walkthrough.md)。





