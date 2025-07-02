# 第二章：自动化简单任务

正如我们在上一章中提到的，Ansible 可以用于创建和管理整个基础设施，也可以集成到已经存在的基础设施中。

在这一章中，我们将看到：

+   什么是 playbook，以及它是如何工作的

+   如何使用 Ansible 创建一个 Web 服务器

+   深入了解 Jinja2 模板引擎

但首先我们将讨论 **YAML Ain't Markup Language**（**YAML**），这是一种人类可读的数据序列化语言，在 Ansible 中被广泛使用。

# YAML

YAML 像许多其他数据序列化语言（如 JSON）一样，只有非常少数、基本的概念：

+   声明

+   列表

+   关联数组

声明与任何其他编程语言中的变量非常相似，也就是说：

```
name: 'This is the name' 

```

要创建一个列表，我们需要使用 '`-`'：

```
- 'item1' 
- 'item2' 
- 'item3' 

```

YAML 使用缩进来逻辑性地划分父子关系。所以，如果我们想要创建关联数组（也称为对象），只需添加缩进：

```
item: 
  name: TheName 
  location: TheLocation 

```

显然，我们可以将这些结合起来，也就是说：

```
people: 
  - name: Albert 
    number: +1000000000 
    country: USA 
  - name: David 
    number: +44000000000 
    country: UK 

```

这些就是 YAML 的基础。YAML 可以做更多的事情，但目前这些足够了。

# 你好，Ansible

正如我们在上一章所见，Ansible 可以用于自动化你可能每天都会做的简单任务。

让我们首先检查远程机器是否可达；换句话说，让我们开始 ping 一台机器。最简单的做法是运行以下命令：

```
$ ansible all -i HOST, -m ping

```

在这里，`HOST` 是一个 IP 地址、**完全限定域名**（**FQDN**）或你有 SSH 访问权限的机器别名（如我们在上一章所看到的，你可以使用 **基于内核的虚拟机**（**KVM**））。

### 提示

在 "`HOST,`" 后面的逗号是必需的，否则它将不会被视为一个列表，而是一个字符串。

在这种情况下，我们对系统中的虚拟机进行了操作：

```
$ ansible all -i test01.fale.io, -m ping

```

你应该收到如下结果：

```
test01.fale.io | SUCCESS => {
 "changed": false,
 "ping": "pong"
}

```

现在，让我们看看我们做了什么以及为什么这样做。我们从 Ansible 的帮助开始。查询帮助文档可以使用以下命令：

```
$ ansible --help

```

为了便于阅读，我们删除了所有与未使用的选项相关的输出：

```
Usage: ansible <host-pattern> [options]
Options:
 -i INVENTORY, --inventory-file=INVENTORY
 specify inventory host path
 (default=/etc/ansible/hosts) or comma
 separated host list.
 -m MODULE_NAME, --module-name=MODULE_NAME
 module name to execute (default=command)

```

所以，我们做的就是：

1.  我们调用了 Ansible。

1.  我们指示 Ansible 在所有主机上运行。

1.  我们指定了我们的清单（也就是主机列表）。

1.  我们指定了我们想要运行的模块（`ping`）。

现在我们可以 ping 服务器了，接下来让我们`echo hello ansible!`

```
$ ansible all -i test01.fale.io, -m shell -a '/bin/echo hello ansible!'

```

你应该收到如下结果：

```
test01.fale.io | SUCCESS | rc=0 >>
hello ansible!

```

在这个示例中，我们使用了一个额外的选项。让我们查看帮助文档，看看它的作用：

```
Usage: ansible <host-pattern> [options]
Options:
 -a MODULE_ARGS, --args=MODULE_ARGS
 module arguments

```

正如你从上下文和名称中可能已经猜到的那样，`args` 选项允许你将额外的参数传递给模块。一些模块（比如 `ping`）不支持任何参数，而其他模块（如 `shell`）则需要参数。

# 使用 playbooks

Playbook 是 Ansible 的核心功能之一，用于告诉 Ansible 执行什么操作。它们像是 Ansible 的待办事项列表，包含了一系列任务；每个任务内部都链接到一段被称为 **模块** 的代码。Playbook 是简单、易读的 YAML 文件，而模块则是一段可以用任何语言编写的代码，前提是其输出必须是 JSON 格式。你可以在 Playbook 中列出多个任务，这些任务会被 Ansible 按顺序执行。你可以把 Playbook 看作是 Puppet 中的清单、Salt 中的状态或 Chef 中的食谱；它们让你能够列出希望在远程系统上执行的任务或命令。

## 学习 Playbook 的结构

Playbook 可以包含远程主机列表、用户变量、任务、处理程序等。你还可以通过 Playbook 重写大多数配置设置。让我们开始查看 Playbook 的结构。

我们现在要讨论的 Playbook 目的是确保 `httpd` 包已安装，并且服务已 **启用** 且 **启动**。以下是 `setup_apache.yaml` 文件的内容：

```
--- 
- hosts: all 
  remote_user: fale 
  tasks: 
  - name: Ensure the HTTPd package is installed 
    yum: 
      name: httpd 
      state: present 
      become: True 
  - name: Ensure the HTTPd service is enabled and running 
    service: 
      name: httpd 
      state: started 
      enabled: True 
    become: True 

```

`setup_apache.yaml` 文件是一个 Playbook 的示例。该文件由三个主要部分组成，如下所示：

+   `hosts`：这列出了我们希望在哪些主机上执行任务的主机或主机组。`hosts` 字段是必填项，每个 Playbook 都应该有它。它告诉 Ansible 在哪些主机上运行列出的任务。当提供了主机组时，Ansible 会从 Playbook 中获取主机组并尝试在库存文件中查找它。如果没有找到匹配项，Ansible 会跳过该主机组的所有任务。使用 `--list-hosts` 选项和 Playbook（`ansible-playbook <playbook> --list-hosts`）将告诉你 Playbook 会在哪些主机上执行。

+   `remote_user`：这是 Ansible 的配置参数之一（例如，`tom -remote_user`），它告诉 Ansible 在登录系统时使用特定的用户（在此例中为 `tom`）。

+   `tasks`：最后，我们来讲任务。所有的 Playbook 都应该包含任务。任务是你想要执行的一系列操作。一个任务字段包含任务的名称（即任务的帮助文本）、应该执行的模块以及模块所需的参数。让我们来看一下在 Playbook 中列出的单个任务，如前面代码片段所示：

### 注意

本书中的所有示例将在 CentOS 上执行，但同一组示例经过一些更改后，也能在其他发行版上运行。

在上述情况下，有两个任务。`name` 参数表示任务正在执行的操作，`present` 主要是为了提高可读性，正如我们在 playbook 执行过程中会看到的。`name` 参数是可选的。`modules`，如 `yum` 和 `service`，有各自的一组参数。几乎所有模块都有 `name` 参数（例如 `debug` 模块是例外），它指示执行操作的组件。让我们来看一下其他参数：

+   在 `yum` 模块的情况下，`state` 参数的最新值表示应该安装最新的 `httpd` 包。这个命令大致等同于 `yum install httpd`。

