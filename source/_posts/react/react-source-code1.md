---
title: react源码解析_创建流程_rende阶段
tags: ['react']
---
首先我们从一个普通的react应用的入口开始：
### render函数
```javascript
    function render(element, container, callback) {
        if (!isValidContainer(container)) {}
        var isModernRoot = isContainerMarkedAsRoot(container) && container._reactRootContainer === undefined;
        if (isModernRoot) {}
        return legacyRenderSubtreeIntoContainer(null, element, container, false, callback);
    }
```
这个方法非常简单，接受三个参数element（react节点）, container（容器dom节点）, callback（成功回调函数）；
首先判断了container是不是一个有效的且没被react使用过的dom节点，然后调用了legacyRenderSubtreeIntoContainer这个方法。

#### legacyRenderSubtreeIntoContainer

```javascript 
function legacyRenderSubtreeIntoContainer(parentComponent, children, container, forceHydrate, callback) {

  var root = container._reactRootContainer;
  var fiberRoot;

  if (!root) {
    // Initial mount
    root = container._reactRootContainer = legacyCreateRootFromDOMContainer(container, forceHydrate);
    fiberRoot = root._internalRoot;

    if (typeof callback === 'function') {
      var originalCallback = callback;

      callback = function () {
        var instance = getPublicRootInstance(fiberRoot);
        originalCallback.call(instance);
      };
    } 
    unbatchedUpdates(function () {
      updateContainer(children, fiberRoot, parentComponent, callback);
    });
  } else {
    fiberRoot = root._internalRoot;

    if (typeof callback === 'function') {
      var _originalCallback = callback;

      callback = function () {
        var instance = getPublicRootInstance(fiberRoot);

        _originalCallback.call(instance);
      };
    } // Update
    updateContainer(children, fiberRoot, parentComponent, callback);
  }

  return getPublicRootInstance(fiberRoot);
}
```

这个方法主要做了两件事：
* 判断存不存在FiberRootNode，存在的话走创建流程，不存在则是更新，如果不存在则调用legacyCreateRootFromDOMContainer方法创建，在创建过成功调用markContainerAsRoot方法对容器节点进行标记；
* 调用更新方法updateContainer

#### updateContainer

```javascript
function updateContainer(element, container, parentComponent, callback) {
  var current$1 = container.current;
  var eventTime = requestEventTime();
  var lane = requestUpdateLane(current$1);
  var context = getContextForSubtree(parentComponent);

  if (container.context === null) {
    container.context = context;
  } else {
    container.pendingContext = context;
  }

  var update = createUpdate(eventTime, lane); // Caution: React DevTools currently depends on this property
  // being called "element".
  update.payload = {
    element: element
  };
  callback = callback === undefined ? null : callback;
  if (callback !== null) {
    update.callback = callback;
  }

  enqueueUpdate(current$1, update);
  scheduleUpdateOnFiber(current$1, lane, eventTime);
  return lane;
}
```
这个方法主要做的事情是：
* 获取优先级lane，一般创建的react应用都是SyncLane同步优先级，concurrent模式下的react应用不同的更新方式会有不同的优先级，这个后面再说
* 创建更新对象update
* enqueueUpdate这个方法中，如果当前存在更新队列，则将update插入到队列中，以环状链表的方式保存
* scheduleUpdateOnFiber方法是react的核心，创建fiber树，diff，生命周期等逻辑都是在这个方法中实现的

**为什么使用链表而不使用数组，数组需要在内存中开辟出一段连续的空间来保存数据，可以通过那个地址的偏移量来找到对应的数据，而链表则是分散的空间地址，每一项数据保存下一项和上一项数据的引用，对于插入删除的操作来讲，链表等性能更好，不用频繁的更改内存地址，拓展性更强，但是查找效率比数组索引的方式要慢，但是针对于目前的场景来讲，链表更为合适；**
**为什么使用环状链表，环状链表可以更快的找到链表的首尾项，UpdateQueue指向的是最后一项，而最后一项的next指向第一项数据，如果不是环状链表要找到第一项需要便利整个链表，性能较差**

#### scheduleUpdateOnFiber
```javascript
function scheduleUpdateOnFiber(fiber, lane, eventTime) {

  if (lane === SyncLane) {
    if (
    (executionContext & LegacyUnbatchedContext) !== NoContext && 
    (executionContext & (RenderContext | CommitContext)) === NoContext) {
      schedulePendingInteractions(root, lane); 
      performSyncWorkOnRoot(root);
    } else {}
  } else {}
  mostRecentlyUpdatedRoot = root;
}
```
目前我们只看同步模式创建部分的逻辑，当上下文环境为LegacyUnbatchedContext（传统非批量更新上下文，创建时会被设置为此值），并且不处于render和commit阶段，执行两个方法：
* schedulePendingInteractions对更新任务做记录
* performSyncWorkOnRoot中依次调用renderRootSync、commitRoot，进入render和commit阶段

