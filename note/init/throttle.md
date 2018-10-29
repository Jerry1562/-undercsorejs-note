## `underscore`中的节流
详情可见冴羽大大的博客[《JavaScript专题之跟着 underscore 学节流》](https://github.com/mqyqingfeng/Blog/issues/26)，博客由浅入深地讲解了五个版本的代码，在这里主要记录一下自己理解第三版代码的思路。
```javascript
// 第三版
function throttle(func, wait) {
    var timeout, context, args, result;
    var previous = 0;

    var later = function() {
        previous = +new Date();
        timeout = null;
        func.apply(context, args)
    };

    var throttled = function() {
        var now = +new Date();
        //下次触发 func 剩余的时间
        var remaining = wait - (now - previous);
        context = this;
        args = arguments;
         // 如果没有剩余的时间了或者你改了系统时间
        if (remaining <= 0 || remaining > wait) {
            if (timeout) {
                clearTimeout(timeout);
                timeout = null;
            }
            previous = now;
            func.apply(context, args);
        } else if (!timeout) {
            timeout = setTimeout(later, remaining);
        }
    };
    return throttled;
}
```
我们要实现的效果是：
1. 首次触发，函数执行。
2. 不断触发的情况下，节流效果显现，每隔一段时间只执行一次。
3. 在停止触发一段时间后，还能执行一次。

这套代码的思路是：
1. 第一次触发，执行函数，设定`pervious`值。
2. 第二次触发，设定定时器，设定时间为剩余时间。
3. 在到达预定时间前，无论如何触发都不执行函数。
4. 到达预定时间点，有两种情况，当remaining等于0时，

#### 为什么定时器时间要使用`remaining`而不是`wait`？  
答：事实上，无论使用哪个值程序都能正常运行。当我们不断触发`mousemove`事件的时候，remaining的值也在不断变化。在正常的情况下，remaining < wait。如果使用wait，会导致回调函数在remaining <= 0后才执行，然而此时，if中已经通过`clearTimeout`把定时器清除了。它们的执行流程分别是：
+ wait：清除定时器、执行代码、设置定时器
+ remaining：执行定时器回调函数、设置定时器

这样看来使用remaining效率会更高一些。
