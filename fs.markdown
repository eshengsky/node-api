# 文件系统

    稳定性： 2 - 稳定

<!--name=fs-->

文件的 I/O 操作是由标准 POSIX 函数简单包装后提供的。通过 `require('fs')` 使用该模块。所有方法都同时有异步和同步模式。

异步方法的最后一个参数总是回调函数，代表操作完成。该回调函数的参数取决于具体方法，但第一个参数往往都是作为捕获到的异常，如果操作成功完成，没有发生异常，那么第一个参数将是 `null` 或 `undefined`。

当使用同步操作时，发生任何错误将会立即抛出异常，可以使用 try/catch 来处理异常或允许异常冒泡。

这是一个异步操作的例子：

```js
const fs = require('fs');

fs.unlink('/tmp/hello', (err) => {
  if (err) throw err;
  console.log('成功删除 /tmp/hello');
});
```

这是对应的同步操作的例子:

```js
const fs = require('fs');

fs.unlinkSync('/tmp/hello');
console.log('成功删除 /tmp/hello');
```

异步方法无法保证操作顺序，因此下面的代码很容易出错：

```js
fs.rename('/tmp/hello', '/tmp/world', (err) => {
  if (err) throw err;
  console.log('重命名成功');
});
fs.stat('/tmp/world', (err, stats) => {
  if (err) throw err;
  console.log(`stats: ${JSON.stringify(stats)}`);
});
```

有可能 `fs.stat` 在 `fs.rename` 重命名方法完成之前就先执行了，正确的方式应该是采用回调函数嵌套：

```js
fs.rename('/tmp/hello', '/tmp/world', (err) => {
  if (err) throw err;
  fs.stat('/tmp/world', (err, stats) => {
    if (err) throw err;
    console.log(`stats: ${JSON.stringify(stats)}`);
  });
});
```

在繁忙的进程中，我们 _强烈建议_ 使用异步模式的方法。同步方法会阻塞整个进程，直到方法完成。

你可能会用到文件的相对路径，记住，相对路径是对于 `process.cwd()` 而言的。

大部分文件操作的方法允许你省略回调函数，如果你这样做了，会用一个默认的回调函数来抛出异常。为了得到原始调用的堆栈信息，需要设置环境变量 `NODE_DEBUG`。

```
$ cat script.js
function bad() {
  require('fs').readFile('/');
}
bad();

$ env NODE_DEBUG=fs node script.js
fs.js:66
        throw err;
              ^
Error: EISDIR, read
    at rethrow (fs.js:61:21)
    at maybeCallback (fs.js:79:42)
    at Object.fs.readFile (fs.js:153:18)
    at bad (/path/to/script.js:2:17)
    at Object.<anonymous> (/path/to/script.js:5:1)
    <etc.>
```

## 类： fs.FSWatcher

从 `fs.watch()` 返回的对象类型。

### 事件： 'change'

* `event` {String} fs 改变的类型
* `filename` {String} 改变的文件名 (如果能明确得到文件名)

当监听的文件夹或文件发生变化时触发。更多信息请查看 [`fs.watch()`][]。

### 事件： 'error'

* `error` {Error}

当错误发生时触发。

### watcher.close()

停止监听 `fs.FSWatcher` 中的更改。

## 类： fs.ReadStream

`ReadStream` 是 [Readable Stream][] 类型。

### 事件： 'open'

* `fd` {Number} ReadStream 所使用的文件描述符（整数）。

当文件创建的 ReadStream 被打开时触发。

### readStream.path

该文件流所对应的当初读取的文件的路径。

## 类： fs.Stats

[`fs.stat()`][]， [`fs.lstat()`][]， [`fs.fstat()`][] 以及对应的同步方法的返回值类型。

 - `stats.isFile()`
 - `stats.isDirectory()`
 - `stats.isBlockDevice()`
 - `stats.isCharacterDevice()`
 - `stats.isSymbolicLink()` (只对 [`fs.lstat()`][] 有效)
 - `stats.isFIFO()`
 - `stats.isSocket()`

对一个普通的文件调用 `.stat()` 并将返回值传入 [`util.inspect(stats)`][] 会得到类似下面的字符串：

