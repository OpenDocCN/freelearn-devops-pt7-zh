# 第六章 从 Ansible 获取通知

与 bash 脚本相比，Ansible 的一个显著优势是能够在同一系统上多次运行，确保一切井然有序。这是一个非常好的功能，不仅可以确保没有改变服务器上的配置，还可以在短时间内应用新配置。

由于这些原因，许多人每天运行他们的`master.yaml`。当您这样做时（也许您应该！），您希望 Ansible 本身发送某种反馈给您。还有许多其他情况，您可能希望 Ansible 向您或您的团队发送消息。例如，如果您使用 Ansible 部署应用程序，您可能希望向开发团队频道发送 IRC 消息（或其他类型的群聊消息），以便他们了解系统状态。

其他时候，您希望 Ansible 通知 Nagios 即将破坏某些内容，以便 Nagios 不会担心并开始向系统管理员发送电子邮件和消息。

在本章中，我们将探讨以下主题：

+   邮件通知

+   Ansible XMPP/Jabber

+   Slack 和 Rocket Chat

+   向 IRC 频道发送消息（社区信息和贡献）

+   Amazon 简单通知服务

+   Nagios

# 电子邮件

提醒人们的最简单和最常见方法是发送电子邮件。Ansible 允许您在播放本之间使用`mail`模块发送电子邮件。您可以在任何任务之间使用此模块，并在需要时通知用户。此外，在某些情况下，您无法自动化每一件事，因为要么您缺乏权限，要么需要一些手动检查和确认。如果是这种情况，您可以通知负责人员 Ansible 已经完成了工作，现在是他/她执行其职责的时候了。让我们看看如何使用名为`uptime_and_email.yaml`的非常简单的播放本来使用`mail`模块通知您的用户：

```
    - hosts: localhost 
      tasks: 
      - name: Read the machine uptime 
        command: uptime -p 
        register: uptime 
      - name: Send the uptime via e-mail 
        mail: 
          host: mail.fale.io 
          username: ansible@fale.io 
          password: PASSWORD 
          to: me@fale.io 
          subject: Ansible-report 
          body: 'Local system uptime is {{ uptime.stdout }}.' 

```

在前面的手册中，我们将首先读取当前机器的运行时间，然后通过电子邮件发送给某人。这个示例非常简单，将使我们的示例保持简短，但显然你也可以在非常长和复杂的手册中以类似的方式生成电子邮件。如果我们稍微关注一下`mail`任务，我们可以看到我们正在以下数据一起使用它：

+   要用来发送电子邮件的电子邮件服务器（还包括登录信息，这是该服务器所需的）

+   接收者电子邮件地址

+   电子邮件主题

+   电子邮件正文

`mail`模块支持的其他有趣参数包括：

+   `attach`参数：用于向将生成的电子邮件添加附件。当您希望通过电子邮件发送日志时，这非常有用。

+   `port`参数：用于指定电子邮件服务器使用的端口。

关于此模块的一个有趣之处是，唯一强制的字段是`subject`，而不是像许多人期望的那样是主体。

现在我们可以继续执行脚本以验证其功能，方法如下：

```
ansible-playbook -i localhost, uptime_and_email.yaml

```

我们将得到类似于以下的结果：

```
PLAY [localhost] *************************************************
TASK [setup] *****************************************************
ok: [localhost] 
TASK [Read the machine uptime] ***********************************
changed: [localhost] 
TASK [Send the uptime via e-mail] ********************************
changed: [localhost] 
PLAY RECAP *******************************************************
localhost         : ok=3    changed=2    unreachable=0    failed=0

```

同样，正如预期的那样，Ansible 已经向我发送了一封包含以下内容的电子邮件：

```
    Local system uptime is up 38 min. 

```

这个模块可以用多种方式。一个我见过的实际案例是，一个剧本被创建来自动化一个非常长的程序中的某个部分，程序历史上是通过电子邮件交换所有者的，每个参与程序的人在收到前一部分所有者的电子邮件后，会完成自己的部分，并在结束时发送电子邮件给下一个所有者。当我们开始自动化这个程序时，我们只自动化了其中的一部分，而没有人注意到这一部分已经被自动化。这不是处理程序的最佳方式，但它在组织中广泛使用，而且你通常无法改变它。

