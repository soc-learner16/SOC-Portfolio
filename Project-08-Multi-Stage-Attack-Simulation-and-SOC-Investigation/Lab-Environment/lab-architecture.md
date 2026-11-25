
# Lab Environment & Architecture

## 1. Lab Overview

This project was conducted in a controlled virtual cybersecurity lab to simulate multi-stage attack activities and investigate the resulting security events from a SOC analyst's perspective.

The lab environment consists of three virtual machines running through Oracle VM VirtualBox: Kali Linux, Windows 10, and Ubuntu Server.

Each machine was assigned a specific role in the simulation. Kali Linux was used as the attacker machine, Windows 10 represented the victim endpoint, and Ubuntu Server was used as the target server and network monitoring environment.

The lab combined endpoint monitoring, network traffic analysis, and SIEM-based investigation using Sysmon, Suricata, Wireshark, and Elastic Security.

## 2. Virtualization and Network Configuration

The virtual lab was created using **Oracle VM VirtualBox**.

A **Host-Only Adapter** was configured to provide private network communication between the virtual machines. This allowed the systems to communicate within the lab environment while keeping the simulation isolated from the physical network.

The documented Host-Only network addresses were:

- Kali Linux: `192.168.56.102`
- Windows 10: `192.168.56.3`
- Ubuntu Server: `192.168.56.4`

## 3. Virtual Machine Architecture

### Kali Linux – Attacker Machine

Kali Linux was used as the attacker machine to initiate controlled activities during the simulation.

It was involved in the initial testing activities, HTTP communication, and controlled file-transfer exercises.

The Kali machine also provided the environment for examining attacker-side logs and verifying communication with the other lab systems.

**Documented Host-Only IP:** `192.168.56.102`

### Windows 10 – Victim Endpoint

Windows 10 represented the victim endpoint in the simulation.

It was used for PowerShell execution, scheduled task persistence, privilege elevation simulation, and SSH-based lateral movement.

Microsoft Sysmon was used to collect endpoint telemetry, particularly process creation and network connection events.

Windows Event Viewer and PowerShell were used to review and verify the generated endpoint activity.

**Documented Host-Only IP:** `192.168.56.3`

### Ubuntu Server – Target Server and Network Monitoring

Ubuntu Server was used as the target system during SSH-based communication and lateral movement simulation.

It also served as the network monitoring environment through Suricata.

OpenSSH supported SSH connectivity, while Linux authentication logs provided information about SSH-related activity.

Suricata monitored network traffic and generated structured event records in EVE JSON format. These records were examined to investigate network communication and flow information.

**Documented Host-Only IP:** `192.168.56.4`

## 4. Network Architecture and Communication

The main lab communication flow followed this sequence:

**Kali Linux → Windows 10 → Ubuntu Server**

Kali Linux was used to initiate controlled simulation activities against the Windows endpoint.

Windows 10 was used to demonstrate endpoint activities and SSH-based lateral movement toward Ubuntu Server over TCP port 22.

The communication was investigated using Windows Sysmon events, Ubuntu SSH logs, Suricata records, and Wireshark packet captures.

A separate controlled file-transfer exercise was also performed from Ubuntu Server to Kali Linux using SCP and harmless test data.

The Host-Only network provided the private communication path required for these lab activities.

## 5. Security Monitoring and Investigation Tools

### Sysmon – Windows Endpoint Monitoring

Microsoft Sysmon was installed on Windows 10 to collect detailed endpoint activity.

The investigation focused on:

- **Event ID 1 – Process Creation:** Used to examine newly created processes, executable paths, command-line information, and parent process details.
- **Event ID 3 – Network Connection:** Used to investigate network connections initiated by processes.

Sysmon telemetry helped identify process execution and network activity associated with the simulated stages.

### Windows Event Viewer and PowerShell

Windows Event Viewer was used to inspect locally generated Windows and Sysmon events.

PowerShell, including the `Get-WinEvent` cmdlet, was used to retrieve and verify relevant event records from the Sysmon Operational log.

### Suricata – Network Security Monitoring

Suricata was used on Ubuntu Server to monitor network traffic and generate network security event records.

Its EVE JSON logs provided information about observed network activity, including timestamps, source and destination addresses, ports, protocols, and flow details.

These records supported the investigation of HTTP and SSH-related network activity during the simulation.

### Wireshark – Network Packet Analysis

Wireshark was used to capture and inspect network packets.

It helped examine TCP communication, HTTP activity, and SSH traffic between the lab systems.

Packet captures provided additional network-level evidence to support the investigation of the simulated activities.

### Elastic Security – SIEM Investigation

Elastic Security was used to search, filter, and investigate the available security telemetry.

Kibana Discover and ES|QL queries were used to examine relevant process creation events, network connection records, and other available investigation data.

The investigation focused on event details such as timestamps, process information, source and destination addresses, destination ports, and event counts.

Elastic Security provided a centralized interface for reviewing the collected investigation data.

## 6. Log Collection and Investigation Workflow

The investigation combined endpoint, server, and network evidence from the lab environment.

Windows Sysmon logs were reviewed to identify process creation and network connection activity.

Ubuntu authentication logs were examined to verify SSH-related activity, while Suricata EVE JSON records provided network monitoring information.

Wireshark packet captures were used for packet-level inspection.

Finally, Elastic Security was used to search and analyze the available event records using Kibana Discover and ES|QL queries.

This approach helped establish context between endpoint activity, server-side events, and observed network communication.

## 7. Lab Isolation and Safety

All activities were performed within a controlled VirtualBox lab environment.

The Host-Only network was used for private communication between the virtual machines.

The file-transfer simulation used harmless test data created specifically for the lab. No real confidential information or unauthorized external systems were involved.

The project was conducted for educational purposes, focusing on attack simulation, security telemetry, and defensive SOC investigation techniques.

## 8. Architecture Summary

The lab architecture combined three virtual machines and complementary security monitoring tools.

Kali Linux was used for controlled attack simulation, Windows 10 provided endpoint telemetry through Sysmon, and Ubuntu Server supported SSH communication and network monitoring through Suricata.

Wireshark provided packet-level visibility, while Elastic Security supported SIEM-based event investigation.

Together, these components provided practical experience in understanding multi-stage attack activity, examining endpoint and network evidence, and investigating security events from a SOC analyst's perspective.

The following project sections document each simulation stage, its supporting evidence, and the corresponding investigation results.
