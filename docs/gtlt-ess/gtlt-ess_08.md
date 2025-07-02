# 第八章：允许用户创建仓库

直到现在，我们看到的一切表明只有管理员能够创建新仓库。他可以通过授予一些受信任的用户读取和写入`gitolite-admin`仓库的权限来分担这项工作，但这也只是暂时的。

在某些环境中，用户可能需要更多的灵活性。你可能有一些用户，他们不需要是管理员，即他们不希望或不需要管理*所有*仓库，但他们确实希望能够创建自己的仓库并控制对这些仓库的访问。事实上，我们希望管理员的角色仅在初期设置时出现，之后就不需要再对 Gitolite 的*conf*文件进行更改。

如果有多个用户需要这样做，那么思考如何在保持管理员通常方式创建的仓库的必要安全性的同时，允许这种操作似乎是个好主意。

在本章中，我们将解决引言中描述的问题。我们将从一些帮助我们解决部分问题的功能开始讨论，然后逐一添加缺失的部分，最终构建出完整的解决方案。

# 将仓库放入子目录

解决方案的第一部分是，如同第一章中所暗示的，Gitolite 允许你将仓库分组到子目录中，就像在文件系统中管理文件一样。例如，你可以将所有正在管理的开源项目放到名为`foss`的子目录下，像这样：

```
repo foss/apache
 ...access rules for the apache repo...
repo foss/linux
 ...access rules for the linux repo...
...etc...

```

我们可以利用这一点来解决当前的问题。假设我们有用户 Alice 和 Bob，并且我们希望允许他们创建和管理仓库。我们可以想出一种方法，使得 Alice 的仓库位于一个名为`dev/alice`的子目录下，Bob 的仓库同样会位于`dev/bob`中。

# 仓库通配符

仓库通配符是描述一系列可能的仓库名称的正则表达式。例如，`repo` `dev/alice/[a-z].*`这一行表示所有仓库名称以`dev/alice/`开头，后跟一个字母字符，后面可选跟随其他内容。仓库`dev/alice/foo`符合条件，但`dev/alice/123`则不符合，`dev/alice`也不符合。

### 提示

由于需要正确表示诸如`gtk+`和`c++`这样的仓库名称，如果`+`字符是仓库名称中唯一的正则表达式元字符，它将被视为普通的仓库，而不是仓库通配符。要指定`foo.+`，你应该改为使用`foo..*`。你也可以使用`[f]oo.+`—方括号的存在告诉 Gitolite 它是一个正则表达式。

Gitolite 允许`repo`行使用通配符而不是单个仓库名称。这为我们的解决方案迈出了下一步；我们现在可以写：

```
repo dev/alice/[a-z].*
    RW+       =  alice
    RW        =  bob
    R         =  @all
repo dev/bob/[a-z].*
    RW+       =  bob
    RW        =  alice
    R         =  @all
```

这表示 Bob 可以将分支推送（但不能回滚或删除）到 Alice 所有的仓库，反之亦然，而且系统中其他经过认证的用户可以克隆这两组仓库。

可惜的是，通配符仓库规范本身并不会真正*创建*任何仓库，因为这个模式本身可以匹配数十亿个可能的仓库名称！

## 创建通配符仓库

现在是时候介绍 Gitolite 提供的额外语法的第一部分，以帮助解决我们在本章开始时提到的问题。正如我们所指出的，通配符规范行并不会真正创建 Alice 或 Bob 需要的仓库。

为了实现这个目标，我们在访问规则规范中添加了一行新的内容：

```
repo dev/alice/[a-z].*
 C    =  alice
 ...other rules stay the same...
repo dev/bob/[a-z].*
 C    =  bob
 ...other rules stay the same...

```

### 提示

这与我们在《第七章》*高级访问控制与配置*部分看到的 *创建分支* 权限有所不同。这里描述的是单个字母 `C`，而另一个只能作为 `RW` 或 `RW+` 的修饰符存在。

这个访问规则表示，通过克隆或推送一个匹配名称的仓库，Alice *可以导致仓库在服务器上被创建*! 换句话说，Alice 可以运行以下命令：

```
$ git clone  git@host:dev/alice/my-new-repo
Cloning into 'my-new-repo'...

Initialized empty Git repository in /home/git/repositories/dev/alice/my-new-repo.git/

warning: You appear to have cloned an empty repository.

```

这就像管理员添加了一组新的规则，内容如下：

```
repo dev/alice/my-new-repo
 RW+       =  alice
 RW        =  bob
 R         =  @all

```

只是管理员不需要这么做！

别忘了在前面的输出中看到`Initialized empty...`这一行；这是服务器的反馈，告诉你这个克隆操作创建了一个全新的仓库！此外，如果 Alice 执行 `info` 命令，她可能会看到如下内容：

