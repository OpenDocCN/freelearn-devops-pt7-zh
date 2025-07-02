# 第七章：创建自定义模块

本章将重点介绍如何编写和测试自定义模块。我们已经讨论了模块是如何工作的以及如何在任务中使用它们。简单回顾一下，Ansible 中的模块是一段代码，每次运行 Ansible 任务时，它都会被传输并在远程主机上执行（如果你使用了 `local_action`，它也可以在本地执行）。

根据我的经验，每当需要将某些功能暴露为一等任务时，我看到过自定义模块的编写。虽然没有模块也能实现相同的功能，但这通常需要通过一系列现有模块的任务来完成目标，而且通常还需要命令和 shell 模块。例如，假设你想通过**预启动执行环境**（**PXE**）来配置一个服务器。如果没有自定义模块，你可能会使用一些 shell 或命令任务来完成同样的事情。然而，使用自定义模块，你只需将所需参数传递给它，业务逻辑将嵌入到自定义模块中以执行 PXE 启动。这使得你能够编写更易读的 playbook，并且代码的重用性更高，因为你只需要创建一次模块，就可以在角色和 playbook 中随处使用它。

传递给模块的参数，只要它们是键值格式，就会与模块一起转发到一个单独的文件中。Ansible 期望在模块的输出中至少有两个变量（即模块执行结果），无论是成功还是失败，还需要包含一个给用户的消息，这两个变量必须采用 JSON 格式。如果你遵循这个简单的规则，你就可以根据需要进行定制！

本章将涵盖以下主题：

+   Python 模块

+   Bash 模块

+   Ruby 模块

+   测试模块

当你选择一种特定的技术或工具时，通常会从它所提供的功能开始。你慢慢理解构建该工具背后的理念，以及它能帮助你解决什么问题。然而，只有当你深入理解它的工作原理时，你才会真正感到舒适并掌控全局。在某个阶段，为了充分利用工具的所有功能，你将不得不以适应自己需求的方式和手段来定制它。随着时间的推移，能够轻松插入新功能的工具会留下来，而不能做到这一点的工具则会消失在市场上。Ansible 也有类似的故事。Ansible playbook 中的所有任务都是某种类型的模块，它自带数百个模块。你几乎可以找到满足你所有需求的模块。然而，总是会有一些例外，这时扩展它的能力就显得尤为重要。

Chef 提供了**轻量级资源和提供者**（**LWRPs**）来执行此活动，而 Ansible 允许你使用自定义模块扩展其功能。然而，显著的区别在于，你可以用任何你选择的语言编写模块（只要你有该语言的解释器），而在 Chef 中，模块必须用 Ruby 编写。Ansible 开发者推荐对任何复杂模块使用 Python，因为它支持开箱即用的参数解析；几乎所有***nix**系统都默认安装 Python，而 Ansible 本身就是用 Python 编写的。为了完整起见，本章还将介绍如何用其他语言编写模块。

若要使你的自定义模块对 Ansible 可用，你可以执行以下操作之一：

+   在环境变量`ANSIBLE_LIBRARY`中指定自定义模块的路径

+   使用`--module-path`命令行选项

+   将模块放入 Ansible 顶级目录中的`library`目录

有了这些背景信息，接下来我们来看一些代码！

# 使用 Python 模块

Ansible 旨在允许用户用任何语言编写模块。然而，使用 Python 编写模块有其独特的优势。你可以利用 Ansible 的库来简化代码，这是其他语言模块无法享有的优势。借助 Ansible 库，解析用户参数、处理错误和返回所需值变得更加容易。我们将通过两个自定义 Python 模块的例子来展示模块的工作方式，一个使用了 Ansible 库，另一个没有使用。请确保在创建模块之前，按前一节提到的方式组织好目录结构。第一个例子创建了一个名为`check_user`的模块。为此，我们需要在 Ansible 顶级目录中的`library`文件夹中创建`check_user`文件，内容如下：

