

# 第十二章：管理大型仓库

由于其分布式特性，Git 在每个仓库副本中都包括完整的变更历史。每次克隆不仅获取所有文件，还会获取每个文件的所有修订版本。这使得开发变得高效（不涉及网络的本地操作通常足够快速，不会成为瓶颈）并且与他人协作也很高效（其分布式特性支持多种协作工作流）。

但当你想要处理的仓库非常庞大时会发生什么？我们能否避免占用大量磁盘空间进行版本控制存储？是否可以在克隆仓库时减少最终用户需要检索的数据量？我们是否需要所有文件都存在才能在项目中工作？

如果你考虑一下，你会发现仓库变得庞大的原因大致有三种：它们可能会积累非常长的历史（每个修订方向），它们可能包含需要与代码一起管理的大型二进制资产，项目可能包含大量文件（每个文件方向），或者这些原因的任何组合。对于这些情况，技术和解决方法是不同的，并且可以独立应用，尽管现代 Git 也包含了一站式解决方案。

子模块（在前一章中介绍的，*管理子项目*）有时用于管理大型资产。本章将介绍如何在处理大型二进制文件和其他大型资产时，使用子模块以及其他替代解决方案。

在本章中，我们将涵盖以下主题：

+   Git 和大型文件

+   使用浅克隆处理具有非常长历史的仓库

+   将大型二进制文件存储在子模块中或仓库外部

+   使用稀疏检出减少工作目录的大小

+   如何通过稀疏克隆缩小本地仓库的大小

+   不同变种的稀疏克隆中，哪些操作需要网络访问

+   使用文件系统监视器加速操作

# Scalar – 适用于所有人的大规模 Git

配置 Git 以便更好地处理大型仓库的最简单方法，除了启用相关的 Git 功能外，就是使用内置的`scalar`工具。这个可执行文件自 2022 年发布的 Git 2.38 版本以来一直存在，在此之前它是一个独立的项目，后来成为了 Microsoft 的 Git 分支的一部分。

使用它非常简单：你只需要用`scalar clone`代替`git clone`。如果仓库已经被克隆，你可以运行`scalar register`来实现相同的效果。这个命令所做的一项工作是调度后台维护；你可以通过使用`scalar unregister`命令停止这个过程，并将仓库从已注册的仓库列表中移除。`scalar delete`命令会取消注册仓库并将其从文件系统中删除。

在`scalar`升级后（可能是由于迁移到更新的 Git 版本），你可以运行`scalar reconfigure --all`来升级所有使用 Scalar 注册的仓库。

通过使用 Scalar 注册仓库（或项目的顶级目录，在 Scalar 文档中称为**登记**），你可以开启**部分克隆**和**稀疏检出**，配置 Git 使用**文件系统监控**，并开启**后台维护**任务，如**仓库预取**。

所有这些功能将在接下来的章节中描述，还会介绍一些更具体针对用户需求的大型 Git 仓库处理功能。

# 处理具有非常长历史的仓库

尽管 Git 可以有效处理具有长历史的仓库，但非常古老的项目，跨越大量修订版本，可能会让克隆变得非常麻烦。在许多情况下，你对远古历史不感兴趣，也不想花费时间获取项目的所有修订版本以及存储它们的磁盘空间。在本节中，我们将讨论你可以用来克隆截断历史的技术，或者如何让 Git 在长历史的情况下仍然快速。

例如，如果你想提出一个新功能或修复一个 bug，你可能不想等到完整克隆完成，因为这可能需要一段时间。

在线编辑项目文件

一些 Git 仓库托管服务，比如 GitHub，提供一个基于网页的界面来管理仓库，包括浏览器内的文件管理和编辑。它们甚至可能会自动创建仓库的一个分叉，以便你可以编写并提出更改。

但基于网页的界面并不能涵盖所有内容，你可能在使用自托管仓库或不提供此功能的服务。

然而，修复这个 bug 可能需要在你的机器上运行`git bisect`，在这里回归 bug 容易复现（请参见*第四章*，*探索项目历史*，了解如何使用二分法）。如果你时间和空间都紧张，可能想尝试执行浅克隆（在下面的小节中描述）或稀疏克隆（本章稍后描述）。

## 使用浅克隆来获取截断的历史

快速克隆并节省磁盘空间的简单解决方案是使用 Git 执行**浅克隆**。此操作允许你获取一个本地仓库副本，并将历史记录截断到特定的**深度**——即最新的修订版本数量。

