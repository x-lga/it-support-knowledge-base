# SEC-001 - Ransomware: Initial Triage for L1

**Article ID:** SEC-001
**Category:** Security - Incident Response
**Severity:** P1 always
**Cert alignment:** CompTIA Security+
**Last verified:** 2026-07

---

## The Three Rules Before You Do Anything Else

**Rule 1: Do not shut down the machine.**
Ransomware often runs entirely or partially in memory. Powering off destroys
volatile memory that forensics needs. Some ransomware strains also encrypt
partially and the encryption can be reversed from memory artefacts. If the
machine has BitLocker and ransomware has encrypted the keys, powering off
may lock you out of the disk permanently.


**Rule 2: Isolate immediately - from the network, not from power.**
Disconnect the ethernet cable and disable WiFi. Every second of network
connectivity allows the ransomware to spread laterally, encrypt more files,
or exfiltrate data. Isolation stops the spread. Power-off destroys evidence.

**Rule 3: Call L2 Security by voice before doing anything further.**
Ransomware is a P1 that requires a security incident response, not a support
ticket. Your role is to isolate and report, not to investigate and remediate
independently. Attempting remediation without L2 involvement frequently makes
the situation worse - recovery tools run against the wrong strain, encryption
spreads further, or forensic evidence is destroyed.

---

## Signs That This Is Ransomware (Not General Malware)

| Indicator | Description |
|-----------|-------------|
| File extensions changed | Documents renamed to .locky, .wcry, .encrypted, .XXXX, etc. |
| Ransom note files | README.txt, !!! YOUR FILES ARE ENCRYPTED !!!.txt, DECRYPT_INSTRUCTIONS.html |
| Desktop background changed | Replaced with ransom demand image |
| Many files modified at same time | File system shows thousands of files modified in the last few minutes |
| Applications crashing | Applications cannot open files because data files are encrypted |
| "Shadow copies deleted" in Event Log | Event 524 (VSS) or command `vssadmin delete shadows` in Security log |

---

## Immediate Actions (L1 Scope - Do These in Order)

```
1. PHYSICALLY disconnect the ethernet cable from the affected machine
   (Do not use software disconnect - malware may prevent it)

2. DISABLE WiFi (toggle the physical WiFi switch if present, or Fn+WiFi key)

3. CALL L2 Security by phone - do not wait for a ticket response
   State: "I have a suspected ransomware infection on [machine name].
          I have isolated it from the network. I need you now."

4. DO NOT:
   - Run antivirus scans (may delete files needed for recovery)
   - Attempt to decrypt files yourself
   - Pay the ransom (escalation decision - not L1)
   - Restart or shut down
   - Connect the machine to any other network
   - Copy encrypted files to a USB drive (spreads the infection)

5. DOCUMENT while waiting for L2:
   - Machine name and user
   - Exact time the user noticed and/or the time file modification timestamps show
   - What the user was doing (email attachment, browser download, USB?)
   - Any ransom note text or filename (photograph with phone - do not email)
   - Which file types appear to be affected
   - Are any other machines showing similar symptoms?
```

---

## Identification (For L2 Reference - L1 Should Not Act on This Alone)

Once L2 has taken over, identification of the ransomware strain determines
whether decryptors exist:

```
No More Ransom Project: https://www.nomoreransom.org
  Upload a ransom note or sample encrypted file
  The tool identifies the strain and provides decryptors if available

ID Ransomware: https://id-ransomware.malwarehunterteam.com
  Upload encrypted file or ransom note
  Returns strain identification
```

**Why strain identification matters before any remediation:**
Some ransomware strains encrypt with known-weak keys and free decryptors exist.
Running remediation before identification destroys the encrypted files before
the decryptor can recover them. Identification first, remediation second.


---

