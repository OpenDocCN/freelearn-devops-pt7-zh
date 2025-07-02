# 第三章. 扩展到多个主机

在前几章中，我们在命令行中指定了主机。虽然在只有一个主机的情况下，这种方式运行良好，但在管理多个服务器时效果不好。在本章中，我们将具体了解如何管理多个服务器。

我们将探讨以下主题：

+   Ansible 清单

+   Ansible 主机/组变量

+   Ansible 循环

# 使用清单文件

清单文件是 Ansible 的真实数据来源（还有一个称为**动态清单**的高级概念，我们稍后将介绍）。它遵循**初始化**（**INI**）格式，并告诉 Ansible 用户提供的远程主机是否真实存在。

Ansible 可以并行地在多个主机上执行任务。为此，你可以通过清单文件将主机列表直接传递给 Ansible。为了进行并行执行，Ansible 允许你在清单文件中将主机分组；该文件将组名传递给 Ansible。Ansible 会在清单文件中查找该组，并在该组中列出的所有主机上执行任务。

你可以通过使用`-i`或`--inventory-file`选项，后跟文件路径，将清单文件传递给 Ansible。如果你没有明确指定任何清单文件给 Ansible，它将使用`ansible.cfg`中`host_file`参数的默认路径，该默认路径为`/etc/ansible/hosts`。

### 提示

使用`-i`参数时，如果值是一个`list`（即包含至少一个逗号），它将被用作清单列表，而如果该变量是一个字符串，它将被用作清单文件路径。

## 基本清单文件

在深入了解这个概念之前，让我们先看一个基本的清单文件，叫做**hosts**，我们可以使用它代替之前示例中使用的列表：

```
test01.fale.io 

```

### 提示

Ansible 可以在清单文件中使用 FQDN 或 IP 地址。

我们现在可以执行与上一章相同的操作，调整 Ansible 命令参数。例如，要安装 Web 服务器，我们使用了这个命令：

```
$ ansible-playbook -i test01.fale.io, webserver.yaml

```

相反，我们可以使用以下内容：

```
$ ansible-playbook -i hosts webserver.yaml

```

如你所见，我们已经用清单文件名替代了主机列表。

## 清单文件中的组

当我们面临更复杂的情况时，清单文件的优势就显现出来了。假设我们的网站变得更加复杂，现在需要一个更复杂的环境。在我们的示例中，我们的网站将需要一个 MySQL 数据库。此外，我们决定使用两台 Web 服务器。在这种情况下，根据机器在我们基础设施中的角色来分组不同的机器是有意义的。我们的`hosts`文件将更改为：

```
[webserver] 
ws01.fale.io 
ws02.fale.io 

[database] 
db01.fale.io 

```

现在我们可以指示 playbook 仅在特定组中的主机上运行。我们为我们的网站示例创建了三个不同的 playbook：

+   `firstrun.yaml`是通用的，需要在每台机器上运行

+   `common_tasks.yaml`是通用的，需要在每台机器上运行

+   `webserver.yaml` 文件专门用于 Web 服务器，因此不应在其他机器上运行

我们只需要更改 `webserver.yaml` 文件，当前该文件指定它必须在所有机器上运行，而应该仅限于 Web 服务器。为了做到这一点，让我们打开 `webserver.yaml` 文件，并将内容从：

```
- hosts: all 

```

到：

```
- hosts: webserver 

```

仅凭这三个 playbook，我们无法继续创建带有三台服务器的环境。由于我们还没有设置数据库的 playbook（我们将在下一章中看到），我们将完全配置两台 Web 服务器，而对于数据库服务器，我们只配置基础系统。

我们可以通过以下方式运行 `firstrun` playbook：

```
$ ansible-playbook -i hosts firstrun.yaml

```

以下将是结果：

```
PLAY [all] *******************************************************
TASK [setup] *****************************************************
ok: [ws02.fale.io]
ok: [db01.fale.io]
ok: [ws01.fale.io] 
TASK [Ensure ansible user exists] ********************************
changed: [ws01.fale.io]
changed: [db01.fale.io]
changed: [ws02.fale.io] 
TASK [Ensure ansible user accepts the SSH key] *******************
changed: [ws02.fale.io]
changed: [ws01.fale.io]
changed: [db01.fale.io] 
TASK [Ensure the ansible user is sudoer with no password required]
changed: [ws01.fale.io]
changed: [db01.fale.io]
changed: [ws02.fale.io] 
PLAY RECAP *******************************************************
db01.fale.io      : ok=4    changed=3    unreachable=0    failed=0
ws01.fale.io      : ok=4    changed=3    unreachable=0    failed=0
ws02.fale.io      : ok=4    changed=3    unreachable=0    failed=0

```