```
    #!/usr/bin/env python 

    import pwd 
    import sys 
    import shlex 
    import json 

    def main(): 
        # Parsing argument file 
        args = {} 
        args_file = sys.argv[1] 
        args_data = file(args_file).read() 
        arguments = shlex.split(args_data) 
        for arg in arguments: 
            if '=' in arg: 
                (key, value) = arg.split('=') 
                args[key] = value 
        user = args['user'] 

        # Check if user exists 
        try: 
            pwd.getpwnam(user) 
            success = True 
            ret_msg = 'User %s exists' % user 
        except KeyError: 
            success = False 
            ret_msg = 'User %s does not exists' % user 

        # Error handling and JSON return 
        if success: 
            print json.dumps({ 
                'msg': ret_msg 
            }) 
            sys.exit(0) 
        else: 
            print json.dumps({ 
                'failed': True, 
                'msg': ret_msg 
            }) 
            sys.exit(1) 
    main() 

```

前面的自定义模块`check_user`将检查主机上是否存在某个用户。该模块期望 Ansible 传入一个用户参数。让我们分解前面的模块，看看它是如何工作的。我们首先声明**解释器**（Python），并导入解析参数所需的库：

```
    #!/usr/bin/env python 

    import pwd 
    import sys 
    import shlex 
    import json 

```

使用`sys`库，我们然后解析由 Ansible 在文件中传递的参数。参数的格式为`param1=value1 param2=value2`，其中`param1`和`param2`是参数，`value1`和`value2`是参数的值。有多种方法可以分割参数并创建字典，我们选择了一种简单的方式来执行操作。我们首先通过空白字符分割参数创建参数列表，然后通过`=`字符分割参数并将其赋值给 Python 字典中的键和值。例如，如果您有一个字符串如`user=foo gid=1000`，那么您首先创建一个列表，看起来像`["user=foo", "gid=1000"]`，然后循环遍历此列表以创建一个字典。此字典将看起来像`{"user": "foo", "gid": 1000}`。这是通过以下行执行的：

```
    def main(): 
        # Parsing argument file 
        args = {} 
        args_file = sys.argv[1] 
        args_data = file(args_file).read() 
        arguments = shlex.split(args_data) 
        for arg in arguments: 
            if '=' in arg: 
                (key, value) = arg.split('=') 
                args[key] = value 
        user = args['user'] 

```

### 注意

我们基于空白字符分隔参数，因为这是核心 Ansible 模块遵循的标准。您可以使用任何分隔符代替空格，但我们建议保持统一性。

一旦我们有了用户参数，我们就检查该用户是否存在于主机上，如下所示：

```
    # Check if user exists 
    try: 
        pwd.getpwnam(user) 
        success = True 
        ret_msg = 'User %s exists' % user 
    except KeyError: 
        success = False 
        ret_msg = 'User %s does not exists' % user 

```

我们使用`pwd`库来检查用户的`passwd`文件。为简单起见，我们使用两个变量：一个用于存储成功或失败消息，另一个用于存储用户的消息。最后，我们使用在`try-catch`块中创建的变量来检查模块是否成功或失败，正如您在这个片段中所看到的：

```
    # Error handling and JSON return 
    if success: 
        print json.dumps({ 
            'msg': ret_msg 
        }) 
        sys.exit(0) 
    else: 
        print json.dumps({ 
            'failed': True, 
            'msg': ret_msg 
        }) 
        sys.exit(1) 

```

如果模块成功，则将以退出码 0 退出执行[`exit(0)`]；否则，将以非零代码退出。Ansible 将查找失败变量，如果设置为`True`，则将退出，除非您显式要求 Ansible 使用`ignore_errors`参数忽略错误。您可以像使用 Ansible 的任何其他核心模块一样使用自定义模块。要测试自定义模块，我们将需要一个 playbook，所以让我们创建文件`playbooks/check_user.yaml`，内容如下：

