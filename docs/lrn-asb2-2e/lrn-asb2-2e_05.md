# 第五章：进入云端

在本章中，我们将学习如何使用 Ansible 在几分钟内完成基础设施的配置。在我看来，这是 Ansible 最有趣和最强大的功能之一，因为它使你能够以快速且一致的方式（重新）创建环境。当你有多个环境用于部署管道的各个阶段时，这一点尤其重要。实际上，它使你能够创建相同的环境，并在需要进行更改时保持一致，且不会带来任何痛苦。

让 Ansible 配置你的机器还有其他优点，因此我总是建议做以下事情：

+   **审计日志**：近年来，IT 行业吞并了大量其他行业，因此审计过程现在将 IT 视为一个关键部分。当审计员来 IT 部门要求获取一台服务器的历史记录时，从其创建到现在，拥有完整过程的 Ansible playbook 将非常有帮助。

+   **多个预备环境**：正如我们之前提到的，如果你有多个环境，使用 Ansible 配置服务器将对你大有帮助。

+   **迁移服务器**：当一家公司使用全球云服务提供商（如 AWS 或 DigitalOcean）时，他们通常会选择离他们的办公室或客户最近的区域来创建第一台服务器。这些提供商经常开设新的区域，如果他们的新区域靠近你，你可能会想将你的基础设施迁移或扩展到新区域。如果你手动配置了每个资源，这将是一个噩梦。

本章中，我们将大致涵盖以下主题：

+   在 **Amazon Web Services**（**AWS**）中配置机器。

+   在 DigitalOcean 中配置机器。

+   配置 Docker 容器。

大多数新机器的创建有两个阶段：

+   配置一台新机器或一组新机器。

+   运行 playbook，确保新机器正确配置，以在你的基础设施中发挥作用。

我们在前几章中已经了解了配置管理的相关内容。本章将更侧重于新机器的配置，而在配置管理方面的讨论将相对较少。

# 在云中配置资源。

有了这些，让我们进入第一个话题。今天，管理基础设施的团队有很多选择来运行他们的构建、测试和部署。像 Amazon、Rackspace 和 DigitalOcean 这样的提供商主要提供 **基础设施即服务**（**IaaS**）。当我们谈到 IaaS 时，最好谈论资源，而不是虚拟机，原因有很多：

+   那些公司允许你配置的大多数产品并不是机器，而是其他关键资源，如网络和存储。

+   最近，许多公司已经开始提供各种不同类型的计算实例，从裸金属机器到容器。

+   在一些非常简单的环境中，设置没有网络（或存储）的机器可能就足够了，但在生产环境中可能不够用。

这些公司通常提供 API、CLI、GUI 和 SDK 工具，用于创建和管理云资源的整个生命周期。我们更感兴趣的是使用他们的 SDK，因为它将在我们的自动化工作中发挥重要作用。最初，设置新服务器和配置它们可能很有趣，但在某个阶段，它可能会变得乏味，因为这类操作非常重复。每个配置步骤都将涉及多个类似的步骤，以使服务器能够启动并运行。

想象一下，有一天早晨，你收到一封电子邮件，要求为三个新客户进行设置，其中每个客户的设置有三到四个实例，以及一堆服务和依赖项。这对你来说可能是一个简单的任务，但需要多次运行相同的重复命令，随后还要监控服务器启动，确保一切顺利。此外，任何手动操作都有可能引入问题。如果两个客户的设置顺利完成，但由于疲劳，你遗漏了第三个客户的某个步骤，从而引入了问题怎么办？为了解决这种情况，自动化就派上了用场。

云资源自动化配置使得工程师可以尽快地建立一台新服务器，让她能够集中精力处理其他优先事项。通过使用 Ansible，你可以轻松地执行这些操作，并以最小的努力自动化云资源配置。Ansible 为你提供了自动化多种不同云平台的能力，例如亚马逊、Azure、DigitalOcean、谷歌云、Rackspace 等，且在 Ansible 核心或扩展模块包中提供了不同服务的模块。

### 注意

如前所述，启动新机器并不是最终目的。我们还需要确保我们配置它们以执行所需的角色。

在接下来的章节中，我们将配置我们在前几章中使用的环境（两个 Web 服务器和一个数据库服务器），并在以下环境中进行配置：

+   **简单的亚马逊 Web 服务部署**：所有机器将被放置在同一可用区和同一网络中。

+   **复杂的亚马逊 Web 服务部署**：机器将分布在多个可用区和网络中。

+   **DigitalOcean**：DigitalOcean 不允许我们进行很多网络调整，因此它将与第一个类似。

+   **Docker**：在这种情况下，我们将创建一个简单的部署。

