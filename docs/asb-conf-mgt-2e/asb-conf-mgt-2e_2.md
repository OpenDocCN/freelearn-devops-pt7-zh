# 第二章。简单的剧本

Ansible 可以用作命令行工具来进行小改变。但是，它的真正力量在于其脚本能力。在设置机器时，我们几乎总是需要同时做多件事情。Ansible 使用名为**剧本**的概念来实现此目的。使用剧本，我们可以一次执行多个操作，并跨多个系统执行。它们提供了一种协调部署、确保一致配置或简单执行常见任务的方法。

剧本以**YAML**表示，大部分情况下，Ansible 使用标准的 YAML 解析器。这意味着我们在编写时可以利用 YAML 提供给我们的所有功能。例如，在剧本中我们可以像在 YAML 中一样使用相同的注释系统。许多剧本的行也可以用 YAML 数据类型编写和表示。更多信息请参阅 [`www.yaml.org/`](http://www.yaml.org/)。

剧本还打开了许多机会。它们允许我们在一个命令到另一个命令之间传递状态。例如，我们可以在一台机器上获取文件的内容，将其注册为变量，然后在另一台机器上使用该值。这使我们能够创建使用 Ansible 命令单独无法实现的复杂部署机制。此外，由于每个模块都试图是幂等的，我们应该能够多次运行剧本，只有在需要时才会进行更改。

执行剧本的命令是 `ansible-playbook`。它接受类似 Ansible 命令行工具的参数。例如，`-k`（`--ask-pass`）和`-K`（`--ask-sudo`）使 Ansible 分别提示 SSH 和 sudo 密码；`-u` 可用于设置用于 SSH 的用户。但是，这些选项也可以在剧本自身的目标部分中设置。例如，要使用名为 `example-play.yml` 的播放，我们可以使用以下命令：

```
$ ansible-playbook example-play.yml

```

Ansible 剧本由一个或多个播放组成。一个播放包括三个部分：

+   **目标部分** 定义了将在其上运行播放的主机以及如何运行。这是我们设置 SSH 用户名和其他 SSH 相关设置的地方。

+   **变量部分** 定义了在运行时将提供给剧本的变量。

+   **任务部分** 按我们希望 Ansible 运行的模块顺序列出所有模块。

我们可以在单个 YAML 文件中包含尽可能多的剧本。YAML 文件以`---`开头，包含许多键值和列表。在 YAML 中，行缩进用于指示变量的嵌套给解析器，这也使文件更易于阅读。

完整的 Ansible 播放示例如下代码片段所示：

```
---
- hosts: webservers
  user: root
  vars:
    apache_version: 2.6
    motd_warning: 'WARNING: Use by ACME Employees ONLY'
    testserver: yes
  tasks:
    - name: setup a MOTD
      copy:
        dest: /etc/motd
        content: "{{ motd_warning }}"
```

在接下来的几节中，我们将检查每个部分，并详细解释它们的工作方式。

# 目标部分

目标部分看起来像以下的代码片段：

```
- hosts: webservers
  user: root
```

这是一个非常简单的版本，但在大多数情况下可能足够使用。每个播放都存在于一个列表中。根据 YAML 语法，该行必须以破折号开始。播放将运行的主机必须在`hosts`的值中设置。该值使用与在前一章中讨论的 Ansible 命令行中选择主机时相同的语法。Ansible 的主机模式匹配功能也在前一章中讨论过。在下一行中，用户告诉 Ansible 剧本以哪个用户身份连接到机器。

我们可以在此部分提供的其他行如下：

| 名称 | 描述 |
| --- | --- |
| `sudo` | 如果希望 Ansible 在连接到播放中的机器后使用`sudo`成为 root 用户，请将其设置为`yes`。 |
| `user` | 这定义了最初连接到机器的用户名，然后如果已配置`sudo`，将运行`sudo`。 |
| `sudo_user` | 这是 Ansible 尝试使用`sudo`切换为的用户。例如，如果我们将`sudo`设置为`yes`，并将`user`设置为`daniel`，将`sudo_user`设置为`kate`，则会导致 Ansible 在登录后使用`sudo`从`daniel`切换到`kate`。如果你在交互式 SSH 会话中进行此操作，我们可以在以`daniel`登录时使用`sudo -u kate`。 |
| `connection` | 这允许我们告诉 Ansible 使用何种传输方式连接远程主机。对于远程主机，我们通常使用`ssh`或`paramiko`。不过，我们也可以使用`local`来避免在运行`localhost`时产生连接开销。通常，我们会在此使用`local`、`winrm`或`ssh`。 |
| `gather_facts` | 除非我们告诉它不要，否则 Ansible 将自动在远程主机上运行 setup 模块。如果我们不需要来自 setup 模块的变量，可以现在设置此项并节省一些时间。 |

# 变量部分

在这里，我们可以定义适用于所有机器的整个播放的变量。如果在命令行中未提供变量，Ansible 还可以提示用户输入变量。这使得我们可以轻松维护播放，避免在多个地方更改相同的内容。它还允许我们将整个配置存储在播放的顶部，在这里我们可以轻松阅读和修改它，而不必担心播放的其他部分。

该部分的变量可以被机器事实（由模块设置的事实）覆盖，但它们本身会覆盖我们在清单中设置的事实。因此，它们用于定义我们可能在以后通过模块收集的默认值，但不能用于保持清单变量的默认值，因为它们会覆盖这些默认值。

变量声明发生在`vars`部分，类似目标部分中的值，并包含一个 YAML 字典或列表。示例如下所示的代码片段：

```
vars:
  apache_version: 2.6
  motd_warning: 'WARNING: Use by ACME Employees ONLY'
  testserver: yes
```

变量也可以通过提供要加载的变量文件列表从外部 YAML 文件加载。这通过类似的方式使用 `vars_files` 指令完成。然后简单地提供另一个包含自己字典的 YAML 文件的名称。这意味着，我们可以将变量存储并分别分发，而不是将它们存储在同一个文件中，这样我们就可以与他人共享我们的 playbook。

使用 `vars_files`，这些文件在我们的 playbook 中看起来类似于以下代码片段：

```
vars_files:
  conf/country-AU.yml
  conf/datacenter-SYD.yml
  conf/cluster-mysql.yml
```

在之前的示例中，Ansible 会在相对于 playbook 路径的 `conf` 文件夹中查找 `country-AU.yml`、`datacenter-SYD.yml` 和 `cluster-mysql.yml`。每个 YAML 文件看起来类似于以下代码片段：

```
---
ntp: ntp1.au.example.com
TZ: Australia/Sydney
```

最后，我们可以让 Ansible 以交互方式向用户询问每个变量。这在我们有一些不希望用于自动化的变量时非常有用，而是需要人工输入。一个有用的例子是当需要提示用户输入解密 HTTPS 服务器的密钥密码时。

我们可以通过以下代码片段指示 Ansible 提示输入变量：

```
vars_prompt:
  - name: https_passphrase
    prompt: Key Passphrase
    private: yes
```

在之前的示例中，`https_passphrase` 是输入的数据存储的位置。系统会提示用户输入 `Key Passphrase`，由于 `private` 设置为 `yes`，用户输入的值将不会在屏幕上显示。

我们可以使用 `{{ variablename }}` 来引用变量、事实和库存变量。我们甚至可以通过点表示法引用复杂的变量，例如字典。例如，一个名为 `httpd` 的变量，其键为 `maxclients`，可以通过 `{{ httpd.maxclients }}` 进行访问。这对于来自 setup 模块的事实也同样适用。例如，我们可以使用 `{{ ansible_eth0.ipv4.address }}` 获取名为 `eth0` 的网络接口的 IPv4 地址。

在变量部分设置的变量不会在同一个 playbook 中的不同 plays 之间保留。然而，通过 setup 模块收集的事实或通过 `set_fact` 设置的事实会保留。这意味着，如果我们在同一台机器或先前 play 中的一部分机器上运行第二个 play，我们可以在目标部分将 `gather_facts` 设置为 `false`。`setup` 模块有时需要较长时间运行，因此这可以显著加快 plays 的速度，特别是在 serial 设置为较小值的 plays 中。

# 任务部分

任务部分是每个 play 的最后一个部分。它包含我们希望 Ansible 按顺序执行的操作列表。我们可以以几种不同的方式表示每个模块的参数。建议尽可能坚持使用一种方式，只有在需要时才使用其他方式。这样可以使我们的 playbooks 更容易阅读和维护。以下代码片段展示了任务部分的三种风格：

```
tasks:
  - name: install apache
    action: yum name=httpd state=installed

  - name: configure apache
    copy: src=files/httpd.conf dest=/etc/httpd/conf/httpd.conf

  - name: restart apache
    service:
      name: httpd
      state: restarted
```

在这里，我们看到使用三种不同的语法样式来安装、配置并启动 Apache Web 服务器的方式，所展示的是在 CentOS 机器上的表现。第一个任务展示了如何使用原始语法安装 Apache，这需要我们将模块作为 `action` 键中的第一个关键字调用。第二个任务使用第二种任务样式将 Apache 配置文件复制到位。在这种样式中，模块名称取代 `action` 关键字，其值直接成为模块的参数。最后，第三个任务展示了如何使用服务模块重启 Apache。在这种样式中，我们像往常一样将模块名称用作键，但我们将参数提供为 YAML 字典。当我们向单个模块提供大量参数，或模块需要复杂形式的参数时（如云形成模块），这种样式会特别有用。随着越来越多的模块需要复杂的参数，后一种样式正迅速成为编写 playbook 的首选方式。在本书中，为了节省示例空间并避免行换行，我们将使用这种样式。

请注意，任务名称不是必需的。然而，它们有助于良好的文档记录，并允许我们在需要时引用每个任务。尤其是在处理程序部分，这将变得非常有用。当 playbook 执行时，任务名称也会输出到控制台，用户可以看到正在发生什么。如果我们没有提供名称，Ansible 将使用任务或处理程序的操作行。

### 注意

与其他配置管理工具不同，Ansible 不提供完整的依赖系统。这既是一个祝福也是一个诅咒；拥有完整的依赖系统，我们可能会陷入一个困境，即始终不确定哪些更改会应用到特定机器上。然而，Ansible 确保我们的更改会按照编写的顺序执行。因此，如果一个模块依赖于另一个在其之前执行的模块，只需在 playbook 中将一个模块放在另一个之前即可。

# 处理程序部分

处理程序部分在语法上与任务部分相同，并支持相同的调用模块格式。只有当调用该处理程序的任务记录了执行过程中某些内容发生了变化时，处理程序才会被调用。要触发处理程序，在任务中添加一个 `notify` 键，并将值设置为任务的名称。

在 Ansible 完成任务列表执行后，如果之前触发过的处理程序（handlers）会被运行。它们的执行顺序是根据在 handlers 部分中列出的顺序，尽管它们在任务部分可能被多次调用，但每个处理程序只会执行一次。这通常用于在升级和配置完守护进程后重启它们。以下的 playbook 示例演示了如何将 **ISC** **DHCP**（**动态主机配置协议**）服务器升级到最新版本，进行配置并设置为开机启动。如果此 playbook 在一个已运行最新版本 ISC DHCP 守护进程且配置文件未更改的服务器上运行，则处理程序不会被调用，DHCP 也不会重启。考虑以下代码示例：

```
---
- hosts: dhcp
  tasks:
  - name: update to latest DHCP
    yum
      name: dhcp
      state: latest
    notify: restart dhcp

  - name: copy the DHCP config
    copy:
      src: dhcp/dhcpd.conf
      dest: /etc/dhcp/dhcpd.conf
    notify: restart dhcp

  - name: start DHCP at boot
    service:
      name: dhcpd
      state: started
      enabled: yes

  handlers:
  - name: restart dhcp
    service:
      name: dhcpd
      state: restarted
```

每个处理程序只能是一个单独的模块，但我们可以从单个任务中通知多个处理程序。这允许我们从任务列表中的一个步骤触发多个处理程序。例如，如果我们刚刚检出了任何 Django 应用程序的更新版本，我们可以设置一个处理程序来迁移数据库、部署静态文件并重启 Apache。我们只需在 notify 动作中使用 YAML 列表即可实现。这可能类似于以下代码片段：

```
---
- hosts: qroud
  tasks:
  - name: checkout Qroud
    git:
      repo:git@github.com:smarthall/Qroud.git
      dest: /opt/apps/Qroud force=no
    notify:
      - migrate db
      - generate static
      - restart httpd

  handlers:
  - name: migrate db
    command: ./manage.py migrate –all
    args:
      chdir: /opt/apps/Qroud

  - name: generate static
    command: ./manage.py collectstatic -c –noinput
    args:
       chdir: /opt/apps/Qroud

  - name: restart httpd
    service:
      name: httpd
      state: restarted
```

我们可以看到 `git` 模块用于检出一些公共 GitHub 代码，如果这导致了任何更改，它将触发 `migrate db`、`generate static` 和 `restart httpd` 动作。

# playbook 模块

在 playbook 中使用模块与在命令行中使用它们略有不同。这主要是因为我们有很多来自先前模块和 `setup` 模块的事实（facts）。某些模块在 Ansible 命令行中无法使用，因为它们需要访问这些变量。其他模块则可以在命令行版本中使用，但在 playbook 中使用时，能够提供更强大的功能。

## 模板模块

一个经常使用的需要 Ansible 事实的模块示例是 `template` 模块。这个模块允许我们设计配置文件的框架，然后让 Ansible 在正确的位置插入值。为了实现这一点，Ansible 使用 Jinja2 模板语言。实际上，Jinja2 模板可以比这更复杂，包括条件判断、`for` 循环和宏等。以下是一个用于配置 BIND 的 Jinja2 配置模板示例：

```
# {{ ansible_managed }}
options {
  listen-on port 53 {
    127.0.0.1;
    {% for ip in ansible_all_ipv4_addresses %}
      {{ ip }};
    {% endfor %}
  };
  listen-on-v6 port 53 { ::1; };
  directory       "/var/named";
  dump-file       "/var/named/data/cache_dump.db";
  statistics-file "/var/named/data/named_stats.txt";
  memstatistics-file "/var/named/data/named_mem_stats.txt";
};

zone "." IN {
  type hint;
  file "named.ca";
};

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";

{# Variables for zone config #}
{% if 'authorativenames' in group_names %}
  {% set zone_type = 'master' %}
  {% set zone_dir = 'data' %}
{% else %}
  {% set zone_type = 'slave' %}
  {% set zone_dir = 'slaves' %}
{% endif %}

zone "internal.example.com" IN {
  type {{ zone_type }};
  file "{{ zone_dir }}/internal.example.com";
  {% if 'authorativenames' not in group_names %}
    masters { 192.168.2.2; };
  {% endif %}
};
```

按约定，Jinja2 模板的文件扩展名为 `.j2`，但这并非严格要求。现在让我们将这个示例拆解成各个部分。示例从以下代码行开始：

```
# {{ ansible_managed }}
```

这一行在文件顶部添加了注释，显示该文件来自哪个模板、主机、模板的修改时间以及所有者。将这些信息作为注释放在模板中是一个好习惯，它确保了人们知道如果想要永久修改它，应该编辑哪些部分。

接下来，在第五行有一个 `for` 循环：

```
    {% for ip in ansible_all_ipv4_addresses %}
      {{ ip }};
    {% endfor %}
```

`For` 循环会遍历列表中的所有元素，对于列表中的每一项都会执行一次循环。它可以选择将当前项赋值给我们选择的变量，这样我们就可以在循环内使用它。这个循环遍历的是 `ansible_all_ipv4_addresses` 中的所有值，这是 `setup` 模块提供的一个列表，包含了机器的所有 IPv4 地址。在 `for` 循环内，它会将每个地址添加到配置中，确保 BIND 会在该接口上监听。

模板中也可以添加注释，例如第 24 行的示例：

```
{# Variables for zone config #}
```

`{#` 和 `#}` 之间的内容会被 Jinja2 模板处理器忽略。这使我们能够在模板中添加注释，而这些注释不会出现在最终的文件中。如果我们正在做一些复杂的操作、在模板中设置变量，或者如果配置文件不允许注释，这特别有用。

接下来的几行是一个 `if` 语句的一部分，它设置了 `zone_type` 和 `zone_dir` 变量，以便在模板中后续使用：

```
{% if 'authorativenames' in group_names %}
  {% set zone_type = 'master' %}
  {% set zone_dir = 'data' %}
{% else %}
  {% set zone_type = 'slave' %}
  {% set zone_dir = 'slaves' %}
{% endif %}
```

`{% if %}` 和 `{% else %}` 之间的内容会在 `if` 标签中的语句为 `false` 时被忽略。在这里，我们检查 `authorativenames` 是否在适用于该主机的组名列表中。如果为 `true`，接下来的两行将分别设置两个自定义变量。`zone_type` 被设置为 master，`zone_dir` 被设置为 data。如果该主机不在 `authorativenames` 组中，`zone_type` 和 `zone_dir` 将分别设置为 `slave` 和 `slaves`。

最后，从第 33 行开始，我们提供了区域的实际配置：

```
zone "internal.example.com" IN {
  type {{ zone_type }};
  file "{{ zone_dir }}/internal.example.com";
  {% if zone_type == 'slave' %}
    masters { 192.168.2.2; };
  {% endif %}
};
```

我们将类型设置为之前创建的 `zone_type` 变量，并将位置设置为 `zone_dir`。最后，我们检查 zone 类型是否为 slave，如果是的话，我们会将它的主服务器配置为特定的 IP 地址。

为了让这个模板设置一个权威的 DNS 服务器，我们需要在清单文件中创建一个名为 `authorativenames` 的组，并在其下添加一些主机。如何做到这一点在第一章，*开始使用 Ansible* 中有讨论过。

我们可以简单地调用 `templates` 模块，机器的事实信息会被传递过来，包括该机器所在的组。这和调用其他任何模块一样简单。`template` 模块也接受与 `copy` 模块类似的参数，比如 owner、group 和 mode。例如，考虑以下代码：

```
---
- name: Setup BIND
  host: allnames
  tasks:
  - name: configure BIND
    template: src=templates/named.conf.j2 dest=/etc/named.conf owner=root group=named mode=0640
```

## set_fact 模块

`set_fact` 模块允许我们在 Ansible play 中为机器构建我们自己的事实。然后这些事实可以在模板中使用，或者作为 playbook 中的变量。事实就像是来自 `setup` 模块等模块的参数一样，在主机级别起作用。我们应该使用这个模块来避免将复杂的逻辑写入模板中。例如，如果我们要配置一个缓冲区来占用一定比例的内存，我们应该在 playbook 中进行计算。

以下示例展示了如何使用 `set_fact` 配置 MySQL 服务器，使其 InnoDB 缓冲区大小约为机器总内存的一半：

```
---
- name: Configure MySQL
  hosts: mysqlservers
  tasks:
  - name: install MySql
    yum:
      name: mysql-server
      state: installed

  - name: Calculate InnoDB buffer pool size
    set_fact:
      innodb_buffer_pool_size_mb="{{ansible_memtotal_mb/2}}"

  - name: Configure MySQL
    template:
      src: templates/my.cnf.j2
      dest: /etc/my.cnf
      owner: root
      group: root
      mode: 0644
    notify: restart mysql

  - name: Start MySQL
    service:
      name: mysqld
      state: started
      enabled: yes

  handlers:
  - name: restart mysql
    service:
      name: mysqld
      state: restarted
```

这里的第一个任务仅仅是通过 yum 安装 MySQL。第二个任务通过获取管理机的总内存，将其除以二，去掉任何非整数的余数，并将结果存入名为 `innodb_buffer_pool_size_mb` 的事实中。接下来的一行将一个模板加载到 `/etc/my.cnf` 中来配置 MySQL。最后，MySQL 被启动并设置为开机自启。还包括了一个处理程序，用于在 MySQL 配置更改时重新启动 MySQL。

模板只需要获取 `innodb_buffer_pool_size` 的值，并将其放入配置中。这意味着我们可以在需要将缓冲池大小设置为内存的五分之一或八分之一的地方重用相同的模板，并仅仅改变这些主机的剧本。在这种情况下，模板可能会像以下代码片段一样：

```
# {{ ansible_managed }}
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
# Settings user and group are ignored when systemd is used.
# If we need to run mysqld under a different user or group,
# customize our systemd unit file for mysqld according to the
# instructions in http://fedoraproject.org/wiki/Systemd

# Configure the buffer pool
innodb_buffer_pool_size = {{ innodb_buffer_pool_size_mb|default(128) }}M

[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
```

我们可以看到，在之前的模板中，我们只是将从剧本中获得的变量放入模板中。如果模板没有看到 `innodb_buffer_pool_size_mb` 事实，它将使用默认值 `128`。

## `pause` 模块

`pause` 模块会停止剧本的执行一段时间。我们可以配置它等待特定的时间，或者让它提示用户继续。虽然在从 Ansible 命令行使用时它基本上没有用，但在剧本内部使用时它非常方便。

通常，当我们希望用户提供确认继续，或者在某个特定点需要人工干预时，会使用 `pause` 模块。例如，如果我们刚刚将新版本的 Web 应用程序部署到服务器上，并且需要让用户手动检查确保一切正常，然后再配置它们接收生产流量，我们可以在这里加入暂停。它还可以用来警告用户可能存在的问题，并让他们选择是否继续。这将使 Ansible 打印出服务器名称并要求用户按 *Enter* 键继续。如果与目标部分的 serial 键一起使用，它会对 Ansible 正在运行的每组主机询问一次。通过这种方式，我们可以让用户在互动监控进度的同时，以自己的节奏运行部署。

不太有用的是，这个模块可以简单地等待指定的时间。通常它并不是很有用，因为我们通常不知道某个特定操作需要多长时间，猜测可能会导致灾难性的后果。我们不应该使用它来等待网络守护进程启动；相反，我们应该使用 `wait_for` 模块（在下一节中描述）来完成这个任务。以下剧本展示了先在用户交互模式下使用 `pause` 模块，然后在定时模式下使用的例子：

```
---
- hosts: localhost
  tasks:
  - name: wait on user input
    pause:
      prompt: "Warning! Press ENTER to continue or CTRL-C to quit."

  - name: timed wait
    pause:
      seconds: 30
```

## `wait_for` 模块

`wait_for`模块用于轮询特定的 TCP 端口，并且直到该端口接受远程连接时才会继续。轮询是从远程机器执行的。如果我们只提供端口，或者将 host 参数设置为`localhost`，轮询会尝试连接到管理机。我们可以利用`local_action`从控制机运行命令，并使用`ansible_hostname`变量作为主机参数，让它尝试从控制机连接到管理机。

这个模块对于需要较长时间才能启动的守护进程，或我们希望在后台运行的程序特别有用。Apache Tomcat 附带一个初始化脚本，在我们尝试启动它时会立即返回，Tomcat 则在后台启动。根据 Tomcat 配置加载的应用程序，它的启动时间可能从两秒到十分钟不等，直到完全启动并准备好接受连接。我们可以为应用程序的启动时间设置计时，并使用`pause`模块。然而，下次部署可能会需要更长或更短的时间，这会破坏我们的部署机制。使用`wait_for`模块，Ansible 能够识别 Tomcat 何时准备好接受连接。下面是一个完全实现这一点的 play：

```
---
- hosts: webapps
  tasks:
  - name: Install Tomcat
    yum:
      name: tomcat7
      state: installed

  - name: Start Tomcat
    service:
      name: tomcat7
      state: started

  - name: Wait for Tomcat to start
    wait_for:
      port: 8080
      state: started
```

完成此 play 后，Tomcat 应该已安装、启动并准备好接受请求。我们可以在这个例子中添加更多模块，依赖于 Tomcat 已经可用并在监听。

## assemble 模块

`assemble`模块将管理机上的多个文件合并，并保存到管理机上的另一个文件中。在 playbook 中，当我们有一个`config`文件，该文件不允许在其包含的内容中使用 includes 或 glob 时，这个功能非常有用。比如，对于 root 用户的`authorized_keys`文件，这个模块就很有用。下面的 play 将把一组 SSH 公钥发送到管理机，然后将它们组合在一起并放到 root 用户的主目录中：

```
---
- hosts: all
  tasks:
  - name: Make a Directory in /opt
    file:
      path: /opt/sshkeys
      state: directory
      owner: root
      group: root
      mode: 0700

  - name: Copy SSH keys over
    copy:
      src: "keys/{{ item }}.pub"
      dest: "/opt/sshkeys/{{ item }}.pub"
      owner: root
      group: root
      mode: 0600
    with_items:
      - dan
      - kate
      - mal

  - name: Make the root users SSH config directory
    file:
      path: /root/.ssh
      state: directory
      owner: root
      group: root
      mode: 0700

  - name: Build the authorized_keys file
    assemble:
      src: /opt/sshkeys
      dest: /root/.ssh/authorized_keys
      owner: root
      group: root
      mode: 0700
```

到目前为止，这一切应该看起来很熟悉。我们可以注意到在复制密钥的任务中有`with_items`键和`{{ items }}`变量。这些将在第三章中进一步解释，*高级 Playbook*，但现在我们需要知道的是，我们提供给`with_items`键的任何项都会替换成`{{ items }}`变量，类似于`for`循环的工作方式。这让我们能够轻松地一次性将多个文件复制到远程主机。

最后的任务展示了`assemble`模块的用法。我们将包含要合并文件的目录作为`src`参数传递给输出，然后将`dest`作为输出文件。它还接受许多与其他创建文件的模块相同的参数（`owner`、`group`和`mode`）。它还会按`ls -1`命令列出的顺序合并文件。这意味着我们可以像`udev`和`rc.d`那样使用相同的方法，给文件前加上数字，以确保它们按正确的顺序排列。

## add_host 模块

`add_host`模块是 playbooks 中最强大的模块之一。`add_host`让我们可以动态地在 play 中添加新机器。我们可以使用`uri`模块从我们的**配置管理数据库**（**CMDB**）中获取一个主机，然后将其添加到当前 play 中。该模块还将我们的主机添加到一个组中，如果该组尚不存在，则会动态创建该组。

该模块简单地接受`name`和`groups`参数，这些参数相当直观，并设置主机名和组。我们还可以传递额外的参数，这些参数的处理方式与 inventory 文件中的额外值处理方式相同。这意味着我们可以设置`ansible_ssh_user`、`ansible_ssh_port`等。

如果我们使用云服务提供商，如 RackSpace 或 Amazon EC2，Ansible 中有可用的模块可以让我们管理计算资源。如果我们在 inventory 中找不到它们，我们可能决定在 play 开始时创建机器。如果这样做，我们可以使用该模块将机器添加到 inventory 中，以便稍后进行配置。下面是一个使用 Google Compute 模块的示例：

```
---
- name: Create infrastructure
  hosts: localhost
  connection: local
  tasks:
    - name: Make sure the mailserver exists
      gce:
        image: centos-6
        name: mailserver
        tags: mail
        zone: us-central1-a
      register: mailserver
      when: '"mailserver" not in groups.all'

    - name: Add new machine to inventory
      add_hosts:
        name: mailserver
        ansible_ssh_host: "{{ mailserver.instance_data[0].public_ip }}"
        groups: tag_mail
      when: not mailserver|skipped
```

## `group_by`模块

除了在 play 中动态创建主机外，我们还可以创建组。`group_by`模块可以根据机器的事实信息来创建组，包括我们使用前面提到的`add_fact`模块自行设置的事实。`group_by`模块接受一个参数`key`，它接受机器将被添加到的组的名称。通过将其与变量的使用结合起来，我们可以使该模块根据主机的操作系统、虚拟化技术或任何其他可访问的事实将服务器添加到一个组中。然后，我们可以在任何后续 play 的目标部分或模板中使用该组。

所以，如果我们想要创建一个按操作系统对主机进行分组的组，我们将按照以下方式调用该模块：

```
---
- name: Create operating system group
  hosts: all
  tasks:
    - group_by: key=os_{{ ansible_distribution }}

- name: Run on CentOS hosts only
  hosts: os_CentOS
  tasks:
  - name: Install Apache
    yum: name=httpd state=latest

- name: Run on Ubuntu hosts only
  hosts: os_Ubuntu
  tasks:
  - name: Install Apache
    apt: pkg=apache2 state=latest
```

然后我们可以使用这些组来通过正确的打包工具安装软件包。在实际操作中，这通常用于避免 Ansible 在执行时输出大量的“跳过”消息。我们可以通过创建一个组来为需要执行操作的机器配置，而不是为每个需要跳过的任务添加`when`条件。然后，我们可以使用一个单独的 play 来分别配置这些机器。下面是一个在 Debian 和 RedHat 机器上安装 SSL 私钥的示例，而没有使用`when`条件：

```
---
- name: Catergorize hosts
  hosts: all
  tasks:
    - name: Gather hosts by OS
      group_by:
        key: "os_{{ ansible_os_family }}"

- name: Install keys on RedHat
  hosts: os_RedHat
  tasks:
    - name: Install SSL certificate
      copy:
        src: sslcert.pem
        dest: /etc/pki/tls/private/sslcert.pem

- name: Install keys on Debian
  hosts: os_Debian
  tasks:
    - name: Install SSL certificate
      copy:
        src: sslcert.pem
        dest: /etc/ssl/private/sslcert.pem
```

## slurp 模块

`slurp` 模块从远程系统获取文件，使用 base64 编码后返回结果。我们可以利用 register 关键字将文件内容放入 fact 中。当使用 `slurp` 模块获取文件时，应注意文件的大小。此模块会将整个文件加载到内存中，因此使用 `slurp` 处理大文件可能会消耗所有可用的内存，导致系统崩溃。文件还需要从被管理机器传输到控制机，对于大文件，这可能需要相当长的时间。

将该模块与 copy 模块结合使用，可以实现两台机器之间的文件复制。以下是该 playbook 的示例：

```
---
- name: Fetch a SSH key from a machine
  hosts: bastion01
  tasks:
    - name: Fetch key
      slurp:
        src: /root/.ssh/id_rsa.pub
      register: sshkey

- name: Copy the SSH key to all hosts
  hosts: all
  tasks:
    - name: Make directory for key
      file:
        state: directory
        path: /root/.ssh
        owner: root
        group: root
        mode: 0700

    - name: Install SSH key
      copy:
        contents: "{{ hostvars.bastion01.sshkey|b64decode }}"
        dest: /root/.ssh/authorized_keys
        owner: root
        group: root
        mode: 0600
```

### 注意

请注意，由于 `slurp` 模块使用 base64 编码数据，我们必须使用名为 `b64decode` 的 jinja2 过滤器解码数据，才能让 copy 模块使用它。过滤器将在 第三章 *高级 Playbooks* 中更详细地介绍。

# Windows playbook 模块

Windows 支持是 Ansible 的新特性，因此目前可用的模块不多。专门针对 Windows 的模块以 `win_` 开头命名。此外，还有一些既适用于 Windows 又适用于 Unix 系统的模块，例如我们之前提到的 `slurp` 模块。

在使用 Windows 模块时，应该特别小心路径字符串的引号。反斜杠在 YAML 中是一个重要字符，用于转义字符，在 Windows 路径中也表示目录。因此，YAML 可能会将路径的一部分误认为转义序列。为避免这种情况，我们在字符串中使用单引号。此外，如果路径本身是一个目录，我们应去掉结尾的反斜杠，以免 YAML 将字符串的结尾误认为转义序列。如果必须以反斜杠结束路径，应使用双反斜杠，第二个反斜杠会被忽略。以下是一些正确和错误字符串的示例：

```
# Correct
'C:\Users\Daniel\Documents\secrets.txt'
'C:\Program Files\Fancy Software Inc\Directory'
'D:\\' # \\ becomes \
# Incorrect
"C:\Users\Daniel\newcar.jpg" # \n becomes a new line
'C:\Users\Daniel\Documents\' # \' becomes '
```

# 云基础设施模块

基础设施模块不仅可以管理我们机器的配置，还可以创建这些机器本身。除此之外，我们还可以自动化它们周围的大部分基础设施。这可以作为 Amazon Cloud Formation 等服务的简单替代方案。

在创建我们希望在同一个 playbook 中后续 play 中进行管理的机器时，我们需要使用前面章节中讨论的 `add_hosts` 模块，将机器添加到内存中的库存中，以便它成为后续 play 的目标。我们还可能希望运行 `group_by` 模块，将它们按组排列，就像我们排列其他机器一样。还应该使用 `wait_for` 模块检查机器是否响应 SSH 连接，以便在尝试管理它之前确认连接是否正常。

云基础设施模块可能有点复杂，因此我们将展示如何设置和安装 Amazon 模块。有关如何配置其他模块的详细信息，请使用`ansible-doc`查看它们的文档。

## AWS 模块

AWS 模块的工作方式类似于大多数 AWS 工具的工作方式。这是因为它们使用 python **boto** 库，许多其他工具也使用该库，并且遵循 Amazon 发布的原始 AWS 工具的惯例。

最好以与安装 Ansible 相同的方式安装 boto。对于大多数使用场景，我们将在受管机器上运行模块，因此只需要在那里安装 boto 模块即可。我们可以通过以下方式安装 boto 库：

+   Centos/RHEL/Fedora: `yum install python-boto`

+   Ubuntu: `apt-get install python-boto`

+   Pip: `pip install boto`

然后我们需要设置正确的环境变量。最简单的方法是通过在本地机器上使用 localhost 连接运行模块。如果我们这样做，shell 中的变量将被传递并自动对 Ansible 模块可用。以下是 boto 库用于连接 AWS 的变量：

| 变量名称 | 描述 |
| --- | --- |
| `AWS_ACCESS_KEY` | 这是有效 IAM 账户的访问密钥 |
| `AWS_SECRET_KEY` | 这是与上面的访问密钥对应的密钥 |
| `AWS_REGION` | 这是默认的区域，除非被覆盖 |

我们可以在示例中使用以下代码设置这些环境变量：

```
export AWS_ACCESS_KEY="AKIAIOSFODNN7EXAMPLE"
export AWS_SECRET_KEY="wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
export AWS_REGION="us-east-1"
```

这些只是示例凭证，无法使用。设置好这些后，我们可以开始使用 AWS 模块。在接下来的代码块中，我们结合了本章中的几个模块来创建一台机器并将其添加到清单中。以下示例使用了尚未讨论的几个功能，如 `register` 和 `delegate_to`，这些将在第三章 *高级 Playbooks* 中讨论：

```
---
- name: Setup an EC2 instance
  hosts: localhost
  connection: local
  tasks:
    - name: Create an EC2 machine
      ec2:
        key_name: daniel-keypair
        instance_type: t2.micro
        image: ami-b66ed3de
        wait: yes
        group: webserver
        vpc_subnet_id: subnet-59483
        assign_public_ip: yes
      register: newmachines

    - name: Wait for SSH to start
      wait_for:
        host: "{{ newmachines.instances[0].public_ip }}"
        port: 22
        timeout: 300
      delegate_to: localhost

    - name: Add the machine to the inventory
      add_host:
        hostname: "{{ newmachines.instances[0].public_ip }}"
        groupname: new

- name: Configure the new machines
  hosts: new
  sudo: yes
  tasks:
    - name: Install a MOTD
      template:
        src: motd.j2
        dest: /etc/motd
```

# 总结

在本章中，我们覆盖了 playbook 文件中可用的部分。我们还学习了如何使用变量使 playbook 可维护，如何在做出更改时触发处理程序，最后，我们查看了某些模块在 playbook 中使用时的优势。你可以通过官方文档进一步探索 Ansible 提供的其他模块，[`docs.ansible.com/modules_by_category.html`](http://docs.ansible.com/modules_by_category.html)。

在下一章中，我们将深入研究 playbook 的更复杂功能。这将使我们能够构建更复杂的 playbook，能够部署和配置整个系统。