```
    - hosts: localhost 
      vars: 
        user_ok: root 
        user_ko: this_user_does_not_exists 
      tasks: 
      - name: 'Check if user {{ user_ok }} exists' 
        check_user: 
          user: '{{ user_ok }}' 
      - name: 'Check if user {{ user_ko }} exists' 
        check_user: 
          user: '{{ user_ko }}' 

```

正如您所看到的，我们像任何其他核心模块一样使用了`check_user`模块。Ansible 将通过将模块和参数复制到远程主机上执行此模块，并存储在一个单独的文件中。让我们看看以下 playbook 是如何运行的：

```
ansible-playbook playbooks/check_user.yaml

```

我们应该收到以下输出：

```
PLAY [localhost] *************************************************
TASK [setup] *****************************************************
ok: [localhost] 
TASK [Check if user root exists] *********************************
ok: [localhost] 
TASK [Check if user this_user_does_not_exists exists] ************
fatal: [localhost]: FAILED! => {"changed": false, "failed": true, "msg": "User this_user_does_not_exists does not exists"} 
NO MORE HOSTS LEFT ***********************************************
 to retry, use: --limit @playbooks/check_user.retry 
PLAY RECAP *******************************************************
localhost         : ok=2    changed=0    unreachable=0    failed=1

```

正如预期的那样，由于我们有`root`用户，但没有`this_user_does_not_exists`，它通过了第一个检查，但在第二个检查时失败了。

Ansible 还提供了一个 Python 库来解析用户参数并处理错误和返回。现在是时候看看 Ansible Python 库如何帮助您使代码更短、更快且更不容易出错了。为此，让我们创建一个名为`library/check_user_py2.py`的文件，内容如下：

```
    #!/usr/bin/env python 

    import pwd 
    from ansible.module_utils.basic import AnsibleModule 

    def main(): 
        # Parsing argument file 
        module = AnsibleModule( 
            argument_spec = dict( 
                user = dict(required=True) 
            ) 
        ) 
        user = module.params.get('user') 

        # Check if user exists 
        try: 
            pwd.getpwnam(user) 
            success = True 
            ret_msg = 'User %s exists' % user 
        except KeyError: 
            success = False 
            ret_msg = 'User %s does not exists' % user 

        # Error handling and JSON return 
        if success: 
            module.exit_json(msg=ret_msg) 
        else: 
            module.fail_json(msg=ret_msg) 

    if __name__ == "__main__": 
        main() 

```

让我们分解上述模块并看看它是如何工作的，如下所示：

```
    #!/usr/bin/env python 

    import pwd 
    from ansible.module_utils.basic import AnsibleModule 

```

如你所见，我们没有导入`sys`、`shlex`和`json`；我们不再使用它们，因为所有需要它们的操作现在都由 Ansible 的`module_utils`完成。

```
    # Parsing argument file 
    module = AnsibleModule( 
        argument_spec = dict( 
            user = dict(required=True) 
        ) 
    ) 
    user = module.params.get('user') 

```

之前，我们对参数文件进行了大量处理以获取最终的用户参数。Ansible 通过提供一个`AnsibleModule`类使这一过程变得简单，它会自动处理所有操作并提供最终的参数。`required=True`参数表示该参数是必需的，如果没有传递该参数，执行将失败。`required`的默认值是`False`，这意味着用户可以跳过该参数。然后，你可以通过调用`module.params`字典上的`get`方法来访问这些参数的值。远程主机上的用户检查逻辑将保持不变，但错误处理和返回部分将如下所示：

```
    # Error handling and JSON return 
    if success: 
        module.exit_json(msg=ret_msg) 
    else: 
        module.fail_json(msg=ret_msg) 

```

使用`AnsibleModule`对象的一个优点是，它提供了非常好的功能来处理返回值到 playbook。我们将在下一节深入探讨这一点。

### 注意

我们本可以将检查用户的逻辑和返回部分合并，但为了可读性，我们保持了它们分开。