怎么做呢？只需使用`--depth`选项：

```
$ git clone --depth=1 https://git.company.com/project
```

前面的命令只克隆主分支的最新修订。这个技巧可以节省大量时间，并减轻服务器的负担。通常，浅克隆几秒钟就完成，而不是几分钟，这是一项显著的改进。如果你只对查看项目文件感兴趣，而不是整个历史（例如，查看 Git hooks 或 GitHub Actions 中的内容），这种方式非常有用——即，在构建结束后立即删除克隆的情况。

从 Git 1.9 版本开始，Git 即使在浅克隆的情况下，也支持拉取和推送操作，尽管仍需小心。你可以通过向`git fetch`命令提供`--depth=<n>`选项来更改浅克隆的深度（但请注意，深度增加的提交的标签不会被获取）。要将浅克隆仓库转换为完整仓库，请使用`--unshallow`。

重要说明

由于浅克隆中的提交历史被截断，像**git merge-base**和**git log**这样的命令所显示的结果，与完全克隆的情况不同。如果你尝试访问克隆的深度之外的内容，就会发生这种情况。此外，由于 Git 服务器的优化方式，浅克隆仓库中的增量获取可能比完全克隆仓库中的获取花费更长时间。获取操作还可能意外地让仓库变得不再是浅克隆。

请注意，`git clone --depth=1`仍然可能会获取所有分支和标签。如果远程仓库没有`HEAD`，即没有选择主分支，就可能发生这种情况；否则，只有指定的单个分支的最新提交会被获取。长期存在的项目通常在其漫长历史中有许多版本发布。为了节省时间，你需要将浅克隆与下一个解决方案结合使用：分支限制。

使用现代 Git 时，使用**部分克隆**功能可能更有意义。

## 只克隆单个分支

默认情况下，Git 会克隆所有分支和标签（如果你想获取备注或替换，需显式指定）。你可以通过指定只想**克隆单个分支**来限制克隆的历史记录数量：

```
$ git clone --branch master --single-branch \
  https://git.company.com/project
```

由于大多数项目历史（大部分修订的 DAG）在各分支之间是共享的，只有极少数例外，你可能不会发现使用此技巧时有太大的区别。

如果你不希望有分离的孤立分支，或者相反，只希望拥有一个孤立分支（例如，项目的网页分支，或者用于 GitHub Pages 的分支），这个功能可能会非常有用。当与非常浅的克隆（历史记录非常短，以至于大多数分支没有足够的时间进行合并）结合使用时，单分支克隆在节省磁盘空间方面表现良好。

## 在具有长期历史记录的仓库中加速操作

Git 在具有很长历史的仓库中加速的一个特点是 **commit-graph** 文件。使用此功能（从 Git 2.24 版本起默认启用），可以配置 Git 定期写入或更新一个辅助文件，该文件包含一个序列化的（且易于访问的）修订图。这样，查询项目历史的 Git 操作就会变得更快。

你可以通过将 `core.commitGraph` 配置变量设置为 `false` 来关闭此功能。如果需要刷新辅助文件，可以使用 `git commit-graph` `write` 命令来实现。

避免做繁重的工作

一个可能在历史较长时变慢的意外地方是运行 **git status**。这是因为该命令会计算当前分支的详细 ahead/behind 计数（即本地分支领先上游远程分支多少个提交，远程仓库落后多少个提交）。

你可以通过 **--no-ahead-behind** 选项或将 **status.aheadBehind** 配置变量设置为 **false** 来关闭计算这些信息。现在，**git status** 在被 ahead/behind 计算拖慢时会显示此提示。

# 处理包含大型二进制文件的仓库

在某些特定情况下，你可能需要在代码库中跟踪 **巨大的二进制资产**。例如，游戏团队需要处理庞大的 3D 模型，网页开发团队可能需要跟踪原始图像资产或 Photoshop 文档。游戏开发和网页开发都可能需要将视频文件纳入版本控制。此外，有时你可能希望包含那些难以或成本较高的生成的大型二进制交付文件——例如，存储虚拟机镜像的快照。

