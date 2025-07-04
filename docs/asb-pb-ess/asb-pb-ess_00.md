# 前言

随着云计算的发展、敏捷开发方法的应用，以及近年来数据量的爆炸性增长，对大规模管理基础设施的需求也与日俱增。DevOps 工具和实践已成为自动化管理这种可扩展、动态且复杂的基础设施的必需品。配置管理工具是这一 DevOps 工具集的核心。

Ansible 是一个简单、高效、快速的配置管理、编排和应用部署工具，集多种功能于一身。本书帮助你熟悉编写 Playbook，Ansible 的自动化语言。本书采取实践方法，向你展示如何创建灵活、动态、可重用且基于数据驱动的角色。接下来，本书介绍了 Ansible 的高级功能，如节点发现、集群、通过 Vault 安全存储数据以及管理环境，最后展示如何使用 Ansible 编排多层基础设施堆栈。

# 本书内容

第一章，*构建你的基础设施蓝图*，将向你介绍 Playbook、YAML 等内容。你还将学习 Playbook 的组成部分。

第二章，*使用 Ansible 角色实现模块化*，将演示如何使用 Ansible 角色创建可复用的模块化自动化代码，角色是自动化的单元。

第三章，*分离代码和数据 – 变量、事实和模板*，涵盖了使用模板和变量创建灵活、可定制、基于数据驱动的角色。你还将学习自动发现的变量，即事实。

第四章，*引入你的代码 – 自定义命令和脚本*，介绍了如何将现有脚本引入并使用 Ansible 执行 Shell 命令。

第五章，*控制执行流程 – 条件语句*，讨论了 Ansible 提供的控制结构，用于改变执行方向。

第六章，*迭代控制结构 – 循环*，演示了如何使用全能的 `with` 语句遍历数组、哈希表等。

第七章，*节点发现与集群*，讨论了如何发现拓扑信息，并使用魔法变量和事实缓存创建动态配置。

第八章，*使用 Vault 加密数据*，讨论了如何通过 Ansible-vault 存储和共享安全的变量，特别是在版本控制系统中。

第九章，*管理环境*，介绍了如何使用 Ansible 创建和管理隔离的环境，并将自动化代码映射到软件开发工作流中。

第十章，*使用 Ansible 编排基础设施*，介绍了 Ansible 的编排功能，例如滚动更新、预任务和后任务、标签、将测试构建到 playbook 中等内容。

# 本书所需的内容

本书假设你已经安装了 Ansible 并且熟悉 Linux/Unix 环境及系统操作，且对命令行界面也很熟悉。

# 本书适合谁阅读

本书的目标读者是有几年经验的系统或自动化工程师，负责管理基础设施的各个部分，包括操作系统、应用配置和部署。本书也面向那些希望以最短的学习曲线，自动化地管理系统和应用配置的任何人。

假设读者已对 Ansible 有一定的概念理解，已安装 Ansible，并熟悉基本操作，例如创建库存文件和使用 Ansible 运行临时命令。

# 规范

本书中，你会看到几种不同样式的文本，它们区分了不同种类的信息。以下是这些样式的一些示例及其含义的解释。

文本中的代码词汇如下所示：“我们可以通过使用 `include` 指令来包含其他上下文。”

一段代码块的设置如下：

```
---
# site.yml : This is a sitewide playbook
- include: www.yml
```

任何命令行输入或输出都如下所示：

```
$ ansible-playbook simple_playbook.yml -i customhosts

```

**新术语** 和 **重要词汇** 以粗体显示。你在屏幕上看到的单词，例如菜单或对话框中的单词，通常会像这样出现在文本中：“结果变量哈希应包含**默认值**和来自**变量**的覆盖值”。

### 注意

警告或重要的说明会以像这样的框显示。

### 提示

小技巧和窍门以这种形式出现。

# 读者反馈

我们始终欢迎读者反馈。告诉我们你对本书的看法——喜欢哪些部分或不喜欢哪些部分。读者反馈对我们至关重要，帮助我们开发出更符合你需求的书籍。

若要向我们提供一般反馈，只需发送电子邮件至 `<feedback@packtpub.com>`，并在邮件主题中提及书名。

如果你在某个主题上有专业知识，并且有兴趣撰写或贡献书籍，请查看我们作者指南：[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在你已经成为一本 Packt 书籍的骄傲拥有者，我们为你提供了许多资源，帮助你充分利用你的购买。

## 下载示例代码

您可以从您的账户中下载所有已购买的 Packt 书籍的示例代码文件，网址为[`www.packtpub.com`](http://www.packtpub.com)。如果您在其他地方购买了本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便文件直接通过电子邮件发送给您。

## 勘误

尽管我们已尽力确保内容的准确性，但难免会出现错误。如果您在我们的书籍中发现错误——可能是文本或代码错误——我们将非常感激您向我们报告。这样，您可以帮助其他读者避免困扰，并帮助我们改进后续版本的书籍。如果您发现任何勘误，请访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)报告，选择您的书籍，点击**勘误提交表单**链接，并填写勘误的详细信息。一旦您的勘误被验证，您的提交将被接受，勘误将被上传到我们的网站，或添加到该书籍的现有勘误列表中，显示在该书的勘误部分。您可以通过访问[`www.packtpub.com/support`](http://www.packtpub.com/support)查看任何现有的勘误。

## 盗版

网络上侵犯版权的盗版问题在所有媒体中普遍存在。在 Packt，我们非常重视版权和许可证的保护。如果您在互联网上发现我们作品的任何非法复制，无论形式如何，请立即提供该位置地址或网站名称，以便我们采取措施。

如果您发现涉嫌盗版的材料，请通过`<copyright@packtpub.com>`联系我们，并附上该材料的链接。

我们感谢您的帮助，保护我们的作者，并帮助我们继续为您提供有价值的内容。

## 问题

如果您在本书的任何方面遇到问题，请通过`<questions@packtpub.com>`与我们联系，我们将尽力解决。