```js
{
  dev: 2114,
  ino: 48064969,
  mode: 33188,
  nlink: 1,
  uid: 85,
  gid: 100,
  rdev: 0,
  size: 527,
  blksize: 4096,
  blocks: 8,
  atime: Mon, 10 Oct 2011 23:24:11 GMT,
  mtime: Mon, 10 Oct 2011 23:24:11 GMT,
  ctime: Mon, 10 Oct 2011 23:24:11 GMT,
  birthtime: Mon, 10 Oct 2011 23:24:11 GMT
}
```

请注意 `atime`, `mtime`, `birthtime`, 以及 `ctime` 都是 [`Date`][MDN-Date] 的示例，你需要使用恰当的方法进行比较。通常使用 [`getTime()`][MDN-Date-getTime] 来获取从 UTC 时间 1970年1月1日 00:00:00 到现在的毫秒数，这个整数对于比较来说是足够用的。
也有一些额外的方法能够显示模糊信息，更多内容请查看 [MDN JavaScript Reference][MDN-Date] 页。

### 统计时间

统计对象（stat object）中的时间有如下语义：

* `atime` "Access Time" - 文件最后一次访问时间。可以通过 `mknod(2)`, `utimes(2)`, 和 `read(2)` 系统调用来改变。
* `mtime` "Modified Time" - 文件最后一次修改时间。可以通过 `mknod(2)`, `utimes(2)`, 和 `write(2)` 系统调用来改变。
* `ctime` "Change Time" - 文件最后一次变更时间（inode 数据修改)。可以通过 `chmod(2)`, `chown(2)`, `link(2)`, `mknod(2)`, `rename(2)`, `unlink(2)`, `utimes(2)`, `read(2)`, 和 `write(2)` 系统调用来改变。
* `birthtime` "Birth Time" -  文件创建时间。仅文件创建时生成一次。
 在无法获得 birthtime 的文件系统中，这个字段可能会使用 `ctime` or `1970-01-01T00:00Z` (即 unix 的纪元时间戳 `0`)。
 注意该情况下这个值可能会比 `atime` 或 `mtime` 大，在 Darwin 或其它变体中，也使用 `utimes(2)` 系统调用来明确地将 `atime` 设置成比它当前的 `birthtime` 更早的值。

在 Node v0.12 版本之前，Windows 系统下 `ctime` 会保留 `birthtime` 的值。注意在 v0.12 版本中， `ctime` 不是 "creation time（创建时间），"并且在 unix 系统中一直都不是。

## 类： fs.WriteStream

`WriteStream` 是 [Writable Stream][] 类型。

### 事件： 'open'

* `fd` {Number} WriteStream 所使用的文件描述符（整数）。

当文件创建的 WriteStream 被打开时触发。

### writeStream.bytesWritten

已经写入的字节数，不包含等待写入的数据。

### writeStream.path

该文件流写入的文件路径。

## fs.access(path[, mode], callback)

检测用户对 `path` 指向文件的用户权限。 `mode` 是一个可选参数，用来指定要检测的权限。下面列出了 `mode` 可能的值。
可以创建一个或多个值组成的掩码运算来检测权限。

- `fs.F_OK` - 文件对调用进程是可见的。对于检测文件是否存在这很有用，但 `rwx` 权限则什么也不返回。
当 `mode` 未指定时这是默认值。
- `fs.R_OK` - 文件对调用进程是可读的。
- `fs.W_OK` - 文件对调用进程是可写的。
- `fs.X_OK` - 文件可以被调用进程执行。 Windows 下无效(将等同于 `fs.F_OK`).

最后一个参数 `callback` 是回调函数，如果权限检测出错则会将错误传入。下面这个例子会检测当前进程对 `/etc/passwd` 是否有读写权限。

```js
fs.access('/etc/passwd', fs.R_OK | fs.W_OK, (err) => {
  console.log(err ? 'no access!' : 'can read/write');
});
```

## fs.accessSync(path[, mode])

[`fs.access()`][] 的同步版本。若权限检测失败则会抛出异常，否则什么都不做。

## fs.appendFile(file, data[, options], callback)

* `file` {String} 文件名
* `data` {String|Buffer}
* `options` {Object|String}
  * `encoding` {String|Null} 默认值 = `'utf8'`
  * `mode` {Number} 默认值 = `0o666`
  * `flag` {String} 默认值 = `'a'`
