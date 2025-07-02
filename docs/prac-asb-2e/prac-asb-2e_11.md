

# 第十一章：容器和云管理

Ansible 是一个非常灵活的自动化工具，可以轻松用于自动化基础设施的各个方面。近年来，基于容器的工作负载和云工作负载变得越来越流行，因此，我们将探讨如何使用 Ansible 自动化与这些工作负载相关的任务。本章将从使用 Ansible 设计和构建容器开始。接着，我们将探讨如何运行这些容器，最后，我们将探讨如何使用 Ansible 管理各种云平台。

本章将特别涵盖以下主题：

+   使用 Ansible 自动化 Docker 和 Podman

+   使用 Ansible 管理 Kubernetes

+   探索以容器为中心的模块

+   针对 Amazon Web Services 进行自动化

+   通过自动化增强 Google Cloud Platform

+   与 Azure 的无缝自动化集成

+   使用 Rackspace Cloud 扩展你的环境

+   使用 Ansible 编排 OpenStack

让我们开始吧！

# 技术要求

本章假设你已经按照*第一章*《入门 Ansible》中的详细说明设置好了控制主机，并且使用的是最新版本——本章中的示例在 Ansible 2.15 下进行了测试。尽管我们在本章中会给出具体的主机名示例，你可以自由替换成你自己的主机名和/或 IP 地址。如何操作的详细信息将在相应位置提供。本章还假设你可以访问 Docker、Podman、Kubernetes 以及不同的云平台。由于大多数过程在不同的云平台之间非常相似，因此即使你只在几个你有访问权限的环境中测试这些剧本，你也能获得大部分信息。

