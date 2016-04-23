## 目录
* [定时器](#定时器)
  * [clearImmediate(immediateObject)](#clearimmediateimmediateobject)
  * [clearInterval(intervalObject)](#clearintervalintervalobject)
  * [clearTimeout(timeoutObject)](#cleartimeouttimeoutobject)
  * [ref()](#ref)
  * [setImmediate(callback[, arg][, ...])](#setimmediatecallback-arg-)
  * [setInterval(callback, delay[, arg][, ...])](#setintervalcallback-delay-arg-)
  * [setTimeout(callback, delay[, arg][, ...])](#settimeoutcallback-delay-arg-)
  * [unref()](#unref)

# 定时器

    稳定性： 3 - 已锁定

所有的定时器函数都是全局的，即不需要 `require()` 就可以使用它们。

## clearImmediate(immediateObject)

停止一个 `immediateObject` 的触发，`immediateObject` 是由 [`setImmediate`][] 创建的回调函数。

## clearInterval(intervalObject)

停止一个 `intervalObject` 的触发，`intervalObject` 是由 [`setInterval`][] 创建的回调函数。

## clearTimeout(timeoutObject)

停止一个 `timeoutObject` 的触发，`timeoutObject` 是由 [`setTimeout`][] 创建的回调函数。

## ref()

如果一个定时器之前已被 `unref()` 过了，那么调用 `ref()` 可以显式得请求定时器保持程序运行。如果定时器已经被 `ref` 了，再次调用 `ref` 不会产生其他影响。

返回定时器。

## setImmediate(callback[, arg][, ...])

计划一个 "立即" 执行的 `callback`，它会在所有的 I/O 时间的回调触发之后以及由 [`setTimeout`][] 和 [`setInterval`][] 设置的回调触发之前执行。 返回一个 `immediateObject`，[`clearImmediate`][] 可能会用到它。额外的可选参数会被传入回调函数中。

回调已按照它们创建时的顺序加入队列。整个回调队列在每个事件循环迭代中被处理。如果在一个正被执行中的回调上添加一个 immediate，那么它将直到下一个事件循环迭代才会被触发。

## setInterval(callback, delay[, arg][, ...])

计划一个每隔 `delay` 毫秒便执行一次的 `callback` 。返回一个 `intervalObject`，[`clearInterval`][] 可能会用到它。额外的可选参数会被传入回调函数中。

为了与浏览器端行为保持一致，当 `delay` 大于 2147483647 毫秒（约 25 天）或者小于 1，Node.js 将使用 1 作为 `delay` 值。

## setTimeout(callback, delay[, arg][, ...])

计划一个在 `delay` 毫秒后执行一次的 `callback` 。返回一个 `timeoutObject`，[`clearTimeout`][] 可能会用到它。额外的可选参数会被传入回调函数中。

回调可能不会在精确的  `delay` 毫秒后被调用。Node.js 不能保证回调被触发的精确时间和顺序，回调会在尽可能接近指定时间上被调用。

为了与浏览器端行为保持一致，当 `delay` 大于 2147483647 毫秒（约 25 天）或者小于 1，将被立即执行，就好像 `delay` 被设为了 1.

## unref()

[`setTimeout`][] 和 [`setInterval`][] 的返回值也拥有方法 `timer.unref()`，该方法允许创建一个活动的定时器，但如果它是时间循环中的仅剩的那一项则不会运行。如果定时器已经被 `unref` 了，再次调用 `unref` 不会产生其他影响。

对于 [`setTimeout`][]， `unref` 会创建一个单独的定时器，它将唤醒时间循环。创建太多这种定时器可能会对时间循环的性能有不利影响 -- 谨慎使用。

返回定时器。

[`clearImmediate`]: timers.markdown#clearimmediateimmediateobject
[`clearInterval`]: timers.markdown#clearintervalintervalobject
[`clearTimeout`]: timers.markdown#cleartimeouttimeoutobject
[`setImmediate`]: timers.markdown#setimmediatecallback-arg-
[`setInterval`]: timers.markdown#setintervalcallback-delay-arg-
[`setTimeout`]: timers.markdown#settimeoutcallback-delay-arg-
