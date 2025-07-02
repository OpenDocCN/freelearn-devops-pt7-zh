# 第九章. 复杂环境

到目前为止，我们已经看到如何开发剧本并进行测试。最后一个方面是如何将剧本发布到生产环境。在大多数情况下，在剧本发布到生产环境之前，你将需要处理多个环境。这类似于你开发人员编写的软件。许多公司有多个环境，通常你的剧本将遵循以下步骤：

+   开发环境

+   测试环境

+   阶段环境

+   生产环境

一些公司以不同的方式命名这些环境，还有一些公司有额外的环境，如认证环境，在生产环境之前，所有软件必须先通过认证。

当你编写剧本并设置角色时，我们强烈建议从一开始就考虑到环境的概念。与软件和运维团队沟通，搞清楚你的系统需要支持多少个环境可能是值得的。

我们将列出几种方法和示例，你可以在你的环境中使用：

# 基于 Git 分支的代码

假设你需要处理四个环境，如下所示：

+   开发环境

+   测试环境

+   阶段

+   生产环境

在基于 Git 分支的方法中，你将为每个分支拥有一个环境。你将始终首先在**开发环境**中进行更改，然后将这些更改推广到**测试环境**（合并或挑选提交并在 Git 中标记），**阶段环境**和**生产环境**。在这种方法中，你将拥有一个单一的清单文件、一组变量文件，以及每个分支专用的角色和剧本文件夹。

# 一个稳定的主分支，配有多个文件夹

在这种方法中，你将始终保持 dev 和 master 分支。初始代码提交到 dev 分支，一旦稳定，你会将其推送到 master 分支。master 分支中存在的相同角色和剧本将在所有环境中运行。另一方面，你将为每个环境保持独立的文件夹。我们来看一个例子。我们将展示如何为两个环境——阶段和生产——设置单独的配置和清单。你可以根据你的场景扩展，适应所有使用的环境。首先，我们来看一下`playbooks/variables.yaml`中的剧本，它将在这些多个环境中运行，内容如下：

```
- hosts: web 
  user: root 
  tasks: 
  - name: Print environment name 
    debug: 
      var: env 
  - name: Print db server url 
    debug: 
      var: db_url 
  - name: Print domain url 
    debug: 
      var: domain 
- hosts: db 
  user: root 
  tasks: 
  - name: Print environment name 
    debug: 
      var: env 
  - name: Print database username 
    debug: 
      var: db_user 
  - name: Print database password 
    debug: 
      var: db_pass 

```

如你所见，剧本中有两组任务：

+   运行于数据库服务器的任务

+   运行于 Web 服务器的任务

还有一个额外的任务，用于打印特定环境中所有服务器的环境名称。我们还会有两个不同的清单文件。

第一个将被称为`inventory/production`，其内容如下：

```
[web] 
ws01.fale.io 
ws02.fale.io 

[db] 
db01.fale.io 

[production:children] 
db 
web 

```

第二个将被称为`inventory/staging`，其内容如下：

```
[web] 
ws01.stage.fale.io 
ws02.stage.fale.io 

[db] 
db01.stage.fale.io 

[staging:children] 
db 
web 

```

如你所见，在每个环境中，我们为`web`部分配有两台机器，为`db`部分配有一台机器。此外，我们为阶段和生产环境配置了不同的机器。额外的部分`[ENVIRONMENT:children]`允许你创建一个组的组。这意味着在`ENVIRONMENT`部分中定义的任何变量将应用于`db`和`web`组，除非它们在各自的独立部分中被覆盖。下一个有趣的部分是查看每个环境中变量的值，并查看它们在每个环境中的分离方式。

让我们从所有环境通用的变量开始，它们位于`inventory/group_vars/all`中：

```
db_user: mysqluser 

```

唯一在两个环境中相同的变量是`db_user`。

现在我们可以查看生产环境特定的变量，它们位于`inventory/group_vars/production`中：

```
env: production 
domain: fale.io 
db_url: db.fale.io 
db_pass: this_is_a_safe_password 

```

如果我们现在查看位于`inventory/group_vars/staging`中的阶段特定变量，我们会发现它们与生产环境中的变量相同，但值不同：

```
env: staging 
domain: stage.fale.io 
db_url: db.stage.fale.io 
db_pass: this_is_an_unsafe_password 

```

现在我们可以验证是否收到了预期的结果。首先，我们将在暂存环境中运行：

```
ansible-playbook -i staging playbooks/variables.yaml

```

我们应该收到类似于以下内容的输出：

```
PLAY [web] *******************************************************

TASK [setup] ***************************************************** 
ok: [ws02.stage.fale.io] 
ok: [ws01.stage.fale.io] 

TASK [Print environment name] ************************************ 
ok: [ws01.stage.fale.io] => { 
    "env": "staging" 
} 
ok: [ws02.stage.fale.io] => { 
    "env": "staging" 
} 

TASK [Print db server url] *************************************** 
ok: [ws01.stage.fale.io] => { 
    "db_url": "db.stage.fale.io" 
} 
ok: [ws02.stage.fale.io] => { 
    "db_url": "db.stage.fale.io" 
} 

TASK [Print domain url] ****************************************** 
ok: [ws01.stage.fale.io] => { 
    "domain": "stage.fale.io" 
} 
ok: [ws02.stage.fale.io] => { 
    "domain": "stage.fale.io" 
} 

PLAY [db] ******************************************************** 

TASK [setup] ***************************************************** 
ok: [db01.stage.fale.io] 

TASK [Print environment name] ************************************ 
ok: [db01.stage.fale.io] => { 
    "env": "staging" 
} 

TASK [Print database username] *********************************** 
ok: [db01.stage.fale.io] => { 
    "db_user": "mysqluser" 
} 

TASK [Print database password] ********************************** 
ok: [db01.stage.fale.io] => { 
    "db_pass": "this_is_an_unsafe_password" 
} 

PLAY RECAP ******************************************************* 
db01.stage.fale.io: ok=4    changed=0    unreachable=0    failed=0 
ws01.stage.fale.io: ok=4    changed=0    unreachable=0    failed=0 
ws02.stage.fale.io: ok=4    changed=0    unreachable=0    failed=0

```

现在我们可以在生产环境中运行：

```
ansible-playbook -i production playbooks/variables.yaml

```

我们将收到以下结果：

