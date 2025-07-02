# 第十二章：12  

# 扩展 VMware 部署  

现在我们已经了解如何在 AWS 中启动网络和服务，接下来我们将讨论如何在**VMware**环境中部署类似的设置，并深入探讨核心 VMware 模块。  

本章将涵盖以下主题：  

+   VMware 介绍  

+   VMware REST 模块  

# 技术要求  

本章将讨论 VMware 产品家族的各种组件，以及如何使用 Ansible 与它们交互。尽管本章中会有示例的 Playbook 任务，但这些任务可能需要根据您的安装情况进行调整。因此，不建议在未先审查完整文档的情况下使用本章中的任何示例。  

# VMware 介绍  

VMware 拥有超过 25 年的历史，从一个隐形的初创公司发展到今天，经历了显著的变化。到 2023 年 8 月，VMware 的收入已超过 130 亿美元，VMware 的产品组合已发展到涵盖大约 30 种产品，最为人熟知的是其虚拟机监控程序（Hypervisor），并且它是大多数企业的标准配置，使管理员能够在各种基于 x86 的硬件配置上快速部署虚拟机。  

然而，最近的变化发生在博通于 2023 年底收购 VMware 之后，变化十分显著。  

这次收购极大简化了 VMware 的产品组合，这是受到客户和合作伙伴反馈的影响，使各种规模的用户都能从 VMware 解决方案中获得更多价值。两个显著的产品包括 VMware Cloud Foundation 和 VMware vSphere Foundation，每个都有高级附加产品。  

博通实施的首项重大变化是将 VMware 转变为基于订阅的模式。这与云计算消费的行业标准一致，旨在通过逐步淘汰永久许可并用订阅或期限许可证替代，来为客户提供持续的创新、更快的价值实现时间和可预测的投资，从而支持客户和合作伙伴在数字化转型中的成功。

在更广泛的行业中，关于博通收购 VMware 后的战略存在一些担忧。有人猜测博通可能会专注于保留仅有的最大和最盈利的 VMware 客户和合作伙伴。此策略可能导致 VMware 产品组合的重组，以更好地与博通的业务目标对接，可能包括资产处置和更简化的产品范围。  

在写作时（2024 年初），这些变化对 VMware 现有客户群和合作伙伴生态系统的影响尚不清楚，更多细节预计将在全年逐步浮现，因为博通将继续实施其针对 VMware 的战略长期计划。  

# VMware REST 模块  

如前所述，VMware 产品系列中大约有 30 个产品，而 Ansible 拥有可以与其中许多产品交互的模块。  

然而，由于产品简化，我们将集中讨论 `vmware.vmware_rest` 命名空间中的模块，而不会查看 `community.vmware` 中的任何模块，因为这些模块将在 2025 年某个时候失去所有支持。

这两组模块的区别在于，正如其名称所示，`vmware.vmware_rest` 模块使用 VMware REST API 来管理资源，而 `community.vmware` 中的模块则使用 Python 库与各种 VMware 端点进行交互以执行任务。

`vmware.vmware_rest` 命名空间中的模块分为三个领域：

+   **Appliance**：这些模块管理你的 vCenter Appliance，它们是组成 vCenter 部署的基础资源。

+   **Content**：内容库模块允许你管理定义和管理库中项、订阅、发布和存储的服务。

+   **vCenter**：这些模块允许你管理运行在 vCenter 部署上的工作负载，例如虚拟机。

让我们先来看一下 VMware REST appliance 模块。

## VMware REST appliance 模块

在撰写本文时，已有超过 60 个模块；这些模块被划分到各自清晰标记的区域。

### 访问模块

首先，我们有访问模块：

+   `appliance_access_consolecli`：此模块允许你启用或禁用基于控制台的 CLI（TTY1）。

+   `appliance_access_consolecli_info`：这个模块返回基于控制台的 CLI（TTY1）的当前状态；它要么是启用，要么是禁用。

+   `appliance_access_dcui`：使用此模块，你可以配置 **Direct Console User Interface**（**DCUI** TTY2）的状态；同样，你只有两个选项：启用或禁用。

+   `appliance_access_dcui_info`：正如你可能已经猜到的，这个模块返回 DCUI TTY2 状态的启用或禁用。

+   `appliance_access_shell`：同样，这个模块的功能很简单，就是改变 BASH 的启用状态。启用后，你可以在 CLI 中访问 BASH shell。

+   `appliance_access_shell_info`：此模块仅返回 BASH 访问状态；它要么是启用，要么是禁用。

+   `appliance_access_ssh`：此模块设置基于 SSH 控制的 CLI 的启用状态。

+   `appliance_access_ssh_info`：这个模块返回基于 SSH 控制的 CLI 的启用状态。

