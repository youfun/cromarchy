# cromarchy

[English](README.md)

一条脚本，在 Chromebook **内置硬盘上就地**把 ChromeOS 换成 **Ubuntu** 或 **Omarchy**（DHH 维护的 Arch + Hyprland）。

以 [liyafe1997/crobuntu](https://github.com/liyafe1997/crobuntu) 为基础。Omarchy 支持是**非官方**路径。

**目前只支持 x86_64。**

不需要 U 盘、不刷 RW_LEGACY、不解写保护。引导使用 [Submarine](https://github.com/FyraLabs/submarine)。

## 安装

1. [打开开发者模式](https://www.chromium.org/chromium-os/developer-library/guides/device/developer-mode/)（Esc + Refresh + Power，然后 Ctrl + D）。
2. 连上 Wi-Fi。**不必**登录 Google 账号。
3. 按 Ctrl+Alt+F2（Refresh/前进键）进入 VT2，用 `root` 登录（一般无密码）。
4. 执行：

```bash
cd /tmp
curl -LOf https://github.com/youfun/cromarchy/raw/main/cromarchy
bash cromarchy
```

5. 选择 **Ubuntu** 或 **Omarchy**。选 Ubuntu 时再选版本和桌面。
6. 等它跑完，按 Refresh + Power 强制重启。
7. 若要回到 ChromeOS：Esc + Refresh + Power 进入恢复。参见 [恢复 Chromebook](https://support.google.com/chromebook/answer/1080595)。

**不要**在 crosh、Crostini 或 Baguette 里运行本脚本。

## 默认账号

| 目标 | 用户名 | 密码 |
| --- | --- | --- |
| Ubuntu | `ubuntu` | `ubuntu` |
| Omarchy | `omarchy` | `omarchy` |

Ubuntu 可能没有浏览器（ChromeOS 的 chroot 里装不了 snap）。进系统后执行：`snap install firefox`。

## 关于 Omarchy

这**不是**官方 Omarchy ISO。Chromebook 固件（depthcharge）**不能**启动 Limine。cromarchy 通过 Submarine 安装 Arch + GRUB。

第一次以 `omarchy` 用户登录、网络可用后：

```bash
curl -fsSL https://omarchy.org/install | bash
```

安装器如果提示缺少 **Limine** 或 **btrfs**，选择 **Proceed anyway**（继续）。Snapper / Limine 相关功能在这台机器上不可用。Hyprland 和其余 Omarchy 软件包仍可安装。

请用 `omarchy` 用户运行安装器，不要用 root。

## 恢复分区

脚本会尽量保留 MINIOS-B（ChromeOS 云恢复）。支持该功能的机型可以用 Esc + Refresh + Power → 使用互联网恢复，不必做 U 盘。如果你删掉该分区，就必须用 U 盘恢复。

## 没有声音？

参见 [chromebook-linux-audio](https://github.com/WeirdTreeThing/chromebook-linux-audio)。

## 脚本在做什么

从本仓库下载 `submarine-x86_64.zip`（由本仓库 CI 从 [FyraLabs/submarine](https://github.com/FyraLabs/submarine) 指定 commit 构建），解压后把 `submarine.bin` `dd` 到内置盘。zip 的 sha256 钉在安装脚本里；上游 commit SHA 记在 GitHub Release 页。

Submarine 是 depthcharge 能加载的微型 Linux 内核。它会查找 `grub.cfg`，再用 `kexec` 启动真正的发行版内核。

会做一个「假 EFI」分区，好让 `grub-install` / `update-grub`（Arch 上是 `grub-mkconfig`）能跑。以后内核升级仍走 GRUB。

根分区是第 3 分区上的 ext4。安装过程在磁盘**尾部**做约 10GiB 的 staging loop，在那里解开 Ubuntu-base 或 Arch bootstrap，chroot 装包，最后再 `dd` 到真正的根分区。这样大部分时间不会覆盖正在运行的 ChromeOS 根文件系统。

内置盘大约需要 **21GiB 以上**（2×10GiB staging + EFI/MINIOS）。32GB 机型一般可以；16GB 默认不行，除非你改小 `STAGING_SIZE_MIB`。大桌面（Kubuntu，或进系统后再装完整 Omarchy）还需要更多空间。

最后一次 `dd` 完成后，ChromeOS 就没了。Refresh + Power 强制重启，进入 Submarine，然后是 Ubuntu 或 Arch。

## 致谢

- 原始 Ubuntu 安装器：[liyafe1997/crobuntu](https://github.com/liyafe1997/crobuntu)
- 引导：[FyraLabs/submarine](https://github.com/FyraLabs/submarine)
- Omarchy：[basecamp/omarchy](https://github.com/basecamp/omarchy) / [omarchy.org](https://omarchy.org)
