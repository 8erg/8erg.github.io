+++
title = 'DLL Proxying with OneDrive'
date = 2025-07-27
draft = false
+++


## 1. PREFACE
---

To further weaponized my malware, i decided to implement a `DLL` proxying as it is widely used by threat actors and it also gives a lot of place for imagination and creativity. I will be continuing from where i left off with my `Sliver C2` stager that I was injecting while bypassing windows defender by using a combination of techniques (IAT Hiding & Obfuscation, NT API Hashing). In hopes of making my attack a little bit more stealthier.


## 2. Theory (fun part, yeah...?)
---

### What is a DLL?

A Dynamic Link Library contains code that can be reused by a program

- DLL are loaded into memory at runtime
- Each process started on windows uses them

It contains exported functions that can be used externally by other programs

### What is a DLL search order?

When a process is started it will start searching for the necessary `DLL` in the directory where it's located and if not found, it will then move onto other folders such as the system directory

{{< image src="/images/Pasted image 20250727190134.png" alt="20250727190134" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

### What is a DLL hijacking?

If an application load modules by specifying only their name instead of their full path and the `DLL` is not present in the first search candidate, it will try other directories and that's how an attacker could force the application to load his malicious `DLL` instead of the original one.

>[!warning]
>The problem with this is that the original exported functions of the original `DLL` will not be available/present which could break some essential functionality of the running process, which is not very good

### Why all these definitions?

You might be asking why all these definitions, simply because to understand how `DLL` Proxying works you need to grasp these little core concepts. Basically, it performs a `DLL` hijacking, but we're able to redirect the legit exported functions to the legit `DLL` so we don't break the functionality of the process and the same time we can execute our malicious code.

>[!warning]
>You could still slow down the process loading the `DLL` so it's recommended to perform
>+ `Remote Process Injection`
>+ `Remote Thread Creation`
>+ `Thread Hijack


## 3. Walkthrough
---

### Finding the executable and the DLL

+ Select an executable to perform the attack
+ Choose a `dll` that is used by it ( preferably one that do not import a lot of functions)

{{< image src="/images/Pasted image 20250725200025.png" alt="20250725200025" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

{{< image src="/images/Pasted image 20250725200148.png" alt="20250725200148" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

+ We can see that `secur32.dll` is being loaded from `C:\Windows\System32` so let's see if we put one in the same directory as the executable

+ Writing a custom temporary `secur32.dll`

{{< image src="/images/Pasted image 20250725202240.png" alt="20250725202240" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

{{< image src="/images/Pasted image 20250725202530.png" alt="20250725202530" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

+ This way `dll` is not that good because i had to copy is not by default in the same folder as `onedrive.exe` it would be better to identify another `dll` that is besides other `dll` to better camouflage
+ Because as you see it i had to rename the `secur32.dll` to `securr32.dll` to proxy the execution from my malicious `dll`

+ So what i did is that I moved the `onedrive.exe` to another folder to see how it would go about finding `dll` that it needs
+ But now I'm just going to go for the `OneDriveServiceUpdater.exe`, but one thing to note is that on every version there's a new folder being created inside `Microsoft Onedrive` so on a new version they will be a new folder with a new version of `OneDriveServiceUpdater.exe`, so in a real world scenario it would be good to check the registry to know which folder we would have to drop our `dll` into

{{< image src="/images/Pasted image 20250726030951.png" alt="20250726030951" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

+ basically i would have to read this registry in order to know in what folder i should drop it and also probably to move itself for persistence, if we ever need to inject the stager beacon again, but for now, we'll drop it manually

{{< image src="/images/Pasted image 20250725210512.png" alt="20250725210512" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

+ also there's secur32.dll being needed so we can try our previous proof-of-concept

{{< image src="/images/Pasted image 20250725210411.png" alt="20250725210411" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

+ so i renamed the real `secur32.dll` to `updater.dll` and my malicious `dll` to `secur32.dll` and I move them into `C:\Program Files\Microsoft OneDrive\25.122.0624.0004` and guys it works

{{< image src="/images/Pasted image 20250725211358.png" alt="20250725211358" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

{{< image src="/images/Pasted image 20250725211755.png" alt="20250725211755" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}


### Writing our malicious implementation

So as i previously was able to bypass windows defender with combining different techniques such as ((IAT Hiding & Obfuscation, NT API Hashing) so why change something that works?

👉 You can checkout the code [here](https://github.com/8erg/WinBypassIAT)
👉 You can checkout the blog and walkthrough [here](https://8erg.github.io/posts/2025-07-16-iat-hiding-injection/)

I adapt it for the `DLL` and i still hardcode the process ID

>[!warning]
>Disable precompiled headers if you remove `pch.h`, if you deleted it

### Dropping our malicious DLL

- Download the malicious `dll` and renamed it `secur32.dll`
- Copying and renaming valid `secur32.dll` to `updater.dll`
- Dropping both of these inside the folder containing `OneDriveUpdaterService.exe`
- Watch the magic unfold

>[!info]
>You will probably ask me why i dropped here, simple because it's more crowded, since there's a lot of `dll` our own get some camouflage

{{< image src="/images/Pasted image 20250725223707.png" alt="20250725223707" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

{{< image src="/images/Pasted image 20250725224028.png" alt="20250725224028" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

Even after running the execution flow, the malicious is not detected by windows defender  compared to before where it would get detected after executing the payload and running Microsoft defender scan

[UPDATE]

>[!Success] Yeah!
I recently learned that i could just directly reference the `secur32.dll` from the system directories, so no need the renamed the original `dll` move it there and...Anyway no need to do all that anymore🤗

*So now we go from this*

{{< image src="/images/Pasted image 20250725223800.png" alt="20250725223800" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

*To this*

{{< image src="/images/Pasted image 20250727200154.png" alt="20250727200154" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

*And I still get my beacon connection back to my `C2`*

{{< image src="/images/Pasted image 20250727200115.png" alt="20250727200115" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

But analyzing the `DLL` in `pestudio` I see that there is an indication the `GetUserNameEx` is forwarded, which could potentially indicates a `DLL` proxying

{{< image src="/images/Pasted image 20250727200641.png" alt="20250727200641" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

*But you know what, I'll ignore this for now and savor my WIN for today😂*


### 4. Conclusion
---

I think this one might be one of my go to, I just need to find a clever way to drop it on a target. This technique is so customizable, you can legit try it with any legitimate executable and mix with different types of techniques. I think I'm finally ready to try to whisper so these EDRs don't hear me (bars...😂)