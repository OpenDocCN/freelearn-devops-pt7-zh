# 第三章：高级剧本

到目前为止，我们所查看的剧本都很简单，只是按顺序运行了一些模块。Ansible 允许对剧本的执行进行更多控制。通过以下技巧，你应该能够执行即使是最复杂的部署：

+   并行执行操作

+   循环

+   条件执行

+   任务委派

+   额外变量

+   查找带变量的文件

+   环境变量

+   外部数据查找

+   存储数据

+   处理数据

+   调试剧本

# 并行执行操作

默认情况下，Ansible 只会进行最多五次分叉，因此它一次只会在五台不同的机器上运行操作。如果你有大量的机器，或者你已经降低了这个最大分叉值，那么你可能希望异步启动任务。Ansible 执行此操作的方法是启动任务，然后轮询等待它完成。这允许 Ansible 在所有需要的机器上启动作业，同时仍然使用最大分叉数。

要并行执行操作，请使用 `async` 和 `poll` 关键字。`async` 关键字触发 Ansible 并行执行任务，其值将是 Ansible 等待命令完成的最大时间。`poll` 的值告诉 Ansible 每隔多长时间轮询一次，以检查命令是否完成。

如果你想在整个集群上运行 `updatedb`，它可能看起来像以下代码：

```
- hosts: all
  tasks:
    - name: Install mlocate
      yum: name=mlocate state=installed

    - name: Run updatedb
      command: /usr/bin/updatedb
      async: 300
      poll: 10
```

你会注意到，当你在超过五台机器上运行前面的示例时，`yum` 模块与 `command` 模块的行为不同。`yum` 模块会在前五台机器上运行，然后是接下来的五台，依此类推。然而，`command` 模块会在所有机器上并行运行，并在完成后显示状态。

如果你的命令启动了一个守护进程，最终会在某个端口监听，你可以在不进行轮询的情况下启动它，这样 Ansible 就不会检查它是否完成。然后，你可以继续进行其他操作，并稍后使用 `wait_for` 模块检查是否完成。要配置 Ansible 不等待任务完成，将 `poll` 的值设置为 `0`。

最后，如果你的任务需要极长的时间来执行，你可以告诉 Ansible 等待任务完成，无论它需要多长时间。为此，将 `async` 的值设置为 `0`。

在以下情况中，你需要使用 Ansible 的轮询功能：

+   你有一个长期运行的任务，可能会触发超时

+   你需要在大量机器上执行操作

+   你有一个操作，不需要等待其完成

也有一些情况，你不应使用 `async` 或 `poll`：

+   如果你的任务获取了锁，阻止其他操作的运行

+   你的任务只需要很短的时间来执行

# 循环

Ansible 允许你使用不同的输入多次重复运行一个模块，例如，如果你有多个文件需要设置相似的权限。这可以节省你大量的重复工作，并允许你对事实和变量进行迭代。

要做到这一点，您可以在操作上使用`with_items`关键字，并将值设置为您将要迭代的项目列表。这将为模块创建一个名为`item`的变量，该变量将逐个设置为您的模块正在迭代的每个项目。某些模块，例如`yum`，将优化此过程，以便不是为每个软件包执行单独的事务，而是一次性操作它们。

使用`with_items`，代码看起来像这样：

```
tasks:
- name: Secure config files file:
    path: "/etc/{{ item }}"
    mode: 0600
    owner: root
    group: root with_items: - my.cnf - shadow - fstab
```

除了循环固定项目或变量外，Ansible 还为我们提供了一个称为**查找插件**的工具。这些插件允许您告诉 Ansible 从外部某处获取数据。例如，您可能希望找到所有与特定模式匹配的文件，然后上传它们。

在此示例中，我们上传目录中的所有公钥，然后将它们组合成`authorized_keys`文件，如下例所示：

