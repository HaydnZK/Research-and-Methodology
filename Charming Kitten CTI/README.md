# Charming Kitten
## Executive Summary & Threat Actor Profile
Charming Kitten, also known as APT35 or Mint Sandstorm, is an active Iranian state-sponsored cyber-espionage group that has been operating since at least 2011 on behalf of the Islamic Revolutionary Guard Corps (IRGC). The group primarily targets government officials, journalists, academics, defense contractors, and human rights activists across the United States, Europe, and the Middle East to conduct surveillance and harvest sensitive data. Instead of relying solely on sophisticated software exploits, they are notorious for advanced social engineering. They often spend weeks building rapport through fake online personas, like impersonating journalists or event organizers, to trick high-value targets into giving up their login credentials and bypassing multi-factor authentication.

---

## Attacker Profile & Intent
Broadly speaking, Charming Kitten is structured as an intelligence-gathering arm for the Iranian state, with an attacker profile defined by deep state integration rather than independent hacktivism. Operatively backed by the Islamic Revolutionary Guard Corps (IRGC), they function as a patient cyber-espionage unit that values high-effort social manipulation over raw technical complexity. Their primary intent is strategic surveillance and political survival. Rather than deploying destructive malware or seeking financial gain, they aim to systematically compromise the personal and professional accounts of geopolitical adversaries, dissidents, researchers, and journalists to unmask internal critics, monitor political discourse, and steal intelligence that directly supports Iran's foreign policy objectives.