# XMPP

电子邮件速度慢、不可靠，而且人们往往不会立即对其作出反应。有时你希望实时向某个用户发送消息。许多组织依赖 XMPP/Jabber 作为内部聊天系统，而最棒的是，Ansible 能够直接向 XMPP/Jabber 用户和会议室发送消息。

让我们调整一下之前的示例，将运行时间信息发送给文件`uptime_and_xmpp_user.yaml`中的用户：

```
    - hosts: localhost 
      tasks: 
      - name: Read the machine uptime 
        command: 'uptime -p' 
        register: uptime 
      - name: Send the uptime to user 
        jabber: 
          user: ansible@fale.io 
          password: PASSWORD 
          to: me@fale.io 
          msg: 'Local system uptime is {{ uptime.stdout }}.' 

```

### 注意

如果你想使用 Ansible 的`jabber`任务，你需要在执行任务的系统上安装`xmpppy`库。

正如你所看到的，`jabber`模块与`mail`模块非常相似，并且需要类似的参数。在 XMPP 的情况下，我们不需要指定服务器主机和端口，因为这些信息会通过 DNS 由 XMPP 自动获取。在需要使用不同服务器主机或端口的情况下，我们可以分别使用`host`和`port`参数。

现在我们可以继续执行脚本以验证其功能，方法如下：

```
ansible-playbook -i localhost, uptime_and_xmpp_user.yaml

```

我们将得到类似于以下的结果：

```
PLAY [localhost] *************************************************
TASK [setup] *****************************************************
ok: [localhost] 
TASK [Read the machine uptime] ***********************************
changed: [localhost] 
TASK [Send the uptime to user] ***********************************
changed: [localhost] 
PLAY RECAP *******************************************************
localhost         : ok=3    changed=2    unreachable=0    failed=0

```

在需要向会议室而非单个用户发送消息的情况下，只需更改`to`参数，添加适当的目标即可。

```
    to=sysop@conference.fale.io (mailto:sysop@conference.fale.io)/ansiblebot 

```

# Slack

在过去几年里，许多新的聊天和协作平台出现了。其中最常用的平台之一是 Slack。Slack 是一款基于云的团队协作工具，这使得它与 Ansible 的集成变得更加便捷。

让我们把以下几行放入文件`uptime_and_slack.yaml`中：

```
    - hosts: localhost 
      tasks: 
      - name: Read the machine uptime 
        command: 'uptime -p' 
        register: uptime 
      - name: Send the uptime to slack channel 
        slack: 
          token: TOKEN 
          channel: '#ansible' 
          msg: 'Local system uptime is {{ uptime.stdout }}.' 

```

如我们所讨论的，这个模块的语法比 XMPP 模块更简洁，事实上，它只需要知道令牌（你可以在 Slack 网站上生成）、要发送消息的频道以及消息内容本身。

### 注意

自 Ansible 1.8 版本以来，Slack 令牌的新版已经被要求，例如：`G522SJP14/D563DW213/7Qws484asdWD4w12Md3avf4FeD`。

使用以下命令运行剧本：

```
ansible-playbook -i localhost, uptime_and_slack.yaml

```

这将产生如下输出：

```
PLAY [localhost] *************************************************
TASK [setup] *****************************************************
ok: [localhost] 
TASK [Read the machine uptime] ***********************************
changed: [localhost] 
TASK [Send the uptime to slack channel] **************************
changed: [localhost] 
PLAY RECAP *******************************************************
localhost         : ok=3    changed=2    unreachable=0    failed=0

```

由于 Slack 的目标是提高沟通效率，它允许我们调整消息的多个方面。从我的角度来看，最有趣的几点如下：

+   `color`：这允许你指定一个颜色条，在消息开头显示，以识别以下状态：

    +   **好**：绿色条

    +   **正常**：无条

    +   **警告**：黄色条

    +   **危险**：红色条

+   `icon_url`：这允许你更改该消息的用户头像

# 火箭聊天

许多公司喜欢 Slack 的功能，但在权衡 Slack 功能和本地部署服务所带来的隐私保护时遇到问题。Rocket Chat 是开源软件，实施了 Slack 的大多数功能以及它的大部分界面。作为开源软件，每个公司都可以在本地安装它，并以符合其 IT 规则的方式进行管理。

