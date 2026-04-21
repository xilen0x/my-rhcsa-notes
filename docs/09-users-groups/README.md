# Block 9 — Manage Users and Groups

## User Management

```bash
useradd john                                        # create user (no home by default)
useradd -m -s /bin/bash -c "John Doe" -u 1500 john # create with options
userdel -r john                                     # delete user and home directory

usermod -s /bin/bash john    # change shell
usermod -l newname john      # rename user
usermod -L john              # lock account
usermod -U john              # unlock account

id john                      # show UID, GID and groups
cat /etc/passwd | grep john  # view user entry
```

### useradd common options

| Option | Description |
|--------|-------------|
| `-m` | create home directory |
| `-s` | default shell |
| `-c` | comment / full name |
| `-u` | specific UID |
| `-g` | primary group |
| `-G` | secondary groups |
| `-e` | account expiration date (YYYY-MM-DD) |

---

## Passwords and Password Aging

```bash
passwd john          # set or change password
passwd -S john       # show password status
passwd -l john       # lock password
passwd -u john       # unlock password
passwd -e john       # force password change on next login
```

### chage — password aging policy

```bash
chage -l john        # list current password aging settings
chage -E 2026-12-31  john   # account expiration date
chage -M 90 john     # max days before password must be changed
chage -m 7 john      # min days between password changes
chage -W 14 john     # warning days before expiration
```

---

## Group Management

```bash
groupadd developers          # create a group
groupadd -g 2000 developers  # create with specific GID
groupmod -n devs developers  # rename group
groupmod -g 2001 developers  # change GID
groupdel developers          # delete group

getent group developers      # view group info
cat /etc/group | grep developers
```

### Group membership

```bash
usermod -aG developers john   # add john to developers (keeps existing groups)
usermod -g developers john    # change primary group
gpasswd -d john developers    # remove john from group

id john                       # verify groups
groups john
```

**Critical:** `-aG` appends. Without `-a`, `usermod -G developers john` replaces all secondary groups.

---

## Privileged Access — sudo

Always use `visudo` to edit sudoers — it validates syntax before saving.
Never edit `/etc/sudoers` directly.

```bash
visudo                 # edit /etc/sudoers safely
```

### Common sudoers entries

```bash
# Allow john to run any command as root
john  ALL=(ALL)  ALL

# Allow john to run any command without password
john  ALL=(ALL)  NOPASSWD: ALL

# Allow john to restart services only
john  ALL=(ALL)  /bin/systemctl restart *

# Allow all members of wheel group (default in RHEL 9)
%wheel  ALL=(ALL)  ALL
```

### Quickest way to grant sudo in RHEL 9

```bash
# wheel group has sudo access by default
usermod -aG wheel john

# Verify
su - john
sudo whoami    # should return root
```

---

## Exam tips

- Always use `userdel -r` to remove home directory along with the user
- Use `usermod -aG` (with `-a`) to add secondary groups — without `-a` you overwrite existing groups
- `visudo` is mandatory — direct edits to `/etc/sudoers` can break sudo system-wide
- `chage -l` is the quickest way to audit a user's password policy
- In RHEL 9, adding a user to `wheel` is the standard way to grant sudo access