```
PLAY [web] ******************************************************* 
TASK [setup] *****************************************************
ok: [ws02.fale.io]
ok: [ws01.fale.io] 
TASK [Print environment name] ************************************
ok: [ws01.fale.io] => {
 "env": "production"
}
ok: [ws02.fale.io] => {
 "env": "production"
} 
TASK [Print db server url] ***************************************
ok: [ws01.fale.io] => {
 "db_url": "db.fale.io"
}
ok: [ws02.fale.io] => {
 "db_url": "db.fale.io"
} 
TASK [Print domain url] ******************************************
ok: [ws01.fale.io] => {
 "domain": "fale.io"
}
ok: [ws02.fale.io] => {
 "domain": "fale.io"
} 
PLAY [db] ******************************************************** 
TASK [setup] *****************************************************
ok: [db01.fale.io] 
TASK [Print environment name] ************************************
ok: [db01.fale.io] => {
 "env": "production"
} 
TASK [Print database username] ***********************************
ok: [db01.fale.io] => {
 "db_user": "mysqluser"
} 
TASK [Parint database password] **********************************
ok: [db01.fale.io] => {
 "db_pass": "this_is_a_safe_password"
} 
PLAY RECAP *******************************************************
db01.fale.io      : ok=4    changed=0    unreachable=0    failed=0
ws01.fale.io      : ok=4    changed=0    unreachable=0    failed=0
ws02.fale.io      : ok=4    changed=0    unreachable=0    failed=0

```

你可以看到，Ansible 运行已经获取了为暂存环境定义的所有相关变量。

如果你正在使用这种方法来为多个环境获取稳定的主分支，最好结合使用特定于环境的目录、`group_vars`和清单组来解决这个问题。

# 软件分发策略

部署应用程序可能是**信息与通信技术**（**ICT**）领域最复杂的任务之一。其主要原因是它通常需要更改大多数在某种程度上参与该应用程序的机器的状态。事实上，通常你会发现自己不得不在部署过程中同时更改负载均衡器、分发服务器、应用服务器和数据库服务器的状态。像容器这样的新技术正在努力简化这些操作，但将传统应用程序迁移到容器中往往并不容易或可行。

接下来我们将看到各种软件分发策略，以及 Ansible 如何帮助实现每一种策略。

## 从本地机器复制文件

这可能是分发软件的最古老策略。其思想是将文件放在本地机器上（通常用于开发代码），一旦做出更改，就将文件复制到服务器上（通常通过 FTP）。这种部署代码的方式通常用于 Web 开发，其中代码（通常是 PHP）不需要任何编译。

由于其多种问题，这种分发策略应该避免使用。

+   很难回滚

+   无法跟踪不同部署的变化

+   没有部署历史

+   在部署过程中容易出错

尽管这种分发策略可以通过 Ansible 轻松自动化，但我强烈建议你立即转向一种允许你有更安全分发策略的其他方法。

## 带分支的版本控制系统

许多公司正在使用这种技术来分发他们的软件，主要是针对未编译的软件。这种技术的核心思想是将服务器设置为使用本地代码仓库副本。使用 SVN 时，这种方法是可行的，但管理起来不太容易，而 Git 则使这一技术得到了简化，因而变得非常流行。

这种技术相较于我们刚才看到的方法有许多优势，主要有：

+   简单的回滚

+   非常容易获取变更历史

+   非常简便的部署（主要是使用 Git 时）

另一方面，这种技术仍然有多个缺点：

+   没有部署历史

+   对于编译软件来说比较困难

+   可能的安全问题

我想再谈谈你可能会遇到的安全问题。非常诱人的一种做法是将 Git 仓库直接下载到你用来分发内容的文件夹中，比如如果是一个 Web 服务器，那就是`/var/www/`文件夹。这有显而易见的优势，因为部署时你只需要执行`git pull`命令。缺点是 Git 会创建`/var/www/.git`文件夹，其中包含整个 Git 仓库（包括历史记录），如果没有适当保护，它将会被任何人自由下载。

### 注意

大约 1%的 Alexa 前百万网站将 Git 文件夹公开访问，因此如果你打算使用这种分发策略，请非常小心。

## 带标签的版本控制系统

另一种使用版本控制系统的方式稍微复杂一些，但有一些优势，那就是利用标签系统。此方法要求每次部署时都要打标签，然后在服务器上检出特定的标签。

这种方法具备之前方法的所有优势，并且增加了部署历史记录。编译软件问题和可能的安全问题与前述方法相同。

## RPM 包

一种非常常见的软件部署方式（主要适用于编译过的应用程序，但对未编译的应用程序也有优势）是使用某种打包系统。一些语言，如 Java，已经包含了一个系统（在 Java 中是 WAR），但也有一些可以用于任何类型应用程序的打包系统，例如 RPM。这些系统的缺点是，它们比之前的方法稍微复杂一些，但它们可以提供更高的安全性和版本控制。此外，这些系统很容易在 CI/CD 流水线中注入，因此实际的复杂性比初看起来要低，因为 CI/CD 流水线会自动处理构建过程。

# 准备环境

要查看我们如何以我们在前几页中讨论的不同方式部署代码，我们需要一个环境，显然我们将使用 Ansible 来创建它。首先，为了确保我们的角色被正确加载，我们需要`ansible.cfg`文件，内容如下：

```
[defaults] 
roles_path = roles 

```

然后我们需要`playbooks/firstrun.yaml`来确保我们可以使用基本配置来配置我们的机器，内容如下：

```
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

`playbooks/groups/web.yaml`也需要创建，以便我们能够正确引导我们的 Web 服务器：

```
- hosts: web 
  user: ansible 
  roles: 
  - common 
  - webserver 

```

正如你从前面的文件内容中可以想象的那样，我们需要创建角色：`common`和`webserver`，它们与我们在第四章中创建的非常相似，*处理复杂的部署*。我们从`roles/common/tasks/main.yaml`文件开始，文件内容如下：

```
- name: Ensure EPEL is enabled
   yum:
     name: epel-release
     state: present
   become: True
- name: Ensure needed packages are present
   yum:
     name: '{{ item }}'
     state: present
   become: True
   with_items:
   - libsemanage-python
   - libselinux-python
   - ntp
   - firewalld
- name: Ensure we have last version of every package
   yum:
     name: "*"
     state: latest
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

它的`motd`模板位于`roles/common/templates/motd`：

```
                This system is managed by Ansible 
  Any change done on this system could be overwritten by Ansible 

OS: {{ ansible_distribution }} {{ ansible_distribution_version }} 
Hostname: {{ inventory_hostname }} 
eth0 address: {{ ansible_eth0.ipv4.address }} 

            All connections are monitored and recorded 
    Disconnect IMMEDIATELY if you are not an authorized user 

```

现在我们可以继续到`webserver`角色，更具体地说是到`roles/webserver/tasks/main.yaml`文件：

