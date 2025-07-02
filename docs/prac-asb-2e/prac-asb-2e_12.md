# 12

# 故障排除和测试策略

与其他类型的代码类似，Ansible 代码也可能包含问题和错误。Ansible 尽量通过在任务执行前检查任务语法来确保尽可能安全。然而，这个检查只会避免一些类型的错误，比如不正确的任务参数，但无法防止其他错误。

还需要记住，由于其本质，我们在 Ansible 代码中描述的是期望的状态，而不是陈述一系列步骤以达到期望的状态。这一差异意味着系统更不容易出现逻辑错误。

然而，剧本中的一个错误可能意味着你所有机器上的潜在配置错误。这应当引起高度重视。尤其是在更改系统关键部分时，例如 SSH 守护进程或 `sudo` 配置，因为风险在于你可能会把自己锁出系统。

有许多方法可以防止或减轻 Ansible 剧本中的错误。在本章中，我们将讨论以下主题：

+   深入排查剧本执行问题

+   使用主机事实来诊断故障

+   使用剧本进行测试

+   使用检查模式

+   解决主机连接问题

+   通过 CLI 传递有效变量

+   限制主机的执行

+   刷新代码缓存

+   检查语法错误

# 技术要求

本章假设你已经根据*第一章*《Ansible 入门》的详细步骤设置好了控制主机，并且使用的是最新版本——本章中的示例是在 Ansible 2.9 上测试的。虽然本章会给出主机名的具体示例，但你可以根据需要用你自己的主机名和/或 IP 地址替换它们。如何做会在适当的地方提供详细说明。

