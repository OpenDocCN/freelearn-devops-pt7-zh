# 第二章：配置

在本章中，我们将涵盖以下内容：

+   配置目标

+   查询现有配置

+   模板

+   `.git` 目录模板

+   一些配置示例

+   Git 别名

+   示例化的 refspec

# 介绍

Git 在开发者的日常工作中扮演着基本且至关重要的角色，但它也非常复杂且高度可配置。本章将概述最重要的可用选项，并提供正确的工具，以便学习和浏览各种配置标志和字段，从而根据个人需求定制你的 Git 使用体验。

# 配置目标

在本节中，我们将查看可以配置的不同层级。这些层级如下：

+   `SYSTEM`：此层级是全系统范围的，可以在 `/etc/gitconfig` 中找到。

+   `GLOBAL`：此层级是用户的全局配置，可以在 `~/.gitconfig` 中找到。

+   `LOCAL`：此层级仅限于当前仓库，并可以在 `.git/config` 中找到。

# 准备就绪

我们将在此示例中使用 `jgit` 仓库；克隆它，或者使用你在 第一章，《导航 Git》中已经克隆的版本，如以下命令所示：

```
$ git clone https://git.eclipse.org/r/jgit/jgit
$ cd jgit
```

# 如何操作...

在上一章中，我们看到如何使用命令 `git config --list` 列出配置项。这个列表实际上是由 Git 提供的三个不同层级的配置构成的：全系统配置 `SYSTEM`，用户的全局配置 `GLOBAL`，以及本地仓库配置 `LOCAL`。

1.  对于每个配置层级，我们都可以查询现有配置。在默认安装 Git 扩展的 Windows 系统中，不同的配置层级大致如下：

```
$ git config --list --system
core.symlinks=false
core.autocrlf=true
color.diff=auto
color.status=auto
color.branch=auto
color.interactive=true
pack.packsizelimit=2g
help.format=html
http.sslcainfo=/bin/curl-ca-bundle.crt
sendemail.smtpserver=/bin/msmtp.exe
diff.astextplain.textconv=astextplain
rebase.autosquash=true

# list the global configuration    
$ git config --list --global
merge.tool=kdiff3
mergetool.kdiff3.path=C:/Program Files (x86)/KDiff3/kdiff3.exe
diff.guitool=kdiff3
difftool.kdiff3.path=C:/Program Files (x86)/KDiff3/kdiff3.exe
core.editor="C:/Program Files (x86)/GitExtensions/GitExtensions.exe" fileeditor
core.autocrlf=true
credential.helper=!"C:/Program Files (x86)/GitExtensions/GitCredentialWinStore/git-credential-winst
ore.exe"
user.name=John Doe
user.email=john.doe@example.com 
# list the configuration for this repository
$ git config --list --local
core.repositoryformatversion=0
core.filemode=false
core.bare=false
core.logallrefupdates=true
core.symlinks=false
core.ignorecase=true
core.hidedotfiles=dotGitOnly
remote.origin.url=https://git.eclipse.org/r/jgit/jgit
    remote.origin.fetch=+refs/heads/*:refs/remotes/origin/*
    branch.master.remote=origin
    branch.master.merge=refs/heads/master
```

1.  我们还可以通过使用以下命令查询单个键，并将作用范围限制在三层中的一个：

```
$ git config --global user.email
john.doe@example.com 
```

1.  我们可以为当前仓库设置不同的用户电子邮件地址：

```
$ git config --local user.email john@example.com 
```

现在，列出 `GLOBAL` 层级的 `user.email` 将返回 `john.doe@example.com`，列出 `LOCAL` 层级将返回 `john@example.com`，而在不指定层级的情况下列出 `user.email` 将返回当前仓库操作中使用的有效值；在这种情况下，`LOCAL` 层级的值 `john@example.com`。当需要有效值时，它将优先使用该值。如果在不同的层级中指定了两个或更多相同键的值，则较低层级的值会优先。Git 在需要配置值时，首先会查看 `LOCAL` 配置。如果在此未找到，则查询 `GLOBAL` 配置。如果在 `GLOBAL` 配置中也未找到，则使用 `SYSTEM` 配置。

如果这些都无效，则会使用 Git 中的默认值。

