# 第八章 使用 Vault 加密数据

使用变量时，我们看到如何将数据与代码分离。通常，提供的数据是敏感的，例如用户密码、数据库凭证、API 密钥以及其他组织特定的信息。Ansible-playbook 作为源代码，通常存储在版本控制仓库中，如**git**，这使得在协作环境中保护这些敏感信息变得更加困难。从版本 1.5 开始，Ansible 提供了一种名为**vault**的解决方案，用于使用经过验证的加密技术安全地存储和检索这些敏感信息。使用 vault 的目的是加密数据，这样就可以在版本控制系统（如 git）中自由存储和共享，而不会泄露数据值。

在本章中，我们将学习以下主题：

+   理解 Ansible-vault

+   使用 Ansible-vault 保护数据

+   加密、解密和重新密钥操作

# Ansible-vault

Ansible 提供了一种名为 Ansible-vault 的工具，正如其名字所示，它让你可以安全地管理数据。Ansible-vault 工具可以通过启动编辑器界面来创建加密文件，或者加密现有文件。在这两种情况下，它都会要求输入一个 vault 密码，该密码将用于使用 AES 密码进行加密。加密后的内容可以存储在版本控制系统中，而不会泄露。由于 AES 基于共享密钥，加密和解密都需要提供相同的密码。为了提供密码，可以有两种选择，启动 Ansible 时，使用`--ask-vault-pass`选项提示输入密码，或者使用`--vault-password-file`选项提供包含密码的文件路径。

## 高级加密标准

**高级** **加密标准**（**AES**）是一种基于**Rijndael**对称块密码的加密标准，命名自比利时两位密码学家——文森特·瑞门（Vincent Rijmen）和琼·戴门（Joan Daemen）。最初由美国**国家标准与技术研究院**（**NIST**）于 2001 年建立，AES 是美国政府用于共享机密信息的算法，并且是最流行的对称密钥加密算法。AES 也是第一个被**国家安全局**（**NSA**）批准的公开可访问的开放密码算法。

作为一种开放且流行的标准，Ansible 使用 256 位密钥大小的 AES 密码来加密 Vault 中的数据。

## 使用 Vault 加密什么？

Ansible-vault 可以加密任何结构化数据。由于 YAML 本身就是一种结构化语言，几乎你为 Ansible 编写的所有内容都符合这个标准。以下是可以使用 Vault 加密的内容的指引：

+   最常见的是我们加密变量，变量可能如下所示：

    +   角色中的变量文件，例如`vars`和`defaults`

    +   清单变量，例如`host_vars`，`group_vars`

    +   通过`include_vars`或`vars_files`包含的变量文件

    +   通过 `-e` 选项传递给 Ansible-playbook 的变量文件，例如，`-e @vars.yml` 或 `-e @vars.json`

+   由于任务和处理程序也是 JSON 数据，因此可以使用 vault 加密它们。然而，这种做法应该尽量避免。建议你加密变量，并在任务和处理程序中引用它们。

以下是关于什么不能使用 vault 加密的说明：

+   由于 vault 的加密单位是文件，因此无法加密部分文件或值。你只能加密完整的文件，或者不加密任何内容。

+   文件和模板无法加密，因为它们可能不像 JSON 或 YML 文件那样标准化。

以下数据是加密的好候选项：

+   凭证，例如，数据库密码和应用程序凭证

+   API 密钥，例如，AWS 访问密钥和密钥对

+   Web 服务器的 SSL 密钥

+   部署的私有 SSH 密钥

# 使用 Ansible-vault

以下表格列出了 Ansible-vault 工具附带的所有子命令：

| 子命令 | 描述 |
| --- | --- |
| `create` | 该命令用于从头创建一个加密文件，使用编辑器。这需要在启动命令前设置编辑器环境变量。 |
| `edit` | 该命令用于编辑现有的加密文件，无需解密其内容。 |
| `encrypt` | 该命令用于加密一个包含结构化数据的现有文件。 |
| `decrypt` | 该命令用于解密文件。使用时请小心，并且不要将解密后的文件提交到版本控制。 |
| `rekey` | 该命令用于更改加密或解密所使用的密钥或密码。 |

## 加密数据