```
- name: Ensure the HTTPd package is installed 
  yum: 
    name: httpd 
    state: present 
  become: True 
- name: Ensure the PHP is installed 
  yum: 
    name: '{{ item }}'
    state: present 
  become: True 
  with_items:
 - git
 - php 
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

```

我们还需要在`roles/webserver/handlers/main.yaml`中创建处理程序，内容如下：

```
- name: Restart HTTPd 
  service: 
    name: httpd 
    state: restarted 
  become: True 

```

最后，我们需要修改`roles/webserver/files/website.conf`文件，暂时保持为空，但该文件必须存在。

现在我们可以配置几台 CentOS 机器（我配置了`ws01.fale.io`和`ws02.fale.io`）并确保库存是正确的。我们还可以运行`firstrun.yaml`剧本，以确保 Ansible 用户已经存在并且配置正确：

```
ansible-playbook -i inventory/production playbooks/firstrun.yaml

```

你应该收到的输出如下：

```
PLAY [localhost] ************************************************* 
PLAY [all] ******************************************************* 
TASK [setup] *****************************************************
ok: [ws01.fale.io]
ok: [ws02.fale.io] 
TASK [Ensure ansible user exists] ********************************
changed: [ws01.fale.io]
changed: [ws02.fale.io] 
TASK [Ensure ansible user accepts the SSH key] *******************
changed: [ws01.fale.io]
changed: [ws02.fale.io] 
TASK [Ensure the ansible user is sudoer with no password required]
changed: [ws01.fale.io]
changed: [ws02.fale.io] 
PLAY RECAP *******************************************************
ws01.fale.io      : ok=4    changed=3    unreachable=0    failed=0
ws02.fale.io      : ok=4    changed=3    unreachable=0    failed=0

```

现在我们可以通过运行它们的组剧本来配置这些机器：

```
ansible-playbook -i inventory/production playbooks/groups/web.yaml

```

我们将收到以下输出：

```
PLAY [web] ******************************************************* 
TASK [setup] *****************************************************
ok: [ws01.fale.io]
ok: [ws02.fale.io]
 ....
PLAY RECAP *******************************************************
ws01.fale.io      : ok=20   changed=14   unreachable=0    failed=0
ws02.fale.io      : ok=20   changed=14   unreachable=0    failed=0

```

现在我们可以将浏览器指向我们的节点端口`80`，检查 HTTPd 页面是否按预期显示。

# 使用修订控制系统部署 Web 应用程序

对于这个示例，我们将部署一个简单的 PHP 应用程序，它只由一个 PHP 页面组成。源代码可以在以下仓库中找到：https://github.com/Fale/demo-php-app。

为了部署它，我们需要将以下代码放入`playbooks/manual/rcs_deploy.yaml`中：

```
- hosts: web 
  user: ansible 
  tasks: 
  - name: Install or update website 
    git: 
      repo: https://github.com/Fale/demo-php-app.git 
      dest: /var/www/application 
    become: True 

```

现在我们可以使用以下命令运行**deployer**：

```
ansible-playbook -i inventory/production playbooks/manual/rcs_deploy.yaml

```

这是预期的结果：

```
PLAY [web] ******************************************************* 
TASK [setup] *****************************************************
ok: [ws01.fale.io]
ok: [ws02.fale.io]
 .... 
PLAY RECAP *******************************************************
ws01.fale.io      : ok=2    changed=1    unreachable=0    failed=0
ws02.fale.io      : ok=2    changed=1    unreachable=0    failed=0

```

目前，由于我们没有为该文件夹设置 HTTPd 规则，我们的应用程序尚不可访问。为此，我们需要更改`roles/webserver/files/website.conf`文件，内容如下：

```
<VirtualHost *:80> 
    ServerName app.fale.io 
    DocumentRoot /var/www/application 
    <Directory /var/www/application> 
        Options None 
    </Directory> 
    <DirectoryMatch ".git*"> 
        Require all denied 
    </DirectoryMatch> 
</VirtualHost> 

```

如你所见，我们只是将这个应用程序展示给通过`app.fale.io` URL 访问我们服务器的用户，而不是所有人。这将确保所有用户获得一致的体验。此外，你可以看到我们正在阻止所有访问`.git`文件夹（以及其中的所有内容）。这对于之前提到的安全原因是必要的。

现在我们可以重新运行 web 剧本，以确保我们的 HTTPd 配置得到了传播，命令如下：

```
ansible-playbook -i inventory/production playbooks/groups/web.yaml

```

这是我们将收到的结果：

```
PLAY [web] ******************************************************* 
TASK [setup] *****************************************************
ok: [ws01.fale.io]
ok: [ws02.fale.io] 
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
ok: [ws01.fale.io]
ok: [ws02.fale.io] 
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
ok: [ws01.fale.io]
ok: [ws02.fale.io] 
TASK [common : Ensure SSH can pass the firewall] *****************
ok: [ws01.fale.io]
ok: [ws02.fale.io] 
TASK [common : Ensure the MOTD file is present and updated] ******
ok: [ws01.fale.io]
ok: [ws02.fale.io] 
TASK [common : Ensure the hostname is the same of the inventory] *
ok: [ws01.fale.io]
ok: [ws02.fale.io] 
TASK [webserver : Ensure the HTTPd package is installed] *********
ok: [ws01.fale.io]
ok: [ws02.fale.io] 
TASK [webserver : Ensure the PHP is installed] *******************
ok: [ws01.fale.io]
ok: [ws02.fale.io] 
TASK [webserver : Ensure git is installed] ***********************
ok: [ws01.fale.io]
ok: [ws02.fale.io] 
TASK [webserver : Ensure the HTTPd service is enabled and running]
ok: [ws01.fale.io]
ok: [ws02.fale.io] 
TASK [webserver : Ensure HTTP can pass the firewall] *************
ok: [ws01.fale.io]
ok: [ws02.fale.io] 
TASK [webserver : Ensure HTTPd configuration is updated] *********
changed: [ws01.fale.io]
changed: [ws02.fale.io] 
RUNNING HANDLER [webserver : Restart HTTPd] **********************
changed: [ws01.fale.io]
changed: [ws02.fale.io] 
PLAY RECAP *******************************************************
ws01.fale.io      : ok=20   changed=2    unreachable=0    failed=0
ws02.fale.io      : ok=20   changed=2    unreachable=0    failed=0

```

您现在可以检查并确保一切正常运行。

# 使用 RPM 包部署 Web 应用

为了部署一个 RPM 包，我们首先需要创建它。为此，我们需要一个 Spec 文件。

## 创建一个 Spec 文件

首先要做的是创建一个 **Specifics**（**Spec**）文件，这是一个指导 `rpmbuild` 如何实际创建 RPM 包的配方。我们将把 Spec 文件放在 `spec/demo-php-app.spec` 中，并将以下内容放入其中：

