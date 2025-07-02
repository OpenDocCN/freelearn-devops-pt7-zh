# 7

# 创建和使用插件

到目前为止，模块已经是我们使用 Ansible 过程中非常显而易见且关键的一部分。它们用于执行明确的任务，可以通过一次性命令（使用临时命令）或作为更大 playbook 的一部分来使用。插件对 Ansible 同样重要，我们在所有的测试中都使用了插件，只是没有意识到！虽然模块总是用于在 Ansible 中创建某种任务，但插件的使用方式取决于它们的使用场景。插件有许多不同的类型；我们将在本章中介绍它们，并向你展示它们的用途。但是，作为测试人员，你是否意识到当 Ansible 通过 SSH 连接到远程服务器时，是由连接插件提供功能的？这展示了插件在其中扮演的重要角色。

在本章中，我们将为你提供一个关于插件的深入介绍，并向你展示如何探索 Ansible 附带的各种插件。接着，我们将通过演示如何创建自己的插件并在 Ansible 项目中使用它们，进一步扩展这一内容，就像我们在上一章中创建自定义模块的方式一样。希望通过这种方式帮助你理解开源软件（如 Ansible）所提供的无限可能性。

本章将涵盖以下主题：

+   发现插件类型

+   查找包含的插件

+   创建自定义插件

# 技术要求

本章假设你已经按照*第一章*《Ansible 入门》中的详细说明，设置好了控制主机，并且你正在使用最新版本的 Ansible。本章中的示例已在 `ansible-core` 版本 2.15 中进行了测试。本章还假设你至少有一台其他主机进行测试；理想情况下，这台主机应基于 Linux。

尽管我们将在本章中给出具体的主机名示例，但你可以自由地用自己的主机名和/或 IP 地址进行替换，如何操作的详细信息将在相关部分提供。本章涵盖的插件开发工作假设你的计算机上已经安装了 Python 3 开发环境，并且你正在运行 Linux、FreeBSD 或 macOS。需要额外 Python 模块时，我们会提供其安装文档。本章构建模块文档的任务在 Python 3.10 或更高版本中有一些非常具体的要求，因此假设你如果愿意尝试，可以安装合适的 Python 环境。

