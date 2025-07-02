# 使用 Git 钩子、别名和脚本增强你的日常工作

在本章中，我们将涵盖以下内容：

+   在提交信息中使用分支描述

+   创建动态提交信息模板

+   在提交信息中使用外部信息

+   防止推送特定的提交

+   配置和使用 Git 别名

+   配置和使用 Git 脚本

+   设置和使用提交模板

# 介绍

为了在企业环境中高效工作，关于生产的任何代码都有一些前提条件或规则。代码应该能够编译并通过特定的单元测试集。此外，提交信息中还应该包含某些文档内容，例如修复 ID 或实例的引用。这些规则中的大多数可以通过脚本进行自动化。但为什么不把这些规则纳入到流程中呢？在本章中，你将看到一些示例，展示如何在看到提交信息之前将数据从一个位置传输到提交信息中。你还将学习如何验证你是否将代码推送到正确的位置。最后，你将学习如何将脚本添加到 Git 中。

Git 中的钩子是一个在特定事件（如推送、提交或变基）触发时执行的脚本。如果这些脚本以非零值退出，最好取消当前的 Git 操作。你可以在任何 Git 克隆的 `.git/hooks` 文件夹中找到这些钩子脚本。如果它们的文件扩展名是 `.sample`，则表示这些钩子是非激活状态。

# 在提交信息中使用分支描述

在第三章《*分支、合并与选项*》中，我们提到过你可以为你的分支设置描述，并且可以通过 `git config --get branch.<branchname> description` 命令从脚本中获取此信息。在这个例子中，我们将提取这些信息并将其用于提交信息。

我们将使用 `prepare-commit-msg` 钩子。`prepare-commit-msg` 钩子会在每次你想要提交时执行，钩子可以设置为在你实际看到提交信息编辑器之前检查任何你想要检查的内容。

# 准备工作

我们需要一个克隆和一个分支来开始这个练习，因此我们将再次将 `jgit` 克隆到 `chapter7.5` 文件夹中，如下所示：

```
$ git clone https://git.eclipse.org/r/jgit/jgit chapter7.5
Cloning into 'chapter7.5'...
remote: Counting objects: 2170, done
remote: Finding sources: 100% (364/364)
remote: Total 45977 (delta 87), reused 45906 (delta 87)
Receiving objects: 100% (45977/45977), 10.60 MiB | 1.74 MiB/s, done.
Resolving deltas: 100% (24651/24651), done.
Checking connectivity... done.
Checking out files: 100% (1577/1577), done.
```

检出一个本地的 `descriptioInCommit` 分支，该分支跟踪 `origin/stable-3.2` 分支：

```
$ cd chapter7.5
$ git checkout -b descriptioInCommit  --track origin/stable-3.2
Branch descriptioInCommit set up to track remote branch stable-3.2 from origin.
Switched to a new branch 'descriptioInCommit'
```

# 如何操作...

我们将从设置本地分支的描述开始。然后，我们将创建一个钩子来提取此信息并将其放入提交信息中。

我们有本地的 `descriptioInCommit` 分支，我们需要为其设置描述。我们将使用 `--edit-description` Git 分支命令为本地分支添加描述。这样会打开描述编辑器，你可以通过以下步骤输入消息：

1.  当你执行命令时，描述编辑器将打开，你可以输入消息：

```
$ git branch --edit-description descriptioInCommit
```

1.  现在，输入以下消息：

```
Remote agent not connection to server

When the remote agent is trying to connect
it will fail as network services are not up
and running when remote agent tries the first time
```

1.  你应该像编写提交信息一样编写你的分支描述。然后，将描述重复使用在提交信息中是有意义的。现在，我们将验证是否有以下描述的信息：

```
$ git config --get branch.descriptioInCommit.description
Remote agent not connection to server

When the remote agent is trying to connect
it will fail as network services are not up
and running when remote agent tries the first time
```

1.  如预期的那样，我们得到了所需的输出。现在，我们可以继续创建将获取描述并使用它的钩子。

接下来，我们将检查是否有钩子的描述，如果有，我们将使用该描述作为提交信息。

1.  首先，我们将确保能够在期望的位置获取提交信息。实现这一点有多种方法，我们选择了以下方法：打开`.git/hook/prepare-commit-msg`钩子文件，输入以下脚本，并使其可执行（`chmod +x`）：

```
#!/bin/bash
BRANCH=$(git branch | grep '*'| sed 's/*//g'|  sed 's/ //g')
DESCRIPTION=$(git config --get branch.${BRANCH}.description)
if [ -z "$DESCRIPTION" ]; then
 echo "No desc for branch using default template"
else
 # replacing # with n
 DESCRIPTION=$(echo "$DESCRIPTION" | sed 's/#/\n/g')
 # replacing the first \n with \n\n
 DESCRIPTION=$(echo "$DESCRIPTION" | sed 's/\n/\n\n/')
# append default commit message
DESCRIPTION=$(echo "$DESCRIPTION" && cat $1)
# and write it all to the commit message
echo "$DESCRIPTION" > $1
fi
```

1.  现在，我们可以尝试创建一个提交，看看提交信息是否按预期显示。使用`git commit --allow-empty`生成一个空提交，同时触发 prepare-commit-msg 钩子：

```
$ git commit --allow-empty
```

1.  你应该会看到一个带有我们分支描述作为提交信息的编辑器，如下所示：

```
Remote agent not connection to server

When the remote agent is trying to connect
it will fail as network services are not up
and running when remote agent tries the first time

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
# On branch descriptioInCommit
# Your branch is up-to-date with 'origin/stable-3.2'.
#
# Untracked files:
#       hen the remote agent is trying to connect
#
```

