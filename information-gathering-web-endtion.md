# Information gathering web endtion

| Tool                         | Key Features                                                                                            | Use Cases                                                                                                                               |
| ---------------------------- | ------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| `dig`                        | Versatile DNS lookup tool that supports various query types (A, MX, NS, TXT, etc.) and detailed output. | Manual DNS queries, zone transfers (if allowed), troubleshooting DNS issues, and in-depth analysis of DNS records.                      |
| `nslookup`                   | Simpler DNS lookup tool, primarily for A, AAAA, and MX records.                                         | Basic DNS queries, quick checks of domain resolution and mail server records.                                                           |
| `host`                       | Streamlined DNS lookup tool with concise output.                                                        | Quick checks of A, AAAA, and MX records.                                                                                                |
| `dnsenum`                    | Automated DNS enumeration tool, dictionary attacks, brute-forcing, zone transfers (if allowed).         | Discovering subdomains and gathering DNS information efficiently.                                                                       |
| `fierce`                     | DNS reconnaissance and subdomain enumeration tool with recursive search and wildcard detection.         | User-friendly interface for DNS reconnaissance, identifying subdomains and potential targets.                                           |
| `dnsrecon`                   | Combines multiple DNS reconnaissance techniques and supports various output formats.                    | Comprehensive DNS enumeration, identifying subdomains, and gathering DNS records for further analysis.                                  |
| `theHarvester`               | OSINT tool that gathers information from various sources, including DNS records (email addresses).      | Collecting email addresses, employee information, and other data associated with a domain from multiple sources.                        |
| `Online DNS Lookup Services` | User-friendly interfaces for performing DNS lookups.                                                    | Quick and easy DNS lookups, convenient when command-line tools are not available, checking for domain availability or basic information |

### The Domain Information Groper

The `dig` command (`Domain Information Groper`) is a versatile and powerful utility for querying DNS servers and retrieving various types of DNS records. Its flexibility and detailed and customizable output make it a go-to choice.

