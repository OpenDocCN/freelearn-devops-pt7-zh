# 第九章：定制 Gitolite

到目前为止，很明显 Gitolite 是一个管理服务器上 Git 仓库的强大工具。然而，最强大的工具允许管理员添加适合其站点独特需求的功能，因此不能期望将其添加到产品本身。例如，考虑 Git 本身，其 *hooks* 机制（详见 `man githooks`）包含了几个预定义的 hooks，用户可以在其仓库上安装这些 hooks 来定制 Git 在提交、变基、推送等生命周期的各个点上的行为。实际上，Gitolite 之所以能够执行分支级别的访问控制（而不仅仅是仓库级别的访问控制），完全是通过使用 Git 的 `update` hook 实现的。

# 核心与非核心 Gitolite

Gitolite 比仅仅允许您根据特定位置的需求定制它要进一步。实际上，Gitolite *已经* 配备了几个可选功能的定制。其中一些定制已启用，默认情况下启用，而其他一些则被禁用，尽管只需快速编辑 `$HOME/.gitolite.rc` 即可启用它们。

因此，Gitolite 将 **核心** 和 **非核心** Gitolite 代码区分开来。如果你查看过 Gitolite 源代码树（如果你克隆了 Gitolite 源代码，则位于 `src` 目录下），你会注意到顶层有几个目录和一些文件。在这些目录中，Gitolite 认为以下目录包含非核心代码：`commands`、`syntactic-sugar`、`triggers`、`lib/Gitolite/Triggers` 和 `VREF`。其他所有内容都被视为 *核心*。

这种区分还有助于决定是否添加新功能。如果该功能需要更改核心 Gitolite，将需要更加仔细地考虑和思考，即使如此，只有在确实需要该变更的多个用户时才会发生。然而，实际上，Gitolite 的定制功能如此强大，以至于几乎不需要对核心 Gitolite 进行任何更改。

# 非核心代码的类型和示例

Gitolite 允许为您的站点开发四种类型的定制。这可能听起来有些吓人，但实际上大多数人只使用其中的两种。我们现在将逐一描述它们。

## Commands

Gitolite 允许远程用户在服务器上运行特定的命令，格式为 `ssh git@host command-name`。命令在远程使用前需要启用；请参阅 第八章 中关于对 `rc` 文件进行更改的部分，*允许用户创建仓库*。有一种看法是将其视为为用户提供一个非常受限制的 shell，只允许执行特定的命令。

我们已经在第八章中遇到过一些 Gitolite 命令，比如 `perms` 和 `D` 命令，以及之前章节中的 `info` 和 `help`。Gitolite 一共提供了二十多个命令，虽然默认情况下，只有五个命令可以用于远程调用。更多命令列出了在 `$HOME/.gitolite.rc` 中，但由于被注释掉，因此默认是禁用的。只需删除该行的注释标记即可启用它们。

然而，Gitolite 自带的许多命令并不适用于远程使用，因此它们不会出现在 `$HOME/.gitolite.rc` 中（即便是注释掉的形式也没有）。这些命令是作为服务器端脚本或其他非核心程序的辅助工具使用的。最方便的这些命令之一是 `access` 命令，其帮助信息如下：

```
Usage:  gitolite access [-q] <repo> <user> <perm> <ref>

Print access rights for arguments given.  The string printed has the word
DENIED in it if access was denied.  With '-q', returns only an exit code
(shell truth, not perl truth -- 0 is success).

 - repo: mandatory
 - user: mandatory
 - perm: defauts to '+'.  Valid values: R, W, +, C, D, M
 - ref:  defauts to 'any'.  See notes below

Notes:
 - ref: Any fully qualified ref ('refs/heads/master', not 'master') is fine.
 The 'any' ref is special -- it ignores deny rules (see docs for what this
 means and exceptions).

Batch mode: see src/triggers/post-compile/update-git-daemon-access-list for a
good example that shows how to test several repos in one invocation.  This is
orders of magnitude faster than running the command multiple times; you'll
notice if you have more than a hundred or so repos.

```

正如你所看到的，这在编写自己的代码时非常有用，尤其是在你需要检查用户对一个或多个仓库的访问权限时。

在服务器上运行 `gitolite help` 将列出所有可用的命令；而运行 `ssh git@host help` 将列出所有 *远程* 可用的命令。此外，Gitolite 还自带了几个命令，它们是 Gitolite 内部实现的，实际上是 "核心" 的一部分。运行 `gitolite -h` 可以获取这些命令的列表及简要描述。

