

# 第十章：使用 Ansible 进行网络自动化

多年前，标准做法是手动配置每一个网络设备。这种设备管理方式之所以可行，主要是因为路由器和交换机负责物理服务器的流量路由，因此每个网络设备只需要少量配置，且变更速度较慢。此外，只有人类才拥有足够的信息来设置网络。无论是在规划还是执行方面，一切都非常手动。

**虚拟化**改变了这一范式，因为它导致成千上万的机器连接到同一个交换机或路由器，每台机器可能有不同的网络需求。变更的速度非常快，且频繁发生，随着虚拟基础设施通过代码定义，管理员仅仅为了跟上基础设施的变化就需要全职工作。虚拟化编排平台对机器的位置有更好的了解，甚至可以为我们生成清单，正如我们在前面的章节中看到的那样。从实际操作来看，人类无法记住或管理现代、大规模的虚拟化基础设施。因此，当配置网络基础设施时，自动化变得显而易见。

本章将通过以下主题来学习更多关于如何自动化我们的网络：

+   为什么要自动化网络管理？

+   Ansible 如何管理网络设备

+   如何启用网络自动化

+   可用的 Ansible 网络模块

+   连接到网络设备

+   网络设备的环境变量

+   网络设备的自定义条件语句

让我们开始吧！

# 技术要求

本章假设您已经按照*第一章*《Ansible 入门》中的详细说明设置了控制主机，并且正在使用最新版本——本章中的示例是在 Ansible 2.15 版本上测试的。本章还假设您至少有一台额外的主机进行测试；理想情况下，这应该是基于 Linux 的。由于本章以网络设备为中心，我们理解并非每个人都能访问特定的网络设备进行测试（例如，思科交换机）。

在给出示例的地方，如果您能够访问相关设备，欢迎您深入探索这些示例。然而，如果您没有网络硬件可用，我们将提供一个使用免费提供的 Cumulus VX 的示例，它提供了 Cumulus Networks 交换环境的完整功能演示。尽管本章会给出具体的主机名示例，但您可以自由地用您的主机名和/或 IP 地址替换它们。如何进行替换的详细信息将在适当的位置提供。

