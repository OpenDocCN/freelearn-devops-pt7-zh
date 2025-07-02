# 第四章  处理复杂的部署

你一定在想，为什么这章会取这样的名字。原因是到目前为止，我们还没有进入可以在生产环境中部署 playbook 的阶段，尤其是在复杂的情况下。复杂情况包括那些你需要与多个（几百或几千）台机器进行交互的情况，其中每组机器可能依赖于另一组或多组机器。这些组之间可能在所有或部分事务上相互依赖，执行安全的复杂数据备份和主从复制。此外，还有一些 Ansible 的有趣且非常吸引人的特性，我们还没有探讨。在本章中，我们将通过示例来讲解这些特性。我们的目标是，在本章结束时，你应该清楚如何编写可以从配置管理角度部署到生产环境中的 playbook。接下来的章节将基于我们学到的内容，进一步增强使用 Ansible 的体验。

为此，我们将从一个可能在某些情况下派上用场的功能开始：`local_action`。

# 使用 `local_action` 功能

Ansible 的 `local_action` 功能非常强大，尤其是在我们考虑编排时。这个功能允许你在运行 Ansible 的机器上本地执行某些任务。

考虑以下情况：

+   启动一台新机器或创建一个 JIRA 工单

+   管理你的指挥中心(s)，包括安装软件包和配置设置

+   调用负载均衡器 API 以从负载均衡器中禁用某个 Web 服务器条目

这些任务可以在运行 `ansible-playbook` 命令的同一机器上执行，而无需登录到远程主机并运行这些命令。

让我们看一个例子。假设你想在本地系统上运行一个 shell 模块，那里正在运行你的 Ansible playbook。在这种情况下，`local_action` 选项发挥了作用。如果你将模块名称和模块参数传递给 `local_action`，它将在本地运行该模块。让我们看看这个选项如何与 `shell` 模块一起工作。考虑以下代码，显示了 `local_action` 选项的输出：

```
    --- 
    - hosts: database 
      remote_user: ansible 
      tasks: 
      - name: Count processes running on the remote system 
        shell: ps | wc -l 
        register: remote_processes_number 
      - name: Print remote running processes 
        debug: 
          msg: '{{ remote_processes_number.stdout }}' 
      - name: Count processes running on the local system 
        local_action: shell ps | wc -l 
        register: local_processes_number 
      - name: Print local running processes 
        debug: 
          msg: '{{ local_processes_number.stdout }}' 

```

现在我们可以将其保存为 `local_action.yaml` 并用以下命令运行：

```
ansible-playbook -i hosts local_action.yaml

```

我们收到以下结果：

```
PLAY [database] **************************************************
TASK [setup] *****************************************************
ok: [db01.fale.io] 
TASK [Count processes running on the remote system] **************
changed: [db01.fale.io] 
TASK [Print remote running processes] ****************************
ok: [db01.fale.io] => {
 "msg": "7"
} 
TASK [Count processes running on the local system] ***************
changed: [db01.fale.io -> localhost] 
TASK [Print local running processes] *****************************
ok: [db01.fale.io] => {
 "msg": "11"
} 
PLAY RECAP *******************************************************
db01.fale.io      : ok=5    changed=2    unreachable=0    failed=0 

```

如你所见，提供的两个命令给出了不同的数字，因为它们在不同的主机上执行。你可以使用`local_action`运行任何模块，Ansible 会确保该模块在运行 `ansible-playbook` 命令的机器上本地执行。另一个你可以（并且应该！）尝试的简单示例是运行两个任务：

+   远程机器（如上例中的 `db01`）上的 `uname`

+   本地机器上的 `uname`，但启用了 `local_action`

这将进一步明确 `local_action` 的概念。

Ansible 提供了另一种将某些操作委托给特定（或不同）机器的方法：`delegate_to` 系统。

# 委托任务

有时，你可能希望在不同的系统上执行操作。例如，当你在应用服务器节点或本地主机上部署某些内容时，可能需要在数据库节点上执行某个操作。为此，你只需要在任务中添加 `delegate_to: HOST` 属性，它将会在适当的节点上运行。让我们重新调整前面的示例来实现这一点：

```
    --- 
    - hosts: database 
      remote_user: ansible 
      tasks: 
      - name: Count processes running on the remote system 
        shell: ps | wc -l 
        register: remote_processes_number 
      - name: Print remote running processes 
        debug: 
          msg: '{{ remote_processes_number.stdout }}' 
      - name: Count processes running on the local system 
        shell: ps | wc -l 
        delegate_to: localhost 
        register: local_processes_number 
      - name: Print local running processes 
        debug: 
          msg: '{{ local_processes_number.stdout }}' 

```

将其保存为 `delegate_to.yaml`，然后可以使用以下命令运行：

```
ansible-playbook -i hosts delegate_to.yaml

```

我们将得到与之前示例相同的输出：

```
PLAY [database] **************************************************
TASK [setup] *****************************************************
ok: [db01.fale.io] 
TASK [Count processes running on the remote system] **************
changed: [db01.fale.io] 
TASK [Print remote running processes] ****************************
ok: [db01.fale.io] => {
 "msg": "7"
} 
TASK [Count processes running on the local system] ***************
changed: [db01.fale.io -> localhost] 
TASK [Print local running processes] *****************************
ok: [db01.fale.io] => {
 "msg": "11"
} 
PLAY RECAP *******************************************************
db01.fale.io      : ok=5    changed=2    unreachable=0    failed=0 

```

# 使用条件语句

到目前为止，我们只看到了剧本如何工作以及任务如何执行。我们还看到 Ansible 按顺序执行所有任务。然而，这对你在编写包含数十个任务的高级剧本时并没有帮助，因为你可能只希望执行其中的一部分任务。例如，假设你有一个剧本，它将在远程主机上安装 Apache HTTPd 服务器。现在，Apache HTTPd 服务器在基于 Debian 的操作系统中的包名不同，叫做 `apache2`；而在基于 Red-Hat 的操作系统中，包名是 `httpd`。

在剧本中拥有两个任务，一个用于 `httpd` 包（适用于基于 Red-Hat 的系统），另一个用于 `apache2` 包（适用于基于 Debian 的系统），这将导致 Ansible 安装两个包，而执行会失败，因为如果你在基于 Red-Hat 的操作系统上安装，`apache2` 包将不可用。为了解决这类问题，Ansible 提供了条件语句，帮助在指定条件满足时才执行任务。在这种情况下，我们做的操作类似于以下伪代码：

```
    If os = "redhat" 
      Install httpd 
    Else if os = "debian" 
      Install apache2 
    End 

```

