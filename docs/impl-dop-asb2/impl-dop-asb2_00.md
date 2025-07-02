# 前言

Ansible 已经在 DevOps 领域掀起了风暴。其高度可扩展的架构和模块化实现使其成为在大大小小的组织中实施 DevOps 解决方案的理想工具。《使用 Ansible 2 实现 DevOps》将提供详细的教程和信息，教你如何在组织中实现 DevOps 解决方案。《使用 Ansible 2 实现 DevOps》旨在通过教育你 DevOps 实施的实际技术面以及文化和协作实施，来促进组织内良好的软件开发和交付实践。在本书的过程中，我们将逐步展示常见 DevOps 实施缺陷和自动化需求的解决方案。本书通过易读、易跟随的示例，带领你逐步实现解决方案，简单易懂。作为《使用 Ansible 2 实现 DevOps》的作者，我真诚地希望这本书能帮助你在成为 DevOps 和 Ansible 专家的道路上取得进展。因此，我创建了一个网站，帮助读者进一步学习，并可就任何问题与我联系。网址是 [`www.ansibledevops.com`](http://www.ansibledevops.com)。

# 本书涵盖的内容

第一章，*DevOps 基础*，向你介绍 DevOps（一个将开发与运维相结合的词汇），并教你 DevOps 运动以及近年来 DevOps 如何彻底改变全球各地组织的软件开发和交付。

第二章，*配置管理基础*，将教你像 Ansible 这样的工具如何通过消除环境漂移、自动化基础设施的配置以及提供一种简单一致的方式来构建 Bug 重现环境，从而使开发人员、测试人员和运维工作更加轻松。

第三章，*安装、配置和运行 Ansible*，将教你在哪里获取 Ansible，如何安装和设置控制服务器，如何使用库存项授权控制服务器，如何使用 Ansible 命令行运行 playbook 或查询基础设施组。最后，本章将介绍核心模块集以及如何提供与其他技术（如 Git、JIRA、Jenkins 等）的交互式接口。

第四章，*Playbook 和库存文件*，将进一步扩展你对 playbook、playbook 树、角色、任务和库存文件的理解。在这一章中，读者将看到一些简单 playbook 的基本示例，并学习如何以不同的方式运行它们。

第五章，*Playbooks: 超越基础*，将帮助你学习 YAML 标记语言的语法要求。此外，本章还将教你 Ansible 中与角色、包含、变量、循环、块、条件、注册和事实相关的特定语法。

第六章，*Ansible 中的 Jinja*，将深入讲解 Jinja2 模板以及如何在 Ansible 中使用它。

第七章，*Ansible Vault*，将概述 Ansible 管理和部署敏感信息的方式，并讲解如何最好地利用 Ansible Vault 工具确保敏感数据保持机密。你将通过示例学习如何控制和管理高度安全的信息，并了解 Ansible 如何确保信息安全的原理。

第八章，*Ansible 模块与库*，将重点介绍 Ansible 解决方案提供的广泛模块。Ansible 中的模块使得 playbook 能够连接并控制第三方技术（包括一些开源和封闭源技术）。在本章中，我们将讨论最受欢迎的模块，并深入探讨如何创建 playbook 任务来帮助管理开发人员、测试人员和运维人员可以使用的一整套工具和服务。

第九章，*将 Ansible 与 CI 和 CD 解决方案集成*，我们将教你如何利用其他与 DevOps 相关的工具来控制 Ansible。具体的主题包括 Jenkins 和 TeamCity。读者将学习如何将 Ansible 用作构建后操作，Ansible 如何适应持续集成（CI）和持续交付（CD）流水线，并提供每种工具的示例。

第十章，*Ansible 与 Docker*，我们将向你介绍如何使用 Python 扩展 Ansible，并创建与特定技术栈集成的自定义模块。这将通过一系列教程来完成，教读者编写和发布自定义的 Ansible 模块。本章将教你如何读取输入、管理事实、执行自动化任务、与 REST API 交互以及生成文档。

第十一章，*扩展 Ansible*，将帮助你学习 Galaxy 和 Tower 的流行功能，以及如何使用和配置这两者。完成本章后，你将拥有成为你所在组织中这两种独特技术的权威所需的知识。

第十二章，*Ansible Galaxy*，将讲解如何使用 Ansible 配置 Docker 容器，如何将 Ansible 与 Docker 服务集成，如何管理 Docker 镜像事实，以及如何完全控制 Docker 镜像。

# 你需要的本书资源

本书是以 Ubuntu 作为控制服务器并通过 SSH 访问目标服务器编写的。因此，您需要一台 Ubuntu 机器或虚拟机来使用本书中的教程。在模块和插件创建章节中，您需要在 Ubuntu 系统上安装 Python。

# 本书适合谁阅读

本书适合任何对 DevOps 实现以及如何利用 Ansible 创建自动化解决方案感兴趣的人。

# 约定

在本书中，您会看到许多文本样式，用于区分不同类型的信息。以下是这些样式的一些示例及其含义。文本中的代码词、数据库表名、文件夹名称、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名会如下所示：“如果 SSH 密钥共享不可用，Ansible 还提供了通过`--ask-become-pass`命令行参数请求密码的选项。”

一段代码的设置方式如下：

```
# File name: hellodevopsworld.yml
---
- hosts: all
 tasks:
 - shell: echo "hello DevOps world"

```

当我们希望引起您注意某个代码块的特定部分时，相关的行或项会以粗体显示：

```
[databaseservers]
mydbserver105.example.org
mydbserver205.example.org
[webservers]
mywbserver105.example.org
mywbserver205.example.org

```

任何命令行输入或输出均如下所示：

```
$ sudo apt-get install software-properties-common
$ sudo apt-add-repository ppa:ansible/ansible
$ sudo apt-get update
$ sudo apt-get install ansible

```

**新术语**和**重要单词**以粗体显示。您在屏幕上看到的单词，例如在菜单或对话框中，都会以这种方式出现在文本中：“要安装 Ansible 插件，只需在插件管理器（作为 Jenkins 管理员）中导航并从可用插件选项卡中选择 Ansible 插件，然后安装该插件。”

警告或重要提示会以如下框体显示。

提示和技巧会以如下形式显示。

# 读者反馈

我们始终欢迎读者的反馈。告诉我们您对本书的看法——您喜欢或不喜欢的部分。读者的反馈对我们非常重要，它帮助我们开发您真正能从中获益的书籍。要向我们提供一般反馈，只需通过电子邮件发送至`feedback@packtpub.com`，并在邮件主题中提及书名。如果您在某个领域具有专业知识，并且有兴趣编写或参与编写一本书，请参阅我们的作者指南：[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在，您已经成为一本 Packt 图书的骄傲拥有者，我们为您提供了一些帮助，以便您最大限度地从购买中获益。

# 下载示例代码

您可以从[`www.packtpub.com`](http://www.packtpub.com)账户中下载本书的示例代码文件。如果您在其他地方购买了本书，可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，将文件直接通过电子邮件发送给您。您可以通过以下步骤下载代码文件：

1.  使用您的电子邮件地址和密码登录或注册我们的网站。

1.  将鼠标指针悬停在顶部的 SUPPORT 标签上。

1.  点击“代码下载与勘误”。

1.  在搜索框中输入书名。

1.  选择您要下载代码文件的图书。

1.  从下拉菜单中选择您购买此书的地方。

1.  点击“代码下载”。

下载文件后，请确保使用以下最新版本解压或提取文件夹：

+   Windows 下的 WinRAR / 7-Zip

+   Mac 下的 Zipeg / iZip / UnRarX

+   Linux 下的 7-Zip / PeaZip

本书的代码包也托管在 GitHub 上，地址是 [`github.com/PacktPublishing/Implementing-DevOps-with-Ansible-2`](https://github.com/PacktPublishing/Implementing-DevOps-with-Ansible-2)。我们还在 [`github.com/PacktPublishing/`](https://github.com/PacktPublishing/) 提供了其他代码包，来自我们丰富的图书和视频目录。快来查看吧！

# 下载本书的彩色图片

我们还提供了一个 PDF 文件，其中包含本书中使用的屏幕截图/图表的彩色图片。彩色图片将帮助你更好地理解输出中的变化。你可以从 [`www.packtpub.com/sites/default/files/downloads/ImplementingDevOpswithAnsible2_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/ImplementingDevOpswithAnsible2_ColorImages.pdf) 下载此文件。

# 勘误

尽管我们已经尽力确保内容的准确性，但错误难免发生。如果你在我们的书中发现错误——可能是文本或代码上的错误——我们将非常感激你向我们报告。通过这样做，你不仅能避免其他读者的困惑，还能帮助我们改进后续版本的书籍。如果你发现任何勘误，请访问 [`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata) 提交勘误，选择你的书籍，点击勘误提交表单链接，并输入勘误详情。勘误一经验证，将被接受并上传到我们的网站，或者添加到该书勘误列表中。要查看之前提交的勘误，请访问 [`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support) 并在搜索框中输入书名。所需信息将在勘误部分显示。

# 盗版

网络上的版权材料盗版问题在所有媒体中都持续存在。在 Packt，我们非常重视版权和许可的保护。如果你在互联网上发现任何我们作品的非法复制品，请立即提供网址或网站名称，以便我们采取措施。请通过 `copyright@packtpub.com` 联系我们，并附上涉嫌盗版材料的链接。感谢你帮助我们保护作者和我们为您提供有价值内容的能力。

# 问题

如果你对本书的任何方面有问题，可以通过 `questions@packtpub.com` 联系我们，我们将尽力解决问题。
