# Command and Control (C2)

## Description
This is a infrastructure and the method that threat actors use for communication with, and the remote control of compromised systems. This essentially acts as a mechanism for phoning him, and this gives threat actors the ability to issue commands, steal data, and maintain persistence within a network; this frequently blends in with legitimate traffic. 

### Main Uses
The main use for a C2 is to maintain communication with, manage, and remotely control compromised systems after a breach occurs. This essentially is the phone line for things like sending commands, deploying malware, and exfiltrating data while hiding in network traffic. The main uses for this include:
- **Remote Control & Administration**: Threat actors can issue commands to infected systems; this essentially turns the infected device into the threat actor's device. 
- **Data Exfiltration**: Sensitive information can be stolen from victim networks and sent back to the threat actor's server. 
- **Payload Delivery**: Additional malware can be downloaded, such as ransomware or spyware, onto compromised machines. 
- **Maintaining Persistence**: Long-term access within a network can be achieved, which facilitates the threat actor returning to the system post reboot. 
- **Botnet Coordination**: Large networks of zombies (infected machines) can be managed, and directed to launch massive, distributed denial-of-service (DDoS) attacks. 
- **Lateral Movement**: Moving through the network to identify and exploit high-value targets. 

### Common C2 Types & Techniques
The way that threat actors utilize their C2s varies, but involves techniques like HTTP/HTTPS-based, DNS-based, and cloud-based communication. Some key types include centralized servers, decentralized peer-to-peer (P2P) networks, and covert channels that masquerade as legitimate traffic. Common techniques and methods of deploying a C2 include: 
1. **Common C2 Communication Techniques**
- **HTTP/HTTPS-Based C2**: The most common type; this is where the malware receives commands from a server that blends in with standard web traffic. 
- **DNS Tunneling (DNS-based C2)**: Malicious commands get encoded within DNS queries and response; this is frequently capable of bypassing firewalls. 
- **Cloud-Based C2**: Threat actors abuse legitimate services, like Dropbox, Google Drive, or AWS, to send commands; this makes it difficult to block the traffic without impeding on legitimate apps. 
- **Peer-to-Peer C2**: Infected systems (bots/zombies) can talk to each other instead of needing to talk to a central server; this makes it much more difficult to take down the infrastructure. 
- **Domain Generation Algorithms (DGA)**: In an effort to avoid detection, the malware will generate numerous random domain names in order to connect to a few active C2 servers. 
- **Social Media/Instant Messaging**: Platforms like X (Twitter) or Telegram can be used to send commands to malware. 
- **Living-off-the-Land (LotL) C2**: Pre-installed, legitimate system tools like PowerShell or WMI can be used to execute commands. 

2. **Common C2 Frameworks**
- **Cobalt Strike**: This is a widely used, mature tool that's frequently used for deploying ransomware. 
- **Sliver**: This is an open-source, flexible framework that's capable of supporting multiple communication protocols. 
- **Mythic**: This is a well-known framework that supports diverse, customizable agents. 
- **Covenant**: This is a collaborative .NET-based C2 framework that read teamers can use. 

3. **C2 Infrastructure Models**
- **Centralized**: All nodes are controlled by a single command point; this makes it easy to manage but results in a single point of failure. 
- **Decentralized**: Threat actors can change IP addresses (Fast Flux) or use P2P to obfuscate their command server. 

---

