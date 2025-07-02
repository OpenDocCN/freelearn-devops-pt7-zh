

# 第八章：编码最佳实践

Ansible 可以帮助您自动化几乎所有的日常 IT 任务，从应用补丁或部署配置文件等琐事，到部署全新的基础架构作为代码。随着越来越多的人认识到其强大和简单，Ansible 的使用和参与每年都在增长。您将在互联网上找到许多 Ansible playbook、角色、博客文章等示例，并结合像本书这样的资源，您将能够熟练编写自己的 Ansible playbook。

然而，您如何判断在互联网上找到的示例实际上是做事情的好方法？在本章中，我们将为您介绍 Ansible 最佳实践的实际指南，向您展示目录结构和 playbook 布局的最佳实践，如何有效利用清单（特别是在云上），以及如何最好地区分您的环境。通过本章的学习，您应该能够自信地编写从小型单任务 playbook 到复杂环境的大规模 playbook。

在本章中，我们将涵盖以下主题：

+   首选的目录布局

+   区分不同的环境类型

+   定义组和主机变量的正确方法

+   使用顶级 playbook

+   利用版本控制工具

+   设置 OS 和分发差异

+   在 Ansible 版本之间移植

# 技术要求

本章假设您已经像在*第一章*中所述那样使用 Ansible 设置了控制主机，并且您正在使用最新版本；本章中的示例是在 Ansible 2.15 上测试过的。本章还假设您至少有一个额外的主机进行测试；理想情况下，这应该是基于 Linux 的。尽管本章中我们将给出主机名的具体示例，但您可以用您自己的主机名和/或 IP 地址进行替换，如何替换这些内容的详细信息在适当的地方提供。