* `callback` {Function}

异步地给文件附加数据，如果文件不存在则创建一个。
`data` 可以是字符串或 buffer。

示例：

```js
fs.appendFile('message.txt', 'data to append', (err) => {
  if (err) throw err;
  console.log('"data to append" 已附加到该文件上！');
});
```

如果 `options` 是字符串，则代表编码类型。 示例：

```js
fs.appendFile('message.txt', 'data to append', 'utf8', callback);
```

## fs.appendFileSync(file, data[, options])

[`fs.appendFile()`][] 的同步版本。 返回 `undefined`。

## fs.chmod(path, mode, callback)

异步函数 chmod(2)。 回调函数的参数是可能出现的异常。

## fs.chmodSync(path, mode)

同步函数 chmod(2)。 返回 `undefined`。

## fs.chown(path, uid, gid, callback)

异步函数 chown(2)。 回调函数的参数是可能出现的异常。

## fs.chownSync(path, uid, gid)

同步函数 chown(2)。 返回 `undefined`。

## fs.close(fd, callback)

异步函数 close(2)。  回调函数的参数是可能出现的异常。

## fs.closeSync(fd)

同步函数 close(2)。 返回 `undefined`。

## fs.createReadStream(path[, options])

返回 [`ReadStream`][] 对象。（参见 [Readable Stream][]）。

请注意，与 `highWaterMark` 在可读流上设置的默认值（16 kb）不同，这个方法返回的流默认值是 64 kb。

`options` 是一个对象或者字符串，其默认值如下：

```js
{
  flags: 'r',
  encoding: null,
  fd: null,
  mode: 0o666,
  autoClose: true
}
```

`options` 可以包含 `start` 和 `end` 属性来读取文件的指定范围，而非整个文件。 `start` 和 `end` 要都在文件范围内，且从 0 开始，
 `encoding` 可以是 [`Buffer`][] 认可的任意编码类型。

如果指定了 `fd` 的值， `ReadStream` 将会忽略 `path` 参数而使用指定的文件描述符。 这意味着不会触发任何 `'open'` 事件。
注意 `fd` 会被阻塞；非阻塞的 `fd` 应该传递给 [`net.Socket`][]。

如果 `autoClose` 设为 false，即使发生错误文件也不会关闭，这就需要由你自己负责关闭，并确保没有文件泄露。如果 `autoClose` 设为 true （默认值），当遇到 `error` 或者 `end` 文件将会被自动关闭。

`mode` 用来设置文件模式（权限和 sticky 位），但前提是文件已被创建。

这个例子会读取一个大小为 100 字节的文件的末尾 10 个字节：

```js
fs.createReadStream('sample.txt', {start: 90, end: 99});
```

如果 `options` 是字符串，则代表文件的编码。

## fs.createWriteStream(path[, options])

返回 [`WriteStream`][] 对象。（参见 [Writable Stream][]）。

`options` 是一个对象或者字符串，其默认值如下：

```js
{
  flags: 'w',
  defaultEncoding: 'utf8',
  fd: null,
  mode: 0o666
}
```

`options` 可以包含 `start` 属性来从指定的位置开始写入文件。 如果想修改文件而不是替换文件，需要将 `flags` 的模式指定为 `r+` 而不是默认的 `w`。 `defaultEncoding` 可以是 [`Buffer`][] 认可的任意编码类型。

和 [`ReadStream`][] 类似，如果指定了 `fd` 的值， `WriteStream` 将会忽略 `path` 参数而使用指定的文件描述符。这意味着不会触发任何 `'open'` 事件。
注意 `fd` 会被阻塞；非阻塞的 `fd` 应该传递给 [`net.Socket`][]。

如果 `options` 是字符串，则代表文件的编码。

## fs.exists(path, callback)

    稳定性： 0 - 弃用：请使用 [`fs.stat()`][] 或 [`fs.access()`][]。

通过检查文件系统判断给定的文件是否存在，通过回调函数 `callback` 的参数给出 true 或 false。  示例：

```js
fs.exists('/etc/passwd', (exists) => {
  console.log(exists ? 'it\'s there' : 'no passwd!');
});
```

