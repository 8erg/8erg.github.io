+++
title = 'IcedID - Analyzing Second Stage Loader'
date = 2024-07-22
draft = false
+++


### Preface
---

<div style="margin-top: 20px; margin-bottom: 40px;text-align: justify;">
	

In the ever-evolving landscape of cyber threats, the IcedID loader stands out as a formidable banking Trojan with sophisticated capabilities. Originally known for its role as a banking malware, IcedID has grown to serve as a versatile loader for deploying additional malicious payloads. This analysis delves into the intricate mechanisms of IcedID, from its initial delivery through phishing or exploit kits, to its operational intricacies and data exfiltration methods.

Key areas of focus include the loader’s evasion techniques, persistence strategies, and communication with command-and-control servers. By dissecting IcedID's functionality and evolution, this analysis aims to shed light on its impact on targeted systems and provide insights into effective detection and mitigation strategies.

</div>

### Basic Static Analysis
---

<div style="margin-top: 20px;"></div>

In this section I used a simple tool named `pe studio` to get a general understanding of the loader. After unpacking I also used a command line tool named `floss`

{{< image src="/images/Pasted image 20240716223035.png" alt="20240716223035" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}


- In this screenshot we can clearly see that `pe studio` is not able to properly counts the imports and the libraries which indicates to me that this sample might be packed
- also we can note that there is a high entropy. I will have to unpack it to be able to proceed my analysis

{{< image src="/images/Pasted image 20240716223435.png" alt="20240716223435" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

- We can also see that the raw size is lower than the virtual size which might be another indicator that this sample is packed

> ⚠️ | Before proceeding further in our basic static analysis we will perform the unpacking steps with x32bdg, since we are focusing on the analysis, I will not put the process in this blog

- Interesting strings, Before unpacking
{{< image src="/images/Pasted image 20240721140409.png" alt="20240721140409" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}
- After unpacking, I can see some interesting parts of some URLs
	- `/photo.png?id=%0.2X%0.8X%0.8X%s`
	- `boldidiotruss.xyz`
	- `nizaoplov.xyz`
	- `153ishak.best`
	- `ilu21plane.xyz`
- I can also see some interesting libraries being imported which indicates to me that something is being downloaded
	- `WinHttpQueryDataAvailable`
	- `WinHttpConnect`
	- `WinHttpSendRequest`
	- `WinHttpCloseHandle`
	- `WinHttpSetOption`
	- `WinHttpOpenRequest`
	- `WinHttpReadData`
	- `WinHttpQueryHeaders`
	- `WinHttpOpen`
	- `WinHttpReceiveResponse`
   
<div style="margin-bottom: 40px;"></div>

### Basic Dynamic Analysis
---

<div style="margin-top: 20px;"></div>

{{< image src="/images/Pasted image 20240717061554.png" alt="20240717061554" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

- By detonating the malware we notice that it tried to create a file named `photo.png`

{{< image src="/images/Pasted image 20240717062937.png" alt="20240717062937" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

- We can also see that a `TLSv1.2` connection is made to `boldidiotruss.xyz`

<div style="margin-bottom: 40px;"></div>


### Advanced Static Analysis
---

<div style="margin-top: 20px;"></div>

{{< image src="/images/Pasted image 20240717081349.png" alt="20240717081349" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

- Here we can see that loader is calling the function `SHGetFolderPathA` and by looking at the documentation I can see that the second argument indicates the folder of interest in this case it is aiming for `CSIDL_COMMON_DESKTOPDIRECTORY`, which represents a common folder that appear on the desktop for all users and the typical path is `C:\Users\Public\Desktop`

- We can also see that based upon the result the content of `pcVar5` will be different and at the end it will be concatenate to the folder path

{{< image src="/images/Pasted image 20240717081349.png" alt="20240717081349" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

- After that, we see that the `GetUsernameA` is used to fetch the current user, we also see that there is an attempt to create the directory
- At the end, we can see that the string `\\photo.png` is appended to the current `folder_path` string

{{< image src="/images/Pasted image 20240717085042.png" alt="20240717085042" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

- We can see the file path that was constructed earlier is used to create a file
  
{{< image src="/images/Pasted image 20240717085910.png" alt="20240717085910" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

- There's some VM detection being performed by checking information about the CPU running the sample


> ⚠️ | Note that some functions have been renamed by me to facilitate the clarity of my analysis

{{< image src="/images/Pasted image 20240717090929.png" alt="20240717090929" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

- After the VM detection, there's string formatting operation that is performed to construct the URL path to the `.png` file and also to the URL hostname
- we can see that the result of the concatenation of the full URL is being sent to a function that will handle the communication with the possible the command & control center

{{< image src="/images/Pasted image 20240717092802.png" alt="20240717092802" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

{{< image src="/images/Pasted image 20240717092848.png" alt="20240717092848" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}
	
- In these screenshots, we can clearly see the code the of the communication  with the C2 
- Based on my analysis I deduct that a call his being made to verify that the command and control is active and running correctly, because of the parameters of the functions I can see that no data is actually being sent since it's only performing a get request

{{< image src="/images/Pasted image 20240721122821.png" alt="20240721122821" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

- Now, I thought to myself why would they be communicating via an URL that looks like you're fetching an image? Because, in reality you're really just communicating, sending and receiving information stealthily. Think about it, if you look in Wireshark and see some request made to this URL you would only think that a picture his being fetched, but in reality, you're data is probably being exfiltrated. I might be wrong in my analysis but that is what I get from it

{{< image src="/images/Pasted image 20240721123829.png" alt="20240721123829" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

- By looking at everything from a higher perspective, here's what can I make of the general process of this Loader
	1. We create the path where we will store the malicious file
	2. we decrypt probably the shellcode that will be stored in that file temporary
	3. we create the file picture
	4. we decrypt something
	5. we check if the command & control center is active
	6. we write in the data that decrypted in that file
	7. we setting up the shellcode in memory and then we execute it


> ℹ️ In my basic dynamic analysis we don't see the call made to the url in question, that may be because the malware is able to detect that I'm using a vm, so it does not pursue the communication, we will confirm it later when we will be doing the Advanced Dynamic Analysis with our favorite debugger x32dbg.

<div style="margin-bottom: 40px;"></div>

### Advanced Dynamic Analysis
---

<div style="margin-top: 20px;"></div>

{{< image src="/images/Pasted image 20240721131650.png" alt="20240721131650" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

- Here we can see the file path of the `photo.png`, by the way don't mind the username, I got this box from the `PMAT` certification

{{< image src="/images/Pasted image 20240721131953.png" alt="20240721131953" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

- We can also see the URL being formed, you can see that there is type of string that is appended to it. It's probably some data about the host being sent to the `Command & Control` Server , You can see some more details about the the construction of this string in the picture below


{{< image src="/images/Pasted image 20240723093759.png" alt="20240723093759" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

{{< image src="/images/Pasted image 20240723093829.png" alt="20240723093829" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

{{< image src="/images/Pasted image 20240722105526.png" alt="20240722105526" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}


- Here we can see the value 8 and 248 making a total of 256, so we can assume that this is the encryption function where the 8 represent  `key` and the 248 the `data`

{{< image src="/images/Pasted image 20240722110943.png" alt="20240722110943" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

- This screenshot show us clearly that a `GET` request is being made

{{< image src="/images/Pasted image 20240722111231.png" alt="20240722111231" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

<div style="margin-bottom: 40px;"></div>

### Rules & Signatures
---

<div style="margin-top: 20px;"></div>

```
rule IcedId_Ldr_Detection {
    
    meta: 
        last_updated = "2024-07-21"
        author = "8erg"
        description = "Yara detection rule for IcedId Loader"

    strings:
        $magical_path = "c:\\Sizeanger\\CreatePick\\mixpractice\\Sciencescience\\KeyContain\\farterm\\Tiresubtract\\CenterSkinMass.pdb"

    condition:
        $magical_path
}

```

<div style="margin-bottom: 40px;"></div>

### Conclusion
---

<div style="margin-top: 20px; margin-bottom: 40px;text-align: justify;">
	
Well, this is it for this analysis. I really enjoyed this one! I'm really starting to get the hang of it. 
It's interesting to see how some people develop their malware, you can see that there's a lot of thought that are put into it. 
I'm really learning a lot. Well, I'll stop babbling now. See you on the next one!

</div>