1.  这正如我们所预期的那样。保存提交信息并关闭编辑器。尝试使用`git log -1`命令来验证我们是否在提交中有以下信息：

```
$ git log -1
commit 92447c6aac2f6d675f8aa4cb88e5abdfa46c90b0
Author: John Doe <john.doe@example.com>
Date:   Sat Mar 15 00:19:35 2014 +0100

   Remote agent not connection to server
```

```
   When the remote agent is trying to connect
   it will fail as network services are not up
   and running when remote agent tries the first time
```

1.  你应该会得到类似的提交信息，内容与我们的分支描述相同。不过，如果分支描述为空呢？我们的钩子会如何处理？我们可以尝试创建一个名为`noDescriptionBranch`的新分支。使用`git checkout`创建它，并按以下命令检查：

```
$ git checkout -b noDescriptionBranch
Switched to a new branch 'noDescriptionBranch'
```

1.  现在，我们将再创建一个空提交，以查看提交信息是否如下所示：

```
$ git commit --allow-empty
```

1.  你应该会看到带有默认提交信息文本的提交信息编辑器，如下所示：

```
# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit. #
# On branch noDescriptionBranch
```

一切都如我们预期的那样。这个脚本可以与下一个练习结合使用，后者将从一个有缺陷的系统中提取内容。

# 创建动态提交信息模板

可以鼓励开发人员做正确的事，或者可以强迫开发人员做正确的事。然而，最终，开发人员需要花时间进行编码。因此，如果需要良好的提交信息，我们可以使用`prepare-commit-msg`钩子来协助开发人员。

在这个例子中，我们将为开发人员创建一个包含工作区状态信息的提交信息。它还会插入一些来自网页的信息。这些信息也可以是 Bugzilla 中的缺陷信息。

# 准备就绪

为了开始这个练习，我们不会克隆一个仓库，而是创建一个新的仓库。为此，我们将使用`git init`，如以下代码所示。你可以使用`git init <directory>`在某个地方创建一个新仓库，或者你可以进入一个目录并执行`git init`，Git 会为你创建一个仓库。

```
$ git init chapter7
Initialized empty Git repository in /Users/JohnDoe/repos/chapter7/.git/
$ cd chapter7
```

# 如何做到这一点...

我们有我们的`chapter7`目录，在这里我们刚刚初始化了我们的仓库。在此目录中，钩子已经可用。只需查看`.git/hooks`目录即可。我们将使用`prepare-commit-msg`钩子。执行以下步骤：

1.  从以下钩子文件夹开始查找：

```
$ ls .git/hooks/
applypatch-msg.sample  pre-applypatch.sample 
pre-rebase.sample      commit-msg.sample
pre-commit.sample      prepare-commit-msg.sample
post-update.sample     pre-push.sample 
update.sample
```

1.  如您所见，每个钩子文件中都有很多钩子。这里有一个示例脚本，并简要说明了钩子做什么以及何时执行。要启用`prepare-commit-msg`，请按以下代码所示重命名文件：

```
$ cd .git/hooks/
$ mv prepare-commit-msg.sample prepare-commit-msg
$ cd -
```

1.  在您喜欢的编辑器中打开`prepare-commit-msg`文件。

1.  您可以查看文件中的信息，但对于我们的示例，我们将清空文件，以便可以包括脚本。

1.  现在，在文件中包含以下命令：

```
#!/bin/bash 
echo "I refuse to commit"
exit 1
```

1.  保存文件。

1.  最后，尝试提交某些内容或不提交内容。通常，您不能提交空的内容，但使用`--allow-empty`选项，您可以创建一个空的提交，如下所示：

```
$ git commit --allow-empty
I refuse to commit
```

1.  正如您所见，我们得到了在`prepare-commit-msg`脚本文件中输入的消息。您可以使用`git log -1`命令检查我们是否有提交，方法如下：

```
$ git log -1
fatal: your current branch 'master' does not have any commits yet
```

没有提交，我们收到了一个我们以前没有见过的错误消息。消息必须存在，因为到目前为止在这个仓库中还没有提交。在我们进一步更改脚本之前，我们应该知道`prepare-commit-msg`钩子会根据情况接收一些参数。第一个参数始终是`.git/COMMIT_EDITMSG`，第二个参数可以是 merge、commit、squash 或 template，具体取决于情况。我们可以在脚本中使用这些参数。

1.  更改脚本，以便我们可以拒绝修改提交，如下所示：

```
#!/bin/bash
if [ "$2" == "commit" ]; then 
  echo "Not allowed to amend"
  exit 1
fi
```

1.  现在我们已经更改了脚本，让我们创建一个提交并尝试修改它，如下所示：

```
$ echo "alot of fish" > fishtank.txt
$ git add fishtank.txt
$ git commit -m "All my fishes are belong to us"
[master (root-commit) f605886] All my fishes are belong to us
1 file changed, 1 insertion(+)
create mode 100644 fishtank.txt
```

1.  现在我们已经有了提交，让我们尝试使用`git commit --amend`来修改它：

```
$ git commit --amend
Not allowed to amend
```

1.  正如我们所预期的那样，我们没有被允许修改提交。如果我们希望提取一些信息，例如从错误处理系统中提取，我们必须在打开编辑器之前将这些信息放入文件中。所以，我们将再次更改脚本，如下所示：

```
#!/bin/bash
if [ "$2" == "commit" ]; then 
  echo "Not allowed to amend"
  exit 1
fi
MESSAGE=$(curl -s http://whatthecommit.com/index.txt)
echo "$MESSAGE" > $1
```