在上面的示例中，`user.email` 在 `GLOBAL` 和 `LOCAL` 层级中都有指定。因此，将使用 `LOCAL` 层级的值。

# 工作原理...

查询三层配置时，简单地返回配置文件的内容；`/etc/gitconfig`用于系统范围配置，`~/.gitconfig`用于用户特定配置，`.git/config`用于仓库特定配置。如果未指定配置层，则返回的值为有效值。

# 还有更多内容...

除了通过键值在命令行中设置所有配置值外，你还可以直接编辑配置文件来设置它们。打开你喜欢的编辑器中的配置文件，设置你需要的配置，或者使用内置的`git config -e`命令在 Git 配置的编辑器中直接编辑配置。你可以通过更改`$EDITOR`环境变量或使用`core.editor`配置目标来设置你喜欢的编辑器，例如：

```
$ git config --global core.editor vim  
```

# 查询现有配置

在这个例子中，我们将看看如何查询现有的配置并设置配置值。

# 准备工作

我们将再次使用`jgit`，通过以下命令来操作：

```
$ cd jgit
```

# 如何操作...

你可以使用`git config`查询你的本地和全局 Git 配置。在这一部分，我们将展示几个例子。

1.  要查看当前 Git 仓库的所有有效配置，请运行以下命令：

```
$ git config --list
user.name=John Doe
user.email=john.doe@example.com
core.repositoryformatversion=0
core.filemode=false
core.bare=false
core.logallrefupdates=true
remote.origin.url=https://git.eclipse.org/r/jgit/jgit
    remote.origin.fetch=+refs/heads/*:refs/remotes/origin/*
    branch.master.remote=origin
    branch.master.merge=refs/heads/master
```

之前的输出当然会反映运行命令的用户。输出中的名字和电子邮件将反映你的设置，而不是`John Doe`。

1.  如果我们只对单个配置项感兴趣，可以通过其`section.key`或`section.subsection.key`来查询：

```
$ git config user.name
John Doe
$ git config remote.origin.url
https://git.eclipse.org/r/jgit/jgit
```

# 它是如何工作的...

Git 的配置存储在纯文本文件中，类似于键值存储。你可以通过键来设置/查询并获取值。以下是基于文本的配置文件示例（来自`jgit`仓库）：

```
$ cat .git/config
  [core]
    repositoryformatversion = 0
    filemode = false
    bare = false
    logallrefupdates = true
  [remote "origin"]
    url = https://git.eclipse.org/r/jgit/jgit
    fetch = +refs/heads/*:refs/remotes/origin/*
  [branch "master"]
    remote = origin
    merge = refs/heads/master
```

# 还有更多内容...

设置配置值也很简单。你可以使用与查询配置时相同的语法，只不过需要在值后加上一个参数。要在`LOCAL`层设置一个新的电子邮件地址，我们可以执行以下命令：

```
git config user.email john.doe@example.com
```

`LOCAL`层是默认层，如果没有其他指定。如果值中需要空格，你可以像配置姓名时那样用引号将字符串括起来：

```
git config user.name "John Doe"
```

你甚至可以设置自己的配置，这对核心 Git 没有任何影响，但在脚本编写/构建等方面会很有用：

```
$ git config my.own.config "Whatever I need"  
```

列出该值：

```
$ git config my.own.config 
Whatever I need  
```

删除/取消配置项也非常容易：

```
$ git config --unset my.own.config  
```

列出该值：

```
$ git config my.own.config 
```

# 模板

在这个例子中，我们将看到如何创建一个模板提交信息，这将在创建提交时显示在编辑器中。该模板仅适用于本地用户，而不会与仓库一起分发。

# 准备工作

在这个例子中，我们将使用第一章中的示例仓库，*Navigating Git*：

```
$ git clone https://github.com/PacktPublishing/Git-Version-Control-Cookbook-Second-Edition.git
$ cd Git-Version-Control-Cookbook-Second-Edition  
```

我们将使用以下命令作为提交信息的模板：

```
Short description of commit 

Longer explanation of the motivation for the change Fixes-Bug: Enter bug-id or delete line 
Implements-Requirement: Enter requirement-id or delete line 
```

