

# 第九章：Ansible 高级话题

到目前为止，我们已经努力为你打下坚实的 Ansible 基础，无论你需要执行什么自动化任务，都能够轻松自如地实现。然而，当你真正开始大规模自动化时，如何确保在出现任何情况时能够优雅地处理它呢？例如，当你必须启动长期运行的操作时，如何确保可以异步运行这些操作，并且稍后能够可靠地检查结果？或者，如果你正在更新大量服务器，如何确保在少数服务器出现故障时，play 可以及早失败？你最不希望做的事情是将一个有问题的更新（说实话，所有人的代码都有可能会出现问题）推出到 100 台服务器上——如果能够发现少量服务器失败并且基于此中止整个 play，比起继续执行并破坏整个负载均衡集群，要好得多。

本章将探讨如何使用 Ansible 的一些高级功能来控制 playbook 流程和错误处理，解决这些特定问题以及更多问题。我们将通过实际示例，探索如何使用 Ansible 执行滚动更新，如何与代理和跳板主机配合工作（这对安全环境和核心网络配置至关重要），以及如何使用 Ansible Vault 技术来保护静态的敏感数据。通过本章的学习，你将全面了解如何在不仅仅是小型环境中，而是在大型、安全、任务关键型环境中运行 Ansible。

本章将涵盖以下内容：

+   异步与同步操作

+   控制滚动更新的 play 执行

+   配置最大故障百分比

+   设置任务执行委派

+   使用 `run_once` 选项

+   本地运行 playbook

+   与代理和跳板主机协作

+   配置 playbook 提示

+   在 plays 和 tasks 中放置标签

+   使用 Ansible Vault 保护数据

# 技术要求

本章假设你已经按照 *第一章*《Ansible 入门》的详细步骤，成功设置了控制主机，并且正在使用最新的版本。本章中的示例已经在 Ansible 2.15 上经过测试。本章还假设你至少有一台额外的主机用于测试，理想情况下，该主机应为基于 Linux 的。尽管我们将在本章中提供主机名的具体示例，你也可以自由替换为你自己的主机名和/或 IP 地址；如何做到这一点的详细信息会在适当的地方给出。

