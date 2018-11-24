## requestAnimationFrame 与动画

> 这一节的内容里面的所有链接都只是辅助性内容

在web应用中，实现动画的方式有多种，可以通过CSS3的[transition](https://robot.thoughtbot.com/css-animation-for-beginners)或者[animation](https://robots.thoughtbot.com/css-animation-for-beginners)，也可以使用JS中的[setTimeout](https://developer.mozilla.org/en-US/docs/Web/API/WindowOrWorkerGlobalScope/setTimeout)、[setInterval](https://developer.mozilla.org/en-US/docs/Web/API/WindowOrWorkerGlobalScope/setInterval)来实现，除此之外，还可以通过[requestAnimationFrame](https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame)来实现动画。对于图表来说，需要控制的动画是在任意时间节点上的，所以CSS3的动画就不再适用了，而setTimeout和setInterval这两个方法相比于requestAnimationFrame有性能上的劣势,[better-performance-with-requestAnimationFrame](https://dev.opera.com/articles/better-performance-with-requestanimationframe/)这篇文章介绍了为什么requestAnimationFrame更有优势。

#### 运动

初高中的物理知识已经教会我们运动可以是匀速运动，加速运动（匀加速运动和变加速运动），运动最基本的逻辑就是A点与B点不重合，然后我们从A点如何达到B点的过程。在web逻辑中，我们希望某一个元素从屏幕A点移动到屏幕上B点。总长度L已经明确，接下来需要确定运动的总时间，假设为t秒，运动的过程就需要在t秒内，每一秒前进L/t即可完成。不过以秒作为时间单位会显得非常卡顿，所以我们降低时间单位的数量级，以毫秒作为单位，一毫秒需要完成的路程总共是L / (t * 1000)。下一节将介绍怎样让浏览器来完成运动计算。

#### requestAnimationFrame

requestAnimationFrame这个方法的大体意思就是我们请求浏览器在屏幕刷新的时候去执行某一个逻辑，需要注意的是，这个方法只会在下一帧的时候执行一次，所以如果我们要一直让我们的逻辑在每一次刷新的时候都执行的话，那么就必须要在我们的逻辑内部快结束的时候再次调用requestAnimationFrame方法。

计算机屏幕一秒钟大约会刷新60次，人眼的接受频率就稍微小了一些，所以我们看上去电脑的屏幕切换得非常流畅。回到上一节的问题，我们如何让浏览器来完成运动计算呢？首先需要说明一下requestAnimationFrame这个方法是没有状态的，也就是它并不会帮我们记录时间，所以开始的时间需要我们自己去记录。

首先定义几个变量，L代表运动总路程，T代表运动总时间（ms毫秒），速度v代表运动速度（v=L/T），spend代表已经运动的时间，moved代表在spend时间之内走过的路程，startTime代表开始运动的时间。

初始状态：

```
T = T
L = L
v = L / T
spend = 0
moved = 0
// Date.now是一个方法，用来获取当前的时间，返回一个毫秒数
startTime = Date.now()
```

每一次requestAnimationFrame执行的时候。

```
// 已经运动的时间等于当前时间 - startTime
spend = Date.now() - startTime
moved = spend * v
```

所以requestAnimationFrame执行完之后，我们都可以获取到新的moved的值，这就是运动的结果，只要我们不停下来，这个运动将会一直持续下去。

代码实现：

```javascript 1.6

const L = 1000; // 假设总长度为1000
const T = 3000; // ms 假设运动总时间为3秒
const v = L / T;
const startTime = Date.now();
let spend = 0;
let moved = 0;

let timer;

// 定义一个每一帧的回调函数
const eachFrame = () => {
     spend = Date.now() - startTime;
        moved = spend * v;
        
        /*
            接下来你就可以将这个moved写入具体的运动元素中了。
         */
        
        console.log(moved);
        
        // 这里再次调用requestAnimationFrame
        timer = window.requestAnimationFrame(eachFrame);
};

timer = window.requestAnimationFrame(eachFrame);
```

#### 停下来

让运动停下来的方式是判断moved是否大于等于L，如果是的话，直接不执行后面的结果了，另外一种是判断moved是否大于等于L，如果是的话，我们就不需要去调用requestAnimationFrame。

代码如下：

```javascript 1.6
const L = 1000; // 假设总长度为1000
const T = 3000; // ms 假设运动总时间为3秒
const v = L / T;
const startTime = Date.now();
let spend = 0;
let moved = 0;

let timer;

// 定义一个每一帧的回调函数
const eachFrame = () => {
     spend = Date.now() - startTime;
        moved = spend * v;
        
        /*
            接下来你就可以将这个moved写入具体的运动元素中了。
         */
        
        console.log(moved);
        
        if (moved <= L) {
            // 这里再次调用requestAnimationFrame
            timer = window.requestAnimationFrame(eachFrame); 
        }
     
};

timer = window.requestAnimationFrame(eachFrame);
```

那么有一种情况是我们调用了requestAnimationFrame，但是在它没有执行之前我们就反悔了，不想执行了，那应该怎么停下来呢？停下来的方法也很简单，再回头看一下上一节的代码，我们将requestAnimationFrame的返回值赋值给一个timer变量，这是上一节没有提到的内容。requestAnimationFrame方法返回一个数值，这个代表着当前我们调用的requestAnimationFrame的一个id，这些id怎么来的我们不需要关心，如果我们要停掉requestAnimationFrame，只需要告诉管理requestAnimationFrame的家伙我们需要停掉的id就好了。

```javascript 1.6
// ...上面的代码
let timer = window.requestAnimationFrame(eachFrame);
// 取消requestAnimationFrame
window.cancelAnimationFrame(timer);
```

下一节[变速运动](./variableMotion.md)。