+   在 `service` 模块的场景中，`state` 参数的 `started` 值表示 `httpd` 服务应该启动，这大致等同于 `/etc/init.d/httpd start`。在这个模块中，我们还可以使用 "`enabled`" 参数来定义服务是否在启动时自动启动。

+   `become: True` 参数表示任务应该以 `sudo` 权限执行。如果 `sudo` 用户的文件不允许该用户运行特定的命令，那么当 playbook 执行时，它将失败。

### 注意

你可能会有疑问，为什么没有一个包模块可以自动根据系统架构来选择并运行 `yum`、`apt` 或其他包管理选项。Ansible 将包管理器的值填充到一个名为 `ansible_pkg_manager` 的变量中。

一般来说，我们需要记住，跨不同操作系统的具有通用名称的包的数量仅是实际存在的包的一个小子集。例如，`httpd` 包在 Red Hat 系统中被称为 `httpd`，在基于 Debian 的系统中被称为 `apache2`。我们还需要记住，每个包管理器都有一组独特的选项，使其功能强大；因此，使用明确的包管理器名称会更合理，这样就能确保完整的选项集对编写 playbook 的最终用户可用。

## 执行 playbook

现在，是时候（是的，终于！）运行 playbook 了。为了让 Ansible 执行 playbook 而不是模块，我们需要使用一个不同的命令（`ansible-playbooks`），该命令的语法与我们之前看到的 "`ansible`" 命令非常相似：

```
$ ansible-playbook -i HOST, setup_apache.yaml

```

正如你所看到的，除了 playbook 中指定的主机模式（已消失）和被 playbook 名称替代的模块选项外，其他没有变化。所以，要在我的机器上执行这个命令，确切的命令是：

```
$ ansible-playbook -i test01.fale.io, setup_apache.yaml

```

结果如下：

```
PLAY [all] *******************************************************
TASK [setup] *****************************************************
ok: [test01.fale.io] 
TASK [Ensure the HTTPd package is installed] *********************
changed: [test01.fale.io] 
TASK [Ensure the HTTPd service is enabled and running] ***********
changed: [test01.fale.io] 
PLAY RECAP *******************************************************
test01.fale.io    : ok=3    changed=2    unreachable=0    failed=0 

```

哇！示例成功了。现在让我们检查一下 `httpd` 包是否已安装并且正在运行。检查 HTTPd 是否安装的最简单方法是使用 `rpm`：

```
$ rpm -qa | grep httpd

```

如果一切正常，你应该看到类似以下的输出：

```
httpd-tools-2.4.6-40.el7.centos.x86_64
httpd-2.4.6-40.el7.centos.x86_64

```

要查看服务的状态，我们可以询问 `systemd`：

```
$ systemctl status httpd

```

预期的结果类似于以下内容：

```
httpd.service - The Apache HTTP Server
 Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
 Active: active (running) since Sat 2016-05-07 13:22:14 EDT; 7min ago
 Docs: man:httpd(8)
 man:apachectl(8)
 Main PID: 2214 (httpd)
 Status: "Total requests: 0; Current requests/sec: 0; Current traffic:   0 B/sec"
 CGroup: /system.slice/httpd.service
 -2214 /usr/sbin/httpd -DFOREGROUND
 -2215 /usr/sbin/httpd -DFOREGROUND
 -2216 /usr/sbin/httpd -DFOREGROUND
 -2217 /usr/sbin/httpd -DFOREGROUND
 -2218 /usr/sbin/httpd -DFOREGROUND
 -2219 /usr/sbin/httpd -DFOREGROUND

```

根据剧本，最终状态已经实现。我们简要回顾一下剧本运行过程中到底发生了什么：

```
PLAY [all] *******************************************************

```

这一行提醒我们，一个剧本将在此处启动，并将在“`all`”主机上执行：

```
TASK [setup] *****************************************************
ok: [test01.fale.io]

```

`TASK`行显示任务的名称（在本例中是`setup`），以及它对每个主机的影响。有时人们会对`setup`任务感到困惑。事实上，如果你查看剧本，会发现并没有`setup`任务。这是因为在执行我们要求的任务之前，Ansible 会尝试连接到机器并收集关于它的信息，这些信息可能在后续任务中有用。正如你所见，这个任务的结果是绿色的`ok`状态，说明它成功执行且服务器没有发生任何变化：

```
TASK [Ensure the HTTPd package is installed] *********************
changed: [test01.fale.io]
TASK [Ensure the HTTPd service is enabled and running] ***********
changed: [test01.fale.io]

```

这两个任务的状态是黄色，并显示“`changed`”。这意味着这些任务已经执行成功，但实际上改变了机器上的某些内容：

```
PLAY RECAP *******************************************************
test01.fale.io       : ok=3    changed=2    unreachable=0    failed=0

```

最后几行是对剧本执行过程的总结。现在让我们重新运行任务，看看两个任务实际运行后的输出：

```
PLAY [all] *******************************************************
TASK [setup] *****************************************************
ok: [test01.fale.io]
TASK [Ensure the HTTPd package is installed] *********************
ok: [test01.fale.io]
TASK [Ensure the HTTPd service is enabled and running] ***********
ok: [test01.fale.io]
PLAY RECAP *******************************************************
test01.fale.io    : ok=3    changed=0    unreachable=0    failed=0

```

正如你预期的那样，这两个任务的输出是`ok`，这意味着在运行任务之前，期望的状态已经达成。需要记住的是，许多任务（如**Gathering facts**任务）获取有关系统某个组件的信息，并不一定会改变系统上的任何内容；因此，这些任务之前没有显示“已更改”的输出。

第一次和第二次运行中的`PLAY RECAP`部分如下所示。你将在第一次运行时看到以下输出：

```
PLAY RECAP ******************************************************
test01.fale.io    : ok=3    changed=2    unreachable=0    failed=0 

```

你将在第二次运行时看到以下输出：

```
PLAY RECAP *******************************************************
test01.fale.io    : ok=3    changed=0    unreachable=0    failed=0

```

如你所见，区别在于第一个任务的输出显示`changed=2`，这意味着系统状态由于两个任务而发生了两次变化。查看此输出非常有用，因为如果系统已经达到了期望的状态，然后你再次运行剧本，预期的输出应该是`changed=0`。

如果此时你想到**幂等性**这个词，那你完全正确，值得给自己一个赞！幂等性是配置管理的一个关键原则。维基百科定义幂等性为：如果对任何值应用两次操作，结果与应用一次操作时相同。你在童年时期可能会遇到的最早的幂等性例子就是对数字`1`的乘法操作，其中`1*1=1`每次都成立。

大多数配置管理工具已经采用了这一原则，并将其应用于基础设施。在大型基础设施中，强烈建议监控或跟踪基础设施中已更改任务的数量，并在发现异常时警告相关任务；这一点适用于任何配置管理工具。理想的情况是，只有在你引入新的更改时，才应该看到更改，这些更改形式可能是任何 **创建**、**删除**、**更新** 或 **删除**（**CRUD**）操作。如果你在想如何用 Ansible 来实现这一点，继续阅读这本书，你最终会找到答案！

