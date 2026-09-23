# Technical Report: iOS 9.3.5 Legacy Sideloading & Jailbreak Challenge

**Device:** iPad 2 (iPad2,4)
**OS:** iOS 9.3.5 (32-bit)
**Objective:** Achieve a full jailbreak and successfully sideload a custom modified application (`Phoenix6_modified.ipa`) bypassing modern Apple security and legacy limitations.

---

## Phase 1: The macOS Code-Signing Limitation
**The Approach:** Attempted to use Sideloadly on macOS to sign and push the IPA using a free Apple Developer account.
**The Issue:** Sideloadly threw an `ApplicationVerificationFailed 0xe800801c` (Invalid file / No code signature found) error. 
**Root Cause:** Modern macOS `codesign` binaries have deprecated support for generating legacy SHA-1 code signatures. The iOS 9 kernel physically cannot read modern SHA-256 code signatures, resulting in immediate structural rejection.
**The Fix:** Pivoted to a Linux environment using `Legacy iOS Kit`, which utilizes a custom signing engine (`Plumesign`) built to replicate legacy SHA-1 structures.

## Phase 2: Linux Sideloading & The Apple TLS Blackout
**The Approach:** Authenticated `Plumesign` with Apple's servers and pushed the SHA-1 signed IPA to the iPad over USB.
**The Issue:** The app appeared on the iPad, but attempting to "Trust" the developer profile threw a persistent error: `Unable to Verify... A network connection is required`.
**The Troubleshooting:** Attempted to manually install the `ISRG Root X1` and modern `Apple WWDR G3/G4` root certificates via a local Python HTTP server routed through a public SSH tunnel.
**Root Cause:** Even after updating the root certificates and clearing the `trustd` cache, the verification failed. Apple's `ppq.apple.com` verification servers have permanently disabled TLS 1.0/1.1 connections. On-device verification of free Apple ID signatures is now physically impossible for iOS 9.

## Phase 3: The Browser Exploit (Carbon)
**The Approach:** Abandoned Apple verification servers entirely and pivoted to a WebKit-based Safari exploit called **Carbon**.
**The Issue:** Safari repeatedly crashed with the error: `A problem occurred with this webpage so it was reloaded.`
**Root Cause:** WebKit exploits rely on precise memory corruption (use-after-free). On 32-bit legacy devices, memory fragmentation causes the exploit to miss target offsets frequently, triggering Safari's sandbox panic.
**The Fix:** Cleared Safari's Website Data, closed all background apps to defragment the heap, and brute-forced the exploit execution. 
**Result:** The exploit achieved kernel privilege escalation, executing a kernel panic/respring sequence and throwing the `Storage Almost Full` warning (confirming the Cydia bootstrap extraction). **Cydia was successfully installed.**

## Phase 4: AppSync Unified & The Sideload Paradox
**The Approach:** Installed **AppSync Unified** via Cydia to bypass signature checks and finally pushed the custom `Phoenix6_modified.ipa` using `Plumesign`. 
**The Issue:** The custom Phoenix app opened perfectly while the jailbreak was active. However, upon a full system reboot, the Phoenix app immediately crashed on launch.
**Root Cause (The Catch-22):** AppSync Unified is a mobile substrate extension that only functions when the jailbreak is active. Because the custom Phoenix app was fake-signed using AppSync, it reverted to an "unsigned" state upon reboot. We needed Phoenix to kickstart the jailbreak, but Phoenix required the jailbreak to already be active to open. 
**Additional Complication:** Attempting to run Phoenix's exploit *after* Carbon's exploit had already succeeded resulted in patching an already-patched kernel. This double-exploit collision caused an immediate kernel panic, forcing a reboot and continuously stripping the jailbreak state.

## Phase 5: The Ultimate Solution (Local WebClip)
**The Approach:** We abandoned the sideloaded Phoenix app as a kickstart mechanism. 
**The Fix:** We navigated to the Carbon exploit in Safari and utilized Apple's native **"Add to Home Screen"** feature. 
**The Result:** This action stripped the raw HTML/JS WebKit exploit payload from the host server and cached it permanently into the iPad's local storage as a standalone Web App container. 

## Final Status
The device is now fully jailbroken and untethered from a computer. The standalone Carbon Web App acts as a permanent, offline "Kickstart" mechanism. Because it operates inside a local web container, it completely bypasses Apple's certificate revocation checks, TLS 1.0 network drops, and the AppSync paradox. The challenge was completed with 100% success.