`fs.exists()` 不应该在调用 `fs.open()` 之前检查文件是否存在，因为在2个方法的调用间隙其它进程有可能改变了文件状态，从而引发操作风险。
因此，你应该直接调用 `fs.open()`，并根据回调函数是否有错来判断文件是否存在。

## fs.existsSync(path)

    稳定性： 0 - 弃用： 请使用 [`fs.statSync()`][] 或 [`fs.accessSync()`][]。

[`fs.exists()`][] 的同步版本。
如果文件存在返回 `true`，否则返回 `false`。

## fs.fchmod(fd, mode, callback)

异步函数 fchmod(2) 。回调函数的参数是可能出现的异常。

## fs.fchmodSync(fd, mode)

同步函数 fchmod(2)。返回 `undefined`。

## fs.fchown(fd, uid, gid, callback)

异步函数 fchown(2) 。回调函数的参数是可能出现的异常。

## fs.fchownSync(fd, uid, gid)

同步函数 fchown(2)。返回 `undefined`。

## fs.fdatasync(fd, callback)

异步函数 fdatasync(2) 。回调函数的参数是可能出现的异常。

## fs.fdatasyncSync(fd)

同步函数 fdatasync(2)。返回 `undefined`。

## fs.fstat(fd, callback)

异步函数 fstat(2)。回调函数有2个参数 `(err, stats)`，`stats` 是 `fs.Stats` 对象。除了文件是使用文件描述符 `fd` 指定的这一区别外，其它都与 [`stat()`][] 一样。

## fs.fstatSync(fd)

同步函数 fstat(2)。返回 `fs.Stats` 实例。

## fs.fsync(fd, callback)

异步函数 fsync(2)。回调函数的参数是可能出现的异常。

## fs.fsyncSync(fd)

同步函数 fsync(2)。返回 `undefined`。

## fs.ftruncate(fd, len, callback)

异步函数 ftruncate(2)。回调函数的参数是可能出现的异常。

## fs.ftruncateSync(fd, len)

同步函数 ftruncate(2) 。返回 `undefined`。

## fs.futimes(fd, atime, mtime, callback)

修改传入的文件描述符所指向文件的时间戳。

## fs.futimesSync(fd, atime, mtime)

[`fs.futimes()`][] 的同步版本。 返回 `undefined`。

## fs.lchmod(path, mode, callback)

异步函数 lchmod(2)。回调函数的参数是可能出现的异常。

仅 Mac OS X 可用。

## fs.lchmodSync(path, mode)

同步函数 lchmod(2)。返回 `undefined`。

## fs.lchown(path, uid, gid, callback)

异步函数 lchown(2)。回调函数的参数是可能出现的异常。

## fs.lchownSync(path, uid, gid)

同步函数 lchown(2)。返回 `undefined`。

## fs.link(srcpath, dstpath, callback)

异步函数 link(2)。回调函数的参数是可能出现的异常。

## fs.linkSync(srcpath, dstpath)

同步函数 link(2)。返回 `undefined`。

## fs.lstat(path, callback)

异步函数 lstat(2)。回调函数有2个参数 `(err, stats)`，`stats` 是 `fs.Stats` 对象。如果 `path` 是符号链接，读取的是链接本身而非它所链接到的文件。除吃之外，其它都与 `stat()` 一样。

## fs.lstatSync(path)

同步函数 lstat(2)。返回 `fs.Stats` 的实例。

## fs.mkdir(path[, mode], callback)

异步函数 mkdir(2)。回调函数的参数是可能出现的异常。 `mode` 默认值 `0o777`。

## fs.mkdirSync(path[, mode])

异步函数 mkdir(2)。返回 `undefined`。

## fs.open(path, flags[, mode], callback)

异步打开文件。参见 open(2)。 `flags` 可能的值：

* `'r'` - 以只读模式打开文件。
如果文件不存在则抛出异常。

* `'r+'` - 以读写模式打开文件。
如果文件不存在则抛出异常。

* `'rs'` - 以同步只读模式打开文件。命令操作系统忽略本地文件系统缓存。这个功能主要用于打开 NFS 上的文件，它允许你跳过可能过去的本地缓存。这对 I/O 性能有真实的影响，如果你并不需要请不要使用它。

注意这并没有将 `fs.open()` 变成一个同步阻塞的调用。如果你想要同步请求你应该使用 `fs.openSync()`。

