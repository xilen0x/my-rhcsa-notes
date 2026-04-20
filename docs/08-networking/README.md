# Block 8 — Manage Basic Networking

## nmcli — NetworkManager CLI

```bash
nmcli connection show                  # list all network connections
nmcli device status                    # list devices and their state
nmcli connection show enp1s0           # show details of a specific connection
ip addr show                           # show current IP addresses
ip route show                          # show routing table
```

### Configure a static IP

```bash
nmcli connection modify enp1s0 \
  ipv4.addresses 192.168.122.44/24 \
  ipv4.gateway 192.168.122.1 \
  ipv4.dns 8.8.8.8 \
  ipv4.method manual

nmcli connection down enp1s0
nmcli connection up enp1s0

# Verify — should not say 'dynamic' anymore
ip addr show enp1s0
nmcli connection show enp1s0 | grep ipv4
```

### Configure IPv6

```bash
nmcli connection modify enp1s0 \
  ipv6.addresses 2001:db8::1/64 \
  ipv6.gateway 2001:db8::ff \
  ipv6.method manual

nmcli connection up enp1s0
```

---

## Hostname and Name Resolution

```bash
hostnamectl                                        # show current hostname and system info
hostnamectl set-hostname rhcsa-lab.example.com     # set hostname (persists after reboot)
hostname                                           # quick check
```

### /etc/hosts — static DNS entries

```bash
echo "192.168.122.1  gateway.lab.com" >> /etc/hosts

getent hosts gateway.lab.com    # verify resolution
ping -c 1 gateway.lab.com
```

### DNS configuration

```bash
nmcli connection show enp1s0 | grep dns            # view current DNS
nmcli connection modify enp1s0 ipv4.dns "8.8.8.8 1.1.1.1"
nmcli connection up enp1s0

cat /etc/resolv.conf                               # verify applied DNS
```

---

## firewalld

firewalld uses zones to define the trust level of network connections.
`public` is the default zone for most interfaces.

```bash
systemctl status firewalld --no-pager
firewall-cmd --get-default-zone        # show default zone
firewall-cmd --get-zones               # list all available zones
firewall-cmd --list-all                # show all rules in active zone
firewall-cmd --zone=public --list-all  # show rules for a specific zone
```

### Managing services and ports

```bash
# Allow a service by name
firewall-cmd --permanent --add-service=http
firewall-cmd --permanent --add-service=https
firewall-cmd --permanent --add-service=ssh

# Allow a specific port
firewall-cmd --permanent --add-port=8080/tcp

# Remove a service or port
firewall-cmd --permanent --remove-service=http
firewall-cmd --permanent --remove-port=8080/tcp

# Apply permanent changes (required after --permanent)
firewall-cmd --reload

# Verify
firewall-cmd --list-services
firewall-cmd --list-ports
```

### Temporary vs permanent rules

| | Temporary | Permanent |
|--|-----------|-----------|
| Command | without `--permanent` | with `--permanent` |
| Survives reload | No | Yes |
| Survives reboot | No | Yes |
| Active immediately | Yes | Only after `--reload` |

### Common exam pattern

```bash
# Correct way — permanent + reload
firewall-cmd --permanent --add-service=http
firewall-cmd --reload
firewall-cmd --list-services    # verify
```

---

## Exam tips

- Always use `--permanent` + `--reload` — without `--permanent` rules are lost on reboot
- Use service names when available (`http`, `https`, `ssh`) — easier than remembering port numbers
- `nmcli connection up` must be run after modifying a connection to apply changes
- `/etc/hosts` entries take precedence over DNS — useful for quick local name resolution
- After changing hostname with `hostnamectl`, verify with both `hostname` and `hostnamectl`