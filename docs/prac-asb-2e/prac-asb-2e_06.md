

# 第六章：创建和使用集合

如果你之前熟悉 Ansible 2.9 之前的版本，那么到目前为止，这本书的内容对你来说应该非常熟悉。如果你是 Ansible 自动化的新手，那么，当然，所有这些内容都会显得崭新且充满吸引力。无论你到目前为止对 Ansible 的了解如何，本书都不可能在没有深入探讨集合（collections）的情况下完成。集合是解决 Ansible 在自身流行和成功过程中出现的一些问题的方案，而它们现在已经成为每个现代 Ansible 安装的核心。无论你是否意识到，你在本书中至今一直在使用它们，而且它们将会继续存在，因此我们有必要深入了解它们。

在本章中，我们将深入探讨集合，详细了解它们解决的问题以及它们是如何产生的，然后再从技术角度深入了解它们的结构和组成。最后，我们将通过一个动手实践的示例来创建你自己的集合，这样你就能全面理解它们是如何创建、构建、维护和使用的。因此，你将能够轻松地使用这个强大的新功能，并在自己的自动化解决方案中充分利用集合。

具体来说，本章将涵盖以下主题：

+   Ansible 集合简介

+   理解完全限定的集合名称

+   在控制节点上管理集合

+   更新你的 Ansible 集合和核心安装

+   创建你自己的集合

# 技术要求

本章假设你已经按照 *第一章*《*Ansible 入门*》中的详细说明，设置好了你的控制主机，并且正在使用最新版本——本章中的示例是使用 Ansible 8.0 和 `ansible-core` 2.15 进行测试的。本章还假设你至少有一个额外的主机进行测试，并且该主机应为基于 Linux 的。虽然我们将在本章中给出具体的主机名示例，你可以自由地用自己的主机名和/或 IP 地址替代，具体操作方法将在相应位置提供。