```
%define debug_package %{nil} 
%global commit0 b49f595e023e07a8345f47a3ad62a6f50f03121e 
%global shortcommit0 %(c=%{commit0}; echo ${c:0:7}) 

Name:       demo-php-app 
Version:    0 
Release:    1%{?dist} 
Summary:    Demo PHP application 

License:    PD 
URL:        https://github.com/Fale/demo-php-app 
Source0:    %{url}/archive/%{commit0}.tar.gz#/%{name}-%{shortcommit0}.tar.gz 

%description 
This is a demo PHP application in RPM format 

%prep 
%autosetup -n %{name}-%{commit0} 

%build 

%install 
mkdir -p %{buildroot}/var/www/application 
ls -alh 
cp index.php %{buildroot}/var/www/application 

%files 
%dir /var/www/application 
/var/www/application/index.php 

%changelog 
* Tue Oct 04 2016 Fabio Alessandro Locati - 0.1 
- Initial packaging 

```

在继续之前，让我们先了解各部分的作用和含义：

```
%define debug_package %{nil} 
%global commit0 b49f595e023e07a8345f47a3ad62a6f50f03121e 
%global shortcommit0 %(c=%{commit0}; echo ${c:0:7}) 

```

这前三行是变量声明。

第一个将禁用调试包的生成。默认情况下，`rpmbuild` 每次都会创建调试包并包含所有调试符号，但在这种情况下，我们没有任何调试符号，因为我们没有进行任何编译。

第二个将提交的 **hash** 放入变量 `commit0` 中。第三个计算 `shortcommit0` 的值，这是 `commit0` 字符串的前八个字符：

```
Name:       demo-php-app 
Version:    0 
Release:    1%{?dist} 
Summary:    Demo PHP application 

License:    PD 
URL:        https://github.com/Fale/demo-php-app 
Source0:    %{url}/archive/%{commit0}.tar.gz#/%{name}-%{shortcommit0}.tar.gz 

```

在第一行，我们声明名称、版本、发布号和摘要。版本和发布号之间的区别在于，版本是上游版本，而发布号是该上游版本的 Spec 版本。

许可证是源码许可证，而不是 Spec 文件的许可证。URL 用于跟踪上游网站。`source0` 字段用于 `rpmbuild` 知道源文件的名称（如果有多个文件，我们可以使用 `source1`、`source2` 等）。此外，如果源字段是有效的 URI，我们可以使用 `spectool` 自动下载它们。

```
%description 
This is a demo PHP application in RPM format 

```

这是打包在 RPM 包中的软件的 `description`。

```
%prep 
%autosetup -n %{name}-%{commit0} 

```

`prep` 阶段是源码解压缩和可能的补丁应用的阶段。`%autosetup` 将会解压缩第一个源码，并应用所有补丁。在此部分，您还可以执行其他需要在构建阶段之前执行的操作，目的是为构建阶段准备环境：

```
%build 

```

在这里，我们将放置所有 `build` 阶段的操作。在我们的情况下，我们的源代码不需要编译，因此为空：

```
%install 
mkdir -p %{buildroot}/var/www/application 
ls -alh 
cp index.php %{buildroot}/var/www/application 

```

在 `install` 阶段，我们将文件放置在 `%{buildroot}` 文件夹中，该文件夹将模拟目标文件系统。

```
%files 
%dir /var/www/application 
/var/www/application/index.php 

```

`files` 部分用于声明要放入包中的文件。

```
%changelog 
* Tue Oct 04 2016 Fabio Alessandro Locati - 0.1 
- Initial packaging 

```

`changelog` 是用来追踪谁在什么时候发布了新版本以及有哪些变更的。

现在我们有了 Spec 文件，需要构建它。为此，我们可以使用生产机器，但这会增加该机器的攻击面，所以最好避免这种情况。有多种方式可以构建您的 RPM 软件。主要有四种方式：

+   手动地

+   使用 Ansible 自动化手动方式

+   Jenkins

+   Koji

让我们简要看一下它们的不同之处。

## 手动构建 RPM

构建 RPM 包的最简单方法是手动进行。

大的优势是你只需要非常少且易于安装的软件包，正因为如此，许多刚接触 RPM 的人都从这里开始。缺点是过程是手动的，因此人为错误可能会破坏结果，且该过程不容易审计。

要构建 RPM 包，你需要使用 Fedora 或 EL（Red Hat Enterprise Linux，CentOS，Scientific Linux，Oracle Enterprise Linux）系统。如果你使用的是 Fedora，你需要执行以下命令来安装所有必需的软件：

```
sudo dnf install -y fedora-packager

```

如果你正在运行 EL 系统，你需要执行的命令是：

```
sudo yum install -y mock rpm-build spectool

```

无论哪种情况，你都需要将你将使用的用户添加到 `mock` 组中，为此你需要执行：

```
sudo usermod -a -G mock [yourusername]

```

### 注意

Linux 在登录时加载用户，因此要应用组的更改，你需要重新启动会话。

此时，我们可以将 Spec 文件复制到文件夹中（通常 `$HOME` 是一个不错的选择），并执行以下操作：

```
mkdir -p ~/rpmbuild/SOURCES

```

这将创建过程所需的 `$HOME/rpmbuild/SOURCES` 文件夹。`-p` 选项会自动创建路径中缺失的所有文件夹。

```
spectool -R -g demo-php-app.spec

```

我们使用 `spectool` 下载源文件并将其放置在适当的目录中。`spectool` 会自动从 Spec 文件获取 URL，这样我们就不必记住它了。

我们现在需要创建一个 `src.rpm` 文件，为此我们可以使用 `rpmbuild`：

```
rpmbuild -bs demo-php-app.spec

```

这个命令将输出类似以下内容：

```
Wrote: /home/fale/rpmbuild/SRPMS/demo-php-app-0-1.fc24.src.rpm

```

可能存在一些小的差异，例如你可能会有不同的 `$HOME` 文件夹，并且如果你使用的不是 Fedora 24 来构建包，可能会有不同的 `fc24`。此时，我们可以使用以下命令创建二进制文件：

```
mock -r epel-7-x86_64 /home/fale/rpmbuild/SRPMS/demo-php-app-0-1.fc24.src.rpm

```

Mock 允许我们在一个干净的环境中构建 RPM 包，而且 thanks to `-r` 选项，它还允许我们为不同版本的 Fedora、EL 和 Mageia 构建。这个命令将给出非常长的输出，我这里不会列出所有内容，但在最后几行有有用的信息。如果一切构建成功，以下几行应该是你看到的最后内容：