将提交信息模板保存在`$HOME/.gitcommitmsg.txt`。文件名不是固定的，你可以选择任何你喜欢的文件名。

# 如何操作...

1.  为了让 Git 知道我们新的提交信息模板，我们可以设置配置变量`commit.template`，指向我们刚刚创建的包含该模板的文件；我们将全局设置它，以便适用于我们所有的仓库：

```
$ git config --global commit.template $HOME/.gitcommitmsg.txt
```

1.  现在，我们可以尝试修改一个文件，添加它，并创建一个提交。这将打开我们首选的编辑器，并预加载提交信息模板：

```
$ git commit

Short description of commit

Longer explanation of the motivation for the change

Fixes-Bug: Enter bug-id or delete line
Implements-Requirement: Enter requirement-id or delete line

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
# On branch master
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#  modified:   another-file.txt
#
~
~
".git/COMMIT_EDITMSG" 13 lines, 396 characters
```

1.  现在我们可以根据我们的提交编辑信息并保存，以完成提交。

# 它是如何工作的...

当`commit.template`被设置时，Git 会将模板文件的内容作为所有提交信息的起点。如果你有提交信息的策略，这非常方便，因为它大大增加了遵循该策略的机会。你甚至可以为不同的仓库设置不同的模板，因为你可以在本地级别设置配置。

# 一个 .git 目录模板

有时候，单单拥有全局配置是不够的。你还需要触发脚本（也称为 Git 钩子）、排除文件等的执行。可以通过设置模板选项`git init`来实现这一点。它可以作为命令行选项传递给`git clone`和`git init`，或者作为环境变量`$GIT_TEMPLATE_DIR`，或者作为配置选项`init.templatedir`。默认为`/usr/share/git-core/templates`。模板选项通过将模板目录中的文件复制到`.git ($GIT_DIR)`文件夹中来工作。默认目录包含示例钩子和一些建议的排除模式。在下面的示例中，我们将看到如何设置一个新的模板目录，并添加提交信息钩子和排除文件。

# 准备工作

首先，我们将创建模板目录。我们可以使用任何我们想要的名称，这里我们使用`~/.git_template`，如以下命令所示：

```
$ mkdir ~/.git_template
```

现在，我们需要在目录中填充一些模板文件。可以是一个钩子文件或排除文件。我们将创建一个钩子文件和一个排除文件。钩子文件位于`.git/hooks/name-of-hook`，排除文件位于`.git/info/exclude`。创建所需的两个目录，`hooks`和`info`，如以下命令所示：

```
$ mkdir ~/.git_template/{hooks,info}
```

为了保留默认模板目录（Git 安装提供的）中的示例钩子，我们将默认模板目录中的文件复制到新目录中。当我们使用新创建的模板目录时，它会覆盖默认目录。所以，将默认文件复制到我们的模板目录中会确保，除了我们的特定更改外，模板目录与默认目录相似，如以下命令所示：

```
$ cd ~/.git_template/hooks
$ cp /usr/share/git-core/templates/hooks/* .
```

我们将使用`commit-msg`钩子作为示例钩子：

```
#!/bin/sh
MSG_FILE="$1"
echo "\nHi from the template commit-msg hook" >> $MSG_FILE
```

这个钩子非常简单，它只会将`Hi from the template commit-msg hook`添加到提交信息的末尾。将其保存为`commit-msg`文件到`~/.git_template/hooks`目录，并通过以下命令使其可执行：

```
chmod +x ~/.git_template/hooks/commit-msg
```

现在提交信息钩子已完成，我们还将向示例中添加一个排除文件。排除文件的作用类似于`.gitignore`文件，但它不会被仓库跟踪。

我们将创建一个排除文件，排除所有`*.txt`文件，如下所示：

```
$ echo "*.txt" > ~/.git_template/info/exclude
```

现在，我们的模板目录已准备就绪，可以使用了。

# 如何做...

1.  我们的模板目录已准备好，并且可以按照前面描述的方式，作为命令行选项、环境变量，或者像这个例子一样，作为配置来设置：

```
$ git config --global init.templatedir ~/.git_template
```

1.  现在，所有使用`init`或`clone`创建的 Git 仓库将拥有模板目录的默认文件。我们可以通过创建一个新仓库来测试它是否有效，方法如下：

