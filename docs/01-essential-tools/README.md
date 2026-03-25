# Block 1 — Essential Tools

## I/O Redirection

| Operator | Description |
|----------|-------------|
| `>`      | Redirect stdout to file (overwrite) |
| `>>`     | Redirect stdout to file (append) |
| `2>`     | Redirect stderr to file |
| `2>&1`   | Redirect stderr to same destination as stdout |
| `&>`     | Redirect both stdout and stderr to file (bash shorthand) |
| `2>/dev/null` | Discard stderr |
| `\|`     | Pipe stdout of one command to stdin of next |

**Key rule:** After `>`, the order matters. `2>&1` must come after `>` to work correctly.

```bash
# Correct
ls /noexiste > /tmp/out.txt 2>&1

# Wrong — stderr is not redirected to the file
ls /noexiste 2>&1 > /tmp/out.txt
```

---

## grep and Regular Expressions

| Flag | Description |
|------|-------------|
| `-v` | Invert match (lines that do NOT match) |
| `-i` | Case insensitive |
| `-n` | Show line numbers |
| `-r` | Recursive search in directory |
| `-o` | Print only the matched part |
| `-E` | Extended regex (enables `+`, `{n}`, `\|`) |

### Regex anchors and patterns

| Pattern | Description |
|---------|-------------|
| `^`     | Start of line |
| `$`     | End of line |
| `[^x]`  | Any character except x |
| `.*`    | Any character, zero or more times |
| `x*`    | Zero or more repetitions of x |
| `[0-9]{3}` | Exactly 3 digits (requires `-E`) |
| `a\|b`  | Match a or b (requires `-E`) |

```bash
# Lines starting with root
grep "^root" /etc/passwd

# Lines ending with bash
grep "bash$" /etc/passwd

# Extract only usernames from /etc/passwd
grep -o "^[^:]*" /etc/passwd

# Lines that are not comments and not empty
grep -v "^#" /etc/ssh/sshd_config | grep -v "^$"

# Case insensitive recursive search with line numbers
grep -rni "timeout" /etc
```

---

## tar, gzip, bzip2

| Flag | Description |
|------|-------------|
| `c`  | Create archive |
| `x`  | Extract archive |
| `t`  | List contents |
| `z`  | Compress/decompress with gzip |
| `j`  | Compress/decompress with bzip2 |
| `f`  | Specify archive filename (always followed by the filename) |
| `v`  | Verbose output |
| `C`  | Specify destination directory |

```bash
# Create gzip archive
tar -czf /tmp/backup.tar.gz /etc/ssh

# List contents without extracting
tar -tf /tmp/backup.tar.gz

# Extract to specific directory
tar -xzf /tmp/backup.tar.gz -C /tmp/restore/

# Extract a single file (path must match exactly as shown by -t)
tar -xzf /tmp/backup.tar.gz -C /tmp/restore/ etc/ssh/sshd_config
```

**Key rule:** After `-f`, always the archive file first, then the source/destination.

---

## Hard Links and Soft Links

| | Hard Link | Soft Link (Symlink) |
|-|-----------|---------------------|
| Command | `ln src dst` | `ln -s src dst` |
| Points to | Same inode | Filename |
| Works across filesystems | No | Yes |
| Works for directories | No | Yes |
| Survives deletion of original | Yes | No (broken link) |

```bash
# Hard link
ln /tmp/original.txt /tmp/hardlink.txt

# Soft link
ln -s /tmp/original.txt /tmp/softlink.txt

# Verify inodes (hard link shares inode with original)
ls -li /tmp/original.txt /tmp/hardlink.txt /tmp/softlink.txt
```
