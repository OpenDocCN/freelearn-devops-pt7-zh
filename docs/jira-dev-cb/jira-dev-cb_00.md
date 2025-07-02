# 前言

本书是掌握 JIRA 扩展与定制的全方位资源。你将学习如何创建自己的 JIRA 插件，定制 JIRA UI 的外观和体验，处理工作流、问题、自定义字段等，功能丰富。

本书首先介绍了简化插件开发过程的配方，接着有一整章专门讲解 JIRA 插件框架，帮助你掌握 JIRA 中的插件。

接下来，我们将学习编写自定义字段插件，创建新的字段类型或自定义搜索器。然后，我们将学习如何编程和定制工作流，将 JIRA 转变为一个更加用户友好的系统。

然后，我们将讨论如何定制 JIRA 的各种搜索功能，例如 JQL、插件中的搜索、管理过滤器等等。

然后，本书转向编程问题；即创建/编辑/删除问题，创建新的问题操作，使用 JIRA API 管理问题上的其他各种操作等等。

在本书的后半部分，你将学习如何通过添加新的标签页、菜单和 Web 项来定制 JIRA，使用 REST、SOAP 或 XML/RPC 接口与 JIRA 通信，并与 JIRA 数据库进行交互。

本书最后一章是关于有用的 JIRA 常见配方。

# 本书内容

第一章，*插件开发流程*，介绍了 JIRA 插件开发过程的基本知识。详细讲解了开发环境的设置、插件的创建、部署和测试等内容。

第二章，*理解插件框架*，详细讲解了 JIRA 的架构，并介绍了各种插件点。它还介绍了如何从源代码构建 JIRA，并扩展或重写现有的 JIRA 功能。

第三章，*使用自定义字段*，介绍了如何通过编程在 JIRA 中创建自定义字段、编写自定义字段搜索器以及与自定义字段相关的各种有用配方。

第四章，*编程工作流*，介绍了编程 JIRA 工作流的各种方法。包括编写新的条件、验证器、后续功能等，并包含与扩展工作流相关的有用配方。

第五章，*JIRA 中的小工具与报告*，涵盖了 JIRA 的报告功能。详细介绍了编写报告、仪表盘小工具等。

第六章，*JIRA 搜索的强大功能*，讲解了 JIRA 的搜索功能及如何通过 JIRA API 扩展搜索功能。

第七章，*编程问题*，介绍了用于编程管理问题的各种 API 和方法。涵盖了 CRUD 操作、处理附件、编程变更日志和问题链接、时间跟踪等内容。

第八章，*自定义 UI*，介绍了扩展和修改 JIRA 用户界面的各种方法。

第九章，*远程访问 JIRA*，介绍了 JIRA 的远程功能——REST、SOAP 和 XML/RPC，并探讨了扩展这些功能的方法。

第十章，*处理数据库*，介绍了 JIRA 的数据库架构，并详细讨论了主要的表格。还探讨了扩展存储、访问或修改数据的不同方式。

第十一章，*有用的实例*，涵盖了一些有用的实例，这些实例不属于前面章节的内容，但足够强大，值得关注！赶紧阅读吧！！

# 本书所需的内容

本书侧重于 JIRA 开发。最低要求你需要以下软件：

+   JIRA 4.x+

+   JAVA 1.6+

+   Maven 2.x

+   Atlassian 插件 SDK

+   选择你喜欢的 IDE。本书中的示例使用了 Eclipse 和 SQL Developer。

有些实例过于简单，不需要使用完整的插件开发流程，书中会在相关部分特别指出！

# 本书适合谁阅读

如果你是 JIRA 开发者或项目经理，想要充分利用 JIRA 的精彩功能，那么这本书非常适合你。

# 约定

本书中有多种文本风格，用于区分不同类型的信息。以下是这些风格的一些示例，以及它们的含义解释。

文本中的代码词汇如下所示：“字段`oldvalue`和`newvalue`通过方法`getChangelogValue`填充。”

