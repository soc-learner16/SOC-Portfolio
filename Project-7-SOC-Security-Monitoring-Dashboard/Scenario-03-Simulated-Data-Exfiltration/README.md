# Scenario 3 – Simulated Data Exfiltration Through SSH

## 1. Introduction

In this scenario, I performed a controlled file-transfer exercise between an Ubuntu Server and Kali Linux using SSH and SCP.

The objective was to simulate a file transfer that could represent a data exfiltration pattern and investigate the associated network communication using Elastic Security.

I first connected from Kali Linux to the Ubuntu Server through SSH and verified the test file stored on the Ubuntu Server. I calculated the SHA-256 hash of the test file before transferring it.

I then used SCP to copy the test file from the Ubuntu Server to Kali Linux. After the transfer, I investigated the related SSH network activity in Elastic to identify the source, destination, protocol, and destination port.

This exercise helped me understand how network-flow information can support the investigation of suspicious file-transfer activity.

## 2. Objectives

The main objectives of this scenario were:

- Perform a controlled SSH file-transfer simulation.
- Transfer a test file from Ubuntu Server to Kali Linux using SCP.
- Calculate the SHA-256 hash of the test file.
- Investigate the associated SSH network communication in Elastic.
- Identify the source IP address and destination IP address.
- Review the destination port and network protocol.
- Understand how network-flow information can support a data-transfer investigation.

## 3. Lab Environment

**Source System:** Kali Linux

**Destination System:** Ubuntu Server

**SIEM Platform:** Elastic Cloud (ELK Stack)

**Network Investigation:** Kibana Discover

**File Transfer Protocol:** SCP over SSH

**Network Protocol:** TCP

**Destination Port:** 22

### Lab Network Details

- Kali Linux IP Address: `192.168.10.6`
- Ubuntu Server IP Address: `192.168.10.7`
- Destination Host: `ubuntuserver`
- Destination Port: `22`

### Test File Details

The test file used for this exercise was:

`/home/kirthi/exfil_lab/soc_test_data.txt`

The file was used as harmless test data for the controlled transfer simulation.

## 4. SSH File-Transfer Simulation

I first connected from Kali Linux to the Ubuntu Server through SSH and checked the test file located at:

`/home/kirthi/exfil_lab/soc_test_data.txt`

Before transferring the file, I calculated its SHA-256 hash.

After verifying the test file, I used the following SCP command from Kali Linux to copy the file from the Ubuntu Server to the Kali machine:

`scp kirthi@192.168.10.7:/home/kirthi/exfil_lab/soc_test_data.txt ~/`

The transfer completed successfully.

This demonstrated a controlled file transfer from the Ubuntu Server to Kali Linux through SSH.

### Evidence 1 – SSH File-Transfer Simulation

![SSH File Transfer Simulation](./Screenshots/01-SSH-File-Transfer-Simulation.png)

## 5. Network Traffic Investigation in Elastic

After completing the file-transfer exercise, I investigated the related network activity in Elastic.

I reviewed the network events to examine the communication between Kali Linux and the Ubuntu Server.

The observed network event details included:

- Source IP Address: `192.168.10.6`
- Destination IP Address: `192.168.10.7`
- Destination Port: `22`
- Protocol: `TCP`
- Destination Host: `ubuntuserver`

The event details showed SSH network communication between Kali Linux and the Ubuntu Server.

This helped me examine the network activity associated with the controlled file-transfer exercise.

### Evidence 2 – Kali to Ubuntu SSH Network Flow in Elastic

![Kali to Ubuntu SSH Network Flow in Elastic](./Screenshots/02-Kali-Ubuntu-SSH-Network-Flow-in-Elastic.png)

## 6. Network Investigation Findings

The investigation identified the SSH communication between the two lab systems.

The observed flow details were:

- Source: Kali Linux (`192.168.10.6`)
- Destination: Ubuntu Server (`192.168.10.7`)
- Destination Port: `22`
- Transport Protocol: `TCP`
- Destination Host: `ubuntuserver`

The network-flow information helped identify the systems involved in the communication and the service used for the file-transfer exercise.

This information can be useful during a SOC investigation when reviewing unexpected or suspicious file-transfer activity.


## 7. Conclusion

In this scenario, I performed a controlled SSH file-transfer simulation between an Ubuntu Server and Kali Linux using SCP.

I verified the test file, calculated its SHA-256 hash, and successfully transferred it from the Ubuntu Server to Kali Linux.

I then investigated the associated network communication in Elastic and identified the source IP, destination IP, destination port, protocol, and destination host.

Finally, I integrated the relevant network-monitoring visualization into the SOC Security Monitoring Dashboard alongside the SSH brute-force and privilege escalation monitoring panels.

This exercise provided practical experience in SSH file transfers, network-flow investigation, Elastic event analysis, and centralized SOC dashboard monitoring.

All activities were performed in a controlled lab environment using harmless test data for educational and defensive security learning purposes.

**Source System:** Kali Linux  
**Destination System:** Ubuntu Server  
**Purpose:** Cybersecurity Learning and Defensive Security Monitoring