让我们使用 Ansible-vault 执行一些操作。我们将从创建一个加密文件开始。要从头创建新文件，Ansible-vault 使用 `create` 子命令。在使用该子命令之前，重要的是在环境中设置编辑器，如下所示：

```
# setting up vi as editor
$ export EDITOR=vi
# Generate a encrypted file
$ ansible-vault create aws_creds.yml
Vault password:
Confirm Vault password:

```

启动此命令会打开指定的编辑器。以下是你可能创建的 `aws_creds.yml` 文件示例，该文件用于存储 AWS 用户凭证（包括访问密钥和秘密密钥）。这些密钥随后用于向亚马逊 Web 服务云平台发起 API 调用。保存此文件并退出编辑器将生成一个加密文件：

![加密数据](img/B03800_08_01.jpg)

你可以通过运行以下命令检查创建的文件类型及其内容：

```
# Check file type and content
$ file aws_creds.yml
aws_creds.yml: ASCII text
$ cat aws_creds.yml
$ANSIBLE_VAULT;1.1;AES256
64616236666362376630366435623538336565393331333331663663636237636335313234313134
3337303865323239623436646630336239653864356561640a363966393135316661636562333932
61323932313230383433313735646438623032613635623966646232306433383335326566343333
3136646536316261300a616438643463656263636237316136356163646161313365336239653434
36626135313138343939363635353563373865306266363532386537623463623464376134353863
37646638636231303461343564343232343837356662316262356537653066356465353432396436
31336664313661306630653765356161616266653232316637653132356661343162396331353863
34356632373963663230373866313961386435663463656561373461623830656261636564313464
37383465353665623830623363353161363033613064343932663432653666633538

```

## 更新加密数据

要更新已添加到加密文件中的 AWS 密钥，你可以使用 Ansible-vault 的 `edit` 子命令，方法如下：

```
$ ansible-vault edit aws_creds.yml
Vault password:

```

`edit` 命令执行以下操作：

1.  提示输入密码

1.  使用 AES 对称密码解密文件

1.  打开编辑器界面，允许你更改文件的内容

1.  在保存后再次加密文件

还有一种更新文件内容的方法。你可以按如下方式解密文件：

```
$ ansible-vault decrypt aws_creds.yml
Vault password:
Decryption successful

```

更新后，该文件可以再次加密，正如你之前学到的那样。

## 轮换加密密钥

作为一种良好的安全实践，定期更换用于 Ansible-vault 的加密密钥是个好主意。当发生这种情况时，必须重新加密所有之前使用 vault 加密的文件。Ansible vault 提供了一个`rekey`子命令，可以按如下方式使用：

```
$ ansible-vault rekey aws_creds.yml
Vault password:
New Vault password:
Confirm New Vault password:
Rekey successful

```

它会要求输入当前密码，然后允许您指定并确认新密码。请注意，如果您通过版本控制管理此文件，您还需要提交该更改。即使实际内容没有变化，重新设置密钥的操作会更新创建的文件，这是我们仓库的一部分。

# 加密数据库凭证

早些时候，在创建数据库用户时，我们将密码作为明文提供在`group_vars`中。这可能是一个潜在的安全威胁，特别是当它被提交到版本控制仓库时。我们来加密它。我们将使用`encrypt`子命令，因为我们已经有了一个变量文件。

由于我们使用`group_vars`组来提供数据库凭证，因此我们将按照以下方式加密`group_vars/all`文件：

```
$ ansible-vault encrypt group_vars/all
Vault password:
Confirm Vault password:
Encryption successful

```

在加密时，Ansible-vault 要求用户输入密码或密钥。使用此密钥，vault 会加密数据并用加密内容替换文件。以下图示显示了`group_vars/all`文件的左侧为明文内容，右侧为相应的加密内容：

![加密数据库凭证](img/B03800_08_02.jpg)

该文件现在可以安全地提交到版本控制系统并共享。然而，用户应注意以下几点：

+   与明文不同，生成的文件是加密格式。无法获取不同的文件格式，例如使用`git diff`来比较提交到版本控制时的更改。

+   无法直接在此文件上使用`grep`、`sed`或任何文本搜索或处理程序。唯一的办法是先解密它，运行文本处理工具，然后再加密回去。

### 提示

确保在一次 Ansible-playbook 运行中使用相同的密码解密所有文件。Ansible 每次只能接受一个密码值，如果同一 playbook 中的文件使用不同的密码加密，程序将失败。

