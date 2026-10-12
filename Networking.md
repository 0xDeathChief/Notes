# Networking & Cybersecurity Fundamentals
## A Practical Learning Guide for Junior Cybersecurity Engineers

**Platform:** Kali Linux  
**Level:** Beginner to Intermediate  
**Learning method:** 10% Theory + 90% Practical  
**Language:** বাংলা, with English technical terminology  
**Goal:** Understand networking from packet creation to security analysis.

---

# Table of Contents

1. Networking Fundamentals
2. IP Address and IP Packet
3. IPv4 Header: Every Field Explained
4. IP Addressing, Subnetting and CIDR
5. MAC Address, ARP and Ethernet Frames
6. Ports and Network Services
7. TCP and UDP
8. TCP Three-Way Handshake
9. TCP Flags and Port Scanning Concepts
10. ICMP and Network Troubleshooting
11. DHCP and NAT
12. OSI Model: Workflow and Mechanism
13. Physical and Network Devices
14. Wireless Networking and Wi-Fi Security
15. Practical Packet Analysis with Wireshark and tcpdump
16. Integrated Kali Linux Lab
17. Cybersecurity Troubleshooting Scenarios
18. Practice Questions
19. Final Lab Checklist
20. Recommended Learning Resources

---

# 1. Networking Fundamentals

## 1.1 What is a Network?

Network হলো একাধিক device-এর মধ্যে data আদান-প্রদানের ব্যবস্থা।

উদাহরণ:
- তোমার laptop
- Wi-Fi router
- Mobile phone
- Web server
- Printer

এগুলো wired অথবা wireless connection-এর মাধ্যমে একে অপরের সঙ্গে communicate করতে পারে।

### Real-life scenario

তুমি Kali Linux laptop থেকে একটি website open করলে:

1. Browser website-এর domain resolve করার জন্য DNS query করতে পারে।
2. Computer destination-এর IP address জানতে পারে।
3. Computer packet তৈরি করে।
4. Packet local network-এর gateway দিয়ে যেতে পারে।
5. Router packet forward করে।
6. Destination server request গ্রহণ করে।
7. Server response পাঠায়।

মনে রাখবে, বাস্তবে DNS cache, proxy, VPN, routing, TLS এবং অন্যান্য বিষয় অনুযায়ী workflow পরিবর্তিত হতে পারে।

## 1.2 গুরুত্বপূর্ণ Networking Terms

| Term | সহজ ব্যাখ্যা |
|---|---|
| Host | Network-এ যুক্ত একটি device |
| Client | Service ব্যবহার করার জন্য request পাঠায় |
| Server | Client-কে service দেয় |
| IP Address | Network-এ logical addressing |
| MAC Address | Local network-এ interface শনাক্ত করার address |
| Packet | Network layer-এর data unit |
| Frame | Data-link layer-এর data unit |
| Port | TCP/UDP service বা application endpoint শনাক্ত করে |
| Protocol | Communication-এর নিয়ম |
| Router | বিভিন্ন IP network-এর মধ্যে packet forward করে |
| Switch | সাধারণত MAC address ব্যবহার করে Ethernet frame forward করে |

## 1.3 প্রথম Kali Linux Lab

টার্মিনালে চালাও:

```bash
ip addr
ip route
ip neigh
```

কী দেখবে:

- `ip addr`: Network interface এবং IP address।
- `ip route`: Routing table।
- `ip neigh`: Neighbor cache, যেখানে IPv4-এর ARP mapping বা IPv6-এর neighbor information থাকতে পারে।

এবার:

```bash
ping -c 4 127.0.0.1
```

**Expected result:** Loopback interface-এ reply পাবে।

এরপর:

```bash
ping -c 4 1.1.1.1
```

এই পরীক্ষা external IP connectivity যাচাই করতে সাহায্য করে। তবে firewall, routing বা ICMP filtering-এর কারণে reply না পেলেই Internet সম্পূর্ণ বন্ধ, এমন সিদ্ধান্ত নেওয়া যাবে না।

---

# 2. IP Address and IP Packet

## 2.1 IP কী?

IP-এর পূর্ণরূপ Internet Protocol।

IP network layer-এ logical addressing এবং packet forwarding-এর জন্য ব্যবহৃত হয়।

দুটি প্রধান version:

- IPv4
- IPv6

IPv4 address 32-bit এবং IPv6 address 128-bit।

IPv4 example:

```text
192.168.1.10
```

IPv6 example:

```text
2001:db8::10
```

`2001:db8::/32` documentation ও example-এর জন্য reserved।

## 2.2 IP কীভাবে কাজ করে?

ধরো:

```text
Source:      192.168.1.10
Destination: 93.184.216.34
```

Source host destination-এর IP address ব্যবহার করে packet তৈরি করে।

Source ও destination একই subnet-এ থাকলে host সাধারণত সরাসরি destination-এর MAC address resolve করে।

Destination অন্য network-এ থাকলে host সাধারণত default gateway-এর MAC address resolve করে এবং packet gateway-তে পাঠায়।

গুরুত্বপূর্ণ বিষয়:

**IP destination এবং Ethernet destination সবসময় একই device-কে নির্দেশ করে না।**

উদাহরণস্বরূপ, remote web server-এর IP packet-এর destination হতে পারে, কিন্তু প্রথম Ethernet frame-এর destination হবে local router-এর MAC address।

## 2.3 Packet বনাম Frame

একটি সাধারণ Ethernet ও IPv4 transmission-এর ধারণাগত structure:

```text
Ethernet Frame
┌───────────────────────────────────────────┐
│ Ethernet Header                           │
│  Destination MAC | Source MAC | EtherType │
├───────────────────────────────────────────┤
│ IPv4 Packet                               │
│  IP Header                                │
│  TCP Header                               │
│  Application Data                         │
├───────────────────────────────────────────┤
│ Ethernet Trailer (FCS)                    │
└───────────────────────────────────────────┘
```

বাস্তব packet capture-এ FCS সবসময় দেখা যায় না। VLAN tagging থাকলে Ethernet frame-এ অতিরিক্ত field-ও থাকতে পারে।

এখানে:

- Ethernet Frame: Data-link layer-এর unit।
- IP Packet: Network layer-এর unit।
- TCP Segment: TCP-এর unit।
- Application Data: HTTP-এর মতো protocol-এর content হতে পারে।

সহজভাবে মনে রাখো:

**Application Data → TCP Segment → IP Packet → Ethernet Frame**

Receiving side-এ decapsulation-এর মাধ্যমে উল্টো দিকে data উপরের layer-এ যায়।

## 2.4 IP Packet-এর practical scenario

তুমি একটি local web server-এ request পাঠাচ্ছ।

```text
Kali IP:       192.168.1.10
Server IP:     192.168.1.20
Server Port:   8080
Protocol:      TCP
```

যদি দুই host একই subnet-এ থাকে, Ethernet frame-এর destination MAC সাধারণত server interface-এর MAC হবে।

যদি server অন্য network-এ থাকে, Kali সাধারণত gateway-এর MAC ব্যবহার করবে।

## 2.5 Kali Lab: IP Information

```bash
ip -4 addr show
ip -6 addr show
ip route
ip link
```