```
tasks: - name: Make key directory file:
    path: /root/.sshkeys
    ensure: directory
    mode: 0700
    owner: root
    group: root - name: Upload public keys copy:
    src: "{{ item }}"
    dest: /root/.sshkeys
    mode: 0600
    owner: root
    group: root with_fileglob: - keys/*.pub - name: Assemble keys into authorized_keys file assemble:
    src: /root/.sshkeys
    dest: /root/.ssh/authorized_keys
    mode: 0600
    owner: root
    group: root
```

可以在以下情况下使用重复模块：

+   多次重复一个模块，并使用类似的设置

+   迭代列表的所有值

+   创建许多文件，稍后使用`assemble`模块将它们合并成一个大文件

+   当与`with_fileglob`查找插件结合使用时，可以复制文件目录

# 条件执行

一些模块，例如`copy`模块，提供配置机制以跳过模块的执行。您还可以配置自己的跳过条件，只有当它们解析为`true`时才执行模块。如果您的服务器使用不同的打包系统或具有不同的文件系统布局，这将非常方便。它还可与`set_fact`模块一起使用，允许您计算许多不同的内容。

要跳过一个模块，您可以使用`when`关键字；这允许您提供一个条件。如果您设置的条件解析为假，则会跳过该模块。您分配给`when`的值是一个 Python 表达式。您可以在此时使用任何可用的变量或事实。

### 注意

如果您想根据条件处理列表中的某些项，则简单地使用`when`子句。`when`子句会分别处理列表中的每个项；正在处理的项可以通过`{{ item }}`作为变量使用。

以下代码是一个示例，显示如何在 Debian 和 Red Hat 系统中选择`apt`和`yum`之间的选择。

```
---
- name: Install VIM
  hosts: all
  tasks:
    - name: Install VIM via yum
      yum:
        name: vim-enhanced
        state: installed
      when: ansible_os_family == "RedHat"

    - name: Install VIM via apt
      apt:
        name: vim
        state: installed
      when: ansible_os_family == "Debian"

    - name: Unexpected OS family
      debug:
        msg: "OS Family {{ ansible_os_family }} is not supported"
        fail: yes
      when: ansible_os_family != "RedHat" and ansible_os_family != "Debian"
```

还有第三个子句，如果未识别操作系统，则打印消息并失败。

### 注意

这个功能可以在特定点暂停，并等待用户干预后继续。通常，当 Ansible 遇到错误时，它会停止当前操作而不运行任何处理程序。使用此功能，你可以添加带有条件的`pause`模块，该条件在意外情况下触发。这样，在正常情况下`pause`模块会被忽略；但在意外情况下，它将允许用户干预并在安全时继续。任务会像这样：

```
name: pause for unexpected conditions
pause: prompt="Unexpected OS"
when: ansible_os_family != "RedHat"
```

跳过某些操作有很多用法，以下是其中的一些：

+   解决操作系统之间的差异

+   提示用户并仅在他们请求时执行相应操作

+   通过避免使用你知道不会改变任何内容但可能需要较长时间的模块来提高性能

+   拒绝修改具有特定文件存在的系统

+   检查自定义脚本是否已经执行

# 任务委派

默认情况下，Ansible 会在配置的机器上同时运行所有任务。这对于需要配置大量独立机器，或者每台机器都负责与其他远程机器通信其状态的情况非常有效。然而，如果你需要在与 Ansible 当前操作的主机不同的主机上执行操作，你可以使用委派功能。

Ansible 可以配置为在与当前配置的主机不同的主机上运行任务，使用`delegate_to`键。该模块仍然会针对每台机器运行一次，但它将不再在目标机器上运行，而是在委派的主机上运行。可用的事实将是适用于当前主机的事实。这里，我们展示一个使用`get_url`选项的 playbook，从一堆 Web 服务器下载配置。

```
---
- name: Fetch configuration from all webservers
  hosts: webservers
  tasks:
    - name: Get config
      get_url:
        dest: "configs/{{ ansible_hostname }}"
        force: yes
        url: "http://{{ ansible_hostname }}/diagnostic/config"
      delegate_to: localhost
```

如果你要委派到`localhost`，在定义操作时可以使用快捷方式，自动使用本地机器。如果你将操作行的键定义为`local_action`，则意味着将委派到`localhost`。如果我们在前面的示例中使用这个，它会稍微简短一些，看起来会是这样：

