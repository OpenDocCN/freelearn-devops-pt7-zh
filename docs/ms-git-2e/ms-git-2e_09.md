# 7

# 发布你的更改

*第六章*《与 Git 协作开发》（上一章）教你如何使用 Git 进行团队协作，重点介绍仓库与仓库之间的交互。它描述了不同的仓库协作设置方式，展示了不同的协作工作流，如集中式工作流和集成管理工作流。它还展示了 Git 如何管理有关远程仓库的信息。

在本章中，你将了解如何在本地仓库和远程仓库之间交换信息，以及 Git 如何管理可能需要的远程仓库凭证。

本章还将教你如何将你的更改推送到上游，使其出现在项目的官方历史中，在其标准仓库中。这可以通过将更改推送到中央仓库，推送到你自己的发布仓库并向集成管理者发送某种拉取请求，或甚至交换补丁来完成。

本章将涵盖以下主题：

+   Git 使用的传输协议及其优缺点

+   管理远程仓库的凭证（密码、密钥）

+   发布更改：推送和拉取请求，以及交换补丁

+   使用捆绑包进行离线传输和加速初始克隆

+   远程传输助手及其使用

# 传输协议和远程助手

通常，远程仓库的配置中的 URL 包含有关传输协议、远程服务器地址（如果适用）以及仓库路径的信息。有时，提供对远程仓库访问的服务器支持多种传输协议；你需要选择使用哪种协议。本节旨在帮助做出这一选择。

## 本地传输

如果远程仓库位于同一个本地文件系统中，你可以使用仓库的路径或 `file://` 方案来指定仓库 URL：

```
/path/to/repo.git/
file:///path/to/repo.git/
```

前者意味着 Git 克隆时使用 `--local` 选项，这会绕过智能的 Git 机制，直接复制（或者对于 `.git/objects` 下不可变文件创建硬链接，虽然可以通过 `--no-hardlinks` 选项避免）；后者较慢，但可以用于获取一个干净的仓库副本（例如，在历史重写后移除意外提交的密码或其他秘密；这在*第十章*《保持历史清洁》的“重写历史”部分中有描述）。

这种传输是一个不错的选择，适合快速从他人的工作仓库中获取工作，或使用具有适当权限的共享文件系统共享工作。

作为一个特殊情况，单个点（`.`）表示当前仓库。这意味着

```
$ git pull . next
```

假设 `pull.rebase` 设置为 false，基本等价于

```
$ git merge next
```

## 智能传输

当你想从另一个机器上的代码库获取时，需要访问 Git 服务器。如今，最常见的是遇到支持 Git 的智能服务器。智能下载器会协商哪些版本是必需的，并创建一个定制的 `packfile` 发送给客户端。类似地，在推送过程中，Git 服务器会与用户机器上的 Git 进行通信（即与客户端）来确定需要上传哪些版本。

支持 Git 的智能服务器使用 `git upload-pack` 下载器来获取数据，使用 `git receive-pack` 来推送数据。如果这些工具不在 `PATH` 中（但例如安装在用户的主目录中），你可以通过 `--upload-pack` 和 `--receive-pack` 选项来指定获取和推送的路径，或者在 `remote.<name>` 部分使用 `uploadpack` 和 `receivepack` 配置变量。

几乎没有例外（例如，代码库使用了被旧版 Git 实例访问的子模块，而该版本无法理解子模块），Git 传输是向后和向前兼容的——客户端和服务器会协商双方都能使用的功能。

### 原生 Git 协议

使用 `git://` URLs 的本地传输方式提供了只读的匿名访问权限（尽管理论上，你可以配置 Git 允许推送，通过启用 `receive-pack` 服务来实现，方法是通过命令行选项 `--enable=receive-pack` 或通过 `daemon.receivePack` 布尔配置变量—不过这种方式完全不推荐，甚至在封闭的本地网络中也不应使用）。

Git 协议没有进行任何认证，包括没有服务器认证，因此在不安全的网络上使用时需要小心。这个协议的 `git daemon` TCP 服务器通常监听在 `9418` 端口；你需要能够访问该端口（通过防火墙）才能使用原生 Git 协议。

小知识

**git://** 协议没有安全版本。与 FTP 和 HTTP 协议不同，**git://** 协议没有像 FTPS 和 HTTPS 那样支持 TLS。另一方面，可以认为 Git 使用的 SSH 传输就是 **git://** 协议通过 SSH 的实现。

### SSH 协议

服务器上的 `git upload-pack` 或 `git receive-pack` 使用 SSH 执行远程命令。该协议不支持匿名的未认证访问，尽管作为变通方法，你可以为其设置一个访客账户（没有密码或密码为空）。

使用公钥-私钥认证可以在每次连接时无需提供密码。不过，你可能需要提供密码一次，以解锁受密码保护的私钥。你可以在本章的*凭证/密码管理*部分了解更多关于认证的信息。许多 Git 托管站点和软件开发平台要求通过 SSH 进行密钥认证来访问代码库。

对于 SSH 协议，你可以使用 `ssh://` 作为协议部分的 URL 语法：

```
ssh://[user@]host.example.com[:port]/path/to/repo.git/
```

或者，你可以使用类似于 `scp` 的语法：

```
[user@]host.example.com:path/to/repo.git/
```

SSH 协议还支持 `~username` 扩展，就像原生 Git 传输一样（`~` 是你登录的用户的家目录，`~user` 是 `user` 的家目录），有两种语法形式：

```
ssh://[user@]host.example.com/~[user]/path/to/repo.git/
[user@]host.example.com:~[user]/path/to/repo.git/
```

