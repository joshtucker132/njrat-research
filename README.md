# njrat-research

## Important Links

[Executable] https://github.com/brian8544/njRAT

[Source Code] https://github.com/AliBawazeEer/RAT-NjRat-0.7d-modded-source-code

[Cynet Report] (https://www.cynet.com/attack-techniques-hands-on/njrat-report-bladabindi/)

[njRAT Injector Code Analysis] (https://medium.com/@shimasakisan/njrat-injector-code-analysis-d005b6808e96)

[VMWare Threat Analysis] (https://www.carbonblack.com/blog/threat-analysis-unit-tau-threat-intelligence-notification-njrat/)

[MITRE ATT&CK Report] (https://attack.mitre.org/software/S0385/)
                         

## Research and Analysis

  As of 2019, the Advanced Persistant Threat (APT) group 33, often linked to an Iranian state actor, has been reported to make extensive use of Remote Administration Tools (RATs). These tools give malicious actors complete remote access to a victim's device, with the ability to view webcam feeds, add or modify files, record keystrokes, etc. APT33 has shown a particular preference for one of these tools known as njRAT.
  
### Setup

  First appearing in 2013, njRAT can be broken down into two main components: the actual malware that runs on the attacker's device and the executable generated by the malware that is then run on the victim's device (often referred to as the injector). In order to perform this analysis, I downloaded 'njRAT v0.7d.exe' as well as the necessary helper files. I then tried to run the malware on both Windows 10 and 7 virtual machines(VMs). On both VMs, the user interface opened and gave the option to build a new injector. 
  
![building njrat injector](https://github.com/joshtucker132/njrat-research/blob/master/screenshots/building-injector.png?raw=true)

  The 'Host' field requires the name of the domain that will host the Command and Control (C&C) server. In order to avoid detection, njRAT uses DynamicDNS to change the IP address of the C&C server without changing the hostname. For this analysis, I made an account with NoIP, a DynamicDNS manager, and registered a domain name as 'njrattest.zapto.org'. I then downloaded the Domain Update Client to the machine running the malware, which switches the server to a new IP address every five minutes and updates the DNS servers.
  
![domain update client](https://github.com/joshtucker132/njrat-research/blob/master/screenshots/duc.png?raw=true)

  The remaining fields in the injector builder are optional to change. I choose to use the defaults, setting the injector executable name to 'server.exe' and the directory the injector is copied into on the victim machine to '%TEMP%'. I also left the options to copy the injector into the startup processes selected. When I clicked 'Build', the 'server.exe' injector was created.
  
![injector created](https://github.com/joshtucker132/njrat-research/blob/master/screenshots/injector-created.png?raw=true)

### Testing

Unfortunately when I tried to run the injector on both Windows 10 and 7 VMs, the injector was unable to establish a C&C channel with the njRAT server. Therefore, I could not test out any of the RAT functionality or observe the C&C channel in real-time. I suspect that this was due to a port forwarding issue. I tried forwarding the port I was using for njRAT through VirtualBox, but that did not resolve the issue. Despite this block, I was still able to see the persistence features of the injector code. If I looked into the '%temp%' directory of my machine, it was clear that 'server.exe' had successfully copied itself over. (I determined that 'njrat.bmp' was not actually related to the malware and only reflecting my username.)

![copying to temp](https://github.com/joshtucker132/njrat-research/blob/master/screenshots/temp-copy.png?raw=true)

I also examined the list of processes designated to run at startup and saw that the injector had created a hidden executable named after a Mutex. This Mutex was initiated immediately after running the injector to ensure a previous one is not already running.

![copying to startup](https://github.com/joshtucker132/njrat-research/blob/master/screenshots/startup-process.png?raw=true)

As stated previously, I was unable to successfully establish a C&C channel between the injector and njRAT server. However, by reviewing outside analysis of this malware, I was able to learn that the injector will initiate a TCP connection with the server and immediately launch a keylogger. At that point, the attacker can begin sending commands that will be executed on the victim's machine.

### Assessment

While njRAT is sufficient as a remote administration tool, it is not particularly sophisticated or innovative. njRAT's strongest attribute is arguably its persitence capabilites. For example, the injector makes entries in the victim's registry every second, so even if one is deleted it is immediately replaced. However, the injector runs as a process on the victim's machine, which is easy to detect. It also makes frequent DNS queries to maintain the C&C channel, which means that blocking DNS calls on the victim's machine would prevent any communication with the attacker. 

If the use of njRAT is a reflection on APT33, it suggests that the group also lacks sophistication. They choose to use generic malware that is several years old and easily detected and blocked, rather than writing their own exploits for specific targets that could avoid detection and inflict significant damage. Despite claims made in recent years that APT is ramping up their operations and becoming a serious cyber threat, it is clear they still have significant advancements to make before they reach that level.
