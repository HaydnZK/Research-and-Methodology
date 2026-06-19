# Network Hardening & Secure Architecture

---

## Switch Hardening
### Port Security: Limiting physical access
Switch port security stops unauthorized devices from gaining network access by restricting what can connect to a specific port based on MAC addresses. By setting these up, you prevent rogue devices like unauthorized laptops or packet sniffers from getting on your network, even if someone manages to bypass physical building security.

#### Core Steps for Switch Port Security Hardening
To build a solid port-level defense, you've got to handle both active and inactive ports while setting clear, automated rules for what happens during a violation.

1. **Blackhole Inactive Ports**
Never leave unused switch ports open or connected to production networks. If someone plugs a device into a random wall jack, it shouldn't go anywhere.
* **Administrative Shutdown**: Manually disable every single unused switch port.
* **Isolate to a Blackhole VLAN**: Assign unused ports to a dead, non-routed VLAN that has no path to corporate resources or the internet.


2. **Define Port Security Parameters**
On ports where you expect devices like workstations, printers, or IP phones, you need to enforce limits on what hardware is allowed to talk to the switch.
* **Limit MAC Address Counts**: Restrict the max number of MAC addresses allowed on a port. Usually, you'll set this to one for a standard workstation, or maybe two or three if an IP phone is sharing the connection.
* **Implement Sticky MAC Addresses**: Configure the port to automatically learn the MAC address of the device plugged in and save it directly into the running configuration. This keeps you from having to manually type in every MAC address.


3. **Establish Strict Violation Modes**
When the switch catches an unauthorized MAC or someone tries to plug in an extra device, it needs to take action.

| Violation Mode | Drops Traffic? | Logs Alert? | Shuts Down Port? | Best Use Case |
| --- | --- | --- | --- | --- |
| Protect | Yes | No | No | Low-severity zones where you want silent packet drops without alerting. |
| Restrict | Yes | Yes | No | High-traffic zones where you need alerts but want to avoid accidental downtime. |
| Shutdown | Yes | Yes | Yes | The enterprise standard. Shuts the port down completely until you manually reset it. |

#### Command Blueprint: Cisco IOS Port Security
Here is the baseline configuration for securing an active user port and isolating an unused one.
* **Securing an Active User Port (Workstation)**

```text
interface GigabitEthernet0/1
  description Secure_User_Workstation
  switchport mode access
  switchport port-security
  switchport port-security maximum 1
  switchport port-security mac-address sticky
  switchport port-security violation shutdown

```

* **Neutralizing an Unused Port**

```text
interface GigabitEthernet0/2
  description Disabled_Unused_Port
  switchport mode access
  switchport access vlan 999
  shutdown

```

#### **Advanced Layers: Moving Beyond MAC Filters**
While MAC-based port security stops casual physical intruders, sophisticated attackers can clone or spoof authorized MAC addresses. For enterprise-grade hardening, you need to stack layers of automated cryptographic authentication:
* **Implement 802.1X Network Access Control (NAC)**: Move past static MAC rules by requiring every device to dynamically authenticate using digital certificates or user credentials before the port opens. Solutions like Cisco Identity Services Engine (ISE) or open-source FreeRADIUS orchestrate this globally.
* **Deploy DHCP Snooping**: Configure the switch to track legitimate DHCP assignments. This creates a binding database that blocks devices trying to manually configure static spoofed IP addresses to bypass controls.
* **Enable Dynamic ARP Inspection (DAI)**: Pair DAI with DHCP snooping to intercept and drop malicious Address Resolution Protocol (ARP) packets, which effectively stops local physical man-in-the-middle and eavesdropping attempts.

#### **Hardening Goal Reached**
By disabling unused connections and enforcing sticky MAC rules with an automatic shutdown violation, any unauthorized physical device insertion is neutralized at the hardware layer before a single packet can traverse the corporate network.


### VLAN Segmentation: Isolating network traffic
VLAN segmentation isolates network traffic at Layer 2 by breaking a single physical broadcast domain into multiple distinct logical networks. Without VLANs, every device on a switch can talk to any other device, and broadcast traffic like ARP requests floods the entire network. This lack of boundaries makes it easy for attackers to sniff traffic, map assets, and move laterally through your infrastructure.

When you implement VLAN segmentation, you ensure that devices in different zones can't talk to each other unless they pass through a Layer 3 filter, like a firewall or a secure router.

#### **Core Principles of Secure VLAN Design**
To keep the network's blast radius small, your architecture needs to follow strict boundary rules.
* **Segment by Function, Not Location**: Group devices by their roles, trust levels, and security needs. Don't just lump devices together because they are in the same room.
* **Enforce Inter-VLAN Firewalled Routing**: Restrict traffic moving between different VLANs by routing it through a Next-Generation Firewall (NGFW) or a router using strict Access Control Lists (ACLs).
* **Minimize the Native VLAN Attack Surface**: Change the default Native VLAN (usually VLAN 1) to an unused, isolated VLAN ID across all trunk links to stop VLAN hopping attacks.

#### **Standard Enterprise VLAN Segmentation Schema**
A resilient architecture breaks networks into isolated blocks. Here is a standard blueprint:

| VLAN ID | Segment Name | Trust Level | Security Controls & Isolation Rules |
| --- | --- | --- | --- |
| 10 | Management | High | Network admins only. No internet access. Only VLAN allowed to access switch/firewall management interfaces. |
| 20 | Corporate Data | Medium | Employee workstations. Full internet access, but blocked from management and data center access. |
| 30 | Production Servers | High | Corporate apps and databases. Heavily protected by stateful firewall rules; no direct internet browsing. |
| 40 | IoT / Smart Devices | Low | Cameras and thermostats. Internet-only access. Blocked from all other internal VLANs. |
| 50 | Guest Network | None | Public Wi-Fi. Completely isolated from everything else using client isolation. |
| 999 | Native / Blackhole | None | Unused, non-routed VLAN. Used for trunk link defaults and disabling unused ports. No traffic permitted. |

#### **Cisco IOS Configuration Blueprint**
Here is the CLI configuration for defining secure VLANs, assigning access ports, and hardening trunk links.
1. **Defining Secure VLANs on the Switch**

```text
vlan 20
  name Corporate_Workstations
vlan 40
  name IoT_Peripherals
vlan 999
  name Native_Trunk_Blackhole

```

2. **Configuring an Edge Access Port for a Workstation**

```text
interface GigabitEthernet0/1
  description Employee_PC_Port
  switchport mode access
  switchport access vlan 20
  spanning-tree portfast
  spanning-tree bpduguard enable

```

3. **Hardening a Trunk Link between Switch and Firewall**

```text
interface GigabitEthernet0/24
  description Core_Trunk_To_Firewall
  switchport mode trunk
  switchport trunk allowed vlan 10,20,30,40
  switchport trunk native vlan 999

```

