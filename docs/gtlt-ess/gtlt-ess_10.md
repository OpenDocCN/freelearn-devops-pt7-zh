# 第十章：理解 VREFs

我们在第七章，*高级访问控制与配置* 中简要介绍了 VREF，包括一个小示例，展示了 Gitolite 如何根据推送中修改的文件来允许或拒绝推送。在本章中，我们将更详细地探讨 VREF，因为这是 Gitolite 最强大的功能之一。我们将从简单的用法开始，然后逐步深入更复杂的用法。

# 迁移更新钩子

一些站点可能已经在其现有的（Gitolite 之前的）仓库设置中使用了更新钩子。由于 Gitolite 将更新钩子保留给自己，这在切换过程中会带来一些问题。

如果您的站点有这样的更新钩子，VREF 机制可以帮助替代它们。替换更新钩子是 VREF 最简单的用法之一，但理解如何做到这一点也是理解 Gitolite VREF 机制全部功能的一个良好起点。

要将现有的更新钩子转换为 VREFs，首先在 `$HOME/local` 中创建一个名为 VREF 的目录（我们延续第九章，*自定义 Gitolite* 中的约定，即 `rc` 文件中的 `LOCAL_CODE` 变量指向此目录）。然后，将每个独特的更新钩子复制到这个新创建的目录中，并对每个更新钩子进行某种重命名。

举个例子，假设有一个仓库经常由使用 Windows 的新手用户操作，因此更新钩子被用来确保没有行结束符问题。你可以将其重命名为 `check-crlf`。

现在，对于每个需要进行此检查的仓库（即每个在 Gitolite 之前的设置中使用了该特定更新钩子的仓库），添加如下规则：

```
-  VREF/check-crlf  =  @all

```

当 Gitolite 的更新钩子处理一个推送时，它会遇到这个 VREF 规则，并调用 `check-crlf` 程序。传递的前三个参数与 Git 自身传递给更新钩子的参数相同，如果程序以非零退出代码退出，Gitolite 将拒绝推送。为了实现这一切，无需更改 `check-crlf` 代码；它直接生效。

敏锐的读者可能已经注意到，他们可以使用以下规则来替代上面所示的规则：

```
-  VREF/check-crlf  =  @junior-developers

```

这有助于将检查限制为规则中指定的用户。*换句话说，Gitolite 允许选择性地应用普通的更新钩子*，这可能非常有用。

# 向 VREF 代码传递参数

假设我们在（Gitolite 之前的设置中）有一个更新钩子，用于防止某些用户对某些文件进行更改。一个方法是编写一个更新钩子，检查这些文件，并将其作为 VREF 使用，如前一节所示。然而，检查的文件列表或模式需要在 VREF 代码中以某种方式进行编码，或者需要找到其他方法来传递这些信息。

Gitolite 允许你将额外的参数传递给 VREF 代码。假设使用的 VREF 名称为 `NAME`，那么你可以像这样传递参数，而不仅仅是说：

```
-  VREF/NAME           =  @junior-developers

```

确保 `NAME` VREF 的代码知道我们正在讨论哪些文件，你可以这样说：

```
-  VREF/NAME/.*\.h$    =  @junior-developers

```

如果你稍后发现另一组用户需要以类似的方式进行限制，但限制的是不同的文件集，你会发现这种方法真的非常有用。假设我们有一组技术写作人员在编写文档；他们没有理由去触碰构成源代码的文件：

```
-  VREF/NAME/.*\.[ch]$    =  @tech-writers

```

当然，VREF 代码现在与其作为更新钩子时的形式不完全相同。除了前面提到的三个参数（这些参数与 `update` 钩子中列出的参数相同）之外，现在还有其他一些参数，而我们刚刚添加的文件模式就是其中之一（具体来说是第八个参数）。VREF 代码必须从传入的参数中提取该模式，并用它来决定是否允许或拒绝该推送。

# 使用权限字段

假设我们有几种不同类型的源代码文件，将它们全部列在技术写作人员的规则中不仅繁琐，而且容易出错，因为我们可能会遗漏一些。不过，我们确实知道技术写作人员只处理文档，所以我们宁愿限制他们只能操作 `*.odt` 文件。

直到现在，我们一直认为 `NAME` VREF 的行为是这样的：接收一个文件名模式，如果在推送中有任何文件更改匹配该模式，则退出并返回非零代码，以向 Gitolite 发出拒绝推送的信号。正如你所看到的，这种行为完全忽略了权限字段；也就是说，即使这个规则也会产生相同的效果：

