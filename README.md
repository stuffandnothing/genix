# Genix

Gentoo with one configuration file, and a way to roll a change back.

Site: https://genixos.org/  
Repo: https://github.com/zubbledew6/genix

```
configuration.toml  →  genix-rebuild switch  →  generation N
```

## What it is

- A live USB installer. It downloads a Gentoo stage3 and installs OpenRC on btrfs `@`. It does not copy the live system onto the disk.
- After install, edit `/etc/genix/configuration.toml` and run `genix-rebuild switch`.
- Generations can be listed, rolled back, pruned, and deleted.
- With GRUB and btrfs, each generation can also be a boot-menu entry (`@genix-N`).
- Wi-Fi profiles from the live USB are copied across (iwd).

The release install is OpenRC. `genix-rebuild` can still enable services on systemd or sysv if those are already what the machine boots.

Saying no at the config-edit prompt installs this machine from Gentoo's binary packages. The saved config keeps `binary = false`, so later `genix-rebuild switch` builds from source. Saying yes means you edit the file and this install compiles.

## Install from the ISO

```bash
git clone https://github.com/zubbledew6/genix.git
cd genix
sudo ./iso/build.sh
```

The image is `iso/out/genix-live.iso`. Flash it and boot the machine you intend to install, not the one you use every day.

```bash
genix-install
```

On the live USB, connect first if you are on Wi-Fi:

```bash
iwctl station wlan0 connect "SSID"
```

Build notes, including Artix: [iso/README.md](iso/README.md)

## Already have Gentoo or LFS

`install.sh` sets up Genix on a system that already has Portage. It does not repartition the disk. Moving an ext4 root to btrfs is a separate step: [docs/LAPTOP-BTRFS-MIGRATE.md](docs/LAPTOP-BTRFS-MIGRATE.md).

```bash
scp -r genix user@host:/tmp/
ssh user@host
su -
cd /tmp/genix && ./install.sh
```

## Requirements

- Portage
- gcc (`make` builds the C tools)
- Python 3.11+ for `genix-install` only
- root for install, switch, and rollback
- btrfs and GRUB if you want generations in the boot menu

LFS bootstrap: `./bootstrap-portage.sh`

## Configuration

```toml
[system]
hostname = "genix"
use = ["-systemd", "elogind"]

[system.identity]
name = "Genix"
id = "genix"
id_like = "gentoo"
home_url = "https://genixos.org/"
logo = "genix"

[system.portage]
accept_keywords = "amd64"
makeopts = "-j8"
binary = false

[packages]
want = [
  "app-editors/vim",
  { name = "www-client/firefox", binary = true },
]
mask = ["net-misc/networkmanager"]

[packages.use]
"sys-kernel/installkernel" = ["dracut"]

[packages.accept_keywords]
"www-client/firefox-bin" = "~amd64"

[packages.license]
"www-client/google-chrome" = "google-chrome"

[packages.env]
"www-client/firefox" = "ccache.conf"

[system.portage.env_files]
"ccache.conf" = { CCACHE_DIR = "/var/cache/ccache" }

[services]
enable = ["iwd", "dhcpcd", "sshd"]
```

`packages.use`, `packages.accept_keywords`, `packages.license`, and `packages.env` each render straight to the matching `/etc/portage/package.*` dropin (one atom per key; a string or array of flags as the value). `system.portage.env_files` renders each table entry to `/etc/portage/env/<name>`. If the dropin directory already has hand-written files in it, genix only owns one file named `genix` inside it and leaves the rest alone.

Full example: `example/configuration.toml`

## Commands

```bash
genix-rebuild switch
genix-rebuild switch --dry-run
genix-rebuild rollback 3
genix-rebuild list
genix-rebuild doctor
genix-rebuild boot sync
genix-install
```

## Notes

- [Daily driver](docs/DAILY-DRIVER.md)
- [Boot generations](docs/BOOT-GENERATIONS.md)
- [Fastfetch logo](docs/FASTFETCH.md)

GPLv3. See [LICENSE](LICENSE).