有一些调整可以改进 Git 对二进制资产的处理。对于从版本到版本变化很大的二进制文件（而不仅仅是某些元数据头部的变化），你可能希望在 `.gitattributes` 文件中显式关闭 `-delta`（参见 *第三章*，*管理你的工作树*，以及 *第十三章*，*定制和扩展 Git*）。Git 会自动对任何超过 `core.bigFileThreshold` 大小的文件关闭增量压缩，默认值为 512 MiB。你可能还希望关闭压缩（例如，如果文件已经是压缩格式）。但是，由于 `core.compression` 和 `core.looseCompression` 对整个仓库都是全局设置，因此如果二进制资产位于单独的仓库（子模块）中，会更有意义。

## 将二进制资产文件夹拆分为单独的子模块

处理大规模二进制资源*文件夹*的一种可能方法是将其拆分到一个单独的仓库中，并将资源作为**子模块**拉取到你的主项目中。使用子模块可以让你控制何时更新资源。此外，如果开发者不需要这些二进制资源来工作，他们可以简单地排除包含资源的子模块，从而避免拉取这些资源。

限制是你需要为这些巨大的二进制资源准备一个单独的文件夹。如果选择这种方式进行处理，托管包含这些大资源的子模块仓库的服务还需要能够存储这些大文件；许多 Git 托管网站对单个文件或整个仓库的最大大小设置了严格的限制。

## 将大规模二进制文件存储在仓库之外

另一种解决方案是使用许多第三方工具之一，这些工具尝试解决在 Git 仓库中处理大二进制文件的问题。它们中的许多使用类似的方案，即将庞大的二进制文件内容存储在仓库外部，同时提供某种方式的指针来引用检出的内容。

每种实现有三个部分：

+   它们如何在仓库内存储关于管理文件内容的信息

+   它们如何在团队之间共享大规模二进制文件

+   它们如何与 Git 集成（以及它们的性能惩罚）

在选择解决方案时，你需要考虑这些数据以及操作系统支持、易用性和社区的大小。

仓库中存储的内容和被检入的内容可能是文件的*符号链接*或密钥，或者可能是一个*指针文件*（通常是纯文本），它作为实际文件内容的引用（通过名称或文件内容的加密哈希）。被跟踪的文件需要存储在某种后端中以便协作（云服务、rsync、共享目录等）。后端可以由客户端直接访问，或者可能有一个单独的服务器，提供一个定义好的 API，供二进制文件（blob）写入，并将存储卸载到其他地方。

该工具可能要求使用单独的命令来检出和提交大文件，或从后端获取和推送文件，或者它可能已经集成到 Git 中。集成解决方案使用`clean`/`smudge`过滤器透明地处理检出和检入操作，并使用`pre-push`钩子将大文件内容一同透明地推送。你只需要指定要跟踪的文件，并且当然需要初始化仓库以便工具使用。

基于过滤器的方法的优点在于其易用性；然而，由于该方法的工作方式，它会带来性能上的惩罚。使用单独的命令来处理大规模二进制资源会让学习曲线稍微陡峭，但它提供了更好的性能。一些工具同时提供这两种接口。

在不同的解决方案中，有**git-annex**，它拥有一个庞大的社区，并支持各种后端，还有**Git 大文件存储**（**Git-LFS**），由 GitHub 创建，提供了良好的 Microsoft Windows 支持、客户端-服务器模式，并具有透明性（支持基于过滤器的方式）。Git-LFS 扩展不仅由 GitHub 支持，其他 Git 托管站点和软件平台，如 GitLab、Bitbucket 和 Gitea，也都支持 Git-LFS。也存在专门的服务和项目来实现 Git-LFS。

还有许多其他类似的工具，但这两种是最流行的，而且都仍在维护中。

数据分析和机器学习的数据文件版本控制

机器学习项目通常处理大文件或大量文件。这些文件包括原始数据集，但也包括各种预处理步骤的结果，以及训练后的模型。

你希望将这些大文件或目录存储到某个地方，以避免需要重新下载或重新计算它们。另一方面，你也希望能够从头开始重建一切，以使科学研究具有可重复性。这些需求与典型的软件项目中处理大资产所遇到的需求有很大不同，后者需要专门的解决方案来整合数据处理和版本控制。在这些解决方案中，有**数据版本控制**（**DVC**）和**Pachyderm**。

# 处理包含大量文件的仓库

单一仓库（monorepo）使用的增多（这一概念在*第十一章**《管理子项目》*中有详细解释）导致了对处理大量文件的仓库的需求。在一个单一仓库中——即由多个相互关联的子项目组成的仓库——你通常会专注于一个子项目，并仅在特定的子目录中访问和修改文件。

