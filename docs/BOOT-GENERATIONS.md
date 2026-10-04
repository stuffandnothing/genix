# Boot menu generations

Boot an older generation from GRUB when the running system no longer starts.
`genix-rebuild rollback N` is for when you still have a shell.

Requires:

- a btrfs root with subvolumes
- GRUB
- generations already stored under `/var/lib/genix/generations/`

Layout:

```
@              the subvolume you normally boot
@genix-1
@genix-15
```

On `switch`, with boot enabled: emerge, save the generation, snapshot it to `@genix-N`, and refresh GRUB.
After reboot, choose that generation in the menu. The kernel gets `rootflags=subvol=@genix-N`, and that snapshot's `/etc/fstab` is updated to the same subvolume so OpenRC does not remount `@` over it.

```toml
[system.boot]
enabled = true
backend = "grub"
subvol_prefix = "@genix-"
default_subvol = "@"
```

## Commands

```bash
genix-rebuild boot status
genix-rebuild boot sync
```

`prune` and `delete` also remove the `@genix-N` subvolumes when boot is enabled.

`boot sync` rewrites existing snapshot fstabs and the GRUB snippet. Use it once after upgrading `genix-rebuild` on a machine that already has generations.

## Coming from ext4

This is a one-time migration, done from a live USB. Back up `/etc/genix`, `/var/lib/genix`, portage, and `/boot`, create a btrfs `@` subvolume, copy the system across, fix fstab, reinstall GRUB, then enable boot and run `boot sync`.

Script: `scripts/migrate-ext4-to-btrfs.sh`  
Notes: `docs/LAPTOP-BTRFS-MIGRATE.md`