让我们继续。你也可以按如下方式编写之前的任务，但从终端用户的角度来看，任务的可读性非常高（我们将把这个文件称为 `setup_apache_no_com.yaml`）：

```
--- 
- hosts: all 
  remote_user: fale 
  tasks: 
  - yum: 
      name: httpd 
      state: present 
    become: True 
  - service: 
      name: httpd 
      state: started 
      enabled: True 
    become: True 

```

让我们重新运行 playbook 以查看输出是否有任何变化：

```
$ ansible-playbook -i test01.fale.io, setup_apache_no_com.yaml

```

输出将是：

```
PLAY [all] *******************************************************
TASK [setup] *****************************************************
ok: [test01.fale.io] 
TASK [yum] *******************************************************
ok: [test01.fale.io] 
TASK [service] ***************************************************
ok: [test01.fale.io] 
PLAY RECAP *******************************************************
test01.fale.io    : ok=3    changed=0    unreachable=0    failed=0 

```

如你所见，区别在于可读性。尽可能地，建议将任务保持简单（**KISS** 原则：**Keep It Simple Stupid**），以便于长期维护脚本。

现在我们已经了解了如何编写基本的 playbook 并在主机上运行，接下来让我们看看其他可以帮助你在运行 playbooks 时的选项。

# Ansible 可见性

第一个常用的选项就是调试选项。为了理解在运行 playbook 时发生了什么，你可以使用 **verbose** (`-v`) 选项运行它。每增加一个 `v`，都会为终端用户提供更多的调试输出。

让我们看一个使用 playbook 调试单个任务的示例，使用以下调试选项：

+   `-v` 选项提供默认输出，如前面的示例所示。

+   `-vv` 选项提供稍微多一点的信息，如以下示例所示：

```
 Using /etc/ansible/ansible.cfg as config file 

    PLAYBOOK: setup_apache.yaml ******************************* 
    1 plays in setup_apache.yaml 

    PLAY [all] ************************************************ 

    TASK [setup] ********************************************** 
    ok: [test01.fale.io] 

    TASK [Ensure the HTTPd package is installed] ************** 
    task path: /home/fale/setup_apache.yaml:5 
    ok: [test01.fale.io] => {"changed": false, "msg": "", "rc": 0, "results": ["httpd-2.4.6-40.el7.centos.x86_64 providing httpd is already installed"]} 

    TASK [Ensure the HTTPd service is enabled and running] **** 
    task path: /home/fale/setup_apache.yaml:10 
    ok: [test01.fale.io] => {"changed": false, "enabled": true, "name": "httpd", "state": "started"} 

    PLAY RECAP ************************************************ 
    test01.fale.io   : ok=3  changed=0   unreachable=0  failed=0 

```

+   `-vvv` 选项会提供更多信息，如以下代码所示。这展示了 Ansible 用于在远程主机上创建临时文件并远程运行脚本的 `ssh` 命令：

```
TASK [Ensure the HTTPd package is installed] **************
task path: /home/fale/setup_apache.yaml:5
<test01.fale.io> ESTABLISH SSH CONNECTION FOR USER: fale
<test01.fale.io> SSH: EXEC ssh -C -q -o ControlMaster=auto -o ControlPersist=60s -o KbdInteractiveAuthentication=no -o PreferredAuthentications=gssapi-with-mic,gssapi-keyex,hostbased,publickey -o PasswordAuthentication=no -o User=fale -o ConnectTimeout=10 -o ControlPath=/home/fale/.ansible/cp/ansible-ssh-%C test01.fale.io '/bin/sh -c '"'"'( umask 22 && mkdir -p "` echo $HOME/.ansible/tmp/ansible-tmp-1462644055.19-51001413558638 `" && echo "` echo $HOME/.ansible/tmp/ansible-tmp-1462644055.19-51001413558638 `" )'"'"''
<test01.fale.io> PUT /tmp/tmp9JSYiP TO /home/fale/.ansible/tmp/ansible-tmp-1462644055.19-51001413558638/yum
<test01.fale.io> SSH: EXEC sftp -b - -C -o ControlMaster=auto -o ControlPersist=60s -o KbdInteractiveAuthentication=no -o PreferredAuthentications=gssapi-with-mic,gssapi-keyex,hostbased,publickey -o PasswordAuthentication=no -o User=fale -o ConnectTimeout=10 -o ControlPath=/home/fale/.ansible/cp/ansible-ssh-%C '[test01.fale.io]'
<test01.fale.io> ESTABLISH SSH CONNECTION FOR USER: fale
<test01.fale.io> SSH: EXEC ssh -C -q -o ControlMaster=auto -o ControlPersist=60s -o KbdInteractiveAuthentication=no -o PreferredAuthentications=gssapi-with-mic,gssapi-keyex,hostbased,publickey -o PasswordAuthentication=no -o User=fale -o ConnectTimeout=10 -o ControlPath=/home/fale/.ansible/cp/ansible-ssh-%C -tt test01.fale.io '/bin/sh -c '"'"'sudo -H -S -n -u root /bin/sh -c '"'"'"'"'"'"'"'"'echo BECOME-SUCCESS-axnwopicemeccmdhnlmhawtwlysgfgjc; LANG=en_US.utf8 LC_ALL=en_US.utf8 LC_MESSAGES=en_US.utf8 /usr/bin/python -tt /home/fale/.ansible/tmp/ansible-tmp-1462644055.19-51001413558638/yum; rm -rf "/home/fale/.ansible/tmp/ansible-tmp-1462644055.19-51001413558638/" > /dev/null 2>&1'"'"'"'"'"'"'"'"''"'"''
ok: [test01.fale.io] => {"changed": false, "invocation": {"module_args": {"conf_file": null, "disable_gpg_check": false, "disablerepo": null, "enablerepo": null, "exclude": null, "install_repoquery": true, "list": null, "name": ["httpd"], "state": "present", "update_cache": false}, "module_name": "yum"}, "msg": "", "rc": 0, "results": ["httpd-2.4.6-40.el7.centos.x86_64 providing httpd is already installed"]}

```

# Playbook 中的变量

有时在 playbook 中设置和获取变量是很重要的。

很多时候，你需要自动化多个相似的操作。在这种情况下，你会希望创建一个可以通过不同变量调用的单一 playbook，以确保代码的可重用性。

变量非常重要的另一个情况是当你有多个数据中心，并且某些值将是数据中心特定的。一个常见的例子是 DNS 服务器。让我们分析以下简单代码，它将向我们介绍如何在 Ansible 中设置和获取变量：

```
--- 
- hosts: all 
  remote_user: fale 
  tasks: 
  - name: Set variable 'name' 
    set_fact: 
      name: Test machine 
  - name: Print variable 'name' 
    debug: 
      msg: '{{ name }}' 

```

让我们按常规方式运行它：

```
$ ansible-playbook -i test01.fale.io, variables.yaml

```

你应该看到如下结果：