## 使用稀疏检出限制工作目录中文件的数量

Git 包含了 `core.sparseCheckout` 配置变量，并将其设置为 `true`，同时使用 `.git/info/sparse-checkout` 文件，采用类似 gitignore 的语法来指定工作目录中要出现的内容。索引（也称为暂存区）被完全填充，对于缺少的文件，设置了 `skip-worktree` 标志。

虽然如果你有一个庞大的文件夹树，这种方式可能会有所帮助，但它并不会影响本地仓库本身的整体大小。为了减少仓库的大小，必须结合使用**稀疏克隆**（稍后会介绍）。

然而，稀疏检出定义非常通用。这使得该功能非常灵活，但代价是对于大规模定义和大量文件来说性能较差。对于单一仓库，你不需要这种灵活性，因为每个子项目都包含在自己的子目录中——你只需要在稀疏检出定义中匹配目录即可。

要实现这一点，你需要使用已经弃用的 `sparse-checkout` 功能（例如，参见 `git sparse-checkout` 命令的文档）。这种模式的额外优点是它更容易使用。所有操作都由 `git sparse-checkout` 命令管理。

要将你的工作目录限制为特定目录集，运行以下命令：

```
$ git sparse-checkout set <directory_1> <directory_2>
```

该功能的早期版本要求你先运行 `git sparse-checkout init --cone`，但现在不再需要使用此命令，`init` 子命令本身也正在被弃用。

提示

如果你正在克隆一个包含大量文件的仓库，你可以通过使用 **--no-checkout** 或 **--sparse** 选项来避免将文件填充到工作目录中（第二个选项只会签出项目顶级目录中的文件）。你还可以添加 **--filter=blob:none** 选项以获得更快的速度（启用无 blob 的稀疏克隆）。

在任何时候，你都可以使用以下命令检查哪些目录被包括在你的 `sparse-checkout` 定义中，并存在于你的工作目录中：

```
$ git sparse-checkout list
```

你可以通过 `add` 子命令将一个新目录添加到现有的稀疏签出中，如下所示的示例所示：

```
$ git sparse-checkout add <new_directory>
```

在撰写本文时，尚无 `remove` 子命令。要从已签出的文件列表中删除一个目录，你需要编辑 `.git/info/sparse-checkout` 文件的内容，然后运行以下子命令：

```
$ git sparse-checkout reapply
```

此子命令重新应用现有的稀疏目录规格，以使工作目录匹配。当某些操作更新工作目录但未完全遵循 `sparse-checkout` 定义时，也可以使用此命令。这可能是由使用 Git 外部工具，或者运行不完全支持稀疏签出的 Git 命令导致的。

你可以通过运行 `git sparse-checkout` `disable` 命令关闭此功能，并恢复工作目录以包括所有文件。

## 使用稀疏克隆减少本地仓库大小

*第十一章*的初始部分，*管理子项目*，描述了 Git 如何存储项目的历史记录，其中包括每次修订的变更描述、目录结构和文件内容。这些数据使用不同类型的对象存储：标签对象、提交对象、树对象和二进制对象。对象之间互相引用：标签指向提交，提交指向父提交，树代表某一修订时项目的状态，树指向其他树和二进制对象。

当运行普通的 `git clone` 命令时，客户端会向服务器请求最新的提交（代表最新的修订）。服务器提供这些对象、它们所指向的所有对象、以及这些对象所指向的对象，依此类推。简而言之，服务器提供了那些提交对象和所有其他可达的对象（可能排除客户端已经拥有的那些对象）。结果就是你在本地拥有了整个项目的完整历史。

然而，如今许多开发者在工作时都有可用的网络连接。现代 Git 只允许你通过 **部分克隆** 下载对象的一个子集。在这种情况下，Git 会记住从哪里可以获取其余的对象，当有必要时，它会稍后向服务器请求更多数据。

通过在运行 `git clone` 命令时指定 `--filter` 选项，可以启用 Git 的部分克隆功能。有几种可用的过滤器，但托管你克隆的仓库的服务器可以选择拒绝你的过滤器并恢复到创建完全克隆。

在稀疏克隆中运行 `git fetch` 会保持稀疏克隆过滤器，并且不会下载初次克隆时不会下载的那些类型的对象。

最常用的两种过滤器应该是大多数 Git 托管站点支持的，如下所示：