#### **Mitigating VLAN-Specific Exploits**
Attackers use layer-2 manipulation to bypass boundaries, so you have to harden against these methods:
* **Prevent VLAN Hopping via Dynamic Trunking Protocol (DTP)**: Attackers can spoof DTP messages to trick a switch into forming a trunk link. To fix this, always disable auto-trunk negotiation by running `switchport nonegotiate` or setting ports to `switchport mode access`.
* **Defend Against Double-Tagging Attacks**: An attacker can craft a packet with two nested VLAN tags. If the first tag matches the Native VLAN, the switch strips it and passes the packet into the victim's VLAN. To fix this, ensure the Native VLAN ID is completely separate from any production VLANs and force the switch to tag all native traffic using the `vlan dot1q tag native` global command.

#### **Hardening Goal Reached**
By isolating network functional zones into dedicated VLANs, disabling dynamic trunking, and forcing all inter-zone communication through a centralized firewall, lateral movement is stopped at Layer 2. An infection on an IoT device stays confined to its segment and can't reach your critical production databases.


### ARP Inspection: Mitigating spoofing attacks
Dynamic ARP Inspection (DAI) is a hardware-level security feature that neutralizes ARP spoofing and poisoning attacks. ARP normally maps a Layer 3 IP address to a Layer 2 MAC address, but it lacks built-in authentication, meaning any device can broadcast an unprompted ARP reply.

Attackers exploit this by sending fraudulent messages to associate their MAC address with a critical asset's IP, like the default gateway. This is how they launch Man-in-the-Middle (MitM) and eavesdropping attacks. DAI intercepts every ARP packet and validates it against a trusted database before letting it through.

#### **How Dynamic ARP Inspection Operates**
DAI relies on a database to verify IP-to-MAC bindings. If an incoming packet doesn't match this index, the switch drops it and logs a violation.
1. **The DHCP Snooping Binding Database**: DAI uses a live database built by DHCP Snooping. When a device gets an IP from a legitimate DHCP server, the switch records the IP, MAC, lease time, and port. DAI references this to police the network.
2. **Static ARP ACLs for Fixed Assets**: For servers or gateways with static IPs, you create manual ARP Access Control Lists. DAI checks these to avoid false positives.
3. **Trusted vs. Untrusted Ports**:
* **Untrusted Ports**: Standard edge ports like wall jacks and Wi-Fi access points. Every ARP packet here is inspected.
* **Trusted Ports**: Uplinks to other switches, routers, or firewalls. Traffic here bypasses filtering to save CPU cycles.

#### **Cisco IOS Configuration Blueprint**
Here is the setup for DHCP snooping and DAI on a production VLAN.
1. **Enable DHCP Snooping**

```text
ip dhcp snooping
ip dhcp snooping vlan 20
interface GigabitEthernet0/24
  description Core_Uplink_To_DHCP_Server
  ip dhcp snooping trust

```

2. **Configure Static ARP Bindings**

```text
arp access-list Static_Server_Bindings
  permit ip 192.168.20.10 mac 00aa.bbcc.ddee

```

3. **Activate DAI**

```text
ip arp inspection vlan 20
ip arp inspection filter Static_Server_Bindings vlan 20
interface GigabitEthernet0/24
  description Core_Uplink_To_Trusted_Router
  ip arp inspection trust

```

4. **Protect Against DoS**

```text
interface GigabitEthernet0/1
  description Employee_PC_Port
  ip arp inspection limit rate 15

```

#### **Advanced Validation Parameters**
For higher security, you can use `ip arp inspection validate` to perform deep-packet inspection:

* **src-mac**: Compares the MAC inside the ARP payload against the hardware MAC in the Ethernet header. If they don't match, it's a forgery.
* **dst-mac**: Checks the destination MAC against the target hardware MAC to stop malformed packets.
* **ip**: Scans for illegal or impossible IPs (like 0.0.0.0 or multicast ranges) and drops them.

#### **Hardening Goal Reached**
By tying DAI to your DHCP snooping database, you completely eradicate local network spoofing. If an attacker tries to impersonate a default gateway, the switch drops the rogue frame instantly, logs the incident, and keeps the rest of the VLAN clean.

---

## Router Hardening
### Edge Protection: Shielding the perimeter
Edge router hardening secures your network perimeter by ensuring the gateway drops malicious packets, ignores unsolicited probes, and protects its own control plane from denial-of-service (DoS) attacks. As your first line of defense, a hardened perimeter router stops attacks at the boundary before they ever reach your firewalls, intrusion prevention systems, or endpoints.

#### **Core Pillars of Edge Router Protection**
Securing the edge requires a mix of strict access policies, packet authenticity validation, and control plane shielding.
* **Infrastructure Access Control Lists (iACLs)**: These filters block unauthorized traffic destined directly for the router itself—like management IPs, routing protocols, and control services—while still letting legitimate transit traffic pass through.
* **Source IP Validation (Anti-Spoofing)**: Perimeter routers must drop incoming packets that claim to originate from internal network addresses, preventing attackers from bypassing perimeter controls through IP spoofing.
* **Control Plane Policing (CoPP)**: This stops the router's processor from being overwhelmed by flooding attacks (like ICMP floods) by rate-limiting traffic destined for the CPU.

#### **Perimeter Hardening Configuration Blueprint**
Here is the production-ready Cisco IOS blueprint for shielding an enterprise-facing perimeter router.
1. **Implement Unicast Reverse Path Forwarding (uRPF)**: uRPF forces the router to check the source IP of any incoming packet on the untrusted interface. If the router doesn't have a route back to that source IP via the same interface it arrived on, it assumes the packet is spoofed and drops it.

```text
interface GigabitEthernet0/0
  description UNTRUSTED_INTERNET_EDGE
  ip verify unicast source reachable-via rx

```

2. **Deploy Anti-Spoofing and Bogon Filtering via iACL**: Your router must explicitly drop packets from impossible, reserved, or internal-only source IP addresses coming from the public internet.

```text
ip access-list extended PERIMETER_INBOUND
  ! Block your own internal IP space
  deny ip 10.0.0.0 0.255.255.255 any
  deny ip 172.16.0.0 0.15.255.255 any
  deny ip 192.168.0.0 0.0.255.255 any
  ! Block Bogon IP space
  deny ip 127.0.0.0 0.255.255.255 any
  deny ip 169.254.0.0 0.0.255.255 any
  ! Permit legitimate transit traffic to internal resources
  permit tcp any host 203.0.113.50 eq 443
  deny ip any any log

interface GigabitEthernet0/0
  ip access-group PERIMETER_INBOUND in

```

3. **Disable Unnecessary Network Services**: Perimeter routers often run legacy protocols that leak info or present attack vectors.

```text
no ip http server
no ip http secure-server
no ip source-route
no ip directed-broadcast
no ip proxy-arp
no cdp run

```

4. **Configure Control Plane Policing (CoPP)**: CoPP separates transit traffic from CPU-bound management traffic, placing speed traps on the latter.

```text
ip access-list extended ICMP_TRAFFIC
  permit icmp any any

class-map match-all COPP_ICMP_CLASS
  match access-group name ICMP_TRAFFIC

policy-map COPP_SYSTEM_POLICY
  class COPP_ICMP_CLASS
    police 100000 conform-action transmit exceed-action drop

control-plane
  service-policy input COPP_SYSTEM_POLICY

```

#### **Edge Attack Vectors and Mitigation Summary**