现在让我们使用以下命令运行 Ansible playbook：

```
$ ansible-playbook -i customhosts site.yml
ERROR: A vault password must be specified to decrypt /vagrant/chap8/group_vars/all

```

它会失败并显示错误！这是因为我们提供了加密数据的 playbook，但没有解密它的密钥。Vault 的主要用途是在 Ansible 仓库中保护数据。最终，这些值需要在运行 playbook 时被解密。解密密码可以通过`--ask-vault-pass`选项指定，如下所示：

```
$ ansible-playbook -i customhosts site.yml --ask-vault-pass

```

这将提示输入“Vault 密码”，然后继续按常规方式运行 Ansible 代码。

# 使用密码文件

每次输入密码可能不是理想的做法。通常情况下，你可能还希望自动化 Ansible playbook 的执行过程，在这种情况下，交互式方式不可行。通过将密码存储在文件中，并将该文件提供给 Ansible playbook 运行，可以避免这种情况。密码应作为单行字符串提供在此文件中。

让我们创建一个密码文件，并通过正确的权限来保护它：

```
$ echo "password" > ~/.vault_pass
(replace password with your own secret)
$ chmod 600 ~/.vault_pass

```

### 提示

当 vault 密码以明文存储时，任何访问此文件的人都可以解密数据。确保密码文件以适当的权限保护，并且不被添加到版本控制中。如果你决定进行版本控制，请使用 **gpg** 或同等措施。

现在可以将此文件提供给 Ansible playbook，如下所示：

```
$ ansible-playbook -i customhosts site.yml --vault-password-file ~/.vault_pass

```

# 将 vault 密码文件选项添加到 Ansible 配置中

在版本 1.7 中，还可以在默认部分的 `ansible.cfg` 文件中添加 `vault_password_file` 选项。

请考虑以下内容：

```
[defaults]
  vault_password_file = ~/.vault_pass
```

上述选项让你不必每次都指定加密密码或密码文件。让我们看一下以下命令：

```
# launch ansible playbook run with encrypted data
# with vault_password_file option set in the config
$ ansible-playbook -i customhosts site.yml
$ ansible-vault encrypt roles/mysql/defaults/main.yml
Encryption successful
$ ansible-vault decrypt roles/mysql/defaults/main.yml
Decryption successful

```

此外，从版本 1.7 开始，除了将明文密码存储在文件中外，还可以在 `vault_password_file` 选项中提供一个脚本。在使用脚本时，请确保：

+   脚本上启用了执行权限

+   调用此脚本会将密码输出到标准输出

+   如果脚本提示用户输入，它可以发送到标准错误

## 在模板中使用加密数据

你之前已经了解到，由于模板可能不是像 YAML 或 JSON 这样的结构化文件，因此它不能被加密。不过，还是有一种方法可以向模板中添加加密数据。记住，模板毕竟是动态生成的，而动态内容实际上来自于变量，这些变量是可以加密的。接下来，我们讨论如何通过为 Nginx Web 服务器添加 SSL 支持来实现这一点。

## 为 Nginx 添加 SSL 支持

我们已经设置好了 Nginx Web 服务器，现在让我们通过以下步骤为默认站点添加 SSL 支持：

1.  我们从添加变量开始，如下所示：

    ```
    #file: roles/nginx/defaults/main.yml 
    nginx_ssl: true
    nginx_port_ssl: 443
    nginx_ssl_path: /etc/nginx/ssl
    nginx_ssl_cert_file: nginx.crt
    nginx_ssl_key_file: nginx.key
    ```

1.  让我们还创建自签名的 SSL 证书：

    ```
    $ openssl req -x509 -nodes -newkey rsa:2048 -keyout nginx.key -out nginx.crt

    ```

    上述命令将生成两个文件，`nginx.key` 和 `nginx.crt`。这些文件将被复制到 Web 服务器上。

1.  让我们将这些文件的内容添加到变量中，并创建 `group_vars/www` 文件：

    ```
    # file: group_vars/www
    ---
    nginx_ssl_cert_content: |
        -----BEGIN CERTIFICATE-----
        -----END CERTIFICATE-----
    nginx_ssl_key_content: |
        -----BEGIN PRIVATE KEY-----
        -----END PRIVATE KEY-----
    ```

    在上述示例中，我们仅添加了占位符，这些占位符将被实际的密钥和证书内容替换。这些密钥和证书不应在版本控制系统中暴露。