+   **无 Blob 克隆**：**git clone --****filter=blob:none <url>**

+   **无树克隆**：**git clone --****filter=tree:0 <url>**

使用 `--filter=blob:none` 选项时，初始的 `git clone` 命令会下载所有内容，除了 blob 对象（这些对象通常包含不同版本的文件内容）。如果没有被抑制，`clone` 操作的 checkout 部分会下载当前版本项目文件的 blobs。Git 客户端知道如何将这些下载请求批量化，只向服务器请求缺失的 blobs。

使用 `git log`、`git merge-base` 和其他不检查文件内容的命令时，运行不会需要额外下载 blob 对象。

此外，为了检查文件是否已更改，Git 只需比较对象 ID，而无需访问实际内容。因此，使用 `git log -- <path>` 检查文件历史记录时，也无需下载任何对象。此命令的运行性能与完全克隆时相同。这是因为对象 ID 基于文件内容的加密哈希（目前 Git 使用 SHA-1 实现此功能）。

Git 命令如 `git checkout`/`git switch`、`git reset --hard <revision>` 和 `git merge` 需要下载 blobs 以填充工作目录、索引（暂存区）或两者。为了计算差异，Git 还需要有 blobs 来进行比较；因此，像 `git diff` 或 `git blame <path>` 这样的命令，第一次运行时可能会根据特定的参数触发 blob 下载。

在某些代码库中，树状数据可能占据代码库大小的相当大一部分。如果代码库包含大量文件和目录，并且目录层次结构深且宽，则可能会发生这种情况。在这种情况下，使用`--filter=tree:0`选项可能会提供更好的解决方案。

请注意，任何仅由由于所选过滤器而跳过的对象所引用的对象也将丢失。这意味着无树克隆比无 blob 克隆更加稀疏（因为只有树能指向 blob…当然，标签对象也可以指向 blob 对象，但你通常不会遇到这种情况）。

无树克隆相对于无 blob 克隆的优点是初始克隆速度更快，后续获取速度更快。缺点是，在无树克隆中工作更为困难，因为在需要时下载缺失的树的成本更高。服务器也更难以察觉客户端已经在本地拥有某些树对象，因此请求可能会发送比必要的更多数据。此外，更多的命令需要额外的数据下载。例如，`git log -- <file>`命令在无 blob 克隆中可以运行而无需额外下载任何内容。而在无树克隆中，这个命令几乎会为每个历史提交下载树。

无树克隆与子模块

包含子模块的代码库（参见*第十一章*，*管理子项目*）在使用无树克隆时可能会表现不佳。如果收到过多的树下载请求，你可以通过确保**fetch.recurseSubmodules**配置变量设置为**false**（或使用**--no-recurse-submodules**选项）来*关闭自动获取子模块*，或者通过设置**clone.filterSubmodules**配置选项（或使用**--recurse-submodules --filter=tree:0 --also-filter-submodules**命令行选项组合）来*同时过滤子模块*。

无树克隆对于自动构建非常有用，当你想快速克隆项目，检出单个修订版本，编译它和/或运行测试，然后丢弃代码库（而不是使用浅克隆）时，它非常有效。如果你只对查看整个项目的历史感兴趣，它也很有用。

无树克隆是`--filter=tree:<depth>`的一个特例。在这种情况下，克隆会省略所有树和 blob，这些树的深度从根树（从项目的顶级目录）到达或超过指定的限制。可以很容易地看出，当`<depth>`等于 0（即`--filter=tree:0`）时，克隆将不包括任何树或 blob（除了初始检出所需的那些）。

### 通过稀疏克隆省略大型文件内容

部分克隆也可以作为处理大文件的工具。这要求 Git 服务器（Git 托管站点）支持特定类型的过滤器。它同样不会取消至少一个远程仓库必须包含那些大文件及其历史记录的要求，以便你可以按需下载它们。

你可以通过在克隆时提供 `--filter=blob:limit=<size>` 选项来实现，其中 `<size>` 可以包括 `<size>` 字节，无论是 KiB、MiB 还是 GiB（取决于后缀）。例如，`blob:limit=1k` 等同于 `blob:limit=1024`。

### 将克隆稀疏性与检出稀疏性匹配