由于 Rocket Chat 的目标是作为 Slack 的替代品，从我们的角度来看，几乎不需要做任何更改，实际上，我们可以创建一个名为`uptime_and_rocket.yaml`的文件，并包含以下内容：

```
    - hosts: localhost 
      tasks: 
      - name: Read the machine uptime 
        command: 'uptime -p' 
        register: uptime 
      - name: Send the uptime to rocketchat channel 
        rocketchat: 
          token: TOKEN 
          domain: chat.example.com 
          channel: '#ansible' 
          msg: 'Local system uptime is {{ uptime.stdout }}.' 

```

如你所见，唯一发生变化的行是第 6 行和第 7 行，其中`slack`被替换成了`rocketchat`。另外，我们需要添加域名字段，指定我们安装的 Rocket Chat 所在位置。

运行以下代码：

```
ansible-playbook -i localhost, uptime_and_rocketchat.yaml

```

这将产生以下输出：

```
PLAY [localhost] *************************************************
TASK [setup] *****************************************************
ok: [localhost] 
TASK [Read the machine uptime] ***********************************
changed: [localhost] 
TASK [Send the uptime to rocketchat channel] *********************
changed: [localhost] 
PLAY RECAP *******************************************************
localhost         : ok=3    changed=2    unreachable=0    failed=0

```

# 互联网中继聊天（IRC）

IRC 可能是 1990 年代最著名且广泛使用的聊天协议，今天仍在使用，主要得益于它在开源社区中的应用以及其简洁性。从 Ansible 的角度来看，IRC 是一个相当直接的模块，我们可以按照以下示例使用它（将其放入`uptime_and_irc.yaml`文件中）：

```
    - hosts: localhost 
      tasks: 
      - name: Read the machine uptime 
        command: 'uptime -p' 
        register: uptime 
      - name: Send the uptime to IRC channel 
        irc: 
          port: 6669 
          server: irc.example.net 
          channel: #desired_channel 
          msg: 'Local system uptime is {{ uptime.stdout }}.' 
          color: green 

```

### 注意

你需要安装`socket` Python 库才能使用 Ansible 的 IRC 模块。

在 IRC 模块中，以下字段是必需的：

+   `channel`：这是指定你的消息将发送到的频道

+   `msg`：这是你要发送的消息

你通常需要指定的其他配置包括：

+   `server`：选择连接的`server`，如果不是`localhost`

+   `port`：选择连接的`port`，如果不是`6667`

+   `color`：如果不是`black`，这用于指定消息的`color`

+   `nick`：这是指定发送消息的`nick`，如果不是`ansible`

+   `use_ssl`：使用 SSL 和 TLS 安全

+   `style`：如果你希望以粗体、斜体、下划线或反向样式发送消息

运行以下代码：

```
ansible-playbook uptime_and_irc.yaml

```

这将产生以下输出：

```
PLAY [localhost] *************************************************
TASK [setup] *****************************************************
ok: [localhost] 
TASK [Read the machine uptime] ***********************************
changed: [localhost] 
TASK [Send the uptime to IRC channel] ****************************
changed: [localhost] 
PLAY RECAP *******************************************************
localhost         : ok=3    changed=2    unreachable=0    failed=0

```

# 亚马逊简单通知服务

有时，你希望你的剧本在接收警报的方式上是无关的。这有几个优点，主要体现在灵活性方面。实际上，在这种模式下，Ansible 将消息传递给通知服务，然后由通知服务负责交付。**亚马逊简单通知服务**（**SNS**）并不是唯一可用的通知服务，但它可能是使用最多的。SNS 具有以下组件：

+   **消息**：由发布者生成的消息，具有唯一的 UUID 标识

+   **发布者**：生成消息的程序

+   **主题**：消息的命名组，可以类似于聊天室或讨论组来理解

+   **订阅者**：将接收他们订阅的主题中发布的所有消息的客户端

所以在我们的例子中，具体来说，我们将有：

+   **消息**：Ansible 通知

+   **发布者**：Ansible 本身

+   **主题**：可能是不同的主题，用于根据系统和/或通知类型（例如存储、网络、计算）对消息进行分组

+   **订阅者**：需要接收通知的团队成员

