# TryHackMe: Wi-Fi Fundamentals and Reconnaissance

## Complete Practical Notes and Command Reference

**Language:** সহজ বাংলা
**Level:** Beginner
**Environment:** TryHackMe Linux lab
**Purpose:** Learning, revision, and authorized Wi-Fi security assessment

Official room: [https://tryhackme.com/room/wififundamentalsandrecon](https://tryhackme.com/room/wififundamentalsandrecon)

---

# Table of Contents

1. Room Overview
2. Task 1 — Introduction and Lab Setup
3. Task 2 — How Wi-Fi Works
4. Task 3 — 802.11 Frames
5. Task 4 — Monitor Mode
6. Task 5 — Scanning the Airwaves
7. Task 6 — Uncovering Hidden Networks
8. Task 7 — Organising Your Recon
9. Task 8 — Conclusion
10. Troubleshooting
11. Quick Revision Sheet
12. Task Answers

---

# 1. Room Overview

এই room-এ আমরা শিখব কীভাবে wireless network সম্পর্কে তথ্য সংগ্রহ করতে হয় এবং সেই তথ্য থেকে network-এর configuration বুঝতে হয়।

এই প্রক্রিয়াকে **Wireless Reconnaissance** বলে।

একটি real-world scenario কল্পনা করো: একটি প্রতিষ্ঠান তোমাকে তাদের Wi-Fi security audit করার অনুমতি দিয়েছে। তোমার প্রথম কাজ হলো তাদের access point, channel, encryption, authentication এবং connected client সম্পর্কে তথ্য সংগ্রহ করা।

এই room-এর workflow:

1. Wireless interface শনাক্ত করা।
2. Wi-Fi-এর মৌলিক ধারণা বোঝা।
3. Monitor mode চালু করা।
4. 2.4 GHz ও 5 GHz network scan করা।
5. Network security configuration বিশ্লেষণ করা।
6. Lab-এর hidden SSID শনাক্ত করা।
7. Capture file ও inventory সংরক্ষণ করা।

**Lab safety:** নিচের active probing, network connection ও capture command কেবল নিজের lab অথবা স্পষ্ট অনুমোদিত environment-এ ব্যবহার করবে। এই room software-simulated wireless radio ব্যবহার করে।

---

# 2. Task 1 — Introduction and Lab Setup

## 2.1 Lab শুরু করা

TryHackMe room খুলে **Start Machine** চাপো। Machine চালু হলে browser-এর terminal ব্যবহার করো অথবা room-এর নির্দেশনা অনুযায়ী SSH দিয়ে connect করো।

Lab-এর terminal prompt এমন হতে পারে:

```bash
user@ip-10-49-169-221:~$
```

Prompt-এর IP প্রতিবার এক নাও থাকতে পারে।

## 2.2 Wireless interface দেখা

Command:

```bash
sudo iw dev
```

### ব্যাখ্যা

- `sudo` — administrator privilege দিয়ে command চালায়।
- `iw` — Linux wireless device configuration ও information দেখার tool।
- `dev` — wireless interface সম্পর্কে তথ্য দেখায়।

Output-এ `wlan0`, `wlan1`, `wlan2` ইত্যাদি দেখতে পারো।

একটি interface-এর তথ্য:

```text
Interface wlan0
    ifindex 7
    addr 02:00:00:00:00:00
    type managed
    wiphy 2
```

গুরুত্বপূর্ণ field:

| Field          | অর্থ                                          |
| -------------- | --------------------------------------------- |
| `Interface`    | Wireless interface-এর নাম                     |
| `addr`         | Interface-এর MAC address                      |
| `type managed` | সাধারণ Wi-Fi client mode                      |
| `wiphy`        | Kernel-এর wireless physical-device identifier |

এই lab-এ অনেক interface দেখা যেতে পারে, কারণ radio-গুলো software-এ simulate করা হয়েছে।

## 2.3 Interface connected কি না পরীক্ষা করা

```bash
sudo iw dev wlan0 link
```

যদি output হয়:

```text
Not connected.
```

তাহলে interface বর্তমানে কোনো access point-এর সঙ্গে connected নয়।

## 2.4 Supported interface modes দেখা

```bash
iw list | grep -A 12 "Supported interface modes"
```

সমর্থিত mode-এর মধ্যে থাকতে পারে:

- `managed`
- `AP`
- `monitor`

এগুলোর কাজ:

- **Managed mode:** সাধারণত Wi-Fi client হিসেবে connect করা।
- **AP mode:** Access Point হিসেবে কাজ করা।
- **Monitor mode:** Wireless frame পর্যবেক্ষণ করা।

---

# 3. Task 2 — How Wi-Fi Works

Wi-Fi হলো IEEE 802.11 standard family-এর ওপর ভিত্তি করে তৈরি wireless networking technology।

## 3.1 গুরুত্বপূর্ণ technical terms

### Access Point (AP)

Access Point হলো সেই device, যার মাধ্যমে wireless client network-এ যুক্ত হয়।

বাস্তব উদাহরণ: অফিসের ceiling-এ লাগানো Wi-Fi device।

### Station (STA)

যে device Wi-Fi network ব্যবহার করে, সেটি Station হতে পারে।

উদাহরণ:

- Laptop
- Smartphone
- Printer
- IoT device

### SSID এবং ESSID

SSID হলো Wi-Fi network-এর নাম।

উদাহরণ:

```text
CorpNet
Home-2G
Guest-WiFi
```

একই network name একাধিক access point ব্যবহার করতে পারে। সেই ক্ষেত্রে ESSID সাধারণত পুরো wireless network-এর নাম বোঝায়।

### BSSID

BSSID সাধারণত একটি নির্দিষ্ট access point-এর radio MAC address।

উদাহরণ:

```text
F0:9F:C2:71:22:15
```

একই `CorpNet` নামের দুটি access point থাকলে তাদের BSSID আলাদা হতে পারে।

### Channel

Wi-Fi radio নির্দিষ্ট channel-এ কাজ করে।

উদাহরণ:

- 2.4 GHz: Channel 1, 6, 11
- 5 GHz: Channel 44

Channel number নিজে frequency নয়; এটি নির্দিষ্ট frequency range চিহ্নিত করে।

### Frequency bands

- **2.4 GHz:** তুলনামূলক বেশি range, তবে interference বেশি হতে পারে।
- **5 GHz:** অনেক ক্ষেত্রে বেশি capacity এবং কম congestion।
- **6 GHz:** সমর্থিত Wi-Fi 6E/7 equipment-এ ব্যবহৃত হয়।

সব device সব band সমর্থন করে না।

## 3.2 BSS কী?

**Basic Service Set (BSS)** হলো একটি access point এবং তার সঙ্গে associated wireless station-গুলোর group।

সহজ উদাহরণ: একটি অফিসের নির্দিষ্ট access point এবং তার সঙ্গে connected laptop ও phone-গুলো।

---

# 4. Task 3 — 802.11 Frames

Wireless device-গুলোর মধ্যে communication হয় 802.11 frame-এর মাধ্যমে।

Frame-এর তিনটি প্রধান category রয়েছে।

## 4.1 Management frames

Network খোঁজা, authentication, association এবং connection management-এ ব্যবহৃত হয়।

গুরুত্বপূর্ণ উদাহরণ:

- **Beacon:** Access Point নিয়মিত broadcast করে নিজের network-এর উপস্থিতি জানায়।
- **Probe Request:** Client সক্রিয়ভাবে network খোঁজার জন্য পাঠায়।
- **Probe Response:** Access Point উপযুক্ত probe-এর উত্তর দেয়।
- **Authentication:** Connection establishment-এর authentication stage।
- **Association:** Client-কে access point-এর সঙ্গে যুক্ত করার stage।
- **Deauthentication:** একটি security association শেষ করার frame।

### Beacon বনাম Probe Request

| Beacon                        | Probe Request                          |
| ----------------------------- | -------------------------------------- |
| Access Point পাঠায়           | Client পাঠায়                          |
| Network-এর উপস্থিতি জানায়    | Network খোঁজে                          |
| সাধারণত নিয়মিত broadcast হয় | Client-এর activity অনুযায়ী পাঠানো হয় |

## 4.2 Control frames

Wireless medium ব্যবহার নিয়ন্ত্রণে সাহায্য করে।

উদাহরণ:

- RTS — Request to Send
- CTS — Clear to Send
- ACK — Acknowledgement

## 4.3 Data frames

Network-এর মূল data বহন করে।

Encrypted network-এ payload সাধারণত protected থাকে। তবে packet-এর কিছু header তথ্য পর্যবেক্ষণ করা যেতে পারে।

## 4.4 কেন frame জানা দরকার?

Beacon দেখে access point শনাক্ত করা যায়। Probe Request থেকে client কোন network খুঁজছে, তা জানা যেতে পারে।

তাই wireless reconnaissance-এ frame-এর ধরন বোঝা গুরুত্বপূর্ণ।

---

# 5. Task 4 — Monitor Mode

## 5.1 Managed mode বনাম Monitor mode

Managed mode-এ interface সাধারণত একটি access point-এর সঙ্গে client হিসেবে কাজ করে।

Monitor mode-এ interface তার বর্তমান channel-এ receive করা 802.11 frame পর্যবেক্ষণ করতে পারে, এমনকি frame অন্য device-এর জন্য হলেও।

**মনে রাখবে:** Monitor mode সব channel-এর traffic একই সঙ্গে ধরে না। Radio সাধারণত এক সময়ে একটি channel-এ থাকে।

## 5.2 Interfering process পরীক্ষা করা

```bash
sudo airmon-ng check
```

এটি monitor mode-এ বাধা দিতে পারে এমন process দেখায়।

Lab-এর নির্দেশনা অনুযায়ী সেগুলো বন্ধ করতে:

```bash
sudo airmon-ng check kill
```

এই command NetworkManager বা `wpa_supplicant`-এর মতো process বন্ধ করতে পারে। তাই নিজের সাধারণ computer-এ চালালে Wi-Fi connection বিচ্ছিন্ন হতে পারে।

## 5.3 Monitor mode চালু করা

```bash
sudo airmon-ng start wlan0
```

Lab-এ output এমন হতে পারে:

```text
monitor mode enabled
wlan0mon
```

এখানে `wlan0mon` হলো monitor-mode interface-এর নাম।

## 5.4 Mode যাচাই করা

```bash
sudo iw dev
```

Monitor mode সক্রিয় হলে এমন output দেখতে পারো:

```text
Interface wlan0mon
    type monitor
    channel 1 (2412 MHz)
```

`type monitor` হলো গুরুত্বপূর্ণ confirmation।

## 5.5 Monitor mode বন্ধ করা

Lab শেষে normal operation ফিরিয়ে আনতে:

```bash
sudo airmon-ng stop wlan0mon
sudo systemctl start NetworkManager
```

এটি lab-এর network setup-এর ওপর নির্ভর করে। সব environment-এ interface একই নামে ফিরে আসবে এমন নিশ্চয়তা নেই।

---

# 6. Task 5 — Scanning the Airwaves

এই task-এ `airodump-ng` দিয়ে Wi-Fi network ও client সম্পর্কে তথ্য সংগ্রহ করা হয়।

## 6.1 2.4 GHz band scan করা

```bash
sudo airodump-ng --band bg wlan0mon
```

ব্যাখ্যা:

- `sudo` — elevated privilege।
- `airodump-ng` — wireless network ও frame পর্যবেক্ষণ করে।
- `--band bg` — 2.4 GHz-এর b/g-compatible band scan করে।
- `wlan0mon` — monitor-mode interface।

Scan 30–60 সেকেন্ড চালালে বিভিন্ন channel থেকে আরও তথ্য পাওয়া যেতে পারে।

বন্ধ করতে:

```text
Ctrl+C
```

## 6.2 5 GHz network scan করা

Lab-এর enterprise network-গুলো Channel 44-এ রয়েছে।

```bash
sudo airodump-ng --band a -c 44 wlan0mon
```

ব্যাখ্যা:

- `--band a` — 5 GHz band-এর scan।
- `-c 44` — Channel 44-এ scan সীমাবদ্ধ করে।
- `wlan0mon` — monitor interface।

এটি channel 44-এ কাজ করা network-গুলো দেখাবে।

## 6.3 একটি নির্দিষ্ট BSSID scan করা

```bash
sudo airodump-ng -c 44 \
  --bssid F0:9F:C2:71:22:15 \
  -w corpnet-capture wlan0mon
```

ব্যাখ্যা:

| Option               | কাজ                             |
| -------------------- | ------------------------------- |
| `-c 44`              | Channel 44 নির্বাচন             |
| `--bssid ...`        | নির্দিষ্ট AP-তে filter          |
| `-w corpnet-capture` | Output files-এর filename prefix |
| `wlan0mon`           | Monitor interface               |

Lab-এ output file-এর নাম হতে পারে:

```text
corpnet-capture-01.cap
corpnet-capture-01.csv
```

Suffix এবং file set run অনুযায়ী পরিবর্তিত হতে পারে।

## 6.4 `airodump-ng` output পড়া

সাধারণত উপরের section-এ Access Point এবং নিচের section-এ Station বা client-এর তথ্য দেখা যায়।

### Access Point columns

| Column    | অর্থ                                         |
| --------- | -------------------------------------------- |
| `BSSID`   | Access Point-এর radio MAC address            |
| `PWR`     | Signal strength, dBm-এ                       |
| `RXQ`     | নির্দিষ্ট পরিস্থিতিতে receive quality        |
| `Beacons` | Capture চলাকালীন দেখা beacon frame-এর সংখ্যা |
| `#Data`   | Captured data frame-এর সংখ্যা                |
| `#/s`     | সাম্প্রতিক data-frame rate                   |
| `CH`      | Channel                                      |
| `MB`      | Advertised maximum speed information         |
| `ENC`     | Encryption/security family                   |
| `CIPHER`  | ব্যবহৃত cipher                               |
| `AUTH`    | Authentication type                          |
| `ESSID`   | Network name                                 |

### Signal strength

`PWR`-এর মান সাধারণত শূন্যের যত কাছাকাছি, signal তত শক্তিশালী।

উদাহরণ:

- `-30 dBm` — শক্তিশালী signal
- `-70 dBm` — তুলনামূলক দুর্বল signal

এগুলো আনুমানিক উদাহরণ; actual result environment অনুযায়ী বদলায়।

### Security columns

| Output         | সাধারণ অর্থ                                                         |
| -------------- | ------------------------------------------------------------------- |
| `OPN`          | Open network                                                        |
| `WEP`          | দুর্বল ও obsolete security                                          |
| `WPA2` + `PSK` | WPA2-Personal                                                       |
| `WPA3` + `SAE` | WPA3-Personal                                                       |
| `MGT`          | Enterprise/802.1X-style authentication                              |
| `OWE`          | Open-style network-এ encryption, shared password authentication নয় |
| `CCMP`         | AES-based modern cipher                                             |
| `TKIP`         | পুরোনো cipher                                                       |

`MGT` দেখালে সাধারণত enterprise authentication configuration বোঝায়। এটি কোনো নির্দিষ্ট username বা password প্রকাশ করে না।

## 6.5 একই ESSID-এর একাধিক AP

Lab-এ `CorpNet`-এর উদাহরণ:

```text
F0:9F:C2:71:22:15  ...  44 ... MGT  CorpNet
F0:9F:C2:71:22:1A  ...  44 ... MGT  CorpNet
```

দুটি আলাদা BSSID একই ESSID `CorpNet` broadcast করছে।

উত্তর: **2 access points**।

## 6.6 Station section পড়া

নিচের section-এ connected client-গুলো দেখা যেতে পারে।

`(not associated)` মানে client বর্তমানে scan-এ দেখা কোনো AP-এর সঙ্গে associated নয়।

`Probes` column-এ client কোন network name খুঁজছে, তা দেখা যেতে পারে।

যদি কোনো client `Office-WiFi` probe করে, কিন্তু access point list-এ সেই SSID না থাকে, তাহলে এটি এমন network হতে পারে যেটি client আগে ব্যবহার করেছিল।

## 6.7 WPA handshake notice

কখনো header-এ এমন লেখা দেখা যেতে পারে:

```text
WPA handshake: F0:9F:C2:71:22:1A
```

এর অর্থ capture-এ ওই BSSID-এর সঙ্গে সম্পর্কিত handshake traffic শনাক্ত হয়েছে।

এটি **password নিজে থেকে প্রকাশ করে না**।

---

# 7. Task 6 — Uncovering Hidden Networks

## 7.1 Hidden SSID কী?

সাধারণত Access Point-এর beacon frame-এ SSID প্রকাশিত হয়। কিছু configuration-এ SSID field লুকানো থাকে।

তবে SSID hide করা প্রকৃত security control নয়। BSSID, channel এবং অন্য management-frame তথ্য তবুও দৃশ্যমান থাকতে পারে।

## 7.2 Lab-এ hidden network শনাক্ত করা

আগের scan-এ hidden network:

```text
BSSID: F0:9F:C2:6A:88:26
Channel: 11
ESSID: <length: 9>
```

`<length: 9>` অর্থ network name-এর দৈর্ঘ্য 9 character, নামটি নয়।

## 7.3 Hidden SSID প্রকাশ করা

Lab-এর নির্দেশনা অনুযায়ী প্রথমে channel নির্দিষ্ট করো:

```bash
sudo iw dev wlan0mon set channel 11
```

তারপর lab-provided wordlist দিয়ে probe করো:

```bash
sudo mdk4 wlan0mon p \
  -t F0:9F:C2:6A:88:26 \
  -f /home/user/wordlists/ssid-names.txt
```

ব্যাখ্যা:

- `mdk4` — wireless frame testing toolkit।
- `p` — probing mode।
- `-t` — target BSSID।
- `-f` — candidate SSID wordlist।

Lab-এর expected result:

```text
SSID length is 9
Probe Response from target AP with SSID Staff-Net
```

এখানে পাওয়া SSID:

```text
Staff-Net
```

এই active probing কেবল অনুমোদিত lab-এ করবে।

## 7.4 Hidden network-এ connect করার ধারণা

এই lab-এর hidden network হলো open network (`OPN`)। তাই এখানে WPA2 shared password crack করার প্রয়োজন নেই।

Client configuration-এ গুরুত্বপূর্ণ option:

```text
scan_ssid=1
```

এটি `wpa_supplicant`-কে hidden SSID-এর জন্য directed scan/probe করতে নির্দেশ দেয়।

Lab-এর connection profile:

```bash
cat > staff.conf <<'EOF'
network={
    ssid="Staff-Net"
    scan_ssid=1
    key_mgmt=NONE
}
EOF
```

ব্যাখ্যা:

- `ssid="Staff-Net"` — target network name।
- `scan_ssid=1` — hidden SSID-এর জন্য active scan।
- `key_mgmt=NONE` — এই open-network lab profile-এ password-based key management নেই।

**গুরুত্বপূর্ণ:** `wlan1` interface তখনই ব্যবহার করবে, যখন তোমার lab-এ সেটি উপযুক্ত interface হিসেবে দেওয়া আছে। নিজের মতো করে interface name অনুমান করবে না।

Lab-এর profile ব্যবহার করে connection চালু করতে:

```bash
sudo wpa_supplicant -B -i wlan1 -c staff.conf
```

এখানে:

- `-B` — background mode।
- `-i wlan1` — নির্বাচিত wireless interface।
- `-c staff.conf` — configuration file।

Connection যাচাই:

```bash
sudo iw dev wlan1 link
```

সফল হলে `SSID: Staff-Net` এবং `Connected to ...`-এর মতো তথ্য দেখা যেতে পারে।

## 7.5 IP address নেওয়া

Lab-এর নির্দেশনা অনুযায়ী:

```bash
sudo dhclient -v wlan1
```

DHCP server address দিলে `DHCPOFFER` ও `DHCPACK`-এর মতো message দেখা যেতে পারে।

## 7.6 Gateway panel-এ প্রবেশ

Lab-এ gateway address:

```text
192.168.16.1
```

নিচের commands lab-এর router panel-এর জন্য:

```bash
curl -s -c cookies.txt \
  -d "Username=admin&Password=admin&Submit=Submit" \
  http://192.168.16.1/login.php -o /dev/null
```

তারপর authenticated page পড়ো:

```bash
curl -s -b cookies.txt http://192.168.16.1/index.php
```

ব্যাখ্যা:

- `curl` — command-line HTTP client।
- `-s` — silent output mode।
- `-c cookies.txt` — server থেকে পাওয়া cookies file-এ save করে।
- `-d` — POST request-এ form data পাঠায়।
- `-o /dev/null` — প্রথম response body discard করে।
- `-b cookies.txt` — পরবর্তী request-এ saved cookies পাঠায়।

Lab-এর response-এ gateway panel-এর flag দেখা যায়।

এই login command-এ ব্যবহৃত `admin` credentials কেবল room-এর lab-এর জন্য; অন্য কোনো router-এ চেষ্টা করবে না।

### আমার lab-এর flag

```text
THM{0e6521916c3adf08e5b6830f6675351e4eebec92}
```

এটি তোমার terminal output থেকে পাওয়া flag।

---

# 8. Task 7 — Organising Your Recon

Live terminal output হারিয়ে যেতে পারে। তাই capture file এবং network inventory সংরক্ষণ করতে হয়।

## 8.1 Capture save করা

```bash
sudo airodump-ng -c 44 \
  --bssid F0:9F:C2:71:22:15 \
  -w corpnet-capture wlan0mon
```

এখানে `-w` output filename prefix নির্ধারণ করে।

উদাহরণ:

```text
corpnet-capture-01.cap
corpnet-capture-01.csv
```

একই prefix দিয়ে পরবর্তী run করলে `-02`, `-03` ইত্যাদি suffix তৈরি হতে পারে।

## 8.2 File দেখা

```bash
ls -lh corpnet-capture*
```

এতে matching filename-গুলো এবং তাদের size দেখা যায়।

## 8.3 File type বোঝা

| Extension        | ব্যবহার                     |
| ---------------- | --------------------------- |
| `.cap`           | Raw packet capture          |
| `.csv`           | Network/client summary      |
| `.kismet.csv`    | Kismet-style CSV summary    |
| `.kismet.netxml` | Kismet-style XML summary    |
| `.log.csv`       | Capture run-এর activity log |

সব file সব run-এ তৈরি হবে না।

## 8.4 শুধু CSV output তৈরি করা

```bash
sudo airodump-ng --output-format csv \
  --band a -c 44 \
  -w corpnet-inventory wlan0mon
```

এই command channel 44-এর network inventory CSV format-এ সংরক্ষণ করে।

File দেখতে:

```bash
ls -lh corpnet-inventory*
```

CSV content পড়তে:

```bash
cat corpnet-inventory-01.csv
```

যদি filename আলাদা হয়, `ls`-এ দেখানো সঠিক filename ব্যবহার করবে।

## 8.5 Inventory কীভাবে তৈরি করবে

একটি security assessment-এর inventory-তে রাখো:

| Field   | কী লিখবে                           |
| ------- | ---------------------------------- |
| ESSID   | Network name                       |
| BSSID   | Access point MAC                   |
| Channel | Channel number                     |
| Band    | 2.4 GHz বা 5 GHz                   |
| ENC     | Security family                    |
| CIPHER  | Cipher                             |
| AUTH    | Authentication type                |
| Clients | Observed associated stations       |
| Notes   | অনুমোদিত assessment-এর observation |

উদাহরণ:

```text
ESSID: CorpNet
BSSID: F0:9F:C2:71:22:15
Channel: 44
Band: 5 GHz
ENC: WPA2
CIPHER: CCMP
AUTH: MGT
Notes: Enterprise authentication
```

এই ধরনের inventory পরবর্তী analysis এবং report তৈরিতে কাজে লাগে।

---

# 9. Task 8 — Conclusion

এই room-এ তুমি যে workflow শিখেছ:

1. `iw dev` দিয়ে wireless interface শনাক্ত করা।
2. `airmon-ng` দিয়ে monitor mode চালু করা।
3. `airodump-ng --band bg` দিয়ে 2.4 GHz scan করা।
4. `airodump-ng --band a -c 44` দিয়ে 5 GHz-এর নির্দিষ্ট channel scan করা।
5. `BSSID`, `ESSID`, `ENC`, `CIPHER` ও `AUTH` পড়া।
6. Hidden SSID সম্পর্কে lab-এর তথ্য সংগ্রহ করা।
7. `wpa_supplicant` configuration-এ `scan_ssid=1` ব্যবহার করা।
8. `curl` ও cookies দিয়ে lab gateway page access করা।
9. `-w` দিয়ে capture সংরক্ষণ করা।
10. CSV ও inventory থেকে findings সংগঠিত করা।

মূল শিক্ষা হলো: Wireless reconnaissance কেবল network খুঁজে পাওয়া নয়; সংগৃহীত তথ্য থেকে কোন configuration চলছে এবং পরবর্তী security assessment-এর জন্য কী জানা দরকার, সেটি বোঝা।

---

# 10. Troubleshooting

## Problem 1: `wlan0mon` পাওয়া যাচ্ছে না

চালাও:

```bash
sudo iw dev
```

তারপর:

```bash
sudo airmon-ng start wlan0
```

Output-এ যে monitor interface দেখায়, সেটিই ব্যবহার করো।

## Problem 2: Scan-এ network দেখা যাচ্ছে না

1. নিশ্চিত হও interface monitor mode-এ আছে।
2. Scan 30–60 সেকেন্ড চালাও।
3. সঠিক band নির্বাচন করো।
4. Target নির্দিষ্ট channel-এ থাকলে `-c` দিয়ে সেটি নির্বাচন করো।

উদাহরণ:

```bash
sudo airodump-ng --band a -c 44 wlan0mon
```

## Problem 3: `wlan1` connect হচ্ছে না

```bash
sudo iw dev
```

দিয়ে interface-গুলোর নাম যাচাই করো।

তারপর:

```bash
sudo iw dev wlan1 link
```

`Not connected` দেখালে connection সফল হয়নি। Interface, configuration এবং lab-এর network state যাচাই করো।

## Problem 4: Capture file খুঁজে পাচ্ছ না

```bash
pwd
ls -lh
```

`pwd` current directory দেখায়। `ls` সেই directory-এর file দেখায়।

Capture command যে directory-তে চালিয়েছিলে, সেই directory-তেই প্রথমে খুঁজবে।

## Problem 5: `dhclient` command পাওয়া যাচ্ছে না

Lab-এর environment ও instructions যাচাই করো। সব Linux distribution-এ DHCP client tool একই নামে বা installed অবস্থায় থাকে না।

---

# 11. Quick Revision Sheet

## Commands

```bash
# Wireless interfaces
sudo iw dev

# Connection status
sudo iw dev wlan0 link

# Supported interface modes
iw list | grep -A 12 "Supported interface modes"

# Check processes
sudo airmon-ng check

# Stop interfering processes in the lab
sudo airmon-ng check kill

# Start monitor mode
sudo airmon-ng start wlan0

# Confirm monitor mode
sudo iw dev

# Scan 2.4 GHz
sudo airodump-ng --band bg wlan0mon

# Scan 5 GHz, channel 44
sudo airodump-ng --band a -c 44 wlan0mon

# Filter by BSSID and save capture
sudo airodump-ng -c 44 \
  --bssid F0:9F:C2:71:22:15 \
  -w corpnet-capture wlan0mon

# Check saved files
ls -lh corpnet-capture*

# Restore normal operation after the lab
sudo airmon-ng stop wlan0mon
sudo systemctl start NetworkManager
```

## Hidden network commands

```bash
# Select channel 11
sudo iw dev wlan0mon set channel 11

# Probe the lab's hidden network
sudo mdk4 wlan0mon p \
  -t F0:9F:C2:6A:88:26 \
  -f /home/user/wordlists/ssid-names.txt

# Create lab connection profile
cat > staff.conf <<'EOF'
network={
    ssid="Staff-Net"
    scan_ssid=1
    key_mgmt=NONE
}
EOF

# Connect using the lab's specified interface
sudo wpa_supplicant -B -i wlan1 -c staff.conf

# Verify connection
sudo iw dev wlan1 link

# Request a DHCP address
sudo dhclient -v wlan1
```

## Gateway panel commands

```bash
curl -s -c cookies.txt \
  -d "Username=admin&Password=admin&Submit=Submit" \
  http://192.168.16.1/login.php -o /dev/null

curl -s -b cookies.txt http://192.168.16.1/index.php
```

**Note:** Hidden-network ও gateway commands এই TryHackMe lab-এর জন্য। অন্য network-এ প্রয়োগের আগে অনুমতি এবং environment যাচাই করবে।

---

# 12. Task Answers

| Task   | Question                                              | Answer                                          |
| ------ | ----------------------------------------------------- | ----------------------------------------------- |
| Task 2 | MAC address identifying an AP                         | `BSSID`                                         |
| Task 2 | Human-readable wireless network name                  | `SSID`                                          |
| Task 3 | AP periodically advertises presence with which frame? | `Beacon`                                        |
| Task 3 | Client actively searches by name with which frame?    | `Probe Request`                                 |
| Task 4 | Monitor-mode interface created from `wlan0`           | `wlan0mon`                                      |
| Task 5 | AUTH value for enterprise authentication              | `MGT`                                           |
| Task 5 | Enterprise network channel                            | `44`                                            |
| Task 5 | Number of `CorpNet` access points                     | `2`                                             |
| Task 6 | Hidden SSID                                           | `Staff-Net`                                     |
| Task 6 | Hidden network channel                                | `11`                                            |
| Task 6 | Hidden SSID length                                    | `9`                                             |
| Task 6 | `wpa_supplicant` option                               | `scan_ssid=1`                                   |
| Task 6 | Gateway flag                                          | `THM{0e6521916c3adf08e5b6830f6675351e4eebec92}` |
| Task 7 | Flag for writing captured data to files               | `-w`                                            |

---

## Final Checklist

- [ ] আমি wireless interface শনাক্ত করতে পারি।
- [ ] Managed mode এবং monitor mode-এর পার্থক্য বুঝি।
- [ ] 2.4 GHz ও 5 GHz scan করতে পারি।
- [ ] BSSID, ESSID, channel, ENC, CIPHER ও AUTH পড়তে পারি।
- [ ] Beacon এবং Probe Request-এর পার্থক্য বুঝি।
- [ ] Hidden SSID-এর সীমাবদ্ধতা ব্যাখ্যা করতে পারি।
- [ ] Capture file এবং CSV inventory সংরক্ষণ করতে পারি।
- [ ] Lab শেষে interface ও network services স্বাভাবিক অবস্থায় ফেরানোর গুরুত্ব বুঝি।

**End of document — Wi-Fi Fundamentals and Reconnaissance**

