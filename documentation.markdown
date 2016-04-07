# 关于本文档

<!-- type=misc -->

本文档的目标是从参考引用和概念性观点来全面地描述 Node.js 的 API。每个章节都描述了一个内置模块或高级概念。

该中文翻译文档不包含[官方api](https://nodejs.org/dist/latest-v4.x/docs/api/)的属性方法列表导航、html文件、json文件等特性。


如果你在文档中发现了错误，请 [提交问题][https://github.com/nodejs/node/issues/new]或者查看 [贡献指南][https://github.com/nodejs/node/blob/master/CONTRIBUTING.md] 来直接提交修补代码。

## 稳定性指数

<!--type=misc-->

贯穿整个文档，你将看到每个章节都会有稳定性指数。Node.js API 还在或多或少地改进中，成熟的部分会比其他章节更值得信赖。  一些久经考验的、被大量依赖的 API 几乎不会再改变。其它的一些新增的、实验性的、或者已知具有危险性的部分正在被重新设计中。

稳定性指数如下：

```
稳定性： 0 - 弃用
该部分功能存在已知问题，并已计划改变。不要使用它们，否则可能会引起警告。不要期待向后兼容了。
```

```
稳定性： 1 - 实验性
该部分功能可能发生变化，并且命令行标志是封闭的。在未来的版本中可能会发生修改或者被移除。
```

```
稳定性： 2 - 稳定
该 API 久经考验且让人满意。会优先和 npm 系统兼容，除非真的有必要否则不会变化。
```

```
稳定性： 3 - 已锁定
只有遇到安全、性能、bug才会接受修改。请不要提出更改建议，否则会被拒绝。
```

## 系统调用与操作说明

类似 open(2) 和 read(2) 的系统调用定义了用户程序和底层操作系统之间的接口。简单包装了一个系统调用的 Node 函数，类似 `fs.open()`，将会文档化。该文档将会链接到相关的操作说明（简称手册页），并描述该系统调用是如何工作的。

**警告：** 某些系统调用， 就像 lchown(2)， are 特定于 BSD 的。这意味着，像 `fs.lchown()` 仅仅工作在 Mac OS X 和其它 BSD 驱动的系统，在 Linux 上就不可用。

大多数 Unix 系统调用与 Windows 等价，但相对于 Linux 和 OS X，Windows 上的行为可能有所不同。看一个微妙的例子，有时候在 Windows 上不可能替代 Unix 的系统调用语义，查看 [Node issue 4760](https://github.com/nodejs/node/issues/4760)。

[提交问题]: https://github.com/nodejs/node/issues/new
[贡献指南]: https://github.com/nodejs/node/blob/master/CONTRIBUTING.md