| Perimeter Threat | Exploit Vector | Hardening Mitigation |
| --- | --- | --- |
| IP Spoofing | Attacker masks identity using internal IP ranges. | uRPF (Strict Mode) and Inbound Anti-Spoofing iACLs. |
| Reconnaissance | Mapping the network via ping sweeps. | Disabling ICMP Redirects and rate-limiting ICMP via CoPP. |
| Bogon Routing | Packets from unallocated global IP spaces. | Bogon ACL filtering updated via automated BGP feeds. |
| CPU Exhaustion | Flooding the router with management requests. | Control Plane Policing and Out-of-Band Management. |

#### **Hardening Goal Reached**
By implementing uRPF checking, dropping bogon and spoofed internal spaces, and configuring control plane policing, your edge router acts as a literal armored shield. It absorbs and drops malicious protocol manipulations, scans, and volumetric floods, ensuring only clean transit traffic ever hits your internal firewalls.


### Protocol Security: Securing routing communications
Routing protocol security hardens the control plane by enforcing cryptographic authentication and structural updates, which stops attackers from hijacking traffic or injecting fraudulent routes. If routing communications aren't authenticated, an attacker who gets on a local segment can announce a fake, high-priority path to the core routers. This lets them redirect confidential company data through a malicious machine, creating an immediate, undetectable Man-in-the-Middle (MitM) exploit or causing a complete network blackhole.

#### **Core Security Patterns for Routing Protocols**
Securing routing boundaries relies on three main defenses: authentication, topology boundary containment, and geometric path control.
* **Cryptographic Peer Authentication**: Every router must prove its identity to its neighbors before they exchange routing updates. Cleartext authentication doesn't cut it anymore. Modern enterprise protocols need to use HMAC-SHA-256 or Keychain-driven MD5 hashes.
* **Passive-Interface Enforcements**: Routing updates should never go out of edge interfaces where only user devices sit, like employee VLANs or guest Wi-Fi. Enabling passive interfaces silences updates on edge ports while keeping the protocol active on core router-to-router links.
* **Maximum Prefix Limits**: Routers should be configured with thresholds for the number of routes they'll accept from an external peer like an ISP. If the peer accidentally or maliciously announces too many routes, the connection drops to protect internal memory.

#### **Multi-Protocol Hardening Blueprint (Cisco IOS-XE)**
The following configurations are the blueprints for securing the most common corporate routing protocols: OSPFv3, EIGRP, and BGP.
1. **Hardening OSPFv3 (Open Shortest Path First)**: Modern OSPF uses IPsec framing directly inside the protocol configuration to encrypt and authenticate all link-state advertisements (LSAs) natively.

```text
router ospfv3 1
  address-family ipv4 unicast
    authentication ipsec spi 1000 sha1 0123456789abcdef0123456789abcdef01234567
    passive-interface GigabitEthernet0/1

```

2. **Hardening EIGRP (Enhanced Interior Gateway Routing Protocol)**: EIGRP Named Mode allows for modern, robust HMAC-SHA-256 authentication directly within the address family layout.

```text
router eigrp SECURE_FABRIC
  address-family ipv4 unicast autonomous-system 100
    af-interface default
      passive-interface
    af-interface GigabitEthernet0/0
      no passive-interface
      authentication mode hmac-sha-256 SecretRoutingPassword123!

```

3. **Hardening Border Gateway Protocol (BGP)**: BGP forms connections across the public internet or external WANs. It has to be protected via TCP MD5 signatures, TTL security tracking, and prefix bounding.

```text
router bgp 65001
  neighbor 203.0.113.5 remote-as 65002
  neighbor 203.0.113.5 password StrictBGPPeerPassword99$
  neighbor 203.0.113.5 maximum-prefix 10000 80
  neighbor 203.0.113.5 ttl-security hops 1

```

#### **Protocol Threat and Defense Matrix**

| Attack Vector | Exploit Mechanism | Hardening Countermeasure |
| --- | --- | --- |
| Route Injection | Rogue device advertises a better path to internal networks. | Cryptographic Authentication (HMAC-SHA-256/MD5). |
| Reconnaissance | Sniffing routing updates to map company topology. | Passive-Interface configurations on all user networks. |
| BGP Route Leaks | ISPs accidentally flooding a router with millions of paths. | Maximum Prefix Limits and outbound route map filtering. |
| Remote Peer Spoofing | Attacker injects a fake BGP packet from across the globe. | BGP TTL Security (GTSM) to validate physical hop counts. |

* **Advanced Layer: Securing Internet-Scale BGP (RPKI)**: If you're managing an internet-facing edge router, you've got to implement Resource Public Key Infrastructure (RPKI). RPKI uses cryptographic certificates to bind an IP address block to an authorized Autonomous System Number (ASN). By turning on RPKI validation on your BGP edge, your router automatically cross-checks routes against a global ledger and discards invalid or hijacked paths before they disrupt your traffic.

#### **Hardening Goal Reached**
By forcing cryptographic peer verification, disabling routing advertisements on all user access domains, and configuring strict prefix metrics, you've structurally isolated the network control plane. Rogue route injection and eavesdropping via protocol manipulation are neutralized at the architectural boundary.


### ACL Implementation: Filtering traffic flow
Access Control Lists (ACLs) are the foundational firewalling mechanism in routers, regulating traffic flow by matching packets against strict security criteria. Operating at Layer 3 (IP addresses) and Layer 4 (protocols and port numbers), ACLs dictate what traffic is permitted to traverse network boundaries. Without properly structured ACLs, routers forward all traffic unconditionally, which forces downstream devices to handle the entire security burden and allows unhindered lateral movement across your network.

#### **Core Principles of Secure ACL Design**
To minimize gaps and maximize router performance, your ACLs need to follow these rules:
* **Enforce an Implicit Deny**: Routers evaluate ACL statements from the top down. If a packet doesn't match an explicit rule, it hits the unwritten "deny ip any any" statement at the very end and gets dropped. Design your rules by explicitly permitting trusted traffic and letting the implicit deny catch the rest.
* **Apply Filters Closest to the Source**: Standard ACLs (source IP only) go as close to the destination as possible. Extended ACLs (source, destination, protocol, and port) should go as close to the source of the traffic as possible to save network bandwidth and CPU cycles.
* **Filter in Both Directions (Inbound vs. Outbound)**: Traffic is bidirectional. An inbound ACL filters packets entering the router, while an outbound ACL filters packets exiting toward a target. Each direction requires its own, specific logic.

#### **Standard Enterprise ACL Architecture Matrix**
A secure internal architecture maps access parameters between your segments. Here is the flow logic for a standard enterprise network:

| Source Segment | Destination Segment | Permitted Protocols & Ports | Traffic Direction | Security Objective |
| --- | --- | --- | --- | --- |
| Corporate Workstations | Production Servers | TCP 443 (HTTPS), TCP 22 (SSH) | Inbound on Workstation Interface | Restrict access to verified management and app ports only. |
| Production Servers | Corporate Workstations | TCP Established (Return Traffic) | Outbound on Workstation Interface | Allow servers to reply, but block them from initiating new connections. |
| IoT / Peripherals | Any Internal Segment | None (Drop Everything) | Inbound on IoT Interface | Complete local isolation; prevent smart device scanning. |
| Any Segment | Router Infrastructure | UDP 123 (NTP), UDP 514 (Syslog) | Inbound on Local Interface | Limit interactions to core sync and telemetry services. |