如前所述，这些模块中的每一个要么允许你设置访问系统的状态，要么返回当前配置的状态：

```
- name: "Enable SSH access"
  vmware.vmware_rest.appliance_access_ssh:
    enabled: true
  register: access_ssh_result
```

每个非信息类模块都有一个值为 `enabled`，其值可以是 `true` 或 `false`，如前所示。

### 健康信息模块

下一个模块组仅返回关于系统健康状态的信息：

+   `appliance_health_applmgmt_info`

+   `appliance_health_database_info`

+   `appliance_health_databasestorage_info`

+   `appliance_health_load_info`

+   `appliance_health_mem_info`

+   `appliance_health_softwarepackages_info`

+   `appliance_health_storage_info`

+   `appliance_health_swap_info`

+   `appliance_health_system_info`

您可以像这样调用其中一个模块:

```
- name: "Get the system health status"
  vmware.vmware_rest.appliance_health_system_info:
  register: health_system_result
```

这将返回您查询的任何服务的当前健康状态。

### Infraprofile 模块

在这里，我们只有两个模块:

+   `appliance_infraprofile_configs`: 此模块导出所选配置文件

+   `appliance_infraprofile_configs_info`: 此模块列出所有已注册的配置文件

`appliance_infraprofile_configs`模块的唯一有效状态是`export`:

```
- name: "Export the ApplianceManagement profile"
  vmware.vmware_rest.appliance_infraprofile_configs:
    state: "export"
    profiles:
    - "ApplianceManagement"
  register: infraprofile_configs_result
```

这里的输出是包含所选配置文件的 JSON。在上述示例中，这是`ApplianceManagement`。

### 本地帐户模块

在这里，我们有三个模块:

+   `appliance_localaccounts_globalpolicy`

+   `appliance_localaccounts_globalpolicy_info`

+   `appliance_localaccounts_info`

这些模块允许您设置和查询全局策略，并返回所有或只返回一个本地帐户的信息。

### 监控模块

虽然这里只有两个模块，但是当您组合它们时它们可以非常强大:

+   `appliance_monitoring_info`: 此模块返回一组监视器

+   `appliance_monitoring_query`: 此模块允许您查询监视器

这里是一个示例查询:

```
- name: "Query the monitoring backend"
  vmware.vmware_rest.appliance_monitoring_query:
    start_time: "2024-01-01 09:00:00+00:00"
    end_time: "2024-01-01 10:00:00+00:00"
    names:
    - "mem.total"
    interval: "MINUTES5"
    function: "AVG"
  register: mem_total_result
```

如您所见，在前述任务中，我们正在查询 2024 年 1 月 1 日上午 9 点至 10 点之间的总内存量，耗时 5 分钟。

### 网络模块

这是事情开始变得有点复杂的地方; 每个模块都有一个*info*等效项，其中突出显示:

+   `appliance_networking`（以及信息）: 此模块重置和重新启动所有接口的网络配置。它还更新 DHCP 分配的 DHCP IP 地址的租约。

+   `appliance_networking_dns_domains`（以及信息）: 此模块用于管理 DNS 搜索域。

+   `appliance_networking_dns_hostname`（以及信息）: 此模块配置**完全合格的域名**（**FQDN**）主机名。

+   `appliance_networking_dns_servers`（以及信息）: 此模块可以管理 DNS 服务器配置。

+   `appliance_networking_firewall_inbound`（以及信息）: 此模块设置防火墙规则的有序列表。

+   `appliance_networking_interfaces_info`: 此模块获取单个网络接口的信息。

+   `appliance_networking_interfaces_ipv4`（以及信息）: 此模块管理指定网络接口的 IPv4 网络配置。

+   `appliance_networking_interfaces_ipv6`（以及信息）: 此模块管理指定网络接口的 IPv6 网络配置。

+   `appliance_networking_noproxy`（以及信息）: 此模块配置不应应用代理配置的服务器。

+   `Appliance_networking_proxy`（以及信息）: 此模块配置用于指定协议的代理服务器。

### 时间和日期模块

下列模块在某种程度上影响时间和日期设置:

+   `appliance_ntp`（以及信息）: 此模块管理 NTP 服务器配置。

+   `appliance_system_time_info`: 此模块获取系统时间

+   `appliance_system_time_timezone`（以及信息）: 此模块设置时区

+   `appliance_timesync module`（附加信息）：此模块配置时间同步模式

### 其余模块

其余模块涵盖设备配置和管理：

+   `appliance_services`（附加信息）：您可以使用此模块重启给定的服务

+   `appliance_shutdown`（附加信息）：此模块允许您取消挂起的关机操作

+   `appliance_system_globalfips`（附加信息）：使用此模块，您可以启用或禁用设备的全局 FIPS 模式

