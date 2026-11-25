
# Project 8 – Multi-Stage Attack Simulation and SOC Investigation

## Project Overview

This project demonstrates a controlled multi-stage attack simulation and defensive security investigation using a virtual cybersecurity lab environment.

The project was designed to examine how an attacker-like sequence of activities can progress across Windows and Linux systems, and how the resulting endpoint and network telemetry can be investigated from a Security Operations Center (SOC) perspective.

The simulation covered six stages, beginning with a harmless security-awareness web page and continuing through PowerShell HTTP communication, scheduled task persistence, UAC-approved privilege elevation, SSH-based lateral movement, and controlled file transfer.

Following the simulation, I performed a separate detection and investigation phase using Microsoft Sysmon, Suricata, Wireshark, and Elastic Security.

The project uses the **Lockheed Martin Cyber Kill Chain** as a high-level model for discussing attack progression and **MITRE ATT&CK** as a reference for mapping selected observed behaviours to documented adversary techniques.

The main focus was not simply to execute simulated activities, but to understand what evidence each activity generated, where that evidence could be collected, and how a SOC analyst could investigate it.

> **Project Scope:** Controlled APT-style attack simulation, endpoint monitoring, network investigation, and SIEM-based analysis.

## 1. Project Objectives

- Simulate a sequence of security activities across Windows and Linux systems.
- Understand the progression of a multi-stage attack scenario.
- Examine PowerShell execution and HTTP communication.
- Demonstrate scheduled task-based recurring execution.
- Investigate Windows UAC-approved privilege elevation.
- Simulate SSH-based lateral movement between lab systems.
- Perform controlled file transfer using harmless test data.
- Collect endpoint, server, and network evidence.
- Investigate Sysmon telemetry using Elastic Security and ES|QL.
- Understand how the observed activities relate to Cyber Kill Chain and MITRE ATT&CK concepts.

## 2. Lab Environment

The project was performed using three virtual machines configured through Oracle VM VirtualBox.

**Kali Linux – Attacker Machine**

Used to host the security-awareness test page, receive HTTP heartbeat requests, and receive the final controlled HTTPS file upload.

**Windows 10 – Victim Endpoint**

Used for the simulated endpoint activities, including PowerShell execution, scheduled task creation, UAC-approved elevation, SSH access, and file transfer.

**Ubuntu Server – Target Server and Network Monitoring**

Used as the SSH destination, source of the harmless test file, and network monitoring environment through Suricata.

### Lab Network

The documented Host-Only IP addresses were:

- Kali Linux: `192.168.56.102`
- Windows 10: `192.168.56.3`
- Ubuntu Server: `192.168.56.4`

### Security Monitoring Tools

**Sysmon:** Collected Windows process creation and network connection telemetry.

**Suricata:** Monitored network traffic and generated EVE JSON records.

**Wireshark:** Captured and inspected HTTP and SSH network packets.

**Elastic Security:** Provided the environment for searching and investigating available security telemetry.

**Kibana Discover and ES|QL:** Used to filter, query, summarize, and examine investigation records.

For the detailed lab configuration, refer to:

[Lab Environment and Architecture](./Lab-Environment/lab-architecture.md)

## 3. Attack Simulation – Stage 1 to Stage 6

The simulation was divided into six stages. Each stage generated supporting evidence that was examined through the relevant system logs and monitoring tools.

### Stage 1 – Security Awareness Web Simulation and HTTP Monitoring

I created a harmless HTML page titled **Security Awareness Simulation** on Kali Linux.

The page explicitly stated that it was an awareness-testing page and that no credentials or personal information were collected.

I hosted the page using Python's HTTP server on port `8000`, bound to the Kali Linux address `192.168.56.102`.

The Windows endpoint accessed the page through its browser.

I then examined the resulting HTTP activity using:

- Kali Linux HTTP access logs.
- Ubuntu Suricata EVE JSON records.
- Elastic Security investigation.

The HTTP access log recorded a `GET /` request from Windows IP `192.168.56.3` with a successful `200 OK` response.

Suricata and Elastic provided additional visibility into the related network activity.

**Key Finding:** The web page was accessed successfully, and the associated HTTP communication was visible through server logs, Suricata, and Elastic.

