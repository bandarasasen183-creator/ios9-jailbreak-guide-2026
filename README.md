# The Modern Guide to iOS 9 Jailbreaking & Sideloading (2026 Edition)
Last Updated: September 2026

*(Want to see every single error, typo, and roadblock I hit along the way? [Read my Full Detailed Developer Log here](iOS9_Detailed_Journey_Log.md))*

Welcome! If your trying to jailbreak an iOS 9 device (like an iPad 2) or sideload custom .ipa files today, youve probably realized that almost all of the tutorials online no longer work at all. 

This repository documents the modern workarounds for all the silent security updates and server shut downs Apple has implemented over the years. 

Just a quick note: this exact method worked perfectly for me, but I honestly cannot guarantee this will work for everyone around the world. These legacy servers and methods change all the time. If you notice any changes or find new fixes, please reply or update this repo so we can help other people out.

## Essential Links & Websites Used
* Carbon Jailbreak / Web Exploits: `https://jailbreaks.app/legacy.html`
* Legacy iOS Kit (Linux/Mac Tool): `https://github.com/LukeZGD/Legacy-iOS-Kit`
* AppSync Unified Official Repo: `https://cydia.akemi.ai/`
* Legacy Certificate Fixes: `http://cydia.invoxiplaygames.uk/certificates/`

---

## Why Old Methods Fail Today

If youve spent hours pulling your hair out, its not your fault. Here is why the old tutorials are totally broken:

1. macOS Sonoma/Sequoia Code-Signing Drops: Modern mac os has completely removed support for generating legacy SHA-1 code signatures. If you try to use Sideloadly or Xcode on a modern Mac to sign an iOS 9 app, the iPad will instantly reject it with `ApplicationVerificationFailed 0xe800801c` becuase it physically cannot read modern SHA-256 signatures.
2. The TLS 1.0 Server Blackout: If you managed to sideload an app and hit "Verify App" in Settings, you probably got a Network Connection Required error even if you have wifi. This is because Apples verification server (ppq.apple.com) disabled the old TLS 1.0 connection type that iOS 9 relies on. On-device verification of free Apple ID signatures is permenantly dead for iOS 9.
4. The AppSync Paradox: If you use AppSync to fake-sign a jailbreak kickstart app (like Phoenix), the app crashes when you reboot. AppSync needs the jailbreak to be awake, but you need the app to wake the jailbreak. It doesnt work.

---

## The Ultimate Solution (How to actually do it)

### Step 0: Fixing General iOS 9 Internet 
Before starting, alot of the internet is broken on iOS 9 because the Let's Encrypt root cert expired in 2021. 
1. Open Safari and go to **`http://cydia.invoxiplaygames.uk/certificates/`** (Make sure it is HTTP, not HTTPS).
2. Install the **ISRG Root X1** certificate to fix general HTTPS browsing. 

### Step 1: The Computer-Less Jailbreak
Forget the computer. Were bypassing Apples verification servers completely by using a WebKit browser exploit.

1. On your iOS 9 device, open Safari.
2. Navigate to **`https://jailbreaks.app/legacy.html`** and click the link for Carbon.
3. CRUCIAL STEP: Before you run it, tap the Share button in Safari and select Add to Home Screen. 
4. Why? This caches the raw HTML/JS exploit locally as a standalone Web App. You now have a permenant, offline jailbreak app that completly ignores Apples certificate system and never gets revoked.
6. Go to your home screen, tap your new Carbon app, and hit RUN.
7. The browser will glitch, the screen goes black, and the device will respring. Cydia is now installed. 
**Things to remember:** It probably wont work the first time. I once actually had to do it like 10 times for it to work. You might need to click it 1, 2, 3, 4, 5, 6, 7 times. Also make sure to close all your other Safari tabs and swipe away all other apps running in the background before you try it, or the memory gets messed up and it fails.

### Step 2: Enabling Sideloading (AppSync)
To install custom modified IPAs or old games, you need to disable signature checks.

1. Open Cydia.
2. Add Karens Repo: **`https://cydia.akemi.ai/`** 
3. Search for and install AppSync Unified (`ai.akemi.appsyncunified`).
4. Restart SpringBoard when prompted.

### Step 3: Sideloading Custom IPAs (The Linux Way)
Since modern macOS breaks legacy signatures, you should use a Linux environment (like Ubuntu or Zorin OS) with Legacy iOS Kit.

1. On your Linux machine, run: `git clone https://github.com/LukeZGD/Legacy-iOS-Kit.git` and run `./restore.sh`.
2. Connect your jailbroken iPad with USB.
3. If your custom .ipa is raw/unsigned, ideviceinstaller will fail with an entitlements error. 
5. To fix this, use Legacy iOS Kits menu: App Management -> Install IPA (appinst) OR Sideload IPA -> Install using Plumesign.
6. These tools will automatically generate a dummy signature and inject the required XML entitlements before pushing it over USB. AppSync will blindly accept it.

#### What about Windows or Mac?
I did all of this on a Linux machine (Zorin OS) and it worked perfectly for me. You might be able to use Windows or other types of Linux, but youll have to find your own documentation for setting those up. Here are some quick links to other developers tools if you want to try them. **Disclaimer: I did not try these other methods and I am not sure if they actually work or not, its just a quick link for you to visit.**
* Legacy iOS Kit (Windows/Mac Docs): `https://github.com/LukeZGD/Legacy-iOS-Kit/wiki`
* Sideloadly (Windows Alternative): `https://sideloadly.io/`

---

## ⚠️ Important Warnings
* Do NOT run Phoenix after running Carbon. If you run a second exploit while already jailbroken, you will cause a kernel panic, forcing a reboot and stripping your jailbreak state. Carbon is all you need.
* Do NOT use AppSync-signed apps to kickstart. Stick to your offline Carbon WebClip for waking up the iPad.

## Contributing
The legacy iOS community stays alive because we share what we learn. If you find a new workaround or a dead repo, please open an Issue or submit a Pull Request to keep this updated!

## Special Thanks & Credits
Huge shoutout and special thanks to the developers of Legacy iOS Kit (LukeZGD), jailbreaks.app, Karen (angelXwind) for AppSync, and InvoxiPlayGames for the certificate fixes. This guide is just built on top of all the incredible tools and guides you guys have already made. Im just trying to be kind to the community and put the puzzle pieces together for 2026. Thank you for keeping these old devices alive!