为了验证一切是否按预期工作，我们可以在`playbooks/check_user_py2.yaml`中创建一个新的 playbook，内容如下：

```
    - hosts: localhost 
      vars: 
        user_ok: root 
        user_ko: this_user_does_not_exists 
      tasks: 
      - name: 'Check if user {{ user_ok }} exists' 
        check_user_py2: 
          user: '{{ user_ok }}' 
      - name: 'Check if user {{ user_ko }} exists' 
        check_user_py2: 
          user: '{{ user_ko }}' 

```

使用以下命令运行：

```
ansible-playbook playbooks/check_user.yaml

```

我们应该收到以下输出：

```
PLAY [localhost] *************************************************
TASK [setup] *****************************************************
ok: [localhost] 
TASK [Check if user root exists] *********************************
ok: [localhost] 
TASK [Check if user this_user_does_not_exists exists] ************
fatal: [localhost]: FAILED! => {"changed": false, "failed": true, "msg": "User this_user_does_not_exists does not exists"} 
NO MORE HOSTS LEFT ***********************************************
 to retry, use: --limit @playbooks/check_user_py2.retry 
PLAY RECAP *******************************************************
localhost         : ok=2    changed=0    unreachable=0    failed=1

```

这与我们的预期一致。

## 使用`exit_json`和`fail_json`处理

Ansible 通过提供`exit_json`和`fail_json`方法，提供了一种更简便的方式来处理成功和失败。你可以直接将消息传递给这些方法，Ansible 会处理其余部分。你还可以向这些方法传递额外的变量，Ansible 会将这些变量打印到`stdout`。例如，除了消息外，你可能还想打印用户的`uid`和`gid`参数。你可以通过将这些变量与逗号分隔后传递给`exit_json`方法来实现。

让我们看看如何将多个值返回到`stdout`，这在以下代码中得以演示，该代码位于`library/check_user_id.py`中：

```
    #!/usr/bin/env python 

    import pwd 
    from ansible.module_utils.basic import AnsibleModule 

    class CheckUser: 
        def __init__(self, user): 
            self.user = user 

        # Check if user exists 
        def check_user(self): 
            uid = '' 
            gid = '' 
            try: 
                user = pwd.getpwnam(self.user) 
                success = True 
                ret_msg = 'User %s exists' % self.user 
                uid = user.pw_uid 
                gid = user.pw_gid 
            except KeyError: 
                success = False 
                ret_msg = 'User %s does not exists' % self.user 
            return success, ret_msg, uid, gid 

    def main(): 
        # Parsing argument file 
        module = AnsibleModule( 
            argument_spec = dict( 
                user = dict(required=True) 
            ) 
        ) 
        user = module.params.get('user') 

        chkusr = CheckUser(user) 
        success, ret_msg, uid, gid = chkusr.check_user() 

        # Error handling and JSON return 
        if success: 
            module.exit_json(msg=ret_msg, uid=uid, gid=gid) 
        else: 
            module.fail_json(msg=ret_msg) 

    if __name__ == "__main__": 
        main() 

```

如你所见，我们返回了用户的`uid`和`gid`以及消息`msg`。你可以有多个值，Ansible 会以字典格式打印所有这些值。我们可以在`playbooks/check_user_id.yaml`中创建一个 playbook，内容如下：

```
    - hosts: localhost 
      vars: 
        user: root 
      tasks: 
      - name: 'Retrive {{ user }} data if it exists' 
        check_user_id: 
          user: '{{ user }}' 
        register: user_data 
      - name: 'Print user {{ user }} data' 
        debug: 
          msg: '{{ user_data }}' 

```

使用以下命令运行：

```
ansible-playbook playbooks/check_user.yaml

```

我们应该收到以下输出：

