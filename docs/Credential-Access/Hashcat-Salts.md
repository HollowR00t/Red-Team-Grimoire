# Hashcat: Cracking Hashes with Salt (SHA-256)

**Tactic:** Credential Access
**Tool:** Hashcat
**Scenario:** Database dump where the *Hash* and *Salt* are in separate columns.

## 1. File Preparation (Format)
Hashcat cannot read the hash and salt on separate lines. You must concatenate them in a plain text file using the standard colon format (':').

**Required Format:**
```text
<HASH>:<SAL>

Example
bbff8b0413949da762c8506c30ea080cf2db511d2b939f64124ddf0fb:caroline
```

## 2. Attack Modes (SHA-256)
The order in which the developer concatenated the password and the salt before encryption changes everything. There are two main variants for SHA-256:

Option A: Password + Salt (Mode 1410)
This is the most common method The system concatenates the password and then the salt sha256($pass.$salt).

Command:
```bash
hashcat -m 1410 hashes.txt /usr/share/wordlists/rockyou.txt
```
Option B: Salt + Password  (Mode 1420)
The system reversed the order of sha256($salt.$pass).

Command
```bash
hashcat -m 1420 hashes.txt /usr/share/wordlists/rockyou.txt
```

## 3. Tactical Notes on Evasion and Troubleshooting
[!WARNING] The Premature "Exhausted" Error.
If you launch the attack (for example, with mode 1410) and Hashcat terminates within seconds, returning the status "Exhausted" without providing the password, it doesn't mean the password isn't on RockYou.

It means that Hashcat's engine failed to structure the cipher. Immediately swith to reverse mode (1420) and launch the attack again.

[!TIP] Performance Optimization
If you're running this in a virtual machine (VM) and Hashcat complains about insufficient GPU resources, tou can force CPU usage by adding the `--force` flag.
