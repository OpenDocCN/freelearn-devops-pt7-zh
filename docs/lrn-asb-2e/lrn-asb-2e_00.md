# 前言

Ansible 是一个开源的编排工具，已经经历了显著的增长，现在成为一个综合的编排和配置管理解决方案，由红帽公司拥有。本书将指导你使用核心模块、供应商提供的模块以及社区 Ansible 模块来编写 playbook，以部署各种系统，从简单的 LAMP 堆栈到高可用的公共云基础设施。

到本书结束时，你将掌握以下技能：

+   对 Ansible 及其各种支持工具的坚实基础知识

+   能够编写自定义 playbook 来配置 Linux 和 Windows 服务器

+   能够使用代码定义高可用的云基础设施，从而能够轻松地将你的基础设施配置与代码库一起分发

+   理解如何使用 Ansible Galaxy，使用社区贡献的角色，并创建和贡献你自己的角色

+   能够通过 GitHub Actions 和 Azure DevOps 运行你的 Ansible playbook

+   能够部署和配置 Ansible AWX，这是一个基于 Web 的 Ansible 界面

+   从多个使用案例中获得的各种技能，展示如何将 Ansible 集成到你的日常任务和项目中

你将扎实地理解如何将 Ansible 融入到你作为系统管理员、开发者或 DevOps 实践者的日常职责中。

# 本书适用人群

本书是为以下角色的人编写的，他们希望通过利用 Ansible 的功能来简化工作流程：

+   **系统管理员**：本书将帮助你自动化重复性任务，并确保在你的系统中实现一致的配置，如果你负责管理和维护服务器、网络和其他基础设施组件。

+   **开发者**：作为开发者，你可以通过本书学习如何使用 Ansible 来配置和管理开发环境、部署应用程序，并将基础设施即代码实践融入到你的开发工作流中。

+   **DevOps 实践者**：如果你是一个负责弥合开发和运维之间差距的 DevOps 实践者，本书将为你提供使用 Ansible 创建高效、可重复和可扩展部署流程的工具和知识。

本书开始时无需具备 Ansible 的先前经验。

# 本书内容涵盖的主题

*第一章*，*安装和运行 Ansible*，讨论了 Ansible 开发的背景和所解决的问题。在介绍背景后，我们将介绍如何在 macOS 和 Linux 上安装 Ansible。我们还将讨论为什么没有原生的 Windows 安装程序，并介绍如何在 Windows 子系统 for Linux 上安装 Ansible。

*第二章*，*探索 Ansible Galaxy*，讨论了 Ansible Galaxy，这是一个社区和供应商贡献的角色的在线库。在本章中，我们将发现一些最好的角色，如何使用它们，以及如何创建自己的角色并将其托管在 Ansible Galaxy 上。

*第三章*，*Ansible 命令*，解释了如何在编写和执行更高级的 playbook 之前，检查 Ansible 命令。在这里，我们将介绍构成 Ansible 的工具的使用。

*第四章*，*部署 LAMP 堆栈*，讨论了使用 Ansible 自带的各种核心模块部署完整的 LAMP 堆栈。我们将以本地运行的 Ubuntu 机器为目标。

*第五章*，*部署 WordPress*，扩展了我们在上一章中部署的 LAMP 堆栈 playbook，作为我们的基础。我们将使用 Ansible 下载、安装并配置 WordPress —— 一个流行的内容管理系统（CMS）。

*第六章*，*目标多发行版*，解释了如何调整上一章的 playbook，以便它可以同时在 Debian（我们到目前为止的目标）和基于 Red Hat 的 Linux 发行版上运行。

*第七章*，*Ansible Windows 模块*，探索了支持并与基于 Windows 的服务器交互的 Ansible 模块的不断增长的集合。

*第八章*，*Ansible 网络模块*，讨论了通过 Ansible Galaxy 提供的来自不同供应商的网络模块。由于它们的要求，我们将仅讨论这些模块的功能。

*第九章*，*迁移到云端*，讨论了如何从使用本地虚拟机迁移到使用 Ansible 在 Microsoft Azure 中部署网络和计算资源。然后，我们将使用前几章的 playbook 来安装和配置 LAMP 堆栈和 WordPress。

*第十章*，*构建云网络*，由于我们刚刚在 Microsoft Azure 上启动了虚拟机，本章转向亚马逊 Web 服务（AWS）；然而，在启动任何计算实例之前，我们必须创建一个可以托管这些实例的网络。

*第十一章*，*高度可用的云部署*，继续我们的亚马逊 Web 服务（AWS）部署。我们将开始在我们在上一章创建的网络中部署计算和存储服务，到本章结束时，我们将拥有一个高度可用的 WordPress 安装。

*第十二章*，*构建 VMware 部署*，讨论了允许与典型 VMware 安装的各个组件交互的模块。

*第十三章*，*扫描你的 Ansible 剧本*，提供了运行两个第三方工具——Checkov 和 KICS——的实际示例。这些工具旨在扫描你的 Ansible 剧本代码，查找常见错误和潜在的安全问题。

*第十四章*，*使用 Ansible 加固你的服务器*，解释了如何安装和执行 OpenSCAP。我们还将自动生成修复 Ansible 剧本和 Bash 脚本，以解决扫描中发现的任何问题。我们还将探讨如何对使用前面章节中的剧本部署的资源运行 WPScan 和 OWASP ZAP 扫描。

*第十五章*，*在 GitHub Actions 和 Azure DevOps 中使用 Ansible*，将探讨如何在这两个 CI/CD 平台上运行我们的 Ansible 剧本。由于这两个平台都没有原生的 Ansible 支持，我们将讨论如何安装和运行 Ansible，以便最大程度地利用这些平台。