代码块如下所示：

```
<!-- entity to represent a single change to an issue. Always part of a change group -->
    <entity entity-name="ChangeItem" table-name="changeitem" package-
     name="">
        <field name="id" type="numeric"/>
        <field name="group" col-name="groupid" type="numeric"/>
        <!—relations and indexes -->
    </entity>
```

当我们希望引起你对代码块中特定部分的注意时，相关行或项目会以粗体显示：

```
<!-- entity to represent a single change to an issue. Always part of a change group -->
    <entity entity-name="ChangeItem" table-name="changeitem" package-
     name="">
 <field name="oldvalue" type="extremely-long"/>
 <!-- a string representation of the new value (i.e. 
 "Documentation" instead of "4" for a component which might be 
 deleted) -->
        <!—relations and indexes -->
    </entity>
```

任何命令行输入或输出如下所示：

```
maven war:webapp

```

**新术语**和**重要词汇**以粗体显示。你在屏幕上、菜单或对话框中看到的词汇，通常会这样显示：“你一定注意到了新的**查看问题**页面。”

### 注意

警告或重要说明会以这样的框框显示。

### 提示

提示和技巧通常是这样的形式。

# 读者反馈

我们始终欢迎读者反馈。告诉我们你对这本书的看法——你喜欢或可能不喜欢的部分。读者反馈对我们开发出你真正能够从中获得最大收益的书籍非常重要。

要给我们发送一般反馈，只需通过电子邮件发送至`<feedback@packtpub.com>`，并在邮件主题中注明书名。

如果您有需要的书籍，并希望我们出版， 请通过[www.packtpub.com](http://www.packtpub.com)上的**建议书名**表单，或者通过电子邮件`<suggest@packtpub.com>`告诉我们。

如果您在某个领域有专业知识并且有兴趣撰写或贡献书籍内容，请查看我们在[www.packtpub.com/authors](http://www.packtpub.com/authors)上的作者指南。

# 客户支持

既然您已经是 Packt 书籍的骄傲拥有者，我们提供了一些帮助，帮助您最大限度地利用您的购买。

# 下载本书的示例代码

您可以从您的账户下载所有购买的 Packt 书籍的示例代码，网址是[`www.PacktPub.com`](http://www.PacktPub.com)。如果您在其他地方购买了本书，您可以访问[`www.PacktPub.com/support`](http://www.PacktPub.com/support)并注册，以便直接通过电子邮件将文件发送给您。

## 勘误

尽管我们已经尽力确保内容的准确性，但错误仍然会发生。如果您在我们的书籍中发现错误——可能是文本或代码中的错误——我们将非常感激您能报告给我们。通过这样做，您可以避免其他读者的困扰，并帮助我们改进后续版本的书籍。如果您发现任何勘误，请访问[`www.packtpub.com/support`](http://www.packtpub.com/support)报告，选择您的书籍，点击**勘误****提交****表单**链接，填写您的勘误详情。一旦您的勘误被验证，您的提交将被接受，并且勘误将被上传到我们的网站，或添加到该书标题下的任何现有勘误列表中。您可以通过选择您的书籍标题来查看任何现有勘误，网址是[`www.packtpub.com/support`](http://www.packtpub.com/support)。

## 盗版

互联网上的版权材料盗版是所有媒体面临的持续问题。在 Packt，我们非常重视版权和许可证的保护。如果您在互联网上遇到我们作品的任何非法复制品，请立即向我们提供地址或网站名称，以便我们采取相应措施。

如果您发现涉嫌盗版的材料，请通过`<copyright@packtpub.com>`与我们联系，并提供链接。

我们感谢您在保护我们作者方面的帮助，以及我们为您提供有价值内容的能力。

## 问题

如果您在书籍的任何部分遇到问题，可以通过`<questions@packtpub.com>`联系我们，我们将尽力解决。