```
--- #1
- name: Fetch configuration from all webservers     #2
  hosts: webservers     #3
  tasks:     #4
    - name: Get config     #5
      local_action: get_url dest=configs/{{ ansible_hostname }}.cfg url=http://{{ ansible_hostname }}/diagnostic/config     #6
```

委派不仅限于本地机器。你可以委派给清单中的任何主机。你可能希望进行委派的其他原因包括：

+   在部署前将主机从负载均衡器中移除

+   更改 DNS 以将流量引导到你即将更改的服务器之外

+   在存储设备上创建 iSCSI 卷

+   使用外部服务器检查是否能正常访问网络外部

# 附加变量

你可能已经在上一章的模板示例中看到了我们使用了一个名为`group_names`的变量。这是 Ansible 本身提供的魔法变量之一。在写这篇文章时，共有七个这样的变量，接下来的章节将描述这些变量。

## 主机变量

`hostvars` 变量允许你检索当前 play 所处理的所有主机的变量。如果当前 play 尚未在该托管主机上运行 `setup` 模块，则只会提供该主机的变量。你可以像访问其他复杂变量一样访问它，例如 `${hostvars.hostname.fact}`，因此，要获取名为 `ns1` 的服务器上运行的 Linux 发行版，可以使用 `${hostvars.ns1.ansible_distribution}`。以下示例将名为 `ns1` 的服务器设置为名为 zone master 的变量。然后它调用 `template` 模块，利用此信息为每个区域设置主机。

```
---
- name: Setup DNS Servers
  hosts: allnameservers
  tasks:
    - name: Install BIND
      yum:
        name: named
        state: installed

- name: Setup Slaves
  hosts: slavenamesservers
  tasks:
    - name: Get the masters IP
      set_fact:
        dns_master: "{{ hostvars.ns1.ansible_default_ipv4.address }}"

    - name: Configure BIND
      template:
        dest: /etc/named.conf src: templates/named.conf.j2
```

### 注意

使用 `hostvars`，你可以进一步抽象环境中的模板。如果你嵌套变量调用，那么你就可以在 play 的变量部分中添加主机名，而不是直接放入 IP 地址。要找到名为 `the_machine` 的机器的地址，你可以使用 `{{ hostvars.[the_machine].default_ipv4.address }}`。

## `groups` 变量

`groups` 变量包含按清单组分组的所有主机的列表。它让你访问你配置的所有主机。这是一个非常强大的工具，它允许你遍历整个组，并为每个主机应用一个动作。

```
---
- name: Configure the database
  hosts: dbservers
  user: root
  tasks:
    - name: Install mysql
      yum:
        name: "{{ item }}"
        state: installed
      with_items:
      - mysql-server
      - MySQL-python

    - name: Start mysql
      service:
        name: mysqld
        state: started
        enabled: true

    - name: Create a user for all app servers
      with_items: groups.appservers
      mysql_user:
        name: kate
        password: test
        host: "{{ hostvars.[item].ansible_eth0.ipv4.address }}" state: present
```

### 注意

`groups` 变量不包含组中的实际主机；它包含表示主机名的字符串，这些字符串来自清单。这意味着，如果需要，你必须使用嵌套变量扩展来访问 `hostvars` 变量。

你甚至可以使用这个变量为所有机器创建 `known_hosts` 文件，其中包含所有其他机器的 `host` 密钥。这将允许你在一台机器与另一台机器之间进行 SSH 连接，而无需确认远程主机的身份。它还会在机器退出服务时自动删除它们，或在机器被替换时进行更新。以下是一个模板，用于创建一个 `known_hosts` 文件：

```
{% for host in groups['all'] %}
{{ hostvars[host]['ansible_hostname'] }}
{{ hostvars[host]['ansible_ssh_host_key_rsa_public'] }}
{% endfor %}
```

使用此模板的 playbook 将如下所示：

