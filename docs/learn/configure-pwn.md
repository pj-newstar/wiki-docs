---
titleTemplate: ":title | 快速入门 - NewStar CTF"
---

# Pwn 环境配置

你需要具备 Ubuntu 或其它 Linux 操作系统，可参见 [获取 Linux 发行版](/learn/get-linux)。

## 获取 IDE

::: tip
一般只获取一个 IDE 即可，PyCharm 是专为 Python 开发的 IDE，VSCode 是通用的 IDE，可用于多种语言。
:::

点击左下角菜单按钮，搜索 Software，进入 Ubuntu 软件商店。

若需安装 VSCode，则搜索「VSCode」下载即可。若需安装 PyCharm，搜索「PyCharm」，下载 Community 版本。

## 配置 root 密码

root 用户是 Linux 系统的超级管理员，可以执行任何操作。默认情况下，你以你的普通用户身份使用操作系统。

打开终端，你会发现你的用户并不是 root 用户。若要以 root 用户身份执行命令，需要使用 `sudo` 命令。例如：

```bash
sudo apt-get update
```

随后输入你当前账户的密码，即可以 root 身份执行命令。

修改 root 密码也是需要 root 身份的，因此我们用 `sudo` 命令执行 `passwd` 命令：

```bash
sudo passwd root
```

随后输入你希望设置的 root 密码，即可完成设置。

使用 `su` 命令可以切换到 root 用户：

```bash
su
```

随后输入你设置的 root 密码并回车即可。

退出当前的会话，可以使用 `exit` 命令，或使用快捷键<kbd>Ctrl</kbd><kbd>D</kbd>.

## 安装必备软件包

国内下载软件包速度较慢，可以使用镜像源，参见 [更换软件源](/learn/change-software-source)。

> [!TIP]
> Kali Linux 默认自带这些工具，可跳过该步骤。

打开终端，输入以下命令：

```bash
sudo apt-get install python3-pip git gcc vim nano
```

- `pip` 是 Python 的包管理工具，在学习 Python 的过程中必然会被介绍到。
- `git` 是强大的版本管理工具，可用于下载互联网中的开源库。
- `gcc` 是 GNU 的编译套件，Linux 下 编译 C 的必备，`gdb` 是 GNU 中的调试工具。
- `vim` 是 Linux 下强大的命令行文本编辑器，可自行学习它的基本用法。
- `nano` 也是 Linux 下的命令行文本编辑器，比起 `vim` 的操作更便捷。

加快 pip 软件包下载速度，可参考 [PyPI 软件仓库](https://mirrors.help/pypi/)。

## 安装 Pwn tools

Pwn tools 是一个专为 CTF 设计的工具集，包含了许多用于 Pwn 题目的工具。

使用 pip 安装：

```bash
pip3 install pwntools
```

## 安装 Pwndbg

Pwndbg 是一个专为 GDB（GNU 调试器）设计的插件，提供了更直观和实用的调试界面，帮助底层软件开发人员、硬件黑客、逆向工程师和开发人员分析程序漏洞。

Pwndbg 是 Pwn 中常用的动态调试工具，可以观察寄存器，汇编代码，栈空间等，功能强大。

访问 [Pwndbg 的 GitHub 仓库 Release 页面](https://github.com/pwndbg/pwndbg/releases)，下载对应 Linux，此处以 `pwndbg-lldb_2026.02.18_amd64.deb` 为例。

打开命令行，安装该 deb 包，其中 `install` 后面为 deb 包所在的路径：

```bash
sudo apt install ./pwndbg-lldb_2026.02.18_amd64.deb
```

安装插件依赖：

```bash
sudo pip3 install keystone-engine ropper keystone-engine
```

## 安装 ROPgadget

ROPgadget 是一个用于搜索二进制文件中可用 ROP（返回导向编程） 链的工具，帮助漏洞利用时构建rop链以执行恶意代码。

安装命令：

```bash
sudo -H python3 -m pip install ROPgadget
```

安装完后，运行 `ROPgadget --version` 查看是否安装成功。

## 安装 one_gadget

在 CTF 的 Pwn 题目中，通常需要获取远程 shell，`one_gadget` 会自动查找 ELF 文件中能够调用 `execve("/bin/sh", NULL, NULL)` 的位置，帮助通过漏洞利用实现远程代码执行（RCE）。

安装命令：

```bash
sudo apt install -y ruby ruby-dev
sudo gem install one_gadget
```

安装完后，运行 `one_gadget --version` 查看是否安装成功。

## 安装 seccomp-tools

一些题可能会遇到沙箱，seccomp 可以很好的帮助我们识别系统调用黑名单和白名单

安装命令：

```bash
sudo gem install seccomp-tools
```

安装完后，运行 `seccomp-tools --version` 查看是否安装成功。

## 安装 patchelf

`patchelf` 是一个用于修改 ELF 文件属性的工具，支持更改运行时依赖、RPATH 和入口点等，以便灵活调整二进制文件的行为。

等学到 ret2libc 之后，会遇到题目所给附件的编译环境以及 libc 和本地环境不匹配的情况。但是如果按照 libc 版本重装多个虚拟机的话实在是太费劲了，patchelf 就可以直接修改题目所需的 libc 和 ld.

安装命令：

```bash
sudo apt install patchelf
```

安装完后，运行 `patchelf --version` 查看是否安装成功。

## 安装 LibcSearch

LibcSearcher 是一个用于查询 libc 版本的工具，可以根据泄露的数据查询 libc 版本，以便进行 ret2libc 攻击。

::: tip
如果学会了 patchelf，LibcSearcher 就不是必要的了。且一般情况下，对于没有给 libc 附件，却需要 libc 版本的题，面对远程泄露的 data，也可以在 [libc-database](https://libc.rip/) 上查询。
:::

安装命令：

```shell
pip3 install LibcSearcher
```