在 Red-Hat 系统上安装 `httpd` 时，我们首先检查远程系统是否运行的是基于 Red-Hat 的操作系统，如果是，我们就安装 `httpd` 包；否则，跳过此任务。为了不浪费时间，让我们直接进入一个名为 `conditional_httpd.yaml` 的示例剧本，其内容如下：

```
    --- 
    - hosts: webserver 
      remote_user: ansible 
      tasks: 
      - name: Print the ansible_os_family value 
        debug: 
          msg: '{{ ansible_os_family }}' 
      - name: Ensure the httpd package is updated 
        yum: 
          name: httpd 
          state: latest 
        become: True 
        when: ansible_os_family == 'RedHat' 
      - name: Ensure the apache2 package is updated 
        apt: 
          name: apache2 
          state: latest 
        become: True 
        when: ansible_os_family == 'Debian' 

```

使用以下命令运行：

```
ansible-playbook -i hosts conditional_httpd.yaml

```

这是结果：

```
PLAY [webserver] *************************************************
TASK [setup] *****************************************************
ok: [ws03.fale.io]
ok: [ws01.fale.io]
ok: [ws02.fale.io] 
TASK [Print the ansible_os_family value] *************************
ok: [ws01.fale.io] => {
 "msg": "RedHat"
}
ok: [ws02.fale.io] => {
 "msg": "RedHat"
}
ok: [ws03.fale.io] => {
 "msg": "Debian"
} 
TASK [Ensure the httpd package is updated] ***********************
skipping: [ws03.fale.io]
changed: [ws01.fale.io]
changed: [ws02.fale.io] 
TASK [Ensure the apache2 package is updated] *********************
skipping: [ws02.fale.io]
skipping: [ws01.fale.io]
changed: [ws03.fale.io] 
PLAY RECAP *******************************************************
ws01.fale.io      : ok=3    changed=1    unreachable=0    failed=0 
ws02.fale.io      : ok=3    changed=1    unreachable=0    failed=0 
ws03.fale.io      : ok=3    changed=1    unreachable=0    failed=0 

```

如你所见，我为这个例子创建了一个新的服务器（`ws03`），它是基于 Debian 的。正如预期的那样，`httpd` 包在两个 CentOS 节点上被安装，而 `apache2` 包则在 Debian 节点上安装。

### 提示：

Ansible 只区分少数几个操作系统家族（AIX、Alpine、Altlinux、Archlinux、Darwin、Debian、FreeBSD、Gentoo、HP-UX、Mandrake、Red Hat、Slackware、Solaris 和 Suse，本文写作时如此），因此 CentOS 机器的 `ansible_os_family` 值为 `'RedHat'`。

同样，你也可以匹配不同的条件。Ansible 支持等于（`==`）、不等于（`!=`）、大于（`>`）、小于（`<`）、大于或等于（`>=`）以及小于或等于（`<=`）等条件。

到目前为止，我们看到的运算符会匹配变量的整个内容，但如果你只想检查一个特定的字符或字符串是否出现在变量中呢？为了执行这些检查，Ansible 提供了 `in` 和 `not` 运算符。你还可以使用 `AND` 和 `OR` 运算符匹配多个条件。`AND` 运算符会确保所有条件都匹配后才执行任务，而 `OR` 运算符则确保至少有一个条件匹配。例如，你可以使用 `foo >= 0` 和 `foo <= 5`。

## 布尔条件语句

除了字符串匹配外，你还可以检查一个变量是否为 `True`。这种类型的验证在你想要检查变量是否被赋值时非常有用。你甚至可以根据变量的布尔值来执行任务。

例如，假设我们将以下代码放在名为 `crontab_backup.yaml` 的文件中：

```
    --- 
    - hosts: all 
      remote_user: ansible 
      vars: 
        backup: True 
      tasks: 
      - name: Copy the crontab in tmp if the backup variable is true 
        copy: 
          src: /etc/crontab 
          dest: /tmp/crontab 
          remote_src: True 
        when: backup 

```

如果我们使用以下命令执行：

```
ansible-playbook -i hosts crontab_backup.yaml

```

我们将获得以下结果：

```
PLAY [all] *******************************************************
TASK [setup] *****************************************************
ok: [db01.fale.io]
ok: [ws02.fale.io]
ok: [ws01.fale.io] 
TASK [Copy the crontab in tmp if the backup variable is true] ****
changed: [ws02.fale.io]
changed: [db01.fale.io]
changed: [ws01.fale.io] 
PLAY RECAP *******************************************************
db01.fale.io      : ok=2    changed=1    unreachable=0    failed=0 
ws01.fale.io      : ok=2    changed=1    unreachable=0    failed=0 
ws02.fale.io      : ok=2    changed=1    unreachable=0    failed=0 

```

但如果我们稍微修改命令为：

```
ansible-playbook -i hosts crontab_backup.yaml --extra-vars="backup=False"

```

我们将收到以下输出：

```
PLAY [all] *******************************************************
TASK [setup] *****************************************************
ok: [db01.fale.io]
ok: [ws02.fale.io]
ok: [ws01.fale.io] 
TASK [Copy the crontab in tmp if the backup variable is true] ****
skipping: [ws01.fale.io]
skipping: [ws02.fale.io]
skipping: [db01.fale.io] 
PLAY RECAP *******************************************************
db01.fale.io      : ok=1    changed=0    unreachable=0    failed=0 
ws01.fale.io      : ok=1    changed=0    unreachable=0    failed=0 
ws02.fale.io      : ok=1    changed=0    unreachable=0    failed=0 

```

如你所见，在第一种情况下，操作已执行，而在第二种情况下，它被跳过了。我们本可以使用配置文件、`host` 变量或 `group` 变量来覆盖备份值。

### 提示

如果以这种方式检查并且变量未设置，Ansible 将假设它为 `False`。

## 检查变量是否已设置

有时你可能需要在命令中使用一个变量。每次这样做时，你都需要确保变量已*设置*。这是因为如果使用*未设置*的变量调用某些命令，可能会导致灾难性后果（例如：如果执行 `rm -rf $VAR/*` 且 `$VAR` 未设置或为空，它将摧毁你的机器）。为此，Ansible 提供了一种检查变量是否已定义的方法。

我们可以通过以下方式改进前面的示例：

```
    --- 
    - hosts: all 
      remote_user: ansible 
      vars: 
        backup: True 
      tasks: 
      - name: Check if the backup_folder is set 
        fail: 
          msg: 'The backup_folder needs to be set' 
        when: backup_folder is not defined 
      - name: Copy the crontab in tmp if the backup variable is true 
        copy: 
          src: /etc/crontab 
          dest: '{{ backup_folder }}/crontab' 
          remote_src: True 
        when: backup 

```

