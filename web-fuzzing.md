# Web fuzzing

1. `Initial Fuzzing`:
   * The fuzzing process begins with the top-level directory, typically the web root (`/`).
   * The fuzzer starts sending requests based on the provided wordlist containing the potential directory and file names.
   * The fuzzer analyzes server responses, looking for successful results (e.g., HTTP 200 OK) that indicate the existence of a directory.
2. `Directory Discovery and Expansion`:
   * When a valid directory is found, the fuzzer doesn't just note it down. It creates a new branch for that directory, essentially appending the directory name to the base URL.
   * For example, if the fuzzer finds a directory named `admin` at the root level, it will create a new branch like `http://localhost/admin/`.
   * This new branch becomes the starting point for a fresh fuzzing process. The fuzzer will again iterate through the wordlist, appending each entry to the new branch's URL (e.g., `http://localhost/admin/FUZZ`).
3. `Iterative Depth`:
   * The process repeats for each discovered directory, creating further branches and expanding the fuzzing scope deeper into the web application's structure.
   * This continues until a specified depth limit is reached (e.g., a maximum of three levels deep) or no more valid directories are found.

```
ffuf -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -ic -u http://IP:PORT/FUZZ -e .html -recursion -recursion-depth 2 -rate 500
```

->in the above command uses the recursion method recursion means if it first finds the admin dir it changes the url also as per the above http://IP:PORT/FUZZ to it converts like these after finding admin dir http://IP:PORT/admin/FUZZ ->recursion depth means how many times the recursion needs to done

```
ffuf -w users.txt:USER -w pass.txt:PASS \
-u http://154.57.164.80:31014/login.php \
-X POST -d "username=USER&password=PASS" \
-fc 200
```

in here we want to fuzz the two values at the same time means we need to fuzz the both words at the same request we can use the above like syntax

```
(umedh㉿kali)-[~]
└─$ gobuster dns --domain inlanefreight.com \
-w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
--timeout 5s --resolver 1.1.1.1 -t 20 --quiet
ns1.inlanefreight.com 178.128.39.165
ns2.inlanefreight.com 206.189.119.186
www.inlanefreight.com 134.209.24.248,2a03:b0c0:1:e0::32c:b001
blog.inlanefreight.com 134.209.24.248,2a03:b0c0:1:e0::32c:b001
support.inlanefreight.com 134.209.24.248
ns3.inlanefreight.com 134.209.24.248
my.inlanefreight.com 134.209.24.248
customer.inlanefreight.com 134.209.24.248,2a03:b0c0:1:e0::32c:b001

```

in here we are using the dns means we are doing the fuzzing by using the domain name

```
┌──(umedh㉿kali)-[~]
└─$ ffuf -w /usr/share/seclists/Discovery/Web-Content/common.txt \
-u http://154.57.164.76:31673 \
-H "Host: FUZZ.inlanefreight.htb" \
-ac -v

        /'___\  /'___\           /'___\
       /\ \__/ /\ \__/  __  __  /\ \__/
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/
         \ \_\   \ \_\  \ \____/  \ \_\
          \/_/    \/_/   \/___/    \/_/

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://154.57.164.76:31673
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/common.txt
 :: Header           : Host: FUZZ.inlanefreight.htb
 :: Follow redirects : false
 :: Calibration      : true
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

[Status: 200, Size: 100, Words: 4, Lines: 2, Duration: 189ms]
| URL | http://154.57.164.76:31673
    * FUZZ: ADMIN

[Status: 200, Size: 100, Words: 4, Lines: 2, Duration: 193ms]
| URL | http://154.57.164.76:31673
    * FUZZ: Admin

[Status: 200, Size: 100, Words: 4, Lines: 2, Duration: 196ms]
| URL | http://154.57.164.76:31673
    * FUZZ: admin

[Status: 200, Size: 104, Words: 4, Lines: 2, Duration: 212ms]
| URL | http://154.57.164.76:31673
    * FUZZ: awmdata

[Status: 200, Size: 102, Words: 4, Lines: 2, Duration: 208ms]
| URL | http://154.57.164.76:31673
    * FUZZ: ipdata

[Status: 200, Size: 108, Words: 4, Lines: 2, Duration: 190ms]
| URL | http://154.57.164.76:31673
    * FUZZ: web-beans

:: Progress: [4750/4750] :: Job [1/1] :: 203 req/sec :: Duration: [0:00:27] :: Errors: 0 ::
```

fuzzing vhost by using the ffuf

#### Gobuster

`Gobuster` offers various filtering options depending on the module being run, to help you focus on specific responses and streamline your analysis. There is a small caveat, the `-s` and `-b` options are only available in the `dir` fuzzing mode.

| Flag               | Description                                                                         | Example Scenario                                                                       |
| ------------------ | ----------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| `-s` (include)     | Include only responses with the specified status codes (comma-separated).           | You're looking for redirects, so you filter for codes `301,302,307`                    |
| `-b` (exclude)     | Exclude responses with the specified status codes (comma-separated).                | The server returns many 404 errors. Exclude them with `-b 404`                         |
| `--exclude-length` | Exclude responses with specific content lengths (comma-separated, supports ranges). | You're not interested in 0-byte or 404-byte responses, so use `--exclude-length 0,404` |

```
# Find directories with status codes 200 or 301, but exclude responses with a size of 0 (empty responses) Umedh@htb[/htb]$ gobuster dir -u http://example.com/ -w wordlist.txt -s 200,301 --exclude-length 0
```

