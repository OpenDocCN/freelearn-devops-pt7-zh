# 序言

多年来，Jira 已经从最初为工程师设计的简单故障跟踪系统发展为通用问题跟踪解决方案。现在，Jira 这个术语指的是一套产品，包括 Jira 软件、Jira 服务管理、Jira 核心等。随着这一变化，每个产品更专注于其功能和提供的价值。现在，客户更容易选择最适合其需求的产品，无论是运行敏捷软件开发项目、客户支持门户还是通用任务管理系统。

借助最新的数据中心产品，Jira 现在是任何希望通过性能、可靠性、可扩展性和安全性未来验证其团队的企业的首选解决方案。

在这本书中，我们将涵盖 Jira 的所有基础知识及其核心功能，以及数据中心提供的高级功能。我们还将探讨第三方应用程序，这些应用程序为 Jira 平台添加了额外的功能。本书充满了现实生活中的例子和逐步说明，将帮助您成为 Jira 专家。

# 本书适合谁

这本书将对项目经理特别有用，但也适用于其他 Jira 用户，包括开发人员，以及除软件开发外的任何其他行业，他们希望利用 Jira 强大的任务管理和工作流功能来更好地管理业务流程。

# 本书内容包括

*第一章*，*开始使用 Jira 数据中心*，作为 Jira 的整体介绍，涵盖其高级架构。我们将讨论新部署以及如何从现有部署升级。我们还将介绍 Atlassian 的 Jira 数据中心产品。这也将作为读者将要进行的项目的起点。

*第二章*，*使用 Jira 进行业务项目*，介绍了在不基于软件开发的项目中使用 Jira，例如通用任务管理解决方案。本章重点介绍 Jira 核心产品提供的基本功能的使用。

*第三章*，*使用 Jira 进行敏捷项目*，介绍了特定于 Jira 软件的功能。本章重点介绍使用 Jira 进行软件开发项目，特别是使用 Scrum 和 Kanban 等敏捷方法。

*第四章*，*处理问题*，介绍了问题，这是使用 Jira 的基石。重点是确保用户了解问题及其功能。您还将学习如何使每个功能可用并将其自定义超出开箱即用设置。

*第五章*，*字段管理*，介绍了字段，特别是如何使用自定义字段来定制 Jira，以便更有效地收集数据。你将学习如何创建新的自定义字段，以及如何控制字段配置，如可见性和渲染选项。

*第六章*，*屏幕管理*，介绍了屏幕。你将学习如何从零开始创建新的屏幕，并指定哪些字段（系统字段和自定义字段）将被显示。我们还将讲解复杂的方案映射，以便将新的屏幕应用到项目中。

*第七章*，*工作流与业务流程*，探索了 Jira 提供的最强大功能——工作流。介绍了问题生命周期的概念，并解释了工作流的各个方面。本章还探讨了工作流与之前介绍过的其他 Jira 方面（如屏幕）之间的关系。在示例项目中，我们还简要提到了 Jira 应用的概念，并使用了一些流行的应用。

*第八章*，*电子邮件与通知*，讲述了电子邮件以及 Jira 如何使用它们向最终用户发送通知。我们将首先解释 Jira 如何向用户发送通知，然后介绍 Jira 如何处理传入的电子邮件，创建、评论以及更新问题。

*第九章*，*Jira 安全性*，解释了 Jira 的安全模型，从如何管理用户、组和角色开始。读者将学习 Jira 的权限管理安全层次结构。最后，我们将介绍如何使用 SAML 设置单点登录，这是大多数企业组织的常见需求。

*第十章*，*搜索、报告与分析*，专注于如何使用 Jira 收集的数据做更多的事情，包括搜索、报告和使用仪表板。你还将学习如何将这些数据和报告提供给 Jira 之外的用户，无论是通过电子邮件，还是通过其他应用展示。

*第十一章*，*Jira 服务管理*，介绍了 Jira 服务管理，它允许你将 Jira 用作客户支持门户。读者将学习如何使用 Jira 服务管理在内部运行和管理支持队列，同时与客户进行有效沟通。

*第十二章*，*Jira 与第三方应用*，讲解了如何使用第三方应用扩展 Jira 的功能。你将学习如何查找、安装、更新和管理 Jira 的应用。我们还将介绍一些流行的第三方应用，以及它们如何帮助 Jira 提升到一个新水平。

# 要充分利用本书

