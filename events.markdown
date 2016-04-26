## 目录
* [事件](#事件)
  * [传递参数和 `this` 给监听器](#传递参数和-this-给监听器)
  * [异步 vs 同步](#异步-vs-同步)
  * [仅触发一次的处理事件](#仅触发一次的处理事件)
  * [错误事件](#错误事件)
  * [类： EventEmitter](#类-eventemitter)
    * [事件： 'newListener'](#事件-newlistener)
    * [事件： 'removeListener'](#事件-removelistener)
    * [EventEmitter.listenerCount(emitter, eventName)](#eventemitterlistenercountemitter-eventname)
    * [EventEmitter.defaultMaxListeners](#eventemitterdefaultmaxlisteners)
    * [emitter.addListener(eventName, listener)](#emitteraddlistenereventname-listener)
    * [emitter.emit(eventName[, arg1][, arg2][, ...])](#emitteremiteventname-arg1-arg2-)
    * [emitter.getMaxListeners()](#emittergetmaxlisteners)
    * [emitter.listenerCount(eventName)](#emitterlistenercounteventname)
    * [emitter.listeners(eventName)](#emitterlistenerseventname)
    * [emitter.on(eventName, listener)](#emitteroneventname-listener)
    * [emitter.once(eventName, listener)](#emitteronceeventname-listener)
    * [emitter.removeAllListeners([eventName])](#emitterremovealllistenerseventname)
    * [emitter.removeListener(eventName, listener)](#emitterremovelistenereventname-listener)
    * [emitter.setMaxListeners(n)](#emittersetmaxlistenersn)

# 事件

    稳定性： 2 - 稳定

<!--type=module-->

Node.js 的很多核心 API 都是围绕一个惯用的异步事件驱动架构构建的，某些对象类型（称为"emitters 分发器"）会定期分发命名事件，这将导致函数对象（"listeners 监听器"）被调用。

例如：一个 [`net.Server`][] 对象每当接收到一个连接就会分发（emit）一个事件；一个 [`fs.ReadStream`][] 对象会在文件被打开时分发一个事件；一个 [stream][] 对象会在数据读取可用时分发一个事件。

所有能够分发事件的对象都是 `EventEmitter` 类的实例。这些对象会暴露一个 `eventEmitter.on()` 方法允许一个或多个函数附加到由该对象分发的命名事件上。一般来说，事件名称是驼峰字符串，但其实任何合法的 JavaScript 属性键都可以被使用。

当 `EventEmitter` 对象分发一个事件，所有的附加到该事件的函数都会被_同步_调用。调用的监听器返回的任何值将被_忽略_并丢弃。

接下来的例子展示了一个简单的带监听器的 `EventEmitter` 实例。`eventEmitter.on()` 方法用来注册监听器，`eventEmitter.emit()` 方法用来触发该监听器事件。

```js
const EventEmitter = require('events');
const util = require('util');

function MyEmitter() {
  EventEmitter.call(this);
}
util.inherits(MyEmitter, EventEmitter);

const myEmitter = new MyEmitter();
myEmitter.on('event', () => {
  console.log('触发了一个事件！');
});
myEmitter.emit('event');
```

任何对象都可以通过继承成为一个 `EventEmitter` 对象。上面的例子通过 `util.inherits()` 方法使用了传统 Node.js 风格的原型继承。当然也可以使用 ES6 的类继承：

```js
const EventEmitter = require('events');

class MyEmitter extends EventEmitter {}

const myEmitter = new MyEmitter();
myEmitter.on('event', () => {
  console.log('触发了一个事件！');
});
myEmitter.emit('event');
```

## 传递参数和 `this` 给监听器

`eventEmitter.emit()` 方法允许传递任意参数到监听器函数。重要的是要记住，当一个普通的监听器函数通过 `EventEmitter` 被调用，标准 `this` 关键字会被特意设置为指向监听器所附加的 `EventEmitter` 的引用。

```js
const myEmitter = new MyEmitter();
myEmitter.on('event', function(a, b) {
  console.log(a, b, this);
    // Prints:
    //   a b MyEmitter {
    //     domain: null,
    //     _events: { 事件： [Function] },
    //     _eventsCount: 1,
    //     _maxListeners: undefined }
});
myEmitter.emit('event', 'a', 'b');
```

也可以使用 ES6 的箭头函数作为监听器，然而，如果这样做的话 `this` 关键字将不再指向 `EventEmitter` 实例的引用：

```js
const myEmitter = new MyEmitter();
myEmitter.on('event', (a, b) => {
  console.log(a, b, this);
    // Prints: a b {}
});
myEmitter.emit('event', 'a', 'b');
```

## 异步 vs 同步

`EventListener` 会按照监听器当初注册时的顺序同步调用它们。这对于确保适当的排序以及避免竞态条件或逻辑错误非常重要。当在恰当的时候，也可以使用 `setImmediate()` 或 `process.nextTick()` 方法将监听器函数切换到异步操作模式。

```js
const myEmitter = new MyEmitter();
myEmitter.on('event', (a, b) => {
  setImmediate(() => {
    console.log('异步触发');
  });
});
myEmitter.emit('event', 'a', 'b');
```

## 仅触发一次的处理事件

当监听器是通过 `eventEmitter.on()` 方法注册的，那么每一次分发该命名事件时该监听器都会被调用。

```js
const myEmitter = new MyEmitter();
var m = 0;
myEmitter.on('event', () => {
  console.log(++m);
});
myEmitter.emit('event');
  // 打印：1
myEmitter.emit('event');
  // 打印：2
```

使用 `eventEmitter.once()` 方法注册一个监听器可以让该监听器被调用之后立即解除注册。

```js
const myEmitter = new MyEmitter();
var m = 0;
myEmitter.once('event', () => {
  console.log(++m);
});
myEmitter.emit('event');
  // 打印：1
myEmitter.emit('event');
  // 被忽略
```

## 错误事件

当在 `EventEmitter` 实例中发生了一个错误，典型的行为是分发一个 `'error'` 事件。这在 Node.js 中被视作一种特殊情况进行处理。

如果一个 `EventEmitter` _没有_ 在 `'error'` 事件上注册任何监听器，但又分发了一个 `'error'` 事件，则会抛出异常，打印错误堆栈信息，同时退出 Node.js 进程。

```js
const myEmitter = new MyEmitter();
myEmitter.emit('error', new Error('whoops!'));
  // 抛出异常并退出 Node.js
```

为了防范 Node.js 进程崩溃，开发者可以在 `process.on('uncaughtException')` 事件上注册一个监听器，或者使用 [`domain`][] 模块（_注意 `domain` 模块已被弃用_）。

```js
const myEmitter = new MyEmitter();

process.on('uncaughtException', (err) => {
  console.log('哎呀！有一个错误！');
});

myEmitter.emit('error', new Error('whoops!'));
  // 打印：哎呀！有一个错误！
```

作为一项最佳实践，开发者应该总是为 `'error'` 事件注册监听器：

```js
const myEmitter = new MyEmitter();
myEmitter.on('error', (err) => {
  console.log('哎呀！有一个错误！');
});
myEmitter.emit('error', new Error('whoops!'));
  // 打印：哎呀！有一个错误！
```

## 类： EventEmitter

`EventEmitter` 类是由 `events` 模块定义并公开：

```js
const EventEmitter = require('events');
```

当添加新的监听器时，所有的 EventEmitters 都会分发 `'newListener'` 事件，当监听器会移除时，会分发 `'removeListener'` 事件。

### 事件： 'newListener'

* `eventName` {String|Symbol} 监听的事件的名称
* `listener` {Function} 事件处理函数

`EventEmitter` 实例会在将一个监听器添加到它内部的监听器数组*之前*分发 `'newListener'` 事件。

`'newListener'` 事件上注册的监听器会传递事件名称和被添加的监听器的引用作为参数。

`'newListener'` 事件会在添加其它监听器之前就被触发，这带来一个微妙但很重要的副作用：在 `'newListener'` 回调*内部*注册的具有相同 `name` 的监听器会添加到同名监听器之前。
```js
const myEmitter = new MyEmitter();
// 只执行一次
myEmitter.once('newListener', (event, listener) => {
  if (event === 'event') {
    // 在前面插入一个新的监听器
    myEmitter.on('event', () => {
      console.log('B');
    });
  }
});
myEmitter.on('event', () => {
  console.log('A');
});
myEmitter.emit('event');
  // 打印：
  //   B
  //   A
```

### 事件： 'removeListener'

* `eventName` {String|Symbol} 事件名称
* `listener` {Function} 事件处理函数

`'removeListener'` 事件会在一个监听器被移除*之后*分发。

### EventEmitter.listenerCount(emitter, eventName)

    稳定性： 0 - 弃用，使用 [`emitter.listenerCount()`][] 代替。

一个类方法，会返回给定 `emitter` 上注册的 `eventName` 监听器的数量。

```js
const myEmitter = new MyEmitter();
myEmitter.on('event', () => {});
myEmitter.on('event', () => {});
console.log(EventEmitter.listenerCount(myEmitter, 'event'));
  // 打印： 2
```

### EventEmitter.defaultMaxListeners

默认情况下任何单个事件最多可以注册 `10` 个监听器。该限制可以在单个的 `EventEmitter` 实例上使用 [`emitter.setMaxListeners(n)`][] 方法进行修改。要修改*所有* `EventEmitter` 实例的默认值，可以使用 `EventEmitter.defaultMaxListeners` 属性。

请注意修改 `EventEmitter.defaultMaxListeners` 会影响到*所有*的 `EventEmitter` 实例，包括哪些在这之前创建的实例。然而，[`emitter.setMaxListeners(n)`][] 优先级依然高于 `EventEmitter.defaultMaxListeners`。

注意这不是一个硬性限制，`EventEmitter` 实例允许添加更多的监听器，但是会输出一个追踪警告的标准错误，表示检测到 `possible EventEmitter memory leak`（可能的 EventEmitter 内存泄漏）。对任何一个 `EventEmitter`，`emitter.getMaxListeners()` 和 `emitter.setMaxListeners()` 方法可以用来临时调整上限值以忽略该警告。

```js
emitter.setMaxListeners(emitter.getMaxListeners() + 1);
emitter.once('event', () => {
  // 做些事情
  emitter.setMaxListeners(Math.max(emitter.getMaxListeners() - 1, 0));
});
```

### emitter.addListener(eventName, listener)

[`emitter.on(eventName, listener)`][] 的别名。

### emitter.emit(eventName[, arg1][, arg2][, ...])

同步调用名为 `eventName` 的事件上注册的每一个监听器，会按照监听器注册时的顺序执行，并给每个监听器传入提供的参数。

如果该事件有监听器则返回 `true`，否则返回 `false`。

### emitter.getMaxListeners()

返回当前由 [`emitter.setMaxListeners(n)`][] 或 [`EventEmitter.defaultMaxListeners`][] 设置的最大的监听器上限值。

### emitter.listenerCount(eventName)

* `eventName` {Value} 被监听事件的名称

返回正在监听 `eventName` 事件的监听器的数量。

### emitter.listeners(eventName)

返回 `eventName` 事件上监听器数组的一个副本。

```js
server.on('connection', (stream) => {
  console.log('产生了一个连接！');
});
console.log(util.inspect(server.listeners('connection')));
  // 打印： [ [Function] ]
```

### emitter.on(eventName, listener)

给 `eventName` 事件添加 `listener` 监听器函数到监听器数组的末尾。不会检查 `listener` 是否已被添加过。重复传入相同的 `eventName` 和 `listener` 也将导致 `listener` 被重复地添加和调用。

```js
server.on('connection', (stream) => {
  console.log('产生了一个连接！');
});
```

返回一个 `EventEmitter` 的引用以便链式调用。

### emitter.once(eventName, listener)

给 `eventName` 事件添加一个**一次性**的 `listener` 监听器函数。该监听器仅会在下一次的 `eventName` 事件被触发时调用，然后就会被移除。

```js
server.once('connection', (stream) => {
  console.log('啊，我们有了第一个用户了！');
});
```

返回一个 `EventEmitter` 的引用以便链式调用。

### emitter.removeAllListeners([eventName])

移除所有的监听器，或者指定 `eventName` 事件上的所有监听器。

请注意移除代码中别的地方的监听器是一个不好的实践，尤其当 `EventEmitter` 实例是由其他组件或模块创建的（比如 sockets 套接字或 file streams 文件流）。

返回一个 `EventEmitter` 的引用以便链式调用。

### emitter.removeListener(eventName, listener)

从 `eventName` 事件的监听器数组中移除指定的 `listener` 监听器。

```js
var callback = (stream) => {
  console.log('产生了一个连接！');
};
server.on('connection', callback);
// ...
server.removeListener('connection', callback);
```

`removeListener` 将会从监听器数组中移除至多一个监听器的实例。如果任何一个监听器被多次添加到了特定的 `eventName` 事件的监听器数组中，那么同样需要多次调用 `removeListener` 来逐个移除它们。

请注意一旦分发了一个事件，所有附加到该事件上的监听器会同时依次被调用。这意味着在事件分发*之后*、最后一个监听器完成执行*之前*调用任何 `removeListener()` 或 `removeAllListeners()` 将不会移除进行中的监听器。之后的事件将遵循预期行为。

```js
const myEmitter = new MyEmitter();

var callbackA = () => {
  console.log('A');
  myEmitter.removeListener('event', callbackB);
};

var callbackB = () => {
  console.log('B');
};

myEmitter.on('event', callbackA);

myEmitter.on('event', callbackB);

// callbackA 移除了 callbackB，但 callbackB 依然会被调用.
// 分发事件依次调用内部监听器数组 [callbackA, callbackB]
myEmitter.emit('event');
  // 打印：
  //   A
  //   B

// callbackB 已被移除。
// 内部监听器数组 [callbackA]
myEmitter.emit('event');
  // 打印：
  //   A

```

因为监听器是被一个内部数组管理的，调用该方法将会改变移除监听器*之后*的注册的监听器的位置索引。这不会影响监听器调用时的顺序，但 `emitter.listeners()` 方法返回的任何监听器数组都需要被重建。

返回一个 `EventEmitter` 的引用以便链式调用。

### emitter.setMaxListeners(n)

默认情况下如果在一个特定的事件上添加超过 `10` 个监听器 EventEmitters 将会打印一个警告信息。该默认设置有助于发现内存泄漏。显然并非所有的事件都需要限制 10 个监听器。`emitter.setMaxListeners()` 方法允许在指定 `EventEmitter` 实例上修改该限制。值可以被设为 `Infinity`（或 `0`）表示无限制。

返回一个 `EventEmitter` 的引用以便链式调用。

[`net.Server`]: net.markdown#类-netserver
[`fs.ReadStream`]: fs.markdown#类-fsreadstream
[`emitter.on(eventName, listener)`]: #emitteroneventname-listener
[`emitter.setMaxListeners(n)`]: #emittersetmaxlistenersn
[`EventEmitter.defaultMaxListeners`]: #事件-eventemitterdefaultmaxlisteners
[`emitter.listenerCount()`]: #emitterlistenercounteventname
[`domain`]: domain.markdown
[stream]: stream.markdown