如你所见，我们使用了 `fail` 模块，这可以让我们在 `backup_folder` 变量未设置时将 Ansible 剧本置于失败状态。

# 使用 include

`include` 功能有助于减少编写任务时的重复性。这也使我们能够通过使用**不要重复自己**（**DRY**）原则，将可重用代码包含在独立的任务中，从而拥有更小的剧本。

要触发另一个文件的包含，你需要将以下内容放在任务对象下：

```
- include: FILENAME.yaml

```

你还可以将一些变量传递给包含的文件。为此，我们可以以以下方式指定它们：

```
- include: FILENAME.yaml variable1="value1" variable2="value2"

```

除了传递变量，你还可以使用条件语句，仅在满足特定条件时包含文件，例如，仅在机器运行 Red Hat 系列操作系统时，使用以下代码包含 `redhat.yaml` 文件：

```
 - name: Include the file only for Red Hat OSes
    include: redhat.yaml
    when: ansible_os_family == "RedHat"

```

# 与处理程序一起使用

在许多情况下，你会有一个任务或一组任务，改变远程机器上的某些资源，这些任务需要触发一个事件才能生效。例如，当你更改服务配置时，需要重新启动或重新加载该服务。在 Ansible 中，你可以使用 `notify` 动作触发这个事件。

每个处理任务在收到通知后将在 playbook 执行的最后运行。例如，你多次更改了 HTTPd 服务器的配置，并且你希望重启 HTTPd 服务以应用这些更改。现在，每次做出配置更改时重启 HTTPd 并不是一个好做法；即使配置没有更改，也不应该随便重启服务器。为了解决这种情况，你可以通知 Ansible 在每次配置更改时重启 HTTPd 服务，但 Ansible 会确保无论你多少次通知它重启 HTTPd，它只会在所有其他任务完成后执行该任务一次。我们来稍微修改一下之前章节中创建的 `webserver.yaml` 文件，修改方式如下：

```
--- 
- hosts: webserver 
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
  - name: Ensure HTTPd configuration is updated 
    copy: 
      src: website.conf 
      dest: /etc/httpd/conf.d 
    become: True 
    notify: Restart HTTPd 
  handlers: 
  - name: Restart HTTPd 
    service: 
      name: httpd 
      state: restarted 
    become: True 

```

使用以下命令运行此脚本：

```
ansible-playbook -i hosts webserver.yaml

```

我们将得到以下输出：

```
PLAY [webserver] *************************************************
TASK [setup] *****************************************************
ok: [ws01.fale.io]
ok: [ws02.fale.io] 
TASK [Ensure the HTTPd package is installed] *********************
ok: [ws02.fale.io]
ok: [ws01.fale.io] 
TASK [Ensure the HTTPd service is enabled and running] ***********
ok: [ws02.fale.io]
ok: [ws01.fale.io] 
TASK [Ensure HTTP can pass the firewall] *************************
ok: [ws02.fale.io]
ok: [ws01.fale.io] 
TASK [Ensure HTTPd configuration is updated] *********************
changed: [ws02.fale.io]
changed: [ws01.fale.io] 
RUNNING HANDLER [Restart HTTPd] **********************************
changed: [ws02.fale.io]
changed: [ws01.fale.io] 
PLAY RECAP *******************************************************
ws01.fale.io      : ok=6    changed=2    unreachable=0    failed=0 
ws02.fale.io      : ok=6    changed=2    unreachable=0    failed=0 

```

在这种情况下，处理器是通过配置文件的更改被触发的。但是，如果我们第二次运行它，配置不会改变，因此我们将得到以下结果：

```
PLAY [webserver] *************************************************
TASK [setup] *****************************************************
ok: [ws01.fale.io]
ok: [ws02.fale.io] 
TASK [Ensure the HTTPd package is installed] *********************
ok: [ws02.fale.io]
ok: [ws01.fale.io] 
TASK [Ensure the HTTPd service is enabled and running] ***********
ok: [ws02.fale.io]
ok: [ws01.fale.io] 
TASK [Ensure HTTP can pass the firewall] *************************
ok: [ws01.fale.io]
ok: [ws02.fale.io] 
TASK [Ensure HTTPd configuration is updated] *********************
ok: [ws02.fale.io]
ok: [ws01.fale.io] 
PLAY RECAP *******************************************************
ws01.fale.io      : ok=5    changed=0    unreachable=0    failed=0 
ws02.fale.io      : ok=5    changed=0    unreachable=0    failed=0

```

### 注意

当使用处理器时，即使它们在 playbook 执行过程中被多次调用，它们也只会触发一次。默认情况下，处理器会在 playbook 执行结束时执行，但你可以使用 `meta` 任务并加上 `flush_handlers` 选项来强制它们按需运行，例如：`- meta: flush_handlers`

# 使用角色

我们已经看到了如何自动化一些简单的任务，但到目前为止，我们所看到的并不能解决所有问题。这是因为 playbook 非常擅长执行操作，但不适合配置大量机器，因为它们很快就会变得凌乱。为了解决这个问题，Ansible 引入了角色（roles）。

我对角色的定义是：一组 playbook、模板、文件或变量，用于实现特定目标。例如，我们可以有一个数据库角色和一个 Web 服务器角色，这样这些配置就能保持清晰的分离。

在开始查看角色的内部内容之前，我们先来谈谈项目的组织结构。

## 项目组织结构

在过去几年里，我为多个组织的多个 Ansible 仓库工作过，其中许多都非常混乱。为了确保你的仓库易于管理，我将给你一个我总是使用的模板。

首先，我总是会在 `root` 文件夹中创建三个文件：

+   `ansible.cfg`：一个小的配置文件，用来告诉 Ansible 在我们的文件夹结构中哪里可以找到相关文件

+   `hosts`：我们在之前章节中已经看到的 hosts 文件

+   `master.yaml`：一个用于对齐整个基础架构的 playbook

除了那三个文件，我还创建了两个文件夹：

+   `playbooks`：这个文件夹将包含所有的 playbooks 以及一个名为 *groups* 的文件夹，用于群组管理。

+   `roles`：这个文件夹将包含我们需要的所有角色

为了澄清这一点，让我们使用 Linux 的 `tree` 命令，查看一个简单的 Web 应用的 Ansible 仓库结构，该应用需要 Web 服务器和数据库服务器：

```
 ansible.cfg
    hosts
    master.yaml
    playbooks
        firstrun.yaml
        groups
            database.yaml
            webserver.yaml
    roles
         common
         database
         webserver 

```

