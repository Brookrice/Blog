
# requestidlecallback

它提供了一种机制，允许开发者在浏览器空闲时运行低优先级的任务，而不会影响关键任务和动画的性能。

## requestidlecallback 执行阶段

浏览器一针里面做的任务
1. 处理事件的回调： 用户的点击 input 
2. 处理计时器的回调，event loop 
3. 开始渲染 begin帧
4. 执行requestAnimationFrame 动画回调
5. 计算机页面布局计算 合并到主线程
6. 绘制 回流和重绘
7. 如果此时还有空闲时间，执行requestIdleCallback （这个是有条件的！）

## requestidlecallback 基本用法
requestidlecallback 接受一个回调函数 `callback` 并且在回调函数中会注入参数 `deadline`

deadline有两个值:

- deadline.timeRemaining() 返回是否还有空闲时间(毫秒)
- deadline.didTimeout 返回是否因为超时被强制执行(布尔值)
options:
- { timeout: 1000 } 指定回调的最大等待时间（以毫秒为单位）。如果在指定的 timeout 时间内没有空闲时间，回调会强制执行，避免任务无限期推迟

这个案例模拟了在浏览器空闲时，渲染1000条dom元素，非常流畅

```javascript
const nums = 30000; // 定义需要生成的函数数量，即30000个任务
const arr = [];    // 存储任务函数的数组

// 生成1000个函数并将其添加到数组中
function generateDom() {
  for (let i = 0; i < nums; i++) {
    // 每个函数的作用是将一个 <div> 元素插入到页面的 body 中
    arr.push(function() {
      document.body.innerHTML += `<div>${i + 1}</div>`; // 将当前索引 + 1 作为内容
    });
  }
}
generateDom(); // 调用函数生成任务数组

// 用于调度和执行任务的函数
function workLoop(deadline) {
  // 检查当前空闲时间是否大于1毫秒，并且任务数组中还有任务未执行
  if (deadline.timeRemaining() > 1 && arr.length > 0) {
    const fn = arr.shift(); // 从任务数组中取出第一个函数
    fn(); // 执行该函数，即插入对应的 <div> 元素到页面中
  }
  // 再次使用 requestIdleCallback 调度下一个空闲时间执行任务
  requestIdleCallback(workLoop);
}

// 开始调度任务，在浏览器空闲时执行 workLoop
requestIdleCallback(workLoop,{ timeout: 1000});
```