#### **Production Configuration Blueprint (Cisco IOS-XE)**
Here is how to configure and apply an Extended Named ACL to isolate an employee workstation segment while allowing controlled server access.
1. **Constructing the Named Extended ACL**

```text
ip access-list extended SECURE_WORKSTATION_FLOW
  remark --- Allow Essential Infrastructure Services ---
  permit udp 10.20.0.0 0.0.255.255 host 10.10.0.5 eq domain
  permit udp 10.20.0.0 0.0.255.255 host 10.10.0.6 eq ntp

  remark --- Allow Controlled Access to Production Servers ---
  permit tcp 10.20.0.0 0.0.255.255 host 10.30.0.100 eq 443
  permit tcp 10.20.0.0 0.0.255.255 host 10.30.0.200 eq 22

  remark --- Allow General Internet Access ---
  deny ip 10.20.0.0 0.0.255.255 10.0.0.0 0.255.255.255
  permit ip 10.20.0.0 0.0.255.255 any

```

2. **Applying the ACL to the Target Interface**

```text
interface GigabitEthernet0/1
  description Employee_Workstation_Gateway
  ip address 10.20.0.1 255.255.0.0
  ip access-group SECURE_WORKSTATION_FLOW in

```

#### **Optimizing and Verifying ACL Performance**
Poorly managed ACLs can degrade hardware performance, so use these baselines:
* **Sequence Rules by Hit Density**: Put the most frequently matched rules at the top. This cuts down on the sequential processing load for the router's hardware.
* **Enforce Established Packet Filtering**: Use the `established` keyword for TCP rules on return traffic. This checks for ACK or RST bits, ensuring external entities can't initiate new, unauthorized inbound connections.
* **Utilize Object Groups for Scalability**: Instead of writing separate lines for every server IP, group them logically into objects. This shortens the config and makes future policy changes much easier.

```text
object-group network PRODUCTION_SERVERS
  host 10.30.0.100
  host 10.30.0.200

ip access-list extended OPTIMIZED_FLOW
  permit tcp 10.20.0.0 0.0.255.255 object-group PRODUCTION_SERVERS eq 443

```

#### **Hardening Goal Reached**
By implementing strict Extended ACLs at your gateway, enforcing an implicit deny pattern, and restricting lateral inter-VLAN traversal, you've completely regimented traffic flow across your network. Unauthorized communication paths get dropped at the first routing hop, keeping potential security breaches confined to their local subnet.

---

## Secure Management Plane
### Secure Access Protocols: Implementing SSH and HTTPS
Securing the management plane guarantees that administrative access to network infrastructure is encrypted, authenticated, and shielded from eavesdropping or unauthorized modifications. Legacy protocols like Telnet and HTTP transmit credentials in cleartext, leaving sessions vulnerable to credential harvesting and hijacking. Replacing these with Secure Shell version 2 (SSHv2) and Hypertext Transfer Protocol Secure (HTTPS/TLS) ensures that all management traffic is cryptographically locked down.

#### **Core Security Standards for Management Isolation**
Transitioning to secure protocols requires enforcing strong cryptographic baselines and strict network-level isolation.
* **Enforce Strong Cryptographic Ciphers**: Disable legacy encryption like 3DES, RC4, MD5, and SHA-1 within your SSH and HTTPS daemons. Force the use of secure alternatives like AES-GCM and SHA-256.
* **Generate High-Entropy Encryption Keys**: Utilize RSA keys with a minimum length of 3072 bits or use modern Elliptic Curve Cryptography (ECC) keys to protect session key generation.
* **Restrict Access via Management ACLs (vty ACLs)**: Never expose management interfaces to general user segments. Use dedicated ACLs to ensure only authorized IPs, such as a secure management VLAN or an administrative bastion host, can attempt to connect.

#### **Production Configuration Blueprint (Cisco IOS-XE)**
This blueprint covers generating encryption keys, mandating SSHv2, securing the web server, and isolating management lines.
1. **Global Management Identity and Key Generation**

```text
hostname Core-Switch-01
ip domain-name enterprise.local
crypto key generate rsa general-keys modulus 3072

```

2. **Hardening Secure Shell (SSHv2)**

```text
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 3

ip ssh server algorithm encryption aes256-gcm aes256-ctr
ip ssh server algorithm mac hmac-sha2-256

```

3. **Hardening Web Management (HTTPS/TLS)**

```text
no ip http server
ip http secure-server
ip http secure-tls-version tls1.2

```

4. **Restricting Administrative Lines (vty Lines)**

```text
ip access-list extended MANAGEMENT_ACCESS_ONLY
  permit ip host 10.10.10.50 any
  permit ip 10.10.5.0 0.0.0.255 any
  deny ip any any log

line vty 0 15
  access-class MANAGEMENT_ACCESS_ONLY in
  transport input ssh
  exec-timeout 10 0

```

#### **Protocol Migration and Controls Summary**

| Operational Control | Insecure Legacy State | Hardened Secure State | Security Benefit |
| --- | --- | --- | --- |
| CLI Administration | Telnet (TCP 23) | SSHv2 (TCP 22) | Prevents cleartext sniffing and session injection. |
| Web Administration | HTTP (TCP 80) | HTTPS (TCP 443) | Encrypts web-based management dashboards. |
| Session Cryptography | Default / Low-bit keys | 3072-bit RSA / TLS 1.2+ | Protects traffic from decryption techniques. |
| Line Access Rules | Permissive (Any IP) | Strict vty Access ACLs | Drops brute-force attempts at the network layer. |

#### **Hardening Goal Reached**
By disabling Telnet and HTTP, generating high-entropy keys, mandating modern TLS/SSH versions, and enforcing strict vty access, your management plane is heavily shielded. Unauthorized segments can't reach administrative interfaces, and all configurations occur over tamper-resistant, fully encrypted streams.


### Out-of-Band Management: Isolating administrative traffic
Out-of-Band (OOB) management physically isolates administrative traffic by routing all device management through a network that's completely independent of your production data. In a standard In-Band setup, management traffic like SSH, HTTPS, and SNMP shares the same cables, switches, and paths as user traffic. If the network suffers an outage, a malware outbreak, or a massive DDoS attack, your In-Band access is usually the first thing to go, leaving you unable to log in and fix the problem.

OOB management builds an operational safety net, ensuring you always have a clear, secure path to remediate your hardware regardless of what's happening on the production side.

#### **Core Pillars of a Secure OOB Architecture**
Building a real OOB fabric requires total separation across physical, logical, and access planes.
* **Physical and Logical Air-Gapping**: Use physically separate switches, cabling, and dedicated edge ports like the MgmtEth0 interface on a router. This hardware should never share a backplane, trunk link, or routing table with production traffic.
* **Enforce VRF Isolation**: When physical separation is too difficult, assign management ports to a dedicated Management VRF. This acts as a logical router inside the chassis, maintaining its own isolated routing table that can't talk to the production table.
* **Centralized Consoles and Terminal Servers**: If a device loses its IP configuration, you need a way in. Deploy an OOB Terminal Server directly in the rack that connects to the physical console ports, allowing you remote serial-line access over an isolated connection.

