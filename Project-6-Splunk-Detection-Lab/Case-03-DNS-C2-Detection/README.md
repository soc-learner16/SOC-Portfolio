# SOC Case 03 — DNS C2 Detection & Investigation

## Project Overview

This project demonstrates a controlled DNS-based Command and Control (C2) activity in a Windows and Ubuntu lab environment.

A PowerShell-based lab client on the Windows endpoint contacted a controlled DNS server running on Ubuntu. The server returned predefined, allowlisted commands through DNS responses. The Windows client executed only those commands and sent the command output back through DNS queries.

Sysmon was used to record DNS activity on the Windows endpoint. The logs were forwarded to Splunk, where the DNS activity was searched, a detection was created, and an alert was triggered for the simulated C2 activity.

The activity was then investigated using Splunk and supported with Wireshark packet evidence.

> **Note:** This is a controlled lab simulation using benign commands. It is intended for learning DNS-based C2 concepts and SOC investigation techniques.

---

## Objectives

- Understand how DNS can be used as a C2 communication channel.
- Generate controlled DNS C2 activity in a lab environment.
- Capture DNS activity using Sysmon.
- Forward Windows telemetry to Splunk.
- Create a simple SPL-based detection for the lab C2 activity.
- Configure and trigger a Splunk alert.
- Investigate the triggered alert.
- Identify the process responsible for the DNS activity.
- Identify Base32-encoded data inside DNS queries.
- Validate the DNS communication using Wireshark.

---

## Lab Environment

| Component | Purpose |
|---|---|
| Windows 10 | Endpoint running the controlled C2 client |
| Ubuntu Server | Controlled DNS C2 server |
| Sysmon | Windows endpoint telemetry |
| Splunk Universal Forwarder | Forward Windows logs |
| Splunk | Log search, detection, and alerting |
| Wireshark | DNS packet analysis |
| VirtualBox | Virtual lab environment |

### Network

The main lab communication used an isolated VirtualBox Host-only network:


192.168.10.0/24


| System | IP Address |
|---|---|
| Windows 10 | `192.168.10.5` |
| Ubuntu Server | `192.168.10.7` |

---

## Network Architecture

```text
                Isolated Host-only Network
                   192.168.10.0/24

        ┌──────────────────────────────┐
        │        Windows 10            │
        │        192.168.10.5          │
        │                              │
        │ PowerShell C2 Client         │
        │ Sysmon                       │
        └──────────────┬───────────────┘
                       │
                       │ DNS C2
                       │
                       ▼
        ┌──────────────────────────────┐
        │       Ubuntu Server          │
        │        192.168.10.7          │
        │                              │
        │ Controlled DNS C2 Server     │
        └──────────────────────────────┘
```

---

## DNS C2 Scenario

The lab used the domain:

```text
c2-sync.lab
```

The Windows client performed a DNS check-in to the Ubuntu server.

The Ubuntu server returned an encoded task through a DNS response.

The Windows client:

1. Received the encoded task.
2. Decoded the task.
3. Checked whether the task was allowlisted.
4. Executed the command.
5. Encoded the command output.
6. Sent the encoded result back through a DNS query.

Only the following commands were allowed:

```text
whoami
hostname
ipconfig
systeminfo
```

No unrestricted command execution was implemented.

---

## DNS C2 Communication Flow

```text
Windows Client
      │
      │ 1. DNS check-in
      │    checkin.<sequence>.c2-sync.lab
      ▼
Ubuntu C2 Server
      │
      │ 2. Encoded task in DNS response
      ▼
Windows Client
      │
      │ 3. Decode task
      │
      │ 4. Execute allowlisted command
      │
      │ 5. Encode result
      │
      │ 6. DNS result query
      ▼
Ubuntu C2 Server
```

Example:

```text
Check-in:
checkin.022906.c2-sync.lab

Encoded task:
ON4XG5DFNVUW4ZTP

Decoded task:
systeminfo
```

The command result was then encoded and placed inside a DNS query name.