```
PLAY [localhost] *************************************************
TASK [setup] *****************************************************
ok: [localhost] 
TASK [Retrieve fale data if it exists] ****************************
ok: [localhost] 
TASK [Print user fale data] **************************************
ok: [localhost] => {
 "msg": {
 "changed": false,
 "gid": 1000,
 "msg": "User root exists",
 "uid": 1000
 }
} 
PLAY RECAP *******************************************************
localhost         : ok=3    changed=0    unreachable=0    failed=0

```

## 测试 Python 模块

正如我们所看到的，你可以通过创建非常简单的 playbook 来测试你的模块。你也可以通过更直接的方式来测试你的模块。为此，我们需要克隆 Ansible 的官方仓库（如果你还没有克隆的话）：

```
git clone git://github.com/ansible/ansible.git --recursive

```

引入环境文件：

```
source ansible/hacking/env-setup

```

我们现在可以使用`test-module`工具运行脚本，并将文件名作为命令行参数传递：

```
ansible/hacking/test-module -m library/check_user_id.py -a "user=root"

```

结果将类似于以下内容：

```
    * including generated source, if any, saving to: /home/fale/.ansible_module_generated 
    * ansiballz module detected; extracted module source to: /home/fale/debug_dir 
    *********************************** 
    RAW OUTPUT 

    {"msg": "User root exists", "invocation": {"module_args": {"user": "root"}}, "gid":     0, "uid": 0, "changed": false} 

    *********************************** 
    PARSED OUTPUT 
    { 
        "changed": false, 
        "gid": 0, 
        "invocation": { 
            "module_args": { 
                "user": "root" 
            } 
        }, 
        "msg": "User root exists", 
        "uid": 0 
    }

```

### 注意

如果你没有使用`AnsibleModule`，直接执行脚本也很简单，这主要是因为该模块需要大量的 Ansible 特定变量，因此模拟 Ansible 的运行比直接运行 Ansible 本身要复杂。

# 使用 Bash 模块

Ansible 中的 Bash 模块与其他 Bash 脚本没有什么不同，区别仅在于它打印数据的方式，即打印到`stdout`。Bash 模块可以非常简单，比如检查远程主机上是否有进程在运行，也可以运行一些复杂的命令。

### 注意

如前所述，一般建议使用 Python 编写模块。在我看来，第二个最佳选择（仅适用于非常简单的模块）是`bash`模块，因为它简单且有广泛的用户基础。

让我们创建一个名为`library/kill_java.sh`的文件，内容如下：

```
    #!/bin/bash 
    source $1 

    SERVICE=$service_name 

    JAVA_PIDS=$(/usr/java/default/bin/jps | grep ${SERVICE} | awk '{print $1}') 

    if [ ${JAVA_PIDS} ]; then 
        for JAVA_PID in ${JAVA_PIDS}; do 
            /usr/bin/kill -9 ${JAVA_PID} 
        done 
        echo "failed=False msg="Killed all the orphaned processes for ${SERVICE}"" 
        exit 0 
    else 
        echo "failed=False msg="No orphaned processes to kill for ${SERVICE}"" 
        exit 0 
    fi 

```

上面的`bash`模块将获取`service_name`参数，并强制终止属于该服务的所有 Java 进程。正如你所知道的，Ansible 将参数文件传递给模块。然后，我们使用`source $1`来加载参数文件，这实际上会将名为`service_name`的环境变量设置好。接着，我们通过`$service_name`来访问该变量，如下所示：

```
    source $1 

    SERVICE=$service_name 

```

然后我们检查是否获得了任何该服务的 PID，并对其进行循环操作，强制终止所有与`service_name`匹配的 Java 进程。进程结束后，我们以`failed=False`和退出码`0`退出模块，并附带一条消息，正如你在这里看到的：

```
    if [ ${JAVA_PIDS} ]; then 
        for JAVA_PID in ${JAVA_PIDS}; do 
            /usr/bin/kill -9 ${JAVA_PID} 
        done 
        echo "failed=False msg="Killed all the orphaned processes for ${SERVICE}"" 
        exit 0 

```

