# 第八章 调试与错误处理

就像软件代码一样，测试基础设施代码是一项至关重要的任务。理想情况下，生产环境中不应该存在未经测试的代码，尤其是当你有严格的客户 SLA 需要满足时，这对于基础设施同样适用。本章中，我们将讨论语法检查、在不将代码应用到机器上的情况下进行测试（无操作模式）以及功能测试，这些都是 Ansible 的核心内容，并触发你想要在远程主机上执行的各种任务。建议将这些集成到你的**持续集成**（**CI**）系统中，以便更好地测试你的 playbook。我们将关注以下几点：

+   语法检查

+   使用和不使用 diff 的检查模式

+   功能测试

作为功能测试的一部分，我们将关注以下内容：

+   对系统最终状态的断言

+   使用标签进行测试

+   Serverspec（一个不同的工具，但可以与 Ansible 很好地配合使用）

+   使用`--syntax-check`选项

每当你运行一个 playbook 时，Ansible 首先会检查 playbook 文件的语法。如果遇到错误，Ansible 会提示语法错误并停止执行，除非你修复该错误。此语法检查仅在你运行`ansible-playbook`命令时进行。当编写大型 playbook 或包含任务文件时，可能很难修复所有错误，这可能会浪费更多时间。为了解决这种情况，Ansible 提供了一种方法，可以在你继续编写 playbook 时检查 YAML 语法。对于这个示例，我们需要创建文件`playbooks/setup_apache.yaml`，其内容如下：

```
    - hosts: localhost 
      tasks: 
      - name: Install Apache 
        yum: 
          name: httpd 
          state: present 
      - name: Enable Apache 
      service: 
          name: httpd 
          state: running 
          enabled: True 

```

现在我们有了示例文件，我们需要使用`--syntax-check`参数来运行它，因此你将通过以下方式调用 Ansible：

```
ansible-playbook playbooks/setup_apache.yaml --syntax-check

```

`ansible-playbook`命令检查了`setup_apache.yml` playbook 的 YAML 语法，并显示该 playbook 的语法是正确的。让我们来看看由于 playbook 中无效语法导致的错误：

```
    ERROR! Syntax Error while loading YAML. 
    The error appears to have been in '~/08_code/playbooks/setup_apache.yaml': line 9, column 4, but may 
    be elsewhere in the file depending on the exact syntax problem. 

    The offending line appears to be: 

      - name: Enable Apache 
      service: 
      ^ here 

```

错误显示在`Enable Apache`任务中存在缩进错误。Ansible 还会给出出错的行号、列号和文件名（即使这并不能保证错误的准确位置）。这应该绝对是你在 Ansible 的持续集成中应该运行的基础测试之一。

# 检查模式

检查模式（也称为**干运行**或**无操作模式**）将在没有执行任何操作的情况下运行你的剧本，即它不会对远程主机应用任何更改；相反，它只会显示任务运行时将引入的更改。是否启用检查模式取决于每个任务。你可能会对以下命令感兴趣。所有这些模块必须在`/usr/lib/python2.7/site-packages/ansible/modules`或你 Ansible 模块文件夹所在的路径中运行（不同的路径可能根据你使用的操作系统及 Ansible 的安装方式而有所不同）。

要统计安装中可用的模块数量，你可以执行以下命令：

```
find . -type f | grep '.py$' | grep -v '__init__' | wc -l

```

使用 Ansible 2.1.1 时，这个命令的结果是`569`，因为 Ansible 有这么多模块。

如果你想查看有多少模块支持检查模式，你可以运行：

```
grep -r 'supports_check_mode=True' | awk -F: '{print $1}' | sort | uniq | wc -l

```

使用 Ansible 2.1.1 时，这个命令的结果是`242`。

你可能还会发现以下命令在列出所有支持检查模式的模块时很有用：

```
grep -r 'supports_check_mode=True' | awk -F: '{print $1}' | sort | uniq

```

这有助于你测试剧本的行为，并在将其运行在生产服务器之前检查是否可能出现任何失败。你只需将`--check`选项传递给`ansible-playbook`命令，就可以在检查模式下运行剧本。让我们看看检查模式如何与`setup_apache.yml`剧本一起工作，如下所示：