---

## 1. DNS Baseline

Before running the controlled C2 activity, normal DNS activity was observed on the Windows endpoint.

The baseline was used to understand normal DNS query patterns and the processes generating DNS requests.

### Evidence

- [Screenshot 01 — DNS Baseline Query Frequency](evidence/01_DNS_Baseline_Query_Frequency.png)
- [Screenshot 03 — DNS Baseline Process/Query Relationship](evidence/03_DNS_Baseline_Process_Query_Relationship.png)

---

## 2. Controlled DNS C2 Simulation

The Ubuntu server was configured to provide a small set of predefined tasks.

The Windows PowerShell client contacted the server and received an encoded task.

The server then received the encoded command result from the Windows endpoint.

### Evidence

- [Screenshot 06 — DNS C2 Server Task and Result](evidence/06_DNS_C2_Server_Task_Result.png)
- [Screenshot 07 — DNS C2 Client Task Execution](evidence/07_DNS_C2_Client_Task_Execution.png)

---

## 3. Sysmon DNS Events

Sysmon Event ID 22 was used to record DNS queries generated by the Windows endpoint.

The events provided information such as:

- DNS query name
- Process image
- User
- Process ID
- Query status
- DNS results

Example activity was associated with:

```text
powershell.exe
```

### Evidence

- [Screenshot 08 — Sysmon DNS C2 Events](evidence/08_Sysmon_DNS_C2_Events.png)

---

## 4. Splunk Log Collection

The Windows Sysmon logs were forwarded to Splunk using the Splunk Universal Forwarder.

The DNS events were then searched in Splunk using Sysmon Event ID 22.

The lab domain was filtered using:

```text
QueryName="*c2-sync.lab*"
```

### Evidence

- [Screenshot 09 — Splunk DNS C2 Events](evidence/09_Splunk_DNS_C2_Events.png)

---

## 5. DNS and Process Analysis

The DNS events were searched to identify which process was generating the C2 queries.

The following SPL was used:

```spl
index=* host=DESKTOP-B6GVH9G sourcetype="WinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode=22 earliest=-30m
| search QueryName="*c2-sync.lab*"
| stats count by Image QueryName
| sort - count
```

### What the query does

- `index=*` — searches across available indexes.
- `host=DESKTOP-B6GVH9G` — limits the search to the Windows endpoint.
- `EventCode=22` — searches Sysmon DNS events.
- `earliest=-30m` — searches the previous 30 minutes.
- `QueryName="*c2-sync.lab*"` — finds queries containing the lab C2 domain.
- `stats count by Image QueryName` — counts DNS events grouped by process and query.
- `sort - count` — displays the highest counts first.

The results showed PowerShell activity associated with the lab DNS domain.

### Evidence

- [Screenshot 10 — Splunk DNS Process/Query Relationship](evidence/10_Splunk_DNS_Process_Query_Relationship.png)

---

## 6. DNS C2 Detection

A simple SPL detection was created to identify repeated DNS queries to the controlled C2 domain combined with longer query names.

```spl
index=* host=DESKTOP-B6GVH9G sourcetype="WinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode=22 earliest=-2h
| search QueryName="*c2-sync.lab*"
| eval QueryLength=len(QueryName)
| bin _time span=5m
| stats count as dns_queries
    max(QueryLength) as max_query_length
    values(Image) as process
    values(QueryName) as queries
    by _time
| where dns_queries >= 4 AND max_query_length >= 50
| eval detection="Suspicious DNS C2 Activity"
| table _time host dns_queries max_query_length process detection queries
| sort _time
```

### How the detection works

The search:

1. Looks for Sysmon DNS events.
2. Filters for the controlled C2 domain.
3. Calculates the DNS query length.
4. Groups the events into 5-minute periods.
5. Counts the DNS queries in each period.
6. Finds the longest query in each period.
7. Generates a result when:
   - at least 4 DNS queries occur, and
   - the longest query is at least 50 characters.

