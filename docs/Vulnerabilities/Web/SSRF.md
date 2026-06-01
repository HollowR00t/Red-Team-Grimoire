# Server-Side Request Forgery (SSRF)

## Description

This occurs when an attacker abuses a server's functionality to force it to read or send data to an arbitrary URL. The server acts as a proxy, allowing the attacker to bypass firewalls, access internal networks (intranets), read local files or steal cloud credentials.

## Blind vs Response-Based SSRF

The impact is the same, but the way to confirm and explout it changes radically:

### 1. Response-Based SSRF (Regular/Non-Blind)

* **Behavior:** The response from the internal server or local file is directly reflected on the screen or in the HTTP response received by the attacker.

* **Explotation:** Direct. You read `/etc/passwd`, scan internal ports by viewing banners, or steal AWS tokens by viewing the JSON on the screen.

### 2. Blind SSRF

* **Behavior:** The server makes the HTTP request on the backend, but **doesn't** show you the result. At most, you see a change in response time or a simple HTTP code: `200 OK` vs `500 Error`.

* **Explotation (OOB - Out of Band):** You need the server to communicate with you . You force the target to make a request to a server you control (e.g., Burp Collaborator `interactsh`, webhook.site).

* **Confirmation:** If you see in your logs that the victim server's IP address attempted to resolve your DNS or made a GET request to your webhook, you confirm the blind SSRF.

---

## Cloud Metadata Explotation

In cloud environments, the magic IP address is `169.254.169.254`. If you can get an SSRF to point there, you can steal the credentials (tokens) that the virtual machine uses to authenticate itself against the rest of the cloud.

### AWS (Amazon Web Services)

* **IMDSv1 (Classic):** `http://169.254.169.254/latest/metadata/iam/security-credentials/`

* **IMDSv2 (Modern - Requires header injection):** If the server allows you inject or control HTTP headers (CRLF Injection), you first need to generate a token:

`PUT http://169.254.169.254/latest/api/token` with the header: `X-aws-ec2-metadata-token-ttl-seconds: 21600`

### Google Cloud Plataform (GCP)
*GCP always requires a specific header, unless you use the legacy v1beta1 endpoint.*

* **Endpoint:** `http://169.254.169.254/metadata/instance?api-version=2021-02-01`

* **Required Header:** `Metadata: true`

### DigitalOcean

* **Endpoint:** `http://169.254.169.254/metadata/v1.json`

---

## Filter Evasion (WAF Bypass and Logic)

Developers often block `127.0.0.1` or `localhost`. Here are the techniques to bypass those validations:

### 1. Ip Encoding (The Mathematical Trick)

Operating systems understand IPs in multiple formats, but text filters (Regex) do not.

* **Target IP:** `127.0.0.1`
* **Decimal:** `http://2130706433/`
* **Octal:** `http://0177.0.0.01/`
* **Hexadecimal:** `http://0x7f000001`
* **Hybrid/Shorthand:** `http://127.1/` or `http://0.0.0.0/`
* **Localhost IPv6:** `http://[::1]/`

### 2. Open Redirects (HTTP 30x)
If the filter verifies that the URL doesn't point to an internal network *before* making the request, it uses a redirect.

1. You create a server (or use a service) that responds with an `HTTP 302 Redirect` to `http://127.0.0.1/admin`.

2. You send your malicious URL (e.g., `http://yourdomain.com/redirect`).

3. The filter sees it and says, "it's an external domain, all good".

4. The backend HTTP client blindly follows the redirect and attacks the internal localhost.

### 3. DNS Rebinding (The Time Attack)

This is used when the server has strict validation: *Resolve the domain -> Checks if the IP is internal -> If it's safe, makes the HTTP request*.

* **The Tecnique:** You configure a custom DNS server with a TTL (Time To Live) of `0` seconds.

* **Step 1:** The victim server resolves `trap.yourdomain.com`. Your DNS responds with a safe IP (`8.8.8.8`). It passes the filter.

* **Step 2:** Fractions of a seconds later, the server is about to make the HTTP request. Since the TTL is 0, it queries the DNS again. This time your DNS responds with `127.0.0.1`.

* **Tools:** `rbndr.us` or the NCC Group's `singularity` tool.

## Explotation Checklist

- [] Test external URL injection (`http://yourinteractsh.com`) to confirm SSRF (Blind or Regular).

- [] Attempt to access local services on common ports: `http://127.0.0.1:22` (SSH), `80`, `8080`, `3306` (MySQL).

- [] In case of blocking, apply IP encoding (Decimal, Octal, Hex).

- [] Test for metadata leaks in the cloud (`169.254.169.254`) if the target is hosted on AWS/GCP/Azure.