```
PLAY [all] **********************************************************
TASK [setup] ********************************************************
ok: [test01.fale.io] 
TASK [Set variable 'name'] ******************************************
ok: [test01.fale.io] 
TASK [Print variable 'name'] ****************************************
ok: [test01.fale.io] => {
 "msg": "Test machine"
} 
PLAY RECAP **********************************************************
test01.fale.io       : ok=3    changed=0    unreachable=0    failed=0

```

如果我们分析刚刚执行的代码，应该很清楚发生了什么。我们设置了一个变量（在 Ansible 中称为 `facts`），然后通过 `debug` 函数将其打印出来。

### 提示

当你使用这种扩展版本的 YAML 时，变量应该始终用引号括起来。

Ansible 允许你以多种方式设置变量，也就是通过传递变量文件、在 playbook 中声明变量、通过 `-e / --extra-vars` 传递给 `ansible-playbook` 命令，或者在清单文件中声明变量（我们将在下一章中更深入地讨论这个话题）。

现在是时候开始使用 Ansible 在设置阶段获取的一些元数据了。我们先来看看 Ansible 收集的数据。为此，我们将执行：

```
$ ansible all -i HOST, -m setup

```

在我们的具体情况下，这意味着执行以下操作：

```
$ ansible all -i test01.fale.io, -m setup

```

我们显然也可以使用 playbook 来实现这一点，但这种方式更快。而且，对于“设置”类的情况，你只需要在开发过程中查看输出，以确保使用正确的变量名来实现你的目标。

输出将类似于以下内容：

```
test01.fale.io | SUCCESS => {
 "ansible_facts": {
 "ansible_all_ipv4_addresses": [
 "178.62.36.208",
 "10.16.0.7"
 ],
 "ansible_all_ipv6_addresses": [
 "fe80::601:e2ff:fef1:1301"
 ],
 "ansible_architecture": "x86_64",
 "ansible_bios_date": "04/25/2016",
 "ansible_bios_version": "20160425",
 "ansible_cmdline": {
 "ro": true,
 "root": "LABEL=DOROOT"
 },
 "ansible_date_time": {
 "date": "2016-05-14",
 "day": "14",
 "epoch": "1463244633",
 "hour": "12",
 "iso8601": "2016-05-14T16:50:33Z",
 "iso8601_basic": "20160514T125033231663",
 "iso8601_basic_short": "20160514T125033",
 "iso8601_micro": "2016-05-14T16:50:33.231770Z",
 "minute": "50",
 "month": "05",
 "second": "33",
 "time": "12:50:33",
 "tz": "EDT",
 "tz_offset": "-0400",
 "weekday": "Saturday",
 "weekday_number": "6",
 "weeknumber": "19",
 "year": "2016"
 },
 "ansible_default_ipv4": {
 "address": "178.62.36.208",
 "alias": "eth0",
 "broadcast": "178.62.63.255",
 "gateway": "178.62.0.1",
 "interface": "eth0",
 "macaddress": "04:01:e2:f1:13:01",
 "mtu": 1500,
 "netmask": "255.255.192.0",
 "network": "178.62.0.0",
 "type": "ether"
 },
 "ansible_default_ipv6": {},
 "ansible_devices": {
 "vda": {
 "holders": [],
 "host": "",
 "model": null,
 "partitions": {
 "vda1": {
 "sectors": "41943040",
 "sectorsize": 512,
 "size": "20.00 GB",
 "start": "2048"
 }
 },
 "removable": "0",
 "rotational": "1",
 "scheduler_mode": "",
 "sectors": "41947136",
 "sectorsize": "512",
 "size": "20.00 GB",
 "support_discard": "0",
 "vendor": "0x1af4"
 }
 },
 "ansible_distribution": "CentOS",
 "ansible_distribution_major_version": "7",
 "ansible_distribution_release": "Core",
 "ansible_distribution_version": "7.2.1511",
 "ansible_dns": {
 "nameservers": [
 "8.8.8.8",
 "8.8.4.4"
 ]
 },
 "ansible_domain": "",
 "ansible_env": {
 "HOME": "/home/fale",
 "LANG": "en_US.utf8",
 "LC_ALL": "en_US.utf8",
 "LC_MESSAGES": "en_US.utf8",
 "LESSOPEN": "||/usr/bin/lesspipe.sh %s",
 "LOGNAME": "fale",
 "MAIL": "/var/mail/fale",
 "PATH": "/usr/local/bin:/usr/bin",
 "PWD": "/home/fale",
 "SHELL": "/bin/bash",
 "SHLVL": "2",
 "SSH_CLIENT": "86.187.141.39 37764 22",
 "SSH_CONNECTION": "86.187.141.39 37764 178.62.36.208 22",
 "SSH_TTY": "/dev/pts/0",
 "TERM": "rxvt-unicode-256color",
 "USER": "fale",
 "XDG_RUNTIME_DIR": "/run/user/1000",
 "XDG_SESSION_ID": "180",
 "_": "/usr/bin/python"
 },
 "ansible_eth0": {
 "active": true,
 "device": "eth0",
 "ipv4": {
 "address": "178.62.36.208",
 "broadcast": "178.62.63.255",
 "netmask": "255.255.192.0",
 "network": "178.62.0.0"
 },
 "ipv4_secondaries": [
 {
 "address": "10.16.0.7",
 "broadcast": "10.16.255.255",
 "netmask": "255.255.0.0",
 "network": "10.16.0.0"
 }
 ],
 "ipv6": [
 {
 "address": "fe80::601:e2ff:fef1:1301",
 "prefix": "64",
 "scope": "link"
 }
 ],
 "macaddress": "04:01:e2:f1:13:01",
 "module": "virtio_net",
 "mtu": 1500,
 "pciid": "virtio0",
 "promisc": false,
 "type": "ether"
 },
 "ansible_eth1": {
 "active": false,
 "device": "eth1",
 "macaddress": "04:01:e2:f1:13:02",
 "module": "virtio_net",
 "mtu": 1500,
 "pciid": "virtio1",
 "promisc": false,
 "type": "ether"
 },
 "ansible_fips": false,
 "ansible_form_factor": "Other",
 "ansible_fqdn": "test",
 "ansible_hostname": "test",
 "ansible_interfaces": [
 "lo",
 "eth1",
 "eth0"
 ],
 "ansible_kernel": "3.10.0-327.10.1.el7.x86_64",
 "ansible_lo": {
 "active": true,
 "device": "lo",
 "ipv4": {
 "address": "127.0.0.1",
 "broadcast": "host",
 "netmask": "255.0.0.0",
 "network": "127.0.0.0"
 },
 "ipv6": [
 {
 "address": "::1",
 "prefix": "128",
 "scope": "host"
 }
 ],
 "mtu": 65536,
 "promisc": false,
 "type": "loopback"
 },
 "ansible_machine": "x86_64",
 "ansible_machine_id": "fd8cf26e06e411e4a9d004010897bd01",
 "ansible_memfree_mb": 6,
 "ansible_memory_mb": {
 "nocache": {
 "free": 381,
 "used": 108
 },
 "real": {
 "free": 6,
 "total": 489,
 "used": 483
 },
 "swap": {
 "cached": 0,
 "free": 0,
 "total": 0,
 "used": 0
 }
 },
 "ansible_memtotal_mb": 489,
 "ansible_mounts": [
 {
 "device": "/dev/vda1",
 "fstype": "ext4",
 "mount": "/",
 "options": "rw,relatime,data=ordered",
 "size_available": 18368385024,
 "size_total": 21004894208,
 "uuid": "c5845b43-fe98-499a-bf31-4eccae14261b"
 }
 ],
 "ansible_nodename": "test",
 "ansible_os_family": "RedHat",
 "ansible_pkg_mgr": "yum",
 "ansible_processor": [
 "GenuineIntel",
 "Intel(R) Xeon(R) CPU E5-2630L v2 @ 2.40GHz"
 ],
 "ansible_processor_cores": 1,
 "ansible_processor_count": 1,
 "ansible_processor_threads_per_core": 1,
 "ansible_processor_vcpus": 1,
 "ansible_product_name": "Droplet",
 "ansible_product_serial": "NA",
 "ansible_product_uuid": "NA",
 "ansible_product_version": "20160415",
 "ansible_python_version": "2.7.5",
 "ansible_selinux": {
 "status": "disabled"
 },
 "ansible_service_mgr": "systemd",
 "ansible_ssh_host_key_dsa_public": "AAAAB3NzaC1kc3MAAACBAPEf4dzeET6ukHemTASsamoRLxo2R8iHg5J1bYQUyuggtRKlbRrHMtpQ8qN5CQNtp8J+2Hq6/JKiDF+cdxgOehf9b7F4araVvJxqx967RvLNBrMWXv7/4hi+efgXG9eejGoGQNAD66up/fkLMd0L8fwSwmTJoZXwOxFwcbnxCZsFAAAAFQDgK7fka+1AKjYZNFIfCB2b0ZitGQAAAIADeofiC5q+SLgEvkBCUCTyJ+EVb6WHeHbVdrpE2GdnUr03R6MmmYhYZMijruS/rcpzBLmi8juDkqAWy6Xqxd+DwixykntXPeUFS3F7LK5vNwFalaRltPwr4Azh+EeSUQ2Zz2AdKx6zSqtLOD8ZMPkRDvz4WGHGmeR+i7UFsFDZdgAAAIEAy26Tx0jAlY3mEaTW9lQ9DoGXgPBxsSX/XqeLh5wBaBO6AJaIrs0dQJdNeHcMhFy0seVkOMN1SpeoBTJSoTOx15HAGsKsAcmnA5mcJeUZqptVR6JxROztHw3zQePQ3/V3KQzAN31tIm3PbKztlEZbXRUM7RV5WsdRHTb8rutENhY=",
 "ansible_ssh_host_key_ecdsa_public": "AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBPDXQ9rjgDmUKsEWH4U2vg4iqtK+75urlj9nwW+rNNTFHTE5oG82sOlO6o0tUY8LXgB/tJnIcJ1hINdrWrZNpn4=",
 "ansible_ssh_host_key_rsa_public": "AAAAB3NzaC1yc2EAAAADAQABAAABAQCwQx5EElH7FeD/agB/gCJfBUEVhk44tldzdEzwc2IEbI59relTGNOU7soCCMcSH7nwlEbOOvmLa2R/YaXdHv/cb1aXBC/wj/m4ZHylBeF5qzECUkeaB3+CT+hp8qHHApclFr2lm2CwZ+YXjEyjJ3en4K3gLlIQyQjgE2F57kmD1FVVDSJFvNTn+NQvb3DPppND+HKEeHwrJ0GgznoP62yobEgriAIBSGf//0WHCO/9shEvauoRpPM+U9pU7lv637s7qyubIqyrs5fz3u34qBj8oCATOefRN1wsfJDeMG0D5ryI6BI6t/eAi8BPr7VHJSQBk+buM9Jr1yoMQTEasq2J",
 "ansible_swapfree_mb": 0,
 "ansible_swaptotal_mb": 0,
 "ansible_system": "Linux",
 "ansible_system_vendor": "DigitalOcean",
 "ansible_uptime_seconds": 603067,
 "ansible_user_dir": "/home/fale",
 "ansible_user_gecos": "",
 "ansible_user_gid": 1000,
 "ansible_user_id": "fale",
 "ansible_user_shell": "/bin/bash",
 "ansible_user_uid": 1000,
 "ansible_userspace_architecture": "x86_64",
 "ansible_userspace_bits": "64",
 "ansible_virtualization_role": "host",
 "ansible_virtualization_type": "kvm",
 "module_setup": true
 },
 "changed": false
}

```