# 亚马逊 Web 服务

亚马逊 Web 服务是使用最广泛的公共云之一，通常因为其提供的大量服务以及丰富的文档、解答问题和相关文章而被选择，这些都是如此受欢迎的产品所能期待的。

由于 AWS 的目标是成为一个完整的虚拟数据中心提供商（以及更多），我们需要像搭建真实数据中心一样创建和管理我们的网络。显然，由于这是一个虚拟数据中心，我们不需要布线。因此，几行 Ansible 剧本就足够了。

## AWS 全球基础设施

亚马逊一直对其云服务由多少数据中心组成、具体位置等信息保持较为低调。截至我写这篇文章时，AWS 拥有 13 个区域（还有 4 个区域已经规划中），共计 35 个 **可用区**（**AZ**）和超过 50 个边缘位置。亚马逊将一个区域定义为一个物理位置，亚马逊在该位置有多个可用区。根据亚马逊对可用区的定义，一个 AZ 由一个或多个独立的数据中心组成，每个数据中心都有冗余的电力、网络和连接，并位于不同的设施中。至于边缘位置，目前没有官方定义。

如你所见，从现实生活的角度来看，这些定义并不能提供太多帮助。当我尝试解释这些概念时，我通常会使用我自己创建的不同定义：

+   **区域**：由物理上相邻的多个可用区组成的群体

+   **可用区**：一个区域中的数据中心（亚马逊表示这可能不止一个数据中心，但由于没有文档列出每个 AZ 的具体布局，我假设最坏的情况）

+   **边缘位置**：互联网交换点或第三方数据中心，亚马逊在这些地方拥有 S3 和 Route 53 的端点

尽管我尽力让这些定义尽可能简单和有用，但其中一些仍然比较模糊。当我们开始讨论实际的世界差异时，这些定义会立刻变得清晰。例如，从网络速度的角度来看，当你在同一个 AZ 内移动内容时，带宽非常高。当你在同一地区的两个 AZ 之间进行相同的操作时，带宽仍然很高；然而，如果你使用两个来自不同区域的 AZ，带宽将大大降低。此外，价格也有所不同，因为同一地区内的所有流量都是免费的，而不同地区之间的流量则不免费。

## AWS 简单存储服务

Amazon S3 是第一个 AWS 服务，也是最著名的 AWS 服务之一。Amazon S3 是一个对象存储服务，具有公共端点和私人端点。它使用桶（bucket）这一概念，允许你存储不同类型的文件，并以简单的方式管理它们。Amazon S3 还提供了更多高级功能，例如使用内置的 Web 服务器来提供桶中的内容。这也是许多人决定将自己网站的内容或图片托管在 Amazon S3 上的原因之一。

S3 的主要优势有：

+   **定价模式**：按使用的千兆字节/月和传输的千兆字节计费。

+   **可靠性**：亚马逊确认 AWS S3 上的对象在任何一年内生存的概率为 99.999999999%。这个概率比任何硬盘都要高出几个数量级。

+   **工具**：由于 S3 已经推出多年，因此有许多工具已经实现以便利用该服务。

## AWS 弹性计算云（EC2）

AWS 推出的第二个服务是 EC2 服务。此服务允许您在 AWS 基础设施上启动虚拟机。可以将这些 EC2 实例视为 OpenStack 计算实例或 VMware 虚拟机。最初，这些机器与 VPS 非常相似，但过了一段时间，亚马逊决定对这些机器提供更大的灵活性，推出了非常先进的网络选项。旧款机器仍然可以在最老的数据中心找到，名为 **EC2 Classic**，而新款机器则是当前的默认版本，简称 **EC2**。

## AWS 虚拟专用云（VPC）

VPC 是我们在前文提到的亚马逊的网络实现。VPC 更像是一组工具，而不是单一工具。实际上，它提供的功能曾由经典数据中心中的多个硬件设备提供。通过 VPC，您可以创建的主要内容有：

+   交换机

+   路由器

+   DHCP

+   网关

+   防火墙

+   虚拟私人网络

在使用 VPC 时需要理解的一件重要事情是，网络的布局并非完全随意的，因为亚马逊为了简化网络设计，设置了一些限制。基本的限制有：

+   不能在可用区（AZ）之间创建子网络

+   不能在不同区域之间创建网络

+   不能直接在不同区域之间路由网络

对于前两种情况，唯一的解决方案是创建多个网络和子网络；而对于第三种情况，您实际上可以通过 VPN 服务实现一个变通方案，可以是自行提供的，也可以使用官方的 AWS VPN 服务来提供。

我们将主要使用 VPC 的交换和路由功能。

## AWS Route 53

