+++
title = 'IAT Hiding Injection'
date = 2025-07-16
draft = false
+++


## 1. PREFACE
---

So last week I tried to bypass windows defender and injecting my obfuscated stager to drop my `Sliver C2` beacon but failed miserably. My executable was undetectable, but the process where I was injecting my shellcode kept crashing so I had to take an other path. 

So, I decided to combine a couple of techniques (I got it from `maldev academy` and also some malware that I've reversed engineers) such as :
- IAT Hiding and Obfuscation
- API hashing
- using `Nt* API` calls 

And i got to say getting that notification of my beacon while having my executable undetected on the target machine was a good feeling.

{{< image src="/images/Pasted image 20250716022846.png" alt="20250716022846" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}


## 2. SETUP
---

For the setup of `Sliver C2` you can check out my earlier blog post [here](https://8erg.github.io/posts/2025-07-10-injecting-shellcode-via-rwx-protected-memory-regions/)


## 3. Walkthrough
---

So first, I tried implementing my custom function of `GetProcAddress` and `GetModuleHandle`, there's some resource on how to do this in `maldev academy`.

Then i tried to identified which API calls I wanted to call `NT*`, since it's undocumented I've used this website to generate my own signatures of it : https://ntdoc.m417z.com/

+ `NtOpenProcess`
+ `NtAllocateVirtualMemory`
+ `NtWriteVirtualMemory`
+ `NtCreateThreadEx`

But then, I said to myself even if I've implemented, if i just paste the string of the functions i want to use, it will easily gets detected, so i decided to implement some API hashing, which I've seen being used in one of the malware I've tried (the key word, here, is tried😂) to reverse engineered `Qakbot web injector`. 

So by combining these different approach into one, I was able to get my executable undetected by windows

You can checkout the source code on my [Github](https://github.com/8erg/WinBypassIAT)

{{< image src="/images/Pasted image 20250716021847.png" alt="20250716021847" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

{{< image src="/images/Pasted image 20250716020604.png" alt="20250716020604" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

`I will mention it later in this post, but having outbound connection coming from explorer.exe is a major red flag`


### Why it's not detected

{{< image src="/images/Pasted image 20250715032601.png" alt="20250715032601" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

+ The entropy isn't that high and a lot of it is do to my encrypted stager and the hashing the API calls

{{< image src="/images/Pasted image 20250715032649.png" alt="20250715032649" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

{{< image src="/images/Pasted image 20250715033243.png" alt="20250715033243" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

+ None of my API calls are reflected in the import address table, it would be also a plus to add dummy function implementation to camouflage my behavior a little bit more


### Possible IOCs

I want to mention that even though my executable gets undetected by windows, there's other possible indicators of compromise (IOCs)

+ I've injected my payload in `explorer.exe`, as soon as someone sees that is connecting to an outside connection, it will raise some eyebrows
+ since I've set a connection back to my `C2` from beacon every 5 seconds, this could potentially raise an alarm
+ Also, i didn't even talk about `ETW`, might try to patch it next, i've watched a [video](https://www.youtube.com/watch?v=U5dhuyPm6n8&t=1321s) from `Mr.Un1k0d3r` about it 
+ Obviously I've left some debugging outputs, which can rat me out

{{< image src="/images/Pasted image 20250715032748.png" alt="20250715032748" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

{{< image src="/images/Pasted image 20250715035835.png" alt="20250715035835" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

{{< image src="/images/Pasted image 20250716020625.png" alt="20250716020625" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

+ You might be wondering why do we see 2 connetion one from port `8443` and one from port `8888`, it's because the first is the connection from the stager and then the last one is the connection that will be used by the beacon, which checks in every 5 seconds


### Being a dummy

`Of course i can't leave this section out`

+ so i wanted to get additional proof, so i ran windows defender check while having my payload running and it immediately identified the `meterpreter` signature in notepad memory and i successfully burned my malware as it not able to perform that injection

+ but i was able to remediate this problem by slightly modifying the execution workflow and injecting into a different process


### 4. Conclusion
---

Honestly, i've had a lot of fun with this one and I was able to also better understand a malware that I was reverse engineered a couple months ago, which i've yet to release a post about it, they were using some API hashing to evade detection. Guys, that's why I say if you want to become a better offensive operator you need to learn `REVERSE ENGINEERING` i think it's mendatory and it think it comes many different forms, but that's just my personal opinion, I still suck btw (just putting it out there😂). My next steps would probably be to try and remove as much iocs as i can, by patching `ETW` and trying the `SysWhisperer` to make syscall directly. But first, I want to try the `armory`...