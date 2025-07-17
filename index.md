# WirelessSec 2.0 — OWASP-Style Wireless Security Testing Guide  
**License:** CC BY-SA 4.0  
**Version:** 2.0 (July 2025)  
**Maintainers:** WirelessSec Community, Marcelo Burgos  

<img width="500" height="500" alt="image" src="https://github.com/user-attachments/assets/e9174dc3-c2a3-4dde-8f0e-750e2988208b" />


## Table of Contents
1. [Introduction](#1-introduction)  
2. [Scope & Objectives](#2-scope--objectives)  
3. [Methodology Framework](#3-methodology-framework)  
4. [Testing Categories](#4-testing-categories)  
    - [4.1 Reconnaissance & OSINT](#41-reconnaissance--osint)  
    - [4.2 Traffic Capture & Analysis](#42-traffic-capture--analysis-passiveactive-connecteddisconnected)  
    - [4.3 Credential & Session Attacks](#43-credential--session-attacks)  
    - [4.4 Network Configuration & Segmentation](#44-network-configuration--segmentation)  
    - [4.5 Physical Access & Infrastructure Security](#45-physical-access--infrastructure-security)  
    - [4.6 BLE & Wireless IoT](#46-ble--wireless-iot)  
    - [4.7 Denial of Service](#47-denial-of-service)  
5. [Threat Modeling & Risk Scoring](#5-threat-modeling--risk-scoring)  
6. [Tools & Techniques](#6-tools--techniques)  
7. [Reporting & Deliverables](#7-reporting--deliverables)  
8. [References](#8-references)  
9. [Appendix: Test Case & Report Templates](#9-appendix-test-case--report-templates)  

---

## 1. Introduction  
WirelessSec 2.0 is a modular, in-depth methodology for auditing and hardening wireless networks—including Wi‑Fi 6/6E, BLE, and IoT—in enterprise, SMB, and public deployments. It integrates modern adversary emulation, STRIDE threat modeling, risk scoring (CVSS 4.0, OWASP Risk Rating), and OSINT reconnaissance. Its goal is to yield reproducible, actionable findings aligned with OWASP standards.  

---

## 2. Scope & Objectives  
**Protocols:** 802.11a/b/g/n/ac/ax, BLE, Zigbee, OWE  

**Assets:** APs, routers, switches, controllers, firewalls, clients, IoT/BLE devices  

**Environments:** Private & public Wi‑Fi, guest networks, captive portals  

**Focus Areas:** Logical (radio attacks), physical (infrastructure access), hybrid scenarios  

**Outcomes:** Prioritized remediation, proof-of-concept (PoC), threat intelligence integration  

---

## 3. Methodology Framework  
Phases follow OWASP WSTG style, each with test IDs (WLS-XXX):  

1. Reconnaissance & OSINT  
2. Traffic Capture & Analysis  
3. Credential & Session Attacks  
4. Network Configuration & Segmentation  
5. Physical Access & Infrastructure Security  
6. BLE & Wireless IoT Testing  
7. Denial of Service  

**Each test includes:** Objective, Procedure, Tools, Risk, Findings, Remediation.  

---

## 4. Testing Categories  
### 4.1 Reconnaissance & OSINT  
- **Passive scan:** Kismet, airodump-ng — capture beacons, probe requests, metadata  
- **Active scan:** Directed probe requests to enumerate hidden SSIDs  
- **OSINT:** Wigle, Shodan, Google Dorks to find exposed SSIDs, AP GUIs, default credentials  
- **Asset inventory:** Document SSID/BSSID, channel, encryption, vendor, BLE advertisements  

### 4.2 Traffic Capture & Analysis (Passive/Active; Connected/Disconnected)  
| Mode           | Description                                                                 |
|----------------|-----------------------------------------------------------------------------|
| Passive        | Monitor mode captures (EAPOL, PMKID) without joining                       |
| Active         | Deauth attacks to force reauth and capture sessions                        |
| Connected      | Sniff traffic as legitimate client (EAPOL, data)                           |
| Disconnected   | Sniff broadcast/multicast traffic without connect                          |

- Use `hcxdumptool`/`hcxpcapngtool` for PMKID dumps  
- Analyze unencrypted metadata (DNS, mDNS, NetBIOS)  

### 4.3 Credential & Session Attacks  
- Handshake/PMKID cracking: Hashcat, JohnTheRipper with GPU  
- WPA3-SAE downgrade attacks (Dragonblood vulnerability tests)  
- Evil Twin & Rogue Portal: `hostapd-wpe`, Bettercap; harvest credentials  
- MAC spoofing to bypass MAC-based ACLs and whitelists  
- ARP spoofing in wireless contexts to intercept or redirect traffic  

### 4.4 Network Configuration & Segmentation  
- VLAN isolation: Test inter-VLAN traffic via tagged frames  
- Firewall placement: Validate restrictions on guest/public VLANs  
- WPS auditing: Confirm WPS disabled; brute force if enabled  
- SSID segmentation: Ensure private vs guest SSIDs mapped to distinct security zones  
- MAC randomization handling: Document policy for pseudorandom MACs vs allowlists  

### 4.5 Physical Access & Infrastructure Security  
- Unsupervised RJ45 ports bridging Wi-Fi VLANs; test direct LAN access  
- AP hardware security: Inspect WPS buttons, serial ports, physical locks  
- Rogue device detection: GhostESP, AngryOxide, manual sweeps  
- Hybrid bypass scenarios: Demonstrate physical port access bypassing wireless controls  
- Mitigations: Lock network outlets, enable 802.1X port security, disable unused ports  

### 4.6 BLE & Wireless IoT  
- Device enumeration: Ubertooth, BLEAH to list devices and services  
- Pairing security: Sniff pairing requests, MITM GATT operations  
- Persistent connections: Test BLE auto-pair to known AP or hub  
- Signal spoofing: Replay BLE broadcasts to trigger unintended actions  
- Firmware analysis: Assess IoT devices for vulnerable OTAP mechanisms  

### 4.7 Denial of Service  
- Deauth/disassoc floods: `aireplay-ng`, `mdk3`  
- Beacon flooding: Scapy, GhostESP  
- Channel saturation: Measure resilience under high RF noise  
- Band steering abuse: Force clients onto less secure bands  

---

## 5. Threat Modeling & Risk Scoring  
- **STRIDE:** Map each test to Spoofing, Tampering, Repudiation, Info Disclosure, DoS, Elevation  
- **CVSS v4.0:** Score base, temporal, environmental metrics  
- **OWASP Risk Rating:** Qualitative Likelihood × Impact  

**Example:**  
- **Test:** Evil Twin Captive Portal  
- **STRIDE:** S, I, E  
- **CVSS:** 8.9 (AV:A/AC:L/PR:N/UI:R/S:C/C:H/I:H/A:N)  
- **OWASP Risk:** Likelihood 4/5 × Impact 5/5 = Critical  

---

## 6. Tools & Techniques  
| Category               | Tools & Purpose                                                                 |
|------------------------|---------------------------------------------------------------------------------|
| Passive Monitoring     | Kismet, airodump-ng, Wireshark — capture beacons, probe requests, EAPOL, PMKID |
| Frame Injection        | Bettercap, Scapy, aircrack-ng — inject deauth/disassoc frames, ARP poisoning   |
| Handshake & PMKID Capture | hcxdumptool, hcxpcapngtool, AngryOxide — capture WPA handshakes, PMKIDs       |
| Password Cracking      | Hashcat, JohnTheRipper — GPU-accelerated dictionary attacks                    |
| Rogue AP Emulation     | hostapd-wpe, WifiPhisher — create Evil Twin, captive portals                   |
| BLE Analysis           | Ubertooth, BLEAH — enumerate BLE devices, sniff pairing                        |
| Physical Detection     | GhostESP, AngryOxide — detect rogue APs, scan RF anomalies                     |
| OSINT Reconnaissance   | Shodan, Wigle — find exposed SSIDs, management interfaces                      |

---

## 7. Reporting & Deliverables  
- **Executive Summary:** High-level overview of critical risks and impact.  
- **Key Findings:** Detailed vulnerabilities with evidence (logs, screenshots).  
- **Proofs of Concept (PoC):** Step-by-step exploit recreations.  
- **Risk Model Summary:** STRIDE, CVSS v4.0, OWASP Risk Rating per finding.  
- **Threat Intelligence Appendix:** OSINT data on network exposure.  
- **Physical Security Findings:** Diagrams of infrastructure weaknesses.  
- **Remediation Plan:** Prioritized fixes mapped to risk scores.  
- **Certification Appendix:** WirelessSec 2.0 compliance checklist.  

**Delivery:** Encrypted PDFs/secured portals with stakeholder walkthrough.  

---

## 8. References  
- [OWASP Wi-Fi Security Testing Guide](https://owasp.org/www-project-wi-fi-security-testing-guide/)  
- [OWASP IoT Security Testing Guide](https://owasp.org/owasp-istg/03_test_cases/wireless_interfaces/index.html)  
- [Wi-Fi Alliance WPA3 Specifications](https://www.wi-fi.org/discover-wi-fi/security)  
- [Dragonblood WPA3 Vulnerabilities](https://www.dragonblood.io/)  
- [GhostESP Repo](https://github.com/ghostsecurity/ghostesp)  
- [Aircrack-ng Suite](https://www.aircrack-ng.org/)  

---

## 9. Appendix: Test Case & Report Templates  
```vbnet
Test ID: WLS-123  
Title: <Concise test name>  
Category: <e.g., Traffic Capture>  
Objective: <What this test aims to validate>  
Procedure:  
  1. <Step 1>  
  2. <Step 2>  
Tools: <List tools>  
Expected Result: <Describe success criteria>  
Findings: <Observed behavior>  
Impact & Risk: <STRIDE, CVSS, OWASP rating>  
Remediation: <Fix recommendation>  
References: <Links to standards or proof>  
