# API Attacks

### OWASP Top 10 API Security Risks

To categorize and standardize the security vulnerabilities and misconfigurations that APIs can face, [OWASP](https://owasp.org/) has curated the [OWASP API Security Top 10](https://owasp.org/API-Security/editions/2023/en/0x11-t10/), a comprehensive list of the most critical security risks specifically related to APIs:

| **API Vulnerability & Official Link**                                                                                                                                                                         | **Description of the Vulnerability**                                                                                                                                                                                                                                            |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [API1:2023 - Broken Object Level Authorization (BOLA)](https://www.google.com/search?q=https://owasp.org/API-Security/editions/2023/en/0x11-t10/api1-broken-object-level-authorization/)                      | An API endpoint fails to verify if the authenticated user actually has permission to access the specific object they are requesting. Attackers exploit this by simply changing object IDs in the URL or payload to view or modify someone else's data.                          |
| [API2:2023 - Broken Authentication](https://www.google.com/search?q=https://owasp.org/API-Security/editions/2023/en/0x11-t10/api2-broken-authentication/)                                                     | Authentication mechanisms (like login, password resets, or token management) are implemented incorrectly. This allows attackers to steal or forge tokens and passwords, effectively hijacking another user's identity.                                                          |
| [API3:2023 - Broken Object Property Level Authorization](https://www.google.com/search?q=https://owasp.org/API-Security/editions/2023/en/0x11-t10/api3-broken-object-property-level-authorization/)           | This merges older excessive data exposure and mass assignment flaws. It happens when an API exposes sensitive fields it shouldn't, or allows users to update restricted properties (like blindly passing a JSON payload that changes a user's role to "admin").                 |
| [API4:2023 - Unrestricted Resource Consumption](https://www.google.com/search?q=https://owasp.org/API-Security/editions/2023/en/0x11-t10/api4-unrestricted-resource-consumption/)                             | The API lacks strict limits on how many requests a user can make, or how much data they can request at once. Attackers abuse this to cause Denial of Service (DoS) or drive up massive cloud infrastructure and third-party integration costs.                                  |
| [API5:2023 - Broken Function Level Authorization](https://www.google.com/search?q=https://owasp.org/API-Security/editions/2023/en/0x11-t10/api5-broken-function-level-authorization/)                         | The API fails to enforce role-based access control properly between different types of users. By changing the HTTP method (like using `DELETE` instead of `GET`) or manipulating the URL path, a standard user can access administrative or highly privileged functions.        |
| [API6:2023 - Unrestricted Access to Sensitive Business Flows](https://www.google.com/search?q=https://owasp.org/API-Security/editions/2023/en/0x11-t10/api6-unrestricted-access-to-sensitive-business-flows/) | The API works exactly as designed, but fails to prevent automated abuse (bots). Attackers exploit these flows at scale for activities like scalping concert tickets, credential stuffing, or spamming comments.                                                                 |
| [API7:2023 - Server Side Request Forgery (SSRF)](https://www.google.com/search?q=https://owasp.org/API-Security/editions/2023/en/0x11-t10/api7-server-side-request-forgery/)                                  | The API takes a user-supplied URL and fetches data from it without validating the destination. This acts as a proxy, allowing attackers to breach the firewall and force the server to connect to internal cloud metadata services, internal databases, or restricted networks. |
| [API8:2023 - Security Misconfiguration](https://www.google.com/search?q=https://owasp.org/API-Security/editions/2023/en/0x11-t10/api8-security-misconfiguration/)                                             | A broad category covering missing security headers, improperly configured CORS policies, unpatched systems, and error messages that leak verbose technical details (like stack traces) to the end user.                                                                         |
| [API9:2023 - Improper Inventory Management](https://www.google.com/search?q=https://owasp.org/API-Security/editions/2023/en/0x11-t10/api9-improper-inventory-management/)                                     | Organizations lose track of their API endpoints. This results in "shadow APIs" (undocumented endpoints) and "zombie APIs" (deprecated, old versions still running), which remain unpatched and provide an easy backdoor for attackers.                                          |
| [API10:2023 - Unsafe Consumption of APIs](https://www.google.com/search?q=https://owasp.org/API-Security/editions/2023/en/0x11-t10/api10-unsafe-consumption-of-apis/)                                         | Developers tend to blindly trust data coming back from other third-party APIs. If attackers compromise that integrated third-party service, they can inject malicious payloads into the target API, leading to downstream exploits.                                             |

#### Broken Object Level Authorization (BOLA)

This attack is a classic case of **Broken Object Level Authorization (BOLA)**, frequently referred to in the security world as **IDOR** (Insecure Direct Object Reference).

In simple terms: the server knows **who** you are (Authentication), but it fails to check if you **own** what you're asking for (Authorization).

The application correctly identifies you as a "Supplier" via your JWT (JSON Web Token). However, it makes a fatal assumption: it assumes that if you have the role `SupplierCompanies_GetYearlyReportByID`, you are allowed to see **any** report in the database, not just your own.

While the user's account uses a complex, unguessable **UUID** (`b75a7c76-...`), the sensitive reports use simple **Integer IDs** (`1`, `2`, `3`).

* **Expected Behavior:** You ask for Report #1 $\rightarrow$ Server checks if Report #1 belongs to your UUID $\rightarrow$ Access Granted/Denied.
* **Actual Behavior:** You ask for Report #1 $\rightarrow$ Server checks if you are a Supplier $\rightarrow$ Access Granted.

### ## 🚀 The Attack Flow

1. **Authentication:** You log in and get a JWT. This "locks" the lock icon in Swagger, proving you are a valid user.
2. **Discovery:** You look at the `/api/v1/supplier-companies/yearly-reports/{ID}` endpoint. You notice it takes a simple number.
3. **Testing the Boundary:** You try `ID=1`. You see a report. You check the `companyID` in the response and realize it **doesn't match yours**. This is the confirmation of the BOLA vulnerability.
4. **Mass Enumeration:** Instead of clicking one by one, you use a **Bash for-loop** to automate the request. This allows you to "scrape" the entire database of yearly reports in seconds.

> \[!IMPORTANT] **Mass Abuse:** The `curl` command combined with `jq` allows a researcher to dump sensitive data (like `revenue` and `commentsFromCLevel`) for every company in the system simultaneously.

***

### ## 🛡️ How to Fix It (The "Defense" Summary)

The fix isn't about hiding the IDs; it's about **Ownership Verification**.

The backend code must be updated to follow this logic:

1. **Extract** the user's `companyID` from the authenticated JWT.
2. **Look up** the requested Report ID in the database.
3. **Compare:** If `Report.companyID` `Report.companyID == User.companyID`, return the data.
4. **Reject:** If they don't match, return a `403 Forbidden` (not a `404`, as the object exists, the user just isn't allowed to see it).

***

***

This page covers **Broken Authentication** (CWE-307), a critical API vulnerability where authentication mechanisms are bypassed or exploited due to poor implementation.

The core of the lesson is that even if an API requires a login, it is "broken" if it doesn't protect itself against automated guessing or use strong enough password requirements.

***

#### Broken Authentication

The primary issue identified is the **Improper Restriction of Excessive Authentication Attempts**. The API endpoint `/api/v1/authentication/customers/sign-in` allows an attacker to try thousands of password combinations without being blocked or slowed down.

#### ## 🚀 The Attack Scenario

The lab walks through a multi-step exploitation process:

1. **Information Leakage:** By accessing the `/api/v1/customers` endpoint (a separate "Broken Object Property Level Authorization" flaw), the attacker gathers a list of target emails.
2. **Weak Password Policy Discovery:** By attempting to update a password, the researcher discovers the system only requires 6 characters and lacks complexity checks.
3. **Automated Brute-Force:** Using **ffuf**, the attacker performs a "Cluster Bomb" style attack, fuzzing both the `EMAIL` and `PASS` fields simultaneously.
   * **The Payload:** `{"Email": "EMAIL", "Password": "PASS"}`
   * **The Filter:** `-fr "Invalid Credentials"` (This hides all failed attempts so only the success stands out).
4. **Credential Discovery:** The attack successfully identifies a valid login: `IsabellaRichardson@gmail.com:qwerasdfzxcv`.

#### ## 🔑 Beyond Passwords: OTP & Security Questions

The page highlights that if passwords are too strong to guess, attackers shift focus to:

* **One-Time Passwords (OTP):** Often have low entropy (only 4 or 6 digits), making them easy to brute-force if rate limiting is absent.
* **Security Questions:** Often have predictable available answers.

#### ## 🛡️ Prevention & Mitigation

To secure an API against these attacks, the following must be implemented:

* **Rate Limiting:** Throttle or block IPs/accounts after a certain number of failed attempts.
* **Strong Password Policies:** Enforce a minimum of 12 characters with a mix of uppercase, lowercase, numbers, and symbols.
* **Multi-Factor Authentication (MFA):** Add a second layer of defense so a stolen password isn't enough to compromise the account.
* **Password History:** Prevent users from cycling through the same weak passwords.

***

***

### Broken Object Property Level Authorization

#### 1. Excessive Data Exposure (CWE-213)

This occurs when an API response includes sensitive data that the user is not supposed to see. Even if you are authorized to access the "Object" (the Supplier), you shouldn't see all of its "Properties" (private contact info).

* **The Scenario:** A customer uses the `/api/v1/suppliers` endpoint to look for vendors.
* **The Flaw:** The API returns the entire database record, including the supplier's **email** and **phone number**.
* **The Impact:** Customers can "bypass" the marketplace by contacting suppliers directly. This leads to a loss of marketplace fees and potential privacy violations.

#### 2. Mass Assignment (CWE-915)

This occurs when an API allows a user to modify sensitive object properties that should be "read-only" or restricted to administrators.

* **The Scenario:** A supplier wants to update their company profile via a `PATCH` request to `/api/v1/supplier-companies`.
* **The Flaw:** The endpoint blindly accepts any field in the JSON payload, including `isExemptedFromMarketplaceFee`.
* **The Exploit:** By sending `{"isExemptedFromMarketplaceFee": 1}`, the supplier can grant themselves a "tax-free" status on the platform.
* **The Impact:** Direct financial loss for the marketplace owners and unauthorized privilege escalation.

#### ## Comparison: Request vs. Response

| **Attack Type**             | **Direction**                            | **Logic Flaw**                                 |
| --------------------------- | ---------------------------------------- | ---------------------------------------------- |
| **Excessive Data Exposure** | **Response** (Server $\rightarrow$ User) | Sending too much info to the user.             |
| **Mass Assignment**         | **Request** (User $\rightarrow$ Server)  | Trusting the user to set their own attributes. |

***

***

**Unrestricted Resource Consumption**

**Unrestricted Resource Consumption** (specifically **CWE-400**). In the context of the SMS/OTP lab you're working on, it means the API is missing a "throttle" or "speed limit" on an expensive or sensitive operation.

***

#### ## 1. What does this attack actually mean?

Think of the API like a vending machine that gives out free samples.

* **A Secure API** says: "You can have one sample every 24 hours."
* **An Unrestricted API** says: "You can have as many as you want, as fast as you can press the button."

When you "spam" the OTP button, you are forcing the server to perform a **background task** (contacting an SMS gateway, generating a code, and sending a text) thousands of times. Because the developer didn't implement **Rate Limiting**, the server never tells you to "wait."

***

#### ## 2. What are the problems (Impact)?

If this vulnerability exists in a real application, it creates three major problems for the company:

**A. Financial Damage (The "Bill" Problem)**

Sending an SMS isn't free. Most companies pay a third-party service (like Twilio or AWS SNS) about **$0.01 to $0.05 per message**.

* An attacker using a script can send **10,000 requests per minute**.
* In just one hour, an attacker could cost the company **$6,000 to $30,000** in SMS fees alone. This is often called "SMS Premium Rate Fraud."

**B. Denial of Service (The "Congestion" Problem)**

SMS gateways have queues. If an attacker floods the system with 100,000 fake OTP requests for one email, the legitimate requests from _real_ users get stuck at the back of the line.

* **Result:** Real users can't log in because their OTP arrives 20 minutes late or never arrives at all.

**C. User Harassment (The "Spam" Problem)**

If you enter a victim's email address, their phone will be flooded with hundreds of text messages. This is a form of **denial-of-service against the human**, making their phone unusable and damaging the company's reputation.

***

#### ## 3. Why is it considered a "Vulnerability"?

It’s a vulnerability because it allows a **Standard User** to exert **Maximum Control** over the company’s resources.

In security, we follow the **Principle of Least Privilege**. A guest user should not have the "privilege" to spend $10,000 of the company's money just by clicking a button. When they can, the **Authorization** and **Availability** pillars of security are broken.

***

#### ### 🛠️ Summary Table

| **Problem**             | **Type of Risk** | **Real-World Consequence**                            |
| ----------------------- | ---------------- | ----------------------------------------------------- |
| **High Costs**          | Financial        | Massive bills from SMS providers.                     |
| **System Lag**          | Operational      | Legitimate users can't receive their login codes.     |
| **Reputation**          | Brand Risk       | Users think the app is broken or "spammy."            |
| **Resource Exhaustion** | Technical        | Database or SMS queues crash due to too many entries. |

***

***

#### Broken Function Level Authorization

**Broken Function Level Authorization (BFLA)**. If BOLA (which you studied earlier) is about accessing someone else's **data**, BFLA is about accessing **features or actions** that you shouldn't be allowed to use.

Think of it this way:

* **BOLA:** You are a customer allowed to view your own orders, but you find a way to view _another_ customer's orders.
* **BFLA:** You are a customer, but you find a way to use the **"Delete User"** or **"View All Discounts"** button that only an Admin should see.

***

#### ## 💀 The Core Vulnerability (BFLA)

BFLA happens when the server fails to verify if a user has the right "clearance" (Role or Permission) before running a specific function.

In this lab scenario:

1. **The Goal:** Access `/api/v1/products/discounts`.
2. **The Requirement:** The API documentation says you need the role `ProductDiscounts_GetAll`.
3. **The Reality:** Your user has **zero roles**, yet when you send the request, the server says "Sure, here you go!" and gives you the data anyway.

The developer created the "Staff Only" door but forgot to actually put a lock on it.

***

#### ## ⚖️ BOLA vs. BFLA: The Difference

This is a very common question in certification exams like the CWES.

| **Vulnerability**   | **Focus**           | **Example**                                                                |
| ------------------- | ------------------- | -------------------------------------------------------------------------- |
| **BOLA** (Object)   | **Data**            | Accessing `GET /api/v1/user/123` when you are user `456`.                  |
| **BFLA** (Function) | **Action/Endpoint** | Accessing `GET /api/v1/admin/export-all` when you are just a regular user. |

***

#### ## 🚀 Why is this a problem?

* **Information Disclosure:** In this specific lab, you're seeing product discounts. In a real company, this could be seeing employee salaries, private financial reports, or "unreleased" internal data.
* **Horizontal & Vertical Escalation:** If you can access an admin function, you have moved "up" the chain of command (Vertical Escalation).
* **Business Logic Manipulation:** If a customer can see all discounts, they can wait for the biggest drop or find codes that weren't meant to be public yet, hurting the company's profit.

***

#### ## 🛡️ How to Fix It

The fix is **Role-Based Access Control (RBAC)** implemented at the code level. Every time an endpoint is called, the code must perform a "Gatekeeper" check:

1. **Identify:** Who is making the request? (From the JWT).
2. **Verify:** Does this specific user ID have the `ProductDiscounts_GetAll` role in the database?
3. **Action:** If yes, show data. If no, return a **403 Forbidden**.

***

***

#### Unrestricted Access To Sensitive Business Flows

**Business Logic Vulnerability** known as **Unrestricted Access to Sensitive Business Flows**.

While other attacks like SQL Injection or BOLA are purely technical "bugs," this attack focuses on exploiting the **logic and processes** of the business itself. It’s about using the application exactly how it was designed, but in a way that is unfair or harmful to the company’s bottom line.

***

#### ## 🧠 What does this attack actually mean?

Every business has a "flow"—a sequence of steps or a set of rules for how things should work (e.g., "Discounts are for loyal customers only" or "Users can only buy 5 items at a time").

This vulnerability occurs when an API exposes data or functions that allow an attacker to "see behind the curtain" and manipulate that flow. In your lab's case:

1. **The Leak:** You used a previous bug (BFLA) to see the discount schedule.
2. **The Abuse:** You now know exactly **when** a product will be at its cheapest (e.g., 70% off).
3. **The Flow Break:** Instead of buying a product when you need it, you wait for the "secret" discount date, buy the entire stock, and ruin the marketplace's profit margins.

***

#### ## 💀 The Problems: Why is this dangerous?

If a business flow is unrestricted, it leads to several non-technical but "expensive" problems:

* **Inventory Exhaustion (Scalping):** An attacker can wait for the 70% discount, buy all 500 units of a product in one second, and then resell them on eBay for a profit.
* **Predatory Timing:** Competitors could use this API to see your upcoming sales and lower their prices just enough to steal your customers.
* **Loss of Revenue:** The marketplace makes money through fees and full-price sales. If everyone waits for the leaked 70% discount, the marketplace loses its ability to stay profitable.
* **Market Manipulation:** By controlling the supply of a discounted item, an attacker effectively controls the price of that item for everyone else.

***

#### ## 🔗 The "Kill Chain" (How it connects)

This section of the lab is showing you how vulnerabilities "stack" together:

1. **Step 1 (BFLA):** You accessed an endpoint you shouldn't have (`/api/v1/products/discounts`).
2. **Step 2 (Sensitive Flow):** That endpoint gave you the "secret" business schedule.
3. **Step 3 (Resource Consumption):** If the purchase button doesn't have a "Rate Limit," you can use a script to buy the whole warehouse in 5 seconds.

***

#### ## 🛡️ How to Fix It

The solution isn't just about "patching a bug"; it's about **defensive design**:

* **Restrict the Data:** Only show a discount _when it is active_, never leak the future schedule to the public.
* **Implement Rate Limiting:** Even if someone knows about the discount, don't let one person buy more than 2 or 3 items at that price.
* **Role-Based Access (RBAC):** Ensure that only internal "Marketing Managers" can see the full discount table.

***

***

#### Server-Side Request Forgey

**Server-Side Request Forgery (SSRF)**, specifically targeting the server's local file system. In this scenario, the vulnerability is particularly dangerous because it allows you to trick the server into reading its own internal files and handing them over to you.

***

### ## 💀 What is SSRF?

At its core, SSRF occurs when an attacker can tell the server, "Go fetch the data at this location," and the server blindly obeys. Usually, this refers to fetching a remote website, but it can also be used to access internal services or local files that are not supposed to be public.

#### ### The "Kill Chain" in this Lab

This specific exploit is a "chained" attack. It doesn't just use one bug; it uses three:

1. **Mass Assignment (CWE-915):** You use the `PATCH` endpoint to manually overwrite the `CertificateOfIncorporationPDFFileURI` field.
2. **Protocol Manipulation:** Instead of a standard URL, you inject the **`file://` URI scheme**. This tells the backend to look at its own hard drive instead of the internet.
3. **Local File Inclusion (LFI):** When the `GET` endpoint tries to "show" you the certificate, it actually reads `/etc/passwd` (or any file you specified) and encodes it into Base64 for you to download.

***

### ## 🚀 Breakdown of the Attack

| **Phase**               | **Action**                        | **Technical Result**                                                         |
| ----------------------- | --------------------------------- | ---------------------------------------------------------------------------- |
| **1. The Setup**        | Identify the `fileURI` field      | You realize the server stores paths as URIs.                                 |
| **2. The Injection**    | `PATCH` with `file:///etc/passwd` | You've successfully changed the server's pointer to a sensitive system file. |
| **3. The Trigger**      | Call the `GET` endpoint           | The server's internal function reads the file you pointed to.                |
| **4. The Exfiltration** | Decode the Base64                 | You now have the actual text of the server's configuration or user files.    |

***

### ## ⚠️ Why is this so dangerous?

While `/etc/passwd` is a classic "proof of concept" (showing you a list of users), a real-world attacker would go much deeper:

* **Cloud Metadata:** If the server is on AWS/Azure, an attacker could use SSRF to reach `http://169.254.169.254` to steal temporary **IAM credentials** and take over the entire cloud infrastructure.
* **Internal Scanning:** The attacker can use the server to scan the internal network (bypass firewalls) to find unauthenticated databases (like Redis or MongoDB) that aren't reachable from the internet.
* **Source Code Theft:** By pointing the URI to the application's own directory (e.g., `file:///app/appsettings.json`), they can find database passwords and API keys.

***

### ## 🛡️ How to Fix It (The Prevention)

The developers made a classic mistake: they **trusted the user's URI**.

1. **Strict Allow-listing:** The API should only allow URIs that start with a specific "safe" prefix (like `https://trusted-storage.com/`).
2. **Block Dangerous Schemes:** The backend should explicitly reject any input containing `file://`, `gopher://`, `dict://`, or even `http://localhost`.
3. **Path Sanitization:** Before reading a file, the code should verify that the file sits inside the designated `/wwwroot/SupplierCompaniesCertificatesOfIncorporations/` folder and nowhere else.

***

***

#### Security Misconfiguration

**Security Misconfiguration**, focusing heavily on **SQL Injection (SQLi)** within an API context. It also touches on how missing or weak **HTTP Security Headers** can open doors to other attacks like **CSRF**.

The core vulnerability exists because the API takes user input (the `{Name}` parameter) and glues it directly into a SQL query string. This allows an attacker to "break out" of the intended search and run their own database commands.

#### ### 1. The Detection Phase

The first sign of trouble is the "Error Message." When the researcher enters `laptop'`, the server crashes because the trailing single quote creates a syntax error in the SQL engine.

* **Intended Query:** `SELECT COUNT(*) FROM products WHERE name LIKE '%laptop%';`
* **Broken Query:** `SELECT COUNT(*) FROM products WHERE name LIKE '%laptop'%';` (The extra quote breaks the string).

#### ### 2. The Exploitation Phase (The Tautology)

The researcher uses the payload: `laptop' OR 1=1 --`. This is a classic **Tautology-based attack**.

* **The Logic:** In SQL, `1=1` is always **True**.
*   **The Result:** The query becomes:

    $$SELECT \text{ COUNT}(*) \text{ FROM products WHERE name LIKE } '\%laptop' \text{ OR } 1=1 \text{ --}\%';$$

Because `OR 1=1` is true for every single row in the database, the `WHERE` clause stops being a filter. Instead of counting only laptops (18), the server counts **every record in the table (720)**. The `--` is a comment that hides the rest of the developer's original code, preventing further syntax errors

### ## 🌐 Secondary Attack: HTTP Header Misconfiguration

The page also notes that APIs aren't just vulnerable through their parameters, but also through their **Response Headers**.

#### ### CORS Misconfiguration

If an API has a weak `Access-Control-Allow-Origin` policy (like setting it to `*`), any website on the internet can make requests to the API on behalf of a logged-in user. This is a primary cause of **Cross-Site Request Forgery (CSRF)**.

### ## 🛡️ The Prevention Strategy

The lab outlines two main ways to stop these misconfigurations:

1. **Parameterized Queries (Prepared Statements):** This is the "Gold Standard." Instead of building a string, the developer sends the query template and the data separately. The database then treats the input strictly as **data**, never as **executable code**.
2. **Using an ORM:** Tools like Entity Framework or TypeORM usually handle this safely by default.
3. **Secure Headers:** Implementing headers like `Content-Security-Policy`, `X-Content-Type-Options`, and strict CORS rules to ensure only trusted sources can interact with the API.

#### ### 🎯 Key Takeaway

A **Security Misconfiguration** isn't always a "broken" feature. Often, it's a feature that works _too well_ or is left in a "default" state that is too trusting of the user. In this lab, the "Product Count" feature works exactly as requested, but it trusts the user's string enough to let them rewrite the database logic.

***

***

#### Improper Inventory Management

**Improper Inventory Management**. In the world of APIs, it is often referred to as "Zombie APIs" or "Shadow APIs."

It happens when a company releases new versions of its software (like moving from `v0` to `v1`) but forgets to turn off or secure the old versions.

***

#### ## 💀 What is the core of this attack?

As an API matures, developers add new security features and fix bugs. However, the old versions (`v0`, `beta`, `test`) often remain sitting on the server.

**The Problem:** While the new version (`v1`) might have strong authentication and data filtering, the old version (`v0`) likely uses outdated security logic or, as seen in your lab, **no authentication at all.**

***

#### ## 🚀 Breakdown of the Lab Scenario

1. **Discovery:** You found a drop-down menu in Swagger that revealed a hidden `v0` definition.
2. **The Flaw (No Auth):** Unlike `v1`, which has lock icons (indicating JWT is required), `v0` has no locks. This means anyone on the internet can call these endpoints without a password.
3. **The Leak (Sensitive Data):** The `/api/v0/customers/deleted` endpoint is a "trash can" of old data. It not only leaks PII (Personally Identifiable Information) but also **password hashes**.
4. **The Impact:** Attackers can take those hashes and "crack" them offline. Since many people use the same password for years, an attacker can use a password from a "deleted" `v0` account to log into a "live" `v1` account.

***

#### ## ⚖️ Why is this so dangerous?

* **Forgotten Security:** Developers often stop patching old versions. A vulnerability fixed in `v1` might still be wide open in `v0`.
* **Larger Attack Surface:** Every extra version is a new set of doors for an attacker to kick.
* **Documentation Leaks:** Swagger and Postman collections often make it trivial for an attacker to map out these forgotten versions.

***

#### ## 🛡️ How to Prevent It

To stop this, organizations must treat their API inventory like a physical warehouse:

* **Sunset Policy:** Create a strict date for when old versions will be turned off (e.g., "v0 will be deleted 3 months after v1 launches").
* **Unified Auth:** Ensure that security headers and authentication are applied at the **Gateway level**, so even if a developer forgets a lock on an endpoint, the system blocks the request.
* **Inventory Scanning:** Use tools to find "hidden" versions or subdomains that aren't officially documented

***

***

### Unsafe Consumption Of API'S

**Unsafe Consumption of APIs** (CWE-1357).

It happens when a developer assumes that data coming from a "trusted" third-party API (like Google, Stripe, or a partner service) is safe and doesn't need to be checked. This "blind trust" can be exploited to bypass your own security controls.

***

#### ## Unsafe API Consumption: Cause and Prevention

| **Vulnerability Type**         | **How the Vulnerability Happens**                                                                                        | **How to Prevent It**                                                                                                             |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------- |
| **Insecure Transmission**      | APIs communicate over unencrypted HTTP channels, allowing data to be intercepted via Man-in-the-Middle (MITM).           | **Encrypt Everything:** Use TLS/HTTPS for all internal and external API-to-API communication.                                     |
| **Inadequate Data Validation** | The backend accepts data from an external API without sanitizing it, leading to SQLi, XSS, or RCE in downstream systems. | **Zero Trust Validation:** Treat third-party data as "untrusted" user input. Sanitize and validate every field before processing. |
| **Weak Authentication**        | APIs use hardcoded keys, weak tokens, or no authentication at all when talking to "internal" or "partner" services.      | **Robust Auth:** Implement strong authentication (like OAuth2 or mTLS) for every service-to-service connection.                   |
| **Insufficient Rate-Limiting** | One API floods another with requests, either by mistake or through an attack, causing a Denial-of-Service (DoS).         | **Throttling:** Implement rate-limiting on both the "sending" and "receiving" sides of the API interaction.                       |
| **Inadequate Monitoring**      | There is no logging for background API-to-API calls, making it impossible to see when an integration is being abused.    | **Full Observability:** Log and monitor all inter-service traffic to detect anomalies or failed authentication attempts.          |

***

#### ## The "Blind Trust" Problem

The most important takeaway here is that **reputation $\neq$ security**. Just because an API belongs to a reputable organization doesn't mean the data it sends you is safe.

> \[!CAUTION]
>
> If a partner API is compromised, they could send malicious payloads (like a SQL injection string) to your API. If you don't validate that data, your system will be the one that gets breached.
