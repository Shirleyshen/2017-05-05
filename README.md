Root Causes for Slow Boots and Slow Logons (SBSL)
This article describes Microsoft Support experiences in troubleshooting boot and logon delays, specifically the root causes. Other related topics include:
Tools for Troubleshooting Slow Boots and Slow Logons (sbsl) 
Troubleshooting Slow Operating System Boot Times and Slow User Logons (sbsl) 
The Windows 7 Boot Process (sbsl) 
The goal of this content is to create awareness among IT administrators, support professionals, and consultants, about the tools, causes, and resolutions for boot and user logon delays. The primary focus is on domain-joined clients and servers. The article does not pertain to slow boots and logons on consumer desktops in a workgroup, but some of the tools and methods would still apply, such as enabling verbose logging and noting the message and duration where the boot or logon hung.
 
Credits
Written by: A. Conner, David Everett, and Joey Seifert 
Edited by: Justin Hall
 
Introduction
There is no shortage of root causes for boot and logon delays. Some delays are caused by code-defects in the OS or applications on the computer experiencing slow boot or logon. Other root causes lie with the underlying network, remote servers, or Administrator misconfiguration. 
Don’t worry about tracking the QFEs for Windows. We’ll be publishing a list of recommended fixes in the near future. 
The following sections explain some of the problems that have been seen to date, and how to resolve them.
Table of Contents

Credits
Introduction
Applications
Hardware
Locator
Network
Operating System
Policy including Group Policy Preferences
Profiles
Readyboot
Remote Desktop
WMI
Summary
Related links
Appendix
Glossary

 

Applications
Boot and logon delays can be caused by applications as innocuous as utilities and applets to mission critical applications. Examples of utilities and applets that can cause boot and logon delays include software that monitors ambient light sensors to dynamically adjust monitor brightness or applications that mute built-in microphones on laptop computers. Hardware developers are generally diligent about finding such delays and resolving them in later software versions. Hardware developers also identify and report delays in the core operating system too. 
Antivirus and security software performance is essential to enjoying fast boot and logon times. Security software developers are constantly improving application performance in incremental software updates. Performance improvements that rely on major architectural changes tend to come in major version updates. Finally, application developers and administrators get to choose between slow and safe versus fast and less secure settings in security applications, which can also have an impact on system performance. The following TABLE width="99%" lists examples of performance improvements for security software. 
  
Vendor 
Product
Summary
Avecto 
Privilege Guard
Version 2.8.300 resolves known delays during user logon. 
McAfee 
VirusScan
Performance improvements in VirusScan 8.8 over 8.7. Link  
McAfee 
VirusScan
Disabling “process on enable” scans may improve boot and logon performance. The setting is enabled on 8.7. The setting is disabled in version 8.8 but a setting may be persisted by policy. Retry boot and logon with setting disabled if a boot trace shows McShield consuming sufficient disk or CPU resources or has long execution times in the auto-start base of a boot trace.
Microsoft
Microsoft Application Virtualization (App-V) 4.6
App-V prior to version 4.6 SP1 should have Hotfix 3 (KB 2571168  ) or later installed. Delay identified by long execution time for SFTLST.EXE in autostart pane of XBOOTMGR trace. Due to WMI dependency, also install KB 2617858  .
Microsoft
Data Protection Manager (DPM) 2010
Scheduled tasks can delay OS startup. Install KB 2615782  . 
Microsoft 
Forefront Endpoint Protection (FEP) 2010
FEP 2010 with pre-6903 engines may cause boot and logon delays. To check engine version, click Help, and then click About in Forefront Endpoint Protection. Ensure that FEP and other Microsoft security software is configured to receive monthly updates. 
Microsoft 
Lync 2010
Verbose logging enabled by "Turn on logging in Lync" can degrade system performance, especially if beta versions of Lync were installed. 
Symantec
Endpoint Protection
Performance improvements in SEP 12.1
Link  . 
 

Hardware
Laptop computers can have an “acoustic mode” BIOS setting, which reduces drive noise by slowing the rotational speed of platters on the drive. Battery life improves but disk throughput is decreased during the i/o intensive boot and logon phases. The point is to be aware of such settings and configure the laptop depending on whether you need a quieter work environment with longer battery life or faster boot, logon and post logon disk throughput. 
Just as software vendors continually improve performance through incremental and milestone updates, hardware vendors improve performance in Bios and Firmware updates, so it’s a good idea to check with your hardware support sites for potential performance updates then do some testing with XBOOTMGR.
Also, it’s a good idea to run a query on your make and model computer in your favorite search engine  to see if the community has latched onto any make and model specific defects. 
Boot and logon performance depends heavily on disk drive speed so you can see performance gains as you transition from 5400 to 7200-rpm drives or to solid state drives (SSD). 
 

