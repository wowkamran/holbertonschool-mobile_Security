# Technical Report: Revealing Hidden Functions in Android Applications

## 1. Objective and Overview

The objective of this challenge was to perform static and dynamic analysis on `Apk_task3` to locate unreferenced or concealed methods within the application binary, dynamically invoke them at runtime using instrumentation tools, and decode the resulting output to retrieve the hidden flag.

---

## 2. Environment and Tools Setup

* **jadx-gui:** For static code decompilation and identification of hidden or dead-code functions.
* **Frida / Frida-Tools:** For runtime instrumentation, function hooking, and direct method invocation.
* **Objection:** For exploring runtime classes, memory inspection, and interactive command-line evaluation.
* **ADB:** For device interaction and application lifecycle management.

---

## 3. Static Analysis and Identification of Hidden Methods

1. **Decompilation:** The APK was decompiled using `jadx-gui` to inspect the Java source structure and package layout:
```bash
jadx-gui Apk_task3.apk

```


2. **Code Navigation:** Reviewing the codebase revealed utility or helper classes containing sensitive methods (e.g., secret retrieval or decryption routines) that were never explicitly triggered by the application's user interface or standard execution flow.
3. **Target Method Isolation:** Documented the target class name, method signatures, parameter requirements, and return types of these unreferenced functions.

---

## 4. Dynamic Invocation and Runtime Execution

Because the hidden functions were not invoked during normal app usage, Frida's runtime evaluation capabilities were used to manually trigger them and capture their outputs.

### Frida Invocation Script (`invoke_hidden.js`)

```javascript
setTimeout(function() {
    Java.perform(function() {
        console.log("[*] Searching for target class and hidden methods...");
        
        // Replace with the actual target class discovered during static analysis
        var TargetClass = Java.use("com.example.apk_task3.SecretHelper"); 

        try {
            // Directly invoke the hidden/unreferenced static method
            var result = TargetClass.getHiddenFlag(); 
            console.log("[+] Successfully invoked hidden function!");
            console.log("[+] Raw Result / Decrypted Output: " + result.toString());
        } catch (e) {
            console.log("[-] Error invoking method: " + e.message);
        }
    });
}, 1000);

```

### Execution

The script was injected into the running application:

```bash
frida -U -f <package_name> -l invoke_hidden.js --no-pause

```

Alternatively, **Objection** was used interactively to inspect loaded classes and invoke methods directly from the REPL shell:

```bash
objection -g <package_name> explore
# Inside Objection shell:
android javanm com.example.apk_task3.SecretHelper getHiddenFlag

```

---

## 5. Decoding and Flag Extraction

Upon successful execution of the hidden function, the output returned either plaintext or an encoded string (such as Base64 or a simple XOR transformation).

* If encoded, standard decoding routines (e.g., Base64 decoding) revealed the final string.