### FFUF

`FFUF` offers a highly customizable filtering system, enabling precise control over the displayed output. This allows you to efficiently sift through potentially large amounts of data and focus on the most relevant findings. `FFUF's` filtering options are categorized into multiple types, each serving a specific purpose in refining your results.

| Flag                                           | Description                                                                                                                                                                                                                                                                                       | Example Scenario                                                                                                                             |
| ---------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| `-mc` (match code)                             | Include only responses that match the specified status codes. You can provide a single code, multiple codes separated by commas, or ranges of codes separated by hyphens (e.g., `200,204,301`, `400-499`). The default behavior is to match codes 200-299, 301, 302, 307, 401, 403, 405, and 500. | After fuzzing, you notice many 302 (Found) redirects, but you're primarily interested in 200 (OK) responses. Use `-mc 200` to isolate these. |
| `-fc` (filter code)                            | Exclude responses that match the specified status codes, using the same format as `-mc`. This is useful for removing common error codes like 404 Not Found.                                                                                                                                       | A scan returns many 404 errors. Use `-fc 404` to remove them from the output.                                                                |
| `-fs` (filter size)                            | Exclude responses with a specific size or range of sizes. You can specify single sizes or ranges using hyphens (e.g., `-fs 0` for empty responses, `-fs 100-200` for responses between 100 and 200 bytes).                                                                                        | You suspect the interesting responses will be larger than 1KB. Use `-fs 0-1023` to filter out smaller responses.                             |
| `-ms` (match size)                             | Include only responses that match a specific size or range of sizes, using the same format as `-fs`.                                                                                                                                                                                              | You are looking for a backup file that you know is exactly 3456 bytes in size. Use `-ms 3456` to find it.                                    |
| `-fw` (filter out number of words in response) | Exclude responses containing the specified number of words in the response.                                                                                                                                                                                                                       | You're filtering out a specific number of words from the responses. Use `-fw 219` to filter for responses containing that amount of words.   |
| `-mw` (match word count)                       | Include only responses that have the specified amount of words in the response body.                                                                                                                                                                                                              | You're looking for short, specific error messages. Use `-mw 5-10` to filter for responses with 5 to 10 words.                                |
| `-fl` (filter line)                            | Exclude responses with a specific number of lines or range of lines. For example, `-fl 5` will filter out responses with 5 lines.                                                                                                                                                                 | You notice a pattern of 10-line error messages. Use `-fl 10` to filter them out.                                                             |
| `-ml` (match line count)                       | Include only responses that have the specified amount of lines in the response body.                                                                                                                                                                                                              | You're looking for responses with a specific format, such as 20 lines. Use `-ml 20` to isolate them.                                         |
| `-mt` (match time)                             | Include only responses that meet a specific time-to-first-byte (TTFB) condition. This is useful for identifying responses that are unusually slow or fast, potentially indicating interesting behavior.                                                                                           | The application responds slowly when processing certain inputs. Use `-mt >500` to find responses with a TTFB greater than 500 milliseconds.  |

```
# Find directories with status code 200, based on the amount of words, and a response size greater than 500 bytes Umedh@htb[/htb]$ ffuf -u http://example.com/FUZZ -w wordlist.txt -mc 200 -fw 427 -ms >500 # Filter out responses with status codes 404, 401, and 302 Umedh@htb[/htb]$ ffuf -u http://example.com/FUZZ -w wordlist.txt -fc 404,401,302 # Find backup files with the .bak extension and size between 10KB and 100KB Umedh@htb[/htb]$ ffuf -u http://example.com/FUZZ.bak -w wordlist.txt -fs 0-10239 -ms 10240-102400 # Discover endpoints that take longer than 500ms to respond Umedh@htb[/htb]$ ffuf -u http://example.com/FUZZ -w wordlist.txt -mt >500
```

```

┌──(umedh㉿kali)-[~]
└─$ ffuf -u http://154.57.164.76:32636/post.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "y=FUZZ" -w /usr/share/seclists/Discovery/Web-Content/common.txt -v -mc 200-400

        /'___\  /'___\           /'___\
       /\ \__/ /\ \__/  __  __  /\ \__/
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/
         \ \_\   \ \_\  \ \____/  \ \_\
          \/_/    \/_/   \/___/    \/_/

       v2.1.0-dev
________________________________________________

 :: Method           : POST
 :: URL              : http://154.57.164.76:32636/post.php
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/common.txt
 :: Header           : Content-Type: application/x-www-form-urlencoded
 :: Data             : y=FUZZ
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-400
________________________________________________

[Status: 200, Size: 26, Words: 1, Lines: 2, Duration: 198ms]
| URL | http://154.57.164.76:32636/post.php
    * FUZZ: SUNWmc

:: Progress: [4750/4750] :: Job [1/1] :: 198 req/sec :: Duration: [0:00:24] :: Errors: 0 ::


```

| **Type of Fuzzing**      | **Recommended SecLists Path**                        |
| ------------------------ | ---------------------------------------------------- |
| **Common API Endpoints** | `Discovery/Web-Content/api/api-endpoints.txt`        |
| **API Versioning**       | `Discovery/Web-Content/api/common-api-endpoints.txt` |
| **Parameter Names**      | `Discovery/Web-Content/burp-parameter-names.txt`     |
| **JSON Properties**      | `Fuzzing/JSON.fuzz.txt`                              |