+   `appliance_system_storage`（附加信息）：此模块将所有分区调整为磁盘大小的 100%

+   `appliance_system_version_info`：此模块获取版本信息

+   `appliance_update_info`：此模块获取设备更新的状态

+   `appliance_vmon_service`（附加信息）：此模块列出 vmon 管理的服务的详细信息

这部分内容到此结束。接下来，我们将查看内容模块。

## VMware REST 内容模块

有少数几个模块允许您管理和收集有关内容库的信息：

+   `content_configuration`（附加信息）：此模块用于更新配置

+   `content_library_item_info`：此模块在提供标识符时返回 `{@link ItemModel}`

+   `content_locallibrary`（附加信息）：此模块用于创建一个新的本地库

+   `content_subscribedlibrary`（附加信息）：此模块用于创建一个新的订阅

## vCenter 模块

在这里，您将看到更有趣的内容。使用这些模块，您可以启动、配置并管理虚拟机的整个生命周期。在查看虚拟机之前，我们将先看看一些支持 vCenter 的模块。

### 支持的 vCenter 模块

这些支持模块允许您管理托管在 vCenter 中的资源池、数据中心、文件夹和数据存储：

+   `vcenter_cluster_info`：此模块检索与 `{@``param.name cluster_name}` 相对应的集群信息

+   `vcenter_datacenter`（附加信息）：此模块向您的 vCenter 库中添加一个新的数据中心

+   `vcenter_datastore_info`：此模块获取有关数据存储的信息，使用 `{@``param.name datastore_name}`

+   `vcenter_folder_info`：此模块检索最多 1,000 个文件夹的详细信息，这些文件夹符合 `{@link FilterSpec}` 并且当前用户有权限查看

+   `vcenter_host`（附加信息）：此模块可用于将新的独立主机添加到您的 vCenter

+   `vcenter_network_info`：此模块返回关于 vCenter 中前 1,000 个可见网络的信息，具体取决于您的权限，符合 `{@link FilterSpec}`

+   `vcenter_ovf_libraryitem`：此模块用于从虚拟机或虚拟设备创建内容库中的项目

+   `vcenter_resourcepool`（附加信息）：此模块用于部署资源池

+   `vcenter_storage_policies_info`：此模块获取 vCenter 中可用的存储策略信息；最多返回 1,024 条结果

### 虚拟机模块

最后一组模块涉及创建和管理虚拟机及其关联资源。我们先来看看主要的模块 `vcenter_vm`。

`vcenter_vm` 模块用于创建虚拟机。例如，一个基本的任务如下所示：

```
- name: "Create a Virtual Machine"
  vmware.vmware_rest.vcenter_vm:
    placement:
      cluster: "{{ lookup('vmware.vmware_rest.cluster_moid', '/learnansible_dc/host/learnansible_cluster') }}"
folder: "{{ lookup('vmware.vmware_rest.folder_moid', '/learnansible_dc/vm') }}"
      resource_pool: "{{ lookup('vmware.vmware_rest.resource_pool_moid', '/learnansible_dc/host/learnansible_cluster/Resources') }}"
    name: "LearnAnsibleVM"
    guest_OS: "UBUNTU_64"
    hardware_version: "VMX_11"
    memory:
      hot_add_enabled: true
      size_MiB: 4000
  register: LearnAnsibleVM_output
```

如你所见，我们使用了不同的查找模块来查找集群、数据存储、文件夹和资源池的 ID——如果我们有这些信息，可以直接提供这些 ID。

一旦虚拟机创建完成，我们可以使用其余的模块进一步配置虚拟机或管理其状态：

+   `vcenter_vm_guest_customization`：该模块对虚拟机应用客制化设置，例如运行脚本。

+   `vcenter_vm_guest_filesystem_directories`：使用该模块，你可以在虚拟机操作系统中创建一个目录。

+   `vcenter_vm_guest_identity_info`：该模块获取关于虚拟机的身份信息。

+   `vcenter_vm_guest_localfilesystem_info`：该模块获取虚拟机操作系统中本地文件系统的详细信息。

+   `vcenter_vm_guest_networking_info`：该模块获取虚拟机操作系统中网络配置的详细信息。

+   `vcenter_vm_guest_networking_interfaces_info`：该模块显示虚拟机操作系统中的网络接口信息。

+   `vcenter_vm_guest_networking_routes_info`：该模块显示虚拟机操作系统中的网络路由信息。

+   `vcenter_vm_guest_operations_info`：该模块获取关于虚拟机操作系统状态的信息。

+   `vcenter_vm_guest_power`（附加信息）：该模块请求从虚拟机操作系统内部进行软关机、待机（挂起）或软重启。

+   `vcenter_vm_hardware`：该模块用于更新所请求虚拟机的硬件设置。