本章中使用的代码包可在 [`github.com/PacktPublishing/Practical-Ansible-Second-Edition/tree/main/Chapter%208`](https://github.com/PacktPublishing/Practical-Ansible-Second-Edition/tree/main/Chapter%208) 获取。

# 首选的目录布局

正如我们在本书中多次探索的那样，随着你的剧本在大小和规模上的增长，你会更有可能希望将它分成多个文件和目录。一个很好的例子是角色（roles），我们在*第四章*中讲解过的，*剧本与角色*，我们定义了角色，不仅可以让我们重用公共的自动化代码，还能将原本可能庞大的单一剧本分割成更小、更有逻辑组织的可管理的部分。我们还在*第三章*中讲解过了，*定义清单*，即如何定义清单文件，并且也可以将其拆分成多个文件和目录。然而，我们尚未探讨的是，如何将这些内容组合在一起。所有这些都可以在官方的 Ansible 文档中找到：[`docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.xhtml#content-organization`](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.xhtml#content-organization)。

然而，在本章中，让我们通过一个实际示例开始，向你展示如何为一个简单的基于角色的剧本设置目录结构，该剧本具有两个不同的清单——一个用于开发环境，一个用于生产环境（在任何实际使用案例中，你会希望将它们分开；虽然理想情况下，你应该能够在两个环境中执行相同的剧本，以确保一致性和进行测试）。

让我们从构建目录结构开始：

1.  使用以下命令为你的开发清单创建目录树：

    ```
    $ mkdir -p inventories/development/group_vars
    inventories/development/hosts:

    ```

    [app]

    app01.dev.example.com

    app02.dev.example.com

    ```

    ```

1.  为了进一步完善我们的示例，我们将为我们的 app 组添加一个组变量。如*第三章*中讨论的那样，*定义清单*，在我们在前一步创建的`group_vars`目录中创建一个名为`app.yml`的文件：

    ```
    ---
    http_port: 8080
    ```

1.  接下来，使用相同的方法创建一个`production`目录结构：

    ```
    $ mkdir -p inventories/production/group_vars
    hosts in the newly created production directory with the following contents:

    ```

    [app]

    app01.prod.example.com

    app02.prod.example.com

    ```

    ```

1.  现在，我们将为我们的`production`清单定义一个不同的`http_port`组变量。将以下内容添加到`inventories/production/group_vars/app.yml`中：

    ```
    ---
    http_port: 80
    ```

这完成了我们的清单定义。接下来，我们将添加可能对我们的剧本有用的任何自定义模块或插件。假设我们想使用在*第五章*中创建的`remote_filecopy.py`模块，*创建与使用* *模块*。正如我们在那一章中讨论的那样，我们首先为此模块创建目录：

```
$ mkdir library
```

接下来，我们将`remote_filecopy.py`模块添加到这个库中。为了节省空间，我们不在这里重新列出代码，但你可以从*第五章*中的*开发自定义模块*部分复制它，或者使用本书随附的 GitHub 示例代码。

对于插件，同样的操作也可以进行；如果我们还想使用在*第六章*中创建的`filter`插件，*创建和使用* *插件*，我们将创建一个适当命名的目录：

```
$ mkdir filter_plugins
```

然后，将`filter`插件代码复制到该目录中。

最后，我们将创建一个角色，用于我们新的 playbook 结构。当然，你将有许多角色，但我们先创建一个作为示例，然后你可以为每个角色重复这个过程。我们将角色命名为`installapp`，并使用`ansible-galaxy`命令（在*第四章*中讲解，*Playbooks 和角色*）来为我们创建目录结构：

```
$ mkdir roles
$ ansible-galaxy role init --init-path roles/ installapp
- Role installapp was created successfully
```

然后，在我们的`roles/installapp/tasks/main.yml`文件中，我们将添加以下内容：

```
---
- name: Display http_port variable contents
  debug:
    var: http_port
- name: Create /tmp/foo
  file:
    path: /tmp/foo
    state: file
- name: Use custom module to copy /tmp/foo
  remote_filecopy:
    source: /tmp/foo
  dest: /tmp/bar
- name: Define a fact about automation
  set_fact:
    about_automation: "Puppet is an excellent automation tool"
- name: Tell us about automation with a custom filter applied
  debug:
    msg: "{{ about_automation | improve_automation }}"
```

在前面的代码中，我们重复使用了本书前几章中的一些示例。你也可以根据之前的讨论，将处理程序、变量、默认值等定义到角色中，但对于我们的示例来说，这样就足够了。

创建最佳实践目录结构的最后一步是添加一个顶层 playbook 来运行。按照约定，这个 playbook 会被命名为`site.yml`，并且它将包含以下简单内容（注意，我们所构建的目录结构处理了许多事情，使得顶层 playbook 变得非常简单）：

```
---
- name: Play using best practice directory structure
hosts: all
  roles:
    - installapp
```

为了清晰起见，最终的目录结构应该如下所示：

```
.
├── filter_plugins
│ ├── custom_filter.py
│ └── custom_filter.pyc
├── inventories
│ ├── development
│ │ ├── group_vars
│ │ │ └── app.yml
│ │ ├── hosts
│ │ └── host_vars
│ └── production
│ ├── group_vars
│ │ └── app.yml
│ ├── hosts
│ └── host_vars
├── library
│ └── remote_filecopy.py
├── roles
│ └── installapp
│ ├── defaults
│ │ └── main.yml
│ ├── files
│ ├── handlers
│ │ └── main.yml
│ ├── meta
│ │ └── main.yml
│ ├── README.md
│ ├── tasks
│ │ └── main.yml
│ ├── templates
│ ├── tests
│ │ ├── inventory
│ │ └── test.yml
│ └── vars
│ └── main.yml
└── site.yml
```

现在，我们可以以正常的方式运行我们的 playbook。例如，要在`development`清单上运行它，请执行以下命令：

```
$ ansible-playbook -i inventories/development/hosts site.yml
PLAY [Play using best practice directory structure] ****************************
TASK [Gathering Facts] *********************************************************
ok: [app02.dev.example.com]
ok: [app01.dev.example.com]
TASK [installapp : Display http_port variable contents] ************************
ok: [app01.dev.example.com] => {
 "http_port": 8080
}
ok: [app02.dev.example.com] => {
 "http_port": 8080
}
TASK [installapp : Create /tmp/foo] ********************************************
changed: [app02.dev.example.com]
changed: [app01.dev.example.com]
TASK [installapp : Use custom module to copy /tmp/foo] *************************
changed: [app02.dev.example.com]
changed: [app01.dev.example.com]
TASK [installapp : Define a fact about automation] *****************************
ok: [app01.dev.example.com]
ok: [app02.dev.example.com]
TASK [installapp : Tell us about automation with a custom filter applied] ******
ok: [app01.dev.example.com] => {
 "msg": "Ansible is an excellent automation tool"
}
ok: [app02.dev.example.com] => {
 "msg": "Ansible is an excellent automation tool"
}
PLAY RECAP *********************************************************************
app01.dev.example.com : ok=6 changed=2 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
app02.dev.example.com : ok=6 changed=2 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

同样地，针对`production`清单运行以下命令：

```
$ ansible-playbook -i inventories/production/hosts site.yml
PLAY [Play using best practice directory structure] ****************************
TASK [Gathering Facts] *********************************************************
ok: [app02.prod.example.com]
ok: [app01.prod.example.com]
TASK [installapp : Display http_port variable contents] ************************
ok: [app01.prod.example.com] => {
 "http_port": 80
}
ok: [app02.prod.example.com] => {
 "http_port": 80
}
TASK [installapp : Create /tmp/foo] ********************************************
changed: [app01.prod.example.com]
changed: [app02.prod.example.com]
TASK [installapp : Use custom module to copy /tmp/foo] *************************
changed: [app02.prod.example.com]
changed: [app01.prod.example.com]
TASK [installapp : Define a fact about automation] *****************************
ok: [app01.prod.example.com]
ok: [app02.prod.example.com]
TASK [installapp : Tell us about automation with a custom filter applied] ******
ok: [app01.prod.example.com] => {
 "msg": "Ansible is an excellent automation tool"
}
ok: [app02.prod.example.com] => {
 "msg": "Ansible is an excellent automation tool"
}
PLAY RECAP *********************************************************************
app01.prod.example.com : ok=6 changed=2 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
app02.prod.example.com : ok=6 changed=2 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

注意，如何为每个清单选择适当的主机和相关变量，以及我们的目录结构是如何井井有条的。这是你布局 playbook 的理想方式，它将确保你可以根据需要扩展它们的规模，而不会让它们变得臃肿、难以管理或排除故障。在下一节中，我们将探讨如何区分不同的环境类型。

# 区分不同环境类型

在几乎每个企业中，你都需要根据类型拆分你的技术环境。例如，你几乎肯定会有一个开发环境，在那里进行所有的测试和开发工作，还有一个生产环境，在那里运行所有稳定的测试代码。这些环境应该（在最佳情况下）使用相同的 Ansible playbooks——毕竟，逻辑是，如果你能在开发环境中成功部署和测试一个应用，那么你应该能够在生产环境中以相同的方式部署它并且同样成功运行。然而，两个环境之间总是存在差异，不仅仅是在主机名上，有时还包括参数、负载均衡器名称、端口号等等——这些差异似乎无穷无尽。

在本章的*首选目录布局*部分，我们介绍了一种使用两个独立的清单目录树来区分开发和生产环境的方法。对于区分这些环境，你应该按照这种方式进行；显然，我们不会重复这些示例，但需要注意的是，当处理多个环境时，你的目标应该是：

+   尝试为所有运行相同代码的环境重用相同的 playbooks。例如，如果你在开发环境中部署了一个 web 应用，你应该确信你的 playbooks 能够在生产环境中部署相同的应用（以及在**质量保证**（**QA**）环境中，或其他可能需要部署的环境中）。

+   这意味着，你不仅在测试应用程序的部署和代码，还在将你的 Ansible playbooks 和角色作为整体测试过程的一部分进行测试。

+   每个环境的清单应该保存在不同的目录树中（正如我们在本章的*首选目录布局*部分中看到的那样），但所有的角色、playbooks、插件和模块（如果使用的话）应该放在相同的目录结构中（这应该适用于两个环境）。

+   不同的环境需要不同的认证凭证是正常的；你应该将这些凭证分开存储，不仅为了安全性，还为了确保 playbooks 不会在错误的环境中意外执行。

+   你的 playbooks 应该像你的代码一样存储在版本控制系统中。这使你能够跟踪变化，并确保每个人都在使用同一份自动化代码。

如果你注意到这些简单的提示，你会发现你的自动化工作流程会成为你业务的真正资产，并确保你所有部署的可靠性和一致性。相反，如果不遵循这些提示，你将面临那些令人生畏的*在开发中可以工作，但在生产中无法工作的*部署失败，这种问题经常困扰着科技行业。接下来，我们将继续在下一节中讨论如何处理主机和组变量的最佳实践——正如我们在*首选目录布局*部分中所看到的，你需要应用这些最佳实践，特别是在处理多个环境时。

# 定义组变量和主机变量的正确方法

在处理组和主机变量时，你可以使用我们在*首选目录布局*部分中使用的基于目录的方法将它们拆分开。然而，在管理这些变量时，有一些额外的注意事项需要你留意。首先，最重要的是，你应当始终注意变量的优先级。关于变量优先级顺序的详细列表可以在[`docs.ansible.com/ansible/latest/user_guide/playbooks_variables.xhtml#variable-precedence-where-should-i-put-a-variable`](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.xhtml#variable-precedence-where-should-i-put-a-variable)中找到。然而，处理多个环境时的关键要点如下：

+   主机变量的优先级总是高于组变量的，因此你可以用主机变量覆盖任何组变量。如果你以受控方式利用这一行为，它是很有用的，但如果你没有意识到这一点，可能会导致意想不到的结果。

+   有一个特别的组变量定义，叫做`all`，它适用于所有库存组。它的优先级低于专门定义的组变量。

+   如果你在两个组中定义相同的变量会发生什么？如果发生这种情况，两个组的优先级顺序是相同的，那么哪个会生效呢？为了演示这一点（以及我们之前的例子），我们将为你创建一个简单的实际示例。

为了开始，我们将为我们的库存创建一个目录结构。为了使这个例子尽可能简洁，我们只会创建一个开发环境。然而，你可以自由地扩展这些概念，构建在本章*首选目录布局*部分中我们涵盖的更完整的例子之上：

1.  使用以下命令创建一个库存目录结构：

    ```
    $ mkdir -p inventories/development/group_vars
    inventories/development/hosts file; the contents should be as follows:

    ```

    [app]

    app01.dev.example.com

    app02.dev.example.com

    ```

    ```

1.  现在，让我们为库存中的所有组创建一个特殊的组变量文件；该文件将命名为`inventories/development/group_vars/all.yml`，并应包含以下内容：

    ```
    ---
    http_port: 8080
    ```

1.  最后，让我们创建一个简单的 playbook，名为`site.yml`，用来查询并打印我们刚刚创建的变量的值：

    ```
    ---
    - name: Play using best practice directory structure
    hosts: all
    tasks:
      - name: Display the value of our inventory variable
        debug:
          var: http_port
    ```

1.  现在，如果我们运行这个 playbook，我们会看到该变量（我们只在一个地方定义的）将会得到我们预期的值：

    ```
    $ ansible-playbook -i inventories/development/hosts site.yml
    PLAY [Play using best practice directory structure] ****************************
    TASK [Gathering Facts] *********************************************************
    ok: [app01.dev.example.com]
    ok: [app02.dev.example.com]
    TASK [Display the value of our inventory variable] *****************************
    ok: [app01.dev.example.com] => {
     "http_port": 8080
    }
    ok: [app02.dev.example.com] => {
     "http_port": 8080
    }
    PLAY RECAP *********************************************************************
    app01.dev.example.com : ok=2 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
    all.yml file remaining unchanged. Let’s also create a new file located in inventories/development/group_vars/app.yml, which will contain the following content:

    ```

    ---

    http_port: 8081

    ```

    ```

1.  我们现在定义了相同的变量两次——一次是在一个名为`all`的特殊组中，另一次是在`app`组中（我们`development`清单中的两个服务器都属于该组）。那么，如果我们现在运行我们的 playbook，会发生什么呢？输出应该如下所示：

    ```
    $ ansible-playbook -i inventories/development/hosts site.yml
    PLAY [Play using best practice directory structure] ****************************
    TASK [Gathering Facts] *********************************************************
    ok: [app02.dev.example.com]
    ok: [app01.dev.example.com]
    TASK [Display the value of our inventory variable] *****************************
    ok: [app01.dev.example.com] => {
     "http_port": 8081
    }
    ok: [app02.dev.example.com] => {
     "http_port": 8081
    }
    PLAY RECAP *********************************************************************
    app01.dev.example.com : ok=2 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
    centos, and another group that could notionally contain hosts built to a new build standard, called newcentos, of which both application servers will be a member. This means modifying inventories/development/hosts so that it now looks as follows:

    ```

    [app]

    app01.dev.example.com

    app02.dev.example.com

    [centos:children]

    app

    [newcentos:children]

    app

    ```

    ```

1.  现在，让我们通过创建一个名为`inventories/development/group_vars/centos.yml`的文件，重新定义`centos`组的`http_port`变量，该文件包含以下内容：

    ```
    ---
    http_port: 8082
    ```

1.  进一步增加困惑的是，让我们还在`inventories/development/group_vars/newcentos.yml`中为`newcentos`组定义这个变量，该文件将包含以下内容：

    ```
    ---
    http_port: 8083
    ```

1.  我们现在在组级别定义了相同的变量四次！让我们重新运行我们的 playbook，看看哪个值会生效：

    ```
    $ ansible-playbook -i inventories/development/hosts site.yml
    PLAY [Play using best practice directory structure] ****************************
    TASK [Gathering Facts] *********************************************************
    ok: [app01.dev.example.com]
    ok: [app02.dev.example.com]
    TASK [Display the value of our inventory variable] *****************************
    ok: [app01.dev.example.com] => {
     "http_port": 8083
    }
    ok: [app02.dev.example.com] => {
     "http_port": 8083
    }
    PLAY RECAP *********************************************************************
    app01.dev.example.com : ok=2 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
    newcentos.yml won—but why? The Ansible documentation states that where identical variables are defined at the group level in the inventory (the one place you can do this), the one from the last-loaded group wins. Just for completeness, we can override all of this by leaving the group_vars directory untouched, but adding a file called inventories/development/host_vars/app01.dev.example.com.yml, which will contain the following content:

    ```

    ---

    http_port: 9090

    ```

    ```

1.  现在，如果我们最后一次运行我们的 playbook，我们会看到在主机级别定义的值完全覆盖了在`app01.dev.example.com`的组级别设置的任何值。`app02.dev.example.com`没有受到影响，因为我们没有为它定义主机变量，因此优先级次高的——来自`newcentos`组的组变量——生效了。

    ```
    $ ansible-playbook -i inventories/development/hosts site.yml
    PLAY [Play using best practice directory structure] ****************************
    TASK [Gathering Facts] *********************************************************
    ok: [app01.dev.example.com]
    ok: [app02.dev.example.com]
    TASK [Display the value of our inventory variable] *****************************
    ok: [app01.dev.example.com] => {
     "http_port": 9090
    }
    ok: [app02.dev.example.com] => {
     "http_port": 8083
    }
    PLAY RECAP *********************************************************************
    app01.dev.example.com : ok=2 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
    app02.dev.example.com : ok=2 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
    ```

有了这些知识，你现在可以做出更高级的决策，关于如何在你的清单中组织变量，确保在主机和组级别都能达到预期的结果。了解变量优先级顺序很重要，正如这些示例所展示的那样，但遵循文档中的顺序还将使你能够生成强大、灵活的 playbook 清单，在多个环境中都能良好工作。现在，你可能已经注意到，在整个章节中，我们在目录结构中使用了一个顶级 playbook，名为`site.yml`。我们将在下一节中更详细地查看这个 playbook。

# 使用顶级 playbooks

在到目前为止的所有示例中，我们都使用了 Ansible 推荐的最佳实践目录结构，并且持续引用了一个顶级的 playbook，通常称为`site.yml`。这个 playbook（以及它在我们所有目录结构中常见的名称）的目的是能够在整个服务器环境中使用，也就是说，它适用于你的**站点**。

当然，这并不是说你必须在你的基础设施中的每个服务器或每个功能上使用相同的剧本集；相反，这仅仅意味着你可以根据自己的环境做出最佳决策。然而，Ansible 自动化的整个目标是创建一个简单易运行和操作的解决方案。试想一下，将包含 100 个不同剧本的剧本目录结构交给一位新的系统管理员——他们如何知道在哪些情况下运行哪些剧本呢？培训某人使用剧本的任务将是巨大的，只会将复杂性从一个领域转移到另一个领域。

在另一个极端，你可以利用`when`子句与事实和清单分组，使得剧本在每种可能的情况下都能精确地知道在每台服务器上运行什么。当然，这种情况不太可能发生，事实是，你的自动化解决方案最终会处于某个中间状态。

最重要的是，当收到新的剧本目录结构时，一个新的操作员至少知道运行剧本和理解代码的起点是什么。如果他们遇到的顶层剧本始终是`site.yml`，那么至少每个人都知道从哪里开始。通过巧妙地使用角色以及`import_*`和`include_*`语句，你可以将剧本拆分为可重用代码的逻辑部分，就像我们之前讨论的那样，所有这些都来自一个剧本文件。

现在你已经了解了顶层剧本的重要性，接下来我们将在下一节中讨论如何利用版本控制工具来确保在集中和维护自动化代码时遵循最佳实践。

# 利用版本控制工具

正如我们在本章早些时候讨论的那样，至关重要的是，你不仅要对代码进行版本控制和测试，还要对你的 Ansible 自动化代码进行版本控制和测试。这应包括清单（或动态清单集合）、任何自定义模块、插件、角色和剧本代码。原因很简单——Ansible 自动化的最终目标很可能是使用剧本（或一组剧本）部署整个环境。这甚至可能涉及将基础设施作为代码进行部署，特别是如果你正在部署到云环境中。

对你 Ansible 代码的任何修改都可能对你的环境产生重大影响，甚至决定一个重要的生产服务是否正常运行。因此，保持 Ansible 代码的版本历史记录，并确保每个人都使用相同版本至关重要。你可以自由选择最适合你的版本控制系统；大多数企业环境已经有了某种版本控制系统。然而，如果你以前没有使用过版本控制系统，建议你注册一个免费的账户，例如 GitHub 或 GitLab，它们都提供免费的版本控制仓库，同时也有更高级的付费计划。

版本控制与 Git 的完整讨论超出了本书的范围，但有许多专门的书籍讲解这个话题。然而，我们会带你了解最简单的用例。在以下示例中，假设你使用的是 GitHub 的免费账户，但如果你使用的是其他提供商，只需将 URL 更改为与你的版本控制仓库主机提供给你的 URL 相匹配即可。

此外，你还需要在 Linux 主机上安装命令行 Git 工具。

在 CentOS 上，你可以通过如下方式安装这些工具：

```
$ sudo yum install git
```

在 Ubuntu 上，过程同样简单：

```
$ sudo apt-get update
$ sudo apt-get install git
```

在 macOS 上，你可以使用`brew`包管理器按如下方式安装 Git 工具：

```
$ brew install git
```

工具安装并设置好账户后，接下来的任务是将 Git 仓库克隆到你的机器上。如果你想开始使用自己的仓库，你需要通过你的提供商设置它——GitHub 和 GitLab 都提供了优秀的文档，你应该按照这些文档设置你的第一个仓库。

设置并初始化后，你可以克隆一份代码到本地机器进行修改。这个本地副本称为工作副本，你可以按如下步骤进行克隆并修改（请注意，这些完全是假设的示例，旨在让你了解需要运行的命令；你应根据自己的用例进行调整）：

1.  使用如下命令将你的 `git` 仓库克隆到本地机器，创建一个工作副本：

    ```
    $ git clone https://github.com/<YOUR_GIT_ACCOUNT>/<GIT_REPO>.git
    Cloning into '<GIT_REPO>'...
    remote: Enumerating objects: 7, done.
    remote: Total 7 (delta 0), reused 0 (delta 0), pack-reused 7
    Unpacking objects: 100% (7/7), done.
    ```

1.  切换到你克隆的代码目录（工作副本），并进行所需的代码更改：

    ```
    $ cd <GIT_REPO>
    $ vim myplaybook.yml
    ```

1.  确保测试你的代码，当你对其满意时，使用如下命令将已修改的文件添加到待提交的新版本中：

    ```
    commit message (specified in quotes after the -m switch), as follows:

    ```

    $ git commit -m '已将新的 spongle-widget 部署添加到 myplaybook.yml'

    [master ed14138] 已将新的 spongle-widget 部署添加到 myplaybook.yml

    提交者：Daniel Oh <doh@danieloh.redhat.com>

    你的姓名和电子邮件地址已自动配置

    在你的用户名和主机名上。请检查它们是否准确。

    你可以通过显式设置它们来抑制此消息。运行以下命令：

    执行以下命令并按照编辑器中的提示进行编辑

    你的配置文件：

    git config --global --edit

    完成此操作后，你可以使用以下命令修复此提交使用的身份：

    git commit --amend --reset-author

    1 个文件已更改，1 个插入（+），1 个删除（-）

    ```

    ```

1.  目前，所有这些更改仅存在于本地机器上的工作副本中。这本身是好的，但如果代码能够在版本控制系统中供需要查看的所有人访问，那会更好。若要将更新的提交推送回（例如）GitHub，请运行以下命令：

    ```
    $ git push
    Enumerating objects: 5, done.
    Counting objects: 100% (5/5), done.
    Delta compression using up to 8 threads
    Compressing objects: 100% (3/3), done.
    Writing objects: 100% (3/3), 297 bytes | 297.00 KiB/s, done.
    Total 3 (delta 2), reused 0 (delta 0)
    remote: Resolving deltas: 100% (2/2), completed with 2 local objects.
    To https://github.com/<YOUR_GIT_ACCOUNT>/<GIT_REPO>.git
    0d00263..ed14138 master -> master
    ```

就是这么简单！

1.  现在，其他协作者可以像我们在*步骤 1*中做的那样克隆你的代码。或者，如果他们已经有了你的仓库的工作副本，他们可以使用以下命令更新他们的工作副本（如果你想更新你的工作副本以查看其他人做的更改，你也可以这样做）：

    ```
    $ git pull
    ```

Git 有一些非常高级的话题和使用案例，超出了本书的范围。然而，你会发现，大约 80%的时间，前面提到的命令就是你所需要的所有 Git 命令行知识。也有许多图形化的 Git 前端工具，以及与 Git 仓库集成的代码编辑器和**集成开发环境**（**IDEs**），它们可以进一步帮助你充分利用 Git。完成这些后，让我们来看一下如何确保你可以在多个主机上使用相同的剧本（或角色），即使它们可能有不同的操作系统和版本。

# 设置操作系统和发行版的差异

正如前面所述，我们的目标是尽可能广泛地使用相同的自动化代码。然而，尽管我们尽力标准化技术环境，但总会有变种悄然出现。例如，无法一次性对所有服务器进行重大升级，因此，当新的操作系统版本发布时，例如**Red Hat Enterprise Linux**（**RHEL**）8 或 Ubuntu Server 20.04，一些机器在其他机器升级时不可避免地会保持旧版本。类似地，一个环境可能已经标准化为 Ubuntu，但随后引入的某个应用程序只能在 CentOS 上运行。总之，尽管标准化非常重要，但差异总是会悄然出现。

在编写 Ansible 剧本时，尤其是角色时，你的目标应该是让它们尽可能广泛地适用于你的环境。一个经典的例子是包管理——假设你正在编写一个安装 Apache 2 web 服务器的角色。如果你必须同时支持 Ubuntu 和 CentOS，不仅需要处理不同的包管理工具（`yum`和`apt`），而且还有不同的包名（`httpd`和`apache2`）。

在 *第四章*，*Playbooks 和 Roles* 中，我们学习了如何使用 `when` 子句应用任务条件，并结合 Ansible 收集的事实（例如 `ansible_distribution`）。然而，还有另一种方法可以在特定主机上运行任务，我们还没有看过。在同一章节中，我们还讨论了如何在一个 playbook 中定义多个 plays——有一个特殊的模块可以根据 Ansible 事实为我们创建库存组，我们可以将其与多个 plays 结合使用，创建一个在每个主机上根据其类型运行适当任务的 playbook。通过一个实际的例子来解释这一点最为清晰，所以我们开始吧。

假设我们为这个例子使用以下简单的库存文件，该文件中有两个主机，位于一个名为 `app` 的单一组中：

```
[app]
app01.dev.example.com
app02.dev.example.com
```

现在让我们构建一个简单的 playbook，演示如何使用 Ansible 事实将组进行分组，使得操作系统分发决定 playbook 中运行哪个 play。

按照以下步骤创建此 playbook，并观察其操作：

1.  首先创建一个新的 playbook——我们将其命名为 `osvariants.yml`——并使用以下 `Play` 定义。它还将包含一个单独的任务，如下所示：

    ```
    ---
    - name: Play to demonstrate group_by module
      hosts: all
      tasks:
      - name: Create inventory groups based on host facts
        group_by:
          key: os_{{ ansible_facts['distribution'] }}
    ```

目前，playbook 结构应该对你来说非常熟悉。然而，`group_by` 模块的使用是新的。它根据我们指定的键动态创建新的库存组——在这个例子中，我们根据一个由 `os_` 固定字符串和 `Gathering Facts` 阶段获取的操作系统分发事实组成的键来创建组。原始的库存组结构得以保留且未被修改，但所有主机也根据它们的事实被添加到新创建的组中。

因此，我们简单库存中的两个服务器仍然保留在 `app` 组中，但如果它们基于 Ubuntu，它们将被添加到一个新创建的名为 `os_Ubuntu` 的库存组中。同样，如果它们基于 CentOS，它们将被添加到一个名为 `os_CentOS` 的组中。

1.  有了这些信息，我们可以继续创建基于新创建组的额外 plays。让我们将以下 `Play` 定义添加到相同的 playbook 文件中，以在 CentOS 上安装 Apache：

    ```
    - name: Play to install Apache on CentOS
      hosts: os_CentOS
      become: true
      tasks:
        - name: Install Apache on CentOS
          yum:
            name: httpd
            state: present
    ```

这是一个完全正常的 `Play` 定义，使用 `yum` 模块安装 `httpd` 包（在 CentOS 上需要）。唯一与我们之前的工作不同的是 play 顶部的 `hosts` 定义。这个定义使用了第一步中 `group_by` 模块创建的新的库存组。

1.  类似地，我们可以添加第三个 `Play` 定义——这次使用 `apt` 模块在 Ubuntu 上安装 `apache2` 包：

    ```
    - name: Play to install Apache on Ubuntu
      hosts: os_Ubuntu
      become: true
      tasks:
        - name: Install Apache on Ubuntu
        apt:
          name: apache2
          state: present
    ```

1.  如果我们的环境基于 CentOS 服务器，并且运行此 playbook，结果如下：

    ```
    $ ansible-playbook -i hosts osvariants.yml
    PLAY [Play to demonstrate group_by module] *************************************
    TASK [Gathering Facts] *********************************************************
    ok: [app02.dev.example.com]
    ok: [app01.dev.example.com]
    TASK [Create inventory groups based on host facts] *****************************
    ok: [app01.dev.example.com]
    ok: [app02.dev.example.com]
    PLAY [Play to install Apache on CentOS] ****************************************
    TASK [Gathering Facts] *********************************************************
    ok: [app01.dev.example.com]
    ok: [app02.dev.example.com]
    TASK [Install Apache on CentOS] ************************************************
    changed: [app02.dev.example.com]
    changed: [app01.dev.example.com]
    [WARNING]: Could not match supplied host pattern, ignoring: os_Ubuntu
    PLAY [Play to install Apache on Ubuntu] ****************************************
    skipping: no hosts matched
    PLAY RECAP *********************************************************************
    app01.dev.example.com : ok=4 changed=2 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
    app02.dev.example.com : ok=4 changed=2 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
    ```

注意安装 Apache 到 CentOS 的任务是如何执行的。它是这样执行的，因为`group_by`模块创建了一个名为`os_CentOS`的组，而我们的第二个 play 仅在名为`os_CentOS`的组中的主机上运行。由于在库存中没有运行 Ubuntu 的服务器，`os_Ubuntu`组从未创建，因此第三个 play 不会运行。我们收到关于没有与`os_Ubuntu`匹配的主机模式的警告，但 playbook 并没有失败——它只是跳过了这一部分。

我们提供了这个例子，展示了另一种管理自动化编码中不可避免的操作系统类型差异的方法。最终，选择最适合你的编码风格是由你来决定的。你可以使用这里详细说明的`group_by`模块，或者将任务写成块，并为这些块添加`when`条件，以便仅在满足某些基于事实的条件时执行（例如，操作系统发行版是 CentOS）——或者也可以两者结合使用。最终的选择是你的，这些不同的示例旨在为你提供多种选择，帮助你为你的场景创建最佳解决方案。

最后，让我们通过查看如何在不同版本的 Ansible 之间移植自动化代码来结束本章。

# 在 Ansible 版本之间移植

Ansible 是一个快速发展的项目，随着版本发布和新功能的增加，新模块（以及模块增强）也会发布，软件中不可避免的 bug 也会被修复。毫无疑问，你最终会在一个版本的 Ansible 上编写代码，然后在某个时刻需要将它运行在更新的版本上。举个例子，当我们开始编写第二版时，当前发布的 Ansible 版本是 2.15。

通常，当你升级代码时，你会发现早期版本的代码*勉强能够运行*，但这并不总是能保证的。模块有时会被弃用（虽然通常会有警告），功能也会发生变化。你可以在这里找到更多关于 2.15 版本发布说明的细节：[`github.com/ansible/ansible/blob/v2.15.2/changelogs/CHANGELOG-v2.15.rst`](https://github.com/ansible/ansible/blob/v2.15.2/changelogs/CHANGELOG-v2.15.rst)。

所以，问题依然存在——如何确保在更新 Ansible 安装时，你的 playbooks、roles、modules 和 plugins 仍然可以正常工作？

答案的第一部分是确定你从哪个版本的 Ansible 开始。例如，假设你正在为 Ansible 2.10 的发布做准备。如果你查询已经安装的 Ansible 版本，并看到如下所示的内容，那么你就知道你是从 Ansible 2.15.0 版本开始的：

```
$ ansible [core 2.15.0]
  config file = None
  configured module search path = ['/Users/danieloh/Library/Python/3.11/bin/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /Users/danieloh/Library/Python/3.11/lib/python/site-packages/ansible
  ansible collection location =
/Users/danieloh/Library/Python/3.11/bin/collections:/usr/share/ansible/collections
  executable location = /Users/danieloh/Library/Python/3.11/bin/ansible
  python version = 3.11.3 (main, Apr  7 2023, 20:13:31) [Clang 14.0.0 (clang-1400.0.29.202)] (/opt/homebrew/opt/python@3.11/bin/python3.11)
  jinja version = 3.1.2
 libyaml = True
```

所以，你的第一个步骤应该是查看 Ansible 版本的移植指南；通常，每个主要版本（如 2.10、8 等）都会有一份移植指南。请注意，Ansible 8 包含 Ansible Core 2.15。

Ansible 8 的指南可以在[`docs.ansible.com/ansible/latest/porting_guides/porting_guide_8.xhtml`](https://docs.ansible.com/ansible/latest/porting_guides/porting_guide_8.xhtml)找到。

如果我们回顾一下这份文档，可以看到有一些即将到来的变化——这些变化是否对你重要，实际上取决于你运行的代码。例如，如果我们回顾指南中的*添加的集合*部分，我们可以看到`grafana.grafana`（版本`2.0.0`）集合已被添加。

正如前面的链接所示，Ansible 的八个版本中存在一些已知问题。为此，同样需要注意的是，迁移指南是从上一个主要版本的升级角度编写的。如果你查询你的 Ansible 版本并返回以下信息，则说明你正在从 Ansible 2.8 进行迁移：

```
$ ansible --version
ansible 2.8.4
 config file = /etc/ansible/ansible.cfg
 configured module search path = [u'/home/james/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
 ansible python module location = /usr/lib/python2.7/site-packages/ansible
 executable location = /usr/bin/ansible
 python version = 2.7.5 (default, Aug 7 2019, 00:51:29) [GCC 4.8.5 20150623 (Red Hat 4.8.5-39)]
```

如果你直接迁移到 Ansible 8（Core 2.15），那么你需要查看 2.9 和 2.10 的迁移指南。2.9 的指南涵盖了 2.8 和 2.9 版本之间代码所需的更改，而 2.10 的指南则涵盖了从 2.9 升级到 2.10 所需的更改。所有迁移指南的索引可以在 Ansible 官方网站找到，地址为[`docs.ansible.com/ansible/devel/porting_guides/porting_guides.xhtml`](https://docs.ansible.com/ansible/devel/porting_guides/porting_guides.xhtml)。

另一个非常重要的信息来源，特别是关于版本之间变化的更详细信息，就是更新日志（changelogs）。这些日志会在每个次要版本发布时更新，并且可以在你想要查询的版本的`stable`分支上找到，位于官方的 Ansible GitHub 仓库中。例如，如果你想查看 Ansible 2.15 的所有更新日志，你需要访问[`github.com/ansible/ansible/blob/stable-2.15/changelogs/CHANGELOG-v2.15.rst`](https://github.com/ansible/ansible/blob/stable-2.15/changelogs/CHANGELOG-v2.15.rst)。

将代码从一个 Ansible 版本迁移到另一个版本的技巧（如果你能称之为技巧的话）就是阅读 Ansible 项目团队发布的优秀文档。创建这些文档付出了大量努力，因此建议你充分利用它。以上就是我们对使用 Ansible 的最佳实践的总结。希望你在这一章中获得了有价值的信息。

# 摘要

Ansible 自动化项目通常从小规模开始，但随着人们逐渐认识到 Ansible 的强大和简易，代码和清单通常会以指数级的速度增长（至少根据我的经验）。在推动更大规模的自动化时，重要的是确保 Ansible 的自动化代码和基础设施本身不会变成另一个麻烦。通过在早期嵌入一些良好的实践，并在整个 Ansible 自动化过程中始终如一地应用它们，你会发现管理 Ansible 自动化是轻松的，并且会对你的技术基础设施带来真正的益处。

在本章中，你学习了按操作系统类型区分环境的新方法，还了解了变量优先级及其在处理主机和组变量时的应用。接着，你探索了顶级 playbook 的重要性，之后学习了如何利用版本控制工具来管理自动化代码。最后，你研究了创建单个 playbook 的新技术，这些 playbook 将管理不同操作系统版本和发行版的服务器，最后你学习了将代码迁移到新的 Ansible 版本这一重要话题。

在下一章，我们将介绍一些更高级的使用 Ansible 的方法，以处理在自动化过程中可能出现的特殊情况。

# 问题

1.  有什么安全且简单的方法来持续管理（即修改、修复和创建）代码变更，并与他人共享？

    1.  Playbook 修订

    1.  任务历史

    1.  临时创建

    1.  使用 Git 仓库

    1.  日志管理

1.  对还是错？Ansible Galaxy 支持从中央社区支持的仓库与其他用户共享角色：

    1.  正确

    1.  错误

1.  对还是错？Ansible 模块在所有未来版本的 Ansible 中都能保证可用：

    1.  正确

    1.  错误

# 深入阅读

通过创建分支和标签来管理多个仓库、版本或任务，以有效控制多个版本。有关更多详情，请参阅以下链接：

+   如何使用 Git 标签：[`git-scm.com/book/en/v2/Git-Basics-Tagging`](https://git-scm.com/book/en/v2/Git-Basics-Tagging%0D)

+   如何使用 Git 分支：[`git-scm.com/docs/git-branch`](https://git-scm.com/docs/git-branch)
