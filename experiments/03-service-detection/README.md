
# 🔎 Experiment 3 — Nmap Service & Version Detection

## 📌 Objective

The objective of this experiment was to use **Nmap service and version detection (`-sV`)** to identify services running on open TCP ports and observe the resulting network traffic using **Wireshark**.

This experiment builds on **Experiment 2 — Port Scanning**.

---

## 🖥️ Lab Environment

| Component          | Details                        |
| ------------------ | ------------------------------ |
| 🐉 Kali Linux      | `192.168.100.50`               |
| 🪟 Windows 11      | `192.168.100.5`                |
| 🌐 Network         | VirtualBox Internal Network    |
| 🛡️ Firewall       | Windows Defender Firewall — ON |
| 🔍 Tool            | Nmap 7.99                      |
| 📡 Packet Analysis | Wireshark                      |

---

## 🎯 Ports Investigated

Experiment 2 identified:

```text
135/tcp
139/tcp
445/tcp
```

These open ports were investigated using Nmap service detection.

---

## 🧪 Methodology

### 1️⃣ Service & Version Detection

```bash
nmap -sV -p 135,139,445 192.168.100.5
```

### 2️⃣ Wireshark Capture

Wireshark was started before the Nmap scan to capture the service-detection traffic.

The capture was saved as:

```text
service-detection.pcapng
```

---

## 📊 Nmap Results

```text
PORT     STATE SERVICE      VERSION
135/tcp  open  msrpc        Microsoft Windows RPC
139/tcp  open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds
```

Nmap also identified:

```text
OS: Windows
```

Port `445/tcp` was identified as `microsoft-ds`, without a specific version being reported.

---

## 📡 Wireshark Analysis

The capture showed communication between:

```text
192.168.100.50 → 192.168.100.5
```

Observed traffic included:

* TCP connection establishment
* Probes to ports 135, 139 and 445
* RPC-related traffic
* NetBIOS/SMB-related traffic
* Additional service-detection probes
* TCP resets following individual probes

The capture demonstrates that Nmap performs additional communication when `-sV` is used.

---

## 🔍 Findings

The experiment demonstrated that:

1. Nmap can identify services running on open TCP ports.
2. Port `135` was identified as **Microsoft Windows RPC**.
3. Port `139` was identified as **Microsoft Windows netbios-ssn**.
4. Port `445` was identified as **microsoft-ds**.
5. Wireshark provided packet-level evidence of the service-detection process.
6. Service detection requires useful responses from the target.

---

## 📁 [Evidence](https://github.com/manunair16/nmap-wireshark-network-analysis/tree/main/experiments/03-service-detection/evidence)

```text
service-detection.pcapng
```

Supporting screenshots:

```text
01-nmap.png
02-wireshark-capture.png
```

---

## ✅ Conclusion

Nmap `-sV` successfully identified the services associated with the open ports discovered during Experiment 2.

Wireshark analysis showed that service detection generates additional network traffic beyond the initial port scan, providing packet-level evidence of the detection process.

---

## ⚠️ Lab Disclaimer

This experiment was performed in an isolated virtual lab environment for learning and portfolio development.

No production systems were involved.
