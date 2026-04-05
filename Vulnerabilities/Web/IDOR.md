# Insecure Direct Object Reference (IDOR)

## 1. Definition
An IDOR occurs when a web application exposes a direct reference to an internal object (such as a file, database, or record) through user input, without first verifying whether the user has authorization to access that object.

> **Tactical Analogy:** It's like handing over ticket #2 in the cloarkroom of a restaurant, but asking the manager to hand you coat #0 and having him do so without verifying your identity.

## 2. Impact
**Information Leakage:** Unauthorized access to other users' confidential data (messages, invoices, network captures).

**Data Modification:** Possibility of altering or deleting foreign records.

**Privilege Escalation:** If the exposed object belongs to an administrator, it can lead to total system compromise.

## 3. Attack Example ("Cap" Machine)
During the resolution of the Cap machine, a vulnerable endpoint was identified in the security dashboard that managed network captures (PCAP) using sequential IDs.

**Original request:**
The system assigned us the ID `2`.

``` http
GET /data/2 HTTP/1.1
Host: 10.129.25.132
```
**Exploitation:**
By altering the ID in the URL to 0, the server did not validate the permissions and returned the first network trap recorded on the system by the administrator.

```http
GET /data/0 HTTP/1.1
Host: 10.129.25.132
```
**Poc Result:**
Successful download of file 0.pcap, which when analyzed in Wireshark revealed FTP credentials in plain text.

## 4. Mitigation

**Authorization Validation:** Always verify at the backend level if the authenticated user has explicit permission on the requested resource. (Ex: `if(user.id == resource.owner_id)`).

**Indirect Identifiers:** Avoid exposing predictable or sequential IDs (1,2,3...). Instead, use random, alphanumeric UUIDs (universally Unique Identifiers) (Ex:/data/d3b07384-d9a7-40f1...)