```
$ ssh git@host info 

hello alice, this is git@host running gitolite3 v3.5.3.1-7-g31d11b9 on git 1.8.3.1

 C  dev/CREATOR/..*
 R W    dev/u1/my-new-repo
 R W    testing

```

它显示的是刚刚创建的仓库。

### 提示

一个缺点是，一个简单的拼写错误就可能导致创建无用的仓库。如果你希望防止这种情况发生，可以编辑 `$HOME/.gitolite.rc` 并取消注释 `create` 命令以及 `no-auto-create` 选项。然后你的用户可以运行更明确的 `create` 命令，例如：`ssh git@host create dev/alice/my-new-repo`。

## 赋予其他用户访问权限

到目前为止，我们只是硬编码了权限—Alice 和 Bob 可以互相访问对方的仓库，其他人可以读取这两个仓库。这显然不够灵活，Alice 可能希望某些仓库可以被 David 写入，某些仓库不允许 `@all` 读取，等等。

从表面上看，这似乎是一个难题，因为这意味着 Alice 可能需要对规则本身进行更改或添加，因此直接或间接地触及到 Gitolite 的 *conf* 文件。

Gitolite 解决这些问题的方式是允许管理员定义角色，然后让用户指定她希望将哪些用户分配到每个角色。以下是使用 Gitolite 默认定义的角色名称的示例：

```
repo dev/alice/[a-z].*
 C          =  alice
 RW+        =  alice
 RW         =  WRITERS
 R          =  READERS

```

`READERS`和`WRITERS`是 Gitolite 中预定义的角色名称。请注意，角色名称本身并没有特殊的含义，关于角色的访问权限完全由管理员决定。

目前，然而，这还不完整。规则并没有明确说明 Bob 是一个写作者（因此在 Alice 的仓库上拥有`RW`权限），或者其他人（`@all`）可以读取它。

这时`perms`命令就派上用场了。以下是 Alice 如何使用它将 Bob 添加到`WRITERS`角色的方式：

```
ssh  git@host perms dev/alice/my-new-repo + WRITERS bob

```

类似地，要将`@all`添加到`READERS`角色，她将运行以下命令：

```
ssh  git@host perms dev/alice/my-new-repo + READERS @all

```

如果她希望检查当前仓库的角色分配情况，可以运行。

```
ssh  git@host perms -l dev/alice/my-new-repo

```

它将恭敬地打印：

```
READERS @all
WRITERS bob

```

你应该立刻看到的一个优势是，现在角色分配是*按仓库来划分*的。也就是说，Alice 可以为她拥有的其他仓库设置完全不同的角色分配。事实上，如果她没有在某个仓库上运行`perms`命令，其他人将无法访问该仓库——它将变成她的私人仓库。或者，如果她愿意，也可以为每个角色添加多个用户。`perms`命令一次只能添加一个用户到一个角色。因此，Alice 可能需要多次运行这个命令。如果她添加了某个用户，现在又想移除他，也是可以的。与所有 Gitolite 命令一样，`perms`支持`-h`参数来提供用法说明。

## 泛化规则集

现在我们将查看每个用户当前的规则集，并讨论如何将其泛化到任意数量的用户。此时，如果你还记得上一节的内容，规则看起来如下：

```
repo dev/alice/[a-z].*
 C         =  alice
 RW+       =  alice
 RW        =  WRITERS
 R         =  READERS

```

你可能会注意到，除了前面三行中的`alice`被替换成 Bob 外，这实际上正是你为 Bob 的仓库所需要的。显然，要求为每个可能需要此功能的用户重复这一点是没有意义的！

允许用户创建并（在某种程度上）管理自己的仓库的解决方案的最后部分是`CREATOR`关键字。这里是经典的示例：

```
repo dev/CREATOR/[a-z].*
 C         =  alice bob carol dave
 RW+       =  CREATOR
 RW        =  WRITERS
 R         =  READERS

```

注意我们所做的更改。首先，`C`权限行现在列出了所有被允许创建和管理自己仓库的用户，正如我们之前所描述的那样。在这个规则集中，只有这四个用户可以执行此操作。或者，你也可以用某个已定义的组名替换这四个用户名，甚至使用`@all`来允许所有经过身份验证的用户使用创建私有仓库并选择性地向其他人开放的功能。

接下来，仓库名称模式包含`CREATOR`而不是`alice`，`RW+`行也是如此。对于尚不存在的仓库，这些实际上被视为试图创建仓库的用户的名字。对于已存在的仓库，它被视为创建该仓库的用户的名字，并且该信息会被记录和追踪。

### 提示

当仓库不存在时，Gitolite 关心的唯一权限是允许创建仓库的`C`权限。符合规则右侧的任何人都可以创建匹配模式的仓库。要注意的一个错误是将`C = CREATOR`放入其中，而不是实际用户或用户组的列表。因为如上所述，这会被视为尝试创建仓库的用户的名称，这允许任何经过身份验证的用户创建这样的仓库。如果这是你想要的，最好明确说明，实际上使用`@all`而不是`CREATOR`；后者只是一个副作用，不是受支持的行为。

