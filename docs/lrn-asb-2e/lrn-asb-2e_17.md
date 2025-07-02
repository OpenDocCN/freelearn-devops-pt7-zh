# 17

# Ansible 的后续步骤

在本章的最后一部分，我们将讨论如何将 Ansible 集成到你的日常工作流程中。我们将讨论持续集成工具、监控工具和故障排除。

我们将讨论以下主题：

+   集成第三方服务

+   如何使用 Ansible 在问题发生时进行故障排除

+   一些实际案例

让我们直接进入主题，看看如何将我们的剧本与第三方服务进行集成。

# 技术要求

本章与之前的章节有所不同。虽然本章和 GitHub 库中给出了代码示例，但它们并不是完整的可运行示例。相反，我们将讨论如何将它们集成到你的项目中，使它们更像是可能的艺术，而不是完全形成的示例。

# 集成第三方服务

虽然你可能自己运行剧本，但保持剧本运行日志或更新其他团队成员或部门的结果是个好主意。Ansible 有多个模块可以帮助你与第三方服务合作，提供实时通知。

让我们从 Slack 开始。

## Slack

Slack 迅速成为不同 IT 部门团队协作服务的首选。Slack 的一个主要优势是通过其应用目录支持第三方应用；Ansible 通过`community.general.slack`模块支持 Slack 的传入 Webhooks。

记住，如果你还没有安装`community.general`集合，你可以运行以下命令来安装它：

```
$ ansible-galaxy collection install community.general
```

在我们查看 Ansible 代码之前，先快速讨论一下如何创建一个 Slack 应用并启用 webhooks。