如你所见，我还添加了一个 `common` 角色。这对于放置每个服务器都必须执行的任务非常有用。通常，我在这个角色中配置 NTP、motd 和其他类似的服务，以及机器的主机名。

现在我们来看一下如何构建一个角色。

## 角色的结构

角色中的文件夹结构是标准的，你不能做太多改变。

角色中最重要的文件夹是 `tasks` 文件夹，因为它是唯一一个必需的文件夹。它必须包含一个 `main.yaml` 文件，其中列出了需要执行的任务。角色中常见的其他文件夹包括 templates 和 files。第一个用于存放 **template task** 使用的模板，而第二个用于存放 **copy task** 使用的文件。

## 将你的 playbooks 转换为一个完整的 Ansible 项目

让我们来看看如何将我们用来设置 Web 基础设施的三个 playbooks（`common_tasks.yaml`、`firstrun.yaml` 和 `webserver.yaml`）转换为适合这种文件组织方式。我们还需要记住，我们在这些角色中使用了两个文件（`index.html.j2` 和 `motd`），所以我们也要正确地放置这些文件。

首先，我们将创建在前一段中看到的文件夹结构。

最容易移植的 playbook 是 `firstrun.yaml`，因为我们只需要将它复制到 playbooks 文件夹中。这个 playbook 将保持为 playbook，因为它是一组只需要在每台服务器上执行一次的操作。

我们现在进入 `common_tasks.yaml` playbook，它需要一些修改，以符合角色的范式。

### 将 playbook 转换为角色

我们需要做的第一件事是创建 `roles/common/tasks` 和 `roles/common/templates` 文件夹。在第一个文件夹中，我们将添加以下 `main.yaml` 文件：

```
    --- 
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

如你所见，这与我们的 `common_tasks.yaml` playbooks 非常相似。事实上，只有两个区别：

+   `hosts`、`remote_user` 和 `tasks`（第 2、3、4 行）已经被删除

+   文件其余部分的缩进已相应修正

在这个角色中，我们使用了模板任务来创建一个包含机器 IP 和其他有用信息的 `motd` 文件。因此，我们需要创建 `roles/common/templates` 文件夹，并将 `motd` 模板放在其中。

到此为止，我们的公共任务将具有以下结构：

```
    common/ 
        tasks 
            main.yaml 
        templates 
            motd 

```

现在，我们需要指导 Ansible 在哪些机器上执行`common`角色中指定的所有任务。为此，我们应该查看`playbooks/groups`目录。在这个目录中，为每组逻辑上相似的机器（即执行相同操作的机器）创建一个文件是非常方便的。在我们的案例中，分别是数据库和 Web 服务器。

所以，让我们在`playbooks/groups`中创建一个`database.yaml`文件，内容如下：

```
    --- 
    - hosts: database 
      user: ansible 
      roles: 
      - common 

```

在同一个文件夹中创建一个`webserver.yaml`文件，内容如下：

```
    --- 
    - hosts: webserver 
      user: ansible 
      roles: 
      - common 

```

如你所见，这些文件指定了我们希望操作的主机组、在这些主机上使用的远程用户以及我们希望执行的角色。

### 辅助文件

当我们在上一章创建`hosts`文件时，我们注意到它帮助简化了我们的命令行。所以，现在我们开始将之前在 Ansible 仓库的`root`文件夹中使用的 hosts 文件复制过来。到目前为止，我们总是在命令行中指定这个文件的路径。如果我们创建一个`ansible.cfg`文件来告诉 Ansible 我们`hosts`文件的位置，那么就不再需要这样做了。因此，让我们在 Ansible 仓库的根目录下创建一个`ansible.cfg`文件，内容如下：

```
    [defaults] 
    hostfile = hosts 
    host_key_checking = False 
    roles_path = roles 

```

在这个文件中，我们除了之前提到的`hostfile`变量外，还指定了另外两个变量，它们分别是`host_key_checking`和`roles_path`。

`host_key_checking`标志有助于避免要求验证远程系统的 SSH 密钥。这不建议在生产环境中使用，因为建议在此类环境中使用公共密钥传播系统，但在测试环境中非常有用，因为它将帮助你减少 Ansible 等待用户输入的时间。

`roles_path`用于告诉 Ansible 在哪里查找我们剧本的角色。

我通常会添加一个额外的文件，叫做`master.yaml`。我发现这个文件非常有用，因为你通常需要将基础设施与 Ansible 代码保持一致。为了通过一个命令完成这件事，你需要一个文件来运行`playbooks/groups`中的所有文件。因此，让我们在 Ansible 仓库的根文件夹中创建一个`master.yaml`文件，内容如下：

```
    --- 
    - include: playbooks/groups/database.yaml 
    - include: playbooks/groups/webserver.yaml 

```

此时，我们可以执行以下操作：

```
ansible-playbook master.yaml

```

结果如下所示：

```
PLAY [database] **************************************************
TASK [setup] *****************************************************
ok: [db01.fale.io] 
TASK [common : Ensure EPEL is enabled] ***************************
ok: [db01.fale.io] 
TASK [common : Ensure libselinux-python is present] **************
ok: [db01.fale.io] 
TASK [common : Ensure libsemanage-python is present] *************
ok: [db01.fale.io] 
TASK [common : Ensure we have last version of every package] *****
changed: [db01.fale.io] 
TASK [common : Ensure NTP is installed] **************************
ok: [db01.fale.io] 
TASK [common : Ensure the timezone is set to UTC] ****************
ok: [db01.fale.io] 
TASK [common : Ensure the NTP service is running and enabled] ****
ok: [db01.fale.io] 
TASK [common : Ensure FirewallD is installed] ********************
ok: [db01.fale.io] 
TASK [common : Ensure FirewallD is running] **********************
ok: [db01.fale.io] 
TASK [common : Ensure SSH can pass the firewall] *****************
ok: [db01.fale.io] 
TASK [common : Ensure the MOTD file is present and updated] ******
ok: [db01.fale.io] 
TASK [common : Ensure the hostname is the same of the inventory] *
ok: [db01.fale.io] 
PLAY [webserver] *************************************************
TASK [setup] *****************************************************
ok: [ws01.fale.io]
ok: [ws02.fale.io] 
TASK [common : Ensure EPEL is enabled] ***************************
ok: [ws01.fale.io]
ok: [ws02.fale.io] 
TASK [common : Ensure libselinux-python is present] **************
ok: [ws02.fale.io]
ok: [ws01.fale.io] 
TASK [common : Ensure libsemanage-python is present] *************
ok: [ws01.fale.io]
ok: [ws02.fale.io] 
TASK [common : Ensure we have last version of every package] *****
changed: [ws01.fale.io]
changed: [ws02.fale.io] 
TASK [common : Ensure NTP is installed] **************************
ok: [ws01.fale.io]
ok: [ws02.fale.io] 
TASK [common : Ensure the timezone is set to UTC] ****************
ok: [ws01.fale.io]
ok: [ws02.fale.io] 
TASK [common : Ensure the NTP service is running and enabled] ****
ok: [ws02.fale.io]
ok: [ws01.fale.io] 
TASK [common : Ensure FirewallD is installed] ********************
ok: [ws02.fale.io]
ok: [ws01.fale.io] 
TASK [common : Ensure FirewallD is running] **********************
ok: [ws01.fale.io]
ok: [ws02.fale.io] 
TASK [common : Ensure SSH can pass the firewall] *****************
ok: [ws01.fale.io]
ok: [ws02.fale.io] 
TASK [common : Ensure the MOTD file is present and updated] ******
ok: [ws01.fale.io]
ok: [ws02.fale.io] 
TASK [common : Ensure the hostname is the same of the inventory] *
ok: [ws02.fale.io]
ok: [ws01.fale.io] 
PLAY RECAP *******************************************************
db01.fale.io      : ok=13   changed=1    unreachable=0    failed=0 
ws01.fale.io      : ok=13   changed=1    unreachable=0    failed=0 
ws02.fale.io      : ok=13   changed=1    unreachable=0    failed=0