# 向你的用户解释野生仓库

当然，你的用户并不需要所有这些解释！事实上，此功能的一个目标是使 Gitolite 用户（而不是 Gitolite *管理员*）不必负担学习`RW`、`RW+`、*deny*规则等细微差别。

到目前为止，我们一直在使用的示例是经典示例：它包含一个所有者（只有该所有者被允许回滚或删除分支的用户），一组作者（可以推送/创建但不能回滚/删除），以及一组读者根本不能推送。

唯一剩下的事情就是向用户解释她被允许创建哪些仓库（大多数用户对正则表达式不是很熟悉，因此最好保持模式足够简单，可以用英语解释），以及她可以向每个*用户列表*添加或删除人员。

列表名称（在我们的示例中是`READERS`和`WRITERS`）应当提供，并解释为代表每个列表中的用户可以做什么。

最后，展示三个示例`perms`命令用于维护这些用户列表：

+   `ssh` git@host `perms dev/alice/repo + WRITERS dave` 以添加一个用户

+   `ssh` git@host `perms dev/alice/repo - WRITERS dave` 以移除一个用户（请注意减号而不是加号）

+   `ssh` git@host `perms -l dev/alice/repo` 列出当前用户列表

# 只使用野生仓库进行管理

如果你考虑我们在本章中一直在工作的示例，它不允许用户信任任何其他人拥有`RW+`权限；如果需要回滚或删除分支，必须是所有者自己执行。

我们可以通过将`RW+`权限行更改为以下内容来纠正这一点：

```
 RW+    =  CREATOR TRUSTED

```

因此，定义一个名为`TRUSTED`的新角色（或者说用户列表）。当然，为了使其生效，作为管理员的你需要登录服务器，并编辑`$HOME/.gitolite.rc`文件，在`ROLES`哈希下的角色列表中添加这个新角色。然后你可以告诉你的用户，有一个名为`TRUSTED`的第三个用户列表，她可以用来指定她想要允许回滚或删除分支或标签的用户。

既然我们已经开始走这条路，那么我们可以继续走得更远，直到最终我们得到的东西基本上就是 Gitolite 的一次性设置，管理员在设置完后几乎不需要再进行任何维护。这在大多数用户相对自主的网站上非常有用。

下面是一个完整的规则集示例。为了方便复制和使用，我们在规则集的描述中添加了注释，这样你也可以复制它们：

```
# completely private repo; no sharing even possible
repo private/CREATOR/..*
 C          =  @all
 RW+        =  CREATOR

# public template; anyone can read but writes only by owner
repo public/CREATOR/..*
 C          =  @all
 RW+        =  CREATOR
 R          =  @all

# a controlled repo with 3 roles allowing RW+, RW, and R
repo controlled/CREATOR/..*
 C         =  @all
 RW+       =  CREATOR
 RW+       =  TRUSTED
 RW        =  WRITERS
 R         =  READERS

# a "corporate" type template with managers, testers, etc.
# Junior devs cant write 'master', can't rewind.  Testers
# (and *only* testers) can push versioned tags.  Managers
# can read any repo.
repo corporate/CREATOR/..*
 C               =  @all
 RW+             =  CREATOR
 RW refs/tags/v[0-9]  =  TESTERS
 -  refs/tags/v[0-9]  =  @all
 RW+             =  SENIOR_DEVS
 -  master       =  JUNIOR_DEVS
 RW              =  JUNIOR_DEVS
 R               =  MANAGERS

```

# 删除不再需要的仓库

使用上一节中的示例，管理员的工作负担大大减轻（尽管这是以*失去一些控制权*为代价的）。然而，仍然有一个功能是你的用户最终会需要的：删除那些已经完成任务的仓库。

为了允许用户删除他们创建的仓库（无需提及用户不能删除其他任何内容！），管理员需要通过取消注释 `$HOME/.gitolite.rc` 文件中相应行的命令来启用 `D` 命令。然后用户可以运行 `D` 命令删除仓库。

默认情况下，仓库会被锁定，防止意外删除，因此每次删除实际上分为两个步骤——`unlock` 子命令和 `rm` 子命令：

```
ssh  git@host D unlock dev/alice/my-new-repo
ssh  git@host D rm dev/alice/my-new-repo

```

# 总结

这一章可能是本书中最重要的一章，因为它讲解了 Gitolite 的一个非常流行的功能。虽然在需要严格控制和可审计性的网站上不适用，但在大多数其他站点中非常有用，不仅能为管理员节省大量时间，而且用户也不需要再等待管理员处理他们急需的事项。

在下一章中，我们将讨论 **核心** Gitolite 和 **非核心** Gitolite 的区别，查看 Gitolite 附带的一些非核心程序，并讨论如何通过添加我们自己的非核心代码来自定义 Gitolite。
