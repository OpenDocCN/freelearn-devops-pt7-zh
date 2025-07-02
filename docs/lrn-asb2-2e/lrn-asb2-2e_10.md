# 第十章. 介绍企业版 Ansible

在本章中，我们将讨论 Ansible 在不同操作系统上的状态。我们还将看看 Ansible Galaxy 和 Ansible Tower。

我们将探讨以下主题：

+   Ansible 在 Windows 上

+   Ansible 用于网络设备

+   Ansible Galaxy

+   Ansible Tower

# Ansible 在 Windows 上

Ansible 版本 1.7 开始能够通过一些基本模块管理 Windows 机器。在 Red Hat 收购 Ansible 后，微软和其他许多公司及个人为这个任务投入了大量精力。到 2.1 版本发布时，Ansible 管理 Windows 机器的能力接近完成。某些模块已扩展为可以在 Unix 和 Windows 上无缝工作，而在其他情况下，Windows 的逻辑与 Unix 差异如此之大，以至于需要创建新的模块。

### 注意

目前，使用 Windows 作为控制机并不被支持，尽管一些用户已通过调整代码和环境使其能够工作。

控制机到 Windows 机器的连接不是通过 SSH 进行的，而是通过**Windows 远程管理**（**WinRM**）进行的。你可以访问微软的网站，获取详细的解释和实现：[`msdn.microsoft.com/en-us/library/aa384426(v=vs.85).aspx`](http://msdn.microsoft.com/en-us/library/aa384426(v=vs.85).aspx)。

在控制机上，安装了 Ansible 后，重要的是你需要安装 WinRM。你可以通过`pip`命令安装：

```
pip install "pywinrm>=0.1.1"

```

### 注意

你可能需要使用`sudo`或`root`账户来执行此命令。

在每台远程 Windows 机器上，你需要安装 PowerShell 3.0 或更高版本。Ansible 提供了一些有用的脚本来帮助设置：

+   **WinRM**: [`github.com/ansible/ansible/blob/devel/examples/scripts/ConfigureRemotingForAnsible.ps1`](https://github.com/ansible/ansible/blob/devel/examples/scripts/ConfigureRemotingForAnsible.ps1)

+   **PowerShell 3.0 升级**: [`github.com/cchurch/ansible/blob/devel/examples/scripts/upgrade_to_ps3.ps1`](https://github.com/cchurch/ansible/blob/devel/examples/scripts/upgrade_to_ps3.ps1)

你还需要通过防火墙允许端口`5986`，因为这是默认的 WinRM 连接端口，并确保从命令中心可以访问该端口。

为了确保你可以远程访问服务，运行一个`curl`命令：

```
curl -vk -d `` -u "$USER:$PASSWORD" "https://<IP>:5986/wsman". 

```

如果基本身份验证工作正常，你就可以开始运行命令了。设置完成后，你就可以开始运行 Ansible 了！让我们通过运行`win_ping`来执行 Ansible 中的 Windows 版本的`Hello, world!`程序。为了做到这一点，让我们设置我们的凭证文件。

这可以通过 Ansible vault 来完成，如下所示：

```
$ ansible-vault create group_vars/windows.yml

```

正如我们所见，Ansible vault 将交互式地要求设置密码：

```
Vault password:
Confirm Vault password:

```

此时，我们可以添加所需的变量：

```
    ansible_ssh_user: Administrator 
    ansible_ssh_pass: <password> 
    ansible_ssh_port: 5986 
    ansible_connection: winrm 

```

让我们设置我们的`inventory`文件，如下所示：

```
    [windows] 
    174.129.181.242 

```

接下来，我们运行`win_ping`：

```
ansible windows -i inventory -m win_ping --ask-vault-pass

```

Ansible 将要求我们输入 vault 密码，然后打印运行结果，如下所示：

```
    Vault password: 
    174.129.181.242 | success >> { 
        "changed": false, 
        "ping": "pong" 
    } 

```

# Ansible 用于网络设备

自版本 2.1 以来，我们看到许多新的模块用于网络设备和软件的管理。许多这些模块是由创建设备（或软件）的公司直接贡献的。这样做的一个巨大优势是，基于**软件定义网络**（**SDN**）的理念，Ansible 可以管理您的所有网络基础设施，从而使您能够在 Ansible 中完全管理整个数据中心。这意味着，所有 IT 组件和所有人员使用相同的语言，这将有助于人们更好地理解公司 IT 的运作方式，并促进团队之间的紧密合作。

# Ansible Galaxy

Ansible Galaxy 是一个免费的站点，您可以在此下载由社区开发的 Ansible 角色，并在几分钟内启动自动化。您可以分享或审核社区角色，以便其他人能够轻松找到 Ansible Galaxy 上最受信任的角色。您可以通过简单地使用 Twitter、Google 或 GitHub 等社交媒体应用程序注册，或者在 Ansible Galaxy 网站上创建一个新账户，网址为[`galaxy.ansible.com/`](https://galaxy.ansible.com/)，并使用`ansible-galaxy`命令下载所需的角色，该命令随 Ansible 版本 1.4.2 及更高版本一起提供。

### 注意

如果您想托管自己的本地 Ansible Galaxy 实例，可以通过从[`github.com/ansible/galaxy`](https://github.com/ansible/galaxy)获取代码来实现。

要从 Ansible Galaxy 下载 Ansible 角色，请使用以下语法：

```
ansible-galaxy install username.rolename

```

您还可以通过如下方式指定版本：

```
ansible-galaxy install username.rolename[,version]

```

如果不指定版本，`ansible-galaxy`命令将下载最新的可用版本。您可以通过两种方式安装多个角色；首先，通过传递多个角色名称并用空格分隔，如下所示：

```
ansible-galaxy install username.rolename[,version] username.rolename[,version]

```

其次，您可以通过在文件中指定角色名称并将该文件名传递给`-r/--role-file`选项来执行此操作。例如，您可以创建`requirements.txt`文件，内容如下：

```
    user1.rolename,v1.0.0 
    user2.rolename,v1.1.0 
    user3.rolename,v1.2.1 

```

然后，您可以通过将文件名传递给`ansible-galaxy`命令来安装角色，如下所示：

```
ansible-galaxy install -r requirements

```

让我们看看如何使用`ansible-galaxy`下载 Apache HTTPd 角色：

```
sudo ansible-galaxy install geerlingguy.apache

```

您将看到如下输出：

```
- downloading role 'apache', owned by geerlingguy
- downloading role from https://github.com/geerlingguy/ansible-role-apache/archive/1.7.3.tar.gz
- extracting geerlingguy.apache to /etc/ansible/roles/geerlingguy.apache
- geerlingguy.apache was installed successfully

```

上述`ansible-galaxy`命令将把 Apache HTTPd 角色下载到`/etc/ansible/roles`目录中。现在，您可以在剧本中直接使用上述角色，创建`playbooks/galaxy.yaml`文件，内容如下：

```
    - hosts: web 
      user: ansible 
      become: True 
      roles: 
      - geerlingguy.apache 

```

如您所见，我们创建了一个简单的剧本，并使用了`geerlingguy.apache`角色。现在我们可以进行测试：

```
ansible-playbook playbooks/galaxy.yaml

```

这将给我们如下输出：

```
PLAY [web] ******************************************************* 
TASK [setup] *****************************************************
ok: [ws01.fale.io]
ok: [ws02.fale.io] 
TASK [geerlingguy.apache : Include OS-specific variables.] *******
ok: [ws01.fale.io]
ok: [ws02.fale.io] 
TASK [geerlingguy.apache : Define apache_packages.] **************
ok: [ws01.fale.io]
ok: [ws02.fale.io] 
TASK [geerlingguy.apache : Ensure Apache is installed on RHEL.] **
changed: [ws01.fale.io] => (item=[u'httpd', u'httpd-devel', u'mod_ssl', u'openssh'])
changed: [ws02.fale.io] => (item=[u'httpd', u'httpd-devel', u'mod_ssl', u'openssh']) 
TASK [geerlingguy.apache : Ensure Apache is installed on Suse.] **
skipping: [ws01.fale.io] => (item=[])
skipping: [ws02.fale.io] => (item=[]) 
TASK [geerlingguy.apache : Update apt cache.] ********************
skipping: [ws01.fale.io]
skipping: [ws02.fale.io] 
TASK [geerlingguy.apache : Ensure Apache is installed on Debian.]
skipping: [ws01.fale.io] => (item=[])
skipping: [ws02.fale.io] => (item=[]) 
TASK [geerlingguy.apache : Ensure Apache is installed on Solaris.]
skipping: [ws01.fale.io] => (item=httpd)
skipping: [ws02.fale.io] => (item=httpd)
skipping: [ws01.fale.io] => (item=httpd-devel)
skipping: [ws02.fale.io] => (item=httpd-devel)
skipping: [ws02.fale.io] => (item=mod_ssl)
skipping: [ws01.fale.io] => (item=mod_ssl)
skipping: [ws02.fale.io] => (item=openssh)
skipping: [ws01.fale.io] => (item=openssh) 
TASK [geerlingguy.apache : Get installed version of Apache.] *****
ok: [ws01.fale.io]
ok: [ws02.fale.io] 
TASK [geerlingguy.apache : Create apache_version variable.] ******
ok: [ws01.fale.io]
ok: [ws02.fale.io] 
TASK [geerlingguy.apache : include_vars] *************************
skipping: [ws01.fale.io]
skipping: [ws02.fale.io] 
TASK [geerlingguy.apache : include_vars] *************************
ok: [ws01.fale.io]
ok: [ws02.fale.io] 
TASK [geerlingguy.apache : Configure Apache.] ********************
ok: [ws01.fale.io] => (item={u'regexp': u'^Listen ', u'line': u'Listen 80'})
ok: [ws02.fale.io] => (item={u'regexp': u'^Listen ', u'line': u'Listen 80'}) 
TASK [geerlingguy.apache : Check whether certificates defined in vhosts exist.] 
TASK [geerlingguy.apache : Add apache vhosts configuration.] *****
changed: [ws01.fale.io]
changed: [ws02.fale.io] 
TASK [geerlingguy.apache : Configure Apache.] ********************
skipping: [ws01.fale.io] => (item={u'regexp': u'^Listen ', u'line': u'Listen 80'})
skipping: [ws02.fale.io] => (item={u'regexp': u'^Listen ', u'line': u'Listen 80'}) 
TASK [geerlingguy.apache : Check whether certificates defined in vhosts exist.] 
TASK [geerlingguy.apache : Add apache vhosts configuration.] *****
skipping: [ws01.fale.io]
skipping: [ws02.fale.io] 
TASK [geerlingguy.apache : Configure Apache.] ********************
skipping: [ws01.fale.io] => (item={u'regexp': u'^Listen ', u'line': u'Listen 80'})
skipping: [ws02.fale.io] => (item={u'regexp': u'^Listen ', u'line': u'Listen 80'}) 
TASK [geerlingguy.apache : Enable Apache mods.] ******************
skipping: [ws01.fale.io] => (item=rewrite.load)
skipping: [ws02.fale.io] => (item=rewrite.load)
skipping: [ws01.fale.io] => (item=ssl.load)
skipping: [ws02.fale.io] => (item=ssl.load) 
TASK [geerlingguy.apache : Disable Apache mods.] ***************** 
TASK [geerlingguy.apache : Check whether certificates defined in vhosts exist.] 
TASK [geerlingguy.apache : Add apache vhosts configuration.] *****
skipping: [ws01.fale.io]
skipping: [ws02.fale.io] 
TASK [geerlingguy.apache : Add vhost symlink in sites-enabled.] **
skipping: [ws01.fale.io]
skipping: [ws02.fale.io] 
TASK [geerlingguy.apache : Remove default vhost in sites-enabled.]
skipping: [ws01.fale.io]
skipping: [ws02.fale.io] 
TASK [geerlingguy.apache : Configure Apache.] ********************
skipping: [ws01.fale.io] => (item={u'regexp': u'^Listen ', u'line': u'Listen 80'})
skipping: [ws02.fale.io] => (item={u'regexp': u'^Listen ', u'line': u'Listen 80'}) 
TASK [geerlingguy.apache : Add apache vhosts configuration.] *****
skipping: [ws01.fale.io]
skipping: [ws02.fale.io] 
TASK [geerlingguy.apache : Ensure Apache has selected state and enabled on boot.]
ok: [ws01.fale.io]
ok: [ws02.fale.io] 
RUNNING HANDLER [geerlingguy.apache : restart apache] ************
changed: [ws01.fale.io]
changed: [ws02.fale.io] 
PLAY RECAP *******************************************************
ws01.fale.io      : ok=11   changed=3    unreachable=0    failed=0
ws02.fale.io      : ok=11   changed=3    unreachable=0    failed=0

```

### 注意

正如您可能已经注意到的，由于该角色设计用于在许多不同的 Linux 发行版上运行，因此许多步骤被跳过了。

# Ansible Tower

Ansible Tower 是红帽开发的一个基于 Web 的图形用户界面（GUI）。Ansible Tower 提供了一个易于使用的仪表盘，让您管理节点，并通过基于角色的身份验证控制对 Ansible Tower 仪表盘的访问。Ansible Tower 的主要特点如下：

+   **LDAP/AD 集成**：您可以根据 Ansible Tower 在您的 LDAP/AD 服务器上执行的 LDAP/AD 查询结果导入（并赋予权限）用户。

+   **基于角色的访问控制**：限制用户仅运行他们被授权运行的 playbook，并/或仅针对有限数量的主机。

+   **REST API**：所有 Ansible Tower 的功能都通过 REST API 暴露。

+   **作业调度**：Ansible Tower 允许我们调度作业（playbook 执行）。

+   **图形化库存管理**：Ansible Tower 以比 Ansible 更动态的方式管理库存。

+   **仪表盘**：Ansible Tower 允许我们查看所有当前和过去作业执行的情况。

+   **日志记录**：Ansible Tower 记录每个作业执行的所有结果，以便在需要时回溯检查。

尽管红帽（Red Hat）承诺很快将 Ansible Tower 开源，但目前它并非免费提供，您需要根据要管理的节点数量支付费用。

在撰写本文时，红帽为 10 个节点提供了免费的 Ansible Tower 副本。有关更多详情，请访问 Ansible Tower 网站：[`www.ansible.com/tower`](http://www.ansible.com/tower)；用户指南可在 [`docs.ansible.com/ansible-tower/`](http://docs.ansible.com/ansible-tower/) 中找到。

# 总结

在本章中，我们已经看到了一些 Ansible 及其生态系统提供的选项。本章还希望教会你如何在 Ansible 文档中查找那些不那么规范的内容，因为 Ansible 可能具有这样的功能。此外，正如你可能已经注意到的，本章涵盖的许多主题在 2.1 版本（发布于本书第二版出版前不到 6 个月）中发生了重大变化，并且这些领域仍在积极开发中，因此查看官方文档是了解这些主题当前状态的最佳途径。