本章中的示例可以在本书的 GitHub 仓库中找到：[`github.com/PacktPublishing/Practical-Ansible-Second-Edition/tree/main/Chapter%2012`](https://github.com/PacktPublishing/Practical-Ansible-Second-Edition/tree/main/Chapter%2012)

# 深入排查剧本执行问题

有些情况下，Ansible 执行会中断，许多原因可能导致这种情况发生。

网络是我在执行 Ansible 剧本时发现的最常见问题原因。由于发出命令的机器和执行命令的机器通常通过网络连接，因此网络出现问题时，通常会立即表现为 Ansible 执行问题。

你可以通过注册变量并使用 `until` 关键字，告诉 Ansible 重复执行某个任务。

有时，尤其是对于某些模块（如`ansible.builtin.shell` 或 `ansible.builtin.command`），即使执行成功，返回码也可能非零。在这种情况下，你可以通过在模块中使用以下行来忽略错误：

```
ignore_errors: yes
```

例如，如果您运行 `/bin/false` 命令，它将始终返回 `1`。为了在 Playbook 中执行该命令并避免它在此处阻塞，您可以编写如下内容：

```
- name: Run a command that will return 1
  ansible.builtin.command: /bin/false
  ignore_errors: yes
```

如我们所见，`/bin/false` 总是返回 `1` 作为返回码，但我们仍然成功地继续执行。请注意，这是一个特殊情况。通常，最好的做法是修复您的应用程序，使其遵循 UNIX 标准，在应用程序正常运行时返回 `0`，而不是在 Playbook 中进行变通处理。

接下来，我们将进一步讨论可以用来诊断 Ansible 执行问题的方法。

# 使用主机事实来诊断失败

一些执行失败是由于目标机器的状态引起的。最常见的此类问题是 Ansible 期望某个文件或变量存在，但实际上它并不存在。

有时，打印机器事实信息足以找到问题。

为此，我们需要创建一个名为 `print_facts.yaml` 的简单 Playbook，其中包含以下内容：

```
---
- hosts: all
  tasks:
  - name: Display all variables/facts known for a host
    ansible.builtin.debug:
      var: hostvars[inventory_hostname]
```

这种技术将在 Ansible 执行期间提供有关目标机器状态的很多信息。

# 使用 Playbook 进行测试

IT 领域最复杂的事情之一不是创建软件和系统，而是当它们出现问题时进行调试。Ansible 也不例外。无论您在编写 Ansible Playbook 方面多么出色，迟早您会发现自己在调试一个没有按预期执行的 Playbook。

执行基本测试的最简单方法是打印出执行过程中变量的值。让我们学习如何在 Ansible 中实现这一点：

1.  首先，我们需要一个名为 `debug.yaml` 的 Playbook，内容如下：

    ```
    ---
    - hosts: localhost
      tasks:
      - ansible.builtin.shell: /usr/bin/uptime
        register: result
      - ansible.builtin.debug:
          var: result
    ```

1.  使用以下命令运行它：

    ```
    $ ansible-playbook debug.yaml
    ```

您将收到类似以下的输出：

```
PLAY [localhost] **********************************************************************************
TASK [Gathering Facts] ****************************************************************************
ok: [localhost]
TASK [shell] **************************************************************************************
changed: [localhost]
TASK [debug] **************************************************************************************
ok: [localhost] => {
    "result": {
        "changed": true,
        "cmd": "/usr/bin/uptime",
        "delta": "0:00:00.003461",
        "end": "2019-06-16 11:30:51.087322",
        "failed": false,
        "rc": 0,
        "start": "2019-06-16 11:30:51.083861",
        "stderr": "",
        "stderr_lines": [],
        "stdout": " 11:30:51 up 40 min, 1 user, load average: 1.11, 0.73, 0.53",
        "stdout_lines": [
            " 11:30:51 up 40 min, 1 user, load average: 1.11, 0.73, 0.53"
        ]
    }
}
PLAY RECAP ****************************************************************************************
ansible.builtin.command module to execute the uptime command and saved its output in the result variable. Then, in the second task, we used the ansible.builtin.debug module to print the content of the result variable.
The `ansible.builtin.debug` module is the module that allows you to print the value of a variable (by using the `var` option) or a fixed string (by using the `msg` option) during Ansible’s execution.
The `ansible.builtin.debug` module also provides the `verbosity` option. Let’s say you change the playbook in the following way:

```

---

- 主机：localhost

任务：

- ansible.builtin.shell: /usr/bin/uptime

注册：result

- ansible.builtin.debug:

变量：result

详细程度：2

```

 Now, if you try to execute it in the same way you did previously, you will notice that the debug step won’t be executed and that the following line will appear in the output instead:

```

任务 [调试] **************************************************************************************

跳过：[localhost]

```

 This is because we set the minimum required `verbosity` to `2`, and by default, Ansible runs with `verbosity` set to `0`.
To see the result of using the debug module with this new playbook, we will need to run a slightly different command:

```

$ ansible-playbook debug2.yaml -vv

```

 By putting two `-v` options in the command line, we will be running Ansible with `verbosity` set to `2`. This will not only affect this specific module but all the modules (or Ansible itself) that are set to behave differently at different debug levels.
Now that you have learned how to test with a playbook, let’s learn how to use check mode.
Using check mode
Although you might be confident in the code you have written, it still pays to test it before running it for real in a production environment. In such cases, it is a good idea to be able to run your code, but with a safety net in place. This is what check mode is for. Follow these steps:

1.  First of all, we need to create an easy playbook to test this feature. Let’s create a playbook called `check-mode.yaml` that contains the following content:

    ```

    ---

    - 主机：localhost

    任务：

    - 名称：触摸一个文件

    ansible.builtin.file:

    路径：/tmp/myfile

    状态：touch

    ```

     2.  Now, we can run the playbook in check mode by specifying the `--check` option in the invocation:

    ```

    $ ansible-playbook check-mode.yaml --check

    ```

This will output everything as if it were really performing the operation, as follows:

```

执行 [localhost] **********************************************************************************

任务 [收集事实] ****************************************************************************

成功：[localhost]

任务 [触摸一个文件] *******************************************************************************

成功：[localhost]

执行回顾 ****************************************************************************************

/tmp，您将找不到 myfile。

Ansible 的检查模式通常被称为干运行（dry run）。其思想是该运行不会改变机器的状态，仅会突出显示当前状态与 Playbook 中声明的状态之间的差异。

并不是所有模块都支持检查模式，但所有主要模块都支持，而且每次发布时，越来越多的模块也在添加支持。特别要注意的是，`ansible.builtin.command` 和 `ansible.builtin.shell` 模块不支持检查模式，因为这些模块无法判断哪些命令会导致变化，哪些不会。因此，这些模块在非检查模式下运行时，始终会返回已更改，因为它们假定已经发生了变化。要了解特定模块是否支持检查模式，可以查看该模块文档中的 *Attributes* 部分。与检查模式类似的功能是 `--diff` 标志。这两者之间的一个重要区别是，虽然检查模式不会对系统进行更改，但 `--diff` 会对系统进行更改！`--diff` 标志允许我们跟踪在 Ansible 执行过程中具体发生了哪些变化。所以，假设我们用以下命令运行相同的剧本：

```
$ ansible-playbook check-mode.yaml --diff
```

这将返回类似以下的内容：

```
PLAY [localhost] **********************************************************************************
TASK [Gathering Facts] ****************************************************************************
ok: [localhost]
TASK [Touch a file] *******************************************************************************
--- before
+++ after
@@ -1,6 +1,6 @@
 {
- "atime": 1560693571.3594637,
- "mtime": 1560693571.3594637,
+ "atime": 1560693571.3620908,
+ "mtime": 1560693571.3620908,
 "path": "/tmp/myfile",
- "state": "absent"
+ "state": "touch"
 }
changed: [localhost]
PLAY RECAP ****************************************************************************************
localhost : ok=2 changed=1 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

如你所见，输出显示 `changed`，这意味着发生了变化（更具体地说，文件已创建），在输出中我们可以看到类似 diff 的输出，告诉我们状态从 `absent` 变为 `touch`，这意味着文件已创建。`mtime` 和 `atime` 也发生了变化，但这可能是由于文件创建和检查的方式所致。

现在你已经学会了如何使用检查模式，让我们学习如何解决主机连接问题。

解决主机连接问题

Ansible 经常用于管理远程主机或系统。为此，Ansible 需要能够连接到远程主机，只有在连接成功后，它才能发出命令。有时，问题在于 Ansible 无法连接到远程主机。一个典型的例子是，当你尝试管理一个尚未启动的机器时。能够快速识别这些问题并及时修复，将帮助你节省大量时间。

按照以下步骤开始操作：

1.  我们创建一个名为 `remote.yaml` 的剧本，内容如下：

    ```
    ---
    - hosts: all
      tasks:
      - name: Touch a file
        ansible.builtin.file:
          path: /tmp/myfile
          state: touch
    ```

    2. 我们可以尝试对一个不存在的 FQDN 运行 `remote.yaml` 剧本，如下所示：

    ```
    $ ansible-playbook -i host.example.com, remote.yaml
    ```

在这种情况下，输出将告诉我们 SSH 服务没有及时回应：

```
PLAY [all] ****************************************************************************************
TASK [Gathering Facts] ****************************************************************************
fatal: [host.example.com]: UNREACHABLE! => {"changed": false, "msg": "Failed to connect to the host via ssh: ssh: Could not resolve hostname host.example.com: Name or service not known", "unreachable": true}
PLAY RECAP ****************************************************************************************
host.example.com : ok=0 changed=0 unreachable=1 failed=0 skipped=0 rescued=0 ignored=0
There is also the possibility that we'll receive a different error:
PLAY [all] ****************************************************************************************
TASK [Gathering Facts] ****************************************************************************
fatal: [host.example.com]: UNREACHABLE! => {"changed": false, "msg": "Failed to connect to the host via ssh: fale@host.example.com: Permission denied (publickey,gssapi-keyex,gssapi-with-mic).", "unreachable": true}
PLAY RECAP ****************************************************************************************
host.example.com : ok=0 changed=0 unreachable=1 failed=0 skipped=0 rescued=0 ignored=0
```

在这种情况下，主机确实有回应，但我们没有足够的权限进行 SSH 连接。

SSH 连接失败通常有两个原因：

+   SSH 客户端无法与 SSH 服务器建立连接

+   SSH 服务器拒绝了 SSH 客户端提供的凭证

由于 OpenSSH 的高度稳定性和向后兼容性，当第一个问题发生时，很可能是 IP 地址或端口错误，因此 TCP 连接不可行。这种错误在 SSH 特定的问题中很少发生。通常，重新检查 IP 地址和主机名（如果是 DNS，检查它是否解析为正确的 IP）就能解决问题。为了进一步调查，你可以尝试从同一台机器上进行 SSH 连接，以检查是否有问题。例如，我会这样做：

```
$ ssh host.example.com -vvv
```

我从错误中获取了主机名，以确保我正在模拟 Ansible 的实际操作。我这样做是为了确保我能看到 SSH 可能提供的所有日志信息，以便排查问题。

第二个问题可能会稍微复杂一些，因为它可能由多种原因引起。其中一种原因是你尝试连接到错误的主机，并且你没有该机器的凭证。另一个常见的情况是用户名错误。要进行调试，你可以使用错误中显示的`user@host`地址（在我的情况下是`fale@host.example.com`），并使用之前使用的相同命令：

```
$ ssh fale@host.example.com -vvv
```

这应该会引发与 Ansible 报告给你的相同的错误，只不过会包含更多细节。

现在你已经学会了如何解决主机连接问题，让我们来学习如何通过 CLI 传递工作变量。

通过 CLI 传递工作变量

在调试过程中，有一件事可以帮助你，且对于代码的可重用性非常有帮助，那就是通过命令行向剧本传递变量。每当你的应用程序——无论是 Ansible 剧本还是任何其他应用程序——接收到来自第三方（在这里是人类）的输入时，应该确保该值是合理的。一个例子就是检查变量是否已设置，并因此不为空字符串。这是一个安全黄金法则，但即使在信任用户的情况下也应应用，因为用户可能会打错变量的名称。应用程序应该识别这一点，并通过保护自身来保护整个系统。请按照以下步骤操作：

1.  我们首先需要的是一个简单的剧本，它打印出变量的内容。让我们创建一个名为`printvar.yaml`的剧本，包含以下内容：

    ```
    ---
    - hosts: localhost
      tasks:
      - ansible.builtin.debug:
          var: variable
    ```

    现在我们有了一个 Ansible 剧本，它可以帮助我们查看一个变量是否已被设置为我们期望的值，让我们用在执行语句中声明的`variable`来运行它：

    ```
    $ ansible-playbook printvar.yaml --extra-vars='{"variable": "Hello, World!"}'
    ```

运行此命令后，我们将得到类似于以下的输出：

```
PLAY [localhost] **********************************************************************************
TASK [Gathering Facts] ****************************************************************************
ok: [localhost]
TASK [debug] **************************************************************************************
ok: [localhost] => {
 "variable": "Hello, World!"
}
PLAY RECAP ****************************************************************************************
localhost : ok=2 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

Ansible 允许以不同的方式和优先级设置变量。更具体地说，你可以通过以下方式设置它们：

+   `/etc/ansible/ansible.cfg`中的系统配置设置（优先级最低）

+   `~/.ansible.cfg`中的用户配置设置

+   当前目录中的`ansible.cfg`项目配置

+   `ANSIBLE_CONFIG`环境变量

+   命令行选项

+   角色默认值

+   库存文件或 `vars` 脚本组

+   `group_vars/all` 库存

+   `group_vars/all` 剧本

+   `group_vars/*` 库存

+   `group_vars/*` 剧本

+   库存文件或 `vars` 脚本主机

+   `host_vars/*` 库存

+   `host_vars/*` 剧本

+   主机事实/缓存的 `set_facts`

+   剧本 `vars`

+   剧本 `vars_prompt`

+   剧本 `vars_files`

+   `vars` 角色（定义在 `role/vars/main.yml`）

+   `vars` 块（仅适用于该块中的任务）

+   `vars` 任务（仅适用于该任务）

+   `include_vars`

+   `set_facts`/注册的变量

+   角色（以及 `include_role`）参数

+   `include` 参数

+   命令行额外的 `vars`（优先级最高）

正如你所看到的，最后一个选项（所有选项中优先级最高的）是使用执行命令中的 `--extra-vars`（或 `-e`）。

现在你已经学习了如何通过 CLI 传递工作变量，接下来我们来学习如何限制主机的执行。

限制主机的执行

在测试剧本时，可能只需要在有限数量的机器上进行测试；例如，只有一台。让我们开始吧：

1.  要使用 Ansible 上的目标主机限制，我们需要一个剧本。创建一个名为 `helloworld.yaml` 的剧本，内容如下：

    ```
    ---
    - hosts: all
      tasks:
      - ansible.builtin.debug:
          msg: "Hello, World!"
    ```

    2. 我们还需要创建一个至少包含两个主机的库存。在我的例子中，我创建了一个名为`inventory`的文件，其中包含以下内容：

    ```
    [hosts]
    host1.example.com
    host2.example.com
    host3.example.com
    ```

我们以常规方式运行剧本，使用以下命令：

```
$ ansible-playbook -i inventory helloworld.yaml
```

通过这样做，我们将收到以下输出：

```
PLAY [all] ****************************************************************************************
TASK [Gathering Facts] ****************************************************************************
ok: [host1.example.com]
ok: [host3.example.com]
ok: [host2.example.com]
TASK [debug] **************************************************************************************
ok: [host1.example.com] => {
 "msg": "Hello, World!"
}
ok: [host2.example.com] => {
 "msg": "Hello, World!"
}
ok: [host3.example.com] => {
 "msg": "Hello, World!"
}
PLAY RECAP ****************************************************************************************
host1.example.com : ok=2 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
host2.example.com : ok=2 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
host3.example.com, we will need to specify this on the command line, as follows:

```

$ ansible-playbook -i inventory helloworld.yaml --limit=host3.example.com

```

 To prove that this works as expected, we can run it. By doing this, we will receive the following output:

```

PLAY [所有] ****************************************************************************************

TASK [收集事实] ****************************************************************************

ok: [host3.example.com]

TASK [调试] **************************************************************************************

ok: [host3.example.com] => {

"msg": "Hello, World!"

}

PLAY RECAP ****************************************************************************************

--limit 关键字，我们可以强制 Ansible 忽略超出限制参数指定的主机。

可以指定多个主机作为列表或使用模式，因此以下两个命令都将对 `host2.example.com` 和 `host3.example.com` 执行剧本：

```
$ ansible-playbook -i inventory helloworld.yaml --limit=host2.example.com,host3.example.com
$ ansible-playbook -i inventory helloworld.yaml --limit=host[2-3].example.com
```

限制不会覆盖库存，而是对其添加限制。假设我们限制为不属于库存的主机，如下所示：

```
$ ansible-playbook -i inventory helloworld.yaml --limit=host4.example.com
```

在这里，我们将收到以下错误，什么也不会做：

```
[WARNING]: Could not match supplied host pattern, ignoring: host4.example.com
ERROR! Specified hosts and/or --limit does not match any hosts
```

现在你已经学习了如何限制主机的执行，接下来我们来学习如何刷新代码缓存。

刷新代码缓存

在 IT 领域，各种操作中都使用缓存来加速操作，Ansible 也不例外。

通常，缓存是有益的，因此它们在各个地方被广泛使用。然而，如果缓存了不应该被缓存的值，或者它们没有被刷新，即使值已经发生了变化，也可能会引发一些问题。

在 Ansible 中刷新缓存非常简单，只需运行`ansible-playbook`，并添加`--flush-cache`选项，如下所示：

```
ansible-playbook -i inventory helloworld.yaml --flush-cache
```

Ansible 可以使用多个缓存插件来保存主机变量和执行变量。有时，这些变量可能会被留下并影响后续的执行。当 Ansible 发现应该在刚刚启动的步骤中设置的变量时，可能会假设该步骤已经完成，因此将旧变量作为刚刚创建的变量。通过使用`--flush-cache`选项，我们可以避免这种情况，因为它会确保在执行期间刷新 Ansible 事实缓存。

现在您已经学会了如何清除代码缓存，让我们学习如何检查错误语法。

检查错误语法

对于机器来说，定义文件是否具有正确的语法非常容易，但对于人类来说可能更复杂。这并不意味着机器可以为您修复代码，但它们可以快速识别问题是否存在。要使用 Ansible 的内置语法检查器，我们需要一个带有语法错误的 playbook。让我们开始吧：

1.  让我们创建一个名为`syntaxcheck.yaml`的文件，其中包含以下内容：

    ```
    ---
    - hosts: all
      tasks:
      - ansible.builtin.debug:
        msg: "Hello, World!"
    ```

    2.现在，我们可以使用`--``syntax-check`命令：

    ```
    $ ansible-playbook syntaxcheck.yaml --syntax-check
    ```

通过这样做，我们将收到以下输出：

```
ERROR! 'msg' is not a valid attribute for a Task
The error appears to be in '/home/fale/ansible/Ansible2Cookbook/Ch11/syntaxcheck.yaml': line 4, column 7, but may
be elsewhere in the file depending on the exact syntax problem.
The offending line appears to be:
 tasks:
 - debug:
 ^ here
This error can be suppressed as a warning using the "invalid_task_attribute_failed" configuration
```

1.  现在我们可以继续修复第 4 行的缩进问题：

    ```
    ---
    - hosts: all
      tasks:
      - ansible.builtin.debug:
          msg: "Hello, World!"
    ```

当我们重新检查语法时，我们将看到它现在没有返回错误：

```
$ ansible-playbook syntaxcheck-fixed.yaml --syntax-check
playbook: syntaxcheck.yaml
```

当语法检查未发现任何错误时，输出将类似于先前的输出，其中列出了已分析文件而未列出任何错误。

由于 Ansible 了解所有支持模块中的所有支持选项，它可以快速读取您的代码并验证您提供的 YAML 是否包含所有必需字段，并且不包含任何不受支持的字段。

摘要

在本章中，您学习了 Ansible 提供的各种选项，以便查找 Ansible 代码中的问题。更具体地说，您学习了如何使用主机事实来诊断故障，如何在 playbook 中包含测试，如何使用检查模式，如何解决主机连接问题，如何从 CLI 传递变量，如何限制执行到主机子集，如何刷新代码缓存以及如何检查错误语法。

在下一章中，您将学习如何开始使用 Ansible 自动化控制器。

问题

回答以下问题来测试您对本章的了解：

1.  真或假：`ansible.builtin.debug`模块允许您在 Ansible 执行期间打印变量值或固定字符串。

    1.  True

    1.  False

1.  哪个关键字允许 Ansible 强制限制主机的执行？

    1.  `--``limit`

    1.  `--``max`

    1.  `--``restrict`

    1.  `--``force`

    1.  `--``except`

进一步阅读

Ansible 的官方文档关于错误处理的内容可以在[`docs.ansible.com/ansible/latest/playbook_guide/playbooks_error_handling.xhtml`](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_error_handling.xhtml)找到。

```

```

```

```