1.  让我们使用 vault 加密这个文件：

    ```
    $ ansible-vault encrypt group_vars/www
    Encryption successful

    ```

    由于我们已经在配置中提供了 vault 密码的路径，Ansible-vault 就不会再询问密码。

1.  现在让我们创建模板，并添加这些密钥：

    ```
    # filename: roles/nginx/templates/nginx.crt.j2
    {{ nginx_ssl_cert_content }}

    # filename: roles/nginx/templates/nginx.key.j2
    {{ nginx_ssl_key_content }}
    ```

1.  此外，让我们为 SSL 添加一个虚拟主机 `config` 文件：

    ```
    # filename: roles/nginx/templates/nginx.key.j2
    server {
      listen {{ nginx_port_ssl }};
      server_name {{ ansible_hostname }};
      ssl on;
      ssl_certificate {{ nginx_ssl_path }}/{{ nginx_ssl_cert_file }};
      ssl_certificate_key {{ nginx_ssl_path }}/{{ nginx_ssl_key_file }};

      location / {
        root {{ nginx_root }};
        index {{ nginx_index }};
      }
    }
    ```

1.  我们还需要创建一个任务文件来配置 SSL 网站，该文件将创建所需的目录、文件和配置：

    ```
    ---
    # filename: roles/nginx/tasks/configure_ssl.yml
     - name: create ssl directory
        file: path="{{ nginx_ssl_path }}" state=directory owner=root group=root
     - name: add ssl key 
        template: src=nginx.key.j2 dest="{{ nginx_ssl_path }}/nginx.key" mode=0644
     - name: add ssl cert 
        template: src=nginx.crt.j2 dest="{{ nginx_ssl_path }}/nginx.crt" mode=0644
     - name: create ssl site configurations 
        template: src=default_ssl.conf.j2 dest="{{ nginx_ssl_path }}/default_ssl.conf" mode=0644
        notify:
        - restart nginx service
    ```

1.  最后，基于 `nginx_ssl var` 参数是否设置为 true，选择性地调用此任务：

    ```
    # filename: roles/nginx/tasks/main.yml
     - include: configure_ssl.yml
        when: nginx_ssl
    ```

1.  现在，按照以下方式运行 playbook：

    ```
    $ ansible-playbook -i customhosts  site.yml

    ```

这应该配置默认的 SSL 网站，运行在 `443` 端口，并使用自签名证书。现在，你应该能够使用 `https` 安全协议打开 Web 服务器地址，如下所示：

![为 Nginx 添加 SSL 支持](img/B03800_08_03.jpg)

当然，由于我们的证书是自签名的，并未由指定的认证机构提供，因此它应显示一个警告。

# 复习问题

你觉得你已经充分理解了本章的内容吗？尝试回答以下问题来测试你的理解：

1.  为什么需要加密提供给 Ansible playbooks 的数据？

1.  什么是 AES，什么是对称密钥加密算法？

1.  更新之前用 vault 加密的文件的两种方法是什么？

1.  添加到 Ansible 配置文件中的参数是什么，以使其知道 vault 密码文件的位置？

# 摘要

在本章中，你学习了如何使用 Ansible-vault 来保护传递给 playbooks 的数据。我们从加密数据的需求开始，讲解了 vault 的工作原理以及它使用的加密算法。接着，我们深入探讨了 Ansible-vault 工具及其基本操作，如创建加密文件、解密、重新加密等。你还学习了如何通过运行 Ansible-vault 对保存数据库凭证的 `vars` 文件进行加密。最后，我们为 Nginx 添加了 SSL 支持，并学习了如何使用 vault 安全存储私钥和证书，并通过模板复制它们。请注意，Ansible vault 提供了一种安全地向 Ansible 模块提供数据的方法。除了使用 vault 之外，建议采取额外的系统安全措施，这些内容不在本书的范围内。

在了解了 vault 后，在下一章中，我们将开始学习如何使用 Ansible 管理多个环境（如开发、预发布和生产环境）。这些环境通常映射到软件开发工作流中。
