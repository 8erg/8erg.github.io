+++
title = 'WannaCry - Analyzing Initial Stage'
date = 2024-07-16
draft = false
+++


### Preface
---

<div style="margin-top: 20px; margin-bottom: 40px;text-align: justify;">

In the early summer of 2017, WannaCry was unleashed on the world. Widely considered to be one of the most devastating malware infections to date, WannaCry left a trail of destruction in its wake. 
WannaCry is a classic ransomware sample; more specifically, it is a ransomware crypto worm, which means that it can encrypt individual hosts and had the capability to propagate through a network on its own.
Here’s my own analysis of this particular specimen.

In this blog post, I will analyze the initial stage of WannaCry ransomware sample. Please note that I'm new at this and I will try to provide a detail technical analysis of the initial stage to the best of my abilities. 
Hope you will enjoy this!!

</div>

### Basic Static Analysis
---

<div style="margin-top: 20px;"></div>

{{< image src="/MyBlog2/images/Pasted image 20240709173746.png" alt="20240709173746" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

>  ℹ️ | For this part I used a command line tool named floss to get the strings from the ransomware executable.

- By looking as some of the strings, we can see some interesting library related to encryptions such as:
	- `CryptAcquireContextA`
	- `CryptGenRandom`
	- `CryptServiceA`
	- `CryptReleaseContext`
	- `CryptGenKey`
	- `CryptDecrypt`
	- `CryptEncrypt`
	- `CryptDestroyKey`
	- `CryptImportKey`
- We can also see that the malware will probably modify or add some registry based on the library imported
	
- In the strings, we can also find words such as :
	- `mssecsvc.exe` (already flagged for been used for ransomware infection)
	- `tasksche.exe`
- We also see alot of base64 encoded strings
- We can see some randomly named folder that will dynamically named on runtime
	- `C:\%s\%s`
	- `C:\%s\qeriuwjhrf`
- We can also some interesting urls, such as :
	- `http://www.iuqerfsodp9ifjaposdfjhgosurijfaewrwergwea.com`
- An interesting string that let's us know for sure that we're dealing with the WannaCry malware : 
	- `c.wnry`
	- `t.wnry`
	- `msg/m_croatian.wnry`
	- `msg/m_dutch.wnry9`
	- `msg/m_english.wnryF`
	- `msg/m_korean.wnry`
	- `msg/m_latvian.wnry`
- Interesting strings:
	- `115p7UMMngoj1pMvkpHijcRdfJNXj6LrLn`
	- `12t9YDPgwueZ9NyMgw519p7AA8isjr6SMw` 
	- `13AM4VW2dhxYgXeQepoHkHSQuy6NgaEb94`
- Suspicious commands:
	- `icacls . /grant Everyone:F /T /C /Q` (Granting everyon acces to ACL)


<div style="margin-bottom: 60px;"></div>

### Basic Dynamic Analysis
---

<div style="margin-top: 20px;"></div>

- When triggering the malware with internet nothing happens, as far as encrypting my files, but I see that some calls are made to a malicious url, probably to drop something else on my file system
- When triggering the malware without internet connection, i can see that my file system get automatically encrypted
- There's seems to be a lot of call to different DNS

*Triggering the malware with fake internet simulation*

{{< image src="/images/Pasted image 20240709182444.png" alt="20240709182444" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

{{< image src="/images/Pasted image 20240709182542.png" alt="20240709182542" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

{{< image src="/images/Pasted image 20240709182931.png" alt="20240709182931" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

- It seems that the malware is making an http request to a malicious url

*Triggering the malware without internet simulation*

{{< image src="/images/Pasted image 20240709210004.png" alt="20240709210004" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

- A window pops up, that asks for payment and all my files are now encrypted
- There's new executable for decryption being created on my desktop
	- `@WanaDecryptor@.exe`

{{< image src="/images/Pasted image 20240712150050.png" alt="20240712150050" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

- a new executable is created named : `tasksche.exe`

{{< image src="/images/Pasted image 20240712150435.png" alt="20240712150435" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

- we can also see that a folder is created by this newly executed process

{{< image src="/images/Pasted image 20240714222810.png" alt="20240714222810" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

{{< image src="/images/Pasted image 20240714223416.png" alt="20240714223416" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

- We also see that 2 new services have been created
	- `mssecsvc2.0`
	- `dveqybpwqzws072`

<div style="margin-bottom: 40px;"></div>

### Advanced Static Analysis
---

<div style="margin-top: 20px;"></div>

> ℹ️ | In this section, I will be using cutter and ghidra for the advanced static analysis

{{< image src="/images/Pasted image 20240712212059.png" alt="20240712212059" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

- Here we can see that based on the response from the malicious domain, we will enter the real program
	- Option 1 : We received a response from the domain and the program execute normally
	- Option 2: We receive no response and the WannaCry program gets executed

{{< image src="/images/Pasted image 20240712212327.png" alt="20240712212327" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

- based on the numbers arguments received we will enter in this function


{{< image src="/images/Pasted image 20240712213151.png" alt="20240712213151" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

- Here there's two function available, we will enter in the first one

{{< image src="/images/Pasted image 20240709220438.png" alt="20240709220438" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

- Here we can see that the malware is creating and starting a new service called : `mssecsvc2.0`

> ⚠️ | The second function will be analyzed via ghidra, since it was difficult to debug it in cutter

{{< image src="/images/Pasted image 20240714204704.png" alt="20240714204704" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

- Here we can see that 4 function are loaded from `kernek32.dll`, will probably be used to create a malicious process that will run on the host machine

{{< image src="/images/Pasted image 20240714204902.png" alt="20240714204902" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

- Here we can see that a resource is loaded, let's wait and see what it will be used for

{{< image src="/images/Pasted image 20240714205402.png" alt="20240714205402" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

- In the first 2 lines containing the `sprintf` statement we're formatting some strings
- The line containing the `MoveFileExA`, to rename a file

{{< image src="/images/Pasted image 20240714170027.png" alt="20240714170027" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

- By comparing the same lines in cutter, we can clearly see that the malware is renaming the temporary file `qeriuwjhrf` to `tasksche.exe`
- Then we're creating the file named `tasksche.exe`
- Finally we can see that the program is writing the resource that was loaded earlier in the newly create file named `tasksche.exe`


<div style="margin-bottom: 40px;"></div>

### Advanced Dynamic Analysis
---

<div style="margin-top: 20px;"></div>

> ℹ️ | Since I covered most of the initial program of the WannaCry program, in this section I just wanted to see how the newly created executable would be run. I used the famous debugger x32dbg

{{< image src="/images/Pasted image 20240714215122.png" alt="20240714215122" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

- Here we can see that the file `tasksche.exe` will be executed with the `/i`. This is the program that starts the encryption process.

<div style="margin-bottom: 40px;"></div>

### Rules & Signatures
---

<div style="margin: 20px;">
</div>
	
```
rule WannaCry_Detection {
    
    meta: 
        last_updated = "2024-07-16"
        author = "8erg"
        description = "Yare detection rule for WannaCry Ransom"

    strings:
        $string1 = "mssecsvc.exe"
        $string2 = "tasksche.exe"
        $PE_magic_byte = "MZ"
        $malicious_url = "http://www.iuqerfsodp9ifjaposdfjhgosurijfaewrwergwea.com"

    condition:
        $PE_magic_byte at 0 and
        ($string1 and $string2) or
        $malicious_url
}
```

{{< image src="/images/Pasted image 20240716122337.png" alt="20240716122337" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

<div style="margin-bottom: 40px;"></div>

### Conclusion
---

<div style="margin-top: 20px;"></div>

To  conclude, this analysis was done following a certification that i went through recently and it is called Practical Malware Analysis & Triage. Even though, this is an old ransomware and many research has been peroformed onit. I still think that it was a great learning curve for me and I really enjoyed walking through the steps of the malware author and seeing how this ransomware worked.
