# Daily driver notes

Add a few packages, run `switch`, and confirm the system still boots before adding more. Generations are there if a switch needs to be undone.

A starter list is in `example/configuration-daily.toml`.

## Small packages

```
app-misc/fastfetch
app-misc/tmux
sys-apps/man-pages
app-archives/unzip
```

```bash
genix-rebuild switch --dry-run
genix-rebuild switch
```

## Packages with dependencies

Add one package at a time. When a package should be built with its dependencies:

```toml
{ name = "net-wireless/wpa_supplicant", nodeps = false }
```

On an LFS install with `lfs = true`, Portage may still be missing pieces those dependencies expect.

## Config only

```bash
genix-rebuild switch --no-emerge
```

Rewrites the portage files, os-release, and services, and saves a generation. Does not run emerge.

## Desktop

A full desktop on a small LFS base takes a long time. Add one package at a time, and add the browser last.

When Portage can see the whole system:

```toml
[system.portage]
lfs = false
```

## Copy the tree to another machine

```bash
tar -C ~/Projects/arch-hyprland -cf - \
  --exclude='genix/iso/work' --exclude='genix/build' \
  genix | ssh user@LAPTOP 'rm -rf /tmp/genix && tar -xf - -C /tmp'
```

On that machine, as root: `cd /tmp/genix && ./install.sh`