如你所见，从这个庞大的选项列表中，你可以获得大量信息，并且可以像使用其他变量一样使用它们。让我们打印操作系统的名称和版本。为此，我们可以创建一个新的 playbook，名为 `setup_variables.yaml`，内容如下：

```
--- 
- hosts: all 
  remote_user: fale 
  tasks: 
  - name: Print OS and version 
    debug: 
      msg: '{{ ansible_distribution }} {{ ansible_distribution_version }}' 

```

使用以下方式执行：

```
$ ansible-playbook -itest01.fale.io, setup_variables.yaml

```

这将给我们以下输出：

```
PLAY [all] *******************************************************
TASK [setup] *****************************************************
ok: [test01.fale.io] 
TASK [Print OS and version] **************************************
ok: [test01.fale.io] => {
 "msg": "CentOS 7.2.1511" 
} 
PLAY RECAP *******************************************************
test01.fale.io     : ok=2   changed=0    unreachable=0    failed=0

```

如你所见，它按预期打印了操作系统的名称和版本。除了之前看到的方法，还可以通过命令行参数传递变量。事实上，如果我们查看 Ansible 的帮助文档，我们会注意到以下内容：

```
-e EXTRA_VARS, --extra-vars=EXTRA_VARS
set additional variables as key=value or YAML/JSON

```

这些相同的行也出现在 `ansible-playbook` 命令中。让我们做一个简单的 playbook，叫做 `cli_variables.yaml`，内容如下：

```
--- 
- hosts: all 
  remote_user: fale 
  tasks: 
  - name: Print variable 'name' 
    debug: 
      msg: '{{ name }}' 

```

使用以下方式执行：

```
$ ansible-playbook -i test01.fale.io, cli_variables.yaml -e 'name=test01'

```

我们将收到以下内容：

```
PLAY [all] *******************************************************
TASK [setup] *****************************************************
ok: [test01.fale.io]
TASK [Print variable 'name'] *************************************
ok: [test01.fale.io] => {
 "msg": "test01"
}
PLAY RECAP *******************************************************
test01.fale.io     : ok=2   changed=0    unreachable=0    failed=0

```

如果我们忘记添加额外的参数来指定变量，我们将按如下方式执行：

```
$ ansible-playbook -i test01.fale.io, cli_variables.yaml

```

我们将收到以下输出：

```
PLAY [all] *******************************************************
TASK [setup] *****************************************************
ok: [test01.fale.io] 
TASK [Print variable 'name'] *************************************
fatal: [test01.fale.io]: FAILED! => {"failed": true, "msg": "'name' is undefined"} 
NO MORE HOSTS LEFT ***********************************************
to retry, use: --limit @cli_variables.retry 
PLAY RECAP *******************************************************
test01.fale.io     : ok=1   changed=0    unreachable=0    failed=1

```