现代 Git 包括对稀疏克隆过滤器的基本支持，该过滤器使其省略所有对于稀疏检出不需要的 blob。然而，由于安全原因，Git 放弃了对 `--filter=sparse:path=<path>` 的易用形式的支持。支持的形式是 `--filter=sparse:oid=<blob-ish>`。这种形式可以防止“检查时到使用时”问题，因为 `<blob-ish>`（即对 blob 对象的引用）最终解析为定义其内容的对象 ID。

在写作时，你很难找到一个支持此过滤器且不会响应 **警告：服务器不识别过滤器，已忽略** 的 Git 服务器。但当它开始得到广泛支持时，一个可能的解决方案是为每个感兴趣的稀疏检出模式创建一个标签，然后使用选定的标签作为 blob-ish：

```
$ git sparse-checkout set <subdir>
$ git hash-object -t blob -w .git/info/sparse-checkout
bd177ff9527327c67f50c644c421d280bb8b55f5
$ git tag -a -m 'sparse-checkout pattern for <subdir>'
  sparse/<subdir> bd177ff9527327c67f50c644c421d280bb8b55f5
$ git push origin tag sparse-checkout/<subdirectory>
To <repository url>
* [new tag]         sparse/<subdir> -> sparse/<subdir>
```

当然，在第三步中，你需要使用之前命令输出的 SHA-1。

在这种情况下，克隆将使用`--filter=sparse:oid=sparse/<subdir>^{blob}`选项（其中需要使用之前显示的命令序列所创建的标签名）。

## 使用文件系统监控器更快地检查文件变更

当你运行一个操作工作树的 Git 命令时，如`git status`或`git diff`，Git 必须发现相对于索引或相对于指定版本的变化。它通过搜索整个工作树来完成这项任务，而对于包含大量文件的仓库，这可能需要很长时间。每次你运行这样的命令时，它还必须从头开始重新发现相同的信息。

文件系统监控器是一个长期运行的守护进程或服务进程，其功能如下：

+   向操作系统注册以监视指定目录，并接收相关目录和文件的变更通知事件

+   将已更改的文件和目录的路径名保存在某些（内存中的）数据结构中，这些数据结构可以快速查询

+   响应客户端请求，提供最近修改的文件和目录的列表

从版本 2.37 开始，Git 包含了`git fsmonitor--daemon`。目前它在 macOS 和 Windows 上可用。该守护进程监听来自客户端进程（如`git status`）的 IPC 连接，并通过 Unix 域套接字或命名管道发送已更改文件的列表。

启用它非常简单；你只需要配置 Git 使用它。可以通过以下命令完成：

```
$ git config core.fsmonitor true
```

该监视器与`core.untrackedCache`一起工作良好，因此建议将此配置选项设置为`true`。

你可以查询此守护进程以获取被监视仓库的列表：

```
$ git fsmonitor--daemon status
fsmonitor-daemon is watching 'C:/work/chromium'
```

如果操作系统或仓库所在的文件系统不允许你使用此监视器，可以使用`core.fsmonitor`配置选项指定文件系统监视器钩子的路径。该钩子必须支持`fsmonitor-watchman`钩子协议，并且在运行时会将已更改的文件列表输出到标准输出。

