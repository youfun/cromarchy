# cromarchy

[中文说明](README.zh-CN.md)

One script, in-place **Ubuntu** or **Omarchy** installer that replaces ChromeOS on a Chromebook.

Based on [liyafe1997/crobuntu](https://github.com/liyafe1997/crobuntu). Omarchy support is unofficial.

**Only x86_64 devices are supported.**

No USB drive, no RW_LEGACY, no write-protect unlocking. Boot uses [Submarine](https://github.com/FyraLabs/submarine).

## Install

1. [Turn on Developer mode](https://www.chromium.org/chromium-os/developer-library/guides/device/developer-mode/) (Esc + Refresh + Power, then Ctrl + D).
2. Connect to Wi-Fi. You do **not** need to sign in with a Google account.
3. Press Ctrl+Alt+F2 (Refresh/Forward) for VT2. Login as `root` (usually no password).
4. Run:

```bash
cd /tmp
curl -LOf https://github.com/youfun/cromarchy/raw/main/cromarchy
bash cromarchy
```

5. Choose **Ubuntu** or **Omarchy**. For Ubuntu, pick a version and desktop.
6. Wait until it finishes, then press Refresh + Power to reboot.
7. To get ChromeOS back: Esc + Refresh + Power and recover. See [Recover your Chromebook](https://support.google.com/chromebook/answer/1080595).

Do **not** run this from crosh, Crostini, or Baguette.

## Defaults

| Target | User | Password |
| --- | --- | --- |
| Ubuntu | `ubuntu` | `ubuntu` |
| Omarchy | `omarchy` | `omarchy` |

Ubuntu may not include a browser (snap cannot run in the ChromeOS chroot). After boot: `snap install firefox`.

## Omarchy notes

This is **not** the official Omarchy ISO. Chromebook firmware (depthcharge) cannot boot Limine. cromarchy installs Arch + GRUB via Submarine.

After first login (as `omarchy`, with network up):

```bash
curl -fsSL https://omarchy.org/install | bash
```

When the installer says you need **Limine** or **btrfs**, choose **Proceed anyway**. Snapper/Limine features will not work. Hyprland and the rest of Omarchy can still install.

## Recovery partition

The script tries to keep MINIOS-B (ChromeOS cloud recovery). If your device supports it, Esc + Refresh + Power → Recover using internet can restore ChromeOS without a USB stick. If you delete that partition, you need USB recovery.

## No sound?

See [chromebook-linux-audio](https://github.com/WeirdTreeThing/chromebook-linux-audio).

## What the script does

It downloads `submarine-x86_64.zip` from this repo (originally from [FyraLabs/submarine](https://nightly.link/FyraLabs/submarine/workflows/build/main/submarine-x86_64.zip)), unzips it, and `dd`s `submarine.bin` onto the internal disk.

Submarine is a tiny Linux kernel that depthcharge can load. It finds `grub.cfg` and `kexec`s the real distro kernel.

A fake EFI partition exists so `grub-install` / `update-grub` (or Arch `grub-mkconfig`) can run. Kernel upgrades keep working through GRUB.

Rootfs is ext4 on partition 3. The installer builds a ~10GiB staging loop at the **end** of the disk, unpacks Ubuntu-base or Arch bootstrap there, chroots, then `dd`s the image onto the real rootfs. That avoids overwriting the live ChromeOS rootfs for most of the process.

You need about **21GiB+** internal disk (2×10GiB staging + EFI/MINIOS). 32GB machines usually work; 16GB models do not unless you shrink `STAGING_SIZE_MIB`. Large desktops (Kubuntu, full Omarchy packages after first boot) need extra free space.

After the final `dd`, ChromeOS is gone. Force reboot (Refresh + Power) into Submarine, then Ubuntu or Arch.

## Credits

- Original Ubuntu installer: [liyafe1997/crobuntu](https://github.com/liyafe1997/crobuntu)
- Bootloader: [FyraLabs/submarine](https://github.com/FyraLabs/submarine)
- Omarchy: [basecamp/omarchy](https://github.com/basecamp/omarchy) / [omarchy.org](https://omarchy.org)
