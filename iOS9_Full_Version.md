# Full Deep-Dive: Hacking iOS 9 in 2026 (Prompts, Errors, and Fixes)
Last Updated: September 2026

*(Just want the quick steps without my life story? [Click here to read the Short Guide](README.md))*

So I spent hours on this and finally got it working. If your trying to jailbreak and sideload modified apps onto an iPad 2 (or any 32-bit iOS 9 device) today, you are going to run into a ton of walls. This is the extremely detailed, step-by-step log of every single small issue I ran into, all the wrong steps I took, and exactly how I fixed them.

Disclaimer: this worked for me on my specific setup, but I honestly cannot guarantee this will work for everyone around the world. Things break and change literally all the time with this old software. If there are any changes or you guys figure out a new workaround, please reply or update this guide!

---

### Attempt 1: macOS, Sideloadly, and the SHA-1 Nightmare
My goal was simple: take a modified `Phoenix6_modified.ipa` and sideload it onto an un-jailbroken iPad 2. 
I started on a modern Mac running macOS Sonoma/Sequoia using Sideloadly. Right off the bat, Sideloadly blocked me with a Patreon paywall because it needed something called "Anisette V3" to log into Apple's servers. 
**The Fix:** I actually had to go create a free Patreon account and link it just to bypass the login prompt.

Once it finally let me type in my Apple ID, it pushed the app to the iPad, but it immediately threw this exact hexadecimal error:
`ApplicationVerificationFailed 0xe800801c: No code signature found`

We dug into the logs and figured out the horrible truth: Modern macOS versions completely removed the backend libraries needed to generate legacy **SHA-1** code signatures. The iOS 9 kernel physically cannot read modern SHA-256 signatures, so it just rejects the file immediately thinking its unsigned. macOS is basically useless for iOS 9 sideloading now.

### Attempt 2: Switching to Linux & Plumesign Freezing
To get around the Mac issue, I switched over to my Zorin OS Linux laptop. I found a tool called **Legacy iOS Kit** because it uses a custom signing engine called `Plumesign` which forces perfect old-school SHA-1 signatures.