和许多其他云服务一样，亚马逊提供了 **DNS 即服务**（**DNSaaS**）功能，在亚马逊的情况下，它叫做 **Route 53**。Route 53 是一个分布式的 DNS 服务，全球有超过 50 个端点（Route 53 存在于所有 AWS 边缘位置）。

Route 53 允许您为一个域创建不同的区域，支持分割视图（split-horizon）情况。根据客户端请求 DNS 解析时是否位于 VPC 内部或外部，会返回不同的响应。这在你希望将应用程序轻松地进出 VPC 时非常有用，而不需要更改，同时又希望尽可能让流量保持在私有（虚拟）网络中。

## AWS 弹性块存储（EBS）

AWS **EBS** 是一种块存储服务，允许您的 EC2 实例保存数据，这些数据即使重启也能得以保留，并且非常灵活。从用户角度看，EBS 很像任何其他 SAN 产品，但界面更简洁，因为您只需创建卷并告诉 EBS 需要将其附加到哪个机器，EBS 会自动完成其余操作。您可以将多个卷附加到同一台服务器，但每个卷在任何时刻只能连接到一台服务器。

## AWS 身份与访问管理

为了让您管理用户和访问方式，Amazon 提供了 **IAM** 服务。IAM 服务的主要功能包括：

+   创建、编辑和删除用户

+   更改用户密码

+   创建、编辑和删除组

+   管理用户和组关联

+   管理令牌

+   管理双重身份验证

+   管理 SSH 密钥

我们将使用此服务来设置用户及其权限。

## 亚马逊关系型数据库服务

设置和维护关系型数据库是复杂且耗时的。为简化此过程，Amazon 提供了一些广泛使用的数据库即服务（DBaaS），具体包括：

+   Aurora

+   MariaDB

+   MySQL

+   Oracle

+   PostgreSQL

+   SQL Server

对于每种数据库引擎，Amazon 提供不同的功能和定价模型，但每种的具体内容超出了本书的范围。

## 设置 AWS 账户

在开始使用 Amazon Web Services 之前，首先需要一个账户。创建 Amazon Web Services 账户相当简单，并且有详细的官方文档和多个独立网站的支持，因此本书不会涉及这部分内容。

在创建了 AWS 账户后，您需要进入 AWS 并执行以下操作：

+   上传您的 SSH 密钥至 **EC2** | **密钥对**

+   在 **身份与访问管理** | **用户** | **创建新用户** 中创建一个新用户，并在 `~/.aws/credentials` 文件中加入以下内容：

```
    [default] 
    aws_access_key_id = YOUR_ACCESS_KEY 
    aws_secret_access_key = YOUR_SECRET_KEY 

```

在您创建了 AWS 密钥并上传了 SSH 密钥后，您需要设置 Route53。在 Route53 中，您需要为您的域创建两个区域（如果没有未使用的域名，也可以使用子域）：一个 **公共** 区域和一个 **私有** 区域。

如果您仅创建公共区域，Route53 会在所有地方传播此区域；但如果创建了公共和私有区域，Route53 会在所有地方提供公共区域，但在创建私有区域时指定的 VPC 中，私有区域将被使用。如果您从该 VPC 内查询这些 DNS 记录，私有区域将被使用。这种方式有多个优点：

+   仅公开公共机器的 IP 地址

+   始终使用 DNS 名称而非 IP 地址，即使是内部流量

+   确保您的内部机器直接通信，避免流量通过公共网络

+   由于 Amazon Web Services 中的外部 IP 是由 Amazon 管理的虚拟 IP，并通过 NAT 与您的实例关联，这种方式提供了最少的跳数，因此具有较低的延迟。

### 注意

如果你为公共区域声明了一个条目，但没有在私有区域中声明，VPC 中的机器将无法解析该条目。

在你创建了公共区域之后，Amazon Web Services 会提供几个名称服务器 IP 地址，你需要将这些地址放入你的注册/根区域 DNS 中，以便你可以实际解析这些 DNS。

## 简单的 AWS 部署

正如我们之前所说的，首先我们需要将网络搭建好。对于这个例子，我们只需要在一个可用区内配置一个网络，所有的机器将都留在这个网络中。

在这一部分，我们将会在 `playbooks/aws_simple_provision.yaml` 文件中进行操作。

前两行只是用于声明将执行命令的主机（`localhost`）以及 `tasks` 部分的开始：

```
    - hosts: localhost 
      tasks:

```

在 AWS 中，我们需要有一个 VPC 网络和子网络，但如果你需要的话，你可以按如下方式创建 VPC 网络：

