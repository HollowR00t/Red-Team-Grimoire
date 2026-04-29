# Manual API Enumeration (Bypassing Anti-Bot Filters)

**Tactic:** Recognition/Enumeration
**Tool:** `curl`
**Scenario:** Extracting information (such as version or configuration) from *endpoints* of a REST API that blocks direct requests from scannig tools or the terminal.

## 1. The Problem: Server Defenses 
When attempting to query an API directly (`curl http://target/api/version`), it is common to encouter defensive barriesrrs configured by the developers:
* **SSL Error:** The server uses a self-signed certificate, and the connection is rejected.

* **400 Error (Bad Request):** The server requires a specific header (such as `X-Requested-With`) to ensure that the request comes from the legitimate web application and not from an automated script or a CSRF attack.

## 2. Execution: The Surgical Command
To force the server to respond, we must forge the HTTP headers and silence the local security warnings.

**Tactical Command**
```bash
curl -k -s -H "X-Requested-With: OpenAPI" https://<IP>/api/server/version
```

Command Anatomy:
-k (--insecure): Ignores SSL certificate validation (vital on CTF machines or internal environments).

-s (--silent): Hides the curl progress bar so that only the clean API response is printed in the terminal.

-H "Header: Value": Injects custom headers. Forging X-Requested-With (with values like OpenAPI or XMLHttpRequest) is usually enough to bypass the API's header.

## 3. Tactical Notes and Evasion
[!TIP] Reverse Engineering Exploits
If you find a public CVE but your manual API enumeration fails, read the exploit source code in Python/Ruby. Exploit creators always include the exact HTTP headers (headers={...}) that the server requires to function.
Copy those headers and use them in your curl requests.

[!WARNING] Beware of Line Breaks (%Symbol)
If you see a % symbol at the end of the response when extracting data from an API using curl (e.g. 4.4.0%), it's not part of the data. It's your terminal (e.g. ZSH) telling you that the server didn't send a line break (\n) at the end of the message. Ignore the % when copying version or hashes.