如你所见，输出与我们在单一主机上收到的结果非常相似，但每个步骤每个主机都有一行。在这种情况下，所有机器的状态相同，且执行了相同的步骤，因此我们看到它们的行为一致。但在更复杂的场景中，可能会有不同的机器在同一步骤上返回不同的状态。我们也可以执行其他两个 playbook，得到类似的结果。

## 库存文件中的正则表达式

当你有大量服务器时，给它们起个可预测的名字通常是很常见也很有帮助的。例如，可以将所有 Web 服务器命名为 `wsXY` 或 `webXY`，将数据库服务器命名为 `dbXY`。这样，你可以减少 `hosts` 文件中的行数，从而提高可读性。例如，我们的 `hosts` 文件可以简化为：

```
[webserver] 
ws[01:02].fale.io 

[database] 
db01.fale.io 

```

在这个例子中，我们使用了 `[01:02]`，它会匹配从第一个数字（在我们这个例子中是 `01`）到最后一个数字（在我们这个例子中是 `02`）的所有出现情况。虽然我们这个例子的收益不大，但如果你有 40 台 Web 服务器，你就能从 `hosts` 文件中减少 39 行。

# 使用变量

Ansible 允许你通过多种方式定义变量：可以通过 playbook 中的变量文件，使用 `-e / --extra-vars` 选项从 Ansible 命令传递，或通过库存文件传递。你可以在库存文件中定义变量，可以按每个主机、按整个组，或者在库存文件所在的目录中创建一个变量文件。

## 主机变量

可以在 `hosts` 文件中为特定主机声明变量。例如，我们可能想为 Web 服务器指定不同的引擎。假设其中一台需要响应一个特定的域名，而另一台需要响应不同的域名。在这种情况下，我们可以通过以下 `hosts` 文件来实现：

```
[webserver] 
ws01.fale.io domainname=example1.fale.io 
ws02.fale.io domainname=example2.fale.io 

[database] 
db01.fale.io 

```

这样，所有在 Web 服务器上运行的 playbook 都能够引用域名变量。

## 分组变量

还有其他情况，你可能希望设置适用于整个组的变量。假设我们要将变量 `https_enabled` 设置为 `True`，并且它的值必须对所有 Web 服务器一致。在这种情况下，我们可以创建一个 `[webserver:vars]` 部分，所以下面是我们将使用的 `hosts` 文件：

```
[webserver] 
ws01.fale.io 
ws02.fale.io 

[webserver:vars] 
https_enabled=True 

[database] 
db01.fale.io 

```

### 注意

请记住，如果同一变量在主机和组中都被声明，`host` 变量将覆盖 `group` 变量。

## 变量文件

有时，你需要为每个 `host` 和 `group` 声明很多变量，`hosts` 文件变得难以阅读。在这种情况下，你可以将变量移到特定的文件中。对于主机级变量，你需要在 `host_vars` 文件夹中创建一个与主机名称相同的文件，而对于 `group` 变量，你需要使用组名作为文件名，并将其放入 `group_vars` 文件夹中。

所以，如果我们想使用文件复制前面基于主机的变量示例，我们需要创建一个 `host_vars/ws01.fale.io` 文件，文件内容如下：

```
domainname=example1.fale.io 

```

创建 `host_vars/ws02.fale.io` 文件，文件内容如下：

```
domainname=example2.fale.io 

```

如果我们想要复制基于组的变量示例，我们需要有一个 `group_vars/webserver` 文件，文件内容如下：

```
https_enabled=True 

```

### 注意

库存变量遵循层级结构；在顶部是公共变量文件（我们在前一节 *Working with inventory files* 中讨论过），它将覆盖任何主机变量、组变量和库存变量文件。接下来是主机变量，它将覆盖组变量；最后，组变量将覆盖库存变量文件。

## 通过库存文件覆盖配置参数

你可以通过库存文件直接覆盖某些 Ansible 的配置参数。这些配置参数将覆盖通过 `ansible.cfg`、环境变量或在 playbook 中设置的所有其他参数。传递给 `ansible-playbook/ansible` 命令的变量优先于任何其他变量，包括在库存文件中设置的变量。

以下是你可以从库存文件中覆盖的参数列表：