Locator
Locator, or DCLocator, is the component used by a domain-joined computer to locate domain controllers for computer authentication, user authentication, ldap queries and more. Due to the iterative network I/O generated, operating system boots and user logons on domain-joined computers enjoy the fastest performance when serviced by a domain controller connected by a fast LAN. Conversely, performance is degraded when boots and logons are serviced by DCs and file servers in in remote Active Directory sites and subnets. 
When clients do not use site-optimal domain controllers, profile servers, and application servers, the boot and logon process becomes vulnerable to the following challenges: 
Network latency introduced by the use of a remote-site DC can have a major impact on the iterative network I/O that takes place during OS boot and user logon, including but not limited to 
the loading of user profiles 
The execution of startup scripts and logon scripts 
The application of computer and user policies, especially Group Policy preference settings that target security groups or LDAP-filtered items, which can generate hundreds or thousands of LDAP queries or SID lookups per logon. 
Network I/O is increasingly subject to reduced packet sizes which reduce network throughput 
Network I/O across a WAN is increasingly subject to dropped packets, packet retransmits, or the blockage of network ports. 
Failure conditions can invoke timeout conditions in code which can cause extremely long delays for users, services and applications. 
A domain-joined client computer may discover a DC in a remote site rather than its own site for the following reasons:
The in-site DC is offline or non-operational 
The in-site DC has not registered its site-specific DNS records, either because the in-site DC points to an invalid DNS server or replication issues are preventing clients from resolving site-specific SRV records. 
The in-site DC is not “advertising” as a DC, including a failure to share the NETLOGON and SYSVOL folders. 
No valid subnet, Active Directory site, and subnet-to-site mapping exists in Active Directory to map an optimal DC to the client computers subnet. 
Windows Server 2003 DCs auto-register site-specific SRV records for RODC-covered sites because the RODC compatibility pack (KB 944043) is not installed on Windows Server 2003 DCs. 
DCs in a remote-site have been configured by an administrator to register site-specific SRV records on behalf of the covered site. 
An OS or application component residing in an RODC-covered site is making a DSGETDC call with the DS_WRITABLE width="99%" _REQUIRED flag, which cannot be satisfied by the in-site RODC. 
An OS or application component is making a DSGETDC call with some locator flag that cannot be satisfied by an in-site DC. 
The client computer has cached a remote site DC while one of the above conditions existed temporarily. In earlier versions of Windows, especially unpatched versions of Windows XP, the use of a remote site DC can be persistent (technical term is “sticky”) until that DC becomes unresponsive or the client gets rebooted. This problem is resolved with KB 939252, which describes legacy locator caching logic and the enhancements that shipped first in Windows Vista. 
There are also root causes that can prevent clients from connecting to in-site DFS targets including the SYSVOL share on domain controllers. Clients accessing group policy, logon scripts, startup scripts, or paths referenced in executing scripts on remote DFS shares may result in boot and logon delays. 
Known issues
Vendor 
Product
Summary
Citrix
XenApp
User logons to W2K8 R2 DCs with XenApp residing RODC-covered sites are slow due to the credential provider making an ADSI call that makes an implicit DsGetDC call which cannot be satisfied by the RODC.
Workaround: ensure that XenApp servers reside in RWDC-covered sites. Citrix has a technical support article on this issue: http://support.citrix.com/article/CTX133873.  
Microsoft 
Windows Server 2003
Windows Server 2008
Windows Server 2008 R2 
Failure to define valid subnet, site and subnet-to-site mapping increases the probability that clients will use a remote-site DC. Check System event logs on DCs for NETLOGON event 5807: “During the past (x) hours there have been (y) connections to this Domain Controller from client machines whose IP addresses don't map to any of the existing sites in the enterprise.”
Review KB  306602  to determine whether branch-site DCs should continue to register siteless SRV records. 
Microsoft 
Windows Server 2003
Windows Server 2008
Windows Server 2008 R2 
A story: User logons from Windows 7 clients were taking 5 minutes. Enabling verbose status messages revealed that the Group Policy Local Users and Groups client-side extension was taking about 160 seconds to apply. GPRESULT /Z <filename.ext> confirmed that the client was applying Local Users and Groups Group Policy preferences. A review of the clients NETLOGON.LOG revealed that the client had initially discovered an in-site DC but subsequently issued a second DSGETDC call specifying the /TIMESERV flag. The client then discovered a remote-site DC advertising the time flag located half-way around the world to source the remainder of policy. Root cause: causes for the logon delay: 
The ntptype registry setting under hklm\..\w32time\timeproviders had been set to 0 on the in-site DC. 
The client was being asked to process about 200 OU-filtered Local Group Preference settings. Logon times increased dramatically when serviced by a remote-site DC connected by a highly latent network link.  
Microsoft
Windows Server 2003 
Windows Server 2003 DCs auto-register sitefull DNS SRV records for a site covered by an RODC unless the RODC compatibility pack is installed on the Windows Server 2003 DCs. See KB 944043  .
Microsoft
Windows Server 2008 R2
Reverse-name lookups in un-mapped sites are slower on Windows Server 2008 R2 DCs compared to Windows Server 2003 and Windows Server 2008. This problem is fixed in Windows 8 Server Beta. 
Workaround: ensure that valid subnet; site and subnet-to-site mappings are defined for authenticating computers.
In extreme cases, this condition can cause LDAP queries to run long, exhausting the worker thread pool on DCs
 

