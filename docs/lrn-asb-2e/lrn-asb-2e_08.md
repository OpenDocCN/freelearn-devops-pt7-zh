# 8

# Ansible 网络模块

欢迎来到*第八章*；本章将探讨 Ansible 的一个常被忽视的使用案例，并深入了解 Ansible 庞大的网络模块社区。

本章将提供一个概述，而不是深入探讨每个集合——因为那可能需要一本完整的书——重点介绍这些模块的功能和灵活性。

本章讨论以下内容：

+   制造商和设备支持

# 制造商和设备支持

到目前为止，我们一直在关注与服务器交互的模块。在我们的案例中，这些模块大多数是本地运行的。在接下来的章节中，我们将更多地与远程云托管的服务器进行通信。但是，在与远程服务器交互之前，我们应该先了解核心网络模块。

这些模块的设计旨在与各种网络设备交互并管理其配置，从传统的机架顶部交换机和完全虚拟化的网络基础设施，到防火墙和负载均衡器。Ansible 支持许多设备，从开源虚拟设备到硬件解决方案，其中一些设备的起始价格可能超过 50 万美元，具体取决于你的配置。

那么，这些集合和模块有什么共同点呢？

这些模块的共同点是，它们都与传统上配置复杂的设备进行交互，而这些设备在大多数部署和环境中，既是核心也是关键元素；毕竟，所有与它们连接的设备都需要某种程度的网络连接性。

这些模块为许多设备提供了一个标准接口，即 Ansible，避免了工程师直接访问这些设备的需要。相反，他们可以运行 Ansible playbook，这些 playbook 由经验丰富的网络工程师创建角色，以受控且一致的方式配置设备，只需更改几个变量。

使用 Ansible 管理这些关键核心基础设施的唯一缺点是，运行 Ansible 的主机需要能够访问设备上的管理接口或 API，这有时会引发一些安全问题。因此，Ansible 提供的关于如何管理网络设备的指南需要经过深思熟虑。

## 这些集合

以下是按命名空间顺序列出的集合；每个条目末尾列出了**[命名空间.集合名称]**。

### Apstra 可扩展操作系统 (EOS) [arista.eos]

有超过 30 个模块可以让你管理运行 EOS 的设备。这些模块可以让你操作**访问控制列表**（**ACLs**）接口，配置**边界网关协议**（**BGP**）设置，在设备上运行任意命令，管理主机名、接口配置、日志记录等。此外，还有一个模块允许你从每个设备收集事实数据。

此外，还提供了用于命令行和 HTTP API 交互的插件。

### Check Point [check_point.mgmt]

Check Point 管理的 Ansible 集合包含许多模块；截至写作时，已有超过 250 个模块。

每个模块管理你 Check Point 设备的不同方面，如访问层、规则、管理员或通过 Web 服务 API 管理 Check Point 防火墙上的网络流量源。它们提供的功能包括从获取事实和添加或管理对象，到工作流功能，如批准和发布 Check Point 防火墙上的会话。

### Cisco

考虑到 Cisco 设备类型和类别的数量，Cisco 命名空间中有多个集合。

#### Cisco 应用程序中心基础设施（ACI）[cisco.aci]

150 多个 ACI 模块用于管理 Cisco ACI 的各个方面，这是 Cisco 下一代 API 驱动网络堆栈的预期功能。

有多个模块可用于管理 Cisco ACI 的不同方面，如 AAAA 记录（这些是存储 IPv6 地址的地址记录）、角色、用户、证书、访问 SPAN 配置、**桥接域**（**BDs**）和子网、BGP 路由反射器等。还有用于管理云应用程序配置文件和云 AWS 提供商配置的模块。

#### Cisco 自适应安全设备（ASA）[cisco.asa]

通过五个 ASA 模块，你可以管理访问控制列表、运行命令并管理物理和虚拟 Cisco ASA 设备的配置。

#### Cisco DNA Center (DNAC) [cisco.dnac]

Cisco DNAC 的 Ansible 集合包括近 400 个模块，用于管理 Cisco DNAC 部署的不同方面。这些模块涵盖了从获取接入点配置详情、管理应用策略、将设备分配到站点、导入认证证书、运行合规性检查，到管理配置模板等多个功能。

#### Cisco IOS 和 IOS XR [cisco.ios 和 cisco.iosxr]

这两个集合包含用于管理 Cisco IOS 和 IOS XR 驱动的设备的模块。你可以使用它们收集设备信息，并配置用户、接口、日志、横幅等。

#### 身份服务引擎（ISE）[cisco.ise]

该集合管理你的 ISE；它包含多个模块，用于管理设置和配置，例如处理 ACI 绑定和设置、管理 Active Directory 设置、处理允许的协议、管理 ANC 端点和策略、管理备份配置和计划、处理证书等。

#### Cisco Meraki [cisco.meraki]

