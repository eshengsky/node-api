## 目录
* [URL](#url)
  * [URL 解析](#url-解析)
    * [转义字符](#转义字符)
  * [url.format(urlObj)](#urlformaturlobj)
  * [url.parse(urlStr[, parseQueryString][, slashesDenoteHost])](#urlparseurlstr-parsequerystring-slashesdenotehost)
  * [url.resolve(from, to)](#urlresolvefrom-to)

# URL

    稳定性： 2 - 稳定

该模块包含对 URL 进行解析的工具。通过 `require('url')` 来使用该模块。

## URL 解析

解析后的 URL 对象包括如下部分或全部的字段，取决于 URL 字符串中是否存在对应字段。URL 字符串中不存在的部分也不会出现在解析后的对象中。这是例子所示的 URL：

`'http://user:pass@host.com:8080/p/a/t/h?query=string#hash'`

* `href`：所解析的完整原始的 URL。协议和主机名都被换转为小写。

    示例：`'http://user:pass@host.com:8080/p/a/t/h?query=string#hash'`

* `protocol`：请求协议，小写。

    示例：`'http:'`

* `slashes`：协议在冒号之后是否有斜杠。

    示例：true 或 false

* `host`：URL 中完整且小写的主机部分，包括端口信息。

    示例：`'host.com:8080'`

* `auth`：URL 中验证信息部分。

    示例：`'user:pass'`

* `hostname`：小写的主机名部分。

    示例：`'host.com'`

* `port`：端口号部分。

    示例：`'8080'`

* `pathname`：URL 中的路径部分，位于主机部分之后、查询之前，包括开头的斜杠——如果有的话。路径未解码。

    示例：`'/p/a/t/h'`

* `search`：URL 中的查询字符串部分，包括开头的问号。

    示例：`'?query=string'`

* `path`：`pathname` 和 `search` 连接在一起的部分。并没有解码。

    示例：`'/p/a/t/h?query=string'`

* `query`：查询字符串中的参数部分，或者使用 `querystring` 转换后的对象。

    示例：`'query=string'` 或 `{'query':'string'}`

* `hash`：URL 中的信息片段部分，包括 # 符号。

    示例：`'#hash'`

### 转义字符

空格 (`' '`) 以及如下的字符在被解析为 URL 对象时会自动转义：

```
< > " ` \r \n \t { } | \ ^ '
```

---

以下是 URL 模块提供的方法：

## url.format(urlObj)

给定一个解析后的 URL 对象，返回一个经过格式化的 URL 字符串。

格式化进程是如何工作的：

* `href` 将被忽略。
* `path` 将被忽略。
* `protocol` 无论结尾是否有 `:`（冒号），都以相同的方式进行处理：
  * 协议 `http`, `https`, `ftp`, `gopher`, `file` 将添加后缀 `://`（冒号斜杠斜杠），只要 `host`/`hostname` 存在。
  * 其它的任何协议如 `mailto`, `xmpp`, `aim`, `sftp`, `foo` 将添加后缀 `:`（冒号）。
* `slashes` 如果协议要求 `://`  则设为 `true`。
  * 仅当协议不是上文所列出的默认加 `://` 的才需要手动设置该属性。例如 `mongodb://localhost:8000/`，或者当 `host`/`hostname` 缺失时。
* `auth` 如果存在会被使用。
* `hostname` 仅当 `host` 缺失时被使用。
* `port` 仅当 `host` 缺失时使用。
* `host` 将代替 `hostname` 和 `port` 被使用。
* `pathname` 无论开头是否有 `/`（斜杠）都会以相同的方式进行处理。
* `query` （对象，查看 `querystring`）仅当 `search` 缺失时被使用。
* `search` 将代替 `query` 被使用。
  * 无论开头是否有 `?`（问号）都会以相同的方式进行处理。
* `hash` 无论开头是否有 `#`（井号，锚点）都会以相同的方式进行处理。

## url.parse(urlStr[, parseQueryString][, slashesDenoteHost])

给定一个 URL 字符串，返回一个对象。

传入 `true` 作为第二个参数会使用 `querystring` 模块来解析查询字符串。如果传入 `true` 那么 `query` 属性将总是一个对象，`search` 属性将总是一个字符串（可能为空）。如果传入 `false` 那么 `query` 属性将不会被解析或解码。默认是 `false`。

传入 `true` 作为第三个参数会将 `//foo/bar` 解析成 `{ host: 'foo', pathname: '/bar' }` 而不是 `{ pathname: '//foo/bar' }`。默认是 `false`。

## url.resolve(from, to)

给定一个基础的 URL 和一个额外的链接，会像浏览器一样解析它们并可以带上锚点。示例：

```js
url.resolve('/one/two/three', 'four')         // '/one/two/four'
url.resolve('http://example.com/', '/one')    // 'http://example.com/one'
url.resolve('http://example.com/one', '/two') // 'http://example.com/two'
```
