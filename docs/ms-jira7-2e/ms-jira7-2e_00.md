# 序言

JIRA 是 Atlassian 推出的一个流行的缺陷跟踪工具，具有出色的自定义能力和对各种功能的精细控制。开箱即用，JIRA 提供了问题和缺陷跟踪功能，能够创建任务、分配给用户并生成有用的报告。然而，JIRA 的真正强大之处在于它所提供的自定义功能。

有经验的 JIRA 管理员希望学习高级主题并扩展其知识的将从本书中受益。本书提供了对 JIRA 7 所有组件的全面解释，包括 JIRA Software、JIRA Core 和 JIRA Service Desk。

本书包含了实际案例和使用场景，你将首先学习如何规划 JIRA 安装。接着，你将快速回顾基础概念，并详细理解自定义内容，包括各种使用场景的示例数据，以及 JIRA 管理的各个方面，如用户管理、组、角色和安全级别，重点考虑企业应用。接下来，本书将带你学习如何进行插件开发，以扩展 JIRA 的功能，并介绍如何使用 REST API 在 JIRA 之上构建应用程序。还将讨论如何通过 CSV 文件从其他工具迁移数据。本书还专门有一部分内容，介绍 JIRA Service Desk 应用的实施，这是一个非常受欢迎的支持请求和票务系统插件。

本书还将讨论 Scrum 和 Kanban 技术的实施，以及敏捷报告。我们将介绍 Groovy 脚本，这是一种强大的工具，可以为 JIRA 管理员提供巨大的灵活性。此外，我们还将探讨一些常见的数据库表格，用于获取有用的结果，并讨论在 JIRA 实例中添加自定义 CSS 和 JavaScript 的可能性。最后，我们将通过最佳实践和故障排除总结本书，帮助你找出问题所在并解决它。

# 本书内容概述

第一章，*规划你的 JIRA 安装*，涵盖了如何规划 JIRA 安装，以确保安装的长期性，使其未来能够容纳更多用户和数据。本章还简要讨论了安装和更新过程。

第二章，*在 JIRA 中搜索*，详细解释了如何使用基本搜索功能从 JIRA 中获取数据，并通过编写高级查询使用 JQL。

第三章，*报告 – 使用图表可视化数据*，涵盖了 JIRA 中内置的各种项目报告，以及如何将它们展示在仪表板上。

第四章, *定制 JIRA 以进行测试管理*，说明了如何修改配置以实现新的问题类型，用于测试活动和测试用例。还详细讨论了实施具有条件的新工作流程以及新权限方案的过程。

第五章, *理解 Zephyr 及其特性*，详细演示了在 JIRA 中管理测试的流程实现。

第六章, *使用案例的示例实施*，提供了许多不同实施的例子，例如帮助台系统和需求管理，读者可以在其公司中加以利用。

第七章, *用户管理、组和项目角色*，解释了如何管理 JIRA 中的用户以及如何将它们组织成各种组。

第八章, *配置 JIRA 用户目录以连接 LDAP、Crowd 和 JIRA 用户服务器*，讨论了如何将您的 JIRA 实例与 LDAP 和 Crowd 集成，用于外部用户管理。

第九章, *JIRA 插件开发和利用 REST API*，说明了如何开始为 JIRA 开发插件以扩展功能。还讨论了 JIRA REST API 的使用，该 API 使我们可以从外部工具访问 JIRA 功能，并提供了示例。

第十章, *在 JIRA 中导入和导出数据及迁移配置*，讲述了如何使用 CSV 导入和项目导入功能从外部工具导入数据。本章还解释了定期备份的重要性以及从备份文件恢复 JIRA 的步骤。

第十一章, *在 JIRA 软件中使用敏捷看板*，解释了如何在 JIRA 中实施 Scrum 和 Kanban 技术。详细讨论了在这些看板上进行 Sprint 计划以及各种定制功能，还介绍了跟踪项目进度的燃尽图和速度图表。

第十二章, *使用 Script Runner 和 CLI 插件进行 JIRA 管理*，介绍了管理员可以安装的插件以及使用脚本进行的各种额外功能，这些功能帮助管理员进行各种定制，这些定制在其他情况下是不可能的。

第十三章，*数据库访问*，解释了如何直接从 JIRA 数据库中获取数据。本章包含了许多有用的查询，用于从数据库中检索信息。同时还解释了如何从嵌入的 HSSQL 数据库访问数据。

第十四章，*定制外观、感觉与行为*，讨论了如何通过自定义样式表对 JIRA 设计进行极端修改，同时还解释了如何使用 JavaScript 控制 HTML 字段。

第十五章，*实施 JIRA Service Desk*，解释了如何配置和设置 JIRA Service Desk 应用程序以处理您的支持请求。

第十六章，*将 JIRA 与常见的 Atlassian 应用程序及其他工具集成*，提供了如何将 JIRA 与 Confluence、SVN 和 Git 连接的信息。

第十七章，*JIRA 最佳实践*，讨论了 JIRA 管理员应牢记的各种事项，不仅仅是在实施 JIRA 之前，还包括他们在持续过程中应遵循的各种实践。

第十八章，*故障排除 JIRA*，介绍了识别实例中问题的各种方法。本章列出了人们在 JIRA 中常遇到的问题。

# 本书所需内容

要安装和运行 JIRA，需要以下软件和工具：