在这里，我们有近 500 个模块，用于管理 Meraki 部署的不同元素，如管理的身份、设备详情、摄像头设置、蜂窝网关配置和传感器关系。每个模块都旨在获取信息或修改设置，帮助你通过自动化管理 Cisco Meraki 设备。

#### Cisco 网络服务编排器（NSO）[cisco.nso]

一些模块允许你与 Cisco NSO 管理的设备进行交互。你可以执行 NSO 操作、查询你的安装数据，并验证你的配置，同时进行服务同步和配置。

#### Cisco 网络操作系统软件 (NX-OS) [cisco.nxos]

如你所想，管理运行 Cisco NXOS 的设备的模块非常多；有超过 80 个模块，涵盖了诸如管理 AAA 服务器配置、ACL、BGP 配置、执行任意命令、管理接口以及处理 Cisco NX-OS 设备上的各种其他配置和设置等功能。

#### Cisco 统一计算系统 (UCS) [cisco.ucs]

尽管这些模块不严格属于网络设备，但管理 Cisco 统一计算、存储和网络系统的模块包括一个允许你管理 DNS 服务器、IP 地址池、局域网连接策略、MAC 地址池、QoS 设置、VLAN 和 vNIC 的模块。其余模块允许你以编程方式管理刀片和机箱中的计算和存储。

### F5 BIG-IP 命令式 [F5Networks.F5_Modules]

有 160 个模块，所有模块都以 BIG-IP 为前缀，允许你管理 F5 BIG-IP 应用交付控制器的各个方面。

### Fortinet

Fortinet 命名空间中只有两个集合，但正如你从每个集合中的模块数量中看到的，它们功能非常丰富。

#### Fortinet FortiManager [fortinet.fortimanager]

总共有超过 1,100 个模块（没错，你没看错），包括配置防病毒配置文件和选项、管理 AP 本地配置文件和命令列表、配置自定义应用程序签名和防火墙应用程序组、管理互联网服务应用程序等。

#### Fortinet FortiOS v6 (fortinet.fortios)

尽管这个集合的模块比 FortiManager 集合少，但仍有超过 650 个模块用于配置防病毒设置、应用程序控制列表、身份验证方案和证书设置。

### Free Range Routing (FRR) [Frr.Frr]

这里只有两个模块：一个允许你配置 BGP，另一个让你收集运行 FRR 的设备的事实数据。

### Juniper Networks Junos [junipernetworks.junos]

总共有 40 个模块使你能够在播放本中与运行 Junos 的 Juniper 设备进行交互。这些模块包括标准的命令、配置和事实收集模块，也包括允许你安装包并将文件复制到设备上的模块。

### Open vSwitch [Openvswitch.Openvswitch]

该命名空间中的四个模块允许你管理 OVS 虚拟交换机上的绑定、桥接、端口和数据库。

### VyOS [vyos.vyos]

VyOS 集合包括用于管理 VyOS 设备上各种配置和资源的模块。 其中一些模块包括管理多行横幅、配置 BGP 全局设置和地址族设置、运行命令、管理防火墙设置、接口配置、日志记录、NTP、OSPF、SNMP、静态路由、系统命令、用户管理和 VLAN 配置等。

### 社区网络集合 [Community.Network]

此集合是所有没有专用命名空间或开发团队的其他网络模块的集合；模块前缀现在在方括号中。

#### A10 Networks [a10]

A10 模块支持 A10 Networks 的 AX、SoftAX、Thunder 和 vThunder 设备。 这些都是提供负载均衡的应用交付平台。

#### Cisco AireOS [aireos]

两个 AireOS 模块允许您与运行 AireOS 的 Cisco 无线局域网控制器进行交互。 其中一个模块将使您能够直接在设备上运行命令，另一个用于管理配置。

#### APCON [apcon]

一个模块允许您在 APCON 设备上运行命令。

#### Aruba 移动控制器 [aruba]

只有两个 Aruba 模块。 这些模块允许您管理 Hewlett Packard 的 Aruba 移动控制器的配置并执行命令。

#### Avi Networks [avi]

总共有 65 个 Avi 模块，允许您与 Avi 应用服务平台的所有方面进行交互，包括负载均衡和 **Web 应用防火墙** (**WAF**) 功能。

#### Big Cloud Fabric 和 Big Switch Network [bcf + bigmon]

有三个 Big Switch Network 模块。**Big Cloud Fabric** (**BCF**) 允许您创建和删除 BCF 交换机。其余两个模块使您能够创建 **Big Monitoring Fabric** (**Big Mon**) 服务链和策略。

#### Huawei Cloud Engine [ce]

超过 75 个 Cloud Engine 模块使您能够管理来自华为的这些强大交换机的所有方面，包括 BGP、访问控制列表、MTU、静态路由、VXLAN 和 SNMP 配置。

#### Lenovo CNOS [cnos]