如果我们没有找到任何正在运行的服务进程，我们仍然会以退出码`0`退出模块，因为终止 Ansible 的运行可能没有意义；这一部分代码如下：

```
    else 
        echo "failed=False msg="No orphaned processes to kill for ${SERVICE}"" 
        exit 0 
    fi 

```

### 注意

你还可以通过打印`failed=True`并退出码为`1`来终止 Ansible 的运行。

Ansible 允许你返回键值输出，即使语言本身不支持 JSON。这使得 Ansible 对开发者和系统管理员更友好，并允许用任何自己选择的语言编写自定义模块。让我们通过将参数文件传递给模块来测试`bash`模块。现在，我们可以在`/tmp/arguments`目录下创建一个参数文件，并将`service_name`参数设置为 Jenkins，如下所示：

```
    service_name=jenkins 

```

现在，你可以像运行其他 Bash 脚本一样运行该模块。让我们看看当我们运行它时会发生什么：

```
bash library/kill_java.sh /tmp/arguments

```

我们应该收到以下输出：

```
failed=False msg="No orphaned processes to kill for jenkins"

```

正如预期的那样，即使本地主机上没有 Jenkins 进程运行，模块也没有失败。

# 使用 Ruby 模块

在 Ruby 中编写模块和在 Python 或 Bash 中编写模块一样简单。你只需要处理好参数、错误、返回语句，当然，还需要掌握基本的 Ruby！让我们创建一个名为`library/rsync.rb`的文件，并写入以下代码：

```
    #!/usr/bin/env ruby 

    require 'rsync' 
    require 'json' 

    src = '' 
    dest = '' 
    ret_msg = '' 
    SUCCESS = '' 

    def print_message(state, mdg, key='Failed') 
        message = { 
            key => state, 
            "msg" => msg 
        } 
        print message.to_json 
        exit 1 if state == false 
        exit 0 
    end 

    args_file = ARGV[0] 
    data = File.read(args_file) 
    arguments = data.split(" ") 
    arguments.each do |argument| 
        print_message(false, "Argument should be name-value pairs. Example name=foo") if not argument.include("=") 
        field.value = argument.split("=") 
        if field == "src" 
            src = value 
        elseif field == "dest" 
            dest = value 
        else print_message(false, "Invalid argument provided. Valid arguments are src and dest.") 
        end 
    end 

    result - Rsync.run("#{src}", "#{dest}") 
    if result.success? 
        success = true 
        ret_msg = "Copied file successfully" 
    else 
        success = false 
        ret_msg = result.error 
    end 

    if success 
        print_message(false, "#{ret_msg}") 
    else 
        print_message(true, "#{ret_msg}") 
    end 

```

在前面的模块中，我们首先处理用户的参数，然后使用`rsync`库复制文件，最后返回输出。让我们分解前面的代码，看看它是如何工作的。

我们首先编写了一个方法 `print_message`，该方法将输出以 JSON 格式打印。通过这样做，我们可以在多个地方重用相同的代码。记住，如果你希望 Ansible 运行失败，你的模块输出应包含 `failed=true`；否则，Ansible 会认为模块成功，并继续执行下一个任务。得到的输出如下：

```
    #!/usr/bin/env ruby 

    require 'rsync' 
    require 'json' 

    src = '' 
    dest = '' 
    ret_msg = '' 
    SUCCESS = '' 

    def print_message(state, mdg, key='Failed') 
        message = { 
            key => state, 
            "msg" => msg 
        } 
        print message.to_json 
        exit 1 if state == false 
        exit 0 
    end 

```

接下来我们处理参数文件，该文件包含由空格分隔的键值对。这与我们之前使用 Python 模块的方式类似，我们也要解析出这些参数。同时，我们进行一些检查，确保用户没有遗漏任何必需的参数。在此例中，我们检查是否已指定 `src` 和 `dest` 参数，并在参数未提供时打印消息。进一步的检查可以包括参数的格式和类型。你可以添加这些检查以及任何你认为重要的检查。例如，如果某个参数是 `date`，你可能希望验证输入的确是正确的日期。考虑以下代码片段，它展示了讨论过的参数：

