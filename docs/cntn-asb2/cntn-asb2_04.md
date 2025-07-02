# 第四章：一个 role 里有什么？

在第三章，*Your First Ansible Container Project*，我们学习了关于 Ansible Container roles 的基础知识，它们的功能及如何从 Ansible Galaxy 下载、安装和调整它们。在本章中，我们将学习如何编写我们自己的 Ansible Container roles，用于从头开始构建自定义的容器镜像。您将了解到，Ansible 提供了一种易于学习、表达力强的语言，用于定义所需的状态和服务配置。为了说明 Ansible Container 如何快速构建服务和运行容器，本章的过程中我们将编写一个 role，用于构建一个可以在您的本地工作站上运行的 MariaDB MySQL 容器。在本章中，我们将涵盖以下内容：

+   使用 Ansible Container 的自定义 roles

+   MariaDB 的简要概述

+   初始化一个 Ansible Container role

+   一个容器化 role 里有什么？

+   创建 MariaDB 项目和 role

+   编写一个容器化 role

+   自定义一个容器化 role

# 使用 Ansible Container 的自定义 roles

本书内容逐渐深入时的一个主题是 Ansible Container 给予您多大的自由，可以快速、高效、安全和可靠地构建和部署自定义的容器镜像。到目前为止，我们已经学习了使用 Ansible Container 来定义和运行来自预构建社区容器的服务，以及利用社区编写的 roles 来实例化、构建和定制我们的容器。这是开始使用 Ansible Container 并熟悉 Ansible Container 工作流程的绝佳方式。然而，当您开始编写构建自定义容器镜像的 roles 时，Ansible Container 的真正力量开始显现出来。

如果您有使用 Ansible 作为配置管理工具的经验，您可能已经熟悉了编写 Ansible playbook 和 roles。这肯定会让您在编写容器化 roles 方面有一定的优势，但这并不是本章示例工作的先决条件。为了让每个人都处于同一起跑线上，我假设您没有编写 Ansible playbook 或 roles 的经验，我们将从头开始。对于那些 Ansible 老手来说，这些内容很大程度上会是回顾，但希望您也能学到一些新东西。对于 Ansible 初学者，我希望本章能激发您的好奇心，不仅仅是进一步构建更高级的 Ansible Container roles，还要深入探索 Ansible Core 配置管理概念。

Ansible 的最初动机是创建一个配置管理和编排系统，使得几乎任何人都能轻松上手并开始使用。Ansible 很快在软件开发人员、系统管理员和 DevOps 工程师中变得极为流行，因为它不仅易于采用，而且易于定制，甚至能够集成到现有的平台和配置管理工具中。我第一次开始使用 Ansible 是因为当时我在做的项目需要我登录到大量的裸机服务器和虚拟机上，反复执行同一组命令。当时，我试图通过编写不稳定的 shell 脚本来简化这一过程，这些脚本会使用 SSH 将远程命令推送到服务器。通过研究如何使这些脚本更加可靠，我发现了 Ansible，并立即采用了它，它使我的工作变得比我想象的更加简单和可靠。我相信，Ansible 在 IT 行业如此受欢迎的原因有两个主要方面：

+   易于理解的 YAML 语法，适用于 playbook 和角色。YAML 易于学习和编写，非常适合 Ansible。

+   数百个，如果不是成千上万的，Ansible Core 内置模块。这些模块使我们能够开箱即用地做几乎任何你能想象的事情。

让我们来看看 Ansible 的这两个独特方面，理解我们如何在自己的项目中利用这种易用性。

# YAML 语法

YAML 是一种数据序列化格式，其递归地代表了 *YAML Ain't Markup Language*。你可能以前使用过其他序列化格式，例如 XML 或 JSON。YAML 的独特之处在于它易于编写，而且可能是当前最人类可读的数据格式。Ansible 选择使用 YAML 作为定义其 playbook 语法和语言的基础，原因是即使你没有编程背景，YAML 也非常容易入门，编写、使用和理解都很简单。YAML 的独特之处在于它通过使用一系列冒号（`:`）、破折号（`-`）和缩进（空格，不是制表符）来定义键值对。这些键值对可以用来定义几乎所有类型的计算机科学数据类型，例如整数、布尔值、字符串、数组和哈希表。以下是一个 YAML 文档的示例，展示了这些构造的一些用法：

```
---
#This is a Comment in a YAML document. Notice the YAML document starts with a series of three dashes: ---

MyString: "This is a string"

MyArray:
  - "Item1"
  - "Item2"

MyBoolean: true

MyInteger: 10

MyHashTable:
  KeyOne: "ValueOne"
  KeyTwo: "ValueTwo"
  KeyThree: "ValueThree"
```

上面的示例展示了一个简单的 YAML 文件，包含最基本的构造：一个字符串变量，一个数组（项目列表），一个布尔值（真/假）变量，一个整数变量，以及一个包含一系列键值对的哈希表。这可能看起来与我们在本书中修改 `container.yml` 文件以及 Docker Compose 文件时做的工作非常相似。这些格式也是定义在 YAML 中，并且包含许多相同的构造。

在前面的例子中，我想要提醒您一些事情（当您开始编写 Ansible playbook 和角色时，也应该注意到这些）：

+   所有 YAML 文档都以三个破折号开头：`---`。这很重要，因为您可以在同一个文件中定义多个 YAML 文档。文档之间用三个破折号分隔。

+   注释使用井号符号`#`定义。

