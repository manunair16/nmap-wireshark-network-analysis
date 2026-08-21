# 🤝 Collaboration, Comparison & Network Analysis

## 🎯 Purpose

This document records **my individual network analysis and the collaborative comparison** carried out during the Nmap & Wireshark project.

The project uses independent laboratory environments. Although the three participants followed the same overall progression, each maintained their own systems, network configuration, captures, screenshots, and findings.

My analysis focuses on connecting:

```text
Nmap
  ↓
Scan Probe
  ↓
Network Response
  ↓
Wireshark
  ↓
Packet Evidence
  ↓
Interpretation
```

The purpose is to understand what actually happens on the network rather than relying only on the Nmap summary.

---

# 🖥️ My Lab Environment

| Component              | Details          |
| ---------------------- | ---------------- |
| 🐧 Analysis / Scanning | Kali Linux       |
| 🪟 Target              | Windows 11       |
| 💻 Virtualization      | VirtualBox       |
| 🌐 Network             | Internal network |
| 🔍 Scanner             | Nmap             |
| 📡 Analysis            | Wireshark        |
| Kali IP                | `192.168.100.50` |
| Windows IP             | `192.168.100.5`  |

My baseline evidence documents the VirtualBox configuration, network connectivity, Windows listening ports, and lab topology.

📁 [Baseline & Topology](../baseline-and-topology/)

---

# 🔬 My Analysis Approach

For each experiment, I used four questions:

1. **What did Nmap send?**
2. **What did Windows return?**
3. **What does Wireshark show?**
4. **Does the packet behaviour explain the Nmap result?**

This produces a useful evidence chain:

```text
Command
  ↓
Packet
  ↓
Response
  ↓
Nmap State
  ↓
Explanation
```

---

# 01 — Host Discovery

Host discovery is intended to identify active systems before more extensive scanning.

A common misconception is that Nmap host discovery simply means sending an ICMP ping.

It does not.

## 🦈 ARP and ICMP

When the target is on the same local Ethernet network, ARP can be used to establish that an IPv4 address is present.

```text
Kali → ARP Request → Windows
Kali ← ARP Reply  ← Windows
```

This is different from:

```text
Kali → ICMP Echo Request → Windows
Kali ← ICMP Echo Reply  ← Windows
```

ARP operates at Layer 2 while ICMP operates at Layer 3.

The reason this matters in Wireshark is that a host-discovery capture can show ARP traffic even when the user expected to see ICMP.

Nmap's documentation explains that ARP/Neighbor Discovery is used by default for local Ethernet targets because it is normally faster and more effective.

### Practical interpretation

| Packet observed      | Interpretation                                |
| -------------------- | --------------------------------------------- |
| ARP Request + Reply  | Local IPv4 address resolved successfully      |
| ICMP Echo + Reply    | ICMP discovery succeeded                      |
| TCP SYN + SYN/ACK    | TCP discovery received a positive response    |
| TCP SYN + RST        | Target responded, but that port may be closed |
| No expected response | Further investigation may be required         |

Therefore:

> **ICMP failure does not automatically mean that the host is offline.**

---

# 🔎 Host Discovery Options

Useful discovery options include:

| Option               | Function                                   |
| -------------------- | ------------------------------------------ |
| `-sn`                | Perform host discovery without a port scan |
| `-Pn`                | Skip host discovery                        |
| `-PE`                | ICMP Echo                                  |
| `-PP`                | ICMP Timestamp                             |
| `-PS`                | TCP SYN discovery                          |
| `-PA`                | TCP ACK discovery                          |
| `-PU`                | UDP discovery                              |
| `--disable-arp-ping` | Disable ARP discovery                      |

These are useful when comparing why one discovery method succeeds while another fails.

For example:

```bash
nmap -sn 192.168.100.5
```

can be compared with:

```bash
nmap -sn -PE 192.168.100.5
```

and, where appropriate:

```bash
nmap -sn -PS80,443 192.168.100.5
```

The exact discovery method should always be confirmed from the Nmap command and packet capture rather than assumed.

---

# 02 — Port Scanning

Port scanning moves from:

> "Is the host reachable?"

to:

> "Which ports appear accessible?"