```
---
hosts: all
tasks:
- name: Setup known hosts
  hosts: all
  tasks:
    - name: Create known_hosts
      template:
        src: templates/known_hosts.j2 dest: /etc/ssh/ssh_known_hosts
        owner: root
        group: root mode: 0644
```

## `group_names` 变量

`group_names` 变量包含一个字符串列表，列出了当前主机所属的所有组。这不仅对于调试有用，还可以用于检测组成员身份的条件。在上一章中，我们使用这个变量设置了一个名称服务器。

这个变量主要用于跳过某个任务或在模板中作为条件。例如，如果你有两种 SSH 守护进程配置，一种是安全的，另一种是较不安全的，但你只希望在安全组中的机器上使用安全配置，你可以这样做：

```
- name: Setup SSH
  hosts: sshservers
  tasks:
    - name: For secure machines
      set_fact:
        sshconfig: files/ssh/sshd_config_secure
      when: "'secure' in group_names"

    - name: For non-secure machines
      set_fact:
        sshconfig: files/ssh/sshd_config_default
      when: "'secure' not in group_names"

    - name: Copy over the config
      copy:
        src: "{{ sshconfig }}"
        dest: /tmp/sshd_config
```

### 注意

在之前的示例中，我们使用了 `set_fact` 模块来为每个情况设置事实值，然后使用了 `copy` 模块。我们本可以用 `copy` 模块代替 `set_fact` 模块，从而减少一个任务。这么做的原因是 `set_fact` 模块在本地运行，而 `copy` 模块在远程运行。当你首先使用 `set_fact` 模块并仅调用一次 `copy` 模块时，所有机器上的复制任务会并行执行。如果你使用两个带有条件的 `copy` 模块，那么每个模块会分别在相关机器上执行。由于 `copy` 是这两者中更长的任务，最适合并行执行。

## `inventory_hostname` 变量

`inventory_hostname` 变量存储了作为清单中记录的服务器主机名。如果你选择不在当前主机上运行 `setup` 模块，或者由于各种原因 `setup` 模块检测到的值不正确，那么应该使用此变量。这在你进行机器初始设置并更改主机名时非常有用。

## `inventory_hostname_short` 变量

`inventory_hostname_short` 变量与之前的变量相同；然而，它仅包括第一个点之前的字符。因此，对于 `host.example.com`，它将返回 `host`。

## `inventory_dir` 变量

`inventory_dir` 变量是包含清单文件的目录的路径名。

## `inventory_file` 变量

`inventory_file` 变量与之前的变量相同，唯一的不同是它还包含了文件名。

# 使用变量查找文件

所有模块都可以通过解引用 `{{` 和 `}}` 来将变量作为其参数的一部分。你可以利用这一点根据变量加载特定文件。例如，你可能想根据正在使用的架构选择不同的 `config` 文件来配置 NRPE（一个 Nagios 检查守护进程）。以下是这样的实现方式：

```
---
- name: Configure NRPE for the right architecture
  hosts: ansibletest
  user: root
  tasks:
    - name: Copy in the correct NRPE config file
      copy:
        src: "files/nrpe.{{ ansible_architecture }}.conf" dest: "/etc/nagios/nrpe.cfg"
```

在 `copy` 和 `template` 模块中，你还可以配置 Ansible 查找一组文件，并通过第一个文件来找到它们。这允许你配置文件查找路径；如果未找到该文件，将使用第二个文件，以此类推，直到找到文件或到达列表末尾。如果文件未找到，则模块将失败。此功能通过 `first_available_file` 键触发，并在操作中引用 `{{ item }}`。以下代码是该功能的示例：

```
---
- name: Install an Apache config file
  hosts: ansibletest
  user: root
  tasks:
   - name: Get the best match for the machine
     copy:
       dest: /etc/apache.conf
       src: "{{ item }}"
     first_available_file:
      - "files/apache/{{ ansible_os_family }}-{{ ansible_architecture }}.cfg"
      - "files/apache/default-{{ ansible_architecture }}.cfg"
      - files/apache/default.cfg
```

### 注意