正如我们所说，SNS 的一个主要优势是你可以解耦 Ansible 发送消息的方式（SNS API）和用户接收消息的方式。实际上，你可以为每个用户和每个主题规则选择不同的传递系统，并且最终可以动态更改这些系统，以确保消息以最佳方式发送，以适应任何情况。目前，SNS 发送消息的五种方式是：

+   亚马逊 **Lambda** 函数（使用 Python、Java 和 JavaScript 编写的无服务器函数）

+   亚马逊 **简单队列服务** (**SQS**)（一个消息队列系统）

+   电子邮件

+   HTTP(S) 调用

+   短信

让我们来看一下如何使用 Ansible 发送 SNS 消息。为此，我们可以创建一个名为 `uptime_and_sns.yaml` 的文件，内容如下：

```
    - hosts: localhost 
      tasks: 
      - name: Read the machine uptime 
        command: 'uptime -p' 
        register: uptime 
      - name: Send the uptime to SNS 
        sns: 
          msg: 'Local system uptime is {{ uptime.stdout }}.' 
          subject: "System uptime" 
          topic: "uptime" 

```

在这个例子中，我们使用 `msg` 键来设置将要发送的消息，使用 `topic` 来选择最合适的主题，使用 `subject` 作为电子邮件通知的主题。你可以设置许多其他选项。主要是，它们用于通过不同的传送方式发送不同的消息。例如，通过短信发送简短的消息是有意义的（毕竟，短信中的第一个 S 代表 **简短**），而更长且更详细的消息则通过电子邮件发送。为此，SNS 模块为我们提供了以下特定传送选项：

+   电子邮件

+   HTTP

+   HTTPS

+   短信

+   SQS

此模块还允许我们设置三个 AWS 特定的参数，我没有指定这些参数，因为我有一个配置文件来存储 AWS 凭证和选项：

+   `aws_access_key`：AWS 访问密钥，如果未指定，则会使用环境变量 `aws_access_key` 或 `~/.aws/credentials` 文件中的内容

+   `aws_secret_key`：AWS 秘密密钥，如果未指定，则会使用环境变量 `aws_secret_key` 或 `~/.aws/credentials` 文件中的内容

+   `region`：要使用的 AWS 区域，如果未指定，则会使用环境变量 `ec2_region` 或 `~/.aws/config` 文件中的内容

使用以下命令运行代码：

```
ansible-playbook uptime_and_sns.yaml

```

这将导致以下输出：

```
    PLAY [localhost] ************************************************* 

    TASK [setup] ***************************************************** 
    ok: [localhost] 

    TASK [Read the machine uptime] *********************************** 
    changed: [localhost] 

    TASK [Send the uptime to SNS] ************************************ 
    changed: [localhost] 

    PLAY RECAP ******************************************************* 
    localhost         : ok=3    changed=2    unreachable=0    failed=0

```

# Nagios

Nagios 是最常用的服务和服务器状态监控工具之一。Nagios 能定期审核服务器和服务的状态，并在发生问题时通知用户。如果你的环境中有 Nagios，你需要在管理机器时非常小心，因为当 Nagios 发现服务器或服务处于不健康状态时，它会开始向你的整个团队发送电子邮件、短信和电话。当你对被 Nagios 控制的节点运行 Ansible 脚本时，你必须更加小心，因为这可能会在夜间或其他不合适的时间触发电子邮件、短信和电话。为了避免这种情况，Ansible 可以提前通知 Nagios，这样即使某些服务因为重启等原因处于停机状态，或其他检查失败，Nagios 也不会在该时间窗口内发送通知。

在这个例子中，我们将停止一个服务，等待 5 分钟，然后重新启动它，因为这实际上会在大多数配置中触发一个 Nagios 故障。实际上，通常情况下，Nagios 被配置为接受最多两次连续的故障（通常每分钟执行一次），在触发关键状态之前将服务置于警告状态。我们将创建一个文件 `long_restart_service.yaml`，它将触发 Nagios 关键状态：

```
    - hosts: ws01.fale.io 
      tasks: 
      - name: Stop the HTTPd service 
        service: 
          name: httpd 
          state: stopped 
      - name: Wait for 5 minutes 
        pause: 
          minutes: 5 
      - name: Start the HTTPd service 
        service: 
          name: httpd 
          state: stopped 

```