This was a web-access awareness simulation, not a confirmed phishing compromise or successful initial access into the endpoint.

[View Stage 1 – Detailed Documentation](./Attack-Simulation/Stage-1.md)

### Stage 2 – PowerShell HTTP Heartbeat Simulation

I created and executed a PowerShell script named `Stage2-Heartbeat.ps1` on the Windows endpoint.

The script generated five periodic HTTP heartbeat requests to a Python HTTP server hosted on Kali Linux.

The server listened on port `8080` and recorded the requests from Windows.

The successful heartbeat responses returned HTTP `200 OK`.

I investigated the generated activity using:

- Windows Sysmon Event ID 1 – Process Creation.
- Windows Sysmon Event ID 3 – Network Connection.
- Kali Linux HTTP access logs.
- Wireshark packet capture.
- Ubuntu Suricata HTTP records.
- Elastic network investigation.

The observed connection involved:

- Source: `192.168.56.3`
- Destination: `192.168.56.102`
- Destination Port: `8080`
- Protocol: TCP
- Initiating Process: `powershell.exe`

The investigation demonstrated how periodic HTTP communication initiated by PowerShell can be observed through endpoint and network telemetry.

**Key Finding:** Five successful heartbeat requests were recorded, and the associated PowerShell process and network activity were visible in the monitoring evidence.

This simulated a heartbeat or beacon-like pattern. It did not establish an interactive command-and-control channel.

[View Stage 2 – Detailed Documentation](./Attack-Simulation/Stage-2.md)

### Stage 3 – Scheduled Task Persistence Simulation

I created a PowerShell script named `Stage3-Task.ps1` on the Windows endpoint.

The script recorded its execution timestamp, username, and computer name in `C:\Lab\Stage3\execution.log`.

I then created a Windows Scheduled Task named `Stage3-Lab-Persistence`.

The task was configured to execute the PowerShell script every two minutes using Windows Task Scheduler.

I verified the configuration and execution through:

- `schtasks` configuration output.
- PowerShell execution log.
- Task Scheduler Event ID 200.
- Task Scheduler Event ID 201.
- Sysmon Event ID 1.
- Elastic process creation investigation.

The task execution events included both successful and non-zero return codes, which were reviewed individually.

**Key Finding:** The scheduled task was created with a two-minute interval, and its associated PowerShell process activity was recorded in Windows logs and investigated in Elastic.

[View Stage 3 – Detailed Documentation](./Attack-Simulation/Stage-3.md)

### Stage 4 – UAC-Approved Privilege Elevation Simulation

I prepared a PowerShell script named `Stage4-Elevation.ps1` to collect privilege-context information.

The script recorded the timestamp, username, computer name, and the output of `whoami /all`.

I examined the Windows User Account Control (UAC) prompt and compared standard and elevated PowerShell sessions.

The comparison showed:

- Standard PowerShell session: `Medium Mandatory Level`
- Elevated PowerShell session: `High Mandatory Level`

I also investigated the associated process creation records using Sysmon Event ID 1 and Elastic Security.

**Key Finding:** The UAC approval prompt and Medium-versus-High integrity comparison provided evidence of the change in Windows process security context.

This was a controlled, user-approved elevation exercise. It was not a UAC bypass or exploitation of a privilege-escalation vulnerability.

[View Stage 4 – Detailed Documentation](./Attack-Simulation/Stage-4.md)

### Stage 5 – SSH-Based Lateral Movement Simulation

I initiated an SSH connection from the Windows 10 endpoint to Ubuntu Server using the account `kirthi`.

The connection was established using:

`ssh kirthi@192.168.56.4`

After successful authentication, I verified the remote username, hostname, and system time.

I then examined the connection through:

- Ubuntu SSH service status.
- Ubuntu authentication logs.
- Suricata network-flow records.
- Windows Sysmon Event ID 3.
- Wireshark SSH packet capture.
- Elastic network investigation.

The observed connection details were:

- Source IP: `192.168.56.3`
- Destination IP: `192.168.56.4`
- Source Port: `50639`
- Destination Port: `22`
- Protocol: TCP
- Process: `ssh.exe`

Ubuntu authentication logs recorded successful password authentication for `kirthi` from the Windows endpoint.

Wireshark showed the TCP connection, SSH protocol negotiation, key exchange, and encrypted SSH packets.