Network
The CTS SBSL team has experienced a number of network-related causes for slow boots and logons. From an operating system perspective, the best network throughput occurs when “up-level” clients (Windows Vista, Windows 7, Windows Server 2008, Windows Server 2008 R2) are communicating with “up-level” servers to take advantage of performance enhancements in the SMB 2.0 protocol. 
To complicate troubleshooting, XPERF does not always identify underlying network delays, so one of the XPERF skill sets you need to develop is determining when to put XPERF aside and get a network-trace or component specific trace that clearly exposes network delays. 
XPERF can help identify network delays when the stack trace in a CPU Scheduling -> Summary TABLE width="99%" with Ready Thread report shows a slow performing operation being placed onto the wire. Also, the TCP/IP Counts Graph summary TABLE width="99%" may also expose network-related error codes like “ 0xc0000043 “ which maps to “A file cannot be opened because the share access flags are incompatible.”. That’s your cue to get a network trace to look for: 
DNS name lookup failures 
A failure to connect to mapped network drives including home drive, profile paths and the like 
The use of remote site servers when suiTABLE width="99%" in-site servers and domain controllers do or should exist 
Network protocol, security protocol negotiation failures 
SMB dialect negotiation, session setup or tree connect failures 
Authentication failures or delays 
Excessive network I/O due to a code defect, misconfiguration or excessive workload 
Dropped outbound or inbound frames 
TCP resets (some of which are normal) 
2-byte reads 
Reduced network throughput due to excessively small MTU size 
The network client inefficiently iterating over a target directory 
Blocked ports 
Sharing violations or oplock failures 
In some cases it is sufficient to take a network trace that only captures traffic to and from the slow boot or logon computer. In cases where network packets are being dropped, it’s important to take concurrent traces from both the caller and the target computers, then reconcile whether packets issued by the caller actually made it to the target computer and vice versa. One suggestion when reviewing such traces is identify ports and IP addresses that the caller is using to reach the target computer, then create display filter that isolate conversations of interest in both traces.
When XPERF or network traces show a remote computer returning a network error code, start your investigation on the remote computer.
Where network errors occur on a [remote] computer that has TDI filter drivers installed, we recommend that you install the latest the latest TDX.SYS files on the target computer, set any registry keys listed in the corresponding KB, reboot, and then retry the failing operation. For Windows Server 2008 and Windows Vista, the current hotfix is KB 961775  . For Windows Server 2008 R2 and Windows 7, see KB 981344  . If still failing, retry the operation with TDI or filter drivers removed from the target computer, followed by a reboot. 
Finally, where a network trace shows a computer receiving frames of interest, investigate the firewall service state and the firewall rules in effect to ensure that network frames were actually passed “up” the stack.
Display filters in NETMON can help identify some network errors that cause boot and logon delays. Sample display filters that can be applied to Microsoft Network monitor traces include: 
For retransmits: Property.TCPRetransmit == 1 
For resets: resets: tcp.flags.reset==1 
For examining retransmits over the port used by the client to talk to the server or vice versa: tcp.port== 1234 and property.tcpretranmit==1 
A more comprehensive NETMON 3.4 display capture filter created by Joel Christenson of the Microsoft CTS networking team can be found in the Appendix. 
Feel free to apply that display filter to your slow boot and logon and other network traces. Modify as needed.
 