```

如你所见，`common`角色中列出的动作首先在`database`组的节点上执行，然后在`webserver`组的节点上执行。

### 转换`webserver`角色

当我们将`common`剧本转换为`common`角色时，我们可以对`webserver`角色做同样的事情。

在角色中，我们需要有一个`webserver`文件夹，并在其中创建`tasks`子文件夹。在这个文件夹里，我们需要放置一个`main.yaml`文件，其中包含从剧本中复制过来的`tasks`，它应该如下所示：

```
    --- 
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
    - name: Ensure HTTPd configuration is updated 
      copy: 
        src: website.conf 
        dest: /etc/httpd/conf.d 
      become: True 
      notify: Restart HTTPd 
    - name: Ensure the website is present and updated 
      template: 
        src: index.html.j2 
        dest: /var/www/html/index.html 
        owner: root 
        group: root 
        mode: 0644 
      become: True 

```

在这个角色中，我们使用了多个任务，这些任务需要额外的资源才能正常工作，更具体地说，我们需要：

+   将`website.conf`文件放入`roles/webserver/files`中

+   将`index.html.j2`模板放入`roles/webserver/templates`中

+   创建`Restart HTTPd`处理程序

前两个应该是相当直接的。事实上，第一个是一个空文件（我们还没有在其中放入任何内容，因为默认配置已经足够适合我们的使用），而`index.html.j2`文件应包含以下内容：

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

### 角色中的处理程序

完成此角色所需的最后一件事是创建`Restart HTTPd`通知的处理程序。为此，我们需要在`roles/webserver/handlers`中创建一个`main.yaml`文件，内容如下：

```
    --- 
    - name: Restart HTTPd 
      service: 
        name: httpd 
        state: restarted 
      become: True 

```

如你所注意到的，这与我们在剧本中使用的处理程序非常相似，只是文件位置和缩进不同。

为了使我们的角色适用，我们仍然需要做的唯一事情是添加条目到`playbooks/groups/webserver.yaml`文件中，这样 Ansible 就知道`webserver`组中的服务器应该应用`webserver`角色以及公共角色。我们的`playbooks/groups/webserver.yaml`文件需要如下所示：

```
    --- 
    - hosts: webserver 
      user: ansible 
      roles: 
      - common 
      - webserver 

```

我们现在可以再次执行`master.yaml`来将`webserver`角色应用到相关的服务器，但我们也可以只执行`playbooks/groups/webserver.yaml`，因为我们刚刚做的更改仅与这组服务器相关。为此，我们运行：

```
    ansible-playbook playbooks/groups/webserver.yaml

```

我们应该收到类似以下的输出：

```
PLAY [webserver] *************************************************
TASK [setup] *****************************************************
ok: [ws02.fale.io]
ok: [ws01.fale.io] 
TASK [common : Ensure EPEL is enabled] ***************************
ok: [ws01.fale.io]
ok: [ws02.fale.io] 
TASK [common : Ensure libselinux-python is present] **************
ok: [ws01.fale.io]
ok: [ws02.fale.io] 
TASK [common : Ensure libsemanage-python is present] *************
ok: [ws01.fale.io]
ok: [ws02.fale.io] 
TASK [common : Ensure we have last version of every package] *****
ok: [ws01.fale.io]
ok: [ws02.fale.io] 
TASK [common : Ensure NTP is installed] **************************
ok: [ws02.fale.io]
ok: [ws01.fale.io] 
TASK [common : Ensure the timezone is set to UTC] ****************
ok: [ws01.fale.io]
ok: [ws02.fale.io] 
TASK [common : Ensure the NTP service is running and enabled] ****
ok: [ws01.fale.io]
ok: [ws02.fale.io] 
TASK [common : Ensure FirewallD is installed] ********************
ok: [ws01.fale.io]
ok: [ws02.fale.io] 
TASK [common : Ensure FirewallD is running] **********************
ok: [ws02.fale.io]
ok: [ws01.fale.io] 
TASK [common : Ensure SSH can pass the firewall] *****************
ok: [ws01.fale.io]
ok: [ws02.fale.io] 
TASK [common : Ensure the MOTD file is present and updated] ******
ok: [ws01.fale.io]
ok: [ws02.fale.io] 
TASK [common : Ensure the hostname is the same of the inventory] *
ok: [ws02.fale.io]
ok: [ws01.fale.io] 
TASK [webserver : Ensure the HTTPd package is installed] *********
ok: [ws01.fale.io]
ok: [ws02.fale.io] 
TASK [webserver : Ensure the HTTPd service is enabled and running]
ok: [ws01.fale.io]
ok: [ws02.fale.io] 
TASK [webserver : Ensure HTTP can pass the firewall] *************
ok: [ws02.fale.io]
ok: [ws01.fale.io] 
TASK [webserver : Ensure HTTPd configuration is updated] *********
ok: [ws01.fale.io]
ok: [ws02.fale.io] 
TASK [webserver : Ensure the website is present and updated] *****
changed: [ws01.fale.io]
changed: [ws02.fale.io] 
PLAY RECAP *******************************************************
ws01.fale.io      : ok=18   changed=1    unreachable=0    failed=0 
ws02.fale.io      : ok=18   changed=1    unreachable=0    failed=0