```
    To create the VPC subnetwork: 
      - name: Ensure the VPC subnetwork is present 
        ec2_vpc_subnet: 
          state: present 
          az: AWS_AZ 
          vpc_id: '{{ aws_simple_net.vpc_id }}' 
          cidr: 10.0.1.0/24 
        register: aws_subnet 

```

现在我们已经有了网络和子网的所有信息，我们可以继续处理**安全组**了。我们可以通过 `ec2_group` 模块来实现这一点。在亚马逊 Web 服务中，安全组用于防火墙。安全组类似于具有相同目标（对于入口规则）或相同目标（对于出口规则）的防火墙规则组。值得一提的是，标准防火墙规则与安全组有三个区别：

+   可以将多个安全组应用到同一个 EC2 实例。

+   作为源（对于入口规则）或目的地（对于出口规则），你可以指定以下其中之一：

    +   一个实例 ID

    +   另一个安全组

    +   一个 IP 范围

+   你不需要在链的末尾指定默认的拒绝规则，因为 AWS 默认会添加它。

```
      - name: Ensure websg Security Group is present 
        ec2_group: 
          name: web 
          description: Web Security Group 
          region: AWS_AZ 
          vpc_id: VPC_ID 
          rules: 
          - proto: tcp 
            from_port: 80 
            to_port: 80 
            cidr_ip: 0.0.0.0/0 
          - proto: tcp 
            from_port: 443 
            to_port: 443 
            cidr_ip: 0.0.0.0/0 
          rules_egress: 
          - proto: all 
            cidr_ip: 0.0.0.0/0 
        register: aws_simple_websg 

```

所以，在我的情况下，以下代码将会被添加到 `playbooks/aws_simple_provision.yaml`：

```
      - name: Ensure wssg Security Group is present 
        ec2_group: 
          name: wssg 
          description: Web Security Group 
          region: eu-west-1 
          vpc_id: '{{ aws_simple_net.vpcs.0.id }}' 
          rules: 
          - proto: tcp 
            from_port: 22 
            to_port: 22 
            cidr_ip: 0.0.0.0/0 
          - proto: tcp 
            from_port: 80 
            to_port: 80 
            cidr_ip: 0.0.0.0/0 
          - proto: tcp 
            from_port: 443 
            to_port: 443 
            cidr_ip: 0.0.0.0/0 
          rules_egress: 
          - proto: all 
            cidr_ip: 0.0.0.0/0 
        register: aws_simple_wssg 

```

我们现在将为我们的数据库创建另一个安全组。在这种情况下，我们只需要将端口 `3036` 开放给 Web 安全组中的服务器：

```
      - name: Ensure dbsg Security Group is present 
        ec2_group: 
          name: dbsg 
          description: DB Security Group 
          region: eu-west-1 
          vpc_id: '{{ aws_simple_net.vpcs.0.id }}' 
          rules: 
          - proto: tcp 
            from_port: 3036 
            to_port: 3036 
            group_id: '{{ aws_simple_wssg.group_id }}' 
          rules_egress: 
          - proto: all 
            cidr_ip: 0.0.0.0/0 
        register: aws_simple_dbsg 

```

### 注意

如你所见，我们允许所有出口流量通过。这并不是安全最佳实践所建议的，因此你可能需要对出口流量进行调控。一个常常迫使你调控出口流量的情况是，当你希望目标机器符合 PCI-DSS 标准时。

现在我们已经有了 VPC、子网和所需的安全组，接下来我们可以开始实际创建 EC2 实例了：

```
      - name: Setup instances 
        ec2: 
          assign_public_ip: '{{ item.assign_public_ip }}' 
          image: ami-7abd0209 
          region: eu-west-1 
          exact_count: 1 
          key_name: fale 
          count_tag: 
            Name: '{{ item.name }}' 
          instance_tags: 
            Name: '{{ item.name }}' 
          instance_type: t2.micro 
          group_id: '{{ item.group_id }}' 
          vpc_subnet_id: '{{ aws_simple_subnet.subnets.0.id }}' 
          volumes: 
            - device_name: /dev/sda1 
              volume_type: gp2 
              volume_size: 10 
              delete_on_termination: True 
        register: aws_simple_instances 
        with_items: 
        - name: ws01.simple.aws.fale.io 
          group_id: '{{ aws_simple_wssg.group_id }}' 
          assign_public_ip: True 
        - name: ws02.simple.aws.fale.io 
          group_id: '{{ aws_simple_wssg.group_id }}' 
          assign_public_ip: True 
        - name: db01.simple.aws.fale.io 
          group_id: '{{ aws_simple_dbsg.group_id }}' 
          assign_public_ip: False 

```

### 注意