*第十六章*，*介绍 Ansible AWX 和 Red Hat Ansible 自动化平台*，探讨了两个基于 Web 的界面：我们将讨论商业版的 Red Hat Ansible 自动化平台，之后将深入探讨如何部署和配置开源的 Ansible AWX。

*第十七章*，*使用 Ansible 的下一步*，讨论了如何将 Ansible 集成到我们的日常工作流中，从与协作服务的交互到使用内置调试器排查剧本问题。我们还将查看一些我在多个组织中使用 Ansible 的真实案例。

# 为了最大程度地从本书中获益

为了最大程度地从本书中获益，我假设你具备以下条件：

+   在 Linux 系统和 macOS 上使用命令行的经验

+   具备在 Linux 服务器上安装和配置服务的基本理解

+   具备 Git、YAML 和虚拟化等服务与语言的工作知识

    | **本书中涉及的软件/硬件** | **操作系统要求** |
    | --- | --- |
    | Ansible | 通过 Linux 子系统在 macOS、Linux 或 Windows 上运行 |
    | Canonical Multipass | macOS、Linux 或 Windows |
    | 各种公共云提供商的 CLI 工具 | 通过 Linux 子系统在 macOS、Linux 或 Windows 上运行 |

**如果你使用的是本书的数字版本，我们建议你亲自输入代码，或通过本书 GitHub 仓库访问代码（下节中会提供链接）。这样做可以帮助你避免因复制粘贴代码而引发的潜在错误。**

# 下载示例代码文件

你可以从 GitHub 下载本书的示例代码文件，链接：[`github.com/PacktPublishing/Learn-Ansible-Second-Edition`](http://github.com/PacktPublishing/Learn-Ansible-Second-Edition)。如果代码有更新，将会在 GitHub 仓库中更新。

我们还有其他代码包，来自我们丰富的书籍和视频目录，您可以在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)查看。快来看看吧！

# 使用的约定

本书中使用了许多文本约定。

`文本中的代码`：表示文本中的代码词汇、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 账号。以下是一个示例：“如您所见，它调用了一个名为`{{ apache_packages }}`的变量，该变量在`roles/apache/defaults/main.yml`中定义，如下所示。”

一段代码会设置如下：

```
- name: "Install apache packages"
  ansible.builtin.apt:
    state: "present"
    pkg: "{{ apache_packages }}"
```

当我们希望引起您对某个代码块中特定部分的注意时，相关的行或项会以粗体显示：

```
- name: "Install apache packages"
  ansible.builtin.apt:
    state: "present"
    pkg: "{{ apache_packages }}"
```

任何命令行输入或输出的格式如下所示：

```
$ ansible-playbook -i hosts site.yml
```

**粗体**：表示新术语、重要词汇或您在屏幕上看到的单词。例如，菜单或对话框中的词汇会以**粗体**显示。以下是一个示例：“您可以保持其他选项为默认设置，然后点击表单底部的**创建仓库**按钮。”

提示或重要说明

显示如下。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：如果您对本书的任何内容有疑问，请通过电子邮件联系我们：customercare@packtpub.com，并在邮件主题中注明书名。

**勘误表**：虽然我们已经尽力确保内容的准确性，但错误难免发生。如果您发现本书中有错误，我们将非常感谢您向我们报告。请访问[www.packtpub.com/support/errata](http://www.packtpub.com/support/errata)并填写表格。

**盗版**：如果您在互联网上发现任何我们作品的非法复制形式，烦请提供该材料的所在位置或网站名称。请通过版权@packt.com 与我们联系，并附上该材料的链接。

**如果您有兴趣成为作者**：如果您在某个主题上有专长，并且有兴趣撰写或参与编写书籍，请访问 authors.packtpub.com。

# 分享您的想法

一旦您读完了*《学习 Ansible》*，我们很希望听到您的想法！[请点击这里直接进入本书的亚马逊评价页面并分享您的反馈](https://packt.link/r/1835088910)。

您的评价对我们和技术社区非常重要，并且帮助我们确保提供优质内容。

# 下载本书的免费 PDF 副本

感谢您购买本书！

您喜欢随时随地阅读，但无法随身携带纸质书籍吗？

您购买的电子书与您选择的设备不兼容吗？

不用担心，现在购买每本 Packt 书籍时，您将免费获得该书的无 DRM PDF 版本。

在任何地方，任何设备上阅读。搜索、复制并粘贴你最喜爱的技术书籍中的代码，直接将其应用到你的项目中。

好处不止于此，你还可以获得独家折扣、新闻简报，并且每天都有精彩的免费内容发送到你的邮箱。

按照以下简单步骤，获取更多好处：

1.  扫描二维码或访问下面的链接

![](img/B21620_QR_Free_PDF.jpg)

[`packt.link/free-ebook/9781835088913`](https://packt.link/free-ebook/9781835088913)

1.  提交你的购买凭证

1.  就这些！我们将直接把你的免费 PDF 和其他福利发送到你的邮箱。

# 第一部分：介绍、安装和运行 Ansible

在这一部分，我们将深入探讨 Ansible 的世界，探索其基本概念。你将学习如何在不同的操作系统上安装 Ansible，并熟悉 Ansible 命令和 Playbook 的基本结构。到这一部分结束时，你将具备坚实的基础，能够在后续深入学习 Ansible 自动化任务时有更好的基础。

本部分包含以下章节：

+   *第一章**, 安装与运行 Ansible*

+   *第二章**, 探索 Ansible Galaxy*

+   *第三章**, Ansible 命令*