使用以下命令运行代码：

```
ansible-playbook long_restart_service.yaml

```

这应该会触发一个 Nagios 警报，并产生以下输出：

```
    PLAY [ws01.fale.io] ********************************************** 

    TASK [setup] ***************************************************** 
    ok: [ws01.fale.io] 

    TASK [Stop the HTTpd service] ************************************ 
    changed: [ws01.fale.io] 

    TASK [Wait for 5 minutes] **************************************** 
    changed: [ws01.fale.io] 

    TASK [Start the HTTpd service] *********************************** 
    changed: [ws01.fale.io] 

    PLAY RECAP ******************************************************* 
    ws01.fale.io      : ok=4    changed=3    unreachable=0    failed=0

```

### 注意

如果没有触发 Nagios 警报，可能是你的 Nagios 安装没有跟踪该服务，或者 5 分钟不足以让它触发关键状态。

现在我们可以创建一个非常相似的 playbook，确保 Nagios 不会发送任何警报。我们将创建一个名为 `long_restart_service_no_alert.yaml` 的文件，内容如下：

```
    - hosts: ws01.fale.io 
      tasks: 
      - name: Silence Nagios 
    nagios: 
      action: disable_alerts 
      service: httpd 
      host: '{{ inventory_hostname }}' 
    delegate_to: nagios.fale.io 
  - name: Stop the HTTPd service 
    service: 
      name: httpd 
      state: stopped 
  - name: Wait for 5 minutes 
    pause: 
      minutes: 5 
  - name: Start the HTTPd service 
    service: 
      name: httpd 
      state: stopped 
  - name: Desilence Nagios 
    nagios: 
      action: enable_alerts 
      service: httpd 
      host: '{{ inventory_hostname }}' 
    delegate_to: nagios.fale.io 

```

如你所见，我们添加了两个任务。第一个任务是通知 Nagios 在指定主机上不发送 HTTPd 服务的警报，第二个任务是通知 Nagios 重新开始发送该服务的警报。即使你没有指定具体的服务，从而导致该主机上的所有警报都被静音，我的建议是仅禁用你将要中断的警报，以便 Nagios 仍能在大部分基础设施上正常工作。

### 注意

如果 playbook 执行失败并未达到重新启用警报的步骤，那么你的警报将保持 *禁用* 状态。

该模块的目标是切换 Nagios 警报并安排停机时间，从 Ansible 2.2 开始，该模块还可以取消安排的停机时间。

使用以下命令运行代码：

```
ansible-playbook long_restart_service_no_alert.yaml

```

这应该会触发一个 Nagios 警报，并产生以下输出：

```
PLAY [ws01.fale.io] **********************************************
TASK [setup] *****************************************************
ok: [ws01.fale.io] 
TASK [Silence Nagios] ********************************************
changed: [nagios.fale.io] 
TASK [Stop the HTTpd service] ************************************
changed: [ws01.fale.io] 
TASK [Wait for 5 minutes] ****************************************
changed: [ws01.fale.io] 
TASK [Start the HTTpd service] ***********************************
changed: [ws01.fale.io] 
TASK [Desilence Nagios] ******************************************
changed: [nagios.fale.io] 
PLAY RECAP *******************************************************
ws01.fale.io      : ok=4    changed=3    unreachable=0    failed=0
nagios.fale.io    : ok=2    changed=2    unreachable=0    failed=0

```

### 注意

要使用 Nagios 模块，你需要将操作委托给你的 Nagios 服务器。

有时候，您希望通过 Nagios 集成实现的目标恰恰相反，实际上，您并不想使其静默，而是希望 Nagios 处理您的测试结果。一个常见的情况是，您希望利用 Nagios 配置来通知管理员某个任务的输出。为此，我们可以使用 Nagios 的`nsca`工具，并将其集成到我们的 playbook 中。Ansible 目前还没有专门管理它的模块，但您始终可以使用命令模块来运行它，借助 `send_nsca` CLI 程序。

# 摘要

在本章中，我们已经了解了如何让 Ansible 向其他系统和/或人员发送通知。

在下一章，我们将学习如何创建一个模块，以便您可以扩展 Ansible 执行任何任务。