既然我们已经学习了 playbook 的基础知识，接下来让我们使用它们从零开始创建一个 Web 服务器。为此，我们从头开始，首先创建一个 Ansible 用户，然后再继续进行。

# 创建 Ansible 用户

当你创建一台机器（或从任何托管公司租用一台机器）时，它默认只会有 `root` 用户。让我们开始创建一个 playbook，确保创建一个 Ansible 用户，该用户可以通过 SSH 密钥访问，并且能够以其他用户的身份执行操作（`sudo`），且无需输入密码。我通常称这个 playbook 为 `firstrun.yaml`，因为我在创建新机器后立刻执行它，但之后就不再使用它了，因为它使用了出于安全原因禁用的 root 用户。我们的脚本将如下所示：

```
--- 
- hosts: all 
  user: root 
  tasks: 
  - name: Ensure ansible user exists 
    user: 
      name: ansible 
      state: present 
      comment: Ansible 
  - name: Ensure ansible user accepts the SSH key 
    authorized_key: 
      user: ansible 
      key: https://github.com/fale.keys 
   state: present 
  - name: Ensure the ansible user is sudoer with no password required 
    lineinfile: 
      dest: /etc/sudoers 
      state: present 
      regexp: '^ansible ALL\=' 
      line: 'ansible ALL=(ALL) NOPASSWD:ALL' 
      validate: 'visudo -cf %s' 

```

在运行之前，我们先看一下它。我们使用了三个不同的模块（`user`、`authorized_key` 和 `lineinfile`），这些模块我们之前从未见过。顾名思义，`user` 模块可以确保某个用户存在（或不存在）。

`authorized_key` 模块允许我们确保某个 SSH 密钥可以用来作为特定用户登录到该机器。这个模块不会替换该用户已经启用的所有 SSH 密钥，而只是添加（或删除）指定的密钥。如果你想更改此行为，可以使用 *exclusive* 选项，它允许你删除所有未在此步骤中指定的 SSH 密钥。

`lineinfile` 模块允许我们修改文件的内容。它的工作方式非常类似于 **sed**（流编辑器），你需要指定将用于匹配行的正则表达式，然后指定将替换匹配行的新行。如果没有匹配的行，将在文件末尾添加该行。现在让我们用以下方式运行它：

```
$ ansible-playbook -i test01.fale.io, firstrun.yaml

```

这将给我们以下结果：

```
PLAY [all] *******************************************************
TASK [setup] *****************************************************
ok: [test01.fale.io] 
TASK [Ensure ansible user exists] ********************************
changed: [test01.fale.io] 
TASK [Ensure ansible user accepts the SSH key] *******************
changed: [test01.fale.io] 
TASK [Ensure the anisble user is sudoer with no password required] *
changed: [test01.fale.io] 
PLAY RECAP *******************************************************
test01.fale.io     : ok=4   changed=3    unreachable=0    failed=0

```

# 配置基础服务器

在为 Ansible 创建了具有必要权限的用户之后，我们可以继续对操作系统进行一些其他小的更改。为了更清楚地了解，我们将逐步查看每个操作的执行方式，然后再来看整个 playbook。

## 启用 EPEL

EPEL 是企业 Linux 最重要的仓库，它包含了大量额外的包。它也是一个安全的仓库，因为 EPEL 中的任何包都不会与基础仓库中的包发生冲突。要在 RHEL/CentOS 7 中启用 EPEL，只需安装 `epel-release` 包即可。为了在 Ansible 中执行此操作，我们将使用：

```
- name: Ensure EPEL is enabled 
  yum: 
    name: epel-release 
    state: present 
  become: True 

```

如你所见，我们使用了 `yum` 模块，就像在本章的第一个示例中一样，指定了包名并要求它处于 present 状态。

## 安装 SELinux 的 Python 绑定

由于 Ansible 是用 Python 编写的，并且主要使用 Python 绑定来操作操作系统，我们将需要安装 SELinux 的 Python 绑定：

```
- name: Ensure libselinux-python is present 
  yum: 
    name: libselinux-python 
    state: present 
  become: True 
- name: Ensure libsemanage-python is present 
  yum: 
    name: libsemanage-python 
    state: present 
  become: True 

```

### 提示

这可以用更简短的方式写成，使用一个循环，但我们将在下一章看到如何做到这一点。

## 升级所有已安装的包

要升级所有已安装的包，我们需要再次使用 `yum` 模块，但使用不同的参数，实际上我们会使用：

```
- name: Ensure we have last version of every package 
  yum: 
    name: "*" 
    state: latest 
  become: True 

```

如你所见，我们将 "`*`" 作为包名（这表示一个通配符，用于匹配所有已安装的包），并且 `state` 是 `latest`。这将把所有已安装的包升级到最新版本。

如果你记得，我们在讲解 "`present`" 状态时提到，它会安装最后一个可用版本。那么，"`present`" 和 "`latest`" 有什么区别呢？Present 会在包未安装时安装最新版本，而如果包已安装（无论版本如何），它会继续前进，不做任何更改。Latest 会在包未安装时安装最新版本，而如果包已安装，则会检查是否有更新版本，如果有，Ansible 将更新该包。

## 确保 NTP 已安装、配置并正在运行

为了确保 NTP 已安装，我们使用了`yum`模块：

```
- name: Ensure NTP is installed 
  yum: 
    name: ntp 
    state: present 
  become: True 

```

现在我们知道 NTP 已经安装，我们应该确保服务器使用的是我们需要的`timezone`。为此，我们将在`/etc/localtime`中创建一个符号链接，指向所需的`zoneinfo`文件：

```
- name: Ensure the timezone is set to UTC 
  file: 
    src: /usr/share/zoneinfo/GMT 
    dest: /etc/localtime 
    state: link 
  become: True 

```

正如你所看到的，我们使用了`file`模块告诉 Ansible，指定它需要是一个符号链接（`state: link`）。

为了完成 NTP 配置，我们需要启动`ntpd`服务，并确保它在每次启动时都会运行：

```
- name: Ensure the NTP service is running and enabled 
  service: 
    name: ntpd 
    state: started 
    enabled: True 
  become: True 

```

## 确保 FirewallD 已安装并启用

如你所想，第一步是确保 FirewallD 已安装：

```
- name: Ensure FirewallD is installed 
  yum: 
    name: firewalld 
    state: present 
  become: True 

```

由于我们想确保启用 FirewallD 时不会丢失 SSH 连接，我们确保 SSH 流量始终可以通过防火墙：

```
- name: Ensure SSH can pass the firewall 
  firewalld: 
    service: ssh 
    state: enabled 
    permanent: True 
    immediate: True 
  become: True 

```

为此，我们使用了`firewalld`模块。该模块的参数与`firewall-cmd`控制台非常相似。你需要指定允许通过防火墙的服务，是否希望此规则立即生效，以及是否希望此规则是永久性的，以便在重启后规则仍然存在。

