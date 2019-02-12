# CVE-2017-5638

## TLDR; The vunerability that caused Equifax to loose 145.5 million records of Personal Identifiable Information of the general public.

## Who did it affect?
- Equifax customers
- potentially anyone with Apache struts unpatched.
## Impact to stakeholders.

- Credit card data for 209k of US consumers.
- Credit data for UK BT customers potentially affected.
- Stock prices dropping for Equifax.

## What is it?
The vulnerability [CVE-2017-5638](https://nvd.nist.gov/vuln/detail/CVE-2017-5638#vulnCurrentDescriptionTitle) is a remote code execution bug that affects the Jakarta Multipart parser in Apache Struts.

> [Apache Struts](https://struts.apache.org/) is an MVC framework for web applications in Java.

*"The Jakarta Multipart parser in Apache Struts 2 2.3.x before 2.3.32 and 2.5.x before 2.5.10.1 has incorrect exception handling and error-message generation during file-upload attempts, which **allows remote attackers to execute arbitrary commands** via a crafted **Content-Type, Content-Disposition, or Content-Length HTTP header**, as exploited in the wild in **March 2017** with a **Content-Type header containing a #cmd= string**."* [-MITRE](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-5638)


## What does it do, and how does it work?


**[MITRE](https://attack.mitre.org/matrices/enterprise/) Classification** - [Inproper input validation- CWE20](http://cwe.mitre.org/data/definitions/20.html)

What does that mean?

That the attacker can use it to remotely command and control the host by exploiting the mechanism in which the application spews out errors. The application doesn't verify what is inputed as correct or not. I.e. trying to put a cube in a triangle shaped hole and it working. This means the attacker can use this to inject commands.
>[Example of the vunerability being used to read passwords and escalate privilages to root.](https://securonix.com/web/wp-content/uploads/2017/11/equifax-1.png)


## When did it happen?
The Equifax hack happened in March 2017. What exactly happened?

Lets start from the begginning...

### Incident Timeline

[Timeline infographic](https://image.slidesharecdn.com/strutstimelineinfo-171012175148/95/equifax-apache-struts-vulnerability-cve20175638-1-1024.jpg?cb=1507831817)

## The git-blame game - who did it?
 Everyone is blaming each other. Exquifax, blames a SysAdmin and Apache. Everyone else is blaming Equifax.
## Version shaming

Apache Struts versions Struts 2.3.5 – Struts 2.3.31, Struts 2.5 – Struts 2.5.10 are reported to be affected. If you are using the Jakarta-based file upload Multipart parser, you should upgrade to Apache Struts version 2.3.32 or 2.5.10.1.

## Attack vectors
Network.

## Exploits
We're all thinking...
### So how do I use it? ;)
>Don't work harder, think smarter! Do it like a script kiddy! Have no shame. Remember that hack value is important. If the asset is low value, you may aswell not put in as much effort.
1) Scope your target - Recon.
>... comply with your country's laws on this. If in doubt, attack your own servers, since you own them, or don't do it. Always get it on paper first if you don't own it.
2) Use a Vunerability scanner that actively updates their CVE database.
> HINT: Nessus has one ;)
3) Pick your exploit

>The vunerability has been around for a while now. There are a few exploit kits and vunerability scanning tools that can identify a host with Apache struts installed and even do the tedious bit of finding a vunerable version, rather than you doing it manually. This can also be stopped from having a direct risk by applying a WAF as it will prevent any incomplete connections from penetrating (common characteristic of scans).

Lucky for you here's some that were cooked eariler...

- https://www.exploit-db.com/exploits/41570
- https://www.exploit-db.com/exploits/41614
- https://github.com/mazen160/struts-pwn
- https://github.com/rapid7/metasploit-framework/issues/8064


## Playing the detective - using Unique Identifiers and Indicators of Attack


### Hackers are lazy...
As above, creating exploits yourself is effort and time consuming, so you might have used one of the ones already on the interwebs.

>Think like the hackers do... If there is a known expoit, then you are are literally being handed the holy grail of detecting an attack. **The expoit code**.

This will help you understand the mechanism of the exploit and what is the entry point.

In this case the class `LocalizedTextUtil` in  `FileUploadInterceptor.java` was supposed to give an error message via HTTP for when an upload failed. This was then exploited to gain a foothold. From here an attacker can attempt to aquire high value assets or destroy them.

### Dependencies and libraries
Look for the dependencies required for the exploit to work, because again, hackers are lazy- if they want to code the exploit themselves, they probably will use similar libraries to publicly disclosed expoits. Github and other public code deposits are a good place to start.

### User agents

Most applications will have a user agent in a HTTP header when the application is communicating on a network.

>If you are using a vunerability scanner, that scanner will have its own User Agent. So you are able to use this to your advantage when sniffing network traffic e.g if you are using Nessus to scan for the vunerability, you may see the Nessus User Agent in the HTTP traffic. 

Without using a vunerability sanning tool, the client User Agent was used. This could be whatever browser the victim has installed.

### Hash list
Every file has its own file hash. This can be in various different formats.

MD5,SHA1, SHA256, SHA512, JA3.

**Network traffic:** When files are downloaded onto a host, the connection logs can contain the file hash. If the file hash isn't present, then the exploit has not been downloaded to that host directly. In this case there is no payload to be downloaded onto the host to gain a foothold, as the vunerablity is already part of infrastructure.

This does not mean in other attacks a payload would not be used before or after gaining foothold.

[Example hashes of one with a payload](https://otx.alienvault.com/indicator/file/445537787eac24bea8a4989d23031e49)


`[MD5: 445537787eac24bea8a4989d23031e49]`

`[SHA1: a60a2cabdad8620988ab4d9ab7d24a064d8b6295]`

`[SHA256: 2d414c4fdc809777a47a0764beea6008576125cdc81d17cdc8076d289b508d56]`

>Nessus scans for this vunerability are visible when banner grabbing or even just by looking at the URIs. Try searching for `%nessus%` in your  HTTP logs.

> Lateral movement from a different host can still occur. This is a bit harder to detect as they will probably already have root credentials..

**Forensics:** If the exploit has been used, because of what it does (Remote command execution), you might see commands in your history that you didn't do.

> [The attack seen in server logs.](https://www.securonix.com/web/wp-content/uploads/2017/11/xequifax-3.png.pagespeed.ic.FHV-p3NAGZ.webp)

### Domains

Hackers, depending on their goals might also attempt to exfiltrate data through a Command and Control channel which will eventually need to be collected or stored at a domain.

Sometimes these domains are recorded in Open Source Intelligence tools.

> [Current pulses that exploit the vunerability- AlienVault](https://otx.alienvault.com/indicator/cve/CVE-2017-5638)

### Rules

Automating scanning rules that we could potentially use.

- [Snort rule](https://gist.github.com/stamparm/a9cf56d40ac3ce5e48e36971946093f8) SIDs: 41818, 41819
- [YARA rules](https://github.com/Yara-Rules/rules/blob/master/CVE_Rules/CVE-2017-11882.yar)

## Learning from Equifax's mistakes
What could have prevented the Equifax hack... 
### Automatic updates
Not all of Equifax's SysAdmin team were aware of a critical patch needing to be applied to hosts. So not all hosts were patched. Automated patch pipes would have solved this problem.

### Mature your Security Software Development Life Cycle
 Inplement continuous deployment so software developers can have an in pipe dependency scanner to make sure dependencies are up to date.
> Here's where Synk come in.
## What can Apache do?

Start fuzzing their applications or using a mutation testing platform to introduce incorrect inputs during development, so exception error bugs are caught earlier.

## Appendix

- [Apatche Advisory](https://cwiki.apache.org/confluence/display/WW/S2-045)
- [Apatche patch guide](https://cwiki.apache.org/confluence/display/WW/S2-046)
- [Trend Micro's exploit analysis -Reverse engineered](https://blog.trendmicro.com/trendlabs-security-intelligence/cve-2017-5638-apache-struts-vulnerability-remote-code-execution/)
- [POC - not in English](https://github.com/tengzhangchao/Struts2_045-Poc)
- [Threat intelligence- Current pulses that exploit the vunerability- AlienVault](https://otx.alienvault.com/indicator/cve/CVE-2017-5638)
