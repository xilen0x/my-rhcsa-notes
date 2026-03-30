# Block 3 — Shell Scripts

## Script Structure

```bash
#!/bin/bash
# Shebang line — tells the system which interpreter to use.
# Always include it as the first line of every script.
```

```bash
chmod +x script.sh    # make the script executable before running
/path/to/script.sh    # run using absolute path
./script.sh           # run using relative path (only works from same directory)
```

---

## Conditionals

```bash
# Basic if/elif/else
if [ condition ]; then
    # commands
elif [ condition ]; then
    # commands
else
    # commands
fi
```

### Common test operators

| Operator | Description |
|----------|-------------|
| `-z "$var"` | True if string is empty |
| `-n "$var"` | True if string is not empty |
| `-f "$path"` | True if path is a regular file |
| `-d "$path"` | True if path is a directory |
| `-e "$path"` | True if path exists (file or directory) |
| `$a -eq $b` | Equal (numeric) |
| `$a -ne $b` | Not equal (numeric) |
| `$a -lt $b` | Less than (numeric) |
| `$a -le $b` | Less than or equal (numeric) |
| `$a -gt $b` | Greater than (numeric) |
| `$a -ge $b` | Greater than or equal (numeric) |

```bash
# Validate argument and exit if missing
if [ -z "$1" ]; then
    echo "Usage: $0 <argument>"
    exit 1
fi

# Check if path is a file or directory
if [ -f "$1" ]; then
    echo "$1 is a file"
elif [ -d "$1" ]; then
    echo "$1 is a directory"
else
    echo "$1 does not exist"
fi
```

---

## Loops

```bash
# Iterate over a fixed list
for user in alice bob charlie; do
    echo "User: $user"
done

# Iterate over files matching a pattern
for f in /etc/*.conf; do
    echo "Config: $f"
done

# Iterate over all script arguments
for arg in "$@"; do
    echo "Argument: $arg"
done

# While loop with arithmetic
counter=1
while [ $counter -le 3 ]; do
    echo "Counter: $counter"
    counter=$((counter + 1))
done
```

---

## Script Arguments

| Variable | Description |
|----------|-------------|
| `$0` | Script name |
| `$1`, `$2` ... | Positional arguments |
| `$@` | All arguments as separate words |
| `$#` | Number of arguments passed |
| `$?` | Exit code of last command (0=success) |

---

## Command Substitution

```bash
# $() executes the command inside and captures its stdout.
# The result can be assigned to a variable or used inline.
current_user=$(whoami)
timestamp=$(date +%Y%m%d_%H%M%S)

# Used inline without a variable
echo "Today is $(date)"

# Nested
echo "$(basename $(which bash))"
```

---

## Practical Example — Backup Script

```bash
#!/bin/bash

if [ -z "$1" ]; then
    echo "Usage: $0 <directory>"
    exit 1
fi

if [ ! -d "$1" ]; then
    echo "Error: '$1' is not a valid directory"
    exit 1
fi

timestamp=$(date +%Y%m%d_%H%M%S)
backup_path="/tmp/backup_${timestamp}.tar.gz"

tar -czf "$backup_path" -C "$(dirname "$1")" "$(basename "$1")"

if [ $? -ne 0 ]; then
    echo "Error creating backup"
    exit 1
fi

echo "Backup created: $backup_path"
```