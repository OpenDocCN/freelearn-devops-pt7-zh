# 第一章：开始使用 Ansible

信息与通信技术（ICT）常被描述为一个快速增长的行业。我认为 ICT 行业的最佳品质与其能以超高速度增长无关，而在于其能以惊人的速度革新自己和整个世界。

每隔 10 到 15 年，这个行业就会发生重大变化，每一次变化都解决了之前很难管理的问题，同时创造了新的挑战。此外，在每次重大变化中，许多前一轮的最佳实践被视为反模式，新的最佳实践则应运而生。尽管这些变化看似难以预测，但事实并非总是如此。显然，我们无法精确知道会发生什么变化，也无法预知这些变化何时发生，但观察拥有大量服务器和代码行数的公司，通常能揭示下一步会是什么。

当前的转变已经在像亚马逊 Web 服务、Facebook 和 Google 这样的公司中发生。这就是实施 IT 自动化系统来创建和管理服务器的过程。

在本章中，我们将涵盖：

+   IT 自动化

+   什么是 Ansible？

+   安全外壳

+   安装 Ansible

+   使用 QEMU 和 KVM 创建测试环境

+   版本控制系统

+   使用 Ansible 与 Git

# IT 自动化

IT 自动化广义上是指帮助管理 IT 基础设施（服务器、网络和存储）的过程和软件。在当前的转变中，我们正在见证这种过程和软件的巨大实施。

## IT 自动化的历史

在 IT 历史的早期，服务器数量很少，通常需要大量的人力来使它们正常工作，通常每台机器需要一个以上的人来操作。随着时间的推移，服务器变得更加可靠且易于管理，因此单个系统管理员可以管理多个服务器。在那一时期，管理员手动安装软件，手动升级软件，并手动更改配置文件。显然，这是一项劳动密集型且容易出错的过程，因此许多管理员开始实施脚本和其他手段来简化工作。这些脚本（通常）相当复杂，且不太具备可扩展性。

本世纪初，随着企业需求的增长，数据中心开始迅速扩展。虚拟化有助于保持低成本，而许多服务是 Web 服务，这意味着许多服务器彼此非常相似。此时，需要新的工具来替代之前使用的脚本，即配置管理工具。

**CFEngine** 是上世纪 90 年代最早展示配置管理能力的工具之一；最近出现了 Puppet、Chef、Salt，以及 Ansible。

## IT 自动化的优势

人们常常会疑惑，考虑到实施 IT 自动化会有一些直接和间接的成本，是否真的带来了足够的好处。IT 自动化的主要优点是：

+   能够快速配置机器

+   能够在几分钟内从头创建一台机器

+   能够追踪基础设施上执行的任何更改

因此，通过减少系统管理员经常执行的重复性操作，可以降低管理 IT 基础设施的成本。

## IT 自动化的缺点

和其他技术一样，IT 自动化也有一些缺点。依我看，这些是最大的缺点：

+   自动化所有曾经用于培训新系统管理员的小任务

+   如果发生错误，它将被传播到各个地方

第一个后果是，需要实施新的方式来培训初级系统管理员。

### 限制错误传播可能带来的损害

第二个问题更棘手。有很多方法可以限制这种损害，但没有任何方法可以完全防止它。以下是可用的缓解选项：

+   **始终有备份**：备份不能防止你摧毁机器，它们只是让恢复过程成为可能。

+   **始终在非生产环境中测试你的基础设施代码（剧本/角色）**：公司已经开发了不同的流水线来部署代码，这些流水线通常包括开发、测试、预发布和生产等环境。使用相同的流水线来测试你的基础设施代码。如果一个有问题的应用程序进入生产环境，可能会出现问题。如果一个有问题的剧本进入生产环境，可能会造成灾难。

+   **始终进行基础设施代码的同行评审**：一些公司已经为应用程序代码引入了同行评审，但很少有公司为基础设施代码引入同行评审。正如我在前面提到的，我认为基础设施代码比应用程序代码更为关键，所以无论是否对应用程序代码进行同行评审，你都应该始终进行基础设施代码的同行评审。

+   **启用 SELinux**：SELinux 是一个安全内核模块，适用于所有 Linux 发行版（在 Fedora、Red Hat Enterprise Linux、CentOS、Scientific Linux 和 Unbreakable Linux 中默认安装）。它允许你以非常细粒度的方式限制用户和进程的权限。我建议使用 SELinux，而不是其他类似模块（如 AppArmor），因为它能处理更多的情况和权限。SELinux 将防止大量的损害，因为如果正确配置，它将防止许多危险命令的执行。