```
RW+    VREF/NAME/.*\.[ch]$  =  @tech-writers

```

我们需要的是一种方法来考虑权限字段。我们的第一反应可能是开发某种方式将此字段传递给 VREF 代码，也许是通过一些新的语法，并让 VREF 代码在需要时反向检查。

然而，这使得 VREF 代码变得更加复杂，同时也没有利用 Gitolite 的规则处理逻辑。

Gitolite 会根据匹配每条规则的 *引用*（通常类似 `refs/heads/master` 或 `refs/tags/v1.0`）来处理访问控制规则。所以，利用这一点的一种方法是，不让 VREF 代码真正做出决定，而只是 *输出一些东西*，让 Gitolite 捕获并通过其访问控制规则，就像处理正常的 *引用* 一样。

你也可以称之为 *虚拟引用*！

### 提示

让我们简要回顾一下术语，VREF 是执行的代码，而虚拟引用是它可能返回给 Gitolite 的内容。

## 维护更新钩子功能

但是，我们不希望在作为虚拟引用使用时，影响标准（Git）更新钩子的行为，正如本章前面所描述的那样。这一点相当简单——Gitolite 仅当 VREF 输出的行以 `VREF/` 开头时，才会将其视为虚拟引用，即使如此，只有当该 VREF 以零状态退出时才会如此。

# 默认情况下为成功

在此时，我们需要更改 `NAME` 虚拟引用。它不再做出决定，而是简单地列出所有已更改的文件，每个文件名前面都加上 `VREF/NAME/`。

一旦完成了这些，接下来可能看起来只有以下规则就足够了：

```
RW+    VREF/NAME/.*\.odt$  =  @tech-writers

```

然而，这并不是全部。

虚拟引用与真实引用的处理方式略有不同。对于真实引用，如果没有访问规则匹配该引用（以及用户和实际的写入类型），默认情况下会拒绝推送。

然而，虚拟引用是作为*附加*规则设计的，它们增加了常规 Gitolite 访问规则无法提供的检查。因此，合理的做法是，如果没有虚拟引用匹配，就应当视为没有任何额外的检查*应用*于此次推送，因此默认情况下是*允许*推送的。

因此，我们需要再添加一条规则，将我们的最终规则集变为这样：

```
RW+    VREF/NAME/.*\.odt$  =  @tech-writers
-      VREF/NAME/          =  @tech-writers

```

大致来说，这样做的效果是，对于每个已更改的文件，通过在文件名前加上 `VREF/NAME/` 来生成一个虚拟引用，并将该虚拟引用通过规则集进行检查。其余的很明显，例如，修改一个名为 `foo.c` 的文件会创建一个名为 `VREF/NAME/foo.c` 的虚拟引用，这将仅匹配第二条规则，从而导致此次推送被拒绝。而文件名以 `.odt` 结尾的文件会匹配第一条规则，不会导致拒绝。

# 示例虚拟引用及其用法

Gitolite 源代码树带有一些准备好使用的虚拟引用（VREF）。要使用它们，你只需添加类似于上一节末尾所看到的规则。我们将查看其中的几个，以便了解它们是如何使用的，然后从头开始设计一个，以便我们知道如何添加自己的规则。

### 提示

如果你查看 Gitolite 源代码树，你实际上不会找到名为 `NAME` 的虚拟引用。这是因为 `NAME` 是特殊的，其代码内建在 Gitolite 中。

对 Git 新手来说，有时可能会创建一个提交，修改的文件比实际需要修改的文件多得多。也许他们在其他文件中添加了调试语句，或者可能他们不小心将文件保存为不同的行结尾格式（Unix LF 与 Windows CRLF），等等。

如果你确定你的新开发者只会被分配相对简单的任务，并且在任何时候都不应该有任何特定的任务修改超过五个文件，你可以使用 `COUNT` 虚拟引用来防止他们推送更多文件，从而保护仓库不被大量修改，避免前一段中提到的那种情况。以下是实现这一目标的规则：

```
-  VREF/COUNT/5    =  @new-devs

```

`COUNT` 虚拟引用本质上计算当前推送中所更改的所有文件（并且这些文件不存在于任何其他分支或标签中）。

如果最后这一部分听起来有点复杂，想象一下当开发者仅仅从一个已有的分支创建一个新分支并推送时会发生什么。我们不希望`COUNT` VREF 仅仅因为被推送的分支的旧值为空，而将分支中的所有文件都视为已更改。这就是为什么`COUNT`代码会查看那些在其他任何引用中都没有出现的提交。