```
$ git init template-example
$ cd template-example
```

1.  让我们尝试创建一个`.txt`文件，并查看`git status`告诉我们什么。它应该会被模板目录中的排除文件忽略：

```
$ echo "this is the readme file" > README.txt
$ git status 
```

排除文件生效了！你可以自己添加文件扩展名，或者直接留空并继续使用`.gitignore`文件。

1.  为了测试`commit-msg`钩子是否有效，我们来尝试创建一个提交。首先，我们需要一个文件来提交。接下来，我们按如下方式创建并提交它：

```
$ echo "something to commit" > somefile
$ git add somefile
$ git commit -m "Committed something"
```

1.  现在，我们可以通过`git log`来查看历史记录：

```
$ git log -1
commit 1f7d63d7e08e96dda3da63eadc17f35132d24064
Author: John Doe <john.doe@example.com>
Date:   Mon Jan 6 20:14:21 2014 +0100

  Committed something

  Hi from the template commit-msg hook
```

# 它是如何工作的...

当 Git 创建一个新仓库时，无论是通过`init`还是`clone`，它会在创建目录结构时将`template`目录中的文件（默认位置是`/usr/share/git-core/templates`）复制到新仓库中。模板目录可以通过命令行参数、环境变量或配置选项来定义。如果没有指定，将使用默认的模板目录（与 Git 安装一起分发）。通过将配置设置为`--global`选项，所定义的模板目录将适用于所有用户（新的）仓库。这是一个很好的方式来在仓库之间分发相同的钩子，但它也有一些缺点。由于模板目录中的文件仅复制到 Git 仓库，因此对模板目录的更新不会影响现有的仓库。这可以通过在每个现有仓库中运行`git init`来重新初始化仓库来解决，但这可能相当繁琐。另外，模板目录可能会强制某些仓库应用钩子，而你并不希望这样做。这个问题可以通过简单地删除该仓库中`.git/hooks`目录里的钩子文件来轻松解决。

# 另请参阅

欲了解更多关于 Git 钩子的信息，请参考第七章，*使用 Git 钩子、别名和脚本提升你的日常工作效率*。

# 一些配置示例

核心 Git 系统中有配置目标。在本节中，我们将更详细地查看一些在日常工作中可能有用的配置目标。

我们将查看以下三个不同的配置区域：

+   重基与合并设置

+   对象过期

+   自动更正

# 准备就绪

在本练习中，我们将设置一些配置。我们将使用第一章《Navigating Git》中的数据模型仓库：

```
$ cd Git-Version-Control-Cookbook-Second-Edition  
```

# 如何操作...

让我们仔细看看前面提到的配置区域。

# 重基与合并设置

默认情况下，当执行`git pull`时，如果本地分支的历史与远程分支发生分歧，Git 将创建一个合并提交。然而，为了避免所有这些合并提交，可以配置仓库，使其在执行`git pull`时默认使用重基（rebase）而不是合并。与此选项相关的几个配置目标如下：

+   `pull.rebase`：当此配置设置为`true`时，在执行`git pull`时，会将当前分支拉取并重基到获取的分支上。也可以设置为`preserve`，以便在重基时不会压平本地的合并提交，方法是将`--preserve-merges`传递给`git rebase`。默认值为`false`，即该配置未设置。要在本地仓库中设置此选项，请运行以下命令：

```
$ git config pull.rebase true  
```

+   `branch.autosetuprebase`：当此配置设置为`always`时，通过`<git branch`或`git checkout`创建的任何新分支，如果跟踪其他分支，将会设置为拉取时进行重基（而非合并）。有效的选项如下：

    +   `never`：此设置用于拉取时默认执行重基（rebase）。

    +   `local`：此设置用于本地跟踪分支执行拉取时重基（rebase）。

    +   `remote`：此设置用于远程跟踪分支执行拉取时重基（rebase）。

    +   `always`：此设置用于所有被跟踪的分支执行拉取时重基（rebase）。

    +   要为所有新分支设置此选项，无论其跟踪远程还是本地分支，请运行以下命令：

```
$ git config branch.autosetuprebase always
```