```
Wrote: /builddir/build/RPMS/demo-php-app-0-1.el7.centos.x86_64.rpm
Executing(%clean): /bin/sh -e /var/tmp/rpm-tmp.d4vPhr
+ umask 022
+ cd /builddir/build/BUILD
+ cd demo-php-app-b49f595e023e07a8345f47a3ad62a6f50f03121e
+ /usr/bin/rm -rf /builddir/build/BUILDROOT/demo-php-app-0-1.el7.centos.x86_64
+ exit 0
Finish: rpmbuild demo-php-app-0-1.fc24.src.rpm
Finish: build phase for demo-php-app-0-1.fc24.src.rpm
INFO: Done(/home/fale/rpmbuild/SRPMS/demo-php-app-0-1.fc24.src.rpm) Config(epel-7-x86_64) 0 minutes 58 seconds
INFO: Results and/or logs in: /var/lib/mock/epel-7-x86_64/result
Finish: run

```

倒数第二行包含了你可以找到结果的路径。如果你查看该文件夹，你应该能找到以下文件：

```
drwxrwsr-x. 2 fale mock 4.0K Oct 10 12:26 .
drwxrwsr-x. 4 root mock 4.0K Oct 10 12:25 ..
-rw-rw-r--. 1 fale mock 4.6K Oct 10 12:26 build.log
-rw-rw-r--. 1 fale mock 3.3K Oct 10 12:26 demo-php-app-0-1.el7.centos.src.rpm
-rw-rw-r--. 1 fale mock 3.1K Oct 10 12:26 demo-php-app-0-1.el7.centos.x86_64.rpm
-rw-rw-r--. 1 fale mock 184K Oct 10 12:26 root.log
-rw-rw-r--. 1 fale mock  792 Oct 10 12:26 state.log

```

这三个日志文件在编译过程中出现问题时非常有用。`src.rpm` 文件将是我们用第一个命令创建的 `src.rpm` 文件的副本，而 `x86_64.rpm` 文件是 mock 创建的文件，也是我们需要安装到机器上的文件。

## 使用 Ansible 构建 RPM 包

由于手动执行所有这些步骤可能很繁琐、无聊且容易出错，我们可以使用 Ansible 自动化这些步骤。生成的 playbook 可能不是最干净的，但它能以可重复的方式执行所有操作。

因此，我们将从头开始构建一台新机器。我将这台机器命名为`builder01.fale.io`，我们还将更改库存/生产文件以匹配此更改：

```
[web] 
ws01.fale.io 
ws02.fale.io 

[db] 
db01.fale.io 

[builders] 
builder01.fale.io 

[production:children] 
db 
web 
builders 

```

在深入了解 `builders` 角色之前，我们需要对 `webserver` 角色做一些更改，以启用新的仓库。第一个更改是在文件末尾的 `roles/webserver/tasks/main.yaml` 中添加一个任务，代码如下：

```
- name: Install our private repository 
  copy: 
    src: privaterepo.repo 
    dest: /etc/yum.repos.d/privaterepo.repo 
  become: True 

```

第二个更改是实际创建 `roles/webserver/files/privaterepo.repo` 文件，内容如下：

```
[privaterepo] 
name=Private repo that will keep our apps packages 
baseurl=http://repo.fale.io/ 
skip_if_unavailable=True 
gpgcheck=0 
enabled=1 
enabled_metadata=1 

```

我们现在可以执行 `webserver` 组的 playbook 以使更改生效，命令如下：

```
ansible-playbook -i inventory/production playbooks/groups/web.yaml

```

以下输出应该会出现：

```
PLAY [web] ******************************************************* 
TASK [setup] *****************************************************
ok: [ws01.fale.io]
ok: [ws02.fale.io]
    ....
PLAY RECAP ******************************************************
ws01.fale.io       : ok=20   changed=1    unreachable=0    failed=0
ws02.fale.io       : ok=20   changed=1    unreachable=0    failed=0

```

如预期的那样，唯一的变化是我们新生成的仓库文件的部署。

我们还需要为 `builders` 创建一个角色，并在 `roles/builder/tasks/main.yaml` 中创建一个 `tasks` 文件，内容如下：

```
- name: Ensure needed packages are present 
  yum: 
    name: '{{ item }}' 
    state: present 
  become: True 
  with_items: 
  - mock 
  - rpm-build 
  - spectool 
  - createrepo 
  - httpd 

- name: Ensure the user ansible is in the mock group 
  user: 
    name: ansible 
    groups: mock 
    append: True 
  become: True 

- name: Ensure the /var/www/repo folder is present 
  file: 
    name: /var/www/repo 
    state: directory 
    group: ansible 
    owner: ansible 
    mode: 0755 
  become: True 

- name: Ensure the HTTPd zone for the repo is present 
  copy: 
    src: repo.conf 
    dest: /etc/httpd/conf.d/repo.conf 
  become: True 
  notify: Restart HTTPd 

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

```

另外，作为 `builders` 角色的一部分，我们需要 `roles/builder/handlers/main.yaml` 处理器文件，内容如下：

```
- name: Restart HTTPd 
  service: 
    name: httpd 
    state: restarted 
  become: True 

```

正如你从任务文件中可以猜到的那样，我们还需要 `roles/builder/files/repo.conf` 文件，内容如下：

```
<VirtualHost *:80> 
    ServerName repo.fale.io 
    DocumentRoot /var/www/repo 
    <Directory /var/www/repo> 
        Options Indexes FollowSymLinks 
    </Directory> 
</VirtualHost> 

```

我们还需要在 `playbooks/groups/builders.yaml` 中创建一个新的 `group` playbook，内容如下：

```
- hosts: builders 
  user: ansible 
  roles: 
  - common 
  - builder 

```

我们现在可以执行 `firstrun` playbook，命令如下：

```
ansible-playbook -i inventory/production playbooks/firstrun.yaml -lbuilder01.fale.io

```

我们将收到如下输出：

```
PLAY [all] ******************************************************* 
TASK [setup] *****************************************************
ok: [builder01.fale.io] 
TASK [Ensure ansible user exists] ********************************
changed: [builder01.fale.io] 
TASK [Ensure ansible user accepts the SSH key] *******************
changed: [builder01.fale.io] 
TASK [Ensure the ansible user is sudoer with no password required]
changed: [builder01.fale.io] 
PLAY RECAP *******************************************************
builder01.fale.io : ok=4    changed=3    unreachable=0    failed=0

```

我们现在可以继续创建主机，命令如下：

```
ansible-playbook -i inventory/production playbooks/groups/builders.yaml

```

我们预期的结果类似于：