本章的代码包可以在此处找到：[`github.com/PacktPublishing/Practical-Ansible-2/tree/master/Chapter%2010`](https://github.com/PacktPublishing/Practical-Ansible-2/tree/master/Chapter%2010)。

# 为什么要自动化网络管理？

过去 30 年中，我们设计数据中心的方式发生了根本性的变化。在 90 年代，一个典型的数据中心充满了每个具有特定用途的物理机器。在许多公司中，服务器是根据机器的用途从不同供应商处购买的。这意味着每次需要新的服务器、网络设备和存储设备时，都会购买、配置、设置并交付这些设备。

这里的一个重大缺点是，从识别出服务器需求到实际交付之间的显著延迟。当时，这种方式是可以接受的，因为大多数公司拥有的系统很少，而且这些系统也很少变动。此外，这种方式成本高昂，因为很多设备被低效利用。

随着全球社会和企业在技术领域的进步，公司必须从其基础设施中获得更高的效率，并减少基础设施的部署时间和成本。这些新需求为一种新思路铺平了道路：虚拟化。通过创建虚拟化集群，你不需要具备合适尺寸的物理主机，因此可以提前配置一批物理主机，将它们添加到资源池中，然后在虚拟化平台中创建合适尺寸的虚拟机。这种解耦意味着，当需要新服务器时，你只需几次点击，就能在几秒钟内创建并启用它。

这种转变还使企业能够从按项目配置基础设施的方式转向一个大型中央基础设施，在这种基础设施中，行为可以通过软件和配置进行定义。新的架构意味着单一的网络基础设施可以支持多个项目，无论其规模如何。我们称之为**虚拟数据中心基础设施**；在这种基础设施中，我们尽可能使用通用的设计模式。虚拟数据中心基础设施允许企业大规模部署、切换和服务基础设施，以支持多种项目，从而通过简单的细分（例如，创建虚拟服务器）成功实现它们。

虚拟化的另一个显著优势是工作负载与物理主机的解耦。历史上，由于工作负载与物理主机绑定，如果主机宕机且未正确复制到其他硬件上，工作负载也会丢失。虚拟化解决了这个问题，因为工作负载现在绑定到一个或多个虚拟主机上，而这些虚拟主机可以通过虚拟化控制器自由地从一个物理主机迁移到另一个。

这种快速配置服务器的能力，以及将它们从一个主机迁移到另一个主机的能力，带来了网络配置管理的问题。以前，人工在安装新机器时调整配置细节是可以接受的，但现在，机器从一个主机迁移到另一个主机（因此从一个物理交换机端口迁移到另一个）时无需任何人工干预。这种特殊性意味着系统也需要更新网络配置。

与此同时，出于类似的原因，VLAN 在网络中得到了广泛应用，这使得网络设备的利用率得到了显著提升，因此其成本得以优化。

如今，我们在更大规模的环境中工作，其中虚拟对象（机器、容器、功能等）在数据中心中移动，完全由软件系统管理，人类的参与越来越少。

在这种环境下，自动化网络配置是成功的关键部分。

如今，一些公司（著名的*云服务提供商*）在一个规模上工作，其中手动网络管理不仅不可行，甚至是不可能的，即使他们雇用了大量的网络工程师。另一方面，也有许多环境，在技术上至少部分可以手动管理网络配置，但实际上依然不切实际。

除了配置网络设备所需的时间之外，网络自动化的最大优势——从我的角度来看——是能够显著减少人为错误。如果一个人需要在 100 台设备上配置 VLAN，肯定会在过程中犯一些错误。这是正常的，但问题在于，这些配置必须经过彻底测试和修改。问题往往不仅止于此，因为当设备出现故障并需要更换时，人工需要将新设备配置成与旧设备相同的方式。随着时间的推移，配置通常会发生变化，且通常没有清晰的方式追溯这一变化。因此，在更换故障的网络设备时，可能会出现一些规则问题，这些规则在之前的设备中存在，但在新设备中却没有。

既然我们已经讨论了自动化网络管理的必要性，接下来我们来看看如何使用 Ansible 管理网络设备。

# Ansible 如何管理网络设备

Ansible 允许你管理许多不同的网络设备，包括 Arista EOS、Cisco ASA、Cisco IOS、Cisco IOS XR、Cisco NX-OS、Dell OS 6、Dell OS 9、Dell OS 10、Extreme EXOS、Extreme IronWare、Extreme NOS、Extreme SLX-OS、Extreme VOSS、F5 BIG-IP、F5 BIG-IQ、Junos OS、Lenovo CNOS、Lenovo ENOS、MikroTik RouterOS、Nokia SR OS、Pluribus Netvisor 和 VyOS，以及所有支持 NETCONF 的操作系统。正如你可以想象的那样，我们可以通过各种方式使 Ansible 与这些设备进行通信。

此外，我们还必须记住，Ansible 网络模块是在控制主机上运行的（即你执行`ansible`命令的主机），而通常情况下，Ansible 模块是在目标主机上运行的。这一差异至关重要，因为它使得 Ansible 可以根据目标设备的类型使用不同的连接机制。请记住，即使你有一台支持 SSH 管理功能的主机（许多交换机都有此功能），Ansible 仍然需要目标主机上存在 Python 才能在其上运行模块。由于大多数交换机（和嵌入式硬件）没有 Python，Ansible 通常会使用特定的连接协议。Ansible 支持的用于网络设备管理的主要协议如下。

Ansible 用于连接这些网络设备的五种主要连接类型如下：

+   `ansible.netcommon.network_cli`

+   `ansible.netcommon.netconf`

+   `ansible.netcommon.httpapi`

+   `local`

+   `ssh`

当你与网络设备建立连接时，你需要根据设备支持的连接机制以及你的需求来选择连接方式：

+   `ansible.netcommon.network_cli` 被大多数模块支持，它与 Ansible 通常如何与非网络模块一起工作最为相似。这种模式通过 SSH 使用命令行界面（CLI）。此协议在执行开始时创建一个持久连接，并在整个执行过程中保持活动状态，这样你就不必为每个任务提供凭证。

+   `ansible.netcommon.netconf` 被一些模块支持。这种模式通过 SSH 使用 XML，因此它将基于 XML 的配置应用到设备上。此协议在执行开始时创建一个持久连接，并在整个执行过程中保持活动状态，这样你就不必为每个任务提供凭证。

+   `ansible.netcommon.httpapi` 被少数模块支持。这种模式使用设备发布的 HTTP API。此协议在执行开始时创建一个持久连接，并在整个执行过程中保持活动状态，这样你就不必为每个任务提供凭证。

+   `local` 被大多数设备支持，但它是一种已弃用的连接模式。这种连接模式依赖于厂商，并通常需要在 Ansible 执行主机上安装一些特定厂商的包。此模式不会创建持久连接，因此你必须在每个任务中提供凭证。尽可能避免使用这种模式。

+   `ssh` 作为一个选项不容忽视。尽管大量设备依赖前述连接模式，但一种新的网络设备类型正在被创造，它在**白盒**交换机硬件上本地运行 Linux。例如 Cumulus Networks（现已成为 NVIDIA 的一部分），由于其软件基于 Linux，所有配置都可以通过 SSH 执行，就像该交换机只是另一台 Linux 主机一样。

了解 Ansible 如何连接和与您的网络硬件通信非常重要，因为它为您提供了构建 Ansible 剧本和调试问题时所需的理解。

在本节中，我们介绍了与网络硬件工作时会遇到的通信协议。在下一节中，我们将继续讨论如何通过 Ansible 开始我们的网络自动化之旅的基础知识。

## 如何启用网络自动化

在使用 Ansible 进行网络自动化之前，您必须确保拥有所需的一切。根据我们将使用的连接方式的不同，我们需要不同的要求。以我们的示例为例，我们将使用一台支持 `network_cli` 连接的 Cisco IOS 设备。

Ansible 网络自动化工作的唯一要求如下：

+   Ansible 2.5+

+   与网络设备的正常连接

首先，我们需要检查 Ansible 版本：

1.  为了确保您拥有最新版本的 Ansible，您可以运行以下命令：

    ```
    $ ansible --version
    ```

该命令将告诉您 Ansible 安装的版本。

1.  如果版本是 2.5 或更高，可以使用以下命令（带有适当的选项）检查网络设备的连接性：

    ```
    $ ansible all -i n1.example.org, -c network_cli -u my_user -k -m cisco.ios.ios_facts -e ansible_network_os=cisco.ios.ios all
    ```

该命令应返回设备的事实，证明我们能够连接。与任何其他目标一样，Ansible 可以获取事实，这通常是 Ansible 与目标交互时的第一步。

在此特定情况下，我们使用 `–k` 参数告知 Ansible 需要提示输入密码，用于 SSH 登录。

获取事实是一个关键步骤，因为它使 Ansible 能够了解设备的当前状态并采取相应的行动。

通过在目标设备上运行 `cisco.ios.ios_facts` 模块，我们实际上只是在执行第一步标准操作（因此不会执行任何更改），但这确认了 Ansible 能够连接到设备并对其执行命令。

如您所料，只有当您能够访问运行 Cisco IOS 的网络设备时，才能运行上述命令并探索其行为。我们理解并非每个人都能获得相同的网络设备进行测试（甚至没有设备！）。幸运的是，一种新型的交换机已经开始普及——白盒交换机。这些交换机由多个制造商生产，基于标准化硬件，您可以在其上安装自己的网络操作系统。一个这样的操作系统是 NVIDIA Cumulus Linux，您可以免费下载其测试版 NVIDIA Cumulus VX。

注意

在撰写时，NVIDIA Cumulus VX 的下载链接为 [`www.nvidia.com/en-us/networking/ethernet-switching/cumulus-vx/`](https://www.nvidia.com/en-us/networking/ethernet-switching/cumulus-vx/)。您需要注册才能下载，但注册后您可以免费访问开源网络的世界。

下载适合您虚拟化平台（例如，VirtualBox）的镜像文件，然后像运行其他 Linux 虚拟机一样运行它。一旦完成，您可以像连接任何其他 SSH 设备一样连接到 NVIDIA Cumulus VX 交换机。例如，要运行一个临时命令以收集所有交换机端口接口的事实（这些接口在 Cumulus VX 上被列为 `swp1`、`swp2` 和 `swpX`），您可以运行以下命令：

```
$ ansible -i vx01.example.org, -u cumulus -m ansible.builtin.setup -a 'filter=ansible_swp*' all --ask-pass
```

如果成功，这应该会显示关于 Cumulus VX 虚拟交换机的交换机端口接口的详细信息。在我的测试系统中，这些输出的第一部分如下所示：

```
vx01.example.org | SUCCESS => {
  "ansible_facts": {
    "ansible_swp1": {
      "active": false,
      "device": "swp1",
      "features": {
        "esp_hw_offload": "off [fixed]",
        "esp_tx_csum_hw_offload": "off [fixed]",
        "fcoe_mtu": "off [fixed]",
        "generic_receive_offload": "on",
        "generic_segmentation_offload": "on",
        "highdma": "off [fixed]",
        ...
```

正如您所看到的，使用像 NVIDIA Cumulus Linux 这样的操作系统与白盒交换机一起工作具有一个优点，那就是您可以使用标准的 SSH 协议进行连接，甚至可以使用 `ansible.builtin.setup` 模块来收集关于它的信息。与其他专有硬件一起工作并不更加困难，但需要指定更多的参数，正如我们在本章前面所展示的那样。

现在您已经了解了启用网络自动化的基础知识，接下来让我们学习如何在 Ansible 中发现适合的网络模块，以实现我们期望的自动化任务。

## 可用的 Ansible 网络模块

随着 Galaxy 和集合的出现，Ansible 网络内容的可用性大大增加。目前，Galaxy 上有超过 150 个集合和超过 1,000 个角色。您还可以在官方的 Ansible 文档中找到最重要的集合。要找到这些集合，您应采取以下步骤：

+   首先，查看官方文档。通过访问 [`docs.ansible.com/ansible/latest/network/user_guide/platform_index.xhtml`](https://docs.ansible.com/ansible/latest/network/user_guide/platform_index.xhtml)，您可以找到不同设备系列及其使用的连接类型。

+   如果您希望管理的设备不在列表中，您可以访问 [`galaxy.ansible.com`](https://galaxy.ansible.com) 并使用该网站的搜索功能在 Galaxy 上进行搜索。

模块和集合的列表过长且具体到不同的硬件系列，我们无法在此深入讨论。此外，这些列表会不断更新，并且通常会不断增长。

如果您熟悉手动配置设备，您会很快发现模块的名称非常自然，因此您很容易理解它们的功能。然而，让我们通过一些来自 Cisco IOS 模块的示例来了解——特别是参考 [`galaxy.ansible.com/cisco/ios`](https://galaxy.ansible.com/cisco/ios)：

+   `cisco.ios.ios_banner`：顾名思义，此模块允许您调整和修改登录横幅（在许多系统中称为 *motd*）。

+   `cisco.ios.ios_bgp`：此模块允许您配置 BGP 路由。

+   `cisco.ios.ios_command`：这是 IOS 版本的 `ansible.builtin.command` 模块，允许执行许多不同的命令。至于 `ansible.builtin.command` 模块，这是一个非常强大的模块，但如果有可用的特定模块用于我们要执行的操作，最好使用它们。

+   `cisco.ios.ios_config`：此模块允许我们对设备的配置文件进行任何更改。与 `cisco.ios.ios_command` 模块类似，这是一个非常强大的模块，但如果有可用的特定模块用于我们要执行的操作，最好使用它们。该模块的幂等性仅在未使用缩写命令时才有保证。

+   `cisco.ios.ios_vlan`：此模块允许配置 VLAN。

这些只是一些示例，但对于 Cisco IOS 还有许多其他模块，如果您找不到执行所需操作的特定模块，您始终可以退回到 `cisco.ios.ios_command` 和 `cisco.ios.ios_config`，由于它们的灵活性，您可以执行任何您能想到的操作。

相比之下，如果您正在使用 Cumulus Linux 交换机，您会发现可用的模块较少，因为它们更加通用。

一如既往，Ansible 文档是您的朋友，当您学习如何在新设备类上自动化命令时，它应该是您的首选参考。在本节中，我们展示了一个简单的过程，用于查找可用于您的网络设备的 Ansible 模块，使用 Cisco 作为特定示例（虽然您可以将这些原则应用于任何其他设备）。现在，让我们来看一下 Ansible 如何连接到网络设备。

# 连接到网络设备

如我们所见，Ansible 网络中存在一些特殊性，因此需要进行特定的配置。

要使用 Ansible 管理网络设备，您需要至少有一个设备进行测试。假设我们有一个可用的 Cisco IOS 系统。由于并非每个人都会有这样的设备进行测试，因此以下内容仅作为假设性示例提供。

根据 [`docs.ansible.com/ansible/latest/network/user_guide/platform_index.xhtml`](https://docs.ansible.com/ansible/latest/network/user_guide/platform_index.xhtml) 页面，我们可以看到该设备的正确 `ansible_network_os` 是 `cisco.ios.ios`，并且我们可以使用 `network_cli` 和 `local` 两种方式连接它。由于 `local` 已经被弃用，我们将使用 `network_cli`。按照以下步骤配置 Ansible，以便您可以管理 IOS 设备：

1.  首先，让我们创建一个包含 `routers` 组中设备的库存文件：

    ```
    [routers]
    n1.example.org
    n2.example.org
    [cumulusvx]
    vx01.example.org
    ```

1.  为了知道使用哪些连接参数，我们将设置 Ansible 的特殊连接变量，以便它们定义连接参数。我们将在我们的 playbook 的一个 group variables 子目录中执行此操作，因此我们需要创建 `group_vars/routers.yml` 文件，并包含以下内容：

    ```
    ---
    ansible_connection: network_cli
    ansible_network_os: cisco.ios.ios
    ansible_become: True
    ansible_become_method: enable
    ```

多亏了这些特殊变量，Ansible 将知道如何连接到你的设备。我们在本书的前面部分提到过一些这些示例，但作为回顾，Ansible 使用这些变量的值来以以下方式决定它们的行为：

+   `ansible_connection`：此变量由 Ansible 用于决定如何连接到设备。选择 `network_cli` 时，我们指示 Ansible 以 SSH 模式连接到 CLI。

+   `ansible_network_os`：此变量由 Ansible 用于了解我们将使用的设备的设备家族。选择 `cisco.ios.ios` 时，我们指示 Ansible 期望连接到 Cisco IOS 设备。

+   `ansible_become`：此变量由 Ansible 用于决定是否在设备上执行特权提升。通过指定 `True`，我们告知 Ansible 执行特权提升。

+   `ansible_become_method`：有多种方式可以在不同的设备上执行特权提升（通常在 Linux 服务器上使用 `sudo` —— 这是默认设置），对于 Cisco IOS，我们必须将其设置为 `enable`。

有了这些，你已经学会了连接到网络设备所需的步骤。

为了验证连接是否按预期工作（假设你可以访问运行 Cisco IOS 的路由器），你可以运行这个简单的 playbook，名为 `ios_facts.yaml`：

```
---
- name: Play to return facts from a Cisco IOS device
  hosts: routers
  gather_facts: False
  tasks:
  - name: Gather IOS facts
    cisco.ios.ios_facts:
      gather_subset: all
```

你可以通过使用如下命令运行此操作：

```
$ ansible-playbook -i hosts ios_facts.yml --ask-pass
```

如果返回成功，这意味着你的配置是正确的，并且你已经成功地授权 Ansible 管理你的 IOS 设备。

类似地，如果你想连接到 Cumulus VX 设备，你可以添加另一个名为 `group_vars/cumulusvx.yml` 的组变量文件，文件内容如下：

```
---
ansible_user: cumulus
become: false
```

一个类似的 playbook，将返回有关我们 Cumulus VX 交换机的所有事实，可能如下所示：

```
---
- name: Simply play to gather Cumulus VX switch facts
  hosts: cumulusvx
  gather_facts: no
  tasks:
  - name: Gather facts
    ansible.builtin.setup:
      gather_subset: all
```

你可以通过使用如下命令以正常方式运行：

```
$ ansible-playbook -i hosts cumulusvx_facts.yml --ask-pass
```

如果成功，你应该会看到 playbook 运行的以下输出：

```
SSH password:
PLAY [Simply play to gather Cumulus VX switch facts] ************************************************************************************************
TASK [Gather facts] ************************************************************************************************
ok: [vx01.example.org]
PLAY RECAP ************************************************************************************************
vx01.example.org : ok=1 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

这展示了在 Ansible 中连接两种不同类型网络设备的技巧，其中一种你可以自己测试，而无需特殊硬件。

现在，让我们继续探讨如何在 Ansible 中为网络设备设置环境变量。

## 网络设备的环境变量

网络的复杂性通常较高，而且网络系统非常多样化。由于这些原因，Ansible 提供了许多变量，可以帮助你进行调整，使 Ansible 适应你的环境。

假设你有两个不同的网络（一个用于计算，另一个用于网络设备），它们无法直接通信，但必须通过跳板主机才能相互访问。由于我们在计算网络中有 Ansible，我们必须通过跳板主机跳跃网络，以配置管理网络中的 IOS 路由器。同时，我们的目标交换机需要设置代理才能访问互联网。

要连接到管理网络中的 IOS 路由器，我们需要为网络设备创建一个新组，这些设备位于单独的网络中。对于此示例，可能需要如下指定：

```
[bastion_routers]
n1.example.org
n2.example.org
[bastion_cumulusvx]
vx01.example.org
```

在创建更新后的清单之后，我们可以创建一个新的组变量文件，例如 `group_vars/bastion_routers.yaml`，内容如下：

```
---
ansible_connection: network_cli
ansible_network_os: cisco.ios.ios
ansible_become: True
ansible_become_method: enable
ansible_ssh_common_args: '-o ProxyCommand="ssh -W %h:%p -q bastion.example.org"'
proxy_env:
http_proxy: http://proxy.example.org:8080
```

如果我们的 Cumulus VX 交换机位于堡垒服务器后面，我们也可以对它们执行相同的操作，方法是创建一个 `group_vars/bastion_cumulusvx.yml` 文件：

```
---
ansible_user: cumulus
ansible_become: false
ansible_ssh_common_args: '-o ProxyCommand="ssh -W %h:%p -q bastion.example.org"'
proxy_env:
http_proxy: http://proxy.example.org:8080
```

除了我们在上一节讨论的选项外，我们现在有两个额外的选项：

+   `ansible_ssh_common_args`：这是一个非常强大的选项，允许我们在 SSH 连接中添加额外的选项来调整其行为。这些选项应该很容易识别，因为你已经在 SSH 配置中使用它们，仅仅是为了通过 SSH 连接到目标机器。在此特定情况下，我们添加了 `ProxyCommand`，这是一个 SSH 指令，用于跳转到一个主机（通常是堡垒主机），以便我们可以安全地进入目标主机。

+   `http_proxy`：此选项位于 `proxy_env` 选项下，在网络隔离较强的环境中至关重要，因此，除非使用代理，否则你的机器无法与互联网互动。

假设你已经设置了无密码访问（例如，基于 SSH 密钥的访问）到你的堡垒主机，你应该能够对你的 Cumulus VX 主机运行一个临时的 Ansible `ping` 命令，如下所示：

```
$ ansible -i hosts -m ping -u cumulus bastion_cumulusvx
vx01.example.org | SUCCESS => {
    "ansible_facts": {
    "discovered_interpreter_python": "/usr/bin/python"
  },
  "changed": false,
  "ping": "pong"
}
```

请注意，使用堡垒服务器变得透明——你可以像在同一局域网中一样使用 Ansible 自动化。如果你能访问一个基于 Cisco IOS 的设备，你应该能够对 `bastion_routers` 组运行类似的命令，并获得相似的良好结果。

现在你已经学会了如何为网络设备设置环境变量并通过 Ansible 访问它们，即使它们位于隔离网络中，接下来我们将学习如何为网络设备设置条件语句。

# 针对网络设备的自定义条件语句

尽管没有专门针对网络的 Ansible 条件语句，但在与网络相关的 Ansible 使用中，条件语句经常发挥作用。

在网络中，启用和禁用端口是常见的操作。为了让数据能够通过电缆传输，电缆两端的端口都应该启用，从而实现连接状态（一些供应商会使用不同的名称，但基本概念相同）。

假设我们有两台 Arista Networks EOS 设备，并且我们在端口上设置了 ON 状态，需要等待连接建立后才能继续。

要等待 `Ethernet4` 接口启用，我们需要在我们的剧本中添加以下任务：

```
- name: Wait for interface to be enabled
  arista.eos.eos_command:
    commands:
    - show interface Ethernet4 | json
    wait_for:
    - "result[0].interfaces.Ethernet4.interfaceStatus eq connected"
```

`arista.eos.eos_command`是一个模块，它允许我们向 Arista Networks EOS 设备发布自由格式的命令。命令本身需要在`commands`选项中以数组形式指定。通过`wait_for`选项，我们可以指定一个条件，Ansible 将反复执行指定的任务，直到条件满足为止。由于命令的输出被重定向到`json`实用工具，因此输出将是 JSON 格式，我们可以利用 Ansible 处理 JSON 数据的能力来遍历其结构。

我们可以在 Cumulus VX 上实现类似的结果——例如，我们可以查询从交换机收集到的事实，查看`swp2`端口是否已启用。如果没有启用，则启用该端口；如果已经启用，则跳过该命令。我们可以通过一个简单的剧本来实现，如下所示：

```
---
- name: Simple play to demonstrate conditional on Cumulus Linux
  hosts: cumulusvx
  tasks:
  - name: Enable swp2 if it is disabled
    community.network.nclu:
      commands:
      - add int swp2
    commit: true
    when: not ansible_swp2.active
```

请注意我们任务中`when`子句的使用，意味着只有在`swp2`端口未激活时，我们才会发布配置指令。如果我们首次在未配置的 Cumulus Linux 交换机上运行此剧本，我们应该看到类似以下的输出：

```
PLAY [Simple play to demonstrate conditional on Cumulus Linux] ***************************************************************
TASK [Gathering Facts]
***************************************************************
ok: [vx01.example.org]
TASK [Enable swp2 if it is disabled] ***************************************************************
changed: [vx01.example.org]
PLAY RECAP
***************************************************************
vx01.example.org : ok=2 changed=1 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

如我们所见，`Enable swp2`任务如果在`community.network.nclu`模块的基础上被禁用，会返回`changed`状态，这意味着它更改了交换机的配置。然而，如果我们第二次运行该剧本，输出应该更像这样：

```
PLAY [Simple play to demonstrate conditional on Cumulus Linux] ***************************************************************
TASK [Gathering Facts]
***************************************************************
ok: [vx01.example.org]
TASK [Enable swp2 if it is disabled] ***************************************************************
skipping: [vx01.example.org]
PLAY RECAP
***************************************************************
vx01.example.org : ok=1 changed=0 unreachable=0 failed=0 skipped=1 rescued=0 ignored=0
```

这次任务被跳过了，因为 Ansible 的事实显示`swp2`端口已经启用。这个例子虽然简单，但展示了如何在网络设备上使用条件语句，就像你在本书前面看到的 Linux 服务器上的条件语句一样。

这就是我们对使用 Ansible 进行网络设备自动化的简要介绍——更深入的工作需要查看网络配置，并且需要更多的硬件，这超出了本书的范围。不过，我希望这些信息能展示 Ansible 如何有效地用于自动化和配置各种网络设备。

# 总结

现代大规模基础设施的快速变化需要网络任务的自动化。幸运的是，Ansible 支持多种网络设备，从专有硬件如基于 Cisco IOS 的设备，到开放标准如运行 Cumulus Linux 操作系统的白盒交换机。Ansible 是一个强大且支持广泛的工具，帮助管理网络配置，并能够快速、安全地实施更改。通过 Ansible 剧本，你甚至可以替换网络中的整个设备，并确保能够将正确的配置应用到新设备上。

在本章中，你了解了自动化网络管理的原因。接着，你学习了 Ansible 如何管理网络设备，如何在 Ansible 中启用网络自动化，以及如何找到执行所需自动化任务的 Ansible 模块。然后，通过实际示例，你学习了如何连接到网络设备，设置环境变量（并通过堡垒主机连接到隔离的网络），以及如何在 Ansible 任务中应用条件语句来配置网络设备。

在下一章中，我们将学习如何使用 Ansible 管理 Linux 容器和云基础设施。

# 问题

1.  以下哪一项**不是**Ansible 用于连接网络设备的四种主要连接类型之一？

    1.  `netconf`

    1.  `network_cli`

    1.  `local`

    1.  `netstat`

    1.  `httpapi`

1.  对还是错：`ansible_network_os`变量被 Ansible 用来了解我们将要使用的设备系列。

    1.  正确

    1.  错误

1.  对还是错：要连接到一个位于独立网络中的 IOS 路由器，你需要为主机指定特殊的连接变量，可能作为清单组变量。

    1.  正确

    1.  错误

# 进一步阅读

关于 Ansible 网络的官方文档可以在这里查看：[`docs.ansible.com/ansible/latest/network/index.xhtml`](https://docs.ansible.com/ansible/latest/network/index.xhtml)。