```
PLAY [localhost] ************************************************* 

    TASK [setup] ***************************************************** 
    ok: [localhost] 

    TASK [Install Apache] ******************************************** 
    ok: [localhost] 

    TASK [Enable Apache] ********************************************* 
    changed: [localhost] 

    PLAY RECAP ******************************************************* 
    localhost         : ok=3    changed=1    unreachable=0    failed=0 

```

在之前的运行中，Ansible 没有在目标主机上做出更改，而是突出显示了实际运行时将发生的所有更改。从前一次的运行中，你可以发现`httpd`服务已经安装在目标主机上，因此该任务的 Ansible 退出消息为“ok”。

```
    TASK [Install Apache] ******************************************** 
    ok: [localhost]

```

然而，第二个任务发现目标主机上的`httpd`服务没有运行：

```
    TASK [Enable Apache] ********************************************* 
    changed: [localhost]

```

当你再次运行前面的剧本时，如果没有启用检查模式，Ansible 将确保服务状态为运行中。

# 使用`--diff`指示文件之间的差异

在检查模式下，你可以使用`--diff`选项来显示将应用于文件的更改。为了能够看到`--diff`选项的使用，我们需要将`playbooks/setup_apache.yaml`剧本修改为如下所示：

```
    - hosts: localhost 
      tasks: 
      - name: Ensure Apache is installed 
        yum: 
          name: httpd 
          state: present 
      - name: Ensure Apache in enabled 
        service: 
          name: httpd 
          state: running 
          enabled: True 
      - name: Ensure Apache userdirs are properly configured 
        template: 
          src: '../templates/userdir.conf' 
          dest: '/etc/httpd/conf.d/userdir.conf' 

```

如你所见，我们添加了一个任务，它将确保`/etc/httpd/conf.d/userdir.conf`文件的某个特定状态。

我们还需要创建一个模板文件，并将其放在`templates/userdir.conf`中，内容如下：

```
    # UserDir: The name of the directory that is appended onto a user's home 
    # directory if a ~user request is received. 
    # The path to the end user account 'public_html' directory must be 
    # accessible to the webserver userid.  This usually means that ~userid 
    # must have permissions of 711, ~userid/public_html must have permissions 
    # of 755, and documents contained therein must be world-readable. 
    # Otherwise, the client will only receive a "403 Forbidden" message. 
    # 
    <IfModule mod_userdir.c> 
        # 
        # UserDir is disabled by default since it can confirm the presence 
        # of a username on the system (depending on home directory 
        # permissions). 
        # 
        UserDir enabled 

        # 
        # To enable requests to /~user/ to serve the user's public_html 
        # directory, remove the "UserDir disabled" line above, and uncomment 
        # the following line instead: 
        # 
        #UserDir public_html 
    </IfModule> 

    # 
    # Control access to UserDir directories.  The following is an example 
    # for a site where these directories are restricted to read-only. 
    # 
    <Directory "/home/*/public_html"> 
        AllowOverride FileInfo AuthConfig Limit Indexes 
        Options MultiViews Indexes SymLinksIfOwnerMatch IncludesNoExec 
        Require method GET POST OPTIONS 
    </Directory> 

```

在这个模板中，我们只更改了`UserDir enabled`这一行，默认情况下它是`UserDir disabled`。

### 注意

`--diff`选项无法与文件模块一起使用；你只能使用模板模块。

我们现在可以通过以下命令测试这个结果：

```
ansible-playbook playbooks/setup_apache.yaml --diff --check

```

如你所见，我们使用了`--check`参数，这将确保这是一个干运行。我们将收到以下输出：

```
    PLAY [localhost] ************************************************* 

    TASK [setup] ***************************************************** 
    ok: [localhost] 

    TASK [Ensure Apache is installed] ******************************** 
    ok: [localhost] 

    TASK [Ensure Apache in enabled] ********************************** 
    changed: [localhost] 

    TASK [Ensure Apache userdirs are properly configured] ************ 
    changed: [localhost] 
    --- before: /etc/httpd/conf.d/userdir.conf 
    +++ after: dynamically generated 
    @@ -14,7 +14,7 @@ 
        # of a username on the system (depending on home directory 
        # permissions). 
        # 
    -    UserDir disabled 
    +    UserDir enabled 

        # 
        # To enable requests to /~user/ to serve the user's public_html 
    @@ -33,4 +33,3 @@ 
        Options MultiViews Indexes SymLinksIfOwnerMatch IncludesNoExec 
        Require method GET POST OPTIONS 
    </Directory> 
    - 

    PLAY RECAP ******************************************************* 
    localhost         : ok=4    changed=2    unreachable=0    failed=0

```