## render阶段
#### renderRootSync
这是render阶段的入口方法，此方法中将RenderContext添加到环境上下文，然后调用workLoopSync方法构建fiber树，处理节点状态，生成节点dom树，然后还原上下文
```javascript
function renderRootSync(root, lanes) {
    var prevExecutionContext = executionContext;
    executionContext |= RenderContext;
    do {
        try {
        workLoopSync();
        break;
        } catch (thrownValue) {
        handleError(root, thrownValue);
        }
    } while (true);
    executionContext = prevExecutionContext;
}
```
#### workLoopSync
通过workInProgress字段保存当前处理的fiber节点，循环调用performUnitOfWork，深度遍历生成fiber树
```javascript
function workLoopSync() {
  while (workInProgress !== null) {
    performUnitOfWork(workInProgress);
  }
}
```
#### performUnitOfWork
深度优先遍历的入口函数，分为两个阶段：
* beginWork：构建fiber树
* completeUnitOfWork：处理节点更新状态，构建dom树
```javascript
function performUnitOfWork(unitOfWork) {
  var current = unitOfWork.alternate;
  setCurrentFiber(unitOfWork);
  var next;
  //ProfileMode性能监控相关
  if ( (unitOfWork.mode & ProfileMode) !== NoMode) {
    startProfilerTimer(unitOfWork);
    next = beginWork$1(current, unitOfWork, subtreeRenderLanes);
    stopProfilerTimerIfRunningAndRecordDelta(unitOfWork, true);
  } else {
    next = beginWork$1(current, unitOfWork, subtreeRenderLanes);
  }

  resetCurrentFiber();
  unitOfWork.memoizedProps = unitOfWork.pendingProps;

  if (next === null) {
    completeUnitOfWork(unitOfWork);
  } else {
    workInProgress = next;
  }

  ReactCurrentOwner$2.current = null;
}
```
#### beginWork
根据组件类型的不同，调用不同方法创建fiber对象，以HostComponent为例，原生dom类型的组件调用的是updateHostComponent方法，在该方法里调用reconcileChildren位workInProgress.child赋值并返回
```javascript
var reconcileChildFibers = ChildReconciler(true);
var mountChildFibers = ChildReconciler(false);

function ChildReconciler(shouldTrackSideEffects) {
    function placeSingleChild(newFiber) {
      if (shouldTrackSideEffects && newFiber.alternate === null) {
        newFiber.flags = Placement;
      }

      return newFiber;
    }
    function reconcileChildFibers(returnFiber, currentFirstChild, newChild, lanes) {
        return placeSingleChild(reconcileSingleElement(returnFiber, currentFirstChild, newChild, lanes));
    }
    return reconcileChildFibers
}

function reconcileChildren(current, workInProgress, nextChildren, renderLanes) {
  if (current === null) {
    workInProgress.child = mountChildFibers(workInProgress, null, nextChildren, renderLanes);
  } else {
    workInProgress.child = reconcileChildFibers(workInProgress, current.child, nextChildren, renderLanes);
  }
}
```
通过current是否存在判断当前处于更新还是创建阶段，如果current存在则为更新阶段，调用reconcileChildFibers方法创建filer节点，此阶段中涉及到新旧节点的diff，我们放到后面在看，创建阶段则调用mountChildFibers方法；
在mountChildFibers方法供根据组件的type、props等不同，实例化构造函数FiberNode创建对应的fiber对象；

**reconcileChildFibers和mountChildFibers实现基本一样，只有一个shouldTrackSideEffects参数的区别，通过控制shouldTrackSideEffects的值来判断是否设置flags为Placement，flags这个参数代表着组件的副作用，值为Placement、Update、Deletion等对应着插入、更新、删除等操作，通过shouldTrackSideEffects来避免重复插入节点的情况**

#### completeUnitOfWork
当遍历到最底部组件时，workInProgress.child === null，则调用completeUnitOfWork向上回归遍历；
在completeUnitOfWork中调用completeWork方法针对组件类型不同，对props进行处理，以HostComponent类型组件为例，props里面保存了classname、child、ref等数据，根据props创建对应dom节点，插入子dom节点向上递归生成dom树，挂在到跟节点上；
```javascript
var currentHostContext = getHostContext();
var instance = createInstance(type, newProps, rootContainerInstance, currentHostContext, workInProgress);
appendAllChildren(instance, workInProgress, false, false);
workInProgress.stateNode = instance;
if (finalizeInitialChildren(instance, type, newProps, rootContainerInstance)) {
    markUpdate(workInProgress);
}
if (workInProgress.ref !== null) {
    markRef$1(workInProgress);
}
```
上面值展示了创建部分的逻辑，createInstance方法创建当前节点的dom，appendAllChildren方法将子节点的dom插入当前dom中，然后将dom挂载到stateNode属性上，markRef$1方法处理ref，向上回归，生成完整的dom树；
接下来处理副作用链表
```javascript
if (returnFiber !== null && // Do not append effects to parents if a sibling failed to complete
    (returnFiber.flags & Incomplete) === NoFlags) {
    if (returnFiber.firstEffect === null) {
        returnFiber.firstEffect = completedWork.firstEffect;
    }
    if (completedWork.lastEffect !== null) {
        if (returnFiber.lastEffect !== null) {
        returnFiber.lastEffect.nextEffect = completedWork.firstEffect;
        }

        returnFiber.lastEffect = completedWork.lastEffect;
    }
    var flags = completedWork.flags; 
    if (flags > PerformedWork) {
        if (returnFiber.lastEffect !== null) {
        returnFiber.lastEffect.nextEffect = completedWork;
        } else {
        returnFiber.firstEffect = completedWork;
        }

        returnFiber.lastEffect = completedWork;
    }
    }
}
```
这部分逻辑主要是将当前fiber节点的副作用挂载到副节点上，向上回归，生成一条副作用链表，firstEffect、lastEffect分别代表链表的收尾，commit阶段会遍历这条链表更新界面；
completeUnitOfWork的最后则是判断当前节点有没有兄弟节点，有的话将兄弟节点赋值给workInProgress，继续调用beginWork，没有的话将父级节点赋值给completedWork，继续循环。
```javascript
function completeUnitOfWork(unitOfWork) {
  var completedWork = unitOfWork;
  do {
      //前面省略
    var siblingFiber = completedWork.sibling;

    if (siblingFiber !== null) {
        workInProgress = siblingFiber;
        return;
    }
    completedWork = returnFiber;
    workInProgress = completedWork;
  } while (completedWork !== null);
}
```

到此render阶段结束