1.  这个脚本从`http://www.whatthecommit.com/`下载一个提交消息并将其插入到提交信息中。每次提交时，您都会从网页上获取一条新的消息。让我们使用以下命令试一下：

```
$ echo "gravel, plants, and food" >>fishtank.txt
$ git add fishtank.txt
$ git commit
```

1.  当提交信息编辑器打开时，您应该看到来自`whatthecommit.com`的消息。关闭编辑器后，使用`git log -1`命令验证我们是否已经有了提交，方法如下：

```
$ git log -1
commit c087f75665bf516af2fe30ef7d8ed1b775bcb97d
Author: John Doe <john.doe@example.com>
Date:   Wed Mar 5 21:12:13 2014 +0100

   640K ought to be enough for anybody
```

1.  正如预期的那样，我们已经成功完成了提交。显然，这不是为提交者准备的最佳消息。更典型的做法是在提交信息中列出分配给开发者的错误，如下所示：

```
# You have the following artifacts assigned
# Remove the # to add the artifact ID to the commit message

#[artf23456] Error 02 when using update handler on wlan
#[artf43567] Enable Unicode characters for usernames
#[artf23451] Use stars instead of & when keying pword
```

1.  这样，开发者可以轻松地从 TeamForge 中选择正确的错误 ID，或者在这种情况下，使用其他系统查看提交信息时所需的正确格式的工件 ID。

# 还有更多内容...

你可以轻松扩展 `prepare-commit-msg` 钩子的功能，但你应该记住，获取一些信息的等待时间应该是值得的。一个通常很容易检查的事情是工作区是否有修改。

在这里，我们需要在准备提交信息的钩子中使用 `git status` 命令，并且我们需要预测提交后是否会有修改的文件：

1.  要检查这一点，我们需要有一些已暂存的更改和一些未暂存的更改，如下所示：

```
$ git status
On branch master
nothing to commit, working directory clean
```

1.  现在，修改 `fishtank.txt` 文件：

```
$ echo "saltwater" >> fishtank.txt
```

1.  使用 `git status --porcelain` 检查工作区：

```
$ git status --porcelain
M fishtank.txt
```

1.  使用 `git add` 将文件添加到暂存区：

```
$ git add fishtank.txt
```

1.  现在尝试 `git status --porcelain`：

```
$ git status --porcelain
M  fishtank.txt
```

1.  你需要注意的是，第一次使用 `--porcelain` 选项查看 Git 状态时，`M` 前面有一个空格。`porcelain` 选项提供了机器友好的输出，显示 Git 状态下文件的状态。第一个字符表示暂存区的状态，而第二个字符表示工作区的状态。因此，`MM fishtank.txt` 表示该文件在工作区和暂存区都有修改。所以，如果你再次修改 `fishtank.txt`，你可以预期如下结果：

```
$ echo "sharks and oysters" >> fishtank.txt
$ git status --porcelain
MM fishtank.txt
```

1.  如预期的那样，Git 状态的输出为 `MM fishtank.txt`。我们可以在钩子中使用这个输出，来判断提交后工作区是否会有未提交的更改。将以下命令添加到 `prepare-commit-msg` 文件中：

```
for file in $(git status --porcelain)
do
 if [ ${file:1:1} ]; then 
    DIRTY=1
  fi
  done
  if [ "${DIRTY}" ]; then
    # -i '' is not needed on Linux 
    sed -i '' "s/# Please/You have a dirty workarea are you sure you wish to commit ?&/" $1
  fi
```

1.  首先，我们使用 `git status --porcelain` 列出所有已更改的文件。然后，对于每个文件，我们检查是否有第二个字符。如果有第二个字符，那么提交后工作区就会有修改。最后，我们将信息插入到提交信息中，以便开发人员查看。让我们尝试并通过以下命令提交更改：

```
$ git commit
```

1.  检查是否有类似以下内容的消息。第一行可能会有所不同，因为我们仍然有来自 `http://www.whatthecommit.com/` 的消息：

```
somebody keeps erasing my changes.
You have a dirty workarea are you sure you wish to commit ?
# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
# On branch master
# Changes to be committed:
#       modified:   fishtank.txt
# 
# Changes not staged for commit:
#       modified:   fishtank.txt
#
```

1.  保存文件并关闭编辑器将创建提交。使用 `git log -1` 验证此操作，如下所示：

```
$ git log -1
commit 70cad5f7a2c3f6a8a4781da9c7bb21b87886b462
Author: John Doe <john.doe@example.com>
Date:   Thu Mar 6 08:25:21 2014 +0100

    somebody keeps erasing my changes.
    You have a dirty workarea are you sure you wish to commit ?
```

1.  我们得到了预期的信息。有关脏工作区的文本已出现在提交信息中。为了在下一次练习前做一个干净的清理，我们应该将工作区重置为 `HEAD`，如下所示：

```
$ git reset --hard HEAD
HEAD is now at 70cad5f somebody keeps erasing my changes.
```

现在，只需找出什么最适合你。在提交并可能推送代码到远程分支之前，你是否希望检查任何信息？这可能包括：

+   代码中的样式检查

+   使用 Pylint 检查你的 Python 脚本

+   检查是否有不允许添加到 Git 的文件

这个列表并不详尽；对于世界上每个组织或开发团队，可能还有其他需要添加的内容。然而，这显然是一种方法，可以减少开发者繁琐的手动工作，让他们能够专注于编码。

# 在提交信息中使用外部信息

