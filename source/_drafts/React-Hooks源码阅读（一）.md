---
title: React Hooks源码阅读（一）
date: 2021-04-29 18:11:00
tags:
---

本文着重研究一下React Hooks的源码，React Hooks的源码中也穿插着Fiber，本文以Hooks为线索，先忽略Fiber的部分。  
这里以 `useState` 和 `useReducer` 钩子为例子展开 

<!-- more -->

## `useState` 和 `useReducer` 的简单demo

### `useState` 的官方文档demo：
```js
import React, { useState } from 'react';

function Example() {
  // 声明一个叫 "count" 的 state 变量
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

### `useReducer` 的demo：
```js
import React, { useReducer } from 'react';

const initialState = { count: 0 }

const Reducer = (state, action) => {
	switch (aciton.type) {
		case 'plus':
			return {...state, count: state.count + action.count }
		case 'minus':
			return {...state, count: state.count - action.count }
		default:
			return null
	}
}

const Counter = () => {
	const [state, dispatch] = useReducer(Reducer, initialState)
	return <div>
		<div> Count: {state.count} </div>
		<button onClick={()=> dispatch({ type: 'plus', count: 1 })} > + </button>
		<button onClick={()=> dispatch({ type: 'minus', count: 1 })} > - </button>
	</div>
}
```

## 追溯 `useState` 和 `useReducer` 源码

从 `import { useState, useReducer } from 'react'` 可以看出须要找到 `useState` 和 `useReducer` 这两个函数，从全局搜索 `useState` 或 `useReducer`，可以找到隐藏在如下路径：

`packages\react\src\ReactHooks.js` 
```js

export function useState<S>(
  initialState: (() => S) | S,
): [S, Dispatch<BasicStateAction<S>>] {
  const dispatcher = resolveDispatcher();
  return dispatcher.useState(initialState);
}

export function useReducer<S, I, A>(
  reducer: (S, A) => S,
  initialArg: I,
  init?: I => S,
): [S, Dispatch<A>] {
  const dispatcher = resolveDispatcher();
  return dispatcher.useReducer(reducer, initialArg, init);
}
```

可见他们都 `return dispatcher` 方法，`dispatcher` 执行函数 `resolveDispatcher()` ——>    

`resolveDispatcher` 函数中定义了 `const dispatcher = ReactCurrentDispatcher.current;` ——>    

找到 `import ReactCurrentDispatcher from './ReactCurrentDispatcher';` ———>

`packages\react\src\ReactCurrentDispatcher.js` 文件很简单，如下：

```js
import type {Dispatcher} from 'react-reconciler/src/ReactInternalTypes';

/**
 * Keeps track of the current dispatcher.
 */
const ReactCurrentDispatcher = {
  /**
   * @internal
   * @type {ReactComponent}
   */
  current: (null: null | Dispatcher),
};

export default ReactCurrentDispatcher;
```

最后找到源码在 `packages\react-reconciler\src\ReactFiberHooks.new.js` 里

Hooks都有两个定义：都有 挂载 `HooksDispatcherOnMount` 和 更新 `HooksDispatcherOnUpdate` 两个阶段
这里为了方便展示只放了 `useReducer` 和 `useState` :

```js
const HooksDispatcherOnMount: Dispatcher = {
  useReducer: mountReducer,,
  useState: mountState,
};

