# 前言

信息技术行业是一个快速发展的领域，总是试图加速。为了跟上这一变化，企业需要能够快速行动并频繁迭代。直到几年前，这主要适用于软件，但现在我们开始看到需要以类似的速度改变基础设施。未来，我们将需要以软件本身的速度来改变我们运行软件的基础设施。

在这种情况下，许多技术，如软件定义的一切（存储、网络、计算等），将是关键，但这些技术也需要以同样可扩展的方式进行管理，而这种方式就是使用 Ansible 和类似的产品。

Ansible 在今天非常相关，因为与竞争产品不同，它是无代理的，这使得部署更快、安全性更高，并且审计能力更强。

# 本书涵盖内容

第一章，*Ansible 入门*，解释了如何安装 Ansible。

第二章，*自动化简单任务*，解释了如何创建简单的 playbook，帮助你自动化日常已经执行的一些简单任务。

第三章，*扩展到多主机*，解释了如何以易于扩展的方式在 Ansible 中处理多个主机。

第四章，*处理复杂部署*，解释了如何创建具有多个阶段和多台机器的部署。

第五章，*上云*，解释了 Ansible 如何与各种云服务集成，以及它如何简化你的工作，自动化管理云服务。

第六章，*从 Ansible 获取通知*，解释了如何设置 Ansible 返回有价值的信息给你和其他相关方。

第七章，*创建自定义模块*，解释了如何创建自定义模块，以利用 Ansible 提供的灵活性。

第八章，*调试和错误处理*，解释了如何调试和测试 Ansible，以确保你的 playbook 始终能够正常运行。

第九章，*复杂环境*，解释了如何使用 Ansible 管理多层级、多环境和多部署。

第十章，*企业级 Ansible 入门*，解释了如何通过 Ansible 管理 Windows 节点，并如何利用 Ansible Galaxy 和 Ansible Tower 最大化你的生产力。

# 本书所需工具

本书适用于所有 Linux 发行版。由于为所有可能的发行版提供相同的信息并不实际，示例命令默认适用于控制机上的 Fedora 和受控机器上的 CentOS（如果没有特别说明）。有经验的用户可以轻松将命令转换为自己喜欢的发行版。

# 本书适用对象

本书面向希望使用 Ansible 2 自动化组织基础设施的开发人员和系统管理员。无需提前了解 Ansible。

# 约定

在本书中，您会看到多种文本样式，用来区分不同类型的信息。以下是一些样式示例及其含义的解释。

文本中的代码词、数据库表名、文件夹名称、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名会以如下方式显示：“`ec2.py`文件将根据区域、可用区、标签等创建多个组。”

代码块的格式如下：

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
```

当我们希望引起您对代码块中特定部分的注意时，相关行或项目会以粗体显示：

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

```

任何命令行输入或输出都如下所示：

```
$ ansible-playbook -i test01.fale.io, webserver.yaml

```

**新术语**和**重要词汇**以粗体显示。您在屏幕上看到的词汇，例如菜单或对话框中的内容，会像这样出现在文本中：“点击‘下一步’按钮将带您进入下一屏。”

### 注意

警告或重要提示会以类似这样的框显示。

### 提示

小贴士和技巧会以这种形式出现。

# 读者反馈

我们始终欢迎读者的反馈。请告诉我们您对本书的看法——喜欢或不喜欢的部分。读者反馈对我们非常重要，因为它帮助我们开发您真正能从中受益的书籍。

要向我们发送一般反馈，只需发送电子邮件到 feedback@packtpub.com，并在邮件主题中注明书名。

如果您在某个话题上有专业知识并且有兴趣编写或贡献本书，请参考我们的作者指南，网址是[www.packtpub.com/authors](https://www.packtpub.com/books/info/packt/authors)。

# 客户支持

现在您已经是一本 Packt 书籍的骄傲拥有者，我们提供了许多帮助您充分利用购买的资源。

## 下载示例代码

您可以从您的账户下载本书的示例代码文件，网址是[`www.packtpub.com`](http://www.packtpub.com)。如果您在其他地方购买了本书，可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便我们将文件直接通过电子邮件发送给您。

您可以按照以下步骤下载代码文件：

1.  使用您的电子邮件地址和密码登录或注册我们的官方网站。

1.  将鼠标指针悬停在顶部的**支持**标签上。

1.  点击**代码下载与勘误表**。

1.  在**搜索**框中输入书名。

1.  选择您要下载代码文件的书籍。

1.  从下拉菜单中选择你购买此书的来源。

1.  点击**代码下载**。

你还可以通过点击本书网页上的**代码文件**按钮来下载代码文件。该网页可以通过在**搜索框**中输入书名进行访问。请注意，您需要登录 Packt 帐户。

文件下载完成后，请确保使用最新版本的工具解压或提取文件夹：

+   WinRAR / 7-Zip for Windows

+   Zipeg / iZip / UnRarX for Mac

+   7-Zip / PeaZip for Linux

本书的代码包也托管在 GitHub 上，地址为[`github.com/PacktPublishing/Learning-Ansible-2-Second-Edition`](https://github.com/PacktPublishing/Learning-Ansible-2-Second-Edition)。我们还有其他来自丰富书籍和视频目录的代码包，地址为[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)。快去看看吧！

## 勘误

虽然我们已经尽力确保内容的准确性，但错误仍然可能发生。如果你在我们的书籍中发现错误——可能是文本或代码中的错误——我们非常感激你能向我们报告。通过这样做，你不仅能帮助其他读者避免困扰，还能帮助我们改进后续版本。如果你发现任何勘误，请访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)报告，选择你的书籍，点击**Errata Submission Form**链接，并输入勘误的详细信息。一旦勘误被验证，您的提交将被接受，勘误将被上传到我们的网站或添加到该书籍标题下的勘误列表中。

要查看之前提交的勘误，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索框中输入书名。相关信息将出现在**勘误**部分。

## 盗版

盗版问题一直是网络上所有媒体面临的持续性问题。在 Packt，我们非常重视版权和许可证的保护。如果你在互联网上发现任何形式的非法盗版作品，请立即提供其网址或网站名称，以便我们采取相应措施。

如果你发现涉嫌盗版的内容，请通过 copyright@packtpub.com 联系我们，并提供相关链接。

我们感谢你帮助保护我们的作者和我们向你提供有价值内容的能力。

## 问题

如果你对本书的任何方面有问题，可以通过 questions@packtpub.com 与我们联系，我们将尽力解决问题。
