# Technical Report: Native Function Hooking in Android

## 1. Objective

Intercept and analyze the native JNI function `getSecretMessage` in `Apk_task1` using Frida to extract a hidden decrypted flag.

## 2. Tools Used

* **Frida & Frida-Tools:** Runtime instrumentation and script injection.
* **ADB:** Application management and log monitoring (`adb logcat`).
* **Objection:** Internal exploration and memory inspection.

## 3. Methodology

1. **Reconnaissance:** Installed and launched the target APK. Used Frida to inspect loaded symbols and locate the native library (`libnative-lib.so`) and target export.
2. **Frida Hooking Script:** Implemented a JavaScript hook using `Interceptor.attach()` to monitor the target native function:
```javascript
setTimeout(function() {
    Java.perform(function() {
        var target = Module.findExportByName("libnative-lib.so", "Java_com_example_apk_1task1_MainActivity_getSecretMessage");
        if (target) {
            Interceptor.attach(target, {
                onEnter: function(args) { console.log("[+] Intercepted getSecretMessage()"); },
                onLeave: function(retval) {
                    if (retval) {
                        console.log("[+] Flag: " + Memory.readUtf8String(retval));
                    }
                }
            });
        }
    });
}, 1000);

```


3. **Execution:** Spawned the app with the script attached:
```bash
frida -U -f <package_name> -l hook.js --no-pause

```



## 4. Results

The `onLeave` callback successfully read the memory pointer returned by the native function, revealing the plaintext flag.
