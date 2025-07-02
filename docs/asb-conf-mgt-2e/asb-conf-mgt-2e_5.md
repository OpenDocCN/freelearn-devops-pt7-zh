# 第五章 自定义模块

直到现在，我们一直在使用 Ansible 提供的工具。虽然这赋予了我们很大的能力，并使许多事情成为可能，但如果你遇到特别复杂的情况，或者经常使用脚本模块，那么你很可能希望学习如何扩展 Ansible。

在本章中，你将学习以下主题：

+   如何在 Bash 脚本或 Python 中编写模块

+   使用你开发的自定义模块

+   编写一个脚本，将外部数据源作为库存

当你在 Ansible 中处理复杂的任务时，通常会编写一个脚本模块。脚本模块的问题在于，你不能轻松地处理它们的输出或根据输出触发处理程序。因此，尽管脚本模块在某些情况下有效，但使用模块可能会更好。

当出现以下情况时，请使用模块而不是编写脚本：

+   你不想每次都运行脚本

+   你需要处理输出

+   你的脚本需要生成事实

+   你需要将复杂的变量作为参数传递

如果你想开始编写模块，应该查看 Ansible 的代码库。如果你希望你的模块与某个特定版本兼容，也应该切换到该版本以确保兼容性。以下命令将帮助你为 Ansible 1.3.0 开发模块。

```
$ git clone (https://github.com/ansible/ansible.git)
$ cd ansible
$ git checkout v1.3.0
$ chmod +x hacking/test-module

```

查看 Ansible 代码库可以让你获得一个方便的脚本，我们将在后面用它来测试我们的模块。我们还将使这个脚本可执行，以便在本章后续部分使用。

# 使用 Bash 编写模块

