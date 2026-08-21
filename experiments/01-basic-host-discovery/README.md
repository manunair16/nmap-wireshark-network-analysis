# 🔎 Experiment 01 — Basic Host Discovery

## 🎯 Objective

The objective of this experiment was to:

* Discover the Windows 11 target using Nmap
* Confirm that the target host is active
* Observe the network traffic generated during host discovery using Wireshark
* Correlate the Nmap result with packet-level evidence

---

## 🖥️ Lab Details

| Component          | Details          |
| ------------------ | ---------------- |
| 🐧 Kali Linux      | `192.168.100.50` |
| 🪟 Windows 11      | `192.168.100.5`  |
| 💻 Virtualization  | VirtualBox       |
| 🔍 Tool            | Nmap             |
| 📡 Packet Analysis | Wireshark        |

---

## 🧪 Method

Wireshark was started on the Kali Linux network interface before running the Nmap discovery scan.

The following command was used:

```bash
sudo nmap -sn 192.168.100.5
```

The `-sn` option performs host discovery without performing a port scan.

A clean Wireshark capture was then saved for packet analysis.

---

## 🔎 Nmap Result

Nmap reported:

```text
Nmap scan report for 192.168.100.5
Host is up
MAC Address: 08:00:27:70:67:EC
1 IP address (1 host up) scanned
```

The target Windows 11 system was successfully identified as active.

---

## 📡 Wireshark Observation

Wireshark captured an ARP request and response between the two systems.

### ARP Request

Kali Linux requested the MAC address associated with:

```text
192.168.100.5
```

### ARP Reply

The Windows 11 system responded with:

```text
192.168.100.5 → 08:00:27:70:67:EC
```

The MAC address observed in the ARP response matched the MAC address reported by Nmap.

---

## 🔗 Nmap ↔ Wireshark Correlation

The experiment demonstrated the relationship between the Nmap result and the underlying network traffic:

```text
Nmap Host Discovery
        ↓
Target identified as active
        ↓
ARP Request
        ↓
ARP Reply
        ↓
Target IP mapped to MAC address
        ↓
MAC address matches Nmap result
```

This provided packet-level evidence supporting the Nmap discovery result.

---

## ✅ Conclusion

The Windows 11 target was successfully discovered using Nmap.

Wireshark captured the ARP request and response used to resolve the target IP address to its MAC address. The MAC address reported by Nmap matched the MAC address observed in the ARP response.

The experiment demonstrated how Nmap results can be correlated with actual network traffic using Wireshark.

---

## 📸 [Evidence](https://github.com/manunair16/nmap-wireshark-network-analysis/tree/main/experiments/01-basic-host-discovery/evidence)

The experiment evidence includes:

* Nmap host discovery result
* Wireshark ARP request
* Wireshark ARP reply
* Original Wireshark `.pcapng` capture