This detection was designed specifically for the controlled lab activity.

It should **not** be considered a general-purpose DNS tunneling detection rule.

---

## 7. Splunk Alert

The detection search was saved as a scheduled Splunk alert:

```text
Alert:
Suspicious DNS C2 Activity - PowerShell
```

Configuration:

| Setting | Value |
|---|---|
| Alert type | Scheduled |
| Schedule | Every hour |
| Search window | Previous 1 hour |
| Trigger condition | Number of results > 0 |
| Trigger | Once |
| Severity | Medium |
| Triggered Alerts | Enabled |

### Evidence

- [Screenshot 12 — Splunk DNS C2 Alert Enabled](evidence/12_Splunk_DNS_C2_Alert_Enabled.png)

---

## 8. Alert Trigger

The detection successfully produced a result and triggered the Splunk alert.

The alert appeared in the **Triggered Alerts** section.

### Evidence

- [Screenshot 13 — Splunk DNS C2 Alert Triggered](evidence/13_Splunk_DNS_C2_Alert_Triggered.png)

This confirmed that the detection was able to identify the simulated DNS C2 activity.

---

## 9. Investigation

After the alert triggered, the DNS events were investigated to understand the activity.

### 9.1 Raw DNS Result Event

A raw Sysmon Event ID 22 event showed a DNS query containing encoded data:

```text
res.022906.JBXXG5BAJZQW2JZ2EAQCAIBAEAQCAIBAEAQEIRKT.c2-sync.lab
```

The event also showed:

```text
Image:
C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
```

This connected the DNS activity with PowerShell.

### Evidence

- [Screenshot 14 — Raw DNS C2 Result Event](evidence/14_Splunk_Raw_DNS_C2_Result_Event.png)

---

### 9.2 DNS C2 Check-in

A separate DNS event showed the initial check-in:

```text
checkin.022906.c2-sync.lab
```

The DNS response contained:

```text
ON4XG5DFNVUW4ZTP
```

This value was confirmed to represent:

```text
systeminfo
```

### Evidence

- [Screenshot 15 — DNS C2 Check-in](evidence/15_Splunk_DNS_C2_Checkin_Event.png)

---

### 9.3 Encoded Data Extraction

The encoded data was extracted from the DNS query name for investigation.

```spl
index=* host=DESKTOP-B6GVH9G sourcetype="WinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode=22
| search QueryName="res.022906.*.c2-sync.lab"
| rex field=QueryName "^res\.\d+\.(?<EncodedData>[^.]+)\.c2-sync\.lab$"
| table _time QueryName EncodedData
```

The extracted value was:

```text
JBXXG5BAJZQW2JZ2EAQCAIBAEAQCAIBAEAQEIRKT
```

### Evidence

- [Screenshot 16 — Encoded Payload Extraction](evidence/16_Encoded_Payload_Extraction.png)

---

### 9.4 Process Activity

The Windows process events were searched to confirm execution of the received command.

```spl
index=* host=DESKTOP-B6GVH9G sourcetype="WinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode=1
| search CommandLine="*systeminfo*"
| table _time Image CommandLine ParentImage User ProcessId
| sort _time
```

The results showed:

```text
Image:
C:\Windows\System32\systeminfo.exe

ParentImage:
C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
```

This provided supporting evidence that the `systeminfo` command was executed by PowerShell.

### Evidence

- [Screenshot 17 — Systeminfo Process Execution](evidence/17_Systeminfo_Process_Execution.png)

---

### 9.5 Base32 Validation

The encoded task and result were validated using Base32 decoding.

Example:

```text
ON4XG5DFNVUW4ZTP
        ↓
systeminfo
```

The result data was also decoded and contained system information returned by the Windows endpoint.

### Evidence

- [Screenshot 18 — DNS C2 Base32 Decoding](evidence/18_DNS_C2_Base32_Decoding.png)

---

## 10. Wireshark Packet Validation