+   `ansible_user`：此参数用于覆盖与远程主机通信时使用的用户。有时，某些机器需要使用不同的用户，在这种情况下，使用此变量会对你有所帮助。

+   `ansible_port`：此参数将覆盖默认的 SSH 端口，使用用户指定的端口。有时，系统管理员选择在非标准端口运行 SSH。在这种情况下，你需要告知 Ansible 这个变化。

+   `ansible_host`：此参数用于覆盖别名的主机。

+   `ansible_connection`：指定与远程主机的连接类型。可选值有 SSH、Paramiko 或 local。

+   `ansible_private_key_file`：此参数将覆盖用于 SSH 的私钥；如果你想为特定主机使用特定的密钥，这将非常有用。一种常见的用例是，如果你有主机分布在多个数据中心、多个 AWS 区域或不同类型的应用程序中。此类场景下，私钥可能会有所不同。

+   `ansible__type`：默认情况下，Ansible 使用 **sh shell**；你可以通过 `ansible_shell_type` 参数覆盖这个设置。将其更改为 `csh`、`ksh` 等，会使 Ansible 使用该 shell 的命令。

# 使用动态清单

在某些环境中，你可能有一个系统自动创建和销毁机器。我们将在 第五章，*云计算之路* 中看到如何使用 Ansible 实现这一点。在这样的环境中，机器列表变化非常快，维护 `hosts` 文件变得复杂。在这种情况下，我们可以使用动态清单来解决问题。

动态清单的理念是，Ansible 不会读取 `hosts` 文件，而是执行一个脚本，该脚本将以 JSON 格式将主机列表返回给 Ansible。例如，这使得你可以直接向你的云服务提供商查询，了解在任何时刻，你整个基础设施中哪些机器正在运行。