**Key Finding:** The SSH connection from Windows to Ubuntu was successfully established and supported by endpoint, server-side, and network-level evidence.

[View Stage 5 – Detailed Documentation](./Attack-Simulation/Stage-5.md)

### Stage 6 – Simulated Data Exfiltration Through HTTPS

I performed a controlled file-transfer exercise using harmless test data across Ubuntu Server, Windows 10, and Kali Linux.

First, I created `stage6_test_data.txt` on Ubuntu Server and calculated its SHA-256 hash.

The file was transferred from Ubuntu Server to Windows using SCP.

I verified the downloaded file on Windows using PowerShell's `Get-FileHash` command.

Next, I uploaded the file from Windows to a custom HTTPS receiver running on Kali Linux at `192.168.56.102:8443`.

The Kali receiver recorded:

- Source IP: `192.168.56.3`
- Received File: `stage6_test_data.txt`
- Received Size: `184 bytes`
- Upload Endpoint: `/upload`
- HTTP Response: `200 OK`

I then calculated the SHA-256 hash of the received file on Kali Linux.

The original, Windows-transferred, and Kali-received file hashes matched.

**Observed SHA-256 Hash:**

`197b82dceecf6d3796cc7eb6dc0689e20610792607e524d62908ce459649947b`

**Key Finding:** The controlled file-transfer process completed successfully, and matching SHA-256 hashes verified the integrity of the test file.

This exercise simulated a data-transfer pattern using harmless dummy data. It was not a confirmed malicious data-exfiltration incident.

[View Stage 6 – Detailed Documentation](./Attack-Simulation/Stage-6.md)

## 4. Cyber Kill Chain Alignment

The Lockheed Martin Cyber Kill Chain provides a high-level model for understanding the progression of cyber intrusion activity through seven phases.

I used this framework to interpret the simulation sequence and identify which activities resembled particular phases.

The mapping below is an analytical alignment of the lab exercises. It does not mean that every Cyber Kill Chain phase was fully executed.

### 1. Reconnaissance

**Status: Not performed as a dedicated activity.**

The project did not include a separate reconnaissance exercise such as external target discovery, vulnerability scanning, or infrastructure profiling.

### 2. Weaponization

**Status: Not performed.**

The lab used harmless HTML and PowerShell scripts created for testing. No malicious payload, exploit package, or weaponized document was developed.

### 3. Delivery

**Related Activity: Stage 1 – Security Awareness Web Simulation**

A harmless awareness page was hosted on Kali Linux and accessed from the Windows endpoint.

This provided a limited web-page delivery/access simulation. No phishing email delivery mechanism was implemented.

### 4. Exploitation

**Status: Not performed.**

The project did not exploit a software vulnerability or execute an exploit payload to gain unauthorized access.

The UAC-approved elevation exercise in Stage 4 was a separate privilege-context demonstration, not exploitation.

### 5. Installation

**Related Activity: Stage 3 – Scheduled Task Persistence Simulation**

A recurring Windows Scheduled Task was created to execute a PowerShell script at two-minute intervals.

This represented a persistence-related activity similar to the installation/persistence phase concept. No malware or persistent malicious implant was installed.

### 6. Command and Control

**Related Activity: Stage 2 – PowerShell HTTP Heartbeat Simulation**

The Windows endpoint generated periodic HTTP requests to the Kali HTTP server.

This demonstrated a beacon-like communication pattern over HTTP.

However, no interactive command execution or remote command-and-control functionality was implemented.

### 7. Actions on Objectives

**Related Activity: Stage 6 – Simulated Data Exfiltration**

A harmless test file was transferred across the lab systems and verified using SHA-256 hashing.

This represented a controlled simulation of data movement towards a receiving system. No actual sensitive information was collected or stolen.

### Additional Activities Across the Sequence

Stage 4's UAC-approved privilege elevation and Stage 5's SSH-based lateral movement were included as supporting activities in the overall simulation.

These activities help describe the progression of the lab exercise, but they are not separate named phases in the seven-stage Cyber Kill Chain model.