#### **Structural Blueprint: In-Band vs. True Out-of-Band**

| Architectural Metric | In-Band Management | Out-of-Band Management |
| --- | --- | --- |
| Physical Interface | Shared production interfaces | Dedicated, distinct management ports |
| Routing Table | Shares the global table | Isolated Management VRF |
| Blast Radius Isolation | Vulnerable to loops and DDoS | Unaffected by production failures |
| Access Gateways | Open to LAN or standard VPN | Isolated, MFA-authenticated OOB VPN |

#### **Production Configuration Blueprint (Cisco IOS-XE)**
This guide shows how to define a management VRF, assign a port to it, and force management protocols to answer only within that domain.
1. **Define the Isolated Management VRF**

```text
vrf definition Mgmt-vrf
  address-family ipv4
  exit-address-family

```

2. **Configure the Dedicated Physical Management Interface**

```text
interface GigabitEthernet0
  description PHYSICALLY ISOLATED MGMT PORT
  vrf forwarding Mgmt-vrf
  ip address 192.168.100.15 255.255.255.0
  no shutdown

```

3. **Establish a Dedicated Gateway for Management Traffic**

```text
ip route vrf Mgmt-vrf 0.0.0.0 0.0.0.0 192.168.100.1

```

4. **Force SSH and vty Lines to Require the Management VRF**

```text
ip ssh server vrf Mgmt-vrf

line vty 0 15
  access-class OOB_ADMIN_SUBNET in vrf-also
  transport input ssh

```

#### **Securing the OOB Access Perimeter**
Just separating the wires isn't enough; the entry point into your OOB network has to be just as secure.
* **Dedicated Jump Hosts**: Don't connect directly from your laptop to an OOB switch. Log in to a hardened Jump Host in a restricted DMZ first.
* **Enforce Hardware MFA**: Access to the OOB network or Jump Host must require cryptographic token validation, like FIDO2 keys or push notifications.
* **Implement LTE Failover**: For critical data centers, give your console server an encrypted cellular LTE uplink. If the fiber is cut, you can still tunnel into the hardware over cellular bands to start recovery.

#### **Hardening Goal Reached**
By implementing a dedicated OOB management architecture backed by VRF separation, your control mechanisms are completely invisible to the production environment. Even during a catastrophic collapse or a major security incident, your engineers retain an uninterrupted, secure path to analyze and restore the network.


### Session Controls: Managing timeout and concurrency
Session controls harden the management plane by terminating inactive administrative sessions and limiting concurrent logins. This prevents unauthorized users from hijacking abandoned terminals or running massive parallel brute-force attacks. Leaving sessions open indefinitely creates a massive risk if an administrator walks away from their workstation, while allowing unlimited concurrent connections gives attackers a playground for credential stuffing.

Enforcing strict timeout thresholds and concurrency limits ensures that your management access remains controlled, resource-efficient, and highly resistant to unauthorized exposure.

#### **Core Security Standards for Session Control**
To build a resilient session policy, you need to enforce strict boundaries on how long sessions last and how many can exist at once.
* **Enforce Strict Absolute and Inactivity Timeouts**: Implement timers that terminate a session if no command is entered for a set time (inactivity) or if the session reaches a maximum total duration (absolute).
* **Limit Concurrent Administrative Logins**: Restrict the maximum number of sessions globally, per user, or per communication line. This disrupts automated password spraying and mitigates DoS attempts on the management processor.
* **Enforce Single-Session Line Allocation**: Configure the management interface to allow only one user per terminal line, which automatically clears hung sessions when a new login starts.

#### **Production Configuration Blueprint (Cisco IOS-XE)**
This blueprint shows how to configure automated timeouts, limit concurrency, and police virtual terminal (vty) lines.
1. **Configuring Executive Inactivity Timeouts**
The `exec-timeout` command acts as a dead-man's switch. If an admin leaves a terminal open, the device tears down the connection automatically.

```text
line vty 0 15
  exec-timeout 5 0

line console 0
  exec-timeout 5 0

```

2. **Restricting Concurrent Virtual Terminal (vty) Lines**
Network devices often leave 16+ lines open by default. Scale this back to the absolute minimum needed for your team.

```text
line vty 0 2
  transport input ssh

line vty 3 15
  transport input none
  login block-for 1000 attempts 1 within 1

```

3. **Implementing Global Access and Session Bounding**

```text
login block-for 60 attempts 3 within 30
configuration mode exclusive auto

```

#### **AAA Server Concurrency Controls**
In enterprise environments, you should enforce these controls via centralized platforms like Cisco ISE or Aruba ClearPass.
* **Max-Sessions Per User**: Restricts a specific username to a set number of concurrent logins across the entire fabric. If an admin is logged into one switch, they can't log in elsewhere with the same account.
* **Session-Timeout (Absolute)**: Enforces a hard stop—like 60 or 120 minutes—forcing re-authentication and MFA checks regardless of active typing.

#### **Session Control Parameter Matrix**

| Session Control Metric | Insecure Default State | Hardened Enterprise Baseline | Security Objective |
| --- | --- | --- | --- |
| Inactivity Timeout | Infinite | 5 to 10 Minutes Max | Stop hijacking on unattended endpoints. |
| Active vty Lines | 16+ Lines | 2 to 4 Lines Max | Shrink the surface for brute-force attacks. |
| Failed Login Action | Infinite Retries | Block IP after 3 Failures | Thwart password spraying. |
| User Concurrency | Unlimited | 1 Active Session Per Admin | Ensure accountability and stop credential sharing. |

#### **Hardening Goal Reached**
By restricting terminal allocations, implementing an aggressive 5-minute inactivity timeout, and enforcing automated IP-blocking for failed logins, your session plane is now resilient. Dormant connections are gone, and automated brute-force attempts are suppressed at the terminal boundary.

---

## AAA Basics
### Authentication Methods: Verifying user identities
AAA (Authentication, Authorization, and Accounting) is the foundational framework used to manage access, enforce security policies, and audit activity across your network. It provides a scalable way to verify who a user is, dictate what resources they can access, and track exactly what they do.

Authentication is the first and most critical pillar here; it validates a user's identity before they get any network or configuration access.

#### **The Three Pillars of AAA Explained**
To understand authentication, you have to look at the full AAA lifecycle:
* **Authentication (Who are you?)**: Verifies identity using credentials like passwords, digital certificates, or multi-factor tokens.
* **Authorization (What can you do?)**: Sets permissions after verification. It decides if a user gets read-only access or full administrative control.
* **Accounting (What did you do?)**: Records session activity, like commands entered and configuration changes, and sends this data to a SIEM for auditing.

#### **Centralized vs. Local Authentication Methods**
When you configure network hardware, you define a fallback list. If the most secure method fails, the device drops to the next method in the sequence.
1. **Centralized Authentication (The Enterprise Standard)**: This offloads verification to a dedicated security server. It removes the risk of managing unique password databases on hundreds of different devices.
* **TACACS+**: The top choice for device administration. It encrypts the entire payload (usernames and passwords) and separates authentication from authorization, allowing you to control individual command execution.
* **RADIUS**: The standard for network edge access (Wi-Fi or VPNs). It only encrypts the password field and combines authentication and authorization into a single transaction.