SSH 使用首个接触认证用于服务器（**TOFU**——即**首次使用时信任**），它记住服务器端先前使用的密钥，并在密钥更改时提醒用户，要求确认（服务器密钥可能是合法更改的，例如，由于 SSH 服务器重新安装）。你可以在第一次连接时检查服务器密钥的指纹。

### 智能 HTTP(S) 协议

Git 还支持智能 HTTP(S) 协议，这需要一个 Git 知识的 CGI 或服务器模块——例如，`git-http-backend`（它本身是一个 CGI 模块）。该协议使用以下 URL 语法：

```
http[s]://[user@]host.example.com[:port]/path/to/repo.git/
```

默认情况下，在没有任何其他配置的情况下，Git 允许匿名下载（`git fetch`、`git pull`、`git clone` 和 `git ls-remote`），但要求客户端必须进行身份验证才能上传（`git push`）。

如果需要认证才能访问仓库，则使用标准的 HTTP 认证，由 HTTP 服务器软件完成。使用 SSL/TLS 配合 HTTPS 可以确保如果需要密码（例如，如果服务器使用基本 HTTP 认证），密码会被加密发送，并且服务器身份会被验证（使用服务器的 CA 证书）。

## 传统（愚蠢）传输

一些传输方式不需要任何 Git 知识的智能服务器——它们不需要在服务器上安装 Git（对于智能传输，至少需要 `git-upload-pack` 和/或 `git-receive-pack`）。这些是 FTP(S) 和“愚蠢”HTTP(S) 协议的传输方式（如今，使用 `remote-curl` 辅助工具实现）。

这些传输仅需要适当的标准服务器（FTP 服务器或 web 服务器）以及来自 `git update-server-info` 的最新数据。从此类服务器拉取时，Git 使用所谓的 **提交遍历器** 下载器：从已获取的分支和标签开始，Git 依次向下遍历提交链，下载包含缺失修订版本和其他数据（例如，修订版下的文件内容）的对象或包。

这种传输方式效率低下（在带宽方面，尤其是在延迟方面），但另一方面，如果被中断，可以恢复。尽管如此，相较于使用“愚蠢”协议，还有更好的解决方案——即使用捆绑包（参见本章中的 *离线传输与捆绑包* 部分），当网络连接不可靠到无法完成克隆操作时，使用捆绑包可以是一种替代方案。

向一个“愚蠢”服务器推送仅能通过 HTTP 和 HTTPS 协议进行。这要求 web 服务器支持 WebDAV，并且 Git 必须与 *expat* 库链接编译。FTP 和 FTPS 协议为只读（仅支持 `clone`、`fetch` 和 `pull`）。

作为设计特性，Git 可以自动将简单协议的 URL 升级为智能 URL。反之，一个支持 Git 的 HTTP 服务器也可以降级为向后兼容的简单协议（至少在获取操作时：智能 HTTP 服务器不支持基于 WebDAV 的简单 HTTP 推送操作）。这个特性允许使用相同的 HTTP(S) URL 进行简单和智能访问：

```
http[s]://[user@]host.example.com[:port]/path/to/repo.git/
```

## 离线传输与存档

有时，你的机器和持有你想要获取的 Git 仓库的服务器之间没有直接连接。或者，服务器可能没有运行，你仍然想将更改复制到另一台机器上。也许你的网络出现故障，或者你在某个现场工作，并且出于安全原因无法访问本地网络。也可能是你的无线/以太网卡坏了。

输入`git bundle`命令。该命令会将所有通常通过网络传输的内容打包，将对象和引用放入一个特殊的二进制存档文件中，称为`bundle`（类似于`packfile`，只是包含了分支等信息）。你需要指定要打包哪些提交—这是网络协议自动为你完成的操作，通常用于在线传输。

细节

当你使用智能传输时，会进行一个**want/have 协商**阶段，客户端告诉服务器它在自己的仓库中已有的内容，以及它希望从服务器上获取哪些已发布的引用，以找到公共的版本。这些信息将被服务器用于创建一个 packfile，并且只发送必要的内容给客户端，从而最小化带宽使用。

接下来，你需要通过某种方式将这个包（这个存档）传输到你的机器上。可以通过例如使用`git clone`或`git fetch`命令，替换仓库 URL 为包的文件名来完成此操作。

### Git 传输的代理

当无法直接访问服务器时，例如，在防火墙保护的局域网内部，有时可以通过代理连接。

对于原生 Git 协议（`git://`），你可以使用`core.gitProxy`配置变量，或`GIT_PROXY_COMMAND`环境变量来指定代理命令—例如，`ssh`。这可以通过`core.gitProxy`值的特殊语法为每个远程仓库设置，例如，`"ssh" for kernel.org`。

你可以使用`http.proxy`配置变量或适当的*curl*环境变量，如`http_proxy`，来指定用于 HTTP(S)协议的 HTTP 代理服务器（`http(s)://`）。这可以通过`remote.<remote name>.proxy`配置变量为每个远程仓库单独设置。

你可以配置 SSH（使用其配置文件，例如`~/.ssh/config`）来使用隧道（端口转发）或代理命令（例如，`netcat/nc;`或 SSH 的`netcat`模式——即`ssh -W –`——前提是你的 SSH 实现支持此功能）。这是一个推荐的 SSH 代理解决方案；如果无法使用隧道或代理，你可以使用`ext::`传输助手，正如本章后面*使用远程助手的传输中继*部分所展示的那样。

### 使用捆绑包克隆和更新

假设你想将一个项目的历史（为了简单起见，仅限于`master`分支）从`machineA`（例如，你的工作电脑）转移到`machineB`（例如，一个现场电脑）。然而，这两台机器之间没有直接连接。