| Command                         | Description                                                                                                                                                                                          |
| ------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `dig domain.com`                | Performs a default A record lookup for the domain.                                                                                                                                                   |
| `dig domain.com A`              | Retrieves the IPv4 address (A record) associated with the domain.                                                                                                                                    |
| `dig domain.com AAAA`           | Retrieves the IPv6 address (AAAA record) associated with the domain.                                                                                                                                 |
| `dig domain.com MX`             | Finds the mail servers (MX records) responsible for the domain.                                                                                                                                      |
| `dig domain.com NS`             | Identifies the authoritative name servers for the domain.                                                                                                                                            |
| `dig domain.com TXT`            | Retrieves any TXT records associated with the domain.                                                                                                                                                |
| `dig domain.com CNAME`          | Retrieves the canonical name (CNAME) record for the domain.                                                                                                                                          |
| `dig domain.com SOA`            | Retrieves the start of authority (SOA) record for the domain.                                                                                                                                        |
| `dig @1.1.1.1 domain.com`       | Specifies a specific name server to query; in this case 1.1.1.1                                                                                                                                      |
| `dig +trace domain.com`         | Shows the full path of DNS resolution.                                                                                                                                                               |
| `dig -x 192.168.1.1`            | Performs a reverse lookup on the IP address 192.168.1.1 to find the associated host name. You may need to specify a name server.                                                                     |
| `dig +short domain.com`         | Provides a short, concise answer to the query.                                                                                                                                                       |
| `dig +noall +answer domain.com` | Displays only the answer section of the query output.                                                                                                                                                |
| `dig domain.com ANY`            | Retrieves all available DNS records for the domain (Note: Many DNS servers ignore `ANY` queries to reduce load and prevent abuse, as per [RFC 8482](https://datatracker.ietf.org/doc/html/rfc8482)). |

\*\*dnsenum

dnsenum is the tool that does the active enuramution for the domain

```
┌──(umedh㉿kali)-[/usr/…/wordlists/seclists/Discovery/DNS]
└─$ dnsenum --enum inlanefreight.com -f subdomains-top1million-5000.txt

dnsenum VERSION:1.3.1

-----   inlanefreight.com   -----


Host's addresses:
__________________

inlanefreight.com.                       300      IN    A        134.209.24.248


Name Servers:
______________

ns1.inlanefreight.com.                   90       IN    A        178.128.39.165
ns2.inlanefreight.com.                   160      IN    A        206.189.119.186


Mail (MX) Servers:
___________________



Trying Zone Transfers and getting Bind Versions:
_________________________________________________


Trying Zone Transfer for inlanefreight.com on ns1.inlanefreight.com ...
AXFR record query failed: Connection timed out

Trying Zone Transfer for inlanefreight.com on ns2.inlanefreight.com ...
AXFR record query failed: Connection timed out


Scraping inlanefreight.com subdomains from Google:
___________________________________________________


 ----   Google search page: 1   ----



Google Results:
________________

  perhaps Google is blocking our queries.
 Check manually.


Brute forcing with subdomains-top1million-5000.txt:
____________________________________________________

www.inlanefreight.com.                   115      IN    A        134.209.24.248
ns1.inlanefreight.com.                   45       IN    A        178.128.39.165
ns2.inlanefreight.com.                   45       IN    A        206.189.119.186
blog.inlanefreight.com.                  115      IN    A        134.209.24.248
ns3.inlanefreight.com.                   60       IN    A        134.209.24.248
support.inlanefreight.com.               42       IN    A        134.209.24.248
my.inlanefreight.com.                    141      IN    A        134.209.24.248

```

\*\*AXFR

While brute-forcing can be a fruitful approach, there's a less invasive and potentially more efficient method for uncovering subdomains – DNS zone transfers. This mechanism, designed for replicating DNS records between name servers, can inadvertently become a goldmine of information for prying eyes if misconfigured.

A DNS zone transfer is essentially a wholesale copy of all DNS records within a zone (a domain and its subdomains) from one name server to another. This process is essential for maintaining consistency and redundancy across DNS servers. However, if not adequately secured, unauthorised parties can download the entire zone file, revealing a complete list of subdomains, their associated IP addresses, and other sensitive DNS data.

_But now a days most of the application disabled the AXFR due to security reasons_

```
┌──(umedh㉿kali)-[~]
└─$ dig axfr @10.129.14.198 inlanefreight.htb

; <<>> DiG 9.20.20-1-Debian <<>> axfr @10.129.14.198 inlanefreight.htb
; (1 server found)
;; global options: +cmd
inlanefreight.htb.	604800	IN	SOA	inlanefreight.htb. root.inlanefreight.htb. 2 604800 86400 2419200 604800
inlanefreight.htb.	604800	IN	NS	ns.inlanefreight.htb.
admin.inlanefreight.htb. 604800	IN	A	10.10.34.2
ftp.admin.inlanefreight.htb. 604800 IN	A	10.10.34.2
careers.inlanefreight.htb. 604800 IN	A	10.10.34.50
dc1.inlanefreight.htb.	604800	IN	A	10.10.34.16
dc2.inlanefreight.htb.	604800	IN	A	10.10.34.11
internal.inlanefreight.htb. 604800 IN	A	127.0.0.1
admin.internal.inlanefreight.htb. 604800 IN A	10.10.1.11
wsus.internal.inlanefreight.htb. 604800	IN A	10.10.1.240
ir.inlanefreight.htb.	604800	IN	A	10.10.45.5
dev.ir.inlanefreight.htb. 604800 IN	A	10.10.45.6
ns.inlanefreight.htb.	604800	IN	A	127.0.0.1
resources.inlanefreight.htb. 604800 IN	A	10.10.34.100
securemessaging.inlanefreight.htb. 604800 IN A	10.10.34.52
test1.inlanefreight.htb. 604800	IN	A	10.10.34.101
us.inlanefreight.htb.	604800	IN	A	10.10.200.5
cluster14.us.inlanefreight.htb.	604800 IN A	10.10.200.14
messagecenter.us.inlanefreight.htb. 604800 IN A	10.10.200.10
ww02.inlanefreight.htb.	604800	IN	A	10.10.34.112
www1.inlanefreight.htb.	604800	IN	A	10.10.34.111
inlanefreight.htb.	604800	IN	SOA	inlanefreight.htb. root.inlanefreight.htb. 2 604800 86400 2419200 604800
;; Query time: 584 msec
;; SERVER: 10.129.14.198#53(10.129.14.198) (TCP)
;; WHEN: Sun Mar 15 23:19:23 IST 2026
;; XFR size: 22 records (messages 1, bytes 594)
```

\*\*vhost

A **Virtual Host** is a configuration on a web server (like Apache or Nginx) that allows **multiple domain names** to be hosted on a **single IP address**.

When you type `dev.example.com` into your browser, your computer resolves it to an IP (e.g., `192.168.1.10`). However, `api.example.com` might resolve to that same IP. The web server uses the **HTTP `Host` header** in your request to determine which specific folder or application to serve to you.

_In below is there the some of the examples for the vhost for the application_

| **Type**                 | **Description**                                                            | **Security Significance**                                                                                                               |
| ------------------------ | -------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| **Versioned Subdomains** | Hosts like `v1.api.example.com` or `v2.api.example.com`.                   | Older versions (`v1`) often have unpatched vulnerabilities (e.g., Broken Object Level Authorization) that were fixed in newer versions. |
| **Environment VHosts**   | Hosts like `dev.example.com` or `beta.example.com`.                        | These often run "debug" mode, contain detailed error messages, or have less strict authentication than the production version.          |
| **Legacy Apps**          | Old versions of the site kept for a few clients (e.g., `old.example.com`). | These are prime targets for exploit research as they often run on outdated middleware (old PHP, Python, or Apache versions).            |

```

┌──(umedh㉿kali)-[~]
└─$ gobuster vhost -u http://inlanefreight.htb:31002 -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt --append-domain
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                       http://inlanefreight.htb:31002
[+] Method:                    GET
[+] Threads:                   10
[+] Wordlist:                  /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt
[+] User Agent:                gobuster/3.8.2
[+] Timeout:                   10s
[+] Append Domain:             true
[+] Exclude Hostname Length:   false
===============================================================
Starting gobuster in VHOST enumeration mode
===============================================================
blog.inlanefreight.htb:31002 Status: 200 [Size: 98]
admin.inlanefreight.htb:31002 Status: 200 [Size: 100]
forum.inlanefreight.htb:31002 Status: 200 [Size: 100]
support.inlanefreight.htb:31002 Status: 200 [Size: 104]
Progress: 4989 / 4989 (100.00%)
===============================================================
Finished
===============================================================
```

### Searching CT Logs

There are two popular options for searching CT logs:

| Tool                                | Key Features                                                                                                     | Use Cases                                                                                                 | Pros                                              | Cons                                         |
| ----------------------------------- | ---------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------- | ------------------------------------------------- | -------------------------------------------- |
| [crt.sh](https://crt.sh/)           | User-friendly web interface, simple search by domain, displays certificate details, SAN entries.                 | Quick and easy searches, identifying subdomains, checking certificate issuance history.                   | Free, easy to use, no registration required.      | Limited filtering and analysis options.      |
| [Censys](https://search.censys.io/) | Powerful search engine for internet-connected devices, advanced filtering by domain, IP, certificate attributes. | In-depth analysis of certificates, identifying misconfigurations, finding related certificates and hosts. | Extensive data and filtering options, API access. | Requires registration (free tier available). |

#### crt.sh lookup

While `crt.sh` offers a convenient web interface, you can also leverage its API for automated searches directly from your terminal. Let's see how to find all 'dev' subdomains on `facebook.com` using `curl` and `jq`:

```
Umedh@htb[/htb]$ curl -s "https://crt.sh/?q=facebook.com&output=json" | jq -r '.[]
 | select(.name_value | contains("dev")) | .name_value' | sort -u
 
*.dev.facebook.com
*.newdev.facebook.com
*.secure.dev.facebook.com
dev.facebook.com
devvm1958.ftw3.facebook.com
facebook-amex-dev.facebook.com
facebook-amex-sign-enc-dev.facebook.com
newdev.facebook.com
secure.dev.facebook.com

```

## Fingerprinting

***

Fingerprinting focuses on extracting technical details about the technologies powering a website or web application. Similar to how a fingerprint uniquely identifies a person, the digital signatures of web servers, operating systems, and software components can reveal critical information about a target's infrastructure and potential security weaknesses.

_Wafw00f_

These is a tool that detect the application is using firewall or not if the application is using which firewall is using .

`Web Application Firewalls` (`WAFs`) are security solutions designed to protect web applications from various attacks. Before proceeding with further fingerprinting, it's crucial to determine if `inlanefreight.com` employs a WAF, as it could interfere with our probes or potentially block our requests.

\*\*whatweb

**WhatWeb** is a high-level "next-generation" web scanner used for **fingerprinting** websites. Instead of just checking if a site is "up," it identifies the underlying technologies (the "tech stack") without performing a heavy vulnerability scan.

It is a standard tool in the Kali Linux arsenal for the **Information Gathering** phase

```
┌──(umedh㉿kali)-[~]
└─$ whatweb app.inlanefreight.local
ERROR Opening: https://app.inlanefreight.local - Connection refused - connect(2) for "10.129.14.234" port 443
http://app.inlanefreight.local [200 OK] Apache[2.4.41], Bootstrap, Cookies[72af8f2b24261272e581a49f5c56de40], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.41 (Ubuntu)], HttpOnly[72af8f2b24261272e581a49f5c56de40], IP[10.129.14.234], JQuery, MetaGenerator[Joomla! - Open Source Content Management], OpenSearch[http://app.inlanefreight.local/index.php/component/search/?layout=blog&amp;id=9&amp;Itemid=101&amp;format=opensearch], Script, Title[Home], UncommonHeaders[permissions-policy]

```

#### Nikto

`Nikto` is a powerful open-source web server scanner. In addition to its primary function as a vulnerability assessment tool, `Nikto's` fingerprinting capabilities provide insights into a website's technology stack.

Here are the most important flags categorized by their function:

### **Essential Nikto Flags**

| Category         | Flag            | Description                                                 | Example Usage                     |
| ---------------- | --------------- | ----------------------------------------------------------- | --------------------------------- |
| **Targeting**    | `-h`            | Specifies the target **host** (IP, URL, or hostname).       | `-h 10.10.10.123`                 |
|                  | `-p`            | Specifies the **port** (default is 80).                     | `-p 80,443,8080`                  |
|                  | `-ssl`          | Forces the use of **HTTPS** on the specified port.          | `-h <IP> -p 443 -ssl`             |
| **Optimization** | `-Tuning`       | Specifices types of tests (1-9, a, b, c, x).                | `-Tuning 123b`                    |
|                  | `-maxtime`      | Max time for a scan (e.g., `30m` for 30 minutes).           | `-maxtime 1h`                     |
|                  | `-timeout`      | Seconds to wait before a request times out.                 | `-timeout 10`                     |
| **Evasion**      | `-evasion`      | Uses encoding techniques to bypass IDS/WAF.                 | `-evasion 1`                      |
|                  | `-useproxy`     | Routes the scan through a specified **proxy**.              | `-useproxy http://localhost:8080` |
| **Output**       | `-o`            | Saves the scan results to a **file**.                       | `-o scan_results.txt`             |
|                  | `-Format`       | Sets file type (`csv`, `json`, `xml`, `htm`, `txt`).        | `-Format json`                    |
|                  | `-Display`      | Controls what is shown in terminal (e.g., `V` for verbose). | `-Display V`                      |
| **System**       | `-update`       | Updates the Nikto plugin database.                          | `nikto -update`                   |
|                  | `-list-plugins` | Lists all available plugins for scanning.                   | `nikto -list-plugins`             |

***

### **Tuning Flag Reference (Numerical Values)**

When using the `-Tuning` flag, you can combine numbers to run specific tests:

* **1:** Interesting File (check for `config.php`, `.env`, etc.)
* **2:** Misconfiguration / Default Files
* **3:** Information Disclosure
* **4:** Injection (XSS/HTML)
* **8:** Command Execution (Shells)
* **9:** SQL Injection
* **0:** File Upload
* **b:** Software Identification

\*\*Evasion Reference Table

| **Value** | **Technique**               | **Description**                                                   |
| --------- | --------------------------- | ----------------------------------------------------------------- |
| **1**     | Random URI Encoding         | Uses random non-UTF8 encoding for the URI.                        |
| **2**     | Directory Self-Reference    | Inserts `/./` into the path (e.g., `/./bin/php`).                 |
| **3**     | Premature URL Ending        | Attempts to trick the IDS by ending the URL early.                |
| **4**     | Prepend Random Long Strings | Adds a long string of random characters to the request.           |
| **5**     | Fake Parameter              | Adds a fake, random parameter to the URL to confuse logging.      |
| **6**     | TAB as Request Spacer       | Uses a TAB character instead of a SPACE in the HTTP request.      |
| **7**     | Change Case                 | Randomly changes the case of the URL (only works on Windows/IIS). |
| **8**     | Use Windows Separator       | Uses the backslash `\` instead of `/` as a directory separator.   |
| **A**     | Session Hijacking           | Attempts to use a specific session ID.                            |
| **B**     | Binary Value                | Adds binary values to the request headers.                        |

**Common Practice:** If you only care about finding **SQL Injection** and **Information Disclosure**, you would use: `nikto -h <target> -Tuning 39`

```

┌──(umedh㉿kali)-[~]
└─$ nikto -h dev.inlanefreight.local -Tuning -b
- Nikto v2.5.0
---------------------------------------------------------------------------
+ Target IP:          10.129.14.234
+ Target Hostname:    dev.inlanefreight.local
+ Target Port:        80
+ Start Time:         2026-03-16 00:50:12 (GMT5.5)
---------------------------------------------------------------------------
+ Server: Apache/2.4.41 (Ubuntu)
+ /: The anti-clickjacking X-Frame-Options header is not present. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options
+ /: The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type. See: https://www.netsparker.com/web-vulnerability-scanner/vulnerabilities/missing-content-type-header/
+ /robots.txt: Entry '/cache/' is returned a non-forbidden or redirect HTTP code (200). See: https://portswigger.net/kb/issues/00600600_robots-txt-file
+ /robots.txt: Entry '/cli/' is returned a non-forbidden or redirect HTTP code (200). See: https://portswigger.net/kb/issues/00600600_robots-txt-file
+ /robots.txt: Entry '/libraries/' is returned a non-forbidden or redirect HTTP code (200). See: https://portswigger.net/kb/issues/00600600_robots-txt-file
+ /robots.txt: Entry '/layouts/' is returned a non-forbidden or redirect HTTP code (200). See: https://portswigger.net/kb/issues/00600600_robots-txt-file
+ /robots.txt: Entry '/includes/' is returned a non-forbidden or redirect HTTP code (200). See: https://portswigger.net/kb/issues/00600600_robots-txt-file
+ /robots.txt: Entry '/components/' is returned a non-forbidden or redirect HTTP code (200). See: https://portswigger.net/kb/issues/00600600_robots-txt-file
+ /robots.txt: Entry '/bin/' is returned a non-forbidden or redirect HTTP code (200). See: https://portswigger.net/kb/issues/00600600_robots-txt-file
+ /robots.txt: Entry '/administrator/' is returned a non-forbidden or redirect HTTP code (200). See: https://portswigger.net/kb/issues/00600600_robots-txt-file
+ /robots.txt: Entry '/language/' is returned a non-forbidden or redirect HTTP code (200). See: https://portswigger.net/kb/issues/00600600_robots-txt-file
+ /robots.txt: Entry '/plugins/' is returned a non-forbidden or redirect HTTP code (200). See: https://portswigger.net/kb/issues/00600600_robots-txt-file
+ /robots.txt: Entry '/tmp/' is returned a non-forbidden or redirect HTTP code (200). See: https://portswigger.net/kb/issues/00600600_robots-txt-file
+ /robots.txt: Entry '/modules/' is returned a non-forbidden or redirect HTTP code (200). See: https://portswigger.net/kb/issues/00600600_robots-txt-file
+ /robots.txt: contains 14 entries which should be manually viewed. See: https://developer.mozilla.org/en-US/docs/Glossary/Robots.txt

```

\*\*robots.txt

Technically, `robots.txt` is a simple text file placed in the root directory of a website (e.g., `www.example.com/robots.txt`).t is used to communicate with web crawlers (like Googlebot) and tell them which parts of the website they are **not** allowed to visit or index.

```

┌──(umedh㉿kali)-[~]
└─$ curl "http://dev.inlanefreight.local/robots.txt"
# If the Joomla site is installed within a folder
# eg www.example.com/joomla/ then the robots.txt file
# MUST be moved to the site root
# eg www.example.com/robots.txt
# AND the joomla folder name MUST be prefixed to all of the
# paths.
# eg the Disallow rule for the /administrator/ folder MUST
# be changed to read
# Disallow: /joomla/administrator/
#
# For more information about the robots.txt standard, see:
# http://www.robotstxt.org/orig.html
#
# For syntax checking, see:
# http://tool.motoricerca.info/robots-checker.phtml

User-agent: *
Disallow: /administrator/
Disallow: /bin/
Disallow: /cache/
Disallow: /cli/
Disallow: /components/
Disallow: /includes/
Disallow: /installation/
Disallow: /language/
Disallow: /layouts/
Disallow: /libraries/
Disallow: /logs/
Disallow: /modules/
Disallow: /plugins/
Disallow: /tmp/
```

|                 |                                                                                                     |                                                      |
| --------------- | --------------------------------------------------------------------------------------------------- | ---------------------------------------------------- |
| `Crawl-delay`   | Sets a delay (in seconds) between successive requests from the bot to avoid overloading the server. | `Crawl-delay: 10` (10-second delay between requests) |
| `Sitemap`       | Provides the URL to an XML sitemap for more efficient crawling.                                     | `Sitemap: https://www.example.com/sitemap.xml`       |
| \*\*reconspider |                                                                                                     |                                                      |

**ReconSpider** is an advanced **Open Source Intelligence (OSINT)** framework designed for deep reconnaissance on IP addresses, emails, websites, organizations, and individuals. It acts as a comprehensive aggregator, pulling data from various public sources and visualizing it in a consolidated manner.

| **Module**           | **Purpose**                       | **Key Data Gathered**                                                     |
| -------------------- | --------------------------------- | ------------------------------------------------------------------------- |
| **IP Enumerate**     | Public resource scanning for IPs. | Location, ISP, ASN, and associated hostnames.                             |
| **Domain Scan**      | Infrastructure discovery.         | DNS records, CMS detection, port scanning, and vulnerability checks.      |
| **Phone Number**     | International OSINT for numbers.  | Country, carrier, line type, and reputation reports.                      |
| **DNS Map**          | Attack surface visualization.     | Virtual map of all DNS records associated with a target.                  |
| **Metadata**         | File analysis.                    | Extraction of hidden metadata from uploaded files (JPEG, PDF, etc.).      |
| **Honeypot**         | Defensive detection.              | Calculates a "HoneyScore" to determine if an IP is a trap.                |
| **Breach Detection** | Email security audits.            | Identifies all breached email IDs associated with a specific domain.      |
| **Social Media**     | Username tracking.                | Checks account availability/info across Facebook, Twitter, and Instagram. |

\*\*finalrecon

**FinalRecon** is an all-in-one automatic web reconnaissance tool written in Python. It is designed to be a "one-stop-shop" for the information-gathering phase of a penetration test, allowing you to collect a massive amount of data about a target website with a single command instead of running multiple tools individually.

### **Core Modules**

FinalRecon follows a modular structure, meaning you can run specific tests or all of them at once.

| **Module**    | **Purpose**                                     | **Key Data Collected**                                                         |
| ------------- | ----------------------------------------------- | ------------------------------------------------------------------------------ |
| **Headers**   | Analysis of HTTP response headers.              | Server type, security headers (HSTS, CSP), and X-Powered-By banners.           |
| **SSL Info**  | Inspection of the target's SSL/TLS certificate. | Issuer details, expiration date, serial number, and cipher suites.             |
| **WHOIS**     | Querying domain registration databases.         | Owner information, registrar, name servers, and creation dates.                |
| **Crawler**   | Deep-diving into the website's structure.       | Internal/external links, JavaScript files, images, `robots.txt`, and sitemaps. |
| **DNS Enum**  | Querying DNS records.                           | A, AAAA, MX, NS, TXT, and **DMARC** records for email security.                |
| **Subdomain** | Discovery of additional sub-assets.             | Uses sources like `crt.sh`, ThreatCrowd, and VirusTotal to find subdomains.    |
| **Directory** | Brute-forcing for hidden folders.               | Searches for common directories and files (supports custom extensions).        |
| **Wayback**   | Historical data retrieval.                      | Pulls URLs from the **Wayback Machine** to find old/forgotten files.           |
| **Port Scan** | Lightweight network analysis.                   | Fast scan of the top 1000 ports to identify open services.                     |

```


┌──(venv)─(umedh㉿kali)-[~/tools/Reconnaissance/FinalRecon]
└─$ python3 finalrecon.py --url http://inlanefreight.com --sslinfo --crawl

 ______  __   __   __   ______   __
/\  ___\/\ \ /\ "-.\ \ /\  __ \ /\ \
\ \  __\\ \ \\ \ \-.  \\ \  __ \\ \ \____
 \ \_\   \ \_\\ \_\\"\_\\ \_\ \_\\ \_____\
  \/_/    \/_/ \/_/ \/_/ \/_/\/_/ \/_____/
 ______   ______   ______   ______   __   __
/\  == \ /\  ___\ /\  ___\ /\  __ \ /\ "-.\ \
\ \  __< \ \  __\ \ \ \____\ \ \/\ \\ \ \-.  \
 \ \_\ \_\\ \_____\\ \_____\\ \_____\\ \_\\"\_\
  \/_/ /_/ \/_____/ \/_____/ \/_____/ \/_/ \/_/

[>] Created By   : thewhiteh4t
 |---> Twitter   : https://twitter.com/thewhiteh4t
 |---> Community : https://twc1rcle.com/
[>] Version      : 1.1.7

[+] Target : http://inlanefreight.com

[+] IP Address : 134.209.24.248

[!] SSL Certificate Information :

[+] protocol : TLSv1.3
[+] cipher
	└╴0: TLS_AES_256_GCM_SHA384
	└╴1: TLSv1.3
	└╴2: 256
[+] subject
	└╴commonName: inlanefreight.com
[+] issuer
	└╴countryName: US
	└╴organizationName: Let's Encrypt
	└╴commonName: R12
[+] version : Version.v3
[+] serialNumber : 578498569282364651005458703820450372257908
[+] notBefore : Mar 09 04:32:49 2026 GMT
[+] notAfter : Jun 07 04:32:48 2026 GMT
[+] subjectAltName
	└╴0: inlanefreight.com
	└╴1: www.inlanefreight.com

[!] Starting Crawler...

[+] Looking for robots.txt........[ Not Found ]
[+] Looking for sitemap.xml.......[ Not Found ]
[+] Extracting CSS Links..........[ 11 ]
[+] Extracting Javascript Links...[ 8 ]
[+] Extracting Internal Links.....[ 6 ]
[+] Extracting External Links.....[ 1 ]
[+] Extracting Images.............[ 0 ]
[+] Crawling Sitemaps.............[ 0 ]
[+] Crawling Javascripts..........[ 0 ]

[+] Total Unique Links Extracted : 26

[+] Completed in 0:00:12.733893

[+] Exported : /home/umedh/.local/share/finalrecon/dumps/fr_inlanefreight.com_16-03-2026_01:40:52

```