const HooksDispatcherOnUpdate: Dispatcher = {
  useReducer: updateReducer,
  useState: updateState,
};
```

继续看 `mountState` 函数：
```js
function mountState<S>(
  initialState: (() => S) | S,
): [S, Dispatch<BasicStateAction<S>>] {

  // 3. 获取当前hook节点，将当前hook添加到链表当中：
  const hook = mountWorkInProgressHook();
  if (typeof initialState === 'function') {
    // $FlowFixMe: Flow doesn't like mixed types
    initialState = initialState();
  }
  hook.memoizedState = hook.baseState = initialState;

  // 4. 声明一个链表存放更新：
  const queue = (hook.queue = {
    pending: null,
    interleaved: null,
    lanes: NoLanes,
    dispatch: null,
    lastRenderedReducer: basicStateReducer,
    lastRenderedState: (initialState: any),
  });

  // 5. 返回一个dispatch方法用来修改状态，并将此次更新添加update链表中
  const dispatch: Dispatch<
    BasicStateAction<S>,
  > = (queue.dispatch = (dispatchAction.bind(
    null,
    currentlyRenderingFiber,
    queue,
  ): any));

  // 6. 返回当前状态和改状态的函数：
  return [hook.memoizedState, dispatch];
}
```

以上注释的第3，是创建链表的步骤，可找到链表创建如下：

```js
// 3.1 hook链表的源头：
function mountWorkInProgressHook(): Hook {
  // 3.1.1 例如：const [initialState, useXxx ] = useState(0) 
  const hook: Hook = {
    // 3.1.1.1 相当于上面的0
    memoizedState: null,
    // 3.1.1.2 相当于上面的initialState
    baseState: null,
    // 3.1.1.3 尚需处理的update，通常是上一轮render中遗留下的优先级过低而暂缓执行的update
    baseQueue: null,
    // 3.1.1.4 当前触发的update链表
    queue: null,
    // 3.1.1.5 下一个调用hooks函数的地方
    next: null,
  };

  if (workInProgressHook === null) {
    // This is the first hook in the list
    // 3.2 链表为空，指向null，即第一个元素
    currentlyRenderingFiber.memoizedState = workInProgressHook = hook;
  } else {
    // Append to the end of the list
    // 3.3 链表不为空，添加在最后一个元素
    workInProgressHook = workInProgressHook.next = hook;
  }
  return workInProgressHook;
}
```
而在同一个函数里如果连续调用，如：
```js
const [count, setCount] = useState(0);
const [state, dispatch] = useReducer(Reducer, { count: 0 })
```

则对应的hook链表结构为：
```js
const hook = {
  memoizedState: 0,
  baseState: 0,
  baseQueue: null,
  queue: null,
  next: {
    memoizedState: { count: 0 },
    baseState: { count: 0 },
    baseQueue: null,
    queue: null,
	},
}
```

而我们注意到，hook里有一个 `queue` 的值来存放对象的更新状态，实质上 `queue` 也是一个链表，而且他是一个循环链表，
即形成, 下面是简化版代码：

```js
// 5.1 dispatch函数：即useXxx
function dispatchAction<S, A>(
  fiber: Fiber,
  queue: UpdateQueue<S, A>,
  action: A,
) {

  // 5.2 创造一个 update 对象，这部分也只有 useState useReducer 两个钩子会用到
  const update: Update<S, A> = {
    lane,
    action,
    // 触发 dispatch 的 reduer
    eagerReducer: null,
    // 触发 dispatch 时的 state
    eagerState: null,
    next: (null: any),
  };

  // 把update链接到更新队列中，组成单循环链表
  const pending = queue.pending;
  if (pending === null) {
    update.next = update;
  } else {
    update.next = pending.next;
    pending.next = update;
  }
  // pending指向链表的最后一个
  queue.pending = update;
  
  // 开启调度，触发新的一轮更新，也就是走beginWork,completeWork那一套流程。
  scheduleUpdateOnFiber(fiber, lane, eventTime);

```

每触发一次setState，会生成一个update对象，并链接到hook对象的更新队列中，也就是下文的pending。

```js
export type UpdateQueue<S, A> = {|
  // 存放当前触发的update
  pending: Update<S, A> | null,
  interleaved: Update<S, A> | null,
  lanes: Lanes,
  dispatch: (A => mixed) | null,
  // 上一次render的reducer
  lastRenderedReducer: ((S, A) => S) | null,
  // 上一次render的state
  lastRenderedState: S | null,
|};
```
例如：
```js
const [count, setCount] = useState(0);
setCount(1)
```

对应的结构为：
```js
const hook = {
  memoizedState: 0,
  baseState: 0,
  baseQueue: null,
  queue: {
    {
      action: 1,
    }
	},
  next: null,
}
```
在render 时，会遍历queue来执行每个update并计算更新。