有近 30 个模块，允许您管理运行 Lenovo CNOS 操作系统的设备；它们使您能够配置从 BGP 和端口聚合到 VLAG、VLAN 以及在需要时恢复出厂设置的设备。

#### Arista Cloud Vision [cv]

一个模块允许您使用配置文件配置 Arista Cloud Vision 服务器端口。

#### illumos [dladm + flowadm + ipadm]

illumos 是 Open Solaris 操作系统的一个分支。其强大的网络功能使其成为作为自建路由器或防火墙部署的完美候选。 这些模块使您能够管理接口、NetFlow 和隧道。此外，由于 illumos 是 Open Solaris 的一个分支，您的剧本应该也适用于基于 Open Solaris 的操作系统。

#### Ubiquiti EdgeOS [edgeos + edgeswitch]

EdgeOS 模块使您能够管理配置、执行临时命令并收集运行 EdgeOS 设备（如 Ubiquiti Edge 路由器）上的事实。

也有一些模块用于 Edge 交换机。

#### 联想企业网络操作系统 [enos]

有三个模块用于 Lenovo ENOS。像其他设备一样，它们允许你收集信息、执行命令并管理配置。

#### Ericsson [eccli]

这个单一模块允许你在运行爱立信命令行界面的设备上执行命令。

#### ExtremeXOS [exos + nos + slxos]

这六个模块允许你与 Extreme Networks 交换机上的 ExtremeXOS、Extreme Networks SLX-OS 和 Extreme Networks NOS 软件进行交互。

#### Cisco Firepower 威胁防御 [ftd]

一些模块允许你配置并上传/下载文件到 Cisco Firepower 威胁防御设备。

#### Itential 自动化平台 [iap]

一些模块允许你与托管在 Itential 自动化平台上的工作流进行交互，以及为混合云网络提供低代码自动化和编排。

#### Ruckus ICX 7000 [icx]

这些模块允许你配置 Ruckus ICX 7000 系列校园交换机。

#### Ingate 会话边界控制器 [ig]

虽然这些主要用于**SIP**，或者说它的全名是**会话启动协议**服务，但也有一些模块帮助配置网络元素。

#### NVIDIA 网络命令行工具 [nclu]

一个单一模块允许你在兼容设备上使用 NVIDIA 网络命令行工具管理网络接口。

#### 诺基亚 NetAct [netact]

单个模块允许你上传并应用由诺基亚 NetAct 驱动的核心和无线网络。

#### Citrix Netscaler [netscaler]

这些模块旨在管理和配置 Netscaler 设备的各个方面。它们涵盖的功能包括内容交换、**全球服务器负载均衡**（**GSLB**）、负载均衡、发出 Nitro API 请求、保存配置，以及管理服务器配置、服务、服务组和 SSL 证书密钥。

#### 诺基亚 Nuage 网络虚拟化服务平台（VSP）[nuage]

有一个单一模块允许你管理诺基亚 Nuage 网络 VSP 上的企业。

#### OpenSwitch [opx]

一个单一模块，通过利用 CPS API 在运行 OpenSwitch 的网络设备上对 YANG 对象执行指定操作。

#### Ordnance 虚拟路由器 [ordnance]

有两个模块：一个用于管理配置，另一个用于收集 Ordnance 虚拟路由器的信息。

### Pluribus Networks Netvisor OS [pn]

这 40 个模块允许你管理你的**Pluribus Networks**（**PN**）Netvisor OS 驱动的设备，从创建集群和路由器到在白盒交换机上执行命令。

#### 诺基亚网络服务路由器操作系统 [sros]

有三个模块可以让你对诺基亚网络的 SROS 设备执行命令、配置以及回滚更改。

#### Radware [vidrect]

少量模块允许你通过 vDirect 服务器管理 Radware 设备。

### Ansible Net Common [ansible.netcommon]

最终的集合是一组模块，可以视为帮助支持本章中所有设备的工具。这些模块能够 ping 目标并使用自定义提示和答案运行通用命令。

# 总结

我怀疑你们中的大多数人可能没听说过本章列出的许多设备，对于你们听说过的设备——比如思科的设备——你们可能也没有直接接触过它们，所有的配置工作都交给了网络管理员。

当我们在*第十五章*，“*使用 GitHub Actions 和 Azure DevOps 调用 Ansible*”以及*第十六章*，“*介绍 Ansible AWX 和 Red Hat Ansible 自动化平台*”中讨论通过 CI/CD 触发 Ansible 时，我们将了解一些部署选项，这些选项可能有助于缓解我们在章节开头提到的一些问题，比如关于运行 Ansible playbook 的主机需要与潜在的核心基础设施保持视距的问题。

在我们进入这些章节之前，我们将探讨如何将工作负载迁移到云端，这一过程将在下一章开始。

# 进一步阅读

+   **Ansible 集合** **索引**：[`docs.ansible.com/ansible/latest/collections/index.html`](https://docs.ansible.com/ansible/latest/collections/index.html)