Git 附带了`fsmonitor-watchman.sample`文件，该文件安装在`.git/hooks/`目录中。在启用之前，如前所述，需通过删除`*.sample`后缀来重命名该文件。如果文件丢失，你可以从[`github.com/git/git/tree/master/templates`](https://github.com/git/git/tree/master/templates)下载该文件。此钩子需要安装**Watchman**文件的监视服务([`facebook.github.io/watchman/`](https://facebook.github.io/watchman/))。

# 总结

本章提供了处理大型 Git 仓库的解决方案，从使用 Scalar 工具到专门的解决方案。

首先，你学会了如何使用浅克隆下载并操作项目历史的选定浅子集。

然后，你学会了如何通过将大文件存储在仓库外部或将其拆分成子模块来处理大文件。还简要提到了数据科学项目中大数据的问题，以及针对这一问题的专门解决方案。

最终，你学会了如何使用稀疏检出、稀疏克隆和文件系统监控来管理大型单体仓库。

下一章将帮助你使 Git 更易于使用，并更好地适应你的特定需求。这包括配置仓库维护，这对于使大型仓库的工作更加顺畅至关重要。

# 问题

回答以下问题以测试你对本章的理解：

1.  处理大型仓库的最简单解决方案是什么？

1.  如何加速长历史仓库的克隆？

1.  如何处理只被部分开发者需要的大文件？

1.  使用大量文件的仓库时，哪些技术可以加速工作？

1.  浅克隆、稀疏克隆和稀疏检出的区别是什么？

# 答案

以下是本章问题的答案：

1.  使用内置的**scalar**工具，无论是使用它来克隆仓库，还是将给定仓库注册到该工具。

1.  你可以使用浅克隆或无 Blob 稀疏克隆。在第一种情况下，你将获得一个简化的历史记录，而在第二种情况下，仓库的大小将更小，但某些操作需要网络访问以下载额外的数据。

1.  你可以使用 Git-LFS 或 git-annex（或类似解决方案）将大型文件存储在仓库之外。你可以使用稀疏克隆功能克隆仓库，而无需下载大型文件数据。

1.  如果你只在特定子目录中工作，请使用稀疏签出功能；使用稀疏克隆来减少仓库大小；并且使用文件系统监控（如果可能的话）来加快操作。

1.  浅克隆只下载选定的部分仓库历史记录，所有本地操作仅限于该选择，尽管改变历史深度很容易。稀疏克隆通过仅下载选定的对象子集来减少仓库大小，按需获取这些对象，随着操作的进行，当它们的存在变得必要时才会下载。稀疏签出减少了已签出的文件数量，使工作目录变得更小（并且操作更快）。

# 深入阅读

要了解本章涉及的更多内容，请查阅以下资源：

+   *介绍 Scalar：面向每个人的 Git 大规模管理*，作者 Derrick Stolee（2020）：[`devblogs.microsoft.com/devops/introducing-scalar/`](https://devblogs.microsoft.com/devops/introducing-scalar/)

+   *Scalar 的故事*，作者 Derrick Stolee 和 Victoria Dye（2022）：[`github.blog/2022-10-13-the-story-of-scalar/`](https://github.blog/2022-10-13-the-story-of-scalar/)

+   *scalar(1) - 管理大型 Git* *仓库的工具*：[`git-scm.com/docs/scalar`](https://git-scm.com/docs/scalar)

+   *超级增强 Git 提交图*，作者 Derrick Stolee（2018）：[`devblogs.microsoft.com/devops/supercharging-the-git-commit-graph/`](https://devblogs.microsoft.com/devops/supercharging-the-git-commit-graph/)

+   *git-commit-graph(1) - 写入和验证 Git 提交图* *文件*：[`git-scm.com/docs/git-commit-graph`](https://git-scm.com/docs/git-commit-graph)

+   *Git LFS - Git 大文件* *存储*：[`git-lfs.com/`](https://git-lfs.com/)

+   *git-annex*：[`git-annex.branchable.com/`](https://git-annex.branchable.com/)

+   *跟上部分克隆和浅克隆的进度*，作者 Derrick Stolee（2020）：[`github.blog/2020-12-21-get-up-to-speed-with-partial-clone-and-shallow-clone/`](https://github.blog/2020-12-21-get-up-to-speed-with-partial-clone-and-shallow-clone/)

+   *使用稀疏签出将你的单体仓库缩小尺寸*，作者 Derrick Stolee（2020）：[`github.blog/2020-01-17-bring-your-monorepo-down-to-size-with-sparse-checkout/`](https://github.blog/2020-01-17-bring-your-monorepo-down-to-size-with-sparse-checkout/)

+   *git-sparse-checkout(1) - 将工作树缩减为跟踪的* *文件子集*：[`git-scm.com/docs/git-sparse-checkout`](https://git-scm.com/docs/git-sparse-checkout)

+   *git-clone(1) - 克隆一个仓库到新的* *目录中*: [`git-scm.com/docs/git-clone`](https://git-scm.com/docs/git-clone)

+   *通过文件系统监视器提高 Git 单体仓库性能*，Jeff Hostetler（2022）：[`git.blog/2022-06-29-improve-git-monorepo-performance-with-a-file-system-monitor/`](https://git.blog/2022-06-29-improve-git-monorepo-performance-with-a-file-system-monitor/)

+   *git-fsmonitor--daemon - 内建文件系统* *监视器*: [`git-scm.com/docs/git-fsmonitor--daemon`](https://git-scm.com/docs/git-fsmonitor--daemon)

+   *githooks - Git 使用的钩子：* *fsmonitor-watchman*: [`git-scm.com/docs/githooks#_fsmonitor_watchman`](https://git-scm.com/docs/githooks#_fsmonitor_watchman)
