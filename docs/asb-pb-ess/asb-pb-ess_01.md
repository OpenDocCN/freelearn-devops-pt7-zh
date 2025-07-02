# 设置学习环境

为了最有效地使用本书并检查、运行和编写书中提供的练习代码，设置学习环境是必要的。虽然 Ansible 可以与任何类型的节点、虚拟机、云服务器或已安装操作系统并运行 SSH 服务的裸机主机配合使用，但首选的模式是使用虚拟机。

在本节中，我们将覆盖以下主题：

+   理解学习环境

+   理解先决条件

+   安装和配置 VirtualBox 和 Vagrant

+   创建虚拟机

+   安装 Ansible

+   使用示例代码

# 理解学习环境

我们假设大多数学习者希望在本地设置环境，因此建议使用开源且免费提供的软件 VirtualBox 和 Vagrant，这些软件支持大多数桌面操作系统，包括 Windows、OSX 和 Linux。

理想的设置包括五台虚拟机，其目的如下所述。你也可以合并一些服务，例如，负载均衡器和 Web 服务器可以是同一主机：

+   **控制器**：这是唯一需要安装 Ansible 的主机，充当控制器。它用于从控制器启动 `ansible-playbook` 命令。

+   **数据库（Ubuntu）**：此主机配置了 Ansible 以运行 MySQL 数据库服务，且运行 Ubuntu 发行版的 Linux。

+   **数据库（CentOS）**：此主机配置了 Ansible 以运行 MySQL 数据库服务，但它运行的是 CentOS 发行版的 Linux。此配置用于测试在编写 MySQL 角色时的多平台支持。

+   **Web 服务器**：此主机配置了 Ansible 以运行 Apache Web 服务器应用程序。

+   **负载均衡器**：此主机配置了 haproxy 应用程序，它是一个开源的 HTTP 代理服务。此主机充当负载均衡器，接受 HTTP 请求并将负载分配到可用的 Web 服务器上。

## 先决条件

有关先决条件、软件和硬件要求以及设置说明的最新指引，请参阅以下 GitHub 仓库：