请记住，你可以从 Ansible 命令行工具运行 `setup` 模块。当你在 playbook 或模板中大量使用变量时，这非常有用。要检查某个 play 将可用的事实，只需复制主机模式的值并运行以下命令：

```
ansible [host pattern] -m setup

```

在 CentOS x86_64 机器上，此配置会在浏览`files/apache/`时首先查找`RedHat-x86_64.cfg`文件。如果该文件不存在，则会在浏览`file/apache/`时查找`default-x86_64.cfg`文件，最后如果什么都没有找到，它会尝试使用`default.cfg`。

# 环境变量

通常，Unix 命令会利用某些环境变量。常见的例子有 C Makefile、安装程序和 AWS 命令行工具。幸运的是，Ansible 使这变得非常简单。如果你想将文件从远程机器上传到 Amazon S3，你可以按如下方式设置 Amazon 访问密钥。你还会看到我们安装了 EPEL，以便能够安装 pip，pip 用于安装 AWS 工具。

```
---
- name: Upload a remote file via S3
  hosts: ansibletest
  user: root
  tasks:
    - name: Setup EPEL
      command: >
        rpm -ivh http://download.fedoraproject.org/pub/epel/6/i386/ epel-release-6-8.noarch.rpm
        creates=/etc/yum.repos.d/epel.repo

    - name: Install pip
      yum:
        name: python-pip
        state: installed

    - name: Install the AWS tools
      pip:
        name: awscli
        state: present

    - name: Upload the file
      shell: >
        aws s3 put-object
        --bucket=my-test-bucket
        --key={{ ansible_hostname }}/fstab
        --body=/etc/fstab
        --region=eu-west-1
      environment:
        AWS_ACCESS_KEY_ID: XXXXXXXXXXXXXXXXXXX
        AWS_SECRET_ACCESS_KEY: XXXXXXXXXXXXXXXXXXXXX
```

### 注释

在内部，Ansible 将环境变量设置到 Python 代码中；这意味着任何已经使用环境变量的模块都可以利用这里设置的变量。如果你编写自己的模块，你应该考虑某些参数是否应该作为环境变量而不是参数使用。

一些 Ansible 模块，如`get_url`、`yum`和`apt`，也将使用环境变量来设置它们的代理服务器。以下是一些你可能想要设置环境变量的其他情况：

+   运行应用程序安装程序

+   在使用`shell`模块时向路径中添加额外的项

+   从系统库搜索路径中未包含的位置加载库

+   在运行模块时使用`LD_PRELOAD`黑客技术

# 外部数据查找

Ansible 在版本 0.9 中引入了查找插件。这些插件允许 Ansible 从外部源获取数据。Ansible 提供了多个插件，但你也可以编写自己的插件。这为配置的灵活性打开了大门。

查找插件是用 Python 编写的，并且在控制机器上运行。它们有两种执行方式：直接调用和`with_*`键。直接调用在你想像使用变量一样使用它们时很有用。使用`with_*`键在你想将它们作为循环使用时很有用。在前面的部分中，我们讨论了`with_fileglob`，它就是这种情况的一个例子。

在下一个示例中，我们直接使用查找插件从`environment`中获取`http_proxy`值，并将其传递到配置的机器。这确保了我们正在配置的机器将使用相同的代理服务器来下载文件。

```
---
- name: Downloads a file using a proxy
  hosts: all
  tasks:
    - name: Download file
      get_url:
        dest: /var/tmp/file.tar.gz url: http://server/file.tar.gz
      environment:
        http_proxy: "{{ lookup('env', 'http_proxy') }}"
```

### 注释

你也可以在变量部分使用查找插件。这不会立即查找结果并将其放入变量中，就像你可能想的那样；相反，它会将结果作为宏存储，并在每次使用时查找它。如果你正在使用某些值可能随时间变化的内容，这一点很有用。

使用 `with_*` 形式的 lookup 插件将允许你遍历一些通常无法遍历的内容。你可以像这样使用任何插件，但返回列表的插件最为有用。以下代码演示了如何动态地注册一个 `webapp` farm。