নিজের active interface-এর নাম খুঁজে বের করো:

```bash
ip -br addr
```

সম্ভাব্য interface:

```text
eth0
ens33
wlan0
```

তোমার interface-এর নাম আলাদা হতে পারে।

**Task:** নিজের IP address, subnet prefix, MAC address এবং default gateway লিখে রাখো।

---

# 3. IPv4 Header: Every Field Explained

IPv4 header সাধারণত 20 bytes হয়, যদি optional field না থাকে।

Options থাকলে header বড় হতে পারে, সর্বোচ্চ 60 bytes পর্যন্ত।

## 3.1 IPv4 Header Structure

```text
0                   15                  31
+-------------------+-------------------+
| Version |   IHL   | DSCP/ECN          |
+-------------------+-------------------+
| Total Length                          |
+---------------------------------------+
| Identification      | Flags | Frag Off|
+---------------------------------------+
| TTL     | Protocol  | Header Checksum |
+---------------------------------------+
| Source IPv4 Address                  |
+---------------------------------------+
| Destination IPv4 Address             |
+---------------------------------------+
| Options and Padding (if present)      |
+---------------------------------------+
```

## 3.2 Version

Version field IPv4 না IPv6, তা নির্দেশ করে।

- IPv4-এর value: `4`
- IPv6-এর value: `6`

তবে IPv4 header-এর Version field নিজে IPv6 packet-এর structure বর্ণনা করে না। IPv6-এর header format আলাদা।

## 3.3 IHL: Internet Header Length

IHL IPv4 header-এর দৈর্ঘ্য নির্দেশ করে।

এটি 4-bit field এবং 32-bit word-এ length প্রকাশ করে।

```text
Minimum IHL = 5
Header Size = 5 × 4 = 20 bytes
```

যদি IHL `6` হয়:

```text
Header Size = 6 × 4 = 24 bytes
```

অর্থাৎ options থাকার কারণে header বড় হয়েছে।

## 3.4 TOS: Type of Service

পুরোনো নাম Type of Service বা TOS।

বর্তমানে এই byte-এর field-কে DS field হিসেবে ব্যাখ্যা করা হয়:

- DSCP: Traffic classification ও differentiated services।
- ECN: Explicit Congestion Notification।

উদাহরণ: Voice traffic-কে network-এ বিশেষ forwarding treatment দেওয়ার জন্য DSCP ব্যবহার করা হতে পারে।

মনে রাখবে, DSCP value দিলেই সব router QoS guarantee করবে না। Network configuration-ও গুরুত্বপূর্ণ।

## 3.5 Total Length

Total Length সম্পূর্ণ IPv4 packet-এর length নির্দেশ করে।

এর মধ্যে অন্তর্ভুক্ত:

```text
IPv4 Header + IPv4 Payload
```

উদাহরণ:

```text
IPv4 Header = 20 bytes
Payload     = 100 bytes
Total Length = 120 bytes
```

IPv4 Total Length field 16-bit। তাই সর্বোচ্চ theoretical packet length 65,535 bytes।

## 3.6 Identification

Identification field fragmentation এবং fragmented packet reassembly-তে সাহায্য করে।

একটি fragmented IPv4 datagram-এর fragment-গুলোতে একই Identification value থাকে, যাতে receiving host সঠিক fragment-গুলো শনাক্ত করতে পারে।

## 3.7 IP Flags

IPv4 Flags field 3-bit।

মূলত গুরুত্বপূর্ণ flag:

- Reserved bit
- DF: Don't Fragment
- MF: More Fragments

### DF

DF সেট থাকলে packet fragment করা যাবে না। Path MTU-এর তুলনায় packet বড় হলে IPv4 router packet drop করে ICMP error পাঠাতে পারে, যদি প্রযোজ্য নিয়মে অনুমোদিত হয়।

### MF

MF সেট থাকলে আরও fragment আসার কথা। শেষ fragment-এ MF সাধারণত zero থাকে।

## 3.8 Fragment Offset

Fragment Offset জানায় মূল datagram-এর payload-এর কোন অংশটি এই fragment বহন করছে।

এটি 8-byte unit-এ প্রকাশ করা হয়।

উদাহরণ:

```text
Fragment Offset = 100
Payload Position = 100 × 8 = 800 bytes
```

অর্থাৎ এই fragment-এর payload মূল datagram-এর payload-এর 800-byte position থেকে শুরু।

## 3.9 TTL: Time To Live

TTL-এর উদ্দেশ্য packet যেন routing loop-এ অনির্দিষ্টকাল ঘুরতে না থাকে, তা নিশ্চিত করা।

প্রতিটি router সাধারণত forwarding-এর সময় TTL কমায়।

TTL শূন্য হয়ে গেলে packet আর forward করা হয় না।

উদাহরণ:

```text
Initial TTL: 64
After one router: 63
After two routers: 62
```

এটি একটি উদাহরণ। Initial TTL বিভিন্ন operating system ও device-এ আলাদা হতে পারে।

TTL ব্যবহার করে `traceroute` network path সম্পর্কে ধারণা দেয়।

## 3.10 Protocol

Protocol field IPv4 payload-এ কোন protocol আছে তা নির্দেশ করে।

| Decimal value | Protocol |
|---:|---|
| 1 | ICMP |
| 6 | TCP |
| 17 | UDP |
| 41 | IPv6 encapsulation |
| 47 | GRE |
| 50 | ESP |
| 89 | OSPF |

এই field-এর value TCP বা UDP port number নয়।

## 3.11 Header Checksum

IPv4 Header Checksum শুধু IPv4 header-এর error detection-এর জন্য ব্যবহৃত হয়।

এটি TCP payload-এর checksum নয় এবং সম্পূর্ণ application data-এর integrity guarantee করে না।

প্রতিটি router TTL পরিবর্তন করলে IPv4 header checksum-ও update করতে হয়।

IPv6-এ IPv4-এর মতো header checksum নেই।

## 3.12 Source এবং Destination Address

Source Address হলো packet-এর source IPv4 address।

Destination Address হলো packet-এর destination IPv4 address।

উদাহরণ:

```text
Source:      192.168.1.10
Destination: 93.184.216.34
```

NAT ব্যবহার হলে network-এর বিভিন্ন পর্যায়ে source বা destination address পরিবর্তিত হতে পারে।

## 3.13 TCP Fragment এবং IP Fragment-এর পার্থক্য

এখানে একটি গুরুত্বপূর্ণ বিষয়:

**TCP নিজে IP fragmentation করে না।**

- TCP segment তৈরি করে।
- IPv4 fragmentation IP layer-এ ঘটে।
- TCP segmentation application data-কে TCP segment-এ ভাগ করে।
- IP fragmentation একটি IPv4 datagram-কে একাধিক IP fragment-এ ভাগ করে।

TCP segment-এর কিছু অংশ একাধিক IP fragment-এ ছড়িয়ে যেতে পারে। Receiver-এর IP layer-এ reassembly সম্পন্ন হওয়ার পর TCP সেই segment process করতে পারে।

## 3.14 Kali Lab: Packet Header দেখা

প্রথমে interface ও route দেখো:

```bash
ip -br addr
ip route
```

