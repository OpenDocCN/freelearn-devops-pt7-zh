# 第一章：开始使用 Ansible

**Ansible** 与目前市面上其他配置管理工具有着根本的不同。它的设计目的就是在几乎所有方面都让配置变得简单，从它简单的英文配置语法到易于设置的特点。你会发现，Ansible 让你不再需要编写自定义的配置和部署脚本，而是可以直接专注于你的工作。

Ansible 只需要在你用来管理基础设施的机器上安装。它不需要在被管理的机器上安装客户端，也不需要在使用之前设置任何服务器基础设施。你甚至可以在安装后几分钟内就开始使用它，正如我们在本章中将要展示的那样。

本章涵盖的主题如下：

+   安装 Ansible

+   配置 Ansible

+   从命令行使用 Ansible

+   使用 Ansible 管理 Windows 机器

+   如何获取帮助

# 所需的硬件和软件

你将从一台机器的命令行使用 Ansible，这台机器我们称之为 **控制机器**，然后使用它配置另一台机器，我们称之为 **被管理机器**。Ansible 目前只支持 Linux 或 OS X 控制机器；然而，被管理机器可以是 Linux、OS X、其他类 Unix 机器或 Windows。Ansible 对控制机器的要求不高，对被管理机器的要求更少。

控制机器的要求如下：

+   Python 2.6 或更高版本

+   paramiko

+   PyYAML

+   Jinja2

+   httplib2

+   基于 Unix 的操作系统

被管理的机器需要 Python 2.4 或更高版本以及 simplejson；然而，如果你的 Python 版本是 2.5 或更高，你只需要 Python。被管理的 Windows 机器需要开启 Windows 远程管理，并且 PowerShell 版本要大于 3.0。虽然 Windows 机器的要求更多，但所有这些工具都是免费的，Ansible 项目甚至包括了帮助你轻松设置依赖项的脚本。

# 安装方法

如果你想使用 Ansible 管理一组现有的机器或基础设施，你可能希望使用这些系统上附带的包管理器。这意味着你将随着分发版的更新而获得 Ansible 的更新，可能会落后于其他方法的几个版本。然而，这也意味着你将运行一个已经过测试并能在你使用的系统上正常工作的版本。

如果你正在运行一个现有的基础设施，但需要一个更新版本的 Ansible，你可以通过 pip 安装 Ansible。**Pip** 是一个用于管理 Python 软件包和库的工具。Ansible 的发布版本会在发布后立即推送到 pip，因此如果你使用的是最新的 pip，你应该总是运行最新版本。

如果你打算开发大量模块并可能为 Ansible 贡献代码，你应该使用从源代码安装的版本。由于你将运行最新且未经充分测试的版本，可能会遇到一些小问题。

## 从发行版安装

大多数现代发行版都包括一个包管理器，可以自动管理包的依赖关系和更新。这使得通过包管理器安装 Ansible 成为启动 Ansible 的最简单方式；通常只需要一个命令即可完成安装。它还会随着你的机器更新而自动更新，尽管可能会落后一两个版本。以下是在最常见的发行版上安装 Ansible 的命令。如果你使用的是其他发行版，请参考你的包管理工具的用户指南或发行版的包列表：

+   Fedora、RHEL、CentOS 及兼容系统：

    ```
    $ yum install ansible

    ```

+   Ubuntu、Debian 及兼容系统：

    ```
    $ apt-get install ansible

    ```

### 注意