+   `vcenter_vm_hardware_adapter_sata`（附加信息）：该模块配置一个虚拟 SATA 适配器。

+   `vcenter_vm_hardware_adapter_scsi`（附加信息）：该模块添加一个虚拟 SCSI 适配器。

+   `vcenter_vm_hardware_boot`（附加信息）：该模块用于管理虚拟机与启动相关的设置。

+   `vcenter_vm_hardware_boot_device`（附加信息）：该模块可以设置将作为启动驱动器使用的虚拟设备。

+   `vcenter_vm_hardware_cdrom`（附加信息）：该模块将一个虚拟 CD-ROM 附加到虚拟机。

+   `vcenter_vm_hardware_cpu`（附加信息）：该模块管理虚拟机的 CPU 设置。

+   `vcenter_vm_hardware_disk`（附加信息）：该模块将虚拟磁盘连接到虚拟机。

+   `vcenter_vm_hardware_ethernet`（附加信息）：该模块将虚拟以太网适配器连接到虚拟机。

+   `vcenter_vm_hardware_floppy`（附加信息）：该模块向虚拟机添加一个虚拟软盘驱动器。

+   `vcenter_vm_hardware_info`：该模块获取虚拟机的虚拟硬件设置信息。

+   `vcenter_vm_hardware_memory`（附加信息）：该模块配置虚拟机的内存设置。

+   `vcenter_vm_hardware_parallel`（附加信息）：此模块添加虚拟并行端口。

+   `vcenter_vm_hardware_serial`（附加信息）：此模块添加虚拟串行端口。

+   `vcenter_vm_info`：此模块返回关于虚拟机的信息。

+   `vcenter_vm_libraryitem_info`：此模块检索与你的虚拟机相关联的库项信息。

+   `vcenter_vm_power`（附加信息）：此模块对虚拟机执行启动、硬关机、硬重置或硬暂停操作——也就是说，它按下虚拟机前面的电源按钮。

+   `vcenter_vm_storage_policy`（附加信息）：此模块更新虚拟机虚拟硬盘的存储策略。

+   `vcenter_vm_storage_policy_compliance`：此模块更新并收集关于虚拟机存储策略合规性的相关信息。

+   `vcenter_vm_tools`（附加信息）：此模块用于管理 VMware Tools 的配置。

+   `vcenter_vm_tools_installer`（附加信息）：此模块将 VMware Tools 安装程序作为 CD-ROM 挂载，使其在虚拟机操作系统中可用。

+   `vcenter_vmtemplate_libraryitems`（附加信息）：此模块创建并返回内容库中项目信息。

如你所见，`vmware.vmware_rest` 集合提供了全面的虚拟机资源管理支持，更棒的是，这些模块都是设计来使用官方 REST API，这意味着无论你是使用命令行界面、Web 界面还是 Ansible，你都可以安全地混合搭配管理 VMware 资源的方式。所有内容都是通过相同的 REST API 进行管理。

# 总结

如你所见，从这一长串模块中，你可以使用 Ansible 完成作为 VMware 管理员每天需要执行的大部分管理和配置任务。

再加上我们在*第八章*中查看的模块，*Ansible 网络模块*，用于管理网络设备，以及支持如 NetApp 存储设备等硬件的模块。

通过这样做，你可以构建跨越物理设备、VMware 元素和你本地企业级虚拟化基础设施中运行的虚拟机的复杂剧本。

正如本章开始时提到的，写作时 VMware 正处于动荡之中。本章旨在展示可能的操作方法，而不是为你提供一个实际的 VMware 资源管理实践指南。有关 `vmware.vmware_rest` 集合当前状态的更多详情，请访问 [`galaxy.ansible.com/ui/repo/published/vmware/vmware_rest/`](https://galaxy.ansible.com/ui/repo/published/vmware/vmware_rest/)。

在下一章，我们将讨论如何通过扫描常见问题和潜在安全问题，确保我们的剧本遵循最佳实践。

# 第四部分：Ansible 工作流

在本书的最后部分，您将学习高级 Ansible 工作流和最佳实践，包括安全实践、剧本扫描、服务器加固、CI/CD 集成、Ansible AWX 和 Red Hat Ansible 自动化平台。最终，您将掌握在实际场景中有效利用 Ansible 所需的知识。

本部分包含以下章节：

+   *第十三章*，*扫描您的 Ansible 剧本*

+   *第十四章*，*使用 Ansible 强化您的服务器*

+   *第十五章*，*将 Ansible 与 GitHub Actions 和 Azure DevOps 结合使用*

+   *第十六章*，*介绍 Ansible AWX 和 Red Hat Ansible 自动化平台*

+   *第十七章*，*与 Ansible 的下一步*
