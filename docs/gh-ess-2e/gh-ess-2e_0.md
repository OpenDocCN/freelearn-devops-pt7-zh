# 前言

GitHub 是领先的代码托管平台，几乎所有开源项目都在其上托管其代码。与 Git 结合使用，它提供了高效的开发工作流，并是开发者首选的工具。

从创建仓库的基础开始，您将学习如何管理问题追踪器，让您的项目可以在此讨论。继续我们的旅程，我们将探讨如何使用 Wiki 并编写丰富的文档来伴随您的项目。组织和团队管理将是下一个重点，然后是拉取请求，这使得 GitHub 如此著名。

接下来，我们将专注于创建托管在 GitHub 上的简单网页，最后，我们将探索可配置给用户和仓库的设置。

# 本书适合谁

本书适合有经验或初学者的开发者，具有 Git 的基本知识。如果您曾想了解 Twitter、Google 甚至 GitHub 等大型项目如何协作编码，那么这本书适合您。

# 本书内容

第一章，*仓库概述及问题追踪使用*，解释了 GitHub 提供的一些主要特性及其用途。问题追踪是项目开发者和/或用户之间沟通的核心。可以将其看作是每个仓库专用的记事本，用于跟踪错误、报告、功能请求以及所有能够记录的其他内容。GitHub 还实施了许多其他功能，如标签和里程碑，用于更好地可视化和分类所有问题。

第二章，*使用 Wiki 和管理代码版本*，帮助您学习如何创建、编辑和维护 Wiki，为您的文档提供一个家。您还将学习如何从现有分支或标签创建新的发布，并附带可选的发布说明。通过这种方式，最终用户可以了解与任何先前版本的变更。

第三章，*管理组织和团队*，教您如何创建和管理您拥有的组织。您还将学习如何创建团队，向其添加用户，并根据需要分配不同的访问级别。

第四章，*使用 GitHub 工作流进行协作*，重点介绍如何使用分支和拉取请求，这是 GitHub 最强大的功能。

第五章，*GitHub Pages 和 Web 分析*，详细介绍了如何围绕您的项目构建网页，这些网页专门托管在 GitHub 上。您可以使用 HTML、CSS 和 JavaScript 创建静态网页。

第六章，*探索用户和仓库设置*，探讨了用户和仓库最常见和最基本的设置。作为用户，你可以在用户设置页面上设置很多信息，例如将多个电子邮件与帐户关联，添加多个 SSH 密钥，或设置双重身份验证。类似地，仓库的一些功能可以通过其设置页面进行设置。例如，你可以启用或禁用 wiki 页面，并授予公众写权限，或者完全禁用问题跟踪器。

# 充分利用本书

本书需要 Git（任何版本均可）和一个 GitHub 帐户。

# 下载示例代码文件

你可以从你的帐户在 [www.packtpub.com](http://www.packtpub.com) 下载本书的示例代码文件。如果你在其他地方购买了本书，你可以访问 [www.packtpub.com/support](http://www.packtpub.com/support) 并注册，将文件直接通过电子邮件发送给你。

你可以按照以下步骤下载代码文件：

1.  登录或注册到 [www.packtpub.com](http://www.packtpub.com/support)。

1.  选择“支持”标签。

1.  点击“代码下载与勘误表”。

1.  在搜索框中输入书名，并按照屏幕上的说明进行操作。

下载文件后，请确保使用最新版本的以下工具解压或提取文件夹：

+   WinRAR/7-Zip for Windows

+   Zipeg/iZip/UnRarX for Mac

+   7-Zip/PeaZip for Linux

本书的代码包也托管在 GitHub 上，网址为 [`github.com/PacktPublishing/GitHub-Essentials-Second-Edition`](https://github.com/PacktPublishing/GitHub-Essentials-Second-Edition)。如果代码有更新，将会在现有的 GitHub 仓库中进行更新。

我们还提供了来自我们丰富书籍和视频目录的其他代码包，网址为 **[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**。快去看看吧！

# 下载彩色图像

我们还提供了一份 PDF 文件，其中包含本书中使用的截图/图表的彩色图像。你可以在此下载：[`www.packtpub.com/sites/default/files/downloads/GitHubEssentialsSecondEdition_ColorImages.pdf`](http://www.packtpub.com/sites/default/files/downloads/GitHubEssentialsSecondEdition_ColorImages.pdf)。

# 使用的约定

本书中使用了若干文本约定。

`CodeInText`：表示文本中的代码词、数据库表名、文件夹名称、文件名、文件扩展名、路径名、虚拟 URL、用户输入以及 Twitter 用户名。示例：“将下载的 `WebStorm-10*.dmg` 磁盘映像文件挂载为系统中的另一个磁盘。”

代码块的格式如下：

```
echo "\n## Description\n\nGitHub for dummies" >> README.md
git add README.md
git commit -m "Add second level header to README file"
git push origin add_description
```

当我们希望引起你对某一代码块中特定部分的注意时，相关行或项会用**粗体**标出：

```
echo "\n## Description\n\nGitHub for dummies" >> README.md
git add README.md
git commit -m "Add second level header to README file"
git push origin add_description
```

任何命令行输入或输出都将按以下方式显示：

```
mkdir -p ~/github-essentials
cd $_
```

**粗体**：表示新术语、重要词汇或屏幕上看到的词汇。例如，菜单或对话框中的词汇会以这种方式显示在文本中。以下是一个示例：“从管理面板中选择系统信息。”

警告或重要说明如下所示。

提示和技巧如下所示。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：请通过电子邮件`feedback@packtpub.com`与我们联系，并在邮件主题中注明书名。如果您对本书的任何方面有疑问，请通过`questions@packtpub.com`与我们联系。

**勘误表**：尽管我们已经尽力确保内容的准确性，但错误仍然可能发生。如果您在本书中发现任何错误，欢迎您向我们报告。请访问 [www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata)，选择您的书籍，点击勘误表提交链接，并填写相关信息。

**盗版**：如果您在互联网上遇到任何我们作品的非法复制品，恳请您提供该位置地址或网站名称。请通过`copyright@packtpub.com`与我们联系，并附上材料的链接。

**如果您有兴趣成为作者**：如果您在某个领域拥有专业知识，并且有意愿写书或为书籍做贡献，请访问 [authors.packtpub.com](http://authors.packtpub.com/)。

# 评价

请留下评价。在您阅读并使用本书后，为什么不在您购买本书的网站上留下评价呢？潜在读者可以看到您的无偏见意见，帮助他们做出购买决策，我们也能了解您对我们产品的看法，而我们的作者则可以看到您对其书籍的反馈。谢谢！

欲了解更多关于 Packt 的信息，请访问 [packtpub.com](https://www.packtpub.com/)。