请注意，RHEL 和 CentOS 需要先安装 EPEL 仓库。有关 EPEL 的详细信息，包括如何安装，可以参考[`fedoraproject.org/wiki/EPEL`](https://fedoraproject.org/wiki/EPEL)。

如果你使用的是 Ubuntu，并希望使用最新版本而不是操作系统提供的版本，你可以使用 Ansible 提供的 Ubuntu PPA。有关如何设置的详细信息，可以参考[`launchpad.net/~ansible/+archive/ubuntu/ansible`](https://launchpad.net/~ansible/+archive/ubuntu/ansible)。

## 从 pip 安装

Pip，像发行版的包管理器一样，会处理你请求的包及其依赖项的查找、安装和更新。这使得通过 pip 安装 Ansible 和通过包管理器安装一样简单。然而，需要注意的是，它不会随着操作系统的更新而更新。此外，更新操作系统可能会破坏你的 Ansible 安装；不过，这种情况不太可能发生。如果你是 Python 用户，可能希望在隔离环境（虚拟环境）中安装 Ansible：但这不被支持，因为 Ansible 尝试将其模块安装到系统中。你应该使用 pip 在全系统范围内安装 Ansible。

以下是通过 pip 安装 Ansible 的命令：

```
$ pip install ansible

```

## 从源代码安装

从源代码安装是获取最新版本的好方法，但它可能没有经过像正式发布版本那样的测试。此外，你还需要自行管理更新到新版本，并确保 Ansible 在操作系统更新后依然能够正常工作。要克隆 `git` 仓库并安装，请运行以下命令。你可能需要系统的 root 权限才能执行此操作：

```
$ git clone git://github.com/ansible/ansible.git
$ cd ansible
$ sudo make install

```

### 提示

**下载示例代码**

你可以从 [`www.packtpub.com`](http://www.packtpub.com) 的帐户中下载你购买的所有 Packt 书籍的示例代码文件。如果你是从其他地方购买的本书，可以访问 [`www.packtpub.com/support`](http://www.packtpub.com/support) 并注册，文件将通过电子邮件直接发送给你。

# 设置 Ansible

Ansible 需要能够获取你想要配置的机器的清单，才能管理它们。由于有清单插件，这可以通过多种方式实现。基础安装包含了几个不同的清单插件。我们将在本书后面介绍这些插件。现在，我们将介绍简单的主机文件清单。

默认的 Ansible 清单文件名为 hosts，存放在 `/etc/ansible` 目录下。它的格式类似于 `INI` 文件。组名用方括号括起来，所有位于组名下的内容，直到下一个组名，都会分配给该组。机器可以同时属于多个组。组的作用是让你可以一次配置多个机器。在接下来的示例中，你可以使用组名代替主机名作为主机模式，Ansible 将对整个组同时运行模块。

在以下示例中，我们有三台机器在名为 `webservers` 的组中，分别是 `site01`、`site02` 和 `site01-dr`。我们还有一个名为 `production` 的组，包含了 `site01`、`site02`、`db01` 和 `bastion`。

```
[webservers]
site01
site02
site01-dr

[production]
site01
site02
db01
bastion

```

一旦你将主机添加到 Ansible 清单中，你就可以开始对其运行命令。Ansible 包含一个简单的模块叫做 `ping`，可以让你测试自己与主机之间的连接。让我们从命令行使用 Ansible 对我们的其中一台机器进行测试，确认我们可以配置它们。

Ansible 设计上追求简洁，而开发者通过使用 SSH 来连接被管理的机器实现了这一点。然后，它通过 SSH 连接发送代码并执行。这意味着你不需要在被管理的机器上安装 Ansible。这也意味着 Ansible 使用的是你已经用来管理机器的通道。这使得设置更简单，因为在大多数情况下，通常不需要额外的配置，也不需要在防火墙中打开端口。

首先，我们使用 Ansible `ping` 模块检查与服务器的连接。这个模块只是连接到以下服务器：

```
$ ansible site01 -u root -k -m ping

```

这应该会要求输入 SSH 密码，然后输出如下结果：

```
site01 | success >> {
 "changed": false,
 "ping": "pong"
}

```

如果你为远程系统设置了 SSH 密钥，你将能够省略 `-k` 参数，跳过提示并使用密钥。你还可以通过在清单中为每个主机单独配置，或者在全局 Ansible 配置中配置，来让 Ansible 始终使用特定的用户名。

要全局设置用户名，请编辑`/etc/ansible/ansible.cfg`并更改在`[defaults]`部分设置`remote_user`的行。您还可以更改`remote_port`以更改 Ansible 将 SSH 连接到的默认端口。这将更改所有机器的默认设置，但可以在清单文件中按服务器或组进行覆盖。

要在清单文件中设置用户名，只需在清单的相应行后附加`ansible_ssh_user`。例如，以下代码部分显示了一个清单，其中`site01`主机使用用户名`root`，`site02`主机使用用户名`daniel`。还有其他变量可以使用。`ansible_ssh_host`变量允许您设置不同的主机名，`ansible_ssh_port`变量允许您设置不同的端口，这在`site01-dr`主机上有所示。最后，`db01`主机使用用户名`fred`，并使用`ansible_ssh_private_key_file`设置私钥。

```
[webservers]      #1
site01 ansible_ssh_user=root     #2
site02 ansible_ssh_user=daniel      #3
site01-dr ansible_ssh_host=site01.dr ansible_ssh_port=65422      #4
[production]      #5
site01      #6
site02      #7
db01 ansible_ssh_user=fred ansible_ssh_private_key_file=/home/fred/.ssh.id_rsa     #8
bastion      #9

```

如果您不希望将 Ansible 直接访问受管机器上的 root 帐户，或者您的机器不允许 SSH 访问 root 帐户（例如 Ubuntu 的默认配置），您可以配置 Ansible 使用`sudo`来获取 root 访问权限。使用`sudo`的 Ansible 意味着您可以强制进行与否则相同的审核。配置 Ansible 使用`sudo`与配置端口一样简单，只是它要求在受管机器上配置`sudo`。

第一步是向`/etc/sudoers`文件添加一行；在受管节点上，如果选择使用自己的帐户，可能已经设置了这一点。您可以使用`sudo`与密码，也可以使用无密码的`sudo`。如果决定使用密码，您将需要对 Ansible 使用`-k`参数，或者在`/etc/ansible/ansible.cfg`中将`ask_sudo_pass`值设置为`true`。要使 Ansible 使用 sudo，请像这样在命令行中添加`--sudo`：

```
ansible site01 -s -m command -a 'id -a'

```

如果正常工作，它应返回类似于：

```
site01 | success | rc=0 >>
uid=0(root) gid=0(root) groups=0(root) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023

```

## 在 Windows 上设置它

Ansible 最近添加了管理 Windows 机器的功能。现在，您可以使用 Ansible 轻松管理 Windows 机器，就像管理 Linux 机器一样。

这与在 Linux 机器上使用 SSH 执行模块的方式相同使用 Windows PowerShell Remoting 工具远程执行。已添加了几个明确支持 Windows 的新模块，但也使一些现有模块能够与 Windows 受管机器一起使用。

要开始管理您的 Windows 机器，您需要执行一些复杂的设置。您需要遵循以下步骤：

1.  在清单中创建一些 Windows 机器

1.  安装 Python-winrm 以允许 Ansible 连接到 Windows 机器

1.  升级到 PowerShell 3.0+ 以支持 Windows 模块

1.  启用 Windows 远程管理以便 Ansible 连接

Windows 机器的创建方式与清单中其他所有机器相同。它们通过 `ansible_connection` 变量的值来区分。当 `ansible_connection` 设置为 `winrm` 时，它将尝试通过 winrm 连接到远程机器上的 Windows PowerShell。Ansible 还像在其他机器上一样使用 `ansible_ssh_user`、`ansible_ssh_pass` 和 `ansible_ssh_port` 的值。尽管这些名字里包含 ssh，但它们是用来提供连接 Windows PowerShell 远程服务所需的端口和凭证。以下是一个 Windows 机器示例：

```
[windows]
dc.ad.example.com
web01.ad.example.com
web02.ad.example.com

[windows:vars]
ansible_connection=winrm
ansible_ssh_user=daniel
ansible_ssh_pass=s3cr3t
ansible_ssh_port=5986

```

出于安全考虑，你可能不希望将密码存储在清单文件中。你可以像我们之前对 Unix 系统所展示的那样让 Ansible 提示输入密码，只需要去掉 `ansible_ssh_user` 和 `ansible_ssh_pass` 变量，而改为使用 `-k` 和 `-u` 参数传递给 Ansible。如果你愿意，也可以选择将密码存储在 Ansible 密库中，后续书中将详细介绍这个方法。

创建完清单文件后，你需要在控制器机器上安装 winrm Python 库。这个库将赋予 Ansible 连接 Windows 远程管理服务并配置远程 Windows 系统的能力。

目前，这个库还处于实验阶段，它与 Ansible 的连接并不完美，因此你需要安装与当前使用的 Ansible 版本匹配的特定版本。随着 Ansible 1.8 版本的发布，这个问题应该会有所解决。大多数发行版尚未打包该库，因此你可能需要通过 pip 安装。作为 root 用户，你需要运行以下命令：

```
$ pip install https://github.com/diyan/pywinrm/archive/df049454a9309280866e0156805ccda12d71c93a.zip

```

然而，对于较新的版本，你只需要运行以下命令：

```
pip install http://github.com/diyan/pywinrm/archive/master.zip

```

这将安装与 Ansible 1.7 兼容的特定版本 winrm。对于其他较新的 Ansible 版本，你可能需要不同的版本，最终 winrm Python 库应该会被不同的发行版打包。现在，你的机器应该能够使用 Ansible 连接和管理 Windows 机器。

接下来，你需要在将要管理的机器上进行一些设置。第一步是确保你已安装 PowerShell 3.0 或更高版本。你可以使用以下命令检查已安装的版本：

```
$PSVersionTable.PSVersion.Major

```

如果你得到的返回值不是 3 或者大于 3，那么你需要升级你的 PowerShell 版本。你可以选择手动下载并安装适合你系统的最新 Windows 管理框架，或者使用 Ansible 项目提供的脚本。为了节省空间，我们将在这里解释脚本化安装；手动安装留给读者自行操作。

```
Invoke-WebRequest https://raw.githubusercontent.com/ansible/ansible/release1.7.0/examples/scripts/upgrade_to_ps3.ps1 -OutFile upgrade_to_ps3.ps1
.\upgrade_to_ps3.ps1

```

第一个命令从 Ansible 项目的 GitHub 存储库下载升级脚本并保存到磁盘。第二个命令将检测您的操作系统，以下载适合的 Windows 管理框架版本并安装它。

接下来，您需要配置 Windows 远程管理服务。Ansible 项目提供了一个脚本，该脚本会自动按 Ansible 期望的方式配置 Windows 远程管理。虽然您可以手动设置，但强烈建议您使用此脚本，以避免配置错误。要下载并运行此脚本，请打开 PowerShell 终端并运行以下命令：

```
Invoke-WebRequest https://raw.githubusercontent.com/ansible/ansible/release1.7.0/examples/scripts/ConfigureRemotingForAnsible.ps1 -OutFile ConfigureRemotingForAnsible.ps1
.\ConfigureRemotingForAnsible.ps1

```

第一个命令从 Ansible 项目的 GitHub 上下载配置脚本，第二个命令运行该脚本。如果一切正常，第二个脚本应该返回`Ok`的输出。

您现在应该能够连接到您的机器并使用 Ansible 进行配置。像之前一样，我们运行一个 ping 命令来确认 Ansible 是否能够远程执行其模块。虽然 Unix 机器可以使用`ping`模块，但 Windows 机器使用`win_ping`模块。用法几乎完全相同；不过，由于我们已经将密码添加到库存文件中，因此无需使用`-k`选项。

```
$ ansible web01.ad.example.com -u daniel -m win_ping

```

如果一切正常，您应该看到以下输出：

```
web01.ad.example.com | success >> {
 "changed": false,
 "ping": "pong"
}

```

输出表示 Ansible 成功连接到 Windows 远程管理服务，成功登录并在远程主机上执行了一个模块。如果一切正常，那么您应该能够使用所有其他 Windows 模块来管理您的机器。

# 使用 Ansible 的第一步

Ansible 模块以类似`key=value`的键值对形式接收参数，执行远程服务器上的任务，并以`JSON`格式返回任务信息。键值对允许模块在请求时知道应该做什么。它们可以是硬编码的值，或者在播放书中，它们可以使用变量，关于这一点将在第二章，*简单播放书*中介绍。模块返回的数据让 Ansible 知道是否在受管理的主机中发生了任何更改，或者是否之后应该更改 Ansible 保存的任何信息。

模块通常在播放书中运行，因为这样可以将多个模块串联起来，但也可以在命令行中使用。之前，我们使用了`ping`命令来检查 Ansible 是否已正确设置并能够访问配置的节点。`ping`模块仅检查 Ansible 核心是否能够在远程机器上运行，但实际上什么都不做。

一个稍微更有用的模块名为`setup`。该模块连接到配置的节点，收集有关系统的数据，然后返回这些值。虽然在命令行运行时这并不特别方便，但在播放书中，您可以稍后在其他模块中使用收集到的值。

要从命令行运行 Ansible，你需要传递两个参数，通常是三个。第一个是一个主机模式，用于匹配你想要应用模块的机器。第二个是你需要提供你希望运行的模块名称，和可选的任何参数。对于主机模式，你可以使用一个组名、一个机器名、一个 glob 模式，或一个波浪符（~），后面跟一个正则表达式匹配主机名。或者，为了表示所有这些，你可以使用`all`这个词或简单地使用`*`。以这种方式在命令行运行 Ansible 模块被称为临时 Ansible 命令。

要在你的节点上运行 `setup` 模块，你需要以下命令行：

```
$ ansible machinename -u root -k -m setup

```

然后，`setup`模块将连接到机器并返回许多有用的信息。`setup`模块提供的所有事实信息都以`ansible_`为前缀，以便与变量区分开来。

该模块可以在 Windows 和 Unix 机器上工作。目前，Unix 机器会提供比 Windows 机器更多的信息。然而，随着 Ansible 新版本的发布，你可以期待看到更多 Windows 功能被加入到 Ansible 中。

```
machinename | success >> {
 "ansible_facts": {
 "ansible_distribution": "Microsoft Windows NT 6.3.9600.0",
 "ansible_distribution_version": "6.3.9600.0",
 "ansible_fqdn": "ansibletest",
 "ansible_hostname": "ANSIBLETEST",
 "ansible_ip_addresses": [
 "100.72.124.51",
 "fe80::1fd:fc3b:1eff:350d"
 ],
 "ansible_os_family": "Windows",
 "ansible_system": "Win32NT",
 "ansible_totalmem": "System.Object[]"
 },
 "changed": false
}

```

以下是你将使用的最常见值的表格；并非所有这些值都在所有机器上可用。尤其是 Windows 机器从 `setup` 模块返回的数据要少得多。

| 字段 | 示例 | 描述 |
| --- | --- | --- |
| `ansible_architecture` | x86_64 | 这是受管机器的架构 |
| `ansible_distribution` | CentOS | 这是受管机器上的 Linux 或 Unix 发行版 |
| `ansible_distribution_version` | 6.3 | 这是前述发行版的版本 |
| `ansible_domain` | example.com | 这是服务器主机名中的域名部分 |
| `ansible_fqdn` | machinename.example.com | 这是受管机器的完全合格域名 |
| `ansible_interfaces` | ["lo", "eth0"] | 这是该机器所有接口的列表，包括回环接口 |
| `ansible_kernel` | 2.6.32-279.el6.x86_64 | 这是受管机器上安装的内核版本 |
| `ansible_memtotal_mb` | 996 | 这是受管机器上可用的总内存，以兆字节为单位 |
| `ansible_processor_count` | 1 | 这是受管机器上可用的 CPU 总数 |
| `ansible_virtualization_role` | guest | 这决定了机器是客机还是主机 |
| `ansible_virtualization_type` | kvm | 这是受管机器上虚拟化的类型 |

在 Unix 机器上，这些变量通过 Python 从受管机器收集；如果远程节点上安装了 facter 或 ohai，`setup` 模块将执行它们并返回它们的数据。与其他事实一样，ohai 的事实前缀是 `ohai_`，facter 的事实前缀是 `facter_`。尽管在命令行中 `setup` 模块看起来没什么用处，但一旦开始编写 playbook，它会变得非常有用。请注意，facter 和 ohai 在 Windows 主机上不可用。

如果 Ansible 中的所有模块像 `setup` 和 `ping` 模块那样仅做很少的事情，我们将无法更改远程机器上的任何内容。Ansible 提供的几乎所有其他模块，如 `file` 模块，都允许我们实际配置远程机器。

`file` 模块可以通过单个路径参数调用；这将使其返回有关该文件的信息。如果你提供更多参数，它将尝试更改文件的属性，并告诉你是否已进行更改。Ansible 模块会告诉你是否进行了更改，这在你编写 playbooks 时变得更加重要。

你可以调用 `file` 模块，如下所示，查看 `/etc/fstab` 的详细信息：

```
$ ansible machinename -u root -k -m file -a 'path=/etc/fstab'

```

前述命令应该产生类似以下内容的响应：

```
machinename | success >> {
 "changed": false,
 "group": "root",
 "mode": "0644",
 "owner": "root",
 "path": "/etc/fstab",
 "size": 779,
 "state":
 "file"
}

```

另外，响应可能是类似以下命令的内容，用于在 `/tmp` 创建一个新的测试目录：

```
$ ansible machinename -u root -k -m file -a 'path=/tmp/teststate=directory mode=0700 owner=root'

```

前述命令应该返回类似以下内容：

```
machinename | success >> {
 "changed": true,
 "group": "root",
 "mode": "0700",
 "owner": "root",
 "path": "/tmp/test",
 "size": 4096,
 "state": "directory"
}

```

我们可以看到响应中 `changed` 变量被设置为 `true`，因为目录不存在或具有不同的属性，需要进行更改以使其与提供的参数所指定的状态匹配。如果第二次使用相同的参数运行该命令，`changed` 的值将设置为 `false`，意味着该模块没有对系统做出任何更改。

有几个模块接受与 `file` 模块类似的参数，其中一个例子是 `copy` 模块。`copy` 模块会将控制机上的文件复制到受管机器，并按要求设置其属性。例如，要将 `/etc/fstab` 文件复制到受管机器的 `/tmp`，你可以使用以下命令：

```
$ ansible machinename -m copy -a 'src=/etc/fstab dest=/tmp/fstab'

```

前述命令在第一次运行时，应该返回类似以下内容：

```
machinename | success >> {
 "changed": true,
 "dest": "/tmp/fstab",
 "group": "root",
 "md5sum": "fe9304aa7b683f58609ec7d3ee9eea2f",
 "mode": "0700",
 "owner": "root",
 "size": 637,
 "src": "/root/.ansible/tmp/ansible-1374060150.96- 77605185106940/source",
 "state": "file"
}

```

还有一个名为 `command` 的模块，可以在受管机器上运行任何任意命令。这使得你可以使用任何任意命令进行配置，比如一个 `preprovided` 安装程序或自编写的脚本；它对于重启机器也非常有用。请注意，这个模块不会在 shell 中运行命令，因此你无法执行重定向、使用管道、展开 shell 变量或在后台运行命令。

Ansible 模块致力于在不需要更改时避免进行修改。这被称为幂等性，它可以使得在多个服务器上运行命令时变得更快。不幸的是，Ansible 无法知道你的命令是否做出了更改，因此为了帮助它更具幂等性，你需要提供一些帮助。它可以通过`creates`或`removes`参数来实现这一点。如果你给出`creates`参数，则如果文件名参数存在，命令将不会运行。`removes`参数则相反；如果文件名存在，命令将会执行。

你可以按以下方式运行该命令：

```
$ ansible machinename -m command -a 'rm -rf /tmp/testing removes=/tmp/testing'

```

如果没有名为`/tmp/testing`的文件或目录，命令输出将指示该文件已被跳过，如下所示：

```
machinename | skipped

```

否则，如果文件确实存在，它将像以下代码一样显示：

```
ansibletest | success | rc=0 >>

```

通常情况下，使用其他模块代替`command`模块会更好。其他模块提供更多选项，并且能够更好地捕获它们所工作的问题领域。例如，在这个例子中，使用`file`模块对于 Ansible 和编写配置的人员来说都会减少工作量，因为`file`模块如果状态设置为`absent`，会递归地删除某些东西。所以，前面的命令相当于以下命令：

```
$ ansible machinename -m file -a 'path=/tmp/testing state=absent'

```

如果你需要在运行命令时使用通常在 shell 中可用的功能，你需要使用`shell`模块。这样你就可以使用重定向、管道或后台作业。你可以使用`executable`参数来选择要使用的 shell。你可以按以下方式使用`shell`模块：

```
$ ansible machinename -m shell -a '/opt/fancyapp/bin/installer.sh >/var/log/fancyappinstall.log creates=/var/log/fancyappinstall.log'

```

# 模块帮助

不幸的是，我们没有足够的空间来涵盖 Ansible 中所有可用的模块；不过幸运的是，Ansible 包含了一个名为`ansible-doc`的命令，可以用来检索帮助信息。Ansible 中包含的所有模块都有这个数据，但从其他地方收集的模块可能帮助信息较少。`ansible-doc`命令还允许你查看所有可用模块的列表。

要获取所有可用模块的列表，并附带每种类型的简短描述，可以使用以下命令：

```
$ ansible-doc -l

```

要查看某个特定模块的帮助文件，你需要将模块作为`ansible-doc`命令的唯一参数。例如，要查看`file`模块的帮助信息，可以使用以下命令：

```
$ ansible-doc file

```

# 总结

在本章中，我们讨论了选择哪种安装类型来安装 Ansible，以及如何构建一个清单文件来反映你的环境。接着，我们展示了如何以临时方式使用 Ansible 模块来执行简单任务。最后，我们讨论了如何了解系统中可用的模块，并如何使用命令行获取使用模块的指令。

在接下来的章节中，你将学习如何在 Playbook 中一起使用多个模块。这使得你能够执行比单独使用模块时更加复杂的任务。