Ansible 允许你使用任何你喜欢的语言编写模块。虽然 Ansible 中的大多数模块使用 JSON，但如果你没有 JSON 解析工具，你可以使用快捷方式。Ansible 会以原始键值对的形式传递参数给你，如果它们是以这种格式提供的。如果提供了复杂的参数，你将收到 JSON 编码的数据。你可以使用 jsawk ([`github.com/micha/jsawk`](https://github.com/micha/jsawk)) 或 jq ([`stedolan.github.io/jq/`](http://stedolan.github.io/jq/)) 等工具解析这些数据，但前提是它们已经安装在你的远程机器上。

Ansible 已经有一个模块，可以让你更改系统的主机名，但它只适用于基于 systemd 的系统。所以我们来编写一个适用于标准`hostname`命令的模块。我们将从打印当前主机名开始，然后再逐步扩展脚本。下面是这个简单模块的样子：

```
#!/bin/bash

HOSTNAME="$(hostname)"

echo "hostname=${HOSTNAME}"
```

如果你之前写过 Bash 脚本，这应该显得非常基础。基本上，我们做的就是获取主机名并以键值对的形式打印出来。现在我们已经编写了模块的初步版本，接下来我们应该进行测试。

为了测试 Ansible 模块，我们使用之前运行过`chmod`命令的脚本。该命令简单地运行你的模块，记录输出并返回给你。它还展示了 Ansible 如何解释模块的输出。我们将使用的命令如下所示：

```
ansible/hacking/test-module -m ./hostname

```

上一个命令的输出应如下所示：

```
* module boilerplate substitution not requested in module, line numbers will be unaltered
***********************************
RAW OUTPUT
hostname=admin01.int.example.com

***********************************
PARSED OUTPUT
{
    "hostname": "admin01.int.example.com"
}
```

忽略顶部的通知；它与使用 bash 构建的模块无关。你可以看到我们脚本发送的原始输出，完全符合我们的预期。测试脚本还会显示解析后的输出。在我们的例子中，我们使用的是简短的输出格式，我们可以看到 Ansible 正确地将其解释为模块通常接受的 JSON 格式。

让我们扩展模块以允许设置`hostname`。我们应该编写它，以便仅在需要时才进行更改，并让 Ansible 知道是否进行了更改。对于我们编写的小命令来说，这实际上是相当简单的。新的脚本应该类似于以下内容：

```
#!/bin/bash

set -e

# This is potentially dangerous
source ${1}

OLDHOSTNAME="$(hostname)"
CHANGED="False"

if [ ! -z "$hostname" -a "${hostname}x" != "${OLDHOSTNAME}x" ]; then
  hostname $hostname
  OLDHOSTNAME="$hostname"
  CHANGED="True"
fi

echo "hostname=${OLDHOSTNAME} changed=${CHANGED}"
exit 0
```

上一个脚本的工作原理如下：

1.  我们设置了 Bash 的错误退出模式，这样我们就不必处理`hostname`方法的错误。Bash 将在失败时自动退出并返回退出代码。这将通知 Ansible 出现了问题。

1.  我们加载参数文件。这个文件作为第一个参数从 Ansible 传递给脚本。它包含发送给我们模块的参数。因为我们加载了这个文件，它可以用来运行任意命令；然而，Ansible 已经可以做到这一点，所以这并不会造成太大的安全问题。

1.  我们收集旧的主机名并将默认值`CHANGED`设为`False`。这使我们能够查看模块是否需要执行任何更改。

1.  我们检查是否收到新的主机名，并确认该主机名是否与当前设置的不同。

1.  如果这两个测试都为真，我们尝试更改主机名，并将`CHANGED`设置为`True`。

1.  最后，我们输出结果并退出。结果包括当前的主机名以及是否进行了更改。

在 Unix 机器上更改主机名需要 root 权限。因此，在测试此脚本时，你需要确保以 root 用户身份运行它。我们使用`sudo`来测试此脚本，看看它是否有效。你将使用以下命令：

```
sudo ansible/hacking/test-module -m ./hostname -a 'hostname=test.example.com'

```

如果`test.example.com`不是当前机器的主机名，你应该看到以下输出：

```
* module boilerplate substitution not requested in module, line numbers will be unaltered
***********************************
RAW OUTPUT
hostname=test.example.com changed=True

***********************************
PARSED OUTPUT
{
    "changed": true,
    "hostname": "test.example.com"
}
```

如你所见，我们的输出被正确解析，模块声称已经对系统进行了更改。你可以通过`hostname`命令自己验证这一点。现在，再次使用相同的主机名运行该模块。你应该会看到如下输出：

```
* module boilerplate substitution not requested in module, line numbers will be unaltered
***********************************
RAW OUTPUT
hostname=test.example.com changed=False

***********************************
PARSED OUTPUT
{
    "changed": false,
    "hostname": "test.example.com"
}
```

再次，我们看到输出被正确解析了。然而，这一次，模块声称没有进行任何更改，这正是我们预期的结果。你也可以通过`hostname`命令来验证这一点。

# 使用自定义模块

现在我们已经为 Ansible 编写了第一个模块，我们应该在 playbook 中试一试。Ansible 会在多个位置查找模块——首先，它会查看`config`文件中`library`键指定的地方（`/etc/ansible/ansible.cfg`），然后它会查看命令行中使用`--module-path`参数指定的位置，接着它会查看与 playbook 同目录下的`library`目录，最后它会在`library`目录中查找任何可能设置的角色。

让我们创建一个使用新模块的 playbook，并将其放置在相同位置的`library`目录中，以便我们可以看到它的实际效果。以下是一个使用`hostname`模块的 playbook：

```
---
- name: Test the hostname file
  hosts: testmachine
  tasks:
    - name: Set the hostname
      hostname: hostname=testmachine.example.com
```

然后在与 playbook 文件相同的目录下创建一个名为`library`的目录。将`hostname`模块放在 library 目录中。你的目录结构应该如下所示：

![使用自定义模块](img/image00106.jpeg)

现在，当你运行 playbook 时，它会在`library`目录中找到`hostname`模块并执行它。你应该能看到类似这样的输出：

```
PLAY [Test the hostname file] ***************************************

GATHERING FACTS *****************************************************
ok: [ansibletest]

TASK: [Set the hostname] ********************************************
changed: [ansibletest]

PLAY RECAP **********************************************************
ansibletest                : ok=2    changed=1    unreachable=0    failed=0
```

再次运行时，结果应该从`changed`变为`ok`。恭喜！你现在已经创建并执行了你的第一个模块。这个模块现在非常简单，但你可以扩展它，支持`hostname`文件，或其他方法来在启动时配置主机名。

# 用 Python 编写模块

所有随 Ansible 分发的模块都是用 Python 编写的。因为 Ansible 本身也是用 Python 编写的，所以这些模块可以直接与 Ansible 集成。以下是你应该用 Python 编写模块的一些理由：

+   用 Python 编写的模块可以使用模板代码，这样可以减少所需代码量。

+   Python 模块可以提供文档供 Ansible 使用。

+   模块的参数会自动处理。

+   输出会自动转换为 JSON 格式。

+   Ansible 上游只接受使用 Python 并包含模板代码的插件。

你仍然可以在没有此集成的情况下构建 Python 模块，通过解析参数并自行输出 JSON。然而，考虑到你可以免费获得的所有功能，要为这种做法辩护是很困难的。

让我们构建一个 Python 模块，允许我们更改系统当前运行的初始化级别。我们有一个名为`pyutmp`的 Python 模块，它可以让我们解析`utmp`文件。不幸的是，由于 Ansible 模块必须包含在一个单独的文件中，除非我们知道它会被安装到远程系统上，否则我们无法使用它。因此，我们将使用`runlevel`命令并解析其输出。设置运行级别可以通过`init`命令来完成。

第一步是弄清楚该模块支持哪些参数和功能。为了简化起见，我们的模块只接受一个参数。我们将使用`runlevel`参数来获取用户想要切换的运行级别。为此，我们将按如下方式实例化`AnsibleModule`类并传入数据：

```
module = AnsibleModule(
  argument_spec = dict(
    runlevel=dict(default=None, type='str')
  )
)
```

现在，我们需要实现模块的实际内容。我们之前创建的模块对象为我们提供了一些快捷方式。接下来的步骤中我们将使用三种快捷方式。由于有太多方法无法在此文档中一一说明，你可以查看整个`AnsibleModule`类以及在`lib/ansible/module_common.py`中所有可用的帮助函数。

+   `run_command`：此方法用于启动外部命令并获取返回代码、`stdout`的输出以及`stderr`的输出。

+   `exit_json`：此方法用于在模块成功完成时将数据返回给 Ansible。

+   `fail_json`：此方法用于向 Ansible 报告失败，带有错误信息和返回代码。

以下代码实际上管理了系统的初始化级别，并附有注释来解释它的作用：

```
def main():     #1
  module = AnsibleModule(    #2
    argument_spec = dict(    #3
      runlevel=dict(default=None, type='str')     #4
    )     #5
  )     #6

  # Ansible helps us run commands     #7
  rc, out, err = module.run_command('/sbin/runlevel')     #8
  if rc != 0:     #9
    module.fail_json(msg="Could not determine current runlevel.", rc=rc, err=err)     #10

  # Get the runlevel, exit if its not what we expect     #11
  last_runlevel, cur_runlevel = out.split(' ', 1)     #12
  cur_runlevel = cur_runlevel.rstrip()     #13
  if len(cur_runlevel) > 1:     #14
    module.fail_json(msg="Got unexpected output from runlevel.", rc=rc)     #15

  # Do we need to change anything     #16
  if module.params['runlevel'] is None or module.params['runlevel'] == cur_runlevel:     #17
    module.exit_json(changed=False, runlevel=cur_runlevel)     #18

  # Check if we are root     #19
  uid = os.geteuid()     #20
  if uid != 0:     #21
    module.fail_json(msg="You need to be root to change the runlevel")     #22

  # Attempt to change the runlevel     #23
  rc, out, err = module.run_command('/sbin/init %s' % module.params['runlevel'])     #24
  if rc != 0:     #25
    module.fail_json(msg="Could not change runlevel.", rc=rc, err=err)     #26

  # Tell ansible the results     #27
  module.exit_json(changed=True, runlevel=cur_runlevel)     #28
```

最后需要在模板中添加一行代码，告诉 Ansible 需要动态地将集成代码添加到我们的模块中。这就是让我们能够使用`AnsibleModule`类并实现与 Ansible 紧密集成的魔法。模板代码需要放在文件的最底部，后面不能有任何代码。实现此功能的代码如下所示：

```
# include magic from lib/ansible/module_common.py
#<<INCLUDE_ANSIBLE_MODULE_COMMON>>
main()
So, finally, we have the code for our module built. Putting it all together, it should look like the following code:
#!/usr/bin/python     #1
# -*- coding: utf-8 -*-    #2

import os     #3

def main():     #4
  module = AnsibleModule(    #5
    argument_spec = dict(    #6
      runlevel=dict(default=None, type='str'),     #7
    ),     #8
  )     #9

  # Ansible helps us run commands     #10
  rc, out, err = module.run_command('/sbin/runlevel')     #11
  if rc != 0:     #12
    module.fail_json(msg="Could not determine current runlevel.", rc=rc, err=err)     #13

  # Get the runlevel, exit if its not what we expect     #14
  last_runlevel, cur_runlevel = out.split(' ', 1)     #15
  cur_runlevel = cur_runlevel.rstrip()     #16
  if len(cur_runlevel) > 1:     #17
    module.fail_json(msg="Got unexpected output from runlevel.", rc=rc)     #18

  # Do we need to change anything     #19
  if (module.params['runlevel'] is None or module.params['runlevel'] == cur_runlevel):     #20
    module.exit_json(changed=False, runlevel=cur_runlevel)     #21

  # Check if we are root     #22
  uid = os.geteuid()     #23
  if uid != 0:     #24
    module.fail_json(msg="You need to be root to change the runlevel")     #25

  # Attempt to change the runlevel     #26
  rc, out, err = module.run_command('/sbin/init %s' % module.params['runlevel'])     #27
  if rc != 0:     #28
    module.fail_json(msg="Could not change runlevel.", rc=rc, err=err)     #29

  # Tell ansible the results     #30
  module.exit_json(changed=True, runlevel=cur_runlevel)     #31

# include magic from lib/ansible/module_common.py     #32
#<<INCLUDE_ANSIBLE_MODULE_COMMON>>     #33
main()     #34
```

你可以使用`test-module`脚本以与测试 Bash 模块相同的方式测试这个模块。然而，你需要小心，因为如果你使用`sudo`运行它，可能会重启你的机器或者将初始化级别更改为你不希望的状态。这个模块可能通过在远程测试机器上使用 Ansible 本身来进行更好的测试。我们按照本章前面*编写 Bash 模块*部分中描述的相同过程进行操作。我们创建一个使用该模块的 playbook，然后将该模块放置在与 playbook 相同目录下的库目录中。以下是我们需要使用的 playbook：

```
---
- name: Test the new init module
  hosts: testmachine
  user: root
  tasks:
    - name: Set the init level to 5
      init: runlevel=5
```

现在你应该能够尝试在远程机器上运行它。第一次运行时，如果机器的运行级别尚未是 5 级，你应该看到它更改运行级别。然后你可以再次运行它，看到没有任何变化。你可能还想检查一下，确保模块在未以 root 身份运行时会正确失败。

# 外部库存

在第一章，*Ansible 入门*中，我们看到 Ansible 需要一个库存文件，这样它就能知道主机的位置以及如何访问它们。Ansible 还允许你指定一个脚本，通过它从其他来源获取库存。外部库存脚本可以使用任何你喜欢的语言编写，只要它输出有效的 JSON。

外部清单脚本必须接受 Ansible 的两个不同调用。如果使用`–list`调用，它必须返回所有可用组和主机的列表。此外，它也可以使用`--host`调用。在这种情况下，第二个参数将是主机名，脚本应该返回该主机的变量列表。所有输出都应以 JSON 格式返回，因此你应该使用自然支持它的语言。

让我们编写一个模块，接受一个包含所有机器的 CSV 文件，并将其作为清单呈现给 Ansible。如果你有一个**配置管理数据库**（**CMDB**），允许你将机器列表导出为 CSV，或者如果有人将其机器记录保存在电子表格中，这将非常有用。此外，它不需要 Python 以外的任何依赖，因为 Python 已经包含了一个 CSV 处理模块。这实际上只是将 CSV 文件解析成正确的数据结构，并以 JSON 数据结构的形式输出。以下是我们希望处理的 CSV 文件示例；你可能希望根据你环境中的机器自定义它：

```
Group,Host,Variables
test,example,ansible_ssh_user=root
test,localhost,connection=local
```

该文件需要转换为两种不同的 JSON 输出。当调用`--list`时，我们需要以如下格式输出整个内容：

```
{"test": ["example", "localhost"]}
```

当使用参数`--host example`调用时，它应返回以下内容：

```
{"ansible_ssh_user": "root"}
```

这是一个脚本，它打开一个名为`machines.csv`的文件，并在给定`--list`时生成组的字典：此外，当给定`--host`和主机名时，它解析该主机的变量并将其作为字典返回。脚本有详细注释，你可以看到它的工作过程。你可以手动运行脚本，使用`--list`和`--host`参数来确认其行为是否正确。

```
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import sys
import csv
import json

def getlist(csvfile):
  # Init local variables
  glist = dict()
  rowcount = 0

  # Iterate over all the rows
  for row in csvfile:
    # Throw away the header (Row 0)
    if rowcount != 0:
      # Get the values out of the row
      (group, host, variables) = row

      # If this is the first time we've
      # read this group create an empty
      # list for it
      if group not in glist:
        glist[group] = list()

      # Add the host to the list
      glist[group].append(host)

    # Count the rows we've processed
    rowcount += 1

  return glist

def gethost(csvfile, host):
  # Init local variables
  rowcount = 0

  # Iterate over all the rows
  for row in csvfile:
    # Throw away the header (Row 0)
    if rowcount != 0 and row[1] == host:
      # Get the values out of the row
      variables = dict()
      for kvpair in row[2].split():
        key, value = kvpair.split('=', 1)
        variables[key] = value

      return variables

    # Count the rows we've processed
    rowcount += 1

command = sys.argv[1]

#Open the CSV and start parsing it
with open('machines.csv', 'r') as infile:
  result = dict()
  csvfile = csv.reader(infile)

  if command == '--list':
    result = getlist(csvfile)
  elif command == '--host':
    result = gethost(csvfile, sys.argv[2])

  print json.dumps(result)
```

现在，你可以使用这个清单脚本在使用 Ansible 时提供清单。测试一切是否正常工作的一个快速方法是使用`ping`模块来测试与所有机器的连接。此命令不会测试主机是否在正确的组中；如果你想做这个，你可以使用相同的`ping`模块命令，但不仅仅是对所有主机运行它，你可以简单地使用你想要测试的组**。**如果你的清单文件是可执行的，Ansible 将运行它并使用输出。你还可以使用目录，Ansible 会包括其中的所有文件，如果它们是可执行的，将运行它们。

```
$ ansible -i csvinventory –list-hosts -m ping all

```

类似于你在第一章中使用`ping`模块时，*Ansible 入门*，你应该看到如下输出：

```
localhost | success >> {
  "changed": false,
  "ping": "pong"
}

example | success >> {
  "changed": false,
  "ping": "pong"
}
```

这表明你可以连接并在所有来自你清单的主机上使用 Ansible。你可以使用相同的`-i`参数与`ansible-playbook`一起运行你的剧本，并使用相同的清单。

# 扩展 Ansible

除了编写模块和外部清单脚本外，你还可以扩展 Ansible 本身的核心功能。这允许你通过 Python 向 Ansible 添加更多功能。通过为 Ansible 编写插件，你可以做到以下几点：

+   为了通过连接插件控制其他机器，可以添加新的方法

+   使用查找插件在循环或查找中使用来自 Ansible 外部的数据

+   使用过滤插件为变量或模板添加新的过滤器

+   使用回调插件包含在 Ansible 内部某些操作发生时运行的回调

要将额外的插件添加到你的 Ansible 项目中，我们需要在 `ansible.cfg` 文件中指定的插件目录下创建一个 Python 文件。或者，我们可以将包含插件的新目录添加到现有目录列表中。

### 注意

请不要删除任何现有的目录，因为删除这些目录会移除提供核心 Ansible 功能的插件，正如我们在本书前面提到的那些插件。

在为 Ansible 编写插件时，你应该尽量使插件灵活且可重用。这样，你可以将一些复杂性从剧本和模板中移到少数几个复杂的 Python 文件中。专注于插件的重用性还意味着你有可能将插件提交回 Ansible 项目，通过 GitHub pull 请求的方式。如果你将插件提交回 Ansible，那么每个人都能利用你的插件，而你也为 Ansible 的发展做出了贡献。关于如何为 Ansible 做贡献的更多信息，可以在 Ansible 源代码中的 `CONTRIBUTORS.md` 文件中找到。

## 连接插件

连接插件负责将文件传输到远程机器并执行模块。你无疑已经在本书前面的剧本中使用过 SSH、本地插件，可能还使用过 winrm 插件。

除了常规的 `__init__()` 方法外，连接插件还必须实现以下方法：

| 方法 | 目的 |
| --- | --- |
| `connect()` | 这个方法打开与我们正在管理的主机的连接 |
| `exec_command()` | 这个方法在被管理的主机上执行一个命令 |
| `put_file()` | 这个方法将文件复制到被管理的主机上 |
| `fetch_file()` | 这个方法从被管理的主机下载一个文件 |
| `close()` | 这个方法关闭与我们正在管理的主机的连接 |

## 查找插件

查找插件有两种使用方式：通过 `lookup()` 包含来自外部的数据，或者以 `with_` 风格遍历项目。你甚至可以将两者结合使用，像在 `with_fileglob` 查找插件中那样遍历外部数据。本书前面特别是在 第三章的 *循环* 部分展示了几个查找插件。

查找插件很容易编写，除了常规的`__init__()`方法外，它们只需要你实现一个`run()`方法。这个方法使用 Ansible `utils`包中的`listify_lookup_plugin_terms()`方法来收集传入的参数列表，并返回结果。作为示例，我们现在演示一个查找插件，从 JSON 编码文件中读取数据：

```
import json

class LookupModule(object):
    def __init__(self, basedir=None, **kwargs):
        pass

    def run(self, terms, inject=None, **kwargs):
        with open(terms, 'r') as f:
            json_obj = json.load(f)

        return json_obj
```

这可以用作查找插件来提取复杂数据，或者如果文件包含 JSON 列表，则可以使用`with_jsonfile`作为循环。将上述示例保存为`jsonfile.py`，并放在你其中一个查找插件目录中。你可以看到我们声明了一个新的类`LookupModule`；这是 Ansible 在 Python 文件中要查找的类名，因此你必须使用这个名称。然后，我们创建一个构造函数（命名为`__init__`），以便 Ansible 可以创建我们的类。最后，我们编写一个小方法，它只会打开一个 JSON 文件，解析它，并将结果返回给 Ansible。

我们应该注意到，这个示例其实是简化过的，它只会在当前工作目录中查找文件。以后可以扩展为在角色文件目录或其他地方查找，以便更好地符合其他 Ansible 模块设定的惯例。

然后，你可以在剧本中像这样使用这个查找插件：

```
- name: Print the JSON data we received
  debug:
    msg: "{{ lookup('json', 'file.json') }}"
```

## 过滤器插件

过滤器插件是 Ansible 用来处理变量并从模板生成文件的 Jinja2 模板引擎的扩展。这些扩展可以在剧本中使用，对变量进行数据处理，或者它们可以在模板内使用，处理数据后再包含到文件中。它们通过将复杂的处理逻辑移到 Python 文件中，从而简化了数据处理，避免了在模板或 Ansible 配置中增加复杂度。

过滤器插件与其他插件有些不同。要实现一个过滤器插件，首先你需要写一个简单的函数，它只需要接收所需的输入并返回结果。接着，你创建一个名为`FilterModule`的类，并在其中实现一个`filters`方法，该方法返回一个 Python 字典，其中键是过滤器名称，值是要调用的函数。

这是一个插件的示例实现，可以用来计算在任何组中避免脑裂情况所需的最小服务器数：在大多数系统中，这个数字是节点总数的 50%以上，再加 1。

```
def quorum(list_of_machines):

    n = len(list_of_machines)
    quorum = n / 2 + 1

    return quorum

class FilterModule(object):
    def filters(self):
        return {
            'quorum': quorum,
        }
```

简单来说，这个模块统计传入列表中有多少个项目，将其除以二，然后加一。所有操作都以整数数学计算，因此余数会被忽略，所有的操作都以整数进行，这符合我们的目的。

这个过滤器可以在剧本或模板中使用。例如，如果我们想配置一个 Elasticsearch 集群以确保有法定人数并避免脑裂问题，我们将使用以下代码行：

```
discovery.zen.minimum_master_nodes: {{ play_hosts|quorum }}
```

这将获取当前正在运行的主机列表（来自`play_hosts`变量），然后计算其中需要多少个才能获得法定人数。

## 回调插件

回调插件用于向外部系统提供有关 Ansible 中正在发生的操作的信息。如果在 Ansible 配置中指定的`callback_plugins`目录下找到回调插件，它们会自动激活。它们在自动化任务中运行剧本时尤其有用，因为它们可以通过标准输出之外的其他渠道提供反馈。回调插件有广泛的用途，具体如下：

+   在剧本结束时，发送包含更改统计信息的电子邮件

+   记录`syslog`中正在进行的更改的运行日志

+   在剧本任务失败时通知聊天频道

+   在系统配置发生变化时，更新 CMDB 以确保每个系统的配置视图准确。

+   当所有主机都失败导致剧本提前退出时，通知管理员

回调插件是最复杂的插件之一，因为它们可以钩入 Ansible 的大多数功能。然而，选项虽然很多，但并不意味着你需要实现所有的功能。你只需实现回调插件将使用的那些方法。以下是你可以实现的方法列表及其描述：

+   `def on_any(self, *args, **kwargs)`：在其他回调方法调用之前会调用此方法。由于不同回调方法的参数不同，它将参数展开为`args`和`kwargs`。此方法适用于日志记录，若用于其他用途会变得比较复杂。

+   `runner_on_failed(self, host, res, ignore_errors=False)`：任务失败后运行。`host`参数包含任务运行的主机，`res`包含来自剧本的任务数据以及返回的任何数据，`ignore_errors`包含一个布尔值，指定是否忽略剧本中指示的错误。

+   `runner_on_ok(self, host, res)`：在任务成功完成或异步任务轮询成功时运行。参数`host`包含任务运行的主机，`res`包含来自剧本的任务数据以及返回的任何数据。

+   `runner_on_skipped(self, host, item=None)`：在任务被跳过后运行。参数`host`包含如果任务没有被跳过的话将运行的主机，`item`参数包含当前正在迭代的循环项。

+   `runner_on_unreachable(self, host, res)`：当某个主机不可达时运行。`host`参数包含不可达的主机，`res`包含连接插件返回的错误信息。

+   `runner_on_no_hosts(self)`：当任务在没有任何主机的情况下启动时，这个回调会被调用。它没有任何变量。

+   `runner_on_async_poll(self, host, res, jid, clock)`：每当异步任务轮询状态时，这个方法会被调用。变量`host`包含正在被轮询的主机，`res`包含轮询的详细信息，`jid`包含任务 ID，`clock`包含任务失败之前剩余的时间。

+   `runner_on_async_ok(self, host, res, jid)`：当轮询完成且没有错误时，会运行此回调。`host` 参数包含正在轮询的主机，`res` 存储任务的结果，`jid` 包含作业 ID。

+   `runner_on_async_failed(self, host, res, jid)`：当轮询完成并出现错误时，会运行此回调。`host` 参数包含正在轮询的主机，`res` 存储任务的结果，`jid` 包含作业 ID。

+   `playbook_on_start(self)`：此回调在通过 `ansible-playbook` 启动 playbook 时执行。它不使用任何变量。

+   `playbook_on_notify(self, host, handler)`：每当一个 handler 被通知时，此回调函数会被触发。由于它在通知发生时运行，而不是在 handler 执行时运行，因此每个 handler 可能会多次触发。它有两个变量：`host` 存储通知任务的主机名，`handler` 存储被通知的 handler 名称。

+   `playbook_on_no_hosts_matched(self)`：如果一个 play 开始时没有匹配任何主机，则会运行此回调。它没有任何变量。

+   `playbook_on_no_hosts_remaining(self)`：当 play 中所有主机都有错误，且 play 无法继续时，会运行此回调。

+   `playbook_on_task_start(self, name, is_conditional)`：每个任务开始前都会运行此回调，即使该任务将被跳过。`name` 变量设置为任务的名称，`is_conditional` 设置为 when 条件的结果——如果任务将执行，`True`；如果不会执行，`False`。

+   `playbook_on_setup(self)`：此回调会在 setup 模块在所有主机上执行之前运行。无论包含多少主机，它只会运行一次。它不包含任何变量。

+   `playbook_on_play_start(self, name)`：每个 play 开始时都会运行此回调。`name` 变量包含即将开始的 play 的名称。

+   `playbook_on_stats(self, stats)`：此回调会在 playbook 结束时、即将打印统计信息之前运行。`stats` 变量包含 playbook 的详细信息。

# 总结

阅读完本章后，你应该能够使用 Bash 或任何你熟悉的语言构建模块。你应该能够安装从互联网获取或自己编写的模块。我们还介绍了如何通过使用 Python 中的模板代码更高效地编写模块，并编写了一个库存脚本，允许你从外部来源提取库存。最后，我们介绍了如何通过编写连接、查找、过滤和回调插件来向 Ansible 本身添加新功能。

我们已经尽力涵盖了你了解 Ansible 时所需的大部分内容，但我们不可能涵盖所有内容。如果你想继续学习 Ansible，可以访问官方 Ansible 文档：[`docs.ansible.com/`](http://docs.ansible.com/)。

Ansible 项目目前正在进行重写，最终将作为 2.0 版本发布。本书应与该版本及未来的其他版本保持兼容，但将会有一些新功能未在此涵盖。在 Ansible 2.0 版本中，您可以期待以下功能，这些功能可能会在未来发生变化（因为该版本尚未发布）：

+   在剧本中的故障恢复能力

+   允许您并行运行大量任务

+   兼容 Python 3

+   更容易的调试，因为错误信息将包含行号