+   `branch.<name>.rebase`：当此配置设置为`true`时，仅适用于`<name>`分支，指示 Git 在执行`git pull`时进行重基。它也可以设置为`preserve`，以便在执行`git pull`时不会压平本地的合并提交。默认情况下，任何分支都未设置此配置。要将仓库中的`feature/2`分支设置为默认重基，而非合并，可以运行以下命令：

```
$ git config branch.feature/2.rebase true
```

# 对象过期

默认情况下，Git 将在未引用的对象上执行垃圾回收，并清理`reflog`中超过 90 天的条目。为了让某个对象被引用，必须有某些内容指向它；如树、提交、标签、分支，或者一些内部 Git 记录，例如`stash`或`reflog`。有三个设置可以用来更改这个时间，如下所示：

+   `gc.reflogexpire`：这是了解分支历史在`reflog`中保存时间的一般设置。默认时间为 90 天。该设置是一个时间长度，例如`10 days`、`6 months`，也可以通过值`never`完全禁用。此设置可以通过在配置中提供模式来匹配`refs`模式。`gc.<pattern>.reflogexpire`：例如，此模式可以是`/refs/remotes/*`，并且过期设置仅适用于这些 refs。

+   `gc.reflogexpireunreachable`：此设置控制不属于当前分支历史的`reflog`条目在仓库中的可用时间。默认值为`30 days`，与前一个选项类似，它表示一个时间长度，或者设置为`never`以关闭该设置。此设置可以像前一个选项一样，设置为匹配`refs`模式。

+   `gc.pruneexpire`：此选项告知`git gc`修剪比设定时间更久的对象。默认值为`2.weeks.ago`，并且值可以表示为相对日期，例如`3.months.ago`。若要禁用宽限期，可以使用`now`作为值。要仅在远程分支上设置非默认的过期日期，可以使用以下命令：

```
$ git config gc./refs/remote/*.reflogexpire never
$ git config gc./refs/remote/*.reflogexpireunreachable "2 months"  
```

+   我们还可以设置一个日期，以便`git gc`更早地修剪对象：

```
$ git config gc.pruneexpire 3.days.ago  
```

# 自动更正

当你因为键入错误而看到如下消息时，这个配置非常有用：

```
$ git statis
git: 'statis' is not a git command. See 'git --help'.

Did you mean this?
 status 
```

通过设置`help.autocorrect`配置，你可以控制 Git 在意外输入错误时的行为。默认值为`0`，表示列出与输入相似的可选项（例如输入`statis`时，显示`status`）。负值表示立即执行相应的命令。正值表示在执行命令之前等待指定的十分之一秒（0.1 秒），因此在这段时间内可以取消命令。如果可以从输入的文本中推断出多个命令，则什么也不会发生。将值设置为半秒，给你一些时间来取消错误的命令，如下所示：

```
$ git config help.autocorrect 5
$ git statis
WARNING: You called a Git command named 'statis', which does not exist.
Continuing under the assumption that you meant 'status'
in 0.5 seconds automatically...
# On branch master
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#       modified:   another-file.txt
#
```

# 它的工作原理...

设置配置目标会改变 Git 的行为。前面的示例描述了几种常用的方法，使 Git 的行为不同于默认设置。在更改配置时，你应确保完全理解该配置的作用。因此，可以通过使用`git help config`来查看 Git 配置帮助页面。

# 还有更多...

Git 中有许多可用的配置目标。你可以运行`git help config`，所有配置项将显示并在几页中解释。

# Git 别名

别名是配置长且/或复杂的 Git 命令以表示简短实用命令的好方法。别名只是别名部分下的一个配置项。通常配置为`--global`，使其在任何地方都适用。

# 准备就绪

在这个示例中，我们将使用`jgit`仓库，这也是在第一章《导航 Git》中使用的，`master`分支指向`b14a93971837610156e815ae2eee3baaa5b7a44b`。可以使用第一章《导航 Git》中的克隆，也可以重新克隆该仓库，如下所示：

```
$ git clone https://git.eclipse.org/r/jgit/jgit
$ cd jgit
$ git checkout master && git reset --hard b14a939  
```

# 如何做到这一点...