提交钩子在你关闭提交信息编辑器时执行。它可以用于操作提交信息或自动审核提交信息，以检查其是否具有特定的格式。

在这个示例中，我们将操作并检查提交信息的内容。

# 准备就绪

为了开始这个练习，我们只需要创建一个分支并切换到它。我们需要禁用当前的`prepare-commit-msg`钩子；可以通过简单地重命名它来实现。现在，我们可以通过以下命令开始处理`commit-msg`钩子：

```
$ git checkout -b commit-msg-example
Switched to a new branch 'commit-msg-example'
$ mv .git/hooks/prepare-commit-msg .git/hooks/prepare-commit-msg.example
```

# 如何操作...

在第一个示例中，我们要做的是检查缺陷信息是否正确。无需发布引用不存在的缺陷的提交：

1.  我们将从测试`commit-msg`钩子开始。首先，复制当前的钩子文件，然后我们将强制使钩子以非零值退出，从而中止提交的创建：

```
$ cp .git/hooks/commit-msg.sample .git/hooks/commit-msg
```

1.  现在，在你喜欢的编辑器中打开文件，并将以下行添加到文件中：

```
#!/bin/bash
echo "you are not allowed to commit"
exit 1
```

1.  现在，我们将尝试进行一次提交，看看会发生什么，具体如下：

```
$ echo "Frogs, scallops, and coco shell" >> fishtank.txt
$ git add fishtank.txt
$ git commit
```

1.  编辑器将打开，你可以写一个简短的提交信息，然后关闭编辑器。你应该会看到`you are not allowed to commit`的消息，如果你使用`git log -1`检查，你会发现没有你刚才写的提交信息，具体如下：

```
you are not allowed to commit
$ git log -1
commit 70cad5f7a2c3f6a8a4781da9c7bb21b87886b462
```

```
Author: John Doe <john.doe@example.com>
Date:   Thu Mar 6 08:25:21 2014 +0100

    somebody keeps erasing my changes.
You have a dirty workarea are you sure you wish to commit ?
```

1.  如你所见，提交信息钩子在你关闭消息编辑器后执行，而`prepare-commit-msg`钩子在消息编辑器之前执行。为了验证，如果我们在提交信息中有对钩子的正确引用，我们将检查 Jenkins-CI 项目是否有特定的错误。将`commit-msg`钩子中的行替换成以下命令：

```
#!/bin/bash
JIRA_ID=$(cat $1 | grep jenkins | sed 's/jenkins //g')
ISSUE_INFO=$(curl -g "https://issues.jenkins-ci.org/browse/JENKINS-${JIRA_ID}")
if [ -z "${ISSUE_INFO}" ]; then 
  echo "Jenkins issue ${JIRA_ID} does not exist"
  echo "Please try again"
  exit 1
else
  TITLE=$(curl -g "https://issues.jenkins-ci.org/browse/JENKINS-$JIRA_ID}" | grep -E "<title>.*</title>")
  echo "Jenkins issue ${JIRA_ID}"
  echo "${TITLE}"
  exit 0
fi
```

1.  我们使用 curl 来检索网页，如果网页为空，我们就知道该 ID 不存在。现在，我们应该创建一个提交，看看如果我们输入错误的 ID（如`jenkins 384895`）或者一个存在的 ID（如`jenkins 3157`）会发生什么。为此，我们将按如下方式创建提交：

```
$ echo "more water" >> fishtank.txt
$ git add fishtank.txt
$ git commit
```

1.  在提交信息中，写入类似`Feature cascading...`的提交信息标题。然后，在提交信息的正文中插入`jenkins 384895`。这是关键部分，因为钩子将使用该号码在 Jenkins 问题追踪器中查找：

```
Feature: Cascading...

jenkins 384895
```

1.  你应该得到以下输出：

```
Jenkins issue 384895 does not exist
Please try again
```

1.  这是我们预期的结果。现在，使用`git status`验证更改是否已经提交：

```
$ git status
On branch commit-msg-example
Changes to be committed:
(use "git reset HEAD <file>..." to unstage)

  modified:   fishtank.txt
```

1.  现在，我们将再次尝试提交；这次，我们将使用正确的 JIRA ID：

```
$ git commit
```

1.  输入一个像之前那样的提交信息；这次，确保 Jenkins 问题 ID 是存在的。你可以使用`51444`：

```
Feature: Cascading...

jenkins 51444
```

1.  保存提交信息后，应该得到如下输出。我们可以通过去除标题 HTML 标签进一步清理它：

```
<title>[#JENKINS-51444] Maven Parser creates errors during affectedFilesResolving - Jenkins JIRA</title>
[commit-msg-example 3d39ca3] Feature: Cascading...
1 file changed, 2 insertions(+)
```

1.  如你所见，我们可以获取信息并输出。我们也可以将这些信息添加到提交信息中。然后，我们可以更改并将其作为`else`分支插入脚本：

```
TITLE=$(curl https://issues.jenkins-ci.org/browse/JENKINS-${JIRA_ID} | grep -E "<title>.*</title>")
TITLE=$(echo ${TITLE} | sed 's/^<title>//' | sed 's/<\/title>$//')
echo "${TITLE}" >> $1
echo "Jenkins issue ${JIRA_ID}"
echo "${TITLE}"
exit 0
```

1.  为了测试这一点，我们将再次创建一个提交，并且在信息中需要指定存在的 JIRA ID：