```
PLAY [builders] ************************************************** 
TASK [setup] *****************************************************
ok: [builder01.fale.io]
 .... 
PLAY RECAP *******************************************************
builder01.fale.io : ok=23   changed=5    unreachable=0    failed=0

```

现在所有基础设施部分都已准备好，我们可以创建 `playbooks/manual/rpm_deploy.yaml`，内容如下：

```
- hosts: builders 
  user: ansible 
  tasks: 
  - name: Copy Spec file to user folder 
    copy: 
      src: ../../spec/demo-php-app.spec 
      dest: /home/ansible 
  - name: Ensure rpmbuild exists 
    file: 
      name: ~/rpmbuild 
      state: directory 
  - name: Ensure rpmbuild/SOURCES exists 
    file: 
      name: ~/rpmbuild/SOURCES 
      state: directory 
  - name: Download the sources 
    command: spectool -R -g demo-php-app.spec 
  - name: Ensure no SRPM files are present 
    command: rm -f ~/rpmbuild/SRPMS/* 
  - name: Build the SRPM file 
    command: rpmbuild -bs demo-php-app.spec 
  - name: Execute mock 
    shell: mock ~/rpmbuild/SRPMS/* 
  - name: Copy the arch binaries in the repo path 
    shell: cp -f /var/lib/mock/epel-7-x86_64/result/*.x86_64.rpm /var/www/repo 
  - name: Recreate the repo metadata 
    command: createrepo --database /var/www/repo 
- hosts: web 
  user: ansible 
  tasks: 
  - name: Ensure last version of demo-php-app is present 
    yum: 
      state: latest 
      update_cache: True 
      disable_gpg_check: True 
      name: demo-php-app 
    become: True 

```

如前所述，这个 playbook 包含了很多命令和 shell，可能会显得不太整洁。未来可能会有机会写出一个功能相同但使用模块的 playbook。大部分操作和我们在前一节中讨论的相同。新的操作是在后面，实际上，在这个案例中，我们将生成的 RPM 文件复制到一个特定文件夹中，使用 `createrepo` 在该文件夹中生成一个仓库，然后强制所有 Web 服务器更新到最新版本的生成包。

### 注意

为了确保应用程序的安全，重要的是仓库只能在内部访问，而不能公开访问。

我们现在可以使用以下命令运行 playbook：

```
ansible-playbook -i inventory/production playbooks/manual/rpm_deploy.yaml

```

我们预期的结果如下所示：

```
PLAY [builders] ************************************************** 
TASK [setup] *****************************************************
ok: [builder01.fale.io] 
TASK [Copy SPEC file to user folder] *****************************
changed: [builder01.fale.io] 
TASK [Ensure rpmbuild exists] ************************************
changed: [builder01.fale.io] 
TASK [Ensure rpmbuild/SOURCES exists] ****************************
changed: [builder01.fale.io] 
TASK [Download the sources] **************************************
changed: [builder01.fale.io] 
TASK [Ensure no SRPM files are present] **************************
changed: [builder01.fale.io] 
TASK [Build the SRPM file] ***************************************
changed: [builder01.fale.io] 
TASK [Execute mock] **********************************************
changed: [builder01.fale.io] 
TASK [Copy the arch binaries in the repo path] *******************
changed: [builder01.fale.io] 
TASK [Recreate the repo metadata] ********************************
changed: [builder01.fale.io] 
PLAY [web] ******************************************************* 
TASK [setup] *****************************************************
ok: [ws01.fale.io]
ok: [ws02.fale.io] 
TASK [Update all packages] ***************************************
changed: [ws01.fale.io]
changed: [ws02.fale.io] 
PLAY RECAP *******************************************************
builder01.fale.io : ok=10   changed=9    unreachable=0    failed=0
ws01.fale.io      : ok=2    changed=1    unreachable=0    failed=0
ws02.fale.io      : ok=2    changed=1    unreachable=0    failed=0

```

## 使用 CI/CD 管道构建 RPM 包

尽管本书未涉及这一部分，但在更复杂的情况下，你可能希望使用 CI/CD 管道来创建和管理 RPM 包。主要的两种管道基于两个不同的软件：Jenkins 和 Koji。

Koji 软件由 Fedora 社区和 Red Hat 开发。它根据 LGPL 2.1 许可协议发布。这是目前 Fedora、CentOS 以及许多其他公司和社区用来创建所有 RPM 包的管道（无论是官方版本还是测试版——即**临时构建**）。Koji 默认不会通过提交触发，而是需要用户通过 web 界面或命令行界面“手动”调用。Koji 将自动下载最新版本的 Spec Git，从侧缓存中下载源代码（这一步是可选的，但推荐）或从原始位置下载，并触发 mock 构建。由于 mock 是唯一允许一致和可重复构建的系统，Koji 仅支持 mock。Koji 可以根据配置永久或在有限时间内存储所有输出工件。这是为了确保很高的审计能力。

Jenkins 是最常用的 CI/CD 管理工具之一，也可以用于 RPM 管道。它的一个大缺点是需要从头开始配置，这意味着需要更多时间，但也因此具有更多的灵活性。Jenkins 的另一个大优势是许多公司已经有了 Jenkins 实例，这使得搭建和维护基础设施变得更加容易，因为您可以重用已有的安装，整体上减少了系统管理的工作。

# 使用 RPM 打包构建已编译的软件

RPM 打包对于非二进制应用程序非常有用，对于二进制应用程序几乎是必需的。这也是因为非二进制和二进制的复杂性差异非常小。事实上，构建和安装的方式完全相同。唯一的区别是 Spec 文件。

让我们来看一个例子，查看需要编译和打包一个简单的用 C 编写的 Hello World! 应用程序所需的 Spec 文件：

```
%global commit0 7c288b9d80a6ef525c0cca8a744b32e018eaa386 
%global shortcommit0 %(c=%{commit0}; echo ${c:0:7}) 

Name:           hello-world 
Version:        1.0 
Release:        1%{?dist} 
Summary:        Hello World example implemented in C 

License:        GPLv3+ 
URL:            https://github.com/Fale/hello-world 
Source0:        %{url}/archive/%{commit0}.tar.gz#/%{name}-%{shortcommit0}.tar.gz 

BuildRequires:  gcc 
BuildRequires:  make 

%description 
The description for our Hello World Example implemented in C 

%prep 
%autosetup -n %{name}-%{commit0} 

%build 
make %{?_smp_mflags} 

%install 
%make_install 

%files 
%license LICENSE 
%{_bindir}/hello 

%changelog 
* Tue Oct 11 2016 Fabio Alessandro Locati - 1.0-1 
- Initial packaging 

```