1.  首先，我们将创建一些简单的别名，然后创建几个更特殊的别名，最后创建几个使用外部命令的别名。我们可以为每次需要切换分支时创建一个别名，而不是每次都输入`git checkout`，并命名为`git co`。我们可以对`git branch`、`git commit`和`git status`做相同的操作，如下所示：

```
$ git config --global alias.co checkout 
$ git config --global alias.br branch
$ git config --global alias.ci commit
$ git config --global alias.st status 
```

1.  现在，尝试在`jgit`仓库中运行`git st`，如下所示：

```
$ git st
# On branch master
nothing to commit, working directory clean  
```

1.  `alias`方法对于创建你认为 Git 缺失的命令也很有用。一个常见的 Git 别名是`unstage`，它用于将文件从暂存区移出，如下所示：

```
$ git config --global alias.unstage 'reset HEAD --' 
```

尝试编辑`jgit`仓库根目录中的`README.md`文件并将其添加到根目录中。

1.  现在，`git status/git st`应该显示如下内容：

```
$ git st
# On branch master
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#       modified:   README.md
#  
```

1.  让我们尝试取消暂存`README.md`，然后查看`git st`，如下所示：

```
$ git unstage README.md
Unstaged changes after reset:
M       README.md

$ git st
# On branch master
# Changes not staged for commit:
#   (use "git add <file>..." to update what will be committed)
#   (use "git checkout -- <file>..." to discard changes in working directory)
#
#       modified:   README.md
#
no changes added to commit (use "git add" and/or "git commit -a")
```

1.  别名的一个常见用例是以特定方式格式化 Git 的历史记录。假设你希望在提交时显示每个文件的新增和删除行数，并附带一些常见的提交数据。为此，我们可以创建以下别名，这样每次就不必输入所有内容：

```
$ git config --global alias.ll "log --pretty=format:"%C(yellow)%h%Cred%d %Creset%s %Cgreen(%cr) %C(bold blue)<%an>%Creset" --numstat"
```

1.  现在，我们可以在终端中执行`git ll`命令并获得一个漂亮的统计输出，如下所示：

```
$ git ll
b14a939 (HEAD, master) Prepare 3.3.0-SNAPSHOT builds (8 days ago) <Matthias Sohn>
6       6       org.eclipse.jgit.ant.test/META-INF/MANIFEST.MF
1       1       org.eclipse.jgit.ant.test/pom.xml
3       3       org.eclipse.jgit.ant/META-INF/MANIFEST.MF
1       1       org.eclipse.jgit.ant/pom.xml
4       4       org.eclipse.jgit.archive/META-INF/MANIFEST.MF
2       2       org.eclipse.jgit.archive/META-INF/SOURCE-MANIFEST.MF
1       1       org.eclipse.jgit.archive/pom.xml
6       6       org.eclipse.jgit.console/META-INF/MANIFEST.MF
1       1       org.eclipse.jgit.console/pom.xml
12      12      org.eclipse.jgit.http.server/META-INF/MANIFEST.MF
...
```

1.  也可以使用外部命令代替 Git 命令。因此，小的 Shell 脚本等可以被嵌入。要使用外部命令创建`alias`方法，别名必须以感叹号`!`开头。当解决 rebase 或 merge 冲突时，可以使用这些示例。在`~/.gitconfig`文件的`[alias]`下，添加以下内容：

```
editconflicted = "!f() {git ls-files --unmerged | cut -f2 | sort -u ; }; $EDITOR 'f'"
```

这将调出你配置的`$EDITOR`，并显示由于合并/rebase 而处于冲突状态的所有文件。这使得你可以快速修复冲突并继续合并/rebase。

1.  在`jgit`仓库中，我们可以在较早的时间点创建两个分支并合并这两个分支：

```
$ git branch A 03f78fc
$ git branch B 9891497
$ git checkout A
Switched to branch 'A'

$ git merge B  
```

现在，你会看到这个操作无法执行合并，你可以运行`git st`来检查很多处于冲突状态的文件的状态，`both modified`。要打开并编辑所有冲突文件，我们可以运行`git editconflicted`。这将使用`$EDITOR`打开文件。如果你的环境变量未设置，可以使用`EDITOR=<your-favorite-editor>`来设置它。

在这个示例中，我们实际上并不解决冲突。只需检查别名是否有效，然后准备好下一个别名。