```
    args_file = ARGV[0] 
    data = File.read(args_file) 
    arguments = data.split(" ") 
    arguments.each do |argument| 
        print_message(false, "Argument should be name-value pairs. Example name=foo") if not argument.include("=") 
        field.value = argument.split("=") 
        if field == "src" 
            src = value 
        elseif field == "dest" 
            dest = value 
        else print_message(false, "Invalid argument provided. Valid arguments are src and dest.") 
        end 
    end 

```

一旦我们获取到所需的参数，我们将使用 `rsync` 库来复制文件，如下所示：

```
    result - Rsync.run("#{src}", "#{dest}") 
    if result.success? 
        success = true 
        ret_msg = "Copied file successfully" 
    else 
        success = false 
        ret_msg = result.error 
    end 

```

最后，我们检查 `rsync` 任务是通过还是失败，并调用 `print_message` 函数将输出打印到 `stdout`，如下所示：

```
    if success 
        print_message(false, "#{ret_msg}") 
    else 
        print_message(true, "#{ret_msg}") 
    end 

```

你可以通过简单地将参数文件传递给模块来测试 Ruby 模块。为此，我们可以创建一个名为 `/tmp/arguments` 的文件，内容如下：

```
    src=/var/log/ansible.log dest=/tmp/ansible_backup.log 

```

现在让我们运行该模块，如下所示：

```
ruby library/rsync.rb /tmp/arguments

```

我们将收到以下输出：

```
    {"failed":false,"msg":"Copied file successfully"} 

```

我们将留给你完成 `serverspec` 测试部分。

# 测试模块

测试往往被低估，因为对其目的和带来的好处缺乏理解。测试模块和测试 Ansible 剧本的其他部分一样重要，因为模块中的一个小改动可能会破坏整个剧本。我们将以本章第一部分编写的 Python 模块为例，使用 Python 的 nose 测试框架编写集成测试。单元测试也是被推荐的，但对于我们的场景——检查某个用户是否存在远程主机，集成测试更加合适。

### 注释