当我们创建 `db` 机器时并未指定 `assign_public_ip`: `True` 这一行。在这种情况下，机器将不会获得公共 IP，因此它将无法从我们 VPC 外部访问。由于我们为这台服务器使用了非常严格的安全组，它无论如何都无法从 `wssg` 外的任何机器访问。

正如你猜测的那样，我们刚刚看到的这段代码将会创建我们的三个实例（两个 Web 服务器和一个数据库服务器）。

我们现在可以将这些新创建的实例添加到我们的 Route 53 账户中，以便解析这些机器的 FQDN。为了与 AWS Route 53 交互，我们将使用 `route53` 模块，该模块允许我们创建条目、查询条目和删除条目。要创建新的条目，我们将使用以下代码：

```
      - name: Add route53 entry for server SERVER_NAME 
        route53: 
          command: create 
          zone: ZONE_NAME 
          record: RECORD_TO_ADD 
          type: RECORD_TYPE 
          ttl: TIME_TO_LIVE 
          value: IP_VALUES 
          wait: True 

```

因此，要为我们的服务器创建条目，我们将添加以下代码：

```
      - name: Add route53 rules for instances 
        route53: 
          command: create 
          zone: aws.fale.io 
          record: '{{ item.tagged_instances.0.tags.Name }}' 
          type: A 
          ttl: 1 
          value: '{{ item.tagged_instances.0.public_ip }}' 
          wait: True 
        with_items: '{{ aws_simple_instances.results }}' 
        when: item.tagged_instances.0.public_ip 
      - name: Add internal route53 rules for instances 
        route53: 
          command: create 
          zone: aws.fale.io 
          private_zone: True 
          record: '{{ item.tagged_instances.0.tags.Name }}' 
          type: A 
          ttl: 1 
          value: '{{ item.tagged_instances.0.private_ip }}' 
          wait: True 
        with_items: '{{ aws_simple_instances.results }}' 

```

### 注意

由于数据库服务器没有公共地址，将这台机器发布到公共区域没有意义，因此我们只在内部区域创建了这台机器的条目。

综合起来，`playbooks/aws_simple_provision.yaml` 的内容将如下：

```
    - hosts: localhost 
      tasks: 
      - name: Gather information of the EC2 VPC net in eu-west-1 
        ec2_vpc_net_facts: 
          region: eu-west-1 
        register: aws_simple_net 
      - name: Gather information of the EC2 VPC subnet in eu-west-1 
        ec2_vpc_subnet_facts: 
          region: eu-west-1 
          filters: 
            vpc-id: '{{ aws_simple_net.vpcs.0.id }}' 
        register: aws_simple_subnet 
      - name: Ensure wssg Security Group is present 
        ec2_group: 
          name: wssg 
          description: Web Security Group 
          region: eu-west-1 
          vpc_id: '{{ aws_simple_net.vpcs.0.id }}' 
          rules: 
          - proto: tcp 
            from_port: 22 
            to_port: 22 
            cidr_ip: 0.0.0.0/0 
          - proto: tcp 
            from_port: 80 
            to_port: 80 
            cidr_ip: 0.0.0.0/0 
          - proto: tcp 
            from_port: 443 
            to_port: 443 
            cidr_ip: 0.0.0.0/0 
          rules_egress: 
          - proto: all 
            cidr_ip: 0.0.0.0/0 
        register: aws_simple_wssg 
      - name: Ensure dbsg Security Group is present 
        ec2_group: 
          name: dbsg 
          description: DB Security Group 
          region: eu-west-1 
          vpc_id: '{{ aws_simple_net.vpcs.0.id }}' 
          rules: 
          - proto: tcp 
            from_port: 3036 
            to_port: 3036 
            group_id: '{{ aws_simple_wssg.group_id }}' 
          rules_egress: 
          - proto: all 
            cidr_ip: 0.0.0.0/0 
        register: aws_simple_dbsg 
      - name: Setup instances 
        ec2: 
          assign_public_ip: '{{ item.assign_public_ip }}' 
          image: ami-7abd0209 
          region: eu-west-1 
          exact_count: 1 
          key_name: fale 
          count_tag: 
            Name: '{{ item.name }}' 
          instance_tags: 
            Name: '{{ item.name }}' 
          instance_type: t2.micro 
          group_id: '{{ item.group_id }}' 
          vpc_subnet_id: '{{ aws_simple_subnet.subnets.0.id }}' 
          volumes: 
            - device_name: /dev/sda1 
              volume_type: gp2 
              volume_size: 10 
              delete_on_termination: True 
        register: aws_simple_instances 
        with_items: 
        - name: ws01.simple.aws.fale.io 
          group_id: '{{ aws_simple_wssg.group_id }}' 
          assign_public_ip: True 
        - name: ws02.simple.aws.fale.io 
          group_id: '{{ aws_simple_wssg.group_id }}' 
          assign_public_ip: True 
        - name: db01.simple.aws.fale.io 
          group_id: '{{ aws_simple_dbsg.group_id }}' 
          assign_public_ip: False 
      - name: Add route53 rules for instances 
        route53: 
          command: create 
          zone: aws.fale.io 
          record: '{{ item.tagged_instances.0.tags.Name }}' 
          type: A 
          ttl: 1 
          value: '{{ item.tagged_instances.0.public_ip }}' 
          wait: True 
        with_items: '{{ aws_simple_instances.results }}' 
        when: item.tagged_instances.0.public_ip 
      - name: Add internal route53 rules for instances 
        route53: 
          command: create 
          zone: aws.fale.io 
          private_zone: True 
          record: '{{ item.tagged_instances.0.tags.Name }}' 
          type: A 
          ttl: 1 
          value: '{{ item.tagged_instances.0.private_ip }}' 
          wait: True 
        with_items: '{{ aws_simple_instances.results }}' 

```