```
---
- name: Registers the app server farm
  hosts: localhost
  connection: local
  vars:
    hostcount: 5
  tasks:
   - name: Register the webapp farm
      local_action: add_host name={{ item }} groupname=webapp
      with_sequence: start=1 end={{ hostcount }} format=webapp%02x
```

如果你使用这个例子，你可以添加一个任务来创建每个虚拟机，然后创建一个新的 play 来配置它们。

lookup 插件有用的场景如下：

+   将整个 Apache 配置目录复制到 `conf.d` 风格的目录中

+   使用环境变量来调整 playbook 的行为

+   从 DNS TXT 记录中获取配置

+   将命令输出提取到变量中

# 存储结果

几乎每个模块都会输出一些内容，甚至 `debug` 模块也是如此。大多数情况下，唯一使用的变量是名为 `changed` 的变量。`changed` 变量帮助 Ansible 决定是否运行处理程序，以及以什么颜色打印输出。然而，如果你愿意，你可以存储返回的值并在 playbook 中稍后使用它们。在这个示例中，我们查看 `/tmp` 目录的模式，并创建一个新的目录 `/tmp/subtmp`，其模式与之相同，如下所示。

```
---
- name: Using register
  hosts: ansibletest
  user: root
  tasks:
    - name: Get /tmp info
      file:
        dest: /tmp
        state: directory
      register: tmp

    - name: Set mode on /var/tmp
      file:
        dest: /tmp/subtmp
        mode: "{{ tmp.mode }}"
        state: directory
```

一些模块，如前面例子中的 `file` 模块，可以配置为仅提供信息。通过将其与 register 功能结合，你可以创建能够检查环境并计算如何继续的 playbook。

### 注意

将 register 功能与 `set_fact` 模块结合使用，可以对从模块返回的数据进行处理。这让你能够计算值并对这些值进行数据处理，使得你的 playbook 比以往更加智能和灵活。

Register 允许你基于已有的模块为主机创建自定义的事实。这在许多不同情况下都非常有用：

+   获取远程目录中的文件列表，并使用 fetch 下载所有文件

+   当前一个任务发生变化时，在处理程序运行之前执行任务

+   获取远程主机的 SSH 密钥内容，并构建一个 `known_hosts` 文件

# 处理数据

Ansible 使用 Jinja2 过滤器来允许你以基本模板无法实现的方式转换数据。当 playbook 中可用的数据格式不符合需求，或者需要在与模块或模板配合使用之前进行复杂的处理时，我们会使用过滤器。过滤器可以在我们通常使用变量的地方使用，例如在模板中、作为模块的参数或在条件语句中。使用过滤器时，只需提供变量名称、一个管道符号，然后是过滤器名称。我们还可以使用多个过滤器名称，过滤器通过管道符分隔，按从左到右的顺序应用。以下是一个示例，确保所有用户都使用小写的用户名创建：

```
---
- name: Create user accounts
  hosts: all
  vars:
    users:
  tasks:
    - name: Create accounts
      user: name={{ item|lower }} state=present
      with_items:
        - Fred
        - John
        - DanielH
```

以下是一些你可能会发现有用的常见过滤器：

| 过滤器 | 描述 |
| --- | --- |
| `min` | 当参数为列表时，返回列表中最小的值。 |
| `max` | 当参数为列表时，返回列表中最大的值。 |
| `random` | 当参数为列表时，从列表中随机选择一个项目。 |
| `changed` | 当用于使用 register 关键字创建的变量时，如果任务更改了任何内容则返回`true`；否则返回`false`。 |
| `failed` | 当用于使用 register 关键字创建的变量时，如果任务失败则返回`true`；否则返回`false`。 |
| `skipped` | 当用于使用 register 关键字创建的变量时，如果任务未更改任何内容则返回`true`；否则返回`false`。 |
| `default(X)` | 如果变量不存在，则使用 X 的值。 |
| `unique` | 当参数为列表时，返回一个没有重复项的列表。 |
| `b64decode` | 将变量中的 base64 编码字符串转换为其二进制表示。这在与 slurp 模块一起使用时非常有用，因为它将其数据作为 base64 编码的字符串返回。 |
| `replace(X, Y)` | 返回字符串的副本，其中任何出现的`X`都被`Y`替换。 |
| `join(X)` | 当变量为列表时，返回所有条目用`X`分隔的字符串。 |

