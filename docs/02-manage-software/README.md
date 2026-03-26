# Block 2 — Manage Software

## dnf — Package Manager

```bash
dnf search httpd               # search for a package by name or description
dnf info httpd                 # show details before installing
dnf install -y httpd           # install a package (-y skips confirmation)
dnf remove -y httpd            # uninstall a package
dnf update -y                  # update all installed packages
dnf list installed             # list all installed packages
dnf provides /var/www/html     # find which package owns a file or provides a command
dnf history                    # show history of all dnf operations
dnf history undo last          # revert the last dnf operation
dnf clean all                  # clear local repo metadata cache
```

---

## Configuring RPM Repositories

Repository files live in `/etc/yum.repos.d/` with `.repo` extension.

```ini
[myrepo]
name=My Custom Repository
baseurl=http://repo.example.com/almalinux/9/BaseOS/x86_64/
enabled=1
gpgcheck=0
```

| Field | Description |
|-------|-------------|
| `[id]` | Unique repo identifier |
| `baseurl` | URL or local path (`file:///mnt/repo`) |
| `enabled` | 1=active, 0=inactive |
| `gpgcheck` | 1=verify GPG signature, 0=skip |

```bash
dnf repolist                          # list active repos
dnf repolist all                      # list all repos including disabled
dnf install pkg --disablerepo=myrepo  # ignore a repo for this operation
dnf install pkg --enablerepo=myrepo   # force use of a disabled repo
```

**Exam tip:** In the exam you will be given a URL. Create the `.repo` file manually with the correct `baseurl`, `enabled=1`, and `gpgcheck=0`.

---

## rpm — Low-level Package Manager

Unlike `dnf`, `rpm` does not resolve dependencies automatically.

```bash
rpm -ivh package.rpm          # install a local .rpm file (verbose + progress)
rpm -e package-name           # uninstall a package by name (no .rpm extension)
rpm -qa                       # list all installed RPM packages
rpm -qf /etc/ssh/sshd_config  # find which package owns a specific file
rpm -qi openssh-server        # show detailed info of an installed package
rpm -ql openssh-server        # list all files installed by a package
```

---

## Flatpak

```bash
dnf install -y flatpak        # install flatpak (not included by default in RHEL)

# Add Flathub remote repository
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo

flatpak remotes               # list configured remote repositories
flatpak search firefox        # search for an application
flatpak install flathub org.mozilla.firefox   # install an application
flatpak list                  # list installed Flatpak applications
flatpak uninstall org.mozilla.firefox         # uninstall an application
```
