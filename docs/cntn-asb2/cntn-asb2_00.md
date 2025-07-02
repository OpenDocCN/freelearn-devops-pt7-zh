# 前言

在过去的几年中，IT 领域在软件应用开发和部署的方式上发生了剧变。自动化、云计算和虚拟化的兴起，深刻改变了系统管理员、软件开发人员以及整个组织如何看待和管理基础设施。几年前，很多 IT 行业内的人认为，允许关键任务应用程序在企业数据中心外运行几乎是不可能的。然而，现在比以往更多的组织将基础设施迁移到云服务，如 AWS、Azure 和 Google Compute，以节省时间并削减与运行物理基础设施相关的开销成本。通过抽象化硬件，企业可以将注意力集中在真正重要的事物上——为用户提供服务的软件应用。

IT 领域的下一波巨大浪潮正式始于 2013 年，Docker 容器引擎的首次发布。Docker 让用户能够轻松地将软件打包成小型、可重用的执行环境，称为容器，并利用 Linux 内核中的特性与 LXC（Linux 容器）一起使用。使用 Docker，开发人员可以创建微服务应用程序，这些应用程序可以快速构建、在任何环境中都能运行，并利用可版本控制的可重用服务工件（容器镜像）。随着越来越多的用户采用容器化工作流，执行过程中开始出现一些问题。尽管 Docker 在构建和运行容器方面表现出色，但它在整个容器生命周期中难以成为一个真正的端到端解决方案。

Ansible Container 项目旨在将 Ansible 配置管理和自动化平台的强大功能引入容器世界。Ansible Container 弥补了容器生命周期管理的空白，它允许容器的构建和部署流水线使用 Ansible 语言。通过使用 Ansible Container，你不仅可以构建容器，还能利用强大的 Ansible 配置管理语言，在远程服务器和云平台上部署全规模应用。

本书将作为使用 Ansible Container 项目的指南。我的目标是通过本书的学习，你将牢固地理解 Ansible Container 的工作原理，并能够利用其众多功能，从开发到生产环境，构建稳健的容器化软件堆栈。

# 本书涵盖内容

第一章，*使用 Docker 构建容器*，向读者介绍了 Docker 是什么，它是如何工作的，以及如何使用 Dockerfile 和 Docker Compose 的基础知识。本章奠定了学习如何使用 Ansible Container 所需的基础概念。

第二章，*使用 Ansible Container*，探讨了 Ansible Container 工作流。本章使读者熟悉 Ansible Container 的核心概念，如构建、运行和销毁。

第三章，*你的第一个 Ansible 容器项目*，通过利用 Ansible Galaxy 上的社区角色，带给用户构建一个简单 Ansible 容器项目的体验。在本章结束时，读者将熟悉如何构建项目并将容器制品推送到像 Docker Hub 这样的容器镜像仓库。

第四章，*角色中有什么？*，向用户概述了如何编写自定义容器启用的角色以供 Ansible Container 使用。本章的总体目标是编写一个角色，从头开始构建一个功能完备的 MariaDB 容器镜像。在本章结束时，用户应对使用正确风格和语法编写 Ansible playbook 有基本了解。

第五章，*使用 Kubernetes 大规模管理容器*，向读者概述了 Kubernetes 平台及其核心功能。在本章中，读者将有机会在 Google Cloud 中创建一个多节点的 Kubernetes 集群并在其中运行容器。

第六章，*使用 OpenShift 管理容器*，向读者介绍了 Redhat 的 OpenShift 平台。本章向读者展示了使用 Minishift 部署本地 OpenShift 集群并在其上运行容器化工作负载所需的步骤。本章还探讨了 Kubernetes 和 OpenShift 之间的关键区别，尽管它们的架构本质上是相似的。

第七章，*部署你的第一个项目*，深入探讨了 Ansible Container 工作流中的最终命令——部署。通过使用部署，读者将获得将先前构建的项目部署到 Kubernetes 和 OpenShift 上的第一手经验，利用 Ansible Container 作为端到端工作流工具。

第八章，*构建和部署多容器项目*，探讨了如何使用 Ansible Container 构建一个涉及多个应用容器的项目。要全面理解这一主题，必须先了解容器网络并配置容器以访问网络资源。本章将为读者提供构建和部署一个多容器项目的机会，该项目使用 Django、Gulp、NGINX 和 PostgreSQL 容器。

第九章，*深入了解 Ansible Container*，为读者提供了掌握整个 Ansible Container 工作流后应采取的下一步措施。此部分讨论的主题包括将 Ansible Container 与 CICD 工具集成，以及在 Ansible Galaxy 上共享项目。

# 本书所需的内容