[`github.com/schoolofdevops/ansible-playbook-essentials`](https://github.com/schoolofdevops/ansible-playbook-essentials)

### 系统先决条件

一台适度配置的台式机或笔记本电脑系统应该足够用来设置学习环境。以下是软件和硬件方面的推荐先决条件：

| **处理器** | 2 核 |
| --- | --- |
| **内存** | 可用 2.5 GB RAM |
| **磁盘空间** | 20 GB 可用空间 |
| **操作系统** | Windows、OS X（Mac）、Linux |

## 基础软件

为了设置学习环境，我们建议使用以下软件：

+   **VirtualBox**：Oracle 的 VirtualBox 是一款桌面虚拟化软件，可以免费使用。它支持多种操作系统，包括 Windows、OS X、Linux、FreeBSD、Solaris 等。它提供了一个虚拟机管理程序层，并允许用户在现有操作系统上创建并运行虚拟机。与本书一起提供的代码已在 VirtualBox 4.3x 版本上进行了测试。然而，任何与 Vagrant 版本兼容的 VirtualBox 版本都可以使用。

+   **Vagrant**：这是一款工具，允许用户在大多数虚拟机管理程序和云平台上轻松创建和共享虚拟环境，包括但不限于 VirtualBox。它可以自动化任务，如导入镜像、指定虚拟机的资源（如内存和 CPU）以及设置网络接口、主机名、用户凭证等。由于它以 Vagrant 文件的文本配置形式提供，虚拟机可以通过编程方式进行配置，从而使它们能够与其他工具（如 **Jenkins**）结合使用，以自动化构建和测试管道。

+   **Git for Windows**：尽管我们不打算使用 Git（一款版本控制软件），但我们使用此软件来在 Windows 系统上安装 SSH 工具。Vagrant 需要在路径中提供 SSH 二进制文件。Windows 系统并不自带 SSH 工具，而 Git for Windows 是在 Windows 上安装 SSH 工具的最简便方式。也有其他选项，如 **Cygwin**。

下表列出了用于开发本书附带代码的软件的操作系统版本及其下载链接：

| 软件 | 版本 | 下载链接 |
| --- | --- | --- |
| VirtualBox | 4.3.30 | [`www.virtualbox.org/wiki/Downloads`](https://www.virtualbox.org/wiki/Downloads) |
| Vagrant | 1.7.3 | [`www.vagrantup.com/downloads.html`](https://www.vagrantup.com/downloads.html) |
| Git for Windows | 1.9.5 | [`git-scm.com/download/win`](https://git-scm.com/download/win) |

建议学习者在继续操作之前，先下载、安装并参考相关文档页面，以便熟悉这些工具。

## 创建虚拟机

一旦安装了基础软件，你可以使用 Vagrant 来启动所需的虚拟机。Vagrant 使用名为 `Vagrantfile` 的配置文件，以下是一个示例：

```
# -*- mode: ruby -*-
# vi: set ft=ruby :
# Sample Vagranfile to setup Learning Environment
# for Ansible Playbook Essentials

VAGRANTFILE_API_VERSION = "2"
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "ansible-ubuntu-1204-i386"
  config.vm.box_url = "https://cloud-images.ubuntu.com/vagrant/precise/current/precise-server-cloudimg-i386-vagrant-disk1.box"
  config.vm.define "control" do |control|
    control.vm.network :private_network, ip: "192.168.61.10"
  end
  config.vm.define "db" do |db|
    db.vm.network :private_network, ip: "192.168.61.11"
  end
  config.vm.define "dbel" do |db|
    db.vm.network :private_network, ip: "192.168.61.14"
    db.vm.box = "opscode_centos-6.5-i386"
    db.vm.box = "http://opscode-vm-bento.s3.amazonaws.com/vagrant/virtualbox/opscode_centos-6.5_chef-provisionerless.box"
  end
  config.vm.define "www" do |www|
    www.vm.network :private_network, ip: "192.168.61.12"
  end
  config.vm.define "lb" do |lb|
    lb.vm.network :private_network, ip: "192.168.61.13"
  end
end
```

上述 Vagrant 文件包含了设置五个虚拟机的配置，正如本章开头所描述的，分别是 `control`、`db`、`dbel`、`www` 和 `lb`。

建议学习者按照以下说明创建并启动所需的虚拟机，以设置学习环境：

1.  在系统上的任意位置创建一个学习环境设置的目录结构，例如 `learn/ansible`。

1.  将之前提供的 `Vagrantfile` 文件复制到 `learn/ansible` 目录中。目录结构应如下所示：

    ```
                         learn
                            \_ ansible
                                  \_ Vagrantfile
    ```

    ### 注意

    `Vagrantfile` 文件包含前面章节中描述的虚拟机的规范。

1.  打开终端并进入 `learn/ansible` 目录。

1.  启动控制节点并登录，操作如下：

    ```
    $ vagrant up control 
    $ vagrant ssh control

    ```

1.  在一个单独的终端窗口中，从 `learn/ansible` 目录，依次启动剩余的虚拟机，操作如下：

    ```
    $ vagrant up db
    $ vagrant up www
    $ vagrant up lb
    optionally (for centos based mysql configurations)
    $ vagrant up dbel 
    Optionally, to login to to the virtual machines as
    $ vagrant ssh db
    $ vagrant ssh www
    $ vagrant ssh lb
    optionally (for centos based mysql configurations)
    $ vagrant ssh dbel 

    ```

## 在控制器上安装 Ansible

一旦虚拟机创建并启动，Ansible 需要在控制器上安装。由于 Ansible 是无代理的，通过 SSH 传输管理节点，因此除了确保 SSH 服务正在运行之外，不需要在节点上进行额外的设置。要在控制器上安装 Ansible，请参考以下步骤。这些说明特定于我们在控制器上使用的 Ubuntu Linux 发行版。有关通用安装说明，请参考以下页面：

[`docs.ansible.com/intro_installation.html`](http://docs.ansible.com/intro_installation.html)

步骤如下：

1.  使用以下命令登录到控制器：

    ```
    # from inside learn/ansible directory 
    $ vagrant ssh control 

    ```

1.  使用以下命令更新仓库缓存：

    ```
    $ sudo apt-get update

    ```

1.  安装必要的软件和仓库：

    ```
    # On Ubuntu 14.04 and above 
    $ sudo apt-get install -y software-properties-common
    $ sudo apt-get install -y python-software-properties
    $ sudo apt-add-repository ppa:ansible/ansible

    ```

1.  添加新仓库后，使用以下命令更新仓库缓存：

    ```
    $ sudo apt-get update 

    ```

1.  使用以下命令安装 Ansible：

    ```
    $ sudo apt-get install -y ansible 

    ```

1.  使用以下命令验证 Ansible：

    ```
    $ ansible --version
    [sample output]
    vagrant@vagrant:~$ ansible --version
    ansible 1.9.2
     configured module search path = None

    ```

## 使用示例代码

本书提供的示例代码按章节编号进行划分。以章节编号命名的目录中包含该章节末尾代码状态的快照。建议学习者独立编写代码，并将示例代码作为参考。此外，如果读者跳过一个或多个章节，他们可以使用上一章节的示例代码作为基础。

例如，在使用 第六章，*迭代控制结构 – 循环* 时，您可以使用 第五章，*控制执行流 – 条件语句* 的示例代码作为基础。

### 提示

**下载示例代码**

您可以从您的账户下载您购买的所有 Packt 书籍的示例代码文件，网址为 [`www.packtpub.com`](http://www.packtpub.com)。如果您在其他地方购买了本书，可以访问 [`www.packtpub.com/support`](http://www.packtpub.com/support)，注册后，您将直接通过电子邮件接收文件。
