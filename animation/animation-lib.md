## 构建一个自己的动画库

> 在这一节中，更多地是代码的表述，文字性较少

#### 目标

构建一个动画库，该动画库需要提供的以下接口：

```javascript 1.6
start()   // starting animation
stop()    // stop animation
duration(Number)    // set animation duration time
ease(Function)    // set animation ease function
complete(Function)  // invoke callback when animation end.

```

#### 马上开始

```javascript 1.6

const linearEase = (t) => {return t;};
const interpolator = (startValue, endValue) => {
    endValue -= startValue;
    return (t) => {
        return t * endValue;
    };
};

class Animate {
    constructor(start, end, duration, callback, ease) {
        this.startTime = null;
        this.durationTime = duration || 1000;
        this.easeFunc = ease || linearEase;
        this.callback = callback;
        this.timer = null;
        this.interpolator = interpolator(start, end);
    }
    
    start () {
        this.startTime = Date.now();
        this._eachFrame();
    }
    
    stop () {
        window.cancelAnimationFrame(this.timer);
    }
    
    duration (duration) {
        this.durationTime = duration;
    }
    
    ease (ease) {
        this.easeFunc = ease;
    }
    
    complete (callback) {
        this.completeCallback = callback;
    }
    
    _eachFrame () {
        // 已经消耗的时间
        let elapsedTime = Date.now() - this.startTime;
        let t = this.easeFunc(elapsedTime / this.durationTime);
        let value = this.interpolator(t);
        
        this.callback(value);
                    
        if (elapsedTime <= this.durationTime) {
            this.timer = window.requestAnimationFrame(() => {
                    this._eachFrame();
            });
        } else {
            this.completeCallback && this.completeCallback();
        }
    }
}
```

调用方式示例：

```javascript 1.6
const eachFrame = v => {
    console.log(v);
}

new Animate(0, 1000, 3000, eachFrame).start();
```