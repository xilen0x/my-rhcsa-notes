# Block 5 — Local Storage

## GPT Partitions with gdisk

`gdisk` is the partitioning tool for GPT disks. Use it instead of `fdisk` on RHEL 9.

```bash
gdisk /dev/vdb
# n → partition number → first sector (Enter=default) → size (+500M) → type code → w → y
```

### Common partition type codes

| Code | Type |
|------|------|
| `8300` | Linux filesystem (default) |
| `8e00` | Linux LVM |
| `8200` | Linux swap |

```bash
lsblk          # verify partitions after creation
partprobe      # force kernel to re-read partition table if needed
```

---

## LVM — Logical Volume Manager

LVM adds a flexible abstraction layer between physical disks and filesystems.
It allows resizing, extending, and reorganizing storage without downtime.

### Creation order (bottom to top)

| Step | Layer | Command |
|------|-------|---------|
| 1 | Physical Drives | — |
| 2 | Partitions | `gdisk` |
| 3 | Physical Volumes | `pvcreate` |
| 4 | Volume Group | `vgcreate` |
| 5 | Logical Volumes | `lvcreate` |
| 6 | File Systems | `mkfs` + `mount` |

### Commands

```bash
# Physical Volumes
pvcreate /dev/vdb1 /dev/vdc1
pvs
pvdisplay

# Volume Group
vgcreate vg_data /dev/vdb1 /dev/vdc1
vgs
vgdisplay

# Logical Volumes
lvcreate -L 500M -n lv_data vg_data       # fixed size
lvcreate -l 100%FREE -n lv_backup vg_data  # all remaining space
lvs
lvdisplay

# Format
mkfs.xfs  /dev/vg_data/lv_data
mkfs.ext4 /dev/vg_data/lv_backup

# Mount
mkdir -p /data /backup
mount /dev/vg_data/lv_data   /data
mount /dev/vg_data/lv_backup /backup
```

---

## Persistent Mounts — /etc/fstab

After mounting, always add entries to `/etc/fstab` to survive reboots.

```bash
# Get UUIDs
blkid /dev/vg_data/lv_data
blkid /dev/vg_data/lv_backup

# /etc/fstab entries
UUID=xxxx  /data    xfs   defaults  0 0
UUID=xxxx  /backup  ext4  defaults  0 0

# Verify without rebooting
mount -a
systemctl daemon-reload
```

### fstab field reference

| Field | Description |
|-------|-------------|
| device | UUID=xxx or /path/to/file |
| mount point | /data, /backup, none (swap) |
| filesystem | xfs, ext4, swap |
| options | defaults |
| dump | 0 (always) |
| fsck | 0 for LVM/xfs, 1 for /, 2 for ext4 partitions |

**Always use UUID, not device names** — device names like `/dev/vdb1` can change after reboot, UUIDs never do.

---

## Filesystems

```bash
mkfs.xfs  /dev/vg_data/lv_data     # XFS — default in RHEL 9
mkfs.ext4 /dev/vg_data/lv_backup   # ext4
mkfs.vfat /dev/vdb1                 # VFAT (FAT32)

mount /dev/vg_data/lv_data /data    # mount
umount /data                        # unmount
df -hT                              # verify mounted filesystems
```

---

## Swap

Swap is disk space used as RAM extension when physical memory is exhausted.
It is slower than RAM but prevents the kernel from killing processes OOM.

### Option 1 — Swap on partition

```bash
mkswap /dev/vdb1          # format partition as swap
swapon /dev/vdb1          # activate immediately
swapon --show             # verify

# /etc/fstab — use UUID for partitions
UUID=xxxx  none  swap  defaults  0 0
```

### Option 2 — Swap on file (when no free partitions available)

```bash
dd if=/dev/zero of=/swapfile bs=1M count=512
chmod 600 /swapfile       # required — mkswap refuses insecure permissions
mkswap /swapfile
swapon /swapfile
swapon --show

# /etc/fstab — use path for files (no UUID)
/swapfile  none  swap  defaults  0 0
```

### Disable swap

```bash
swapoff /dev/vdb1
swapoff /swapfile
```

---

## Lab — LVM Practice Exercise

**Goal:** Create LVM storage using two new disks with two logical volumes and persistent mounts.

| LV | Size | Filesystem | Mount |
|----|------|------------|-------|
| `lv_data` | 500 MiB | xfs | `/data` |
| `lv_backup` | remaining | ext4 | `/backup` |

```bash
# Two disks: /dev/vdb (vdb1) and /dev/vdc (vdc1)
pvcreate /dev/vdb1 /dev/vdc1
vgcreate vg_data /dev/vdb1 /dev/vdc1
lvcreate -L 500M -n lv_data vg_data
lvcreate -l 100%FREE -n lv_backup vg_data
mkfs.xfs  /dev/vg_data/lv_data
mkfs.ext4 /dev/vg_data/lv_backup
mkdir -p /data /backup
mount /dev/vg_data/lv_data   /data
mount /dev/vg_data/lv_backup /backup
# Add UUIDs to /etc/fstab, then:
mount -a && systemctl daemon-reload
```

**Verification:**

```bash
vgs        # vg_data: 2 PVs, 2 LVs
lvs        # lv_data 500MiB, lv_backup ~2.5GiB
df -hT     # both mounted and accessible
lsblk -f   # full tree with UUIDs and mount points
```
# LVM Notes
[Ver LVM completo](lvm.md)