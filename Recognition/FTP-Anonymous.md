# FTP Anonymous Login

## 1. Definition

The FTP (File Transfer Protocol) protocol includes a legacy functionality called "Anonymous Access". This feature allows any user to connect to the server using the username `anonymous` without requering a valid password (usually left blank or any fake email is used).

## 2. Impact
Leaving this setting enabled by mistake (Misconfiguration) can lead to:

**Information Leakage:** Access to sensitive files, backups, SSH keys, or configuration files.

**Code Execution/RCE:** If the FTP directory is writable and linked to the web server, an attacker can upload a reverse shell (e.g.a `.php` file) and execute it from the browser.

## 3. Attack Example

During the enumeration phase, open port 21 (FTP) is identified. The interactive connection is attempted from the terminal:

``` bash
# Connection to the server
ftp 10.10.10.245

# When the user requests:
Name (10.10.10.245:root): anonymous

#When prompted for the password(leave blank and hit Enter)
Password:
```

Common answrs:
230 Login successful -> Vulnerable. Accesss granted. (Useful commands: ls -la, get file.txt).

530 Login incorret -> Secure. The administrator disabled anonymous access.

## 4. Mitigation

Explicitly disable `anonymous` access in the FTP server configuration file. For example, in `vsftpd` (usually in /etc/vsftpd.conf):
```Ini
anonymous_enable=NO
```