许多常见云服务提供商的脚本已经在 Ansible 中提供，位置在：[`github.com/ansible/ansible/tree/devel/contrib/inventory`](https://github.com/ansible/ansible/tree/devel/contrib/inventory)，但如果你有不同的需求，也可以创建自定义脚本。Ansible 清单脚本可以使用任何语言编写，但出于一致性考虑，动态清单脚本应该使用 Python 编写。记住，这些脚本需要直接可执行，因此请确保为其设置可执行标志（`chmod + x inventory.py`）。

在本章中，我们将介绍可以从官方 Ansible 仓库下载的 Amazon Web Services 和 DigitalOcean 脚本。

## Amazon Web Services

为了允许 Ansible 从 **Amazon Web Services** (**AWS**) 获取你的 EC2 实例数据，你需要从 Ansible 的 GitHub 仓库下载以下两个文件：[`github.com/ansible/ansible`](https://github.com/ansible/ansible)：

+   `ec2.py` 清单脚本

+   `ec2.ini` 文件，其中包含 EC2 清单脚本的配置。

Ansible 使用 AWS Python 库 `boto` 通过 API 与 AWS 进行通信。为了允许这种通信，你需要导出 `AWS_ACCESS_KEY_ID` 和 `AWS_SECRET_ACCESS_KEY` 环境变量。

你可以通过两种方式使用清单：

+   直接将其传递给 `ansible-playbook` 命令，使用 `-i` 选项，并将 `ec2.ini` 文件复制到你运行 Ansible 命令的当前目录。

+   将 `ec2.py` 文件复制到 `/etc/ansible/hosts`，使用 `chmod +x` 命令使其可执行，并将 `ec2.ini` 文件复制到 `/etc/ansible/ec2.ini`。

`ec2.py`文件将根据区域、可用区、标签等创建多个组。你可以通过运行`./ec2.py --list`来查看清单文件的内容。

让我们看一个使用 EC2 动态清单的示例 playbook，它将简单地 ping 我账户中的所有机器。

```
ansible -i ec2.py all -m ping

```

如预期的那样，我在账户上的两个 Droplet 返回了以下信息：

```
52.28.138.231 | SUCCESS => { 
    "changed": false, 
    "ping": "pong" 
} 

```

在前面的例子中，我们使用了`ec2.py`脚本，而不是使用静态清单文件，并通过`-i`选项和`ping`命令。

同样，你可以使用这些清单脚本执行各种类型的操作。例如，如果你按照区域（一个区域代表一个数据中心）在 AWS 中逐区部署，你可以将它与部署脚本集成，找出所有位于同一区域的节点并部署。

如果你只是想知道云中有哪些 Web 服务器，并且已经使用某种约定标记了它们，你可以通过过滤标签来使用动态清单脚本。更进一步，如果你有特殊场景，这些场景未被当前的脚本覆盖，你可以增强脚本，使其提供所需的 JSON 格式的节点集，然后从 playbooks 中操作这些节点。如果你使用数据库管理清单，你的清单脚本可以查询数据库并导出 JSON，甚至可以与云同步并定期更新数据库。

## DigitalOcean

我们使用了[`github.com/ansible/ansible/tree/devel/contrib/inventory`](https://github.com/ansible/ansible/tree/devel/contrib/inventory)中的 EC2 文件从 AWS 获取数据，现在我们可以对 DigitalOcean 做同样的操作。唯一的区别是，我们需要获取`digital_ocean.ini`和`digital_ocean.py`文件。

如之前所述，我们可能需要调整`digital_ocean.ini`的选项（如果需要），并使 Python 文件可执行。你可能需要更改的唯一选项是`api_token`。

现在我们可以尝试通过以下命令 ping 所有在`digital_ocean`上可用的机器：

```
ansible -i digital_ocean.py all -m ping

```

如预期的那样，我在账户上的两个 Droplet 返回了以下信息：

```
188.166.150.79 | SUCCESS => { 
    "changed": false, 
    "ping": "pong" 
} 
46.101.77.55 | SUCCESS => { 
    "changed": false, 
    "ping": "pong" 
} 

```

现在我们已经看到从多个不同的云服务提供商获取数据是多么容易。

# 在 Ansible 中使用迭代器

你可能注意到到目前为止我们从未使用过循环，因此每次需要执行多个相似的操作时，我们都会多次编写相同的代码。一个例子就是`webserver.yaml`代码。

实际上，这就是`webserver.yaml`文件的内容：

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
  - name: Ensure HTTPS can pass the firewall 
    firewalld: 
      service: https 
      state: enabled 
      permanent: True 
      immediate: True 
    become: True 

```

如你所见，最后两个模块执行了相同的操作（确保防火墙的某个端口是开放的）。

## 标准迭代 - `with_items`

为了改进上述代码，我们可以使用一个简单的迭代器：`with_items`。

这使我们能够在项列表中进行迭代，每次迭代时，列表中的指定项将通过`item`变量供我们使用。

因此，我们可以将代码更改为以下内容：

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
  - name: Ensure HTTP and HTTPS can pass the firewall 
    firewalld: 
      service: '{{ item }}' 
      state: enabled 
      permanent: True 
      immediate: True 
    become: True 
    with_items: 
    - http 
    - https 

```

我们可以按如下方式执行它：

```
ansible-playbook -i hosts webserver.yaml

```

我们接收到以下内容：

```
PLAY [webserver] *************************************************
TASK [setup] *****************************************************
ok: [ws01.fale.io]
ok: [ws02.fale.io] 
TASK [Ensure the HTTPd package is installed] *********************
ok: [ws01.fale.io]
ok: [ws02.fale.io] 
TASK [Ensure the HTTPd service is enabled and running] ***********
ok: [ws02.fale.io]
ok: [ws01.fale.io] 
TASK [Ensure HTTP can pass the firewall] *************************
ok: [ws02.fale.io]
ok: [ws01.fale.io] 
TASK [Ensure HTTP and HTTPS can pass the firewall] ***************
ok: [ws02.fale.io] => (item=http)
ok: [ws01.fale.io] => (item=http)
ok: [ws02.fale.io] => (item=https)
ok: [ws01.fale.io] => (item=https) 
PLAY RECAP *******************************************************
ws01.fale.io      : ok=5    changed=0    unreachable=0    failed=0 
ws02.fale.io      : ok=5    changed=0    unreachable=0    failed=0 

```

如你所见，输出与之前的执行略有不同，实际上在涉及循环操作的行中，我们可以看到处理的项——“确保 HTTP 和 HTTPS 能通过防火墙”块

我们现在已经了解了如何在一个项目列表上进行迭代，但 Ansible 也支持其他类型的迭代。

## 嵌套循环 - with_nested

有些情况下，你需要对一个列表中的所有元素与其他列表中的所有项进行迭代（**笛卡尔积**）。一个非常常见的情况是，当你需要在多个路径中创建多个文件夹时。在我们的示例中，我们将分别在用户 `alice` 和 `bob` 的 `home` 文件夹中创建 `mail` 和 `public_html` 文件夹。

我们可以使用以下来自 `with_nested.yaml` 文件的代码实现：

```
--- 
- hosts: all 
  remote_user: ansible 
  vars: 
    users: 
    - alice 
    - bob 
    folders: 
    - mail 
    - public_html 
  tasks: 
  - name: Ensure the users exist 
    user: 
      name: '{{ item }}' 
    become: True 
    with_items: 
    - '{{ users }}' 
  - name: Ensure the folders exist 
    file: 
      path: '/home/{{ item.0 }}/{{ item.1 }}' 
      state: directory 
    become: True 
    with_nested: 
    - '{{ users }}' 
    - '{{ folders }}' 

```

使用以下命令运行：

```
ansible-playbook -i hosts with_nested.yaml

```

我们收到以下结果：

```
PLAY [all] *******************************************************
TASK [setup] *****************************************************
ok: [ws01.fale.io]
ok: [db01.fale.io]
ok: [ws02.fale.io] 
TASK [Ensure the users exist] ************************************
changed: [db01.fale.io] => (item=alice)
changed: [ws01.fale.io] => (item=alice)
changed: [ws02.fale.io] => (item=alice)
changed: [db01.fale.io] => (item=bob)
changed: [ws01.fale.io] => (item=bob)
changed: [ws02.fale.io] => (item=bob) 
TASK [Ensure the folders exist] **********************************
changed: [ws02.fale.io] => (item=[u'alice', u'mail'])
changed: [ws01.fale.io] => (item=[u'alice', u'mail'])
changed: [db01.fale.io] => (item=[u'alice', u'mail'])
changed: [ws01.fale.io] => (item=[u'alice', u'public_html'])
changed: [ws02.fale.io] => (item=[u'alice', u'public_html'])
changed: [db01.fale.io] => (item=[u'alice', u'public_html'])
changed: [ws02.fale.io] => (item=[u'bob', u'mail'])
changed: [ws01.fale.io] => (item=[u'bob', u'mail'])
changed: [db01.fale.io] => (item=[u'bob', u'mail'])
changed: [ws02.fale.io] => (item=[u'bob', u'public_html'])
changed: [ws01.fale.io] => (item=[u'bob', u'public_html'])
changed: [db01.fale.io] => (item=[u'bob', u'public_html']) 
PLAY RECAP *******************************************************
db01.fale.io      : ok=3    changed=2    unreachable=0    failed=0 
ws01.fale.io      : ok=3    changed=2    unreachable=0    failed=0 
ws02.fale.io      : ok=3    changed=2    unreachable=0    failed=0 

```

## 文件通配符循环 - with_fileglobs

有时，我们希望对某个文件夹中每个文件执行某种操作。如果你想将多个具有相似名称的文件从一个文件夹复制到另一个文件夹，这可能会非常方便。为此，你可以创建一个名为 `with_fileglobs.yaml` 的文件，并在其中写入以下代码：

```
--- 
- hosts: all 
  remote_user: ansible 
  tasks: 
  - name: Ensure the folder /tmp/iproute2 is present 
    file: 
      dest: '/tmp/iproute2' 
      state: directory 
    become: True 
  - name: Copy files that start with rt to the tmp folder 
    copy: 
      src: '{{ item }}' 
      dest: '/tmp/iproute2' 
      remote_src: True 
    become: True 
    with_fileglob: 
    - '/etc/iproute2/rt_*' 

```

我们可以使用以下命令执行：

```
ansible-playbook -i hosts with_fileglobs.yaml

```

以接收到类似以下的输出：

```
PLAY [all] *******************************************************
TASK [setup] *****************************************************
ok: [ws02.fale.io]
ok: [db01.fale.io]
ok: [ws01.fale.io] 
TASK [Ensure the folder /tmp/iproute2 is present] ****************
changed: [db01.fale.io]
changed: [ws01.fale.io]
changed: [ws02.fale.io] 
TASK [Copy files that start with rt to the tmp folder] ***********
changed: [db01.fale.io] => (item=/etc/iproute2/rt_dsfield)
changed: [ws02.fale.io] => (item=/etc/iproute2/rt_dsfield)
changed: [ws01.fale.io] => (item=/etc/iproute2/rt_dsfield)
changed: [db01.fale.io] => (item=/etc/iproute2/rt_protos)
changed: [ws01.fale.io] => (item=/etc/iproute2/rt_protos)
changed: [ws02.fale.io] => (item=/etc/iproute2/rt_protos)
changed: [db01.fale.io] => (item=/etc/iproute2/rt_tables)
changed: [ws01.fale.io] => (item=/etc/iproute2/rt_tables)
changed: [ws02.fale.io] => (item=/etc/iproute2/rt_tables)
changed: [db01.fale.io] => (item=/etc/iproute2/rt_scopes)
changed: [ws01.fale.io] => (item=/etc/iproute2/rt_scopes)
changed: [ws02.fale.io] => (item=/etc/iproute2/rt_scopes)
changed: [db01.fale.io] => (item=/etc/iproute2/rt_realms)
changed: [ws01.fale.io] => (item=/etc/iproute2/rt_realms)
changed: [ws02.fale.io] => (item=/etc/iproute2/rt_realms) 
PLAY RECAP *******************************************************
db01.fale.io      : ok=3    changed=2    unreachable=0    failed=0 
ws01.fale.io      : ok=3    changed=2    unreachable=0    failed=0 
ws02.fale.io      : ok=3    changed=2    unreachable=0    failed=0 

```

## 整数循环 - with_sequence

许多时候，你需要对整数进行迭代。一个例子可能是创建十个名为 `fileXY` 的文件夹，其中 `XY` 是从 `1` 到 `10` 的顺序数字。为此，我们可以创建一个名为 `with_sequence.yaml` 的文件，并在其中写入以下代码：

```
--- 
- hosts: all 
  remote_user: ansible 
  tasks: 
  - name: Create the folders /tmp/dirXY with XY from 1 to 10 
    file: 
      dest: '/tmp/dir{{ item }}' 
      state: directory 
    with_sequence: start=1 end=10 
    become: True 

```

### 注意

对于 `with_sequence`，我们必须使用单行表示法。

然后，我们可以执行以下操作：

```
ansible-playbook -i hosts with_sequence.yaml

```

我们将收到：

```
PLAY [all] *******************************************************
TASK [setup] *****************************************************
ok: [db01.fale.io]
ok: [ws01.fale.io]
ok: [ws02.fale.io] 
TASK [Create the folders /tmp/dirXY with XY from 1 to 10] ********
changed: [ws02.fale.io] => (item=1)
changed: [ws01.fale.io] => (item=1)
changed: [db01.fale.io] => (item=1)
changed: [db01.fale.io] => (item=2)
changed: [ws02.fale.io] => (item=2)
changed: [ws01.fale.io] => (item=2)
changed: [db01.fale.io] => (item=3)
changed: [ws01.fale.io] => (item=3)
changed: [ws02.fale.io] => (item=3)
changed: [db01.fale.io] => (item=4)
changed: [ws01.fale.io] => (item=4)
changed: [ws02.fale.io] => (item=4)
changed: [db01.fale.io] => (item=5)
changed: [ws01.fale.io] => (item=5)
changed: [ws02.fale.io] => (item=5)
changed: [db01.fale.io] => (item=6)
changed: [ws01.fale.io] => (item=6)
changed: [ws02.fale.io] => (item=6)
changed: [db01.fale.io] => (item=7)
changed: [ws01.fale.io] => (item=7)
changed: [ws02.fale.io] => (item=7)
changed: [db01.fale.io] => (item=8)
changed: [ws01.fale.io] => (item=8)
changed: [ws02.fale.io] => (item=8)
changed: [db01.fale.io] => (item=9)
changed: [ws01.fale.io] => (item=9)
changed: [ws02.fale.io] => (item=9)
changed: [db01.fale.io] => (item=10)
changed: [ws01.fale.io] => (item=10)
changed: [ws02.fale.io] => (item=10) 
PLAY RECAP *******************************************************
db01.fale.io      : ok=2    changed=1    unreachable=0    failed=0 
ws01.fale.io      : ok=2    changed=1    unreachable=0    failed=0 
ws02.fale.io      : ok=2    changed=1    unreachable=0    failed=0 

```

Ansible 支持更多种类的循环，但由于它们使用的较少，你可以直接参考官方文档中的循环部分：[`docs.ansible.com/ansible/playbooks_loops.html`](http://docs.ansible.com/ansible/playbooks_loops.html)。

# 总结

在本章中，我们介绍了许多概念，帮助你将基础设施扩展到单一节点之外。我们从用于指示 Ansible 关于机器的清单文件开始，接着讨论了如何在对多个异构主机运行相同命令时使用主机特定和组特定的变量。然后，我们介绍了动态清单，这些清单是由其他系统（通常是云提供商）直接填充的。最后，我们分析了 Ansible playbook 中的多种迭代方式。

在下一章中，我们将以更合理的方式组织我们的 Ansible 文件，以确保最大程度的可读性。为此，我们引入了角色，这进一步简化了复杂环境的管理。