本书假设读者具有初级到中级的 Linux 操作系统使用经验、应用部署经验和服务器管理经验。本书将引导你完成所需步骤，在本地笔记本电脑上建立一个完全功能的实验环境，并快速使用 VirtualBox 和 Vagrant 环境进行操作。在开始之前，建议读者已经安装并运行 VirtualBox、Vagrant 和 Git 命令行客户端。为了完全符合本环境的要求，必须满足或超过以下系统规格：

+   CPU：2 核（Intel Core i5 或同等处理器）

+   内存：8 GB RAM

+   硬盘空间：80 GB

在本书中，你将需要以下软件列表：

+   VirtualBox 5.1 或更高版本

+   Vagrant 1.9.1 或更高版本

+   一个用于编辑 YAML 文件的文本编辑器（推荐使用 GitHub Atom 或 VIM）

安装必要的软件包需要互联网连接。

# 本书适合谁阅读

本书旨在帮助目前担任系统管理员、DevOps 工程师、技术架构师（或类似角色）的人，快速掌握 Ansible 容器工作流。如果读者在阅读本书之前已经具备 Docker、Ansible 或其他相关自动化平台的基本知识，那么本书将更加有帮助，尽管这不是必需的。我希望读者在阅读本书时能够深入理解这些基础知识。最终目标是帮助读者在了解如何使用 Ansible Container 加速构建、运行、测试和部署应用容器的过程中，建立从开发到生产环境的坚实基础。

# 约定

在本书中，你将看到许多文本样式，它们用于区分不同类型的信息。以下是这些样式的一些示例及其含义解释：文本中的代码词汇、数据库表名、文件夹名称、文件名、文件扩展名、路径名、虚拟 URL、用户输入以及 Twitter 用户名显示如下：“我们可以通过使用 `include` 指令来包含其他上下文。”

代码块如下所示：

```
---
- name: Create User Account
  user:
    name: MyUser
    state: present

- name: Install Vim text editor
  apt:
    name: vim
    state: present
```

所有命令行输入或输出将如下所示：

```
ubuntu@node01:/tmp$ ansible-galaxy init MyRole --container-enabled
- MyRole was created successfully
```

**新术语**和**重要词汇**以粗体显示。屏幕上出现的词汇，例如菜单或对话框中的内容，会像这样出现在文本中：“点击 Next 按钮将带你进入下一个屏幕。”

警告或重要说明会以如下方式出现。

提示和技巧会以如下方式出现。

# 读者反馈

我们始终欢迎读者的反馈。让我们知道您对本书的看法——您喜欢或不喜欢什么。读者反馈对我们很重要，因为它帮助我们开发出您真正能够从中受益的书籍。若要向我们提供一般反馈，只需发送电子邮件至`feedback@packtpub.com`，并在邮件主题中注明书名。如果您在某个领域有专业知识并且有兴趣编写或参与编写书籍，请参阅我们的作者指南，网址为[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在，您已经成为一本 Packt 书籍的骄傲拥有者，我们提供了一些内容，帮助您最大限度地从您的购买中获益。

# 下载本书的彩色图像

我们还为您提供了一个包含本书中使用的截图/图表彩色图像的 PDF 文件。这些彩色图像将帮助您更好地理解输出中的变化。您可以从[`www.packtpub.com/sites/default/files/downloads/ContainerizationwithAnsible2_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/ContainerizationwithAnsible2_ColorImages.pdf)下载此文件。

# 勘误表

尽管我们已尽力确保内容的准确性，但错误仍然可能发生。如果您在我们的书籍中发现错误——可能是文本或代码中的错误——我们将非常感激您能向我们报告。通过这样做，您可以帮助其他读者避免困扰，并帮助我们改进本书的后续版本。如果您发现任何勘误，请访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)报告勘误，选择您的书籍，点击“勘误提交表格”链接，并输入勘误的详细信息。一旦您的勘误被验证，您的提交将被接受，并且勘误将上传到我们的网站或添加到该书籍的勘误列表中。要查看已提交的勘误，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索框中输入书名。所需信息将显示在“勘误”部分。

# 盗版

互联网版权材料盗版是一个跨媒体的持续问题。在 Packt，我们非常重视保护我们的版权和许可。如果您在互联网上发现我们作品的非法副本，请立即提供该内容的链接或网站名称，以便我们采取相应措施。请通过`copyright@packtpub.com`联系我们，并附上涉嫌盗版内容的链接。感谢您在保护我们的作者及我们为您提供有价值内容方面的帮助。

# 问题

如果您在本书的任何方面遇到问题，可以通过`questions@packtpub.com`联系我们，我们会尽力解决问题。