首先，我们创建一个包含`master`分支全部历史的捆绑包（见*第四章*，*探索项目历史*），并标记这个历史点以便知道我们捆绑了什么，稍后会用到：

```
user@machineA ~$ cd repo
user@machineA repo$ git bundle create ../repo.bundle master
user@machineA repo$ git tag -f lastbundle master
```

这里，捆绑包文件是在工作目录外创建的。这是一个选择问题；将其存储在仓库外意味着你不必担心不小心将其添加到项目历史中，或者需要添加新的`ignore`规则。`*.bundle`文件扩展名只是命名约定的一部分。

重要提示

出于安全原因，为了避免泄露已删除但未清除的历史部分（例如，意外提交的含密码的文件），Git 只允许从**git show-ref**兼容的引用中获取：分支、远程追踪分支和标签。

创建捆绑包时适用相同的限制。这意味着，例如，由于实现原因，你不能运行**git bundle creates master¹**。尽管当然，因为你控制着服务器端，作为一种解决方法，你可以为**master^**创建一个新分支，（暂时）回退**master**，或者检出**master^**的分离**HEAD**。

然后，你将刚刚创建的`repo.bundle`文件转移到`machineB`（通过电子邮件、USB 闪存驱动器等）。由于这个捆绑包包含了一个完整的、独立的历史子集，直到第一个（没有父提交的）根提交，你可以通过克隆它来创建一个新的仓库，只需将捆绑包文件名替换为仓库 URL：

```
user@machineB ~$ git clone repo.bundle repo
Cloning into 'repo'...
warning: remote HEAD refers to non-existent ref, unable to checkout.
user@machineB ~$ cd repo
user@machineB repo$ git branch -a
  remotes/origin/master
```

哎呀。我们没有捆绑`HEAD`，所以 Git 克隆时不知道当前应检出的分支：

```
user@machineB repo$ git bundle list-heads ../repo.bundle
5d2584867fe4e94ab7d211a206bc0bc3804d37a9 refs/heads/master
```

提示

因为捆绑包可以视作远程仓库，我们本可以直接使用**git ls-remote ../repo.bundle**命令，而不是**git bundle** **list-heads ../repo.bundle**。

因此，考虑到这个捆绑包的状态，我们需要指定要检出的分支，以避免问题（如果我们也捆绑了`HEAD`，则不需要这样做）：

```
user@machineB ~$ git clone repo.bundle --branch master repo
```

不需要再次克隆，我们可以通过选择当前分支来修复失败的检出问题：

```
user@machineB repo$ git switch master
Already on 'master'
Branch 'master' set up to track remote branch 'master' from 'origin'.
```

正如你所看到的，这里 Git 猜测当我们尝试切换到一个不存在的本地分支`master`时，实际上我们想做的是创建一个本地分支来提交新的提交到远程`master`分支。换句话说，就是创建一个本地分支来跟踪（追踪）远程`origin`中存在的同名分支。Git 所做的事情与我们运行以下命令的效果相同：

```
$ git switch --create master --track origin/master
```

要更新从包中克隆的`machineB`上的仓库，你可以在替换掉存储在`/home/user/repo.bundle`的原始包后，执行`fetch`或`pull`操作来更新。

要创建一个包含自上次传输以来更改的包，在`machineA`上运行以下命令：

```
user@machineA repo$ git bundle create ../repo.bundle \
  lastbundle..master
user@machineA repo$ git tag -f lastbundle master
```

这将打包所有自`lastbundle`标签以来的更改；该标签表示之前包中复制的内容（有关双点语法的解释，请参见*第四章*，*探索项目历史*，*双点符号*部分）。创建包后，这将更新标签（使用`-f`或`--force`来替换它），就像第一次创建包时一样，以便下一个包也可以从当前点增量创建。

然后，你需要将包复制到`machineB`，*替换*旧的包。此时，可以简单地执行拉取操作来更新仓库，示例如下：

```
user@machineB repo$ git pullFrom /home/user/repo.bundle
   ba5807e..5d25848  master     -> origin/master
Updating ba5807e..5d25848
Fast-forward
```

### 使用包更新现有仓库

有时，你可能已经克隆了一个仓库，但网络出现故障。或者，也许你已经移动到了**局域网**（**LAN**）之外，现在无法访问服务器。最终结果是你有一个现有的仓库，但没有与上游的直接连接（即我们克隆的那个仓库）。

现在，如果你不想将整个仓库打包，这样会浪费空间，就像在*使用包克隆和更新*一节中那样，你需要找到一种方法，指定一个截止点（基点），确保它一定存在于目标仓库中（你想要更新的那个仓库）。你可以使用几乎任何技术来指定要打包到包中的修订范围，参考*第四章*，*探索项目历史*。唯一的限制是，如前所述，历史记录必须从一个分支或标签开始（任何`git show-ref`接受的内容）。当然，你可以使用`git`的`log`命令检查这个范围。

常用的指定修订范围打包到包中的解决方案如下：

+   使用两个仓库中都存在的标签：

    `git bundle create ../``repo.bundle v0.1..master`

+   根据提交创建时间创建一个截止点：

    `git bundle create ../repo.bundle --``since=1.week master`

+   只打包最后几次修订，通过提交次数来限制修订范围：

    `git bundle create ../repo.bundle -``5 master`

提示

打包时最好多装一点而不是太少。你可以通过**git bundle verify**检查仓库是否包含需要从捆绑包中获取的必要提交。如果打包内容过少，你会收到以下错误：

