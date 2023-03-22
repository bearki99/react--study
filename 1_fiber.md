#  Fiber
Fiber实际上就是React中的虚拟DOM  

React15以前，Reconciler采用深度优先遍历创建虚拟DOM，递归的过程不可中断，如果组件的层级很深，会导致占用线程时间很长，在用户的界面看起来就会有一大片空白，很卡顿

React16以后，将递归的不可中断的更新变成异步的可中断更新，并在React18时提出concurrent，这些实现与Fiber架构是离不开的

FiberNode具体到底长什么样子？
https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiber.new.js#L117
``` javascript
function FiberNode(tag: Worktag, pendingProps: mixed, key: null | string, mode: TypeOfMode) {
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

#  双缓存
双缓存Fiber Tree
- 屏幕上正在显示的：current Tree
- 即将更新的：workInProgress Tree
- 二者通过alternate属性进行连接

``` javascript
currentFiber.alternate = workInProgressFiber;
workInProgressFiber.alternate = currentFiber;
```
二者的切换通过current指针在不同的Fiber tree之间进行切换

##  组件创建
ReactDOM.render()  
会先创建fiberRootNode（根Fiber）和rootFiber（所在组件树的根节点）

##  如何创建Fiber Tree
递归的过程

递 - rootFiber开始向下dfs，为每个fiber节点调用beginWork方法
当遍历到叶子节点时，会进入到归阶段

归 - 调用completeWork处理Fiber节点

beginWork:
- current: 当前组件对应的fiber节点在上一次更新时的Fiber节点
- workInProgress：当前组件对应的Fiber节点