Known issues
Vendor 
Product
Summary
Microsoft
Windows Server 2008
Windows Server 2008 R2
Story: Windows 7 clients report intermittent logon delays of 45 to 90 seconds while applying user settings  A Network Monitor trace with the display filter “Property.TcpSynRetransmit==1” shows the client issuing multiple SynReTransmits in the “high” ephemeral port range between ports 49152 and 65535. User logons are fast when randomly serviced by Windows Server 2003 DCs.  Root cause for the slow logon occurred because network connectivity over the “high” RPC port range was blocked by intermediate network devices. Logons serviced by Windows Server 2003 DCs remained “fast” as the connectivity over the “low” RPC port remained unblocked. See Active Directory and Active Directory Domain Services Port Requirements  . 
Microsoft 
Windows Server 2008 R2
Large collections of 6to4 adapters in Networks in Control Panel delays OS boot until KB 980486  is installed. 
Microsoft
Windows XP
Windows Server 2003 
A failure to negotiate opportunistic locking can delay the application of policy and other files from remote servers being read in 2-byte increments until you add the BufferPolicyReads registry setting on Windows Server 2003 and XP clients explained in KB 319440  . 
Microsoft
<need to figure this out>
Accessing files with long file names or in long paths may generate repetitive SMB transact2 network traffic that can delay the application of computer policy, user policy, user profile load or logon script load until the Infocache registry setting is set to 10 as specified in KB 834350  .
Microsoft
Windows – All versions
Sample experience: Logons to one server take some number of seconds longer than logons on another identical server. Root cause is that server 1 resides in a resource domain whose DNS Server is unable to resolve the DNS name for the profile server, \\<single label host name>\sharename, to an IP address. The inability to resolve DNS names to home directories, startup scripts, logon scripts or paths referenced in scripts can trigger long boot and logon delays. We recommend that you use fully qualified DNS names in paths for home directories, user profile, startup and logon scripts as well as paths referenced in those scripts. For example, file:///public vs. file:///public.  
Common DNS blockers include:
DNS servers in resource domains are not correctly configured to resolve DNS names in account domains and vice versa. 
DNS suffixes are not always configured on clients to resolve DNS names in non-local domains 
Windows Vista and later stopped resolving partially qualified DNS names by default 
Protective DNS devolution logic was added to prevent DNS queries from leaking beyond the organizational boundary 
Scavenging is not enabled in DNS zones causing lookups using stale DNS records to fail 
Microsoft
Windows 7
Windows Server 2008 R2
A common problem the CTS SBL team sees in XPERF traces is a 30 second delay waiting for the network to start before a user profile is loaded. The problem exists only on Windows 7 and Windows Server 2008 R2 computers. Resolved by installing MS KB 2709630  .