নিজের lab network-এর একটি reachable host নির্ধারণ করো। তারপর terminal-এ চালাও:

```bash
sudo tcpdump -ni any 'icmp'
```

অন্য terminal থেকে:

```bash
ping -c 4 127.0.0.1
```

Loopback interface-এ capture করলে ICMP request ও reply দেখতে পারবে।

Capture বন্ধ করতে `Ctrl+C` চাপো।

Internet-এ capture করলে interface ও filter অনুযায়ী ফল ভিন্ন হবে। Wireshark-এ IPv4 packet খুলে Version, IHL, Total Length, TTL, Protocol, Source এবং Destination field পরীক্ষা করো।

---

# 4. IP Addressing, Subnetting and CIDR

## 4.1 Classful Addressing

ঐতিহাসিকভাবে IPv4 address-কে Class A, B, C, D এবং E হিসেবে ভাগ করা হতো।

| Class | First octet | Historical default prefix | ব্যবহার |
|---|---|---|---|
| A | 1–126 | /8 | বড় network |
| B | 128–191 | /16 | মাঝারি network |
| C | 192–223 | /24 | ছোট network |
| D | 224–239 | প্রযোজ্য নয় | Multicast |
| E | 240–255 | প্রযোজ্য নয় | Reserved/experimental |

বিশেষভাবে:

- `0.0.0.0/8` বিশেষ কাজে ব্যবহৃত হয়।
- `127.0.0.0/8` loopback-এর জন্য।
- `255.255.255.255` limited broadcast address।

বর্তমান Internet routing মূলত classless addressing ব্যবহার করে।

## 4.2 Classless Addressing

Classless Inter-Domain Routing বা CIDR-এ prefix length দিয়ে network portion নির্ধারণ করা হয়।

উদাহরণ:

```text
192.168.10.0/26
```

`/26` অর্থ প্রথম 26 bit network prefix-এর অংশ।

Subnet mask:

```text
255.255.255.192
```

## 4.3 CIDR থেকে Subnet Mask

| Prefix | Subnet mask | Total IPv4 addresses |
|---|---|---:|
| /24 | 255.255.255.0 | 256 |
| /25 | 255.255.255.128 | 128 |
| /26 | 255.255.255.192 | 64 |
| /27 | 255.255.255.224 | 32 |
| /28 | 255.255.255.240 | 16 |
| /29 | 255.255.255.248 | 8 |
| /30 | 255.255.255.252 | 4 |

সাধারণ subnet-এ network ও broadcast address বাদ দিলে usable host count হয়:

```text
Usable hosts = 2^(32 - prefix) - 2
```

এই সূত্র সাধারণ IPv4 subnet-এর ক্ষেত্রে ব্যবহার করবে। `/31` point-to-point link এবং `/32` host route-এর বিশেষ নিয়ম রয়েছে।

## 4.4 Subnetting Example

Question:

```text
IP: 192.168.10.70/26
```

Step 1: Subnet mask বের করো।

```text
255.255.255.192
```

Step 2: Block size:

```text
256 - 192 = 64
```

Step 3: Subnet boundary:

```text
192.168.10.0
192.168.10.64
192.168.10.128
192.168.10.192
```

`192.168.10.70` দ্বিতীয় subnet-এর মধ্যে আছে।

Final answer:

```text
Network:       192.168.10.64
Broadcast:     192.168.10.127
First usable:  192.168.10.65
Last usable:   192.168.10.126
Usable hosts:  62
Total address: 64
```

## 4.5 Public এবং Private IP

Private IPv4 range:

| Range | CIDR |
|---|---|
| 10.0.0.0 – 10.255.255.255 | 10.0.0.0/8 |
| 172.16.0.0 – 172.31.255.255 | 172.16.0.0/12 |
| 192.168.0.0 – 192.168.255.255 | 192.168.0.0/16 |

Private IP সাধারণত public Internet-এ globally routable নয়।

Public IP globally routable হতে পারে, তবে address allocation, routing policy এবং filtering-এর শর্ত রয়েছে।

`172.20.1.5` private, কিন্তু `172.40.1.5` এই private range-এর মধ্যে পড়ে না।

## 4.6 Kali Lab: Subnet Calculation

নিজের IP ও prefix দেখো:

```bash
ip -4 addr show
```

তারপর নিজের network-এর সঙ্গে মিলিয়ে নিচের subnet calculation করো।

Practice:

```text
192.168.10.70/26
10.10.20.150/27
172.16.5.200/28
192.168.1.130/25
```

প্রতিটির জন্য বের করবে:

- Network address
- Broadcast address
- First usable IP
- Last usable IP
- Total address
- Usable host

ঐচ্ছিকভাবে Python দিয়ে calculation verify করতে পারো:

```python
import ipaddress

network = ipaddress.ip_network("192.168.10.70/26", strict=False)

print("Network:", network.network_address)
print("Broadcast:", network.broadcast_address)
print("Netmask:", network.netmask)
print("Total addresses:", network.num_addresses)
print("Usable hosts:", network.num_addresses - 2)
```

এটি সাধারণ IPv4 subnet-এর জন্য। `/31` ও `/32`-এর ক্ষেত্রে usable-host calculation আলাদা করে বিবেচনা করবে।

---

# 5. MAC Address, ARP and Ethernet Frames

## 5.1 MAC Address কী?

MAC address সাধারণত Ethernet বা Wi-Fi interface-কে local link-এ শনাক্ত করতে ব্যবহৃত হয়।

সাধারণ 48-bit MAC address:

```text
00:1A:2B:3C:4D:5E
```

MAC address IP address-এর মতো routed Internet address নয়।

## 5.2 MAC Address এবং IP Address

| বিষয় | IP Address | MAC Address |
|---|---|---|
| Layer | Network | Data Link |
| প্রধান কাজ | Logical addressing | Local link addressing |
| Router crossing | Source/destination IP সাধারণত থাকে, NAT হলে বদলাতে পারে | প্রতিটি Ethernet hop-এ frame MAC বদলায় |
| Example | 192.168.1.10 | 00:1A:2B:3C:4D:5E |

## 5.3 ARP কী?

ARP-এর পূর্ণরূপ Address Resolution Protocol।

IPv4 local network-এ কোনো IPv4 address-এর MAC address জানতে ARP ব্যবহৃত হয়।

উদাহরণ:

```text
Who has 192.168.1.20?
Tell 192.168.1.10.
```

যে host-এর IP `192.168.1.20`, সে reply করে নিজের MAC address জানায়।

Remote destination হলে host সাধারণত remote server-এর MAC নয়, বরং default gateway-এর MAC resolve করে।

IPv6-এ ARP ব্যবহার হয় না। IPv6-এর Neighbor Discovery Protocol (NDP) ICMPv6-এর মাধ্যমে কাজ করে।

## 5.4 Kali Lab: ARP ও Neighbor Table

```bash
ip neigh
```

একটি lab host-এর সঙ্গে যোগাযোগের পর আবার চালাও:

```bash
ping -c 1 192.168.1.1
ip neigh
```

`192.168.1.1`-এর জায়গায় নিজের actual gateway ব্যবহার করবে।

Capture:

```bash
sudo tcpdump -ni any arp
```