### 提示

你可以使用`service`参数指定服务名称（例如`'ssh'`），也可以使用`port`参数指定端口（例如`'22/tcp'`）。

现在我们已经安装了 FirewallD，并且确认我们的 SSH 连接会继续保持，我们可以像启用任何其他服务一样启用它：

```
- name: Ensure FirewallD is running 
  service: 
    name: firewalld 
    state: started 
    enabled: True 
  become: True 

```

## 添加定制的 MOTD

要添加 MOTD，我们需要一个模板，所有服务器都使用相同的模板，并且需要一个任务来使用该模板。

我发现为每个服务器添加 MOTD 非常有用。如果使用 Ansible，它更加有用，因为你可以用它来提醒用户，系统的更改可能会被 Ansible 覆盖。我的常用模板叫做`'motd'`，内容如下：

```
                This system is managed by Ansible 
  Any change done on this system could be overwritten by Ansible 

OS: {{ ansible_distribution }} {{ ansible_distribution_version }} 
Hostname: {{ inventory_hostname }} 
eth0 address: {{ ansible_eth0.ipv4.address }} 
All connections are monitored and recorded 
    Disconnect IMMEDIATELY if you are not an authorized user

```

这是一个`jinja2`模板，它允许我们使用在 playbooks 中设置的所有变量。这也使我们能够使用复杂的条件和循环语法，这些将在本章稍后看到。要使用模板填充文件，我们需要使用：

```
- name: Ensure the MOTD file is present and updated 
  template: 
    src: motd 
    dest: /etc/motd 
    owner: root 
    group: root 
    mode: 0644 
  become: True 

```

`template`模块允许我们指定一个本地文件（`src`），该文件将由`jinja2`解释，并且此操作的输出将保存在远程机器上的特定路径（`dest`），由特定用户（`owner`）和组（`group`）拥有，并具有特定的访问模式（`mode`）。

## 更改主机名

为了简化操作，我发现一个有用的方法是将机器的主机名设置为有意义的名称。为此，我们可以使用一个非常简单的 Ansible 模块，名为`hostname`：

```
- name: Ensure the hostname is the same of the inventory 
  hostname: 
    name: "{{ inventory_hostname }}" 
  become: True 

```

## 审查并运行 playbook

将所有内容整合起来，我们现在有以下 playbook（为简化起见，命名为`common_tasks.yaml`）：

```
--- 
- hosts: all 
  remote_user: ansible 
  tasks: 
  - name: Ensure EPEL is enabled 
    yum: 
      name: epel-release 
      state: present 
    become: True 
  - name: Ensure libselinux-python is present 
    yum: 
      name: libselinux-python  
      state: present 
    become: True 
  - name: Ensure libsemanage-python is present 
    yum: 
      name: libsemanage-python 
      state: present 
    become: True 
  - name: Ensure we have last version of every package 
    yum: 
      name: "*" 
      state: latest 
    become: True 
  - name: Ensure NTP is installed 
    yum: 
      name: ntp 
      state: present 
    become: True 
  - name: Ensure the timezone is set to UTC 
    file: 
      src: /usr/share/zoneinfo/GMT 
      dest: /etc/localtime 
      state: link 
    become: True 
  - name: Ensure the NTP service is running and enabled 
    service: 
      name: ntpd 
      state: started 
      enabled: True 
    become: True 
  - name: Ensure FirewallD is installed 
    yum: 
      name: firewalld 
      state: present 
    become: True 
  - name: Ensure FirewallD is running 
    service: 
      name: firewalld 
      state: started 
      enabled: True 
    become: True 
  - name: Ensure SSH can pass the firewall 
    firewalld: 
      service: ssh 
      state: enabled 
      permanent: True 
      immediate: True 
    become: True 
  - name: Ensure the MOTD file is present and updated 
    template: 
      src: motd 
      dest: /etc/motd 
      owner: root 
      group: root 
      mode: 0644 
    become: True 
  - name: Ensure the hostname is the same of the inventory 
    hostname: 
      name: "{{ inventory_hostname }}" 
    become: True  

```

由于这个`playbook`相当复杂，我们可以运行以下命令：

```
$ ansible-playbook common_tasks.yaml --list-tasks

```

这要求 Ansible 以简短的形式打印出所有任务，以便我们可以快速查看 `playbook` 执行了哪些任务。输出应该类似于以下内容：

```
playbook: common_tasks.yaml 
  play #1 (all): all TAGS: [] 
    tasks: 
      Ensure EPEL is enabled TAGS: [] 
      Ensure libselinux-python is present TAGS: [] 
      Ensure libsemanage-python is present TAGS: [] 
      Ensure we have last version of every package TAGS: [] 
      Ensure NTP is installed TAGS: [] 
      Ensure the timezone is set to UTC TAGS: [] 
      Ensure the NTP service is running and enabled TAGS: [] 
      Ensure FirewallD is installed TAGS: [] 
      Ensure FirewallD is running TAGS: [] 
      Ensure SSH can pass the firewall TAGS: [] 
      Ensure the MOTD file is present and updated TAGS: [] 
      Ensure the hostname is the same of the inventory TAGS: [] 

```

现在我们可以使用以下命令运行 `playbook`：

```
$ ansible-playbook -itest01.fale.io, common_tasks.yaml

```

我们将收到以下输出：

```
PLAY [all] ******************************************************* 

TASK [setup] ***************************************************** 
ok: [test01.fale.io] 

TASK [Ensure EPEL is enabled] ************************************ 
changed: [test01.fale.io] 

TASK [Ensure libselinux-python is present] *********************** 
ok: [test01.fale.io] 

TASK [Esure libsemanage-python is present] *********************** 
ok: [test01.fale.io] 

TASK [Ensure we have last version of every package] ************** 
changed: [test01.fale.io] 

TASK [Ensure NTP is installed] *********************************** 
ok: [test01.fale.io] 

TASK [Ensure the timezone is set to UTC] ************************* 
changed: [test01.fale.io] 

TASK [Ensure the NTP service is running and enabled] ************* 
changed: [test01.fale.io] 

TASK [Ensure FirewallD is installed] ***************************** 
ok: [test01.fale.io] 

TASK [Ensure FirewallD is running] ******************************* 
changed: [test01.fale.io] 

TASK [Ensure SSH can pass the firewall] ************************** 
ok: [test01.fale.io] 

TASK [Ensure the MOTD file is present and updated] *************** 
changed: [test01.fale.io] 

TASK [Ensure the hostname is the same of the inventory] ********** 
changed: [test01.fale.io] 

PLAY RECAP ******************************************************* 
test01.fale.io     : ok=9   changed=7    unreachable=0    failed=0

```

# 安装和配置 Web 服务器

现在我们已经对操作系统做了一些通用的更改，让我们开始实际创建一个 Web 服务器。我们将这两个阶段分开，以便可以在每台机器上共享第一阶段，并仅将第二阶段应用于 Web 服务器。

在第二阶段，我们将创建一个名为 `webserver.yaml` 的新 playbook，内容如下：