Microsoft
Windows 7
Windows Server 2008 R2 
Window 7 clients and Windows Server 2008 R2 servers that receive DHCP addresses using a DHCP relay experience a delay getting IP addresses. Boot delays of 5-30 seconds have been reported. Computer policy may fail to apply until the next refresh interval. Netlogon event 5719 with extended error “there are currently no logon servers available to service the logon request” in the System event is a partial telltale event. A network trace will show that the client acknowledged the DHCP request but the boot time firewall policy drops the unicast response from the DHCP relay. By the time the timeout interval expires, the firewall has started and the retry operation succeeds. Resolve by installing KB  2459530  on DHCP clients
Microsoft
Windows Server 2008
Windows Server 2008 R2
Story: Users logons with cached profiles take 2 minutes to complete. The WINLOGON pane in XPERF shows that the profile service, which loads user profiles and maps home directories, has a long execution (up to 2 minutes). The stack trace in CPU Scheduling with Ready thread shows the profile service mapping a home directory to a DFS path on the network. The TCP/IP Count Summary TABLE width="99%" time-cloned during the logon delay shows two session resets. A network trace shows the client sending an SMB Dialect negotiation request which the server never responds to, at which point the client resets the TCP session. No packets are exchanged for 60 seconds until the pattern is repeated for a second 1-minute delay. That the server fails to reply with an SMB dialect indicates a server side problem. The failure in the netmon trace is consistent with third-party TDI drivers interfering with the network stack. Start by installing the latest TDX.SYS from KB 961775  , and then enable the TdxPrematureConnectIndDisabled registry on the remote server followed by a remote. Remove TDI filter drivers as a last resort.
Microsoft
Windows Vista and Windows Server 2008 and later
User logons from domain-joined W7 clients normally take 30 seconds but intermittently take 8 minutes. XPERF shows that a series of logon scripts called by Group Policy preferences to establish mapped network drives are taking a long time to execute. A network trace shows clients experiencing slow logons authenticating with a local DC, pulling policy from a local DC but sourcing scripts defined in Group Policy preferences from a remote DC half-way around the world. On the client computer, IPv6 is bound in Network Connections in Control Panel (NCPA.CPL) but unbound on all DCs (that is, the TCP/IPv6 check box is cleared in the network adapter properties). 
Clearing the IPv6 check box does not disable the IPv6 protocol. 
Enabling the IPv6 binding in NCPA.CPL and setting disabledcomponents=FFFFFFFF on the Windows 7 client results in consistently fast logons as Group Policy preferences scripts are sourced from the local DC. One change to investigate is whether creating IPv6 subnet and subnet-to-site mappings would have resolved this problem.
 
 Microsoft
 Windows 7
Windows 7 clients experiencing slow logons due to the inability to resolve certain IPv6 records to a valid IP address for target computers used in the logon process. A review of the forward lookup zone showed duplicate AAAA records present for each DC in the client domain. Each DC had a single NIC which was assigned a single IP address. PINGS of current AAAA records would resolve while others would not. Problem resolved by removing an intermediate device on the network which was issuing router advertisements.
 Microsoft
 
Users experience slow logon times. The WINLOGON pane from an XPERF trace shows execution time by GPCLIENT. Similarly, the Generic Events Summary TABLE width="99%" shows large delays in gpclient execution. CPU Scheduling with ready thread summary TABLE width="99%" reveals that the GPSVC is waiting on GPSCRIPT.EXE to execute scripts. GPSCRIPT in turn is waiting on WSCRIPT.EXE to execute WSH scripts such as .vbs and .js scripts. The File I/O summary TABLE width="99%" reports a 45-second sharing violation encountered while accessing a script from a remote server. The next action is to identify the source of the file lock. 
 N/A
 Network infrastructure