Wireshark was used as supporting evidence to verify that the DNS C2 communication occurred between the two lab systems.

The capture showed:

```text
Windows → Ubuntu
DNS check-in

Ubuntu → Windows
DNS response

Windows → Ubuntu
DNS result query

Ubuntu → Windows
DNS response
```

The traffic was filtered using the lab domain:

```text
c2-sync.lab
```

### Evidence

- [Screenshot 19 — Wireshark DNS C2 Exchange](evidence/19_Wireshark_DNS_C2_Exchange.png)

---

## 11. Investigation Timeline

| Stage | Activity |
|---|---|
| 1 | Normal DNS activity observed |
| 2 | Controlled DNS C2 client started |
| 3 | Windows endpoint sent DNS check-in |
| 4 | Ubuntu server returned encoded task |
| 5 | Windows decoded and executed the allowlisted task |
| 6 | Command result was encoded |
| 7 | Encoded result was sent through DNS |
| 8 | Sysmon recorded the DNS activity |
| 9 | Events were forwarded to Splunk |
| 10 | SPL detection identified the activity |
| 11 | Splunk alert was triggered |
| 12 | DNS, process, and packet evidence were investigated |

---

## 12. Key Findings

The investigation confirmed:

- DNS communication occurred between the Windows endpoint and Ubuntu server.
- The communication used the controlled domain `c2-sync.lab`.
- PowerShell generated the DNS queries.
- DNS responses contained encoded task information.
- The Windows client executed only allowlisted commands.
- Command output was encoded and placed into DNS query names.
- Sysmon Event ID 22 captured the DNS activity.
- Splunk successfully received and searched the events.
- The SPL detection generated a result.
- The Splunk alert was successfully triggered.
- Wireshark provided supporting packet-level evidence.

---

## 13. Detection Limitations

The detection created in this project was designed for the controlled lab environment.

It specifically searched for:

```text
c2-sync.lab
```

Therefore, it is **not a general-purpose DNS tunneling detector**.

A production environment would require additional analysis such as:

- unusual DNS query frequency
- unusually long DNS names
- encoded or random-looking subdomains
- unusual processes generating DNS traffic
- repeated DNS communication patterns
- known suspicious domains or infrastructure

This project focused on understanding the basic process of creating, detecting, and investigating controlled DNS C2 activity.

---

## 14. MITRE ATT&CK Mapping

### T1071.004 — DNS

DNS can be used as an application-layer protocol for Command and Control.

This project demonstrates DNS being used to exchange commands and command results between the controlled Windows client and Ubuntu server.