কোনো ARP traffic না দেখলে interface selection, cache এবং network activity পরীক্ষা করো।

---

# 6. Ports and Network Services

## 6.1 Port কী?

Port number TCP ও UDP transport layer-এ application endpoint শনাক্ত করতে সাহায্য করে।

Port number 16-bit, তাই range:

```text
0–65535
```

| Range | Category |
|---|---|
| 0–1023 | Well-known ports |
| 1024–49151 | Registered ports |
| 49152–65535 | Dynamic/private ports, commonly used by clients |

Operating system ও application configuration অনুযায়ী ব্যবহার ভিন্ন হতে পারে।

## 6.2 গুরুত্বপূর্ণ Port

| Port | Protocol/Service | ব্যবহার |
|---:|---|---|
| 20/21 | TCP FTP | File transfer |
| 22 | TCP SSH | Secure remote shell |
| 25 | TCP SMTP | Mail transfer |
| 53 | TCP/UDP DNS | Name resolution |
| 67/68 | UDP DHCPv4 | Address configuration |
| 80 | TCP HTTP | Web traffic |
| 110 | TCP POP3 | Email retrieval |
| 123 | UDP NTP | Time synchronization |
| 143 | TCP IMAP | Email access |
| 161/162 | UDP SNMP | Network management |
| 443 | TCP HTTPS | Secure web traffic |
| 445 | TCP SMB | File sharing |
| 514 | UDP Syslog, commonly | Log transport |
| 3306 | TCP MySQL | Database |
| 3389 | TCP/UDP RDP | Remote desktop |
| 5432 | TCP PostgreSQL | Database |
| 27017 | TCP MongoDB, default | Database |

Port number দেখে service-এর উপস্থিতি নিশ্চিত করা যায় না। Service অন্য port-এও চলতে পারে।

## 6.3 Local Port Lab

নিজের Kali machine-এ একটি HTTP server চালাও:

```bash
mkdir -p ~/network-lab
cd ~/network-lab
echo "Hello from Kali Lab" > index.html
python3 -m http.server 8000 --bind 127.0.0.1
```

অন্য terminal-এ:

```bash
curl http://127.0.0.1:8000
```

Expected output:

```text
Hello from Kali Lab
```

Listening socket দেখো:

```bash
ss -lntp
```

এখানে TCP listening socket-এর local port `8000` দেখতে পারবে।

Server বন্ধ করতে প্রথম terminal-এ `Ctrl+C` চাপো।

---

# 7. TCP and UDP

## 7.1 TCP কী?

TCP-এর পূর্ণরূপ Transmission Control Protocol।

TCP connection-oriented এবং reliable byte-stream service দেয়।

মূল বৈশিষ্ট্য:

- Connection establishment
- Sequence numbers
- Acknowledgement
- Retransmission
- Flow control
- Congestion control
- Ordered byte stream

উদাহরণ: SSH, HTTPS over TCP, অনেক database connection।

## 7.2 UDP কী?

UDP-এর পূর্ণরূপ User Datagram Protocol।

UDP connection establishment ছাড়াই datagram পাঠাতে পারে। TCP-এর মতো built-in retransmission বা ordered delivery guarantee দেয় না।

উদাহরণ:

- DNS query-এর অনেক ব্যবহার
- Real-time voice/video
- NTP
- QUIC, যা UDP-এর ওপর transport functions বাস্তবায়ন করে

UDP-তে application নিজেই reliability প্রয়োজন হলে তা implement করতে পারে।

## 7.3 TCP বনাম UDP

| Feature | TCP | UDP |
|---|---|---|
| Connection | Connection-oriented | Connectionless |
| Reliability | Built-in mechanisms | Built-in delivery guarantee নেই |
| Ordering | Ordered byte stream | Message ordering guarantee নেই |
| Header size | Minimum 20 bytes | 8 bytes |
| Data unit | Segment | Datagram |
| Example | SSH, HTTP/1.1, HTTP/2 over TCP | DNS, NTP, QUIC transport |

## 7.4 Practical Scenario

ধরো, একটি file download হচ্ছে।

TCP missing data detect করে retransmission করতে পারে।

অন্যদিকে, live voice communication-এ application latency-কে বেশি গুরুত্ব দিতে পারে। UDP datagram হারিয়ে গেলে application পরিস্থিতি অনুযায়ী সেই loss সামলাতে পারে।

## 7.5 Kali Lab: TCP এবং UDP পর্যবেক্ষণ

Terminal 1:

```bash
nc -lv 9001
```

Terminal 2:

```bash
nc 127.0.0.1 9001
```

এক terminal-এ লিখলে অন্য terminal-এ message দেখা যাবে।

TCP traffic capture:

```bash
sudo tcpdump -ni lo 'tcp port 9001'
```

একইভাবে UDP পরীক্ষা:

Terminal 1:

```bash
nc -u -l 9002
```

Terminal 2:

```bash
nc -u 127.0.0.1 9002
```

UDP traffic capture:

```bash
sudo tcpdump -ni lo 'udp port 9002'
```

কিছু `nc` implementation-এ option-এর syntax আলাদা হতে পারে। `nc -h` দিয়ে নিজের version-এর usage দেখবে।

---

# 8. TCP Three-Way Handshake

TCP connection স্থাপনের আগে সাধারণত তিনটি message exchange হয়।

## 8.1 Workflow

```text
Client                                  Server
  |                                        |
  |-------- SYN, Seq = x ----------------->|
  |                                        |
  |<---- SYN-ACK, Seq = y, Ack = x+1 ------|
  |                                        |
  |-------- ACK, Ack = y+1 --------------->|
  |                                        |
  |========== Connection Established ======|
```

## 8.2 Step-by-step

### Step 1: SYN

Client connection শুরু করার জন্য SYN পাঠায়।

ধরা যাক:

```text
Client ISN = 1000
```

### Step 2: SYN-ACK

Server SYN গ্রহণ করে এবং নিজের initial sequence number পাঠায়।

```text
Server ISN = 5000
Acknowledgement = 1001
```

### Step 3: ACK

Client server-এর SYN acknowledge করে।

```text
Acknowledgement = 5001
```

Connection প্রতিষ্ঠিত হয়।

## 8.3 Acknowledgement Number

TCP Acknowledgement Number হলো পরবর্তী যে sequence number receiver আশা করছে।

সাধারণ ক্ষেত্রে:

```text
ACK = received sequence number + received bytes
```

SYN ও FIN প্রত্যেকে sequence space-এর এক unit ব্যবহার করে, যদিও application data না-ও থাকতে পারে।

উদাহরণ:

```text
Received SYN Seq = 1000
ACK = 1001
```

## 8.4 Kali Lab: Handshake Capture

Terminal 1:

```bash
python3 -m http.server 8000 --bind 127.0.0.1
```

Terminal 2:

```bash
sudo tcpdump -ni lo -nn -S 'tcp port 8000'
```

Terminal 3:

```bash
curl http://127.0.0.1:8000
```

Capture-এ SYN, SYN-ACK এবং ACK packet খুঁজে বের করো।

`-S` option absolute sequence number দেখাতে সাহায্য করে। এটি বাদ দিলে tcpdump relative sequence number দেখাতে পারে।

