# Broken Authentication

_The wordlist used in these module are the_

```
/opt/SecLists/Usernames/xato-net-10-million-usernames.txt
https://github.com/datasets/world-cities/blob/main/data/world-cities.csv
```

**Enumerating Users**

Enumerating users means we are finding the users by brute forcing various usernames to find the valid usernames

```
ffuf -w /opt/useful/seclists/Usernames/xato-net-10-million-usernames.txt \
-u 'http://154.57.164.68:31060/index.php' \
-H 'Content-Type: application/x-www-form-urlencoded' \
-H 'Cookie: PHPSESSID=e6ep73srrtmmvbdmopli7kv4vr' \
-d 'username=FUZZ&password=ff' \
-fr 'Unknown user' \
-X POST
```

By using these command we can brute force the usernames

\*\*Brute-Forcing Passwords

```
Umedh@htb[/htb]$ grep '[[:upper:]]' /opt/useful/seclists/Passwords/Leaked-Databases/rockyou.txt | grep '[[:lower:]]' | grep '[[:digit:]]' | grep -E '.{10}' > custom_wordlist.txt 
Umedh@htb[/htb]$ wc -l custom_wordlist.txt 151647 custom_wordlist.txt
Umedh@htb[/htb]$ awk 'length($0) >= 10 && /[a-z]/ && /[A-Z]/ && /[0-9]/' /opt/useful/seclists/Passwords/Leaked-Databases/rockyou.txt > custom_wordlist.txt
```

By using the above commands we are sorting the passwords Normally in the rockyou.txt contains more than 14 million passwords it takes long time to complete that why in here we are sorting like these the password must contain of all these things (upper case, lowercase, digits, length of password is min 10) Normally an attacker can sort passwords according to the application

```
ffuf -w custom_wordlist.txt -u http://154.57.164.65:30300/index.php -X POST -H 'Cookie: PHPSESSID=go34si3rm8q11t14m4ujgg6jhf' -H 'Content-Type: application/x-www-form-urlencoded' -d 'username=admin&password=FUZZ' -fr 'Invalid username or password'
```

By using these command we can brute force the password for the admin user

\*\*Brute-Forcing Password Reset Tokens

```
 ffuf -w s.txt -u http://154.57.164.64:32744/reset_password.php?token=FUZZ -fr "The provided token is invalid"
```

In any application if user forgot the password is there option reset password at that time user can get the reset token to bypass the reset token

\*\*Brute-Forcing 2FA Codes

```
ffuf -w s.txt -u http://154.57.164.72:30866/2fa.php -X POST -H 'Content-Type: application/x-www-form-urlencoded' -H 'Cookie: PHPSESSID=91rfggskaro12gjrtt2gmjgag3' -d 'otp=FUZZ' -fr ' Invalid 2FA Code'
```

In here after getting the username and the password some of the applications have the 2FA to bypass it

***

***

\*\*Vulnerable Password Reset

```
ffuf -u http://154.57.164.78:30777/security_question.php -X POST -w city_wordlist.txt -H 'Content-Type: application/x-www-form-urlencoded' -b 'PHPSESSID=4ga0itn0g0el02irj9av9924bh' -d 'security_response=FUZZ' -fr ='Incorrect response'
```

**Business Logic Flaws** in password reset functions can allow an attacker to take over accounts even when traditional protections like rate-limiting are in place. Many applications use predefined security questions (e.g., "What city were you born in?"). Since the questions are the same for everyone, they become a target for **OSINT** and **Dictionary Attacks**.

* **The Workflow:**
  1. **Identify the Target:** Enter the victim's username (e.g., `admin`) in the reset flow.
  2. **Generate a Wordlist:** Use a specialized list (like a CSV of global cities) and clean it using Linux commands like `cut` and `grep`.
  3. **Execute the Attack:** Use `ffuf` to blast the `security_question.php` endpoint.
* **Crucial Detail:** You must include your **PHPSESSID** cookie in the `ffuf` command so the server knows which user’s "session" you are currently trying to reset.