* `'rs+'` - 以同步读写模式打开文件。注意事项请查看 `'rs'`。

* `'w'` - 以只写模式打开文件。如果文件不存在则会自动创建，如果文件存在则会覆盖。

* `'wx'` - 类似 `'w'`，但如果 `path` 存在则会失败。

* `'w+'` - 以读写模式打开文件。如果文件不存在则会自动创建，如果文件存在则会覆盖。

* `'wx+'` - 类似 `'w+'`，但如果 `path` 存在则会失败。

* `'a'` - 以追加模式打开文件。如果文件不存在则会自动创建。

* `'ax'` - 类似 `'a'`，但如果 `path` 存在则会失败。

* `'a+'` - 以只读追加模式打开文件。如果文件不存在则会自动创建。

* `'ax+'` - 类似 `'a+'`，但如果 `path` 存在则会失败。

`mode` 用来设置文件模式（权限和 sticky 位），但前提是文件已被创建。默认值是 `0666`，可读写。

回调函数有2个参数 `(err, fd)`。

排除标记 `'x'` (open(2) 中的 `O_EXCL` 标记) 保证了 `path` 是新创建的。在 POSIX 系统中，即使符号链接指向了一个不存在的文件，`path` 也被认为是存在的。排除标记在网络文件系统中作用情况无法确定。

`flags` 也可以是 open(2) 给的数字。常用的常数可以通过 `require('constants')` 获得。在 Windows 操作系统下，flag 会被转换成一个对应值，例如： `O_WRONLY` 转换成 `FILE_GENERIC_WRITE`，
 `O_EXCL|O_CREAT` 转换成 `CREATE_NEW`，作为 CreateFileW 可以接收的值。

在 Linux 系统下，以追加模式打开的文件不支持指定位置写入。系统内核会忽略位置参数，而总是将数据写到文件的末尾。

## fs.openSync(path, flags[, mode])

[`fs.open()`][] 的异步版本。返回一个表示文件描述符的整数。

## fs.read(fd, buffer, offset, length, position, callback)

读取 `fd` 指定文件的数据。

`buffer` 数据将会写入的缓冲区。

`offset` 开始写入时缓冲区起始位置的偏移量。

`length` 要读取的文件的长度。

`position` 读取文件时的起始位置，如果 `position` 是 `null`，将会从当前的文件位置读起。

回调函数有3个参数 `(err, bytesRead, buffer)`。

## fs.readdir(path, callback)

异步函数 readdir(3)。读取文件夹的内容。
回调函数有2个参数 `(err, files)` ， `files` 是文件夹内所有文件的文件名的数组，但排除了文件名包含 `'.'` 和 `'..'` 的文件。

## fs.readdirSync(path)

同步函数 readdir(3)。返回一个排除了 `'.'` 和 `'..'` 的文件名的数组。

## fs.readFile(file[, options], callback)

* `file` {String} 文件名
* `options` {Object | String}
  * `encoding` {String | Null} 默认值 = `null`
  * `flag` {String} 默认值 = `'r'`
* `callback` {Function}

异步读取整个文件的内容。 示例：

```js
fs.readFile('/etc/passwd', (err, data) => {
  if (err) throw err;
  console.log(data);
});
```

回调函数传递了2个参数 `(err, data)`， `data` 是文件的内容。

如果没有指定文件编码，则返回一个原生 buffer。

如果 `options` 是一个字符串，则代表文件编码。 示例：

```js
fs.readFile('/etc/passwd', 'utf8', callback);
```

## fs.readFileSync(file[, options])

[`fs.readFile`][] 的同步版本。返回文件 `file` 的内容。

如果指定了 `encoding` 则返回一个字符串，否则返回一个 buffer。

## fs.readlink(path, callback)

异步函数 readlink(2)。回调函数有2个参数 `(err,
linkString)`。

## fs.readlinkSync(path)

同步函数 readlink(2)。返回符号链接的字符串值。

## fs.realpath(path[, cache], callback)

异步函数 realpath(2)。 回调函数 `callback` 有2个参数 `(err, resolvedPath)`。可以使用 `process.cwd` 来解决相对路径问题。
`cache` 是一个映射路径的对象字面量，可以用来强制指定路径方案或者避免对已知的真实路径调用 `fs.stat` 。

