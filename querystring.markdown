## 目录
* [查询字符串](#查询字符串)
  * [querystring.escape](#querystringescape)
  * [querystring.parse(str[, sep][, eq][, options])](#querystringparsestr-sep-eq-options)
  * [querystring.stringify(obj[, sep][, eq][, options])](#querystringstringifyobj-sep-eq-options)
  * [querystring.unescape](#querystringunescape)

# 查询字符串

    稳定性： 2 - 稳定

<!--name=querystring-->

该模块提供处理查询字符串的工具，有如下方法：

## querystring.escape

`querystring.stringify` 所使用的转义函数，必要时可被重写。

## querystring.parse(str[, sep][, eq][, options])

将查询字符串反序列化为一个对象。可选择重写默认的分隔符（`'&'`）和分配符（`'='`）。

Options 对象可以包含 `maxKeys` 属性（默认值为 1000），它用来限制处理键的数量，设为 0 可以去除该限制。

Options 对象还可以包含 `decodeURIComponent` 属性（默认值为 `querystring.unescape`），如果需要可以用来解码 `non-utf8` 编码的字符串。

示例：

```js
querystring.parse('foo=bar&baz=qux&baz=quux&corge')
// 返回 { foo: 'bar', baz: ['qux', 'quux'], corge: '' }

// 假定 gbkDecodeURIComponent 函数已经存在，这将会解码一个 `gbk` 编码的字符串。
querystring.parse('w=%D6%D0%CE%C4&foo=bar', null, null, { decodeURIComponent: gbkDecodeURIComponent })
// 返回 { w: '中文', foo: 'bar' }
```

## querystring.stringify(obj[, sep][, eq][, options])

序列号一个对象到查询字符串。可选择重写默认的分隔符（`'&'`）和分配符（`'='`）。

Options 对象可以包含 `encodeURIComponent` 属性（默认值为 `querystring.escape`），如果需要可以用来编码字符串为 `non-utf8` 。

示例：

```js
querystring.stringify({ foo: 'bar', baz: ['qux', 'quux'], corge: '' })
// 返回 'foo=bar&baz=qux&baz=quux&corge='

querystring.stringify({foo: 'bar', baz: 'qux'}, ';', ':')
// 返回 'foo:bar;baz:qux'

// 假定 gbkEncodeURIComponent 函数已经存在，这会将字符串编码为 `gbk`。
querystring.stringify({ w: '中文', foo: 'bar' }, null, null, { encodeURIComponent: gbkEncodeURIComponent })
// 返回 'w=%D6%D0%CE%C4&foo=bar'
```

## querystring.unescape

`querystring.parse` 使用的反转义函数，必要时可被重写。

它将首先尝试使用 `decodeURIComponent` ，如果失败则回滚到一个更安全的等效值，而不会抛出一个错误格式的 URL。