```

如你所见，`common`和`webserver`角色已经应用于`webserver`节点。

应用所有与特定节点相关的角色非常重要，而不仅仅是你更改过的角色，因为通常情况下，当一个或多个节点出现问题，而同一组中的其他节点没有问题时，问题通常是因为组内某些角色的应用不均衡。只有将所有相关角色应用于一个组，才能确保该组中节点的平等性。

# 执行策略

在 Ansible 2 之前，每个任务都需要在每台机器上执行（并完成）后，Ansible 才会向所有机器发布新任务。这意味着如果你在一百台机器上执行任务，而其中一台机器性能较差，所有机器都会以性能较差的机器的速度运行。

在 Ansible 2 中，执行策略已被模块化，因此你现在可以选择自己偏好的执行策略。你还可以编写自定义的执行策略，但这超出了本书的范围。目前（在 Ansible 2.1 中）只有三种执行策略：**线性**、**串行**和**自由**：

+   **线性执行**：此策略与 Ansible 2 之前的行为完全相同。这是默认的执行策略。

+   **串行执行**：此策略将选取一部分主机（默认是五台），并对这些主机执行所有任务，完成后再移至下一个子集并从头开始。此类执行策略可以帮助你在有限数量的主机上工作，以便始终保持一些主机可供用户使用。如果你需要这种类型的部署，你将需要在主机前放置负载均衡器，并实时通知负载均衡器哪些节点处于维护状态。

+   **自由执行**：此策略将在每个主机完成前一个任务后立即将新任务分配给该主机。这将允许更快的主机在较慢的节点之前完成剧本。如果你选择此执行策略，你必须记住某些任务可能需要所有节点完成前一个任务（例如，集群数据库要求所有数据库节点都安装并运行数据库），在这种情况下，它们可能会失败。

# 任务块

在 Ansible 2.0 中，任务块已经被引入。任务块允许你以逻辑方式对任务进行分组，并且它们也有助于更好的错误处理。你可以添加到标准任务的多数属性，你也可以将它们添加到任务块中。如果机器是 CentOS，你可能需要执行一个 yum 任务来安装 NTPd 并启用该服务。为此，可以使用以下代码：

```
    tasks:
    - block:
       - name: Ensure NTPd is present
       yum:
         name: ntpd
         state: present
       - name: Ensure NTPd is running
       service:
         name: ntpd
         state: started
       enabled: True
     when: ansible_distribution == 'CentOS'
```

如你所见，`when`子句已应用于任务块，因此只有在`when`子句为真时，块中的所有任务才会执行。

# Ansible 模板 - Jinja 过滤器

我们在第二章中已经看到，模板允许你根据动态数据（如`host`和`group`变量）动态地完成你的剧本并将文件放置在服务器上。在本节中，我们将继续前进，看看 Jinja2 过滤器如何与 Ansible 一起工作。

Jinja2 过滤器是简单的 Python 函数，它们接收一些参数，处理这些参数并返回结果。例如，考虑以下命令：

```
{{ myvar | filter }}

```

在前面的例子中，`myvar`是一个变量；Ansible 将`myvar`作为参数传递给 Jinja2 过滤器。然后，Jinja2 过滤器将处理它并返回结果数据。Jinja2 过滤器甚至接受如下的额外参数：

```
{{ myvar | filter(2) }}

```

在这个例子中，Ansible 现在将传递两个参数，即`myvar`和`2`。同样，你可以通过逗号分隔来传递多个参数给过滤器。

Ansible 支持各种各样的 Jinja2 过滤器，我们将看到一些在编写剧本时可能需要使用的重要 Jinja2 过滤器。

## 使用过滤器格式化数据

Ansible 支持 Jinja2 过滤器将数据格式化为 JSON 或 YAML。你将一个字典变量传递给该过滤器，它将把数据格式化为 JSON 或 YAML。例如，考虑以下命令行：

```
{{ users | to_nice_json }}

```

在前面的示例中，`users`是变量，`to_nice_json`是 Jinja2 过滤器。正如我们之前看到的，Ansible 会将`users`作为参数传递给 Jinja2 过滤器`to_nice_json`。同样，你也可以使用以下命令将数据格式化为 YAML：

```
{{ users | to_nice_yaml }}

```

## 使用带有条件语句的过滤器

你可以使用 Jinja2 过滤器结合条件语句来检查任务的状态是失败、已更改、成功还是跳过。让我们开始在`playbooks`文件夹中创建一个文件，内容如下：

```
    --- 
    - hosts: webserver 
      remote_user: ansible 
      tasks: 
      - name: Checking HTTPd service status 
        service: 
          name: httpd 
          state: running 
        register: httpd_result 
        ignore_errors: true 
      - debug: 
          msg: Previous task failed 
        when: httpd_result|failed 

```

在前面的示例中，我们首先检查了`httpd`服务是否正在运行，并将该模块的输出存储在`httpd_result`变量中。然后，我们使用 Jinja2 过滤器`httpd_result|failed`检查前一个任务是否失败。如果`when`条件失败，即前一个任务通过，Ansible 将跳过此任务。同样，你可以使用`changed`、`success`或`skipped`过滤器。

我们现在可以检查之前的`playbook`是否按预期执行，执行它的方法是：

```
ansible-playbook playbooks/http_status.yaml

```

我已经在`ws01.fale.io`服务器上停止了 HTTPd 服务，命令是`systemctl stop httpd`，执行它将给我以下结果：

```
PLAY [webserver] *************************************************
TASK [setup] *****************************************************
ok: [ws02.fale.io]
ok: [ws01.fale.io] 
TASK [Checking HTTPd service status] *****************************
ok: [ws02.fale.io]
fatal: [ws01.fale.io]: FAILED! => {"changed": false, "failed": true, "msg": "Failed to start httpd.service: Interactive authentication required.\n"}
...ignoring 
TASK [debug] *****************************************************
skipping: [ws02.fale.io]
ok: [ws01.fale.io] => {
 "msg": "Previous task failed"
} 
PLAY RECAP *******************************************************
ws01.fale.io      : ok=3    changed=0    unreachable=0    failed=0 
ws02.fale.io      : ok=2    changed=0    unreachable=0    failed=0 

```

## 默认未定义的变量

我们在前面的章节中看到，在使用变量之前，检查变量是否已定义总是明智的做法。我们可以为变量设置一个`default`值，这样如果变量未定义，Ansible 将使用该值，而不会失败。要做到这一点，我们使用：

```
{{ backup_disk | default("/dev/sdf") }}

