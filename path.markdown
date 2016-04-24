## 目录
* [路径](#路径)
  * [path.basename(p[, ext])](#pathbasenamep-ext)
  * [path.delimiter](#pathdelimiter)
  * [path.dirname(p)](#pathdirnamep)
  * [path.extname(p)](#pathextnamep)
  * [path.format(pathObject)](#pathformatpathobject)
  * [path.isAbsolute(path)](#pathisabsolutepath)
  * [path.join([path1][, path2][, ...])](#pathjoinpath1-path2-)
  * [path.normalize(p)](#pathnormalizep)
  * [path.parse(pathString)](#pathparsepathstring)
  * [path.posix](#pathposix)
  * [path.relative(from, to)](#pathrelativefrom-to)
  * [path.resolve([from ...], to)](#pathresolvefrom--to)
  * [path.sep](#pathsep)
  * [path.win32](#pathwin32)

# 路径

    稳定性： 2 - 稳定

该模块包含处理和转换文件路径的工具。几乎所有这些方法仅执行字符串转换。文件系统并不会去检查路径是否有效。

通过 `require('path')` 来使用该模块。提供以下方法：

## path.basename(p[, ext])

返回路径的最后一个部分。类似于Unix `basename` 命令。

示例：

```js
path.basename('/foo/bar/baz/asdf/quux.html')
// 返回 'quux.html'

path.basename('/foo/bar/baz/asdf/quux.html', '.html')
// 返回 'quux'
```

## path.delimiter

特定平台上路径间的分隔符，`;` 或 `':'`。

\*nix 平台的一个例子：

```js
console.log(process.env.PATH)
// '/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/bin'

process.env.PATH.split(path.delimiter)
// 返回 ['/usr/bin', '/bin', '/usr/sbin', '/sbin', '/usr/local/bin']
```

Windows 平台的一个例子：

```js
console.log(process.env.PATH)
// 'C:\Windows\system32;C:\Windows;C:\Program Files\node\'

process.env.PATH.split(path.delimiter)
// 返回 ['C:\\Windows\\system32', 'C:\\Windows', 'C:\\Program Files\\node\\']
```

## path.dirname(p)

返回路径所在的目录名。类似于 Unix `dirname` 命令。

示例：

```js
path.dirname('/foo/bar/baz/asdf/quux')
// 返回 '/foo/bar/baz/asdf'
```

## path.extname(p)

返回路径的扩展名，从路径最后一个部分的最后的 '.' 到路径字符串的结尾。如果路径最后一个部分没有 '.' 或者路径最后一个部分是以 '.' 开头，则会返回一个空字符串。示例：

```js
path.extname('index.html')
// 返回 '.html'

path.extname('index.coffee.md')
// 返回 '.md'

path.extname('index.')
// 返回 '.'

path.extname('index')
// 返回 ''

path.extname('.index')
// 返回 ''
```

## path.format(pathObject)

从一个对象中返回一个路径字符串。与 [`path.parse`][] 正好相反。

如果 `pathObject` 包含了 `dir` 和 `base` 属性，返回的字符串将是 `dir` 属性、平台相关的路径分隔符以及 `base` 属性连接起来的字符串。

如果没有 `dir` 属性，`root` 属性将被视为 `dir` 属性使用。然而，这将假定 `root` 属性已经以平台相关的路径分隔符结束了。在这种情况下，返回的字符串将是 `root` 属性和 `base` 属性连接起来的字符串。

如果 `dir` 和 `root` 属性都不存在，那么返回的字符串将是 `base` 属性的内容。

如果 `base` 属性不存在，那么会将 `name` 属性和 `ext` 属性连接起来作为 `base` 属性。

示例：

Posix 系统的一个例子：

```js
path.format({
    root : "/",
    dir : "/home/user/dir",
    base : "file.txt",
    ext : ".txt",
    name : "file"
});
// 返回 '/home/user/dir/file.txt'

// 如果未指定 `dir` 则使用 `root`，如果未指定 `base` 则使用 `name` + `ext` 。
path.format({
    root : "/",
    ext : ".txt",
    name : "file"
})
// returns '/file.txt'
```

Windows 系统的一个例子：

```js
path.format({
    root : "C:\\",
    dir : "C:\\path\\dir",
    base : "file.txt",
    ext : ".txt",
    name : "file"
})
// 返回 'C:\\path\\dir\\file.txt'
```

## path.isAbsolute(path)

判断 `path` 是否是一个绝对路径。绝对路径将总是解析为相同的位置，与当前工作目录无关。

Posix 示例：

```js
path.isAbsolute('/foo/bar') // true
path.isAbsolute('/baz/..')  // true
path.isAbsolute('qux/')     // false
path.isAbsolute('.')        // false
```

Windows 示例：

```js
path.isAbsolute('//server')  // true
path.isAbsolute('C:/foo/..') // true
path.isAbsolute('bar\\baz')  // false
path.isAbsolute('.')         // false
```

*注意：* 如果路径字符串传入了一个长度为 0 的字符串，与其他路径模块相关方法不同的是，这将按原样使用并返回 `false` 。

## path.join([path1][, path2][, ...])

连接所有参数并规范化结果路径。

参数必须都是字符串。在 v0.8 版本，非字符串的参数被静默忽略，在 v0.10 及以上版本，会抛出一个异常。

示例：

```js
path.join('/foo', 'bar', 'baz/asdf', 'quux', '..')
// 返回 '/foo/bar/baz/asdf'

path.join('foo', {}, 'bar')
// 抛出异常
TypeError: Arguments to path.join must be strings
```

*注意：* 如果传入 `join` 的参数有 0 长度的字符串，与其他路径模块相关方法不同的是，这些 0 长度的字符串将被忽略。如果如果连接后的路径字符串是一个 0 长度的字符串，则会返回 `'.'`，代表当前的工作目录。

## path.normalize(p)

规范化一个字符串路径，会关注 `'..'` 和 `'.'` 部分。

当发现多个斜杠时，会替换成一个斜杠；当路径结尾包含斜杠时，斜杠会被保留。Windows 平台下会使用反斜杠。

示例：

```js
path.normalize('/foo/bar//baz/asdf/quux/..')
// 返回 '/foo/bar/baz/asdf'
```

*注意：* 如果传入的路径字符串时一个 0 长度的字符串，则会返回 `'.'`，代表当前的工作目录。

## path.parse(pathString)

返回由路径字符串解析后的一个对象。

\*nix 平台的一个例子：

```js
path.parse('/home/user/dir/file.txt')
// 返回：
// {
//    root : "/",
//    dir : "/home/user/dir",
//    base : "file.txt",
//    ext : ".txt",
//    name : "file"
// }
```

Windows 平台的一个例子：

```js
path.parse('C:\\path\\dir\\index.html')
// 返回：
// {
//    root : "C:\\",
//    dir : "C:\\path\\dir",
//    base : "index.html",
//    ext : ".html",
//    name : "index"
// }
```

## path.posix

提供上述 `path` 方法的访问，但总是以 posix 兼容的方式交互。

## path.relative(from, to)

解析从 `from` 到 `to` 的相对路径。

有时我们会有2个绝对路径，我们需要获得从一个路径到另一个路径的相对路径。这实际上是 `path.resolve` 的逆实现，这意味着我们可以看到：

```js
path.resolve(from, path.relative(from, to)) == path.resolve(to)
```

示例：

```js
path.relative('C:\\orandea\\test\\aaa', 'C:\\orandea\\impl\\bbb')
// 返回 '..\\..\\impl\\bbb'

path.relative('/data/orandea/test/aaa', '/data/orandea/impl/bbb')
// 返回 '../../impl/bbb'
```

*注意：* 如果传入 `relative` 的参数包括 0 长度的字符串，则会使用当前工作目录代替。如果 2 个参数路径一样，则会返回一个 0 长度字符串。

## path.resolve([from ...], to)

将 `to` 解析成一个绝对路径。

如果 `to` 不是绝对路径，会将 `from` 参数按照从右往左的次序依次添加到 `to` 的左边，知道找到一个绝对路径为止。如果使用所有的 `from` 路径后还是没找到绝对路径，将会使用当前工作目录。结果路径已经被规范化，并且除了根目录外结尾的斜杠也被移除。非字符串的 `from` 参数会被忽略。

另一种理解的思路是在 shell 中执行一系列的 `cd` 命令。

```js
path.resolve('foo/bar', '/tmp/file/', '..', 'a/../subfile')
```

类似于：

```
cd foo/bar
cd /tmp/file/
cd ..
cd a/../subfile
pwd
```

差异之处在于 `resolve` 参数的路径不需要真实存在，并且还可以是文件。

示例：

```js
path.resolve('/foo/bar', './baz')
// 返回 '/foo/bar/baz'

path.resolve('/foo/bar', '/tmp/file/')
// 返回 '/tmp/file'

path.resolve('wwwroot', 'static_files/png/', '../gif/image.gif')
// 如果当前工作目录是 /home/myself/node，则会返回
// '/home/myself/node/wwwroot/static_files/gif/image.gif'
```

*注意：* 如果传入 `resolve` 的参数包括 0 长度的字符串，则会使用当前工作目录代替。

## path.sep

特定平台的文件分隔符，`'\\'` 或 `'/'`。

\*nix 平台的一个例子：

```js
'foo/bar/baz'.split(path.sep)
// 返回 ['foo', 'bar', 'baz']
```

Windows 平台的一个例子：

```js
'foo\\bar\\baz'.split(path.sep)
// 返回 ['foo', 'bar', 'baz']
```

## path.win32

提供上述 `path` 方法的访问，但总是以 win32 兼容的方式交互。

[`path.parse`]: #pathparsepathstring
