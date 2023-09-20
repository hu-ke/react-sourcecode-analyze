# 一、什么是scheduler？
从react16开始整个架构分层了三层，scheduler，reconciler，renderer。其中scheduler实现了将一个同步任务变成异步可中断的任务。今天结合实际例子，来看一下如何实现一个简单的scheduler。
# 二、搬盒子📦
假设仓库中有10万个盒子，每搬一个盒子即为一次任务。
```javascript
let storehouse = {
    boxes: 100000
}
// 每次任务搬一个盒子
let execSingleTask = () => {
    for (let i = 0; i < 10000; i++) { // 故意增加单个task的时间
    }
    targetProxy.boxes--
}
```
为了方便记录盒子搬完的时刻，这里我们使用proxy方法。
```javascript
let storehouseProxy = new Proxy(storehouse, {
    set(target, prop, value, receiver) {
        target[prop] = value;
        if (prop === 'boxes' && target['boxes'] === 0) {
            console.log('盒子搬完啦，任务完成~', getCurrentTime())
        }
        return true
    }
})
```
1.不交给scheduler搬
触发搬盒子事件
```javascript
let myTaskList = (new Array(storehouse.boxes)).fill(execSingleTask)

document.getElementById('taskBtn').addEventListener('click', () => {
    console.log('开始执行任务：', getCurrentTime())
    myTaskList.forEach(taskFn => {
        taskFn()
    })
})
```
问题
会阻塞主线程，造成卡顿。比如我们的页面有小球滚动的动画，小球会卡住一段时间。
```css
<style>
.ball{
  position: absolute;
  width: 50px;
  height: 50px;
  border-radius: 50%;
  background-color: red;
  top: 0;
  animation: jump 1s ease-in-out infinite alternate;
}
@keyframes jump{
  from{
    top: 0;
  }
  to{
    top: 400px;
  }
}
</style>
```
2.交给scheduler搬
```javascript
document.getElementById('taskBtnScheduler').addEventListener('click', () => {
    console.log('开始执行任务（scheduler）：', getCurrentTime())
    myTaskList.forEach(taskFn => {
        unstable_scheduleCallback(taskFn)
    })
})
```
解决了小球卡顿问题。
三、scheduler实现
1.流程图

2.代码