本章的代码包可以通过 [`github.com/PacktPublishing/Practical-Ansible-Second-Edition/tree/main/Chapter%209`](https://github.com/PacktPublishing/Practical-Ansible-Second-Edition/tree/main/Chapter%209) 获取。

# 异步与同步操作

正如我们在本书中所看到的那样，Ansible 的 play 是按顺序执行的，每个任务在开始下一个任务之前都会运行完成。尽管这种顺序执行对于流程控制和逻辑顺序通常是有利的，但有时你可能并不希望这样。特别是，某些任务可能会运行超过配置的 SSH 连接超时时间，由于 Ansible 在大多数平台上使用 SSH 来执行自动化任务，这可能会成为一个问题。

幸运的是，Ansible 任务可以异步执行——也就是说，任务可以在目标主机的后台运行，并定期进行轮询。这与同步任务不同，后者在任务完成之前会保持与目标主机的连接（这会有超时的风险）。

一如既往，让我们通过一个实际示例来探讨这个问题。假设我们有两台服务器，位于一个简单的 INI 格式的清单中：

```
[frontends]
frt01.example.com
frt02.example.com
```

现在，为了模拟一个长时间运行的任务，我们将使用`shell`模块运行`sleep`命令。然而，为了避免在`sleep`命令执行期间 SSH 连接被阻塞，我们将向任务中添加两个特殊参数，如下所示：

```
---
- name: Play to demonstrate asynchronous tasks
  hosts: frontends
  become: true
tasks:
  - name: A simulated long running task
    shell: "sleep 20"
    async: 30
    poll: 5
```

这两个新参数是`async`和`poll`。`async`参数告诉 Ansible，这个任务应该异步执行（以便 SSH 连接不会被阻塞），最大执行时间为`30`秒。如果任务执行时间超过这个配置的时间，Ansible 会认为任务失败，相应地整个 play 也会失败。当`poll`被设置为正整数时，Ansible 会在指定的间隔检查异步任务的状态——在这个例子中是每`5`秒检查一次。如果`poll`被设置为`0`，那么任务会在后台运行，并且不会被检查——你需要编写一个任务来稍后手动检查它的状态。

注意

如果你没有指定`poll`值，它将会被设置为 Ansible 的`DEFAULT_POLL_INTERVAL`配置参数定义的默认值（即`10`秒）。

当你运行这个 playbook 时，你会发现它和其他 playbook 的运行方式一样；从终端输出来看，你无法察觉任何不同。但在幕后，Ansible 每`5`秒检查一次任务的状态，直到任务成功或达到`async`超时值`30`秒：

```
$ ansible-playbook -i hosts async.yml
PLAY [Play to demonstrate asynchronous tasks] **********************************
TASK [Gathering Facts] *********************************************************
ok: [frt02.example.com]
ok: [frt01.example.com]
TASK [A simulated long running task] *******************************************
changed: [frt02.example.com]
changed: [frt01.example.com]
PLAY RECAP *********************************************************************
frt01.example.com : ok=2 changed=1 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
frt02.example.com : ok=2 changed=1 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

如果你想稍后检查任务的状态（也就是说，如果`poll`被设置为`0`），你可以在 playbook 中添加第二个任务，使其看起来如下：

```
---
- name: Play to demonstrate asynchronous tasks
  hosts: frontends
  become: true
tasks:
  - name: A simulated long running task
    shell: "sleep 20"
    async: 30
    poll: 0
    register: long_task
  - name: Check on the asynchronous task
  async_status:
    jid: "{{ long_task.ansible_job_id }}"
  register: async_result
  until: async_result.finished
  retries: 30
```

在这个剧本中，初始的异步任务定义与之前一样，只是我们现在将`poll`设置为`0`。我们还选择将这个任务的结果注册到一个名为`long_task`的变量中——这样，在稍后检查任务时，我们可以查询任务的作业 ID。接下来的（新的）任务使用`async_status`模块来检查我们从第一个任务注册的作业 ID，并循环直到任务完成或达到`30`次重试——以先到者为准。在剧本中使用这些时，你几乎肯定不会像这样将两个任务紧挨着添加在一起——通常，你会在它们之间执行其他任务——但为了简化这个例子，我们将按顺序运行这两个任务。运行这个剧本应该会产生类似如下的输出：

```
$ ansible-playbook -i hosts async2.yml
PLAY [Play to demonstrate asynchronous tasks] **********************************
TASK [Gathering Facts] *********************************************************
ok: [frt01.example.com]
ok: [frt02.example.com]
TASK [A simulated long running task] *******************************************
changed: [frt02.example.com]
changed: [frt01.example.com]
TASK [Check on the asynchronous task] ******************************************
FAILED - RETRYING: Check on the asynchronous task (30 retries left).
FAILED - RETRYING: Check on the asynchronous task (30 retries left).
FAILED - RETRYING: Check on the asynchronous task (29 retries left).
FAILED - RETRYING: Check on the asynchronous task (29 retries left).
FAILED - RETRYING: Check on the asynchronous task (28 retries left).
FAILED - RETRYING: Check on the asynchronous task (28 retries left).
FAILED - RETRYING: Check on the asynchronous task (27 retries left).
FAILED - RETRYING: Check on the asynchronous task (27 retries left).
changed: [frt01.example.com]
changed: [frt02.example.com]
PLAY RECAP *********************************************************************
frt01.example.com : ok=3 changed=2 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
frt02.example.com : ok=3 changed=2 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

在前面的代码块中，我们可以看到长时间运行的任务仍在运行，而下一个任务轮询其状态，直到我们设置的条件满足。在这种情况下，我们可以看到任务成功完成，整个剧本执行结果也成功。异步操作对于大型下载、软件包更新和其他可能需要较长时间运行的任务尤其有用。你可能会在剧本开发中发现它们的价值，特别是在更复杂的基础设施中。

在掌握了这些基础后，让我们来看看另一个在大型基础设施中可能有用的高级技巧——使用 Ansible 执行滚动更新。

# 控制滚动更新的执行

默认情况下，Ansible 会在多个主机上并行化任务，以加速大型清单中的自动化任务。这个设置由 Ansible 配置文件中的`forks`参数定义，默认为`5`（因此，默认情况下，Ansible 尝试在五台主机上同时运行自动化作业）。

在负载均衡环境中，这并不理想，特别是如果你想避免停机时间。假设我们在清单中有五台前端服务器（或者可能更少）。如果我们允许 Ansible 同时更新所有这些服务器，最终用户可能会经历服务中断。因此，考虑在不同时间更新所有服务器是很重要的。让我们复用前面部分的清单，其中只有两台服务器。显然，如果这些服务器在负载均衡环境中运行，那么我们只更新其中一台是至关重要的；如果两台服务器同时下线，最终用户肯定会失去对服务的访问，直到 Ansible 剧本成功完成。

解决这个问题的方法是在剧本定义中使用`serial`关键字来确定一次操作多少主机。让我们通过一个实际的例子来演示这一点：

1.  创建以下简单的 playbook，在我们的清单中的两个主机上运行两个命令。命令的内容在此阶段不重要，但如果你使用`command`模块运行`date`命令，你将能看到每个任务运行时的时间，以及如果你指定`-v`来提高运行时的详细程度，你将能够看到更多信息：

    ```
    ---
    - name: Simple serial demonstration play
      hosts: frontends
      gather_facts: false
      tasks:
        - name: First task
          command: date
        - name: Second task
          command: date
    ```

1.  现在，如果你运行这个任务，你会看到它在每个主机上同时执行所有操作，因为我们比默认的 forks 数量（`5`）要少。这种行为是 Ansible 的正常行为，但这并不是我们想要的，因为用户将会经历服务中断：

    ```
    $ ansible-playbook -i hosts serial.yml
    PLAY [Simple serial demonstration play] ****************************************
    TASK [First task] **************************************************************
    changed: [frt02.example.com]
    changed: [frt01.example.com]
    TASK [Second task] *************************************************************
    changed: [frt01.example.com]
    changed: [frt02.example.com]
    PLAY RECAP *********************************************************************
    frt01.example.com : ok=2 changed=2 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
    tasks sections exactly as they were in *Step 1*:

    ```

    ---

    - name: 简单的串行演示任务

    hosts: frontends

    serial: 1

    gather_facts: false

    ```

    ```

1.  注意到`serial: 1`这一行的存在。这告诉 Ansible 每次只在`1`个主机上完成任务，然后再继续下一个。如果我们再次运行这个任务，可以看到它的实际效果：

    ```
    $ ansible-playbook -i hosts serial.yml
    PLAY [Simple serial demonstration play] ****************************************
    TASK [First task] **************************************************************
    changed: [frt01.example.com]
    TASK [Second task] *************************************************************
    changed: [frt01.example.com]
    PLAY [Simple serial demonstration play] ****************************************
    TASK [First task] **************************************************************
    changed: [frt02.example.com]
    TASK [Second task] *************************************************************
    changed: [frt02.example.com]
    PLAY RECAP *********************************************************************
    frt01.example.com : ok=2 changed=2 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
    frt02.example.com : ok=2 changed=2 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
    ```

好得多！如果你想象这个 playbook 实际上是在负载均衡器上禁用这些主机，执行升级，然后再在负载均衡器上重新启用这些主机，这正是你希望操作进行的方式。如果没有`serial: 1`指令，所有主机会一次性从负载均衡器中移除，导致服务中断。

值得注意的是，`serial`指令也可以使用百分比而非整数。当你指定百分比时，你是在告诉 Ansible 每次在该百分比的主机上运行任务。所以，如果你的清单中有四个主机并指定`serial: 25%`，Ansible 将每次只在一个主机上运行任务。如果你的清单中有八个主机，它将每次在两个主机上运行任务。我相信你已经明白了！

你甚至可以通过传递一个列表来扩展这个功能，给`serial`指令提供更多选项。考虑以下代码：

```
serial:
- 1
- 3
- 5
```

这告诉 Ansible 初始时只在`1`个主机上运行任务，然后在接下来的`3`个主机上运行，之后每次在`5`个主机上同时运行，直到整个清单完成，以便在特定服务器上部署新的更改。你也可以使用百分比代替主机的整数数量来指定。通过这种方式，你将构建一个强健的 playbook，能够执行滚动更新，而不会对最终用户造成服务中断。完成这一切后，我们可以继续深入了解如何控制 Ansible 在中止任务之前能容忍的最大失败百分比，这在像这样的高可用或负载均衡环境中非常有用。

# 配置最大失败百分比

在默认的操作模式下，只要清单中还有主机，并且没有记录到失败，Ansible 将继续在一批服务器上执行 play（批次大小由我们在前面部分讨论的 `serial` 指令决定）。显然，在一个高可用或负载均衡的环境中（比如我们之前讨论的那种情况），这种方式并不理想。如果你的 play 中有 bug，或者代码部署有问题，你最不希望看到的就是 Ansible 忠实地将它推送到集群中的所有服务器，导致服务中断，因为所有节点都遭遇了升级失败。在这种环境下，最好是尽早失败，至少让集群中的一些主机保持原状，直到有人介入并解决问题。

对于我们的实际示例，假设有一个包含 10 个主机的扩展清单。我们将按如下方式定义它：

```
[frontends]
frt[01:10].example.com
```

现在，让我们创建一个简单的 playbook 在这些主机上运行。我们将在 play 定义中将批次大小设置为 `5`，并将 `max_fail_percentage` 设置为 `50%`：

1.  创建以下 play 定义来演示如何使用 `max_fail_percentage` 指令：

    ```
    ---
    - name: A simple play to demonstrate use of max_fail_percentage
      hosts: frontends
      gather_facts: no
      serial: 5
      max_fail_percentage: 50
    ```

我们在清单中定义了 10 个主机，因此它将按每批 5 个主机进行处理（如 `serial: 5` 所指定）。如果某批次中超过 50% 的主机失败，play 将被中止，处理将停止。

注意

只有失败主机的百分比超过 `max_fail_percentage` 的值时，play 才会中止；如果失败百分比相等，play 将继续执行。

1.  接下来，我们将定义两个简单的任务。第一个任务下有一个特殊的条件，我们用它故意模拟一个失败——这行以 `failed_when` 开头，我们用它告诉任务，如果在批次中的前三台主机上运行该任务，则无论结果如何，都应该故意使任务失败；否则，任务将按正常方式运行：

    ```
    tasks:
    - name: A task that will sometimes fail
      debug:
        msg: This might fail
      failed_when: inventory_hostname in ansible_play_batch[0:3]
    ```

1.  最后，我们将添加第二个任务，该任务将始终成功。它将在 play 允许继续时运行，但如果 play 被中止，则不会运行：

    ```
    - name: A task that will succeed
      debug:
        msg: Success!
    ```

因此，我们故意构建了一个 playbook，它将在包含 10 台主机的清单上每次处理 5 台主机，但如果某个批次中超过 50% 的主机出现失败，play 将被中止。我们还故意设置了一个失败条件，导致第一批 5 台主机中的 3 台（即 60%）失败。

1.  运行 playbook 并观察发生了什么：

    ```
    $ ansible-playbook -i morehosts maxfail.yml
    PLAY [A simple play to demonstrate use of max_fail_percentage] *****************
    TASK [A task that will sometimes fail] *****************************************
    fatal: [frt01.example.com]: FAILED! => {
     "msg": "This might fail"
    }
    fatal: [frt02.example.com]: FAILED! => {
     "msg": "This might fail"
    }
    fatal: [frt03.example.com]: FAILED! => {
     "msg": "This might fail"
    }
    ok: [frt04.example.com] => {
     "msg": "This might fail"
    }
    ok: [frt05.example.com] => {
     "msg": "This might fail"
    }
    NO MORE HOSTS LEFT *************************************************************
    NO MORE HOSTS LEFT *************************************************************
    PLAY RECAP *********************************************************************
    frt01.example.com : ok=0 changed=0 unreachable=0 failed=1 skipped=0 rescued=0 ignored=0
    frt02.example.com : ok=0 changed=0 unreachable=0 failed=1 skipped=0 rescued=0 ignored=0
    frt03.example.com : ok=0 changed=0 unreachable=0 failed=1 skipped=0 rescued=0 ignored=0
    frt04.example.com : ok=1 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
    frt05.example.com : ok=1 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
    ```

注意这个剧本的结果。我们故意让前五个任务中的三个失败，超过了我们设置的`max_fail_percentage`阈值。这立即导致剧本中止，第二个任务不会在前五个主机上执行。你还会注意到，第二批 10 个主机中的 5 个，根本没有被处理，因此我们的剧本确实被中止了。这正是你希望看到的行为，以防止失败的更新在集群中全面扩展。通过巧妙使用批次和`max_fail_percentage`，你可以安全地在整个集群中运行自动化任务，而不必担心在出现问题时会导致整个集群崩溃。在下一节中，我们将看看 Ansible 的另一个非常有用的功能——任务委托。

# 设置任务执行委托

在我们目前为止运行的每个剧本中，我们假设所有任务依次在清单中的每个主机上执行。然而，如果你需要在不同的主机上运行一到两个任务该怎么办？例如，我们讨论过在集群上自动化升级的概念。但从逻辑上讲，我们希望自动化整个过程，包括按顺序将每个主机从负载均衡器中移除，并在任务完成后将其重新加入。

虽然我们仍然希望在整个清单上运行我们的剧本，但我们当然不希望从这些主机上运行负载均衡器命令。让我们通过一个实际的例子，再次详细解释这个问题。我们将重用本章早些时候使用的两个简单的主机清单：

```
[frontends]
frt01.example.com
frt02.example.com
```

现在，为了完成这个任务，让我们在与剧本相同的目录中创建两个简单的 Shell 脚本。这些只是示例，因为设置负载均衡器超出了本书的范围。然而，假设你有一个可以调用的 Shell 脚本（或其他可执行文件），它可以将主机添加到负载均衡器中，也可以从负载均衡器中移除主机：

1.  对于我们的示例，创建一个名为`remove_from_loadbalancer.sh`的脚本，内容如下：

    ```
    #!/bin/sh
    add_to_loadbalancer.sh, which will contain the following:

    ```

    #!/bin/sh

    echo 将 $1 添加到负载均衡器...

    ```

    ```

显然，在实际的例子中，这些脚本中会有更多的代码！

1.  现在，创建一个剧本，执行我们在这里概述的逻辑。我们将首先创建一个非常简单的剧本定义（你可以自由实验`serial`和`max_fail_percentage`指令），并定义一个初始任务：

    ```
    ---
    - name: Play to demonstrate task delegation
      hosts: frontends
      tasks:
        - name: Remove host from the load balancer
          command: ./remove_from_loadbalancer.sh {{ inventory_hostname }}
          args:
            chdir: "{{ playbook_dir }}"
          delegate_to: localhost
    ```

注意任务结构——大部分你应该已经熟悉了。我们正在使用`command`模块调用之前创建的脚本，并将从负载均衡器中移除的主机名传递给脚本。我们使用`chdir`参数和`playbook_dir`魔法变量告诉 Ansible，脚本将在与剧本相同的目录中运行。

这个任务的特殊之处在于`delegate_to`指令，它告诉 Ansible，尽管我们正在遍历一个不包含`localhost`的清单，我们仍然应该在`localhost`上执行此操作（我们并没有将脚本复制到远程主机，因此如果从远程主机尝试运行，它不会执行）。

1.  在这之后，我们添加一个任务，进行升级操作。这个任务没有`delegate_to`指令，因此它会在远程清单主机上运行（如所需）：

    ```
    - name: Deploy code to host
      debug:
        msg: Deployment code would go here....
    ```

1.  最后，我们使用之前创建的第二个脚本将主机重新添加到负载均衡器中。这个任务几乎与第一个任务完全相同：

    ```
    - name: Add host back to the load balancer
      command: ./add_to_loadbalancer.sh {{ inventory_hostname }}
      args:
        chdir: "{{ playbook_dir }}"
      delegate_to: localhost
    ```

1.  参见此剧本的实际运行：

    ```
    $ ansible-playbook -i hosts delegate.yml
    PLAY [Play to demonstrate task delegation] *************************************
    TASK [Gathering Facts] *********************************************************
    ok: [frt01.example.com]
    ok: [frt02.example.com]
    TASK [Remove host from the load balancer] **************************************
    changed: [frt02.example.com -> localhost]
    changed: [frt01.example.com -> localhost]
    TASK [Deploy code to host] *****************************************************
    ok: [frt01.example.com] => {
     "msg": "Deployment code would go here...."
    }
    ok: [frt02.example.com] => {
     "msg": "Deployment code would go here...."
    }
    TASK [Add host back to the load balancer] **************************************
    changed: [frt01.example.com -> localhost]
    changed: [frt02.example.com -> localhost]
    PLAY RECAP *********************************************************************
    frt01.example.com : ok=4 changed=2 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
    frt02.example.com : ok=4 changed=2 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
    ```

请注意，尽管 Ansible 正在遍历清单（其中不包含`localhost`），但与负载均衡器相关的脚本实际上是在`localhost`上运行的，而升级任务则直接在远程主机上执行。当然，这并不是任务委派的唯一用途，但它是一个常见的例子，说明它如何帮助你。

事实上，你可以将任何任务委派给`localhost`，或者甚至是其他非清单主机。例如，你可以运行一个委派给`localhost`的`rsync`命令，使用类似之前的任务定义将文件复制到远程主机。另请注意，你可以选择在剧本（和角色）中使用一种简写符号来表示`delegate_to`，称为`local_action`。这样，你就可以在一行中指定一个任务，通常情况下会在下面添加`delegate_to: localhost`。将这一切汇总成第二个示例，我们的剧本将如下所示：

```
---
- name: Second task delegation example
  hosts: frontends
  tasks:
  - name: Perform an rsync from localhost to inventory hosts
    local_action: command rsync -a /tmp/ {{ inventory_hostname }}:/tmp/target/
```

前述简写符号等价于以下内容：

```
tasks:
- name: Perform an rsync from localhost to inventory hosts
  command: rsync -a /tmp/ {{ inventory_hostname }}:/tmp/target/
  delegate_to: localhost
```

如果我们运行这个剧本，我们可以看到`local_action`实际上是在运行 Ansible 的机器上执行一个模块（通常是`localhost`，但不一定总是），使我们能够高效地将整个目录树复制到清单中的远程服务器上：

```
$ ansible-playbook -i hosts delegate2.yml
PLAY [Second task delegation example] ******************************************
TASK [Gathering Facts] *********************************************************
ok: [frt02.example.com]
ok: [frt01.example.com]
TASK [Perform a rsync from localhost to inventory hosts] **********************
changed: [frt02.example.com -> localhost]
changed: [frt01.example.com -> localhost]
PLAY RECAP *********************************************************************
frt01.example.com: ok=2 changed=1 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
frt02.example.com: ok=2 changed=1 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

这就结束了我们对任务委派的讲解；尽管如前所述，这只是两个常见的示例。我相信你可以想出一些更高级的使用场景。让我们继续探讨控制 Ansible 代码流的内容，接下来我们将进入看`run_once`选项的特殊用法。

# 使用`run_once`选项

在处理集群时，你有时会遇到一个任务，只需对整个集群执行一次。例如，你可能想要升级集群数据库的模式，或者发出命令重新配置 Pacemaker 集群，这通常会在一个节点上执行，然后由集群管理软件自动传播到所有其他节点。你当然可以通过一个只包含一个主机的特殊清单来解决这个问题，或者通过编写一个特殊的剧本来引用清单中的一个主机，但这样效率低下，且开始使代码变得零散。

另外，您可以像平常一样编写代码，但对于希望在清单中只执行一次的任务，使用特殊的`run_once`指令。例如，让我们重新使用本章前面定义的 10 台主机清单。现在，让我们继续演示此选项，如下所示：

1.  创建一个简单的剧本，如以下代码块所示。我们使用`debug`语句来显示一些输出，但在实际应用中，您会在此插入执行一次性集群操作的脚本或命令（例如，升级数据库架构）：

    ```
    ---
    - name: Play to demonstrate the run_once directive
      hosts: frontends
      tasks:
        - name: Upgrade database schema
          debug:
            msg: Upgrading database schema...
          run_once: true
    ```

1.  现在，运行此剧本并查看会发生什么：

    ```
    $ ansible-playbook -i morehosts runonce.yml
    PLAY [Play to demonstrate the run_once directive] ******************************
    TASK [Gathering Facts] *********************************************************
    ok: [frt02.example.com]
    ok: [frt05.example.com]
    ok: [frt03.example.com]
    ok: [frt01.example.com]
    ok: [frt04.example.com]
    ok: [frt06.example.com]
    ok: [frt08.example.com]
    ok: [frt09.example.com]
    ok: [frt07.example.com]
    ok: [frt10.example.com]
    TASK [Upgrade database schema] *************************************************
    ok: [frt01.example.com] => {
     "msg": "Upgrading database schema..."
    }
    ---
    PLAY RECAP *********************************************************************
    frt01.example.com : ok=2 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
    frt02.example.com : ok=1 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
    frt03.example.com : ok=1 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
    frt04.example.com : ok=1 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
    frt05.example.com : ok=1 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
    frt06.example.com : ok=1 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
    frt07.example.com : ok=1 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
    frt08.example.com : ok=1 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
    frt09.example.com : ok=1 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
    frt10.example.com : ok=1 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
    ```

请注意，正如所期望的，尽管剧本在所有 10 个主机上运行（并且确实收集了所有 10 个主机的事实数据），但 Ansible 只在一台主机上执行了升级任务。

1.  需要注意的是，`run_once`选项是针对每批服务器的，因此如果我们在任务定义中添加`serial: 5`（在 10 个服务器的清单中分成 2 批，每批 5 台服务器执行），则架构升级任务会执行两次！它按要求执行了一次，但每批服务器执行一次，而不是对整个清单执行一次。处理该指令时，在集群环境中要小心这一细微差别。

在您的任务定义中添加`serial: 5`并重新运行剧本。输出应如下所示：

```
$ ansible-playbook -i morehosts runonce.yml
PLAY [Play to demonstrate the run_once directive] ******************************
TASK [Gathering Facts] *********************************************************
ok: [frt04.example.com]
ok: [frt01.example.com]
ok: [frt02.example.com]
ok: [frt03.example.com]
ok: [frt05.example.com]
TASK [Upgrade database schema] *************************************************
ok: [frt01.example.com] => {
 "msg": "Upgrading database schema..."
}
PLAY [Play to demonstrate the run_once directive] ******************************
TASK [Gathering Facts] *********************************************************
ok: [frt08.example.com]
ok: [frt06.example.com]
ok: [frt07.example.com]
ok: [frt10.example.com]
ok: [frt09.example.com]
TASK [Upgrade database schema] *************************************************
ok: [frt06.example.com] => {
 "msg": "Upgrading database schema..."
}
PLAY RECAP *********************************************************************
frt01.example.com : ok=2 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
frt02.example.com : ok=1 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
frt03.example.com : ok=1 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
frt04.example.com : ok=1 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
frt05.example.com : ok=1 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
frt06.example.com : ok=2 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
frt07.example.com : ok=1 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
frt08.example.com : ok=1 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
frt09.example.com : ok=1 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
run_once option is designed to work—you can observe, in the preceding output, that our schema upgrade ran twice, which is probably not something we wanted! However, with this awareness, you should be able to take advantage of this option to control your playbook flow across clusters and still achieve the results you want. Let’s now move away from cluster-related Ansible tasks and look at the subtle but important difference between running playbooks locally and running them on localhost.
Running playbooks locally
It is important to note that when we talk about running a playbook locally with Ansible, it is not the same as talking about running it on `localhost`. If we run a playbook on `localhost`, Ansible actually sets up an SSH connection to `localhost` (it doesn’t differentiate its behavior or attempt to detect whether a host in the inventory is local or remote—it simply tries faithfully to connect).
Indeed, we can try creating a `local` inventory file with the following contents:

```

[local]

localhost

```

 Now, if we attempt to run the `ping` module in an ad hoc command against this inventory, we see the following:

```

$ ansible -i localhosts -m ping all --ask-pass

无法确认主机 'localhost (::1)' 的真实性。

ECDSA 密钥指纹为 SHA256:DUwVxH+45432pSr9qsN8Av4l0KJJ+r5jTo123n3XGvZs。

ECDSA 密钥指纹为 MD5:78:d1:dc:23:cc:28:51:42:eb:fb:58:49:ab:92:b6:96。

您确定要继续连接吗（是/否）？ 是

SSH 密码：

localhost | SUCCESS => {

"ansible_facts": {

"discovered_interpreter_python": "/usr/bin/python"

},

"changed": false,

"ping": "pong"

}

```

 As you can see, Ansible set up an SSH connection that needed the host key to validate, as well as our SSH password. Now, although you could add the host key (as we did in the preceding code block), add key-based SSH authentication to your `localhost`, and so on, there is a more direct way of doing this.
We can now modify our inventory so that it looks as follows:

```

[local]

localhost ansible_connection=local

```

 We’ve added a special variable to our `localhost` entry—the `ansible_connection` variable—which defines which protocol is used to connect to this inventory host. So, we have told it to use a direct local connection instead of SSH-based connectivity (which is the default).
It should be noted that `ansible_connection` specifies the type of communication, such as `local`/`ssh`/`etc`. So, if we change our inventory to look as follows, Ansible will not even attempt to connect to the remote host called `frt01.example.com`—it will connect locally to the machine running the playbook (without SSH):

```

[local]

frt01.example.com ansible_connection=local

```

 We can demonstrate this very simply. Let’s first check for the absence of a test file in our local `/``tmp` directory:

```

ls -l /tmp/foo

ls: 无法访问 /tmp/foo: 没有那个文件或目录

```

 Now, let’s run an ad hoc command to touch this file on all hosts in the new inventory we just defined:

```

$ ansible -i localhosts2 -m file -a "path=/tmp/foo state=touch" all

frt01.example.com | CHANGED => {

"ansible_facts": {

"discovered_interpreter_python": "/usr/bin/python"

},

"changed": true,

"dest": "/tmp/foo",

"gid": 0,

"group": "root",

"mode": "0644",

"owner": "root",

"size": 0,

"state": "file",

"uid": 0

}

```

 The command ran successfully, so let’s see whether the test file is present on the local machine:

```

$ ls -l /tmp/foo

-rw-r--r-- 1 root root 0 Apr 24 16:28 /tmp/foo

```

 It is! So, the ad hoc command did not attempt to connect to `frt01.example.com`, even though this hostname was in the inventory. The presence of `ansible_connection=local` meant that this command was run on the local machine without using SSH.
This ability to run commands locally without the need to set up SSH connectivity, SSH keys, and so on can be incredibly valuable, especially if you need to get things up and running quickly on your local machine. With this complete, let’s take a look at how you can work with proxies and jump hosts using Ansible.
Working with proxies and jump hosts
Often, when it comes to configuring core network devices, these are isolated from the main network via a proxy or jump host. Ansible lends itself well to automating network device configuration as most of it is performed over SSH; however, this is only helpful in a scenario where Ansible can either be installed and operated from the jump host or, better yet, can operate via a host such as this.
Fortunately, Ansible can do exactly that. Let’s assume that you have two Cumulus Networks switches in your network (these are based on a special distribution of Linux for switching hardware, which is very similar to Debian). These two switches have the `cmls01.example.com` and `cmls02.example.com` hostnames, but both can only be accessed from a host called `bastion.example.com`.
The configuration to support our `bastion` host is performed in the inventory, rather than in the playbook. We begin by defining an inventory group with the switches in, in the normal manner:

```

[switches]

cmls01.example.com

cmls02.example.com

```

 However, we can now start to get clever by adding some special SSH arguments into the inventory variables for this group. Add the following code to your inventory file:

```

[switches:vars]

ansible_ssh_common_args='-o ProxyCommand="ssh -W %h:%p -q bastion.example.com"'

```

 This special variable content tells Ansible to add extra options when it sets up an SSH connection, including to proxy via the `bastion.example.com` host. The `-W %h:%p` options tell SSH to proxy the connection and to connect to the host specified by `%h` (this is either `cmls01.example.com` or `cmls02.example.com`) on the port specified by `%p` (usually port `22`).
Now, if we attempt to run the Ansible `ping` module against this inventory, we can see whether it works:

```

$ ansible -i switches -m ping all

cmls02.example.com | SUCCESS => {

"ansible_facts": {

"discovered_interpreter_python": "/usr/bin/python"

},

"changed": false,

cmls01.example.com | SUCCESS => {

"ansible_facts": {

"discovered_interpreter_python": "/usr/bin/python"

},

"changed": false,

"ping": "pong"

}

```

 You will notice that we can’t actually see any differences in Ansible’s behavior from the command-line output. On the surface, Ansible works just as it normally does and connects successfully to the two hosts. However, behind the scenes, it proxies via `bastion.example.com`.
Note that this simple example assumes that you are connecting to both the `bastion` host and switches using the same username and SSH credentials (or in this case, keys). There are ways to provide separate credentials for both variables, but this involves more advanced usage of OpenSSH, which is beyond the scope of this book. However, this section intends to give you a starting point and demonstrate the possibility of this, and you are free to explore OpenSSH proxying by yourself.
Let’s now change track and explore how it is possible to set up Ansible to prompt you for data during a playbook run.
Configuring playbook prompts
So far, all of our playbooks have had their data specified for them at runtime in variables we defined within the playbook. However, what if you actually want to obtain information from someone during a playbook run? Perhaps you want a user to select a version of a package to install? Or, perhaps you want to obtain a password from a user for an authentication task without storing it anywhere. (Although Ansible Vault can encrypt the data at rest, some companies may forbid the storing of passwords and other such credentials in tools that they have not evaluated.) Fortunately, for these instances (and many more), Ansible can prompt you for user input and store the input in a variable for future processing.
Let’s reuse the two host frontend inventories we defined at the beginning of this chapter. Now, let’s demonstrate how to capture data from users during a playbook run with a practical example:

1.  Create a simple play definition in the usual manner, as follows:

    ```

    ---

    - name: 一个简单的示例，演示如何在剧本中提示输入

    主机：frontends

    ```

     2.  Now, we will add a special section to the play definition. We previously defined a `vars` section, but this time, we will define one called `vars_prompt` (which enables you to do just that—define variables through user prompts). In this section, we will prompt for two variables—one for a user ID and one for a password. One will be echoed to the screen, while the other will not be, by setting `private: yes`:

    ```

    vars_prompt:

    - name: loginid

    提示：请输入您的用户名

    private: no

    - name: password

    提示：请输入您的密码

    private: yes

    ```

     3.  We will now add a single task to our playbook to demonstrate this prompting process of setting the variables:

    ```

    任务：

    - name: 继续登录

    debug:

    msg: "正在以 {{ loginid }} 身份登录..."

    ```

     4.  Now, run the playbook and see how it behaves:

    ```

    $ ansible-playbook -i hosts prompt.yml

    请输入您的用户名：james

    请输入您的密码：

    PLAY [一个简单的示例，演示如何在 playbook 中进行提示] ********************

    任务 [收集信息] *********************************************************

    ok: [frt01.example.com]

    ok: [frt02.example.com]

    任务 [继续登录] ******************************************************

    ok: [frt01.example.com] => {

    "msg": "正在以 james 身份登录..."

    }

    ok: [frt02.example.com] => {

    "msg": "正在以 james 身份登录..."

    }

    PLAY RECAP *********************************************************************

    frt01.example.com : ok=2 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0

    frt02.example.com : ok=2 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0

    ```

As you can see, we are prompted for both variables, yet the password is not echoed to the terminal, which is important for security reasons. We can then make use of the variables later in the playbook. Here, we just used a simple `debug` command to demonstrate that the variables have been set; however, you would instead implement an actual authentication function in place of this.
With this complete, let’s proceed to the next section and look at how you can selectively run your tasks from within your plays with the use of tags.
Placing tags in the plays and tasks
We have discussed, at many points in this book, that as your confidence and experience with Ansible grows, it is likely that your playbooks will grow, both in scale and complexity. While this is undoubtedly a good thing, there may be times when you only want to run a subset of a playbook, rather than running it from beginning to end. We discussed how to conditionally run tasks based on the value of a variable or fact, but is there a way we can run them based on a selection made when the playbook is run?
Tags in Ansible plays are the solution to this, and in this section, we will build a simple playbook with two tasks—each bearing a different tag—to show you how tags work. We will work with the two simple host inventories that we worked with previously:

1.  Create the following simple playbook to perform two tasks—one to install the `nginx` package and the other to deploy a configuration file from a template:

    ```

    ---

    - name: 简单示例，演示如何使用标签

    hosts: frontends

    任务：

    - name: 安装 nginx

    yum:

    name: nginx

    state: present

    tags:

    - install

    - name: 从模板安装 nginx 配置

    模板：

    src: templates/nginx.conf.j2

    dest: /etc/nginx.conf

    tags:

    - customize

    ```

     2.  Now, run the playbook in the usual manner, but with one difference—this time, we will add the `--tags` switch to the command line. This switch tells Ansible to only run the tasks that have tags matching the ones that are specified. So, for example, run the following command:

    ```

    $ ansible-playbook -i hosts tags.yml --tags install

    PLAY [简单示例，演示如何使用标签] **********************************

    任务 [收集信息] *********************************************************

    ok: [frt02.example.com]

    ok: [frt01.example.com]

    任务 [安装 nginx] ***********************************************************

    changed: [frt02.example.com]

    changed: [frt01.example.com]

    PLAY RECAP *********************************************************************

    frt01.example.com : ok=2 changed=1 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0

    frt02.example.com : ok=2 changed=1 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0

    ```

Notice that the task to deploy the configuration file does not run. This is because it is tagged with `customize` and we did not specify this tag when running the playbook.

1.  There is also a `--skip-tags` switch that does the reverse of the previous switch—it tells Ansible to skip the tags listed. So, if we run the playbook again but skip the `customize` tag, we should see an output like the following:

    ```

    $ ansible-playbook -i hosts tags.yml --skip-tags customize

    PLAY [简单示例，演示如何使用标签] **********************************

    任务 [收集信息] *********************************************************

    ok: [frt02.example.com]

    ok: [frt01.example.com]

    任务 [安装 nginx] ***********************************************************

    ok: [frt02.example.com]

    ok: [frt01.example.com]

    PLAY RECAP *********************************************************************

    frt01.example.com : ok=2 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0

    frt02.example.com : ok=2 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0

    ```

This play run is identical because, rather than including only the `install`-tagged tasks, we skipped the tasks tagged with `customize`.
Note that if you do not specify either `--tags` or `--skip-tags`, then all the tasks are run, regardless of their tag.
A few notes about tags. First of all, each task can have more than one tag, so we see them specified in a YAML list format. If you use the `--tags` switch, a task will run if any of its tags match the tag that was specified on the command line. Secondly, tags can be reused, so we could have five tasks that are all tagged `install`, and all five tasks would be performed or skipped if you requested them to do so via `--tags` or `--``skip-tags`, respectively.
You can also specify more than one tag on the command line, running all the tasks that match any of the specified tags. Although the logic behind tags is relatively simple, it can take a little while to get used to it, and the last thing you want to do is run your playbook on real hosts to check whether you understand tagging! A great way to figure this out is to add `--list-tasks` to your command, which, rather than running the playbook, lists the tasks from the playbook that would perform if you run it. Some examples are provided for you in the following code block, based on the example playbook we just created:

```

$ ansible-playbook -i hosts tags.yml --skip-tags customize --list-tasks

playbook: tags.yml

play #1 (前端服务器)：简单示例，演示如何使用标签 TAGS: []

任务：

安装 nginx TAGS: [install]

$ ansible-playbook -i hosts tags.yml --tags install,customize --list-tasks

playbook: tags.yml

play #1 (前端服务器)：简单示例，演示如何使用标签 TAGS: []

任务：

安装 nginx TAGS: [install]

从模板安装 nginx 配置 TAGS: [customize]

$ ansible-playbook -i hosts tags.yml --list-tasks

playbook: tags.yml

play #1 (frontends): 用于演示标签使用的简单剧本 TAGS: []

tasks:

安装 nginx TAGS: [install]

安装来自模板的 nginx 配置 TAGS: [customize]

```

 As you can see, not only does `--list-tasks` show you which tasks would run but it also shows you which tags are associated with them, which helps you further understand how tagging works and ensure that you achieve the playbook flow that you wanted. Tags are an incredibly simple yet powerful way to control which parts of your playbook run and, often, when it comes to creating and maintaining large playbooks, it is better to be able to run only selected parts of the playbook at once. From here, we will move on to the final section of this chapter, where we will look at securing your variable data at rest by encrypting it with Ansible Vault.
Securing data with Ansible Vault
Ansible Vault is a tool included with Ansible that allows you to encrypt your sensitive data at rest, while also using it in a playbook. Often, it is necessary to store login credentials or other sensitive data in a variable to allow a playbook to run unattended. However, this risks exposing your data to people who might use it with malicious intent. Fortunately, Ansible Vault secures your data at rest using AES-256 encryption, meaning your sensitive data is safe from prying eyes.
Let’s proceed with a simple example that shows you how you can use Ansible Vault:

1.  Start by creating a new vault to store sensitive data in; we will call this file `secret.yml`. You can create this using the following command:

    ```

    $ ansible-vault create secret.yml

    新 Vault 密码：

    确认新 Vault 密码：

    ```

Enter the password you have chosen for the vault when prompted and confirm it by entering it a second time (the vault that accompanies this book on GitHub is encrypted with the `secure` password).

1.  When you have entered the password, you will be sent to your normal editor (defined by the `EDITOR` shell variable). On my test system, this is `vi`. Within this editor, you should create a `vars` file, in the normal manner, containing your sensitive data:

    ```

    ---

    secretdata: "Ansible 很酷！"

    ```

     2.  Save and exit the editor (press *Esc*, then `:wq` in `vi`). You will exit the shell. Now, if you look at the contents of your file, you will see that they are encrypted and are safe from anyone who should not be able to read the file:

    ```

    $ cat secret.yml

    $ANSIBLE_VAULT;1.1;AES256

    63333734623764633865633237333166333634353334373862346334643631303163653931306138

    6334356465396463643936323163323132373836336461370a343236386266313331653964326334

    62363737663165336539633262366636383364343663396335643635623463626336643732613830

    6139363035373736370a646661396464386364653935636366633663623261633538626230616630

    35346465346430636463323838613037386636333334356265623964633763333532366561323266

    变量文件（虽然，显然你必须告诉 Ansible 你的 Vault 密码）。创建一个简单的剧本如下：

    ```
    ---
    - name: A play that makes use of an Ansible Vault
    hosts: frontends
    vars_files:
    - secret.yml
    tasks:
    - name: Tell me a secret
    debug:
    msg: "Your secret data is: {{ secretdata }}"
    ```

    ```

The `vars_files` directive is used in the same way as it would be if you were using an unencrypted `variables` file. Ansible reads the headers of the `variables` files at runtime and determines whether they are encrypted or not.

1.  Try running the playbook without telling Ansible what the vault password is—in this instance, you should receive an error such as this:

    ```

    $ ansible-playbook -i hosts vaultplaybook.yml

    使用 ansible-vault 加密的变量文件，但我们必须手动提供密码才能继续。有多种方式可以指定 Vault 的密码（稍后会详细介绍），但为了简便，请尝试运行以下命令，并在提示时输入你的 Vault 密码：

    ```
    $ ansible-playbook -i hosts vaultplaybook.yml --ask-vault-pass
    Vault password:
    PLAY [A play that makes use of an Ansible Vault] *******************************
    TASK [Gathering Facts] *********************************************************
    ok: [frt01.example.com]
    ok: [frt02.example.com]
    TASK [Tell me a secret] ********************************************************
    ok: [frt01.example.com] => {
     "msg": "Your secret data is: Ansible is cool!"
    }
    ok: [frt02.example.com] => {
     "msg": "Your secret data is: Ansible is cool!"
    }
    PLAY RECAP *********************************************************************
    frt01.example.com : ok=2 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
    frt02.example.com : ok=2 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
    ```

    ```

Success! Ansible decrypted our vault file and loaded the variables into the playbook, which we can see from the `debug` statement we created. This is a very simple example of what you can do with vaults. There are multiple ways that you can specify passwords; you don’t have to be prompted for them on the command line—they can be provided either by a plain text file that contains the vault password or via a script that could obtain the password from a secure location at runtime (think of a dynamic inventory script, only for returning a password rather than a hostname). The `ansible-vault` tool itself can also be used to edit, view, and change the passwords in a vault file, or even decrypt it and turn it back into plain text. The user guide for Ansible Vault is a great place to start for more information ([`docs.ansible.com/ansible/latest/user_guide/vault.xhtml`](https://docs.ansible.com/ansible/latest/user_guide/vault.xhtml)).
One thing to note is that you don’t actually have to have a separate vault file for your sensitive data; you can include it inline in your playbook. For example, let’s try re-encrypting our sensitive data for inclusion in an otherwise unencrypted playbook (again, use the `secure` password for the vault if you are testing the examples from the GitHub repository accompanying this book). Run the following command in your shell (it should produce an output similar to what is shown):

```

$ ansible-vault encrypt_string 'Ansible is cool!' --name secretdata

新 Vault 密码：

确认新 Vault 密码：

secretdata: !vault |

$ANSIBLE_VAULT;1.1;AES256

34393431303339353735656236656130336664666337363732376262343837663738393465623930

3366623061306364643966666565316235313136633264310a623736643362663035373861343435

62346264313638656363323835323833633264636561366339326332356430383734653030306637

3736336533656230380a316364313831666463643534633530393337346164356634613065396434

33316338336266636666353334643865363830346566666331303763643564323065

加密成功

```

 You can copy and paste the output of this command into a playbook. So, if we modify our earlier example, it would appear as follows:

```

---

- name: 使用 Ansible Vault 的剧本

hosts: frontends

vars:

secretdata: !vault |

$ANSIBLE_VAULT;1.1;AES256

34393431303339353735656236656130336664666337363732376262343837 663738393465623930

3366623061306364643966666565316235313136633264310a623736643362 663035373861343435

62346264313638656363323835323833633264636561366339326332356430 383734653030306637

3736336533656230380a316364313831666463643534633530393337346164 356634613065396434

33316338336266636666353334643865363830346566666331303763643564 323065

tasks:

- name: 告诉我一个秘密

debug:

msg: "你的机密数据是：{{ secretdata }}"

```

 Now, when you run this playbook in exactly the same manner as we did before (specifying the vault password using a user prompt), you should see that it runs just as when we used an external encrypted `variables` file:

```

$ ansible-playbook -i hosts inlinevaultplaybook.yml --ask-vault-pass

Vault 密码：

剧本 [A play that makes use of an Ansible Vault] *******************************

任务 [Gathering Facts] *********************************************************

ok: [frt02.example.com]

ok: [frt01.example.com]

任务 [告诉我一个秘密] *******************************************************

ok: [frt01.example.com] => {

"msg": "你的机密数据是：Ansible 很酷！"

}

ok: [frt02.example.com] => {

"msg": "你的机密数据是：Ansible 很酷！"

}

PLAY RECAP *********************************************************************

frt01.example.com : ok=2 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0

frt02.example.com : ok=2 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0

```

 Ansible Vault is a powerful and versatile tool for encrypting your sensitive playbook data at rest and should enable you (with a little care) to run most of your playbooks unattended without ever leaving passwords or other sensitive data in the clear. If you add multiple encrypted files in your `group_vars` directory, all files need to contain the same password. That concludes this section and this chapter; I hope that it has been useful to you.
Summary
Ansible has many advanced features that allow you to run your playbooks in a variety of scenarios, whether that is upgrading a cluster of servers in a controlled manner; working with devices on a secure, isolated network; or controlling your playbook flow with prompts and tags. Ansible has been adopted by a large and ever-growing user base and, as such, is designed and evolved around solving real-world problems. Most of the advanced features of Ansible we discussed are centered around exactly this—solving real-world problems.
In this chapter, you learned about running tasks asynchronously in Ansible, before looking at the various features available for running playbooks to upgrade a cluster, such as running tasks on small batches of inventory hosts, failing a play early if a certain percentage of hosts fail, delegating tasks to a specific host, and even running tasks once, regardless of your inventory (or batch) size. You also learned about the difference between running playbooks locally as opposed to on `localhost` and how to use SSH-proxying to automate tasks on an isolated network via a `bastion` host. Finally, you learned about handling sensitive data without storing it unencrypted at rest, either through prompting the user at runtime or through the use of Ansible Vault. You even learned about running a subset of your playbook tasks with tagging.
In the next chapter, we will explore a topic we touched on briefly in this chapter in more detail—automating network device management with Ansible.
Questions

1.  Which parameter allows you to configure the maximum number of hosts in a batch that will fail before a play is aborted?
    1.  `percentage`
    2.  `max_fail`
    3.  `max_fail_percentage`
    4.  `max_percentage`
    5.  `fail_percentage`
2.  True or false? You can use the `--ask-vault-pass` parameter to use the vault to keep sensitive data at rest:
    1.  True
    2.  False
3.  True or false? To run a playbook asynchronously, you need to use the `async` keyword:
    1.  True
    2.  False

Further reading
If you install Passlib, which is a password-hashing library for Python 3, `vars_prompt` is encrypted with any crypt scheme (such as `descrypt`, `md5crypt`, `sha56_crypt`, and more). You can learn more about this here:
[`passlib.readthedocs.io/en/stable/`](https://passlib.readthedocs.io/en/stable/)

```

# 第三部分：在企业中使用 Ansible

在本节中，我们将实用地探讨如何在企业环境中充分发挥 Ansible 的作用。我们将从如何使用 Ansible 自动化网络设备开始，然后继续探讨如何使用 Ansible 管理云环境和容器环境。接着，我们将研究一些更高级的测试和故障排除策略，帮助你在企业中使用 Ansible 时解决问题，之后将重点介绍 Ansible 自动化控制器/ **Ansible Web eXecutable**（**AWX**）产品，它提供丰富的**基于角色的访问控制**（**RBAC**）和审计功能，支持多种执行环境中的企业设置。最后，我们将深入探讨执行环境。

本节包含以下章节：

+   *第十章*，*使用 Ansible 进行网络自动化*

+   *第十一章*，*容器和云管理*

+   *第十二章*，*故障排除和测试策略*

+   *第十三章*，*Ansible 自动化控制器入门*

+   *第十四章*，*执行环境*
