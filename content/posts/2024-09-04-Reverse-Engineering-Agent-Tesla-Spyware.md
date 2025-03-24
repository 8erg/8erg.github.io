

## Table of Contents
--- 


| **Section**                         | **Description**                                                                                                                                 |
| ----------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| **Preface**                         | An introduction to the context and purpose of the analysis, including tools and objectives.                                                     |
| **Basic Static & Dynamic Analysis** | Initial inspection of the binary, such as examining file metadata, strings, and imports to gather a high-level understanding.                   |
| **Advanced Static Analysis**        | In-depth dissection of the binary, including reverse engineering with disassemblers, examining code flow, and detecting obfuscation techniques. |
| **Conclusion**                      | A summary of findings, including identified indicators of compromise (IOCs), behaviors, and next steps for further analysis.                    |


### Preface
---

Agent Tesla a Spyware acting as a game and stealing credentials. This sample is packed and has to be unpacked first in order to see the content. Uts's malware developped in C#



### Analysis
---

In this section I used a simple tool named `pe studio` to get a general understanding of the loader.

{{< image src="/images/Pasted image 20241215001442.png" alt="20240813093757" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

{{< image src="/images/Pasted image 20241215002133.png" alt="20240813093757" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

- We can see that that this sample has probably been written in `C#` and we can see that it's trying to make us believe that it's a game by the description `Diablo 3`, I don't play video games but i heard about that particular video game
- There's high entropy being detected, so it's probably packed


*To Unpack it I used `x32dbg` and I put a breakpoint on `WriteProcessMemory`*

- So `agent tesla` will decrypt the encrypted payload into the memory then it will create the agent process and then dump the unpacked data in the child process

{{< image src="/images/Pasted image 20241215230050.png" alt="20241215230050" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

{{< image src="/images/Pasted image 20241215230930.png" alt="20241215230930" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

  > [!info] When a malware create a child process, go to `WriteProcessMemory` to quickly get the unpacked executable
  

- As you can see in the debugger, by hitting the breakpoint on `WriteProcessMemory` we can find the unpacked executable by following the third argument address in the dump. Some of you will probably ask me why, go to the third argument, well simply because on the documentation we can see that third argument of `WriteProcessMemory` contains the `lpBuffer` which is a pointer to the buffer, which in our case represent the unpacked executable that is written in the sub-process that has been created by `agent tesla`.

{{< image src="/images/Pasted image 20241215232905.png" alt="20241215232905" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

- Based on these strings we can definitely conclude that we're dealing with an Info Stealer

{{< image src="/images/Pasted image 20241215234858.png" alt="20241215234858" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}

- I've put the unpacked executable inside the tool `Simple Assembly Explorer` in order to cleaned up most of the obfuscated strings, to ease up my analysis of the program code

{{< image src="/images/Pasted image 20241216001630.png" alt="20241216001630" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}
{{< image src="/images/Pasted image 20241216001831.png" alt="20241216001831" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}
- By Monitoring the `RegSvcs.exe` process we can see that there's a lot of operations being performed on my google chrome credentials
- We can see it goes on Edge too

{{< image src="/images/Pasted image 20241216213452.png" alt="20241216213452" position="center" style="border-radius: 6px; margin-top: 20px; margin-bottom: 20px; margin-left: auto; margin-right: auto" >}}


### Rules & Signatures
---




### Conclusion
--- 