User logons are fast if GPP Printer Policy is take 5 minutes if enabled on XP and Windows 7 clients. XPERF shows GPCLIENT taking in excess of 2 minutes. The Generic Events summary TABLE width="99%" shows a 100+ second delay applying GPP Printer policy. The CPU Scheduling Ready Thread with Summary TABLE width="99%" s shows network related errors as does the TCP Count Summary TABLE width="99%" . A client-side Netmon trace shows multiple periods where the network conversation between the client and DC contains large numbers of fast retransmits ( TCP:[Request Fast-Retransmit ) and Dup Acks ( TCP:[Dup Ack ) packets, as well as 4 to 12 second delays between client requests and server responses over LDAP, SMB, and other protocols. While enabling spanning on the switch to take a concurrent client and server side trace, the network administrator notices that the port speed is set to 100 MB on a Gigabit network. Setting port speed on the switch to match the underlying network speed removes over 4 minutes of user logon delay.
 N/A
 Network infrastructure
Windows 7 clients experience delayed user logon times. A network trace shows the default gateway taking 50 seconds to respond to an ARP request from the client in order to reach a server on a remote subnet. Problem resolved by enabling PORTFAST on a Spanning Tree enabled switch.
 Microsoft
Windows 7 
Windows Server 2008 R2

Slow boot and logon performance caused by Slow network i/o if SMB signing-enabled clients such as Windows 7 negotiate SMB 1 (used by  Windows Server 2003 servers for example).  Install KB 2689311  .
 Microsoft
Windows  XP 
Windows Vista
Windows 7
Windows Server 2003 Windows Server 2008 Windows Server 2008 R2
Windows clients experience OS boot and user logon delays. A network trace shows that all network i/o to remote subnets occurs over 576-byte frames. Found EnablePMTUDiscovery = 0 in the registry, which causes TCP to use a packet size of 576 bytes or smaller on remote subnets. Deleted EnablePMTUDiscovery from the registry after confirming support for larger frames by the underlying network using the PING –F –L command.
Microsoft
Windows Server 2008 R2 Standard Edition
Computers running Windows Server 2008 R2 configured with > 32 GB of system memory are vulnerable to slow network i/o due to a math error in tcpip.sys. Standard Edition only supports 32 GB of RAM. Microsoft Support is seeing computers with 36 GB to 128 GB of system memory installed in computers running Standard Edition which only supports 32 GB of RAM. Symptoms include slow reads, writes and file copies to or from remote servers, or the inability to copy files larger than a given size including the sending of email attachments in Outlook or Outlook Web Access. Network conversations are characterized by out-of-ordered packets, packet retransmits and duplicate acknowledgements (acks) and initially appears to be an underlying “network issue.” Workaround: Remove physical memory at or below 32 GB or use BCDEDIT  to constrain the amount of system memory available to the OS using either REMOVEMEMORY or MAXMEM. 
 ISV
Windows XP 
Windows Vista
Windows 7
Windows Server 2003 Windows Server 2008 Windows Server 2008 R2
Windows XP clients experience 5 minute logons. Logons are fast if a specific policy is enabled. Do to the weak ETL trace logging in Windows XP, Windows 7 is installed so the XPERF traces can be used to analyze performance. XPERF shows a 2+ minute delay for GPCLIENT to execute. The CPU Scheduling thread indicates that the client is waiting on an LDAP response from the server. The TCP Count summary TABLE width="99%" records “LossRecoveryStop” errors suggesting a problem with the underlying network. A network trace shows 10-second delays between frames sent between the client and authenticating DC. Fast retransmits and Duplicate ACKS represent 1% of the network traffic. Link speed for the port on the switch used by the DC was set to 100Mbs on a 1 GB network. Link speed in Networks in Control Panel on the DC was set to “autodetect.” Hardcoding link speed on both the DC and the switch results in 7 second logons. 

Operating System
You could argue that all of the Windows fixes mentioned previously are operating system (OS) bugs, but one point needs to be clear: code fixes for the majority of slow boot and logon root causes in Windows 7 and Windows Server 2008 R2 have long since been resolved by Service Pack 1 and post Service Pack 1 QFEs but as of (today's date = 2012.04.19) were not installed on the production clients and servers the SBSL team reviewed.
A few exceptions are: 
KB 2617858  – a recent fix that resolves an issue that causes the Windows Management Instrumentation (WMI) component to unnecessarily perform a full validation of the WMI repository. 
KB 2709630  - the 30 second delay in network startup that delays profile load. 
KB <being investigated> - a GPP item level targeting bug on filtered OUs that results in excessive network i/o between the client and the domain controller serving up policy. 
Microsoft Support  CTS still gets slow boot and logon calls from deployments running Windows 7 RTM or Windows XP or Windows Server 2003 SP1. Hopefully those deployments are in the minority. 
At the same time, we’re not advocating that administrators “shotgun” fixes on production computers.
Test, measure, observe followed by an incremental deployment is the order of the day. 
Known issues
Vendor 
Product
Summary
Microsoft
Windows 7
Windows Server 2008 R2
Console logons to Windows 7 and Windows Server 2008 R2 computers with multiple monitors hang at black screen. Resolved by KB  2645611  . 
Microsoft
Windows Server 2003
Windows Server 2008 
Windows Server 2008 R2
Windows Servers hosting the domain controller and DNS Server role may hang for nearly 20 minutes and log DNS Event ID 4013 due DNS waiting on Active Directory to replicate before loading DNS zones deadlocked with Active Directory waiting on DNS to load Active Directory-integrated zones so that it can resolve DC CNAME records to source IP addresses. This problem is primarily caused by configuring DCs to point to itself exclusively or as Primary DNS server. Point to one or more alternate DNS Servers and stage DC reboots smartly to ensure that DCs being rebooted point to an online DNS Server that can resolve DNS queries for direct replication partners during OS boot. See KB 2001093  for more information. 
Microsoft
Windows Vista
Windows Server 2008
SSL-enabled web servers may hang while “applying security policy” for several minutes to several hours due to http<-> crypto deadlock. Another clue: Numerous services that are configured to start automatically on reboot fail to auto-start. Install KB 2379016  to resolve the problem.  Extremely common problem on SSL-enabled Windows Server 2008 servers. 
Windows Server 2008 R2 not impacted.
Microsoft
Windows 7
Windows Server 2008 R2
Every other boot is slow on computers configured with more than 96 DPI. Problem resolved by installing KB 977419  or reducing DPI. 
Microsoft 
Windows 7
Windows Server 2008 R2
Wake from hibernate is slow on BitLocker-enabled computers unless KB 2294019  is installed. 
 

Policy including Group Policy Preferences
Boot and logon delays caused by Group Policy range from:
Sourcing Group Policy from a remote DC where the iterative network I/O used to apply policy, especially when preferences with item level targeting, is combined with WAN link latency (see Network). 
Specific policy settings are slow due to high workload. 
Group Policy preferences can inject a higher workload than legacy policy settings. 
Large number of LDAP or LSA calls triggered by Group Policy preferences configured with security group or LDAP-filtered item-level targeting due to misconfiguration and code defects. 
A good practice would be to profile computer and user policy application times to test the performance impact around the number of policies being applied and various policy settings, especially those that involve Group Policy preferences or drive mappings or the sourcing of data over the network (such as application installations, remote program execution, and so on). Also, review the network conversion between client servers and policy services to look for inefficiencies and reduce network I/O as much as possible. A side note, enabling the user environment logging can help to locate which policy takes too much time to execute and load (KB 221833   or KB 944043  (see the last section about gpsvc.log))
 
Known issues
Vendor 
Product
Summary
Microsoft
Windows 7
Enabling “Delete cached copies of roaming profiles” policy setting delays logoff as some yet-to-be-identified component in the OS is triggering Windows desktop search to cache settings on behalf of each locally cached profile at logoff. Larger numbers of cached profiles cause longer logoff delays. Logoff is fast when desktop search is disabled or the policy setting is not applied. Root cause is being investigated. 
Microsoft
Windows 7
Windows Server 2008 R2 
Enabling “Do not automatically make redirected folders available offline” policy causes delayed logon when offline files have been cached on the client, and redirected folders have been manually pinned to be available offline. Resolve by installing KB 2525332  .
Microsoft
Windows Server 2003 and later
Enabling “Shutdown: clear virtual memory pagefile” delays OS shutdown due to the workload associated with “clearing” the contents of pagefile.sys and hiberfil.sys. High workload causes longer wait. 
Microsoft
Internet Explorer 7
Internet Explorer 8 
Enabling Internet Explorer branding on computers running Internet Explorer 7 and Internet Explorer 8 may cause a 20-secondlogon delay unless KB 941158  is installed with the corresponding registry setting.
Microsoft
Windows Vista 
Installation of unsigned or “package-unaware” printer drivers via Group Policy preferences may result in slow logon. Install fixes KB 973772  or KB 974266  .
Microsoft
Vista 
Windows 7
Group Policy preferences installation of printer files causes slow first-time logons due to time spent copying a 300-500 MB file over the network, and executing the printer installer. Group Policy preference-based printer drive installation runs synchronously meaning that the operation must complete before the user logon operation can continue. High workload causes longer wait. Evaluate whether print drivers need to be installed on each unique logon. Install multiuser and "silent" printer driver installers as required.
Microsoft
Windows 7 
Windows 8 RTM version

Group Policy preferences configured with item level-targeting on security groups perform inefficient recursion of group members. The large number of LDAP queries causes boot or logon delays on the client and high CPU utilization on the servicing DC. Install the hotfix in KB 2561285  (for Windows 7) or a later version of GPPREFCL.DLL.
In extreme conditions we have seen item-level-targeting generate 17K and even 30K LDAP queries in a single user logon. 
The hotfix in KB 2561285  is planned to be included in Windows 7 SP1 and an additional hotfix is planned for Windows 8. 
Microsoft
Windows 7
Vista 
Windows 7 RTM clients subject to Group Policy preferences with item-level targeting issue a DSGETDC call requiring a wriTABLE width="99%" DC unless KB  983531   is installed. The RWDC requirement may trigger the selection of a remote-site DC or logon failure in a DMZ scenario.
The Vista version of this fix is KB 977983  .
Microsoft
Windows 7
Windows 7 or Windows Server 2008 R2 computers applying printer drivers by using Group Policy preferences in RODC-covered sites need KB 2537549  installed to remove a wriTABLE width="99%" DC dependency. 
Microsoft
Windows 7 
Windows Server 2008 R2
Windows 7 clients experience OS boot and user logon delays. SceCli event 1202 is logged with extended error 0x534: No mapping between account names and security IDs was done. 
GPMC creates invalid service accounts. Service Control Manager event 7000 indicates that the Diagnostic Service Host service failed to start. Problem resolved by KB 977695  . 

 Microsoft
Windows XP 
Windows Vista
Windows 7
Windows Server 2003 Windows Server 2008 Windows Server 2008 R2
RSOP logging to the WMI repository delays console and especially RDP logons. Delay is visible in USERENV.LOG on Windows XP and Windows Server 2003 and in GPSVC.LOG on Windows 7 and Windows Server 2008 R2. Larger WMI repository sizes in %windir%\system32\wbem\repository result in longer logon delays. Test for logon delay by temporarily setting “Turn off Resultant Set of Policy Logging” to "Enable" in GPEDIT.MSC. Determine if RSOP logging should remain enabled as a steady state configuration or only when GPRESULT output needs to be obtained from RSOP.MSC. RSOP modeling information can be obtained from GPMC.MSC independent of RSOP logging.
 Microsoft
Windows 7 
Windows Server 2008 R2
Windows computers using the Group Policy Application Deployment client-side extension experience slow logons if the client is covered (or the logon is serviced) by an RODC. Logon delays of up to 10 minutes have been reported. Microsoft-Windows-GroupPolicy event 7016 with extended error “2147943755” which maps to friendly error “The specified domain either does not exist or could not be contacted.” Resolved by installing MSKB 2537556  .
 

Profiles
Compared to the networking and policy areas, profiles generate comparatively few logon delays. As with all network I/O, sourcing user profiles over latent WAN links will delay both user logon and user logoff. Also, when you see long execution times for the profile service in the WINLOGON pane of an XPERF trace, don’t assume that it is the profile load that is running long. Other root causes include:
The 30 second network startup delay adds at least 30 seconds to the profile process 
The profile service maps home drives, so a delay in that operation, such as the inability to resolve the home directory path in DNS, can also cause logon delays. 
The profile service is waiting on some upstream dependency 
 
Known issues
Vendor 
Product
Summary
Microsoft
Windows Vista
Windows Server 2008
Windows 7
Windows Server 2008 R2
First-time user logons where new user profiles are derived from default user profiles are delayed by the execution of Active Setup routines that setup desktop shortcuts for Mail, Internet Explorer, and others. Active Setup does not execute on 2nd time logons using roaming profiles. Active Setup does execute for all logons using mandatory profiles or 2nd time logons where the cached roaming profile has been deleted by one of two policies. Artifact of profile change in windows Vista and later. 
Microsoft
Windows XP or Windows Server 2003 and later
Users report that first-time logons to a new computer are “slow” but subsequent logons to the same computer are “fast.” XPERF shows the profile service executing for 98 seconds and shows high disk I/O once profile load completes. A review of the default user profile shows that the c:\users\default\downloads folder contains some 381 MB of data, which gets copied every time a user logs on to a new computer and likely every logon to an RDP server or kiosk computer. High workload causes high delay. 
Microsoft
Windows Vista or Windows Server 2008 and later
User Logons hang for an extended time. Citrix logons hang while displaying “please wait for local session manager.” Microsoft-Windows-User Profiles Service event 1521 indicates that Windows cannot locate a profile due to error “access is denied.” 
Root cause: Folders and subfolders were manually copied into the users profile tree instead of following the steps in KB  973289  
Microsoft
Windows 7
Windows Server 2008
OS startup is delayed, and remote logon can be blocked, when services configured with a custom service account attempt to load a roaming profile while the “delete user profiles older than a specified number of days” policy attempts to delete the same profile. Setting the policy to delete profiles older than 1 day combined with daily reboots exacerbates the delay. 
Microsoft
 Windows 7
User logon hangs at solid blue screen for 45 seconds. XPERF shows that credential dialogs are waiting for username and password information but are not visible to the user. Problem occurs if default user profile contains mapped drives defined with alternate credentials. Logon is fast after drive mappings defined with alternate credentials is removed from default profiles and cached profiles derived from the invalid default user profile. 