**Reference:**  
[MITRE ATT&CK — T1071.004: DNS](https://attack.mitre.org/techniques/T1071/004/)

### T1132.001 — Standard Encoding

The project used Base32 encoding for task and result data transferred through DNS.

**Reference:**  
[MITRE ATT&CK — T1132.001: Standard Encoding](https://attack.mitre.org/techniques/T1132/001/)

---

## 15. Skills Demonstrated

### Windows & Sysmon

- Windows endpoint monitoring
- Sysmon Event ID 22
- Basic process analysis
- DNS activity analysis

### Splunk

- Log searching
- SPL filtering
- SPL statistics
- DNS activity detection
- Scheduled alerts
- Alert investigation

### Network Analysis

- DNS C2 concepts
- DNS query analysis
- Base32-encoded data identification
- Basic Wireshark analysis

### Investigation

- Reviewing DNS events
- Following an alert
- Checking related process activity
- Building a simple investigation timeline

---

## 16. Evidence

The project contains 19 screenshots documenting the lab setup, DNS activity, Splunk investigation, alert, and packet analysis.

All screenshots are available in the [`evidence/`](evidence/) directory.

### Main Evidence

| Screenshot | Description |
|---|---|
| [01 — DNS Baseline Query Frequency](evidence/01_DNS_Baseline_Query_Frequency.png) | DNS baseline query frequency |
| [03 — DNS Baseline Process/Query Relationship](evidence/03_DNS_Baseline_Process_Query_Relationship.png) | DNS process/query relationship |
| [06 — DNS C2 Server Task and Result](evidence/06_DNS_C2_Server_Task_Result.png) | Controlled DNS C2 server activity |
| [07 — DNS C2 Client Task Execution](evidence/07_DNS_C2_Client_Task_Execution.png) | DNS C2 client task execution |
| [09 — Splunk DNS C2 Events](evidence/09_Splunk_DNS_C2_Events.png) | Splunk DNS C2 events |
| [10 — Splunk DNS Process/Query Relationship](evidence/10_Splunk_DNS_Process_Query_Relationship.png) | DNS process/query analysis |
| [13 — Splunk DNS C2 Alert Triggered](evidence/13_Splunk_DNS_C2_Alert_Triggered.png) | Triggered Splunk alert |
| [14 — Raw DNS C2 Result Event](evidence/14_Splunk_Raw_DNS_C2_Result_Event.png) | Raw DNS result event |
| [15 — DNS C2 Check-in](evidence/15_Splunk_DNS_C2_Checkin_Event.png) | DNS C2 check-in |
| [16 — Encoded Payload Extraction](evidence/16_Encoded_Payload_Extraction.png) | Encoded data extraction |
| [17 — Systeminfo Process Execution](evidence/17_Systeminfo_Process_Execution.png) | Systeminfo process execution |
| [19 — Wireshark DNS C2 Exchange](evidence/19_Wireshark_DNS_C2_Exchange.png) | Wireshark DNS C2 exchange |

Additional supporting screenshots are available in the [`evidence/`](evidence/) directory.

---

## 17. Repository Structure

```text
SOC-Case-3-DNS-C2-Detection/
│
├── README.md
│
├── client/
│   └── dns_c2_client.ps1
│
├── server/
│   └── dns_c2_server_controlled.py
│
├── evidence/
│   ├── 01_DNS_Baseline_Query_Frequency.png
│   ├── 02_DNS_Baseline_Process_Frequency.png
│   ├── 03_DNS_Baseline_Process_Query_Relationship.png
│   ├── 04_Windows_Ubuntu_DNS_Test.png
│   ├── 05_DNS_C2_Server_Initial.png
│   ├── 06_DNS_C2_Server_Task_Result.png
│   ├── 07_DNS_C2_Client_Task_Execution.png
│   ├── 08_Sysmon_DNS_C2_Events.png
│   ├── 09_Splunk_DNS_C2_Events.png
│   ├── 10_Splunk_DNS_Process_Query_Relationship.png
│   ├── 11_Splunk_DNS_C2_Unique_Sessions.png
│   ├── 12_Splunk_DNS_C2_Alert_Enabled.png
│   ├── 13_Splunk_DNS_C2_Alert_Triggered.png
│   ├── 14_Splunk_Raw_DNS_C2_Result_Event.png
│   ├── 15_Splunk_DNS_C2_Checkin_Event.png
│   ├── 16_Encoded_Payload_Extraction.png
│   ├── 17_Systeminfo_Process_Execution.png
│   ├── 18_DNS_C2_Base32_Decoding.png
│   └── 19_Wireshark_DNS_C2_Exchange.png
│
└── pcap/
    └── DNS_C2_Wireshark_Capture.pcapng
```

---

## 18. Conclusion

This project provided hands-on practice with a controlled DNS-based C2 scenario.

The activity was generated between a Windows endpoint and an Ubuntu server, recorded through Sysmon, forwarded to Splunk, detected using SPL, and investigated through DNS and process events.

Wireshark was also used to validate the network communication.

The project provided practical beginner-level experience with:

- Windows telemetry
- Sysmon
- Splunk
- SPL
- DNS analysis
- Alert investigation
- Basic network analysis

---

## Note:

This project was created in an isolated virtual lab for educational purposes.

The DNS C2 client was intentionally limited to predefined, benign commands. No unauthorized systems or real-world infrastructure were targeted.

