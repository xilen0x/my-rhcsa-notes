# Block 6 — Create and Configure File Systems

## Filesystems

```bash
mkfs.xfs  /dev/vg_lab/lv_data     # XFS — default in RHEL 9, can only grow
mkfs.ext4 /dev/vg_lab/lv_backup   # ext4 — can grow and shrink
mkfs.vfat /dev/vdb1               # VFAT (FAT32)

mount /dev/vg_lab/lv_data /data   # mount
umount /data                       # unmount
df -hT                             # verify mounted filesystems
```

---

# NFS — Network File System

NFS is the standard protocol for sharing directories between Linux systems over a network.

### Server side

```bash
dnf install -y nfs-utils

# Create shared directory
mkdir -p /srv/nfs/data
echo "test" > /srv/nfs/data/test.txt

# Define exports — who can access and with what permissions
echo "/srv/nfs/data  192.168.122.0/24(rw,sync,no_root_squash)" >> /etc/exports

systemctl enable --now nfs-server

exportfs -av    # apply changes without restarting
exportfs -v     # verify active exports
```

### /etc/exports options

| Option | Description |
|--------|-------------|
| `rw` | read and write |
| `ro` | read only |
| `sync` | write to disk before responding |
| `no_root_squash` | client root acts as server root |
| `root_squash` | client root maps to nobody (default) |

### Client side

```bash
# Check what the server is exporting before mounting
showmount -e 192.168.122.44

mkdir -p /mnt/nfs
mount -t nfs 192.168.122.44:/srv/nfs/data /mnt/nfs

# Persistent in /etc/fstab
echo "192.168.122.44:/srv/nfs/data  /mnt/nfs  nfs  defaults  0 0" >> /etc/fstab
mount -a
```
### NFS NOTES
[Ver CHECKLIST NFS](nfs_rhcsa_checklist.md)

---

# autofs — Automounter

autofs mounts filesystems automatically on access and unmounts them when idle.
Unlike static NFS mounts in fstab, autofs does not fail at boot if the server is unavailable.

### Configuration files

**Master map** — `/etc/auto.master.d/nfs.autofs`:
```
/mnt/auto  /etc/auto.nfs
```

**Mount map** — `/etc/auto.nfs`:
```
share  -rw,sync  192.168.122.44:/srv/nfs/data
dw     -rw,sync  192.168.122.44:/srv/nfs/dw
```

Format: `key  options  server:/export`

```bash
systemctl enable --now autofs

# Access triggers the mount — no need to create subdirectories manually
ls /mnt/auto/share    # autofs mounts it on first access

# Directories disappear from ls when idle — this is expected behavior
```

### Troubleshooting

```bash
# Check what the server exports before configuring auto.nfs
showmount -e <server_ip>

# autofs reads files, not LDAP/SSS
grep automount /etc/nsswitch.conf
# must be: automount: files

journalctl -u autofs --no-pager | tail -20
```
### AUTOFS NOTES
[Ver CHECKLIST AUTOFS](autofs_rhcsa_checklist.md)

---

# Extend Logical Volumes

One of LVM's key advantages — extend storage without unmounting or interrupting the service.

### Two-step process (always both steps)

```bash
# Step 1 — extend the LV (block level)
lvextend -L +200M /dev/vg_lab/lv_data      # add 200MB
lvextend -l +100%FREE /dev/vg_lab/lv_data  # use all free space in VG

# Step 2 — extend the filesystem to use the new space
xfs_growfs /data                           # XFS — use mount point, not device
resize2fs /dev/vg_lab/lv_backup           # ext4 — use device path

# Verify
lvs
df -hT /data
```

### Add a new disk to an existing VG (when VFree = 0)

```bash
# Partition the new disk
gdisk /dev/vdc
# n → 1 → Enter → Enter → 8e00 → w → y

# Initialize as PV and add to existing VG
pvcreate /dev/vdc1
vgextend vg_lab /dev/vdc1

# Verify new free space
vgs

# Now extend the LV
lvextend -l +100%FREE /dev/vg_lab/lv_data
xfs_growfs /data
```

### Key differences

| | XFS | ext4 |
|--|-----|------|
| Grow | `xfs_growfs /mountpoint` | `resize2fs /dev/vg/lv` |
| Shrink | Not supported | `resize2fs /dev/vg/lv SIZE` |
| Argument | mount point | device path |

# Others exercises
## Extend Logical Volumes NOTES
[Ver CHECKLIST Extend LV](extend_lvm.md)



## Recreate Logical Volume (XFS → EXT4)
[Recreate Logical Volume (XFS → EXT4)](lvm_guide.md)

## Real Reduction (EXT4)
[Real Reduction (EXT4)](lvm_guide.md)