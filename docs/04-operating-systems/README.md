# Block 4 — Operate Running Systems

## systemd Targets

Targets are the modern equivalent of runlevels.

| Target | Runlevel | Description |
|--------|----------|-------------|
| `poweroff.target` | 0 | Shutdown |
| `rescue.target` | 1 | Single user mode |
| `multi-user.target` | 3 | Multiuser without GUI |
| `graphical.target` | 5 | Multiuser with GUI |
| `reboot.target` | 6 | Reboot |

```bash
systemctl get-default                        # show current default target
systemctl list-units --type=target           # list all available targets
systemctl isolate multi-user.target          # switch target without rebooting
systemctl set-default multi-user.target      # set default target (persists after reboot)
systemctl poweroff                           # shut down the system
systemctl reboot                             # reboot the system
```

---

## Interrupt Boot to Reset Root Password

1. Reboot the system
2. In GRUB menu press `e` to edit the boot entry
3. Find the line starting with `linux` containing `ro`
4. Replace `ro` with `rw init=/sysroot/bin/sh`
5. Press `Ctrl+X` to boot
6. Once inside run:

```bash
chroot /sysroot
passwd root
touch /.autorelabel    # required — without this SELinux blocks login after reboot
exit
reboot -f
```

---

## GRUB Password Protection

```bash
# Generate password hash
grub2-mkpasswd-pbkdf2

# Add to /etc/grub.d/40_custom
set superusers="root"
password_pbkdf2 root grub.pbkdf2.sha512.10000.HASH_HERE

# Regenerate GRUB config
grub2-mkconfig -o /boot/grub2/grub.cfg
```

---

## Processes

```bash
top                          # real-time process monitor (k=kill, q=quit)
htop                         # improved top (install from EPEL: dnf install epel-release)
ps aux                       # snapshot of all running processes
ps aux | grep httpd          # filter processes by name
ps aux --sort=-%cpu | head -5  # top 5 CPU consuming processes
ps aux --sort=-%mem | head -5  # top 5 memory consuming processes
kill 1234                    # send SIGTERM to PID (graceful shutdown)
kill -9 1234                 # send SIGKILL to PID (forced termination)
killall httpd                # kill all processes by name
pidof httpd                  # get PID of a process by name
```

### Process Scheduling (nice/renice)

Priority range: -20 (highest) to 19 (lowest). Only root can set negative values.

```bash
nice -n 10 tar -czf /tmp/backup.tar.gz /etc   # start process with lower priority
renice -n 5 -p 1234                            # change priority of running process
```

---

## Tuning Profiles

```bash
dnf install -y tuned
systemctl enable --now tuned

tuned-adm active                            # show current active profile
tuned-adm list                              # list all available profiles
tuned-adm profile throughput-performance    # switch to a specific profile
tuned-adm recommend                         # show recommended profile for this system
tuned-adm off                               # disable tuning temporarily
```

| Profile | Use case |
|---------|----------|
| `balanced` | Balance between performance and power saving |
| `throughput-performance` | Maximum throughput for servers |
| `latency-performance` | Low latency applications |
| `virtual-guest` | Optimized for VMs |
| `powersave` | Energy saving |

---

## Logs and Journals

```bash
journalctl                          # all logs since system start
journalctl -f                       # follow logs in real time
journalctl -u sshd                  # logs for a specific service
journalctl -b                       # logs from current boot only
journalctl -b -1                    # logs from previous boot
journalctl -p err                   # filter by priority (emerg,alert,crit,err,warning,notice,info,debug)
journalctl --since "30 min ago"     # logs from last 30 minutes
journalctl --since "2024-01-01" --until "2024-01-02"   # logs in time range
```

### Persistent Journal

By default journals are lost on reboot. To enable persistence:

```bash
mkdir -p /var/log/journal
systemd-tmpfiles --create --prefix /var/log/journal
systemctl restart systemd-journald
```

---

## Services

```bash
systemctl status sshd          # show service status
systemctl start sshd           # start service
systemctl stop sshd            # stop service
systemctl restart sshd         # restart service
systemctl reload sshd          # reload config without restarting
systemctl enable sshd          # enable autostart at boot
systemctl disable sshd         # disable autostart
systemctl enable --now sshd    # enable and start in one command
```

---

## Secure File Transfer

```bash
# scp — secure copy (destination directory must exist)
scp file.txt student@192.168.122.44:/tmp/
scp student@192.168.122.44:/tmp/file.txt /tmp/
scp -r /tmp/dir student@192.168.122.44:/tmp/

# rsync — efficient sync, only transfers changed files
rsync -av ~/synctest/ student@192.168.122.44:/tmp/synctest/
```

**Key difference:** `rsync path/` (with trailing slash) copies the **contents**. `rsync path` (without slash) copies the **directory itself**.