本章的代码包可在 [`github.com/PacktPublishing/Practical-Ansible-Second-Edition/tree/main/Chapter%207`](https://github.com/PacktPublishing/Practical-Ansible-Second-Edition/tree/main/Chapter%207) 获取。

# 发现插件类型

Ansible 的代码一直被设计为模块化的——事实上，这是其核心优势之一。无论是通过使用模块执行任务，还是通过插件（如我们稍后将看到的），Ansible 的模块化设计使得它能够像本书中所展示的那样具有多功能性和强大能力。与模块一样，Ansible 插件都是用 Python 编写的，并且预期按照某种明确定义的格式来获取和返回数据（稍后我们将详细讨论）。Ansible 的插件在功能上通常是隐形的，因为你很少在命令或 playbook 中按名称调用它们，但它们负责 Ansible 提供的一些最重要的功能，包括 SSH 连接能力、解析清单文件（无论是 INI 格式、YAML 格式，还是其他格式）的能力，以及在数据上运行 `jinja2` 过滤器的能力。

一如既往，让我们在继续之前验证你的测试机上是否安装了合适版本的 Ansible：

```
$ ansible-doc --version
ansible-doc [core 2.15.0]
  config file = None
  configured module search path = ['/Users/danieloh/Library/Python/3.11/bin/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /Users/danieloh/Library/Python/3.11/lib/python/site-packages/ansible
  ansible collection location = /Users/danieloh/Library/Python/3.11/bin/collections:/usr/share/ansible/collections
  executable location = /Users/danieloh/Library/Python/3.11/bin/ansible-doc
  python version = 3.11.3 (main, Apr  7 2023, 20:13:31) [Clang 14.0.0 (clang-1400.0.29.202)] (/opt/homebrew/opt/python@3.11/bin/python3.11)
  jinja version = 3.1.2
  libyaml = True
```

在文档化插件的工作量和文档化模块的工作量一样多，因此你会很高兴知道可以在 [`docs.ansible.com/ansible/latest/plugins/plugins.xhtml`](https://docs.ansible.com/ansible/latest/plugins/plugins.xhtml) 找到插件索引。

你也可以像之前一样使用 `ansible-doc` 命令，只不过你需要在模块名称后添加 `-t`。插件总是根据其功能被放置在适当的类别中，因为不同类别之间的功能差异很大。如果你没有在 `ansible-doc` 中指定 `-t` 选项，你将会指定 `ansible-doc -t` 模块，这将返回可用模块的列表。

在撰写本文时，可以在 Ansible 中找到以下插件类型：

+   `become`：负责启用 Ansible 获取超级用户访问权限（例如，通过 `sudo`）

+   `cache`：负责缓存从后台系统检索的事实，以提高自动化性能

+   `callback`：允许你在响应事件时添加新行为——例如，在 Ansible playbook 执行的输出中更改数据打印的格式

+   `cliconf`：为各种网络设备的命令行接口提供抽象，为 Ansible 提供一个标准接口来操作

+   `connection`：提供从 Ansible 到远程系统的连接（例如，通过 SSH、WinRM、Docker 等）

+   `httpapi`：告诉 Ansible 如何与远程系统的 API 交互（例如，用于 Fortinet 防火墙）

+   `inventory`：为 Ansible 提供解析各种静态和动态清单格式的能力

+   `lookup`：允许 Ansible 从外部源查找数据（例如，通过读取纯文本文件）

+   `netconf`：为 Ansible 提供抽象，使其能够与支持 NETCONF 的网络设备进行交互

+   `shell`：为 Ansible 提供在不同系统上使用各种 shell 的能力（例如，在 Windows 上使用 `powershell`，在 Linux 上使用 `sh`）

+   `strategy`：为 Ansible 提供不同执行策略的插件（例如，我们在*第四章*中看到的调试策略，*剧本* *与角色*）。  

+   `vars`：为 Ansible 提供了从特定来源获取变量的能力，例如我们在*第三章*中探索的 `host_vars` 和 `group_vars` 目录，*定义* *你的库存*。

我们将把探索 Ansible 网站上的插件文档作为练习留给你完成。然而，如果你想使用 `ansible-doc` 工具探索各种插件，你需要运行以下命令：

1.  要使用 `ansible-doc` 命令列出给定类别中所有可用的插件，你可以运行以下命令：

    ```
    $ ansible-doc -t connection -l
    ```

这将返回一个关于连接插件的文本索引，类似于我们查看模块文档时看到的索引输出。索引输出的前几行如下所示：

￼

图 7.1 – Ansible 连接插件文档

1.  然后，你可以探索特定插件的文档。例如，如果我们想了解 `paramiko_ssh` 插件，我们可以运行以下命令：

    ```
    $ ansible-doc -t connection paramiko_ssh
    ```

你会发现，插件的文档采用了与我们在*第五章*中看到的模块文档相似的格式，*创建和* *使用* *模块*：

￼

图 7.2 – Ansible paramiko_ssh 连接插件文档

感谢所有为记录 Ansible 各个领域所付出的辛勤努力，你可以轻松了解 Ansible 附带的插件以及如何使用它们。到目前为止，我们已经看到插件的文档与模块的文档同样完整。在本章的下一部分，我们将深入探讨如何找到与 Ansible 发行版一起提供的插件代码。

# 查找包含的插件

正如我们在前一节中讨论的那样，插件在 Ansible 中不像模块那样明显，但我们在迄今为止执行的每个 Ansible 命令中，实际上都在幕后使用了它们！让我们在前一节中查看插件文档的基础上，进一步探讨如何找到插件的源代码。这将为我们构建自己的简单插件奠定基础。

如果你在 Linux 系统上通过包管理器（即通过 RPM 或 DEB 包）安装了 Ansible，那么插件的位置将取决于你的操作系统。例如，在我测试的 CentOS 8 系统上，我通过官方 RPM 包安装了 Ansible，我可以在以下位置看到安装的插件：

```
$ ls /usr/lib/python3.11/site-packages/ansible/plugins/
action cliconf httpapi inventory lookup terminal
become connection __init__.py loader.py netconf test
cache doc_fragments __init__.pyc loader.pyc shell vars
callback filter __init__.pyo loader.pyo strategy
```

请注意，插件被分隔到按类别命名的子目录中。如果我们想查找我们在前一节中回顾的 `paramiko_ssh` 插件，我们可以在 `connection/` 子目录中查找：

```
$ ls -l /usr/lib/python3.11/site-packages/ansible/plugins/connection/paramiko_ssh.py
-rw-r—r—1 root root 23544 Mar 5 05:39 /usr/lib/python3.11/site-packages/ansible/plugins/connection/paramiko_ssh.py
```

然而，通常情况下，我不推荐你编辑或更改从包安装的文件，因为在升级包时你很容易会覆盖它们。由于本章的目标之一是编写一个简单的自定义插件，我们来看看如何在官方 Ansible 源代码中找到插件：

1.  从 GitHub 克隆官方 Ansible 仓库，如我们之前所做的，并切换到你的克隆目录：

    ```
    $ git clone https://github.com/ansible/ansible.git
    lib/ansible/plugins/:

    ```

    connection directory:

    ```
    $ ls -al connection/
    ```

    ```

    ```

这个目录的具体内容将取决于你克隆的 Ansible 源代码版本。在写本文时，它看起来如下，每个插件都有一个 Python 文件（类似于我们在*第五章*中看到的，每个模块有一个 Python 文件，*创建和* *使用* *模块*）：

```
$ ls -al connection/
total 176
drwxr-xr-x 2 root root 109 May 15 17:24 .
drwxr-xr-x 19 root root 297 May 15 17:24 ..
-rw-r--r-- 1 root root 16411 May 15 17:24 __init__.py
-rw-r--r-- 1 root root 6855 May 15 17:24 local.py
-rw-r--r-- 1 root root 23525 May 15 17:24 paramiko_ssh.py
-rw-r--r-- 1 root root 32839 May 15 17:24 psrp.py
-rw-r--r-- 1 root root 55367 May 15 17:24 ssh.py
-rw-r--r-- 1 root root 31277 May 15 17:24 winrm.py
```

1.  你可以查看每个插件的内容，了解它们如何工作，这也是开源软件的一大魅力所在：

    ```
    $ less connection/paramiko_ssh.py
    ```

本文件开始部分的示例如下所示，给你一个大概的输出样式，帮助你理解如果这个命令正确执行时你应看到的输出：

```
# (c) 2012, Michael DeHaan <michael.dehaan@gmail.com>
# (c) 2017 Ansible Project
# GNU General Public License v3.0+ (see COPYING or https://www.gnu.org/licenses/gpl-3.0.txt)
from __future__ import (absolute_import, division, print_function)
__metaclass__ = type
DOCUMENTATION = """
    author: Ansible Core Team
    connection: paramiko
    short_description: Run tasks via python ssh (paramiko)
    description:
        - Use the python ssh implementation (Paramiko) to connect to targets
        - The paramiko transport is provided because many distributions, in particular EL6 and before do not support ControlPersist
          in their SSH implementations.
DOCUMENTATION block, which is very similar to what we saw when we were working with the module source code. If you explore the source code of each plugin, you will find that the structure bears some similarity to the module code structure. However, rather than simply taking this statement at face value, in the next section, we’ll build our very own custom plugin to learn, through a practical example, how they are put together.
Creating custom plugins
In this section, we will take you through a practical guide to creating a plugin. The example will be, by necessity, simple. However, hopefully, it will serve you well in guiding you in the principles and best practices of plugin development and give you a solid foundation to build more complex plugins. We will even show you how to integrate these with your playbooks and, when you’re ready, submit them to the official Ansible project for inclusion.
As we noted when we built a module, Ansible is written in Python, and its plugins are no exception. As a result, you will need to write your plugin in Python; so, to get started on developing a plugin, you will need to make sure you have Python and a few essential tools installed. If you already have Ansible running on your development machine, you probably have the required packages installed.
Let’s get started with creating a plugin. Although there are many similarities between coding modules and plugins, there are also fundamental differences. Each of the different types of plugins that Ansible can work with is coded slightly differently and has different recommendations. Sadly, we don’t have space to go through each one in this book, but you can find out more about the requirements for each plugin type by reading the official Ansible documentation at [`docs.ansible.com/ansible/latest/dev_guide/developing_plugins.xhtml`](https://docs.ansible.com/ansible/latest/dev_guide/developing_plugins.xhtml).
For our simple example, we’ll create a filter plugin that replaces a given string with another. If you refer to the preceding documentation link, filter plugins are perhaps some of the easiest ones to code because there isn’t a stringent requirement on the documentation in the same way that there is for modules. However, if we were to create a `lookup` plugin, we would be expected to create the same `DOCUMENTATION`, `EXAMPLES`, and `RETURN` documentation sections that we created in *Chapter 5*, *Creating and Consuming* *Modules*. We would also need to test and build our web documentation in the same way.
We have already covered this, so it doesn’t serve to repeat the entirety of this process in this chapter. Instead, we will focus on creating a filter plugin. In contrast with other Ansible plugins and modules, you can have several filters defined in a single Python plugin file. Filters are, by nature, quite compact to code. They are also numerous, so having one file per filter doesn’t scale well. However, if you want to code other types of plugins (such as `lookup` plugins), you *will* need to create one Python file per plugin.
Let’s start creating our simple filter plugin. As we are only creating one, it will live in its own single Python file. You could propose a modification to one of the Ansible core filter Python files if you want to submit your code back to the Ansible project; but for now, we’ll leave that as a project for you to complete yourself. Our filter file will be called `custom_filter.py` and it will live in a directory called `filter_plugins`, which must be created in the same directory as your playbook.
For clarity, your final directory structure should look as follows:

```

.

├── hosts

├── lookup_plugins

│ └── firstchar.py

├── myplugin2.yml

└── testdoc.txt

```

 Perform the following steps to create and test your plugin code:

1.  Start your plugin file with a header so that people will know who wrote the plugin and what license it is released under. Naturally, you should update both the copyright and license fields with values appropriate to your plugin, but the following text is given as an example for you to get started with:

    ```

    # (c) 2020, James Freeman <james.freeman@example.com>

    # GNU 通用公共许可证 v3.0+（请参见 COPYING 或 https://www.gnu.org/licenses/gpl-3.0.txt）

    ```

     2.  Next, we’ll add a very simple Python function – yours can be as complex as you want it to be, but for ours, we will simply use the Python `.replace` function to replace one string with another inside a `string` variable. The following example looks for instances of `Puppet` and replaces them with `Ansible`:

    ```

    def improve_automation(a):

    return a.replace("Puppet", "Ansible")

    ```

     3.  Next, we need to create an object of the `FilterModule` class, which is how Ansible will know that this Python file contains a filter. Within this object, we can create a `filters` definition and return the value of our previously defined filter function to Ansible:

    ```

    class FilterModule(object):

    '''improve_automation 过滤器'''

    def filters(self):

    return {'improve_automation': improve_automation}

    ```

     4.  As you can see, this code is all incredibly simple and we’re able to use built-in Python functions, such as `replace`, to manipulate the strings. There isn’t a specific test harness for plugins in Ansible, so we will test out our plugin code by writing a simple playbook that will implement it. The following playbook code defines a simple string that includes the word `Puppet` in it and prints this to the console using the `debug` module, applying our newly defined filter to the string:

    ```

    ---

    - name: 演示自定义过滤器的播放

    hosts: frontends

    gather_facts: false

    vars:

    statement: "Puppet 是一个优秀的自动化工具！"

    tasks:

    - name: 做一个声明

    debug:

    msg: "{{ statement | improve_automation }}"

    ```

Now, before we attempt to run this, let’s recap what the directory structure should look like. Just as we were able to utilize the custom module that we created in *Chapter 5*, *Creating and Consuming* *Modules*, by creating a `library/` subdirectory to house our module, we can also create a `filter_plugins/` subdirectory for our plugin. Your directory tree structure, when you have finished coding the various file details in the preceding code block, should look something like this:

```

.

├── filter_plugins

│ ├── custom_filter.py

├── hosts

├── myplugin.yml

```

 Now, let’s run our little test playbook and see what output we get. If all goes well, it should look something like the following:

```

$ ansible-playbook -i hosts myplugin.yml

PLAY [演示自定义过滤器的播放] ***********************************

TASK [做一个声明] ********************************************************

ok: [frt01.example.com] => {

"msg": "Ansible 是一个优秀的自动化工具！"

}

PLAY RECAP *********************************************************************

frt01.example.com : ok=1 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0

```

 As you can see, our new filter plugin replaced the `Puppet` string in our variable’s contents and replaced it with the `Ansible` string. This, of course, is just a silly test and not one you are likely to contribute back to the Ansible project. However, it shows how, in just six lines of code and with a modicum of Python knowledge, we have created a filter plugin to manipulate a string. You could come up with something far more complex and useful, I’m sure!
Other plugin types require more effort than this; although we won’t go through the process of creating a filter plugin here, you’ll find coding a filter plugin more akin to coding a module, as you need to do the following:

*   Include the `DOCUMENTATION`, `EXAMPLES`, and `RETURN` sections with the appropriate documentation
*   Ensure you have incorporated appropriate and sufficient error handling in the plugin
*   Test it thoroughly, including both the failure and success cases

As an example of this, we’ll repeat the preceding process but create a `lookup` plugin instead. This plugin will be based heavily on a simplified version of the file `lookup` plugin. However, we want to adapt our version so that it only returns the first character of a file. You could adapt this example to perhaps read the header from a file, or you could add arguments to the plugin so that you can extract a substring using character indexes. We will leave this enhancement activity as an exercise for you to carry out yourself.
Let’s get started! Our new lookup plugin will be called `firstchar`, and as `lookup` plugins have a one-to-one mapping with their Python files, the plugin file will be called `firstchar.py`. (In fact, Ansible will use this filename as the name of the plugin – you won’t find a reference to it in the code anywhere!). If you intend to test this from a playbook, as executed previously, you should create this in a directory called `lookup_plugins/`:

1.  Start by adding a header to the plugin file, as before, so that the maintainer and copyright details are clear. We are borrowing a large chunk of the original `lookup` plugin code from `file.py` for our example, so we must include the relevant credit:

    ```

    # (c) 2023, Daniel Oh <daniel.oh@example.com>

    # (c) 2020, James Freeman <james.freeman@example.com>

    # (c) 2012, Daniel Hokka Zakrisson <daniel@hozac.com>

    # (c) 2017 Ansible Project

    # GNU 通用公共许可证 v3.0+（请参见 COPYING 或 https://www.gnu.org/licenses/gpl-3.0.txt）

    ```

     2.  Next, add the Python 3 headers – these are an absolute requirement if you intend to submit your plugin via a **Pull Request** (**PR**) to the Ansible project:

    ```

    from __future__ import (absolute_import, division, print_function)

    __metaclass__ = type

    ```

     3.  Next, add a `DOCUMENTATION` block to your plugin so that other users can understand how to interact with it:

    ```

    DOCUMENTATION = """

    lookup: firstchar

    author: James Freeman <james.freeman@example.com>

    version_added: "2.15"

    short_description: 读取文件内容的第一个字符

    description:

    - 这个查找返回 Ansible 控制器文件系统中文件内容的第一个字符。

    options:

    _terms:

    description: 要读取的文件路径(s)

    required: True

    notes:

    - 如果在变量上下文中读取，则文件可以被解释为 YAML 格式，只要内容符合解析器的有效性要求。

    - 这个查找不支持 'globing'，请改用 fileglob 查找。

    """

    ```

     4.  Add the relevant `EXAMPLES` blocks to show how to use your plugin, just as we did with our modules:

    ```

    EXAMPLES = """

    - debug: msg="foo.txt 文件的第一个字符是 {{lookup('firstchar', '/etc/foo.txt') }}"

    """

    ```

     5.  Also, make sure you document the `RETURN` values from your plugin:

    ```

    RETURN = """

    _raw:

    description:

    - 文件内容的第一个字符

    """

    ```

     6.  With the documentation complete, we can now start working on our Python code. We will start by importing all the Python modules we need to make our module work. We’ll also set up the `display` object, which is used for verbose output and debugging. This should be used in place of the `print` statements in your plugin code if you need to display the `debug` output:

    ```

    from ansible.errors import AnsibleError, AnsibleParserError

    from ansible.plugins.lookup import LookupBase

    from ansible.utils.display import Display

    display = Display()

    ```

     7.  We will now create an object of the `LookupModule` class. Define a default function within this called `run` (this is expected for the Ansible `lookup` plugin framework) and initialize an empty array for our return data:

    ```

    class LookupModule(LookupBase):

    def run(self, terms, variables=None, **kwargs):

    ret = []

    ```

     8.  With this in place, we will start a loop to iterate over each of the terms (which, in our simple plugin, will be the filenames passed to the plugin). Although we will only test this on simple use cases, the way that lookup plugins can be used means that they need to support the lists of `terms` to operate on. Within this loop, we display valuable debugging information and, most importantly, define an object with the details of each of the files we will open, called `lookupfile`:

    ```

    for term in terms:

    display.debug("文件查找术语: %s" % term)

    lookupfile = self.find_file_in_search_path(variables, 'files', term)

    display.vvvv(u"使用 %s 作为文件进行文件查找" % lookupfile)

    ```

     9.  Now, we will read in the file contents. This could be as simple as using one line of Python code, but we know from our work on modules in *Chapter 5*, *Creating and Consuming* *Modules*, that we should not take it for granted that we will be passed a file we can read. As a result, we will put the statement to read our file contents into a `try` block and implement exception handling to ensure that the behavior of the plugin is sensible, even in error cases, and that easy-to-understand error messages are passed back to the user, rather than to Python tracebacks:

    ```

    try:

    if lookupfile:

    contents, show_data = self._loader._get_file_contents(lookupfile)

    ret.append(contents.rstrip()[0])

    else:

    raise AnsibleParserError()

    except AnsibleParserError:

    raise AnsibleError("无法在查找中定位文件: %s" % term)

    ```

Notice that here, we append the first character of the file contents (denoted by the `[0]` index) to our empty array. We also remove any training spaces using `rstrip`.

1.  Finally, we must return the character we gathered from the file to Ansible with a `return` statement:

    ```

    返回 ret

    ```

     2.  Once again, we can create a simple test playbook to test out our newly created plugin:

    ```

    ---

    - name: 播放以演示我们的自定义查找插件

    hosts: frontends

    gather_facts: false

    tasks:

    - name: 创建语句

    debug:

    msg: "{{ lookup('firstchar', 'testdoc.txt')}}"

    ```

Again, we are using the debug module to print output to the console and referencing our `lookup` plugin to obtain the output.

1.  Create the text file referenced in the previous code block, called `testdoc.txt`. This can contain anything you like – mine contains the following simple text:

    ```

    Hello

    ```

     2.  Now, when we run our new playbook, we should see an output similar to the following:

    ```

    $ ansible-playbook -i hosts myplugin2.yml

    PLAY [播放以演示我们的自定义查找插件] ****************************

    TASK [创建语句] ********************************************************

    ok: [frt01.example.com] => {

    "msg": "H"

    }

    PLAY RECAP *********************************************************************

    frt01.example.com : ok=1 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0

    ```

If all goes well, your playbook should return the first character of the text file you created. Naturally, there is a lot we could do to enhance this code, but this serves as a nice, simple example to get you started.
With this foundation in place, you should now have a reasonable idea of how to get started with writing your plugins for Ansible. The next logical step for us is to look in greater depth at how we can test our newly written plugins. We will do this in the next section.
Learning to integrate custom plugins with Ansible source code
So far, we have only tested our plugin in a standalone manner. This is all well and good, but what if you wanted to add it either to your own fork of the Ansible source code – or, better yet, submit it back to the Ansible project for inclusion with a PR? Fortunately, this process is very similar to the one we covered in *Chapter 5*, *Creating and Consuming* *Modules*, only with slightly different folder structures.
Next, you will need to copy your plugin code into one of the appropriate plugin directories:

*   For example, our example filter would be copied to the following directory in the source code you just cloned:

    ```

    查找插件将进入查找插件的目录，使用如下命令：

    ```
    $ cp ~/firstchar.py ./lib/ansible/plugins/lookup/
    ```

    ```

With your code copied into place, you need to test the documentation (that is, whether your plugin includes it) as before. You can build the `webdocs` documentation in the same way as we did in *Chapter 5*, *Creating and Consuming* *Modules*, so we will not recap this here. However, as a refresher, we can quickly check whether the documentation renders correctly using the `ansible-doc` command, as follows:

```

$ . hacking/env-setup

运行 egg_info

创建 lib/ansible.egg-info

写入依赖关系到 lib/ansible.egg-info/requires.txt

写入 lib/ansible.egg-info/PKG-INFO

写入顶级名称到 lib/ansible.egg-info/top_level.txt

写入依赖链接到 lib/ansible.egg-info/dependency_links.txt

写入清单文件 'lib/ansible.egg-info/SOURCES.txt'

读取清单文件 'lib/ansible.egg-info/SOURCES.txt'

读取清单模板 'MANIFEST.in'

warning: 未找到匹配 'SYMLINK_CACHE.json' 的文件

写入清单文件 'lib/ansible.egg-info/SOURCES.txt'

设置 Ansible 以便从检出目录运行...

PATH=/home/james/ansible/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin

PYTHONPATH=/home/james/ansible/lib

MANPATH=/home/james/ansible/docs/man:/usr/local/share/man:/usr/share/man

请记住，您可能希望使用 -i 指定您的主机文件

完成！

$ ansible-doc -t lookup firstchar

> FIRSTCHAR (/home/james/ansible/lib/ansible/plugins/lookup/firstchar.py)

这个查找返回文件内容的第一个字符

Ansible 控制节点文件系统上的文件。

* 该模块由 Ansible 社区维护

OPTIONS (= 是必需的):

= _terms

要读取的文件路径

```

 As you have seen so far, there is a great deal of overlap between plugin development and module development in Ansible. It is especially important to pay attention to error handling with exceptions to produce good quality, easy-to-understand error messages and to adhere to and uphold Ansible’s high documentation standards. One additional item that we have not covered here is the plugin’s output. All plugins must return strings in Unicode; this ensures that they can run through the `jinja2` filters correctly. Further guidance can be found in the official Ansible documentation at [`docs.ansible.com/ansible/latest/dev_guide/developing_plugins.xhtml`](https://docs.ansible.com/ansible/latest/dev_guide/developing_plugins.xhtml).
Armed with this knowledge, you should now be well placed to begin your plugin development work, and even submit your code back to the community, if you desire. We’ll offer a brief recap of this in the next section.
Sharing plugins with the community
You may wish to submit your new plugin to the Ansible project, just as we considered for our custom modules in *Chapter 5*, *Creating and Consuming* *Modules*. The process for doing this with plugins is almost identical to what you do for modules, which this section will recap.
Note
Using the following process will submit a real request to the Ansible project on GitHub to merge the code you submit with their code. Do *not* follow this process unless you genuinely have a new module that is ready to be submitted to the Ansible code base.
To submit your plugin as a PR of the Ansible repository, you need to fork the `devel` branch of the official Ansible repository. To do this, log in to your GitHub account on your web browser (or create an account if you don’t already have one), and then navigate to [`github.com/ansible/ansible.git`](https://github.com/ansible/ansible.git). Click on **Fork** at the top-right corner of the page:
￼
Figure 7.3 – Ansible GitHub repository
Once you have forked the repository to your account, we will walk you through the commands you need to run to add your module code to it and then create the required PRs to merge your new module with the upstream Ansible project:

1.  Clone the `devel` branch that you just forked to your local machine. Use a command similar to the following, but be sure to replace the URL with one that matches your own GitHub account:

    ```

    $ git clone https://github.complugins/ 目录。以下代码块中使用的复制命令只是一个示例，目的是给你提供一个大致的操作思路——实际上，你应选择适合你插件的子目录，因为它不一定适用于查找类别。一旦你添加了 Python 文件，执行 `git add` 命令让 Git 知道这个新文件，然后使用有意义的提交信息进行提交。这里展示了一些示例命令：

    ```
    $ cd ansible
    $ cp ~/ansible-development/plugindev/firstchar.py ./lib/ansible/plugins/lookup
    $ git add lib/ansible/plugins/lookup/firstchar.py
    $ git commit -m 'Added tested version of firstchar.py for pull request creation'
    ```

    ```

     2.  Now, be sure to push the code to your forked repository using the following command:

    ```

    $ git push

    ```

     3.  Return to GitHub in your web browser and navigate to the **Pull requests** page, as shown in the following screenshot. Click on the **New pull** **request** button:

￼
Figure 7.4 – Ansible pull request
Follow the PR creation process, as guided by the GitHub website. Once you have successfully submitted your PR, you should be able to navigate to the list of PRs on the official Ansible source code repository and find yours there. An example of the PR list is shown in the following screenshot for your reference:
￼
Figure 7.5 – Ansible pull request list
As discussed previously, don’t be alarmed if it takes a long time for your PR to be reviewed – this is simply due to how many PRs there are to review and process. You can always use your plugin code locally by adding it to a local `*_plugins/` directory, as we demonstrated earlier, so that the processing speed of your PR doesn’t hinder your work with Ansible. Further details of where to place your plugin code when working locally can be found at [`docs.ansible.com/ansible/latest/dev_guide/developing_locally.xhtml`](https://docs.ansible.com/ansible/latest/dev_guide/developing_locally.xhtml).
That completes our look at creating plugins, including two working examples. Hopefully, you have found this journey informative and valuable and it has enhanced your ability to work with Ansible and extend its functionality where required.
Summary
Ansible plugins are a core part of Ansible’s functionality and as we discovered in this chapter, we have been working with them throughout this book without even realizing it! Ansible’s modular design makes it easy to extend and add functionality, regardless of whether you are working with modules or the various types of plugins that are currently supported. Whether it’s to add a new filter for string processing or a new way of looking up data (or perhaps even a new connection mechanism to new technology), Ansible plugins provide a complete framework that can extend Ansible far beyond its already extensive capabilities.
In this chapter, we learned about the various types of plugins that are supported by Ansible, before exploring them in greater detail and looking at we you can obtain documentation and information on the existing ones. We then completed two practical examples to create two different types of plugins for Ansible while looking at the best practices for plugin development and how this overlaps with module development. We finished off by recapping how to submit our new plugin code as a PR back to the Ansible project.
In the next chapter, we will explore the best practices that you should adhere to when writing your Ansible playbooks to ensure that you produce manageable, high-quality automation code.
Questions
Answer the following questions to test your knowledge of this chapter:

1.  Which of the following `ansible-doc` commands can you use to list the names of all the cache plugins?
    1.  `ansible-doc -a` `cache -l`
    2.  `ansible-doc` `cache -l`
    3.  `ansible-doc -``a cache`
    4.  `ansible-doc -t` `cache -l`
    5.  `ansible-doc cache`
2.  Which class do you need to add to your `lookup` plugin’s code to include the bulk of the plugin code, including `run()`, the `items` loop, `try`, and `except`?
    1.  `LookupModule`
    2.  `RunModule`
    3.  `StartModule`
    4.  `InitModule`
    5.  `LoadModule`
3.  True or false: To create custom plugins with complex operations rather than printing simple hello world text using Python, you need to install Python with the relevant dependencies on your OS.
    1.  True
    2.  False

Further reading
You can find all of the plugins on Ansible by accessing the Ansible repository directly at [`github.com/ansible/ansible/tree/devel/lib/ansible/plugins`](https://github.com/ansible/ansible/tree/devel/lib/ansible/plugins).

```