如我们所见，Ansible 会将远程主机的当前文件与源文件进行比较；以`+`开头的行表示文件中添加了一行内容，而`-`表示删除了一行。

### 注意

你还可以在没有`--check`选项的情况下使用`--diff`，这将允许 Ansible 执行指定的更改并显示两个文件之间的差异。

将`--diff`和`--check`模式一起使用是一个测试步骤，可能作为你 CI 测试的一部分，用来验证在运行过程中有多少步骤发生了变化。另一个可以将这两个功能一起使用的场景是部署过程的一部分，检查在你运行 Ansible 时到底会发生什么变化。

也有一些情况——虽然不该发生，但有时会发生——你可能很长时间没在某台机器上运行 playbook，担心再次运行会破坏某些东西。将这些选项一起使用应能帮助你了解是你过度担心，还是确实存在实际风险。

# Ansible 中的功能测试

Wikipedia 表示功能测试是一个**质量保证（QA）过程**，是一种黑盒测试类型，其测试用例基于被测试软件组件的规格。**通过给功能输入数据并检查输出结果来进行测试**；通常不考虑内部程序结构。功能测试在基础设施中与代码同样重要。

从基础设施的角度来看，在功能测试方面，我们测试的是 Ansible 运行在实际机器上的输出。Ansible 提供了多种方式来执行 playbook 的功能测试；让我们来看一些最常用的方法。

# 使用 assert 进行功能测试

检查模式只会在你想检查某个任务是否会对主机做出更改时起作用。如果你想检查模块的输出是否符合预期，这种模式将无效。例如，假设你编写了一个模块来检查端口是否开启。为了测试这个，你可能需要检查模块的输出，看看它是否与预期结果匹配。为了执行这样的测试，Ansible 提供了一种直接比较模块输出与期望输出的方式。

让我们通过创建文件`playbooks/assert_ls.yaml`并加入以下内容来看这个如何运作：

```
    - hosts: localhost 
      tasks: 
      - name: List files in /tmp 
        command: ls /tmp 
        register: list_files 
      - name: Check if file testfile.txt exists 
        assert: 
          that: 
          - "'testfile.txt' in list_files.stdout_lines" 

```

在上述 playbook 中，我们在目标主机上运行`ls`命令，并将该命令的输出结果注册到`list_files`变量中。此外，我们要求 Ansible 检查`ls`命令的输出是否符合预期结果。我们使用`assert`模块来完成此操作，该模块通过一些条件检查来验证任务的`stdout`值是否符合用户的预期输出。让我们运行上述 playbook 来查看 Ansible 使用该命令返回的输出：

```
ansible-playbook playbooks/assert_ls.yaml

```

由于我们没有这个文件，我们将收到以下输出：

```
    PLAY [localhost] ************************************************* 

    TASK [setup] ***************************************************** 
    ok: [localhost] 

    TASK [List files in /tmp] **************************************** 
    changed: [localhost] 

    TASK [Check if file testfile.txt exists] ************************* 
    fatal: [localhost]: FAILED! => {"assertion": "'testfile.txt' in list_files.stdout_lines", "changed":     false, "evaluated_to": false, "failed": true} 

    NO MORE HOSTS LEFT *********************************************** 
        to retry, use: --limit @playbooks/assert_ls.retry 

    PLAY RECAP ******************************************************* 
    localhost         : ok=2    changed=1    unreachable=0    failed=1

```

如果在创建预期文件后重新运行剧本，它将不会失败，因此结果将是：

```
    PLAY [localhost] ************************************************* 

    TASK [setup] ***************************************************** 
    ok: [localhost] 

    TASK [List files in /tmp] **************************************** 
    changed: [localhost] 

    TASK [Check if file testfile.txt exists] ************************* 
    ok: [localhost] 

    PLAY RECAP ******************************************************* 
    localhost         : ok=3    changed=1    unreachable=0    failed=0

```

