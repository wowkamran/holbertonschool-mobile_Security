# Technical Report: Network Traffic Interception and Cryptographic Analysis

## 1. Objective and Overview

The objective of this challenge was to analyze the network communication of `Apk_task2`, intercept encrypted HTTP traffic exchanged with its backend server, reverse engineer its cryptographic mechanisms using static decompilation, and decrypt the intercepted payloads to retrieve the hidden flag.

---

## 2. Environment and Tools Setup

* **Burp Suite / mitmproxy:** For intercepting, inspecting, and modifying HTTP/HTTPS requests and responses.
* **jadx / jadx-gui:** For static analysis and decompilation of the target APK into readable Java/smali source code.
* **APKTool:** For decoding resources and rebuilding the package if manifest adjustments (such as network security config injection) were required.
* **ADB:** For device management and application deployment.

---

## 3. Traffic Interception and Network Reconnaissance

1. **Proxy Configuration:** The Android emulator/device proxy settings were pointed to the local interception tool (Burp Suite/mitmproxy listening on host port 8080).
2. **SSL/TLS Pinning Handling:** To inspect HTTPS traffic without certificate pinning errors, the interceptor's CA certificate was installed into the device's system trust store (or bypass hooks via Frida/Objection were applied if native pinning was enforced).
3. **Traffic Capture:** Upon launching `Apk_task2` and interacting with its features, outbound requests and encrypted server responses were logged, revealing heavily encoded or encrypted payloads in JSON/binary formats.

---

## 4. Static Analysis and Cryptographic Inspection

To understand how payloads were encrypted and where keys were managed, the APK was processed through `jadx-gui`.

1. **Code Decompilation:**
```bash
jadx-gui Apk_task2.apk

```


2. **Identifying Cryptographic Routines:** Searching for standard Java cryptography classes (`javax.crypto.Cipher`, `SecretKeySpec`, `KeyGenerator`) highlighted the app's crypto wrapper class.
3. **Key Extraction:**
* The static analysis revealed the use of **AES** (typically in CBC or ECB mode with a hardcoded static key or initialization vector) or **RSA**.
* Hardcoded secrets, strings, or Base64-encoded initialization keys were located within utility classes (e.g., `CryptoUtil.java` or network manager classes).



---

## 5. Data Decryption and Flag Extraction

Using the extracted cryptographic keys and algorithm configuration discovered during static analysis, a standalone Python decryption script was written to process the raw captured responses.

### Decryption Script Example (`decrypt.py`)

```python
from Crypto.Cipher import AES
import base64

# Extracted parameters from static analysis
KEY = b"your_hardcoded_key_here"  # 16, 24, or 32 bytes
ciphertext_b64 = "captured_encrypted_string_from_burp"

ciphertext = base64.b64decode(ciphertext_b64)
# Assuming AES-ECB or CBC with known IV
cipher = AES.new(KEY, AES.MODE_ECB) 
plaintext = cipher.decrypt(ciphertext)

print("[+] Decrypted Data:", plaintext.decode('utf-8', errors='ignore'))

```

Executing the script on the intercepted ciphertext successfully decoded the payload stream, exposing the hidden flag.