*(Note: You might be able to use Windows or other types of Linux for this. I did it on Linux so youll have to find your own documentation for Windows. Heres some quick links to other developers stuff like the [Legacy iOS Kit Windows Docs](https://github.com/LukeZGD/Legacy-iOS-Kit/wiki) or [Sideloadly](https://sideloadly.io/). I did not try them and I am not sure if they work or not, its just a quick link for you to visit).*

I cloned the repo and ran it:
`git clone https://github.com/LukeZGD/Legacy-iOS-Kit.git`
`cd Legacy-iOS-Kit`
`./restore.sh`

I went to `Other Utilities -> Sideload IPA -> Install IPA using Plumesign`. It asked for my Apple ID. I typed it in, but the terminal just froze on the line `GiveEndProvisioningData`. It literally just hung there.
**The Fix:** I hit `Ctrl+C` to kill the script, ran `./restore.sh` again, and tried the exact same thing a second time. The second time, Apple GrandSlam authenticated it and it saved my account! It signed the app perfectly and pushed it to the iPad! 

But... when I went to Settings -> Device Management to "Trust" my developer profile, the iPad gave this error:
`Unable to Verify "iPhone Developer..." A network connection is required`

Even though I was connected to wifi, it just kept failing. I even tried connecting to a mobile hotspot in case my router was blocking it. Didn't work.

### Attempt 3: The Crazy Python/SSH Certificate Bypass
I assumed the network error was happening because iOS 9 has expired root certificates (the famous Let's Encrypt bug). So I opened Safari and went to `http://cydia.invoxiplaygames.uk/certificates/` and installed the ISRG Root X1 cert. 
That fixed regular websites! But... the Apple ID *still* wouldnt verify. 

So then we realized we needed Apple's modern WWDR G3 and G4 certificates. The problem? Safari on iOS 9 crashes if you try to use modern URL shorteners (like TinyURL) to download them. 
So I downloaded the certs to my Mac, and ran a local python server to host them to the iPad:
`python3 -m http.server 8000`

But my Mac's firewall (and iCloud Private Relay) blocked the iPad from seeing the local IP address! So I literally had to create a public SSH tunnel on the Mac just to send the files to my iPad in the same room:
`ssh -o StrictHostKeyChecking=no -R 80:localhost:8000 nokey@localhost.run`

I typed the localhost.run URL into the iPad, downloaded both WWDR certificates, installed them, and rebooted. 
**It STILL failed.**
Turns out, Apples verification server (`ppq.apple.com`) has permanently disabled TLS 1.0/1.1 connections. Because iOS 9 is so old, it literally cannot negotiate modern TLS security handshakes. So on-device verification of free Apple ID signatures is 100% permanently dead. Period.

### Attempt 4: The Safari Carbon Exploit
Since I couldn't verify apps legitimately, I abandoned the computer entirely. I went to Safari on the iPad and went to `https://jailbreaks.app/legacy.html` to use the **Carbon** exploit.

Safari kept crashing over and over, saying `A problem occurred with this webpage so it was reloaded.`
This happens becuase 32-bit browser exploits need perfect memory alignment to work. 
**The Fix:** I had to go to Settings and clear Safari data. Then I had to close all other Safari tabs, and double-click the home button to force-close all other apps running in the background. Even then, it wouldn't work the first time. I literally had to click it 1, 2, 3, 4, 5, 6, 7 times... I think I once had to do it 10 times before it actually worked. But eventually it hit the memory offset, the screen glitched out, dimmed, the iPad kernel panicked, and it rebooted with a `Storage Almost Full` warning. Cydia was installed!

### Attempt 5: The Terminal Typos and AppSync Errors
With Cydia installed, I added Karens Repo (`https://cydia.akemi.ai/`) and installed AppSync Unified (`ai.akemi.appsyncunified`) to disable signature checks.

Then I went back to my Linux laptop to push my raw modified IPA file. I opened a new terminal and tried to use the raw tool:
`ideviceinstaller -1 '/home/tradingbot/Downloads/Phoenix6.ipa'`
It immediately gave me a `command not found` error. I realized I had to install it first:
`sudo apt install ideviceinstaller`
But even after that, it failed! Why? Because I accidentally typed the number `-1` instead of the letter `-i`! 

I fixed my typo and ran it again. It started installing (5%, 15%, 30%...), but at 40%, it threw this error:
`ERROR: Install failed. Got error "ApplicationVerificationFailed" with code 0x00000000: Application is missing the application-identifier entitlement.`

Turns out my raw modified IPA was totally stripped of its XML entitlement structure, so the iPad rejected it before AppSync could even intercept it.

### Attempt 6: The `appinst` SSH Failure
To fix the entitlement error, I went back into Legacy iOS Kit and tried using its special AppSync installer menu: `App Management -> Install IPA (appinst)`. 

The script tried to connect, but threw this error:
`Error connecting to device: Connection refused`
`[WARNING] appinst not detected. Please install appinst and OpenSSH first before using this option.`

So I had to go back to Cydia on the iPad, search for **OpenSSH** and install it, and then search for **appinst** and install it. I went back to Linux, ran it again, and... IT STILL FAILED at 40% with the exact same missing entitlement error! The raw file was just too broken.

### The REAL Fix for the IPA
I went back to the main menu and chose `Sideload IPA -> Install IPA using Plumesign`. Even though the device couldn't verify the Apple ID online, Plumesign generated the missing dummy `entitlements.plist` and wrapped the app in a structural signature. I pushed it over, and because AppSync was running, it blindly accepted it! The app appeared on my home screen and opened perfectly!

### The Final Problem: The AppSync Paradox
I thought I was completely done. I opened my custom Phoenix app, hit "Kickstart Jailbreak", and the iPad rebooted. But when it turned back on, Phoenix immediately crashed on launch!

**The Catch-22:** AppSync only works when the jailbreak is awake. Because I fake-signed Phoenix using AppSync, it reverted to an "unsigned" state on reboot. I needed Phoenix to wake the jailbreak, but Phoenix required the jailbreak to be awake to even open. Its a total paradox. 

Also, a warning: If you run Phoenix while Carbon is already active, you double-patch the kernel and cause a panic that forces a reboot anyway. I locked myself out doing this.

### The Ultimate Solution
I abandoned the sideloaded app for kickstarting entirely. 
I went back to Carbon in Safari, and instead of hitting run, I tapped **Share -> Add to Home Screen**.

This is the ultimate trick. It rips the raw HTML/JS exploit payload from the server and caches it permanently into the iPads local storage as a standalone Web App. You get a Carbon app icon on your home screen that completely ignores Apples certificates, doesnt need AppSync, and works 100% offline. 

If your iPad ever dies, you just tap Carbon, hit RUN, and your jailbreak is back. We totally conquered it.

### Special Thanks
A massive special thanks to every single website, developer, and older guide that paved the way for this. Legacy iOS Kit, jailbreaks.app, Karens AppSync repo, and InvoxiPlayGames—this is all built on top of your hard work. Just wanted to be kind to the community and share how I tied it all together. Thanks for everything you do!