这次，任务通过了，并且显示了`testfile.txt`在`list_files`变量中存在的消息。类似地，你可以使用`and`和`or`操作符在变量中或多个变量中匹配多个字符串。断言功能非常强大，曾经在项目中编写过单元测试或集成测试的用户会对这个功能非常高兴！

# 使用标签进行测试

标签是测试一堆任务而不运行整个剧本的好方法。我们可以使用标签对节点执行实际测试，以验证用户意图中的状态，即剧本。我们可以把这当作另一种在实际机器上运行 Ansible 的集成测试方式。标签方法可以在实际运行 Ansible 的机器上进行测试，并且它也可以在部署期间主要用于测试最终系统的状态。在本节中，我们将首先了解如何一般性地使用标签，它的功能如何帮助我们，不仅仅是用于测试，甚至在其他方面也有帮助，最后是用于测试目的。

要在剧本中添加标签，使用`tags`参数，后跟一个或多个标签名称，标签名称之间用逗号分隔。让我们在`playbooks/tags_example.yaml`中创建一个简单的剧本，查看标签如何工作，内容如下：

```
    - hosts: localhost 
      tasks: 
      - name: Ensure the file /tmp/ok exists 
        file: 
          name: /tmp/ok 
          state: touch 
        tags: 
        - file_present 
      - name: Ensure the file /tmp/ok does not exists 
        file: 
          name: /tmp/ok 
          state: absent 
        tags: 
        - file_absent 

```

如果我们现在运行剧本，文件将被创建和销毁。我们可以使用以下命令查看它的运行情况：

```
ansible-playbook playbooks/tags_example.yaml

```

它将给我们这个输出：

```
    PLAY [localhost] ************************************************* 

    TASK [setup] ***************************************************** 
    ok: [localhost] 

    TASK [Ensure the file /tmp/ok exists] **************************** 
    changed: [localhost] 

    TASK [Ensure the file /tmp/ok does not exists] ******************* 
    changed: [localhost] 

    PLAY RECAP ******************************************************* 
    localhost         : ok=3    changed=2    unreachable=0    failed=0

```

由于这不是一个幂等的剧本，如果我们反复运行它，我们将始终看到相同的结果，因为剧本每次都会创建和删除该文件。

现在，你可以简单地传递`file_present`标签或`file_absent`标签，只执行其中一个动作，像下面的示例一样：

```
ansible-playbook playbooks/tags_example.yaml -t file_present

```

由于`-t file_present`部分，只有带有`file_present`标签的任务会被执行，实际上这是输出结果：

```
    PLAY [localhost] ************************************************* 

    TASK [setup] ***************************************************** 
    ok: [localhost] 

    TASK [Ensure the file /tmp/ok exists] **************************** 
    changed: [localhost] 

    PLAY RECAP ******************************************************* 
    localhost         : ok=2    changed=1    unreachable=0    failed=0

```

你还可以使用标签在远程主机上执行一组任务，就像从负载均衡器中移除服务器并重新加入负载均衡器一样。

你还可以将`--check`选项与标签一起使用。通过这样做，你可以在不实际在主机上运行任务的情况下测试任务。这使得你可以直接测试一堆独立的任务，而不需要将任务复制到一个临时的剧本中并从那里运行。

# --skip-tags

Ansible 还提供了一种在剧本中跳过某些标签的方法。如果你有一个包含多个标签（如 10 个标签）的长剧本，并且希望执行其中所有标签，除了一个，那么传递九个标签给 Ansible 并不是一个好主意。如果你忘记传递某个标签，并且`ansible-playbook`命令失败，那情况会变得更加复杂。为了解决这种情况，Ansible 提供了一种跳过某些标签的方法，而不是传递多个标签。其功能非常直接，可以通过以下方式触发：

```
ansible-playbook playbooks/tags_example.yaml --skip-tags file_present

```

输出将类似于：

```
    PLAY [localhost] ************************************************* 

    TASK [setup] ***************************************************** 
    ok: [localhost] 

    TASK [Ensure the file /tmp/ok does not exists] ******************* 
    ok: [localhost] 

    PLAY RECAP ******************************************************* 
    localhost         : ok=2    changed=1    unreachable=0    failed=0

```

