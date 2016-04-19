## 目录
* [概要](#概要)

# 概要

<!--type=misc-->

Node.js 写的一个 [web服务器][] 的例子，会返回 `'Hello World'` 作为响应：

```js
const http = require('http');

http.createServer( (request, response) => {
  response.writeHead(200, {'Content-Type': 'text/plain'});
  response.end('Hello World\n');
}).listen(8124);

console.log('Server running at http://127.0.0.1:8124/');
```

要启动该服务器，将下面的代码放入一个名为 `example.js` 的文件中然后用 node 程序执行它：

```
$ node example.js
Server running at http://127.0.0.1:8124/
```

文档中的所有示例都可以类似地运行。

[web服务器]: http.markdown
