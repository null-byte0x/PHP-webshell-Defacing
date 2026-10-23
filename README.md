<div align="center">

<img src="https://capsule-render.vercel.app/api?type=venom&color=0:050505,40:1a0000,75:8b0000,100:ff2200&height=260&section=header&text=malware-analysis-lab&fontSize=52&fontColor=ffffff&animation=twinkling&fontAlignY=38&desc=IOCs%20%7C%20YARA%20RULES%20%7C%20BEHAVIORAL%20ANALYSIS%20%7C%20DETECTION%20ENGINEERING&descSize=12&descAlignY=62" width="100%"/>

[![Typing SVG](https://readme-typing-svg.demolab.com?font=Fira+Code&weight=700&size=18&duration=2000&pause=700&color=ff4444&center=true&vCenter=true&width=820&lines=%24+file+sample.bin+%3E+PE32%2B+executable+%5BMALWARE%5D;%24+strings+--all+%7C+grep+C2+%3E+iocs.txt;%24+yara+detections%2F*.yar+samples%2F+--print-strings;%5B%2B%5D+3+rules+matched+%7C+writing+report...;%24+observe.+analyze.+detect.)](https://git.io/typing-svg)

<br/>

![YARA Rules](https://img.shields.io/badge/YARA_Rules-4-ff2200?style=for-the-badge&logoColor=white)
![Samples](https://img.shields.io/badge/Samples_Analyzed-4-8b0000?style=for-the-badge&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-333333?style=for-the-badge)
![Lab](https://img.shields.io/badge/Lab-Isolated_VM-111111?style=for-the-badge&logo=virtualbox&logoColor=ff4444)

<br/>

**✦ ━━━━━━━ analyze · detect · document ━━━━━━━ ✦**

</div>

---

> [!CAUTION]
> **This repository contains documentation of malicious software.**
> It does **not** distribute live payloads. All content here — hashes, IOCs, behavioral notes, and YARA rules — exists solely for **detection engineering, threat intelligence, and security research** in isolated lab environments.

---

## `$ cat about.md`

This lab documents malware samples I've analyzed in a sandboxed environment. For each sample, I extract:

```text
 output per sample
 ├── 📋  behavioral summary (what it does, how it does it)
 ├── 🔎  IOCs (file names, mutexes, network indicators, registry keys)
 ├── 🧬  YARA detection rule (signature-based, evidence-backed)
 └── 🛡️  defensive notes (how to detect and respond)
```

All samples are sourced from **public malware repositories** (MalwareBazaar, VirusTotal, public GitHub releases). No original malware is hosted here. Samples are analyzed in an offline VM, snapshotted before execution, with no shared folders or clipboard.

---

## `$ ls -la detections/`

| Rule | Family | Severity | Technique |
|---|---|---|---|
| 🔴 [`telegram_rat.yar`](./detections/telegram_rat.yar) | Telegram-C2 RAT | `HIGH` | HTTP C2 via Bot API · shell exec · file exfil |
| 🔴 [`sinta_locker.yar`](./detections/sinta_locker.yar) | SintaLocker Ransomware | `CRITICAL` | AES-CBC encryption · VSS deletion · RDP backdoor |
| 🟠 [`xenotix_keylogger.yar`](./detections/xenotix_keylogger.yar) | Xenotix Keylogger | `HIGH` | `pyHook` · mutex · SMTP/FTP/Form exfil |
| 🔴 [`dtool_pro17.yar`](./detections/dtool_pro17.yar) | PHP Webshell | `CRITICAL` | RCE · file write · `passthru`/`system`/`popen` |

---

## `$ cat samples/index.md`

Full analysis for each sample lives in [`SAMPLES.md`](./SAMPLES.md). Here's the quick summary:

<table>
<tr>
<td width="50%">

### 🐍 `telegram_rat.py`
**Family:** Telegram-C2 RAT  
**Language:** Python 3  
**C2:** `api.telegram.org` Bot API  

Key capabilities:
- Arbitrary shell command execution
- Screenshot capture & upload
- File exfiltration via `sendDocument`
- IP geolocation fingerprinting

</td>
<td width="50%">

### 🔐 `sinta_locker.py`
**Family:** SintaLocker Ransomware  
**Language:** Python 2  
**Crypto:** AES-256-CBC (per-file key, irreversible)  

Key capabilities:
- Encrypts ~250 file extensions → `.sinta`
- Deletes shadow copies, breaks boot recovery
- Opens RDP backdoor, creates hidden admin
- Disables Task Manager, RegEdit, CMD

</td>
</tr>
<tr>
<td width="50%">

### ⌨️ `xenotix_keylogger.py`
**Family:** Xenotix Keylogger  
**Language:** Python 2  
**Hook:** `pyHook` `WH_KEYBOARD_LL`  

Key capabilities:
- Global keylogging via Windows low-level hook
- Mutex single-instance guard
- Exfil via Google Forms / SMTP / FTP
- Hidden console window

</td>
<td width="50%">

### 🌐 `dtool_pro17.php`
**Family:** PHP Webshell  
**Language:** PHP  
**Author:** r3v3ng4ns (public release)  

Key capabilities:
- RCE via `passthru`/`system`/`exec`/`popen`
- File write, mass overwrite, defacement
- RFI/LFI deployment vector
- `@error_reporting(0)` evasion

</td>
</tr>
</table>

---

## `$ yara detections/*.yar samples/ --stats`

```
┌──────────────────────────────────────────────────────────────┐
│  Rule                          │ Strings  │ Conditions       │
├──────────────────────────────────────────────────────────────┤
│  Python_Telegram_RAT_Machine1337 │  13    │  4 condition branches │
│  Python_Ransomware_SintaLocker   │  20    │  4 condition branches │
│  Python_Xenotix_Keylogger        │  16    │  6 condition branches │
│  PHP_Webshell_DtoolPro17         │  17    │  5 condition branches │
└──────────────────────────────────────────────────────────────┘
  Total rules: 4  |  Total string signatures: 66
```

To run all rules against a directory:
```bash
yara -r detections/*.yar /path/to/samples/
```

To test a single rule with string output:
```bash
yara -s detections/telegram_rat.yar /path/to/sample.py
```

---

## `$ cat lab_setup.md`

```text
 lab environment
 ├── 🖥️  Host: your physical machine (analysis never touches this)
 ├── 🔒  VM: isolated VirtualBox / VMware guest
 │    ├── network: host-only adapter (no internet)
 │    ├── shared folders: DISABLED
 │    ├── clipboard: DISABLED
 │    └── snapshot: taken before every sample
 ├── 📡  Traffic capture: Wireshark on host-only interface
 └── 🧹  After session: revert snapshot · review pcap · update YARA
```

**Safety checklist before any session:**
- [ ] VM is fully isolated — host-only network, no internet
- [ ] Snapshot taken of clean VM state
- [ ] No real credentials in any sample config
- [ ] Samples stored outside synced folders (no Dropbox/OneDrive)
- [ ] Wireshark running on host-only interface for traffic capture

---

## `$ cat repo_layout.md`

```
malware-analysis-lab/
│
├── README.md               ← you are here
├── SAMPLES.md              ← full IOC tables, hashes & defensive notes
│
└── detections/
    ├── telegram_rat.yar        ← Telegram-C2 RAT detection
    ├── sinta_locker.yar        ← SintaLocker ransomware detection
    ├── xenotix_keylogger.yar   ← Xenotix keylogger detection
    └── dtool_pro17.yar         ← PHP webshell detection
```

---

## `$ cat disclaimer.txt`

> [!WARNING]
> **Read before using anything in this repository.**

- ✅ Use these rules and IOCs to **detect and defend** against the documented families
- ✅ Use this research for **learning, CTFs, and lab practice** in isolated environments
- ❌ Do **not** use this material to attack systems you don't own
- ❌ Do **not** deploy any technique documented here without explicit written authorization

```
The author (null-byte0x) holds zero liability for misuse of any
material in this repository. All research is for educational and
defensive security purposes only.
```

---

<div align="center">

[![null-byte0x](https://img.shields.io/badge/by-null--byte0x-111111?style=for-the-badge&logo=github&logoColor=ff4444)](https://github.com/null-byte0x)
[![Malware Analysis](https://img.shields.io/badge/type-Malware_Analysis_Lab-8b0000?style=for-the-badge&logoColor=white)]()

[![Typing SVG](https://readme-typing-svg.demolab.com?font=Fira+Code&weight=600&size=14&duration=2500&pause=900&color=ff4444&center=true&vCenter=true&width=600&lines=0x00+Observe.;0x01+Analyze.;0x02+Detect.;0x03+Document.;End+of+transmission._)](https://git.io/typing-svg)

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:ff2200,50:5c0000,100:050505&height=140&section=footer" width="100%"/>

</div>
