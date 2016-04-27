## 目录
* [全局对象](#全局对象)
  * [类： Buffer](#类-buffer)
  * [__dirname](#__dirname)
  * [__filename](#__filename)
  * [clearInterval(t)](#clearintervalt)
  * [clearTimeout(t)](#cleartimeoutt)
  * [console](#console)
  * [exports](#exports)
  * [global](#global)
  * [module](#module)
  * [process](#process)
  * [require()](#require)
    * [require.cache](#requirecache)
    * [require.extensions](#requireextensions)
    * [require.resolve()](#requireresolve)
  * [setInterval(cb, ms)](#setintervalcb-ms)
  * [setTimeout(cb, ms)](#settimeoutcb-ms)

# 全局对象

<!-- type=misc -->

这些对象在所有模块中都可直接使用。这当中的某些对象并不是真正在全局作用域里，而是在模块作用域里，这会被标注出来。

## 类： Buffer

<!-- type=global -->

* {Function}

用来处理二进制数据。参见 [buffer section][]。

## \_\_dirname

<!-- type=var -->

* {String}

当前执行脚本所在的目录名。

示例：在 `/Users/mjr` 目录中执行 `node example.js` 脚本 

```js
console.log(__dirname);
// /Users/mjr
```

`__dirname` 实际上不是全局的，而是各个本地模块有效。

## \_\_filename

<!-- type=var -->

* {String}

当前被执行的代码的文件名。这是该代码文件解析后的绝对路径。对于一个主程序，和命令行中未必会使用相同的文件名。模块中的值是模块文件的路径。

示例：在 `/Users/mjr` 目录中执行 `node example.js` 脚本 

```js
console.log(__filename);
// /Users/mjr/example.js
```

`__filename` 实际上不是全局的，而是各个本地模块有效。

## clearInterval(t)

停止一个通过 [`setInterval()`][] 创建的定时器。定时器的回调将不会再被执行。

<!--type=global-->

timer 函数是全局变量。参见 [timers][] 章节。

## clearTimeout(t)

停止一个通过 [`setTimeout()`][] 创建的定时器。定时器的回调将不会再被执行。

## console

<!-- type=global -->

* {Object}

用来打印标准输出和标准错误。参见 [`console`][] 章节。

## exports

<!-- type=var -->

`module.exports` 的引用，是它的简短书写形式。查看 [module system documentation][] 了解何时使用 `exports` 何时使用 `module.exports`。

`exports` 实际上不是全局的，而是各个本地模块有效。

更多信息参见 [module system documentation][]。

## global

<!-- type=global -->

* {Object} 全局命名空间对象。

在浏览器中，顶层作用域就是全局作用域。这意味着如果你在全局作用域中 `var something` ，将会定义一个全局变量。而 Node.js 中不同，顶层作用域不是全局作用域，在一个 Node.js 模块中定义变量 `var something` ，变量的作用域只是该模块。

## module

<!-- type=var -->

* {Object}

当前模块的一个引用。`module.exports` 可以用来定义模块的输出内容，通过 `require()` 就可以获取到它们。

`module` 实际上不是全局的，而是各个本地模块有效。

更多信息参见 [module system documentation][]。

## process

<!-- type=global -->

* {Object}

进程对象。参见 [`process` object][] 章节。

## require()

<!-- type=var -->

* {Function}

引入模块。参见 [Modules][] 章节。`require` 实际上不是全局的，而是各个本地模块有效。

### require.cache

* {Object}

引入模块时会将模块缓存到该对象。通过删除该对象的键值，下一次调用 `require` 将会重新加载该模块。

### require.extensions

    稳定性： 0 - 弃用

* {Object}

指示 `require` 如何处理特定文件的扩展名。

将 `.sjs` 文件按照 `.js` 文件进行处理：

```js
require.extensions['.sjs'] = require.extensions['.js'];
```

**弃用**  在过去这个列表被用来加载按需编译的非 JavaScript 的模块到 Node.js 中。然而，事实上有很多更好的解决方案，例如通过某个其它 Node.js 程序来加载模块，或者将模块预先编译成 JavaScript。

由于模块系统已被锁定，该功能可能永远不会被移除。然而，它可能会有一些细微的 bug 和复杂性，最好不要去使用。

### require.resolve()

使用内部的 `require()` 机制查找模块位置，但不会加载模块，仅仅返回解析后的文件名。

## setInterval(cb, ms)

每隔 `ms` 毫秒调用一次 `cb` 回调函数。请注意，实际的间隔时间可能会改变，这取决于一些外部因素，例如操作系统的定时器粒度和系统负载。实际间隔时间一定不会比 `ms` 短，只会相等或更长。

间隔时间必须是 1 - 2,147,483,647 之间的值。如果超出了这个范围，会被修正为 1 毫秒。一般来说，一个定时器不应该超过 24.8 天。

返回一个代表该定时器的句柄值。

## setTimeout(cb, ms)

在*至少* `ms` 毫秒后调用 `cb` 回调函数。实际的延迟取决于一些外部因素，例如操作系统的定时器粒度和系统负载。

延迟时间必须是 1 - 2,147,483,647 之间的值。如果超出了这个范围，会被修正为 1 毫秒。一般来说，一个定时器不应该超过 24.8 天。

返回一个代表该定时器的句柄值。

[`console`]: console.markdown
[`process` object]: process.markdown#进程
[`setInterval()`]: #setintervalcb-ms
[`setTimeout()`]: #settimeoutcb-ms
[buffer section]: buffer.markdown
[module system documentation]: modules.markdown
[Modules]: modules.markdown#模块
[timers]: timers.markdown