**user@machineB repo$ git pull ../****repo.bundle master**

**错误：仓库缺少这些** **先决提交：**

**错误：ca3cdd6bb3fcd0c162a690d5383bdb8e8144b0d2**

然后，在将其传输到`machineB`后，你可以像使用常规仓库一样使用该捆绑文件来执行一次性拉取（将捆绑文件名代替网址或远程名称）：

```
user@machineB repo$ git pull ../repo.bundle master
From ../repo.bundle
 * branch            master     -> FETCH_HEAD
Updating ba5807e..5d25848
```

如果你不想处理合并，你可以将其拉取到远程跟踪分支（这里使用的`<远程分支>:<远程跟踪分支>`符号，称为*refspec*，将在*第八章*，*高级* *分支技巧*中解释）：

```
user@machineB repo$ git fetch ../repo.bundle \
   refs/heads/master:refs/remotes/origin/master
From ../repo.bundle
   ba5807e..5d25848  master     -> origin/master
Updating ba5807e..5d25848
```

或者，你可以使用`git remote add`创建一个新的快捷方式，使用捆绑包文件的路径代替仓库 URL。然后，你可以像上一节所述那样处理捆绑包。

### 使用捆绑包帮助进行初始克隆

智能传输提供比普通传输更有效的传输方式。另一方面，使用智能传输的可恢复克隆概念至今依然难以实现（它在 Git 版本 2.34 中不可用，不过也许将来某人会实现它）。对于历史较长、文件众多的大型项目，初次克隆可能非常庞大（例如，`linux-next`超过 2.7 GB）且需要很长时间。如果网络不可靠，这可能成为一个问题。

提示 – 解决方法

你可以通过使用浅克隆或稀疏克隆来绕过不可靠网络的问题（见*第十二章*，*管理* *大型仓库*），并逐步扩展，直到得到完整的仓库。也有一些第三方工具可以自动执行此操作。

你可以从源仓库创建一个捆绑包，例如，使用以下命令（需要在服务器上运行）：

```
$ git –git-dir=/dir/repo.git bundle create -- all HEAD
```

一些服务器可能提供这样的捆绑包来帮助进行初始克隆。有一种做法是，供克隆使用的捆绑包与仓库位于相同的 URL 下，但后缀为`.bundle`而非`.git`。例如，`https://git.example.com/user/repo.git`的捆绑包可以在`https://git.example.com/user/repo.bundle`找到。

然后，你可以使用任何支持断点续传的传输方式下载这种捆绑包，它是一个普通的静态文件：HTTP(S)、FTP(S)、rsync，甚至是 BitTorrent（使用适当的客户端，支持断点续传）。

在现代 Git 中，用户可以通过 `--bundle-uri` 命令行选项指定 bundle URI，或者 Git 服务器可以广告一个 bundle 列表。bundle URI 列表也可以保存在配置文件中。然后，从 bundle 服务器（例如 [`github.com/git-ecosystem/git-bundle-server`](https://github.com/git-ecosystem/git-bundle-server)）获取数据将是自动的。

## 远程传输助手

当 Git 不知道如何处理某个传输协议时（即尝试使用一个没有内建支持的协议），它会尝试使用适当的 **远程助手** 来处理该协议（如果有的话）。这就是为什么当仓库 URL 的协议部分出现错误时，Git 会返回如下错误信息：

```
$ git clone shh://git@example.com:repo
Cloning into 'repo'…
fatal: Unable to find remote helper for 'shh'
git: 'remote-shh' is not a git command. See 'git --help'.
```

这个错误信息表示 Git 尝试找到 `git-remote-shh` 来处理 `shh` 协议（实际上是 `ssh` 的拼写错误），但是没有找到具有此名称的可执行文件。

你可以通过 `<transport>::<address>` 语法显式地请求特定的远程助手，其中 `<transport>` 定义了助手（`git remote-<transport>`），`<address>` 是助手用来查找仓库的字符串。

现代 Git 支持通过 `curl` 系列远程助手来处理傻瓜 HTTP、HTTPS、FTP 和 FTPS 协议，分别是 `git-remote-http`、`git-remote-https`、`git-remote-ftp` 和 `git-remote-ftps`。

### 使用远程助手进行传输中继

Git 包括两个通用的远程助手，可以用来代理智能传输：`git-remote-fd` 帮助器通过双向套接字或一对管道连接远程服务器，`git-remote-ext` 帮助器则使用外部命令连接远程服务器。

在后者的情况下，Git 使用 `"ext::<command> <arguments>"` 语法指定仓库 URL，Git 运行指定的命令来连接服务器，将数据传递给命令的标准输入，并从其标准输出接收响应。这些数据假设会传递给 `git://` 服务器、`git-upload-pack`、`git-receive-pack` 或 `git-upload-archive`（具体取决于情况）。

例如，假设你的仓库托管在一个局域网主机上，你可以通过 SSH 登录。可是，出于安全原因，这台主机在互联网上不可见，你需要通过网关主机 `login.example.com`：

```
user@home ~$ ssh user@login.example.com
user@login ~$ ssh work
user@work ~$ find . -name .git -type d -print
./repo/.git
```

问题是——同样出于安全原因——这个网关主机要么没有安装 Git（以减少攻击面），要么没有你的仓库（它使用的是不同的文件系统）。这意味着你无法使用普通的 SSH 协议——除非你能设置 `ssh -L`。SSH 传输实际上是通过 SSH 远程访问的 `git-receive-pack` / `git-upload-pack`，其路径作为参数。这意味着你可以使用 `ext::` 远程助手：

```
user@home ~$ git clone \
   "ext::ssh -t  ssh work %S 'repo'" repo
Cloning into 'repo'...
Checking connectivity... done.
```

这里，`%S`将由 Git 展开为适当服务的完整名称——`git-upload-pack`用于拉取，`git-receive-pack`用于推送。如果登录到内部主机时使用交互式身份验证（例如密码），则需要使用`-t`选项。请注意，克隆结果时需要为仓库命名（此处为`repo`）；否则，Git 将使用命令（`ssh`）作为仓库名称。

提示

你还可以使用**"ext::ssh [<parameters>...] %S '<repository>'"**来为 SSH 传输使用特定选项——例如，选择要使用的密钥对，而无需编辑**.ssh/config**。

这不是唯一的解决方案——尽管没有像原生`git://`协议那样内建支持通过代理发送 SSH 传输（例如`core.gitProxy`）和 HTTP（例如`http.proxy`），你仍然可以通过配置 SSH 的`ProxyCommand`选项，或创建 SSH 隧道来实现。

另一方面，你还可以使用`ext::`远程助手来代理`git://`协议——例如，借助`socat`——并且可以使用单个代理来处理多个服务器。有关详细信息和示例，请参阅`git-remote-ext(1)`手册页。

### 使用外部 SCM 仓库作为远程仓库

远程助手机制非常强大。它还可以与其他版本控制系统交互，透明地将其仓库作为本地 Git 仓库使用。尽管没有内建的助手（除非你考虑 Git 源代码中的`contrib/`目录），你可以找到`git-remote-hg`、`gitifyhg`或`git-cinnabar`助手来访问 Mercurial 仓库，以及`git-remote-bzr`来访问 Bazaar 仓库。

一旦安装，这些远程助手桥接将允许你像操作 Git 仓库一样，使用`<helper>::<URL>`语法来克隆、拉取和推送 Mercurial 或 Bazaar 仓库。例如，要克隆 Mercurial 仓库，你只需运行以下命令：

```
$ git clone "hg::https://hg.example.com/repo"
```

还有`remote.<remote name>.vcs`配置变量，如果你不喜欢在仓库 URL 中使用`<helper>::`前缀，可以使用此方法。通过这种方式，你可以对 Git 和原始**版本控制**系统（**VCS**）使用相同的 URL。

外部版本控制系统客户端

使用远程助手桥接的替代方法是使用专用客户端，例如**git-svn**用于 Subversion，或**git-p4**用于 Perforce。这些客户端与外部 VCS（通常是集中式 VCS）交互，基于此交互管理并更新 Git 仓库，同时基于 Git 仓库中的更改更新外部仓库。

当然，需要记住不同版本控制系统之间的阻抗不匹配，以及远程助手机制的限制。有些功能根本无法转换，或者转换效果不佳——例如，Git 中的章鱼合并（包含多个父提交），或者 Mercurial 中的多个匿名分支（头）。

通过远程助手，还可以没有地方修复错误，用目标原生语法替换对其他版本的引用，并清理由仓库转换创建的其他工件——在更改版本控制系统时应执行一次性转换，可以使用第三方工具`reposurgeon`来执行此类清理操作。

通过远程助手，甚至可以使用严格意义上不是版本控制仓库的东西；例如，使用*Git-Mediawiki*项目，可以使用 Git 查看和编辑基于 MediaWiki 的维基（例如维基百科），将页面历史视为 Git 仓库：

```
$ git clone "mediawiki::https://wiki.example.com"
```

此外，还有远程助手允许额外的传输协议或存储选项——例如，`git-remote-s3bundle`将仓库存储为 Amazon S3 上的捆绑文件，或者`git-remote-codecommit`用于 AWS CodeCommit（如果不能或不想使用具有静态凭据的 HTTPS 身份验证）。还有`git-ssb`通过 Secure ScuttleButt 协议在点对点日志存储中编码仓库。

## 凭证/密码管理

在大多数情况下，除了本地传输（文件系统权限控制访问），向远程仓库发布更改需要身份验证（用户识别自身）和授权（给定用户有权限执行推送操作）。有时，获取仓库也需要身份验证和授权，例如对私有仓库。

用于身份验证的常用**凭证**是*用户名*和*密码*。如果不担心信息泄露（关于有效用户名信息泄露的问题），可以将用户名放在 HTTP 和 SSH 存储库 URL 中，或者可以使用**凭证助手**机制。绝对不应该将密码放在 URL 中，即使在技术上对于 HTTP URL 也是可能的——例如当它们列出进程时，密码可能会对其他人可见。

除了底层传输引擎中固有的机制，如 SSH 的`SSH_ASKPASS`或基于 curl 的传输中的`~/.netrc`文件，Git 提供了自己的集成解决方案。

### 请求密码

一些 Git 命令会交互地要求输入密码（如果用户名未知，则需要用户名），例如`git svn`、HTTP 接口或 IMAP 认证——可以告知它们使用外部程序。该程序会使用合适的提示（称为**身份验证域**，描述密码用途），Git 从该程序的标准输出中读取密码。

Git 会尝试以下位置来询问用户用户名和密码；参见`gitcredentials(7)`手册页：

+   如果设置了**GIT_ASKPASS**环境变量指定的程序（Git 特定的环境变量始终优先于配置变量）

+   否则，如果设置了**core.askpass**配置变量，将使用它

+   否则，使用 **SSH_ASKPASS** 环境变量（如果已设置，且它不是 Git 特有的，因此在序列中稍后会被检查）。

+   否则，系统会提示用户在终端中输入。

这个 `askpass` 外部程序通常根据用户的桌面环境来选择（如有必要，安装后使用）：

+   (**x11-**)**ssh-askpass** 提供一个简单的 X 窗口对话框，要求输入用户名和密码。

+   GNOME 有 **ssh-askpass-gnome**，KDE 有 **ksshaskpass**。

+   **mac-ssh-askpass** 可以用于 macOS。

+   **win-ssh-askpass** 可以用于 MS Windows。

Git 带有一个跨平台的密码对话框，使用 Tcl/Tk 编写——`git-gui--askpass`——用于配合 `git gui` 图形界面和 `gitk` 历史查看器。

我们在这里看到的 Git 配置优先级将在 *第十三章* 中详细描述，*定制和* *扩展 Git*。

### SSH 的公钥认证。

对于 SSH 传输协议，除了密码之外，还有其他认证机制。其中之一是 `gitolite` 使用的—[`gitolite.com`](https://gitolite.com)。

公钥认证的思路是，用户创建一个 `ssh-keygen`。然后将公钥发送到服务器，例如，使用 `ssh-copy-id`（它还会将公钥 `*.pub` 添加到远程服务器的 `~/.ssh/authorized_keys` 文件的末尾），或者将其粘贴到托管服务的网页表单中。然后你可以使用存储在本地机器上的私钥进行登录，例如 `~/.ssh/id_rsa`。如果不是默认身份密钥，你可能需要配置 SSH（在 Linux 上的 `~/.ssh/config`，在 MS Windows 上有类似的配置文件）以为给定的连接（主机名）使用特定的身份文件。

使用公钥认证的另一种便捷方法是使用认证代理，例如 `ssh-agent`（或者在 MS Windows 上使用 PuTTY 的 Pageant）。利用代理也使得使用带有密码保护的私钥更加方便——你只需在添加密钥时提供一次密码给代理（这可能需要用户操作，例如，运行 `ssh-add` 对 `ssh-agent`）。

### 凭据助手。

一直重复输入相同的凭据可能会很麻烦。对于 SSH，你可以使用公钥认证，但对于其他传输方式没有真正的等价物。Git 凭据配置提供了两种方法，至少可以减少提问次数。

第一个是为给定的 **认证上下文** 配置默认用户名（如果 URL 中没有提供用户名）——例如，主机名：

```
[credential "https://git.example.com"]
    username = user
```

如果你没有安全存储凭据的方法，它会有所帮助。

第二种是使用外部程序，Git 可以从中请求用户名和密码 — **凭据助手**。这些程序通常与桌面环境或操作系统提供的安全存储（钥匙链、密钥环、钱包、凭据管理器等）进行接口。

默认情况下，Git 至少包含`cache`和`store`助手。`cache`助手（`git-credential-cache`）将凭据存储在内存中，有效期为短暂的时间；默认情况下，它会为用户名和密码缓存 15 分钟。`store`助手（`git-credential-store`）将*未加密*的凭据永久存储在磁盘上，仅由用户可读的文件中（类似于`~/.netrc`）；还有第三方`netrc`助手（`git-credential-netrc`）用于 GPG 加密的`netrc/authinfo`文件。

可以全局或每个身份验证上下文配置选择要使用的凭据助手及其选项，如前面的示例所示。全局凭据配置如下所示：

```
[credential]
    helper = cache --timeout=300
```

这将使 Git 使用`cache`凭据助手，它会缓存凭据 300 秒（5 分钟）。如果凭据助手名称不是绝对路径（例如，`/usr/local/bin/git-kde-credentials-helper`），Git 会在助手名称前加上`git credential-`前缀。你可以通过`git help -a | grep credential-`检查可用的凭据助手类型。Git for Windows 还包括可选的`git credential-helper-selection`。

存在使用桌面环境安全存储的凭据助手。当你使用它们时，只需提供密码一次，以解锁存储（某些助手可以在 Git 源代码的 `contrib/` 区域找到）。有适用于 GNOME 和 KDE 的`git-credential-libsecret`，适用于 macOS Keychain 的 `git-credential-osxkeychain`，以及适用于 Microsoft 跨平台 Git Credential Manager（GCM）的 `git-credential-manager`。

你还可以使用`git-credential-oauth`来避免设置个人访问令牌或 SSH 密钥。使用此解决方案时，第一次验证时，助手会打开浏览器窗口到主机。后续访问使用缓存的凭据。在这里，可以利用 Git 支持多个凭据助手的特性。GitHub、GitLab 和 Bitbucket 是支持 OAuth 身份验证的 Git 托管服务之一。

Git 将为最特定的身份验证上下文使用凭据配置，但如果你想通过路径名区分 HTTP URL（例如，在同一主机上为不同仓库提供不同的用户名），你需要将`useHttpPath`配置变量设置为`true`。如果为上下文配置了多个助手，则会依次尝试每个，直到 Git 获取到用户名和密码。

历史注释

在凭据助手引入之前，人们可以使用与桌面环境钥匙串接口的*askpass*程序——例如，**kwalletaskpass**（用于 KDE 钱包）或**git-password**（用于 macOS 钥匙串）。

# 将你的更改发布到上游

在*第六章*中，*Git 协作开发*节解释了各种仓库设置。在这里，我们将了解一些为项目做贡献的常见模式。我们将看到发布更改的主要选项。

在开始新的更改工作之前，你通常应该与主开发版本同步，将官方版本合并到你的仓库中。这部分内容以及维护者的工作将在*第九章*中，*合并* *更改* *一起* 中描述。

## 推送到公共仓库

在**集中式工作流**中，发布你的更改仅仅是将其**推送**到中央服务器，如*图 6.2*所示。因为你与其他开发者共享这个中央仓库，所以有可能其他人已经推送到你试图更新的分支（即非快进的情况）。在这种情况下，你需要拉取（获取并合并，或获取并变基）其他人的更改，然后才能推送你的更改。这种情况在*第一章*中“*更新你的仓库（使用合并）*”一节开始时有说明，位于《*Git 基础* *实践*》中。

另一种类似工作流的系统是，当你的团队将每一组更改提交到代码审查系统时——例如，Gerrit。一种可用的选项是将更改推送到一个特殊的引用`refs/for/<branchname>`（这个引用是以目标分支命名），并推送到一个特殊的仓库中。然后，变更审查服务器会将每一组更改自动放置到一个单独的每组引用中（例如，`refs/changes/<change-id>`，适用于属于给定更改 ID 系列的提交）。

重要提示

在点对点（见*图 6.3*）和维护者工作流，或分层工作流变体（分别见*图 6.4*和*图 6.5*）中，将更改纳入项目的第一步是执行推送操作，但推送到*你自己的*“公共”仓库（对相应组可见）中的项目分支。然后，你需要请求你的共同开发者或项目维护者合并你的更改。你可以通过生成**拉取请求**来实现这一点，如下所述。

## 生成拉取请求

在除集中式工作流以外的所有工作流中，需要向共同开发者、维护者或集成经理发送通知，告知公共仓库中有可用的更改。`git request-pull` 命令可以帮助完成此步骤。给定起始点（即感兴趣的修订范围的底部），远程公共仓库的 URL 或名称，以及可选的提交结束点（如果不是 `HEAD`），此命令将生成更改的摘要：

```
$ git request-pull origin/master publish
The following changes since commit  ba5807e44d75285244e1d2eacb1c10cbc5cf3935:
  Merge: strtol() + checks (2014-05-31 20:43:42 +0200)
are available in the Git repository at:
  https://git.example.com/random master
for you to fetch changes up to 82006acd359717624fb33a7ae554cba6be717911:
  Merge branch 'master' of https://git.company.com/random (2021-05-30 00:58:23 +0200)
-----------------------------------------------------------
Alice Developer (1):
      Support optional <count> parameter
 src/rand.c |   26 +++++++++++++++++++++-----
 1 files changed, 21 insertions(+), 5 deletions(-)
```

拉取请求包含更改基准的 SHA-1（即提议拉取的系列中第一个提交之前的修订）、基准提交的标题、URL、公共仓库的分支（适合用作 `git pull` 参数）、最终提交的标题、`git shortlog` 输出和 `git diff --stat` 输出。这些输出可以发送给维护者——例如，通过电子邮件。

![](img/B21194_07_01.jpg)

图 7.1 – 在 GitHub 的分支列表中显示的“新建拉取请求”操作

许多 Git 托管软件和服务都包括与 `git request-pull` 相对应的内置功能（例如，GitHub 中的 **创建拉取请求** 操作）。

## 交换补丁

许多大型项目（以及许多开源项目）已经建立了接受补丁形式更改的流程，例如，降低贡献的门槛。如果你想向一个项目发送一次性的代码提案，但不打算成为常规贡献者，发送补丁可能比完整的协作设置更容易（在集中式工作流中获得提交权限、设置个人公共仓库进行分支和类似工作流——在 GitHub 上，这将包括 **fork** 项目）。此外，任何兼容工具都可以生成补丁，项目可以接受补丁，无论版本控制设置如何。

提示

如今，随着各种免费 Git 托管服务的普及，可能更难设置电子邮件客户端以发送格式正确的补丁电子邮件——尽管像 *GitGitGadget*（用于向 Git 项目的邮件列表提交补丁）或较早的 *submitGit* 服务可以提供帮助。Git 本身也包括发送邮件的命令，即 **git send-email** 和 **git imap-send**，这两者都需要配置。

此外，补丁作为更改的文本表示形式，可以被计算机和人类轻松理解。这使得它们具有普遍的吸引力，并且在 *代码审查* 过程中非常有用。许多开源项目历史上使用公共邮件列表进行此目的：你可以将补丁通过电子邮件发送到这个列表，公众可以审查并评论你的更改（通过 *public-inbox* 和 *lore+lei* 等服务，即使没有订阅邮件列表，也可以进行此操作）。

要生成每个提交系列的电子邮件版本，并将其转换为 mbox 格式的补丁，可以使用 `git format-patch` 命令，如下所示：

```
$ git format-patch -M -1
0001-Support-optional-count-parameter.patch
```

你可以使用任何修订范围说明符与此命令配合使用。最常用的是通过提交数量来限制，如前面的示例所示，或者使用双点修订范围语法——例如，`@{u}..`（见*第四章*，*探索项目历史*，*双点符号*部分）。在生成大量补丁时，选择一个目录来保存生成的补丁通常是很有用的。可以使用`-o <directory>`选项来实现。`git format-patch`的`-M`选项（传递给`git diff`）开启了重命名检测；这可以使补丁更小、更容易审查。

补丁文件最终看起来是这样的：

```
From db23d0eb16f553dd17ed476bec731d65cf37cbdc Mon Sep 17 00:00:00 2001
From: Alice Developer <alice@company.com>
Date: Sat, 31 May 2014 20:25:40 +0200
Subject: [PATCH] Initialize random number generator
Signed-off-by: Alice Developer
---
 random.c |    2 ++
 1 files changed, 2 insertions(+), 0 deletions(-)
diff --git a/random.c b/random.c
index cc09a47..5e095ce 100644
--- a/random.c
+++ b/random.c
@@ -1,5 +1,6 @@
 #include <stdio.h>
 #include <stdlib.h>
+#include <time.h>
 int random_int(int max)
@@ -15,6 +16,7 @@ int main(int argc, char *argv[])
int max = atoi(argv[1]);
+    srand(time(NULL));
     int result = random_int(max);
     printf("%d\n", result);
--
2.42.0
```

它实际上是一个完整的 mbox 格式的电子邮件。主题（去掉`[PATCH]`前缀后）以及三连字符线（`---`）之前的所有内容构成了提交信息——即更改的描述。要将其发送到邮件列表或开发人员，可以使用`git send-email`或`git imap-send`，或者任何能够发送纯文本邮件的电子邮件客户端。维护者随后可以使用`git am`来应用补丁系列，自动创建提交；有关更多内容，请参阅*第九章*，*合并更改*部分，*应用补丁系列*章节。

补丁的邮件主题约定

**[PATCH]**前缀是为了更容易区分补丁与其他电子邮件。这个前缀可以——并且通常会——包括额外的信息，如补丁系列中的编号（集合）、系列的修订版、关于它是**进行中的工作**（**WIP**）或**征求意见**（**RFC**）状态的信息——例如，**[****RFC/PATCHv4 3/8]**。

你也可以编辑这些补丁文件，为未来的审阅者添加更多信息——例如，关于替代方法的信息、补丁的前几个版本（之前的尝试）之间的差异，或者实现补丁的讨论的摘要和/或参考资料（例如，邮件列表中的讨论）。你可以在`---`行和补丁开始之间的地方添加此类文本，在更改摘要（`diffstat`）之前；`git am`会忽略这些内容。

提示——范围差异

如果补丁系列正在修订并且需要以不同的方式重新执行，推荐的做法是在封面信中提供**git range-diff**输出，显示该系列的一个版本与另一个版本之间的差异。

# 总结

在本章中，我们学习了如何选择传输协议（如果远程服务器提供此类选择），以及一些技巧，比如将外部仓库当作本地 Git 仓库使用和使用离线传输（通过 bundle）。

与远程仓库的联系可能需要提供凭证——通常是用户名和密码，以便能够执行例如推送到仓库等操作。本章描述了 Git 如何通过凭证助手帮助简化这部分内容。

发布你的更改并将其推送到上游可能涉及不同的机制，这取决于工作流。本章介绍了 push、pull 请求和基于补丁的技术。

以下两章扩展了协作主题：*第八章*，*高级分支技巧*，探讨了本地分支和远程仓库分支之间的关系，以及如何为协作设置分支；而 *第九章*，*合并更改*，则讨论了相反的问题——如何将并行工作的结果合并在一起。

# 问题

请回答以下问题，以测试你对本章内容的掌握情况：

1.  当连接到主机的网络不太稳定时，如何克隆一个大型仓库，但你可以登录到主机并访问远程仓库？

1.  在集中式工作流中，如何将更改提交到规范仓库？在集成管理者工作流中需要做些什么？

1.  如何设置 Git，使得只需提供一次密码，而不是每次与远程连接时都需要输入？

1.  你能否使用 Git 与外国版本控制系统的仓库进行交互，提交提交并下载更新？

# 答案

以下是上述问题的答案：

1.  一种可能的解决方案是使用 **git bundle** 在远程主机上创建打包文件，并通过可恢复的传输方式（如 HTTPS、rsync 或 BitTorrent）发送该文件，或通过可移动存储介质（如 USB 闪存驱动器）传输该文件。

1.  在集中式工作流中，你需要将更改推送到指定的中央规范仓库，这可能需要先合并其他人的更改；在集成管理者工作流中，你需要将更改推送到公共仓库，并发送某种形式的 pull 请求（例如，使用 **git request-pull** 和电子邮件）到规范仓库，或者通过电子邮件将补丁发送给维护者。

1.  你可以根据操作系统和桌面环境设置合适的凭证助手；对于 SSH 传输，你还可以使用 **ssh-agent** 或等效工具。

1.  使用适当的工具，你可以使用 Git 作为外部版本控制系统的客户端（例如，**git svn**），或使用远程传输助手将外部仓库当作 Git 远程仓库来使用（例如，**git-cinnabar**）。

# 深入阅读

若要了解本章中涉及的更多内容，请查看以下资源：

+   Scott Chacon 和 Ben Straub: *Pro Git（第 2 版）*（2014 年）[`git-scm.com/book/en/v2`](https://git-scm.com/book/en/v2)

    +   *第 7.12 章 Git 工具 –* *打包*

    +   *第 7.14 章 Git 工具 -* *凭证存储*

+   打包 URI: [`git-scm.com/docs/bundle-uri`](https://git-scm.com/docs/bundle-uri)

+   Anthony Heddings: *你应该使用 HTTPS 还是 SSH 来进行 Git 操作？*（2021 年）[`www.howtogeek.com/devops/should-you-use-https-or-ssh-for-git/#why-use-https`](https://www.howtogeek.com/devops/should-you-use-https-or-ssh-for-git/#why-use-https)

+   *SSH 隧道* 的视觉指南 [`robotmoon.com/ssh-tunnels/`](https://robotmoon.com/ssh-tunnels/)

+   Carl Tashian: *SSH 技巧与窍门* – *为你的 SSH 登录添加第二因素认证*（2020） [`smallstep.com/blog/ssh-tricks-and-tips/#add-a-second-factor-to-your-ssh`](https://smallstep.com/blog/ssh-tricks-and-tips/#add-a-second-factor-to-your-ssh)

+   Greg Kroah-Hartman: *“刻入石板的补丁”*，或为何 Linux 内核开发者依赖纯文本邮件，2016 年 Kernel Recipes 演讲 [`kernel-recipes.org/en/2016/talks/patches-carved-into-stone-tablets/`](https://kernel-recipes.org/en/2016/talks/patches-carved-into-stone-tablets/)
