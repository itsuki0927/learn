# git 对象模型

## SHA

表示项目历史所需的所有信息都存储在由 40 位"对象名称"引用的文件中，该文件如下所示：

```
6ff87c4664981e4397625791c8ea3bbb5f2279a3
```

Git 中到处可以看到 40 为长度的字符串. 每种情况, 名称都是通过获取对象内容的 SHA1 哈希值来计算的.
SHA1 是一种散列加密函数. 这就意味着几乎不可能找到两个相同名称的不同对象. 这也有优点:

- Git 可以通过比较名称来快速判断两个对象是否相同.
- 由于对象的名字在每个仓库中都是以相同的方式计算的, 因此存储在不同仓库中的相同内容将始终以相同的名称存储.
- Git 可以在读取对象时检测错误, 通过检查对象的名称是否仍然是其内容的 SHA1 哈希值.

## 对象

每个对象都由类型、大小和内容三部分组成, 内容决定了是什么类型的对象, 大小指的是内容的大小,
并有四种不同类型的对象: blob、tree、commit 和 tag.

- blob: 用于存储文件数据, 通常是一个文件.
- tree: 像一个目录, 引用了一堆其他的 tree 或 blob(即子目录和文件).
- commit: 指向一棵树, 标志着该项目在某一个时间点上的样子. 它包含关于该时间点的元信息,
  如时间戳、上次提交后的修改作者、指向上次提交的指针.
- tag: 一种将特定的提交标记为某种特殊方式的方法. 它通常被用来标记某些提交为特定的版本或类似的东西.

## 与 SVN

SVN 都是使用的增量存储系统--它们存储一次提交和下一次提交之间的差异.

Git 不会这么做, 它在每次提交时存储项目中所有文件在此树结构中的外观的快照.

## blob

blob 通常存储文件的内容.

![](http://shafiul.github.io/gitbook/assets/images/figure/object-blob.png)

可以使用`git show`检查任何 blob 的内容. 假设我们 有一个 blob 的 SHA, 我们可以查看它的内容:

```bash
git show 6ff87c4664
```

blob 对象只不过是一块二进制数据. 它不引用任何其他内容 或具有任何类型的属性, 甚至不是文件名.

由于 blob 完全由数据定义, 如果目录树中(或存储库的多个不同版本中)的两个文件具有相同的内容, 它们将共享相同的 blob 对象.
对象完全独立于其在目录树中的位置, 重命名文件不会更改与该文件关联的对象.

## tree

tree 是一个简单对象, 它有一堆指向 blob 和其他 tree 的指针--通常表示目录/子目录的内容.

![](http://shafiul.github.io/gitbook/assets/images/figure/object-tree.png)

我们可以使用通用的`git show`来检查树对象, 也可以使用`git ls-tree`提供更多详细信息.

```bash
git ls-tree fb3a8bdd0ce
```

可以看见, 树对象包含一个条目列表, 每个条目都有一个模式、对象类型、SHA1 名称和名称, 按名称排序,
它表示当个目录树的内容.

树引用的对象可以是 blob, 表示文件的内容, 也可以是另一棵树, 表示子目录的内容.
由于 tree 和 blob 与所有其他对象一样, 由其内容的 SHA1 哈希命名, 因此当且仅当它们的内容相同时,
两棵树才具有相同的 SHA1 名称. 这允许 git 快速确定两个相关树对象之间的差异,
因为它可以忽略任何相同对象名称的条目.

## commit

commit 对象将 tree 的物理状态与对我们如何到达那里以及为什么到达那里的描述联系起来.

![](http://shafiul.github.io/gitbook/assets/images/figure/object-commit.png)

可以使用`git show`/`git log`命令以及`--pretty=raw`选项来查看想要的提交.

```bash
git show -s --pretty=raw 2be7fcb476
```

## 对象模型

既然我们了解了三种主要对象类型(blob、tree 和 commit), 那么它们是怎么组合在一起的呢?

如果我们具有以下目录结构的简单项目:

```tree
.
|-- README
`-- lib
    |-- inc
    |   `-- tricks.rb
    `-- mylib.rb

2 directories, 3 files
```

我们将其提交到 Git 存储库, 它将表示为:

![](http://shafiul.github.io/gitbook/assets/images/figure/objects-example.png)

可以看到我们为每个目录创建了一个 tree 对象, 并为每个文件创建了 blob 对象.
然后我们有一个指向根的 commit 对象, 所以我们可以跟踪我们的项目在提交时的样子.

## 标签对象

![](http://shafiul.github.io/gitbook/assets/images/figure/object-tag.png)

标签对象包含对象名称、对象类型、标签名称、创建标签的人以及 kennel 包含签名的消息, 如图所示使用`git cat-file`:

```git
git cat-file tag v1.5.0

object 437b1b20df4b356c9342dac8d38849f24ef44f27
type commit
tag v1.5.0
tagger Junio C Hamano <junkio@cox.net> 1171411200 +0000

GIT 1.5.0
-----BEGIN PGP SIGNATURE-----
Version: GnuPG v1.4.6 (GNU/Linux)

iD8DBQBF0lGqwMbZpPMRm5oRAuRiAJ9ohBLd7s2kqjkKlq1qqC57SbnmzQCdG4ui
nLE/L9aUXdWeTFPron96DLA=
=2E+0
-----END PGP SIGNATURE-----
```