2. **Local Authentication (The Fallback Mechanism)**: This relies on a database stored directly on the device.
* **Best Use Case**: Emergency fallback. If a network link breaks and the device can't talk to the centralized server, local auth lets an engineer on a physical console port troubleshoot the issue.

#### **Production Configuration Blueprint (Cisco IOS-XE)**
This blueprint shows how to define a local emergency user, map TACACS+ servers, and implement a secure AAA method list for vty management lines.
1. **Configure Local Emergency Fallback Accounts**

```text
username emergency_admin privilege 15 secret SuperSecureFallbackPassword99!

```

2. **Define Centralized TACACS+ Server Infrastructure**

```text
tacacs server TACACS_PRIMARY
  address ipv4 10.10.10.25
  key 7 CryptographicSharedSecretKey#123

```

3. **Create the AAA Authentication Method List**

```text
aaa new-model
aaa group server tacacs+ TACACS_GROUP
  server name TACACS_PRIMARY

aaa authentication login default group TACACS_GROUP local

```

4. **Apply the Policy to Access Lines**

```text
line vty 0 4
  login authentication default
  transport input ssh

```

#### **Essential Authentication Hardening Best Practices**
* **Enforce Account Lockout Policies**: Set your AAA server to lock an account after 3 to 5 failed login attempts to stop dictionary attacks.
* **Implement MFA**: Integrate your TACACS+ or RADIUS server with modern identity providers (like Duo or Okta) to require push notifications or TOTP tokens.
* **Ban Shared Accounts**: Every administrator needs a unique credential set. Generic accounts like "admin" break your accountability chain.

#### **Hardening Goal Reached**
By using a robust AAA framework that prioritizes encrypted TACACS+ verification before falling back to local databases, identity verification is strictly enforced. Attackers can't easily brute-force your interfaces, and you keep unified, scalable control over exactly who gets into your network core.


### Authorization Policies: Enforcing least privilege
AAA Authorization enforces the principle of least privilege by dynamically dictating exactly what commands an authenticated user can execute and what resources they can modify. While authentication verifies who the user is, authorization maps that identity to a specific control policy.

Without explicit authorization, any user who successfully logs into a switch or router could potentially gain full administrative access (Privilege Level 15). Implementing strict authorization ensures that personnel, such as junior technicians or third-party contractors, are restricted to the narrow subset of commands required for their specific roles.

#### **Understanding Privilege Levels vs. Command Authorization**
Enterprise network infrastructure generally regulates permissions using two mechanisms:
* **Static Privilege Levels (Local Control)**: Systems assign users a rank from 0 to 15. Level 1 is standard unprivileged mode, while Level 15 is full, unrestricted root access. This mechanism is rigid and cannot easily restrict a user to a specific subset of interfaces.
* **Granular Command Authorization (Centralized AAA Control)**: The modern enterprise standard. Every time a user hits Enter, the device intercepts the command and sends a real-time verification request to a centralized TACACS+ server. The server evaluates the string against the user's role-based policy and returns a PERMIT or DENY response.

#### **Core Security Models for Least Privilege**
To implement a resilient model, you must categorize users into distinct operational roles.

| Operational Role | Local Privilege Level | Authorized Command Scope | Business Objective |
| --- | --- | --- | --- |
| Network Auditing | Level 1 | show running-config, show version | Read-only security auditing. |
| Junior Admin | Level 7 | show interfaces, interface shutdown | Basic port resets; no global config changes. |
| Senior Engineer | Level 15 | configure terminal, router, crypto | Unrestricted architecture and security policy. |

#### **Production Configuration Blueprint (Cisco IOS-XE)**
This blueprint shows how to enable global authorization, map to a TACACS+ group, and configure fallback rules if the remote server goes offline.
1. **Enable AAA Authorization Globally**

```text
aaa new-model

aaa authorization exec default group TACACS_GROUP local
aaa authorization commands 15 default group TACACS_GROUP local

```

2. **Bind Authorization Policies to Virtual Access Lines**

```text
line vty 0 15
  authorization commands 15 default
  authorization exec default

```

3. **Define Local Authorization Fallback Profiles**
If the centralized servers go offline, the device drops back to local config. Use a parser view to maintain least privilege even during an outage.

```text
parser view JUNIOR_HELPDESK
  secret 7 LayeredViewPassword99$
  commands configure include nested interface
  commands interface include shutdown
  commands interface include no shutdown
  commands exec include show interfaces

```

#### **Managing Authorization via Centralized AAA Servers**
To scale authorization across your infrastructure, manage rules via a centralized engine like Cisco ISE or Aruba ClearPass using TACACS+ Command Sets.

A command set uses regular expression (regex) matching to define what is allowed:
* `Permit: ^show .*` (Allows any diagnostic show command)
* `Permit: ^interface GigabitEthernet.*` (Allows selecting an interface)
* `Permit: ^shutdown$` or `^no shutdown$` (Allows port resets)
* `Deny: .*` (Implicitly blocks all other modifications)
Because TACACS+ separates authentication from authorization, a user can log in with standard, secure credentials but still be restricted to a highly narrow, operational sandbox.

#### **Hardening Goal Reached**
By forcing real-time TACACS+ command authorization and tying fallback mechanisms to restrictive local views, the principle of least privilege is structurally enforced. A compromised junior account cannot alter global routing fabrics, wipe running configurations, or disable your perimeter defense rules.


### Accounting Logs: Tracking administrative activity
AAA Accounting logs create an unalterable trail of all administrative activity across your network, ensuring visibility for security compliance. While authentication and authorization control entry and permissions, accounting documents the exact actions a user takes during their session.

Whenever an engineer connects to a router or switch, changes an ACL, or shuts down a port, the device streams that telemetry in real time to a centralized AAA server. This data provides the forensic evidence needed for incident response and security audits.

#### **Types of AAA Accounting Logs**
To build a complete audit trail, accounting captures activity across two distinct scopes:
* **Exec Accounting (Session Tracking)**: Logs the lifecycle of an administrative shell session. It notes when a user logs in (Start) and logs out (Stop), their source IP, the terminal line used, and total session duration.
* **Command Accounting (Action Tracking)**: Captures every individual configuration or operational command typed by the user. If an admin types `shutdown` or `no ip address`, that string is timestamped and logged along with their username before the switch executes it.

#### **Production Configuration Blueprint (Cisco IOS-XE)**
This blueprint enables global AAA accounting, binds telemetry streams to your TACACS+ server group, and configures logging for sessions and configuration updates.

1. **Initialize Global AAA Accounting Commands**

```text
aaa new-model

aaa accounting exec default start-stop group TACACS_GROUP
aaa accounting commands 15 default start-stop group TACACS_GROUP

```

2. **Enable System and Configuration Mode Logging**

```text
aaa accounting system default start-stop group TACACS_GROUP

```

3. **Bind Accounting Policies to vty Lines**

```text
line vty 0 15
  accounting commands 15 default
  accounting exec default

```

4. **Configure Local Buffer Logging (Syslog Fallback)**
If the centralized server goes unreachable, the device logs adjustments locally and pushes them via Syslog to your SIEM.

```text
logging buffered 16384
logging trap informational
archive
  log config
    logging enable
    hidekeys

```