本章中的所有示例可以在本书的 GitHub 仓库中找到：[`github.com/PacktPublishing/Practical-Ansible-Second-Edition/tree/main/Chapter%2011`](https://github.com/PacktPublishing/Practical-Ansible-Second-Edition/tree/main/Chapter%2011)。

# 使用 Ansible 自动化 Docker 和 Podman

在今天的世界中，单纯能够运行一个镜像并不被视为生产就绪的设置。

要想将一个部署称为**生产就绪**，你需要证明你的应用程序所提供的服务能够在单个应用崩溃以及硬件故障的情况下，仍然合理运行。通常，客户还会对可靠性提出更多的要求。

幸运的是，你的软件并不是唯一有这些要求的，因此已经为此目的开发了编排解决方案。

今天，最成功的一个是 Kubernetes，因其各种发行版/版本而被广泛使用，因此我们将主要聚焦于它。

Kubernetes 的理念是，您向 Kubernetes 控制平面告知要运行 *X* 个 *Y* 应用程序实例。Kubernetes 会计算在 Kubernetes 节点上运行的 *Y* 应用程序实例数量，以确保实例数量为 *X*。如果实例数量不足，Kubernetes 会自动启动更多实例。同时，如果实例数量过多，超出的实例将被停止。

由于 Kubernetes 不断检查请求的实例数量是否在运行，一旦应用程序失败或节点失败，Kubernetes 会启动新的实例来替代丢失的实例。

由于 Kubernetes 安装和管理的复杂性，许多公司已经开始销售简化操作的 Kubernetes 发行版，并且愿意提供支持。

每个 Kubernetes 节点都运行一个容器引擎实例。因此，我们将开始了解如何自动化两种最广泛使用的独立容器引擎。

## 管理 Docker

Docker 现在是一个非常常见的、无处不在的开发工具，您可以利用 Ansible 来轻松管理您的 Docker 实例。由于我们要管理一个 Docker 实例，我们需要确保手头有一个 Docker 实例，并且机器上的 Docker 命令已正确配置。我们需要做这些来确保能够在终端上运行 `docker images`。假设您得到类似以下的结果：

```
REPOSITORY TAG IMAGE ID CREATED SIZE
```

这意味着一切正常。如果您已经克隆了镜像，可能会有更多的输出行。

另一方面，假设它返回类似这样的内容：

```
Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
```

这意味着我们没有运行 Docker 守护进程，或者我们的 Docker 控制台配置不正确。

同时，确保安装了 `docker` Python 模块也很重要，因为 Ansible 会尝试使用它与 Docker 守护进程通信。让我们来看一下：

1.  首先，我们需要创建一个名为 `start-docker-container.yaml` 的 playbook，包含以下代码：

    ```
    ---
    - hosts: localhost
      tasks:
      - name: Start a container with a command
        community.docker.docker_container:
          name: test-container
          image: alpine
          command:
          - echo
          - "Hello, World!"
    ```

1.  现在我们有了 Ansible playbook，只需要执行它：

    ```
    $ ansible-playbook start-docker-container.yaml
    ```

正如你所预期的，它会给出类似于以下的输出：

```
PLAY [localhost] *********************************************************************
TASK [Gathering Facts] ***************************************************************
ok: [localhost]
TASK [Start a container with a command] **********************************************
changed: [localhost]
PLAY RECAP ***************************************************************************
localhost : ok=2 changed=1 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

1.  现在我们可以检查我们的命令是否正确执行，如下所示：

    ```
    $ docker container list -a
    ```

这将显示已运行的容器：

```
CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES
c706ec55fc0d alpine "echo Hello, World!" 3 minutes ago Exited (0) About a minute ago test-container
```

这证明了一个容器已经被执行。

1.  为了检查 `echo` 命令是否已执行，我们可以运行以下代码：

    ```
    $ docker logs test-container
    ```

这将返回以下输出：

```
community.docker.docker_container module. This is not the only module Ansible has to control the Docker daemon, but it is probably one of the most widely used since it’s used to control containers running on Docker.
Other modules include the following:

*   `community.docker.docker_config`: Used to change the configurations of the Docker daemon
*   `community.docker.docker_container_info`: Used to gather information from (inspect) a container
*   `community.docker.docker_network`: Used to manage Docker networking configuration

There are also many modules in the `community.docker` collection, but they are actually used to manage Docker Swarm clusters and not Docker instances. Some examples are as follows:

*   `community.docker.docker_node`: Used to manage a node in a Docker Swarm cluster
*   `community.docker.docker_node_info`: Used to retrieve information about a specific node in a Docker Swarm cluster
*   `community.docker.docker_swarm_info`: Used to retrieve information about a Docker Swarm cluster

As we will see in the next section, there are many more modules that can be used to manage containers that are orchestrated in various ways.
Now that you have learned how to automate Docker with Ansible, we will see how to work with Podman, a Docker alternative.
Managing Podman
Podman is a container engine competitor to Docker. The reason why Red Hat started to develop an alternative container engine is due to the design of Docker. Although Docker changed some design decisions over the years, at the time, Docker was running as a daemon that required root access to the machine, and the CLI `docker` command was just a client. The risk with this approach, at least from Red Hat’s standpoint, was that a bug in the Docker daemon could have endangered the whole system’s security.
Podman, differently from Docker, does not have a daemon running, and all utilities Podman provides are discrete. This approach allows Podman to be much more secure than Docker, allowing containers to be run without any root access.
One of the critical decisions made at the very beginning of Podman development was the CLI and API compatibility between Podman and Docker. This aspect dramatically increases the simplicity of moving from one to the other without much effort.
As you might have imagined, from an Ansible standpoint, it is also going to be very similar.
Due to the simpler design Podman has, to ensure that it is present and usable on your system, it is enough to run the `podman` command. If it returns the following, it means that you do not have it installed:

```

bash: podman: command not found

```

 If it returns all the command options, this means that Podman is properly installed on your system.
We can therefore create our first Ansible Playbook to manage Podman, which we will call `start-podman-container.yaml`, with the following content:

```

---

- hosts: localhost

tasks:

- name: 使用命令启动容器

containers.podman.podman_container:

name: test-container

image: alpine

command:

- echo

- "Hello, World!"

```

 We can now run the playbook by executing the following:

```

$ ansible-playbook start-podman-container.yaml

```

 As you may expect, it will give you an output similar to this:

```

PLAY [localhost] *******************************************************************************************************************************************************************************

任务 [收集信息] *************************************************************************************************************************************************************************

成功: [localhost]

任务 [启动一个容器并执行命令] ********************************************************************************************************************************************************

已更改: [localhost]

执行总结 *************************************************************************************************************************************************************************************

localhost                  : ok=2    已更改=1    无法访问=0    失败=0     跳过=0    恢复=0    忽略=0

```

 As we did for Docker, we can check that the command executed properly by running the following:

```

$ podman 容器列表 -a

```

 And this command will return the information of the container that ran:

```

容器 ID  镜像                                                 命令               创建时间        状态                    端口        名称 96531f2f960e  docker.io/library/alpine:latest                        echo Hello, World...  2 分钟前  已退出 (0) 2 分钟前              test-container

```

 The fact that it exited with `0` means that it successfully ran, as we were expecting.
Similarly to Docker, there are many other modules usable in the `containers.podman` collection. Differently from Docker, though, there is no such thing as Podman Swarm, and therefore all Docker modules focusing on Swarm will not have a Podman counterpart.
Now that we have explored the two major standalone container engines that you might encounter in development and test environments, we will discuss Kubernetes since it is the most widely used container platform in production environments.
Managing Kubernetes with Ansible
We will assume that you have access to either a Kubernetes or OpenShift cluster for testing. Setting these up is out of the scope of this book, so you might want to look at a distribution such as Minikube or Minishift, both of which are designed to be quick and easy to set up so that you can start learning these technologies rapidly. We also need to have the `kubectl` client or the `oc` client, depending on whether we have deployed Kubernetes or OpenShift, properly configured.
Installing Ansible Kubernetes dependencies
First of all, you need to ensure you are running a fairly updated version of Python 3 (>=3.6) and install the required Python packages (you can install it either via PIP or your OS packaging system):

*   Kubernetes >= 12.0.0
*   PyYAML >= 3.11
*   `jsonpatch`

Since the `kubernetes.core` collection is not generally shipped with Ansible, you’ll need to install it by running this:

```

$ ansible-galaxy 集合 安装 kubernetes.core

```

 We are now ready for our first Kubernetes playbook!
Listing Kubernetes namespaces with Ansible
A Kubernetes cluster has multiple namespaces internally, and you can usually find the ones a cluster has with `kubectl get namespaces`. You can do the same with Ansible by creating a file called `k8s-ns-show.yaml` with the following content:

```

---

- 主机: localhost

任务:

- 名称: 从 K8s 获取信息

kubernetes.core.k8s_info:

api_version: v1

类型: 命名空间

注册: ns

- 名称: 打印信息

ansible.builtin.debug:

变量: ns

```

 We can now execute this as follows:

```

$ ansible-playbook k8s-ns-show.yaml

```

 You will now see information regarding the namespaces in the output.
Notice that in the seventh line of the code (`kind: Namespace`), we are specifying the type of resources we are interested in. You can specify other Kubernetes object types to see them (for example, you can try this with Deployments, Services, and Pods).
Creating a Kubernetes namespace with Ansible
So far, we have learned how to show existing namespaces, but usually, Ansible is used in a declarative way to achieve a desired state. So, let’s create a new playbook called `k8s-ns.yaml` with the following content:

```

---

- 主机: localhost

任务:

- 名称: 确保 myns 命名空间存在

kubernetes.core.k8s:

api_version: v1

类型: 命名空间

名称: myns

状态: 存在

```

 Before running it, we can execute `kubectl get ns` so that we can ensure `myns` is not present. In my case, the output is as follows:

```

$ kubectl 获取 ns

名称 状态 年龄

默认 活跃 69 分钟

kube-node-lease 活跃 69 分钟

kube-public 活跃 69 分钟

kube-system 活跃 69 分钟

```

 We can now run the playbook with the following command:

```

$ ansible-playbook k8s-ns.yaml

```

 The output should resemble the following one:

```

执行 [localhost] *******************************************************************

任务 [收集信息] *************************************************************

成功: [localhost]

任务 [确保 myns 命名空间存在] ********************************************

已更改: [localhost]

执行总结 *************************************************************************

localhost : 成功=2 已更改=1 无法访问=0 失败=0 跳过=0 恢复=0 忽略=0

```

 As you can see, Ansible reports that it changed the namespace state. If I execute `kubectl get ns` again, it is clear that Ansible created the namespace we were expecting:

```

$ kubectl 获取 ns

名称 状态 年龄

默认 活跃 74 分钟

kube-node-lease 活跃 74 分钟

kube-public 活跃 74 分钟

kube-system 活跃 74 分钟

myns 活跃 22 秒

```

 Now, let’s create a service.
Creating a Kubernetes service with Ansible
So far, we have seen how to create namespaces from Ansible, so now, let’s put a service in the namespace we just created. Let’s create a new playbook called `k8s-svc.yaml` with the following content:

```

---

- 主机: localhost

任务:

- 名称: 确保服务 mysvc 存在

kubernetes.core.k8s:

状态: 存在

定义:

apiVersion: v1

类型: 服务

元数据:

名称: mysvc

命名空间: myns

规格:

选择器:

应用: myapp

服务: mysvc

端口:

- 协议: TCP

目标端口: 800

名称: port-80-tcp

端口: 80

```

 Before running it, we can execute `kubectl get svc` to ensure that the namespace has no services. Make sure you’re in the right namespace before running it! In my case, the output is as follows:

```

$ kubectl 获取 svc

在 myns 命名空间中未找到资源。

```

 We can now run it with the following command:

```

$ ansible-playbook k8s-svc.yaml

```

 The output should resemble the following one:

```

执行 [localhost] *******************************************************************

任务 [收集信息] *************************************************************

成功: [localhost]

任务 [确保 myns 命名空间存在] ********************************************

已更改: [localhost]

PLAY RECAP *************************************************************************

localhost : ok=2 changed=1 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0

```

 As you can see, Ansible reports that it changed the service state. If I execute `kubectl get svc` again, it is clear that Ansible created the service we were expecting:

```

$ kubectl get svc

NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGE

mysvc ClusterIP 10.0.0.84 <none> 80/TCP 10s

```

 As you can see, we followed the same procedure that we used in the namespace case, but we specified a different Kubernetes object type and specified the various parameters that are needed for the Service type. You can do the same for all other Kubernetes object types.
Now that you have learned how to deal with Kubernetes clusters, you’ll learn how to automate Docker with Ansible.
Exploring container-focused modules
Often, when organizations grow, they start to use multiple technologies in different parts of the organization. Another thing that usually happens is that after a department has found that a vendor worked well for them, they will be more inclined to try new technologies offered by that vendor. A mix of those two factors and time (usually, fewer technology cycles) will end up creating multiple solutions for the same problem within the same organization.
If your organization is in this situation with containers, Ansible can come to the rescue, thanks to its ability to interoperate with the majority of, if not all, container platforms.
Often, the biggest obstacle to doing something with Ansible is finding the name of the modules you need to use to achieve what you want to achieve. In this section, we will try to help in this effort, mainly in terms of the containerization space, but this might help you in the quest to find different kinds of modules.
The starting point for all Ansible module research should be the official Ansible documentation and eventually Ansible Galaxy.
Many Ansible modules relative to container services are in the cloud collections category (ECS, Docker, LXC, or LXD), containers (at the moment only Podman), or in their own one, such as Kubernetes.
To help you further, let’s take a look at some of the main container platforms and the main modules Ansible provides.
**Amazon Web Services** (**AWS**), back in 2014, launched **Elastic Container Service** (**ECS**), which is a way to deploy and orchestrate Docker containers within their infrastructure. In the following year, AWS also launched **Elastic Container Registry** (**ECR**), a managed Docker Registry service. The service did not become as ubiquitous as AWS hoped, so in 2018, AWS launched **Elastic Kubernetes Service** (**EKS**) to allow people that wanted to run Kubernetes on AWS to have a managed service.
If you are using or plan to use EKS, this is just a standard managed Kubernetes cluster, so you can use the Kubernetes-specific modules that we are going to cover shortly. If you decide to use ECS, there are several modules that could help you. The most important ones are `community.aws.ecs_cluster`, which allows you to create or terminate ECS clusters; `community.aws.ecs_ecr`, which allows you to manage ECR; `community.aws.ecs_service`, which allows you to create, terminate, start, or stop a service in ECS; and `community.aws.ecs_task`, which allows you to run, start, or stop a task in ECS. In addition to those, there is `community.aws.ecs_service_info`, which allows Ansible to list or describe services in ECS.
Microsoft Azure, in 2018, announced `azure.azcollection.azure_rm_aks` module allows us to create, update, and delete AKS instances.
Google Cloud launched **Google Kubernetes Engine** (**GKE**) in 2015\. GKE is the **Google Cloud Platform**(**GCP**) version of managed Kubernetes, and is therefore compatible with Ansible Kubernetes modules. In addition to those, there are various GKE-specific modules, some of which are as follows:

*   `google.cloud.gcp_container_cluster`: Allows you to create a GCP cluster
*   `google.cloud.gcp_container_cluster_info`: Allows you to gather facts for a GCP cluster
*   `google.cloud.gcp_container_node_pool`: Allows you to create a GCP node pool
*   `google.cloud.gcp_container_node_pool_info`: Allows you to gather facts for a GCP node pool

Red Hat started OpenShift in 2011, and at the time, it was based on its own container runtime. Version 3, which was released in 2015, was completely based on Kubernetes, so all Ansible Kubernetes modules work. In addition to those, there is the `oc` module, which is currently still present but in a deprecated state, giving preference to the Kubernetes modules.
In 2015, Google released Kubernetes and, quickly, a huge community started to build around it. Ansible allows you to manage your Kubernetes clusters with some modules:

*   `kubernetes.core.k8s`: Allows you to manage any kind of Kubernetes object
*   `kubernetes.core.k8s_auth`: Allows you to authenticate to Kubernetes clusters that require an explicit login step
*   `kubernetes.core.k8s_facts`: Allows you to inspect Kubernetes objects
*   `kubernetes.core.k8s_scale`: Allows you to set a new size for a Deployment, ReplicaSet, Replication Controller, or Job
*   `kubernetes.core.k8s_service`: Allows you to manage Services on Kubernetes

LXC and LXD are also systems that can be used to run containers in Linux. These systems are also supported by Ansible, thanks to the following modules:

*   `community.general.lxc_container`: Allows you to manage LXC containers
*   `community.general.lxd_container`: Allows you to manage LXD containers
*   `community.general.lxd_profile`: Allows you to manage LXD profiles

Now that you have learned how to explore container-focused modules, you’ll learn how to automate with AWS.
Automating with Amazon Web Services
In many organizations, cloud providers are used widely, while in others, they are just being introduced. However, in one way or another, you will probably have to deal with a cloud provider while doing your job. AWS is the biggest and oldest, and is perhaps something you will have to work with.
Installation
To be able to use Ansible to automate your AWS estate, you’ll need to install the `boto3` library. To do so, run the following command:

```

$ pip install boto3

```

 As for collections, at the moment, there are two collections to interact with AWS services: `community.aws` and `amazon.aws`. In many cases, you will need both of them, since many features of the former have not yet been implemented in the latter:

```

$ ansible-galaxy collection install community.aws amazon.aws

```

 Now that you have all the necessary software installed, you can set up authentication.
Authentication
The `boto` library looks up the necessary credentials in the `~/.aws/credentials` file. There are two different ways to ensure that the credentials file is configured properly.
It is possible to use the AWS CLI tool. Alternatively, this can be done with a text editor of your choice by creating a file with the following structure:

```

[default]

aws_access_key_id = [YOUR_KEY_HERE]

aws_secret_access_key = [YOUR_SECRET_ACCESS_KEY_HERE]

```

 Now that you’ve created the file with the necessary credentials, `boto` will be able to work against your AWS environment. Since Ansible uses `boto` for every single communication with AWS systems, this means that Ansible will be appropriately configured, even without you having to change any Ansible-specific configuration.
Creating your first machine
Now that Ansible is able to connect to your AWS environment, you can proceed with the actual playbook by following these steps:

1.  Create the `aws.yaml` Playbook with the following content, ensuring you use your public key in the `key_material` value in the first task:

    ```

    ---

    - hosts: localhost

    tasks:

    - name: 确保密钥对存在

    amazon.aws.ec2_key:

    name: fale

    key_material: "{{ lookup('file', '~/.ssh/fale.pub') }}"

    - name: 收集 eu-west-1 区域 EC2 VPC 网络的信息

    amazon.aws.ec2_vpc_net_info:

    region: eu-west-1

    register: aws_simple_net

    - name: 收集 eu-west-1 区域 EC2 VPC 子网的信息

    amazon.aws.ec2_vpc_subnet_info:

    region: eu-west-1

    filters:

    vpc-id: '{{ aws_simple_net.vpcs.0.id }}'

    register: aws_simple_subnet

    - name: 确保 wssg 安全组存在

    amazon.aws.ec2_security_group:

    name: wssg

    description: Web 安全组

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

    - name: 设置实例

    amazon.aws.ec2_instance:

    assign_public_ip: true

    image: ami-3548444c

    region: eu-west-1

    exact_count: 1

    key_name: fale

    count_tag:

    Name: ws01.ansible2cookbook.com

    instance_tags:

    Name: ws01.ansible2cookbook.com

    instance_type: t2.micro

    group_id: '{{ aws_simple_wssg.group_id }}'

    vpc_subnet_id: '{{ aws_simple_subnet.subnets.0.id }}'

    volumes:

    - device_name: /dev/sda1

    volume_type: gp2

    volume_size: 10

    delete_on_termination: True

    ```

     2.  Run it using the following command:

    ```

    $ ansible-playbook aws.yaml

    ```

This command will return something like the following:

```

PLAY [localhost] **********************************************************************************

TASK [Gathering Facts] ****************************************************************************

ok: [localhost]

TASK [Ensure key pair is present] *****************************************************************

ok: [localhost]

TASK [Gather information of the EC2 VPC net in eu-west-1] *****************************************

ok: [localhost]

TASK [Gather information of the EC2 VPC subnet in eu-west-1] **************************************

ok: [localhost]

TASK [Ensure wssg Security Group is present] ******************************************************

ok: [localhost]

TASK [Setup instance] *****************************************************************************

changed: [localhost]

PLAY RECAP ****************************************************************************************

localhost : ok=6 changed=1 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0

```

 If you check the AWS console, you will see that you now have one machine up and running!
To launch a virtual machine in AWS, we need a few things to be in place:

*   An SSH key pair
*   A network
*   A subnetwork
*   A security group

By default, a network and a subnetwork are already available in your account, but you need to retrieve their IDs.
That’s why we started by uploading the public part of an SSH key pair to AWS, queried for information about the network and the subnetwork, then ensured that the security group we wanted to use was present, and lastly triggered the machine build.
Now you have learned how to automate against AWS, you’ll learn how to complement GCP with automation.
Complementing Google Cloud Platform with automation
Another global cloud provider is Google, with its GCP. Google’s approach to the cloud is different from other providers’ since Google does not try to simulate the data center in a virtual environment. This is because Google wants to rethink the concept of cloud provision in order to simplify it.
Installation
You need to ensure that you have the proper components installed before you can start using GCP with Ansible. More specifically, you will require the Python `requests` and `google-auth` modules. To install these modules, run the following command:

```

$ pip install requests google-auth

```

 We can now proceed to install the Google Cloud collection:

```

$ ansible-galaxy collection install google.cloud

```

 Now that you have all the dependencies present, you can start the authentication process.
Authentication
There are three different approaches to obtaining a working set of credentials in GCP:

*   The service account using environment variables
*   The service account using a JSON file
*   The machine account

The first two approaches are the suggested ones in the majority of cases since the third applies only to circumstances where Ansible is run directly within the GCP environment.
The first approach is, once you have created the service account, set the following environmental variables:

*   `GCP_AUTH_KIND`
*   `GCP_SERVICE_ACCOUNT_EMAIL`
*   `GCP_SERVICE_ACCOUNT_FILE`
*   `GCP_SCOPES`

Now, Ansible can use the proper service account.
If you prefer not to use environment variables, you can download a service account file from the GCP interface directly. For convenience’s sake, we are going to do this and we are going to save the file as `~/sa.json`.
The third approach is by far the easiest since Ansible will be able to auto-detect the machine account if you are running it in a GCP instance.
Creating your first machine
Now that Ansible is able to connect to your GCP environment, you can proceed with the actual Playbook:

1.  Create the `gce.yaml` Playbook with the following content:

    ```

    ---

    - hosts: localhost

    tasks:

    - name: 创建实例

    google.cloud.gcp_compute_instance:

    名称: TestMachine

    机器类型: n1-standard-1

    磁盘:

    - 自动删除: 'true'

    启动: 'true'

    初始化参数:

    源镜像: family/centos-stream-9

    磁盘大小（GB）：10

    区域: eu-west1-c

    认证类型: serviceaccount

    服务账户文件: "~/sa.json"

    状态: 存在

    ```

     2.  Execute it with the following command:

    ```

    $ ansible-playbook gce.yaml

    ```

This will create an output like the following one:

```

执行 [localhost] **********************************************************************************

任务 [收集事实] ****************************************************************************

ok: [localhost]

任务 [创建实例] **************************************************************************

更改: [localhost]

执行总结 ****************************************************************************************

localhost : ok=2 changed=1 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0

```

 As for the AWS example, running a machine in the cloud is very easy with Ansible.
In the case of GCE, you don’t need to set up the networks beforehand since the GCE defaults will kick in and provide a functional machine either way.
As for AWS, the list of modules you can use is huge. You can find the full list at [`docs.ansible.com/ansible/latest/collections/google/cloud/index.xhtml`](https://docs.ansible.com/ansible/latest/collections/google/cloud/index.xhtml).
Now that you have learned how to complement GCP with automation, you will learn how to seamlessly perform automation integration with Azure.
Seamless automation integration with Azure
Another global cloud that Ansible can manage is Microsoft Azure.
Azure integration, like AWS integration, requires quite a few steps to be performed in Playbooks.
The first thing you will need to do is set up the authentication so that Ansible is allowed to control your Azure account.
Installation
To let Ansible manage the Azure cloud, you need to install the Azure SDK for Python. Do this by executing the following command:

```

$ pip install -r ~/.ansible/collections/ansible_collections/azure/azcollection/requirements-azure.txt

```

 We can now proceed to install the Azure collection:

```

$ ansible-galaxy collection install azure.azcollection

```

 Now that you have all the dependencies present, you can start the authentication process.
Authentication
There are different ways to ensure that Ansible is able to manage Azure for you, based on the way your Azure account is set up, but they can all be configured in the `~/.``azure/credentials` file.
If you want Ansible to use the principal credentials for the Azure account, you will need to create a file that resembles the following:

```

[default]

订阅 ID = [YOUR_SUBSCIRPTION_ID_HERE]

客户端 ID = [YOUR_CLIENT_ID_HERE]

密钥 = [YOUR_SECRET_HERE]

租户 = [YOUR_TENANT_HERE]

```

 If you prefer to use Active Directory with a username and password, you can do something like this:

```

[default]

ad_user = [YOUR_AD_USER_HERE]

密码 = [YOUR_AD_PASSWORD_HERE]

```

 Lastly, you can opt for an Active Directory login with ADFS. In this case, you’ll need to set some additional parameters. You’ll end up with something like this:

```

[default]

ad_user = [YOUR_AD_USER_HERE]

密码 = [YOUR_AD_PASSWORD_HERE]

客户端 ID = [YOUR_CLIENT_ID_HERE]

租户 = [YOUR_TENANT_HERE]

adfs_authority_url = [YOUR_ADFS_AUTHORITY_URL_HERE]

```

 The same parameters can be passed as parameters or as environmental variables if it makes more sense.
Creating your first machine
Now that Ansible is able to connect to your Azure environment, you can proceed with the actual Playbook:

1.  Create the `azure.yaml` Playbook with the following content:

    ```

    ---

    - 主机: localhost

    任务:

    - 名称: 确保存储帐户存在

    azure.azcollection.azure_rm_storageaccount:

    资源组: 测试

    名称: mysa

    账户类型: Standard_LRS

    - 名称: 确保虚拟网络存在

    azure.azcollection.azure_rm_virtualnetwork:

    资源组: 测试

    名称: myvn

    地址前缀: "10.10.0.0/16"

    - 名称: 确保子网存在

    azure.azcollection.azure_rm_subnet:

    资源组: 测试

    名称: mysn

    地址前缀: "10.10.0.0/24"

    虚拟网络: myvn

    - 名称: 确保设置了公共 IP

    azure.azcollection.azure_rm_publicipaddress:

    资源组: 测试

    分配方法: 静态

    名称: myip

    - 名称: 确保存在允许 SSH 的安全组

    azure.azcollection.azure_rm_securitygroup:

    资源组: 测试

    名称: mysg

    规则:

    - 名称: SSH

    协议: Tcp

    目标端口范围: 22

    访问: 允许

    优先级: 101

    方向: 入站

    - 名称: 确保 NIC 存在

    azure.azcollection.azure_rm_networkinterface:

    资源组: 测试

    名称: mynic

    虚拟网络: myvn

    子网: mysn

    公共 IP 名称: myip

    安全组: mysg

    - 名称: 确保虚拟机存在

    azure.azcollection.azure_rm_virtualmachine:

    资源组: 测试

    名称: myvm01

    虚拟机大小: Standard_D1

    存储账户: mysa

    存储容器: myvm01

    存储 Blob：myvm01.vhd

    管理员用户名: admin

    管理员密码: Password!

    网络接口: mynic

    镜像:

    提供商: CentOS

    发布者: OpenLogic

    SKU: '8.4'

    版本: 最新

    ```

     2.  We can run it with the following command:

    ```

    $ ansible-playbook azure.yaml

    ```

This will return something like the following:

```

执行 [localhost] **********************************************************************************

TASK [收集事实] ****************************************************************************

ok: [localhost]

TASK [确保存储账户存在] ******************************************************

changed: [localhost]

TASK [确保虚拟网络存在] ******************************************************

changed: [localhost]

TASK [确保子网存在] ***************************************************************

changed: [localhost]

TASK [确保公共 IP 已设置] ***********************************************************

changed: [localhost]

TASK [确保存在允许 SSH 的安全组] ********************************************

changed: [localhost]

TASK [确保 NIC 存在] ******************************************************************

changed: [localhost]

TASK [确保虚拟机存在] ******************************************************

changed: [localhost]

PLAY RECAP ****************************************************************************************

localhost : ok=8 changed=7 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0

```

 You now have your machine running in the Azure cloud!
As you can see, in Azure, you will need all the resources to be ready before you can issue the machine creation command. This is the reason you create the storage account, the virtual network, the subnet, the public IP, the security group, and the NIC first, and only at that point, the machine itself.
Outside the three major players in the market, there are many additional cloud options. Many of which, including many private clouds, leverage OpenStack.
Using Ansible to orchestrate OpenStack
As opposed to the various cloud services we just discussed, all of which are public clouds, OpenStack allows you to create your own (private) cloud.
Private clouds have the disadvantage that they expose more complexity to the administrator and the user, but this is the reason they can be customized to suit an organization perfectly.
Installation
The first step to being able to control an OpenStack cluster with Ansible is to ensure that `openstacksdk` is installed.
To install `openstacksdk`, you need to execute the following command:

```

$ pip install openstacksdk

```

 We can now proceed to install the OpenStack collection:

```

$ ansible-galaxy collection install openstack.cloud

```

 Now that you have installed `openstacksdk`, you can start the authentication process.
Authentication
Since Ansible will use `openstacksdk` as its backend, you will need to ensure that `openstacksdk` is able to connect to the OpenStack cluster.
To do this, you can change the `~/.config/openstack/clouds.yaml` file, ensuring that there is a configuration for the cloud you want to use it for.
An example of what a correct OpenStack credentials set could look like is as follows:

```

clouds:

test_cloud:

region_name: MyRegion

auth:

auth_url: http://[YOUR_AUTH_URL_HERE]:5000/v2.0/

username: [YOUR_USERNAME_HERE]

password: [YOUR_PASSWORD_HERE]

project_name: myProject

```

 It’s also possible to set a different config file location if you are willing to export the `OS_CLIENT_CONFIG_FILE` variable as an environment variable.
Now that you have set up the required security so that Ansible can manage your cluster, you can create your first Playbook.
Creating your first machine
Since OpenStack is very flexible, many of its components can have many different implementations, which means they may differ slightly in terms of their behavior. To be able to accommodate all the various cases, the Ansible modules that manage OpenStack tend to have a lower level of abstraction compared to the ones for many public clouds.
So, to create a machine, you will need to ensure that the public SSH key is known to OpenStack and ensure that the OS image is present as well. After doing this, you can set up networks, subnetworks, and routers to ensure that the machine we are going to create can communicate via the network. Then, you can create the security group and its rules so that the machine can receive connections (pings and SSH traffic, in this case). Finally, you can create a machine instance.
To complete all the steps just described, you need to create a file called `openstack.yaml` with the following content:

```

---

- hosts: localhost

tasks:

- name: 确保 SSH 密钥在 OpenStack 中存在

openstack.cloud.keypair:

state: present

name: ansible_key

public_key_file: "{{ '~' | expanduser }}/.ssh/id_rsa.pub"

- name: 确保我们有 CentOS 镜像

ansible.builtin.get_url:

url: http://cloud.centos.org/centos/9-stream/x86_64/images/CentOS-Stream-GenericCloud-9-20230424.0.x86_64.qcow2

dest: /tmp/CentOS-Stream-GenericCloud-9-20230424.0.x86_64.qcow2

- name: 确保 CentOS 镜像在 OpenStack 中

openstack.cloud.image:

name: centos

container_format: bare

disk_format: qcow2

state: present

filename: /tmp/CentOS-Stream-GenericCloud-9-20230424.0.x86_64.qcow2

- name: 确保网络存在

openstack.cloud.network:

state: present

name: mynet

external: False

shared: False

register: net_out

- name: 确保子网络存在

openstack.cloud.subnet:

state: present

network_name: "{{ net_out.id }}"

name: mysubnet

ip_version: 4

cidr: 192.168.0.0/24

gateway_ip: 192.168.0.1

enable_dhcp: yes

dns_nameservers:

- 8.8.8.8

- name: 确保路由器存在

openstack.cloud.router:

state: present

name: myrouter

network: nova

external_fixed_ips:

- subnet: nova

interfaces:

- mysubnet

- name: 确保安全组存在

openstack.cloud.security_group:

state: present

name: mysg

- name: 确保安全组允许 ICMP 流量

openstack.cloud.security_group_rule:

security_group: mysg

protocol: icmp

remote_ip_prefix: 0.0.0.0/0

- name: 确保安全组允许 SSH 流量

openstack.cloud.security_group_rule:

security_group: mysg

protocol: tcp

port_range_min: 22

port_range_max: 22

remote_ip_prefix: 0.0.0.0/0

- name: 确保实例存在

openstack.cloud.server:

state: present

name: myInstance

image: centos

flavor: m1.small

security_groups: mysg

key_name: ansible_key

nics:

- net-id: "{{ net_out.id }}"

```

 Now, you can run it as follows:

```

$ ansible-playbook openstack.yaml

```

 The output should be as follows:

```

PLAY [localhost] **********************************************************************************

TASK [收集事实] ****************************************************************************

ok: [localhost]

TASK [确保 SSH 密钥在 OpenStack 上存在] *************************************************

changed: [localhost]

TASK [确保我们有一个 CentOS 镜像] **************************************************************

changed: [localhost]

TASK [确保 CentOS 镜像在 OpenStack 中] ****************************************************

changed: [localhost]

TASK [确保网络存在] **************************************************************

changed: [localhost]

TASK [确保子网络存在] ***********************************************************

changed: [localhost]

TASK [确保路由器存在] ***************************************************************

changed: [localhost]

TASK [确保安全组存在] *******************************************************

changed: [localhost]

TASK [确保安全组允许 ICMP 流量] **********************************************

changed: [localhost]

TASK [确保安全组允许 SSH 流量] ***********************************************

changed: [localhost]

TASK [确保实例存在] *****************************************************************

changed: [localhost]

PLAY RECAP ****************************************************************************************

localhost : ok=11 changed=10 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0

```

 As you can see, this process was longer than the public cloud ones we covered. However, you did get to upload the image that you wanted to run, which is something many clouds do not allow (or allow with very complex processes).
Summary
In this chapter, you learned how to automate tasks, such as managing Docker and Podman containers and managing Kubernetes objects. You also explored the modules that can help you automate cloud environments, such as AWS, GCP, Azure, and OpenStack. You also learned about the different approaches various cloud providers use, including their defaults and the parameters that you will always need to add.
Now that you have an understanding of how Ansible interacts with the cloud, you can immediately start to automate your cloud workflows. Also, remember to check the documentation in the *Further reading* section to take a look at all the cloud modules that Ansible supports and their options.
In the next chapter, you will learn how to troubleshoot and create testing strategies.
Questions

1.  Which of the following is NOT a GKE Ansible module?
    1.  `google.cloud.gcp_container_cluster`
    2.  `google.cloud.gcp_container_node_pool`
    3.  `google.cloud.gcp_container_node_pool_info`
    4.  `google.cloud.gcp_container_node_pool_count`
    5.  `google.cloud.gcp_container_cluster_info`
2.  True or false: In order to manage containers in Kubernetes, you need to add `k8s_namespace` in the settings section.
    1.  True
    2.  False
3.  True or false: When working with Azure, instances require a **Network Interface Controller** (**NIC**) to be able to connect to the network.
    1.  True
    2.  False
4.  True or false: When working with AWS, it’s necessary to create a Security Group before creating an EC2 instance.
    1.  True
    2.  False

Further reading

*   More Docker containers: [`docs.ansible.com/ansible/latest/collections/community/docker/index.xhtml`](https://docs.ansible.com/ansible/latest/collections/community/docker/index.xhtml%0D)
*   More Podman modules: [`docs.ansible.com/ansible/latest/collections/containers/podman/index.xhtml`](https://docs.ansible.com/ansible/latest/collections/containers/podman/index.xhtml%0D)
*   More Kubernetes modules: [`docs.ansible.com/ansible/latest/collections/kubernetes/core/index.xhtml`](https://docs.ansible.com/ansible/latest/collections/kubernetes/core/index.xhtml%0D)
*   More AWS modules:
    *   [`docs.ansible.com/ansible/latest/collections/amazon/aws/index.xhtml`](https://docs.ansible.com/ansible/latest/collections/amazon/aws/index.xhtml)
    *   [`docs.ansible.com/ansible/latest/collections/community/aws/index.xhtml`](https://docs.ansible.com/ansible/latest/collections/community/aws/index.xhtml)
*   More GCP modules: [`docs.ansible.com/ansible/latest/collections/google/cloud/index.xhtml`](https://docs.ansible.com/ansible/latest/collections/google/cloud/index.xhtml%0D)
*   More Azure modules: [`docs.ansible.com/ansible/latest/collections/azure/azcollection/index.xhtml`](https://docs.ansible.com/ansible/latest/collections/azure/azcollection/index.xhtml%0D)
*   More OpenStack modules: [`docs.ansible.com/ansible/latest/collections/openstack/cloud/index.xhtml`](https://docs.ansible.com/ansible/latest/collections/openstack/cloud/index.xhtml)

```
