# 第三章：您的用户与 Gitolite

既然我们已经成功安装了 Gitolite，接下来就该讨论用户如何与 Gitolite 管理的系统互动，以及他们需要了解什么才能开始使用它。这将使您能够让一些高级用户开始使用该系统，同时在我们继续深入学习 Gitolite 的过程中，您仍然能够继续了解更多内容。作为管理员，您将需要做出很多决策，诸如仓库和分支的命名规范、访问规则的严格程度等，以及许多您在后续学习中会了解到的其他方面。这些高级用户可以通过反馈或作为决策的参考帮助您。

# 访问 Git 仓库

在讨论如何访问 Gitolite 管理的仓库之前，我们首先需要了解 Git 仓库通常是如何访问的，即当您不使用 Gitolite 时。

## Git 服务器、SSH 和 HTTP

我们首先回顾一下用户如何访问普通 Git 服务器。Git 仓库使用 URL 作为定位符，因此当用户克隆、拉取或推送到远程仓库时，都是通过合适的 URL 完成的。Git URL 与其他 URL 并没有太大区别，而 git-clone 的 man 页面中有一节专门介绍它们，您可以看到所有可以使用的语法变体。

然而，对于需要认证的远程访问，实际上只有两个协议是值得关注的：SSH 和 HTTP。在这两者中，基于 SSH 的访问更为常见，因为它设置快捷且简单；即使是在刚安装好的 Unix 系统上，通常也不需要额外的配置就能让它工作。

如之前主页所述，SSH URL 的一般形式为 `ssh://[user@]host.xz[:port]/path/to/repo.git`。当您输入一个 URL，如 `ssh://git@server.example.com/repo` 以访问远程 Git 仓库时，通常会要求您输入密码，除非已经为访问远程主机配置了 SSH 密钥对。一旦授权访问，SSH 守护进程会在远程端运行相应的 Git 命令，与本地端的 Git 客户端进行通信。

# 访问 Gitolite 仓库

在上一节的背景知识基础上，我们准备好了解当用户通过 Gitolite 访问 Git 仓库时，情况会发生哪些变化。

### 注意

本节将包含大多数管理员需要提供给用户的基本材料，或者向他们解释如何访问 Gitolite 服务器。然而，根据用户对 SSH 和相关主题的熟悉程度，您可能需要通过补充信息、示例或特定站点的说明来扩展这些内容。

## SSH 密钥对

最显著的变化是不再支持密码访问；用户*必须*使用密钥对并将公钥发送给管理员，以便将其添加到 Gitolite 中。如果未执行此操作，Gitolite 无法识别该用户。

如果他们还没有 ssh 密钥对，应该在自己的工作站上生成一个。

你的用户需要使用`ssh-keygen`命令来创建密钥对。如果他们选择了默认选项，这将生成两个文件，分别是包含私钥的`id_rsa`和包含公钥的`id_rsa.pub`。在 Windows 系统中，命令会返回文件创建的完整路径，而在 Unix 系统中，文件会存储在`$HOME/.ssh`中。

### 注意

理想情况下，用户应为私钥设置密码短语以增强安全性，并使用`ssh-agent`以方便使用；然而，这些内容超出了本书的范围。任何与 ssh 相关的文本或网站应提供足够的详细信息，OpenSSH 软件包附带的文档也会有所帮助。

然后，他们会将公钥（文件名以`.pub`结尾）发送给你，以便你可以将他们添加为用户。

## 仓库命名

第二个变化是，每个仓库的名称将是你作为管理员在`conf/gitolite.conf`文件中创建的名称，我们在前一章的最后部分简要地看到过。实际上，Gitolite 会在托管用户账户中的`$HOME/repositories`下创建所有的仓库，但只有你（管理员）需要知道这一点。就用户而言，要访问在配置文件中列出的`repo foo`仓库，使用的 URL 就是`git@server:foo`（或者更长形式的`ssh://git@server/foo`）。

此外，请注意，仓库名称后面的`.git`对于 Git 命令（即`clone`、`fetch`、`push`、`ls-remote`和`archive`）来说是可选的。Git 本身无论有无`.git`都能正常工作，因此为了保持一致，Gitolite 也做了相同的处理。然而，在与 Gitolite 交互时，比如运行涉及仓库的 Gitolite 命令时，你必须使用裸名称（即没有`.git`结尾的名称），这是 Gitolite 在错误信息或输出信息中显示的名称。

# 从 Gitolite 获取信息

一旦你的用户访问了 Gitolite，他们可能会希望查看自己可以访问哪些仓库。最简单的方法是运行`info`命令，这是所有远程用户都可以使用的：

```
$ ssh git@server info 
hello adam, this is git@server running gitolite3 v3.5.3.1-7-g5f88a09 on git 1.8.3.1 
 R W  gitolite-admin 
 R W  t2 
 R W  testing

```

这会告诉你你的 Gitolite 用户名是什么（在此案例中是`adam`），你可以访问哪些仓库，以及你是否被允许读取和写入，或者只能读取但不能写入仓库。除此之外，该命令还会告诉你服务器上运行的 Gitolite 和 Git 的版本，这可能会有帮助。

# Gitolite 命令

`info`命令当然不是唯一可以提供给用户的命令；还有其他一些命令。如你从前面的部分所猜测的，运行 Gitolite 命令的通用格式就是`ssh git@server command-name command-arguments`，其中参数当然是可选的。

方便的是，Gitolite 还提供了一个命令来列出所有可用的命令：

```
$ ssh git@server help 
hello adam, this is gitolite3 v3.5.3.1-7-g5f88a09 on git 1.8.3.1 

list of remote commands available: 

 desc 
 help 
 info 
 perms 
 writable

```