```
--- 
- hosts: all 
  remote_user: ansible 
  tasks: 
  - name: Ensure the HTTPd package is installed 
    yum: 
      name: httpd 
      state: present 
    become: True 
  - name: Ensure the HTTPd service is enabled and running 
    service: 
      name: httpd 
      state: started 
      enabled: True 
    become: True 
  - name: Ensure HTTP can pass the firewall 
    firewalld: 
      service: http 
      state: enabled 
      permanent: True 
      immediate: True 
    become: True 
  - name: Ensure HTTPS can pass the firewall 
    firewalld: 
      service: https 
      state: enabled 
      permanent: True 
      immediate: True 
    become: True   

```

如你所见，前两个任务与本章开始时示例中的任务相同，最后两个任务用于指示 FirewallD 允许 HTTP 和 HTTPS 流量通过。

让我们使用以下命令运行这个脚本：

```
$ ansible-playbook -i test01.fale.io, webserver.yaml

```

这将导致以下结果：

```
PLAY [all] *******************************************************
TASK [setup] *****************************************************
ok: [test01.fale.io] 
TASK [Ensure the HTTPd package is installed] *********************
changed: [test01.fale.io] 
TASK [Ensure the HTTPd service is enabled and running] ***********
changed: [test01.fale.io] 
TASK [Ensure HTTP can pass the firewall] *************************
changed: [test01.fale.io] 
TASK [Ensure HTTPS can pass the firewall] ************************
changed: [test01.fale.io] 
PLAY RECAP *******************************************************
test01.fale.io    : ok=5    changed=4    unreachable=0    failed=0

```

现在我们有了 Web 服务器，让我们发布一个小型的单页静态网站。

# 发布一个网站

由于我们的网站将是一个简单的单页网站，我们可以通过一个 Ansible 任务轻松创建并发布它。为了让这个页面更有趣，我们将从一个模板开始，Ansible 会用一些关于机器的数据填充该模板。发布这个页面的脚本将被命名为 `deploy_website.yaml`，并具有以下内容：

```
--- 
- hosts: all 
  remote_user: ansible 
  tasks: 
  - name: Ensure the website is present and updated 
    template: 
      src: index.html.j2 
      dest: /var/www/html/index.html 
      owner: root 
      group: root 
      mode: 0644 
    become: True 

```

让我们从一个简单的模板开始，我们将其命名为 `index.html.j2`：

```
<html> 
    <body> 
        <h1>Hello World!</h1> 
    </body> 
</html> 

```

现在我们可以通过运行以下命令测试我们的网站部署：

```
$ ansible-playbook -i test01.fale.io, deploy_website.yaml

```

我们应该收到以下输出：

```
PLAY [all] *******************************************************
TASK [setup] *****************************************************
ok: [test01.fale.io] 
TASK [Ensure the website is present and updated] *****************
changed: [test01.fale.io] 
PLAY RECAP *******************************************************
test01.fale.io    : ok=2    changed=1    unreachable=0    failed=0

```

如果你现在在浏览器中访问你的测试机器的 IP/FQDN，你将看到 "Hello World!" 页面。

# Jinja2 模板

**Jinja2** 是一个广泛使用且功能全面的 Python 模板引擎。我们来看看一些有助于 Ansible 的语法。这个段落并不打算替代官方文档，但其目的是教你一些在使用 Ansible 时非常有用的组件。

## 变量

如我们所见，我们可以通过 '`{{ VARIABLE_NAME }}`' 语法简单地打印变量内容。如果我们想打印数组中的某个元素，可以使用 '`{{ ARRAY_NAME['KEY'] }}`'，如果我们想打印对象的属性，则可以使用 '`{{ OBJECT_NAME.PROPERTY_NAME }}`'。

所以我们可以通过以下方式改进我们之前的静态页面：

```
<html> 
    <body> 
        <h1>Hello World!</h1> 
        <p>This page was created on {{ ansible_date_time.date }}.</p> 
    </body> 
</html> 

```

## 过滤器

有时我们可能想稍微改变字符串的样式，而不为此编写特定的代码。例如，我们可能希望将某些文本转换为大写。为此，我们可以使用 Jinja2 的一个过滤器，例如：'`{{ VARIABLE_NAME | capitalize }}`'。Jinja2 提供了许多过滤器，你可以在以下地址找到完整的过滤器列表：[`jinja.pocoo.org/docs/dev/templates/#builtin-filters`](http://jinja.pocoo.org/docs/dev/templates/#builtin-filters)。

## 条件语句

在模板引擎中，你可能经常会发现一个有用的功能，那就是根据字符串的内容（或存在与否）打印不同的字符串。因此，我们可以通过以下方式改善我们的静态网页：

```
<html> 
    <body> 
        <h1>Hello World!</h1> 
        <p>This page was created on {{ ansible_date_time.date }}.</p> 
{% if ansible_eth0.active == True %} 
        <p>eth0 address {{ ansible_eth0.ipv4.address }}.</p> 
{% endif %} 
    </body> 
</html> 

```

如你所见，我们已经添加了打印主 IPv4 地址的功能，针对的是 `eth0` 连接，前提是连接为 `active`。使用条件语句时，我们还可以使用这些测试。

### 注意

如需完整列表，请参考：[`jinja.pocoo.org/docs/dev/templates/#builtin-tests`](http://jinja.pocoo.org/docs/dev/templates/#builtin-tests)。

因此，为了获得相同的结果，我们也可以写出以下内容：

```
<html> 
    <body> 
        <h1>Hello World!</h1> 
        <p>This page was created on {{ ansible_date_time.date }}.</p> 
{% if ansible_eth0.active is equalto True %} 
        <p>eth0 address {{ ansible_eth0.ipv4.address }}.</p> 
{% endif %} 
    </body> 
</html> 

```

有许多不同的测试可以帮助你创建易于阅读且有效的模板。

## 循环

`jinja2` 模板系统还提供了创建循环的功能。让我们在页面中添加一个功能，打印每个设备的主 IPv4 网络地址，而不仅仅是 `eth0`。我们将得到以下代码：

```
<html> 
    <body> 
        <h1>Hello World!</h1> 
        <p>This page was created on {{ ansible_date_time.date }}.</p> 
        <p>This machine can be reached on the following IP addresses</p> 
        <ul> 
{% for address in ansible_all_ipv4_addresses %} 
            <li>{{ address }}</li> 
{% endfor %} 
        </ul> 
    </body> 
</html> 

```

如你所见，如果你已经了解 Python，循环的语法就会很熟悉。

这几页关于 Jinja2 模板的内容并不能替代官方文档。事实上，Jinja2 模板比我们所看到的要强大得多。这里的目标仅仅是给你提供一些在 Ansible 中最常用的基本 Jinja2 模板。

# 总结

在本章中，我们开始了解 YAML，了解了什么是 playbook，它是如何工作的，以及如何使用它来创建一个 web 服务器（以及用于静态网站的部署）。我们还看到了多个 Ansible 模块，如 user、yum、service、FirewalID、lineinfile 和 template 模块。在本章结束时，我们重点讨论了模板。

在下一章，我们将讨论清单，以便我们能够轻松管理多个机器。