如您所见，它与我们为 PHP 演示应用程序看到的非常相似。让我们看看其中的区别。

```
%global commit0 7c288b9d80a6ef525c0cca8a744b32e018eaa386 
%global shortcommit0 %(c=%{commit0}; echo ${c:0:7}) 

```

如您所见，我们没有禁用调试包的行。每次打包一个已编译的应用程序时，您应该让 `rpm` 创建调试符号包，这样在发生崩溃时，调试和理解问题会更容易。

```
Name:           hello-world 
Version:        1.0 
Release:        1%{?dist} 
Summary:        Hello World example implemented in C 

License:        GPLv3+ 
URL:            https://github.com/Fale/hello-world 
Source0:        %{url}/archive/%{commit0}.tar.gz#/%{name}-%{shortcommit0}.tar.gz 

```

如您所见，本节的变化仅仅是由于新包的名称和 `URL` 不同，但与是否是可编译的应用程序无关。

```
BuildRequires:  gcc 
BuildRequires:  make 

```

在非编译应用程序中，我们在构建时不需要任何包，而在这种情况下，我们将需要 make 和 `gcc`（编译器）等应用程序。不同的应用程序可能需要不同的工具和/或库在构建时存在于系统中。

```
%description 
The description for our Hello World Example implemented in C 

%prep 
%autosetup -n %{name}-%{commit0} 

%build 
make %{?_smp_mflags} 

```

`description` 是特定于包的，并不受包编译的影响。类似地，`%prep` 阶段也是如此。

在`%build`阶段，我们现在有了`make %{?_smp_mflags}`。这是为了告诉`rpmbuild`实际运行 make 来构建我们的应用程序。`_smp_mflags`变量将包含一组参数，用于优化编译以支持多线程。

```
%install 
%make_install 

```

在`%install`阶段，我们将执行`%make_install`命令。这个宏将使用一组附加参数调用`%make_install`，以确保**库文件**位于正确的文件夹中，二进制文件等也会被正确处理。

```
%files 
%license LICENSE 
%{_bindir}/hello 

```

在这种情况下，我们只需要将`hello`二进制文件放在`%install`阶段位于`buildroot`的正确文件夹中，并添加包含许可证的`LICENSE`文件。

```
%changelog 
* Tue Oct 11 2016 Fabio Alessandro Locati - 1.0-1 
- Initial packaging 

```

`%changelog`与我们之前看到的其他 Spec 文件非常相似，因为它不受编译过程的影响。

完成后，你可以将其放入`spec/hello-world.spec`中，并调整`playbooks/manual/rpm_deploy.yaml`，将其保存为`playbooks/manual/hello_deploy.yaml`，内容如下：

```
- hosts: builders 
  user: ansible 
  tasks: 
  - name: Copy Spec file to user folder 
    copy: 
      src: ../../spec/hello-world.spec 
      dest: /home/ansible 
  - name: Ensure rpmbuild exists 
    file: 
      name: ~/rpmbuild 
      state: directory 
  - name: Ensure rpmbuild/SOURCES exists 
    file: 
      name: ~/rpmbuild/SOURCES 
      state: directory 
  - name: Download the sources 
    command: spectool -R -g hello-world.spec 
  - name: Ensure no SRPM files are present 
    command: rm -f ~/rpmbuild/SRPMS/* 
  - name: Build the SRPM file 
    command: rpmbuild -bs hello-world.spec 
  - name: Execute mock 
    shell: mock ~/rpmbuild/SRPMS/* 
  - name: Copy the arch binaries in the repo path 
    shell: cp -f /var/lib/mock/epel-7-x86_64/result/*.x86_64.rpm /var/www/repo 
  - name: Recreate the repo metadata 
    command: createrepo --database /var/www/repo 
- hosts: web 
  user: ansible 
  tasks: 
  - name: Ensure last version of hello-world is present 
    yum: 
      state: latest 
      update_cache: True 
      disable_gpg_check: True 
      name: hello-world 
    become: True 

```

如你所见，我们唯一改变的就是所有对`demo-php-app`的引用都被替换为`hello-world`。运行它时：

```
ansible-playbook -i inventory/production playbooks/manual/hello_deploy.yaml
We are going to have the following result:
PLAY [builders] ************************************************** 
TASK [setup] *****************************************************
ok: [builder01.fale.io] 
TASK [Copy SPEC file to user folder] *****************************
changed: [builder01.fale.io] 
TASK [Ensure rpmbuild exists] ************************************
ok: [builder01.fale.io] 
TASK [Ensure rpmbuild/SOURCES exists] ****************************
ok: [builder01.fale.io] 
TASK [Download the sources] **************************************
changed: [builder01.fale.io] 
TASK [Ensure no SRPM files are present] **************************
changed: [builder01.fale.io] 
TASK [Build the SRPM file] ***************************************
changed: [builder01.fale.io] 
TASK [Execute mock] **********************************************
changed: [builder01.fale.io] 
TASK [Copy the arch binaries in the repo path] *******************
changed: [builder01.fale.io] 
TASK [Recreate the repo metadata] ********************************
changed: [builder01.fale.io] 
PLAY [web] ******************************************************* 
TASK [setup] *****************************************************
ok: [ws01.fale.io]
ok: [ws02.fale.io] 
TASK [Ensure last version of hello-world is present] *************
changed: [ws01.fale.io]
changed: [ws02.fale.io] 
PLAY RECAP *******************************************************
builder01.fale.io : ok=10   changed=7    unreachable=0    failed=0
ws01.fale.io      : ok=2    changed=1    unreachable=0    failed=0
ws02.fale.io      : ok=2    changed=1    unreachable=0    failed=0

```

### 注意

你最终可以创建一个接受要构建的包名称作为参数的剧本，这样你就不需要为每个包创建不同的剧本了。

# 部署策略

我们已经看到如何在你的环境中分发软件，现在让我们谈谈部署策略；也就是说，如何在不影响服务的情况下升级你的应用程序。

在更新过程中，你可能会遇到三种不同的问题：

+   更新发布期间的停机时间

+   新版本存在问题

+   新版本似乎能正常工作，直到它失败。

第一个问题是每个系统管理员都知道的。在更新过程中，你可能会重启一些服务，在服务停止和启动的这段时间内，你的应用程序将无法在该机器上使用。避免这种情况意味着你的应用程序根本无法使用；你需要至少有一些机器上能使用应用程序，并在前面放置一个智能负载均衡器，负责移除（并在合适时重新添加）所有无法正常工作的节点。

第二个问题可以通过多种方式来防止。最干净的方法是通过 CI/CD 管道进行测试。事实上，这类问题通过简单的测试就很容易发现。接下来我们将看到的方法也能防止这种情况发生。