For TCP, the most useful comparison is between SYN and Connect scans.

## 🔹 SYN Scan — `-sS`

```text
Kali → Windows : SYN
Windows → Kali : SYN/ACK
```

An open port commonly produces SYN/ACK.

A closed port commonly produces RST.

Filtering may result in no usable response or filtering-related evidence.

The important analytical point is that **filtered means Nmap cannot determine the port state from the available evidence**. It does not, by itself, identify the exact filtering device.

---

# 🔹 Connect Scan — `-sT`

A Connect scan uses the operating system's normal connection mechanism.

```text
SYN
 ↓
SYN/ACK
 ↓
ACK
```

The handshake is completed.

This gives a clear Wireshark distinction:

| Scan  | Handshake              |
| ----- | ---------------------- |
| `-sS` | Normally not completed |
| `-sT` | Completed              |

The distinction is important when explaining scan behaviour during an interview or technical review.

---

# 🔎 Port-Scanning Options

| Option      | Purpose                |
| ----------- | ---------------------- |
| `-p 80,443` | Scan selected ports    |
| `-p-`       | Scan all TCP ports     |
| `-F`        | Fast scan              |
| `-sS`       | TCP SYN                |
| `-sT`       | TCP Connect            |
| `-sU`       | UDP                    |
| `-sA`       | ACK/filtering analysis |

These options should be used deliberately.

A useful packet-analysis principle is:

> **Change one variable at a time when comparing results.**

For example, scan the same port using `-sS` and `-sT`, then compare the corresponding captures.

---

# 🔥 Firewall and Filtering

A firewall can alter the packet behaviour seen by Nmap.

A useful experiment is:

```text
Firewall ON
    ↓
Same scan
    ↓
Capture

Firewall OFF
    ↓
Same scan
    ↓
Capture

       ↓
Compare
```

The comparison should examine:

* Number of probes
* Response type
* TCP flags
* Missing responses
* Nmap state
* Difference in firewall configuration

The correct conclusion should be based on evidence.

For example:

> "The port was reported as filtered and the packet capture showed no expected response."

is stronger than:

> "The firewall definitely blocked it."

The second statement may be true, but the packet capture alone may not prove which filtering component caused the result.

---

# 03 — Service Detection

Service detection builds on discovered ports.

```bash
nmap -sV 192.168.100.5
```

The purpose is to identify the application/service associated with a reachable port and, where possible, its version.

From a Wireshark perspective, service detection is different from simple port scanning because Nmap can send additional application-aware probes.

Therefore:

```text
Port Scan
    ↓
Port appears accessible
    ↓
Service Detection
    ↓
Additional probes
    ↓
Application response
```

This is why the service-detection capture should be analysed separately rather than assuming it will look identical to the port-scan capture.

---

# 📡 TCP Packet Interpretation

A simple analytical table is:

| Nmap Result  | Packet Evidence                                | Interpretation                                               |
| ------------ | ---------------------------------------------- | ------------------------------------------------------------ |
| `open`       | SYN/ACK                                        | TCP service accepted the connection attempt                  |
| `closed`     | RST                                            | Host reachable, but no service accepting that TCP connection |
| `filtered`   | Missing/blocked response or filtering evidence | State cannot be determined confidently                       |
| `unfiltered` | Response to ACK probe                          | Probe reached target; does not mean open                     |

This distinction is especially important for ACK scans.

---

# 🔐 ACK Scan — `-sA`

An ACK scan asks a different question from a SYN scan.

```text
-sS → What is the TCP port state?
-sA → Is the traffic being filtered?
```

A typical response:

```text
Kali → TCP ACK → Windows
Kali ← TCP RST ← Windows
```

can indicate an **unfiltered** result.

However:

> **Unfiltered does not mean open.**

ACK scanning cannot reliably distinguish open from closed.

This is a good example of why Nmap output must be interpreted in the context of the scan type.

---

# 🌐 UDP Analysis

UDP has no TCP three-way handshake.

Possible evidence includes:

```text
UDP Probe
    ↓
UDP Response
    ↓
open
```

or:

```text
UDP Probe
    ↓
ICMP Port Unreachable
    ↓
closed
```