+   字符串被引号包围。这将字符串与字面值（例如没有引号的布尔值（true 或 false）或整数（没有引号的数字））分开。如果您将单词*true*/*false*或数值用引号包围起来，它们将被解释为字符串。

+   冒号（`:`）用于分隔键值对，几乎定义了所有内容。

+   缩进用两个空格表示。在 YAML 格式中不识别制表符。开始编写 YAML 文档时，请确保您的文本编辑器配置为在按下*Tab*键时将两个空格插入到文档中。这样可以在输入时快速自然地缩进文本。

我意识到 YAML 语法比我在这个示例中提供的要多得多。我在这里的目标是比早期章节更深入地挖掘，以帮助读者更深入地理解 YAML 格式。这绝不是整个 YAML 格式的完整描述。如果您想了解更多关于 YAML 的信息，我建议您访问官方 YAML 规范网站：[`yaml.org`](http://yaml.org/)。

# Ansible 模块

使得 Ansible 如此受欢迎且易于使用的第二部分是 Ansible 可以利用的大量模块，这些模块可以做用户能想到的几乎任何事情。将模块视为 Ansible 的构建块，定义您的 playbook 的操作。Ansible 模块可以编辑远程系统上文件的内容，添加或删除用户，安装服务包，甚至与远程应用程序的 API 进行交互。模块本身是用 Python 编写的，并以脚本格式从 YAML playbook 中调用。Playbook 本身只是对 Ansible 模块的一系列调用，执行特定的任务序列。让我们看一个非常简单的 playbook，以了解这在实践中是如何工作的：

```
---
- name: Create User Account
  user:
    name: MyUser
    state: present

- name: Install Vim text editor
  apt:
    name: vim
    state: present
```

这个简单的 playbook 由两个独立的任务组成：创建一个用户账户和安装 Vim 文本编辑器。Ansible 中的每个任务都调用恰好一个模块来执行操作。Ansible 中的任务使用 YAML 中的短横线来定义，接着是任务的名称、模块的名称以及所有要传递给该模块的参数，缩进格式如下所示。在我们的第一个任务中，我们通过调用 `user` 模块来创建一个用户账户。我们为 `user` 模块提供了两个参数：`name` 和 `state`。`name`表示我们要创建的用户的名称，而 `state`表示我们希望远程系统或容器中的用户状态。此时，我们希望存在一个名为 `MyUser` 的用户，且该用户的状态为 `present`。如果这个 Ansible playbook 执行时，名为 `MyUser` 的用户已经存在，Ansible 将不会采取任何操作，因为系统已经处于期望状态。

本手册中的第二个任务是在我们的远程系统或容器上安装文本编辑器 Vim。为此，我们将使用`apt`模块来安装一个 Debian APT 包。如果这是一个 Red Hat 或 CentOS 系统，我们也会使用`yum`或`dnf`模块。`name`表示我们要安装的包的名称，`state`表示服务器或容器的期望状态。因此，我们希望 Vim Debian 包能够被安装。如前所述，Ansible 有成百上千的模块可以在 playbook 和角色中使用。你可以在 Ansible 文档中找到按类别组织的完整模块列表，以及这些模块所需参数的优秀示例，网址为[`docs.ansible.com/ansible/latest/modules_by_category.html`](http://docs.ansible.com/ansible/latest/modules_by_category.html)。

`state` 参数还可以取 `absent` 值，用于删除用户、包或几乎任何可以定义的东西。

Ansible Container 的一个主要优势是，在编写容器配置时使用 Ansible 角色，你可以使用所有可用的 Ansible 模块。不幸的是，并不是所有的 Ansible 模块都能在容器环境中工作。一个典型的例子是管理运行中服务状态的模块，比如 `service` 模块。`service` 模块在容器中无法运行，因为应用容器通常缺少传统的初始化系统（你会在完整操作系统中找到它们），这些系统负责启动、停止和重启运行中的服务。在容器化的环境中，这个过程通过使用 `CMD` 或 `entrypoint` 语句直接执行服务二进制文件来处理。

此外，几乎所有管理云服务编排或调用外部 API 的模块都无法在容器化环境中运行。这一点非常直接，因为在构建一个独立的容器化微服务时，通常不会想要编排外部服务的状态。当然，如果你正在编写一个 Ansible 剧本来部署一个之前使用 Ansible Container 构建的容器化应用，你可以使用这些编排模块来在容器上线时作出响应。不过，在本章中，我们将仅限于讨论编写构建容器化服务的角色。

# MariaDB 简要概述

在本章中，我们将编写一个 Ansible 角色来构建 MariaDB 数据库容器。MariaDB 是 MySQL 关系数据库服务器的一个分支，提供了许多 MySQL 中没有的自定义功能和优化。开箱即用，MariaDB 支持多种优化，如复制、查询优化、加密、性能和速度上的提升，相比标准的 MySQL 更具优势，但仍然完全兼容 MySQL，采用免费开源的 GPL 许可证。选择 MariaDB 作为示例，是因为它相对容易部署，而且该应用本身是免费的。在本章中，我们将构建一个相对基础的单节点 MariaDB 安装，这个安装不包含很多你在生产环境安装中会找到的功能和性能优化。本文的目的不是教你如何构建一个生产就绪的 MariaDB 容器，而是通过 Ansible Container 构建容器化服务的概念。如果你想进一步拓展这个示例，可以根据自己的需求调整这段代码。对那些能够构建生产就绪容器的朋友，将会额外加分！

# 初始化一个 Ansible Container 角色

如前所述，Ansible 角色是一个自包含的、可重用的一组剧本、模板、变量和其他元数据，用于定义一个应用或服务。由于 Ansible 角色是为与 Ansible Galaxy 配合使用而设计的，Ansible Galaxy 命令行工具具有内置功能，可以初始化包含所有正确目录、默认文件和结构的角色，从而创建一个功能齐全的 Ansible 角色，且无需过多的麻烦。这与 `ansible container init` 命令创建 Ansible Container 项目时的工作方式非常相似。

# 容器启用角色中包含什么？

为了在 Ansible Container 中创建一个新的容器启用角色，我们将使用带有 `container-enabled` 标志的 `ansible-galaxy init` 命令来为我们创建新的角色目录结构。为了了解当我们使用这个命令时发生了什么，我们将在我们的 Vagrant 虚拟机的 `/tmp` 目录中初始化一个角色，看看 Ansible 为我们创建了什么：

```
ubuntu@node01:/tmp$ ansible-galaxy init MyRole --container-enabled
- MyRole was created successfully
```

在成功执行 `init` 命令后，Ansible 应返回一条消息，指示你的新角色已 `创建成功`。如果你运行 `ls` 命令，你会看到一个新目录，目录名就是我们刚刚初始化的角色的名称。根据默认的目录结构，角色的所有组成部分都位于此目录中。当你从 Ansible 调用一个角色时，Ansible 会查找你指定的角色所在的所有位置，并查找一个与角色同名的目录。稍后在本章中，我们将更详细地了解这一点。如果你进入该目录，你将看到类似于以下的文件夹结构：

```
MyRole/
├── defaults
│   └── main.yml
├── handlers
│   └── main.yml
├── meta
│   ├── container.yml
│   └── main.yml
├── README.md
├── tasks
│   └── main.yml
├── templates
├── tests
│   ├── ansible.cfg
│   ├── inventory
│   └── test.yml
└── vars
    └── main.yml
```

让我们看一下这些目录和文件的作用：

+   `defaults/`：`defaults` 是一个包含特定于你角色的变量的目录，并且在覆盖值时具有最低优先级。如果你希望将一些变量放入角色中，并且希望用户一定要重写这些变量，则应将它们放在该目录的 `main.yml` 文件中。这与 `vars/` 目录不同，不要混淆。

+   `handlers/`：`handlers` 是 Ansible 中的一个概念，定义了在角色执行过程中，响应其他任务发送的通知事件时应执行的任务。例如，你可能有一个任务用于更新角色中的配置文件。如果服务需要因该配置文件更新而重启，你可以在任务中指定一个 `notify:` 步骤，并指定处理程序的名称。如果父任务执行并且状态为 `CHANGED`，Ansible 会在 `handlers/` 目录中查找由通知语句指定的任务，并执行该任务。请注意，除非另一个 playbook 任务通过 `notify:` 语句明确调用该任务并且导致状态改变，否则 handlers 不会执行。由于容器通常不依赖于外部事件和情况，handlers 在启用容器的角色中并不常见。

+   `meta`：`Meta` 是一个包含 Ansible 角色元数据的目录。在启用容器的角色中，它包含两个主要文件：`main.yml`，该文件包含关于角色的一般元数据，例如依赖关系、Ansible Galaxy 数据以及角色所依赖的条件。为了这个示例的目的，我们不会对这个文件做太多处理。第二个文件，`container.yml`，对我们来说更为重要。这个 `container.yml` 文件是专门针对启用容器的角色的，它对于指定当我们调用角色时将被注入到项目级 `container.yml` 文件中的默认值至关重要。在这里，我们可以指定容器镜像、卷信息，以及我们希望容器默认运行的命令和 `entrypoint` 数据。如果我们愿意，所有这些数据都可以在父级 `container.yml` 文件中覆盖。

+   `tasks`：`tasks` 目录是我们指定实际在容器中执行的任务，并构建服务的地方。默认情况下，Ansible 会执行 `main.yml` 文件，并按照指定的顺序执行所有任务。任何其他的任务文件也可以放入此目录，并可以通过在 `main.yml` 文件中使用 `include:` 语句来执行。

+   `templates`：`templates` 目录存储我们希望在角色中使用的配置文件模板。由于 Ansible 是基于 Python 的，它使用 Jinja2 模板引擎将配置模板放入容器，并根据 `defaults/` 和 `vars/` 目录中识别的变量更新值。此目录中的所有文件应该具有 `.j2` 文件扩展名，尽管这并不是强制要求的。

+   `tests`：任何您希望 CICD 工具执行的自动化测试都应该放在这里。通常，开发者会将任何自定义的 Ansible 配置、参数或清单文件放入这个目录中，以供 CI/CD 工具作为输入使用，这些文件通常是自动生成的。

+   `vars`：`vars/` 目录是开发者可以在此指定其他可用于角色的变量的位置。需要注意的是，`vars/` 目录的优先级低于 `defaults/` 目录，因此在这里定义的变量比在 `defaults` 中指定的变量更难覆盖。通常，当我编写角色时，我会将所有变量都放在 `defaults` 目录中，因为我希望用户能够完全覆盖他们想要的任何内容。但在某些情况下，您可能不希望让变量太容易被访问，这时它们可以被指定在 `vars/` 目录中。

任何在您的角色中名为 `main.yml` 的文件都表示该文件是默认文件，并会被自动执行。

现在我们知道了容器启用角色的组成部分，我们可以利用这些知识创建一个新的 Ansible 容器项目，来构建我们的 MariaDB MySQL 角色。为此，我们将初始化一个新项目，并创建一个名为`roles/`的子目录，里面将包含我们要创建的角色。当我们构建项目时，Ansible 会知道去我们的`roles/`目录中查找我们指定并创建的所有角色。请注意，本章节的以下部分会包含大量代码。为了让跟随过程更容易，完整的示例可以在官方书籍的 GitHub 仓库中找到，位于`AnsibleContainer/mariadb_demo`目录下。然而，学习如何编写 Ansible 代码的最佳方式是通过重复和实践，而这一点只有通过自己编写代码才能实现。强烈建议，尽管逐字复制本章编写的代码可能不太实际，但应该通过使用这些示例创建自己的项目或修改 Git 仓库中的示例来练习编写 Ansible 代码。你写的代码越多，成为更好、更流利的 Ansible 开发者的机会就越大。

# 初始化 MariaDB 项目和角色

现在我们已经了解了容器启用角色的结构，我们可以通过初始化一个新的 Ansible 容器项目来启动我们的 MariaDB 容器。在 Vagrant 主机上的新目录中，像往常一样使用`ansible-container init`命令启动一个新项目：

```
ubuntu@node01:$ ansible-container init 
Ansible Container initialized.
```

在我们的`project`目录中，我们可以创建一个目录来存储我们的角色。在 Ansible 核心中，角色的默认位置是在`/etc/ansible/roles`或相对于你正在执行的 playbook 的`roles/`目录中。然而，需要注意的是，角色可以存储在任何位置，只要 Ansible 安装能够读取该路径。为了本示范，我们将把角色路径创建为项目的`子`目录。在我们的`project`目录中，创建一个名为`roles`的新目录，并在该目录中初始化我们的 Ansible 容器角色。我们将把我们的角色命名为`mariadb_role`：

```
ubuntu@node01:$ mkdir roles/
ubuntu@node01:$ cd roles/
ubuntu@node01:roles$ ansible-galaxy init mariadb_role --container-enabled
- mariadb_role was created successfully
```

现在我们的角色已经在项目中创建，我们需要修改项目中的`container.yml`文件，让它知道我们从哪里加载角色的路径，并创建一个我们将使用角色构建的服务。角色的位置可以通过在`container.yml`文件的`settings:`下设置`roles_path`选项来指定。这里，我们可以使用短横线表示法（`-`）将 Ansible 需要查找角色的路径列出为`roles_path`的列表项。我们将指定刚才创建的`roles`目录。在`services:`子部分下，我们可以创建一个名为`MySQL_database_container`的新服务。该服务将利用我们刚刚创建的`mariadb_role`角色。我们还需要确保指定我们希望用于服务的基准镜像。在本示例中，MariaDB 容器将基于 Ubuntu 16.04，因此我们需要确保我们的`conductor_base`镜像与之相同，以确保兼容性。

# container.yml

以下是提供这些设置的`container.yml`文件示例：

```
version: "2"
settings:
  conductor_base: ubuntu:16.04
  roles_path:
    - ./roles/

services:
services:
  MySQL_database_container:
    roles:
      - role: mariadb_role
registries: {}
```

到这时，我们可以构建我们的项目，但这将导致一个空容器，因为我们的角色没有任何任务可以用来构建容器镜像。让我们通过添加任务并更新角色特定的`container.yml`文件来使事情变得有趣。

始终记得使用与服务容器相同的主机基准镜像。这将确保在构建项目时最大程度的兼容性。

# 编写一个支持容器的角色

正如我们之前讨论过的，由于文件路径很容易变得复杂，导致很容易迷失在其中，因此从零开始编写代码并引导读者是相当困难的。在本节中，我将展示修改后的文件内容，并引导读者关注需要解释的文件部分。由于很容易迷失，我会指导读者前往以下网址的官方书籍 GitHub 仓库进行跟随：[`github.com/aric49/ansible_container_lab/tree/master/AnsibleContainer/mariadb_demo`](https://github.com/aric49/ansible_container_lab/tree/master/AnsibleContainer/mariadb_demo)。

作为一个容器启用的角色的开发者，编写角色最重要的部分是角色特定的`container.yml`文件，该文件指定了容器运行时的默认值，以及用于构建容器并将所有组件放置到位的任务。你用来构建容器的任务通常会决定角色特定的`container.yml`文件中的参数。在编写角色时，开发者通常会在编写 playbook 任务时调整和修改`container.yml`文件。当你从项目特定的`container.yml`文件中调用一个角色时，角色特定的`container.yml`文件的内容将被用来构建你的容器。在任何时候，开发者都可以通过简单地修改项目`container.yml`中的参数来覆盖角色特定的`container.yml`文件。

作为角色开发者，为你的角色编写合理的默认值在`container.yml`中是很重要的，这样可以帮助其他用户迅速使用你的角色。对于我们的 MariaDB 示例，我们将创建一个简单的角色特定`container.yml`文件，其内容如下所示：

# roles/mariadb_role/meta/container.yml

这个文件位于 MariaDB 角色的`meta`目录中：

```
from: ubuntu:16.04
ports:
  - "3306:3306"
entrypoint: ['/usr/bin/dumb-init']
command: ['/usr/sbin/MySQLd']
```

我们在角色特定的`container.yml`中定义的参数应该立即引起你的注意，其方式与在项目特定的`container.yml`服务部分中定义参数是完全一致的。在这里，我们使用的是 Ubuntu 16.04 基础镜像，它与我们的 conductor 容器相同。为了使 MySQL 服务能够被外部用户访问，我们将会把主机上的 MySQL 端口 `3306` 映射到容器内的 `3306` 端口。最后，我们将指定一个默认的入口点和命令，当容器启动时应该运行这些命令。最近，容器开发人员中常见的做法是利用轻量级的初始化系统来启动和管理容器内的进程。一个流行的容器初始化系统是 `dumb-init`，它由 Yelp 在 2013 年编写，目的是提供一个易于安装的轻量级初始化二进制文件，用于管理容器内的进程。`dumb-init` 本质上作为 PID 1 在容器内启动，并将容器服务可执行文件作为参数传递给它。这样做的好处是，由于 `dumb-init` 作为 PID 1 运行，所有内核信号将首先被 `dumb-init` 拦截并转发给容器服务（`mysqld`）。如果容器被突然停止或不规范地重启，`dumb-init` 还将为我们的子进程提供回收服务。请记住，使用容器 `init` 系统并不是构建容器的要求，但在某些情况下，它有助于在进程没有正常退出时，运行、停止和重启容器。在这个示例中，我们将使用 `dumb-init` 作为容器的 `entrypoint`，并使用 `command:` 参数将 `/usr/sbin/MySQLd` 命令作为参数传递给它。这样，`mysqld` 进程将在 `dumb-init` 的监督下启动，`dumb-init` 将拦截所有 POSIX 信号并转发给 `mysqld`。

容器化角色的第二个最重要的方面是实际编写在我们的基础镜像中执行的任务，以创建项目容器。所有任务都是位于角色的 `tasks` 目录中的 YAML 文件。每个任务都有一个名称，并调用一个单一的 Ansible 模块，使用参数执行容器中的一项工作。虽然对任务命名或排列顺序没有严格的要求，但你确实需要考虑到 playbook 的流程，特别是与其他任务的依赖关系，这些任务可能会在某些步骤之前或之后执行。你还需要确保任务的命名方式，使得任何查看构建过程的用户，即使他们不是 Ansible 开发人员，也能大致了解发生了什么。命名任务是 Ansible 被誉为 *自文档化* 的原因之一。这意味着，当你编写代码时，代码基本上会自动进行文档化，因为几乎任何阅读你代码的用户，凭借任务命名，都会立刻明白它的作用。还应注意的是，由于所有服务和应用程序都不同，它们的部署和配置方式也不同。使用 Ansible 和容器化技术，没有一种 *一刀切* 的方法能够充分捕捉部署和配置应用程序的最佳实践。Ansible 的一个优势是，它提供了所需的工具，能够自动化几乎任何人能想到的配置，以确保应用程序能可靠地构建和部署。以下是我们正在编写的 MariaDB 角色中 `tasks/main.yml` 文件的内容。花一点时间阅读当前的 playbook；我们将逐个任务进行解析，详细说明 playbook 是如何运行的。在我描述 playbook 的工作原理时，回顾每个任务的内容会对你理解描述有帮助。

# tasks/main.yml

此文件位于 `roles/mariadb_role/tasks/main.yml`：

```
---
- name: Install Base Packages
  apt:
    name: "{{ item }}"
    state: present
    update_cache: true
  with_items:
    - "ca-certificates"
    - "apt-utils"

- name: Install dumb-init for container init system
  get_url:
    url: https://github.com/Yelp/dumb-init/releases/download/v1.2.0/dumb-init_1.2.0_amd64
    dest: /usr/bin/dumb-init
    owner: root
    group: root
    mode: 0775

- name: Create MySQL Group
  group:
    name: mysql
    state: present

- name: Create MySQL Users
  user:
    name: mysql
    state: present
    groups: mysql
    append: true

- name: Install MySQL Server
  apt:
    name: mariadb-server
    state: present
    update_cache: true

- name: Change Permissions on directories
  file:
    path: "{{ item }}"
    owner: mysql
    group: mysql
    mode: 0777
    state: directory
    recurse: true
  with_items:
    - "/etc/mysql/"
    - "/var/lib/mysql/"
    - "/var/run/mysql/"

- name: Remove my.cnf
  file:
    path: /etc/mysql/my.cnf
    state: absent

- name: Install MySQL Configuration File
  template:
    src: my.cnf.j2
    dest: /etc/mysql/my.cnf
    owner: mysql
    group: mysql

- name: Initialize Database
  include: initialize_database.yml
  when:
    - initialize_database == true
```

# 任务细分（main.yml）

+   `安装基础包`：由于我们正在构建一个 Ubuntu 16.04 镜像，`安装基础包` 任务调用 `apt` 包管理模块来安装两个包：`ca-certificates` 和 `apt-utils`，它们是后续任务所必需的。我们希望这些包的状态在 `container` 中是存在的，并且 `apt` 数据库缓存已更新，以便在安装这些包之前使用。我们可以使用 `with_items` 操作符来安装多个包。`with_items` 会遍历指定的列表项，并通过 `apt` 模块运行每个值。使用 `with_items` 后，我们不需要创建两个或更多独立的任务来重复执行相同的操作。在任务的 `name` 部分，我们指定了 `{{ item }}`，这是一个 Jinja2 关键字变量，它告诉 Ansible 即将遍历一个列表。

+   `为容器初始化系统安装 dumb-init`：此任务利用`get_url`模块从互联网下载远程文件到容器中。在这个特定的情况下，我们下载`dumb-init`二进制文件，并将其放置在容器中的目标位置`/usr/bin/dumb-init`。我们更改文件权限，使其归 root 所有，属于 root 组，并且可以执行。Ansible 允许我们通过一次模块调用执行所有这些操作。

+   `创建 MySQL 用户和组`：接下来的两个任务非常相似。通过这些任务，我们开始为安装 MariaDB MySQL 服务铺设基础。我们调用`user`和`group`模块创建一个名为`mysql`的用户，创建一个同名的组，并将用户添加到该组中。请注意，在组模块中，我们指定了`append: true`。这意味着我们希望将`mysql`组附加到`mysql`用户已经分配的任何组中。这个选项是添加到任何组声明中的安全选择，以确保我们不会意外地将用户从他们可能需要加入的其他组中移除。

+   `安装 MySQL 服务器`：此任务的功能与我们在剧本中看到的第一个任务非常相似。然而，我们并不是安装多个软件包，而是调用 apt 模块只安装一个软件包——MariaDB 服务器。像往常一样，我们希望软件包已安装并存在，并且更新软件包缓存。可以说，我们本可以在第一个任务中将该软件包添加到已安装软件包列表中，这也会起作用。然而，从开发风格上讲，我更喜欢在剧本的步骤之间提供逻辑上的区分，以免之后的操作变得混乱。毕竟，安装基础软件包通常与安装核心服务包是两个不同且独立的步骤。

+   `更改目录权限`：此任务是剧本中较为复杂的任务之一。在这个任务中，我们需要更改一些目录路径的权限，以便 MySQL 服务能够向其中写入数据。`file`模块允许我们创建、删除或修改容器中任何文件。类似于第一个任务，我们将调用文件模块对`{{ item }}`关键字变量进行操作，以便在`with_items`中指定的每个列表项都应用相同的权限和属性。如果指定的路径不存在，Ansible 将以`directory`状态创建这些路径，并应用适当的权限。我们还提供了`recurse: true`选项，这样权限将应用到指定位置的所有子目录。

+   `移除 my.cnf`：`my.cnf`是 MySQL 用来配置数据库服务操作的主要配置文件。当 MariaDB 首次安装时，它会创建一个指向其他配置文件的`my.cnf`符号链接。我们不希望出现这种行为，因此我们将使用文件模块删除默认的`my.cnf`文件，并将状态值设置为`absent`。我们将使用自己的`my.cnf`文件。

+   `安装 MySQL 配置文件`：现在默认的`my.cnf`符号链接已被移除，我们可以调用模板模块，将新的`my.cnf`文件放置在其位置。`templates`模块通过利用本地`templates`目录，并查找与我们指定的源文件名称`my.cnf.j2`匹配的文件来工作。模板使用 Jinja2 模板语言来放置新的配置，并替换任何来自角色的变量。新配置文件的位置将是`/etc/MySQL/my.cnf`，并会应用适当的权限。

+   `初始化数据库`：该剧本中的最终任务被称为`include`任务。`include`语句，顾名思义，会包含其他剧本的 YAML 文件以供执行。通常，`include`语句是将剧本分解成逻辑上分组的相似任务块的好方法。在此场景中，我们希望根据逻辑条件（即变量`initialize_database`设置为 true），包含剧本`initialize_database.yml`。在其他编程语言中，存在如*if, else...if*和*else*的结构，用于表示逻辑评估。Ansible 通过关键字`when`来处理此类逻辑，列出执行某个操作的条件。在本例中，当变量`initialize_database`为 true 时，将执行剧本`initialize_database.yml`；如果变量为 false，则跳过这些任务。

现在我们已经很好地理解了`main.yml`剧本中的任务执行内容，让我们来看一下`initialize_database.yml`剧本中的任务，看看当`initialize_database`变量评估为 true 时会发生什么：

# tasks/initialize_database.yml

该文件位于`roles/mariadb_role/tasks/initialize_database.yml`。

```
---
- name: Temporarily Start MariaDB Server
  shell: MySQLd --user=MySQL &

- name: Create Initial Accounts
  shell: MySQL -e "CREATE USER '{{ default_user }}'@'%' IDENTIFIED BY '{{ default_password }}';"

- name: Grant Privileges to New Account
  shell: MySQL -e "GRANT ALL ON *.* TO '{{ default_user }}'@'%' WITH GRANT OPTION;"

- name: Create Default Databases
  shell: MySQL -e "CREATE DATABASE {{ item }};"
  with_items:
    - "{{ databases }}"

- name: Flush Privileges
  shell: MySQL -e "FLUSH PRIVILEGES;"
```

# 任务细分（initialize_database.yml）

+   `临时启动 MariaDB 服务器`：默认情况下，当 MariaDB 首次安装时，未创建任何数据库，且没有用户访问数据库。在某些情况下，我们可能希望启动一个干净的 MariaDB 服务器，并让外部用户或工具创建默认的数据库和访问凭据。然而，也可能有相同数量的情况，我们需要创建带有内置数据库和用户凭据的数据库实例。为了创建这些默认项，我们首先需要启动 MySQL 服务器，以便能够通过命令行访问它。为了临时启动服务器，我们将调用`shell`模块，该模块以类似于在 Bash 提示符下输入命令的方式执行 shell 命令。我们将运行命令`mysqld`，指定以`mysql`用户身份运行，并使用符号`&`强制服务器在后台运行。此时，MySQL 服务器将继续运行，直到构建完成并且容器被关闭。

+   `创建初始账户`：创建初始账户的步骤类似地调用了 shell 模块，以便利用 MySQL 命令行客户端。`-e`标志允许我们传入可执行的 SQL 命令，这些命令将由服务器进行评估。我们将使用此命令来创建一个默认的用户名和密码，用于登录数据库。默认凭据将从我们的变量中获取，因此使用了双大括号。

+   `授予新账户权限`：再次使用`shell`模块，我们可以调用 MySQL 客户端，授予我们之前创建的新账户权限。在此示例中，我们将授予所有权限，允许从任何网络接口连接并访问该 MySQL 服务器。

+   `创建默认数据库`：通过使用我们的迭代或循环操作符`with_items`，我们可以传入一个我们希望 MySQL 客户端创建的数据库列表。在我们的`defaults/main.yml`文件中，我们将数据库变量指定为一个数组或项列表。Ansible 将识别我们的数据库变量实际上是一个字符串列表，并对其进行迭代。结果是，任何我们指定为数据库变量列表项的数据库都将被迭代并创建在我们的 MySQL 容器中。

+   `刷新权限`：最后一次调用 shell 模块将允许我们执行 SQL 命令`FLUSH PRIVILEGES`，使新用户账户在数据库中生效。此命令执行完毕后，容器构建将完成，通知 Ansible Container 关闭中间容器，并将最终更改提交到我们刚刚构建的容器中。

现在我们已经查看了`tasks`目录并了解了角色如何执行任务，接下来我们看看`templates/`目录，了解我们正在生成并传递到容器中的模板化配置文件。你会发现，在`roles`的 templates 目录中，有一个文件：`my.cnf.j2`。这是我们希望 Ansible 在构建过程中编译并传递到容器中的`my.cnf`文件的模板。最佳实践是始终将你的 Ansible 模板文件命名为目标文件名，并使用`.j2`扩展名。这表示该文件是一个 Jinja2 模板，包含变量和 Jinja2 逻辑，供 Ansible 进行评估。

Jinja2 是一个功能强大的模板语言，可以在你的项目中做很多很酷的事情。虽然不是严格要求，但对 Jinja2 有一定了解会对你的 Ansible 开发大有帮助。你可以在官方网站上了解更多关于 Jinja2 语言的信息：[`jinja.pocoo.org/.`](http://jinja.pocoo.org/)

以下是`templates`目录中`my.cnf.j2`文件的内容：

# templates/my.cnf.j2

该文件位于`roles/mariadb_role/templates/my.cnf.j2`：

```
# Ansible Container Generated MariaDB Config File
[client]
port            = 3306
socket          = /var/lib/MySQL/MySQL.sock

# The MariaDB server
[MySQLd]
user           = MySQL
port            = 3306
socket          = /var/lib/MySQL/MySQL.sock
datadir         = /var/lib/MySQL
bind-address = 0.0.0.0
skip-external-locking
key_buffer_size = {{ key_buffer_size }}
max_allowed_packet = {{ max_allowed_packet }}
table_open_cache = {{ table_open_cache }}
sort_buffer_size = {{ sort_buffer_size }}
```

请注意，在文件的第一行，我们通过注释块明确告诉用户该文件是*Ansible 容器生成的 MariaDB 配置文件*。如果你有连接远程服务器并排查问题的经验，你会知道确切了解文件来源、值的来源以及哪个配置管理工具负责将这些文件放到那里的重要性。虽然这不是严格要求的，当然也是个人喜好的问题，但我喜欢在 Ansible 触及的文件上添加这样的标语。这样，稍后有人查看时，会清楚地知道这个容器是如何以这种状态创建的。

接下来你会注意到，配置文件的最后四行的值被设置为双大括号，其中夹着配置文件的键名。如我们之前讨论的，双大括号表示 Jinja2 变量参数。当 Ansible 在将模板安装到容器中的目标位置之前进行评估时，Ansible 会解析文件中的所有 Jinja2 块，并执行读取到的指令，将模板带到所需的状态。这可能意味着填充变量的值、评估逻辑条件，甚至获取模板所需的环境信息。在这种情况下，Ansible 会看到双大括号并用这些变量的定义值替换它们。通过修改或覆盖变量，Ansible 使得更改容器和应用程序的功能变得非常容易。此外，请注意，变量的名称与它们修改的配置选项相匹配。变量名完全由开发人员决定，因此开发人员可以自由选择变量名。然而，通常的最佳实践是使用描述性强的变量名，以便用户清楚地了解他们正在覆盖或修改哪些设置。

阅读这些角色文件时，你可能已经很清楚，变量与 Ansible 的运行方式、模板的填充方式，甚至任务的执行和控制方式都有很大关系。现在让我们看看变量是如何在角色中定义的，以及我们如何利用变量使角色更加灵活，并使其能够复用。如前所述，角色变量可以存储在两个地方：`defaults/` 目录或 `vars/` 目录。作为开发人员，你可以选择将变量存储在哪个位置（或者两个位置）。唯一的区别在于变量的优先级顺序，这决定了变量的评估顺序。存储在 `defaults/` 目录中的变量最容易被覆盖，而存储在 `vars/` 目录中的变量优先级稍低，因此更难覆盖。在这个例子中，我选择将所有变量存储在 `defaults/` 目录中的 `main.yml` 文件里。让我们看看这个文件长什么样：

```
---
# defaults file for MySQL_role
initialize_database: true
default_user: "root"
default_password: "password"
databases:
  - "TestDB1"
  - "TestDB2"
  - "TestDB3"

#MySQL Basic Tuning for my.cnf
key_buffer_size: "16K"
max_allowed_packet: "1M"
table_open_cache: 4
sort_buffer_size: "64K"
```

在这里，你可以看到这些都是我们之前见过的变量，它们在角色任务以及 `my.cnf` 模板文件中都有引用。变量 YAML 文件本质上只是使用我们在章节开始时探索的相同 YAML 结构的静态 YAML 文件。例如，默认情况下，该文件通过将 `initialize_database` 变量设置为布尔值 `true` 来初始化数据库。我们还可以看到，数据库中将创建的默认凭据设置为字符串 `root` 和 `password`，以及在初始化数据库任务期间将创建的测试数据库列表。最后，在文件底部，我们有一组变量，用于定义将被纳入模板的值。如果我们按原样构建角色，而不提供任何变量覆盖，我们将得到一个完全符合这些规格的容器。然而，如果不探索如何定制我们刚刚编写的角色，这本书将不完整！

# 构建容器启用角色

在我们开始定制角色之前，让我们先构建该角色并演示使用我们指定的默认变量的默认功能。我们继续回到我们的 Ansible 容器工作流程，执行 `ansible-container build`，然后在项目的 `root` 目录下执行 `ansible-container run` 命令：

```
ubuntu@node01:$ ansible-container build
Building Docker Engine context...                                                                                                                                                                            
Starting Docker build of Ansible Container Conductor image (please be patient)...                                                                                                                                                                                      
Parsing conductor CLI args.                                                                                                                                                                                  
Docker™ daemon integration engine loaded. Build starting.       project=mariadb_demo                                                                                                                         
Building service...     project=mariadb_demo service=MySQL_database_container                                                                                                                                

PLAY [MySQL_database_container] ************************************************                                                                                                                             

TASK [Gathering Facts] *********************************************************                                                                                                                             
ok: [MySQL_database_container]                                                                                                                                                                               

TASK [mariadb_role : Install Base Packages] ************************************                                                                                                                             
changed: [MySQL_database_container] => (item=[u'ca-certificates', u'apt-utils'])                                                                                                                             

TASK [mariadb_role : Install dumb-init for Container Init System] **************                                                                                                                             
changed: [MySQL_database_container]

TASK [mariadb_role : Create MySQL Group] ***************************************
changed: [MySQL_database_container]

TASK [mariadb_role : Create MySQL Users] ***************************************
changed: [MySQL_database_container]

TASK [mariadb_role : Install MySQL Server] *************************************
changed: [MySQL_database_container]

TRUNCATED
```

你可能会注意到从构建输出中，Ansible 正在使用 `with_items` 迭代运算符将我们在任务中提供的列表项准确地构建成镜像，并根据我们在角色中提供的变量将其带入所需状态，暂时来说，这些是默认变量。

让我们运行我们的项目并尝试访问 MySQL 服务：

```
ubuntu@node01:$ ansible-container run
Parsing conductor CLI args.
Engine integration loaded. Preparing run.       engine=Docker™ daemon
Verifying service image service=MySQL_database_container

PLAY [localhost] ***************************************************************

TASK [docker_service] **********************************************************
changed: [localhost]

PLAY RECAP *********************************************************************
localhost                  : ok=1    changed=1    unreachable=0    failed=0

All services running.   playbook_rc=0
Conductor terminated. Cleaning up.      command_rc=0 conductor_id=e33fb25670f1bc040a6edd19360ec528be28b9f14d4ccb0d9a8ed34a71d1c561 save_container=False
```

执行 `docker ps -a` 会显示我们的容器正在运行，并且在主机上暴露了端口 `3306`：

```
ubuntu@node01:~$ docker ps -a
CONTAINER ID  IMAGE  COMMAND  CREATED  STATUS  PORTS  NAMES
7cab59c33cfa  mariadb_demo-MySQL_database_container  "/usr/bin/dumb-ini…"  About a minute ago   Up About a minute   0.0.0.0:3306->3306/tcp   mariadb_demo-MySQL_database_container
```

为了测试确保一切正常，我们可以下载并安装 `mariadb-client` 包，或者选择任何你喜欢的 MySQL 客户端：

```
ubuntu@node01:~$ sudo apt-get install -y mariadb-client
```

安装 MariaDB 客户端后，你可以使用以下命令连接到暴露在 Vagrant 虚拟机 localhost 上的 MariaDB 容器。如果你不熟悉 MySQL 客户端，请记住，传递给客户端的所有标志后面不带空格。这看起来有点奇怪，但它应该会把你带入 MySQL 控制台：

```
ubuntu@node01:~$ MySQL -h127.0.0.1 -uroot -ppassword
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 2
Server version: 10.0.31-MariaDB-0ubuntu0.16.04.2 Ubuntu 16.04

Copyright (c) 2000, 2017, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> 
```

让我们运行 `show databases;` 命令，看看我们在默认变量中指定的测试数据库是否已经创建：

```
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| TestDB1            |
| TestDB2            |
| TestDB3            |
| information_schema |
| MySQL              |
| performance_schema |
+--------------------+
6 rows in set (0.00 sec)

MariaDB [(none)]>
```

看起来一切都已经正确创建并按预期工作。当你完成此会话工作时，可以使用 `exit` 命令退出 MySQL CLI。使用 `ansible-container destroy` 来重置你的环境。让我们通过定制我们的角色并引入外部变量值来让事情变得有趣。

# 定制容器启用角色

正如我们在上一章看到的那样，通过将变量直接添加到项目的`container.yml`文件并重新构建项目，抽象掉 Ansible 容器项目中的变化非常简单。这提供了将所有配置更改集中在一个位置的额外便利，并有效地作为单一的真相来源。对于某些用例来说，这可能已经足够，但如果需要为不同的环境或位置（例如开发、测试、QA 和生产环境）提供配置不同的容器呢？你可以简单地更新`container.yml`文件，并为这些场景构建不同的镜像。然而，Ansible Container 提供了更好的处理方式，允许我们通过外部变量文件来管理这一切。`ansible-container`父命令的一部分是`--var-files`标志，它允许为变量定义提供外部 YAML 文件的选项。这为我们提供了一个抽象，允许不同配置选项的独立构建并行运行。这还允许我们使用不同的变量文件来定制我们的角色，几乎可以针对任何能够与项目一起版本控制的情况。

为了启用此功能，让我们在项目根目录（与项目特定的`container.yml`同级）创建一个名为`variable_files`的目录。在此目录内，我们将创建三个独立的文件：`dev.yml`、`test.yml`和`prod.yml`，它们具有略有不同的配置选项。以下是这三个文件的示例。希望你喜欢我的《星际迷航》引用！

在我们开始之前，最好先执行`ansible-container destroy`操作，然后使用不同的变量重新构建容器。这样，你可以清楚地看到在构建过程中究竟发生了什么变化。

在开发环境中，我们数据库的主要用户将是叶曼·兰德。她将主要关注星际舰队数据：

# variable_files/dev.yml

该文件位于`<project_root>/variable_files/dev.yml`：

```
---
# Development Defaults for MySQL role
initialize_database: true
default_user: "yeoman_rand"
default_password: "starfleet"
databases:
  - "starfleet_data"
```

在`系统测试`中，斯波克先生将是我们数据库的主要用户。他对与火山星、舰船条例、穿梭机以及联邦数据相关的数据更感兴趣。

# variable_files/test.yml

该文件位于`<project_root>/variable_files/test.yml`：

```
---
# System Test Defaults for MySQL role
initialize_database: true
default_user: "MrSpock"
default_password: "theBridge"
databases:
  - "planet_vulcan"
  - "federation_data"
  - "shuttle_crafts"
  - "ship_ordinances"
```

# variable_files/prod.yml

在生产环境中，柯克舰长将需要存储与其他船员完全不同的数据。我们需要稍微增强我们的 MySQL 配置，以支持存储舰长日志、企业数据以及联邦指令所增加的负载。该文件位于：`<project_root>/variable_files/prod.yml`：

```
---
# defaults file for MySQL_role
initialize_database: true
default_user: "captainkirk"
default_password: "ussenterprise"
databases:
  - "CaptainsLog"
  - "USS_Enterprise"
  - "InterstellarColonies"
  - "FederationMandates"

#MySQL Basic Tuning for my.cnf
key_buffer_size: "128K"
max_allowed_packet: "20M"
table_open_cache: 12
sort_buffer_size: "128K"
```

你可能还会注意到，并不是所有的变量在这里显示的每个示例中都被覆盖。在变量没有被源文件覆盖的情况下，Ansible 将使用角色中的 `defaults/main.yml` 中的值。重要的是，您的角色默认值必须为所有变量提供值，因为没有值的变量会导致构建过程失败。

变量文件的名称可以是你想要的任何名称。由于我们在构建过程中引用这些文件，并且它们不是 Ansible Container 会自动发现的文件，所以命名规则完全由你决定。

我们可以通过执行 `ansible-container build` 命令，并将 `--vars-files` 标志作为 `ansible-container` 命令的参数来基于这些变量中的任何一个构建容器。记住，我们始终在与项目特定的 `container.yml` 文件位于同一目录下运行 `build` 命令：

```
ansible-container --vars-files variable_files/dev.yml build
```

在构建过程中，你应该注意到，根据我们提供的变量，许多任务的执行方式略有不同。例如，当引用开发变量时，你会看到只创建了一个数据库：`starfleet_data`。这表明新的变量已经被正确引用并在构建过程中正确填充。现在让我们执行一个新的容器版本的 `ansible-container run`，并尝试使用与之前相同的凭证登录：

```
ubuntu@node01:$ ansible-container run
Parsing conductor CLI args.
Engine integration loaded. Preparing run.       engine=Docker™ daemon
Verifying service image service=MySQL_database_container

PLAY [Deploy mariadb_demo] *****************************************************

TASK [docker_service] **********************************************************
changed: [localhost]

PLAY RECAP *********************************************************************
localhost                  : ok=1    changed=1    unreachable=0    failed=0

All services running.   playbook_rc=0
Conductor terminated. Cleaning up.      command_rc=0 conductor_id=bf5221a710736d238b0995dd4ce42b57bcb9338131225d71c0d7d1b3cef85677 save_container=False
```

现在，要使用 MariaDB 客户端登录：

```
ubuntu@node01:$ MySQL -h127.0.0.1 -uroot -ppassword
ERROR 1698 (28000): Access denied for user 'root'@'172.18.0.1'
```

很明显，我们在角色默认值中设置的默认凭证不再有效。让我们再试一次，使用我们在开发变量文件中为 Yeoman Rand 用户指定的凭证：

```
ubuntu@node01:$ MySQL -h127.0.0.1 -uyeoman_rand -pstarfleet
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 3
Server version: 10.0.31-MariaDB-0ubuntu0.16.04.2 Ubuntu 16.04

Copyright (c) 2000, 2017, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]>
```

看起来我们的新容器使用开发环境的源变量文件工作正常。让我们运行 `show databases;` 命令，确保数据库已正确创建并存在：

```
MariaDB [(none)]> show databases;
'+--------------------+
| Database           |
+--------------------+
| information_schema |
| MySQL              |
| performance_schema |
| starfleet_data     |
+--------------------+
4 rows in set (0.04 sec)

MariaDB [(none)]>
```

如你所见，数据库 `starfleet_data` 存在，并与 MariaDB 默认的数据库（如 `information_schema`、`MySQL` 和 `performance_schema`）一起显示。这表明容器已正确构建，并且已准备好在我们的开发环境中部署（以本示例为目的）。现在我们可以将镜像推送到我们选择的容器注册表。在这个例子中，我将在项目特定的 `container.yml` 文件的 `registries` 部分添加 Docker Hub，指定命名空间为我的 Docker Hub 用户名（记得在 registries 段落开始时删除大括号）。保存该文件后，让我们将镜像标记为 `dev` 并将其推送到我们的 Docker Hub 仓库，以便我们有一个可以用来部署应用程序的构建镜像工件：

# container.yml

项目特定的 `container.yml` 文件位于项目的 `root` 目录中：

```
registries:
  docker:
    url: https://index.docker.io/v1/
    namespace: username
```

使用 `--push-to` 标志推送镜像：

```
ubuntu@node01:$ ansible-container push --push-to docker --tag dev
Parsing conductor CLI args.
Engine integration loaded. Preparing push.      engine=Docker™ daemon
Tagging aric49/mariadb_demo-MySQL_database_container
Pushing aric49/mariadb_demo-MySQL_database_container:dev...
The push refers to a repository [docker.io/aric49/mariadb_demo-MySQL_database_container]
Preparing
Waiting
Pushing
Pushed
Pushing
Pushed
Pushing
Pushed
dev: digest: sha256:98d288cfa09acc3f06578532cd6ccd78af0eb65b84ba3b0ee011105e59cfb588 size: 1569
Conductor terminated. Cleaning up.      command_rc=0 conductor_id=323116464d8f687238ae6ab64f86fb54746b6c2f5f0b895754da6ef0ce540d76 save_container=False
```

在`container.yml`文件中配置 Docker Hub 作为镜像仓库并不是完全必要的，因为 Ansible Container 默认会使用 Docker Hub。不过，我喜欢确保不会不小心将镜像推送到错误的仓库，因此最佳实践是始终在`container.yml`文件中提供镜像仓库，并始终使用`-–push-to`标志命令来指定正确的仓库进行推送。

我们可以对`test.yml`配置文件以及`prod.yml`配置文件执行相同的构建过程，并将它们推送到 Docker Hub 仓库（记得在构建之间执行`destroy`操作）。注意，在上传不同版本的镜像时，Docker 会自动识别与先前上传版本相同的镜像层。在这种情况下，Docker 会通过不推送相同的镜像层，仅推送已更改的层，帮助你节省带宽和资源，如下所示。请注意`Layer already exists`行：

```
ubuntu@node01:$ ansible-container push --push-to docker --tag test
Parsing conductor CLI args.
Engine integration loaded. Preparing push.      engine=Docker™ daemon
Tagging aric49/mariadb_demo-MySQL_database_container
Pushing aric49/mariadb_demo-MySQL_database_container:test...
The push refers to a repository [docker.io/aric49/mariadb_demo-MySQL_database_container]
Preparing
Waiting
Layer already exists
Pushing
Layer already exists
Pushing
Layer already exists
Pushing
Pushed
test: digest: sha256:1f9604585e50efe360a927f0a5d6614bb960b109ad8060fa3173d4ab259ee904 size: 1569
Conductor terminated. Cleaning up.      command_rc=0 conductor_id=ef42185fab1cfe20de65101059957844bbc4a166a8ab10352fe1b796e7ea5c3d save_container=False
```

现在我们应该有三个不同的容器镜像可供下载。这些镜像可以在我们假想的开发实验室、系统测试实验室以及生产环境中下载和部署。这些容器镜像保证在这些环境中以与我们本地工作站完全相同的方式运行。此时，我们可以进行最终的操作，并在不同的端口上运行这三个容器，以模拟这些容器在不同环境中、使用不同配置的运行方式。为了快速演示，我们将使用本地的`docker run`命令来指定我们打标签的镜像以及容器服务要使用的端口；我们还指定我们的服务应该使用`-d`标志在后台运行。请注意，我们正在创建的每个容器实例都使用`dev`、`test`和`prod`标签以及我们的用户仓库地址。在我的例子中是`aric49`：

```
docker run -d --name MySQL_Dev -p 3308:3306 aric49/mariadb_demo-MySQL_database_container:dev

docker run -d --name MySQL_Test -p 3309:3306 aric49/mariadb_demo-MySQL_database_container:test

docker run -d --name MySQL_Prod -p 33010:3306 aric49/mariadb_demo-MySQL_database_container:prod
```

测试容器功能的过程与之前完全相同。我们可以使用 MariaDB 客户端登录到我们容器的一个实例。不过这次，我们需要指定我们的服务监听的端口，因为所有三个实例不能在主机网络端口`3306`上监听。如果我们想登录到生产容器，我们可以使用`33010`端口和`ussenterprise`密码为 Captain Kirk 指定凭据：

```
ubuntu@node01:$ MySQL -h127.0.0.1 -ucaptainkirk -pussenterprise -P33010
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 2
Server version: 10.0.31-MariaDB-0ubuntu0.16.04.2 Ubuntu 16.04

Copyright (c) 2000, 2017, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]>
```

利用相同的 Ansible 容器项目和容器启用角色，我们能够使用 Ansible 容器默认的原语构建多种配置的容器，这些容器可以在不同的环境和用例中使用。这种方法使我们能够确保构建过程在未来的所有构建迭代中保持完全相同，但我们也能灵活地为我们的角色提供新的配置值，而无需修改之前编写的代码。通过使用容器标签，可以捕获容器配置的快照并与其他用户共享。现在，我们拥有了一个极其有用且可重复的流水线，确保未来版本的应用程序容器可以追溯到用于生成它们的源角色。即使容器镜像不小心从镜像注册中心被删除，我们也可以随时轻松构建和重建我们的容器，因为所有容器中的配置都是以代码形式声明的，使用的是 Ansible playbook 语言。如果你曾在 IT 相关的 DevOps 或系统管理员岗位工作过很长时间，你会明白，拥有这种级别的基础设施洞察力是多么宝贵。

# 参考资料

+   **官方 YAML 标准指南**: [`yaml.org`](http://yaml.org)

+   **Ansible 模块索引**: [`docs.ansible.com/ansible/latest/modules_by_category.html`](http://docs.ansible.com/ansible/latest/modules_by_category.html)

+   **Ansible Playbook 规范**: [`docs.ansible.com/ansible/latest/playbooks.html`](http://docs.ansible.com/ansible/latest/playbooks.html)

# 概述

在本章中，我们探讨了角色如何不仅使 Ansible 容器中的重用成为可能，而且实际上是 Ansible 容器成为一个强大工具的核心，帮助构建和管理容器。

我们首先了解了如何使用 Ansible Galaxy 命令行工具创建容器启用角色的基础框架，包括所有必需的目录和默认的 YAML 文件，从中我们可以构建我们的角色。接着，我们编写了一个自定义角色，使用合理的默认配置选项构建 MariaDB 容器。最后，我们在角色上方开发了一个抽象层，通过传递自定义的变量配置选项，使我们能够在不修改任何代码的情况下定制我们的容器项目。

我希望本章能够有效展示使用 Ansible Container 项目所能提供的强大功能。到目前为止，我认为很容易做出假设，认为使用 Dockerfiles 构建容器镜像会更简单，而不必担心 Ansible Container 带来的额外开销。希望你现在能够理解，使用 Ansible Container 的好处远远超过了所需的少许额外复杂性。通过使用 Ansible Container，你可以创建一个强大的管道，用于构建、运行、测试和推送容器镜像。通过利用易于理解的 Ansible playbook 语法语言，我们有了一个基础，可以开始构建现代化、敏捷的容器化基础设施，这使得我们能够快速部署变更，并真正开始拥抱真正模块化基础设施的承诺。

现在我们了解了如何构建和部署真正自定义的容器，我们可以开始了解 Kubernetes，一个开源框架，用于自动化容器的大规模部署、编排和管理。