**Task:** তিনটি packet-এর source port, destination port, flags, sequence number এবং acknowledgement number লিখে রাখো।

---

# 9. TCP Flags and Port Scanning Concepts

TCP header-এর flags connection establishment, termination এবং packet handling-এ ব্যবহৃত হয়।

## 9.1 গুরুত্বপূর্ণ TCP Flags

| Flag | অর্থ | সাধারণ ব্যবহার |
|---|---|---|
| SYN | Synchronize | Connection শুরু |
| ACK | Acknowledgement | Acknowledgement field কার্যকর |
| FIN | Finish | Graceful connection termination |
| RST | Reset | Connection reset |
| PSH | Push | Data receiving application-এ দ্রুত deliver করার ইঙ্গিত |
| URG | Urgent | Urgent pointer প্রাসঙ্গিক |
| ECE | ECN Echo | Congestion notification mechanism |
| CWR | Congestion Window Reduced | ECN congestion response |

তুমি যে `PSK` লিখেছ, সেটি সাধারণ TCP flag নয়। সম্ভবত তুমি `PSH` বোঝাতে চেয়েছ। `PSK` শব্দটি cryptography-তে Pre-Shared Key অর্থেও ব্যবহৃত হয়।

`RTS` সাধারণ TCP flag নয়। এটি context অনুযায়ী Request to Send বা অন্য কোনো acronym হতে পারে। Wi-Fi-তে RTS/CTS mechanism আছে।

## 9.2 SYN, FIN এবং RST-এর পার্থক্য

- SYN: TCP connection establish করতে ব্যবহৃত হয়।
- FIN: Sender-এর দিক থেকে orderly shutdown নির্দেশ করে।
- RST: Connection বা segment reset করতে ব্যবহৃত হয়।

## 9.3 NULL Scan

NULL scan-এ TCP packet-এ কোনো TCP flag সেট করা হয় না।

কিছু TCP/IP implementation-এ closed port RST পাঠাতে পারে, কিন্তু open বা filtered port-এর আচরণ OS ও firewall অনুযায়ী পরিবর্তিত হতে পারে।

## 9.4 XMAS Scan

XMAS scan-এ সাধারণত FIN, PSH এবং URG flags একসঙ্গে সেট করা হয়।

সব operating system একইভাবে respond করে না। তাই ফলাফলকে নিশ্চিত open/closed verdict হিসেবে ব্যবহার করা যাবে না।

## 9.5 SYN Scan

SYN scan connection establishment শুরু করে, কিন্তু সাধারণত full TCP handshake সম্পন্ন করে না।

Nmap-এর `-sS` option SYN scan-এর জন্য ব্যবহৃত হয়। এর জন্য সাধারণত elevated privilege প্রয়োজন হয়।

## 9.6 Kali Lab: নিজের Local Lab Scan

প্রথমে local server চালাও:

```bash
python3 -m http.server 8000 --bind 127.0.0.1
```

দ্বিতীয় terminal-এ:

```bash
nmap -sT -p 8000 127.0.0.1
```

Expected result: TCP port `8000` open হিসেবে রিপোর্ট হতে পারে, কারণ server listening করছে।

এবার এমন একটি unused local port পরীক্ষা করো:

```bash
nmap -sT -p 8001 127.0.0.1
```

Server চালু না থাকলে port সাধারণত closed দেখাবে।

নিজের lab-এ SYN scan:

```bash
sudo nmap -sS -p 8000 127.0.0.1
```

NULL এবং XMAS scan বোঝার জন্য নিজের isolated lab-এ:

```bash
sudo nmap -sN -p 8000 127.0.0.1
sudo nmap -sX -p 8000 127.0.0.1
```

ফলাফল OS এবং local packet filtering-এর ওপর নির্ভর করবে।

অনুমতি ছাড়া অন্যের system-এ scan করবে না।

---

# 10. ICMP and Network Troubleshooting

## 10.1 ICMP কী?

ICMP-এর পূর্ণরূপ Internet Control Message Protocol।

IPv4 network-এ error reporting ও diagnostic communication-এর জন্য ICMP ব্যবহৃত হয়।

উদাহরণ:

- Echo Request
- Echo Reply
- Destination Unreachable
- Time Exceeded
- Fragmentation Needed-related errors

IPv6-এ ICMPv6 ব্যবহৃত হয়, যা Neighbor Discovery-এর মতো গুরুত্বপূর্ণ কাজও করে।

## 10.2 Ping কীভাবে কাজ করে?

সাধারণ IPv4 ping:

```text
ICMP Echo Request → Destination
ICMP Echo Reply   ← Destination
```

## 10.3 Kali Lab

```bash
ping -c 4 127.0.0.1
ping -c 4 192.168.1.1
```

দ্বিতীয় command-এ নিজের gateway-এর IP ব্যবহার করবে।

Network path পরীক্ষা:

```bash
traceroute 1.1.1.1
```

Command অনুপস্থিত থাকলে প্রয়োজনীয় package install করতে হতে পারে। কিছু network ICMP বা traceroute probe filter করে, তাই incomplete output অস্বাভাবিক নয়।

---

# 11. DHCP and NAT

## 11.1 DHCP

DHCP-এর পূর্ণরূপ Dynamic Host Configuration Protocol।

DHCPv4 client-কে IP address এবং অন্যান্য network configuration দিতে পারে।

### DORA workflow

```text
Client                    DHCP Server
  |                            |
  |-------- Discover --------->|
  |<--------- Offer -----------|
  |-------- Request ---------->|
  |<---------- ACK ------------|
```

DORA:

- Discover
- Offer
- Request
- Acknowledgement

বাস্তব network-এ DHCP relay, renew, rebinding এবং অন্যান্য message exchange-ও থাকতে পারে।

### Practical scenario

তুমি Wi-Fi-তে connect করলে router বা DHCP server তোমাকে IP address, subnet mask, default gateway এবং DNS server দিতে পারে।

### Kali Lab

```bash
ip -4 addr show
ip route
cat /etc/resolv.conf
```

এগুলো দিয়ে বর্তমান configuration পরীক্ষা করো।

নিজের active interface-এ DHCP lease renew করতে হলে আগে interface name এবং network management service শনাক্ত করো। Remote SSH connection বা গুরুত্বপূর্ণ network interface-এ অনুমতি ছাড়া lease renew করলে connectivity বিচ্ছিন্ন হতে পারে।

DHCP traffic capture:

```bash
sudo tcpdump -ni any 'udp port 67 or udp port 68'
```

এরপর নিজের lab network-এ DHCP renew করলে traffic দেখা যেতে পারে। DHCP server ও network activity-এর ওপর ফলাফল নির্ভর করবে।

## 11.2 NAT

NAT-এর পূর্ণরূপ Network Address Translation।

NAT packet-এর IP address এবং কিছু ক্ষেত্রে port number পরিবর্তন করে।

### SNAT

Source Network Address Translation।

সাধারণত outgoing traffic-এর source address পরিবর্তন করতে ব্যবহৃত হয়।

### DNAT

Destination Network Address Translation।

Incoming traffic-এর destination address পরিবর্তন করতে ব্যবহৃত হয়।

### PAT

Port Address Translation, যা অনেক home router-এ একাধিক private host-এর connection public address দিয়ে পরিচালনা করতে সাহায্য করে।

