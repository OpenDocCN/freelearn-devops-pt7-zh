# 前言

Atlassian Jira 是一种企业级问题跟踪系统。 它的一个关键优势是能够根据组织的需求进行适应，从前端用户界面到为第三方应用程序提供扩展其功能的平台。 然而，理解其灵活性并选择合适的应用程序通常是许多管理员面临的一项艰巨任务。 学会如何在保持整体设计简洁清晰的同时，充分利用 Jira 的强大功能，对于实施的成功和未来的增长至关重要。

通过本书，你可以充分利用有用的教程来解决实际 Jira 管理中的挑战，提供解决方案和示例。 每个教程都包含易于跟随的逐步说明和实际 Jira 应用中的插图。

# 本书适合谁阅读

本书面向那些定制、支持和维护 Jira 的管理员。

你需要熟悉并理解 Jira 的核心概念。 对于某些教程，基本了解 HTML、CSS、JavaScript 以及基本编程（Java 或 Groovy）也会有所帮助。

# 本书内容

第一章，*Jira 服务器管理*，包含了帮助你管理 Jira 服务器的教程，包括升级 Jira 和使用 SSL 证书保护 Jira。

第二章，*为你的项目定制 Jira*，包含了允许你通过自定义字段和屏幕来定制 Jira 的教程。 本章还包括高级技术，如使用脚本和第三方应用程序为 Jira 无法提供的字段添加更多控制。

第三章，*Jira 工作流*，涵盖了 Jira 中最强大的功能之一，并提供了工作流操作的教程，包括权限和用户输入验证。 本章还介绍了有用的第三方应用程序以及使用脚本扩展现成组件的方法。

第四章，*用户管理*，解释了 Jira 中如何管理用户和组。 本章从简单的教程开始，涵盖了现成的用户管理功能，并进一步包括 LDAP 集成以及各种单点登录实现等主题。

第五章，*Jira 安全性*，重点介绍了 Jira 提供的不同安全控制功能，包括不同级别的权限和授权控制。 本章还介绍了其他与安全相关的主题，如用户密码策略和电子签名捕获。

第六章，*电子邮件与通知*，解释了 Jira 的电子邮件处理系统，包括出站和入站电子邮件。 本章还介绍了 Jira 的事件系统以及如何扩展基本的事件和模板集合。

第七章，*与 Jira 的集成*，介绍了如何将 Jira 与其他系统集成，包括其他 Atlassian 应用程序和许多其他流行的云平台，如 Google Drive 和 GitHub。

第八章，*Jira 故障排除与管理*，介绍了在 Jira 中排查各种问题的方法。内容包括诊断与权限和通知相关的常见问题，以及更高级的功能，作为管理员，您可以模拟用户以更好地理解问题。

第九章，*Jira Service Desk*，介绍了 Jira 平台的新成员——Jira Service Desk。Jira Service Desk 使您能够将 Jira 实例转变为一个功能齐全的帮助台系统，充分利用 Jira 强大的工作流和其他定制功能。

# 要充分利用本书的内容。

对于安装和升级教程，您需要有最新的 Jira 8 版本，您可以直接从 Atlassian 网站下载，链接如下：