# 调试 playbook

有几种方法可以调试 playbook。Ansible 包括详细模式和专门用于调试的`debug`模块。您还可以使用诸如`fetch`和`get_url`等模块来获取帮助。这些调试技术还可以用于检查模块在学习如何使用它们时的行为。

## debug 模块

使用`debug`模块非常简单。它接受两个可选参数，`msg`和`fail.msg`，用于设置模块将打印的消息和`fail`，如果设置为`yes`，则表示对 Ansible 的失败，这将导致它停止处理该主机的 playbook。我们在跳过模块部分早些时候使用了此模块，以便在操作系统不被识别时退出 playbook。

在以下示例中，我们将展示如何使用`debug`模块列出机器上所有可用的接口：

```
---
- name: Demonstrate the debug module
  hosts: ansibletest
  user: root
  vars:
    hostcount: 5
  tasks:
    - name: Print interface
      debug:
        msg: "{{ item }}"
      with_items: ansible_interfaces
```

前述代码输出如下：

```
PLAY [Demonstrate the debug module] *********************************

GATHERING FACTS *****************************************************
ok: [ansibletest]

TASK: [Print interface] *********************************************
ok: [ansibletest] => (item=lo) => {"item": "lo", "msg": "lo"}
ok: [ansibletest] => (item=eth0) => {"item": "eth0", "msg": "eth0"}

PLAY RECAP **********************************************************
ansibletest                : ok=2    changed=0    unreachable=0    failed=0
```

正如您所看到的，使用`debug`模块来查看 play 期间变量的当前值非常容易。

## 详细模式

调试的另一个选项是详细选项。在使用详细选项运行 Ansible 时，它会打印每个模块运行后返回的所有值。如果您使用前面介绍的`register`关键字，这尤其有用。要在详细模式下运行`ansible-playbook`，只需在命令行中添加`--verbose`如下：

```
ansible-playbook --verbose playbook.yml

```

## 检查模式

除了详细模式外，Ansible 还包括检查模式和差异模式。你可以通过在命令行中添加 `--check` 来使用检查模式，使用 `--diff` 来启用差异模式。检查模式指示 Ansible 在不对远程系统进行实际更改的情况下遍历剧本。这使你能够获取 Ansible 计划对配置系统进行的更改列表。

### 注意

这里需要注意的是，Ansible 的检查模式并不完美。任何未实现检查功能的模块将被跳过。此外，如果跳过的模块提供了更多的变量，或者这些变量依赖于模块实际更改某些内容（例如文件大小），那么这些变量将无法使用。使用 `command` 或 `shell` 模块时，这是一个明显的局限性。

差异模式显示由 `template` 模块所做的更改。这一局限性是因为 `template` 文件只适用于文本文件。如果你尝试为 `copy` 模块中的二进制文件提供差异，结果几乎是无法读取的。差异模式还可以与检查模式一起使用，展示由于处于检查模式而未做的计划更改。

## 暂停模块

另一种技巧是使用 `pause` 模块，在你检查正在运行的配置机器时暂停剧本。这样，你可以看到模块在当前剧本位置所做的更改，然后继续观察剧本的其余部分。

# 摘要

在本章中，我们探讨了编写剧本的更多高级细节。现在，你应该能够使用委派、循环、条件语句和事实注册等功能，使你的剧本更容易维护和编辑。我们还研究了如何访问其他主机的信息、为模块配置环境以及从外部来源收集数据。最后，我们介绍了一些调试剧本的技巧，帮助解决剧本执行异常的问题。

在下一章，我们将讨论如何在更大规模的环境中使用 Ansible。内容将包括提高执行时间较长的剧本性能的方法。我们还将介绍一些能够使剧本更易于维护的特性，特别是通过目的将其拆分成多个部分。