### Scenario

```text
Laptop: 192.168.1.10
Router WAN IP: 203.0.113.10
Web server: 93.184.216.34
```

Laptop থেকে outgoing connection করার সময় router source address এবং source port translate করতে পারে।

`203.0.113.0/24` documentation-এর জন্য reserved, তাই এটি বাস্তব public IP নয়, example মাত্র।

### Kali Lab

```bash
ip route
```

তারপর নিজের router-এর configuration interface-এ NAT/PAT settings পর্যবেক্ষণ করো। শুধু নিজের network বা অনুমোদিত lab-এ পরিবর্তন করবে।

---

# 12. OSI Model: Workflow and Mechanism

OSI Model network communication-কে সাতটি conceptual layer-এ ভাগ করে।

## 12.1 Seven Layers

| Layer | Name | প্রধান কাজ | Example |
|---:|---|---|---|
| 7 | Application | Application-level network service | HTTP, DNS |
| 6 | Presentation | Data representation, encoding, encryption concepts | TLS-related data representation |
| 5 | Session | Session management concepts | Session establishment/management |
| 4 | Transport | End-to-end transport | TCP, UDP |
| 3 | Network | Logical addressing ও routing | IPv4, IPv6 |
| 2 | Data Link | Local frame delivery | Ethernet, Wi-Fi MAC |
| 1 | Physical | Signals ও physical transmission | Copper, fiber, radio |

বাস্তব Internet stack-এর protocol-গুলো OSI layer-এর সঙ্গে সবসময় এক-একভাবে মেলে না। OSI মূলত বোঝার জন্য একটি model।

## 12.2 Encapsulation Workflow

Browser থেকে web request পাঠানোর সময়:

```text
Application Data
       ↓
Transport Header + Data
       ↓
IP Header + Transport Segment
       ↓
Ethernet Header + IP Packet + Trailer
       ↓
Physical Signals
```

Receiving side-এ:

```text
Physical Signals
       ↓
Ethernet Frame
       ↓
IP Packet
       ↓
TCP Segment / UDP Datagram
       ↓
Application Data
```

## 12.3 Real-life Scenario

তুমি HTTPS website open করলে:

- Application layer: HTTP request/response।
- Transport layer: TCP, যদি HTTP/1.1 বা HTTP/2 over TCP হয়।
- Network layer: IP addressing ও routing।
- Data-link layer: Ethernet বা Wi-Fi frame।
- Physical layer: Cable বা radio signal।

HTTP/3 সাধারণত QUIC-এর মাধ্যমে UDP ব্যবহার করে, তাই সব HTTPS connection-এ TCP থাকে না।

## 12.4 Kali Lab: OSI Layer Analysis

একটি HTTP server চালাও:

```bash
python3 -m http.server 8000 --bind 127.0.0.1
```

অন্য terminal-এ:

```bash
curl -v http://127.0.0.1:8000
```

Capture:

```bash
sudo tcpdump -ni lo -nn -X 'tcp port 8000'
```

যা identify করবে:

1. IP source এবং destination।
2. TCP source ও destination port।
3. TCP flags।
4. HTTP request text।
5. Frame-এর পরিবর্তে loopback capture হওয়ায় Ethernet header নাও থাকতে পারে।

**Task:** OSI-এর প্রতিটি layer-এ এই communication-এর সঙ্গে সম্পর্কিত কোন কাজ হচ্ছে তা লিখে রাখো।

---

# 13. Physical and Network Devices

## 13.1 Router

Router এক IP network থেকে অন্য network-এ packet forward করে।

মূল mechanism:

1. Incoming packet গ্রহণ করে।
2. Destination IP দেখে।
3. Routing table-এ best matching route খুঁজে।
4. Next-hop ও outgoing interface নির্বাচন করে।
5. Link-layer encapsulation করে packet forward করে।

Kali-তে routing table:

```bash
ip route
```

একটি remote destination-এর route দেখতে:

```bash
ip route get 1.1.1.1
```

এটি Linux kernel-এর নির্বাচিত route সম্পর্কে তথ্য দেয়।

## 13.2 Switch

সাধারণ Layer 2 switch MAC address table ব্যবহার করে Ethernet frame forward করে।

যদি destination MAC জানা থাকে, switch সংশ্লিষ্ট port-এ frame পাঠাতে পারে।

Destination MAC অজানা হলে সাধারণত সংশ্লিষ্ট VLAN-এর অন্য port-গুলোতে unknown-unicast flooding হতে পারে।

Broadcast frame সাধারণত VLAN-এর মধ্যে flood হয়।

Managed switch-এ VLAN, port security, monitoring ও অন্যান্য feature থাকতে পারে।

## 13.3 Server

Server একটি service প্রদানকারী host বা software system।

উদাহরণ:

- Web server
- Database server
- DNS server
- File server
- Authentication server

Server মানেই dedicated physical machine নয়। Virtual machine, container বা cloud instance-ও server হিসেবে কাজ করতে পারে।

## 13.4 Network Adapter

Network Interface Card বা NIC computer-কে wired বা wireless network-এ যুক্ত করে।

NIC-এর কাজ:

- Frame send/receive করা।
- MAC addressing।
- Hardware-level processing।
- কিছু ক্ষেত্রে checksum offload ও segmentation offload।

Linux-এ:

```bash
ip link
ethtool eth0
```

`ethtool` না থাকলে installation প্রয়োজন হতে পারে। `eth0`-এর জায়গায় নিজের interface ব্যবহার করবে।

## 13.5 Wireless Adapter

Wireless adapter radio signal ব্যবহার করে Wi-Fi network-এর সঙ্গে communicate করে।

Wi-Fi adapter-এর capability নির্ভর করে hardware, driver, firmware, regulatory domain এবং operating mode-এর ওপর।

দেখো:

```bash
iw dev
iw list
```

`iw list` থেকে supported bands ও interface modes সম্পর্কে তথ্য পেতে পারো।

সব adapter monitor mode বা packet injection support করে না।

## 13.6 Device Lab

নিজের Kali VM-এ পরীক্ষা করো:

```bash
ip -br link
ip -br addr
ip route
ip neigh
```

প্রতিটি command-এর output থেকে device, address, route ও neighbor relationship শনাক্ত করো।

একটি গুরুত্বপূর্ণ পার্থক্য মনে রাখবে: router এবং switch-এর forwarding mechanism আলাদা। কিছু physical device একই hardware-এ routing ও switching দুটোই করতে পারে।

---

# 14. Wireless Networking and Wi-Fi Security

## 14.1 Wi-Fi কীভাবে কাজ করে?

Wi-Fi হলো IEEE 802.11 family-এর wireless networking technology।

একটি সাধারণ infrastructure network-এ:

1. Access Point বা wireless router beacon পাঠাতে পারে।
2. Client network discover করে।
3. Client authentication এবং association প্রক্রিয়ায় অংশ নেয়।
4. Security configuration অনুযায়ী key establishment হয়।
5. Client network traffic পাঠাতে শুরু করে।

## 14.2 WEP

WEP-এর পূর্ণরূপ Wired Equivalent Privacy।