1.  现在，我们已经解决了所有的合并冲突，接下来是添加所有这些文件，在合并结束之前。幸运的是，我们可以创建一个`alias`方法来帮助我们实现，如下所示：

```
addconflicted = "!f() { git ls-files --unmerged | cut -f2 | sort -u ; }; git add 'f'"  
```

1.  现在，我们可以运行`git addconflicted`。稍后，`git status`会告诉我们所有冲突的文件都已添加：

```
$ git st
On branch A
All conflicts fixed but you are still merging.
 (use "git commit" to conclude merge)

Changes to be committed:

 modified:   org.eclipse.jgit.console/META-INF/MANIFEST.MF
 modified:   org.eclipse.jgit.console/pom.xml
 modified:   org.eclipse.jgit.http.server/META-INF/MANIFEST.MF
 modified:   org.eclipse.jgit.http.server/pom.xml
 modified:   org.eclipse.jgit.http.test/META-INF/MANIFEST.MF
 modified:   org.eclipse.jgit.http.test/pom.xml
 ...
Now we can conclude the merge with git commit:
$ git commit
[A 94344ae] Merge branch 'B' into A
```

# 它是如何工作的...

Git 只是运行别名所代表的命令。这对长的 Git 命令或那些难以记住具体如何写的 Git 命令非常方便。现在，你只需要记住别名，并且可以随时查看配置文件来查找它。

# 还有更多...

创建一种 Git 别名的另一种方法是创建一个 shell 脚本，并将文件保存为`git-<你的别名>`。使文件具有可执行权限，并将其放置在你的`$PATH`中。现在，你只需通过从命令行运行`git<你的别名>`即可运行该文件。

# 示例中的 refspec

尽管`refspec`并不是想到 Git 配置时第一个想到的内容，但它实际上是非常接近的。在许多 Git 命令中都会使用`refspec`，但通常是隐式使用的，即`refspec`是从配置文件中获取的。如果你不记得设置过`refspec`配置，可能是对的，但如果你克隆了仓库或添加了远程仓库，那么`.git/config`文件中会有一个类似以下内容的部分（这是针对`jgit`仓库的）：

```
[remote "origin"]
 url = https://git.eclipse.org/r/jgit/jgit
 fetch = +refs/heads/*:refs/remotes/origin/*
```

fetch 行包含与此仓库相关的已配置`refspec`。

# 准备工作

在这个例子中，我们将使用`jgit`仓库作为我们的服务器仓库，但我们需要将其克隆到一个裸仓库中，以便可以推送。你不能推送到非裸仓库的已检出分支，因为这可能会覆盖工作区和索引。

从`jgit`仓库创建一个裸仓库，并创建一个新的 Git 仓库，在其中我们可以操作`refspec`，如下所示：

```
$ git clone --bare https://git.eclipse.org/r/jgit/jgit jgit-bare.git
$ git init refspec-tests
Initialized empty Git repository in /Users/john.doe/refspec-tests/.git/
$ cd refspec-tests
$ git remote add origin ../jgit-bare.git
```

我们还需要更改某些分支的分支名称，以匹配命名空间的示例；以下命令将`stable-xxx`分支重命名为`stable/xxx`：

```
$ for br in $(git branch  -a | grep "stable-"); do new=$(echo $br| sed 's/-///'); git branch $new $br; done
```

在之前的 shell 脚本中，`$new`和`$br`变量没有放在双引号（`"`）中，虽然良好的 shell 脚本实践建议这样做。这是可以的，因为这些变量反映的是仓库中分支的名称，而分支名不能包含空格。

# 如何操作...

1.  让我们设置新的仓库，只获取`master`分支。我们通过在配置文件（`.git/config`）中更改`[remote "origin"]`下的 fetch 行来实现，如下所示：

```
[remote "origin"]
 url = ../jgit-bare.git
  fetch = +refs/heads/master:refs/remotes/origin/master
```

1.  现在，我们在执行`git fetch`、`git pull`或`git remote`更新 origin 时，只会获取`master`分支，而不会获取其他分支，如下所示：