运行 `ansible-playbook playbooks/aws_simple_provision.yaml` 后，我们将得到类似于以下的输出：

```
PLAY [localhost] ***************************************************
TASK [setup] *******************************************************
ok: [localhost] 
TASK [Gather information of the EC2 VPC net in eu-west-1] **********
ok: [localhost] 
TASK [Gather information of the EC2 VPC subnet in eu-west-1] *******
ok: [localhost] 
TASK [Ensure wssg Security Group is present] ***********************
changed: [localhost] 
TASK [Ensure dbsg Security Group is present] ***********************
changed: [localhost] 
TASK [Setup instances] *********************************************
changed: [localhost] => (item={u'group_id': u'sg-950c2cf2', u'name': u'ws01.simple.aws.fale.io', u'assign_public_ip': True})
changed: [localhost] => (item={u'group_id': u'sg-950c2cf2', u'name': u'ws02.simple.aws.fale.io', u'assign_public_ip': True})
changed: [localhost] => (item={u'group_id': u'sg-940c2cf3', u'name': u'db01.simple.aws.fale.io', u'assign_public_ip': False}) 
TASK [Add route53 rules for instances] *****************************
changed: [localhost] =>
    .... 
changed: [localhost] =>
    .... 
skipping: [localhost] =>
    .... 
TASK [Add internal route53 rules for instances] ******************
changed: [localhost] =>
    .... 
changed: [localhost] =>
    .... 
changed: [localhost] =>
    .... 
PLAY RECAP ****************************************************
localhost                  : ok=7    changed=4    unreachable=0    failed=0

```

## 复杂的 AWS 部署

在这一段中，我们将略微修改前面的示例，将其中一台 Web 服务器移动到同一区域的另一个 AZ（可用区）。为此，我们将在 `playbooks/aws_complex_provision.yaml` 中创建一个新文件，该文件与之前的文件非常相似，唯一的区别是在帮助我们配置机器的部分。实际上，我们将使用以下代码，而不是上次运行时使用的代码：

```
      - name: Setup instances 
        ec2: 
          assign_public_ip: '{{ item.assign_public_ip }}' 
          image: ami-7abd0209 
          region: eu-west-1 
          exact_count: 1 
          key_name: fale 
          count_tag: 
            Name: '{{ item.name }}' 
          instance_tags: 
            Name: '{{ item.name }}' 
          instance_type: t2.micro 
          group_id: '{{ item.group_id }}' 
          vpc_subnet_id: '{{ item.vpc_subnet_id }}' 
          volumes: 
            - device_name: /dev/sda1 
              volume_type: gp2 
              volume_size: 10 
              delete_on_termination: True 
        register: aws_simple_instances 
        with_items: 
        - name: ws01.simple.aws.fale.io 
          group_id: '{{ aws_simple_wssg.group_id }}' 
          assign_public_ip: True 
          vpc_subnet_id: '{{ aws_simple_subnet.subnets.0.id }}' 
        - name: ws02.simple.aws.fale.io 
          group_id: '{{ aws_simple_wssg.group_id }}' 
          assign_public_ip: True 
          vpc_subnet_id: '{{ aws_simple_subnet.subnets.1.id }}' 
        - name: db01.simple.aws.fale.io 
          group_id: '{{ aws_simple_dbsg.group_id }}' 
          assign_public_ip: False 
          vpc_subnet_id: '{{ aws_simple_subnet.subnets.0.id }}' 

```

