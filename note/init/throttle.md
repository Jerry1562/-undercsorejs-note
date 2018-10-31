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
4. 执行后在规定时间外触发，流程回到第一步。在规定时间内触发，流程回到第二步。

这套代码的思路是：
1. 第一次触发，执行函数，设定`pervious`值。
2. 第二次触发，在第一次触发设定`pervious`值的情况下，计算剩余时间，设定定时器，设定时间为剩余时间。
3. 在到达预定时间前，无论如何触发都不执行函数。
4. 到达预定时间点，按道理来说，当执行if时，剩余时间`remaining`刚好为0，执行if中的代码，setTimeout中的回调函数会被取消。但是，运行情况并非如此，每次都是回调函数在执行，if中的代码仅仅在第一步中会被执行。为什么？这就涉及到每一条语句的执行时间。我们都知道，程序执行的每一步都是消耗时间的。当计算`remaining`的时候，`remaining`并不为零，但进入下次函数触发之前，`setTimeout`中的计时结束，回调函数被安排在事件处理函数之后执行。执行回调函数会刷新`pervious`的值，下次触发会进入第二步。
5. 停止触发后，此时定时器还在工作，等到预定时间后执行回调函数。

#### 为什么定时器时间要使用`remaining`而不是`wait`？  
答：事实上，无论使用哪个值程序都能正常运行。当我们不断触发`mousemove`事件的时候，remaining的值也在不断变化。在正常的情况下，remaining < wait。如果使用wait，会导致回调函数在remaining <= 0后才执行，然而此时，if中已经通过`clearTimeout`把定时器清除了。它们的执行流程分别是：
+ wait：清除定时器、执行代码、设置定时器
+ remaining：执行定时器回调函数、设置定时器

这样看来使用remaining效率会更高一些。
#### 为什么要使用`if...else if...`而不使用`if...if...`?
答：在第一次触发的过程中，`remaining`的值为负数。如果使用`if...if...`,会在第一次触发的过程中就使用负值`remaining`设置定时器，导致程序错乱。使用`if...else if...`还能带来另一个好处，在一次触发中只能选择设置定时器或者时间戳。程序运行更有条理。