**Framework Reference:** [Lockheed Martin – Cyber Kill Chain](https://www.lockheedmartin.com/en-us/capabilities/cyber/cyber-kill-chain.html)

## 5. MITRE ATT&CK Framework Alignment

MITRE ATT&CK provides a knowledge base of adversary tactics and techniques.

For this project, I used ATT&CK as a reference to identify selected behaviours observed during the controlled simulations.

The following mappings are behavioural references, not claims that a real threat actor or APT group was emulated.

### PowerShell Execution – T1059.001

**Related Stages:** Stage 2, Stage 3, and Stage 4.

PowerShell scripts were executed on the Windows endpoint for heartbeat communication, scheduled task activity, and privilege-context collection.

The observed script execution is relevant to:

[MITRE ATT&CK T1059.001 – PowerShell](https://attack.mitre.org/techniques/T1059/001/)

### Web Protocol Communication – T1071.001

**Related Stage:** Stage 2.

The Windows endpoint generated periodic HTTP heartbeat requests to the Kali HTTP server.

This provides a behavioural reference to web-protocol communication described by:

[MITRE ATT&CK T1071.001 – Application Layer Protocol: Web Protocols](https://attack.mitre.org/techniques/T1071/001/)

The exercise demonstrated HTTP beacon-like traffic only. It did not establish a functional C2 channel, so this mapping is not presented as confirmed malicious C2 activity.

### Scheduled Task – T1053.005

**Related Stage:** Stage 3.

The Windows Scheduled Task `Stage3-Lab-Persistence` was configured to execute a PowerShell script every two minutes.

This behaviour aligns with:

[MITRE ATT&CK T1053.005 – Scheduled Task](https://attack.mitre.org/techniques/T1053/005/)

### Remote Services: SSH – T1021.004

**Related Stage:** Stage 5.

The Windows endpoint established an SSH session with Ubuntu Server using a valid lab account.

The remote access behaviour aligns with:

[MITRE ATT&CK T1021.004 – Remote Services: SSH](https://attack.mitre.org/techniques/T1021/004/)

### Activities Without a Confirmed Technique Mapping

**Stage 1 – Security Awareness Page**

The evidence shows access to a harmless web page. It does not show a phishing email, malicious link delivery, exploit, or credential capture. Therefore, no specific initial-access technique is assigned.

**Stage 4 – UAC-Approved Elevation**

The evidence shows an approved UAC prompt and a change in integrity level.

This should not be mapped to T1548.002 (Bypass User Account Control), because no bypass was performed.

**Stage 6 – Controlled File Transfer**

The exercise demonstrated dummy-file movement over SCP and HTTPS. It is relevant to the general concept of data transfer, but no specific exfiltration sub-technique is assigned because the exercise did not establish a separate C2 channel or a real malicious exfiltration operation.

**Stage 7 – Detection and Investigation**

Stage 7 documents defensive event analysis and SIEM investigation, rather than an additional adversary technique.

**Framework Reference:** [MITRE ATT&CK Enterprise](https://attack.mitre.org/)

## 6. Detection and Investigation – Stage 7

After completing the six simulation stages, I performed a separate investigation of the Windows endpoint telemetry using Sysmon and Elastic Security.

The investigation focused on three areas:

- Sysmon event-code distribution.
- Process creation analysis using Event ID 1.
- Network connection analysis using Event ID 3.

### Sysmon Event Code Summary

I used ES|QL to summarize available Sysmon event records by event code.

The captured investigation view showed approximately 12.1K documents processed and 17 event-code groups.

This provided an overview of the available endpoint telemetry.

### Process Creation Investigation

I examined Sysmon Event ID 1 and investigated the process creation records.

The captured records included:

- Process: `curl.exe`
- Executable: `C:\Windows\System32\curl.exe`
- Parent Process: `powershell.exe`
- User: `kirth`
- Host: `desktop-b6gvh9g`

This helped examine parent-child process relationships and the execution context of the process.

### Network Connection Investigation

I queried Sysmon Event ID 3 to investigate connections involving the Kali Linux and Ubuntu Server addresses.

The captured Elastic results showed:

**PowerShell to Kali Linux**

- Process: `powershell.exe`
- Destination IP: `192.168.56.102`
- Destination Port: `8080`
- Grouped Event Count: `27`

**SSH to Ubuntu Server**

- Process: `ssh.exe`
- Destination IP: `192.168.56.4`
- Destination Port: `22`
- Grouped Event Count: `2`

The query processed 29 documents and returned two grouped results in the captured view.

These results helped correlate the Windows endpoint's network activity with the PowerShell heartbeat and SSH communication exercises.

The event counts represent matching records in the selected query and time range. They should not be interpreted as an exact number of HTTP requests or SSH sessions.

### Investigation Documentation

[View Stage 7 – Detailed Sysmon and Elastic Investigation](./Detection-and-Investigation/Stage-7.md)

## 7. Overall Project Findings

The project demonstrated how different stages of simulated activity can generate evidence across endpoint, server, and network monitoring sources.

The main findings included:

**Web Activity**

The Windows endpoint accessed the harmless awareness page hosted on Kali Linux, and the resulting HTTP request was visible in server logs and Suricata records.

**PowerShell Activity**

The Windows endpoint generated HTTP heartbeat requests. Sysmon process and network records provided endpoint visibility into the activity.

**Persistence-Related Activity**

The scheduled task was created and configured to execute repeatedly. Execution logs, Task Scheduler events, Sysmon, and Elastic supported the investigation.

**Privilege Context**

The UAC approval prompt and integrity-level comparison demonstrated the difference between standard and elevated process contexts.

**Lateral Movement**

The Windows-to-Ubuntu SSH connection was verified through authentication logs, Sysmon, Suricata, Wireshark, and Elastic.

**Controlled File Transfer**

The test file was transferred through the lab systems, and SHA-256 verification confirmed file integrity.

**SIEM Investigation**

Elastic ES|QL queries enabled event summarization, process investigation, and network connection analysis.

## 8. SOC Skills Demonstrated

Through this project, I gained practical experience in:

- Multi-stage security simulation.
- Windows PowerShell execution analysis.
- Windows Scheduled Task investigation.
- UAC and integrity-level analysis.
- SSH-based lateral movement investigation.
- File-transfer verification using SHA-256.
- Sysmon Event ID 1 and Event ID 3 analysis.
- Windows Event Log investigation.
- Linux SSH authentication log analysis.
- Suricata EVE JSON investigation.
- Wireshark packet analysis.
- Elastic Security and Kibana Discover.
- ES|QL query development.
- MITRE ATT&CK behavioural mapping.
- Cyber Kill Chain analysis.
- Endpoint and network evidence correlation.

## 9. Project Scope and Limitations

This project was conducted as a controlled educational simulation.

The following limitations are important when interpreting the results:

- No real external target was attacked.
- No real phishing email campaign was conducted.
- No credentials or personal information were collected through the awareness page.
- No software vulnerability was exploited.
- No UAC bypass was performed.
- The HTTP heartbeat did not implement interactive command-and-control functionality.
- The transferred file contained harmless dummy test data.
- Stage 7 demonstrated event investigation and query-based hunting; it did not document a separately configured automated detection rule or triggered alert.
- MITRE ATT&CK mappings represent selected behavioural similarities, not confirmed malicious techniques performed by a real threat actor.

These limitations distinguish the controlled lab activities from a real-world intrusion.

## 10. Conclusion

This project provided practical experience in simulating a multi-stage attack sequence and investigating the resulting security telemetry from a SOC analyst's perspective.

I performed six controlled simulation stages covering web-page access, PowerShell HTTP communication, scheduled task persistence, UAC-approved elevation, SSH-based lateral movement, and controlled data transfer.

I then investigated the available endpoint and network evidence using Sysmon, Suricata, Wireshark, and Elastic Security.

By relating selected activities to the Cyber Kill Chain and MITRE ATT&CK frameworks, I developed a better understanding of how attack behaviours can be organized, how security telemetry can be interpreted, and how a SOC analyst can correlate endpoint and network evidence during an investigation.

All activities were performed in a controlled virtual lab environment for educational and defensive security learning purposes.

---

**Project:** Multi-Stage Attack Simulation and SOC Investigation

**Environment:** Kali Linux, Windows 10, and Ubuntu Server

**Monitoring:** Sysmon and Suricata

**Investigation:** Wireshark and Elastic Security

**Frameworks:** Cyber Kill Chain and MITRE ATT&CK

**Project Focus:** Attack Simulation, Endpoint Monitoring, Network Investigation, and SOC Analysis
