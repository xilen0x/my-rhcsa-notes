# Block 10 — Manage Security

## SELinux

SELinux (Security Enhanced Linux) adds a mandatory access control (MAC) layer on top of standard Unix permissions. Every file, process, and port has a security label called a **context**. Policies define what contexts can interact with each other.

### Modes of operation

| Mode | Description |
|------|-------------|
| `enforcing` | Active and blocking — default in RHEL 9 |
| `permissive` | Active but only logs, does not block — useful for troubleshooting |
| `disabled` | Completely off — requires reboot to change |

```bash
getenforce                  # show current mode
sestatus                    # show detailed status

setenforce 0                # switch to permissive (temporary, lost on reboot)
setenforce 1                # switch to enforcing (temporary)
```

### Persistent mode change

```bash
vim /etc/selinux/config
# SELINUX=enforcing
# SELINUX=permissive
# SELINUX=disabled
```

---

## SELinux Contexts

Context format: `user:role:type:level` — the **type** field is what policies use to allow or deny access.

```bash
ls -Z /etc/ssh/sshd_config      # view file context
ps auxZ | grep sshd             # view process context
semanage port -l | grep http    # view port labels
```

### Fix wrong context (most common exam scenario)

Moving a file preserves its original context — SELinux may block access in the new location.

```bash
# Restore default context for a file
restorecon -v /var/www/html/index.html

# Restore recursively
restorecon -Rv /var/www/html/
```

---

## SELinux Port Labels

SELinux blocks services from using non-standard ports even if firewalld allows them.

```bash
dnf install -y policycoreutils-python-utils

semanage port -l | grep http              # list current HTTP port labels
semanage port -a -t http_port_t -p tcp 8181   # add port 8181 as HTTP
semanage port -d -t http_port_t -p tcp 8181   # remove port label
```

---

## SELinux Booleans

Booleans are on/off switches that enable or disable specific behaviors in SELinux policy without writing new policies.

```bash
getsebool -a                              # list all booleans
getsebool -a | grep httpd                 # filter by service
getsebool httpd_can_network_connect       # check specific boolean

setsebool httpd_can_network_connect on    # enable temporarily
setsebool -P httpd_can_network_connect on # enable permanently (-P persists after reboot)
```

### Common booleans

| Boolean | Description |
|---------|-------------|
| `httpd_can_network_connect` | allow httpd to connect to the network |
| `httpd_enable_homedirs` | allow httpd to serve files from home directories |
| `ftp_home_dir` | allow FTP to read/write home directories |
| `samba_enable_home_dirs` | allow Samba to share home directories |

---

## SSH Key-based Authentication

More secure than passwords. Uses a public/private key pair — private key stays on the client, public key goes to the server.

```bash
# Generate key pair on the client
ssh-keygen -t rsa -b 4096

# Copy public key to server
ssh-copy-id student@192.168.122.44

# Verify passwordless login
ssh student@192.168.122.44
```

### Manual setup (without ssh-copy-id)

```bash
# On the server
mkdir -p ~/.ssh
chmod 700 ~/.ssh
touch ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys

# Append client's public key
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

### Required permissions — SSH rejects auth if wrong

| File | Permission |
|------|------------|
| `~/.ssh/` | `700` |
| `~/.ssh/authorized_keys` | `600` |
| `~/.ssh/id_rsa` (private) | `600` |
| `~/.ssh/id_rsa.pub` (public) | `644` |

---

## Default File Permissions — umask

`umask` defines permissions that are **subtracted** from the maximum when creating new files and directories.

| | Maximum | umask 022 | umask 027 |
|--|---------|-----------|-----------|
| Files | 666 (rw-rw-rw-) | 644 (rw-r--r--) | 640 (rw-r-----) |
| Directories | 777 (rwxrwxrwx) | 755 (rwxr-xr-x) | 750 (rwxr-x---) |

```bash
umask              # show current umask
umask 027          # change temporarily (current session only)

# Verify
touch /tmp/testfile
mkdir /tmp/testdir
ls -l /tmp/testfile /tmp/testdir
```

### Persistent umask

```bash
# For all users
echo "umask 027" >> /etc/profile.d/umask.sh

# For a specific user
echo "umask 027" >> /home/john/.bashrc
```

---

## Exam tips

- `setenforce` is temporary — always edit `/etc/selinux/config` for persistent changes
- `restorecon` is the fix for wrong SELinux context — use it whenever you move files to a new location
- `setsebool -P` is required for persistent boolean changes — without `-P` it resets on reboot
- SSH auth fails silently if `~/.ssh` permissions are wrong — always verify with `ls -la ~/.ssh`
- `umask` subtracts from maximum — it does not set permissions directly