所有 Gitolite 命令在使用 `-h` 单一参数调用时都会返回用法信息。如果你编写自己的命令，遵循这一约定是个好主意。

以下是 Gitolite 中一些现有命令的列表，并附带简要描述：

+   `access`：此命令打印或测试用户在仓库上的访问权限。当你编写自己的命令时，这非常有用。请参阅下面 `fork` 命令的描述，了解一个例子。

+   `D`：此命令允许用户删除他们创建的仓库（参见第八章，*允许用户创建仓库*）。

+   `desc`：此命令显示或设置用户创建的仓库的描述。

+   `fork`：此命令会在服务器上创建一个仓库的分支。它会复制一个仓库并创建一个新的仓库，内容相同。它会检查确保读取者对源仓库有读取权限，并且允许创建目标仓库（参见第八章，*允许用户创建仓库*）。此命令使用 `-l` 选项调用 `git clone`，因此运行非常快速。（如果没有此命令，用户的替代做法是先克隆源仓库，再用该仓库来创建并推送目标仓库。对于大型仓库，这可能需要花费较长时间。）

+   `git-config`：此命令打印（或测试是否存在）仓库中的 "config" 配置项。

+   `help`：此命令会打印出所有可用命令的列表。

+   `info`：此命令打印你的用户名、git/gitolite 的版本号，以及你可以访问的所有仓库。

+   `perms`：列出或设置用户创建的仓库的权限。

在后续部分中，我们将看到如何创建你自己的命令。

## 语法糖

语法糖脚本是一种定制方式，大多数人很少、甚至从未需要编写或遇到它们。它们在管理员希望在 Gitolite 的访问控制语言中添加一些额外的、纯粹与语法相关的功能时非常有用。在这种情况下，可以编写一个语法糖助手脚本，将管理员编写的内容转换为 Gitolite 可解析的格式。

Gitolite 附带了一些语法糖助手脚本。例如，其中一个是允许在 Gitolite 的 `conf` 文件中使用 C 风格的续行符，因为 Gitolite 通常不允许这样做。另一个是提供简单的宏功能。

## 触发器

可以说，Gitolite 最强大的定制功能是触发器功能。Gitolite 触发器相当于 Git 的钩子。正如 Git 提供了在各个点运行的钩子（例如，`pre-commit`、`pre-receive` 和 `post-receive` 等），类似地，Gitolite 的触发器也会在 Gitolite 管理的推送或获取的生命周期中的特定点运行。

然而，Git 钩子和 Gitolite 触发程序之间是有区别的。Git 定义了几个钩子，并要求你的钩子代码必须精确命名为其中之一（例如，`post-receive` 或 `update`）。而 Gitolite 则允许你定义一组触发程序，当触发点到达时，它会按顺序调用这些程序。只有触发点的名称是固定的。当然，这也意味着，你的程序可以使用任何你喜欢的名称。

从定制角度来看，重要的触发点是 `INPUT`、`POST_CREATE` 和 `POST_COMPILE`，尽管还支持多个其他触发点。

`INPUT` 触发器的目的是以某种方式操作输入参数或环境。由于子程序无法影响父程序的环境，`INPUT` 触发器需要用 Perl 编写，并作为模块安装在 `lib/Gitolite/Triggers` 中（与任何语言编写的普通程序不同，后者是安装在 `triggers` 目录中的）。使用 `INPUT` 触发器的功能示例包括为某些用户提供完全的 shell 访问权限，以及允许仓库具有别名。

### 提示

本章将提及许多本书范围之外的非核心特性。请参阅 Gitolite 的在线文档获取详细信息。

`POST_CREATE` 触发点用于在新仓库创建后执行任何需要完成的整理或报告任务。例如，Gitolite 使用这个触发点来运行代码，在用户创建*野性*仓库时更新 gitweb 和 git-daemon 的访问列表。

`POST_COMPILE`触发点帮助您在推送`gitolite-admin`存储库时执行附加任务。此触发点与 Gitolite 一起提供的程序数量最多相关联。其中大多数与 ssh 密钥有关，或者更新 gitweb 和 git-daemon 的访问列表。

## 虚拟引用

可用的最后一种非核心定制类型是 Gitolite 根据 Gitolite 称为虚拟引用的东西做出访问决策的能力。执行此操作的脚本称为`VREFs`；它们复杂且重要到足以使下一章完全专门介绍它们。

# 编写您自己的非核心代码