+   JIRA 7.1.1 或更高版本

+   MySQL 5.6 或更高版本

+   Java 1.8 或更高版本

+   PHP 5.4

+   Chrome 7 或更高版本

+   Firefox 4 或更高版本

在相关章节中，适用时会解释如何获取此软件及其使用方法。

# 本书适用人群

如果您是管理中小型 JIRA 实例的 JIRA 管理员，并且希望学习如何管理企业级实例，那么本书将帮助您扩展知识，并为您提供高级技能。需要先了解 JIRA 的核心概念。此外，基本的 CSS、JavaScript 和 Java 知识将有助于理解。

# 约定

本书中，您会看到多种文本样式，用来区分不同类型的信息。以下是这些样式的几个示例及其含义的解释。

文本中的代码词汇、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 账户名会按如下方式显示：“`atlas-run-standalone` 命令用于为您设置并启动 JIRA 实例。”

一段代码块如下所示：

```
#if ($mentionable)
 $!rendererParams.put("mentionable", true)
 #if ($issue.project.key && $issue.project.key != "")
 $!rendererParams.put("data-projectkey", "$!issue.project.key")
 #end
 #if ($issue.key && $issue.key != "")
 $!rendererParams.put("data-issuekey", "$!issue.key")
 #end
#end
```

任何命令行输入或输出如下所示：

```
atlas-mvn eclipse:eclipse

```

**新术语** 和 **重要单词** 用粗体显示。在屏幕上看到的单词，例如在菜单或对话框中，文本中会像这样显示：“在菜单栏中，点击 **Windows** | **Preferences**。”

### 注意

警告或重要注意事项会以这样的框出现。

### 提示

提示和技巧将以这样的方式显示。

# 读者反馈

我们非常欢迎读者的反馈。告诉我们你对这本书的看法——你喜欢或不喜欢的部分。读者反馈对我们非常重要，因为它帮助我们开发出你真正能从中受益的书籍。如果你想给我们提供反馈，只需通过电子邮件发送至 feedback@packtpub.com，并在邮件主题中提及书名。如果你在某个领域有专长，且有兴趣写作或为书籍贡献内容，请参阅我们的作者指南：[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在你已经成为一本 Packt 书籍的骄傲拥有者，我们提供了一些帮助你最大化利用购买内容的资源。

## 下载示例代码

你可以从你的账户下载本书的示例代码文件，网址是 [`www.packtpub.com`](http://www.packtpub.com)。如果你是从其他地方购买的本书，你可以访问 [`www.packtpub.com/support`](http://www.packtpub.com/support) 并注册，将文件通过电子邮件直接发送给你。

你可以按照以下步骤下载代码文件：

1.  使用你的电子邮件地址和密码登录或注册我们的官方网站。

1.  将鼠标指针悬停在顶部的 **SUPPORT** 标签上。

1.  点击 **Code Downloads & Errata**。

1.  在 **Search** 框中输入书名。

1.  选择你要下载代码文件的书籍。

1.  从下拉菜单中选择你购买这本书的渠道。

1.  点击 **Code Download**。

下载完文件后，请确保使用最新版本的工具解压或提取文件夹：

+   WinRAR / 7-Zip for Windows

+   Zipeg / iZip / UnRarX for Mac

+   7-Zip / PeaZip for Linux

本书的代码包也托管在 GitHub 上，网址为 [`github.com/PacktPublishing/Mastering-JIRA-7-Second-Edition`](https://github.com/PacktPublishing/Mastering-JIRA-7-Second-Edition)。我们还提供了来自我们丰富书籍和视频目录中的其他代码包，地址是 [`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)。去看看吧！

## 下载本书的彩色图片

我们还为你提供了一份包含本书中所有截图/图表彩色图像的 PDF 文件。这些彩色图片将帮助你更好地理解输出的变化。你可以从 [`www.packtpub.com/sites/default/files/downloads/MasteringJIRA7SecondEdition_ColorImages.pdf`](http://www.packtpub.com/sites/default/files/downloads/Bookname_ColorImages.pdf) 下载该文件。

## 勘误表

尽管我们已尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在我们的书籍中发现错误——无论是文本错误还是代码错误——我们将非常感激您向我们报告。通过这样做，您不仅可以帮助其他读者避免困扰，还能帮助我们改进后续版本。如果您发现任何勘误，请通过访问 [`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata) 提交，选择您的书籍，点击**勘误提交表格**链接，并填写勘误的详细信息。一旦您的勘误被验证，您的提交将被接受，勘误将被上传到我们的网站或添加到该书的现有勘误列表中。

若要查看先前提交的勘误表，请访问 [`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support) 并在搜索框中输入书名。相关信息将出现在**勘误表**部分。

## 盗版

互联网版权材料的盗版问题在所有媒体中都是一个持续存在的问题。在 Packt，我们非常重视版权和许可证的保护。如果您在互联网上发现我们作品的任何非法副本，请立即向我们提供该地址或网站名称，以便我们采取相应措施。

请通过电子邮件联系 copyright@packtpub.com，并提供涉嫌盗版材料的链接。

感谢您帮助我们保护作者的权益，以及我们继续为您提供有价值内容的能力。

## 问题

如果您对本书的任何方面有问题，可以通过 questions@packtpub.com 联系我们，我们将尽力解决问题。