本书使用的安装包是 Windows 安装程序独立发行版，你可以直接从 Atlassian 获取，Jira 软件可通过 [`www.atlassian.com/software/jira/download`](https://www.atlassian.com/software/jira/download) 下载，Jira 服务管理可通过 [`www.atlassian.com/software/jira/service-desk/download`](https://www.atlassian.com/software/jira/service-desk/download) 下载。

你还需要其他软件，包括 Java SDK，你可以从 [`www.oracle.com/technetwork/java/javase/downloads/index.html`](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 获取，MySQL 可以从 [`dev.mysql.com/downloads`](http://dev.mysql.com/downloads) 获取。

| **书中涵盖的软件/硬件** | **操作系统要求** |
| --- | --- |
| Jira 软件 | Windows、macOS 或 Linux |
| Jira 服务管理 |  |
| Java |  |
| MySQL |  |

**如果你正在使用本书的数字版本，我们建议你自己输入代码。这样做有助于避免与复制和粘贴代码相关的潜在错误。**

*有 XML、Java 或 Groovy 编程语言经验将有助于更好地理解书中的代码片段。*

与本书相关的任何勘误信息可以在以下链接找到：[`github.com/packt-pradeeps/Jira-8-Essentials`](https://github.com/packt-pradeeps/Jira-8-Essentials)。

# 下载彩色图片

我们还提供了一个 PDF 文件，其中包含本书中使用的截图和图表的彩色图片。你可以在此下载：[`packt.link/9C0B9`](https://packt.link/9C0B9)。

# 使用的约定

本书中使用了许多文本约定。

`文本中的代码`：表示文本中的代码字、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟网址、用户输入和 Twitter 账号。举个例子：“你应该能看到 `New Employee` 和 `Termination` 问题类型。”

代码块设置如下：

```
<Resource name="mail/JiraMailServer"
  auth="Container"
  type="javax.mail.Session"
```

当我们希望引起你对代码块中特定部分的注意时，相关的行或项会以粗体显示：

```
<security-constraint>
  <web-resource-collection>
    <web-resource-name>all-except-attachments</web-resource-name>
    <url-pattern>*.js</url-pattern>
    <url-pattern>*.jsp</url-pattern>
```

任何命令行输入或输出如下所示：

```
  create database jiradb character set utf8;
```

**粗体**：表示一个新术语、重要词汇或你在屏幕上看到的词语。例如，菜单或对话框中的词语会以**粗体**显示。举个例子：“在**创建**按钮旁边有一个**创建另一个**选项。”

提示或重要说明

看起来是这样的。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：如果你对本书的任何内容有疑问，请发送邮件至 customercare@packtpub.com，并在邮件主题中注明书名。

**勘误**：尽管我们已经尽一切努力确保内容的准确性，但错误难免会发生。如果你发现本书中有错误，我们将感激你向我们报告。请访问[www.packtpub.com/support/errata](https://www.packtpub.com/support/errata)并填写表单。

**盗版**：如果你在互联网上遇到任何我们作品的非法复制品，请提供该位置地址或网站名称，我们将不胜感激。请通过电子邮件联系 copyright@packt.com，并附上该材料的链接。

**如果你有兴趣成为作者**：如果你在某个领域有专长，并且有兴趣写作或参与书籍编写，请访问[authors.packtpub.com](https://authors.packtpub.com)。

# 分享你的想法

阅读完*Jira 8 Essentials*后，我们很想听听你的想法！请[点击这里直接进入亚马逊评论页面](https://packt.link/r/180323265X)并分享你的反馈。

你的评价对我们以及技术社区非常重要，将帮助我们确保提供优质内容。

# 下载本书的免费 PDF 版本

感谢购买本书！

你喜欢在旅途中阅读，但又无法随时携带纸质书籍吗？

你的电子书购买与所选设备不兼容吗？

不用担心，现在每本 Packt 书籍都可以免费获得该书的无 DRM PDF 版本。

在任何地方、任何设备上阅读。搜索、复制并将你最喜欢的技术书籍中的代码直接粘贴到你的应用程序中。

好处不仅仅是这些，你还可以每天在邮箱中独享折扣、新闻通讯和精彩的免费内容。

按照这些简单的步骤获取福利：

1.  扫描二维码或访问以下链接

![](img/B18644_QR_Free_PDF.jpg)

[`packt.link/free-ebook/978-1-80323-265-2`](https://packt.link/free-ebook/978-1-80323-265-2)

1.  提交你的购买凭证

1.  就这样！我们会直接将你的免费 PDF 和其他福利发送到你的邮箱。

# 第一部分：Jira 简介

本节中，你将学习如何从零开始设置 Jira 实例，并了解如何将 Jira 用于你的业务和敏捷项目。

本节包括以下章节：

+   *第一章*，*开始使用 Jira 数据中心*

+   *第二章*，*将 Jira 用于业务项目*
