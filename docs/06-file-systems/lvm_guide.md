# LVM Operations Guide

## 1. Recreate Logical Volume (XFS → EXT4)

### Steps

1.  Unmount filesystem

```{=html}
<!-- -->
```
    umount /mnt/shared-data

2.  Remove Logical Volume

```{=html}
<!-- -->
```
    lvremove /dev/vg_group_a/lv_0001

3.  Create new smaller LV

```{=html}
<!-- -->
```
    lvcreate -L 5G -n lv_ext4_01 vg_group_a

4.  Create ext4 filesystem

```{=html}
<!-- -->
```
    mkfs.ext4 /dev/vg_group_a/lv_ext4_01

5.  Mount filesystem

```{=html}
<!-- -->
```
    mount /dev/vg_group_a/lv_ext4_01 /mnt/shared-data

6.  Get UUID

```{=html}
<!-- -->
```
    blkid /dev/vg_group_a/lv_ext4_01

7.  Add to /etc/fstab

```{=html}
<!-- -->
```
    UUID=<UUID> /mnt/shared-data ext4 defaults 0 0

8.  Reload systemd

```{=html}
<!-- -->
```
    systemctl daemon-reload

9.  Validate

```{=html}
<!-- -->
```
    df -hT
    mount | grep shared

------------------------------------------------------------------------

## 2. Real Reduction (EXT4)

### Steps

1.  Unmount filesystem

```{=html}
<!-- -->
```
    umount /mnt/shared-data

2.  Check filesystem

```{=html}
<!-- -->
```
    e2fsck -f /dev/vg_group_a/lv_ext4_01

3.  Reduce filesystem

```{=html}
<!-- -->
```
    resize2fs /dev/vg_group_a/lv_ext4_01 3G

4.  Reduce Logical Volume

```{=html}
<!-- -->
```
    lvreduce -L 3G /dev/vg_group_a/lv_ext4_01

5.  Mount again

```{=html}
<!-- -->
```
    mount /dev/vg_group_a/lv_ext4_01 /mnt/shared-data

6.  Validate

```{=html}
<!-- -->
```
    df -hT
    mount | grep shared

------------------------------------------------------------------------

## Key Rules

-   XFS cannot be reduced
-   Always reduce filesystem BEFORE LV
-   ext4 reduction must be done offline
-   Always validate after changes