```
$ echo "Shrimps and mosquitos" >> fishtank.txt
$ git add fishtank.txt
$ git commit
After saving the commit message editor you will get an output similar like this. 
Jenkins issue 51444
[JENKINS-51444] Maven Parser creates errors during affectedFilesResolving - Jenkins JIRA
[commit-msg-example 6fa2cb4] Feature: Cascading...
1 file changed, 1 insertion(+)
```

1.  为了验证我们是否在信息中得到了所需的内容，我们将再次使用`git log -1`：

```
$ git log -1
commit 6fa2cb47989e12b05cd2689aa92244cb244426fc
Author: John Doe <john.doe@example.com>
Date:   Thu Mar 6 09:46:18 2014 +0100

    Feature: Cascading...

    jenkins 51444
    [#JENKINS-51444] Maven Parser creates errors during affectedFilesResolving - Jenkins JIRA
```

正如预期的那样，我们在提交的末尾得到了信息。在这些示例中，如果 JIRA ID 不存在，我们将丢弃提交信息。这对开发者来说有点苛刻。所以，你可以将它与`prepare-commit-msg`钩子结合使用。如果`commit-msg`停止提交过程，那么临时保存该信息，以便在开发者再次尝试时，`prepare-commit-msg`钩子可以使用这个信息。

# 防止特定提交的推送

预推送钩子会在使用推送命令时触发，并且脚本执行发生在推送之前。因此，我们可以在发现拒绝推送的原因时阻止推送。

其中一个原因可能是你在提交信息中有`nopush`文本。

# 准备就绪

要使用 Git 的预推送钩子，我们需要有一个远程仓库。我们将再次克隆`jgit`，如以下所示：

```
$ git clone https://git.eclipse.org/r/jgit/jgit chapter7.1
Cloning into 'chapter7.1'...
  remote: Counting objects: 2429, done
  remote: Finding sources: 100% (534/534)
  remote: Total 45639 (delta 145), reused 45578 (delta 145)
  Receiving objects: 100% (45639/45639), 10.44 MiB | 2.07 MiB/s, done.
  Resolving deltas: 100% (24528/24528), done.
  Checking connectivity... done.
  Checking out files: 100% (1576/1576), done.
```

# 如何实现...

我们希望能够推送到远程分支，但不幸的是，Git 会在执行钩子之前通过 HTTPS 尝试对`jgit`仓库进行身份验证。因此，我们将从`chapter7.1`目录创建一个本地克隆，如下所示。这将使我们的远程变为本地文件夹：

```
$ git clone --branch master ./chapter7.1/ chapter7.2
Cloning into ' chapter7.2'...
done.
Checking out files: 100% (1576/1576), done.
$ cd chapter7.2 $ git branch
* master 
```

我们正在将`chapter7.1`目录克隆到名为`chapter7.2`的文件夹中，克隆完成后将检查`master`分支。

我们现在想做的是创建一个提交，提交信息中包含`nopush`。通过在提交信息中添加这个词，钩子中的代码将自动停止推送。我们将在一个分支上进行此操作。所以，首先，你应该检出一个`prepushHook`分支，该分支跟踪`origin/master`分支，然后创建一个提交。

当我们设置好预推送提交时，我们将尝试将其推送到远程，具体如下：

1.  从创建一个名为`prepushHook`的新分支开始，该分支跟踪`origin/master`：

```
$ git checkout -b prepushHook  --track origin/master
Branch prepushHook set up to track remote branch master from origin.
Switched to a new branch 'prepushHook'
```

1.  现在，我们使用`reset`回到一个较早的提交。这并不重要我们回到多远。我们只是选择了一个随机的提交，如下所示：

```
$ git reset --hard 2e0d178
HEAD is now at 2e0d178 Add recursive variant of Config.getNames() methods
```

1.  现在我们可以创建一个提交。我们将使用`sed`进行简单的内联替换，然后添加`pom.xml`并提交：

```
$ sed -i '' 's/2.9.1/3.0.0/g' pom.xml
$ git add pom.xml
$ git commit -m "Please nopush"
[prepushHook 69d571e] Please nopush
1 file changed, 1 insertion(+), 1 deletion(-)
```

1.  要验证我们是否有包含文本的提交，可以运行`git log -1`：

```
$ git log -1
commit 1269d14fe0c32971ea33c95126a69ba6c0d52bbf
Author: John Doe <john.doe@example.com>
Date:   Thu Mar 6 23:07:54 2014 +0100

   Please nopush
```

1.  我们在提交信息中得到了所需的内容。现在，我们只需要准备钩子。我们将从复制示例钩子开始，重命名为实际名称，以便它会在推送时执行：

```
$ cp .git/hooks/pre-push.sample .git/hooks/pre-push
```

1.  编辑钩子，使其代码如以下代码片段所示：

```
#!/bin/bash
echo "You are not allowed to push"
exit 1
```

1.  现在我们准备推送了。我们将把当前分支`HEAD`推送到远程的`master`分支：

```
$ git push origin HEAD:refs/heads/master
You are not allowed to push
error: failed to push some refs to '../chapter7.1/'
```

1.  正如预期的那样，钩子正在执行，推送被钩子拒绝。现在，我们可以实现我们想要进行的检查。如果我们在任何提交信息中有`nopush`这个词，我们希望退出。我们可以使用`git log --grep`来搜索提交信息中包含`nopush`关键词的提交，如下所示的命令：

```
$ git log --grep "nopush"
commit 51201284a618c2def690c9358a07c1c27bba22d5
Author: John Doe <john.doe@example.com>
Date:   Thu Mar 6 23:07:54 2014 +0100

    Please nopush
```

1.  我们已经创建了带有`nopush`关键词的新提交。现在，我们将在钩子中执行一个简单的检查，并编辑 pre-push 钩子，使其包含以下内容：