本章的代码包可以在这里找到： [`github.com/PacktPublishing/Practical-Ansible-Second-Edition/tree/main/Chapter 6`](https://github.com/PacktPublishing/Practical-Ansible-Second-Edition/tree/main/Chapter%206)

# Ansible 集合简介

虽然我们在本书前面已经提到了一些内容，但我们的目标是将其作为关于集合的完整一站式信息源——因此，如果你直接跳到本章，请不用担心——我们已经为你准备好了一切。让我们从一些历史背景开始，因为这对理解集合背后的意图非常重要。

在 2.9 版本之前，Ansible 的所有内容都被管理在一个庞大的单体代码库中。虽然核心的 Ansible 团队拥有这个代码库，但真正构成 Ansible 命脉的模块（毕竟，它们使 Ansible 能够轻松地自动化如此多不同的系统）并不属于他们。如果一个网络设备供应商想要发布一个新模块，也许他们添加了一个新特性，或者修复了先前版本中的一个 bug，他们必须进行这些更改、进行测试，然后将其作为拉取请求提交到 Ansible 的主仓库——这不仅是一个重大的任务，而且根据**拉取请求**（**PRs**）的数量以及要合并的代码量，它可能还需要很长时间，从而拖慢发布周期。

在这一点上，仓库的所有者必须合并代码、自己进行测试并进行发布。这种工作方式在 Ansible 的早期是可以的，但随着其采用度的提高，这种方式变得无法扩展。试想一下，如果一个大型企业用户在使用 Ansible 模块时遇到问题并且需要立即修复，或者供应商有一个令人兴奋的新发布，并希望从第一天开始就支持 Ansible，这两种情况都是在这种工作方式下不可能实现的。再加上 Ansible 2.8 版本已经有成千上万的模块，管理 Ansible 代码的任务变成了一项令人头疼的工作。

当然，这个假设前提是代码有资格被集成到主代码库中。如果某些内容是机密性质的，或者需要迅速分发以修复问题，那么就没有简单的方法将其集成到 Ansible 安装中（明确来说，虽然有一些方法，但它们并不像集合所提供的这种简单直接）。

正是出于这些需求，集合应运而生。简而言之，集合赋予了各个团队和供应商按照自己的节奏独立开发、测试和发布对 Ansible 的贡献，而无需依赖核心 Ansible 代码的发布周期。具体来说，集合提供了一种机制来打包和分发以下内容：

+   角色

+   插件

+   模块

+   Playbooks

+   文档

与 Ansible 的所有方面一样，集合的设计简单且易于操作。集合只是一个包含所需功能的文件的目录集合。这些文件随后被打包成一个 gzip 压缩的 tar 包（一个广为人知并被理解的文件格式），以便于分发。集合可以在本地安装和管理（稍后我们将在*管理控制节点上的集合*部分中看到），也可以通过 Ansible Galaxy 进行管理（[`galaxy.ansible.com`](https://galaxy.ansible.com)）。

任何熟悉 Ansible 早期版本的人都会知道，角色是通过 Ansible Galaxy 分发的，并且可以使用 `ansible-galaxy` 命令行工具轻松管理。因此，一个有效的问题可能是，如果你开发了自己的角色，应该将其作为角色分发，还是将其作为集合中的角色进行分发？毕竟，目前这两种方式都是可行的，并且都可以通过 Ansible Galaxy 分发。虽然这两种方式在目前都是完全可行的，但作者认为，随着时间的推移，角色将会作为集合分发，因为集合提供了更大的扩展空间——例如，如果你需要开发一个插件来扩展角色的功能，你可以将其添加到你已经创建的集合中。另一方面，作为单独的角色分发将需要以后迁移到集合，因此，通过一开始就将其打包为集合，你可以为未来做好准备。

开发者、供应商和企业可以非常轻松地将他们的 Ansible 功能打包成集合，进行打包和分发。如果他们愿意，他们还可以将其贡献回社区（在开源软件的精神下，这无疑是目标），但如果它们包含机密信息，它们同样可以轻松地托管并维护在内部仓库中。

通过这个介绍，你已经对集合及其预期用途有了一个很好的了解。接下来，我们将进入本章的下一部分，开始探讨创建和使用集合的实际方面，以进一步加深你的理解。

# 理解完全限定的集合名称

在集合出现之前，任何创建并贡献给 Ansible 的模块都必须具有唯一的名称。因此，我们经常会看到类似下面这样的模块名称（这两个模块名称来自 Ansible 2.8 版本）：

+   `ios_bgp`

+   `eos_bgp`

+   `fortios_router_bgp`

这三个模块都是用来修改 `bgp`，实现以下功能：

+   确保它们的名称是唯一的

+   确保 Ansible 编码人员能够理解他们的代码做了什么

集合取消了对唯一模块名称的需求，因此现在贡献者可以创建名称重复的模块。这一点非常有价值，因为它消除了冗长模块名称的需求，但也带来了意外代码行为的风险。例如，我们经常使用 `debug` 模块来理解我们的 playbook 代码在做什么，并在执行过程中打印某些内容。我们完全可能会创建一个名为 `debug` 的模块，它做一些稍微不同（或完全不同）的事情。试想，如果你的 playbook 在不同的控制节点上表现不同，或者根本无法工作，结果将会是多么混乱。

正因如此，在本书中，早期版本的 Ansible 代码中，你会习惯看到如下的代码：

```
  - name: Print some debug output
    debug:
      msg: "Hello World!"
```

这种方式今天在 Ansible 中仍然有效（你可以自己测试！），并且为了向后兼容，短格式模块名称的支持仍然保留，以便你现有的任何遗留代码都可以运行。虽然这对让用户在不需要控制节点上安装多个版本的 Ansible 的情况下，按自己的节奏升级 Ansible 代码非常有价值，但由于模块名称冲突导致的意外行为的风险也不容小觑，因此尽早开始使用完全限定集合名称非常重要。

例如，在之前提供的示例代码中，我们现在会写出以下内容：

```
  - name: Print some debug output
    ansible.builtin.debug:
      msg: "Hello World!"
```

这段代码的功能是相同的，但现在模块名称冲突的风险已经消除，因为已经指定了完全限定集合名称。

那么，**完全限定集合名称**（**FQCN**）究竟是什么呢？最好的解释方式是将其拆解为组成部分，如下所示：

```
<namespace>.<collection_name>.<module_name>
```

让我们从命名空间开始——这是一个唯一的命名空间，用于标识集合的开发者。如果你是个人开发者，它可能是你的名字；如果你是为供应商贡献代码，它则是公司的名字。所有命名空间必须唯一，这作为一种限制是合理的——当你查看 Ansible Galaxy 时，你希望知道 `cisco` 命名空间下的所有模块都由 Cisco 管理，这样你就可以确定是谁拥有和管理这些代码。同样，`ansible` 命名空间（我们从中使用了 `ansible.builtin.debug`）由 Ansible 项目本身拥有和管理。

现在，我们也可以安全地假设每个命名空间将有一个或多个集合。将集合按功能划分是有意义的；否则，我们又会回到本章前面讨论的单体代码库管理的问题。例如，在 `cisco` 命名空间下，有一个 `ios` 集合用于管理 Cisco IOS 设备，还有一个单独的 `asa` 集合用于管理他们的 `asa`，但我不能把它放在 Ansible Galaxy 上的 `cisco` 命名空间中。

FQCN 的第三部分和最后一部分是模块名称本身。它与 Ansible 中一直以来的功能相同，并且在一个集合中必须是唯一的。在本书中，我们已经多次使用过模块，并且我们将继续使用；因此，我们在这里假设你已经理解了 Ansible 模块的概念。

把这一切放在一起，例如在 Ansible 2.8（或更早版本）中，你可能会在 Cisco IOS 设备上使用`ios_bgp`模块进行 BGP 配置，而现在这个模块的完全限定类名（FQCN）是`cisco.ios.ios_bgp`。关于这个 FQCN，你可能会问的一个合理问题是，为什么在其独特的命名空间和集合名称内，模块仍然被称为`ios_bgp`而不是简单地称为`bgp`？答案在于向后兼容性——Ansible 8.0 仍然支持为 2.8 及更早版本编写的 playbook，并且支持未完全限定的模块名称。因此，如果 Cisco 将模块名称更改为`bgp`，它们将破坏向后兼容性。

尽管有这种向后兼容性功能，尽快开始与 FQCN 结交是很重要的。模块名称在集合内必须是唯一的，但这是唯一的限制。没有任何阻止我创建自己的模块，例如在我的`ios`集合中创建名为`ios_bgp`的模块，并将其称为`practicalansible.ios.ios_bgp`。在这种情况下，如果你在 playbooks 和 roles 中只通过名称`ios_bgp`指定模块，那么你无法保证调用的是哪个模块，因此使用 FQCN 是至关重要的，以确保你不会遇到任何意外或错误的行为。

尽管这些关于 FQCN 的概念很简单，它们是基础知识，我们将在本章的进程中依赖这些理解，因此尽早搞清楚它们是很重要的。现在我们已经做到了这一点，接下来我们将在下一节中看看如何管理你的控制节点上的集合。

# 在你的控制节点上管理集合

正如我们在*第一章*中讨论的，*开始使用 Ansible*，当你安装 Ansible 时，实际上安装了一组集合，提供了与最新的 2.x 版本相当的功能，以保持向后兼容性（当然还有`ansible-core`）。这个过程对用户来说是不可见的，同时随着你更新 Ansible 的安装，伴随的集合也会得到更新。

鉴于此，你可能会想知道为什么需要学习如何管理集合——毕竟，Ansible 默认提供了大量集合，并且随着安装更新，集合也会更新。然而，这正是集合的美妙之处——如果它们确实可以满足你的需求，那么你就不需要采取进一步的行动。相反，如果你确实需要扩展你的 Ansible 控制节点的功能，你可以做到这一点——力量在于选择和灵活性，正如 Ansible 一直以来的特点。

为了帮助我们理解如何管理我们的集合，让我们从安装它们的位置开始看起。就像 Ansible 中的一切一样，集合安装的路径是可配置的，你可以在任何你喜欢的地方安装集合（当然，前提是你已经设置了正确的设置）。

如果我们登录到我的演示节点，在该节点上我使用`pip3`在我的本地用户帐户下安装了 Ansible，我可以按如下方式查询 Ansible 集合路径：

```
$ ansible-config dump | grep COLLECTIONS_PATHS
COLLECTIONS_PATHS(default) = ['/home/james/.ansible/collections', '/usr/share/ansible/collections']
```

你可以看到的是，`COLLECTIONS_PATHS`环境变量未设置（输出中显示为`(default)`），因此它默认为两个已知位置：

+   `~/.``ansible/collections`

+   `/``usr/share/ansible/collections`

集合路径按指定的顺序进行搜索，因此，我的主目录下的路径会在`/usr/share`下的路径之前被搜索。你可以通过设置我们之前查询过的环境变量来设置你的集合位置，或者你也可以在`ansible.cfg`文件的`[default]`部分创建一个条目，使用`collections_paths`键。更多信息，请参考这个链接：[`docs.ansible.com/ansible/latest/reference_appendices/config.xhtml#collections-paths`](https://docs.ansible.com/ansible/latest/reference_appendices/config.xhtml#collections-paths)。

在我的演示节点上，我到目前为止还没有对集合进行任何操作，因此，如果我尝试列出之前的路径，我会发现这两个路径都不存在。不过，让我们试着从用户`davidban77`安装`gns3`集合，看看会发生什么 ([`galaxy.ansible.com/davidban77/gns3`](https://galaxy.ansible.com/davidban77/gns3))：

```
$ ansible-galaxy collection install davidban77.gns3
Starting galaxy collection install process
Process install dependency map
Starting collection install process
Downloading https://galaxy.ansible.com/download/davidban77-gns3-1.5.0.tar.gz to /home/james/.ansible/tmp/ansible-local-919n6_zrnbj/tmpgf0fuin6/davidban77-gns3-1.5.0-hlfzoaen
Installing 'davidban77.gns3:1.5.0' to '/home/james/.ansible/collections/ansible_collections/davidban77/gns3'
davidban77.gns3:1.5.0 was installed successfully
```

在这里，我们可以看到集合已经成功安装，而且`ansible-galaxy`工具甚至告诉我们安装的位置。然而，请注意，它仅为我的本地用户帐户安装了该集合，因此如果本机上的另一个用户想要使用此集合，他们必须安装自己的副本。这是可以的，但如果他们安装了不同的版本（例如，他们安装了同一个集合，但安装时间较晚，更新后版本不同），这可能会在以后引发问题。

然而，我们可以绕过这个问题。第一种方法是集中安装所需的集合，以便所有用户都可以访问它们。我们知道，默认情况下，`COLLECTIONS_PATHS`变量会搜索`/usr/share/ansible/collections`，并且所有用户都可以访问它，因此需要全局可用的集合可以安装在这里。默认情况下路径并不存在，但你可以通过运行以下命令轻松创建它并在此安装集合：

```
$ sudo mkdir -p /usr/share/ansible/collections
$ sudo chmod a+w /usr/share/ansible/collections/
$ ansible-galaxy collection install -p /usr/share/ansible/collections davidban77.gns3
Starting galaxy collection install process
Process install dependency map
Starting collection install process
Downloading https://galaxy.ansible.com/download/davidban77-gns3-1.5.0.tar.gz to /home/james/.ansible/tmp/ansible-local-1036ifclq7yb/tmpi34u42w5/davidban77-gns3-1.5.0-nvsahsw0
Installing 'davidban77.gns3:1.5.0' to '/usr/share/ansible/collections/ansible_collections/davidban77/gns3'
davidban77.gns3:1.5.0 was installed successfully
```

请注意，使用的`chmod`命令是一种粗暴的方法，用来授予所有用户在此共享目录中安装集合的权限。建议你根据环境设置适当的访问控制，但由于这在不同场景下有所不同，具体操作留给你自行完成。

你现在会注意到，我们在之前运行的命令中添加了`-p`标志，告诉`ansible-galaxy`将我们的集合安装到与默认位置不同的地方（默认位置是`COLLECTIONS_PATHS`变量中的第一个条目）。因此，我们已经成功地将该集合安装到所有用户可以使用的中央位置。

我们可以使用以下命令来验证这一点（为便于阅读，输出已截断）：

```
$ ansible-galaxy collection list
# /home/james/.local/lib/python3.10/site-packages/ansible_collections
Collection                    Version
----------------------------- -------
amazon.aws                    5.4.0
ansible.netcommon             4.1.0
…
# /usr/share/ansible/collections/ansible_collections
Collection      Version
--------------- -------
davidban77.gns3 1.5.0
# /home/james/.ansible/collections/ansible_collections
Collection      Version
--------------- -------
davidban77.gns3 1.5.0
```

注意，命令的输出列出了每个已知位置中的集合，这些位置包括`COLLECTIONS_PATHS`变量列出的所有路径，还包括 Ansible 的安装位置（它排在前面，并且在配置中是隐式的——你无需显式指定）。

集合没有卸载选项，但 Ansible 的魅力一直在于它的简单性，集合就像角色一样，只是一个具有已知结构的目录和文件集（稍后本章会详细介绍）。因此，如果我们想卸载集中式可用的`davidban77.gns3`集合，只需运行以下命令：

```
$ rm -rf /usr/share/ansible/collections/ansible_collections/davidban77/gns3/
```

如果你重新运行集合列出命令，你会看到该集合不再出现在输出中。那么，如果你想安装特定版本的集合该怎么办呢？假设你想安装我们一直在测试的集合的`1.4.0`版本——你只需运行以下命令进行安装：

```
$ ansible-galaxy collection install davidban77.gns3:1.4.0
Starting galaxy collection install process
Process install dependency map
Starting collection install process
Downloading https://galaxy.ansible.com/download/davidban77-gns3-1.4.0.tar.gz to /home/james/.ansible/tmp/ansible-local-1081bdunqhfz/tmp41a0hc0l/davidban77-gns3-1.4.0-cg29hr0y
Installing 'davidban77.gns3:1.4.0' to '/home/james/.ansible/collections/ansible_collections/davidban77/gns3'
davidban77.gns3:1.4.0 was installed successfully
```

在这里，我们成功安装了该集合的版本`1.4.0`，并且它覆盖了我们之前安装的版本`1.5.0`。升级集合非常简单，要么指定最新版本来替代我们之前使用的版本，要么强制重新安装（其本质上会安装集合的最新版本）：

```
$ ansible-galaxy collection install --force davidban77.gns3
Starting galaxy collection install process
Process install dependency map
Starting collection install process
Downloading https://galaxy.ansible.com/download/davidban77-gns3-1.5.0.tar.gz to /home/james/.ansible/tmp/ansible-local-11688lrshtvg/tmpzaaelkpw/davidban77-gns3-1.5.0-3moa8p5j
Installing 'davidban77.gns3:1.5.0' to '/home/james/.ansible/collections/ansible_collections/davidban77/gns3'
davidban77.gns3:1.5.0 was installed successfully
```

你甚至可以指定版本范围——例如，如果我们确定需要使用一个比`1.2.0`版本更新但又比`1.5.0`版本旧的集合，我们可以运行以下命令：

```
$ ansible-galaxy collection install 'davidban77.gns3:>1.2.0,<1.5.0'
Starting galaxy collection install process
Process install dependency map
Starting collection install process
Downloading https://galaxy.ansible.com/download/davidban77-gns3-1.4.0.tar.gz to /home/james/.ansible/tmp/ansible-local-1210j61dbhwq/tmpznhg7oik/davidban77-gns3-1.4.0-u800o2wa
Installing 'davidban77.gns3:1.4.0' to '/home/james/.ansible/collections/ansible_collections/davidban77/gns3'
davidban77.gns3:1.4.0 was installed successfully
```

这里，我们已经安装了我们指定范围内的最新版本——这是一个非常有用且强大的功能，用于维护依赖关系。

关于安装和维护集合版本的讨论将我们带回到最初的问题——如何确保所有用户都安装了开发 Playbook 所需的正确版本集合（或者是被批准的版本）。我们已经确定可以集中安装集合，这无疑是一个可行的途径。然而，一旦有人开始在其他机器上开发和/或运行 Playbook，这种方法就会失效。因此，很显然，我们需要另一种方法。

幸运的是，`ansible-galaxy` 命令也支持与需求文件配合使用——这些文件只是你需要安装的集合（及其版本）的一个列表，由于它们是简单的文本文件，所以可以与所有其他自动化代码一起提交到源代码控制系统中。然后，参与你剧本的所有人只需要运行一个命令来安装所需的集合，他们就可以继续进行开发和/或运行剧本，同时确保已安装了正确的集合，并且版本也是正确的。让我们在之前的例子中再添加一个集合，并创建一个名为 `requirements.yml` 的文件，内容如下：

```
---
collections:
- name: davidban77.gns3
  version: '>1.2.0,<1.5.0'
- marmorag.ansodium
```

在这个文件中，我们声明需要 `davidban77.gns3` 集合，并使用我们之前测试过的版本约束，同时还需要 `marmorag.ansodium` 集合。你可以看到我们在第二个集合中省略了 `name:` 键——只有在你指定了额外的参数（如 `version:`）时才需要这个键；因此，如果你不需要这些参数，创建一个简单的集合名称列表就足够了。一旦有了这个文件，你可以通过运行以下命令来确保集合符合所列要求：

```
$ ansible-galaxy collection install -r requirements.yml
Starting galaxy collection install process
Process install dependency map
Starting collection install process
Downloading https://galaxy.ansible.com/download/marmorag-ansodium-1.0.2.tar.gz to /home/james/.ansible/tmp/ansible-local-1242d4dc7w5j/tmp0rltoudk/marmorag-ansodium-1.0.2-q01qc8if
'davidban77.gns3:1.4.0' is already installed, skipping.
Installing 'marmorag.ansodium:1.0.2' to '/home/james/.ansible/collections/ansible_collections/marmorag/ansodium'
marmorag.ansodium:1.0.2 was installed successfully
```

在这里，你可以看到 `davidban77.gns3` 集合的需求已经满足，因此没有进一步的操作。然而，之前未安装的 `marmorag.ansodium` 集合现在被安装了。

顺便提一下，我们在这里使用的所有集合都可以在 [`galaxy.ansible.com`](https://galaxy.ansible.com) 上公开获取——然而，值得注意的是，你也可以指定自己的服务器，甚至是本地 tarball 来安装集合，我们将在本章后面看到后者的实际例子。

这就是我们关于在控制节点上管理集合的内容介绍，掌握了这些信息后，你应该能够为所有自动化需求建立一个一致的环境。在接下来的部分，我们将探讨这一主题的一个细微变化——如何更新安装 Ansible 包时安装的集合。

# 更新你的 Ansible 集合和核心安装

你可能会问，安装 Ansible 时安装的集合会发生什么——如何维护或升级它们？当然，一种方法是等待下一个 `ansible` 包的发布，然后通过 `pip` 升级——但是这是一种暴力方式，可能无法提供你期望的结果。让我们采取一种更加细致的方法。

假设你想要访问 `amazon.aws` 集合中的最新功能，该集合在安装 `ansible` 包时已经捆绑在其中——你只需再次按如下方式安装它：

```
$ ansible-galaxy collection install amazon.aws
Starting galaxy collection install process
Process install dependency map
Starting collection install process
Downloading https://galaxy.ansible.com/download/amazon-aws-6.1.0.tar.gz to /home/james/.ansible/tmp/ansible-local-12589si7_q1d/tmpytach2d5/amazon-aws-6.1.0-mihyh1f0
Installing 'amazon.aws:6.1.0' to '/home/james/.ansible/collections/ansible_collections/amazon/aws'
amazon.aws:6.1.0 was installed successfully
```

现在，如果你查询已安装的集合，你将看到以下内容：

```
$ ansible-galaxy collection list
# /home/james/.local/lib/python3.10/site-packages/ansible_collections
Collection                    Version
----------------------------- -------
amazon.aws                    5.4.0
…
# /home/james/.ansible/collections/ansible_collections
Collection        Version
----------------- -------
amazon.aws        6.1.0
```

因此，您可以看到集合的原始版本已被保留，但新版本已安装在我们的本地集合目录中（由`COLLECTIONS_PATHS`指定）。Ansible 将使用手动安装的集合优先于最初安装的集合，因此可以轻松升级与 Ansible 包捆绑的集合。如果您对使用哪个集合感到不安，可以使用一个方便的`lookup`插件来查询 Ansible 正在使用的集合的版本。考虑以下代码：

```
---
- name: Play to check collection version
  hosts: localhost
  gather_facts: no
  tasks:
    - name: Get the version of the amazon.aws collection
      ansible.builtin.debug:
        msg: "amazon.aws version {{ lookup('community.general.collection_version', 'amazon.aws') }}"
```

大多数的代码现在对你来说应该已经很熟悉了。我们在这里所做的就是使用一个名为`community.general.collection_version`的`lookup`插件，将我们要查询的集合名称的值传递给它（在我们的例子中是`amazon.aws`）。如果我们运行这个命令，它会友好地告诉我们，我们正在使用我们刚刚安装的版本：

```
$ ansible-playbook check-collection-version.yml
PLAY [Play to check collection version] ****************************************************************************************************************************************
TASK [Get the version of the amazon.aws collection] ****************************************************************************************************************************
ok: [localhost] => {
    "msg": "amazon.aws version 6.1.0"
}
PLAY RECAP *********************************************************************************************************************************************************************
localhost                  : ok=1    changed=0    unreachable=0    failed=0    skipped=0     rescued=0    ignored=0
```

如果我们将集合降级到比与`ansible`包一起安装的集合版本旧的版本（这是一个不太可能的场景，但仅举例说明），我们可以看到 Ansible 会忠实地遵循其路径搜索顺序，不会默认使用作为 Ansible 安装的一部分而安装的`amazon.aws`集合，尽管该版本较新：

```
$ ansible-galaxy collection install amazon.aws:5.0.0
Starting galaxy collection install process
Process install dependency map
Starting collection install process
Downloading https://galaxy.ansible.com/download/amazon-aws-5.0.0.tar.gz to /home/james/.ansible/tmp/ansible-local-1291avom1yll/tmpn2juwhhj/amazon-aws-5.0.0-nzac4kq1
Installing 'amazon.aws:5.0.0' to '/home/james/.ansible/collections/ansible_collections/amazon/aws'
amazon.aws:5.0.0 was installed successfully
james@controlnode:~/Practical-Ansible-Second-Edition/Chapter 6$ ansible-playbook check-collection-version.yml
PLAY [Play to check collection version] ****************************************************************************************************************************************
TASK [Get the version of the amazon.aws collection] ****************************************************************************************************************************
ok: [localhost] => {
    "msg": "amazon.aws version 5.0.0"
}
PLAY RECAP *********************************************************************************************************************************************************************
localhost                  : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

虽然简单，但是在安装 Ansible 时管理集合的这一步骤对于维护你的控制节点至关重要，因此我们有必要覆盖它。现在你已经学习了关于集合的所有内容，它们的定义，为什么它们重要以及如何管理它们，让我们在下一节中创建我们自己的集合来进一步学习。

# 创建你自己的集合

我们已经学习了关于集合及其如何管理和维护的大量知识。现在让我们通过从头开始创建您自己的集合来完成对此主题的知识，这样您将全面了解它们的组成方式和工作原理。

与角色一样（参见*第四章*，*Playbooks and Roles*），集合只是一个目录中文件的有序集合。虽然您可以查找所有这些目录并手动创建它们，但我们也可以使用`ansible-galaxy`工具为我们创建一个空白模板来使用。

让我们从基础知识开始。我们知道我们需要一个命名空间和一个集合名称。如果我们在 Ansible Galaxy 网站上发布，那么命名空间将会是我们的 GitHub 句柄，因为 Ansible Galaxy 会将其作为您的命名空间使用。在我们的情况下，我们不会发布到 Ansible Galaxy，所以我将选择命名空间`practicalansible`，但如果您希望发布您的工作，请随时将其替换为您的 GitHub 句柄以及以下所有示例代码和命令。

一旦我们决定了命名空间，下一步就是选择集合名称。通常，名称会用来表示集合的用途，而在这里，我们将用它来汇集本书中其他地方的示例，因此我们将集合命名为 `examples`。

我们将创建一个空目录来进行工作，以便与其他代码分开，然后我们将创建如下的骨架目录结构：

```
$ mkdir collection-development
$ cd collection-development/
$ ansible-galaxy collection init practicalansible.examples
- Collection practicalansible.examples was created successfully
If we examine this directory structure, we can see that it looks as follows:
$ tree
.
└── practicalansible
    └── examples
        ├── README.md
        ├── docs
        ├── galaxy.yml
        ├── meta
        │   └── runtime.yml
        ├── plugins
        │   └── README.md
        └── roles
6 directories, 4 files
```

在这里，你可以看到以下内容：

+   顶级目录就是我们选择的命名空间。

+   下一层目录是集合名称。

+   在这些顶级目录下，我们有以下内容：

    +   一个 `README.md` 文件（如果提交到 Ansible Galaxy，必须提供文档，在几乎所有场景下都建议提供）

    +   一个 `galaxy.yml` 文件，包含角色的关键信息，如标签、许可证以及文档页面，以确保你的条目在 Ansible Galaxy 上正确展示。

+   然后是 `docs/` 目录——该目录应用于存放所有文档，并将被 `ansible-doc` 命令引用，因此应根据最佳实践进行填充。

+   还有一个 `meta/` 目录——这个目录中预先填充了一个 `runtime.yml` 文件，用于提交到 Ansible Galaxy，包含关于所需最低版本的 `ansible-core` 以及与弃用和代码升级相关的其他重要信息。

+   最后但同样重要的是 `plugins/` 和 `roles/` 目录，这里将存放你要分发的代码。

在骨架目录结构中没有创建的两个目录是 `playbooks/` 和 `tests/`。`playbooks/` 目录用于与集合一起分发 playbook（是的，你甚至可以这么做！），你可以直接从命令行运行它（前提是你使用的是 `ansible-core` 2.11 或更新版本），或者在 playbook 中通过 `import_playbook` 语句来调用。

`tests/` 目录用于存放自动化测试代码，供 `ansible-test` 工具运行。我们在本书中不会重点讲解这个部分，但如果你想深入了解，可以从这个链接开始：[`docs.ansible.com/ansible/latest/dev_guide/developing_collections_testing.xhtml`](https://docs.ansible.com/ansible/latest/dev_guide/developing_collections_testing.xhtml)。

由于并非每个人都能（或愿意）提交到 Ansible Galaxy，本章的重点将放在使用集合来分发和使用代码（如模块和角色）。我们从这里开始。

迄今为止，你已经看到了许多从集合中运行模块的示例——这是大多数用户最常见的使用场景，因此我们从这里开始。

在*第五章*中，*创建和使用*部分，我们创建了一个简单的模块来测试，名为`remotecopy`。我们知道集合是分发这些模块的标准方式，所以让我们将它集成到我们的新集合中。我们通过在`plugins/`目录下创建一个名为`modules/`的目录，然后将代码复制到那里来实现。最终结果应该是这样的：

```
$ mkdir plugins/modules
$ cp ~/Practical-Ansible-Second-Edition/Chapter\ 5/testplaybook/library/remote_filecopy.py plugins/modules/
$ tree
.
├── README.md
├── docs
├── galaxy.yml
├── meta
│   └── runtime.yml
├── plugins
│   ├── README.md
│   └── modules
│       └── remote_filecopy.py
└── roles
5 directories, 5 files
```

请注意，我们没有对模块中的代码做任何修改，它与*第五章*中的*创建和使用*完全相同。要使用集合，你的代码不需要做任何修改——关键是确保将文件放入正确的子目录。完成这些后，我们现在可以将我们的集合打包进行测试。操作方法是切换到集合根目录，然后执行`ansible-galaxy collection` `build`命令：

```
$ cd ~/Practical-Ansible-Second-Edition/Chapter\ 6/collection-development/practicalansible/examples/
$ ansible-galaxy collection build
Created collection for practicalansible.examples at /home/james/Practical-Ansible-Second-Edition/Chapter 6/collection-development/practicalansible/examples/practicalansible-examples-1.0.0.tar.gz
```

请注意，Ansible 已经为文件名附加了版本号`1.0.0`——这个版本号来自于`galaxy.yml`文件，是默认值。我们将在未来的示例中进行编辑，但现在它满足我们的需求。

集合构建完成后，到了测试它的时候了。首先，我们需要创建一个空目录来工作。进入该目录后，我们可以使用`ansible-galaxy`命令来安装我们的新集合：

```
$ mkdir ~/Practical-Ansible-Second-Edition/Chapter\ 6/collection-test/
$ cd ~/Practical-Ansible-Second-Edition/Chapter\ 6/collection-test/
$ ansible-galaxy collection install ~/Practical-Ansible-Second-Edition/Chapter\ 6/collection-development/practicalansible/examples/practicalansible-examples-1.0.0.tar.gz
Starting galaxy collection install process
Process install dependency map
Starting collection install process
Installing 'practicalansible.examples:1.0.0' to '/home/james/.ansible/collections/ansible_collections/practicalansible/examples'
practicalansible.examples:1.0.0 was installed successfully
```

恭喜你！你刚刚创建并安装了你的第一个集合！接下来，我们需要创建一个剧本来进行测试。我们将像之前一样创建它，确保使用我们新构建并安装的集合的 FQCN：

```
---
- name: Playbook to test custom module
  hosts: localhost
  gather_facts: no
  tasks:
    - name: Test the custom module
      practicalansible.examples.remote_filecopy:
        source: /tmp/foo
        dest: /tmp/bar
      register: testresult
    - name: Print the test result data
      ansible.builtin.debug:
        var: testresult
```

现在，我们将以正常的方式运行剧本——同样，不需要其他特别的命令或配置，即使我们刚刚构建并安装了自己的集合：

```
$ ansible-playbook collection-test1.yml
PLAY [Playbook to test custom module] ******************************************
TASK [Test the custom module] **************************************************
changed: [localhost]
TASK [Print the test result data] **********************************************
ok: [localhost] => {
    "testresult": {
        "changed": true,
        "failed": false
    }
}
PLAY RECAP *********************************************************************
localhost                  : ok=2    changed=1    unreachable=0    failed=0    skipped=0     rescued=0    ignored=0
```

我们可以看到我们的模块运行完美，因此你可以感到自豪，因为你刚刚成功地构建、安装并集成了你用本书中已有代码构建的第一个集合。

你也可以以正常的方式查询该模块中内建的文档：

```
$ ansible-doc practicalansible.examples.remote_filecopy
> PRACTICALANSIBLE.EXAMPLES.REMOTE_FILECOPY    (/home/james/.ansible/collections/ansible_collections/practicalansible/examples/plugins/modules/remote_filecopy.py)
        The remote_copy module copies a file on the remote host from a given source to a provided destination.
ADDED IN: version 2.15 of practicalansible.examples
OPTIONS (= is mandatory):
= dest
        Path to the destination on the remote host for the copy
= source
        Path to a file on the source file on the remote host
…
```

当然，我们不仅构建了第一个集合，还写了一个单独的剧本作为测试工具。在很多情况下，你不仅仅会像我们这里写一个简单的测试工具——你会编写一个剧本来解决实际用例，甚至提供一些示例代码来展示如何使用该集合。现在，让我们通过将剧本添加到集合中来做到这一点：

```
$ cd ~/Practical-Ansible-Second-Edition/Chapter\ 6/collection-development/practicalansible/examples/
$ mkdir playbooks/
$ cp ~/Practical-Ansible-Second-Edition/Chapter\ 6/collection-test/collection-test1.yml playbooks/collection_test1.yml
$ sed -i 's/version: .*$/version: 1.1.0/g' galaxy.yml
$ tree
.
├── README.md
├── docs
├── galaxy.yml
├── meta
│   └── runtime.yml
├── playbooks
│   └── collection_test1.yml
├── plugins
│   ├── README.md
│   └── modules
│       └── remote_filecopy.py
└── roles
6 directories, 6 files
```

虽然我们没有修改剧本代码，但我们的剧本在剧本定义中有一个硬编码的`hosts`行，设置为`localhost`——在实际集合中你可能永远不会这样做，Ansible 文档本身也建议使用变量来定义主机，以便用户可以在此处指定自己的模式。不过，我们这里只是通过集合分发示例代码，因此在我们的案例中，这一决策是合理的。

虽然我们没有改变 playbook 文件的内容，但我们在`cp`命令中更改了文件名。这样做是因为 Ansible 不支持在引用集合中的对象时使用文件名中的连字符（`-`）——这是一个重要的细节，稍后可能会让你困扰！另外，注意我们使用了`sed`命令来更改`galaxy.yml`文件中的版本号，使得书中的代码更易读，但你也可以使用你喜欢的文本编辑器进行编辑。完成这些操作后，我们可以像之前一样重新打包集合：

```
$ ansible-galaxy collection build
Created collection for practicalansible.examples at /home/james/Practical-Ansible-Second-Edition/Chapter 6/collection-development/practicalansible/examples/practicalansible-examples-1.1.0.tar.gz
Now that we've built the new tarball, we can install it as follows:
$ ansible-galaxy collection install practicalansible-examples-1.1.0.tar.gz
Starting galaxy collection install process
Process install dependency map
Starting collection install process
Installing 'practicalansible.examples:1.1.0' to '/home/james/.ansible/collections/ansible_collections/practicalansible/examples'
practicalansible.examples:1.1.0 was installed successfully
```

现在，你可以通过提供完全限定的 playbook 名称从命令行运行 playbook：

```
$ ansible-playbook practicalansible.examples.collection_test1
[WARNING]: running playbook inside collection practicalansible.examples
PLAY [Playbook to test custom module] ******************************************
TASK [Test the custom module] **************************************************
changed: [localhost]
TASK [Print the test result data] **********************************************
ok: [localhost] => {
    "testresult": {
        "changed": true,
        "failed": false
    }
}
PLAY RECAP *********************************************************************
localhost                  : ok=2    changed=1    unreachable=0    failed=0    skipped=0     rescued=0    ignored=0
```

你现在已经成功地将一个 playbook 添加到你的集合中，提升了版本号，并升级了本地安装。你甚至直接从集合中运行了 playbook，而不是像本书中的其他例子一样从本地文件中运行。希望你此刻能感受到一种成就感！接下来，我们将通过另一个例子来扩展我们的集合——集成插件。我们将再次使用本书中其他地方提供的代码——具体来说是*第七章*，*创建和* *使用* *插件*。

为此，我们需要将所需的代码复制到我们的集合中。虽然你可能期望插件放在`plugins/`目录下，但它们实际上必须放在该目录下的一个子目录中，并按照插件类型命名。我们将复制一个`lookup`插件，因此我们必须先创建一个名为`lookup/`的目录，然后才能将插件代码复制到其中：

```
$ cd ~/Practical-Ansible-Second-Edition/Chapter\ 6/collection-development/practicalansible/examples/
$ mkdir plugins/lookup
$ cp ~/Practical-Ansible-Second-Edition/Chapter\ 7/lookup_plugins/firstchar.py plugins/lookup/
$ sed -i 's/version: .*$/version: 1.2.0/g' galaxy.yml
$ tree
.
├── README.md
├── docs
├── galaxy.yml
├── meta
│   └── runtime.yml
├── playbooks
│   └── collection_test1.yml
├── plugins
│   ├── README.md
│   ├── lookup
│   │       └── firstchar.py
│   └── modules
│       └── remote_filecopy.py
└── roles
7 directories, 7 files
```

再次说明，我们并没有修改插件代码——我们只是将它放入了我们的目录结构中。同样，我们使用`sed`编辑`galaxy.yml`文件以提升集合的版本号。完成这些操作后，我们可以以正常方式构建并安装新版本的集合：

```
$ ansible-galaxy collection build
Created collection for practicalansible.examples at /home/james/Practical-Ansible-Second-Edition/Chapter 6/collection-development/practicalansible/examples/practicalansible-examples-1.2.0.tar.gz
$ ansible-galaxy collection install practicalansible-examples-1.2.0.tar.gz
Starting galaxy collection install process
Process install dependency map
Starting collection install process
Installing 'practicalansible.examples:1.2.0' to '/home/james/.ansible/collections/ansible_collections/practicalansible/examples'
practicalansible.examples:1.2.0 was installed successfully
```

完成这一点后，又到了通过 playbook 测试我们新增强的集合的时刻。这次，我们不会将 playbook 构建到集合中——我们只需创建它并从之前创建的测试目录中运行它。playbook 代码应如下所示——再次，你应该发现它与*第七章*，*创建和使用* *插件*中的内容完全一致，唯一的不同是我们在 playbook 中使用了`lookup`插件的 FQCN，而不是你可能会使用的简短名称：

```
---
- name: Play to demonstrate our custom lookup plugin
  hosts: localhost
  gather_facts: false
  tasks:
    - name: make a statement
      ansible.builtin.debug:
        msg: "{{ lookup('practicalansible.examples.firstchar', 'testdoc.txt')}}"
```

记得在与 playbook 相同的目录下创建一个名为`testdoc.txt`的文件，以供`lookup`插件引用——我们的示例包含如下内容：

```
$ cat testdoc.txt
I can Ansible!
```

最后，以正常方式运行 playbook，你应该会发现插件如预期般正常工作：

```
$ ansible-playbook collection-test2.yml
PLAY [Play to demonstrate our custom lookup plugin] ****************************
TASK [make a statement] ********************************************************
ok: [localhost] => {
    "msg": "73"
}
PLAY RECAP *********************************************************************
localhost                  : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

如你所见，我们的`lookup`插件在我们新扩展的集合中表现完美。为了完整地展示如何构建自己的集合，我们将通过向其中添加一个角色来进一步扩展它。

提供一些背景信息，`ansible-galaxy`（包括工具和 Ansible Galaxy 网站）最初设想用于管理、分发和安装角色。考虑到管理集合的过程，以及集合作为可重用的 Ansible 代码工件的用途与其最初设计的角色非常相似，因此使用相同的工具来管理集合是非常合理的。

在写本文时，Galaxy 工具和网站同时支持角色和集合，并且预计在可预见的未来仍将继续支持。不过，集合提供了许多优点，包括将多个角色打包到一个集合中的选项，并且你还可以使用集合分发任何可能需要的插件（甚至模块）。

将角色迁移到集合的过程非常简单，如果你有一个仅包含任务、文件、模板、处理程序和变量等对象的简单角色，那么你可以直接将代码复制过去（自然是到正确的位置）而无需任何修改。然而，如果你之前在角色中包含了插件，那么需要特别注意，因为这些插件需要从角色中分离出来，并以与我们在本节稍早时候集成的 `lookup` 插件相同的方式，集成到集合的顶层。

为了扩展我们的示例，我们将取自 *第四章*《剧本与角色》中的一个角色，演示如何将其集成并进行测试。与之前一样，我们需要切换到集合目录，并将角色复制到集合中的 `roles/` 目录中（保留其目录结构）。你可以按照以下方式操作：

```
$ cd ~/Practical-Ansible-Second-Edition/Chapter\ 6/collection-development/practicalansible/examples/
$ cp -r ~/Practical-Ansible-Second-Edition/Chapter\ 4/role-example1/roles/installapache/ roles/
$ sed -i 's/version: .*$/version: 1.3.0/g' galaxy.yml
$ tree
.
├── README.md
├── docs
├── galaxy.yml
├── meta
│   └── runtime.yml
├── playbooks
│   └── collection_test1.yml
├── plugins
│   ├── README.md
│   ├── lookup
│   │   ├── firstchar.py
│   │   └── firstchar.pyc
│   └── modules
│       └── remote_filecopy.py
└── roles
    └── installapache
        └── tasks
            ├── fedora.yml
            ├── main.yml
            └── ubuntu.yml
9 directories, 11 files
```

如你所见，角色现在已被复制到适当位置，并且保持与之前相同的结构，位于集合中的 `roles/` 子目录下。完成此操作后，我们可以构建新的集合（我们再次增加了版本号）并进行安装：

```
$ ansible-galaxy collection build
Created collection for practicalansible.examples at /home/james/Practical-Ansible-Second-Edition/Chapter 6/collection-development/practicalansible/examples/practicalansible-examples-1.3.0.tar.gz
$ ansible-galaxy collection install practicalansible-examples-1.3.0.tar.gz
Starting galaxy collection install process
Process install dependency map
Starting collection install process
Installing 'practicalansible.examples:1.3.0' to '/home/james/.ansible/collections/ansible_collections/practicalansible/examples'
practicalansible.examples:1.3.0 was installed successfully
```

使用新的更新版角色后，剩下的工作就是测试它。我们将借用之前用于测试的相同剧本，只需稍作修改，从集合中调用角色：

```
---
- name: Install Apache using a role
  hosts: frontends
  become: true
  roles:
    - practicalansible.examples.installapache
```

由于这是一个在外部节点上安装 Apache 的角色，我们需要复制之前使用的库存文件——如果你使用了本书附带的 GitHub 仓库，这些都已经为你做了。剩下的就是运行代码，结果应该类似于：

```
$ ansible-playbook -i hosts collection-test3.yml
PLAY [Install Apache using a role] *********************************************
TASK [Gathering Facts] *********************************************************
ok: [web01.example.org]
ok: [web02.example.org]
TASK [practicalansible.examples.installapache : Install Apache using yum] ******
skipping: [web01.example.org]
skipping: [web02.example.org]
TASK [practicalansible.examples.installapache : Start the Apache server] *******
skipping: [web01.example.org]
skipping: [web02.example.org]
TASK [practicalansible.examples.installapache : Install Apache using apt] ******
ok: [web01.example.org]
ok: [web02.example.org]
TASK [practicalansible.examples.installapache : Start the Apache server] *******
ok: [web01.example.org]
ok: [web02.example.org]
PLAY RECAP *********************************************************************
web01.example.org          : ok=3    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
web02.example.org          : ok=3    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
```

如果你之前作为 *第四章*《剧本与角色》的一部分运行了这个角色，那么不会做任何更改，但从前面的内容可以清楚地看到，我们成功地从集合中调用了该角色，并且它运行得非常完美。

到此为止，我们已经成功地将插件、模块、剧本和角色集成到集合中，带您完成了构建和添加到自己集合的整个过程。这也标志着我们对集合的深入探讨的结束，但我希望这些信息足以帮助您进行自己的测试和开发，最终构建您自己的集合。

# 总结

集合现在是 Ansible 整体架构中至关重要的一部分，它为所有代码的分发和管理提供了一个简单而有效的机制，控制节点上的所有代码都能通过它进行管理。学习集合对理解任何现代版本 Ansible 的架构至关重要，掌握这些信息后，您可以在控制节点上管理、构建和维护集合，甚至将其贡献回社区，如果您愿意的话。

在本章中，您了解了集合的历史，它们是如何产生的，以及它们对 Ansible 为什么如此重要。接着，您学会了如何使用 FQCN 引用集合中的对象，然后继续学习如何在控制节点上安装和管理集合。最后，我们查看了构建、安装和测试自己集合的过程，您可以根据需要将自己的集合贡献给更广泛的 Ansible 社区。

在下一章，我们将学习如何使用和创建我们自己的插件，提供扩展 Ansible 功能所需的技能，以适应您的定制环境，并为社区做出贡献。

# 问题

1.  集合是 Ansible 2.9 之后版本的可选功能：

    1.  True

    1.  False

1.  缩写 FQCN 代表什么？

    1.  完全限定集合命名空间

    1.  完全限定控制节点

    1.  完全限定集合名称

    1.  完全限定控制名称

1.  使用哪个命令安装和管理集合？

    1.  `ansible-galaxy`

    1.  `ansible-collection`

    1.  `ansible`

    1.  `collection-manager`

# 进一步阅读

`ansible-galaxy` 及其文档可以在此找到：[`galaxy.ansible.com/docs/`](https://galaxy.ansible.com/docs/)