```
$ git pull
remote: Counting objects: 44033, done.
remote: Compressing objects: 100% (6927/6927), done.
remote: Total 44033 (delta 24063), reused 44033 (delta 24063)
Receiving objects: 100% (44033/44033), 9.45 MiB | 5.70 MiB/s, done.
Resolving deltas: 100% (24063/24063), done.
From ../jgit-bare
     * [new branch]      master     -> origin/master
From ../jgit-bare
     * [new tag]         v0.10.1    -> v0.10.1
     * [new tag]         v0.11.1    -> v0.11.1
     * [new tag]         v0.11.3    -> v0.11.3
    ...
$ git branch -a
    * master
      remotes/origin/master
```

1.  我们还可以设置一个单独的 refspec，将所有`stable/*`分支获取到本地仓库，具体如下：

```
[remote "origin"]
 url = ../jgit-bare.git
  fetch = +refs/heads/master:refs/remotes/origin/master
  fetch = +refs/heads/stable/*:refs/remotes/origin/stable/*
```

1.  现在，按如下命令在本地获取分支：

```
$ git fetch
From ../jgit-bare
     * [new branch]      stable/0.10 -> origin/stable/0.10
     * [new branch]      stable/0.11 -> origin/stable/0.11
     * [new branch]      stable/0.12 -> origin/stable/0.12
     * [new branch]      stable/0.7 -> origin/stable/0.7
     * [new branch]      stable/0.8 -> origin/stable/0.8
     * [new branch]      stable/0.9 -> origin/stable/0.9
     * [new branch]      stable/1.0 -> origin/stable/1.0
     * [new branch]      stable/1.1 -> origin/stable/1.1
     * [new branch]      stable/1.2 -> origin/stable/1.2
     * [new branch]      stable/1.3 -> origin/stable/1.3
     * [new branch]      stable/2.0 -> origin/stable/2.0
     * [new branch]      stable/2.1 -> origin/stable/2.1
     * [new branch]      stable/2.2 -> origin/stable/2.2
     * [new branch]      stable/2.3 -> origin/stable/2.3
     * [new branch]      stable/3.0 -> origin/stable/3.0
     * [new branch]      stable/3.1 -> origin/stable/3.1
     * [new branch]      stable/3.2 -> origin/stable/3.2
```

1.  我们还可以设置一个推送（push）`refspec`，指定默认推送到哪个分支。让我们创建一个名为`develop`的分支并提交一个更改，如以下命令所示：

```
$ git checkout -b develop
Switched to a new branch 'develop'
$ echo "This is the developer setup, read carefully" > readme-dev.txt
$ git add readme-dev.txt
```

```
$ git commit -m "adds readme file for developers"
[develop ccb2f08] adds readme file for developers
 1 file changed, 1 insertion(+)
 create mode 100644 readme-dev.txt
```

1.  现在，让我们创建一个推送（push）`refspec`，将`develop`分支的内容推送到远程`integration/master`：

```
[remote "origin"]
 url = ../jgit-bare.git
  fetch = +refs/heads/master:refs/remotes/origin/master
  fetch = +refs/heads/stable/*:refs/remotes/origin/stable/*
  push = refs/heads/develop:refs/remotes/origin/integration/master
```

1.  让我们将我们的提交推送到`develop`分支，如下所示：

```
$ git push
Counting objects: 4, done.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 345 bytes | 0 bytes/s, done.
Total 3 (delta 1), reused 0 (delta 0)
To ../jgit-bare.git
* [new branch]      develop -> origin/integration/master
```

由于`integration/master`分支在远程端不存在，因此它为我们创建了该分支。

# 它是如何工作的...

`refspec`的格式为`<source>:<destination>`。对于拉取（fetch）`refspec`，这意味着`<source>`是远程端的源，而`<destination>`是本地的。对于推送（push）`refspec`，`<source>`是本地的，`<destination>`是远程的。`refspec`可以通过`+`前缀来表示即使不是快进更新，`ref`模式也可以被更新。`refspec`模式中不能使用部分通配符，如以下行所示：

```
fetch = +refs/heads/stable*:refs/remotes/origin/stable*
```

然而，使用命名空间是可能的。这就是为什么我们必须将`stable-xxx`分支重写为`stable/xxx`以符合命名空间模式的原因：

```
fetch = +refs/heads/stable/*:refs/remotes/origin/stable/*
```