```
#!/bin/bash
COMMITS=$(git log --grep "nopush")
if [ "$COMMITS" ]; then 
  echo "You have commit(s) with nopush message"
  echo "aborting push"
  exit 1
fi
```

1.  现在，我们可以再次尝试推送，看看结果会是什么。我们将尝试将我们的`HEAD`推送到远程`origin`的主分支：

```
$ git push origin HEAD:refs/heads/master
You have commit(s) with nopush message
aborting push
error: failed to push some refs to '/Users/JohnDoe/repos/./chapter7.1/'
```

正如预期的那样，由于我们在提交中有`nopush`信息，系统不允许我们推送。

# 还有更多...

拥有一个钩子来防止你推送不想推送的提交非常方便。你可以指定任何你想要的关键词。诸如`reword`、`temp`、`nopush`、`temporary`或`hack`等词语都可以是你希望停止的内容，但有时你可能还是想把它们推送出去。

你可以做的是有一个小检查器，检查特定的词，然后列出提交，并询问你是否仍然想要推送。

如果你将脚本更改为以下片段，钩子将尝试找到包含`nopush`关键词的提交并列出它们。如果你希望无论如何推送它们，你可以回答问题，Git 将继续推送：

```
#!/bin/bash
COMMITS=$(git log --grep "nopush" --format=format:%H)
if [ "$COMMITS" ]; then
  exitmaybe=1
fi
if [ $exitmaybe -eq 1 ]; then
while true
do
  clear
for commit in $COMMITS
do
  echo "$commit has no push in the message"
done
   echo "Are you sure you want to push the commit(s) "
    read -r REPLY <&1
    case $REPLY in
    [Yy]* ) break;;
    [Nn]* ) exit 1;;
  * ) echo "Please answer yes or no.";;
esac
done
fi
```

再次尝试使用`git push`命令，如下所示：

```
$ git push origin HEAD:refs/heads/master
Commit 70fea355bac0c65fd51f4874d75e65b4a29ad763 has nopush in message
Are you sure you want to push the commit(s)
```

输入`n`并按*Enter*。然后，预期推送将被中止，并显示以下信息：

```
error: failed to push some refs to '/Users/JohnDoe/repos/./chapter7.1/'
```

正如预期的那样，它不会推送。但是，如果你按下 y，Git 将推送到远程。现在使用以下命令尝试一下：

```
$ git push origin HEAD:refs/heads/master
054c5f78fdc82141e9d73e6b6955c38ff79c8b2e has no push in the message
Are you sure you want to push the commit(s)
y
To /Users/JohnDoe/repos/./chapter7.1/
! [rejected]        HEAD -> master (non-fast-forward)
error: failed to push some refs to 'c:/Users/Rasmus/repos/./chapter7.1/'
hint: Updates were rejected because a pushed branch tip is behind its remote
hint: counterpart. Check out this branch and integrate the remote changes
hint: (e.g. 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```

正如预期的那样，推送将被尝试，但正如你从输出中看到的，它被远程拒绝了。这是因为我们产生了分歧，推送在主分支的最新提交上不起作用。

因此，通过这个钩子，你可以让生活更轻松些，防止不小心推送你不希望推送的内容。这个示例也考虑了已经发布的提交；因此，如果你选择一个不同的关键词，那么其他提交——不仅仅是本地创建的——也会被脚本考虑进去。

# 配置和使用 Git 别名

Git 别名，像 Unix 别名一样，是可以在全局或每个仓库中配置的简短命令。它是一种简单的方式来重命名一些 Git 命令，以使用简短的缩写，例如，`git checkout`可以是`git co`，以此类推。

# 如何操作...

创建别名非常简单直接。你只需要使用`git config`进行配置。

我们将做的是检查一个分支，然后逐一创建它的别名并执行它们，通过执行以下步骤来查看它们的输出：

1.  因此，我们将从检查一个名为`gitAlias`的分支开始，该分支跟踪

    `origin/stable-3.2`分支：

```
$ git checkout -b gitAlias --track origin/stable-3.2
Branch gitAlias set up to track remote branch stable-3.2 from origin.
Switched to a new branch 'gitAlias'
```

1.  完成此操作后，我们可以开始创建一些别名。我们将从以下别名开始，它只会简单地修改你的提交：

```
$ git config alias.amm 'commit --amend'
```

1.  执行这个别名将会打开提交信息编辑器，里面有来自`HEAD`提交的以下信息：

```
$ git amm
Prepare post 3.2.0 builds

Change-Id: Ie2bfdee0c492e3d61d92acb04c5bef641f5f132f
Signed-off-by: Matthias Sohn matthias.sohn@sap.com
```

1.  如你所见，使用 Git 别名可以非常简单地加速你日常工作流程的处理。以下命令将只作用于最后 10 次提交，使用`git log`的`--oneline`选项：

```
$ git config alias.lline 'log --oneline -10'
```

1.  使用别名将会得到以下输出：

```
$ git lline
314a19a Prepare post 3.2.0 builds
699900c JGit v3.2.0.201312181205-r
0ff691c Revert "Fix for core.autocrlf=input resulting in mo
1def0a1 Fix for core.autocrlf=input resulting in modified f
0ce61ca Canonicalize worktree path in BaseRepositoryBuilder
be7942f Add missing @since tags for new public methods ig
ea04d23 Don't use API exception in RebaseTodoLine
3a063a0 Merge "Fix aborting rebase with detached head" into 
e90438c Fix aborting rebase with detached head
2e0d178 Add recursive variant of Config.getNames() methods
```

