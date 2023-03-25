# Fiber

React Element 无法表达 React 节点之间的关系，只能保证内部包含哪些数据，数据存储单元

React 相比于 Vue 没有模板优化的过程

Fiber - 介于 React Element 和真实节点之间的一种节点，并且可以表达节点之间的关系，

此外它不仅可以是数据存储单元，也可以是工作单元，是虚拟 Dom 在 React 中的实现

Fiber 实际上就是 React 中的虚拟 DOM

React16 以前，Reconciler 采用深度优先遍历创建虚拟 DOM，递归的过程不可中断，如果组件的层级很深，会导致占用线程时间很长，在用户的界面看起来就会有一大片空白，很卡顿

React16 以后，将递归的不可中断的更新变成**异步**的可中断更新，并在 React18 时正式提出 concurrent，这些实现与 Fiber 架构是离不开的

FiberNode 具体到底长什么样子？
https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiber.new.js#L117

```javascript
function FiberNode(
  tag: Worktag,
  pendingProps: mixed,
  key: null | string,
  mode: TypeOfMode
) {
  // 作为静态结构的属性
  // this.tag - 对应组件的类型 Function / Class
  this.tag = tag;
  this.key = key;
  this.elementType = null;
  this.type = null;
  // Fiber对应的真实DOM节点
  this.stateNode = null;

  // 实际上Fiber是一个链表
  this.return = null;
  this.child = null;
  this.sibling = null;

  this.index = 0;
  this.ref = null;

  // 作为动态工作单元的一些属性
  this.pendingProps = pendingProps;
  this.memoizedProps = null;
  this.updateQueue = null;
  this.memoizedState = null;
  this.dependencies = null;

  this.mode = mode;

  //本次更新会造成的DOM操作
  this.effectTag = NoEffect;
  this.nextEffect = null;

  this.firstEffect = null;
  this.lastEffect = null;

  // 调度优先级相关
  this.lanes = NoLanes;
  this.childLanes = NoLanes;

  // 指向该fiber在另一次更新时对应的fiber，React中有一个双缓存机制，current和workInProgress是相互交替更新的，用alternate去进行更新
  this.alternate = null;
}
```

# 双缓存

双缓存 Fiber Tree

- 屏幕上正在显示的：current Tree
- 即将更新的：workInProgress Tree
- 二者通过 alternate 属性进行连接

```javascript
currentFiber.alternate = workInProgressFiber;
workInProgressFiber.alternate = currentFiber;
```

二者的切换通过 current 指针在不同的 Fiber tree 之间进行切换，最后 Renderer 的时候根据 placement 进行更新

## 组件创建

ReactDOM.render()  
会先创建 rootFiberNode（根 Fiber）和 rootFiber（所在组件树的根节点）

## 如何创建 Fiber Tree

递归的过程

递 - rootFiber 开始向下 dfs，为每个 fiber 节点调用 beginWork 方法
当遍历到叶子节点时，会进入到归阶段

归 - 调用 completeWork 处理 Fiber 节点

beginWork:

- current: 当前组件对应的 fiber 节点在上一次更新时的 Fiber 节点
- workInProgress：当前组件对应的 Fiber 节点

beginWork-会根据传入的 Fiber 节点创建子 Fiber 节点，并把这两个 Fiber 节点连起来，当遍历到叶子节点的时候会进入到归阶段

completeWork - 当某个节点执行完 completeWork，如果其存在兄弟 Fiber 节点，会进入到它兄弟 Fiber 节点的递阶段，如果不存在，就进入到父节点的归阶段