This is a "Logic Bug" where the server trusts the user-supplied data in the final step of the reset process without verifying the state from the previous steps.

* **The Vulnerability:** The web app passes the `username` as a hidden POST parameter in the final `reset_password.php` request.
* **The Exploit:** 1. Start the reset process for your **own** account (`htb-stdnt`). 2. Answer your **own** security question correctly. 3. On the final "Set New Password" screen, intercept the request and change `username=htb-stdnt` to `username=admin`.
* **The Result:** If the server doesn't verify the session state, it will update the password for the `admin` account instead of your

\*\*Authentication Bypass via Direct Access

| **Feature**                                          | **Why it happens (The Root Cause)**                                                                                                                       | **How to bypass it (The Attack)**                                                                                                                         |
| ---------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Server-Side Logic**                                | The developer uses `header("Location: ...")` to redirect unauthenticated users, but **forgets to add `exit;` or `die();`** after it.                      | Use **`curl`** in your terminal. By default, `curl` does not follow redirects, so it will simply display the "hidden" body content of the `302` response. |
| **Browser Behavior**                                 | Browsers are "obedient." When they see a `302 Found` status, they immediately jump to the new page, hiding whatever content was in the original response. | Use **Burp Suite Intercept**. Catch the response coming from the server and change the status code from **`302 Found`** to **`200 OK`**.                  |
| **Information Leakage**                              | Because the script didn't stop, the server finishes rendering the entire "protected" HTML page and sends it in the response body anyway.                  | Use **Burp Suite Match and Replace**. Set a rule to automatically swap `302` to `200` so you can browse the site normally without constant redirects.     |
| \*\*Authentication Bypass via Parameter Modification |                                                                                                                                                           |                                                                                                                                                           |

In this scenario, the application relies on a **URL parameter** (like `user_id=183`) to determine who you are and what you can see. Instead of checking a secure, server-side session to verify your identity, the server looks at the URL.

If the application doesn't verify that the `user_id` in the URL actually matches the person who logged in, an attacker can simply change that number to "become" someone else. This is a form of **IDOR (Insecure Direct Object Reference)** mixed with **Broken Authentication**.

```
ffuf -w s2.txt -u http://154.57.164.79:30151/admin.php?user_id=FUZZ -fr 'Could not load admin data'
```

| **Step**           | **Action**                                                               | **Why it works**                                                                                       |
| ------------------ | ------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------ |
| **1. Identify**    | Log in as a normal user and look at the URL for IDs.                     | You need a "baseline" to see how the app handles IDs.                                                  |
| **2. Verify**      | Remove the parameter or change it to a random number.                    | If the page redirects to login, you know the parameter is **required** for authentication.             |
| **3. Manipulate**  | Change `user_id=183` to something else (e.g., `user_id=1`, `user_id=2`). | Administrators are often assigned low numbers (like 1 or 100) or numbers very close to your own.       |
| **4. Brute-Force** | Use a tool like `ffuf` or `Burp Intruder` to test a range of IDs.        | If you can't guess the Admin ID, you can automate the guessing process until the page content changes. |

\*\*Attacking Session Tokens

| **Attack Type**               | **Why it Happens (The Weakness)**                                                        | **How to Bypass/Exploit it**                                                                  |
| ----------------------------- | ---------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| **Brute-Force (Low Entropy)** | The token is too short (e.g., 4 chars) or mostly static with only a tiny random part.    | Capture multiple tokens, find the "dynamic" part, and use `ffuf` to guess all possibilities.  |
| **Sequential Tokens**         | Tokens are just numbers that increase by 1 (e.g., `1001`, `1002`).                       | Simple math. Subtract `1` from your token to become the person who logged in just before you. |
| **Encoded/Tamperable**        | The token is just plain text hidden in **Base64** or **Hex** (e.g., `dXNlcj1ndWVzdA==`). | Decode the cookie, change `role=user` to `role=admin`, re-encode it, and send it back.        |
| **Predictable Logic**         | The token is generated using a known pattern (like `username + date`).                   | Reverse-engineer the pattern and generate a valid token for the `admin` user.                 |