WEP পুরোনো wireless security mechanism এবং এতে গুরুতর cryptographic দুর্বলতা রয়েছে।

**WEP ব্যবহার করে নতুন network তৈরি করবে না।**

## 14.3 WPA

WPA-এর পূর্ণরূপ Wi-Fi Protected Access।

এটি WEP-এর দুর্বলতা মোকাবিলায় transitional security mechanism হিসেবে তৈরি হয়েছিল। WPA-র পুরোনো implementation এখন নিরাপদ নতুন deployment-এর জন্য উপযুক্ত নয়।

## 14.4 WPA2

WPA2 সাধারণত AES-CCMP-ভিত্তিক security ব্যবহার করে।

WPA2-Personal-এ pre-shared passphrase ব্যবহৃত হতে পারে।

দুর্বল passphrase offline password guessing-এর ঝুঁকি বাড়াতে পারে, যদি attacker প্রয়োজনীয় handshake material পায়।

## 14.5 WPA3

WPA3-Personal SAE ব্যবহার করে, যা পুরোনো WPA2-Personal PSK handshake model থেকে আলাদা।

WPA3-ও সঠিক configuration, শক্তিশালী credential, updated firmware এবং নিরাপদ client implementation-এর ওপর নির্ভরশীল।

## 14.6 PSK কী?

PSK-এর পূর্ণরূপ Pre-Shared Key।

এটি এমন secret key বা credential, যা সংশ্লিষ্ট পক্ষগুলো আগে থেকেই share করে।

Wi-Fi context-এ PSK সাধারণত WPA2-Personal বা WPA-Personal security configuration বোঝাতে ব্যবহৃত হয়।

PSK হলো security credential-এর ধারণা, TCP flag নয়।

## 14.7 Wi-Fi Security Comparison

| Security | মূল ধারণা | সুপারিশ |
|---|---|---|
| WEP | পুরোনো দুর্বল encryption | ব্যবহার করবে না |
| WPA | Transitional legacy security | নতুন network-এর জন্য নয় |
| WPA2 | সাধারণত AES-CCMP | সঠিক configuration-এ এখনও ব্যবহৃত |
| WPA3 | আধুনিক authentication ও security improvements | সমর্থিত হলে উপযুক্ত পছন্দ |

## 14.8 Kali Lab: Wi-Fi Security Audit

নিজের Wi-Fi network-এর security configuration পরীক্ষা করো।

```bash
iw dev
nmcli device status
nmcli connection show
```

Wi-Fi interface ও connection-এর তথ্য দেখো। Password বা অন্যান্য secret প্রকাশ করে এমন output অন্য কারও সঙ্গে share করবে না।

Access Point-এর admin interface থেকে যাচাই করো:

- WEP disabled।
- WPA/TKIP legacy configuration এড়িয়ে চলা।
- WPA2-AES অথবা WPA3 ব্যবহার।
- শক্তিশালী, unique passphrase।
- Router firmware updated।
- WPS অপ্রয়োজনীয় হলে disabled।
- Guest network আলাদা রাখা।
- Router management interface Internet থেকে exposed নয়।

Wi-Fi security testing শুধু নিজের network বা লিখিত অনুমোদন থাকা lab-এ করবে।

---

# 15. Practical Packet Analysis with Wireshark and tcpdump

## 15.1 Wireshark কী?

Wireshark হলো graphical packet analyzer।

এটি packet-এর protocol, address, port, TCP flags, DNS query এবং অন্যান্য field inspect করতে সাহায্য করে।

## 15.2 tcpdump কী?

tcpdump হলো command-line packet capture tool।

Headless system, remote server এবং দ্রুত diagnostic কাজের জন্য এটি বিশেষভাবে উপযোগী।

## 15.3 Useful tcpdump Commands

ICMP:

```bash
sudo tcpdump -ni any icmp
```

DNS:

```bash
sudo tcpdump -ni any 'port 53'
```

HTTP:

```bash
sudo tcpdump -ni any 'tcp port 80'
```

HTTPS:

```bash
sudo tcpdump -ni any 'tcp port 443'
```

TCP SYN packets:

```bash
sudo tcpdump -ni any 'tcp[tcpflags] & tcp-syn != 0'
```

PCAP file-এ capture:

```bash
sudo tcpdump -ni any -w network-lab.pcap
```

Capture বন্ধ করতে `Ctrl+C` চাপো।

Saved capture পড়ো:

```bash
tcpdump -nn -r network-lab.pcap
```

## 15.4 Wireshark Display Filters

| Filter | কী দেখাবে |
|---|---|
| `ip` | IPv4 packets |
| `ipv6` | IPv6 packets |
| `arp` | ARP traffic |
| `icmp` | ICMPv4 traffic |
| `icmpv6` | ICMPv6 traffic |
| `tcp` | TCP packets |
| `udp` | UDP packets |
| `dns` | DNS traffic |
| `tcp.flags.syn == 1` | SYN flag সেট করা TCP packets |
| `tcp.flags.reset == 1` | RST flag সেট করা TCP packets |
| `tcp.port == 8000` | TCP port 8000-সংশ্লিষ্ট traffic |

Capture filter এবং display filter এক জিনিস নয়। Capture filter packet capture করার সময় traffic নির্বাচন করে, আর display filter captured packet-এর দৃশ্যমান তালিকা filter করে।

## 15.5 Practical Analysis Task

নিজের local HTTP server চালাও এবং request পাঠাও:

```bash
python3 -m http.server 8000 --bind 127.0.0.1
```

অন্য terminal-এ:

```bash
curl http://127.0.0.1:8000
```

Wireshark-এ loopback interface select করে:

```text
tcp.port == 8000
```

তারপর বিশ্লেষণ করো:

- SYN packet।
- SYN-ACK packet।
- ACK packet।
- HTTP request।
- HTTP response।
- Connection termination।

---

# 16. Integrated Kali Linux Lab

## Lab Objective

একটি local web service তৈরি করে IP, TCP, port, handshake, HTTP এবং packet analysis একসঙ্গে অনুশীলন করা।

## Step 1: Lab Directory

```bash
mkdir -p ~/network-lab
cd ~/network-lab
echo "Networking Lab Successful" > index.html
```

## Step 2: Start Server

```bash
python3 -m http.server 8000 --bind 127.0.0.1
```

Server loopback-এ bind করায় এটি শুধু একই machine থেকে accessible হবে।

## Step 3: Verify Service

অন্য terminal:

```bash
curl -v http://127.0.0.1:8000
```

Expected response body:

```text
Networking Lab Successful
```

## Step 4: Check Listening Port

```bash
ss -lntp
```

Port `8000` listening অবস্থায় আছে কি না দেখো।

## Step 5: Scan the Port

```bash
nmap -sT -p 8000 127.0.0.1
```

Expected state:

```text
8000/tcp open
```

## Step 6: Capture Traffic

```bash
sudo tcpdump -ni lo -nn -S 'tcp port 8000'
```

আবার `curl` চালাও।

## Step 7: Analyze

Capture থেকে নিচের তথ্য লিখে রাখো:

```text
Source IP:
Destination IP:
Source Port:
Destination Port:
TCP Flags:
Sequence Number:
Acknowledgement Number:
HTTP Method:
HTTP Response:
```