如你所见，所有任务都已执行，除了带有 `file_present` 标签的任务。

# 管理异常

有许多情况下，由于某些原因，你希望在一个或多个任务失败的情况下，继续执行你的 playbook 和角色。一个典型的例子可能是，你想检查软件是否已经安装。让我们看看以下安装 Java 的例子。在`roles/java/tasks/main.ymal`文件中，我们将写入以下代码：

```
    - name: Verify if the current version of Java is installed 
      command: rpm -q jdk1.8.0_91-1.8.0_91-fcs 
      register: java 
      ignore_errors: True 
      changed_when: java|failed 

    - name: Ensure that JavaSE is download 
      uri: 
        url: 'http://download.oracle.com/otn-pub/java/jdk/8u91-b14/jdk-8u91-linux-x64.rpm' 
        method: GET 
        HEADER_Cookie: 'gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie' 
        dest: /tmp 
        creates: /tmp/jdk-8u91-linux-x64.rpm 
      when: java|failed 

    - name: Ensure JavaSE is installed 
      dnf: 
        name: /tmp/jdk-8u91-linux-x64.rpm 
        state: present 
      when: java|failed 

    - name: Set alternatives for java 
      alternatives: 
        path: /usr/java/jdk1.8.0_91/jre/bin/java 
        name: java 
        link: /usr/bin/java 
      when: java|failed 

    - name: Set alternatives for javac 
      alternatives: 
        path: /usr/java/jdk1.8.0_91/bin/javac 
        name: javac 
        link: /usr/bin/javac 
      when: java|failed 

    - name: Set alternatives for javaws 
      alternatives: 
        path: /usr/java/jdk1.8.0_91/bin/javaws 
        name: javaws 
        link: /usr/bin/javaws 
      when: java|failed 

```

在继续执行其他需要的部分之前，我想花点时间讲解这个角色任务列表的各个部分，因为这里有很多新的内容：

```
    - name: Verify if the current version of Java is installed 
      command: rpm -q jdk1.8.0_91-1.8.0_91-fcs 
      register: java 
      ignore_errors: True 
      changed_when: java|failed 

```

在这个任务中，我们执行一个`rpm`命令，该命令可能有两种不同的输出：

+   失败

+   返回 JDK 包的完整名称

由于我们只想检查包是否存在，然后继续前进，我们注册输出（*第三*行）并忽略可能的失败（*第四*行）：

```
    - name: Ensure that JavaSE is download 
      uri: 
        url: 'http://download.oracle.com/otn-pub/java/jdk/8u91-b14/jdk-8u91-linux-x64.rpm' 
        method: GET 
        HEADER_Cookie: 'gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie' 
        dest: /tmp 
        creates: /tmp/jdk-8u91-linux-x64.rpm 
      when: java|failed 

```

在这一部分中，我们使用 `uri` 模块，它允许我们使用 HTTP 请求访问远程 URI。这个模块非常好，因为它允许你使用所有 HTTP 方法，并自定义 HTTP 头部。这样使得该模块非常灵活。由于在最后一行我们有 `when: java|failed`，这将仅在 Java 未安装时执行：

```
    - name: Ensure JavaSE is installed 
      dnf: 
        name: /tmp/jdk-8u91-linux-x64.rpm 
        state: present 
      when: java|failed 

```

在这里我们使用 `dnf` 安装 Java 包。由于在最后一行我们有 `when: java|failed`，这将仅在 Java 未安装时执行：

```
    - name: Set alternatives for java 
      alternatives: 
        path: /usr/java/jdk1.8.0_91/jre/bin/java 
        name: java 
        link: /usr/bin/java 
      when: java|failed 

    - name: Set alternatives for javac 
      alternatives: 
        path: /usr/java/jdk1.8.0_91/bin/javac 
        name: javac 
        link: /usr/bin/javac 
      when: java|failed 

    - name: Set alternatives for javaws 
      alternatives: 
        path: /usr/java/jdk1.8.0_91/bin/javaws 
        name: javaws 
        link: /usr/bin/javaws 
      when: java|failed 

```