or:

```text
UDP Probe
    ↓
No Response
    ↓
open|filtered
```

The last state represents uncertainty.

It should not be simplified to:

> "The UDP port is open."

Nmap can report `open|filtered` because silence may represent either an open service that did not respond or filtering.

---

# 🧩 FIN, NULL & Xmas

These scan types provide another useful packet-level comparison:

| Option | Probe           |
| ------ | --------------- |
| `-sF`  | FIN             |
| `-sN`  | No TCP flags    |
| `-sX`  | FIN + PSH + URG |

The main Wireshark question is:

> **What TCP flags did Nmap actually send?**

For these scans, the packet flags are more informative than simply memorizing the switch names.

They can be useful for a future experiment because they demonstrate how different TCP flag combinations produce different target behaviour.

---

# 🦈 Wireshark Investigation

Useful display filters for my analysis include:

```text
ip.addr == 192.168.100.5
```

```text
arp
```

```text
icmp
```

```text
tcp
```

```text
tcp.port == 80
```

```text
tcp.flags.syn == 1
```

```text
tcp.flags.reset == 1
```

```text
udp
```

A filter should be used to reduce noise, not to replace packet interpretation.

For example:

```text
tcp.flags.syn == 1
```

helps locate SYN packets, but the analyst should still inspect:

* Source
* Destination
* Port
* Sequence/acknowledgement information
* TCP flags
* Response
* Timing

---

# 📊 Comparison With Other Labs

All three participants followed the same general sequence:

```text
Baseline
   ↓
Host Discovery
   ↓
Port Scanning
   ↓
Service Detection
   ↓
Wireshark Analysis
```

However, the environments were independent.

| Participant | Virtualization | Kali             | Windows          |
| ----------- | -------------- | ---------------- | ---------------- |
| Manu        | VirtualBox     | `192.168.100.50` | `192.168.100.5`  |
| Varun       | UTM            | `192.168.128.3`  | `192.168.128.4`  |
| Hari        | VirtualBox     | `192.168.100.10` | `192.168.100.20` |

Therefore, identical packet counts, timestamps, or minor scan behaviour should not be expected.

The useful comparison is based on **protocol behaviour and methodology**.

---

# 👥 Other Participants

### 👨‍💻 Varun M Nair

Varun performed the same project progression using a UTM-based environment and a different subnet.

📄 [Varun — Collaboration & Network Analysis](https://github.com/varunmnair95/nmap-wireshark-network-analysis)

### 👨‍💻 Hari Krishnan R K

Hari performed the project using his own VirtualBox environment and independent evidence.

📄 [Hari — Collaboration & Network Analysis](https://github.com/harikrishnan-rk/nmap-wireshark-network-analysis)

Their detailed observations remain in their respective documents rather than being duplicated here.

---

# 🧠 My Analytical Takeaways

The most useful lesson for me was understanding the relationship between:

```text
Nmap Result
     ↕
Packet Evidence
```

For example, an Nmap `open` result should be investigated through the expected TCP response rather than simply accepted.

Similarly, a `filtered` result should lead to questions about:

* Firewall behaviour
* Missing responses
* Network path
* Target configuration
* Service state

For UDP, `open|filtered` should be treated as uncertainty rather than proof of an open service.

---

# 🎓 Skills Demonstrated

* 🔎 Host discovery
* 🚪 Port scanning
* 🛠️ Service detection
* 📡 Packet analysis
* 🌐 ARP and ICMP interpretation
* 🔥 Firewall/filter analysis
* 🧩 TCP flag analysis
* 📊 Cross-lab comparison
* 🧪 Controlled testing
* 📝 Evidence-based reasoning

---

# 📌 Conclusion

My contribution demonstrates practical use of Nmap and Wireshark while maintaining an evidence-based approach to network analysis.

The important progression was:

**Run the scan → capture the traffic → identify the packets → understand the response → compare with Nmap → explain the result.**

The collaborative comparison then allowed the same methodology to be considered across different independent environments.

---

## ⚠️ Disclaimer

All testing was performed against systems controlled by the participants in an isolated laboratory environment.

Nmap scanning should only be performed against systems where permission to test has been obtained.