1.  你也可以执行一个简单的 checkout。这样，你可以使用`git co <branch>`来代替 Git 的 checkout。按照如下方式进行配置：

```
$ git config alias.co checkout
```

1.  你会看到，别名也像普通的 Git 命令一样接受参数。让我们使用以下命令来试试这个别名：

```
$ git co master
Switched to branch 'master'
Your branch is up-to-date with 'origin/master'.
$ git co gitAlias
Switched to branch 'gitAlias'
Your branch and 'origin/stable-3.2' have diverged,
and have 1 and 1 different commit each, respectively.
(use "git pull" to merge the remote branch into yours)
```

1.  命令按预期工作。你可能会好奇为什么在再次检出`gitAlias`分支后我们发生了分叉。然后，当我们修改`HEAD`提交时，我们发生了分叉。下一个别名是创建一个包含工作区中所有未提交内容的提交，除了未跟踪的文件：

```
$ git config alias.ca 'commit -a -m "Quick commit"'
```

1.  在我们测试这个别名之前，我们应该创建一个文件并修改它，以展示它的实际作用。你可以按下面的命令创建一个文件：

```
$ echo "Sharks" > aquarium
$ echo "New HEADERTEXT" > pom.xml
```

1.  要验证你想要的内容，运行`git status`：

```
Changes not staged for commit:
(use "git add <file>..." to update what will be committed)
(use "git checkout -- <file>..." to discard changes in working directory)

 modified:   pom.xml

  Untracked files:
      (use "git add <file>..." to include in what will be committed)

      aquarium

no changes added to commit (use "git add" and/or "git commit -a")
```

1.  现在，我们可以使用以下命令来测试这个别名：

```
$ git ca
[gitAlias ef9739d] Quick commit
1 file changed, 1 insertion(+), 606 deletions(-)
rewrite pom.xml (100%)
```

1.  要验证`aquarium`文件是否是提交的一部分，使用`git status`：

```
Untracked files:
  (use "git add <file>..." to include in what will be committed)

  aquarium

  nothing added to commit but untracked files present (use "git add" to track)
```

1.  你还可以使用`git log -1`来查看我们刚刚创建的提交：

```
$ git log -1
commit ef9739d0bffe354c75b82f3b785780f5e3832776
Author: John Doe <john.doe@example.com>
Date:   Thu Mar 13 00:01:49 2014 +0100

    Quick commit
```

1.  输出正如我们所预期的那样。下一个别名稍有不同，因为它将计算仓库中的提交次数，可以使用`wc`（`wordcount`）工具来完成此操作。然而，由于这不是一个内置的 Git 工具，我们必须使用感叹号并指定 Git：

```
$ git config alias.count '!git log --all --oneline | wc -l'
```

1.  让我们试试下面的命令：

```
$ git count
    3008
```

1.  目前，仓库中有`3008`个提交。这也意味着你可以像使用 Git 工具一样，通过创建 Git 别名来执行外部工具；例如，如果你正在使用 Windows、Mac 或 Linux，你可以按如下方式创建一个别名：

```
$ git config alias.wa '!explorer .' # Windows
$ git config alias.wa '!open .' # MacOS
$ git config alias.wa '!xdg-open .' # Linux
```

1.  这个别名将会打开你当前所在路径的文件资源管理器。下一个别名展示了`HEAD`提交中发生了什么变更。它使用`git log`的`--name-status`选项来执行：

```
$ git config alias.gl1 'log -1 --name-status'
```

1.  现在，试试下面的命令：

```
$ git gl1
commit ef9739d0bffe354c75b82f3b785780f5e3832776
Author: John Doe <john.doe@example.com>
Date:   Thu Mar 13 00:01:49 2014 +0100

 Quick commit

    M       pom.xml
```

1.  如你所见，它只是简单地列出了提交和文件，包括文件在提交中的变动。由于别名接受参数，我们实际上可以重复利用这个功能来列出另一个分支的信息。让我们试试下面的命令：

```
$ git gl1 origin/stable-2.1
commit 54c4eb69acf700fdf80304e9d0827d3ea13cbc6d
Author: Matthias Sohn <matthias.sohn@sap.com>
Date:   Wed Sep 19 09:00:33 2012 +0200

    Prepare for 2.1 maintenance changes

    Change-Id: I436f36a7c6dc86916eb4cde038b27f9fb183465a
    Signed-off-by: Matthias Sohn <matthias.sohn@sap.com>

M       org.eclipse.jgit.ant.test/META-INF/MANIFEST.MF
M       org.eclipse.jgit.ant.test/pom.xml
M       org.eclipse.jgit.ant/META-INF/MANIFEST.MF
M       org.eclipse.jgit.ant/pom.xml
M       org.eclipse.jgit.console/META-INF/MANIFEST.MF
M       org.eclipse.jgit.console/pom.xml
M       org.eclipse.jgit.http.server/META-INF/MANIFEST.MF
M       org.eclipse.jgit.http.server/pom.xml ... more output
```

如你所见，我们得到了预期的输出。所以，举个例子，如果你一直在为`git diff`使用一组特定的选项，那么你可以将其制作成一个别名，以便轻松使用。

# 它是如何工作的...

这就像是在`config`文件中插入文本一样简单。所以，你可以尝试打开`.git/config`配置文件，或者你也可以通过`git config -list`来列出配置：

```
$ git config --list  | grep alias
alias.amm=commit --amend
alias.lline=log --oneline -10
alias.co=checkout
alias.ca=commit -a -m "Quick commit"
alias.count=!git log --all --oneline | wc -l
```