示例：

```js
var cache = {'/etc':'/private/etc'};
fs.realpath('/etc/passwd', cache, (err, resolvedPath) => {
  if (err) throw err;
  console.log(resolvedPath);
});
```

## fs.readSync(fd, buffer, offset, length, position)

[`fs.read()`][] 的同步版本。返回 `bytesRead` 的数值。

## fs.realpathSync(path[, cache])

同步函数 realpath(2)。返回解析后的路径。`cache` 是一个映射路径的对象字面量，可以用来强制指定路径方案或者避免对已知的真实路径调用 `fs.stat` 。

## fs.rename(oldPath, newPath, callback)

异步函数 rename(2)。回调函数的参数是可能出现的异常。

## fs.renameSync(oldPath, newPath)

同步函数 rename(2)。返回 `undefined`。

## fs.rmdir(path, callback)

异步函数 rmdir(2)。回调函数的参数是可能出现的异常。

## fs.rmdirSync(path)

同步函数 rmdir(2)。返回 `undefined`。

## fs.stat(path, callback)

异步函数 stat(2)。回调函数有2个参数 `(err, stats)` ，
`stats` 是 [`fs.Stats`][] 对象。更多内容参见 [`fs.Stats`][] 。

## fs.statSync(path)

同步函数 stat(2)。返回 [`fs.Stats`][] 的实例。

## fs.symlink(target, path[, type], callback)

异步函数 symlink(2)。回调函数的参数是可能出现的异常。
参数 `type` 可以设置为 `'dir'`， `'file'`， 或者 `'junction'` （默认值为 `'file'`），且仅在 Windows 下有效。
请注意 Windows 交接点（junction points）需要目标路径是绝对路径。当使用 `'junction'`，参数 `target` 会被自动转换成绝对路径。

这是一个示例：

```js
fs.symlink('./foo', './new-port');
```

这会创建一个名为 "new-port" 的符号链接并指向 "foo"。

## fs.symlinkSync(target, path[, type])

同步函数 symlink(2)。返回 `undefined`。

## fs.truncate(path, len, callback)

异步函数 truncate(2)。回调函数的参数是可能出现的异常。文件描述符也可以用作第一个参数，这种情况下，会调用 `fs.ftruncate()` 。

## fs.truncateSync(path, len)

同步函数 truncate(2)。返回 `undefined`。

## fs.unlink(path, callback)

异步函数 unlink(2)。回调函数的参数是可能出现的异常。

## fs.unlinkSync(path)

同步函数 unlink(2)。返回 `undefined`。

## fs.unwatchFile(filename[, listener])

停止监视 `filename` 文件的变化。 如果指定了 `listener` ，只会移除该 `listener`，否则 *所有* listeners 都会被移除并停止监视 `filename`。

调用 `fs.unwatchFile()` 如果传入的 `filename` 没有被监视，则属于一个空操作，不会导致错误。

_提示： [`fs.watch()`][] 比 `fs.watchFile()` 和 `fs.unwatchFile()` 更加高效。
应该尽可能使用 `fs.watch()` 来替代 `fs.watchFile()` 和 `fs.unwatchFile()` 。_

## fs.utimes(path, atime, mtime, callback)

改变指定路径的文件的时间戳。

说明：参数 `atime` 和 `mtime` 的相关功能遵循如下规则：

- 如果值是一个数字型的字符串如 `'123456789'`，则值会被转换成对应的数字。
- 如果值是 `NaN` （Not a Number，非数字） 或者 `Infinity` （无限大）, 值会被转换成 `Date.now()`。

## fs.utimesSync(path, atime, mtime)

[`fs.utimes()`][] 的同步版本。返回 `undefined`。

## fs.watch(filename[, options][, listener])

监视 `filename` 的改变，指定的 `filename` 可以是文件或文件夹。返回的对象类型是 [`fs.FSWatcher`][]。

第2个参数 `options` 是可选的，且必须是一个对象。它有2个 boolean 类型的属性 `persistent` 和 `recursive`。 `persistent` 指定当文件被监视时进程是否继续运行。 `recursive` 指定是否要监视所有的子文件夹，还是仅仅监视当前文件夹。这个参数仅对文件夹及支持的系统中有效（参见 [注意事项][]）。