在这里，我们将设置新的 alternatives，以防我们正在安装 Java。`alternatives`是一个 Ansible 模块，它允许我们管理 Linux `alternatives` 程序的配置。这个程序常用于管理在默认情况下如果安装了多个版本时，哪个版本的程序应该被默认运行。

在我们创建角色之后，我们需要一个包含主机机器的 `hosts` 文件，在我的例子中：

```
    j01.fale.io 

```

以及一个应用该角色的 playbook，放在 `playbooks/hosts/j01.fale.io.yaml` 中，内容如下：

```
    - hosts: j01.fale.io 
      user: root 
      roles: 
      - java 

```

现在我们可以用以下命令执行它：

```
ansible-playbook playbooks/hosts/j01.fale.io.yaml

```

我们将得到以下结果：

```
    PLAY [j01.fale.io] *********************************************** 

    TASK [setup] ***************************************************** 
    ok: [j01.fale.io] 

    TASK [java : Verify if the current version of Java is installed] * 
    fatal: [j01.fale.io]: FAILED! => {"changed": true, "cmd": ["rpm", "-q", "jdk1.8.0_91-1.8.0_91-fcs"],         "delta": "0:00:00.009788", "end": "2016-09-27 11:04:56.185618", "failed": true, "rc": 1, "start":         "2016-    09-27 11:04:56.175830", "stderr": ``, "stdout": "package jdk1.8.0_91-1.8.0_91-fcs is not         installed", "stdout_lines": ["package jdk1.8.0_91-1.8.0_91-fcs is not installed"], "warnings":             ["Consider using yum, dnf or zypper module rather than running rpm"]} 
      ...ignoring 

    TASK [java : Ensure that JavaSE is download] ********************* 
    changed: [j01.fale.io] 

    TASK [java : Ensure JavaSE is installed] ************************* 
    changed: [j01.fale.io] 

    TASK [java : Set alternatives for java] ************************** 
    ok: [j01.fale.io] 

    TASK [java : Set alternatives for javac] ************************* 
    ok: [j01.fale.io] 

    TASK [java : Set alternatives for javaws] ************************ 
    ok: [j01.fale.io] 

    PLAY RECAP ******************************************************* 
    j01.fale.io       : ok=7    changed=2    unreachable=0    failed=0

```

如你所见，安装检查失败，因为 Java 没有安装在机器上，因此所有其他任务按预期执行。

# 触发失败

有时你可能希望直接触发失败。这可能由于多种原因发生，尽管这样做有一定的弊端，因为触发失败时，playbook 会被强行中断，如果不小心，可能会让你的机器处于不一致的状态。一个我看到非常有效的情况是，当你运行一个非幂等的 playbook（例如构建新版应用程序）时，你需要设置一个变量（例如：要部署的版本/分支）。在这种情况下，你可以在开始执行操作之前检查预期的变量是否正确配置，以确保后续一切按预期进行。

让我们将以下代码放入 `playbooks/maven_build.yaml` 文件中：

```
    - hosts: j01.fale.io 
      tasks: 
      - name: Ensure the tag variable is properly set 
        fail: 'The version needs to be defined. To do so, please add: --extra-vars                                 "version=$[TAG/BRANCH]"' 
        when: version is not defined 
      - name: Get last Project version 
        git: 
          repo: https://github.com/org/project.git 
          dest: "/tmp" 
          version: '{{ version }}' 
      - name: Maven clean install 
        shell: "cd /tmp/project && mvn clean install" 

```

如您所见，我们希望用户在脚本中添加 `--extra-vars "version=$[TAG/BRANCH]"` 来调用命令。我们本可以默认使用某个分支，但这样做风险太大，因为用户可能会失去注意力，忘记自己添加正确的分支名称，这将导致编译（和部署）错误版本的应用程序。`fail` 模块还允许我们指定一条消息，该消息将显示给用户。

### 注意

我认为 `fail` 任务在手动运行的 playbook 中更为有用，因为当 playbook 被自动运行时，处理异常往往比直接失败更好。

# 概述

在本章中，我们已经了解了如何使用多种技术调试 Ansible playbook。接着我们讨论了失败管理，最后我们看到了如何故意触发失败。

在下一章中，我们将讨论多层环境以及部署方法。