如您所见，我们已将 `vpc_subnet_id` 放入变量中，以便可以为 `ws02` 机器使用不同的子网。由于 AWS 默认提供两个子网（且每个子网绑定到不同的 AZ），因此使用以下 AZ 就足够了。安全组和 Route 53 代码无需更改，因为它们并不是在子网/AZ 层级上工作，而是在 VPC 层级（对于安全组和内部 Route 53 区域）或全局层级（对于公共 Route 53）上工作。

# DigitalOcean

与 Amazon Web Services 相比，DigitalOcean 看起来非常不完整。直到几个月前，DigitalOcean 只提供了 droplets、SSH 密钥管理和 DNS 管理。撰写本文时，DigitalOcean 最近刚刚推出了一个额外的块存储服务。与许多竞争对手相比，DigitalOcean 的优势包括：

+   价格低于 AWS

+   非常简便的 API

+   有非常完善的 API 文档

+   Droplets 非常类似于标准虚拟机（它们没有做奇怪的定制化）

+   Droplets 启动和停止的速度非常快

+   由于 DigitalOcean 的网络架构非常简单，它比 AWS 更加高效

## Droplets（云主机）

滴水（droplets）是 DigitalOcean 提供的主要服务，它们是计算实例，类似于 Amazon EC2 经典实例。DigitalOcean 依赖**内核虚拟机**（**KVM**）来虚拟化机器，确保非常高的性能和安全性。由于它们不会以任何重要方式改变 KVM，且 KVM 是开源的，且可以在任何 Linux 机器上使用，这使得系统管理员能够在私有和公共云中创建相同的环境。DigitalOcean 的滴水（droplets）将拥有一个外部 IP，并且可以最终被添加到一个虚拟网络中，从而允许您的机器使用内部 IP。

与许多其他类似服务不同，DigitalOcean 允许您的滴水（droplets）除了 IPv4 地址外，还可以拥有 IPv6 地址。此服务是免费的。

## SSH 密钥管理

每次创建滴水（droplet）时，您必须指定是否希望为`root`用户分配特定的 SSH 密钥，或者是否希望使用密码（在首次登录时需要更改）。为了能够选择 SSH 密钥，您需要一个界面来上传它。DigitalOcean 允许您通过一个非常简单的界面来执行此操作，该界面可以列出当前的密钥，并且可以创建和删除密钥。

## 私有网络

如在滴水（droplet）段落中提到的，DigitalOcean 允许我们拥有一个私有网络，在该网络中，我们的机器可以与另一台机器通信。这使得服务（如数据库服务）仅在内部网络上进行隔离，从而提高安全性。由于 MySQL 默认绑定在所有可用接口上，我们需要稍微调整数据库角色，以便只在内部网络上进行绑定。

要区分内部网络与外部网络，可以采用多种方式，这与 DigitalOcean 的一些特性有关：

+   私有网络始终位于`10.0.0.0/8`网络中，而公共 IP 则从不位于该网络中。

+   公共网络始终是`eth0`，而私有网络始终是`eth1`。

根据您的可移植性需求，您可以使用其中一种策略来理解应该将您的服务绑定到哪里。

## 在 DigitalOcean 中添加 SSH 密钥

您需要拥有一个设置了信用卡的 DigitalOcean 用户，并获取 API 密钥。要执行这些操作，您可以使用 DigitalOcean 的 Web 界面。我们现在可以开始使用 Ansible 将我们的 SSH 密钥添加到 DigitalOcean 云中。为此，我们需要创建一个名为`playbooks/do_provision.yaml`的文件，结构如下：

```
    - hosts: localhost 
      tasks: 
      - name: Add the SSH Key to Digital Ocean 
        digital_ocean: 
          state: present 
          command: ssh 
          name: SSH_KEY_NAME 
          ssh_pub_key: 'ssh-rsa AAAA...' 
          api_token: XXX 
        register: ssh_key 

```

在我的例子中，这是我的文件内容：

```
    - hosts: localhost 
      tasks: 
      - name: Add the SSH Key to Digital Ocean 
        digital_ocean: 
          state: present 
          command: ssh 
          name: faleKey 
          ssh_pub_key: 'ssh-rsa AAAA...==' 
          api_token: 259...b3b 
    register: ssh_key 

```

然后我们可以通过以下命令执行它：

```
    ansible-playbook -i localhost, playbooks/do_provision.yaml

```

您将得到类似以下的结果：

```
PLAY [localhost] **************************************************
TASK [setup] ******************************************************
ok: [localhost] 
TASK [Add the SSH Key to Digital Ocean] ***************************
changed: [localhost] 
PLAY RECAP ********************************************************
localhost                  : ok=2    changed=1    unreachable=0    failed=0

```