`nose` 是一个 Python 测试框架。更多信息，请访问 [`nose.readthedocs.org/en/latest/`](https://nose.readthedocs.org/en/latest/)。

为了 `test` 模块，我们将之前的模块转换为 Python 类，以便我们可以在测试中直接导入该类，只运行模块的主要逻辑。以下代码展示了重新结构化后的 `library/check_user_py3.py` 模块，它将检查远程主机上是否存在某个用户：

```
    #!/usr/bin/env python 

    import pwd 
    from ansible.module_utils.basic import AnsibleModule 

    class User: 
        def __init__(self, user): 
            self.user = user 

        # Check if user exists 
        def check_if_user_exists(self): 
            try: 
                user = pwd.getpwnam(self.user) 
                success = True 
                ret_msg = 'User %s exists' % self.user 
            except KeyError: 
                success = False 
                ret_msg = 'User %s does not exists' % self.user 
            return success, ret_msg 

    def main(): 
        # Parsing argument file 
        module = AnsibleModule( 
            argument_spec = dict( 
                user = dict(required=True) 
            ) 
        ) 
        user = module.params.get('user') 

        chkusr = User(user) 
        success, ret_msg = chkusr.check_if_user_exists() 

        # Error handling and JSON return 
        if success: 
            module.exit_json(msg=ret_msg, uid=uid, gid=gid) 
        else: 
            module.fail_json(msg=ret_msg) 

    if __name__ == "__main__": 
        main() 

```

如你在上面的代码中看到的，我们创建了一个名为 `User` 的类。我们实例化了该类，并调用 `check_if_user_exists` 方法来检查用户是否实际存在于远程机器上。现在是时候编写集成测试了。我们假设你已经在系统上安装了 `nose` 包。如果没有，不用担心！你仍然可以通过以下命令安装该包：

```
pip install nose

```

让我们现在编写集成测试文件 `library/test_check_user_py3.py`，内容如下：

```
    from nose.tools import assert_equals, assert_false, assert_true 
    import imp 
    imp.load_source("check_user","check_user_py3.py") 
    from check_user import User 

    def test_check_user_positive(): 
        chkusr = User("root") 
        success, ret_msg = chkusr.check_if_user_exists() 
        assert_true(success) 
        assert_equals('User root exists', ret_msg) 

    def test_check_user_negative(): 
        chkusr = User("this_user_does_not_exists") 
        success, ret_msg = chkusr.check_if_user_exists() 
        assert_false(success) 
        assert_equals('User this_user_does_not_exists does not exists', ret_msg) 

```

在上面的集成测试中，我们导入了 `nose` 包和我们的模块 `check_user`。我们通过传入要检查的用户来调用 `User` 类。然后，我们通过调用 `check_if_user_exists()` 方法来检查用户是否存在于远程主机上。`nose` 的方法 `assert_true`、`assert_false` 和 `assert_equals` 可以用于将预期值与实际值进行比较。只有当断言方法通过时，测试才会通过。你可以在同一个文件中编写多个测试方法，只需让这些方法的名称以 `test_` 开头，例如 `test_check_user_positive()` 和 `test_check_user_negative()` 方法。Nose 测试会执行所有以 `test_` 开头的方法。

### 注意

如你所见，我们实际上为一个函数创建了两个测试。这是测试的一个关键部分。始终尝试测试你知道肯定能成功的情况，但也不要忘记测试那些你预期会失败的情况。

现在我们可以通过以下方式运行 nose 来测试它是否有效：

```
cd library
nosetests -v test_check_users_py3.py

```

你应该会收到类似下面的输出：

```
test_check_user_py3.test_check_user_positive ... ok
test_check_user_py3.test_check_user_negative ... ok
---------------------------------------------------
Ran 2 tests in 0.001s
OK

```

如你所见，测试通过了，因为 `root` 用户存在于主机上，而 `this_user_does_not_exists` 用户则不存在。

### 注意

我们在 `nose` 测试中使用了 `-v` 选项来开启 **详细模式**。

对于更复杂的模块，我们建议你编写单元测试和集成测试。你可能会想，为什么我们不使用 `serverspec` 来测试这个模块。

我们仍然建议在 playbooks 中作为功能测试运行 `serverspec` 测试，但对于单元和集成测试，建议使用知名的框架。同样，如果你编写 Ruby 模块，我们建议你使用诸如 `rspec` 的框架来为它们编写测试。如果你的自定义 Ansible 模块有多个参数和多个组合，那么你将需要编写更多的测试来测试每种场景。最后，我们建议你将所有这些测试作为 CI 系统的一部分来运行，无论是 Jenkins、Travis 还是任何其他系统。

问题

本节给出了一些问题供你思考：

+   你能想到你每天执行的常见任务以及如何为这些任务编写 Ansible 模块吗？请按你如何在 playbook 中调用模块的方式列出这些任务。

+   你认为你的团队会使用哪种语言来编写模块？

+   你能回顾一下你在第三章《扩展到多个主机》中可能编写的角色，并查看哪些可以潜在地转换为自定义模块吗？

# 总结

通过这部分内容，我们来到了这个较小但重要的章节的结束，这一章的重点是如何通过编写自定义模块来扩展 Ansible。你学习了如何使用 Python、Bash 和 Ruby 来编写你的模块。我们还看到了如何为模块编写集成测试，以便它们能够集成到你的 CI 系统中。未来，扩展你的 Ansible 功能通过模块应该会变得更加简单！

接下来，我们将进入配置、部署和编排的世界，看看 Ansible 如何解决我们在配置新实例或想要将软件更新部署到我们环境中各个实例时遇到的基础设施问题。我们保证这段旅程会很有趣！
