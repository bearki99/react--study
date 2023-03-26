#  Hooks

Function Component对应的FiberNode -> memoizedState 指向的是hooks链表

链表的每个元素又有自己的memoizedState去保存自己的状态

这也是为什么我们为什么不能在条件语句里面去使用hooks

``` javascript
interface Hook {
    memoizedState: any;
    updateQueue: unknown;
    next: Hook | null;
}
```

``` javascript
const HooksDispatcherOnMount = {
    useState:
}
    
function mountState(initialState) {
    const hook = mountWorkInProgressHook();
    let memoizedState;
    if(initialState instanceof Function) {
        memoizedState = initialState();
    }else {
        memoizedState = initialState;
    }
    const queue = createUpdateQueue();
    hook.updateQueue = queue;
    const dispatch = dispatchSetState.bind(null, currentlyRenderingFiber);
    return [memoizedState, dispatch];
}

function dispatchSetState<State>(fiber: FiberNode, updateQueue: UpdateQueue<State>, action: Action<State>) {
    const update = createUpdate(action);
    enqueueUpdate(updateQueue, update);
    scheduleUpdateFiber(fiber);
}

function mountWorkInProgressHook(): Hook {
    const hook: Hook = {
        memoizedState: null,
        updateQueue: null,
        next: null
    }
    if(workInProgressHook === null) {
        // 相当于
        if(currentlyRenderingFiber === null) {
            throw new Error("请在函数组件内调用Hook")
        }else {
            workInProgress = hook;
            currentlyRenderingFiber.memoizedState = workInProgressHook;
        }
    }else {
        workInProgressHook.next = hook;
        workInProgressHook = hook;
    }
    return workInProgressHook;
}
```