+   **从有限账户运行 playbook**：尽管用户和权限提升方案在 UNIX 代码中已经存在了 40 多年，但似乎并不是很多公司在使用它们。为所有 playbook 使用一个有限用户，并且只为需要更高权限的命令提升权限，将有助于在尝试清理应用程序临时文件夹时防止误操作导致机器毁灭。

+   **使用水平权限提升**：`sudo`是一个众所周知的命令，但通常以更危险的形式使用。`sudo`命令支持`-u`参数，允许你指定要模拟的用户。如果你必须更改由另一个用户拥有的文件，请不要升级为`root`，而是升级为那个用户。在 Ansible 中，你可以使用`become_user`参数来实现这一点。

+   **可能时，不要同时在所有机器上运行 playbook**：分阶段部署可以帮助你在为时已晚之前检测到问题。有许多问题在开发、测试、暂存和质量保证环境中无法检测到。其中大多数与很难在这些非生产环境中正确模拟的负载相关。你刚刚添加到 Apache HTTPd 或 MySQL 服务器的新配置在语法上可能完全正确，但在生产负载下可能对你的特定应用程序造成灾难性影响。分阶段部署将允许你在实际负载下测试新配置，而不会在出现问题时面临停机风险。

+   **避免猜测命令和修改器**：许多系统管理员会试图记住正确的参数，如果记不清楚就会猜测。我也经常这样做，但这是非常危险的。查看手册页或在线文档通常不会花费超过两分钟的时间，并且经常通过阅读手册，你会发现一些有趣的注记你之前并不知道。猜测修改器是危险的，因为你可能会被非标准的修改器愚弄（比如，`-v`不是`grep`的详细模式，`-h`不是 MySQL CLI 的帮助命令）。

+   **避免容易出错的命令**：并非所有命令都是平等的。有些命令比其他命令（要）危险得多。如果你认为`cat`命令是安全的，那么你必须认为`dd`命令是危险的，因为`dd`用于复制和转换文件和卷。我见过很多人在脚本中使用`dd`来将 DOS 文件转换为 UNIX（而不是使用`dos2unix`），还有其他很多非常危险的例子。请避免使用这些命令，因为如果出现问题，可能会造成巨大的灾难。

+   **避免不必要的修饰符**：如果你需要删除一个简单的文件，请使用`rm ${file}`而不是`rm -rf ${file}`。后者通常是那些学会了"为了保险起见，总是使用`rm -rf`"的用户执行的，因为在他们的过去，可能需要删除一个文件夹。这将防止你在`${file}`变量设置错误时误删整个文件夹。

+   **始终检查如果变量未设置可能发生的情况**：如果你想删除文件夹中的内容，并使用`rm -rf ${folder}/*`命令，可能会遇到麻烦。如果由于某种原因，`${folder}`变量没有设置，shell 会读取`rm -rf /*`命令，这将非常危险（考虑到`rm -rf /`命令在大多数当前操作系统上不起作用，因为它需要`--no-preserve-root`选项，而`rm -rf /*`命令则会按预期工作）。我使用这个特定的命令作为例子，因为我曾经见过类似的情况：变量是从数据库中提取的，由于某些维护工作，数据库出现故障，导致该变量被赋值为空字符串。接下来发生的事情可能很容易猜到。如果你无法避免在危险的地方使用变量，至少在使用之前检查它们是否为空。这不能解决所有问题，但可以捕获一些最常见的问题。

+   **仔细检查你的重定向**：重定向（以及管道）是 Linux Shell 中最强大的元素。它们也可能非常危险：一个`cat /dev/rand > /dev/sda`命令可以摧毁一个磁盘，即使`cat`命令通常是被忽略的，因为它通常不危险。始终仔细检查所有包含重定向的命令。

+   **尽可能使用特定模块**：在这个列表中，我使用了 Shell 命令，因为许多人会尝试将 Ansible 当作仅仅是分发 Shell 命令的工具来使用：它不是。Ansible 提供了很多模块，我们将在本书中看到它们。它们将帮助你创建更加可读、可移植和安全的 Playbooks。

## IT 自动化类型

有很多方法来分类 IT 自动化系统，但最重要的还是与配置如何传播有关。基于此，我们可以区分基于代理的系统和无代理的系统。

### 基于代理的系统

