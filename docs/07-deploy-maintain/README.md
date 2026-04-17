# Block 7 — Deploy, Configure and Maintain Systems

## Scheduling — at

`at` executes a command once at a specific future time.

```bash
dnf install -y at
systemctl enable --now atd

# Schedule a command
at now +5 minutes
# type command, then Ctrl+D to save

at 14:30              # today at 2:30 PM
at 14:30 tomorrow     # tomorrow at 2:30 PM
at noon               # at midday
at midnight           # at midnight

atq                   # list pending jobs
atrm 1                # remove job by number
```

---

## Scheduling — cron

`cron` executes tasks repeatedly on a schedule.

```bash
crontab -e            # edit current user's crontab
crontab -l            # list current user's jobs
crontab -r            # delete all jobs (including comments — use with care)
crontab -e -u student # edit another user's crontab (root only)
```

### Cron format

```
# min  hour  day  month  weekday  command
  *    *     *    *      *        command

*/5    *     *    *      *        echo "tick" >> /tmp/cron.log   # every 5 min
30     2     *    *      *        /usr/local/bin/backup.sh       # daily at 2:30 AM
0      8     *    *      1        /usr/local/bin/report.sh       # every Monday 8 AM
0      0     1    *      *        /usr/local/bin/monthly.sh      # 1st of each month
```

### /etc/crontab — system-wide cron

Includes a `user` field — each task runs as the specified user:

```
# min  hour  day  month  weekday  USER     command
0      2     *    *      *        root     /usr/local/bin/backup.sh
0      8     *    *      1        student  /home/student/report.sh
```

---

## Scheduling — systemd timers

Modern alternative to cron. Timers trigger `.service` units and integrate with `journalctl`.

```bash
# /etc/systemd/system/hello.service
[Unit]
Description=Hello timer job

[Service]
Type=oneshot
ExecStart=/bin/bash -c 'echo "hello from timer $(date)" >> /tmp/timer.log'
```

```bash
# /etc/systemd/system/hello.timer
[Unit]
Description=Run hello every minute

[Timer]
OnBootSec=1min
OnUnitActiveSec=1min

[Install]
WantedBy=timers.target
```

```bash
systemctl daemon-reload
systemctl enable --now hello.timer     # enable the timer, not the service
systemctl list-timers                  # list all active timers
journalctl -u hello.service            # view timer job logs
```

---

## Time Service — chrony

chrony is the NTP client in RHEL 9. It synchronizes the system clock with time servers.

```bash
dnf install -y chrony
systemctl enable --now chronyd

chronyc tracking       # show sync status and reference server
chronyc sources -v     # list configured NTP servers and their state
chronyc makestep       # force immediate sync
```

### Configure a specific NTP server

```bash
vim /etc/chrony.conf
# add or replace pool/server lines:
# server time.cloudflare.com iburst

systemctl restart chronyd
chronyc sources -v
```

### Timezone

NTP syncs in UTC — local time depends on timezone configuration.

```bash
timedatectl                              # show current date, time and timezone
timedatectl list-timezones | grep Chile  # find your timezone
timedatectl set-timezone America/Santiago
```

---

## Bootloader — grub2

Never edit `/boot/grub2/grub.cfg` directly — it gets overwritten. Always edit `/etc/default/grub` and regenerate.

```bash
vim /etc/default/grub
```

| Parameter | Description |
|-----------|-------------|
| `GRUB_TIMEOUT` | Seconds to show menu before booting |
| `GRUB_DEFAULT` | Default boot entry |
| `GRUB_CMDLINE_LINUX` | Kernel parameters passed at every boot |

```bash
# After editing /etc/default/grub always regenerate
grub2-mkconfig -o /boot/grub2/grub.cfg

# Verify
grep timeout /boot/grub2/grub.cfg
```

### GRUB password protection

```bash
grub2-mkpasswd-pbkdf2     # generate password hash

# Add to /etc/grub.d/40_custom:
set superusers="root"
password_pbkdf2 root grub.pbkdf2.sha512.10000.HASH_HERE

grub2-mkconfig -o /boot/grub2/grub.cfg
```

---

## Services at boot

```bash
systemctl enable --now sshd     # enable autostart and start immediately
systemctl disable sshd          # disable autostart
systemctl is-enabled sshd       # check if service starts at boot
systemctl list-unit-files --type=service | grep enabled   # list all enabled services
```

---

## Exam tips

- `crontab -r` deletes everything including comments — prefer `crontab -e` to remove specific jobs
- In `/etc/crontab` always specify the user field — it does not default to root
- Always run `grub2-mkconfig` after editing `/etc/default/grub` — changes are not applied otherwise
- `chronyc makestep` forces immediate sync without waiting for gradual adjustment
- Set timezone with `timedatectl set-timezone` — NTP alone does not fix wrong local time