## Technical Explanation 
This is the operational heart of a cyberattack; allowing threat actors to maintain long-term access, issue commands, and exfiltrate data after a breach. A C2 primarily functions in three main components: The C2 server (the hub), the C2 client (threat actor's interface), and the C2 agent (the malware or implant that's on the victim machine). 
1. **The C2 Agent**: This is a piece of malware that runs in victim's machine. The main function is to act as an ear in the network for the threat actor. 
- **Execution & Persistence**: In an effort to avoid leaving files on the disk, the gent will run in memory (usually through Process Injection) once the initial exploit (usually phishing) has been run. After this, the malware will typically attempt to achieve persistence (such as registry keys or scheduled tasks) to survive, even after rebooting. 
- **The Check-in (Beaconing)**: The agent won't just maintain a connection to the server to help avoid detection. Rather, it'll beacon to the server; usually sleeping for a certain amount of time (60 seconds), before waking up and sending a small encrypted heartbeat to the C2 while asking if there's any tasks, before going back to sleep. 
- **Execution of Tasks**: Once given a task by the server (such as taking a screenshot or running a shell command), the agent will download the task, execute it through system APIs (`CreateProcess` or `VirtualAlloc`), and then upload the results during the next check-in. 

2. **The C2 Server**: This is the threat actor's infrastructure; this usually consists of a Linux VPS (Virtual Private Server) that remains hidden in layers of proxies called Redirectors. 
- **Listener Management**: The server will run listeners; these are services that are configured to wait for incoming traffic on specific ports and protocols (HTTP/80, HTTPS/443, DNS/53). 
- **Data Aggregation**: The server will end up receiving raw, encrypted data from potentially hundreds or thousands of different agents. It must be able to decrypt the traffic, identify which victim it came from, and store the results within the database. 
- **The Buffer**: Through this method, the threat actor doesn't send commands directly to the victims. Instead, the they can save the command on the server, and the server will maintain that command until the next time the specific agent checks in. 

3. **The C2 Client**: This is the software that the threat actor (the operator) uses on their laptop to communicate with the C2 server. 
- **Command Input**: The Operator can use the Client to browse the file system of the victim machine, interact with the shell, or deploy additional modules (such as Mimikatz for credential stealing). 
- **Multi-User Collaboration**: Modern frameworks (such as Cobalt Strike or Sliver) can facilitate more than one operator connecting to a single C2 server. The Client ensures that Operator A doesn't interfere with Operator B's session on a different victim. 
- **Visualizing the Network**: Often, the Client will provide a graph view that shows how the threat actor has pivoted from the first entry machine to other machines within the network. 

### The Grand Scheme
To imagine how all of this works together, we can imagine a threat actor attempting to steal a password. 
1. **Operator (Client)**: Types `logonpasswords` into the Client interface and hits enter. The Client then sends this request to the Server. 
2. **Server**: Saves the command in a Task Queue for Victim-A.
3. **Agent (Victim)**: Wakes up from its sleep and sends a beacon to the Server. 
4. **Server**: Sees the beacon and responds "Yes, here's the task: run the logonpasswords module."
5. **Agent (Victim)**: Receives the module, injects it into a system process, grabs the password from memory, and encrypts the results. 
6. **Agent (Victim)**: In the next beacon, if not immediately after, it'll upload the encrypted password data to the Server. 
7. **Server**: Decrypts the data and notifies the Client.
8. **Operator (Client)**: Sees the passwords pop up on their screen in a nice, readable format. 

---