## Step 8: Stop the Server

Server terminal-এ:

```text
Ctrl+C
```

এরপর scan আবার চালাও:

```bash
nmap -sT -p 8000 127.0.0.1
```

Server বন্ধ থাকায় port সাধারণত `closed` দেখাবে।

**Learning result:** একই machine-এ তুমি একটি service start/stop করে listening port এবং TCP connection behavior পর্যবেক্ষণ করতে পারবে।

---

# 17. Cybersecurity Troubleshooting Scenarios

## Scenario 1: Website Open হচ্ছে না

সম্ভাব্য কারণ:

- DNS resolution failure।
- Incorrect route।
- TCP port blocked।
- Server unavailable।
- TLS error।
- Proxy বা firewall configuration।

Commands:

```bash
ip route
getent hosts example.com
curl -v --connect-timeout 5 https://example.com
```

একটি failure দেখেই কারণ নিশ্চিত করবে না। DNS, routing, TCP connection এবং TLS negotiation আলাদা করে পরীক্ষা করবে।

## Scenario 2: Ping কাজ করছে না

সম্ভাব্য কারণ:

- Destination ICMP block করছে।
- Firewall request বা reply drop করছে।
- Routing সমস্যা।
- Host unavailable।

TCP service accessible হতে পারে, যদিও ping reply না আসে।

## Scenario 3: Port Open দেখাচ্ছে না

Commands:

```bash
ss -lntup
nmap -sT -p 8000 127.0.0.1
```

যাচাই করো:

- Service চলছে কি না।
- সঠিক interface-এ bind করেছে কি না।
- Firewall traffic block করছে কি না।
- তুমি সঠিক host scan করছ কি না।

## Scenario 4: Wrong Subnet Configuration

উদাহরণ:

```text
Host A: 192.168.1.10/24
Host B: 192.168.2.20/24
```

সঠিক routing ছাড়া host দুটির মধ্যে direct local communication সাধারণত হবে না।

সমাধান খুঁজতে IP, prefix, gateway এবং routing table পরীক্ষা করো।

## Scenario 5: ARP Resolution Failure

যদি একই subnet-এর destination-এর MAC address resolve না হয়:

```bash
ip neigh
sudo tcpdump -ni eth0 arp
```

সম্ভাব্য কারণ:

- Host offline।
- VLAN mismatch।
- Wrong subnet।
- Wi-Fi client isolation।
- Layer 2 connectivity সমস্যা।

নিজের interface অনুযায়ী `eth0` পরিবর্তন করবে।

---

# 18. Practice Questions

## Beginner Level

1. IP address এবং MAC address-এর পার্থক্য কী?
2. Packet এবং frame-এর পার্থক্য কী?
3. TCP ও UDP কীভাবে আলাদা?
4. Port number কেন প্রয়োজন?
5. ARP কী কাজ করে?
6. Router এবং switch-এর প্রধান পার্থক্য কী?
7. TTL কেন প্রয়োজন?
8. IHL-এর minimum value কত?
9. DHCP-এর DORA workflow ব্যাখ্যা করো।
10. TCP three-way handshake-এর তিনটি step কী?

## Intermediate Level

11. `192.168.10.70/26`-এর network ও broadcast address বের করো।
12. TCP ACK number `1001` কী বোঝাতে পারে?
13. IP fragmentation এবং TCP segmentation-এর পার্থক্য ব্যাখ্যা করো।
14. একই subnet-এ destination MAC কীভাবে নির্ধারিত হয়?
15. Remote destination-এর জন্য gateway MAC কেন ব্যবহৃত হয়?
16. `SYN`, `FIN` এবং `RST`-এর পার্থক্য কী?
17. WEP কেন নিরাপদ নয়?
18. IPv4 Header Checksum এবং TCP Checksum কীভাবে আলাদা?
19. NAT ও DHCP-এর কাজ কীভাবে আলাদা?
20. কেন `ping` fail করলেও HTTPS কাজ করতে পারে?

---

# 19. Final Lab Checklist

প্রতিটি কাজ সম্পন্ন হলে checkbox-এ mark করো।

- [ ] Kali Linux-এ IP address এবং interface identify করেছি।
- [ ] Routing table পড়তে পেরেছি।
- [ ] MAC address এবং ARP entry দেখেছি।
- [ ] IPv4 header-এর গুরুত্বপূর্ণ field ব্যাখ্যা করতে পারি।
- [ ] CIDR এবং subnetting solve করতে পারি।
- [ ] TCP ও UDP traffic capture করেছি।
- [ ] TCP three-way handshake শনাক্ত করেছি।
- [ ] Sequence number এবং acknowledgement number ব্যাখ্যা করতে পারি।
- [ ] TCP flags inspect করেছি।
- [ ] ICMP এবং traceroute পরীক্ষা করেছি।
- [ ] DHCP configuration inspect করেছি।
- [ ] Local HTTP server তৈরি করেছি।
- [ ] Listening port এবং Nmap result মিলিয়ে দেখেছি।
- [ ] Wireshark বা tcpdump দিয়ে PCAP বিশ্লেষণ করেছি।
- [ ] WPA2/WPA3 security configuration-এর পার্থক্য বুঝেছি।
- [ ] অন্তত পাঁচটি networking troubleshooting scenario সমাধান করেছি।

---

# 20. Recommended Learning Resources

1. **Kali Linux Documentation**  
   https://www.kali.org/docs/

2. **Wireshark Documentation**  
   https://www.wireshark.org/docs/

3. **Nmap Reference Guide**  
   https://nmap.org/book/man.html

4. **RFC 791: Internet Protocol (IPv4)**  
   https://www.rfc-editor.org/rfc/rfc791

5. **RFC 9293: Transmission Control Protocol (TCP)**  
   https://www.rfc-editor.org/rfc/rfc9293

6. **RFC 826: Address Resolution Protocol (ARP)**  
   https://www.rfc-editor.org/rfc/rfc826

7. **RFC 2131: Dynamic Host Configuration Protocol (DHCP)**  
   https://www.rfc-editor.org/rfc/rfc2131

8. **RFC 4632: Classless Inter-Domain Routing (CIDR)**  
   https://www.rfc-editor.org/rfc/rfc4632

---

# Final Advice from Your Senior Engineer

Cybersecurity শেখার সময় command মুখস্থ করার চেয়ে command-এর output বোঝা বেশি গুরুত্বপূর্ণ।

প্রতিটি lab-এ তিনটি প্রশ্ন করবে:

1. **What happened?** কী ঘটেছে?
2. **Why did it happen?** কেন ঘটেছে?
3. **How can I verify it?** কীভাবে প্রমাণ করব?

প্রথমে নিজের Kali Linux machine এবং isolated virtual lab-এ অনুশীলন করো। তারপর Wireshark, routing, firewall এবং service configuration-এর সঙ্গে কাজ করো। Networking-এর foundation ভালো হলে Web Pentesting, API Security, Network Pentesting এবং Incident Response শেখা অনেক সহজ হবে।

**তোমার প্রথম লক্ষ্য:** একটি local TCP service চালানো, তার port scan করা এবং Wireshark বা tcpdump দিয়ে complete TCP handshake শনাক্ত করা।
