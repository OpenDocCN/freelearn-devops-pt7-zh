# 序言

Gitolite 是一个非常流行的 Git 仓库服务器细粒度访问控制工具。它速度快、低调且占用资源小，但提供了多项功能，使得管理员的工作变得更加轻松。

本书帮助你快速掌握 Gitolite。它易于阅读，从基本的安装过程到高级功能（如镜像）都讲解得通顺流畅。特别是更强大和复杂的功能是逐步构建的，帮助你更直观地理解它。任何正在使用或考虑使用 Gitolite 的人都将从本书中受益。

# 本书内容简介

第一章，*Gitolite 入门*，向你展示了 Gitolite 的实用性，以及一些基本的访问控制规则和一些高级功能的示例。它还教你如何创建一个 Gitolite 的测试实例，帮助你安全地进行尝试。

第二章，*安装 Gitolite*，讨论了 Gitolite 的安装和基本管理任务，如添加新用户和创建新仓库。

第三章，*用户与 Gitolite*，讨论了用户如何看待 Gitolite 管理的系统，以及他们开始使用它所需了解的内容。它还提供了关于如何查找和修复 ssh 密钥问题的有用信息。

第四章，*添加与删除用户*，详细介绍了用户管理以及添加用户时背后发生的事情。还讨论了特殊情况，如拥有多个密钥的用户或需要完整 shell 命令行的用户。

第五章，*管理仓库*，讲解了如何添加新仓库，以及如何将现有仓库纳入 Gitolite 控制之下。

第六章，*入门访问控制*，展示了 Gitolite 大多数基本的访问控制功能，包括各种类型的访问规则和配置文件语法。

第七章，*高级访问控制与配置*，深入探讨了个人分支、设置 Git 配置变量和 Gitolite 选项等高级功能。它还讨论了如何使 Gitolite 影响 gitweb 和 git-daemon 的操作。

第八章，*允许用户创建仓库*，讨论了 Gitolite 可能最受欢迎和最重要的功能之一。它讲解了如何允许用户创建自己的仓库，访问规则是如何工作的，以及仓库创建者如何允许其他人访问仓库。

第九章，*自定义 Gitolite*，展示了管理员如何利用 Gitolite 的自定义功能为他们的站点增加独特的功能，例如命令和触发器。

第十章，*理解 VREFs*，讲解了 Gitolite 在用户推送到仓库时使用任意因素来决定允许或拒绝的能力，通常只需要写几行代码。

第十一章，*镜像*，探讨了 Gitolite 的一个功能，该功能在拥有全球各地开发者共同工作的多站点大规模设置中非常有用。Gitolite 镜像功能非常灵活，本章将帮助你开始充分利用它。

# 这本书所需的内容

你将需要一个兼容 POSIX 的 "sh" 的 Unix 系统，Git 版本 1.7.8 或更高版本，以及 Perl 5.8.8 或更高版本。你还需要安装 Openssh 服务器（或兼容的 ssh 服务器）。

# 本书的读者群体

本书非常适合任何希望安装和使用 Gitolite 的人。已经在使用它的人，若希望超越基础并了解其更强大的功能，也会发现许多有用的信息和见解。

# 约定

在本书中，你将看到多种文本样式，它们区分了不同类型的信息。以下是一些样式示例及其含义的解释。

文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入以及 Twitter 账号会如下所示：“将一个非裸仓库转换为裸仓库的一种方式是使用 `--bare` 选项进行克隆。”

所有命令行输入或输出会如下所示：

```
repo    gitolite-admin
 RW+     =   adam
repo    testing
 RW+     =   @all

```

**新术语** 和 **重要词汇** 会以粗体显示。你在屏幕上看到的词语，比如菜单或对话框中的词汇，会像这样出现在文本中：“点击 **下一步** 按钮会将你带到下一个页面。”

### 注意

警告或重要说明会以类似这样的框显示。

### 提示

提示和技巧会像这样显示。

# 读者反馈

我们始终欢迎读者的反馈。让我们知道你对这本书的看法——你喜欢什么，或者可能不喜欢什么。读者反馈对我们来说非常重要，有助于我们开发出你真正能从中获得最大收益的书籍。

若要向我们发送一般反馈，只需发送电子邮件至 `<feedback@packtpub.com>`，并在邮件主题中提及书名。

如果你在某个领域有专业知识，并且有兴趣写作或为一本书提供贡献，请参阅我们的作者指南，网址为 [www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

既然你已经拥有了一本 Packt 书籍，我们提供了一些帮助，让你能最大化地利用你的购买。

## 勘误

虽然我们已经尽最大努力确保内容的准确性，但错误还是会发生。如果您在我们的书籍中发现错误——可能是文本或代码中的错误——我们将非常感激您向我们报告。这样，您可以帮助其他读者避免困扰，并帮助我们改进书籍的后续版本。如果您发现任何勘误，请访问 [`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata) 提交，选择您的书籍，点击 **勘误提交表单** 链接，并填写勘误的详细信息。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站，或添加到该书标题下的勘误列表中。任何现有的勘误都可以通过访问 [`www.packtpub.com/support`](http://www.packtpub.com/support) 查看。

## 盗版

网络上的版权资料盗版问题是一个持续存在的挑战，涉及所有媒体形式。在 Packt，我们非常重视保护我们的版权和许可。如果您在互联网上发现任何我们作品的非法复制，无论形式如何，请立即向我们提供该位置地址或网站名称，以便我们采取相应措施。

如果您发现疑似盗版的资料，请通过 `<copyright@packtpub.com>` 联系我们，并提供相关链接。

感谢您帮助我们保护作者的权益，使我们能够为您提供有价值的内容。

## 问题

如果您在书籍的任何方面遇到问题，可以通过 `<questions@packtpub.com>` 联系我们，我们将尽力为您解决。