基于代理的系统有两个不同的组件：一个**服务器**和一个叫做**代理**的客户端。

只有一个服务器，它包含了整个环境的所有配置，而代理的数量与环境中的机器数量相同。

### 注意

在某些情况下，为了确保高可用性，可能会有多个服务器，但请将其视为单一服务器，因为它们将以相同的方式进行配置。

客户端会定期联系服务器，以查看是否有新的配置文件。如果有新的配置，客户端将下载并应用它。

### 无代理的系统

在无代理系统中，不存在特定的代理。无代理系统并不总是遵循服务器/客户端范式，因为可以有多个服务器，甚至服务器和客户端的数量相同。通信由服务器初始化，服务器将使用标准协议（通常是 SSH 或 PowerShell）联系客户端。

### 基于代理与无代理系统

除了上述差异外，还有其他一些因为这些差异而产生的对比因素。

从安全角度来看，基于代理的系统可能不如无代理的系统安全。由于所有机器都必须能够发起与服务器机器的连接，因此该机器可能比无代理系统更容易受到攻击，因为无代理系统中的机器通常位于防火墙后面，防火墙不会接受任何传入连接。

从性能角度来看，基于代理的系统存在服务器饱和的风险，因此部署可能会变慢。此外，还需要考虑到，在纯粹的基于代理的系统中，无法立即强制推送更新到一组机器。它必须等到那些机器检查到更新后才能应用。因此，多个基于代理的系统已经实现了带外等待以实现这一功能。像 Chef 和 Puppet 这样的工具是基于代理的，但也可以在没有中央服务器的情况下运行，从而扩展大量的机器，通常被称为 **无服务器 Chef** 和 **无主 Puppet**。

无代理系统更容易集成到现有基础架构中，因为它将被客户端视为普通的 SSH 连接，因此不需要额外的配置。

# 什么是 Ansible？

Ansible 是一款无需代理的 IT 自动化工具，于 2012 年由 *Michael DeHaan*（前 Red Hat 员工）开发。Ansible 的设计目标是：简洁、一致、安全、高度可靠且易于学习。Ansible 公司最近被 Red Hat 收购，现在作为 Red Hat, Inc. 的一部分运营。

Ansible 主要通过 SSH 以推送模式运行，但你也可以使用 `ansible-pull` 来运行 Ansible，方法是在每个代理上安装 Ansible，下载本地的 playbook，并在单独的机器上运行它们。如果有大量的机器（“大量”是一个相对概念；在我们看来，大于 500 并且需要并行更新的机器），并且你计划对这些机器进行并行更新，那么这种方式可能是最合适的选择。

# 安全外壳（SSH）

**安全外壳**（也称为 **SSH**）是一种网络服务，允许你通过完全加密的连接远程登录并访问一个 shell。SSH 守护进程如今已经成为 UNIX 系统管理的标准，取代了未加密的 telnet。SSH 协议最常用的实现是 OpenSSH。

在过去几个月中，微软展示了 OpenSSH 在 Windows 上的实现（截至本文写作时）。

由于 Ansible 执行 SSH 连接和命令的方式与其他任何 SSH 客户端相同，因此对 OpenSSH 服务器没有做特殊配置。

为了加速默认的 SSH 连接，你可以始终启用 `ControlPersist` 和管道模式，这样可以使 Ansible 更快且更安全。

# 为什么选择 Ansible？

在本书中，我们将尝试将 Ansible 与 Puppet 和 Chef 进行比较，因为许多人对这些工具有很好的使用经验。我们还将特别指出 Ansible 相对于 Chef 或 Puppet 解决问题的方法。

Ansible、Puppet 和 Chef 都是声明性的，旨在将机器移到配置中指定的期望状态。例如，在这些工具中，为了在某个时间点启动服务并在重启时自动启动，你需要编写一个声明性块或模块；每次工具在机器上运行时，它都会努力实现你在 **playbook**（Ansible）、**cookbook**（Chef）或 **manifest**（Puppet）中定义的状态。

在简单层面上，工具集之间的差异是最小的，但随着情况的增加和复杂性的提高，你将开始发现不同工具集之间的差异。在 Puppet 中，你需要注意顺序，每次在不同的主机上运行时，Puppet 服务器都会创建执行指令的顺序。为了充分利用 Chef 的功能，你需要一个优秀的 Ruby 团队。你的团队需要精通 Ruby 语言，才能定制 Puppet 和 Chef，这两款工具的学习曲线都较大。