#### **Forensic Value of an Accounting Log Entry**
When log data is sent to a centralized server or a SIEM like Splunk, it generates structured metadata. A typical audited event contains these forensic markers:
* **Timestamp**: 2026-06-18 18:24:11 UTC
* **Device_Name**: Core-Router-01
* **User_ID**: admin_jdoe
* **Source_IP**: 192.168.42.115
* **Command_String**: `no ip access-list extended PERIMETER_INBOUND`

This visibility eliminates plausible deniability. If a critical firewall boundary is dropped, security analysts can track the change directly to the user account, their source machine, and the exact second the violation occurred.

#### **Hardening Goal Reached**
By implementing strict command and exec accounting tied to an encrypted TACACS+ fabric, all administrative activity is audited. Every keystroke is logged externally, which stops insider threats from wiping configuration histories or modifying network controls without leaving a forensic footprint.

---

## Secure Config Principles
### Baseline Configuration: Establishing gold standards
A gold master configuration baseline is an unalterable, organization-approved blueprint that specifies the exact security settings, protocols, and services allowed on every network device. Without a standardized baseline, infrastructure components inevitably suffer from configuration drift, where ad-hoc troubleshooting and delayed patches introduce unique security gaps on a device-by-device basis.

Enforcing a centralized baseline ensures that every switch, router, and firewall is uniformly hardened against threats from its first second on the wire.

#### **Core Pillars of a Gold Standard Baseline**
An enterprise-grade baseline must strip away non-essential capabilities and enforce uniform identity, management, and defensive controls.
* **Minimal Functionality**: Disable every protocol, feature, and physical port that doesn't serve an active business purpose, like HTTP or unneeded routing daemons.
* **Uniform Management Separation**: Force all devices to use identical management configs, such as pointing to the same Centralized AAA servers, utilizing the same OOB VRFs, and logging to the same SIEM collectors.
* **Cryptographic Consistency**: Mandate strict encryption baselines globally, ensuring that weak algorithms—like MD5, SHA-1, or RSA keys below 3072 bits—are systematically banned.

#### **Structural Framework of a Network Baseline**
Organize your gold standard configuration document into modular segments. This allows you to update specific security policies without rewriting the entire core template.
* **Device Identity & Banner Rules**: Legal warnings and hostnames.
* **Control Plane Hardening**: AAA, SSHv2, HTTPS, and management ACLs.
* **System Telemetry**: NTP synchronization and centralized Syslog.
* **Defensive Defaults**: BPDU Guard, uRPF, and inactive port shutdown.

#### **Production-Ready Baseline Blueprint (Cisco IOS-XE)**
This template strips away legacy exposure vectors and establishes strict infrastructure defaults.

```text
! SECTION 1: GLOBAL IDENTITY
no boot network
ip domain-name enterprise.local
banner login ^C
UNAUTHORIZED ACCESS PROHIBITED. ALL CONNECTIONS ARE AUDITED.
^C

! SECTION 2: MANAGEMENT PLANE CRYPTOGRAPHY AND AAA
crypto key generate rsa general-keys modulus 3072
ip ssh version 2
ip ssh server algorithm encryption aes256-gcm aes256-ctr
ip ssh server algorithm mac hmac-sha2-256

no ip http server
ip http secure-server
ip http secure-tls-version tls1.2

aaa new-model
aaa authentication login default group TACACS_GROUP local
aaa authorization exec default group TACACS_GROUP local
aaa accounting exec default start-stop group TACACS_GROUP

! SECTION 3: SYSTEM TELEMETRY
clock timezone UTC 0
ntp server 10.10.0.5 prefer
logging host 10.10.0.20

! SECTION 4: UNUSED RESOURCE NEUTRALIZATION
interface range GigabitEthernet0/1 - 24
  shutdown
  switchport mode access
  switchport access vlan 999

```

#### **Auditing Configuration Drift and Automation**
Writing a blueprint is only effective if you continuously audit the live environment to catch unauthorized changes.
* **Utilize Industry Benchmarks**: Map your standards back to authoritative frameworks like CIS Benchmarks or DISA STIGs.
* **Deploy Infrastructure-as-Code (IaC)**: Manage baseline text files in a version-controlled repository (like Git) and push configuration states using tools like Ansible or Terraform instead of manual SSH.
* **Automate Compliance Scanning**: Use network management tools to execute daily, read-only sweeps. If an engineer opens an unapproved port, your compliance engine should flag the drift and alert the SOC immediately.

#### **Hardening Goal Reached**
By deploying a standardized gold master configuration, you eliminate device variation across the enterprise. Security parameters, encryption keys, and log destinations remain perfectly uniform, allowing your team to detect and remediate anomalies in seconds.


### Configuration Auditing: Automating compliance checks
Automated configuration auditing replaces periodic, manual reviews with continuous compliance checking, ensuring network hardware never drifts from your established gold standards. Manual device reviews capture a single point in time, allowing temporary or malicious configuration drifts to persist undetected for months between audit cycles.

Automating this layer treats configuration code as a continually verifiable asset, scanning active environments to flag variations and provide audit-ready proof of compliance against frameworks like CIS Benchmarks or DISA STIGs.

#### The Audit Lifecycle: From Code to Continuous Control
Automated auditing transforms security enforcement into a closed-loop engineering process. Rather than relying on human diligence, the network is monitored against an unyielding source of truth.
* **Define Rules as Code**: Hardware baselines are translated into machine-readable conditional expressions or validation regular expressions (Regex).
* **Continuous Device Polling**: Automated compliance systems pull active configurations securely via SSH or APIs on a scheduled basis.
* **Algorithmic Parser Matching**: System states are compared line-by-line to check for presence, absence, and specific code line order—which is critical for firewalls and ACLs.
* **Exceptions and Alerts**: Non-compliant changes automatically spin up notifications within the Security Operations Center (SOC) or programmatically inject tickets into IT Service Management (ITSM) tools like ServiceNow.

#### Comparison Matrix: Enterprise vs. Open-Source Auditing Tools

| Auditing Platform | Deployment Paradigm | Strength Area | Compliance Reporting Depth |
| --- | --- | --- | --- |
| Itential / NetBrain | Enterprise Commercial | Dynamic Multi-Cloud & Mass Multi-Vendor Parsing. | Full compliance score matrix with automated drift rollbacks. |
| SolarWinds NCM | Enterprise Commercial | Turnkey regulatory templates (PCI-DSS, HIPAA). | Broad out-of-the-box reporting with executive dashboards. |
| Netpicker / rConfig | Focused Security Software | Lightweight, built-in CIS benchmark automation. | Deep diff logs showing exact unauthorized structural modifications. |
| Ansible (with Napalm) | Open-Source Framework | High-velocity IaC native state validation. | Custom output parsing; depends on engineer-written tasks. |

#### Automated Validation Blueprint (Ansible Compliance Playbook)
This YAML blueprint verifies that active devices match key constraints within the gold baseline: banning Telnet and enforcing a 3072-bit minimum encryption key modulus.

