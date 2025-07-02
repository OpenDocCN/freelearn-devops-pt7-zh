# 前言

*Atlassian Confluence 5 Essentials* 是一本实用的操作指南，介绍了 Atlassian Confluence，这是一款强大的企业级协作工具，旨在帮助团队创建、分享和讨论内容。

本书将向你展示如何安装和管理自己的 Confluence 环境。你将学习如何配置和自定义 Confluence，以适应你的组织需求并为你的企业创造价值。本书的章节结构将引导你了解管理和使用 Confluence 的所有关键方面。

你将从设置自己的 Confluence 安装开始，接下来的章节将介绍所有的关键功能。随着每一章的深入，你将学习到创建引人入胜的内容、分享信息以及激励用户彼此协作等重要概念。

本书是一本关于 Atlassian Confluence 所有关键方面的深入指南。书中包含了丰富的示例和逐步指导，帮助你成为 Confluence 的专家。

# 本书涵盖的内容

*第一章, 初识 Confluence*，将引导你完成 Confluence 的安装过程，并为你提供一个本地安装环境，整个书籍将使用该环境。在第一章结束时，你应该能够完成 Confluence 的安装并使其正常运行。

*第二章, 用户管理*，讲解了如何邀请并注册新用户参与内容协作过程。我们还将介绍如何将 Confluence 与现有的用户目录（如 LDAP）进行连接。

*第三章, 创建内容*，也许是最重要的一章，因为内容至关重要。我们将通过空间、页面和博客文章的概念，讲解如何在 Confluence 中添加内容。Confluence 的富文本编辑器拥有许多功能，在这一章中，我们将学习如何通过创建第一个页面来掌握其中的大部分功能。

*第四章, 管理内容*，重点介绍了如何找到相关内容以及如何使用关注来跟踪你的内容。Confluence 自带强大的搜索引擎，我们将在本章结束时全面了解其功能。

*第五章, 在 Confluence 中协作*，介绍了如何在日常使用中操作 Confluence。我们将讲解如何通过@提及和分享来让更多人参与内容创作过程，以及如何通过任务来跟踪进展。如果你在路上，Confluence 移动版将帮助你随时了解最新内容、任务和通知。

*第六章, 保护你的内容*，讲解了如何保护你的私密信息。Confluence 允许在全局、空间和内容级别设置权限，提供了企业解决方案所需的精细安全性。

*第七章，自定义 Confluence*，将介绍如何更改 Confluence 的外观和感觉，使其能够添加公司品牌或仅对某个空间进行自定义。

*第八章，高级 Confluence*，涵盖了许多不同的高级话题，如内容模板、元数据的处理和键盘快捷键的使用。

*第九章，一般管理*，深入探讨了如何查找和管理 Confluence 的插件。插件可以增加额外的功能或集成。如果你的 Confluence 安装出现问题并需要支持，本章将指导你如何从 Atlassian 或本地专家那里获得支持。

*第十章，扩展 Confluence*，重点介绍了插件开发的基础知识和可能性。到本章结束时，你应该知道如何开始构建一个插件，以及我们可以在何种层面上扩展 Confluence。

# 本书所需内容

本书中使用的 Confluence 安装版本为 Windows 独立版（ZIP），你可以直接从 Atlassian 网站获取：[www.atlassian.com/software/confluence/download](http://www.atlassian.com/software/confluence/download)。

在写作时，Confluence 的最新版本为 5.1.1。

你还需要安装多个附加软件库，包括 Java SDK，你可以从[`www.oracle.com/technetwork/java/javase/downloads/index.html`](http://www.oracle.com/technetwork/java/javase/downloads/index.html)下载，以及 PostgreSQL，可以从[`www.postgresql.org/download/`](http://www.postgresql.org/download/)下载。

# 本书适用对象

如果你刚刚开始使用 Confluence，无论是作为用户还是管理员，本书将帮助你快速入门并教你所需的所有知识。即使你已经使用 Confluence 一段时间，本书也能为你提供新的见解和技巧，帮助你更高效地使用 Confluence。

# 约定

本书中，你将找到多种文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义解释。

文本中的代码词汇如下所示：“在你的 Apache 配置目录中创建一个名为`local_machines_only.conf`的文件。”

代码块的设置如下所示：

```
<linklinkId="config-link">/plugins/config/alpha.action</link>
<icon height="16" width="16">
<link>/images/icons/config.gif</link>
</icon>
```

任何命令行输入或输出都会如下书写：

```
netstat –a | find /I "1990"

```

**新术语**和**重要词汇**以粗体显示。你在屏幕上看到的词汇，例如在菜单或对话框中的词汇，会像这样出现在文本中：“点击**下一步**按钮将带你到下一个屏幕”。

### 注意

警告或重要提示会以框框形式出现，如下所示。

### 提示

提示和技巧如下所示。

# 读者反馈

我们始终欢迎读者的反馈。让我们知道您对这本书的看法——您喜欢或可能不喜欢的部分。读者的反馈对我们非常重要，帮助我们开发出真正对您有价值的书籍。

若要向我们发送一般反馈，只需发送电子邮件至`<feedback@packtpub.com>`，并在邮件主题中提及书名。

如果您在某个领域有专业知识，并且有兴趣撰写或参与编写书籍，请查看我们在[www.packtpub.com/authors](http://www.packtpub.com/authors)上的作者指南。

# 客户支持

现在您已经成为一本 Packt 书籍的骄傲拥有者，我们提供了许多资源来帮助您充分利用您的购买。

## 下载示例代码

您可以从[`www.packtpub.com`](http://www.packtpub.com)账户中下载您购买的所有 Packt 书籍的示例代码文件。如果您在其他地方购买了本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

## 勘误

尽管我们已尽最大努力确保内容的准确性，但错误仍然会发生。如果您在我们的书籍中发现错误——可能是文本或代码错误——我们将非常感激您向我们报告。这样，您可以帮助其他读者避免困惑，并帮助我们改进后续版本。如果您发现任何勘误，请访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**勘误** **提交** **表单**链接，并输入勘误的详细信息。一旦您的勘误被验证，您的提交将被接受，勘误将被上传到我们的网站，或者添加到该书籍现有勘误的列表中，显示在该标题的勘误部分。您可以通过访问[`www.packtpub.com/support`](http://www.packtpub.com/support)查看任何现有的勘误。

## 盗版

互联网版权盗版问题在各类媒体中普遍存在，Packt 非常重视版权和许可的保护。如果您在互联网上发现我们作品的任何非法复制品，无论以何种形式，请立即提供相关的地址或网站名称，以便我们采取措施。

请通过`<copyright@packtpub.com>`与我们联系，并提供涉嫌盗版材料的链接。

我们感谢您的帮助，以保护我们的作者和我们为您提供有价值内容的能力。

## 问题

如果您在书籍的任何方面遇到问题，可以通过`<questions@packtpub.com>`与我们联系，我们将尽力解决。