[`www.atlassian.com/software/jira/download`](http://www.atlassian.com/software/jira/download)。

您可能还需要一些其他软件，包括以下内容：

+   Java SDK：您可以从[`java.sun.com/javase/downloads`](http://java.sun.com/javase/downloads)获取。

+   MySQL：您可以从[`dev.mysql.com/downloads`](http://dev.mysql.com/downloads)获取。

对于其他教程，在各自的教程中提供了获取所需工具的详细信息。

# 下载示例代码文件。

您可以从您的账户下载本书的示例代码文件，网址为[www.packtpub.com](http://www.packtpub.com)。如果您是从其他地方购买的本书，您可以访问[www.packtpub.com/support](http://www.packtpub.com/support)，并注册后将文件直接通过电子邮件发送给您。

您可以按照以下步骤下载代码文件：

1.  登录或注册[www.packtpub.com](http://www.packtpub.com/support)。

1.  选择“支持”标签。

1.  点击“代码下载与勘误”。

1.  在搜索框中输入书名，并按照屏幕上的指示操作。

文件下载后，请确保使用最新版本的工具解压或提取文件夹：

+   WinRAR/7-Zip for Windows。

+   Zipeg/iZip/UnRarX for Mac。

+   7-Zip/PeaZip for Linux。

本书的代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Jira-8-Administration-Cookbook-Third-Edition`](https://github.com/PacktPublishing/Jira-8-Administration-Cookbook-Third-Edition)。我们还有其他代码包，来自我们丰富的书籍和视频目录，您可以在**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**查看。快去看看吧！

# 下载彩色图片。

我们还提供了一份 PDF 文件，包含本书中使用的屏幕截图/图表的彩色图像。您可以在此下载：[`static.packt-cdn.com/downloads/9781838558123_ColorImages.pdf`](https://static.packt-cdn.com/downloads/9781838558123_ColorImages.pdf)。

# 使用的约定

本书中使用了许多文本约定。

`CodeInText`：表示文本中的代码字、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟网址、用户输入和 Twitter 用户名。以下是一个例子：“这是一个用于以 HTML 格式发送电子邮件的模板文件，存储在`html`子目录中。”

代码块如下所示：

```
#disable_html_escaping()

$eventTypeName - ($issue.key) $issue.summary
```

当我们希望您特别注意代码块中的某一部分时，相关行或项目将以粗体显示：

```
<blockquote>
  <p>
   #if($comment.body)
      $comment.body
    #else
   <i>No comment</i>

```

任何命令行输入或输出都按以下方式书写：

```
$ mkdir css
$ cd css
```

**粗体**：表示一个新术语、重要词汇或您在屏幕上看到的单词。例如，菜单或对话框中的单词会这样显示。以下是一个例子：“您可以通过点击**通知**链接来更新此默认方案的通知设置。”

警告或重要说明如下所示。

提示和技巧如下所示。

# 章节

在本书中，您会看到几个经常出现的标题（*准备工作*，*如何做...*，*它是如何工作的...*，*还有更多...*，以及*参见*）。

为了清晰地给出如何完成食谱的指示，请按照以下结构使用这些章节：

# 准备工作

本节告诉您在食谱中应预期什么，并描述如何设置任何所需的软件或任何前期设置。

# 如何做…

本节包含了执行食谱所需的步骤。

# 它是如何工作的…

本节通常包含对上一节内容的详细解释。

# 还有更多…

本节包含关于食谱的附加信息，以帮助您更深入了解该食谱。

# 参见

本节提供了一些有用的链接，指向食谱的其他相关信息。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：请发送电子邮件至`feedback@packtpub.com`，并在邮件主题中注明书名。如果您对本书的任何方面有疑问，请通过`questions@packtpub.com`与我们联系。

**勘误**：虽然我们已经尽力确保内容的准确性，但错误总是难免。如果您在本书中发现错误，敬请报告给我们。请访问[www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata)，选择您的书籍，点击勘误提交表格链接，并填写相关信息。

**盗版**：如果您在互联网上发现任何形式的我们作品的非法复制品，请提供相关网址或网站名称，我们将非常感激。请通过`copyright@packtpub.com`与我们联系，并提供相关链接。

**如果你有兴趣成为作者**：如果你在某个领域有专业知识，并且有兴趣写书或为书籍做贡献，请访问 [authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

请留下评论。在阅读并使用本书后，为什么不在你购买本书的网站上留下评论呢？潜在读者可以看到并利用你的公正意见做出购买决定，我们在 Packt 可以了解你对我们产品的看法，而我们的作者也能看到你对他们书籍的反馈。谢谢！

想了解更多关于 Packt 的信息，请访问 [packtpub.com](https://www.packtpub.com/)。