```

此过滤器不会将`default`值赋给变量；它只会将`default`值传递给当前正在使用它的任务。在关闭这一部分之前，让我们看几个 Jinja 过滤器的更多示例：

使用随机数过滤器：要从列表中查找一个随机的数字、字符或字符串，你可以使用随机过滤器：

+   执行此命令以从列表中获取一个随机字符：

```
{{['a', 'b', 'c', 'd'] | random}}

```

+   执行此命令以获取 0 到 100 之间的随机数字：

```
{{100 | random}}

```

+   执行此命令以获取 10 到 50 之间步长为 10 的随机数：

```
{{50 | random(10)}}

```

+   执行此命令以获取 20 到 50 之间步长为 10 的随机数字：

```
{{50 | random(20, 10)}}

```

使用过滤器将列表连接到字符串：Jinja2 过滤器允许你使用`join`过滤器将列表连接到字符串。此过滤器需要一个分隔符作为额外的参数。如果你没有指定分隔符，过滤器将会将列表的所有元素组合在一起，没有任何分隔符。考虑以下示例：

```
{{["This", "is", "a", "string"] | join(" ")}}

```

+   上述过滤器将输出一个字符串。你可以指定任何分隔符，而不是空格。

+   使用过滤器编码或解码数据：你可以使用过滤器来编码或解码数据，如下所示：

+   使用`b64encode`过滤器将数据编码为`base64`：

```
 {{variable | b64encode}}

```

+   使用`b64decode`过滤器解码一个已编码的`base64`字符串：

```
      {{"aGFoYWhhaGE=" | b64decode}}

```

# 安全管理

本章的最后一节讲的是安全管理。如果您告诉您的系统管理员您想引入一个新功能或工具，他们首先会问的一个问题是：“您的工具有哪些安全功能？”我们将在本节中从 Ansible 的角度尝试回答这些问题。让我们更详细地看一下它们。

## 使用 Ansible vault

Ansible vault 是 Ansible 的一个令人兴奋的功能，首次出现在 Ansible 版本 1.5 中。它允许您将加密的密码作为源代码的一部分。推荐的做法是不将密码（以及任何其他敏感信息，如私钥、SSL 证书等）以明文形式存储在代码库中，因为任何检出您代码库的人都可以查看您的密码。Ansible vault 可以通过加密和解密内容来帮助您保护机密信息。

Ansible vault 支持交互模式，在该模式下它会要求您输入密码，或者支持非交互模式，您需要指定包含密码的文件，Ansible vault 将直接读取该文件。

在这些示例中，我们将使用密码`ansible`，所以让我们开始创建一个名为`.password`的隐藏文件，文件内容为`ansible`。为此，让我们执行：

```
echo 'ansible' > .password

```

我们现在可以在交互模式和非交互模式下创建`ansible-vault`。如果我们想在交互模式下进行，我们需要执行：

```
ansible-vault create secret.yaml

```

Ansible 将要求我们输入 vault 密码并确认。然后，它会打开默认的文本编辑器（在我的情况下是**vi**），以明文方式添加内容。我使用的密码是`ansible`，文本内容是“这是一个受密码保护的文件”。我们现在可以保存并关闭编辑器，检查`ansible-vault`是否加密了我们的内容，事实上，如果我们运行：

```
cat secret.yaml

```

这将输出以下内容：

```
$ANSIBLE_VAULT;1.1;AES256
66346431333933663461383331393763666538373163336536353335646532323135383630646366
3432353561393533623764323961666639326132323331370a636363613032616664333039356565
64643735626162646166313861366532323161646137333634333336393062303461343638333737
6534326135326430390a643739336461616334313833313363343030666662653864353138666233
38386266383866353836373036303339383962363362333364346432613062363830316330653866
6431343764386132663066303761346532643632633432643861

```

同样，我们可以使用`vault-password-file=VAULT_PASSWORD_FILE`选项调用`ansible-vault`命令，来指定我们的`.password`文件。例如，我们可以使用以下命令编辑我们的`secret.yaml`文件：

```
ansible-vault --vault-password-file=.password edit secret.yaml

```

这将打开您的默认文本编辑器，您可以像编辑普通文件一样修改文件。当您保存文件时，Ansible vault 将在保存之前执行加密，确保您的内容的机密性。

有时您需要查看文件的内容，但又不想在文本编辑器中打开它，所以通常使用`cat`命令。Ansible vault 有一个类似的功能，叫做`view`，您可以运行：

```
ansible-vault --vault-password-file=.password view secret.yaml

```

Ansible vault 允许您解密一个文件，将其加密内容替换为明文内容。要执行此操作，您可以运行：

```
ansible-vault --vault-password-file=.password decrypt secret.yaml

```

此时，我们可以在`secret.yaml`文件上运行`cat`命令，结果如下：

```
This is a password protected file

```

Ansible vault 还允许你加密已经存在的文件。如果你希望在一个受信任的机器上（例如你自己的本地机器）以明文形式开发所有文件，以提高效率，然后再加密所有敏感文件，这特别有用。为此，你可以执行：

```
ansible-vault --vault-password-file=.password encrypt secret.yaml

```

现在你可以检查 `secret.yaml` 文件是否已经再次加密。

Ansible vault 的最后一个选项非常重要，因为它是一个 `rekey` 功能。此功能允许你通过一个命令更改加密密钥。你可以通过两个命令执行相同的操作（用 **旧密钥** 解密 `secret.yaml` 文件，然后用 **新密钥** 加密），但是能够在单个步骤中完成这项操作有重大优势，因为在整个过程中，文件的明文形式不会存储在磁盘上。为此，我们需要一个包含新密码的文件（在我们这里，文件名为 `.newpassword`，内容为字符串 `ansible2`），然后需要执行以下命令：

```
ansible-vault --vault-password-file=.password --new-vault-password-file=.newpassword rekey secret.yaml

```

现在我们可以使用 `cat` 命令查看 `secret.yaml` 文件，输出结果如下：

```
$ANSIBLE_VAULT;1.1;AES256
63313864643434663939333132333537336362313133616430376463613833353366326662303832
6431316131613033343266373137356166383564326234300a386236633635333939333234643435
64353932383930613934343730386635333030373663313631646462613566313362313363393135
3935613661373263330a316634333536653461356535383662376464656466623536363537386462
31636637346538636161616632313866366365666361633138666134303433316665376237326162
3638653738383830323430313161336465323264613634323434

```

这与我们之前的方法非常不同。

## 保险库和 playbooks

你也可以在 `ansible-playbook` 中使用保险库。你需要使用类似以下的命令即时解密文件：

```
$ ansible-playbook site.yml --vault-password-file .password