首先，你需要创建自己的 Slack 应用；你可以访问[`api.slack.com/apps/new`](https://api.slack.com/apps/new)来完成这一步。进入后，点击**创建应用**按钮，并选择**从零开始**选项。接下来，你需要填写**应用名称**和**选择一个工作区来开发你的应用**，对于我们大多数人来说，这将是你的主工作区，正如以下截图所示：

![图 17.1 – 创建 Slack 应用](img/B21620_11_011.jpg)

图 17.1 – 创建 Slack 应用

一旦 Slack 应用创建完成，你将进入新的应用设置页面。在左侧菜单中，你应该看到**传入 Webhooks**的选项。进入该页面并将**启用传入 Webhooks**开关切换为**开启**。这将扩展选项，并允许你**添加新的 Webhook**到**工作区**。

在这里，你需要选择你希望你的 Slack 应用发布到哪里；如以下截图所示，我选择了**#general**频道：

![图 17.2 – 选择发布位置](img/B21620_11_02.jpg)

图 17.2 – 选择发布位置

一旦选择了，你将被带回到你的应用的**Incoming Webhooks**页面；在这里，你将获得一个 Webhook URL，它应该看起来像以下内容，你需要记录并妥善保管（以下的链接已被撤销）：

```
https://hooks.slack.com/services/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
```

现在我们已经准备好与 Slack 交互，我们可以查看代码。正如本章开始时提到的，我只会深入讲解其中的一些代码，因为很多内容你可能已经很熟悉了。

我们只需要添加一个变量，它是用于识别和验证我们创建的 Webhook 的 token：token 是 Webhook URL 中`https://hooks.slack.com/services/`后面的所有内容，因此在我的例子中，我将该变量放在`group_vars/common.yml`中，如下所示：

```
slack:
  token: "TBCRVDMGA/B06RCMPD6R4/YBTo7ZXZHrRg57fvJXr1sg43"
```

由于这个 token 应该被视为机密，我建议也使用 Ansible Vault 来加密其值，因此你可以运行以下命令来实现这一点：

```
$ ansible-vault encrypt_string 'TBCRVDMGA/B06RCMPD6R4/YBTo7ZXZHrRg57fvJXr1sg43' --name 'token'
```

仓库中的 token 使用 Ansible Vault 进行了加密，且它已经被撤销，因此你需要用你自己的 token 来更新它。

通过直接跳到`roles/slack/tasks/main.yml`，你可以看到剧本启动了 Azure 中的资源组、虚拟网络和子网。

启动 Azure 资源的第一个任务没有变化：

```
- name: "Create the resource group"
  azure.azcollection.azure_rm_resourcegroup:
    name: "{{ resource_group_name }}"
    location: "{{ location }}"
    tags: "{{ common_tags }}"
  register: "resource_group_output"
```

此外，我们在前几章中使用的调试任务仍然存在；在调试任务后面，我们有一个任务（嗯，有点像）会将通知发送到 Slack：

```
- name: "Notify the team on Slack about the resource group status"
  include_tasks: slack_notify_generic.yml
```

如你所见，它触发了`slack_notify_generic.yml`文件中的另一个任务，我们将注册输出的内容传递为一组变量，其中大部分变量是显而易见的：

```
  vars:
    resource_changed: "{{ resource_group_output.changed }}"
    resource_type: "Resource Group"
    resource_name: "{{ resource_group_output.state.name }}"
    resource_location: "{{ resource_group_output.state.location }}"
```

最后两个有点不同；这个需要完整的资源 ID，并将其前缀加上`https://portal.azure.com/#resource`，因为资源 ID 是 Azure 中资源的 URL；这个和 URL 前缀一起，会生成一个可点击的链接，当用户点击时，直接带领他们到资源页面：

```
    azure_portal_link: "https://portal.azure.com/#resource{{ resource_group_output.state.id }}"
```

最终变量使用 Jinja2 模板函数生成一个以逗号分隔的标签和值的列表：

```
    resource_tags: >
      {% for key, value in resource_group_output.state.tags.items() %}
      *{{ key }}:* {{ value }}{% if not loop.last %}, {% endif %}
      {% endfor %}
```

你可能也注意到，`{{ key }}`变量两侧有一个`*`；这不是模板函数的一部分，而是 Markdown 语法中的**粗体**，它会将内容样式化为粗体。

在查看`roles/slack/tasks/`中的`slack_notify_generic.yml`之前，让我们快速讨论一下为什么我们选择了这种方法。

正如我们在标题中多次提到的，自动化部署的主要目标之一是尽可能地简化一切。在这种情况下，我们调用的任务将在整个剧本中作为标准，唯一需要改变的是内容。

所以我们并不需要在剧本中多次重复`community.general.slack`任务，我们可以只定义一次，然后多次调用。这意味着如果我们需要更改`community.general.slack`任务中的内容，我们只需要在一个地方更新它。

任务本身有一点逻辑附加在其中，现在我们来回顾一下：

```
- name: "Notify the team on Slack about resource changes"
  community.general.slack:
    token: "{{ slack.token }}"
    parse: "none"
```

如前面的代码所示，我们传递了 webhook 的 `token`，并将 `parse` 选项设置为 `none`。这意味着 `community.general.slack` 不会处理我们发布到 webhook 的任何内容，不会去除格式等。

我们不是发送一个简单的消息，而是使用 `attachments` 类型。这将把我们的消息格式化为块状，并且我们可以根据内容是否发生变化来设置状态颜色：

```
    attachments:
      - fallback: "Notification about Azure resource changes"
```

设置颜色的逻辑如下：在这里，我们使用 `resource_changed` 变量传递的布尔值 `true` 或 `false`。如果变量等于 `true`，意味着资源发生了变化，因此我们将颜色设置为预定义的 `warning` 颜色，即橙色；否则，颜色设置为 `good`，即绿色：

```
        color: "{% if resource_changed %}warning{% else %}good{% endif %}"
        title: "Ansible: {{ resource_type }}"
```

接下来，我们有消息内容：在这里，我们使用与设置颜色类似的逻辑，根据资源是否发生变化来决定颜色：

```
        text: "{{ resource_name }} has been {% if resource_changed %}created/updated{% else %}checked (no changes){% endif %}."
```

最后，我们有字段；每个字段都以块的形式显示我们传递给任务的信息，除了一个：

```
        fields:
          - title: "Location"
            value: "{{ resource_location }}"
            short: true
          - title: "Azure Portal"
            value: "<{{ azure_portal_link }}|View in Azure Portal>"
            short: true
          - title: "Tags"
            value: "{{ resource_tags }}"
            short: false
```

Azure 门户链接的值稍有不同；Slack 使用 **mrkdwn**，一种类似于 Markdown 的标记语言，但在某些方面有所不同，特别是在格式化链接时。正如你所看到的，我们将其设置为以下内容：

```
<{{ azure_portal_link }}|View in Azure Portal>
```

这是创建可点击链接的 mrkdwn 语法。它将链接到 `{{ azure_portal_link }}` 变量中传递的 URL。管道符号 `|` 后的文本是会出现在 Slack 消息中的可见文本，并充当可点击链接。

当 Slack 渲染此消息时，它会显示 `{{ azure_portal_link }}` 变量，指向 Azure 门户。

现在我们已经了解了 playbook 的样子，让我们运行它：

```
$ ansible-playbook -i hosts site.yml --ask-vault-pass
```

这将提示你提供一个有效的密码，然后部署资源；在这种情况下，我们不需要了解运行 playbook 的输出，应该将注意力转向 Slack 本身：

![图 17.3 – Playbook 的第一次运行](img/B21620_11_03.jpg)

图 17.3 – Playbook 的第一次运行

从前面的输出中可以看出，已经添加了三个资源，因此它们被称为已创建/更新。橙色条形图显示在消息的左侧。

现在，让我们使用以下命令重新运行 playbook：

```
$ ansible-playbook -i hosts site.yml --ask-vault-pass
```

你将看到消息现在是这样的：

![图 17.4 – 第二次运行 playbook](img/B21620_11_04.jpg)

图 17.4 – 第二次运行 playbook

这次没有发生任何变化，消息也反映了这一点。状态显示为绿色，因此我们可以快速看到没有变化。

我唯一想补充的是，如果你查看仓库中的代码，你会注意到对于子网，我们需要做一些适配：

+   `resource_location`：子网没有位置，因此我们使用其所在虚拟网络的位置。

+   `azure_portal_link`：虽然返回的是子网的 ID，但它并没有完全匹配我们用来直接在 Azure 门户中打开资源的逻辑，因此我们链接到配置了子网的虚拟网络。

+   `resource_tags`：你不能为子网添加标签，因此我们将值设置为`N/A`。

正如你从屏幕上看到的，这对于通知他人你的剧本正在运行非常有用。它还可以让你快速访问正在创建/更新或检查的资源，并提供资源变更的审计记录。

虽然我们讨论的代码仅适用于 Slack 和在 Microsoft Azure 中部署的资源，但这个概念应该适用于 Ansible 支持的任何集成。

## 其他集成

Ansible Galaxy 上有数十种其他集成，既有社区支持的，也有供应商支持的。如果你找不到适合你用例的集成，而且你的目标服务有 API，你可以非常快速地使用 `ansible.builtin.uri` 模块构建集成，该模块旨在与 Web API 和服务进行交互。

以下是其他集成模块的一些示例用例。

### 说

大多数现代计算机都内置了一定程度的语音合成功能；通过使用此模块，你可以让 Ansible 通过语音告知你剧本运行的状态：

```
    - name: "Speak an update"
      community.general.say:
        msg: "Hello from Ansible running on {{ inventory_hostname }}"
        voice: "Zarvox"
      delegate_to: localhost
```

虽然这很有趣，但并不太实用，可能很快就会变得烦人，所以让我们继续。

### Syslog

假设你从目标主机发送日志文件。在这种情况下，你可能希望将剧本运行的结果发送到目标主机的 syslog 中，这样它就会被传送到你的中央日志服务中，用于像**SIEM**（即**安全信息和事件管理**）这样的外部服务：

```
- name: "Send a message to the hosts syslog"
  community.general.syslogger:
    msg: "The task has completed and all is well"
    priority: "info"
    facility: "daemon"
    log_pid: "true"
```

这是一种很好的方式，用来记录目标主机上发生的事件，并将其与目标操作系统上发生的其他所有事件一起记录。

### ServiceNow

ServiceNow 是 ServiceNow 公司提供的企业级 IT 服务管理软件。

通过使用`servicenow.servicenow.snow_record`模块，你的剧本可以在 ServiceNow 安装中打开事件：

```
- name: "Create an incident in SNOW"
  servicenow.servicenow.snow_record:
    username: "{{ snow.username}}"
    password: "{{ snow.passord}}"
    instance: "{{ snow.instance }}"
    state: "present"
    data:
      short_description: "Ansible playbook run on {{ inventory_hostname }}"
      severity: 3
      priority: 2
  register: snow_incident_result
```

打开后，你可以像下面这样为它们添加注释：

```
- name: "Update the SNOW incident with work notes"
  servicenow.servicenow.snow_record:
    username: "{{ snow.username}}"
    password: "{{ snow.passord}}"
    instance: "{{ snow.instance }}"
    state: present
    number: "{{snow_incident_result['record']['number']}}"
    data:
      work_notes : "{{ resource_name }} has been {% if resource_changed %}created/updated{% else %}checked (no changes){% endif %}."
```

在剧本运行结束时，你可以关闭事件，这将永久记录你从剧本中传输到 ITSM 工具的所有信息。

### Microsoft Teams

尽管我们在本章中以 Slack 为主要示例，Ansible 还支持多个 Microsoft 365 产品，包括通过`community.general.office_365_connector_card`模块的 Microsoft Teams。Microsoft 365 Connector 卡片非常强大，它们的配置，以及因此而来的 Ansible 模块，可能会变得相当复杂；因此，建议你从以下链接开始了解：

+   [`docs.ansible.com/ansible/latest/collections/community/general/office_365_connector_card_module.html`](https://docs.ansible.com/ansible/latest/collections/community/general/office_365_connector_card_module.html)

+   [`learn.microsoft.com/en-us/microsoftteams/platform/task-modules-and-cards/what-are-cards`](https://learn.microsoft.com/en-us/microsoftteams/platform/task-modules-and-cards/what-are-cards)

+   [`adaptivecards.io/`](https://adaptivecards.io/)

从前面的链接中可以看出，连接器卡片可以根据需要简单或复杂。然而，配置它们可能值得专门成章，所以我们继续往下看。

## 第三方服务总结

我希望你从本书中得到的一个重要收获是，自动化非常棒；它不仅节省了大量时间，而且使用我们在上一章中介绍的工具，*第十六章*，*引入 Ansible AWX 和 Red Hat Ansible 自动化平台*，使得那些不是系统管理员或开发人员的人也可以通过友好的 web 界面执行他们的 playbook。在本章的最后部分，我们将进一步探讨这一点，我会介绍一些 Ansible 在我曾合作的组织中实施的实际案例。

本节中我们介绍的模块不仅可以让你记录结果，还能在执行 playbook 时自动进行一些清理工作，并通知你的用户，从而将自动化提升到一个新的水平。

例如，你需要向服务器部署一个新配置。你的服务台已为你发出了一个更改请求，要求你在 ServiceNow 安装中处理这项工作。

你的 playbook 可以在执行更改之前，使用 fetch 模块将配置文件复制到你的 Ansible Controller。然后，playbook 可以使用 `servicenow.servicenow.snow_record` 模块，将现有配置文件的副本附加到更改请求中，进行更改操作，最后自动更新更改请求的结果。

在我们看一些实际案例之前，让我们来看看你如何在 playbook 执行过程中进行调试。

# Ansible playbook 调试器

Ansible 内置了调试器。让我们通过创建一个带有错误的简单 playbook，来看看如何将其集成到你的 playbook 中。正如我们刚才提到的，我们将编写一个使用 `community.general.say` 模块的 playbook。该 playbook 本身如下所示：

```
- name: "A simple playbook with a mistake"
  hosts: "localhost"
  debugger: "on_failed"
  vars:
    message: "The task has completed and all is well"
    voice: "Daniel"
  tasks:
    - name: "Say a message on your Ansible host"
      community.general.say:
        msg: "{{ massage }}"
        voice: "{{ voice }}"
```

有两件事需要指出：第一是错误。正如你所看到的，我们定义了一个名为 `message` 的变量，但当我在任务中使用它时，我打错了一个字，输入了 `massage`。幸运的是，在开发这个 playbook 时，我指示 Ansible 在任务失败时使用交互式调试器，通过将 `debugger` 选项设置为 `on_failed`。

## 调试任务

让我们运行该 playbook，看看会发生什么：

```
$ ansible-playbook playbook.yml
```

第一个问题是我们没有传递主机清单文件，因此会有警告，显示只有 localhost 可用；这没问题，因为我们只希望在我们的 Ansible 控制器上运行 `Say` 模块：

```
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'
```

接下来，Ansible 执行 playbook 本身；这应该会导致一个致命错误：

```
PLAY [A simple playbook with a mistake] *******************
TASK [Gathering Facts] ************************************
ok: [localhost]
TASK [Say a message on your Ansible host] *****************
fatal: [localhost]: FAILED! => {"msg": "The task includes an option with an undefined variable. The error was: 'massage' is undefined. 'massage' is undefined\n\nThe error appears to be in '/Users/russ.mckendrick/Code/Learn-Ansible-Second-Edition/Chapter17/debugger/playbook.yml': line 12, column 7, but may\nbe elsewhere in the file depending on the exact syntax problem.\n\nThe offending line appears to be:\n\n  tasks:\n    - name: \"Say a message on your Ansible host\"\n      ^ here\n"}
```

通常，playbook 执行会停止，你将返回到你的 shell；然而，因为我们指示 Ansible 进入交互式调试器，所以现在我们看到以下提示：

```
[localhost] TASK: Say a message on your Ansible host (debug)>
```

从这里，我们可以开始进一步调查问题；例如，我们可以通过输入以下命令来查看错误：

```
p result._result
```

在 Ansible 中，使用 debug 模块时，`p` 命令用于美化变量或表达式的输出。它代表 `p result._result`，在 Ansible 调试任务中，它将以更易读和格式化的方式显示 `result._result` 的值。`p` 命令使用 Python 标准库中的 `pprint`（**漂亮打印**）函数来格式化输出。

一旦你按下 *Enter* 键，失败任务的结果将被返回：

```
{'_ansible_no_log': False,
 'failed': True,
 'msg': 'The task includes an option with an undefined variable. The error was: \'massage\' is undefined. \'massage\' is undefined\n\nThe error appears to be in \'/Users/russ.mckendrick/Code/Learn-Ansible-Second-Edition/Chapter17/debugger/playbook.yml\': line 12, column 7, but may\nbe elsewhere in the file depending on the exact syntax problem.\n\nThe offending line appears to be:\n\n  tasks:\n    - name: "Say a message on your Ansible host"\n      ^ here\n'}
```

让我们通过输入以下命令来更仔细地查看任务中使用的变量：

```
p task.args
```

这将返回我们在任务中使用的两个参数：

```
{'msg': '{{ massage }}', 'voice': '{{ voice }}'}
```

现在，让我们通过以下命令查看任务可用的变量：

```
p task_vars
```

你可能已经注意到，我们指示 Ansible 在执行 playbook 时执行 setup 模块，因此任务可用的变量列表非常长：

```
          'inventory_hostname': 'localhost',
          'inventory_hostname_short': 'localhost',
          'message': 'The task has completed and all is well',
          'module_setup': True,
          'omit': '__omit_place_holder__7da4853be448a08d857e98fbabe7afe1b7c97d00',
          'play_hosts': ['localhost'],
          'playbook_dir': '/Users/russ.mckendrick/Code/Learn-Ansible-Second-Edition/Chapter17/debugger',
          'voice': 'Daniel'},
```

如你所见，关于我们的 playbook 执行环境有很多信息。在变量列表中，你会注意到所有由 setup 模块收集的信息都以 `ansible_` 开头，我们的两个变量被列在底部。

我们可以通过运行以下命令来进一步了解这两个变量：

```
p task_vars['message']
p task_vars['voice']
```

这将显示变量的内容：

```
[localhost] TASK: Say a message on your Ansible host (debug)> p task_vars['message']
'The task has completed and all is well'
[localhost] TASK: Say a message on your Ansible host (debug)> p task_vars['voice']
'Daniel'
```

我们知道我们将一个拼写错误的变量传递给了 `msg` 参数，因此我们将进行一些修改并继续执行 playbook。为此，我们将运行以下命令：

```
task.args['msg'] = '{{ message }}'
```

这将更新参数，使用正确的变量；我们现在可以通过发出以下命令重新运行任务：

```
redo
```

这将立即使用正确的参数重新运行任务，如果一切顺利，你应该听到：“*任务已完成，一切* *顺利*：”

```
changed: [localhost]
PLAY RECAP ************************************************
localhost: ok=1  changed=1  unreachable=0  failed=0\. skipped=0  rescued=0\. ignored=0
```

如前面的输出所示，因为我们只有一个任务，所以 playbook 执行完毕。如果有更多任务，它会从上次中断的地方继续。现在你可以更新你的 playbook，修正拼写错误，并继续你接下来的工作。此外，如果我们愿意，也可以输入 `continue` 或 `quit` 来分别继续或停止。

## Ansible 调试器摘要

当你在编写复杂的 playbook 时，Ansible 调试器是一个非常有用的选项；例如，假设你有一个需要大约 20 分钟运行的 playbook，但它在接近结束时抛出错误，比如在运行 playbook 18 分钟后出现问题。

让 Ansible 进入交互式调试器 shell 不仅意味着你可以精确查看已定义和未定义的内容，而且也意味着你不需要盲目地修改 playbook 然后再等 18 分钟来查看这些修改是否解决了致命错误。

# 一些现实世界的例子

在我们结束本章和本书之前，我将举几个例子，说明过去几年我如何使用和与 Ansible 互动。

## 自动化一个复杂的部署

在这个例子中，一个应用程序被分布在数十台公共云服务器上。每个应用程序组件都至少安装在三台不同的主机上，并且需要按特定顺序进行更新。

应用程序开发人员与运维团队合作，优化部署流程并创建了一个 Ansible Playbook。这个 playbook 自动化了应用程序每个组件的以下步骤：

1.  通过连接到目标主机并执行特定命令，将应用程序置于维护模式。

1.  创建涉及所有部署成本的快照，以确保在需要时可以进行回滚。

1.  通过从指定的 GitHub 仓库拉取最新代码并执行一系列命令来更新应用程序，从而启动部署过程。

1.  通过连接到应用程序的 API 并在每个目标主机上运行一组健康检查来验证部署是否成功。

1.  如果部署和健康检查成功通过，将应用程序从维护模式中取出并继续进行下一个组件的部署。然而，如果任何测试失败，立即停止部署并执行命令将主机恢复到之前的快照，确保安全回滚。

在实施 Ansible 自动化之前，手动执行这些部署步骤需要几个小时，因为应用程序和运维团队必须协调并仔细遵循流程。这个手动方法使得部署充满挑战，且容易出现人为错误。

通过使用 Ansible 自动化部署任务，团队可以专注于处理由于真正的问题而产生的异常，而不是手动执行过程中导致的错误。在自动化投入使用之前，几乎每次发布过程中都会出现错误，涉及许多主机和复杂的手动步骤。

Ansible 自动化的引入显著改善了部署过程，减少了所需时间并最小化了人为错误的风险。该 playbook 确保了多个部署过程中的一致性、可靠性和可重复性，使得团队能够更频繁且更有信心地部署应用程序组件。

这个例子展示了 Ansible 如何处理复杂的部署场景，简化流程，并增强开发和运维团队之间在公共云环境中的协作。

## 结合 Ansible 和其他工具

在这个实际场景中，我们与一个团队合作，该团队在使用 Terraform 开发其基础设施自动化方面投入了大量精力。他们的 Terraform 代码成功地部署了基础设施，并使用简单的`cloud-init`脚本执行了基本的主机引导。

然而，随着应用程序需求的日益复杂，显然需要额外的自动化来有效地管理已配置的主机上的应用程序。我们并没有替换现有的 Terraform 代码，而是引入了 Ansible 来补充基础设施自动化。

为了将 Ansible 与现有的 Terraform 工作流集成，我们使用了`community.general.terraform`模块。该模块使我们能够直接在 Ansible 剧本中执行 Terraform 部署。

通过利用这种集成，我们将 Terraform 部署生成的输出传递给 Ansible。这使得 Ansible 能够收集已配置主机的详细信息，并执行必要的应用程序引导任务。

Terraform 和 Ansible 的结合证明了这是一个强大的解决方案：

+   Terraform 处理了基础设施的配置，确保所需资源在目标环境中正确创建和配置。

+   Ansible 接管了应用程序管理，利用 Terraform 提供的主机信息无缝配置和部署应用程序组件。

这种方法使团队能够在保持现有 Terraform 代码库的基础上，利用 Ansible 扩展自动化功能。两种工具之间的集成提供了无缝的工作流，使团队能够更有效地管理基础设施和应用程序，而无需舍弃已有的代码。

通过为特定任务选择合适的工具并发挥它们的优势，团队实现了一个更全面、高效的自动化解决方案。Terraform 的基础设施即代码能力与 Ansible 的应用程序管理和编排功能相结合，最终形成了一个强大且灵活的自动化流水线。

### 部署 Ansible AWX

正如在*第十六章*中讨论的那样，*介绍 Ansible AWX 和 Red Hat Ansible 自动化平台*，Ansible AWX 是一个强大的工具，提供了超越基础功能的广泛特性。除了核心功能外，Ansible AWX 还提供了调查、与身份服务（如 Microsoft Entra）的集成以及 **基于角色的访问控制**（**RBACs**）等功能，这些功能能够为项目和模板提供细粒度的访问管理。

Ansible AWX 中的调查问卷允许你创建交互式表单，在运行 playbook 之前收集用户的输入。当你需要从最终用户那里收集特定信息或参数，而又不暴露底层 playbook 复杂性时，这一功能尤其有用。

与身份服务（如 Microsoft Entra）的集成，使 Ansible AWX 用户能够实现无缝的身份验证和授权。此集成让你可以利用现有的用户账户和访问控制，简化用户管理并确保对 Ansible AWX 资源的安全访问。

Ansible AWX 中的 RBAC 提供了一种灵活且细粒度的方式来管理用户权限。通过 RBAC，你可以定义角色并将其与特定的项目、模板和其他资源关联。这样，你可以控制谁可以访问和执行特定的 playbook，确保用户根据其职责和专业知识具有适当的访问权限。

在以下示例中，我们将探讨 Ansible AWX 如何在我曾合作过的各个组织中应用，以简化流程、自动化任务并帮助团队高效地履行职责，同时保持安全和治理。

### 配置虚拟机

在这种场景下，IT 团队需要为开发人员提供一个自助服务门户，便于他们在不同环境（如开发、暂存和生产环境）中配置**虚拟机**（**VMs**）。每个环境有不同的要求和配置。

为简化流程，部署了 Ansible AWX，并创建了一个调查问卷来捕获开发人员的必要信息。问卷包括用于指定操作系统、虚拟机大小、环境及其他相关参数的字段。

提交调查问卷后，Ansible AWX 触发了一个 playbook，自动化了资源配置过程。根据调查问卷的回答，playbook 动态生成了适当的虚拟机配置并在指定的环境中配置了虚拟机。

此外，playbook 与组织的工单系统进行了集成，自动创建了一个包含虚拟机详情的工单，并将其与变更管理流程相关联，用于追踪和审计目的。

通过利用 Ansible AWX 和调查问卷，IT 团队赋能开发人员按需配置虚拟机，同时保持对整个过程的控制和治理。

### 管理应用部署

在另一个用例中，一个软件开发团队需要将其应用程序部署到多个环境，包括开发、QA 和生产环境。每个环境都有自己的配置和依赖关系。

为简化部署过程，使用了 Ansible AWX。创建了一个调查问卷来捕获必要的部署参数，例如应用版本、目标环境和任何特定的配置选项。

随后，调查问卷的响应被作为变量传递给负责执行部署的 Ansible playbook。该 playbook 处理整个部署过程，包括以下内容：

+   从工件库中检索指定的应用程序版本

+   根据提供的参数配置目标环境

+   部署应用程序组件及其依赖项

+   运行部署后的测试和健康检查

+   更新组织项目管理工具中的部署状态

通过使用 Ansible AWX 和调查问卷，开发团队可以通过一个用户友好的界面发起部署，确保一致性并减少手动错误的风险。Playbook 自动化了复杂的部署步骤，为需要部署的团队节省了时间和精力，同时释放了本应执行部署的团队的时间。

### 更新 DNS 记录

在这个例子中，组织管理着多个 **DNS**（或其全称，**域名系统**）区域，跨多个提供商，他们需要允许前线支持团队更新 DNS 记录，而无需直接授予他们访问提供商管理控制台的权限。

为了实现这一目标，使用了 Ansible AWX。创建了一个调查问卷，用于捕获更新 DNS 记录所需的信息。调查问卷包括指定域名、记录类型（如 A、CNAME、MX）、记录值和**生存时间**（**TTL**）的字段。

提交调查问卷后，Ansible AWX 会触发一个 playbook 来自动化 DNS 记录更新过程。该 playbook 执行以下步骤：

1.  验证提供的调查输入，确保数据完整性并防止无效条目的输入

1.  根据调查问卷中指定的域名确定了适当的 DNS 提供商

1.  使用 Ansible Vault 安全存储的必要凭证连接到 DNS 提供商的 API

1.  检索指定域名和记录类型的现有 DNS 记录

1.  使用调查问卷中提供的新值和 TTL 更新 DNS 记录

1.  使用提供商的 API 保存更新后的 DNS 记录

1.  在组织的变更管理系统中记录该变更，如 ServiceNow，供跟踪和审计使用

通过使用 Ansible AWX，前线支持团队可以轻松更新 DNS 记录，无需直接访问 DNS 提供商的管理控制台。Playbook 自动化了更新多个提供商的 DNS 记录过程中复杂的步骤，确保了一致性并减少了错误的风险。

此外，与变更管理系统的集成提供了所有 DNS 变更的集中记录，便于跟踪、审计，并符合组织的变更控制流程。

这些示例展示了如何利用 Ansible AWX 来运行任务，并简化不同领域（如基础设施配置和应用部署）中最终用户的流程。通过将 Ansible AWX 与调查结合，并与现有工具和流程集成，组织可以启用自助服务功能，同时保持对关键操作的控制和治理。

# 总结

我们已经到达了本章的结尾，也是我们书本的结尾。我一直在思考如何总结 Ansible；我认为《学习 Ansible》第一版的总结依然适用。

在回应一位技术招聘人员时，该招聘人员联系了他并要求至少三年的 Ansible 使用经验，而 Ansible 这款工具实际上只发布了短短一段时间，Ansible 的创始人 Michael DeHaan 在一条现在已经删除的推文中说了以下话：

*“使用 Ansible 几个月的人和使用 Ansible 三年的人一样好。它是故意设计成一个简单的工具。”*

这完美地总结了我对 Ansible 的经验，希望也能总结出你的经验。

一旦掌握了基础知识，继续学习并快速构建更复杂的 playbook 是非常简单的。这些 playbook 可以帮助部署基础代码和应用程序，也可以帮助构建复杂的云架构甚至物理架构。

通过重用你的角色并通过 Ansible Galaxy 访问大量社区贡献的角色和模块，你将拥有许多示例或快速启动点来进行下一个项目。因此，你可以比使用其他工具时更早地卷起袖子开始动手。此外，如果 Ansible 无法完成某项任务，通常可以找到一个可以与其集成的工具来提供缺失的功能。

回到我们在*第一章*中讨论的内容，*安装和运行 Ansible*，能够以可重复和可共享的方式定义你的基础设施和部署，并鼓励他人参与贡献你的 playbook，应当是将 Ansible 引入日常工作流的目标。

通过本书，我希望你已经开始思考日常任务中 Ansible 能帮助你节省时间的地方，并祝你在开发自己的 playbook 时好运。