使用 Ansible 的情况有所不同。它在执行顺序上借鉴了 Chef 的简洁性，采用自上而下的方式，并允许你以 YAML 格式定义最终状态，这使得代码非常易读，开发团队到运维团队的每个人都能轻松理解并做出修改。在许多情况下，甚至没有 Ansible 时，运维团队也会得到 playbook 手册，以便在遇到问题时执行指令。Ansible 模拟了这种行为。如果你发现你的项目经理因为其简洁性而修改 Ansible 代码并将其提交到 Git，不要感到惊讶！

# 安装 Ansible

安装 Ansible 既快速又简单。你可以直接使用源代码，通过从 GitHub 项目中克隆它（[`github.com/ansible/ansible`](https://github.com/ansible/ansible)），使用系统的包管理器进行安装，或者使用 Python 的包管理工具（**pip**）。你可以在任何 Windows、Mac 或类 UNIX 系统上使用 Ansible。Ansible 不需要任何数据库，也不需要运行任何守护进程。这使得维护 Ansible 版本和升级变得更加轻松，不会出现中断。

我们将安装 Ansible 的机器称为我们的 Ansible 工作站。有些人也称其为指挥中心。

## 使用系统的包管理器安装 Ansible

通过系统的包管理器安装 Ansible 是可行的，我个人认为，如果你的系统的包管理器提供至少 Ansible 2.0 的版本，这是首选方案。我们将探讨如何通过 **Yum**、**Apt**、**Homebrew** 和 **pip** 安装 Ansible。

### 通过 Yum 安装

如果你正在运行 Fedora 系统，你可以直接安装 Ansible，因为从 Fedora 22 开始，Ansible 2.0+ 已经可以在官方仓库中找到。你可以按如下方式安装：

```
$ sudo dnf install ansible

```

对于 RHEL 和基于 RHEL 的系统（CentOS、Scientific Linux、Unbreakable Linux），版本 6 和 7 在 EPEL 仓库中提供 Ansible 2.0+，因此你应确保在安装 Ansible 之前已启用 EPEL 仓库，如下所示：

```
$ sudo yum install ansible

```

### 注意

在 Cent 6 或 RHEL 6 上，你必须运行命令`rpm -Uvh`。有关如何安装 EPEL 的说明，请参阅[`dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm`](http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm)。

### 通过 Apt 安装

Ansible 可用于 Ubuntu 和 Debian。要在这些操作系统上安装 Ansible，请使用以下命令：

```
$ sudo apt-get install ansible

```

### 通过 Homebrew 安装

你可以通过 Homebrew 在 Mac OS X 上安装 Ansible，如下所示：

```
$ brew update
$ brew install ansible

```

### 通过 pip 安装

你可以通过 pip 安装 Ansible。如果你的系统没有安装 pip，请先安装它。你也可以使用 pip 在 Windows 上安装 Ansible，使用以下命令行：

```
$ sudo easy_install pip

```

你现在可以使用`pip`安装 Ansible，如下所示：

```
$ sudo pip install ansible

```

安装完成后，运行`ansible --version`来验证是否已安装：

```
$ ansible --version

```

你将从前面的命令行获得以下输出：

```
ansible 2.0.2

```

### 从源代码安装 Ansible

如果之前的方法不适合你的使用场景，你可以直接从源代码安装 Ansible。源代码安装不需要任何管理员权限。让我们克隆一个仓库并激活`virtualenv`，这是 Python 中的一个隔离环境，你可以在其中安装包而不会干扰系统的 Python 包。以下是克隆仓库的命令及其输出：

```
$ git clone git://github.com/ansible/ansible.git
Cloning into 'ansible'...
remote: Counting objects: 116403, done.
remote: Compressing objects: 100% (18/18), done.
remote: Total 116403 (delta 3), reused 0 (delta 0), pack-reused 116384
Receiving objects: 100% (116403/116403), 40.80 MiB | 844.00 KiB/s, done.
Resolving deltas: 100% (69450/69450), done.
Checking connectivity... done.
$ cd ansible/
$ source ./hacking/env-setup
Setting up Ansible to run out of checkout...
PATH=/home/vagrant/ansible/bin:/usr/local/bin:/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/sbin:/home/vagrant/bin
PYTHONPATH=/home/vagrant/ansible/lib:
MANPATH=/home/vagrant/ansible/docs/man:
Remember, you may wish to specify your host file with -i
Done!

```

Ansible 需要一些 Python 包，你可以使用`pip`安装。如果你的系统没有安装 pip，可以使用以下命令安装它。如果没有安装`easy_install`，你可以通过 Red Hat 系统上的 Python `setuptools` 包或通过 Mac 上的 Brew 安装它：

```
$ sudo easy_install pip
<A long output follows>

```

一旦你安装了 `pip`，使用以下命令安装 `paramiko`、`PyYAML`、`jinja2` 和 `httplib2` 包：

```
$ sudo pip install paramiko PyYAML jinja2 httplib2
Requirement already satisfied (use --upgrade to upgrade): paramiko in /usr/lib/python2.6/site-packages
Requirement already satisfied (use --upgrade to upgrade): PyYAML in /usr/lib64/python2.6/site-packages
Requirement already satisfied (use --upgrade to upgrade): jinja2 in /usr/lib/python2.6/site-packages
Requirement already satisfied (use --upgrade to upgrade): httplib2 in /usr/lib/python2.6/site-packages
Downloading/unpacking markupsafe (from jinja2)
 Downloading MarkupSafe-0.23.tar.gz
 Running setup.py (path:/tmp/pip_build_root/markupsafe/setup.py) egg_info for package markupsafe
Installing collected packages: markupsafe
 Running setup.py install for markupsafe
 building 'markupsafe._speedups' extension
 gcc -pthread -fno-strict-aliasing -O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector --param=ssp-buffer-size=4 -m64 -mtune=generic -D_GNU_SOURCE -fPIC -fwrapv -DNDEBUG -O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector --param=ssp-buffer-size=4 -m64 -mtune=generic -D_GNU_SOURCE -fPIC -fwrapv -fPIC -I/usr/include/python2.6 -c markupsafe/_speedups.c -o build/temp.linux-x86_64-2.6/markupsafe/_speedups.o
 gcc -pthread -shared build/temp.linux-x86_64-2.6/markupsafe/_speedups.o -L/usr/lib64 -lpython2.6 -o build/lib.linux-x86_64-2.6/markupsafe/_speedups.so
Successfully installed markupsafe
Cleaning up...

```

### 注意

默认情况下，Ansible 将运行在开发分支上。你可能想查看最新的稳定分支。使用以下命令行检查最新稳定版本：

**$ git branch -a**

复制你想使用的最新版本。2.0.2 版本是撰写本文时可用的最新版本。使用以下命令检查最新版本：

```
[node ansible]$ git checkout v2.0.2
Note: checking out 'v2.0.2'.
[node ansible]$ ansible --version
ansible 2.0.2 (v2.0.2 268e72318f) last updated 2014/09/28 21:27:25 (GMT +000)

```

现在，你有一个工作中的 Ansible 设置了。运行 Ansible 从源代码的好处之一是，你可以立即享受新特性，而无需等待包管理器为你提供它们。

# 使用 QEMU 和 KVM 创建测试环境

为了能够学习 Ansible，我们需要制作相当多的 playbook 并运行它们。

### 提示

直接在你的电脑上操作会非常危险。因此，我建议使用虚拟机。

虽然可以在几秒钟内使用云服务提供商创建测试环境，但通常更有用的是在本地拥有这些机器。为此，我们将使用**基于内核的虚拟机**（**KVM**）和**快速模拟器**（**QEMU**）。

第一件事是安装`qemu-kvm`和`virt-install`。在 Fedora 上，只需要运行：

```
$ sudo dnf install -y @virtualization

```

在 Red Hat/CentOS/Scientific Linux/Unbreakable Linux 上，只需运行：

```
$ sudo yum install -y qemu-kvm virt-install virt-manager

```

如果你使用 Ubuntu，你可以通过以下方式安装：

```
$ sudo apt install virt-manager

```

在 Debian 上，你需要执行：

```
$ sudo apt install qemu-kvm libvirt-bin

```

对于我们的示例，我将使用 CentOS 7。这有多个原因，主要有以下几点：

+   CentOS 是免费的，并且与 Red Hat、Scientific Linux 和 Unbreakable Linux 完全兼容。

+   许多公司使用 Red Hat/CentOS/Scientific Linux/Unbreakable Linux 来运行他们的服务器。

+   这些发行版是唯一内建 SELinux 支持的，正如我们之前看到的，SELinux 可以帮助你使环境更加安全。

在撰写本书时，最新的 CentOS 云镜像是 [`cloud.centos.org/centos/7/images/CentOS-7-x86_64-GenericCloud-1603.qcow2`](http://cloud.centos.org/centos/7/images/CentOS-7-x86_64-GenericCloud-1603.qcow2)，所以让我们使用以下命令下载这个镜像：

```
$ wget http://cloud.centos.org/centos/7/images/CentOS-7-x86_64-GenericCloud-1603.qcow2

```

由于我们可能需要创建许多机器，因此最好创建它的副本，以免修改原始版本：

```
$ cp CentOS-7-x86_64-GenericCloud-1603.qcow2 centos_1.qcow2

```

由于`qcow2`镜像将运行 `cloud-init` 来设置网络、用户等，我们需要提供几个文件。让我们从创建一个网络配置的元数据文件开始：

```
instance-id: centos_1 
local-hostname: centos_1.local 
network-interfaces: | 
  iface eth0 inet static 
  address (An IP in your virtual bridge class) 
  network (The first IP of the virtual bridge class) 
  netmask (Your virtual bridge class netmask) 
  broadcast (Your virtual bridge class broadcast) 
  gateway (Your virtual bridge class gateway) 

```

要查找你的虚拟桥接数据，你需要寻找一个名称为`virbrX`或类似名称的设备，在我的情况下是`virtbr0`，所以我可以使用以下命令查找它的所有信息：

```
$ ip addr show virbr0

```

上一个命令的输出如下：

```
5: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default qlen 1000
 link/ether 52:54:00:38:1a:e6 brd ff:ff:ff:ff:ff:ff
 inet 192.168.124.1/24 brd 192.168.124.255 scope global virbr0
 valid_lft forever preferred_lft forever

```

所以，对我来说，元数据文件看起来像这样：

```
instance-id: centos_1 
local-hostname: centos_1.local 
network-interfaces: | 
  iface eth0 inet static 
  address 192.168.124.10 
  network 192.168.124.1 
  netmask 255.255.255.0 
  broadcast 192.168.124.255 
  gateway 192.168.124.1 

```

这个文件将在虚拟机启动时设置 `eth0` 接口。我们还需要另一个文件（user-data）来正确设置 `users`：

```
users: 
- name: (yourname) 
  shell: /bin/bash 
  sudo: ['ALL=(ALL) NOPASSWD:ALL'] 
  ssh-authorized-keys: 
  - (insert ssh public key here) 

```

对我来说，文件看起来像这样：

```
users: 
- name: fale 
  shell: /bin/bash 
  sudo: ['ALL=(ALL) NOPASSWD:ALL'] 
  ssh-authorized-keys: 
  - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDRoZzfNif+wXFqzsmvHg4jJt8+ZO/dQxm5k7pXYAwdWVbiFrZYGhMQl5FPfzC7rkDaC31fod3Y85QkQVgNKCVYUy5QR5LfxUjSQDv+y2Nfao4be/BKla0ffc7JVSzFFAELGGDLn1lMN0e0D9syqQbKgSRdOdvweq/0Et3KNIF9e7XgEdSuAHls17NDtMkWUfyi5yvEtdtMcp9gO4OlG6Vh0iCXOdx+f0QA2hh1JnvePvzJ4a8CeckN5JwL7Q027nlsHPBYq9K1jvv+diUs48FflPJI4fgMq3Zo7zyCpf8qE7Dlx+u7OvR5kxNdrpnOsDgHeAGNkrzfcmxU7kbU29NX4VFgWd0sdlzu1nOWFEH7Cnd547tx5VFxBzJwEAUCh7QSiU2Ne/hCnjFkZuDZ5pN4pNw+yu+Feoz79gV/utoLHuCodYyAvSQlQ7VSfC+djLD/9wHC2yGksvc9ICnSUv3JyQEEEG4K26z6szF9+a3vU0qIq7YYa8QHgWIHtzSxztYRIWJOzTZlwyuNmhbRNYDaMC5BMzvQ8JREv0obMLmrlvolJPWT4gn1N9sDNNXIC6RDRE5yGsIEf0CliYW1X/8XG40U+g9LG+lrYOGWD4OymZ2P/VDIzZbVT6NG/rdSSGnf4D1AwlOGR7eNTv30AK9o0LVjqGaJWKWYUF9zY6I3+Q== 

```

为了在启动时提供这些文件，我们需要创建一个包含它们的 ISO 文件：

```
$ genisoimage -output centos_1.iso -volid cidata -joliet -rock user-data meta-data

```

在 ISO 文件准备好后，我们可以指示`virt-install`来实际创建虚拟机：

```
virt-install --name CentOS_1 \ 
--ram 2048 \ 
--disk centos_1.qcow2 \ 
--vcpus 2 \ 
--os-variant fedora21 \ 
--connect qemu:///system \ 
--network bridge:br0,model=virtio \ 
--cdrom centos_1.iso \ 
--boot hd 
virt-install --name CentOS_1 \ --ram 2048 \ --disk centos_1.qcow2 \ --vcpus 2 \ --os-variant fedora21 \ --connect qemu:///system \ --network bridge:br0,model=virtio \ --cdrom centos_1.iso \ --boot hd 

```

由于我们的网络配置在 ISO 文件中，我们在每次启动时都需要它。遗憾的是，默认情况下不会发生这种情况，因此我们需要做一些额外的步骤。首先，运行`virsh`：

```
$ virsh

```

此时，应该出现一个 `virsh` 命令行，输出如下：

```
Welcome to virsh, the virtualization interactive terminal.
Type:  'help' for help with commands
 'quit' to quit
virsh #

```

这意味着我们从 bash（或者如果你没有使用 bash 的话，你的 shell）切换到了虚拟化 shell。请输入以下命令：

```
virsh # edit CentOS_1

```

通过这样做，我们将能够调整`CentOS_1`机器的配置。在磁盘部分，你需要找到应该如下所示的`cdrom`设备：

```
    <disk type='block' device='cdrom'> 
      <driver name='qemu' type='raw'/> 
      <target dev='hda' bus='ide'/> 
      <readonly/> 
      <address type='drive' controller='0' bus='0' target='0'
      unit='0'/> 
    </disk> 

```

你需要将其更改为如下所示，已加粗的部分：

```
    <disk type='file' device='cdrom'> 
      <driver name='qemu' type='raw'/> 
        <source file='(Put here your ISO path)/centos_1.iso'/> 
      <target dev='hda' bus='ide'/> 
      <readonly/> 
      <address type='drive' controller='0' bus='0' target='0'
      unit='0'/> 
    </disk> 

```

此时，我们的虚拟机将始终以挂载 ISO 文件作为`cdrom`启动，因此`cloud-init`将能够正确初始化网络。

# 版本控制系统

在本章中，我们已经遇到过表达式*基础设施代码*，用来描述将创建和维护你的基础设施的 Ansible 代码。我们使用“基础设施代码”这个术语，以便与应用程序代码区分开来，应用程序代码是构成你的应用、网站等的代码。这样的区分是为了清晰起见，但最终这两者都是一些文本文件，软件可以读取并解释它们。

因此，版本控制系统将对你大有帮助。它的主要优点包括：

+   能够让多人同时在同一个项目上工作。

+   能够以简单的方式进行代码审查。

+   能够为多个环境（即开发、测试、质量保证、预发布和生产）创建多个分支。

+   能够追踪一个变更，以便我们知道它是什么时候引入的，以及是谁引入的。这使得我们在几年（或几个月）后，理解那段代码为何存在变得更加容易。

这些优点是大多数版本控制系统所提供的。

版本控制系统可以根据它们能够实现的三种不同模型，分为三个主要类别：

+   本地数据模型

+   客户端-服务器模型

+   分布式模型

第一类，本地数据模型，是最古老的（大约 1972 年）方法，通常用于非常具体的应用场景。此模型要求所有用户共享相同的文件系统。著名的例子有**修订控制系统**（**RCS**）和**源代码控制系统**（**SCCS**）。

第二类，即客户端-服务器模型，稍晚出现（大约在 1990 年），旨在解决本地数据模型的局限性，创建了一个遵循本地数据模型的服务器，并设立了一组客户端与服务器交互，而不是直接与仓库本身交互。这一附加层使得多个开发人员能够使用本地文件，并将其与集中式服务器进行同步。此方法的著名例子有 Apache **Subversion**（**SVN**）和**并行版本控制系统**（**CVS**）。

第三类模型，分布式模型，在二十一世纪初出现，试图解决客户端-服务器模型的局限性。事实上，在客户端-服务器模式下，你可以离线工作，但需要*在线*才能提交更改。分布式模型允许你在本地仓库（如本地数据模型）上处理所有内容，并能轻松地合并不同机器上的不同仓库。在这个新模型中，所有操作都可以像在客户端-服务器模型中一样执行，且增加了能够完全离线工作以及在不经过集中式服务器的情况下合并对等节点间更改的优势。此模型的例子有 BitKeeper（专有软件）、Git、GNU Bazaar 和 Mercurial。

只有分布式模型才能提供的一些额外优势包括：

+   即使服务器不可用，也能进行提交、浏览历史记录和执行任何其他操作的可能性

+   更容易管理不同环境的多个分支

当涉及到基础设施代码时，我们必须考虑到，通常，保存和管理基础设施代码的基础设施本身也包含在基础设施代码中。这是一个递归的情况，可能会导致问题。事实上，在没有代码服务器之前，你无法部署你的 Ansible，而在没有 Ansible 之前，你无法部署你的代码服务器。分布式版本控制系统将避免这个问题。

至于管理多个分支的简易性，尽管这不是硬性规则，但通常分布式版本控制系统比其他版本控制系统具有更好的合并处理能力。

# 使用 Ansible 与 Git

由于我们刚刚看到的原因以及其巨大的流行性，我建议始终在你的 Ansible 仓库中使用 Git。

我经常给我所谈论的人提供一些建议，以便 Ansible 能从 Git 中获得最大收益：

+   **创建环境分支**：创建诸如 dev、prod、test 和 stg 等环境分支，可以帮助你轻松追踪不同环境及其各自的更新状态。我常建议将 master 分支用于开发环境，因为我发现很多人习惯直接将新更改推送到 master。如果你将 master 用作生产环境，可能会不小心将更改推送到生产环境，而原本想推送到开发环境。

+   **始终保持环境分支的稳定性**：拥有环境分支的一个大优势是，可以随时从零开始销毁并重新创建任何环境。这只有在环境分支处于稳定（未损坏）状态时才可能实现。

+   **使用特性分支**：为特定的大型开发特性（如重构或其他大改动）使用不同的分支，将使你能够在 Git 仓库中保留你的日常操作，同时进行新特性的开发（这样你就不会失去对谁做了什么、什么时候做的记录）。

+   **频繁推送**：我总是建议人们尽可能频繁地*推送提交*。这将使 Git 同时充当版本控制系统和备份系统。我曾多次看到笔记本电脑损坏、丢失或被盗，而上面有几天甚至几周没有推送的工作。不要浪费时间，频繁推送。另外，频繁推送还能让你更早发现合并冲突，而冲突在被及时发现时更容易处理，而不是等到有多个更改后再去解决。

+   **每次做出更改后都要部署**：我曾见过有开发者在基础设施代码中做了更改，在开发和测试环境中进行测试，推送到生产分支后去吃午餐，却没有立即在生产环境中部署更改。结果他的午餐没有吃好。因为他的同事不小心将代码部署到了生产环境（他当时正尝试部署自己所做的小改动），并且没有准备好处理另一个开发者的部署。生产环境出现了问题，他们花了大量时间搞清楚为什么一个这么小的更改（部署者知道的那个）会导致如此大的麻烦。

+   **选择多个小改动，而不是几个大改动**：尽可能进行小改动，将使调试更容易。调试基础设施并不容易。没有编译器可以帮助你发现*明显的问题*（即使 Ansible 会对代码进行语法检查，也不会执行其他测试），而且用来查找问题的工具并不像你想象的那样好。基础设施即代码的理念还很新，相关工具还不如应用代码的工具那么成熟。

+   **尽量避免二进制文件**：我总是建议将二进制文件保存在 Git 仓库之外，无论是应用程序代码仓库还是基础设施代码仓库。在应用程序代码的例子中，我认为保持仓库轻便非常重要（Git 以及大多数版本控制系统在处理二进制大块数据时表现不佳），而在基础设施代码的例子中尤为关键，因为你会很容易将大量二进制文件放入仓库，因为很多时候将二进制文件放进仓库比找到一个更清洁（更好的）解决方案更为简便。

# 总结

在本章中，我们了解了什么是 IT 自动化、它的优点和缺点、可以找到哪些工具，以及 Ansible 在这一大框架中的作用。我们还了解了如何安装 Ansible 以及如何创建基于 KVM 的虚拟机。最后，我们分析了版本控制系统，并讨论了如果正确使用，Git 为 Ansible 带来的优势。

在下一章，我们将开始探讨本章提到的基础设施代码，虽然我们还没有解释它是什么以及如何编写。此外，在下一章中，我们还将学习如何自动化一些你可能每天都会执行的简单操作，比如管理用户、管理文件以及文件内容。
