---
title: react源码解析_创建流程_commit阶段
tags: ['react']
---
commit阶段的入口是commitRoot函数
```javascript
function commitRoot(root) {
  var renderPriorityLevel = getCurrentPriorityLevel();
  runWithPriority$1(ImmediatePriority$1, commitRootImpl.bind(null, root, renderPriorityLevel));
  return null;
}
```
这个函数只做了一件事，将commitRootImpl方法放入Scheduler调度器队列，有调度器调用commitRootImpl方法，关于Scheduler的内容我们放到后面单独讲解；
#### commitRootImpl