```text
---
- name: Hardening Compliance Audit — Network Infrastructure
  hosts: enterprise_switches
  gather_facts: no
  connection: network_cli

  tasks:
    - name: Pull Running Configuration from Target Switch
      cisco.ios.ios_facts:
        gather_subset:
          - config

    - name: Verify Anti-Telnet Hardening Rule
      ansible.builtin.assert:
        that:
          - "'transport input ssh' in ansible_facts.net_config"
          - "'transport input telnet' not in ansible_facts.net_config"
        fail_msg: "CRITICAL VIOLATION: Insecure Telnet management access detected."
        success_msg: "COMPLIANT: Transport input restricted to SSH."

    - name: Verify Cryptographic Strength Modulus
      ansible.builtin.assert:
        that:
          - "ansible_facts.net_config | regex_search('modulus 3072')"
        fail_msg: "VIOLATION: RSA key depth below 3072-bit baseline."
        success_msg: "COMPLIANT: 3072-bit RSA key confirmed."

```

#### Advanced Enforcement: Transitioning from Audit to Remediation
The ultimate evolution of automated auditing is Continuous Control Automation (CCA). Once your parsing engines identify structural variances, you choose between two primary response mechanisms:
* **Closed-Loop Automated Remediation**: The auditing engine detects an unapproved line change (like an open switchport) and instantly pushes an automated script to re-apply the gold baseline. This slams entry points shut without waiting for manual triage.
* **The DevSecOps CI/CD Pipeline ("Shift-Left")**: Prevent bad configurations from reaching production. Maintain your network configs in a version-controlled Git repository. When an engineer drafts a change, automated compliance checks run inside a pre-deployment pipeline. If the code violates a security policy, the build breaks, and the change is blocked.

#### Hardening Goal Reached
By implementing automated compliance checks via continuous state monitoring, vulnerability exposure due to configuration drift is neutralized. Unauthorized modifications are programmatically flagged within minutes, creating an ironclad, audit-ready operational record.


### Patch Management: Maintaining firmware integrity
Firmware patch management hardens your network architecture by remediating software flaws, preventing boot-level hijacking, and validating operating system integrity. Because switches, routers, and firewalls control the transit pathways for all corporate communication, unpatched edge vulnerabilities offer malicious actors a high-privilege launchpad into your environment.

Establishing a robust lifecycle ensures that patches are authenticated, tested, and committed without introducing downtime or code-level corruption.

#### The Network Firmware Patching Lifecycle
Updating network infrastructure requires handling low-level hardware dependencies. Security teams structure this rollout through precise phases:
* **Vulnerability Tracking**: Cross-reference your hardware inventory against vendor security advisories and the [CISA Known Exploited Vulnerabilities (KEV) Catalog](https://www.cisa.gov/known-exploited-vulnerabilities-catalog).
* **Cryptographic Integrity Validation**: Validate image signatures before transferring binary payloads onto production file systems to neutralize supply-chain injection.
* **Sandbox and Canary Testing**: Deploy updates onto staging hardware replicas to screen for routing protocol degradation, memory leaks, or performance anomalies.
* **Staggered Production Reloading**: Apply changes across high-availability fabrics progressively—updating secondary firewall peers first—to prevent localized processing blackholes.

#### Hardware-Level Image Validation Blueprint (Cisco IOS-XE)
To guarantee a firmware file hasn't been altered by a MitM attack or corrupted during transit, use cryptographic hash validation.
1. **Manually Calculating the Local Hash Value**

```text
! Generates a local SHA-256 fingerprint of the target firmware image
verify /md5 flash:cat9k_iosxe.17.09.04.SPA.bin

```

2. **Enforcing Continuous Automated Validation**

```text
! Instructs the hardware to cryptographically authenticate images during file copies
file verify auto

! Forces the hardware to run a full cryptographic inspection immediately before executing a reboot
reload /verify

```

#### Advanced Firmware Hardening Controls
Modern enterprise hardware uses built-in cryptographic chips to protect boot sequences from persistent rootkits.
* **Secure Boot Architecture**: Hardware embeds an immutable public key directly into the silicon. When a switch boots, this root key validates the digital signature of the microcode, which validates the bootloader, which finally validates the OS image. If an attacker injects a rogue, tampered OS, the hardware detects the broken signature and drops the boot routine immediately.
* **Fallback and Recovery Storage**: Devices should retain a stable, historical OS image in a protected, read-only flash partition. If an update encounters a panic loop mid-boot, the device triggers an automatic rollback to restore connectivity.

#### Patch Deployment Window Allocation Matrix
Prioritize your deployment schedule based on vulnerability severity and exposure zone:

| Device Exposure Class | Metric Threshold | Mandatory Patch Deadline | Mitigating Action |
| --- | --- | --- | --- |
| Edge Perimeter | CVSS 9.0–10.0 | Within 72 Hours | Isolate management access. |
| Core Infrastructure | CVSS 7.0–8.9 | Within 14 Days | Mirror traffic to check for anomalies. |
| Access Layer | CVSS 4.0–6.9 | Within 30–60 Days | Disable dynamic trunking protocols. |

#### Hardening Goal Reached
By implementing mandatory cryptographic hash verification, enforcing secure-boot hardware validations, and executing structured deployment timelines based on asset exposure, the network fabric remains resilient against code-level manipulation. Firmware exploits are neutralized before they can be leveraged to establish persistence within the network architecture.

---

### Final Notes
This series of modules established a comprehensive strategy for hardening network infrastructure, moving beyond basic configurations to implement a defense-in-depth architecture. By securing the management, control, and data planes, you’ve learned to minimize attack surfaces, enforce strict identity-based access, and ensure continuous operational integrity through automation. These practices are critical because they shift network security from a reactive, manual effort to a proactive, resilient standard, ensuring that your core infrastructure remains protected against both external exploits and internal configuration drift.

#### **Switch Hardening**
* Secure physical access by shutting down all unused ports and assigning them to a blackhole VLAN.
* Use port security features like BPDU Guard to protect Layer 2 topology and prevent unauthorized switch insertion.
* Disable legacy protocols like DTP and VTP to minimize the attack surface of the switching fabric.

#### **Router Hardening**
* Deploy Unicast Reverse Path Forwarding (uRPF) in strict mode to prevent IP spoofing at the edge.
* Filter out Bogon and internal address space from untrusted interfaces using iACLs.
* Implement Control Plane Policing (CoPP) to protect the router CPU from volumetric DoS and flooding attacks.

#### **Secure Management Plane**
* Migrate all administration from cleartext protocols (Telnet/HTTP) to encrypted alternatives (SSHv2/HTTPS).
* Use dedicated Out-of-Band (OOB) management networks to maintain access during production outages or DDoS events.
* Enforce strict session management with inactivity timeouts and limited concurrent login attempts.

#### **AAA Basics**
* Centralize all authentication using TACACS+ for command-level granularity and RADIUS for network access control.
* Enforce the principle of least privilege by mapping user roles to specific, restricted command sets.
* Enable comprehensive accounting to create an immutable audit trail of every configuration change and session event.

#### **Secure Config Principles**
* Define a "Gold Master" baseline and enforce it consistently across the entire enterprise to eliminate configuration drift.
* Treat configurations as code, using version control and automation tools to push updates rather than manual sessions.
* Automate compliance auditing to continuously verify that live devices remain compliant with security benchmarks like CIS or DISA STIGs.