编写您自己的代码来添加特定于您站点的功能相当容易。例如，假设我们希望每次开发人员创建一个*wild repository*时都向管理员发送电子邮件。我们假设标准的 Unix 实用程序存在且可用。特别是，我们假设 Unix 邮件命令可用。此命令从标准输入获取消息，并从命令行参数获取主题和收件人数据，然后发送邮件，因此非常适合我们的目的。

由于这是需要在创建存储库时运行的操作，因此需要将其添加到`POST_CREATE`触发器列表中。根据 Gitolite 文档，当创建一个 wild repository 时，每个`POST_CREATE`触发器列表中的程序都将被调用，第二个参数是刚刚创建的存储库的名称，而第三个参数是创建它的用户的名称。（如果为空，则这不是一个 wild repository 的创建，而是通过管理员将存储库添加到 Gitolite `conf`文件并推送更改。）

因此，这段代码可以简化为以下形式：

```
#!/bin/bash
[ -n $3 ] && echo | mail -s "new repo $2 created" admin_group@example.com

```

现在我们已经编写了这段代码，我们需要将其放在 Gitolite 可以找到并在正确时间使用它的地方。

我们决定创建一个名为`$HOME/local`的新目录，用于保存所有我们的本地定制。在此目录中，我们添加一个名为`triggers`的子目录，并在其中放置此脚本，命名为`new-repo-alert`。（不要忘记对脚本执行`chmod +x`！）

现在，我们编辑 Gitolite 的`rc`文件`($HOME/.gitolite.rc)`。在这个文件中，我们找到一行注释掉的定义`LOCAL_CODE`变量的行，但恰好指向我们选择放置自定义的地方，所以我们只需取消注释。

然后我们在`LOCAL_CODE`变量之后立即添加以下代码行：

```
POST_CREATE => [
 'new-repo-alert',
],

```

您是否注意到在右括号后的尾随逗号？这就是您需要做的全部。从现在开始，每当用户创建一个新的“wild”存储库时，将执行`new-repo-alert`脚本。

第二个例子，我们将创建一个小命令。我们使用的例子将允许用户使用`git count-objects`命令检查存储库的大小。我们的命令将默认使用`-v`选项运行，因为这是最通用和有用的。

为此，创建一个名为`$HOME/local/commands`的目录，并在该目录下放置一个名为`count-objects`的脚本。确保脚本是可执行的（`chmod +x`）。该脚本的代码如下：

```
#!/bin/bash

repo=$1

gitolite access -q $repo $GL_USER W any || {
 echo Sorry $GL_USER, you are not authorized
 exit 1
}

cd $GL_REPO_BASE/$repo.git
git count-objects -v

```

这段代码的有趣之处不在于实际的`count-objects`命令。最通用的，也最符合你需求的命令是`gitolite access`命令，其使用信息我们在前面的章节中已经看到过。在这里，我们使用它来确保执行该命令的用户至少拥有目标仓库的写权限，然后再允许命令运行。

最后，将此命令添加到`rc`文件的`ENABLE`命令列表中，最好放在`COMMANDS`部分。

### 提示

请注意，`count-objects`是一个无害的命令，因此可能不需要保护。然而，如果你稍微扩展使用场景，允许用户启动`git gc`操作，甚至是`git fsck`，你就需要更加小心了。这些命令中的一些在被多次执行或被多个人同时执行时，处理起来可能不太好。确保你的命令做一些速率限制或序列化。

其他命令需要提供参数。如果你的脚本接受用户的参数，一定要在执行命令前对其进行清洗。一个写得不小心的命令可能会破坏 Gitolite 的所有访问控制！

从这两个例子中可以看出，向你的网站添加新功能最重要的方面是决定何时以及如何调用该功能——它应该是用户命令，还是一个在特定时刻触发的操作，或许是一个能影响整体命令结果的`VREF`，等等。在某些情况下，它甚至可能是一个组合，例如，一个命令和一个 VREF 一起工作。极端的例子是，Gitolite 的镜像功能完全作为非核心代码编写，它作为一个命令实现，并且一个 Perl 模块被添加到`INPUT`、`PRE_GIT`和`POST_GIT`触发器列表中。

# 摘要

在本章中，我们了解了如何定制 Gitolite 或向其添加特定于你站点的新功能。这是一个相当复杂的话题，但如果你动手实践，开始编写程序，你很快就会对这一概念感到得心应手，并且对这个功能的强大之处有非常好的体会。

下一章将重点讲解`VREF`，这是一个强大的功能，能够实现更细粒度的访问控制，以及基于除 Gitolite 通常使用的因素外的访问控制。