默认值是 `{ persistent: true, recursive: false }`。

回调函数有2个参数 `(event, filename)`，  `event` 是 `'rename'` 或者 `'change'`， `filename` 是触发了事件的文件名。

### 注意事项

<!--type=misc-->

`fs.watch` API 并非 100% 跨平台兼容，且在某些情况下不可用。

`recursive` 参数仅在 OS X 和 Windows 下有效。

#### 可用性

<!--type=misc-->

该功能依赖于底层操作系统提供文件系统变动通知。

* 在 Linux 系统，会使用 `inotify`。
* 在 BSD 系统， 会使用 `kqueue`。
* 在 OS X系统， 对文件会使用 `kqueue` ，对文件夹会使用 'FSEvents' 。
* 在 SunOS 系统，（包括 Solaris 和 SmartOS），会使用 `event ports`。
* 在 Windows 系统， 依赖于 `ReadDirectoryChangesW`。

如果底层系统函数因某些原因不可用，那么 `fs.watch` 将无法正常工作。例如，监视网络文件系统（NFS， SMB等）中文件或文件夹的变动往往是不可靠的。

你仍然可以使用 `fs.watchFile`，它使用了统计调查，但速度很慢且可靠性不佳。

#### 文件名参数

<!--type=misc-->

回调函数仅在 Linux 和 Windows 平台下才会提供 `filename` 参数。即使在支持的平台， `filename` 也不能完全确保提供。因此，不要去假设 `filename` 参数一定会有，需要有 `filename` 为 null 的逻辑判断。

```js
fs.watch('somedir', (event, filename) => {
  console.log(`event is: ${event}`);
  if (filename) {
    console.log(`提供了文件名：${filename}`);
  } else {
    console.log('未提供文件名');
  }
});
```

## fs.watchFile(filename[, options], listener)

监视 `filename` 文件的变化。文件每次被访问都会调用 `listener` 。

参数 `options` 是可选的。如果提供了必须是一个对象。`options` 有一个 boolean 类型的属性 `persistent` ，指定当文件被监视时进程是否继续运行。 `options` 对象还可以指定一个 `interval` 属性，代表目标文件被检测的频率，单位是毫秒。`options` 的默认值是 `{ persistent: true, interval: 5007 }`。

`listener` 有两个参数，第一个是当前的文件统计对象，第二个是上一次的统计对象。

```js
fs.watchFile('message.text', (curr, prev) => {
  console.log(`当前的 mtime 是：${curr.mtime}`);
  console.log(`上一次的 mtime 是：${prev.mtime}`);
});
```

这些统计对象都是 `fs.Stat` 类型。

如果想在文件被修改时得到通知，而不是被访问时就通知，那么就需要比较 `curr.mtime` 和 `prev.mtime`。

_注意：当一个 `fs.watchFile` 操作引起了 `ENOENT` 错误，会调用 `listener` 一次，所有字段都为 0 （对于日期就是 Unix
 Epoch）。在 Windows 下， `blksize` 和 `blocks` 字段将是 `undefined`而非 0. 如果文件在后来被创建，listener 会再次被调用，并传入最新的统计对象。这是自 v0.10 开始的功能变化。_

_提示： [`fs.watch()`][] 比 `fs.watchFile` 和 `fs.unwatchFile` 更加高效。应该尽可能使用
`fs.watch` 来替代 `fs.watchFile` 和 `fs.unwatchFile` 。_

## fs.write(fd, buffer, offset, length[, position], callback)

将 `buffer` 写入 `fd` 指定的文件中。

`offset` 和 `length` 用来确定要写入哪部分的缓冲区。

`position` 指文件写入的起始位置，如果 `typeof position !== 'number'`（不是一个数字），则会从当前位置开始写入。参见 pwrite(2)。

回调函数有3个参数 `(err, written, buffer)` ，`written` 表示 `buffer` 中已经写入了多少_字节_。

注意，在回调函数还没执行前就对同一个文件多次使用 `fs.write` 是不安全的。对这种场景，强烈推荐使用 `fs.createWriteStream` 。

在 Linux 系统下，以追加模式打开的文件不支持指定位置写入。系统内核会忽略位置参数，而总是将数据写到文件的末尾。

## fs.write(fd, data[, position[, encoding]], callback)