## Mappings 
### MITRE ATT&CK
Within the MITRE ATT&CK Framework, there are 16 distinct Command and Control techniques that are defined, each with their own sub techniques. There are hundreds of different software tools that exist for this purpose, however, they all use one or more of these 16 fundamental methods for communication. 
| Technique ID | Technique Name | Technique Description |
| :--- | :--- | :--- | 
| T1071 | Application Layer Protocol | Standard web traffic, such as HTTP, HTTPS, or SMB, is used as a way of blending in | 
| T1092 | Communication Through Removable Media | Commands can be transferred through physical media, such as a USB drive, to jump air-gapped networks | 
| T1132 | Data Encoding | Malicious commands can be made to look like gibberish through encoding, like Base64 or Gzip |
| T1001 | Data Obfuscation | Steganography can be utilized to get through deep packet inspection |
| T1568 | Dynamic Resolution | Techniques, like Fast Flux for rotating IP addresses, can be used to avoid blacklisting |
| T1573 | Encrypted Channel | Delaying defenses through the use of custom or standard encryption, making it non-human-readable |
| T1008 | Fallback Channels | Being prepared for the chance of the main connection getting blocked and having fallback channels (switching to DNS from HTTP for example) |
| T1105 | Ingress Tool Transfer | This is the act of moving malicious tools from the server onto the victim machine |
| T1104 | Multi-Stage Channels | Using a small agent as the first stage for downloading much larger and stronger second stage payloads |
| T1095 | Non-Application Layer Protocol | Utilizing low-level protocols like ICMP (pings) or UDP that's not as deeply inspected by firewalls |
| T1571 | Non-Standard Port | Utilizing random ports, like 8088 for HTTPS, for web traffic to get through basic port-based blocking |
| T1572 | Protocol Tunneling | Hiding a protocol within another (putting SSH traffic within a DNS request |
| T1090 | Proxy | Having traffic travel through a series of intermediary servers to hide the final destination |
| T1219 | Remote Access Software | Abusing tools like TeamViewer or AnyDesk that are legitimate for malicious control |
| T1205 | Traffic Signaling | Using Port Knocking to wake up dormant agents by hitting specific ports in a specific order |
| T1102 | Web Service | Using public sites like Google Drive, Discord, or Twitter to control malware |

### Other Mappings
We can use control mappings as a way to translate the technical C2 behavior into high-level compliance requirements. We can use these mappings to connect the technical tactics (the MITRE mapping) to the governance requirements of frameworks like ISO/IEC 27001, NIST 800-53, and more. 

#### **ISO 27001:2022 to C2 Traffic Mappings**
ISO 27001 provides the what (the goal), while C2 traffic analysis provides the how (technical proof). Some common C2-related control mappings from ISO/IEC 27001 include: 
- **A.8.16 (Monitoring Activities)**: This maps directly to the detection of C2 beaconings and anomalous traffic patterns. 
- **A.8.12 (Data Leakage Prevention)**: Relates to C2 exfiltration techniques where data is tunneled out via protocols like DNS or HTTP. 
- **A.8.20 (Network Security)**: This maps to the architectural defenses against C2; including the use of firewalls and proxies to block known malicious C2 domains. 

#### **NIST 800-53 (Crosswalk)**
There's a comprehensive set of mappings between MITRE ATT&CK and NIST 80-53 that's provided by the Center for Threat-Informed Defense. 
- Due to the fact that NIST 800-53 already maps to ISO 27001, a crosswalk can be made by organizations to map which controls are satisfied by their C2 detection tools. 
  - **Example**: Implementing a DNS query analysis to catch Domain Generation Algorithms (DGAs) satisfies aspects in both NIST SC-7 (Boundary Protection) and ISO A.8.20. 

#### **CIS Controls**
The CIS Controls are generally preferred for C2 mapping because of the fact that they're more actionable than ISO standards. 
- **CIS Control 13 (Network Monitoring and Defense)**: This specifically maps to detecting C2 traffic through tools, such as Intrusion Detection Systems (IDS) and netflow analysis. 
- **CIS Control 9 (Email and Web Browser Protections**: This focuses on blocking the initial delivery of C2 agents. 
- *CIS Controls Navigator**: Using the CIS Controls Navigator to see real-time mappings between technical safeguards and compliance frameworks, like PCI DSS, HIPAA, and ISO 27001. 

#### **Bonus: Specialized Industry Mappings**
- **Automotive (ISO 21434)**: This maps C2 risks to vehicle-to-everything (V2X) communication, focusing on safety-critical system updates. 
- **Industrial/OT (IEC 62443)**: This uses a Zone and Conduit model to help map how C2 traffic might move between business networks and sensitive factory floors. 

---

## Detection & Mitigation 
In order to detect and mitigate/harden your systems against C2 traffic requires efficient defense-in-depth, and a strategy that focuses on the identification of behavioral patterns instead of static signatures. 

### Detecting C2s
**This is a work in progress, more will be added soon**