### Attacker Profile
Charming Kitten operates as a highly structured, state-sanctioned threat actor deeply embedded within Iran's military intelligence apparatus. Their profile is defined by an unusual combination of intense social engineering patience and rapidly evolving technical capabilities that set them apart from traditional cybercriminals. By examining their organizational structure, behavioral traits, and recent tactical shifts, security teams can better understand how this persistent unit functions on the backend.
1. **Organizational Structure & State Affiliation**: Charming Kitten isn't a decentralized collective of hacktivists; it's a bureaucratic cyber unit.
* **The IRGC Umbrella**: The group operates directly under the Islamic Revolutionary Guard Corps (IRGC), specifically linked by intelligence agencies and whistleblowers to Unit 1500 (the IRGC's counterintelligence division).
* **The Composite Identity**: Major security firms track Charming Kitten under a variety of overlapping code names. While Microsoft breaks them down into subgroups under Mint Sandstorm, MITRE tracks their core cluster as Magic Hound (Group G0059), and CrowdStrike tracks them as Charming Kitten.
* **Front Companies & Contractors**: The operators themselves are often young, talented Iranian nationals. They're frequently hired through front-company contractors (such as Emennet Pasargad or Ajman Holding) who mask their government ties behind civilian software development storefronts.

2. **Operational Style & Behavioral DNA**: The group's operational profile is defined by a unique mix of extreme patience and occasionally messy security practices.
* **The High-Touch Social Engineer**: Unlike Western or Chinese APTs that lean heavily on technical sophistication, Charming Kitten's greatest strength is interpersonal manipulation. They're highly conversational. They'll spend weeks chatting with a victim via WhatsApp, LinkedIn, or encrypted emails before ever deploying a malicious link or document.
* **Aggressive and Bold**: Because they're backed by the IRGC, their operational style is highly aggressive. If they burn a domain or get caught by an endpoint sensor, they don't quietly retreat. They'll pivot, register a new domain, and immediately attempt to compromise the same target using a different angle.
* **Fluctuating Operational Security (OPSEC)**: Historically, their OPSEC has been notoriously sloppy. Analysts have caught them leaving their own infrastructure exposed, recording video tutorials of their screens that inadvertently showed their personal Persian accounts, and leaking credentials during operational transitions.

3. **Evolutionary Shift (Recent Profile Changes)**: While early profiles labeled them as a mid-tier technical threat that strictly relied on basic phishing, their profile has evolved significantly:
* **Rapid Vulnerability Weaponization**: They've shifted toward a highly mature subgroup model capable of immediately seizing on critical public software bugs (N-day vulnerabilities like Log4Shell or ProxyShell). Instead of taking weeks to weaponize an exploit, they can now deploy custom payloads within days of a vulnerability being made public.
* **Bespoke Tooling**: They've matured past generic open-source hacking tools. The group now designs and updates highly specialized, lightweight custom backdoors (such as MediaPl, MischiefTut, and BASICSTAR) engineered explicitly to bypass modern corporate endpoint detection systems.


### Attacker Intent
The core intent of Charming Kitten is driven by the strategic necessities, domestic anxieties, and geopolitical priorities of the Iranian regime. Unlike cybercriminal groups that seek financial profit or military APTs that focus on destructive sabotage, Charming Kitten operates almost exclusively as a strategic intelligence-gathering and surveillance machine. Their primary motivations and intent break down into the following major areas:
1. **Political Survival and Internal Dissident Tracking**: A primary driver of the group's operations is the elimination of internal threats to the Iranian regime.
* **Unmasking Dissidents & Activists**: The group seeks to infiltrate the communications of Iranian dissidents, human rights activists, and anti-regime organizers living abroad. By gaining access to these networks, they aim to map out resistance structures, identify coordinators inside Iran, and neutralize domestic political opposition.
* **Monitoring Journalists**: High-profile journalists working for Persian-language media outlets outside of Iran (such as Iran International, BBC Persian, and Voice of America) are constantly targeted. The intent is to identify their sources inside the country, monitor impending critical reporting, and compromise their professional networks.

2. **Geopolitical and Strategic Espionage**: Charming Kitten acts as the eyes and ears of the IRGC on the global stage, aligning its targets directly with Iran's foreign policy friction points.
* **Foreign Policy and Defense Insights**: They aggressively target diplomats, government officials, think-tank analysts, and policy researchers, particularly those specializing in Middle Eastern affairs, nuclear proliferation, and sanctions evasion. The goal is to steal confidential policy drafts, internal memos, and negotiation strategies regarding regional alliances or international sanctions.
* **Strategic Academic Research**: Academics and researchers working at major international universities are targeted not just for their personal data, but for their institutional access. Infiltrating an academic network allows the group to move laterally into defense-adjacent research or gain an understanding of Western policy-making frameworks.

3. **Credential Harvesting and Identity Theft**: The group's operational objective is rarely to destroy data, but rather to silently possess it.
* **Total Account Compromise**: The intent of their social engineering campaigns is to harvest corporate and personal credentials (primarily Google, Microsoft, and Yahoo accounts). By controlling a target's primary email, they can access personal chats, travel schedules, contact lists, and cloud storage repositories.
* **Long-Term Surveillance Access**: Once inside an account, they often configure silent forwarding rules or install malicious browser extensions. This allows them to maintain access over months or years, continuously siphoning data without alerting the victim or security teams.

4. **Supply Chain and Kinetic Target Mapping**: In recent years, a distinct subgroup within Charming Kitten (often tracked as Mint Sandstorm's infrastructure-focused clusters) has demonstrated an intent to map critical infrastructure and defense sectors.
* **Defense Contractor Profiling**: They target employees at defense industrial base organizations in the US and the Middle East to steal intellectual property and technical documentation relating to military hardware, drones, and aerospace technology.
* **Operational Technology (OT) Reconnaissance**: While they rarely execute destructive cyberattacks, they do gather actionable reconnaissance on maritime transport, port authorities, and energy systems. The intent here is dual-purpose: tracking logistics to counter sanctions, and maintaining a library of targeted access points that could be handed over to other Iranian military units for disruptive or kinetic action in the event of an open conflict.

---

## Attack Lifecycle & TTP Mapping
Charming Kitten's operational strategy is defined by a meticulous, multi-staged attack lifecycle that heavily prioritizes human manipulation over purely automated technical exploits. From the initial reconnaissance phase to data exfiltration, their Tactics, Techniques, and Procedures (TTPs) map closely to the MITRE ATT&CK framework, revealing a predictable but highly effective sequence of events. They typically begin with long-term target profiling, transition into high-touch social engineering to establish a foothold, and ultimately deploy specialized, lightweight malware designed to maintain stealthy, long-term access to compromised environments.


### Initial Access Vectors
Charming Kitten's initial access strategy stands apart from many other nation-state threat actors. Instead of utilizing sophisticated technological exploits or wide-reaching zero-days to breach network perimeters, they rely heavily on highly customized, multi-platform social engineering. They play a long game, establishing rapport and trust with victims over weeks before introducing malicious components.
1. **Multi-Channel Persona Impersonation**: The core of Charming Kitten's initial access model is the creation of believable, multi-layered digital personas.
* **The Journalist/Think-Tank Lure**: Operators frequently impersonate prominent journalists from international outlets (like Deutsche Welle, The Jewish Journal, or CNN) or researchers from think tanks. They contact targets, typically scholars, activists, or foreign policy experts, asking for interviews, comments on regional geopolitical affairs, or invitations to speak at a conference.
* **Cross-Platform Verification**: Rather than keeping the conversation confined to a single phishing email, they actively build out corresponding fake profiles on LinkedIn, WhatsApp, and Telegram. They'll message the target across these channels to verify their identity. In some documented cases, operators have even agreed to or initiated brief phone or video calls using voice-altering tools or stolen video clips to clear any suspicions the target might have.

2. **High-Touch Credential Phishing & Identity Spoofing**: Once the target's guard is down, the group shifts toward extracting credentials, usually targeting personal and professional Google, Microsoft Outlook, and Yahoo email accounts.
* **The Shared Document Hook**: The attacker sends a message stating they shared an interview outline, a policy brief, or a conference agenda via an encrypted link or cloud sharing folder (such as Google Drive or OneDrive).
* **Lookalike Login Pages (Reverse Proxy Phishing)**: Clicking the link routes the victim to a lookalike login page controlled by Charming Kitten. They frequently use advanced reverse-proxy frameworks (like Evilginx) that don't just steal the password, but actively capture session cookies and Multi-Factor Authentication (MFA) tokens in real time, allowing them to bypass traditional two-factor security instantly.

3. **Exploitation of Perimeter Vulnerabilities (Recent Pivot)**: While human targets remain their primary focus, specialized subgroups within Charming Kitten (specifically clusters tracked under Mint Sandstorm) have expanded into automated infrastructure targeting.
* **N-Day Vulnerability Scanning**: The group maintains automated scanning infrastructure that continuously crawls the internet for unpatched, internet-facing enterprise software.
* **Targeted Perimeter Infiltration**: They aggressively exploit newly disclosed vulnerabilities, such as ProxyShell/ProxyNotShell in Microsoft Exchange, Log4Shell, and critical remote code execution flaws in Confluence, Ivanti, and ConnectWise appliances. Once exploited, they immediately drop a lightweight web shell or backdoor to secure initial access before the organization can apply patches.

4. **Custom Lure Platforms and Software Malicious-VPNs**: In campaigns targeting highly technical individuals (such as defense sector workers or nuclear scientists), Charming Kitten builds entire pieces of custom deceptive infrastructure.
* **Fake Webinar Platforms**: The group has historically constructed fully realized, fake corporate or academic registration websites. They invite targets to join a private panel or exclusive webinar.
* **The Access Gate Tool**: When the target attempts to log into the webinar or view the conference materials, the site informs them that their connection must be secured. They're prompted to download a proprietary security certificate or a special corporate VPN tool, which is actually a custom malicious downloader or an archive containing a backdoor weaponized through .LNK or shortcut files.


### Pivoting & Escalating
Once Charming Kitten establishes initial access, whether through a compromised browser session via phishing or by dropping a web shell onto a perimeter appliance, they immediately shift to consolidating their position. Historically, the group's internal operations relied heavily on standard, noisy tools, but they've increasingly adopted a living off the land approach combined with tailored privilege escalation and lateral movement. Here's how Charming Kitten executes privilege escalation and lateral network pivoting:
1. **Local Privilege Escalation (LPE)**: Depending on how they entered the environment, Charming Kitten utilizes distinct paths to elevate themselves to administrative or SYSTEM-level control:
* **Exploitation for Privilege Escalation (MITRE ATT&CK T1068)**: When the group breaches a network via internet-facing appliances (like Microsoft Exchange, Ivanti, or Confluence), they often land as a low-privileged service account. They quickly deploy public, locally compiled exploits for known Windows or Linux kernel vulnerabilities to jump straight to SYSTEM or root privileges.
* **Misconfiguration Abuse & Service Manipulations**: They actively look for unsecured service permissions, unquoted service paths, or vulnerable scheduled tasks running under administrative privileges. They alter these configurations to execute their custom PowerShell or VBS scripts during system reboots or task cycles.

2. **Credential Dumping and Defense Evasion**: To move effectively beyond a single machine, Charming Kitten must strip away local security and harvest credentials.
* **Memory Dumping**: They heavily leverage Mimikatz or standard administrative tools like procdump to target the Local Security Authority Subsystem Service (LSASS) process memory. This allows them to harvest cleartext passwords, NTLM hashes, and Kerberos tickets.
* **Blinding Local Defenses**: Before executing aggressive credential dumps, they make an effort to bypass security controls. This includes attempting to disable local antivirus monitoring (like Windows Defender), clearing Windows Event Logs (wevtutil) to obscure their footprint, and tampering with LSA Protection to allow the memory-dumping tools to execute without triggering immediate alerts.

3. **Lateral Movement and Internal Pivoting**: Once administrative credentials or active session hashes are secured, Charming Kitten begins pivoting, moving laterally from the compromised staging machine deeper into the internal network.
* **Remote Desktop Protocol (RDP) & Native Admin Tools**: The group prefers abusing legitimate, native administrative channels to blend in with normal IT traffic. They frequently utilize RDP with stolen credentials or deploy PsExec to run commands silently on remote servers.
* **WMI and Command Execution**: They make extensive use of Windows Management Instrumentation (WMI) and standard network administration commands (such as net use or net view) to query active directory configurations, map internal shares, and execute remote staging payloads on high-value targets like Domain Controllers or file servers.
* **Browser Session Hijacking**: On individual endpoints (such as a targeted journalist's or researcher's workstation), they use custom scripts and backdoors to steal stored browser cookies and database files. This allows them to pivot laterally into internal cloud applications, enterprise email environments, or intranet networks without needing to compromise the underlying network architecture.


### C2 & Exfiltration
Charming Kitten's Command and Control (C2) strategy is designed to balance operational stealth with a high degree of adaptability. Rather than standard, easily blocked standalone C2 servers, they rely on a hybrid architecture that blends legitimate cloud infrastructure, custom communication protocols, and strategic fallbacks to maintain control over infected hosts.
1. **Dynamic GitHub & Cloud Staging**: The group frequently hides its C2 infrastructure behind legitimate, heavily trusted cloud services to bypass firewalls and domain whitelists.
* **GitHub Repository Injection**: In many campaigns utilizing their custom malware (such as Drokbk or CharmPower/POWERSTAR), the malware doesn't contain hardcoded C2 addresses. Instead, it's programmed to silently query specific, attacker-controlled GitHub repositories. The group updates text files within these repositories containing encrypted strings that reveal the actual active C2 domain. If an active C2 domain is blocked by defenders, Charming Kitten simply updates the GitHub repository with a new domain, rotating their infrastructure instantly without modifying the malware binary.
* **Legitimate Webhooks & S3 Buckets**: For rapid data relay and configuration management, subgroups have heavily adopted third-party testing services like webhook.site and public Amazon S3 buckets. Because these domains host massive amounts of legitimate corporate traffic, enterprise security sensors rarely alert on outbound connections to them.

2. **Specialized Implants & Encrypted Communication**: When interacting with high-value targets, the group shifts to bespoke HTTP/HTTPS communication channels managed by custom-designed implants.
* **The MediaPl Implant**: This custom DLL masquerades as legitimate Windows Media Player components. It communicates with C2 infrastructure using AES and Base64 encryption to mask the raw commands and responses inside seemingly normal web requests.
* **BellaCiao & BellaCPP**: Originally deployed as a .NET web shell/tunneling tool, the group recently evolved this framework into a C++ reimplementation (BellaCPP). It maps inbound network traffic, establishes persistent proxy tunnels, and allows the attackers to inject remote commands into internal networks while heavily resisting reverse engineering.

3. **Legacy IRC Fallback Channels**: A unique, identifying behavioral signature of Charming Kitten is their persistent use of Internet Relay Chat (IRC) protocols for C2. If their primary HTTP/HTTPS web tunnels are severed by network security teams, their secondary implants are designed to fall back to communicating via raw IRC text streams (frequently using legacy domains like serveirc.com). This protocol-switching technique is highly effective at evading standard network monitoring tools that only inspect standard web (HTTP) traffic.


#### **Data Exfiltration Tradecraft**
Charming Kitten's data exfiltration is highly systematic, shifting seamlessly between targeted enterprise network data extraction and deep, automated cloud-to-cloud inbox harvesting.
1. **Network Staging and SSH Tunneling**: Once they've collected proprietary data from internal corporate shares, Active Directory databases, or file servers, they orchestrate localized staging before sending it outbound.
* **ProgramData Compressed Archives**: They utilize native Windows utilities (like tar.exe or powershell) to compress targeted file directories into heavily encrypted RAR or ZIP archives, typically storing them in hidden directories like C:\ProgramData\ or user AppData paths to hide from casual inspection.
* **Encrypted Tunnels**: Rather than using basic web uploads, they prefer executing exfiltration over secure, native protocols. They commonly establish SSH tunnels or utilize open-source utilities like Plink/Impacket to push the staged archives out of the victim's network directly onto their remote servers.

2. **Automated Cloud-to-Cloud Inbox Siphoning: HYPERSCRAPE**: Because a core intent of the group is intercepting human conversations, they developed a highly advanced, automated email extraction engine known as HYPERSCRAPE.
* **User Agent Spoofing**: Once they obtain a target's valid credentials through reverse-proxy phishing, the HYPERSCRAPE tool is executed directly from the attacker's machine, meaning it never touches the victim's local network. The tool logs into the target's webmail (Gmail, Yahoo, or Outlook) by spoofing user agents to mimic legacy web browsers.
* **Bulk Download & Clean Up**: The tool automatically changes the inbox language settings to English, downloads every message sequentially as an .eml file, and programmatically resets the emails back to unread status to prevent suspicion. Finally, it parses the inbox to delete any automated security alert emails sent by Google or Microsoft regarding the suspicious login before reverting the inbox back to its original language.


### Persistence Mechanisms
To ensure they maintain access to a compromised system or network even after a machine reboots, a user changes their password, or a network connection resets, Charming Kitten deploys an array of reliable persistence mechanisms. Their tradecraft spans traditional Windows configuration abuse, custom malicious scripts, and deep cloud-based architecture manipulation. Here are Charming Kitten's persistence mechanisms in great detail:

1. **Registry Run Keys & Startup Folder Exploitation**: The group heavily relies on standard Windows registry manipulation to ensure their backdoors automatically execute upon system startup or user login.
* **Registry Run Keys (MITRE ATT&CK T1547.001)**: They frequently add string values to the HKCU\Software\Microsoft\Windows\CurrentVersion\Run or HKLM\Software\Microsoft\Windows\CurrentVersion\Run keys. These strings are typically configured to execute malicious PowerShell scripts, Visual Basic Scripts (VBS), or custom DLLs hidden deep within default directories.
* **Malicious Shortcut Files (.LNK)**: To blend in, they copy weaponized shortcut files or lightweight script components directly into the Windows Startup folder. When the targeted user logs into their workstation, the system automatically runs the script, which beacons back out to Charming Kitten's command and control (C2) infrastructure.

2. **Abuse of Scheduled Tasks and Windows Services**: For broader enterprise persistence that doesn't rely on an active user logging in, the group migrates into system automation tools.
* **Scheduled Tasks (MITRE ATT&CK T1053.005)**: Charming Kitten regularly schedules malicious tasks via the command line (schtasks /create) or PowerShell. These tasks are often disguised using deceptive names that mimic critical security software, updates, or native system utilities (such as naming a task GoogleUpdateTask or OneDriveTelemetry). They set these tasks to trigger at specific intervals, such as every 30 minutes, or upon system boot, acting as a persistent mechanism to re-download or restart their implants if they're killed by defenders.
* **Custom Service Creation**: If administrative or SYSTEM privileges are achieved, they install custom Windows Services. These services run quietly in the background, utilizing their specialized C++ or .NET implants to maintain permanent communication lines.

3. **Application Shimming and Scheduled Script Drop (like PowerLess/POWERSTAR)**: When deploying their flagship modular backdoors, the group configures highly persistent, multi-layered execution loops.
* **The PowerLess Loop**: In campaigns using the PowerLess backdoor, the initial payload drops an encrypted blob onto the host. It sets up a persistent task that executes a native Windows utility (like regsvr32.exe or rundll32.exe) to decrypt and run the malware purely in memory every time the operating system boots. Because the primary payload is encrypted on disk and only uncoils via native processes, it highly resists static detection while maintaining persistence.

4. **Malicious Browser Extensions**: Because a massive portion of Charming Kitten's target profile involves harvesting webmail and cloud service communications, they frequently establish persistence directly inside the victim's web browser.
* **Silent Extension Deployment**: Once they gain initial access to an endpoint, they inject custom, malicious extensions directly into Google Chrome, Microsoft Edge, or Mozilla Firefox.
* **Session and Data Siphoning**: These extensions are written to mirror legitimate ad-blockers or productivity tools. Once installed, they run persistently in the background of the browser application. Even if the user changes their primary account passwords or clears their cookies, the extension remains active, continuously monitoring web traffic, extracting newly entered credentials, and stealing active session tokens to pass back to the attackers.

5. **Cloud-Side Persistence (The Ultimate Fallback)**: Recognizing that endpoint security tools (EDR) may eventually detect and wipe their local malware, Charming Kitten has perfected cloud-side persistence, which requires no active footprint on the victim's physical machine.
* **OAuth Application Abuse (MITRE ATT&CK T1136.003)**: During a successful reverse-proxy phishing attack or account takeover, they authorize a malicious, attacker-controlled OAuth application within the victim's corporate Google Workspace or Microsoft 365 environment.
* **Permanent API Access**: This grants the group long-term API access to the target's mailbox, calendar, and cloud drives. Even if the victim realizes they were phished, changes their corporate password, and enforces mandatory multi-factor authentication, the malicious OAuth token remains valid. This allows Charming Kitten to bypass all authentication barriers and silently read or exfiltrate cloud data directly via API endpoints until the enterprise IT administrator manually revokes the application's permissions.


### Impact
The impact of Charming Kitten's cyber operations is measured primarily in human risk, strategic intelligence loss, and geopolitical destabilization rather than direct financial damages. Because they function as an espionage arm of the IRGC, their successful breaches create severe real-world consequences for individuals and national security networks alike. The downstream impacts of their attacks can be broken down into four major areas:
1. **The Human Toll: Transnational Repression and Physical Danger**: Because a massive portion of Charming Kitten's target profile consists of human rights defenders, anti-regime dissidents, and exiles, a compromise of their digital accounts carries immediate, physical safety risks.
* **Compromising Informant Networks**: When the group successfully infiltrates a journalist's or activist's inbox, they don't just see that individual's data, they gain access to the identities and locations of whistleblowers and family members living inside Iran.
* **Kinetic Retaliation**: Intelligence gathered by Charming Kitten has directly fed into the Iranian state's mechanism for transnational repression. Leaked data has been utilized by the IRGC to facilitate the harassment, arbitrary arrest, or intimidation of dissidents' families inside Iran, and has even been linked to coordinated kidnapping plots or targeted assassinations of exiles abroad.

2. **Geopolitical and Foreign Policy Disruption**: By intercepting the private communications of diplomats, think-tank analysts, and policy advisers, Charming Kitten alters the playing field of international diplomacy.
* **Loss of Strategic Leverage**: Siphoning confidential policy drafts, sanctions-evasion tracking data, and internal government briefings allows Tehran to anticipate Western and Middle Eastern diplomatic moves.
* **Political Interference**: Subgroups have historically leveraged their access to compromise high-profile political campaigns (such as US presidential campaigns) and election infrastructure. While they rarely alter vote counts, stealing internal campaign emails allows them to conduct cyber-enabled influence operations or leak data to manipulate public discourse.

3. **Compromise of Critical Infrastructure and Kinetic Readiness**: In recent years, specialized subgroups of Mint Sandstorm have pivoted away from strictly civil-society targets to run aggressive reconnaissance on critical infrastructure base sectors.
* **Mapping Technical Blueprints**: Infiltrating maritime ports, transportation systems, energy sectors, and defense contractors in the US and Middle East allows the group to map operational technology (OT) and steal sensitive defense schematics.
* **Pre-Positioning for Future Conflict**: The impact here is an active degradation of military and infrastructure security. While Charming Kitten generally avoids pulling the trigger on destructive wiper malware or ransomware, they act as the scouts. They establish persistent backdoors and proxy tunnels, pre-positioning access points that can be handed over to other Iranian military units to execute disruptive or kinetic cyber strikes during an open geopolitical conflict.

4. **Mass Information Exfiltration and Regional Infiltration**: When the group targets regional government frameworks, the scale of data loss is massive.
* **Siphoning Institutional Knowledge**: Documented historical operations, such as Operation Desert Breach in Jordan or Operation Afghan Infiltration, resulted in the theft of tens of gigabytes of sensitive data from government ministries, law firms, and telecommunications providers.
* **Supply Chain Infiltration**: Compromising regional telecoms gives them the ability to intercept domestic traffic and expand their surveillance apparatus across entire geographic populations without needing to target every individual user endpoint sequentially.


### TTP Mapping
To provide a highly scannable and comprehensive overview of Charming Kitten's operations, their Tactics, Techniques, and Procedures (TTPs) are mapped below directly to the MITRE ATT&CK Framework.
1. **Initial Access & Reconnaissance**

| MITRE ATT&CK Technique | Technique ID | Charming Kitten Implementation Details |
|---|---|---|
| Gather Victim Identity Info | T1589 | Aggressively scrapes open-source intelligence (OSINT), LinkedIn, and social media to map out a target's professional network, interests, and colleagues. |
| Phishing: Spearphishing Attachment | T1566.001 | Sends malicious .LNK shortcut files disguised as interview schedules, or weaponized documents that fetch remote templates or drop backdoors. |
| Phishing: Spearphishing Link | T1566.002 | Uses a multi-platform approach (WhatsApp, LinkedIn, Email) to send links to highly accurate reverse-proxy phishing landing pages. |
| Exploit Public-Facing Application | T1583 | Automated mass-scanning and rapid weaponization of N-day perimeter vulnerabilities (Log4Shell, ProxyShell, Ivanti, ConnectWise). |

2. **Execution & Privilege Escalation**

| MITRE ATT&CK Technique | Technique ID | Charming Kitten Implementation Details |
|---|---|---|
| Command and Scripting Interpreter | T1059 | Heavily relies on custom PowerShell and VBScript execution loops to decode payload fragments and launch memory-only implants. |
| Exploitation for Privilege Escalation | T1068 | Deploys public, locally compiled kernel exploits directly onto compromised web servers or perimeter appliances to immediately escalate privileges to SYSTEM or root. |
| User Execution: Malicious File | T1204.002 | Leverages long-term social engineering rapport to trick targets into downloading and executing custom, malicious VPN tools or webinar viewers. |

3. **Persistence & Defense Evasion**

| MITRE ATT&CK Technique | Technique ID | Charming Kitten Implementation Details |
|---|---|---|
| Boot or Logon Autostart Execution | T1547.001 | Adds malicious execution paths into traditional Windows Registry Run Keys (HKCU and HKLM) or copies .LNK files directly to the Startup folder. |
| Scheduled Task/Job: Scheduled Task | T1053.005 | Creates persistent, recurrent tasks (schtasks /create) disguised behind legitimate system names like GoogleUpdateTask or OneDriveTelemetry. |
| Server Software Component: Web Shell | T1505.003 | Drops customized server-side backdoors (like the .NET-based BellaCiao or BellaCPP) onto compromised web servers to maintain permanent, deep access. |
| Account Manipulation: OAuth Applications | T1098.003 | Authorizes malicious, attacker-controlled OAuth applications in a victim's Google Workspace or Microsoft 365 cloud environment, guaranteeing lifetime API access. |
| Browser Extensions | T1176 | Silently installs custom, malicious extensions in Google Chrome or Microsoft Edge that run persistently in the background to capture session data. |
| Impair Defenses: Indicator Blocking | T1562.001 | Programmatically alters local configurations to blind endpoint security tools, kill Windows Defender, and clear local Event Logs (wevtutil). |

4. **Lateral Movement & Credential Access**

| MITRE ATT&CK Technique | Technique ID | Charming Kitten Implementation Details |
|---|---|---|
| OS Credential Dumping | T1003 | Targets the LSASS process memory using specialized variants of Mimikatz or native tools like procdump to extract cleartext passwords and NTLM hashes. |
| Remote Services: Remote Desktop Protocol | T1021.001 | Prefers utilizing native administrative tools like RDP with harvested domain credentials to seamlessly move laterally across internal networks. |
| Windows Management Instrumentation | T1047 | Leverages WMI commands to silently gather system data, query Active Directory environments, and stage payloads on remote corporate servers. |

5. **Command & Control (C2) & Exfiltration**

| MITRE ATT&CK Technique | Technique ID | Charming Kitten Implementation Details |
|---|---|---|
| Data Encoding: Standard Encoding | T1132.001 | Packs commands and exfiltrated payloads inside complex Base64 and AES-encrypted strings embedded natively inside HTTP/HTTPS requests. |
| Web Service: Bidirectional Communication | T1102.002 | Programmatically queries active, attacker-owned GitHub repositories containing hidden text files to dynamically retrieve updated C2 domain strings. |
| Application Layer Protocol: Mail Protocols | T1071.003 | Historically uses custom implants that fall back to standard Internet Relay Chat (IRC) streams if web-based HTTP firewalls block primary channels. |
| Archive Collected Data: Archive via Utility | T1560.001 | Compresses stolen file directories into heavily encrypted RAR or ZIP archives using native command-line functions before pushing them out of the network. |
| Automated Exfiltration | T1020 | Deploys proprietary automated collection engines, like HYPERSCRAPE, to rapidly siphon entire webmail inboxes directly from cloud servers. |

---

## Tooling & Infrastructure
Charming Kitten's operational capability is sustained by a distinct hybrid infrastructure and a modular tooling ecosystem that blends custom-built malware with legitimate cloud services. To balance stealth with adaptability, the group maintains extensive command-and-control (C2) frameworks utilizing trusted platforms like GitHub and OneDrive alongside traditional virtual private servers. Their software arsenal is heavily tailored for persistence and data collection, featuring dynamic, lightweight PowerShell backdoors, such as PowerLess and POWERSTAR, as well as specialized utilities like HYPERSCRAPE designed explicitly for automated cloud-to-cloud inbox harvesting.


### Tooling
Charming Kitten's software development philosophy relies on modular engineering. Instead of bundling a massive, monolithic piece of malware that's easily flagged by Endpoint Detection and Response (EDR) solutions, they favor lightweight loader scripts that decode, assemble, and launch specialized functional modules directly into system memory.
A detailed, technical dissection of their primary tooling categories follows:
1. **Flagship PowerShell Backdoors**: PowerShell remains the group's favorite execution interface due to its native integration within Windows environments.
* **PowerLess (Stealth Inversion Engine)**: One of Charming Kitten's most effective innovations, PowerLess is a modular .NET-based backdoor that executes PowerShell commands without invoking powershell.exe. By loading the core .NET Common Language Runtime (CLR) components natively into alternative system binaries, the malware completely blinds logging mechanisms like PowerShell Script Block Logging (Event ID 4104) and evades simple command-line process audits.
* **POWERSTAR (aka GorjolEcho)**: This is a mature, fully featured modular framework used in high-value espionage targets. POWERSTAR is designed with rigorous operational security (OPSEC); it gathers extensive system environment telemetry, checks for active sandbox or analysis environments, and securely reports to its C2 infrastructure before downloading secondary capability modules. Volexity research notes its primary delivery route includes multi-stage LNK chains or macro-enabled Dropbox links.

2. **Emerging Infiltration Backdoors (C++ Era)**: To bypass the growing signature dictionaries of Western security products, the group has spent recent cycles porting older scripts into compiled, object-oriented languages.
* **BASICSTAR & KORKULOADER**: Commonly distributed in targeted campaigns via compressed archives containing obfuscated .LNK shortcut files. BASICSTAR serves as a fast-acting staging implant that establishes initial contact with the C2, gathers file structures, and sets up a secondary infection lane.
* **BellaCiao & BellaCPP**: Originally deployed as a tailored .NET web shell implant, the group actively evolved this framework into a compiled C++ variant (BellaCPP). Acting as a hybrid proxy tunnel and command injector, BellaCPP is dynamically compiled on a per-victim basis. It leverages the victim's own localized IP geolocation metadata to customize its internal variable strings, making broad signature-based tracking difficult for defenders.
* **MediaPl (aka EYEGLASS)**: This highly specialized backdoor masquerades as a legitimate Windows Media Player dynamic link library (.dll) file. It's primarily engineered to run silently, harvesting keystrokes, clipboard data, and local credentials while utilizing heavily encrypted Base64 communication loops.

3. **Automated Data Harvesting Engines**: When initial access yields a valid set of cloud credentials, the group moves past traditional malware entirely to leverage purpose-built automation tools.
* **HYPERSCRAPE (Cloud Mail Siphoner)**: Developed natively by Charming Kitten, HYPERSCRAPE is an automated script-driven utility built specifically to pull massive amounts of data out of Gmail, Microsoft Outlook, and Yahoo webmail platforms.
* **Operation Protocol**: The tool is executed on the attacker's server infrastructure rather than the victim's workstation. It logs into the account using stolen credentials, systematically manipulates the web session's User-Agent to mimic a legacy or mobile browser, translates the interface to English, and copies every email as a raw .eml file. It then programmatically changes the state of unread messages back to unread and deletes any security alert emails generated by the platform providers to mask the breach entirely.

4. **Auxiliary Post-Exploitation Modules**: Once an environment is thoroughly compromised, Charming Kitten relies on a library of lightweight, tactical utility executables to carry out specific commands:
* **CommandCam (CommandCam.exe)**: A lightweight utility dropped onto endpoints specifically to take silent, automated photos using the victim's laptop or workstation webcam.
* **AudioRecorder (AudioRecorder4.exe)**: A custom background service deployed to tap into the device's internal microphone array, recording local room audio and packaging it into small compressed files for exfiltration.
* **NirSoft Tools Abuse**: They frequently leverage legitimate, renamed security tool binaries, such as Chrome History Viewers or WebBrowserPassView, to harvest browser data without triggering traditional malicious file alarms.


### Infrastructure
Charming Kitten builds dynamic, flexible network infrastructure designed to survive fast domain blacklisting by defenders. Rather than deploying highly elaborate, sprawling server configurations, they prioritize a low-footprint architecture that utilizes trusted global web platforms alongside carefully selected hosting environments. Their infrastructure framework is divided into three functional pillars:
1. **The Cloud Cover Layer (Abusing Trusted Platforms)**: Charming Kitten reduces its reliance on easily flagged virtual private servers (VPS) by routing command, control, and staging signals directly through legitimate public web utilities.
* **Dead Drop Resolvers (GitHub/Firebase/Dropbox)**: In their modern campaigns, the group hides the final destination of their C2 servers inside encrypted text strings posted to legitimate developer platforms like GitHub or public Firebase pages. When an implant executes on a target system, it calls out to these trusted platforms first to fetch the encrypted text. This allows the attackers to instantly swap a blocked C2 domain out for a fresh one simply by editing a file on GitHub, avoiding any need to compile new malware binaries.
* **Cloud-to-Cloud Intermediaries**: For immediate data exfiltration and payload hosting, they rely on commercial cloud solutions like Amazon S3, Dropbox, and OneDrive. Outbound enterprise traffic to these services is rarely blocked, allowing them to stage malicious updates or siphon compressed data under the radar.
* **Operational Webhooks**: When dealing with immediate command execution or testing initial access, they leverage automated platforms like webhook.site. This functions as an external listening post that accepts target machine telemetry via standardized outbound HTTP traffic.

2. **Domain Generation and Lookalike Masquerading**: When the group does deploy standalone domains for reverse-proxy phishing or direct backdoor interaction, they use specific naming and registration pipelines.
* **Dynamic DNS (DDNS) Ecosystems**: Threat intelligence firms track distinct infrastructure clusters that heavily abuse dynamic DNS providers including Dynu, DNSEXIT, and Vitalwerks. This permits rapid, cheap provisioning of hostnames that can be discarded within hours of exposure.
* **Deceptive Naming Conventions**: Their phishing and C2 domains are meticulously tailored to simulate trusted enterprise systems. They lean heavily on themes related to cloud file sharing, document visualization, and administrative portals, frequently combining strings like drive-google, sharepoint-verify, or view-document with slightly tweaked top-level domains.
* **The One-to-One Infra Operational Model**: Unlike cybercriminals who point thousands of compromised machines to a single C2 server, Charming Kitten frequently creates unique domains or subdomains for a single high-value target. If that specific target realizes they're being attacked and reports the domain, only that isolated piece of infrastructure is burned, preserving the rest of the IRGC campaign.

3. **Web Hosting & Physical Infrastructure Alignment**: The backend physical layer of Charming Kitten's architecture reflects the group's state-sponsored nature and its operational boundaries.
* **Bulletproof Providers**: To protect their primary backend management servers from takedown requests by Western authorities, they host their landing applications and databases with bulletproof hosting providers operating out of regions that don't cooperate with international cyber-law enforcement, such as Russia or parts of Eastern Europe.
* **Domestic Staging Clusters**: For core infrastructure management, the group relies on VPS ranges allocated directly inside Iranian sovereign IP space (such as providers operating out of Tehran). While this results in distinct tracking signatures for researchers, it guarantees that their data storage can't be seized or disrupted remotely.
* **Compromised Small Business Infrastructure**: They frequently compromise and hijack legitimate, small-scale commercial websites globally to use as temporary forward proxies or watering holes. This allows them to bounce their real C2 communications through innocent third-party servers, tricking local defense teams into seeing traffic destined for a benign small-business domain.

---

## Real-World Scenarios
Charming Kitten's operational history is marked by aggressive, high-stakes campaigns that align perfectly with Iran's real-world geopolitical flashpoints and domestic anxieties. Over the years, their targeting strategy has evolved from broad credential-harvesting operations to surgical, high-effort strikes against international diplomats, critical infrastructure sectors, and high-profile political events. Whether infiltrating regional government ministries, compromising U.S. presidential campaigns, or systematically tracking down human rights activists and journalists abroad, the group's real-world attacks consistently aim to undermine geopolitical adversaries and secure strategic intelligence for the Iranian state.


### Top Five Scenarios
To understand how Charming Kitten applies its tactics in the real world, it's useful to look at their track record. While they've launched dozens of operations over the past decade, their broad strategic capabilities are best illustrated by five of their most notorious campaigns:
1. **The Newscaster Campaign (2014)**: One of the earliest and largest public exposures of the group. They constructed a massive, completely fake news network called News Arena alongside highly detailed personas of fictional journalists. They used these fake credentials to infiltrate the social networks of high-ranking U.S. military officers, politicians, and defense contractors.
2. **The HBO Cyberattack & Extortion (2017)**: A major pivot from purely quiet espionage. An Iranian actor linked directly to Charming Kitten compromised the networks of Home Box Office (HBO), stealing unreleased episodes of major television shows (including Game of Thrones) and demanding a $6 million Bitcoin ransom before being publicly indicted by the U.S. Department of Justice.
3. **The U.S. Presidential Election Operations (2020 & 2024)**: Coordinated, high-stakes interference attempts targeting both ends of the political spectrum. The group launched thousands of targeted spear-phishing and credential-harvesting attacks directly aimed at campaign staff, political operatives, and foreign policy advisers to steal internal campaign data.
4. **Targeted Vaccine Research Interception (2020)**: Amidst the global pandemic, Charming Kitten aggressively pivoted its targeting toward the healthcare sector. They deployed tailored phishing lures against international pharmaceutical corporations and major academic laboratories in the U.S. and the U.K. to exfiltrate proprietary COVID-19 vaccine research data.
5. **The Multi-State Enterprise Leaks & Breach Operations (2024–2025)**: A massive series of opportunistic, highly automated infrastructure hacks. After a significant backend leak exposed the group's internal files, security researchers uncovered a sprawling campaign where Charming Kitten used automated scanners to hit unpatched perimeter software, stealing massive data caches like 74GB of sensitive records from ministries and law firms.

### The Newscaster Campaign (2014)
Discovered and exposed by the cybersecurity firm iSIGHT Partners in May 2014, Operation Newscaster stands as one of the most structurally elaborate and patient social engineering campaigns ever documented. For over three years, Charming Kitten (then operating as the Newscaster Team) bypassed technical firewalls entirely by exploiting human trust. They systematically targeted more than 2,000 high-value individuals, including U.S. Navy admirals, lawmakers, defense contractors, and foreign diplomats across the U.S., the U.K., Israel, and Saudi Arabia. The anatomy of this highly sophisticated campaign is broken down below:
1. **The Lure Architecture: NewsOnAir.org**: Rather than creating simple fake email addresses, Charming Kitten built a fully operational, legitimate-looking digital front to validate their malicious entities.
* **The Fabrication**: They registered and deployed a fake journalism website called NewsOnAir.org. The website was engineered to look like an authentic independent news aggregator.
* **Content Siphoning**: The site automatically siphoned and republished real articles from valid mainstream media outlets (like BBC, Reuters, and Associated Press). By populating the site with legitimate, unaltered journalism, any target who checked the domain to verify an automated inquiry found a professional, fully functional news platform.

2. **Multi-Tier Social Media Persona Matrix**: With their fake news agency established, the group created 14 highly detailed, distinct online personas mapped across major networking platforms like Facebook, LinkedIn, Twitter, and Google+. These personas were generally split into two categories:
* **Level 1**: The Attractive Scouts: These profiles used stolen photos of young, attractive individuals. Their sole purpose was to aggressively expand their network by sending generic friend requests to defense personnel and government staff. They engaged in light, benign banter to establish a low-level footprint on the target's friend lists.
* **Level 2**: The Credible Journalists: These profiles masqueraded as senior political reporters or editors for NewsOnAir.org. They were given professional headshots, dense resumes on LinkedIn, and active feeds discussing Middle Eastern geopolitics.

3. **Exploitation of the Mutual Friend Bias**: The true technical and psychological brilliance of Operation Newscaster lay in how they bypassed platform security flags and target skepticism.
* **Network Seeding**: The Scout personas first targeted low-level contractors, interns, or retired military personnel who had lower security awareness. Once a few low-level targets accepted the request, the hackers moved up the chain of command.
* **Social Proofing**: When a high-ranking target (like a U.S. Navy Admiral) received a connection request from a fake NewsOnAir.org journalist, the target would see that they already shared 5 or 10 mutual friends within the defense community. This layer of false social proofing regularly tricked high-security individuals into accepting the request without verification.

4. **The Initial Access Mechanics & Parastoo Connection**: Once inside a target's network, the group played a long game, often waiting weeks before attempting an exploitation phase.
* **The Hook**: A Level 2 journalist persona would contact a target to compliment their work, requesting an interview regarding a regional conflict, or sending an exclusive draft article for review.
* **Credential Harvesting**: The link provided to view the interview outline or log into the news portal was a lookalike login page designed to harvest the target's corporate or personal credentials.
* **Malware Execution**: If the target required a download to view the document, the site delivered specialized macro-malware loaders. Security researchers analyzing the malware discovered that the payload fragments were hardcoded to unzip using a specific Persian password: Parastoo (the Farsi word for a swallow bird). This password tied the campaign directly back to early Iranian state-sponsored server attacks.

5. **Discovery and Teardown**: The operation eventually collapsed due to the group's strict adherence to localized working hours. Security analysts tracking anomalous social media activity noticed that all 14 personas, despite claiming to be based in Washington D.C., London, or Cairo, consistently posted updates, responded to messages, and updated the fake news portal precisely aligned with Tehran business hours. Furthermore, researchers traced the underlying domain registration data for NewsOnAir.org back to an IP block registered within Tehran.

By the time iSIGHT Partners coordinated a global takedown with social media platforms in 2014, the group had successfully mapped out significant portions of Western defense networks, laying the behavioral groundwork for the IRGC's modern social engineering playbook.


### The HBO Cyberattack & Extortion (2017)
In the summer of 2017, Home Box Office (HBO) became the target of one of the most high-profile corporate extortion campaigns in entertainment history. The threat actor stole 1.5 terabytes of data, including unreleased episodes of hit shows, confidential corporate documents, and highly protected scripts for Game of Thrones. The U.S. Department of Justice later unsealed a criminal indictment naming the primary perpetrator as Behzad Mesri (operating under the alias Skote Vahshat). Independent threat intelligence firms and state-sponsored tracking analysts subsequently linked Mesri's infrastructure, hacking signature, and historical associations directly to Charming Kitten. The operational mechanics and downstream impact of the breach include the following:
1. **The Compromise: Infrastructure Infiltration**: Rather than launching an advanced software exploit, Mesri focused heavily on targeting credential entry points.
* **Remote Worker Trapping**: Mesri systematically mapped out the remote-access infrastructure utilized by HBO employees. By targeting staff members authorized to log into internal servers from external networks, he successfully harvested legitimate employee user accounts and credentials.
* **Silent Infiltration**: Using these legitimate employee accounts, Mesri authenticated through HBO's remote perimeter barriers. Over a period of approximately three months (May to July 2017), he moved laterally across the entertainment giant's internal databases, staging and compressing files completely undetected.

2. **The Loot: 1.5 Terabytes of Stolen IP**: The payload gathered from the servers struck at the core of HBO's intellectual property and corporate privacy.
* **Unaired Media Assets**: The data haul included full video files of unreleased episodes for multiple flagship series, including Ballers, Barry, Curb Your Enthusiasm, and The Deuce.
* **The Spoiler Leverage**: Crucially, Mesri exfiltrated upcoming plot summaries and written draft scripts for season seven of Game of Thrones, which was airing live at the time.
* **Corporate Exposure**: Beyond entertainment media, the theft captured internal strategic blueprints, executive job offer letters, a spreadsheet detailing sensitive legal claims, and a comprehensive contact database containing the personal phone numbers, home addresses, and emails of the cast and crew (including stars Peter Dinklage, Lena Headey, and Emilia Clarke).

3. **The Extortion & Corporate Taunting**: Once the exfiltration phase wrapped up, the attack shifted into an aggressive, highly publicized psychological extortion phase.
* **The Ultimatum**: In July 2017, Mesri sent an email blast directly to HBO executives and select media programming staff. Opening with the taunting phrase "Hi All losers!", he revealed the breach and demanded a $6 million Bitcoin ransom to halt the release of the data.
* **Pop Culture Threat Framing**: To heighten the pressure, the extortion messages were embedded with images of the Night King (the primary zombie villain from Game of Thrones) accompanied by the altered show catchphrase: "Winter is coming. HBO is falling."
* **Controlled Leaks**: To prove the validity of his threat, Mesri (operating under the media persona Mr. Smith) initiated a slow, weekly leak cadence. He dumped batches of 3.4GB of data online, including cast lists and scripts, deliberately weaponizing entertainment media outlets to pressure HBO into a payout.

4. **The Response & Forensic Fallout**: Faced with a massive PR crisis, HBO launched an internal forensic cleanup led by the FBI and the cybersecurity firm Mandiant.
* **Buying Time**: Leaked negotiation logs later showed that HBO officials attempted to engage the hackers, offering a symbolic $250,000 bug bounty payment. Investigators later confirmed this was a calculated stall tactic engineered by forensic teams to evaluate the scope of the infrastructure compromise and locate the command nodes.
* **The Attacker Profile Link**: The investigation revealed that Behzad Mesri was far from a standard internet script-kiddie. He had a deep profile as an Iranian military hacker who had previously defaced websites under the Turk Black Hat security team and actively conducted network attacks targeting Israeli infrastructure and global nuclear software systems on behalf of the Iranian regime.

While Mesri was officially added to the FBI's Most Wanted list following a federal grand jury indictment in Manhattan, he remains a fugitive within Iranian borders. The incident forced a massive, industry-wide shift in how major Hollywood production studios secure pre-release files, employee credentials, and vendor supply chains.


### The U.S. Presidential Election Operations (2020 & 2024)
Charming Kitten's multi-cycle interference in U.S. presidential politics represents a high-stakes evolution in the group's mission, moving them from passive regional intelligence collection to active, aggressive cyber-enabled influence operations. Tracking firms like Microsoft (operating under the name Mint Sandstorm), Google, and the FBI have documented a persistent blueprint: the group targets campaign staff and political operatives to steal internal data and leak it to media outlets, hoping to manipulate public discourse and sow doubt about election integrity. Their operations across the two primary election cycles broke down into the following phases:
1. **The 2020 Cycle: Broad Scope Scouting**: During the run-up to the 2020 election, Charming Kitten focused on casting a wide net over the campaigns of both political parties, leveraging automated harvesting to map out networks.
* **Mass Campaign Phishing**: Threat intelligence reports confirmed that the group attempted hundreds of distinct spear-phishing attacks against personal and professional email accounts associated with both the Donald J. Trump re-election campaign and the Joe Biden presidential campaign.
* **Targeting the Inner Circle**: Instead of trying to breach highly secured corporate networks directly, the attackers targeted external nodes, such as political consultants, campaign ad agencies, and former administration officials. While many attempts failed, they successfully hijacked secondary accounts to study campaign travel schedules, communication trees, and internal policy briefs.

2. **The 2024 Cycle: Lateral Trusted-Chain Attacks**: In 2024, Charming Kitten abandoned generic mass-phishing in favor of highly targeted, high-touch lateral-chain compromises. Rather than messaging a campaign official out of the blue, they hacked people the target already trusted.
* **The Former Senior Advisor Exploit**: In June 2024, Microsoft's Threat Analysis Center (MTAC) detected that Mint Sandstorm successfully compromised the personal email account of a former senior advisor to a presidential campaign.
* **The High-Ranking Infiltration**: The attackers used that compromised advisor account to send a realistic spear-phishing email directly to a high-ranking official inside the live presidential campaign. Because the email came from a trusted, authentic contact, the link was opened. The malicious link routed the user's traffic through an Iranian-controlled domain to strip session cookies and credentials before redirecting them back to a benign website to avoid alerting the victim.
* **Expanding Scope to Legacy Assets**: On June 13, 2024, the same cluster attempted to break into legacy and archived digital infrastructure belonging to a former presidential candidate. Analysts noted this was an effort to unearth old political dossiers, communications, or strategy frameworks that could be repurposed for blackmail or leaks.

3. **Data Theft, Exfiltration, and the Hack-and-Leak Pipeline**: By late summer 2024, the objective of these operations became clear when stolen documents began circulating among major media outlets.
* **The Politico and Media Leaks**: In August 2024, reporters at Politico, The New York Times, and The Washington Post began receiving anonymous emails containing internal, non-public research dossiers from the Donald J. Trump campaign (including vetting documents for vice-presidential candidate JD Vance).
* **The Intelligence Conundrum**: A joint statement released by CISA, the FBI, and the Office of the Director of National Intelligence (ODNI) officially attributed the theft to Iranian state-sponsored actors. The investigation revealed that the hackers had exfiltrated massive troves of data and were actively attempting to shop the materials to both national media outlets and individuals associated with the opposing political campaign (then the Biden-Harris campaign) to weaponize the data before Election Day.

4. **Downstream Sanctions and Legal Fallout**: The U.S. government responded to this direct interference by deploying defensive blocks and legal indictments to map out the actors behind the screens.
* **DOJ Indictments**: On September 27, 2024, the U.S. Department of Justice unsealed criminal indictments against three IRGC-linked cyber actors involved directly in the campaign hacks.
* **Treasury Sanctions**: Simultaneously, the U.S. Department of the Treasury issued heavy sanctions against the specific front companies and state agents responsible for orchestrating the infrastructure behind Mint Sandstorm.

The 2020 and 2024 campaigns firmly established Charming Kitten as a primary threat vector to Western democratic processes, proving that the group's signature social engineering methods could effectively bypass advanced enterprise security features when directed at the personal accounts of political figures.

---

## Detection & Mitigation
Defending against Charming Kitten requires a comprehensive security posture that balances strict technical controls with aggressive human awareness training. Because the group heavily prioritizes identity-based attacks, such as reverse-proxy phishing and credential harvesting, traditional perimeter defenses like firewalls and basic passwords are often insufficient. Effective detection and mitigation strategies must focus on neutralizing their initial social engineering footprint, securing internet-facing perimeter infrastructure, and implementing robust logging to detect their specific living off the land post-compromise behavior. By combining advanced authentication mechanisms with deep network visibility, organizations can disrupt Charming Kitten's attack lifecycle before they establish a permanent foothold.


### Detection Methods
Because Charming Kitten frequently relies on legitimate administrative credentials and native utilities (living off the land), traditional, signature-based antivirus solutions often fail to flag their activity. Detecting this threat actor requires a multi-layered hunting strategy focused on finding anomalous behaviors across identity platforms, endpoint execution environments, and perimeter applications.
1. **Identity & Authentication Detection (Catching Phishing & Session Theft)**: Since Charming Kitten's primary initial access vector is reverse-proxy phishing (using tools like Evilginx), detection must focus on the abnormalities generated during the authentication process.
* **FIDO2/WebAuthn Telemetry Anomalies**: While standard SMS or push-notification MFA can be bypassed by reverse proxies, FIDO2/WebAuthn hardware tokens natively block them. Hunt for users who suddenly downgrade their authentication method from a hardware token to a less secure factor.
* **Impossibility of Travel & Session Hijacking**: Track sudden geo-velocity anomalies. If a user authenticates from New York and, within 10 minutes, a session token with the exact same cookies is utilized from an IP range associated with an open commercial VPN or an unusual hosting provider, flag the session for hijacking.
* **OAuth Application Consent Monitoring**: Monitor cloud environments (Microsoft 365/Google Workspace) for the creation of new OAuth applications with excessive permissions (like Mail.Read, IMAP.AccessAsUser.All, Directory.ReadWrite.All). Inspect the application developer profile; Charming Kitten frequently names these apps to look like standard productivity tools.
* **User Agent Consistency Audits**: When hunting for tools like HYPERSCRAPE, search email server access logs for active sessions that rapidly toggle between modern browser user agents and obsolete web browser strings (like Internet Explorer 11 or legacy mobile configurations) while executing heavy data downloads.

2. **Endpoint Logging & Process Tracking (Hunting Custom Malware)**: Once on a workstation, Charming Kitten utilizes custom loaders (like PowerLess or POWERSTAR) that manipulate native Windows binaries.
* **PowerShell Native CLR Bypasses**: Because PowerLess runs PowerShell scripts through alternative .NET containers to bypass powershell.exe monitoring, standard command-line auditing won't catch it.
* **Detection**: Configure telemetry rules to trigger alerts when alternative system binaries (like rundll32.exe, regsrv32.exe, or installutil.exe) dynamically load core .NET assembly engines, specifically System.Management.Automation.dll.
* **Script Block Logging (Event ID 4104)**: Ensure Microsoft-Windows-PowerShell/Operational log collection is enabled. Hunt for heavily obfuscated command strings featuring multiple nested Base64 strings, environment variable reassignments, or commands that attempt to disable LSA protection and Windows Defender (Set-MpPreference -DisableRealtimeMonitoring $true).
* **Scheduled Task Creation Disguises**: Create alert rules for any new scheduled task (schtasks.exe /create) initialized out of temporary directories or C:\ProgramData. Focus on tasks that share names with native binaries but execute out of non-standard file paths (such as a file named OneDriveUpdate.exe running from C:\Users\Public).

3. **Perimeter & Server Defenses (Detecting Infiltration & Web Shells)**: For subgroups targeting corporate perimeters via N-day exploits, monitoring the interaction between edge appliances and server operating systems is critical.
* **Anomalous Web Server Child Processes**: In IIS or Apache environments running on Windows/Linux, web servers should rarely spawn administrative shells. Build detection queries that trigger an alert the moment a web application daemon (such as w3wp.exe or tomcat.exe) launches a child instance of cmd.exe, powershell.exe, or bash. This is an almost certain indicator of a web shell injection (such as BellaCiao).
* **Unknown Inbound Proxy Connections**: Track unusual inbound connections to localized network ports. Look for server processes listening on ephemeral ports that start communicating via raw text or unconventional secure protocols, which could indicate an active SSH or custom C++ proxy tunnel.

4. **Network & Infrastructure Hunting (C2 Tracking)**: Detecting the group's network footprint requires watching how endpoints query the internet.
* **GitHub & Webhook.site Baseline Excursions**: Since the group uses GitHub as a Dead Drop Resolver to update its C2 nodes, monitor for corporate workstations (non-developer machines) that establish persistent, low-frequency HTTPS connections to raw GitHub content paths (raw.githubusercontent.com) or outbound text utilities like webhook.site.
* **Protocol Incongruity (IRC Hunting)**: Watch for outbound traffic traveling over ports natively associated with web traffic (like Port 80 or 443) but carrying unencrypted, legacy Internet Relay Chat (IRC) formatting strings (like commands starting with JOIN, NICK, or PRIVMSG). This is a hallmark fallback signature of Charming Kitten's secondary implants.


### Mitigation Methods
Because Charming Kitten favors identity manipulation and perimeter exploitation over sophisticated zero-day exploits, organizations can significantly reduce their attack surface through robust architectural controls. Mitigating this threat actor requires a defense-in-depth approach that addresses identity verification, endpoint hardening, perimeter security, and cloud tenant protection.
1. **Identity & Access Defenses (Neutralizing Social Engineering)**: Since the group's primary weapon is reverse-proxy credential phishing (such as Evilginx), traditional authentication methods are no longer sufficient.
* **Enforce Phishing-Resistant MFA**: Traditional multi-factor authentication (SMS codes, voice calls, and basic push notifications) can be intercepted by reverse proxies. Organizations must transition high-value targets (executives, researchers, and political staff) to FIDO2/WebAuthn hardware security keys or device-bound cryptographic passkeys. These protocols validate the specific browser URL before authorizing the login, instantly neutralizing lookalike phishing domains.
* **Continuous Conditional Access Policies**: Implement risk-based conditional access controls that actively evaluate authentication telemetry. Enforce rules that automatically block or prompt for step-up authentication when a login exhibits impossible travel patterns, originates from an unmanaged device, or utilizes a known commercial VPN or hosting provider IP range.
* **Strict Session Lifetime Management**: Configure short session token expiration windows for webmail and critical cloud interfaces. Implementing Continuous Access Evaluation (CAE) ensures that token revocation occurs immediately if a user's location, risk score, or device compliance status shifts.

2. **Perimeter & Infrastructure Hardening (Blocking Initial Entry)**: To thwart infrastructure-focused subgroups that scan for and exploit public-facing enterprise vulnerabilities (such as Log4Shell or ProxyShell), organizations must harden their internet-facing systems.
* **Aggressive N-Day Patch Management**: Maintain an expedited patch cycle for all internet-accessible assets, with a strict prioritization framework for remote code execution (RCE) flaws in edge infrastructure (including Microsoft Exchange, Ivanti, Confluence, and ConnectWise appliances).
* **Network Segmentation of Edge Appliances**: Ensure that all demilitarized zone (DMZ) servers and public-facing web applications are heavily segmented from the core internal Active Directory network. If a web server is compromised via an N-day vulnerability, strict firewall rules should prevent that server from initiating outbound connections to internal endpoints or domain controllers.
* **Disable Unnecessary Public Protocols**: Audit and completely disable legacy protocols like external IMAP/POP3 access to cloud environments. These older authentication pathways lack robust logging and are frequently targeted by automated password-spraying tools.

3. **Endpoint Security & Post-Exploitation Mitigation**: If an operative successfully establishes a foothold via a malicious payload or stolen credential, endpoint protections must be pre-configured to limit their lateral movement.
* **Enforce Constrained Language Mode (CLM)**: Configure PowerShell to run in Constrained Language Mode via AppLocker or Windows Defender Application Control (WDAC). This restriction strips away advanced PowerShell capabilities, such as direct .NET object manipulation and memory injection, effectively breaking the core execution logic of implants like PowerLess.
* **Credential Guard Activation**: Enable Windows Credential Guard to run the Local Security Authority (LSA) process within a hardware-isolated virtual container. This prevents tools like Mimikatz or procdump from dumping cleartext credentials or NTLM hashes directly from system memory (LSASS).
* **Implement the Principle of Least Privilege**: Strictly limit administrative access across the enterprise. Ensure that standard users lack local administrator rights, and enforce separate, highly restricted accounts for domain administration that are completely barred from logging into standard, internet-connected workstations.

4. **Cloud Tenant & OAuth Security (Preventing Permanent Backdoors)**: To counter Charming Kitten's tactic of establishing permanent cloud persistence via malicious third-party integrations, administrators must actively govern application permissions.
* **Disable User-Led OAuth Consent**: Restrict standard users from independently granting consent to third-party applications requesting access to corporate data. All OAuth application requests must be routed through an Admin Consent Workflow, allowing security teams to manually audit the application's publisher identity and requested permissions (Mail.Read, etc.) before authorization.
* **Automated Cloud Mail-Forwarding Blocks**: Implement strict, tenant-wide exchange transport rules that programmatically block or alert on the creation of automatic mail-forwarding rules destined for external email addresses. This completely neutralizes the group's ability to quietly siphon an inbox after an initial account takeover.