这个任务是幂等的，因此我们可以多次执行它。如果密钥已经上传，SSH 密钥 ID 将在每次运行时返回。

## 在 DigitalOcean 中的部署

在写作时，创建一个 droplet 的唯一方法是使用 `digital_ocean` 模块，但该模块很快可能会被弃用，因为许多功能现在已通过其他模块以更好的、更清晰的方式完成，并且在 Ansible 的 bug 跟踪系统中已经有一个 bug，用于跟踪其完整重写和可能的弃用。我猜测新模块可能会叫做 `digital_ocean_droplet`，并且会有类似的语法，但目前没有相关代码，所以这只是我的猜测。

为了创建 droplet，我们将必须使用 `digital_ocean` 模块，语法类似于以下内容：

```
      - name: Ensure the ws and db servers are present 
        digital_ocean: 
          state: present 
          ssh_key_ids: KEY_ID 
          name: '{{ item }}' 
          api_token: DIGITAL_OCEAN_KEY 
          size_id: 512mb 
          region_id: lon1 
          image_id: centos-7-0-x64 
          unique_name: True 
        with_items: 
        - WEBSERVER 1 
        - WEBSERVER 2 
        - DBSERVER 1 

```

为了确保我们的配置完全且以合理的方式完成，我总是建议为整个基础设施创建一个单一的配置文件。所以，在我的案例中，我将以下任务添加到 `playbooks/do_provision.yaml` 文件中：

```
      - name: Ensure the ws and db servers are present 
        digital_ocean: 
          state: present 
          ssh_key_ids: '{{ ssh_key.ssh_key.id }}' 
          name: '{{ item }}' 
          api_token: 259...b3b 
          size_id: 512mb 
          region_id: lon1 
          image_id: centos-7-0-x64 
          unique_name: True 
        with_items: 
        - ws01.do.fale.io 
        - ws02.do.fale.io 
        - db01.do.fale.io 
        register: droplets 

```

之后，我们可以使用 `digital_ocean_domain` 模块添加域名：

```
      - name: Ensure domain resolve properly
        digital_ocean_domain:
          api_token: 259...b3b
          state: present
          name: '{{ item.droplet.name }}'
          ip: '{{ item.droplet.ip_address }}'
        with_items: '{{ droplets.results }}'

```

所以，将所有内容整合起来，我们的 `playbooks/do_provision.yaml` 文件将如下所示：

```
    - hosts: localhost 
      tasks: 
      - name: Add the SSH Key to Digital Ocean 
        digital_ocean: 
          state: present 
          command: ssh 
          name: faleKey 
          ssh_pub_key: 'ssh-rsa AAAA...==' 
          api_token: 7e7...f6f 
        register: ssh_key 
      - name: Ensure the ws and db servers are present 
        digital_ocean: 
          state: present 
          ssh_key_ids: '{{ ssh_key.ssh_key.id }}' 
          name: '{{ item }}' 
          api_token: 259...b3b 
          size_id: 512mb 
          region_id: lon1 
          image_id: centos-7-0-x64 
          unique_name: True 
        with_items: 
        - ws01.do.fale.io 
        - ws02.do.fale.io 
        - db01.do.fale.io 
        register: droplets 
      - name: Ensure domain resolve properly 
        digital_ocean_domain: 
          api_token: 259...b3b 
          state: present 
          name: '{{ item.droplet.name }}' 
          ip: '{{ item.droplet.ip_address }}' 
        with_items: '{{ droplets.results }}' 

```

现在我们可以通过以下命令运行它：

```
ansible-playbook -i localhost, playbooks/do_provision.yaml

```

我们将看到类似于以下的结果：

```
PLAY [localhost] **************************************************
TASK [setup] ******************************************************
ok: [localhost] 
TASK [Add the SSH Key to Digital Ocean] ***************************
changed: [localhost] 
TASK [Ensure the ws and db servers are present] *******************
changed: [localhost] => (item=ws01.do.fale.io)
changed: [localhost] => (item=ws02.do.fale.io)
changed: [localhost] => (item=db01.do.fale.io) 
TASK [Ensure domain resolve properly] *****************************
changed: [localhost] =>
    .... 
changed: [localhost] =>
    .... 
changed: [localhost] =>
    .... 
PLAY RECAP ************************************************************
localhost                  : ok=4    changed=3    unreachable=0    failed=0

```

# 总结

在这一章中，我们已经看到如何在 AWS 云和 DigitalOcean 云中配置我们的机器。在 AWS 云的案例中，我们看到了两个不同的例子，一个非常简单，另一个稍微复杂一些。

在下一章中，我们将讨论如何在 Ansible 运行出错时收到通知。