`MAX_NEWBIN_SIZE` VREF 的概念类似。这是为了解决有时开发者无意中提交了 JAR 文件或者构建步骤产生的可执行文件的问题。像这样的可执行文件通常比正常的源文件要大一些，因此，如果你有一个合理的大小限制，可以使用这个 VREF 来强制执行它。

# 编写你自己的 VREF

这是一个 VREF 非常有用的实际案例。我们将使用它来设计一个非常简单的 VREF，这是现有规则无法完成的。

要求很简单：对于任何一个仓库`foo`，如果一个名为`l10n`的仓库包含一个名为`foo`的目录，那么你就不能将任何名为`*.po`的文件推送到`foo`。

### 提示

这段内容改编自一个更复杂的现实生活案例，但对于我们的目的来说，这样已经足够了。如你所料，这是一个多仓库系统，逐步朝着集中本地语言文件的方向发展，使得翻译人员只需要处理一个仓库。每个仓库的本地语言文件在准备好后会被迁移过来，从那时起，所有的本地化文件必须提交到为此目的创建的单一仓库。

由于这是一个非常特定的用例，我们可以编写一个不接受任何参数的简单 VREF。我们的规则可以是这样的：

```
repo @all
-  VREF/l10n-check  =  @all

```

如你所见，我们将这一规则应用于所有仓库。（如果你有多个仓库，且这些仓库永远不会符合此条件，那么 VREF 将在每次推送时为它们触发，这样稍显低效。如果是这种情况，可以将`@all`替换为一个仅包含需要此检查的仓库的组名）

### 提示

这些仓库中可能在其他地方有各自的规则——我们不需要在每个仓库部分都放入这个 VREF 规则。这是*规则累积与委托*的一个例子，如在第六章《访问控制入门》中讨论的那样。

下面是编写这个 VREF 的一种方法（理解此代码需要一些基本的 Shell 语法和 Git 概念的知识）：

```
#!/bin/bash

# see Gitolite documentation for arguments and meanings
oldtree=$4
newtree=$5
refex=$7

# no *.po files changed?  No problem!
git diff --name-only $oldtree $newtree | grep '.*\.po$' >/dev/null || exit 0

cd $GL_REPO_BASE/l10n.git
# no directory with the same name as $GL_REPO in the l10n repo?  No problem!
git ls-tree master | grep "\s$GL_REPO$" >/dev/null || exit 0

echo $refex "sorry, PO files must be added to '$GL_REPO' subdirectory in 'l10n' repo"

```

如你所见，第一个检查使用 git diff 命令检查此次推送是否更改了任何`po`文件。如果没有，那么就没有什么需要检查的，我们直接退出，不做任何操作。第二个检查切换到`l10n`仓库，然后在该仓库中运行`ls-tree`命令，检查是否包含一个文件或目录，其名称与用户正在推送的仓库名称相同。如果没有，则可以直接退出，不会报错。

如果这两个检查成功，我们需要触发错误信号。一个方法是直接使用 `exit 1`；Gitolite 会捕获 VREF 代码的终止并拒绝推送。另一方面，我们可以打印 refex 本身（在这种情况下是 `VREF/l10n`，但养成使用参数的习惯是个好习惯）。它将匹配我们设置的访问规则，因为权限是 "-"，所以推送将被拒绝。

但是，VREF 特性提供了更多功能。如果在 refex 后打印一个空格，然后是一些解释性信息，当推送被拒绝时，这条信息将会被打印出来：

```
$ git push
Counting objects: 4, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 351 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
remote: FATAL: W VREF/l10n t2 u1 DENIED by VREF/l10n 
remote: sorry, PO files must be added to 't2' subdirectory in 'l10n' repo 
remote: 
remote: error: hook declined to update refs/heads/master 
To u1:t2 
 ! [remote rejected] master -> master (hook declined) 
error: failed to push some refs to 'u1:t2' 

```

如果你写 VREF 来捕获少见的情况，那么你可能会发现通过一些简单的英语来扩展 Gitolite 相对简陋的错误报告功能非常有用，这样你的用户就不必太费劲思考了！

# 总结

本章我们探讨了 Gitolite 最强大的功能之一——通过编写 VREF 使用任意外部因素进行访问控制决策的能力。下一章将通过讨论镜像来结束我们对 Gitolite 的探索——这是一个大型多站点设置可能会非常有用的功能。