`alias` 特性非常强大，它的理念是让你通过它来缩短那些你经常使用的长命令。你还可以利用这个特性将那些长命令缩短为别名，这样你就能更加频繁和精确地使用命令。如果你将一个长而复杂的 Git 评论设置为别名，你每次运行它时都会按相同的方式操作，而输入长命令则时常容易出错。

# 配置和使用 Git 脚本

是的，我们有别名，别名的作用就是将简短的命令转换为简洁有用的 Git 命令。然而，当涉及到较长的脚本，它们也是你工作流程的一部分，并且你希望将它们整合进 Git 时，你可以简单地将脚本命名为 `git-scriptname`，然后像使用 `git scriptname` 一样调用它。

# 如何实现...

有几点需要记住。脚本必须在你的路径中，这样 Git 才能使用它。除此之外，只有想象力才是界限：

1.  打开你喜欢的编辑器，并将以下内容插入到文件中：

```
#!/bin/bash
NUMBEROFCOMMITS=$(git log --all --oneline | wc -l)
while :
WHICHCOMMIT=$(( ( RANDOM % $NUMBEROFCOMMITS ) + 1 ))
COMMITSUBJECT=$(git log --oneline --all -${WHICHCOMMIT} | tail -n1)
COMMITSUBJECT_=$(echo "$COMMITSUBJECT" | cut -b1-60)
do
 if [ $RANDOM -lt 14000 ]; then 
 echo "${COMMITSUBJECT_} PASSED"
 elif [ $RANDOM -gt 15000 ]; then 
 echo "${COMMITSUBJECT_} FAILED"
 fi 
done
```

1.  将文件保存为 `git-likeaboss`。这是一个非常简单的脚本，它将列出随机的提交主题，结果会显示“通过”或“失败”。它会一直运行，直到你按下 *Ctrl* + *C*：

```
$ git likeaboss
5ec4977 Create a MergeResult for deleted/modified    PASSED
fcc3349 Add reflog message to TagCommand             PASSED
591998c Do not allow non-ff-rebase if there are ed   PASSED
0d7dd66 Make sure not to overwrite untracked notfil  PASSED
5218f7b Propagate IOException where possible where   FAILED
f5fe2dc Teach PackWriter how to reuse an existing s  FAILED
```

1.  注意，你也可以使用 tab 补全这些命令，Git 会在你稍微拼写错误时考虑它们，具体如下：

```
$ git likeboss
git: 'likeboss' is not a git command. See 'git --help'.

Did you mean this?
  likeaboss
```

显然，这个脚本本身在日常工作中并没有太大用处，但我们希望你能理解我们要表达的意思。所有脚本都围绕软件交付链展开，你可以将它们命名为 Git，因为它们是 Git 的一部分。这使得记住哪些脚本适用于你的工作变得更加容易。

无论是 Git 别名还是 Git 脚本，在使用 tab 补全时都会作为 Git 命令显示出来。输入 `git <tab> <tab>` 以查看可能的 Git 命令列表。

# 设置和使用提交模板

在本章中，我们一直在使用动态模板，但 Git 也有静态提交模板的选项。静态模板本质上只是一个配置好的文本文件。使用模板非常简单直接。

# 准备工作

首先，我们需要一个模板。这个模板必须是一个你知道位置的文本文件。创建一个包含以下内容的文件：

```
#subject no more than 74 characters please

#BugFix id in the following formats
#artf [123456]
#PCP [AN12354365478]
#Bug: 123456
#Descriptive text about what you have done 
#Also why you have chosen to do in that way as 
#this will make it easier for reviewers and other
#developers.
```

这是我们提供的一个简单的提交信息模板。你可能会发现有其他模板倾向于将 bug 放在标题或提交信息的底部。将 bug 放在顶部的原因是，人们往往不会阅读文本中的重要部分！这里重要的是格式化外部系统引用的部分。如果我们正确地处理了这些引用，我们也能自动更新缺陷系统。将文件保存为 `~/committemplate`。

# 如何实现...

我们将配置我们新创建的模板，然后进行一次提交，使用这个模板。

要配置模板，我们需要使用`git config commit.template <pathtofile>`来设置它，一旦设置完成，我们就可以尝试创建一次提交，看看它是如何工作的：

1.  从以下配置模板开始：

```
$ git config commit.template  ~/committemplate
```

1.  现在列出`config`文件以查看它是否已被设置：

```
$ git config --list | grep template
commit.template=/Users/JohnDoe/committemplate
```

1.  正如我们预料的那样，配置成功了。模板，就像任何其他配置一样，可以通过`git config --global`在全局级别设置，或者通过不使用`--global`选项在本地仓库级别设置。我们仅为这个仓库配置了提交模板。让我们尝试进行一次提交：

```
$ git commit --allow-empty
```

1.  现在，提交信息编辑器应该已打开，你应该在提交信息编辑器中看到我们的模板：

```
#subject no more than 74 characters please

#BugFix id in the following formats
#artf [123456]
#PCP [AN12354365478]
#Bug: 123456
#Descriptive text about what you have done
#Also why you have chosen to do in that way as
#this will make it easier for reviewers and other
#developers.
```

就是这么简单。

在本章中，我们已经看到了如何在提交信息中存在特定单词时防止推送。我们还看到了如何在提交时动态创建适用于你或其他开发人员的有效提交信息。

我们接着展示了如何通过添加简短的脚本或别名将功能集成到你自己的 Git 中，这些脚本或别名都会通过 Git 执行。希望这些信息能帮助你更加聪明地工作，而不是更加辛苦。
