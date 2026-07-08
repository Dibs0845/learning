# Linux Commands — Complete Reference Guide

> A detailed, category-wise reference of Linux commands with syntax, key options, practical examples, and notes on when to use each. Suitable for daily development work, DevOps tasks, and interview preparation.

---

## Table of Contents

1. [Getting Help](#1-getting-help)
2. [Navigation & Exploring the File System](#2-navigation--exploring-the-file-system)
3. [File & Directory Operations](#3-file--directory-operations)
4. [Viewing File Contents](#4-viewing-file-contents)
5. [Text Processing & Filters](#5-text-processing--filters)
6. [Searching — find, locate, grep](#6-searching--find-locate-grep)
7. [Permissions & Ownership](#7-permissions--ownership)
8. [Users & Groups](#8-users--groups)
9. [Process Management](#9-process-management)
10. [System Information & Monitoring](#10-system-information--monitoring)
11. [Networking](#11-networking)
12. [SSH, SCP & rsync](#12-ssh-scp--rsync)
13. [Archiving & Compression](#13-archiving--compression)
14. [Package Management](#14-package-management)
15. [Disk & Filesystem Management](#15-disk--filesystem-management)
16. [Services & systemd](#16-services--systemd)
17. [Shell, Environment & Redirection](#17-shell-environment--redirection)
18. [Job Scheduling — cron & at](#18-job-scheduling--cron--at)
19. [Advanced & Troubleshooting Tools](#19-advanced--troubleshooting-tools)
20. [Practical One-Liner Cheat Sheet](#20-practical-one-liner-cheat-sheet)
21. [Bash Scripting Essentials](#21-bash-scripting-essentials)
22. [Vim Quick Reference](#22-vim-quick-reference)
23. [Firewall & Security Hardening](#23-firewall--security-hardening)
24. [Log Management & logrotate](#24-log-management--logrotate)
25. [Performance Troubleshooting Playbook](#25-performance-troubleshooting-playbook)
26. [Scenario-Based Interview Questions](#26-scenario-based-interview-questions)
27. [Conceptual Interview Questions & Answers](#27-conceptual-interview-questions--answers)
28. [Advanced Scenario-Based Questions](#28-advanced-scenario-based-questions)

---

## 1. Getting Help

### `man` — manual pages
The primary documentation system on Linux.

```bash
man ls              # full manual for ls
man 5 crontab       # section 5 (file formats) — man page sections matter
man -k "copy files" # search all man pages by keyword (same as apropos)
```

**Man page sections:** 1 = user commands, 2 = system calls, 3 = library functions, 5 = file formats (`/etc/passwd`, crontab), 8 = admin commands.

### `--help`, `info`, `whatis`, `apropos`

```bash
ls --help           # quick option summary (faster than man)
info coreutils      # GNU hypertext manuals (more detailed than man for GNU tools)
whatis grep         # one-line description of a command
apropos network     # find commands related to a topic
```

### `which`, `whereis`, `type`

```bash
which python3       # full path of the executable that would run: /usr/bin/python3
whereis nginx       # binary + source + man page locations
type cd             # tells you if it's a builtin, alias, function, or file
type -a python      # ALL matches in order of precedence
```

> **Interview note:** `cd` is a shell *builtin*, not a program — that's why `which cd` finds nothing but `type cd` says "shell builtin". A child process can't change its parent's directory, so `cd` must run inside the shell itself.

---

## 2. Navigation & Exploring the File System

### `pwd` — print working directory

```bash
pwd                 # /home/user/projects
pwd -P              # physical path — resolves symlinks
```

### `cd` — change directory

```bash
cd /var/log         # absolute path
cd projects/api     # relative path
cd ..               # up one level
cd ../..            # up two levels
cd ~                # home directory (same as plain cd)
cd -                # previous directory (toggle back and forth)
```

### `ls` — list directory contents

```bash
ls                  # basic listing
ls -l               # long format: permissions, owner, size, mtime
ls -a               # include hidden files (dotfiles)
ls -la              # both combined — the most-used form
ls -lh              # human-readable sizes (4.0K, 1.2G)
ls -lt              # sort by modification time, newest first
ls -ltr             # newest LAST (handy: latest file ends up at your cursor)
ls -lS              # sort by size, largest first
ls -ld /etc         # info about the directory itself, not its contents
ls -li              # show inode numbers
ls -R               # recursive listing
```

**Reading `ls -l` output:**

```text
-rwxr-xr--  2 alice devs  4096 Jul  8 10:30 deploy.sh
│└┬┘└┬┘└┬┘  │  │     │     │        │          └ name
││  │  │    │  │     │     │        └ last modification time
││  │  │    │  │     │     └ size in bytes
││  │  │    │  │     └ group
││  │  │    │  └ owner
││  │  │    └ hard link count
││  │  └ others: r-- (read only)
││  └ group: r-x (read + execute)
│└ owner: rwx (read + write + execute)
└ file type: - file, d directory, l symlink, c char device, b block device, s socket, p pipe
```

### `tree` — directory tree view

```bash
tree                # full tree of current directory
tree -L 2           # limit depth to 2 levels
tree -d             # directories only
tree -a -I 'node_modules|.git'   # all files, excluding patterns
```

### Linux Filesystem Hierarchy (essential knowledge)

| Directory | Purpose |
|---|---|
| `/` | Root of everything |
| `/bin`, `/usr/bin` | User commands (ls, cp, bash) |
| `/sbin`, `/usr/sbin` | Admin commands (fdisk, iptables) |
| `/etc` | System-wide configuration files |
| `/var` | Variable data — logs (`/var/log`), spool, caches |
| `/tmp` | Temporary files, cleared on reboot |
| `/home` | User home directories |
| `/root` | Root user's home |
| `/proc` | Virtual FS — kernel & process info (`/proc/cpuinfo`, `/proc/<pid>/`) |
| `/sys` | Virtual FS — device & kernel settings |
| `/dev` | Device files (`/dev/sda`, `/dev/null`) |
| `/opt` | Optional/third-party software |
| `/usr/local` | Locally-compiled software (survives OS package upgrades) |
| `/boot` | Kernel & bootloader files |
| `/mnt`, `/media` | Mount points for filesystems / removable media |
| `/lib`, `/usr/lib` | Shared libraries |

---

## 3. File & Directory Operations

### `touch` — create empty file / update timestamp

```bash
touch notes.txt             # create if missing, else update mtime to now
touch -t 202607081030 f.txt # set a specific timestamp
touch file{1..5}.txt        # brace expansion → file1.txt ... file5.txt
```

### `mkdir` — create directories

```bash
mkdir logs                       # single directory
mkdir -p app/src/utils           # create parent chain, no error if exists
mkdir -m 700 secrets             # create with specific permissions
mkdir -p project/{src,test,docs} # multiple subdirectories at once
```

### `cp` — copy files & directories

```bash
cp file.txt backup.txt          # copy file
cp file.txt /tmp/               # copy into directory
cp -r src/ src-backup/          # recursive — required for directories
cp -p file.txt copy.txt         # preserve mode, ownership, timestamps
cp -a src/ dest/                # archive mode: -r + preserve everything (best for backups)
cp -i file.txt dest.txt         # interactive — prompt before overwrite
cp -u *.conf /etc/app/          # only copy if source is newer than destination
cp file.txt{,.bak}              # quick backup trick → copies to file.txt.bak
```

### `mv` — move / rename

```bash
mv old.txt new.txt              # rename
mv file.txt /tmp/               # move
mv -i a.txt b.txt               # prompt before overwriting
mv -n a.txt b.txt               # never overwrite
mv dir1/ dir2/                  # rename/move a directory (no -r needed, unlike cp)
```

> `mv` within the same filesystem just rewrites the directory entry (instant, even for huge files). Across filesystems it becomes copy + delete.

### `rm` — remove files & directories

```bash
rm file.txt                     # delete a file
rm -i *.log                     # prompt for each file
rm -r build/                    # recursive — delete directory and contents
rm -rf node_modules/            # force recursive — no prompts, no errors on missing
rm -- -weirdfile                # delete a file whose name starts with a dash
```

> ⚠️ **There is no trash/recycle bin.** `rm -rf` is immediate and unrecoverable. Double-check variables in scripts: `rm -rf "$DIR"/` with an empty `$DIR` becomes `rm -rf /`. Use `rm -rf "${DIR:?DIR not set}"/` to make the script abort instead.

### `rmdir` — remove *empty* directories only

```bash
rmdir old-empty-dir             # fails if not empty (a safety feature)
rmdir -p a/b/c                  # remove chain of empty directories
```

### `ln` — links

```bash
ln -s /var/www/app current       # symbolic (soft) link — a pointer by path
ln file.txt hardlink.txt          # hard link — another name for the same inode
ls -li                            # verify: hard links share the same inode number
readlink -f current               # resolve a symlink to its final target
```

| Hard link | Symbolic link |
|---|---|
| Same inode — literally the same file | Separate small file containing a path |
| Can't cross filesystems | Can cross filesystems |
| Can't link directories | Can link directories |
| File survives until *all* hard links deleted | Breaks (dangles) if target is deleted |

### `stat` & `file` — metadata and type

```bash
stat file.txt        # inode, size, permissions (octal + symbolic), atime/mtime/ctime
file mystery.bin     # identifies actual content type (ELF binary, JPEG, ASCII text…)
file *               # quick type survey of a directory
```

> `mtime` = content modified; `ctime` = metadata changed (perms/owner); `atime` = last read. `file` reads magic bytes — it doesn't trust the extension.

---

## 4. Viewing File Contents

### `cat` — concatenate & print

```bash
cat file.txt                     # print whole file
cat -n file.txt                  # with line numbers
cat -A file.txt                  # show invisibles: tabs ^I, line ends $ (debug whitespace)
cat file1 file2 > merged.txt     # concatenate files
cat > quick.txt                  # type content, Ctrl+D to save (quick file creation)
tac file.txt                     # print lines in reverse order
```

### `less` — pager (the right way to read big files)

```bash
less /var/log/syslog
less +F app.log        # follow mode like tail -f, Ctrl+C to stop, F to resume
less -N file.txt       # with line numbers
less -S wide.csv       # don't wrap long lines (scroll horizontally with ←/→)
```

**Inside less:** `Space`/`b` page down/up · `/pattern` search forward · `?pattern` backward · `n`/`N` next/prev match · `g`/`G` start/end · `q` quit. `less` streams the file — it opens a 10GB log instantly, unlike an editor.

### `head` & `tail`

```bash
head file.txt            # first 10 lines
head -n 50 file.txt      # first 50 lines
head -c 1K file.bin      # first kilobyte
tail file.txt            # last 10 lines
tail -n 100 app.log      # last 100 lines
tail -f app.log          # FOLLOW — live-stream appended lines (log watching)
tail -F app.log          # follow across log rotation (re-opens the file by name)
tail -n +2 data.csv      # everything FROM line 2 (skip a header row)
```

> Use `tail -F` (capital) in production — when logrotate swaps the file, `-f` keeps following the old (now renamed) file; `-F` reattaches to the new one.

### `wc` — count lines/words/bytes

```bash
wc -l access.log         # line count
wc -w essay.txt          # word count
wc -c file.bin           # byte count
ls | wc -l               # count files in a directory
```

### `nl`, `more`, `strings`

```bash
nl script.sh             # number non-empty lines
strings binary_file      # extract printable text from a binary (find version strings, URLs)
```

---

## 5. Text Processing & Filters

These commands are designed to be **chained with pipes** — each reads stdin, transforms, writes stdout.

### `grep` — search text by pattern

```bash
grep "error" app.log             # lines containing "error"
grep -i "error" app.log          # case-insensitive
grep -v "DEBUG" app.log          # invert — lines NOT matching
grep -r "TODO" src/              # recursive through a directory
grep -rn "apiKey" .              # recursive + show file:line numbers
grep -c "GET" access.log         # count matching lines
grep -l "main" *.c               # only list filenames that match
grep -w "port" config            # whole-word match (won't match "portable")
grep -A 3 -B 1 "Exception" app.log  # 3 lines After, 1 Before each match
grep -E "50[0-9]" access.log     # extended regex (same as egrep)
grep -F "a.b[c]" file            # fixed string — no regex interpretation (fast, safe)
grep -o 'user=[a-z]*' app.log    # print only the matched part, not whole lines
grep -P '\d{3}-\d{4}' contacts   # Perl-compatible regex (GNU grep)
ps aux | grep '[n]ginx'          # bracket trick: excludes the grep process itself
```

### `sed` — stream editor (find/replace and line surgery)

```bash
sed 's/http/https/' urls.txt          # replace FIRST occurrence per line
sed 's/http/https/g' urls.txt         # replace ALL occurrences per line (g = global)
sed -i 's/debug=true/debug=false/' app.conf   # -i = edit the file in place
sed -i.bak 's/old/new/g' file         # in-place but keep a .bak backup
sed -n '5,10p' file.txt               # print only lines 5–10 (-n suppresses default output)
sed '/^#/d' config                    # delete comment lines
sed '/^$/d' file                      # delete blank lines
sed '1d' data.csv                     # delete the first line (strip header)
sed 's/[0-9]\+/NUM/g' log             # regex replace
sed 's|/var/www|/srv/app|g' conf      # any delimiter works — use | when pattern has /
sed -e 's/a/A/' -e 's/b/B/' file      # multiple expressions
sed '3i\inserted line' file           # insert before line 3
sed '$a\appended line' file           # append after the last line
```

### `awk` — column-oriented processing language

awk splits each line into fields (`$1`, `$2`, … `$NF` = last) on whitespace by default.

```bash
awk '{print $1}' access.log                  # first column (e.g., IP addresses)
awk '{print $1, $NF}' file                   # first and last columns
awk -F: '{print $1}' /etc/passwd             # -F sets the delimiter (here ':') → usernames
awk '$9 == 500' access.log                   # filter: rows where column 9 equals 500
awk '$3 > 1000 {print $1, $3}' data.txt      # condition + action
awk '{sum += $5} END {print sum}' sizes.txt  # sum a column
awk '{sum+=$1; n++} END {print sum/n}' nums  # average
awk 'NR==5' file                             # print line 5 (NR = record number)
awk 'NR>=10 && NR<=20' file                  # lines 10–20
awk '!seen[$0]++' file                       # remove duplicate lines WITHOUT sorting (classic)
awk '{print NR": "$0}' file                  # number every line
awk 'length($0) > 80' code.py                # lines longer than 80 chars
awk -F, '{print $2}' data.csv                # CSV second column (simple CSVs only)
```

> **grep vs sed vs awk:** grep *finds* lines, sed *edits* streams, awk *computes over columns*. If you're reaching for columns or arithmetic, it's awk.

### `cut` — extract columns (simpler than awk)

```bash
cut -d: -f1 /etc/passwd          # field 1, delimiter ':'
cut -d, -f2,4 data.csv           # fields 2 and 4 from CSV
cut -d, -f2- data.csv            # field 2 to end
cut -c1-8 file                   # by character position (fixed-width data)
```

### `sort` — sort lines

```bash
sort names.txt                   # alphabetical
sort -r names.txt                # reverse
sort -n sizes.txt                # numeric (without -n, 9 > 10 alphabetically!)
sort -h du-output.txt            # human-numeric: understands 1K, 2M, 3G
sort -u list.txt                 # sort + drop duplicates
sort -t: -k3 -n /etc/passwd      # sort by field 3, ':' delimiter, numerically (by UID)
sort -k2,2 -k1,1 data.txt        # sort by col 2, then col 1 (multi-key)
```

### `uniq` — collapse repeated adjacent lines

```bash
sort file | uniq                 # dedupe (uniq needs sorted input — adjacent-only!)
sort file | uniq -c              # count occurrences of each line
sort file | uniq -c | sort -rn   # frequency table, most common first ← the classic
sort file | uniq -d              # show only duplicated lines
sort file | uniq -u              # show only lines that appear exactly once
```

### `tr` — translate/delete characters

```bash
tr 'a-z' 'A-Z' < file            # lowercase → uppercase
tr -d '\r' < windows.txt > unix.txt   # strip carriage returns (CRLF → LF)
tr -s ' ' < messy.txt            # squeeze runs of spaces into one
echo "$PATH" | tr ':' '\n'       # split PATH into lines
```

### `diff`, `comm`, `cmp` — compare

```bash
diff old.conf new.conf           # line-by-line differences
diff -u old.conf new.conf        # unified format (what git/patches use)
diff -r dirA/ dirB/              # compare whole directory trees
diff -q dirA/ dirB/ -r           # just report WHICH files differ
comm -12 <(sort a.txt) <(sort b.txt)   # lines common to both (sorted input)
comm -23 <(sort a.txt) <(sort b.txt)   # lines only in a.txt
cmp file1.bin file2.bin          # byte-level compare (binaries)
```

### `xargs` — build commands from stdin

```bash
find . -name "*.tmp" | xargs rm            # delete all found files
find . -name "*.log" -print0 | xargs -0 rm # NUL-separated: safe for spaces in names
cat urls.txt | xargs -n1 curl -sO          # run curl once per line (-n1)
ls *.png | xargs -I{} convert {} {}.webp   # -I{} = placeholder for each item
cat hosts.txt | xargs -P4 -n1 ping -c1     # -P4 = 4 parallel processes
```

### `tee` — write to file AND pass through

```bash
./deploy.sh | tee deploy.log               # see output live AND save it
./build.sh 2>&1 | tee -a build.log         # append, capture stderr too
echo "127.0.0.1 dev.local" | sudo tee -a /etc/hosts   # sudo-write to a root file
```

> The `sudo tee` trick matters: `sudo echo x > /root/file` fails because the *shell* (not sudo) performs the redirection.

### `paste`, `join`, `split`, `column`

```bash
paste names.txt emails.txt        # merge files side by side (tab-separated)
join users.txt scores.txt         # relational join on first field (sorted input)
split -l 1000 big.csv chunk_      # split into 1000-line files: chunk_aa, chunk_ab…
split -b 100M big.iso part_       # split by size
column -t -s: /etc/passwd         # align into a neat table
```

---

## 6. Searching — find, locate, grep

### `find` — real-time filesystem search (the power tool)

```bash
# By name
find /var/log -name "*.log"            # exact glob, case-sensitive
find . -iname "readme*"                # case-insensitive
find . -path "*/test/*" -name "*.py"   # match against full path

# By type
find . -type f                         # files
find . -type d                         # directories
find . -type l                         # symlinks
find /tmp -empty                       # empty files/dirs

# By size
find / -size +100M                     # larger than 100 MB
find . -size -1k                       # smaller than 1 KB

# By time
find . -mtime -1                       # modified in the last 24 hours
find . -mtime +30                      # modified more than 30 days ago
find . -mmin -15                       # modified in the last 15 minutes
find . -newer reference.txt            # modified more recently than this file

# By permissions / ownership
find / -perm -4000 -type f             # setuid binaries (security audit)
find . -perm 777                       # exactly 777
find /home -user alice                 # owned by alice
find / -nouser                         # orphaned files (deleted owner)

# Actions on results
find . -name "*.tmp" -delete                       # delete matches
find . -name "*.sh" -exec chmod +x {} \;           # run command per file
find . -name "*.log" -exec grep -l "ERROR" {} +    # + batches args (faster than \;)
find . -type f -name "*.js" | xargs wc -l          # pipe to xargs

# Control
find . -maxdepth 2 -name "*.conf"      # limit recursion depth
find / -name "x" 2>/dev/null           # hide permission-denied noise
find . -name "*.py" -not -path "*/venv/*"   # exclusion
```

### `locate` — indexed search (fast, possibly stale)

```bash
locate nginx.conf        # instant — searches a prebuilt database
sudo updatedb            # refresh the database (runs daily via cron normally)
locate -i readme         # case-insensitive
```

> `find` walks the disk live (always accurate, slower); `locate` queries an index (instant, but blind to files created since the last `updatedb`).

---

## 7. Permissions & Ownership

### The permission model

Every file has three permission sets — **user (owner)**, **group**, **others** — each with **r**ead (4), **w**rite (2), e**x**ecute (1).

```text
rwx r-x r--   =   7 5 4
 │   │   └ others: read
 │   └ group: read + execute
 └ owner: read + write + execute
```

**For directories the meaning shifts:** `r` = list names, `w` = create/delete/rename entries inside, `x` = *enter* the directory and access inodes. A directory with `r` but no `x` lets you see names but access nothing.

### `chmod` — change permissions

```bash
# Octal form
chmod 755 script.sh        # rwxr-xr-x — standard for scripts/binaries
chmod 644 page.html        # rw-r--r-- — standard for data files
chmod 600 id_rsa           # rw------- — private keys MUST be this
chmod 700 ~/.ssh           # rwx------ — private directory

# Symbolic form
chmod +x deploy.sh         # add execute for everyone
chmod u+x,g-w file         # add x for owner, remove w for group
chmod o= file              # strip all permissions from others
chmod -R g+rX shared/      # recursive; capital X = x only on dirs & already-executable files
```

> `-R` with `X` (capital) is the safe recursive form — plain `chmod -R 755` makes every *data file* executable too.

### Special permission bits

```bash
chmod 4755 /usr/bin/passwd   # setuid (4) — runs AS the file's owner (root)
chmod 2775 /srv/shared       # setgid (2) on dir — new files inherit the dir's group
chmod 1777 /tmp              # sticky bit (1) — only a file's owner can delete it
```

| Bit | On files | On directories |
|---|---|---|
| setuid (`s` in user x) | Process runs as file owner | (ignored) |
| setgid (`s` in group x) | Process runs as file group | New files inherit directory's group |
| sticky (`t` in others x) | (ignored) | Users can delete only their own files — why `/tmp` is 1777 |

### `chown` & `chgrp` — change ownership

```bash
sudo chown alice file.txt          # change owner
sudo chown alice:devs file.txt     # owner AND group
sudo chown -R www-data:www-data /var/www/app   # recursive (web deployments)
sudo chgrp devs shared.txt         # group only
```

### `umask` — default permission mask

```bash
umask            # show current mask, e.g. 0022
umask 077        # new files become 600, dirs 700 (private by default)
```

New file permissions = `666 - umask`; directories = `777 - umask`. Mask `022` → files 644, dirs 755.

### `sudo` & `su`

```bash
sudo systemctl restart nginx    # run one command as root
sudo -u postgres psql           # run as a specific user
sudo -i                         # root login shell (root's env)
sudo !!                         # re-run the previous command with sudo
su - alice                      # switch user with a full login environment
sudo visudo                     # safely edit /etc/sudoers (syntax-checked)
```

---

## 8. Users & Groups

### Key files

```bash
cat /etc/passwd    # user accounts:  name:x:UID:GID:comment:home:shell
cat /etc/group     # groups:         name:x:GID:members
sudo cat /etc/shadow   # hashed passwords + aging (root-only)
```

### Managing users

```bash
sudo useradd -m -s /bin/bash bob     # create with home dir (-m) and shell
sudo passwd bob                      # set password
sudo usermod -aG docker bob          # add to group — -a is CRITICAL (append);
                                     # without it, -G REPLACES all groups
sudo usermod -L bob                  # lock account
sudo usermod -s /usr/sbin/nologin svc # no interactive login (service accounts)
sudo userdel -r bob                  # delete user AND home directory
sudo groupadd devs                   # create a group
```

### Identity & session info

```bash
id                  # your UID, GID, and all groups
id alice            # someone else's
whoami              # effective username
groups              # your group memberships
who                 # who is logged in
w                   # who is logged in + what they're running + load
last                # login history
last -x reboot      # reboot history
```

> After `usermod -aG`, the user must **log out and back in** for the new group to take effect (or use `newgrp docker` for the current shell).

---

## 9. Process Management

### `ps` — process snapshot

```bash
ps aux                       # every process, BSD style — the everyday form
ps -ef                       # every process, System V style (shows PPID)
ps aux --sort=-%mem | head   # top memory consumers
ps aux --sort=-%cpu | head   # top CPU consumers
ps -u alice                  # processes owned by a user
ps -p 1234 -o pid,ppid,cmd,%cpu,%mem   # custom columns for one PID
pgrep -f "node server.js"    # find PIDs matching a full command line
pstree -p                    # process tree with PIDs (see parent/child)
```

**`ps aux` columns:** `%CPU`/`%MEM` usage, `VSZ` virtual memory, `RSS` real memory (KB), `STAT` state — `R` running, `S` sleeping, `D` uninterruptible I/O wait (bad sign if stuck), `Z` zombie, `T` stopped.

### `kill`, `pkill`, `killall` — signals

```bash
kill 1234                # SIGTERM (15) — polite: "clean up and exit"
kill -9 1234             # SIGKILL — force, cannot be caught; NO cleanup happens
kill -HUP 1234           # SIGHUP — many daemons reload config on this
kill -l                  # list all signals
pkill -f "node server"   # kill by matching command line
pkill -u baduser         # kill all of a user's processes
killall nginx            # kill all processes named exactly "nginx"
```

> **Always try SIGTERM first.** SIGKILL skips all cleanup handlers — open files aren't flushed, locks aren't released, temp files remain. A process stuck in `D` state won't even die to `kill -9` (it's waiting on the kernel).

### Jobs & background execution

```bash
./long-task.sh &         # start in background
jobs                     # list this shell's jobs
Ctrl+Z                   # suspend the foreground job
bg                       # resume suspended job in background
fg %1                    # bring job 1 back to foreground
disown -h %1             # detach job so it survives shell exit
nohup ./server.sh &      # immune to hangup — survives closing the terminal
nohup ./job.sh > job.log 2>&1 &   # the standard fire-and-forget form
```

### `top` / `htop` — live monitoring

```bash
top                      # classic live view
htop                     # nicer: colors, mouse, tree view (usually needs install)
```

**Inside top:** `M` sort by memory · `P` by CPU · `k` kill a PID · `1` per-core CPU view · `q` quit.

**Reading load average** (`0.80, 1.20, 2.00` = 1/5/15-min averages): roughly, load ≈ number of CPU cores means fully utilized. Load 4.0 on a 4-core box is busy; on a 16-core box it's light.

### `nice` & `renice` — priorities

```bash
nice -n 19 ./batch-job.sh     # start at lowest priority (be nice to others)
sudo renice -5 -p 1234        # raise priority of a running process (needs root for negative)
```

Niceness runs −20 (highest priority) to +19 (lowest).

---

## 10. System Information & Monitoring

### System identity

```bash
uname -a                 # kernel, hostname, architecture — one line
uname -r                 # kernel version only
hostname                 # machine name
hostnamectl              # hostname + OS + kernel + virtualization (systemd)
cat /etc/os-release      # distribution name & version
uptime                   # time up + logged-in users + load averages
date                     # current date/time
timedatectl              # time, timezone, NTP sync status
```

### Memory

```bash
free -h                  # human-readable memory summary
cat /proc/meminfo        # detailed kernel memory stats
vmstat 2                 # rolling stats every 2s (memory, swap, io, cpu)
```

> **Read the `available` column, not `free`.** Linux deliberately uses spare RAM as disk cache ("free" looks low on healthy systems); `available` is what applications can actually claim. Swap activity in `vmstat` (`si`/`so` columns nonzero) is the real memory-pressure signal.

### Disk usage

```bash
df -h                    # filesystem usage per mount — "is the disk full?"
df -i                    # inode usage — disks can be "full" with free space if inodes run out!
du -sh /var/log          # total size of a directory
du -h --max-depth=1 / 2>/dev/null | sort -hr | head   # what's eating the disk
du -ah . | sort -hr | head -20                        # 20 biggest files/dirs here
ncdu /                   # interactive disk usage browser (install separately)
```

### Hardware

```bash
lscpu                    # CPU model, cores, threads, cache
lsblk                    # block devices & partitions as a tree
lspci                    # PCI devices (network cards, GPUs)
lsusb                    # USB devices
nproc                    # number of CPU cores (handy in scripts: make -j$(nproc))
cat /proc/cpuinfo        # per-core details
```

### Kernel & logs

```bash
dmesg | tail -50             # recent kernel messages
dmesg -T | grep -i "killed process"   # find OOM-killer victims (with human timestamps)
journalctl -k                # kernel log via journald
```

> The **OOM killer** check (`dmesg | grep -i oom`) is the first move when a process "randomly disappeared" — the kernel kills the biggest memory consumer when RAM is exhausted.

### I/O and per-process monitoring

```bash
iostat -x 2              # per-disk I/O stats (util%, await) — needs sysstat
iotop                    # top for disk I/O per process (root)
sar -u 1 5               # CPU history sampling (sysstat)
```

---

## 11. Networking

### `ip` — interfaces, addresses, routes (replaces ifconfig)

```bash
ip addr            # all interfaces and their IPs (short: ip a)
ip -br addr        # brief, one line per interface — the readable form
ip link            # interface link state (up/down, MAC)
ip route           # routing table; first line shows default gateway
ip route get 8.8.8.8    # which interface/route would reach this IP
sudo ip link set eth0 up    # bring an interface up
```

### `ping` — reachability & latency

```bash
ping google.com          # continuous echo (Ctrl+C for stats)
ping -c 4 8.8.8.8        # exactly 4 packets
ping -c 1 -W 2 host      # 1 packet, 2s timeout — good for scripts
```

> Classic triage: `ping 8.8.8.8` works but `ping google.com` fails → **DNS problem**, not connectivity.

### `ss` / `netstat` — sockets & ports

```bash
ss -tulpn                # ALL listening TCP/UDP ports + owning process ← memorize this
ss -tn state established # current established TCP connections
ss -tn 'dport = :443'    # connections to port 443
netstat -tulpn           # legacy equivalent (net-tools package)
```

Flags: `-t` tcp, `-u` udp, `-l` listening, `-p` process, `-n` numeric (skip DNS).

### DNS — `dig`, `nslookup`, `host`

```bash
dig example.com              # full DNS query detail
dig +short example.com       # just the answer (IPs)
dig MX example.com           # specific record type (MX, TXT, CNAME, NS…)
dig @8.8.8.8 example.com     # query a specific DNS server (bypass local resolver)
dig -x 93.184.216.34         # reverse lookup (IP → name)
nslookup example.com         # simpler alternative
cat /etc/resolv.conf         # which resolver this machine uses
cat /etc/hosts               # local overrides (checked before DNS)
```

### `curl` — HTTP requests & API testing

```bash
curl https://api.example.com/users            # GET, body to stdout
curl -i https://example.com                   # include response headers
curl -I https://example.com                   # HEAD — headers only
curl -L http://example.com                    # follow redirects
curl -o page.html https://example.com         # save to file
curl -O https://host/file.tar.gz              # save with remote filename
curl -s https://api.x.com | jq .              # silent + pretty-print JSON
curl -X POST -H "Content-Type: application/json" \
     -d '{"name":"alice"}' https://api.x.com/users     # JSON POST
curl -H "Authorization: Bearer $TOKEN" https://api.x.com/me   # auth header
curl -u user:pass https://api.x.com           # basic auth
curl -w "%{http_code} %{time_total}s\n" -o /dev/null -s https://x.com  # status + timing
curl -v https://example.com                   # verbose: TLS handshake, headers both ways
curl --connect-timeout 5 --max-time 30 URL    # timeouts (always set in scripts)
```

### `wget` — downloads

```bash
wget https://host/file.iso        # simple download with progress
wget -c https://host/big.iso      # resume a partial download
wget -r -np -l 2 https://site/docs/    # recursive mirror, 2 levels, don't ascend
```

### `traceroute` / `tracepath` / `mtr`

```bash
traceroute google.com     # show every router hop to the destination
mtr google.com            # live combined ping+traceroute (find WHERE loss happens)
```

### `nc` (netcat) — TCP/UDP swiss army knife

```bash
nc -zv host 443              # test if a port is open (-z scan, -v verbose)
nc -zv host 20-25            # scan a port range
nc -l 8080                   # listen on a port (test a client)
echo "ping" | nc host 6379   # send raw bytes to a service
```

> `nc -zv db-host 5432` answers "can this box reach the database?" in one second — firewall triage 101.

---

## 12. SSH, SCP & rsync

### `ssh` — secure remote shell

```bash
ssh user@server                     # basic login
ssh -p 2222 user@server             # non-default port
ssh -i ~/.ssh/prod_key user@server  # specific private key
ssh user@server 'df -h; uptime'     # run remote commands without a shell
ssh -L 5433:localhost:5432 user@db-host   # local port forward:
                                          # my :5433 → db-host's :5432 (DB tunnels)
ssh -N -f -L 8080:internal:80 user@jump   # tunnel only (-N), background (-f)
ssh -J user@bastion user@private-host     # jump through a bastion host
```

### SSH keys

```bash
ssh-keygen -t ed25519 -C "me@work"     # generate a modern keypair
ssh-copy-id user@server                # install your public key on the server
                                       # (appends to ~/.ssh/authorized_keys)
ssh-add ~/.ssh/prod_key                # add key to the agent (stop retyping passphrase)
```

**`~/.ssh/config` — aliases that save your sanity:**

```text
Host prod
    HostName 203.0.113.10
    User deploy
    IdentityFile ~/.ssh/prod_key
    Port 2222
```
Then just: `ssh prod`.

> Permissions are enforced: `~/.ssh` must be **700**, private keys **600**, `authorized_keys` **600** — SSH silently refuses keys with loose permissions.

### `scp` — copy over SSH

```bash
scp file.txt user@server:/tmp/               # local → remote
scp user@server:/var/log/app.log ./          # remote → local
scp -r ./build user@server:/var/www/app      # recursive
scp -P 2222 file user@server:/tmp/           # port is capital -P (unlike ssh!)
```

### `rsync` — smart sync (prefer over scp for anything big)

```bash
rsync -avz src/ user@server:/srv/app/        # archive, verbose, compressed
rsync -avz --delete src/ server:/srv/app/    # true mirror — delete extraneous remote files
rsync -avz --progress bigfile server:/data/  # show progress
rsync -avzn --delete src/ server:/srv/app/   # -n = DRY RUN — always test --delete first
rsync -avz --exclude 'node_modules' --exclude '.git' ./ server:/app/
rsync -avz -e "ssh -p 2222" src/ server:/app/   # custom ssh options
```

> rsync transfers only *changed blocks* and resumes cleanly — a re-run after interruption picks up where it left off. **Trailing slash matters:** `src/` copies the *contents* of src; `src` copies the directory itself.

---

## 13. Archiving & Compression

### `tar` — the archive standard

```bash
# Create
tar -czvf app.tar.gz app/          # gzip-compressed archive (.tar.gz)
tar -cjvf app.tar.bz2 app/         # bzip2 (smaller, slower)
tar -cJvf app.tar.xz app/          # xz (smallest, slowest)
tar -czf logs.tar.gz --exclude='*.tmp' /var/log/app/

# Inspect / extract
tar -tzvf app.tar.gz               # LIST contents without extracting (always check first)
tar -xzvf app.tar.gz               # extract here
tar -xzvf app.tar.gz -C /opt/      # extract into a target directory
tar -xzvf app.tar.gz app/config.yml    # extract a single file
```

**Flag memory aid:** `c`reate / e`x`tract / `t` list · `z` gzip / `j` bzip2 / `J` xz · `v` verbose · `f` file (must be last before the filename).

### Standalone compressors

```bash
gzip huge.log            # → huge.log.gz (replaces the original)
gzip -k huge.log         # keep the original
gunzip huge.log.gz       # decompress
zcat huge.log.gz         # read compressed file without extracting
zgrep "ERROR" app.log.gz # grep inside a compressed file ← great for rotated logs
```

### `zip` / `unzip` (cross-platform with Windows)

```bash
zip -r project.zip project/ -x "*/node_modules/*"
unzip project.zip -d /tmp/extracted/
unzip -l project.zip     # list contents
```

---

## 14. Package Management

### Debian / Ubuntu — `apt`

```bash
sudo apt update                  # refresh package index (do this first)
sudo apt upgrade                 # upgrade installed packages
sudo apt install nginx           # install
sudo apt install nginx=1.24.*    # specific version
sudo apt remove nginx            # remove (keeps config)
sudo apt purge nginx             # remove including config files
sudo apt autoremove              # clean orphaned dependencies
apt search redis                 # find packages
apt show nginx                   # package details
apt list --installed | grep node
apt list --upgradable
```

### Low-level Debian — `dpkg`

```bash
sudo dpkg -i package.deb         # install a local .deb
dpkg -l | grep nginx             # is it installed?
dpkg -L nginx                    # what files did this package install?
dpkg -S /usr/sbin/nginx          # which package owns this file?
```

### RHEL / CentOS / Fedora — `dnf` (successor of `yum`)

```bash
sudo dnf install nginx
sudo dnf update
sudo dnf remove nginx
dnf search redis
dnf info nginx
dnf provides /usr/bin/htop       # which package supplies this file
rpm -qa | grep kernel            # low-level query of installed packages
rpm -ql nginx                    # files owned by a package
```

---

## 15. Disk & Filesystem Management

### `mount` / `umount`

```bash
mount                              # show all mounts (or: findmnt for a tree)
sudo mount /dev/sdb1 /mnt/data     # mount a partition
sudo mount -o ro /dev/sdb1 /mnt    # read-only mount
sudo umount /mnt/data              # unmount (fails if "busy")
lsof +D /mnt/data                  # find what's keeping it busy
cat /etc/fstab                     # persistent mounts applied at boot
sudo mount -a                      # mount everything in fstab (test fstab edits!)
```

### Partitioning & filesystems

```bash
lsblk -f                           # devices + filesystems + UUIDs + mountpoints
sudo fdisk -l                      # partition tables
sudo fdisk /dev/sdb                # interactive partitioning (MBR/GPT)
sudo mkfs.ext4 /dev/sdb1           # format as ext4 (DESTROYS data)
sudo mkfs.xfs /dev/sdb1            # XFS (default on RHEL)
blkid                              # UUIDs for fstab entries
sudo fsck /dev/sdb1                # check/repair (unmounted filesystems only!)
sudo e2label /dev/sdb1 data        # label an ext4 filesystem
```

### Swap

```bash
swapon --show                      # current swap
sudo fallocate -l 2G /swapfile && sudo chmod 600 /swapfile
sudo mkswap /swapfile && sudo swapon /swapfile     # create + enable a swap file
```

---

## 16. Services & systemd

### `systemctl` — service control

```bash
sudo systemctl start nginx        # start now
sudo systemctl stop nginx
sudo systemctl restart nginx      # full stop+start
sudo systemctl reload nginx       # re-read config WITHOUT dropping connections
sudo systemctl enable nginx       # start automatically at boot
sudo systemctl enable --now nginx # enable + start in one step
sudo systemctl disable nginx
systemctl status nginx            # state, PID, memory, recent log lines
systemctl is-active nginx         # scripting-friendly: prints active/inactive
systemctl list-units --type=service --state=running
systemctl list-units --failed     # anything broken?
sudo systemctl daemon-reload      # re-read unit files after editing them
```

### Writing a unit file (deploying your own app as a service)

```ini
# /etc/systemd/system/myapp.service
[Unit]
Description=My Node.js App
After=network.target

[Service]
Type=simple
User=appuser
WorkingDirectory=/opt/myapp
ExecStart=/usr/bin/node server.js
Restart=on-failure
RestartSec=5
Environment=NODE_ENV=production

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload && sudo systemctl enable --now myapp
```

### `journalctl` — the systemd log

```bash
journalctl -u nginx               # all logs for one service
journalctl -u nginx -f            # follow live (tail -f for services)
journalctl -u nginx --since "1 hour ago"
journalctl -u nginx --since today -p err    # errors and worse, today
journalctl -b                     # everything since this boot
journalctl -b -1                  # PREVIOUS boot (crash investigations)
journalctl --disk-usage           # how much space logs are taking
sudo journalctl --vacuum-size=500M   # trim old logs
```

---

## 17. Shell, Environment & Redirection

### Environment variables

```bash
env                          # all environment variables
echo $PATH                   # one variable
export API_URL=https://api.x.com     # set for this shell + children
unset API_URL                # remove
NODE_ENV=production node app.js      # set for ONE command only
printenv HOME                # print a specific variable
```

**Startup files:** `~/.bashrc` (interactive shells — aliases, functions), `~/.profile` / `~/.bash_profile` (login shells — PATH, env vars), `/etc/environment` (system-wide). Apply changes with `source ~/.bashrc`.

### `alias` & functions

```bash
alias ll='ls -la'
alias gs='git status'
alias ..='cd ..'
unalias ll
# Functions when you need arguments:
mkcd() { mkdir -p "$1" && cd "$1"; }
```

### Redirection — the core rules

```bash
cmd > out.txt            # stdout → file (overwrite)
cmd >> out.txt           # stdout → file (append)
cmd 2> err.txt           # stderr → file
cmd > all.txt 2>&1       # both → same file (ORDER MATTERS: redirect stdout first)
cmd &> all.txt           # bash shorthand for the same
cmd < input.txt          # file → stdin
cmd 2>/dev/null          # discard errors
cmd > /dev/null 2>&1     # discard everything
cmd1 | cmd2              # stdout of cmd1 → stdin of cmd2
cmd1 2>&1 | cmd2         # pipe stderr too
diff <(sort a.txt) <(sort b.txt)   # process substitution — command output as a "file"
cat <<EOF > config.yml   # heredoc — multi-line content inline
key: value
EOF
```

**File descriptors:** 0 = stdin, 1 = stdout, 2 = stderr. `2>&1` means "point fd2 wherever fd1 currently points" — which is why it must come *after* `> file`.

### Command chaining & exit codes

```bash
cmd1 && cmd2         # run cmd2 only if cmd1 SUCCEEDED (exit 0)
cmd1 || cmd2         # run cmd2 only if cmd1 FAILED
cmd1 ; cmd2          # run both regardless
echo $?              # exit code of the last command (0 = success)
make && make test || echo "build or tests failed"
```

### `history`

```bash
history              # numbered command history
!105                 # re-run command #105
!!                   # re-run last command (sudo !!)
!curl                # re-run last command starting with "curl"
Ctrl+R               # interactive reverse search — the most useful keybinding
history | awk '{print $2}' | sort | uniq -c | sort -rn | head   # your top commands
```

### Quoting rules

```bash
echo "$HOME dir"     # double quotes: variables EXPAND
echo '$HOME dir'     # single quotes: everything literal
echo "It's $(date)"  # command substitution inside double quotes
```

> **Always quote variables** in scripts: `rm "$file"` — unquoted `$file` containing spaces becomes multiple arguments.

---

## 18. Job Scheduling — cron & at

### `crontab`

```bash
crontab -e           # edit your cron jobs
crontab -l           # list them
crontab -r           # remove ALL (careful — no confirmation)
sudo crontab -u alice -l    # another user's crontab
```

**Format:** `minute hour day-of-month month day-of-week command`

```text
┌──────── minute (0-59)
│ ┌────── hour (0-23)
│ │ ┌──── day of month (1-31)
│ │ │ ┌── month (1-12)
│ │ │ │ ┌ day of week (0-7, 0/7 = Sunday)
│ │ │ │ │
* * * * *  command

0 2 * * *        /opt/scripts/backup.sh          # daily at 02:00
*/15 * * * *     /opt/scripts/healthcheck.sh     # every 15 minutes
0 9 * * 1-5      /opt/scripts/report.sh          # weekdays at 09:00
0 0 1 * *        /opt/scripts/monthly.sh         # 1st of each month, midnight
@reboot          /opt/scripts/startup.sh         # once at boot
@daily           /opt/scripts/clean.sh           # shorthand for 0 0 * * *
```

**Cron gotchas (interview favorites):**
- Cron runs with a **minimal environment** — no your `PATH`, no `.bashrc`. Use absolute paths for everything.
- Redirect output or it gets emailed/lost: `0 2 * * * /opt/backup.sh >> /var/log/backup.log 2>&1`
- `%` is special in crontab (newline) — escape as `\%` (bites people using `date +%F`).
- System-wide crons: `/etc/crontab` and `/etc/cron.d/` (these have an extra *user* field).

### `at` — one-time scheduling

```bash
echo "/opt/scripts/report.sh" | at 17:30      # today at 17:30
at now + 2 hours                              # interactive: type commands, Ctrl+D
atq                                           # list pending jobs
atrm 3                                        # cancel job 3
```

---

## 19. Advanced & Troubleshooting Tools

### `lsof` — list open files (everything is a file)

```bash
sudo lsof -i :8080           # what process is using port 8080? ← the classic
sudo lsof -i -P -n           # all network connections, numeric
lsof -p 1234                 # everything a process has open
lsof +D /var/log             # who has files open under this directory
lsof -u alice                # files opened by a user
sudo lsof | grep deleted     # deleted-but-open files (why df says full but du doesn't!)
```

> **"Disk full but I deleted the file!"** — a process still holds the deleted file open, so the space isn't freed. `lsof | grep deleted`, then restart that process.

### `strace` — trace system calls

```bash
strace -f ./myapp                 # every syscall (very verbose)
strace -e open,openat ./myapp     # only file-open calls — "what config is it reading?"
strace -p 1234                    # attach to a running process (why is it stuck?)
strace -c ./myapp                 # syscall count/time summary
```

### `watch` — repeat a command

```bash
watch -n 2 df -h              # re-run every 2 seconds
watch -d free -h              # highlight differences between runs
watch "ss -tn | wc -l"        # live connection count
```

### `tmux` — terminal multiplexer (sessions that survive disconnects)

```bash
tmux new -s deploy           # named session
tmux ls                      # list sessions
tmux attach -t deploy        # reattach
# Inside (prefix = Ctrl+B):  d detach · c new window · n/p next/prev
#                            % split vertical · " split horizontal · arrows to move
```

> Run long jobs on servers inside tmux — an SSH drop no longer kills your work. (`screen` is the older equivalent: `screen -S name`, `Ctrl+A d`, `screen -r`.)

### `ulimit` — per-process resource limits

```bash
ulimit -a                    # all limits
ulimit -n                    # max open files (default 1024 — too low for busy servers)
ulimit -n 65535              # raise for this shell
# Persistent: /etc/security/limits.conf  or  LimitNOFILE= in a systemd unit
```

> "**Too many open files**" (`EMFILE`) under load = raise `nofile` limits. For services, set `LimitNOFILE=65535` in the unit file — limits.conf doesn't apply to systemd services.

### `xxd` / `od` — hex dumps

```bash
xxd file.bin | head          # hex + ASCII view
xxd -l 16 file               # first 16 bytes (check magic numbers)
```

### `date` — timestamps in scripts

```bash
date +%F                     # 2026-07-08
date +%F_%H-%M-%S            # 2026-07-08_14-30-05 (backup filenames)
date +%s                     # unix epoch seconds
date -d @1720000000          # epoch → human
date -d "yesterday" +%F      # relative date math
```

### `basename`, `dirname`, `realpath`

```bash
basename /var/log/app.log        # app.log
basename /var/log/app.log .log   # app (strip extension)
dirname /var/log/app.log         # /var/log
realpath ./../file               # absolute canonical path
```

---

## 20. Practical One-Liner Cheat Sheet

**Top 10 IPs hitting your web server:**
```bash
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head
```

**Find and kill whatever is on port 3000:**
```bash
sudo lsof -ti :3000 | xargs kill -9
```

**Total size of each subdirectory, sorted:**
```bash
du -h --max-depth=1 | sort -hr
```

**Count occurrences of each HTTP status code:**
```bash
awk '{print $9}' access.log | sort | uniq -c | sort -rn
```

**Recursively replace a string across a project:**
```bash
grep -rl "oldname" src/ | xargs sed -i 's/oldname/newname/g'
```

**Files modified in the last 10 minutes (what did the deploy touch?):**
```bash
find /opt/app -mmin -10 -type f
```

**Watch a log for errors, live:**
```bash
tail -F app.log | grep --line-buffered -i error
```

**Delete logs older than 30 days:**
```bash
find /var/log/app -name "*.log" -mtime +30 -delete
```

**Which processes are eating memory:**
```bash
ps aux --sort=-%mem | awk 'NR<=11 {print $2, $4"%", $11}'
```

**Test if a remote port is reachable (no netcat needed — pure bash):**
```bash
timeout 2 bash -c '</dev/tcp/db-host/5432' && echo open || echo closed
```

**Dedupe a file while preserving order:**
```bash
awk '!seen[$0]++' file.txt
```

**Backup with a dated filename:**
```bash
tar -czf "backup_$(date +%F).tar.gz" /opt/app
```

**Show the 20 largest files under the current directory:**
```bash
find . -type f -exec du -h {} + | sort -hr | head -20
```

**Follow all logs of a service since the last deploy:**
```bash
journalctl -u myapp --since "10 min ago" -f
```

**Quick HTTP server from any directory (share files):**
```bash
python3 -m http.server 8000
```

---

## Quick Interview Recall Table

| Task | Command |
|---|---|
| What's listening on port X? | `ss -tulpn` or `sudo lsof -i :X` |
| Disk full — where? | `df -h`, then `du -h --max-depth=1 / \| sort -hr` |
| Disk full but files deleted? | `lsof \| grep deleted` → restart holder |
| Process disappeared randomly? | `dmesg -T \| grep -i oom` |
| Follow logs live | `tail -F file` / `journalctl -u svc -f` |
| Find big/old files | `find / -size +100M` / `find . -mtime +30` |
| Frequency count of anything | `sort \| uniq -c \| sort -rn` |
| Replace text in many files | `grep -rl pat \| xargs sed -i 's/pat/new/g'` |
| Keep a job alive after logout | `nohup cmd &` / `tmux` / `disown` |
| Service management | `systemctl status/restart/enable --now` |
| Copy to server (resumable) | `rsync -avz --progress src/ host:/dest/` |
| Test port reachability | `nc -zv host port` |
| DNS debug | `dig +short name` / `dig @8.8.8.8 name` |
| Who's using a file/dir? | `lsof +D /path` / `fuser -v /path` |
| Safe recursive perms | `chmod -R g+rX dir/` (capital X) |

---

## 21. Bash Scripting Essentials

### Script skeleton — start every script like this

```bash
#!/usr/bin/env bash
set -euo pipefail
# -e : exit immediately on any command failure
# -u : error on undefined variables (catches typos)
# -o pipefail : a pipeline fails if ANY command in it fails (not just the last)

IFS=$'\n\t'   # safer word-splitting (optional but recommended)
```

> Without `pipefail`, `curl bad-url | grep ok` "succeeds" because grep's exit code masks curl's failure — the single most common silent-failure bug in shell scripts.

### Variables

```bash
name="alice"                 # NO spaces around = (name = "alice" is an error)
echo "$name"                 # always quote when using
readonly API_URL="https://x" # constant
count=$(( 3 + 4 ))           # arithmetic
files=$(ls /tmp)             # command substitution — $(...) not backticks

# Parameter expansion — the interview favorites
echo "${name:-default}"      # value, or "default" if unset/empty
echo "${name:?must be set}"  # abort with error if unset (guard clauses)
echo "${#name}"              # string length
echo "${file%.log}"          # strip shortest suffix:  app.log → app
echo "${file##*/}"           # strip longest prefix:  /var/log/app.log → app.log (basename)
echo "${path%/*}"            # dirname equivalent
echo "${name/alice/bob}"     # replace first match
echo "${name//a/A}"          # replace all matches
echo "${name^^}" "${name,,}" # uppercase / lowercase (bash 4+)
```

### Script arguments

```bash
$0        # script name
$1 $2     # positional arguments
$#        # number of arguments
"$@"      # all arguments, individually quoted ← use this, not $*
$?        # exit code of last command
$$        # this script's PID

# Standard argument validation
if [[ $# -lt 2 ]]; then
  echo "Usage: $0 <source> <dest>" >&2
  exit 1
fi

# Named flags with getopts
while getopts "f:vh" opt; do
  case $opt in
    f) file="$OPTARG" ;;
    v) verbose=1 ;;
    h) usage; exit 0 ;;
    *) exit 1 ;;
  esac
done
```

### Conditionals

```bash
if [[ -f "$file" ]]; then echo "file exists"; fi

# File tests
[[ -f path ]]   # regular file exists
[[ -d path ]]   # directory exists
[[ -e path ]]   # anything exists
[[ -s path ]]   # exists AND is non-empty
[[ -r path ]] [[ -w path ]] [[ -x path ]]   # readable / writable / executable
[[ f1 -nt f2 ]] # f1 newer than f2

# Strings
[[ -z "$s" ]]         # empty
[[ -n "$s" ]]         # non-empty
[[ "$a" == "$b" ]]    # equal
[[ "$s" == prod* ]]   # glob match
[[ "$s" =~ ^[0-9]+$ ]] # regex match (BASH_REMATCH holds groups)

# Numbers — different operators!
[[ "$n" -eq 5 ]]  # equal        (also -ne -lt -le -gt -ge)
(( n > 5 ))       # arithmetic context — more natural for math

# Combining
[[ -f "$f" && -r "$f" ]] || { echo "missing/unreadable" >&2; exit 1; }

# case — cleaner than if/elif chains
case "$ENV" in
  prod|production) deploy_prod ;;
  staging)         deploy_staging ;;
  *)               echo "unknown env: $ENV" >&2; exit 1 ;;
esac
```

> Prefer `[[ ]]` over `[ ]` — it's a bash keyword (not a program), doesn't word-split unquoted variables, and supports `&&`, glob and regex matching.

### Loops

```bash
for f in *.log; do gzip "$f"; done            # glob loop
for i in {1..10}; do echo "$i"; done          # range
for ((i=0; i<5; i++)); do echo "$i"; done     # C-style

# Read a file line by line — THE correct pattern
while IFS= read -r line; do
  echo "processing: $line"
done < input.txt
# (for line in $(cat file) breaks on spaces and globs — never do it)

# Loop over find results safely (handles spaces/newlines in names)
find . -name "*.tmp" -print0 | while IFS= read -r -d '' f; do
  rm "$f"
done

while true; do check_health; sleep 30; done   # poll loop
until ping -c1 -W1 db-host &>/dev/null; do sleep 2; done   # wait for dependency
break / continue                               # loop control
```

### Functions

```bash
log() {
  echo "[$(date +%T)] $*" >&2      # log to stderr, keep stdout for data
}

get_user() {
  local id="$1"                    # local = don't pollute global scope
  local result
  result=$(curl -s "https://api/users/$id") || return 1
  echo "$result"                   # "return value" = stdout
}

user_json=$(get_user 42) || { log "lookup failed"; exit 1; }
```

### Error handling & cleanup

```bash
# trap — run cleanup no matter how the script exits
tmpdir=$(mktemp -d)
trap 'rm -rf "$tmpdir"' EXIT           # fires on normal exit, error, Ctrl+C

trap 'echo "failed at line $LINENO" >&2' ERR   # report the failing line

# Allow a command to fail without killing a set -e script
count=$(grep -c ERROR app.log || true)

# Retry with backoff
retry() {
  local n=0 max=5
  until "$@"; do
    n=$((n+1)); (( n >= max )) && return 1
    sleep $((2**n))
  done
}
retry curl -sf https://api.example.com/health
```

### Arrays & associative arrays

```bash
servers=("web1" "web2" "db1")
echo "${servers[0]}"          # first element
echo "${servers[@]}"          # all elements
echo "${#servers[@]}"         # count
servers+=("cache1")           # append
for s in "${servers[@]}"; do ssh "$s" uptime; done

declare -A ports=([http]=80 [https]=443 [ssh]=22)   # associative (bash 4+)
echo "${ports[https]}"
for k in "${!ports[@]}"; do echo "$k → ${ports[$k]}"; done
```

### Practical script template

```bash
#!/usr/bin/env bash
set -euo pipefail

readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly LOG_FILE="/var/log/myscript.log"

log()  { echo "[$(date '+%F %T')] $*" | tee -a "$LOG_FILE" >&2; }
die()  { log "FATAL: $*"; exit 1; }

usage() { echo "Usage: $0 -e <env> [-v]"; }

main() {
  local env="" verbose=0
  while getopts "e:vh" opt; do
    case $opt in
      e) env="$OPTARG" ;;
      v) verbose=1 ;;
      h) usage; exit 0 ;;
      *) usage; exit 1 ;;
    esac
  done
  [[ -n "$env" ]] || die "environment required (-e)"

  log "deploying to $env"
  # ... work ...
  log "done"
}

main "$@"
```

> Lint every script with **shellcheck** (`shellcheck deploy.sh`) — it catches quoting bugs, word-splitting, and `set -e` pitfalls automatically. Non-negotiable in CI.

---

## 22. Vim Quick Reference

You *will* end up in vim on a server (`git commit`, `crontab -e`, `visudo`). Minimum survival plus useful fluency:

### Modes

```text
Normal (default)  → navigate & operate       Esc returns here from anywhere
Insert            → type text                 i (insert), a (append), o (new line below)
Visual            → select                    v (char), V (line), Ctrl+v (block)
Command           → ex commands               : (e.g., :wq)
```

### Survival essentials

```text
:w          save                    :q          quit
:wq or ZZ   save and quit           :q!         quit WITHOUT saving
u           undo                    Ctrl+r      redo
```

### Movement

```text
h j k l     ← ↓ ↑ →                 w / b       next / previous word
0 / $       start / end of line     gg / G      top / bottom of file
42G  :42    go to line 42           { / }       previous / next paragraph
Ctrl+d/u    half-page down / up     %           jump to matching bracket
```

### Editing (operator + motion = vim's grammar)

```text
dd          delete line             dw          delete word
d$  / D     delete to end of line   x           delete character
yy          copy (yank) line        yw          yank word
p / P       paste after / before    cw          change word (delete + insert)
ciw         change inner word       ci"         change inside quotes
di(         delete inside parens    .           REPEAT last change (huge)
3dd         delete 3 lines          >> / <<     indent / outdent
r<char>     replace one char        ~           toggle case
```

### Search & replace

```text
/pattern    search forward          ?pattern    search backward
n / N       next / previous match   *           search word under cursor
:%s/old/new/g       replace in whole file
:%s/old/new/gc      with confirmation per match
:10,20s/old/new/g   only lines 10–20
:noh                clear search highlighting
```

### Files & buffers

```text
:e file     open file               :sp / :vsp  horizontal / vertical split
Ctrl+w w    cycle between splits    :set nu     line numbers
:set paste  paste without auto-indent mangling (then :set nopaste)
vim +42 f   open f at line 42       vimdiff a b side-by-side diff
```

---

## 23. Firewall & Security Hardening

### `ufw` — Ubuntu's simple firewall

```bash
sudo ufw status verbose            # current state & rules
sudo ufw allow 22/tcp              # ALWAYS allow SSH before enabling!
sudo ufw allow 80,443/tcp          # web ports
sudo ufw allow from 10.0.0.0/8 to any port 5432   # DB only from internal net
sudo ufw deny 23                   # explicit deny
sudo ufw limit ssh                 # rate-limit brute force on SSH
sudo ufw enable                    # activate (after allowing SSH!)
sudo ufw delete allow 8080         # remove a rule
sudo ufw default deny incoming     # default posture
sudo ufw default allow outgoing
```

### `firewalld` — RHEL/CentOS/Fedora

```bash
sudo firewall-cmd --state
sudo firewall-cmd --list-all                     # active zone's rules
sudo firewall-cmd --add-service=https --permanent
sudo firewall-cmd --add-port=8080/tcp --permanent
sudo firewall-cmd --reload                       # apply permanent rules
sudo firewall-cmd --remove-port=8080/tcp --permanent
```

### `iptables` — the underlying packet filter (know the concepts)

```bash
sudo iptables -L -n -v --line-numbers    # list rules with counters
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT      # append rule
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A INPUT -j DROP                          # default drop (LAST rule)
sudo iptables -D INPUT 3                                # delete rule #3
sudo iptables-save > /etc/iptables/rules.v4             # persist
```

**Chains:** `INPUT` (to this host), `OUTPUT` (from this host), `FORWARD` (routed through). Rules match top-down; first match wins. Modern kernels use **nftables** underneath; `ufw`/`firewalld` are front-ends.

### SSH hardening (`/etc/ssh/sshd_config`)

```text
PermitRootLogin no             # never log in directly as root
PasswordAuthentication no      # keys only — kills password brute force
Port 22                        # (changing port ≠ security, just less log noise)
AllowUsers deploy alice        # explicit allowlist
MaxAuthTries 3
ClientAliveInterval 300
```

```bash
sudo sshd -t                       # validate config BEFORE restarting
sudo systemctl reload sshd         # apply (keep your current session open until verified!)
```

### fail2ban — auto-ban brute forcers

```bash
sudo apt install fail2ban
sudo fail2ban-client status sshd      # current bans for the ssh jail
sudo fail2ban-client set sshd unbanip 1.2.3.4
# config: /etc/fail2ban/jail.local — bantime, findtime, maxretry
```

### Security auditing basics

```bash
sudo find / -perm -4000 -type f 2>/dev/null   # setuid binaries — should be a short, known list
sudo ss -tulpn                                # anything listening you don't recognize?
last -20 ; sudo lastb | head                  # recent logins / FAILED login attempts
sudo grep "Failed password" /var/log/auth.log | awk '{print $(NF-3)}' | sort | uniq -c | sort -rn | head
                                              # top brute-force source IPs
sudo getenforce                               # SELinux mode (RHEL): Enforcing/Permissive
sudo ausearch -m avc -ts recent               # recent SELinux denials
```

---

## 24. Log Management & logrotate

### Where logs live

| File | Contents |
|---|---|
| `/var/log/syslog` (Debian) / `/var/log/messages` (RHEL) | General system log |
| `/var/log/auth.log` / `/var/log/secure` | Logins, sudo, SSH attempts |
| `/var/log/kern.log` | Kernel messages |
| `/var/log/dmesg` | Boot-time hardware messages |
| `/var/log/nginx/`, `/var/log/apache2/` | Web server access/error logs |
| `journalctl` | Everything systemd manages (binary journal) |

### Reading logs effectively

```bash
sudo tail -F /var/log/syslog                      # live follow
sudo grep -i "error\|fail\|critical" /var/log/syslog | tail -50
zgrep "OOM" /var/log/syslog.*.gz                  # search rotated (compressed) logs
journalctl -p err -b                              # all errors this boot
journalctl -u nginx --since "2026-07-08 09:00" --until "2026-07-08 10:00"
journalctl _PID=1234                              # logs from one PID
```

### logrotate — why your disk doesn't fill up (usually)

Config: `/etc/logrotate.conf` + per-app drop-ins in `/etc/logrotate.d/`.

```text
# /etc/logrotate.d/myapp
/var/log/myapp/*.log {
    daily                  # rotate daily (or weekly/monthly/size 100M)
    rotate 14              # keep 14 old copies
    compress               # gzip old logs
    delaycompress          # keep the most recent rotation uncompressed
    missingok              # no error if the log is missing
    notifempty             # skip if empty
    create 0640 appuser appgroup    # recreate the file with these perms
    postrotate
        systemctl kill -s USR1 myapp    # tell the app to reopen its log file
    endscript
}
```

```bash
sudo logrotate -d /etc/logrotate.d/myapp    # dry run — show what WOULD happen
sudo logrotate -f /etc/logrotate.d/myapp    # force a rotation now (testing)
```

> **The classic bug:** the app keeps writing to the *rotated* file (same open file descriptor), so the "new" log stays empty and the old one keeps growing invisibly. That's what `postrotate` signals (or `copytruncate`, with a small data-loss window) are for — and why `tail -F` beats `tail -f`.

---

## 25. Performance Troubleshooting Playbook

A repeatable triage sequence when someone says **"the server is slow"**:

### Step 0 — 60-second overview

```bash
uptime                # load averages vs. core count (nproc)
dmesg -T | tail -20   # kernel complaints? OOM kills? disk errors?
vmstat 1 5            # r (runnable) > cores = CPU pressure; si/so > 0 = swapping
```

### Step 1 — CPU?

```bash
top                   # press 1: any single core pinned? high %us (app) vs %sy (kernel)?
                      # high %wa = NOT CPU — the CPU is idle waiting on disk I/O
ps aux --sort=-%cpu | head
pidstat 1             # per-process CPU over time (sysstat)
```

### Step 2 — Memory?

```bash
free -h                        # check 'available', not 'free'
ps aux --sort=-%mem | head
dmesg -T | grep -i oom         # has the OOM killer struck?
vmstat 1                       # si/so columns: any swapping = severe pressure
```

### Step 3 — Disk I/O?

```bash
iostat -x 2            # %util near 100 = saturated device; high await = slow responses
iotop                  # WHICH process is doing the I/O
df -h ; df -i          # full filesystem or exhausted inodes also "feels slow"
```

### Step 4 — Network?

```bash
ss -s                        # socket summary — huge timewait/synrecv counts?
ss -tn state established | wc -l    # connection count vs. expectations
ping -c5 gateway ; mtr host  # latency/loss on the path
ip -s link                   # interface errors/drops
```

### Step 5 — The application itself

```bash
journalctl -u myapp --since "15 min ago" -p warning
strace -c -p <PID>           # where are its syscalls spending time?
lsof -p <PID> | wc -l        # fd leak? compare against ulimit -n
cat /proc/<PID>/status | grep -i threads   # thread explosion?
```

**Mental model — USE method:** for every resource (CPU, memory, disk, network) check **U**tilization, **S**aturation, **E**rrors. Load average high but CPU idle → almost always disk I/O wait (`%wa` in top) or processes stuck in `D` state (`ps aux | awk '$8 ~ /D/'`).

---

## 26. Scenario-Based Interview Questions

### S1. Disk shows 100% full, but `du` can't find the space. What's happening?

A process holds a **deleted file open** — the directory entry is gone (invisible to `du`) but the inode and blocks are retained until the last file descriptor closes (still counted by `df`).

```bash
sudo lsof | grep deleted        # find the holder (often a log file)
sudo systemctl restart <that-service>    # or copy /dev/null onto it:
sudo truncate -s 0 /proc/<PID>/fd/<FD>   # reclaim without restart
```

### S2. You ran a deploy over SSH and your connection dropped mid-way. How do you prevent this next time, and what happened to the running process?

The shell's children receive **SIGHUP** on disconnect and die (unless the job ignores it). Prevention: run inside `tmux`/`screen`, or `nohup ./deploy.sh &`, or `disown` after backgrounding. Recovery check: `ps aux | grep deploy` to see if anything survived, and design deploys to be idempotent/resumable.

### S3. A service fails to start. Walk through your debugging sequence.

```bash
systemctl status myapp            # exit code + last log lines
journalctl -u myapp -n 100        # full recent logs
sudo -u appuser /usr/bin/node /opt/app/server.js    # run it manually AS the service user
ss -tulpn | grep 8080             # port already taken?
ls -l /opt/app ; id appuser       # permission/ownership problems?
df -h ; free -h                   # resource exhaustion?
systemd-analyze verify myapp.service   # unit file syntax
```
Most common causes: port conflict, wrong permissions on files/dirs the service user can't read, missing env vars (systemd doesn't load `.bashrc`), and bad `WorkingDirectory`.

### S4. `chmod 777` fixed a permission problem. Why is that the wrong fix, and what's the right one?

777 gives every user on the system write+execute — any compromised account can now modify the files (web shells love this). Right fix: identify *which* user needs *what* access, then set ownership and minimal mode:

```bash
sudo chown -R www-data:www-data /var/www/app
sudo find /var/www/app -type d -exec chmod 755 {} +
sudo find /var/www/app -type f -exec chmod 644 {} +
```

### S5. You need to find which of 200 log files across a directory tree contains a specific request ID, then extract 5 lines of context around it.

```bash
grep -rl "req-8f3a2b" /var/log/services/            # which files
grep -rn -C 5 "req-8f3a2b" /var/log/services/       # with context
zgrep -C 5 "req-8f3a2b" /var/log/services/**/*.gz   # rotated logs too
```

### S6. Load average is 30 on an 8-core box, but CPU usage is only 10%. Explain.

Load counts **runnable + uninterruptible (`D` state)** processes. Low CPU + high load = processes blocked on I/O (disk/NFS). Confirm:

```bash
ps aux | awk '$8 ~ /^D/'      # who's stuck in D state
iostat -x 2                   # which device is saturated (%util, await)
```
Typical culprits: dying disk, saturated volume, hung NFS mount.

### S7. How do you run a command at 2 AM every Sunday, and how do you debug it when it "works manually but not in cron"?

```bash
crontab -e
0 2 * * 0 /opt/scripts/backup.sh >> /var/log/backup.log 2>&1
```
Debug checklist: cron's environment is minimal — **no PATH from your shell, no `.bashrc`**. Use absolute paths for every binary and file, set needed env vars in the script itself, capture output (as above), and check execution actually happened: `grep CRON /var/log/syslog`.

### S8. A teammate accidentally ran `rm -rf` on an application directory. What do you do, and what should have prevented it?

Immediate: stop writes to that filesystem (deleted-but-open files may still be recoverable via `/proc/<PID>/fd` if the app is still running — copy them out *before* restarting anything). Realistic recovery = restore from backup; ext4 undelete tooling (`extundelete`/`testdisk`) is a last resort with poor odds on a busy disk. Prevention: backups with tested restores, deploys owned by a non-human user with least privilege, `rm -rf "${VAR:?}"/` guards in scripts, and immutable/versioned artifacts (rebuild instead of restore).

### S9. Port 443 works from the server itself but not from outside. Diagnose.

```bash
ss -tulpn | grep 443                 # 1. Is it listening — and on what address?
                                     #    127.0.0.1:443 = localhost-only bind → fix app config to 0.0.0.0
sudo ufw status / firewall-cmd --list-all / iptables -L -n   # 2. host firewall?
                                     # 3. cloud security group / NACL (AWS/Azure/GCP)?
curl -v https://localhost ; curl -v https://<public-ip>       # compare behavior
sudo tcpdump -i any port 443 -n     # 4. do packets even ARRIVE? (if not: upstream firewall/SG)
```
Order of likelihood: bind address → security group → host firewall.

### S10. You must transfer a 50 GB database dump to another server on a flaky connection. Best approach?

```bash
rsync -avz --partial --progress dump.sql.gz user@dest:/data/
# --partial keeps incomplete transfer chunks; rerunning resumes instead of restarting
# Alternative: split + checksum
split -b 1G dump.sql.gz part_ && sha256sum part_* > sums
# ... transfer parts, then on the destination:
sha256sum -c sums && cat part_* > dump.sql.gz
```
Never plain `scp` for this — one drop at 49 GB restarts from zero.

### S11. How would you find and stop a runaway process that's forking endlessly (fork bomb)?

```bash
ps aux --sort=-%cpu | head ; pstree -p | less    # identify the tree
kill -STOP -<PGID>     # freeze the whole process GROUP first (negative PID = group)
                       # STOP can't be caught and stops the forking race
kill -KILL -- -<PGID>  # then kill the frozen group
```
Prevention: per-user process limits — `ulimit -u 4096` / `nproc` limits in `/etc/security/limits.conf`, and cgroup limits (`TasksMax=` in systemd units).

### S12. Explain what happens, step by step, when you type `ls -l | grep ".log" > out.txt` and press Enter.

1. Bash **parses** the line: a two-command pipeline with output redirection.
2. Bash creates a **pipe** (kernel buffer with read/write ends).
3. Bash **forks** twice: child 1 will run `ls`, child 2 `grep`.
4. In child 1, stdout (fd 1) is **dup2'd** onto the pipe's write end; in child 2, stdin (fd 0) onto the read end. In child 2, fd 1 is redirected to `out.txt` (created/truncated by the shell via `open()` **before** exec — which is why `> file` empties the file even if the command later fails).
5. Each child calls **execve()** — `ls` and `grep` replace the shell copies. `PATH` lookup finds the binaries.
6. Both run **concurrently**; the pipe applies backpressure (writer blocks when the buffer is full).
7. `ls` exits → pipe write-end closes → `grep` reads EOF and exits → bash **wait()**s, collects exit codes; `$?` is grep's code (or see `PIPESTATUS[@]` for both).

---

## 27. Conceptual Interview Questions & Answers

### Q1. Explain the Linux boot process, step by step.

1. **BIOS/UEFI** — firmware runs POST (hardware checks), locates the boot device.
2. **Bootloader (GRUB2)** — loads the selected kernel and initramfs into memory; kernel parameters (e.g., `root=`, `single`) are passed here.
3. **Kernel** — decompresses itself, initializes hardware drivers, mounts the initramfs (temporary root with just enough drivers to reach the real root filesystem), then mounts the real root FS.
4. **`init` process (PID 1)** — the kernel starts systemd (modern distros), the ancestor of every other process.
5. **systemd targets** — units start in dependency order up to the default target (`multi-user.target` for servers, `graphical.target` for desktops).

```bash
systemd-analyze          # total boot time
systemd-analyze blame    # slowest units during boot
journalctl -b            # everything logged this boot
```

### Q2. What is an inode? What information does it store — and what does it NOT store?

An inode is the on-disk metadata structure for a file: **permissions, owner/group, size, timestamps (atime/mtime/ctime), link count, and pointers to the data blocks**. It does **not** store the filename — names live in *directory entries*, which map `name → inode number`. This is why:
- Hard links are possible (multiple names → same inode).
- Renaming is instant (only the directory entry changes).
- A filesystem can run out of inodes while having free space (`df -i` vs `df -h`) — typically caused by millions of tiny files.

### Q3. Zombie vs. orphan processes — what's the difference, and are zombies harmful?

- **Zombie (`Z` state):** a process that has *exited*, but its parent hasn't called `wait()` to read its exit status. It's just a leftover process-table entry — it consumes no CPU or memory, only a PID slot. You cannot kill a zombie (it's already dead); fix the **parent** (or kill the parent so init/systemd adopts and reaps the zombie).
- **Orphan:** a *running* process whose parent died. It gets re-parented to PID 1 and continues normally — harmless by design (this is how daemons traditionally detached).

```bash
ps aux | awk '$8 ~ /Z/'     # find zombies
ps -o ppid= -p <zombie_pid> # find the negligent parent
```
Danger sign: *thousands* of zombies → the parent has a bug (never reaps children) and you'll eventually exhaust the PID space.

### Q4. What's the difference between a process and a thread in Linux?

Both are represented by the same kernel object (`task_struct`) — Linux creates them with the same syscall (`clone()`), differing only in *what is shared*. A **thread** shares the address space (memory), file descriptors, and signal handlers with its siblings; a **process** gets copies (via copy-on-write from `fork()`). Consequences: threads communicate through shared memory (fast, needs locking); processes need IPC (pipes, sockets, shared memory segments) but are isolated — one crashing doesn't corrupt the others.

```bash
ps -eLf | wc -l              # count all threads system-wide
cat /proc/<PID>/status | grep Threads
top -H -p <PID>              # per-thread view of one process
```

### Q5. Explain file descriptors and what `2>&1` really means.

Every process has a table of open file descriptors — small integers indexing into open files/sockets/pipes. By convention: **0 = stdin, 1 = stdout, 2 = stderr**. Redirection manipulates this table via `dup2()`:

```bash
cmd > f 2>&1     # 1) fd1 → f     2) fd2 → copy of fd1 (also f)      ✓ both in f
cmd 2>&1 > f     # 1) fd2 → wherever fd1 points NOW (terminal)  2) fd1 → f
                 # ✗ stderr still hits the terminal — ORDER MATTERS
```

Everything is a file: sockets, pipes, devices, even `ls /proc/self/fd` shows your shell's descriptor table. "Too many open files" = the per-process fd limit (`ulimit -n`) is exhausted.

### Q6. What is the difference between `su`, `su -`, `sudo`, and `sudo -i`?

| Command | Identity | Environment |
|---|---|---|
| `su alice` | switch user | **keeps your current** env/cwd — half-switched, causes subtle bugs |
| `su - alice` | switch user | full login: alice's env, PATH, home, cwd |
| `sudo cmd` | run ONE command as root | your env (filtered by sudoers `env_reset`) |
| `sudo -i` | root login shell | root's environment, like logging in as root |

`sudo` is preferred over a root password: per-user grants, full audit trail in `auth.log`, no shared secret, fine-grained command allowlists in `/etc/sudoers`.

### Q7. What are runlevels / systemd targets?

Legacy SysV **runlevels** (0 halt, 1 single-user, 3 multi-user, 5 graphical, 6 reboot) are replaced by systemd **targets** — named groups of units:

```bash
systemctl get-default                     # e.g., multi-user.target
sudo systemctl set-default graphical.target
sudo systemctl isolate rescue.target      # like dropping to runlevel 1 (maintenance)
```

`rescue.target` = single-user with basic services; `emergency.target` = root shell, almost nothing mounted — for fixing a broken `/etc/fstab`.

### Q8. Explain swap. When is swapping fine and when is it a problem?

Swap is disk space used as overflow for RAM. The kernel also *proactively* swaps out long-idle pages to keep more RAM available for page cache — so **used swap by itself is not a problem**. The problem is **active swapping (thrashing)**: constant page-in/page-out because the working set exceeds RAM.

```bash
free -h                  # swap used — static picture, not alarming alone
vmstat 1                 # si/so columns — SUSTAINED nonzero = real memory pressure
sysctl vm.swappiness     # 0–100: kernel's eagerness to swap (default 60;
                         # databases often set 1–10)
```

### Q9. What is the sticky bit, setuid, and setgid? Give a real example of each.

- **setuid** (`chmod u+s`): executable runs with the *file owner's* privileges. Example: `/usr/bin/passwd` is root-owned setuid — ordinary users can update `/etc/shadow` through it, in a controlled way.
- **setgid on a directory** (`chmod g+s`): new files inherit the directory's *group* instead of the creator's primary group. Example: a shared `/srv/projects` dir so every team member's files stay group-accessible.
- **sticky bit on a directory** (`chmod +t`): users may delete only files they own. Example: `/tmp` (mode 1777) — world-writable, but you can't delete other users' files.

Security note: audit setuid binaries (`find / -perm -4000`) — each one is potential privilege escalation.

### Q10. `/etc/passwd` vs `/etc/shadow` — why two files?

Historically hashes lived in world-readable `/etc/passwd` (many programs need to map UID↔username, so it must stay readable). Offline cracking made that untenable, so hashes moved to root-only `/etc/shadow`, which also carries password-aging fields (last change, max age, expiry). The `x` in a passwd entry means "look in shadow." Verify: `ls -l /etc/shadow` → `-rw-r-----  root shadow`.

### Q11. What happens exactly when you run `kill <PID>`? Why can a process ignore it?

`kill` sends a **signal** (default SIGTERM/15) — an asynchronous notification delivered by the kernel. The process may have installed a **handler** (to clean up and exit gracefully), may **ignore** it, or take the default action (terminate). Two signals can't be caught or ignored: **SIGKILL (9)** and **SIGSTOP (19)** — enforced by the kernel itself. Even SIGKILL fails on a process in uninterruptible sleep (`D` state) until the blocking I/O syscall returns — which is why a process stuck on a dead NFS mount survives `kill -9`.

```bash
kill -TERM 123   # polite request → handler runs → graceful exit
kill -KILL 123   # kernel destroys it → no cleanup, no flushing, locks left behind
```

### Q12. Explain the page cache. Why does `free -h` show almost no free memory on a healthy server?

Linux uses idle RAM to cache recently-read disk blocks (the **page cache**) — free RAM is wasted RAM. This memory is instantly reclaimable when applications need it, which is why the `available` column, not `free`, reflects real capacity. This also explains why the *second* read of a big file is dramatically faster, and why `sync; echo 3 > /proc/sys/vm/drop_caches` (benchmarking only!) makes things slower, not faster.

### Q13. Hard link vs. symlink — what happens to each when the target is deleted?

Deleting a file removes one *name* (directory entry) and decrements the inode's link count; **data is freed only when the count hits zero and no process holds it open**. A **hard link** is a peer name — the file survives deletion of the "original." A **symlink** stores a *path*; deleting the target leaves the symlink **dangling** (points to nothing; `ls` shows it red, opening fails with ENOENT). Find broken symlinks: `find . -xtype l`.

### Q14. What is `/proc`? Give practical examples of using it.

A **virtual filesystem** — files are generated on-the-fly by the kernel, representing live kernel and process state (they occupy no disk).

```bash
cat /proc/cpuinfo /proc/meminfo      # hardware/memory facts
cat /proc/loadavg                     # the source of 'uptime' numbers
ls /proc/1234/fd                      # a process's open file descriptors
cat /proc/1234/environ | tr '\0' '\n' # its environment variables
cat /proc/1234/limits                 # its effective ulimits
readlink /proc/1234/exe               # the actual binary it's running
cat /proc/sys/net/ipv4/ip_forward     # kernel tunables (sysctl reads here)
```

Recovery trick: if a running process's binary/log was deleted, `/proc/<PID>/exe` and `/proc/<PID>/fd/N` still reference the data — copy them out before the process exits.

### Q15. What is umask 022 vs 077 — and why don't new files ever get execute permission?

`umask` *subtracts* permission bits from the creation default. Files are created with base `666` (never `777` — the kernel/libc convention deliberately withholds `x` so nothing becomes executable by accident; you must `chmod +x` explicitly). Directories start from `777`.

- `umask 022` → files `644`, dirs `755` (world-readable — typical default)
- `umask 077` → files `600`, dirs `700` (private — hardened servers, shared boxes)

### Q16. Difference between `apt update`, `apt upgrade`, and `apt dist-upgrade`/`full-upgrade`?

- `update` — refreshes the **package index** only (what versions exist). Changes nothing installed.
- `upgrade` — installs newer versions of installed packages, but **never removes** packages or installs new dependencies.
- `full-upgrade` (`dist-upgrade`) — upgrades and **may add/remove** packages to resolve changed dependencies — needed for kernel jumps and release upgrades. Always `update` before either; the index is what makes `upgrade` see anything.

### Q17. How does DNS resolution work on a Linux box, in order?

1. **`nsswitch.conf`** (`hosts:` line) defines the order — typically `files dns`.
2. **`/etc/hosts`** — static overrides, checked first.
3. **Stub resolver** sends the query to the nameserver in **`/etc/resolv.conf`** — on modern Ubuntu that's `systemd-resolved` at `127.0.0.53`, which caches and forwards upstream.
4. Upstream **recursive resolver** walks root → TLD → authoritative servers, caches by TTL.

```bash
resolvectl status            # what systemd-resolved is actually using
getent hosts example.com     # resolve the way libc does (respects nsswitch)
dig example.com              # bypasses nsswitch — talks straight to DNS
```
Debug insight: `dig` works but the app fails → the problem is in `/etc/hosts`/nsswitch/resolved, not DNS itself.

### Q18. What are cgroups and namespaces? Why do they matter?

They are the two kernel primitives behind **containers**:
- **Namespaces** = *isolation of visibility* — separate views of PIDs, mounts, network stacks, hostnames, users. A container's PID 1 is just a normal process in an unshared PID namespace.
- **cgroups** = *resource limits & accounting* — cap CPU, memory, I/O, and process counts per group; the OOM killer can act per-cgroup.

systemd puts every service in its own cgroup — that's how `MemoryMax=512M` or `TasksMax=` in a unit file works, and how `systemctl status` knows exactly which processes belong to a service. A "container" is nothing more than namespaces + cgroups + a filesystem image — there is no VM involved.

### Q19. Explain exit codes. What do 0, 1, 126, 127, 137 conventionally mean?

| Code | Meaning |
|---|---|
| 0 | success (the ONLY success value) |
| 1 | generic failure |
| 2 | shell builtin misuse / bad usage |
| 126 | found but **not executable** (permissions) |
| 127 | **command not found** (PATH problem) |
| 128+N | killed by signal N → **137** = 128+9 = SIGKILL (OOM killer's signature!), **143** = 128+15 = SIGTERM |

Seeing exit 137 from a container/CI job almost always means it was OOM-killed — check `dmesg` / cgroup memory limits, not the application logs.

### Q20. `curl` says "connection refused" vs "timeout" — what does each tell you?

- **Connection refused** — the packet *reached* the host, and the kernel answered with TCP RST: **nothing is listening** on that port (service down, wrong port, or listening only on 127.0.0.1). The network path is fine.
- **Timeout** — packets vanish: a **firewall silently drops** them (security group, iptables DROP), routing is broken, or the host is down.

This single distinction cuts a network triage in half: refused → look at the service and its bind address; timeout → look at firewalls and the path (`tcpdump` to confirm whether packets arrive).

---

## 28. Advanced Scenario-Based Questions

### S13. Every few hours, your Node.js service on a VM restarts by itself. No one is touching it. Find out why.

```bash
journalctl -u myapp | grep -iE "killed|oom|signal|start"   # restart timeline
dmesg -T | grep -i "out of memory"                          # kernel OOM kills
systemctl show myapp -p Restart,RestartSec                  # is systemd auto-restarting?
```
Typical finding: `Out of memory: Killed process (node)` in dmesg — a memory leak grows until the OOM killer fires, then `Restart=on-failure` hides the crash by restarting it. Fix the leak (heap snapshots), set `MemoryMax=` in the unit for a cleaner failure mode, and alert on restarts (`systemctl show -p NRestarts`) so "self-healing" doesn't mask real defects.

### S14. A cron job creates files that another service can't read. It works when you run the script manually. Why?

Cron runs with **your user but not your shell environment** — and crucially a possibly different **umask** (cron's default umask is often `022` or `077` depending on distro/PAM config, not your shell's). Manually you create `644` files; cron creates `600`. Fix inside the script — never rely on the caller's environment:

```bash
#!/usr/bin/env bash
umask 022                       # explicit
install -m 644 out.csv /srv/share/   # or set the mode per-file explicitly
```
Same reasoning applies to PATH, locale, and env vars — a cron script should be fully self-contained.

### S15. `df -h` says 40% used, but you cannot create files: "No space left on device". Explain and fix.

**Inode exhaustion.** Block space is free but every inode is consumed — classically by millions of tiny files (session files, cache entries, mail queue).

```bash
df -i                                          # confirm: IUse% = 100%
for d in /var/*; do echo "$(find "$d" -xdev | wc -l) $d"; done | sort -rn | head
                                               # find the directory with the file explosion
find /var/lib/php/sessions -type f -mtime +7 -delete    # purge the culprit
```
Long-term: fix the producer (session GC, log cleanup), or rebuild the FS with a higher inode ratio (`mkfs.ext4 -i 4096`) — inode count is fixed at format time on ext4.

### S16. SSH to a server suddenly takes 30 seconds to give you a prompt, then works normally. What do you check?

Classic causes, in order of likelihood:
1. **Reverse DNS timeout** — sshd tries to resolve your client IP; if the DNS server is unreachable, it waits for timeout. Fix: `UseDNS no` in `sshd_config`.
2. **GSSAPI negotiation delay** — `GSSAPIAuthentication no` (client or server).
3. **Full/slow home filesystem** or slow LDAP/NSS lookups for your user (check `nsswitch.conf`, try a local user).

Diagnose from the client with `ssh -vvv host` — the log stalls exactly at the slow step (e.g., stuck after "debug1: SSH2_MSG_SERVICE_ACCEPT" → auth/DNS on server side).

### S17. You edited `/etc/fstab` and now the server won't boot — it drops into emergency mode. Recover.

At the emergency shell (or via console/rescue ISO):

```bash
journalctl -xb | grep -i mount        # confirm which mount failed
mount -o remount,rw /                 # emergency mode often mounts / read-only
vi /etc/fstab                         # fix or comment out the bad line
mount -a                              # TEST — this is the step people skip
systemctl daemon-reload && reboot
```
Prevention: **always `sudo mount -a` immediately after editing fstab** (it applies fstab without rebooting — errors show up while you can still fix them), and add `nofail` to non-critical mounts so a missing disk degrades instead of blocking boot:
```text
UUID=xxxx  /data  ext4  defaults,nofail,x-systemd.device-timeout=5s  0 2
```

### S18. Two services must talk on localhost:6379, but you see intermittent "Cannot assign requested address" errors under load. What's happening?

**Ephemeral port exhaustion.** Each outbound connection consumes a local port; closed connections linger in TIME_WAIT (~60s). At high connection-per-second rates with no pooling/keep-alive, all ~28k ephemeral ports are in TIME_WAIT.

```bash
ss -tan state time-wait | wc -l                    # confirm the pileup
cat /proc/sys/net/ipv4/ip_local_port_range         # default 32768-60999
sysctl -w net.ipv4.tcp_tw_reuse=1                  # safe mitigation (outbound)
sysctl -w net.ipv4.ip_local_port_range="15000 65000"   # widen the range
```
The *real* fix is architectural: **connection pooling / keep-alive** (one persistent connection instead of thousands of short-lived ones). The sysctls only buy headroom.

### S19. A junior admin ran `chmod -R 777 /` (or `chown -R` on `/`) before stopping it. The system is misbehaving. What now?

Assess honestly: system-wide permission destruction is generally **not repairable in place** — `sudo` stops working (sudoers must be 0440), SSH refuses keys, setuid bits are gone. Immediate steps:
1. Keep any existing root sessions open (new auth may fail).
2. Snapshot/backup application data now.
3. On RPM systems, `rpm --setugids -a && rpm --setperms -a` restores *package-owned* file perms (partial fix); Debian has no full equivalent.
4. Realistic answer interviewers want: **rebuild the host from configuration management** (Ansible/image), restore data from backup. This is *why* infrastructure-as-code and immutable infrastructure exist — hosts should be replaceable, not archaeologically restored.

### S20. Your web app reports "Permission denied" reading a file, but `ls -l` shows the app user has read permission on the file. What else can block access?

Checklist beyond the file's own mode bits:
1. **Directory execute (`x`) permission missing** anywhere along the path — you need `x` on *every* ancestor directory to traverse it: `namei -l /var/www/app/config.yml` shows the whole chain.
2. **SELinux/AppArmor** — `sudo ausearch -m avc -ts recent` (SELinux) or `dmesg | grep apparmor`. Fix context: `restorecon -Rv /var/www/app`. This is *the* classic on RHEL when files were `mv`ed (keeps old context) instead of `cp`ied.
3. **ACLs** — `getfacl file` may reveal a deny/mask beyond `ls -l` (a `+` at the end of the mode string hints ACLs exist).
4. **Filesystem mount options** — `noexec`, `ro` on the mount (`findmnt /var/www`).
5. **systemd sandboxing** — `ProtectSystem=strict`, `ReadOnlyPaths=`, or `PrivateTmp=true` in the unit can deny paths regardless of Unix perms: check `systemctl cat myapp`.

### S21. You need to capture what HTTP requests a misbehaving legacy binary sends — no docs, no source, no logs. How?

```bash
# 1. Watch its syscalls — see connect() targets and write() payloads
strace -f -e trace=network -s 200 ./legacy-bin

# 2. Capture the actual traffic
sudo tcpdump -i any -A 'tcp port 80' -w /tmp/cap.pcap    # -A shows ASCII payloads
# analyze: tcpdump -r /tmp/cap.pcap -A | less  (or open in Wireshark)

# 3. HTTPS? Intercept with a local proxy
HTTP_PROXY=http://127.0.0.1:8080 HTTPS_PROXY=http://127.0.0.1:8080 ./legacy-bin
# with mitmproxy listening — works if the binary honors proxy env vars

# 4. What files/configs does it touch while at it?
strace -f -e trace=openat ./legacy-bin 2>&1 | grep -v ENOENT
```

### S22. Root disk hit 100% and even `rm` misbehaves; the box hosts a busy database you must not stop. Get space back safely, in order.

```bash
# 1. Instant wins — truncate (not rm!) fat logs: frees space even while held open
: > /var/log/myapp/huge.log          # truncate-in-place, fd stays valid
journalctl --vacuum-size=200M        # trim the systemd journal

# 2. Package caches — always safe
apt clean          # or: dnf clean all

# 3. Find the growth (largest dirs, this filesystem only)
du -x -h --max-depth=2 / 2>/dev/null | sort -hr | head -15

# 4. Deleted-but-open files (safe to reclaim via truncate through /proc)
lsof +L1 | sort -k7 -rn | head       # link count 0 = deleted, still held
: > /proc/<PID>/fd/<FD>              # reclaim without touching the process

# 5. Old rotated logs, core dumps, tmp
find /var/log -name "*.gz" -mtime +14 -delete
find / -xdev -name "core.*" -size +100M 2>/dev/null
```
Never delete: anything under the DB's data directory, files you can't identify, or live logs (`truncate`, don't `rm` — `rm` on a held-open file frees *nothing* and hides the file from `du`, making things more confusing).

### S23. After a reboot, a service starts but can't reach the database — yet restarting it manually fixes it every time. Diagnose.

**Startup ordering race**: the unit starts before the network (or the DB) is actually ready. `After=network.target` only orders against network *infrastructure setup*, not "network is usable."

```ini
[Unit]
After=network-online.target postgresql.service
Wants=network-online.target
Requires=postgresql.service         # hard dependency, restart together

[Service]
Restart=on-failure
RestartSec=3
# Belt and braces: make the app itself retry DB connections at startup —
# ordering fixes the common case; retries fix the distributed-systems case
# (remote DB, cloud, k8s — no systemd ordering possible there).
```
Verify the theory before fixing: `journalctl -u myapp -b` timestamps vs. `journalctl -u postgresql -b` — you'll see the connection attempt land before the DB's "ready to accept connections."

### S24. Explain what you'd check if `sudo` suddenly takes ~25 seconds before prompting for the password.

`sudo` tries to resolve the machine's **own hostname**; if `/etc/hostname` doesn't have a matching entry in `/etc/hosts`, it queries DNS and waits for the timeout.

```bash
hostname                     # e.g., app-server-01
grep "$(hostname)" /etc/hosts    # missing? add it:
echo "127.0.1.1 app-server-01" | sudo tee -a /etc/hosts
```
Same root cause family as the slow-SSH scenario (S16) — **name-resolution timeouts masquerading as "the system is slow."** Rule of thumb: any *fixed-length* delay (exactly 5, 10, 25 s) smells like a network timeout, not load.

---

*End of reference — pair this with hands-on practice: every command here is best learned by running it against a throwaway VM or container (`docker run -it ubuntu bash`).*