第三个问题是迄今为止最复杂的。很多大的故障都由这种问题引起。通常，问题是新版本有一些性能问题或内存泄漏。由于大多数部署是在服务器负载最轻的时段进行的，一旦负载增加，性能问题或内存泄漏可能会导致服务器崩溃。

### 注意

为了能够正确使用这些方法，你必须确保软件能够接受回滚。有些情况可能无法做到（例如，更新中删除了数据库表），但应尽量避免。我们不会讨论如何避免这些问题，因为它是开发策略的一部分，与 Ansible 无关。

## 金丝雀发布

金丝雀发布是一种技术，涉及将少量机器（通常是 5%）更新到新版本，并指示负载均衡器只将相应数量的流量发送给它。这有几个优点：

+   在更新期间，你的容量永远不会低于 95%。

+   如果新版本完全失败，你将失去 5% 的容量

+   由于负载均衡器将流量分配到新旧版本之间，如果新版本出现问题，只有 5% 的用户会遇到问题。

+   你只需要比预期负载多 5% 的容量。

金丝雀发布能够以非常小的开销（5%）和低成本（5%）防止我们提到的三种问题。如果回滚发生时，这一成本也是很低的。正因如此，大公司常常采用这种技术；逐步发布。为了确保地理上靠近的用户有类似的用户体验，通常会使用地理位置来决定用户是访问旧版本还是新版本。

当测试显示成功时，可以逐步增加百分比，直到达到 100%。

在 Ansible 中，可以通过多种方式实现金丝雀发布。我建议的方式是最简洁的；使用清单文件，更具体地说，像下面这样：

```
[web-main] 
ws[00:94].fale.io 

[web-canary] 
ws[95:99].fale.io 

[web:children] 
web-main 
web-canary 

```

通过这种方式，你可以在 Web 组上设置所有变量（这些变量在版本间应该是相同的，或者至少应该是），但是你可以轻松地对金丝雀组、主组或两个组同时运行 playbook。另一个选择是创建两个不同的清单文件，一个用于金丝雀组，另一个用于主组，这样两个组的名字相同，以便共享变量。

## 蓝绿发布

**蓝绿**发布与金丝雀发布有很大的不同，且各自有优缺点。主要的优点包括：

+   更容易实现

+   允许更快速的迭代

+   所有用户同时迁移

+   回滚不会导致性能下降。

缺点中，主要的问题是你需要比应用程序要求的机器数多出一倍。这一缺点如果应用程序运行在云环境（无论是私有云、公共云还是混合云）上，通过扩展应用资源以进行部署，然后再缩减，就可以轻松缓解。

在 Ansible 中实现蓝绿部署非常简单。最简单的方法是创建两个不同的清单（一个用于蓝色环境，一个用于绿色环境），然后像管理不同的环境（如生产环境、预生产环境、开发环境等）一样简单地管理你的基础设施。

# 优化

有时，Ansible 可能感觉很慢，尤其是当你有一个非常长的任务列表需要执行，和/或有大量的机器时。其实这种感觉不仅仅是一种感觉，背后有多种原因，我们将探讨其中的三种以及如何避免它们。

## 管道化

Ansible 默认较慢的原因之一是，对于每个模块执行和每个主机，Ansible 会执行以下操作：

+   SSH 握手

+   执行任务

+   关闭 SSH 连接

正如你所看到的，这意味着如果你有 10 个任务需要在单个远程服务器上执行，Ansible 会打开（并关闭）连接 10 次。由于 SSH 协议是加密协议，这使得 SSH 握手过程变得更加漫长，因为每次都需要重新协商加密算法。

Ansible 通过在 playbook 开始时初始化连接并保持连接在整个执行过程中的活跃，从而大幅度减少了执行时间，这样就不需要在每个任务时重新打开连接。在 Ansible 的发展过程中，这个功能的名称已经多次更改，以及启用它的方式也发生了变化。从 1.5 版本开始，它被称为 **管道化（pipelining）**，启用它的方法是向 `ansible.cfg` 文件中添加以下行：

```
pipelining=True 

```

这个功能默认未启用的原因是，许多发行版在 `sudo` 中启用了 `requiretty` 选项。Ansible 中的管道化模式和 `sudo` 中的 `requiretty` 选项存在冲突，会导致 playbook 执行失败。

### 提示

如果你想启用管道模式，请确保目标机器上禁用了 `sudo requiretty` 模式。

## 使用 `with_items` 进行优化

如果你想执行相似的操作多次，可以使用不同的参数重复相同的任务，或者使用 `with_items` 选项。除了 `with_items` 让你的代码更易读、更易理解之外，它还可能提升你的性能。例如，在安装软件包时（即：`apt`、`dnf`、`yum`、`package` 模块），如果你使用 `with_items`，Ansible 会执行一个命令，而如果不使用，它会为每个包执行一个命令。正如你所想象的那样，这可以帮助提升性能。

## 理解任务执行时发生的事情

即使你已经实现了我们刚才讨论的加速 playbook 执行的方法，你仍然可能发现某些任务需要非常长的时间。这对于某些任务来说是很常见的，即使许多其他模块是可能加速的。通常会出现这个问题的模块有以下几种：

+   包管理（即：`apt`、`dnf`、`yum`、`package`）

+   云机器创建（例如：DigitalOcean，EC2）

这种慢速的原因通常不是 Ansible 特有的。一个例子是，如果你使用包管理模块来更新机器。这需要在每台机器上下载数十或数百兆字节的数据，并安装大量的软件。加速这种操作的方法是，在数据中心建立一个本地仓库，并让所有机器指向它，而不是指向分发仓库。这样可以让你的机器以更快的速度下载，而且不会使用常常带宽受限或按流量计费的公共连接。

### 注意

理解模块在后台执行的操作通常很重要，这有助于优化 playbook 的执行。

在云机器创建的案例中，Ansible 仅执行一次 API 调用到选择的云服务提供商，并等待机器准备好。DigitalOcean 的机器创建可能需要最长一分钟（其他云平台可能更长），因此 Ansible 会等待这一时间。一些模块具有异步模式以避免等待时间，但你必须确保机器准备好才能使用它，否则使用该机器的模块将会失败。

# 总结

在本章中，我们已经看到如何使用 Ansible 部署应用程序，以及可以使用的各种分发和部署策略。我们还了解了如何通过 Ansible 创建 RPM 包，并通过不同的方法优化 Ansible 的性能。

在最后一章，我们将讨论 Ansible 在 Windows 上的使用以及网络设备的管理。此外，还会讨论一些 Ansible Tower 的概念。