将 `data` 写入 `fd` 指定文件中。如果 `data` 不是 Buffer 实例，会被强制转为字符串。

`position` 指文件写入的起始位置，如果 `typeof position !== 'number'`（不是一个数字），则会从当前位置开始写入。参见 pwrite(2)。

`encoding` 预期的字符串编码。

回调函数有3个参数 `(err, written, string)` ， `written` 表示已经写入了多少_字节_。注意字节与字符是不同的，参见 [`Buffer.byteLength`][]。

与写入 `buffer` 不同，字符串只能整个写入，不支持截取部分写入。这是因为返回的字节的偏移量和字符串的偏移量是不同的。

注意，在回调函数还没执行前就对同一个文件多次使用 `fs.write` 是不安全的。对这种场景，强烈推荐使用 `fs.createWriteStream` 。

在 Linux 系统下，以追加模式打开的文件不支持指定位置写入。系统内核会忽略位置参数，而总是将数据写到文件的末尾。

## fs.writeFile(file, data[, options], callback)

* `file` {String} 文件名
* `data` {String | Buffer}
* `options` {Object | String}
  * `encoding` {String | Null} 默认值 = `'utf8'`
  * `mode` {Number} 默认值 = `0o666`
  * `flag` {String} 默认值 = `'w'`
* `callback` {Function}

异步地给文件写入数据，如果文件已存在则会覆盖。
`data` 可以是字符串或缓冲区。

 如果 `data` 是缓冲区，则 `encoding` 选项会被忽略。`encoding`  的默认值是 `'utf8'`。

示例：

```js
fs.writeFile('message.txt', 'Hello Node.js', (err) => {
  if (err) throw err;
  console.log('It\'s saved!');
});
```

如果 `options` 是字符串，则表示编码格式。 示例：

```js
fs.writeFile('message.txt', 'Hello Node.js', 'utf8', callback);
```

注意，在回调函数还没执行前就对同一个文件多次使用 `fs.writeFile` 是不安全的。对这种场景，强烈推荐使用 `fs.createWriteStream` 。

## fs.writeFileSync(file, data[, options])

[`fs.writeFile()`][] 的同步版本。返回 `undefined`。

## fs.writeSync(fd, buffer, offset, length[, position])

## fs.writeSync(fd, data[, position[, encoding]])

[`fs.write()`][] 的同步版本。返回写入的字节数。

[`Buffer.byteLength`]: buffer.markdown#buffer_class_method_buffer_bytelength_string_encoding
[`Buffer`]: buffer.markdown#buffer_buffer
[注意事项]: #注意事项
[`fs.access()`]: #fsaccesspath-mode-callback
[`fs.accessSync()`]: #fsaccesssyncpath-mode
[`fs.appendFile()`]: #fsappendfilefile-data-options-callback
[`fs.exists()`]: #fsexistspath-callback
[`fs.fstat()`]: #fsfstatfd-callback
[`fs.FSWatcher`]: #类-fsfswatcher
[`fs.futimes()`]: #fsfutimesfd-atime-mtime-callback
[`fs.lstat()`]: #fslstatpath-callback
[`fs.open()`]: #fsopenpath-flags-mode-callback
[`fs.read()`]: #fsreadfd-buffer-offset-length-position-callback
[`fs.readFile`]: #fsreadfilefile-options-callback
[`fs.stat()`]: #fsstatpath-callback
[`fs.Stats`]: #类-fsstats
[`fs.statSync()`]: #fsstatsyncpath
[`fs.utimes()`]: #fsfutimesfd-atime-mtime-callback
[`fs.watch()`]: #fswatchfilename-options-listener
[`fs.write()`]: #fswritefd-buffer-offset-length-position-callback
[`fs.writeFile()`]: #fswritefilefile-data-options-callback
[`net.Socket`]: net.markdown#net_class_net_socket
[`ReadStream`]: #类-fsreadstream
[`stat()`]: #fsstatpath-callback
[`util.inspect(stats)`]: util.markdown#util_util_inspect_object_options
[`WriteStream`]: #类-fswritestream
[MDN-Date-getTime]: https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/Date/getTime
[MDN-Date]: https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/Date
[Readable Stream]: stream.markdown#stream_class_stream_readable
[Writable Stream]: stream.markdown#stream_class_stream_writable