如你所见，这为远程用户提供了他们可以运行的命令列表。（其中一些命令只能在后续章节中解释）

此外，如果你在 Gitolite 托管用户的命令行下运行 `gitolite help`，你将获得*所有*可用命令的列表，而不仅仅是启用了远程访问的命令。

# 获取命令的帮助

获取命令的帮助很简单。每个 Gitolite 命令在调用时如果只带有 `-h` 参数，都会返回一条帮助信息。例如，info 命令的帮助信息如下：

```
$ ssh git@server info -h 

Usage:  gitolite info [-lc] [-ld] [<repo name pattern>] 

List all existing repos you can access, as well as repo name patterns you can 
create repos from (if any). 

 '-lc'       lists creators as an additional field at the end. 
 '-ld'       lists description as an additional field at the end. 

The optional pattern is an unanchored regex that will limit the repos 
searched, in both cases.  It might speed up things a little if you have more 
than a few thousand repos. 

```

如前所述，某些选项涉及到 Gitolite 中我们尚未遇到的功能，等到相关内容讲解时，这些选项会变得更加清晰。

# 排查 SSH 问题

当你开始为系统添加第一个用户时，可能会遇到一些与 SSH 相关的问题。本节将简要讨论这些问题，并解释如何识别和解决它们。

## 授权，而非身份验证

首先，我们需要一些基本定义。身份验证是验证你是否是你所声称的人。授权是确定你想做什么并决定是否允许你执行该操作的过程。授权发生在身份验证之后（系统只能在确认你身份之后，才能决定你是否被允许做某事！）。

Gitolite 只关注授权；它不进行身份验证。身份验证交由 SSH 服务器或 Web 服务器来处理。

### 提示

本书不涉及 HTTP 模式；请参考 Gitolite 的在线文档来使用该模式。

一旦 SSH 服务器验证了用户身份，它会使用 SSH 授权密钥文件（`$HOME/.ssh/authorized_keys`）中的 `command` 选项来调用 Gitolite 并传递用户名。Gitolite 随后决定该用户是否被允许访问该仓库。

## 重复的公钥

如果用户的公钥在设置 Gitolite 之前已经是授权密钥文件的一部分（可能是为了允许他获得登录 shell 并运行 Unix 命令），那么该密钥将在授权密钥文件中出现两次——一次是原始的，另一次是带有 `command` 选项以及 Gitolite 在 `keydir` 目录中为每个公钥添加的其他选项。

然而，如果授权密钥文件中出现相同的密钥两次，ssh 服务器只会查看第一次出现的密钥。同时，Gitolite 会尽力确保已经拥有正常访问权限的密钥继续保持这种权限，因此它会将 Gitolite 行放在比默认访问权限更严格的行之后。意味着有 shell 访问权限的 Gitolite 主机用户将无法使用 Gitolite，因为该密钥不会触发 Gitolite。他们需要为 Gitolite（即仓库）访问创建并使用不同的 ssh 密钥对。接着，他们需要在客户端管理这两个密钥对，可能会使用 `$HOME/.ssh/config` 来帮助管理。进一步解释 ssh `config` 文件以及如何帮助你选择使用哪个密钥超出了本书的范围。然而，几乎任何一本关于 ssh 的书籍，以及你系统上 ssh 的主页面，都应该提供这些信息。

## 诊断公钥问题

诊断公钥问题的最佳方法，如上一节所述，是运行 Gitolite 随附的 `sshkeys-lint` 程序。以下是一个例子，故意创建了两个公钥问题。第一个问题是我们重复使用了已经有 shell 访问权限的密钥，将其作为 `u2.pub` 添加到 Gitolite 中。第二个问题是我们将文件 `u5.pub` 复制为 `u6.pub`。在这些更改之后，运行 sshkeys-lint 命令的输出如下：

```
$ gitolite sshkeys-lint 
sshkeys-lint: ==== checking authkeys file: 
sshkeys-lint: WARNING: authkeys line 5 (user u2) will be ignored by sshd; same key found on line 1 (shell access) 
sshkeys-lint: WARNING: authkeys line 9 (user u6) will be ignored by sshd; same key found on line 8 (user u5) 
sshkeys-lint: ==== checking pubkeys: 
sshkeys-lint: admin.pub maps to user admin 
sshkeys-lint: u1.pub maps to user u1 
sshkeys-lint: u2.pub maps to shell access 
sshkeys-lint: u3.pub maps to user u3 
sshkeys-lint: u4.pub maps to user u4 
sshkeys-lint: u5.pub maps to user u5 
sshkeys-lint: WARNING: u6.pub appears to be a COPY of u5.pub 
sshkeys-lint: u6.pub maps to user u5 

3 warnings found 

```

如你所见，该命令列出了潜在问题，首先是在授权密钥文件（`$HOME/.ssh/authorized_keys`）中，然后是在 Gitolite 拥有的公钥中。

## SSH 最佳实践

我们现在已经了解了如何排查 ssh 问题。然而，最好从一开始就避免这些问题，一个避免问题的好规则是：不要给任何用户提供服务器的 shell 访问权限。即使是你作为管理员，也应该登录到其他用户 ID，运行 `su - git`，然后在需要在 Gitolite 主机用户的命令行上执行操作时提供密码。除非你非常熟悉 ssh，否则应该让授权密钥文件中的所有密钥都由 Gitolite 管理。这样可以消除大多数常见的 ssh 密钥问题。

# 总结

在本章中，我们学习了如何将用户添加到新的 Gitolite 安装中，以及如何查找和修复 ssh 密钥问题。在下一章中，我们将讨论如何创建新的仓库。
