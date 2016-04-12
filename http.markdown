## 目录
* [HTTP]()
  * [类：http.Agent](#类-httpagent)
    * [new Agent([options])](#new-agentoptions)
    * [agent.destroy()](#agentdestroy)
    * [agent.freeSockets](#agentfreesockets)
    * [agent.getName(options)](#agentgetnameoptions)
    * [agent.maxFreeSockets](#agentmaxfreesockets)
    * [agent.maxSockets](#agentmaxsockets)
    * [agent.requests](#agentrequests)
    * [agent.sockets](#agentsockets)
  * [类：http.ClientRequest](#类-httpclientrequest)
    * [事件：'abort'](#事件-abort)
    * [事件：'connect'](#事件-connect)
    * [事件：'continue'](#事件-continue)
    * [事件：'response'](#事件-response)
    * [事件：'socket'](#事件-socket)
    * [事件：'upgrade'](#事件-upgrade)
    * [request.abort()](#requestabort)
    * [request.end([data][, encoding][, callback])](#requestenddata-encoding-callback)
    * [request.flushHeaders()](#requestflushheaders)
    * [request.setNoDelay([noDelay])](#requestsetnodelaynodelay)
    * [request.setSocketKeepAlive([enable][, initialDelay])](#requestsetsocketkeepaliveenable-initialdelay)
    * [request.setTimeout(timeout[, callback])](#requestsettimeouttimeout-callback)
    * [request.write(chunk[, encoding][, callback])](#requestwritechunk-encoding-callback)
  * [类：http.Server](#类-httpserver)
    * [事件：'checkContinue'](#事件-checkcontinue)
    * [事件：'clientError'](#事件-clienterror)
    * [事件：'close'](#事件-close)
    * [事件：'connect'](#事件-connect)
    * [事件：'connection'](#事件-connection)
    * [事件：'request'](#事件-request)
    * [事件：'upgrade'](#事件-upgrade)
    * [server.close([callback])](#serverclosecallback)
    * [server.listen(handle[, callback])](#serverlistenhandle-callback)
    * [server.listen(path[, callback])](#serverlistenpath-callback)
    * [server.listen(port[, hostname][, backlog][, callback])](#serverlistenport-hostname-backlog-callback)
    * [server.maxHeadersCount](#servermaxheaderscount)
    * [server.setTimeout(msecs, callback)](#serversettimeoutmsecs-callback)
    * [server.timeout](#servertimeout)
  * [类：http.ServerResponse](#类-httpserverresponse)
    * [事件：'close'](#事件-close)
    * [事件：'finish'](#事件-finish)
    * [response.addTrailers(headers)](#responseaddtrailersheaders)
    * [response.end([data][, encoding][, callback])](#responseenddata-encoding-callback)
    * [response.finished](#responsefinished)
    * [response.getHeader(name)](#responsegetheadername)
    * [response.headersSent](#responseheaderssent)
    * [response.removeHeader(name)](#responseremoveheadername)
    * [response.sendDate](#responsesenddate)
    * [response.setHeader(name, value)](#responsesetheadername-value)
    * [response.setTimeout(msecs, callback)](#responsesettimeoutmsecs-callback)
    * [response.statusCode](#responsestatuscode)
    * [response.statusMessage](#responsestatusmessage)
    * [response.write(chunk[, encoding][, callback])](#responsewritechunk-encoding-callback)
    * [response.writeContinue()](#responsewritecontinue)
    * [response.writeHead(statusCode[, statusMessage][, headers])](#responsewriteheadstatuscode-statusmessage-headers)
  * [类：http.IncomingMessage](#类-httpincomingmessage)
    * [事件：'close'](#事件-close)
    * [message.headers](#messageheaders)
    * [message.httpVersion](#messagehttpversion)
    * [message.method](#messagemethod)
    * [message.rawHeaders](#messagerawheaders)
    * [message.rawTrailers](#messagerawtrailers)
    * [message.setTimeout(msecs, callback)](#messagesettimeoutmsecs-callback)
    * [message.statusCode](#messagestatuscode)
    * [message.statusMessage](#messagestatusmessage)
    * [message.socket](#messagesocket)
    * [message.trailers](#messagetrailers)
    * [message.url](#messageurl)
    * [http.METHODS](#httpmethods)
    * [http.STATUS_CODES](#httpstatus_codes)
    * [http.createClient([port][, host])](#httpcreateclientport-host)
    * [http.createServer([requestListener])](#httpcreateserverrequestlistener)
    * [http.get(options[, callback])](#httpgetoptions-callback)
    * [http.globalAgent](#httpglobalagent)
    * [http.request(options[, callback])](#httprequestoptions-callback)

# HTTP

    稳定性： 2 - 稳定

通过 `require('http')` 可以使用 HTTP 服务器或客户端功能。

Node.js 中的 HTTP 接口设计旨在支持 HTTP 协议中的那些原本用起来很复杂的功能，特别是庞大的和块编码的消息。这些接口会谨慎地不去缓存完整的请求或响应信息，用户可以在请求和响应中去使用数据流。

HTTP 消息头可以用类似这样的对象表示：

```
{ 'content-length': '123',
  'content-type': 'text/plain',
  'connection': 'keep-alive',
  'host': 'mysite.com',
  'accept': '*/*' }
```

属性都是小写字母，值不可以修改。

为了能全面地支持尽可能多的 HTTP 应用程序，Node.js 的 HTTP API 非常底层，它仅仅处理流和解析消息。它会将消息解析成消息头和消息体，但它并不会解析实际的消息头和消息体。

参见 [`message.headers`][] 了解重复的消息头是如何处理的。

接收到的原始的消息头保存在 `rawHeaders` 属性中，是一个类似这样的数组 `[key, value, key2, value2, ...]`。例如， 之前那个消息头对象可能会有一个 `rawHeaders` 列表如下：

```
[ 'ConTent-Length', '123456',
  'content-LENGTH', '123',
  'content-type', 'text/plain',
  'CONNECTION', 'keep-alive',
  'Host', 'mysite.com',
  'accepT', '*/*' ]
```

## 类： http.Agent

HTTP Agent 用来将 HTTP 客户端请求中的套接字（socket）做成资源池以供使用。

HTTP Agent 也会将客户端请求默认使用 Connection:keep-alive。如果没有挂起的 HTTP 请求正在等待成为空闲套接字，那么套接字将会关闭。这意味着 Node.js 的资源池在负载情况下对 keep-alive 是有利的，但仍然不需要开发者使用 KeepAlive 去手动关闭 HTTP 客户端。

如果你选择使用 HTTP KeepAlive, 你可以创建一个 Agent 对象并将标记设为 `true`（参见 [构造函数选项][]），这样 Agent 会把未被使用的套接字放入资源池中供将来使用。他们会被显式标记出来以不让 Node.js 运行。然而，当它们不再使用时，显式地调用 [`destroy()`][] 依然是一个好主意，这样套接字就会被关闭。

当套接字触发了 `'close'` 事件或者特殊的 `'agentRemove'` 事件，套接字将会从代理的资源池中移除。这意味着如果你打算保持 HTTP 请求长时间打开，并且又不想它们保持在资源池中，你可以这样做：

```js
http.get(options, (res) => {
  // Do stuff
}).on('socket', (socket) => {
  socket.emit('agentRemove');
});
```

或者，你可以选择使用 `agent:false` 让资源池停用：

```js
http.get({
  hostname: 'localhost',
  port: 80,
  path: '/',
  agent: false  // 仅为这次请求创建一个新的代理
}, (res) => {
  // Do stuff with response
})
```

### new Agent([options])

* `options` {Object} 代理的配置项集合。可以有如下字段：
  * `keepAlive` {Boolean} 保持资源池中的套接字以便在将来可以被其它请求使用。默认值 = `false`。
  * `keepAliveMsecs` {Integer} 当使用 HTTP KeepAlive 时通过保持活动的套接字发送 TCP KeepAlive 包的频率。默认值 = `1000`。仅当 `keepAlive` 设为 `true` 才有效。
  * `maxSockets` {Number} 单个主机允许的套接字的最大数量。默认值 = `Infinity`。
  * `maxFreeSockets` {Number} 空闲状态下依然打开的套接字的最大数量。仅当 `keepAlive` 设为 `true` 才有效。默认值 = `256`。

[`http.request()`][] 使用的默认的 [`http.globalAgent`][] 有全部这些值，且已被设为它们各自的默认值。

要配置这些值，你必须创建你自己的 [`http.Agent`][] 对象。

```js
const http = require('http');
var keepAliveAgent = new http.Agent({ keepAlive: true });
options.agent = keepAliveAgent;
http.request(options, onResponseCallback);
```

### agent.destroy()

销毁当前代理占用的全部套接字。

通常没必要这么做。然而，如果你正在使用一个保持连接（KeepAlive）的代理，当你确定不会再使用该代理后最好显式得把它关闭。否则，套接字在服务器终止它们之前可能会打开相当长的一段时间。

### agent.freeSockets

当使用 HTTP KeepAlive 时的正在等待用于 Agent 的套接字数组对象。请不要修改它。

### agent.getName(options)

为一组请求选项获得一个唯一名称，来确定一个连接是否可以重用。在 http 代理中，它将返回 `host:port:localAddress`。在 https 代理中，名称将包含 CA（认证机构）、cert（证书）、ciphers（密码）以及其他 HTTPS/TLS 特定的选项来决定套接字是否可以重用。

### agent.maxFreeSockets

默认值是 256.对于支持 HTTP KeepAlive 的 Agent，这将设置在空闲状态下最多允许多少个套接字仍然打开。

### agent.maxSockets

默认值是 Infinity。确定每个源最多允许代理可以包含多少个并发套接字。源可以是 'host:port' 或 'host:port:localAddress' 结合体。

### agent.requests

还没有指定套接字的请求队列的一个对象。请不要修改它。

### agent.sockets

当前正被 Agent 使用的套接字的数组对象。请不要修改它。

## 类： http.ClientRequest

该对象在内部创建，由 [`http.request()`][] 返回。它表示一个消息头已经进入了请求队列的 _处理中_ 的请求。消息头还是不确定的，你可以使用 `setHeader(name, value)`， `getHeader(name)`，`removeHeader(name)` 修改它。实际的消息头将会伴随第一个数据块发送或在连接关闭时发送。

为了获得响应，给请求对象添加一个 `'response'` 监听器。当接收到了响应消息头，请求对象会触发 `'response'` 事件。`'response'` 会有一个执行参数，该参数是 [`http.IncomingMessage`][] 的一个实例。

在 `'response'` 事件期间，你可以给响应对象添加监听器，尤其是监听 `'data'` 事件。

如果没有添加 `'response'` 处理函数，响应将会被完全丢弃。然而，如果你添加了一个 `'response'` 时间处理函数，那么你 **必须** 消费掉响应对象中的数据，可以当存在 `'readable'` 事件时调用 `response.read()` ，或者添加一个 `'data'` 处理函数，或者调用 `.resume()` 方法。当数据被消费掉后， `'end'` 事件将会触发。如果不去读取数据，则数据将一直消耗内存，最终导致 'process out of memory' 错误。

注意：Node.js 不会检查 Content-Length 与已被传输的消息体的长度是否相等。

该请求实现了 [Writable Stream][] 接口。这是一个 [`EventEmitter`][]，包含如下事件：

### 事件： 'abort'

`function () { }`

当请求被客户端终止时触发。该事件只在首次调用 `abort()` 时触发。

### 事件： 'connect'

`function (response, socket, head) { }`

每次服务器响应客户端发起的 `CONNECT` 请求时触发。如果未监听该事件，客户端接收到 `CONNECT` 方法时将关闭连接。

下面这个客户端与服务器连接示例会演示如何监听 `'connect'` 事件。

```js
const http = require('http');
const net = require('net');
const url = require('url');

// 创建一个 HTTP 隧道代理
var proxy = http.createServer( (req, res) => {
  res.writeHead(200, {'Content-Type': 'text/plain'});
  res.end('okay');
});
proxy.on('connect', (req, cltSocket, head) => {
  // 连接到一个源服务器
  var srvUrl = url.parse(`http://${req.url}`);
  var srvSocket = net.connect(srvUrl.port, srvUrl.hostname, () => {
    cltSocket.write('HTTP/1.1 200 Connection Established\r\n' +
                    'Proxy-agent: Node.js-Proxy\r\n' +
                    '\r\n');
    srvSocket.write(head);
    srvSocket.pipe(cltSocket);
    cltSocket.pipe(srvSocket);
  });
});

// 现在代理正在运行
proxy.listen(1337, '127.0.0.1', () => {

  // 发起一个请求到隧道代理
  var options = {
    port: 1337,
    hostname: '127.0.0.1',
    method: 'CONNECT',
    path: 'www.google.com:80'
  };

  var req = http.request(options);
  req.end();

  req.on('connect', (res, socket, head) => {
    console.log('已经连接！');

    // 通过 HTTP 隧道发起一个请求
    socket.write('GET / HTTP/1.1\r\n' +
                 'Host: www.google.com:80\r\n' +
                 'Connection: close\r\n' +
                 '\r\n');
    socket.on('data', (chunk) => {
      console.log(chunk.toString());
    });
    socket.on('end', () => {
      proxy.close();
    });
  });
});
```

### 事件： 'continue'

`function () { }`

当服务器发送一个 '100 Continue' 的 HTTP 响应时触发，通常是因为请求包含了 'Expect: 100-continue'。这是一个指令，要求客户端发送请求体。

### 事件： 'response'

`function (response) { }`

当响应被请求接收到时触发。该事件只触发一次。`response` 参数是 [`http.IncomingMessage`][] 的实例。

选项：

- `host`: 请求发给的域名或服务器的 IP 地址。
- `port`: 远程服务器的端口。
- `socketPath`: Unix 域套接字（使用 host:port 或 socketPath）

### 事件： 'socket'

`function (socket) { }`

当一个套接字被指派给该请求时触发。

### 事件： 'upgrade'

`function (response, socket, head) { }`

每次服务器响应客户端发起的升级（upgrade）请求时触发。如果未监听该事件，客户端接收到一个升级消息头时将关闭连接。

下面这个客户端与服务器连接示例会演示如何监听 `'upgrade'` 事件。

```js
const http = require('http');

// 创建一个 HTTP 服务器
var srv = http.createServer( (req, res) => {
  res.writeHead(200, {'Content-Type': 'text/plain'});
  res.end('okay');
});
srv.on('upgrade', (req, socket, head) => {
  socket.write('HTTP/1.1 101 Web Socket Protocol Handshake\r\n' +
               'Upgrade: WebSocket\r\n' +
               'Connection: Upgrade\r\n' +
               '\r\n');

  socket.pipe(socket); // echo back
});

// 现在服务器正在运行
srv.listen(1337, '127.0.0.1', () => {

  // 发起一个请求
  var options = {
    port: 1337,
    hostname: '127.0.0.1',
    headers: {
      'Connection': 'Upgrade',
      'Upgrade': 'websocket'
    }
  };

  var req = http.request(options);
  req.end();

  req.on('upgrade', (res, socket, upgradeHead) => {
    console.log('连接已升级！');
    socket.end();
    process.exit(0);
  });
});
```

### request.abort()

标志着请求终止。调用该方法将导致丢弃响应中的剩余数据，套接字也将被销毁。

### request.end([data][, encoding][, callback])

结束发送请求。如果请求体的某些部分还未发送，这部分将被放入流中。如果请求体是分块的，将会发送终止符 `'0\r\n\r\n'`。

如果指定了 `data`，则相当于先调用 [`response.write(data, encoding)`][] 后再调用 `request.end(callback)`。

如果指定了 `callback` ，当请求流结束时会调用它。

### request.flushHeaders()

刷新请求头。

由于效率的原因，Node.js 通常会缓冲请求头直到你调用了 `request.end()` 或者写了请求的第一个数据块。然后会艰难地尝试把请求头和数据放入一个单独的 TCP 数据包。

通常这就是你想要的（它节省了 TCP 往返时间），但如果第一批数据直到很久以后才发送就不行了。 `request.flushHeaders()` 能让你绕过优化和启动请求。

### request.setNoDelay([noDelay])

一旦一个套接字被分配给了该请求并完成连接，[`socket.setNoDelay()`][] 将被调用。

### request.setSocketKeepAlive([enable][, initialDelay])

一旦一个套接字被分配给了该请求并完成连接，[`socket.setKeepAlive()`][] 将被调用。

### request.setTimeout(timeout[, callback])

一旦一个套接字被分配给了该请求并完成连接，[`socket.setTimeout()`][] 将被调用。

* `timeout` {Number} 请求被发送之前超时的毫秒数。
* `callback` {Function} 可选，当发生一个超时时调用。与绑定到 `timeout` 事件是等价的。

### request.write(chunk[, encoding][, callback])

发送请求体中的一块数据。通过多次调用该方法，用户能以数据流方式发送请求体到服务器——这种情况下当创建请求时建议使用 `['Transfer-Encoding', 'chunked']` 请求头。

`chunk` 参数必须是 [`Buffer`][] 或者字符串。

`encoding` 参数是可选的，且仅当 `chunk` 是字符串时有效。默认值为 `'utf8'`。

`callback` 参数是可选的，当该数据块写入后调用。

返回 `request`。

## 类： http.Server

该类继承自 [`net.Server`][] 并有下列额外事件：

### 事件： 'checkContinue'

`function (request, response) { }`

每次接收到 http Expect: 100-continue 请求时触发。如果未监听该事件，服务器将视情况自动发送100 Continue 响应。

处理该事件时，如果客户端需要继续发送请求体，则调用 [`response.writeContinue()`][] ，否则就生成一个合适的 HTTP 响应（例如： 400 错误的请求）。

请注意，触发和处理该事件后， `'request'` 事件将不会再触发。

### 事件： 'clientError'

`function (exception, socket) { }`

如果一个客户端连接触发了一个 `'error'` 事件，该错误事件会被转发到这里。

`socket` 是 导致错误的 [`net.Socket`][] 对象。

### 事件： 'close'

`function () { }`

当服务器关闭时触发。

### 事件： 'connect'

`function (request, socket, head) { }`

每次客户端发起一个 http `CONNECT` 请求时触发。如果未监听该事件，客户端发起 `CONNECT` 请求时将关闭连接。

* `request` 是该 http 请求的参数，与 request 事件中的参数一样。
* `socket` 是服务器与客户端之间的网络套接字。
* `head` 是 Buffer 的一个实例，隧道流的第一个数据包，可能为空。

当该事件被触发后，请求的套接字将不会存在 `'data'` 事件监听器，这意味着你将需要绑定一个 `data` 监听器以便处理套接字上发送到服务器的数据。

### 事件： 'connection'

`function (socket) { }`

当建立一个新的 TCP 流时触发。 `socket` 是一个 [`net.Socket`][] 对象。通常用户不需要处理该事件。特别注意，协议解析器连接套接字的方式使得套接字不会触发 `'readable'` 事件。也可以通过 `request.connection` 来访问 `socket` 。

### 事件： 'request'

`function (request, response) { }`

每次收到一个请求时触发。注意一个连接可能会有多个请求（在 keep-alive 连接中)。 `request` 是一个 [`http.IncomingMessage`][] 的实例，`response` 是一个 [`http.ServerResponse`][] 的实例。

### 事件： 'upgrade'

`function (request, socket, head) { }`

每次客户端发起一个升级请求时触发。如果未监听该事件，当客户端发起升级请求时将关闭连接。

* `request` 是该 http 请求的参数，与 request 事件中的参数一样。
* `socket` 是服务器与客户端之间的网络套接字。
* `head` 是 Buffer 的一个实例，隧道流的第一个数据包，可能为空。

当该事件被触发后，请求的套接字将不会存在 `'data'` 事件监听器，这意味着你将需要绑定一个 `data` 监听器以便处理套接字上发送到服务器的数据。

### server.close([callback])

让服务器停止接收新的连接。参见 [`net.Server.close()`][]。

### server.listen(handle[, callback])

* `handle` {Object}
* `callback` {Function}

`handle` 可被设置为一个服务器或者套接字（任何以下划线 `_handle` 开头的成员），或者一个 `{fd: <n>}` 对象。

这将导致服务器接收指定 handle 上的连接，但它假设文件描述符或 handle 已经被绑定到了一个端口或域名套接字上。

监听文件描述符在 WIndows 平台不可用。

该函数是异步的。最后一个参数 `callback` 将会被作为监听器添加到 `'listening'` 事件。参见 [`net.Server.listen()`][]。

返回 `server`。

### server.listen(path[, callback])

启动一个 UNIX 套接字服务器在所给的 `path` 上监听连接。

该函数是异步的。最后一个参数 `callback` 将会被作为监听器添加到 `'listening'` 事件。参见 [`net.Server.listen(path)`][]。

### server.listen(port[, hostname][, backlog][, callback])

开始接收指定的 `port` 和 `hostname` 上的连接。如果 `hostname` 未指定，当 IPv6 可用时，服务器将接收任意 IPv6 地址（`::`）的连接，若 IPv6不可用，则接收 IPv4 地址（`0.0.0.0`）地址的连接。端口值为 0 将会随机分配一个端口。

要监听一个 unix 套接字i，需要提供一个文件名而不是端口号和主机名。

backlog 是挂起连接队列的最大长度。实际的长度由操作系统通过 sysctl 设置，例如 linux 上的 `tcp_max_syn_backlog` 和 `somaxconn`。 该参数的默认值是 511（不是 512）。

该函数是异步的。最后一个参数 `callback` 将会被作为监听器添加到 `'listening'` 事件。参见 [`net.Server.listen(port)`][]。

### server.maxHeadersCount

允许的最大请求头数量，默认为 1000. 如果设置为 0 则代表无限制。

### server.setTimeout(msecs, callback)

* `msecs` {Number}
* `callback` {Function}

为套接字设置超时时间，并在 Server 对象上触发一个 `'timeout'` 事件，当发生超时，则将套接字作为参数传入该事件。

如果在 Server 对象上有一个 `'timeout'` 事件监听器，该监听器将被调用，超时的套接字会作为参数传入。

默认情况下，服务器的超时值是 2 分钟，超时后套接字将被自动销毁。然而，如果你为 Server 的 `'timeout'` 事件指定了回调函数，那么你就需要自己负责处理套接字超时。

返回 `server`。

### server.timeout

* {Number} 默认值 = 120000 （2 分钟）

当一个套接字被推断为已超时之前的毫秒数。

注意套接字的超时逻辑是建立连接时设定的，所以修改这个值只会影响到该服务器的 *新的* 连接，已存在的连接不会受影响。

设为 0 可以阻止接收的连接的任何自动超时行为。

## 类： http.ServerResponse

该对象由 HTTP 服务器在内部创建——而不是由用户创建。它作为第二个参数传入 `'request'` 事件。

该响应实现了 [Writable Stream][] 接口。这是一个包含如下事件的 [`EventEmitter`][]：

### 事件： 'close'

`function () { }`

表明底层连接在 [`response.end()`][] 被调用或冲洗之前就被终止了。

### 事件： 'finish'

`function () { }`

当响应已完成发送时触发。更具体地说，该事件会在最后一段响应消息头和消息体交给操作系统通过网络传输时触发。这并不意味着客户端已经接收到了数据。

该事件之后，响应对象不会再触发其它事件。

### response.addTrailers(headers)

该方法给响应添加 HTTP 尾部消息头（一个在消息末尾的消息头）。

**仅当** 数据块编码被用于响应时尾部（trailers）才会被触发，如果不是（例如请求时 HTTP/1.0），它们会被静默丢弃。

请注意如果你想触发尾部，HTTP 要求消息头字段列表和 `Trailer` 消息头要一起发送。例如：

```js
response.writeHead(200, { 'Content-Type': 'text/plain',
                          'Trailer': 'Content-MD5' });
response.write(fileData);
response.addTrailers({'Content-MD5': '7895bf4b8828b55ceaf47747b4bca667'});
response.end();
```

试图给消息头字段的名称或值设置非法字符将会导致 [`TypeError`][] 错误。

### response.end([data][, encoding][, callback])

该方法会通知服务器所有响应消息头和消息体都已被发送。服务器会认为消息已经完成。每次响应完成后都必须调用该方法。

如果指定了 `data`，则相当于先调用 [`response.write(data, encoding)`][] 后再调用 `request.end(callback)`。

如果指定了 `callback` ，当请求流结束时会调用它。

### response.finished

布尔值，用于表明响应是否已完成。初始值为 `false`。在 [`response.end()`][] 执行后，值将变为 `true`。

### response.getHeader(name)

读取一个已存在与队列中但还没发送给客户端的消息头。注意名称是不区分大小写的。注意只能在消息头被冲刷掉之前使用。

Example:

```js
var contentType = response.getHeader('content-type');
```

### response.headersSent

布尔值（只读）。如果消息头已发送则为 true ，否则为 false。

### response.removeHeader(name)

移除一个队列中等待隐式发送的消息头。

Example:

```js
response.removeHeader('Content-Encoding');
```

### response.sendDate

若为 true，当响应的消息头中不存在日期（Date）时会自动生成日期并发送。默认值是 true。

这仅仅应该在测试环境下被禁用。HTTP 要求响应头应包含日期。

### response.setHeader(name, value)

为隐式消息头设置一个单独的值。如果该消息头正等待被发送，它的值将会被覆盖。如果你想发送具有相同名字的多个消息头，请使用一个字符串数组。

示例：

```js
response.setHeader('Content-Type', 'text/html');
```

或

```js
response.setHeader('Set-Cookie', ['type=ninja', 'language=javascript']);
```

试图给消息头字段的名称或值设置非法字符将会导致 [`TypeError`][] 错误。

通过 [`response.setHeader()`][] 设置的消息头，将与传递给 [`response.writeHead()`][] 的消息头合并，传递给 [`response.writeHead()`][] 的优先。

```js
// 响应的 content-type 是 text/plain
const server = http.createServer((req,res) => {
  res.setHeader('Content-Type', 'text/html');
  res.setHeader('X-Foo', 'bar');
  res.writeHead(200, {'Content-Type': 'text/plain'});
  res.end('ok');
});
```

### response.setTimeout(msecs, callback)

* `msecs` {Number}
* `callback` {Function}

`msecs` 用来设置套接字的超时时间值。如果提供了回调函数，则回调函数会被添加为响应对象的 `'timeout'` 事件的监听器。

如果请求、响应、服务器都未添加 `'timeout'` 监听器，套接字将在它们超时后被销毁。如果在请求、响应、服务器上添加了 `'timeout'` 事件监听，那么需要你自己去处理超时的套接字。

返回 `response`。

### response.statusCode

当使用隐式消息头（没有显式地调用 [`response.writeHead()`][] 修改），
这个属性控制了当消息头刷新时被发送到客户端的状态码。

示例：

```js
response.statusCode = 404;
```

当响应消息头被发送到客户端后，该属性表示已经发送出去的状态码。

### response.statusMessage

当使用隐式消息头（没有显式地调用 [`response.writeHead()`][] 修改），
这个属性控制了当消息头刷新时被发送到客户端的状态消息。如果未设置，即 `undefined` ，则会使用状态码的标准消息。

示例：

```js
response.statusMessage = 'Not found';
```

当响应消息头被发送到客户端后，该属性表示已经发送出去的状态消息。

### response.write(chunk[, encoding][, callback])

如果调用了该方法但没有调用 [`response.writeHead()`][]，它将切换到隐式消息头模式并刷新隐式消息头。

该方法用来发送一个响应体的数据块。它可能被多次调用以支持响应体的连续的部分。

`chunk` 可以是字符串或缓冲区。如果 `chunk` 是一个字符串，第二个参数就表示就何种编码将其转为字节流。默认情况下 `encoding` 是 `'utf8'`。最后一个参数 `callback` 会在数据块被冲刷后调用。

**注意**：这是原始的 HTTP 消息体，与可能使用的高级多部分的消息体编码无关。

当 [`response.write()`][] 被首次调用，将会发送缓存的消息头和第一个消息体给客户端。第二次调用 [`response.write()`][]，Node.js 假定你将使用数据流发送，然后分别地发送。也就是说，响应被缓存到消息体的一个数据块中。

当全部数据被冲刷到内核缓冲区后，将返回 `true`，如果全部或部分数据还在用户内存队列中，则返回 `false`。当缓冲区再次空闲时会触发 `'drain'` 事件。

### response.writeContinue()

发送一个 HTTP/1.1 100 Continue 消息给客户端，表明应该发送请求体。参见 `Server` 上的 [`'checkContinue'`][] 事件。

### response.writeHead(statusCode[, statusMessage][, headers])

向请求发送一个响应消息头。`statusCode` 是一个三位数的 HTTP 状态码，如 `404`。最后一个参数 `headers` 是响应消息头。
你还可以添加一个可读性良好的 `statusMessage` 作为第二个参数，这是可选的。

示例：

```js
var body = 'hello world';
response.writeHead(200, {
  'Content-Length': body.length,
  'Content-Type': 'text/plain' });
```

该方法在一个消息上必须仅调用一次，而且必须在 [`response.end()`][] 之前调用。

如果在调用该方法之前调用了 [`response.write()`][] 或者 [`response.end()`][]，隐式不确定的消息头将被计算并调用该函数。

如果已经用 [`response.setHeader()`][] 设置过了消息头，将与传递给 [`response.writeHead()`][] 的消息头合并，传递给 [`response.writeHead()`][] 的优先。

```js
// 响应的 content-type 是 text/plain
const server = http.createServer((req,res) => {
  res.setHeader('Content-Type', 'text/html');
  res.setHeader('X-Foo', 'bar');
  res.writeHead(200, {'Content-Type': 'text/plain'});
  res.end('ok');
});
```

需要注意 Content-Length 是字节而非字符。上面的例子可以正常工作是因为字符串 `'hello world'` 仅包含单字节的字符。如果消息体包含了多字节的字符，就需要使用 `Buffer.byteLength()` 来确定给定编码下的字节数。Node.js 不会检查 Content-Length 与已被传输的消息体的长度是否相等。

试图给消息头字段的名称或值设置非法字符将会导致 [`TypeError`][] 错误。

## 类： http.IncomingMessage

`IncomingMessage` 对象由 [`http.Server`][] 或 [`http.ClientRequest`][] 创建，并作为第一个参数分别传递给 `'request'` 和 `'response'` 事件。它可以被用来访问响应状态，响应头和数据。

它实现了 [Readable Stream][] 接口，还包括如下额外的事件、方法和属性。

### 事件： 'close'

`function () { }`

表明底层连接已被关闭。就跟 `'end'` 事件一样，对每个响应只会触发一次。

### message.headers

请求/响应的消息头对象。

消息头的名称和值组成的键值对。消息头名称是小写的。示例：

```js
// 类似这样显示：
//
// { 'user-agent': 'curl/7.22.0',
//   host: '127.0.0.1:8000',
//   accept: '*/*' }
console.log(request.headers);
```

对于原始消息头中的重复项，会以下面的方式处理，根据消息头的名称：

* `age`, `authorization`, `content-length`, `content-type`,
`etag`, `expires`, `from`, `host`, `if-modified-since`, `if-unmodified-since`,
`last-modified`, `location`, `max-forwards`, `proxy-authorization`, `referer`,
`retry-after`, 或 `user-agent` 的重复项会被舍弃。
* `set-cookie` 总是一个数组。重复项会被添加到数组中。
* 其它的消息头，重复项的值会以 ', ' 合并成一个值。

### message.httpVersion

当客户端发送请求时，客户端发送的 HTTP 版本。或是当服务器响应请求时，服务器的 HTTP 版本。可能是 `'1.1'` 或 `'1.0'`。

`message.httpVersionMajor` 是返回值的第一个整数，`message.httpVersionMinor` 是第二个整数。

### message.method

**仅对从 [`http.Server`][] 获得的请求有效。**

请求的方法是一个只读字符串。例如：
`'GET'`，`'DELETE'`。

### message.rawHeaders

接收到的最原始的请求/响应消息头列表。

请注意键值在同同一个列表中。它*不是*一个元祖列表。所以，偶数偏移量是键，奇数偏移量是对应的值。

消息头名称不是小写的，而且重复的消息头也没有合并。

```js
// 类似这样显示：
//
// [ 'user-agent',
//   'this is invalid because there can be only one',
//   'User-Agent',
//   'curl/7.22.0',
//   'Host',
//   '127.0.0.1:8000',
//   'ACCEPT',
//   '*/*' ]
console.log(request.rawHeaders);
```

### message.rawTrailers

接收到的最原始的请求/响应尾部对象。只会在 `'end'` 事件中存在。

### message.setTimeout(msecs, callback)

* `msecs` {Number}
* `callback` {Function}

调用 `message.connection.setTimeout(msecs, callback)`。

返回 `message`。

### message.statusCode

**仅对从 [`http.ClientRequest`][] 获得的响应有效。**

三位数的 HTTP 响应状态码，如 `404`。

### message.statusMessage

**仅对从 [`http.ClientRequest`][] 获得的响应有效。**

HTTP 响应状态消息（关于状态原因的短语），如 `OK` 或 `Internal Server Error`。

### message.socket

与该连接有关的 [`net.Socket`][] 对象。

通过 HTTPS 的支持，使用 [`request.socket.getPeerCertificate()`][] 可以获得客户端的身份验证信息。

### message.trailers

请求/响应中的尾部对象。只会在 `'end'` 事件中存在。

### message.url

**仅对从 [`http.Server`][] 获得的请求有效。**

请求的 URL 字符串。它仅包含实际 HTTP 请求中出现的 URL。比如下面的请求：

```
GET /status?name=ryan HTTP/1.1\r\n
Accept: text/plain\r\n
\r\n
```

`request.url` 将是：

```
'/status?name=ryan'
```

如果你想要解析该 URL 为各个部分，你可以使用 `require('url').parse(request.url)`。示例：

```
$ node
> require('url').parse('/status?name=ryan')
{
  href: '/status?name=ryan',
  search: '?name=ryan',
  query: 'name=ryan',
  pathname: '/status'
}
```

如果你想要从查询字符串中提取参数，你可以使用 `require('querystring').parse` 函数，或者将 `true` 作为第二个参数传入 `require('url').parse`。示例：

```
$ node
> require('url').parse('/status?name=ryan', true)
{
  href: '/status?name=ryan',
  search: '?name=ryan',
  query: {name: 'ryan'},
  pathname: '/status'
}
```

## http.METHODS

* {Array}

解析器所支持的 HTTP 方法的列表。

## http.STATUS_CODES

* {Object}

所有标准 HTTP 响应状态码及其简短描述的集合。例如：`http.STATUS_CODES[404] === 'Not Found'`。

## http.createClient([port][, host])

    稳定性： 0 - 弃用：请使用 [`http.request()`][] 代替。

创建一个新的 HTTP 客户端。 `port` 和 `host` 指向要连接的服务器。

## http.createServer([requestListener])

返回一个新的 [`http.Server`][] 的实例。

`requestListener` 是一个被自动加入 `'request'` 事件监听队列中的函数。

## http.get(options[, callback])

因为大多数请求都是 GET 请求，没有消息体，Node.js 就提供了这个简便的方法。它与 [`http.request()`][] 唯一的不同在于它设置了 HTTP 方法为 GET 并会自动调用 `req.end()`。

实例：

```js
http.get('http://www.google.com/index.html', (res) => {
  console.log(`Got response: ${res.statusCode}`);
  // 消费响应的消息体
  res.resume();
}).on('error', (e) => {
  console.log(`出错了：${e.message}`);
});
```

## http.globalAgent

Agent 的全局实例，作为所有 http 客户端请求的默认对象。

## http.request(options[, callback])

Node.js 会维护单个服务器上的多个连接的请求。该函数允许透明地发出请求。

`options` 可以是一个对象或者一个字符串。如果 `options` 是一个字符串，会自动使用 [`url.parse()`][] 进行解析。

Options:

- `protocol`：使用的协议。默认是 `'http:'`。
- `host`：请求目标的域名或服务器的 IP 地址。默认是 `'localhost'`。
- `hostname`：`host` 的别名。要支持 [`url.parse()`][] `hostname` 优先于 `host`。
- `family`: 解析 `host` 和 `hostname` 时使用的 IP 地址类型。有效值是 `4` 或 `6`。如果未指定，则会使用 IP v4 和 v6 。
- `port`：远程服务器的端口号。默认是 80.
- `localAddress`：网络连接绑定的本地接口。
- `socketPath`：Unix 域名套接字（使用 主机:端口号 或套接字路径）。
- `method`：HTTP 请求方法。默认值是 `'GET'`。
- `path`：请求路径。默认值是 `'/'`。如果有查询字符串也需要包含在内。例如：`'/index.html?page=12'`。当请求路径包含非法字符时会抛出异常。当前只有空格视作非法，但将来可能会变化。
- `headers`：包含请求消息头的对象。
- `auth`：基础验证，即 `'user:password'` 计算出一个验证消息头。
- `agent`：控制 [`Agent`][] 的行为。当使用了 Agent 则请求将默认是 `Connection: keep-alive`。可能的值：
 - `undefined` （默认值），为当前的主机和端口使用 [`http.globalAgent`][]。
 - `Agent` 对象： 显式地使用传入的 `Agent`。
 - `false`：选择与 Agent 相关的连接池，默认连接是 `Connection: close`。

可选的 `callback` 参数将作为一个一次性的监听器添加到 `'response'` 事件上。

`http.request()` 返回 [`http.ClientRequest`][] 类的一个实例。`ClientRequest` 实例是一个可写流。如果需要通过 POST 请求上传一个文件，就可以通过写入 `ClientRequest` 对象。

示例：

```js
var postData = querystring.stringify({
  'msg' : 'Hello World!'
});

var options = {
  hostname: 'www.google.com',
  port: 80,
  path: '/upload',
  method: 'POST',
  headers: {
    'Content-Type': 'application/x-www-form-urlencoded',
    'Content-Length': postData.length
  }
};

var req = http.request(options, (res) => {
  console.log(`STATUS: ${res.statusCode}`);
  console.log(`HEADERS: ${JSON.stringify(res.headers)}`);
  res.setEncoding('utf8');
  res.on('data', (chunk) => {
    console.log(`BODY: ${chunk}`);
  });
  res.on('end', () => {
    console.log('响应的数据已读取完毕！')
  })
});

req.on('error', (e) => {
  console.log(`请求出错：${e.message}`);
});

// 写入数据到请求体
req.write(postData);
req.end();
```

请注意，这个例子调用了 `req.end()`。 `http.request()` 中你必须总是调用 `req.end()` 来表明你完成了请求——即便没有任何数据写入到请求体。

在请求过程中遇到的任何错误（如DNS解析、TCP 错误或实际的 HTTP 解析错误）将会在返回的请求对象上触发 `'error'` 事件。与所有的 `'error'` 事件一样，如果没有注册任何监听器则错误将会被抛出。

有一些需要注意的特殊的消息头：

* 发送一个 'Connection: keep-alive' 将会通知 Node.js 到服务器的连接应该一直持续到下一个请求。

* 发送一个 'Content-length' 消息头将会禁用默认的块编码。

* 发送一个 'Expect' 消息头将立即发送请求消息头。通常，当发送 'Expect: 100-continue'，你应该设置一个超时，并且监听 `'continue'` 事件。查看 RFC2616 Section 8.2.3。

* 发送一个 Authorization 消息头将会覆盖用于计算基础验证的 `auth`  选项。

[`'checkContinue'`]: #事件-checkcontinue
[`'listening'`]: net.markdown#事件-listening
[`'response'`]: #事件-response
[`Agent`]: #类-httpagent
[`Buffer`]: buffer.markdown#buffer
[`destroy()`]: #agentdestroy
[`EventEmitter`]: events.markdown#类-eventemitter
[`http.Agent`]: #类-httpagent
[`http.ClientRequest`]: #类-httpclientrequest
[`http.globalAgent`]: #httpglobalagent
[`http.IncomingMessage`]: #类-httpincomingmessage
[`http.request()`]: #httprequestoptions-callback
[`http.Server`]: #类-httpserver
[`http.ServerResponse`]: #类-httpserverresponse
[`message.headers`]: #messageheaders
[`net.Server`]: net.markdown#类-server
[`net.Server.close()`]: net.markdown#serverclosecallback
[`net.Server.listen()`]: net.markdown#serverlistenhandle-callback
[`net.Server.listen(path)`]: net.markdown#serverlistenpath-callback
[`net.Server.listen(port)`]: net.markdown#serverlistenport-hostname-backlog-callback
[`net.Socket`]: net.markdown#类-socket
[`request.socket.getPeerCertificate()`]: tls.markdown#tlssocketgetpeercertificatedetailed
[`response.end()`]: #responseenddata-encoding-callback
[`response.setHeader()`]: #responsesetheadername-value
[`response.write()`]: #responsewritechunk-encoding-callback
[`response.write(data, encoding)`]: #responsewritechunk-encoding-callback
[`response.writeContinue()`]: #responsewritecontinue
[`response.writeHead()`]: #responsewriteheadstatuscode-statusmessage-headers
[`socket.setKeepAlive()`]: net.markdown#socketsetkeepaliveenable-initialdelay
[`socket.setNoDelay()`]: net.markdown#socketsetnodelay-nodelay
[`socket.setTimeout()`]: net.markdown#socketsettimeouttimeout-callback
[`stream.setEncoding()`]: stream.markdown#streamsetencodingencoding
[`TypeError`]: errors.markdown#类-typeerror
[`url.parse()`]: url.markdown#urlparseurlstr-parsequerystring-slashesdenotehost
[构造函数选项]: #new-agentoptions
[Readable Stream]: stream.markdown#类-streamreadable
[Writable Stream]: stream.markdown#类-streamwritable
