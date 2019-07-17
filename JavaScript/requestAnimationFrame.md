### requestAnimationFrame

**背景**

​		计时器一直是 javascript 动画的核心技术。而编写动画循环的关键是要知道延迟时间多长合适。一方面，循环间隔必须足够短，这样才能让不同的动画效果显得平滑流畅；另一方面，循环间隔还要足够长，这样才能确保浏览器有能力渲染产生的变化。

　　大多数电脑显示器的刷新频率是 60Hz，大概相当于每秒钟重绘60次。大多数浏览器都会对重绘操作加以限制，不超过显示器的重绘频率，因为即使超过那个频率用户体验也不会有提升。因此，最平滑动画的最佳循环间隔是 1000ms/60，约等于 16.66ms，低于这个时间还可能会丢帧。

　　而 `setTimeout` 和 `setInterval` 的问题是，它们都不精确。事件循环机制决定了时间间隔参数实际上只是指定了把动画代码添加到浏览器 UI 线程队列中以等待执行的时间。如果队列前面已经加入了其他任务，那动画代码就要等前面的任务完成后再执行。

　　`requestAnimationFrame` 采用系统时间间隔，保持最佳绘制效率，不会因为间隔时间过短，造成过度绘制，增加开销；也不会因为间隔时间太长，使用动画卡顿不流畅，让各种网页动画效果能够有一个统一的刷新机制，从而节省系统资源，提高系统性能，改善视觉效果。

**优点**

- `requestAnimationFrame` 会把每一帧中的所有 DOM 操作集中起来，在一次重绘或回流中就完成，并且重绘或回流的时间间隔紧紧跟随浏览器的刷新频率。
- 在隐藏或不可见的元素中，`requestAnimationFrame` 将不会进行重绘或回流，这当然就意味着更少的 CPU、GPU 和内存使用量。
- 当页面处于未激活的状态下，该页面的屏幕刷新任务会被系统暂停，由于`requestAnimationFrame`保持和屏幕刷新同步执行，所以也会被暂停。当页面被激活时，动画从上次停留的地方继续执行，节约 CPU 开销。

**使用**

​		`requestAnimationFrame `的用法与`settimeout`很相似，只是不需要设置时间间隔而已。`requestAnimationFrame`使用一个回调函数作为参数，这个回调函数会在浏览器重绘之前调用。它返回一个整数，表示定时器的编号，这个值可以传递给 **cancelAnimationFrame** 用于取消这个函数的执行。

```js
 rAFId = requestAnimationFrame(callback)
 // 取消
 cancelAnimationFrame(rAFId)
```

**兼容**

```javascript
// 简单兼容一下
if (!window.requestAnimationFrame) {
    window.requestAnimationFrame = function(fn) {
        setTimeout(fn, 17);
    };    
}
if (!window.cancelAnimationFrame) {
    window.cancelAnimationFrame = function(id) {
        clearTimeout(id);
    };
}
```

**应用**

- 简单进度条效果

```JavaScript
<div id="myDiv" style="background-color: lightblue;width: 0;height: 20px;line-height: 20px;">0%</div>
<button id="btn">run</button>
<script>
var timer;
btn.onclick = function(){
    myDiv.style.width = '0';
    cancelAnimationFrame(timer);
    timer = requestAnimationFrame(function fn(){
        if(parseInt(myDiv.style.width) < 500){
            myDiv.style.width = parseInt(myDiv.style.width) + 5 + 'px';
            myDiv.innerHTML =     parseInt(myDiv.style.width)/5 + '%';
            timer = requestAnimationFrame(fn);
        }else{
            cancelAnimationFrame(timer);
        }    
    });
}
</script>
```

- 大数据渲染

​        有个场景，将后台返回的十万条记录插入到表格中，如果一次性在循环中生成 DOM 元素，会导致页面卡顿5s 左右。这时候我们就可以用 `requestAnimationFrame` 进行分步渲染，确定最好的时间间隔，使得页面加载过程中很流畅。

```JavaScript
var total = 100000;
var size = 100;
var count = total / size;
var done = 0;
var ul = document.getElementById('list');

function addItems() {
    var li = null;
    var fg = document.createDocumentFragment();
    for (var i = 0; i < size; i++) {
        li = document.createElement('li');
        li.innerText = 'item ' + (done * size + i);
        fg.appendChild(li);
    }
    ul.appendChild(fg);
    done++;
    if (done < count) {
        requestAnimationFrame(addItems);
    }
};

requestAnimationFrame(addItems);

```

- 重复绘制问题

多次调用带有同一回调函数的 `requestAnimationFrame`，会导致回调在同一帧中执行多次，也就是说它并不管理回调函数。也正因为它不管理回调函数，在滚动、这类高触发频率的事件回调里，可能会造成多余的计算和绘制。

```javascript
window.addEventListener('scroll', e => {
    window.requestAnimationFrame(stamp => {
        animation(stamp)
    })
})
```

这时候可以使用节流函数。但节流函数是通过时间管理队列的，而 `requestAnimationFrame `的触发时间是不固定的。完美的解决方案是通过 `requestAnimationFrame `来管理队列，其思路就是保证 `requestAnimationFrame `的队列里，同样的回调函数只有一个。

```javascript
const onScroll = e => {
    if (framing) return
    let framing = true
    window.requestAnimationFrame(timestamp => {
        framing = false
        animation(timestamp)
    })
}
window.addEventListener('scroll', onScroll)
```