```

还有另一个选项，允许你使用脚本解密文件，然后脚本可以查找其他来源并解密文件。这也可以作为一个有用的选项，以提供更多的安全性。然而，确保 `get_password.py` 脚本具有可执行权限：

```
$ ansible-playbook site.yml --vault-password-file ~/.get_password.py

```

在结束本章之前，我想稍微谈谈密码文件。此文件需要存在于你执行 playbooks 的机器上，并且存放在具有适当权限的位置，以便执行 playbook 的用户可以读取。你可以在启动时创建 `.password` 文件。`.password` 文件名中的 `.` 字符是为了确保文件在查找时默认是隐藏的。这并不直接是一种安全措施，但可以帮助缓解攻击者不清楚自己在寻找什么的情况。

`.password` 文件的内容应该是一个密码或密钥，确保只有有权限运行 Ansible playbooks 的人员可以访问。

最后，确保你不会加密每一个可用的文件！Ansible vault 应该仅用于需要安全保护的重要信息。

### 注意

每次保存加密文件时，不管是否已应用更改，文件都会重新加密，因此加密内容会发生变化。这会导致你的 SCM 工具标记该文件为已修改。

## 加密用户密码

Ansible Vault 负责处理已检查的密码，并帮助你在运行 Ansible 剧本或命令时管理它们。然而，在运行 Ansible play 时，有时你可能需要用户输入密码。你还想确保这些密码不会出现在全面的 Ansible 日志（默认位置：`/var/log/ansible.log`）或`stdout`中。

Ansible 使用`Passlib`，它是一个 Python 的密码哈希库，用于处理提示密码的加密。你可以使用`Passlib`支持的以下任意算法：

+   `des_crypt`: DES Crypt

+   `bsdi_crypt`: BSDi Crypt

+   `bigcrypt`: BigCrypt

+   `crypt16`: Crypt16

+   `md5_crypt`: MD5 Crypt

+   `bcrypt`: BCrypt

+   `sha1_crypt`: SHA-1 Crypt

+   `sun_md5_crypt`: Sun MD5 Crypt

+   `sha256_crypt`: SHA-256 Crypt

+   `sha512_crypt`: SHA-512 Crypt

+   `apr_md5_crypt`: Apache 的 MD5-Crypt 变种

+   `phpass`: PHPass 的便携式哈希

+   `pbkdf2_digest`: 通用 PBKDF2 哈希

+   `cta_pbkdf2_sha1`: Cryptacular 的 PBKDF2 哈希

+   `dlitz_pbkdf2_sha1`: Dwayne Litzenberger 的 PBKDF2 哈希

+   `scram`: SCRAM 哈希

+   `bsd_nthash`: FreeBSD 的 MCF 兼容`nthash`编码

现在让我们来看一下如何使用变量提示进行加密：

```
    vars_prompt:
    - name: ssh_password 
      prompt: Enter ssh_password 
      private: True 
      encryption: md5_crypt 
      confirm: True 
      salt_size: 7 

```

在前面的代码片段中，`vars_prompt`用于提示用户输入一些数据。`vars_prompt`不是任务，而是与`tasks:`同一级别的另一个部分。

`name`模块表示 Ansible 将存储用户密码的实际变量名称，如下命令所示：

```
name: ssh_password

```

我们使用`prompt`工具提示用户输入密码，如下所示：

```
prompt: Enter ssh password

```

我们明确要求 Ansible 通过使用`private`模块从`stdout`中隐藏密码；这就像任何 Unix 系统上的密码提示一样。`private`模块的访问方式如下：

```
private: True

```

我们在这里使用`md5_crypt`算法，并且盐值大小为`7`：

```
encrypt: md5_crypt
salt_size: 7

```

此外，Ansible 将提示用户输入两次密码并进行比较：

```
confirm: True

```

## 隐藏密码

默认情况下，Ansible 会过滤包含`login_password`键、`password`键以及`user:pass`格式的输出。例如，如果你通过`login_password`或`password`键在模块中传递密码，Ansible 将用`VALUE_HIDDEN`替换你的密码。现在我们来看一下如何使用`password`键隐藏密码：

```
- name: Running a script
 shell: script.sh
 password: my_password

```

在前面的`shell`任务中，我们使用`password`键传递密码。这将允许 Ansible 隐藏密码，不显示在`stdout`和其日志文件中。

现在，当你在*详细*模式下运行上述任务时，你应该看不到`mypass`密码；相反，Ansible 将用`VALUE_HIDDEN`替代它，如下所示：

```
REMOTE_MODULE command script.sh password=VALUE_HIDDEN #USE_SHELL

```

### 注意

Ansible 会保护你声明为密码的字符串，即使它们在其他上下文中使用。例如，如果你有另一个变量包含字符串`my_password`，如果你将其打印出来，`HIDDEN_VALUE`将会出现，即使该特定变量没有被声明为密码。

## 使用 no_log

Ansible 只会在你使用特定的密钥集时才会隐藏你的密码。然而，这并不总是如此；此外，你可能还希望隐藏一些其他的机密数据。Ansible 的 `no_log` 功能将隐藏整个任务，不会将其记录到 `syslog` 文件中。它仍然会在 `stdout` 上打印你的任务，并将其记录到其他 Ansible 日志文件中。

### 注意

在编写本书时，Ansible 不支持使用 `no_log` 隐藏来自 `stdout` 的任务。

另一种防止 Ansible 记录日志的方法是在 `ansible.cfg` 文件的 `[defaults]` 部分设置 `log_path` 为 `/dev/null`，这样所有日志都会被保存到 `/dev/null`，因此会丢失。

现在让我们来看一下如何使用 `no_log` 隐藏整个任务，如下所示：

```
    - name: Running a script
      shell: script.sh
        password: my_password
      no_log: True

```

通过将 `no_log`: True 传递给任务，Ansible 将防止整个任务被记录到 syslog 中。

# 总结

在本章中，我们学习了大量的 Ansible 功能。我们从 `local_actions` 开始，用于在一台机器上执行操作，然后转向委托，在第三台机器上执行任务。接着，我们介绍了条件语句和包含功能，使 playbook 更加灵活。我们了解了角色及其如何帮助你保持系统的一致性，并学习了如何正确组织 Ansible 仓库，最大限度地利用 Ansible 和 Git。随后，我们讨论了执行策略和 Jinja 过滤器，以实现更灵活的执行。

本章最后，我们介绍了 Ansible vault 和其他许多提高 Ansible 执行安全性的技巧。

在下一章，我们将学习如何使用 Ansible 创建基础设施，更具体地说，如何使用云服务提供商 AWS 和 DigitalOcean 来完成这一任务。
