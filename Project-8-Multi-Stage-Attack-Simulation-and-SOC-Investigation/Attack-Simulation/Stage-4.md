
# Stage 4 – UAC-Approved Privilege Elevation Simulation

## 1. Overview

In this stage, I performed a controlled Windows privilege elevation simulation using PowerShell and User Account Control (UAC).

I created a PowerShell script to collect privilege-context information from the Windows endpoint. I then examined the UAC elevation prompt and compared the security integrity levels of a standard PowerShell session and an elevated PowerShell session.

The resulting process activity was investigated using Sysmon Event ID 1 and Elastic Security.

The objective was to understand how privilege elevation changes the security context of a process and how related process execution activity can be investigated by a SOC analyst.

## 2. Lab Environment

**Operating System:** Windows 10

**Hostname:** `DESKTOP-B6GVH9G`

**User Account:** `kirth`

**Monitoring Tool:** Microsoft Sysmon

**SIEM Platform:** Elastic Security

**PowerShell Script:** `Stage4-Elevation.ps1`

**Investigation Event:** Sysmon Event ID 1 – Process Creation

## 3. PowerShell Script Creation

I created a PowerShell script named `Stage4-Elevation.ps1` on the Windows endpoint.

The script was designed to collect and record privilege-context information in a text file named `privilege-context.txt`.

The information included:

- Timestamp.
- Current username.
- Computer name.
- Privilege context.
- Output of the `whoami /all` command.

This helped document the security context under which the PowerShell script was executed.

### Evidence 1 – Stage 4 PowerShell Script

![Stage 4 PowerShell Script](<./Screenshots/S4-02 – Stage 4 PowerShell script.png>)

## 4. User Account Control (UAC) Elevation

I initiated the privilege elevation simulation on Windows and examined the User Account Control prompt.

The UAC prompt displayed a request for permission to allow Windows PowerShell to make changes to the device.

This demonstrated the Windows approval mechanism used when an application requests elevated privileges.

### Evidence 2 – Windows UAC Elevation Prompt

![Windows UAC Elevation Prompt](<./Screenshots/S4-03 – UAC elevation prompt.png>)

## 5. Medium and High Integrity-Level Comparison

After the elevation exercise, I compared the integrity levels of standard and elevated PowerShell sessions.

I used the following command:

`whoami /groups | findstr /i "Mandatory Label"`

The standard PowerShell session displayed:

`Medium Mandatory Level`

The elevated PowerShell session displayed:

`High Mandatory Level`

This comparison demonstrated the difference between the standard user security context and the elevated administrative context.

It helped verify the change in integrity level associated with the privilege elevation exercise.

### Evidence 3 – Medium vs High Integrity-Level Comparison

![Medium vs High Integrity Comparison](<./Screenshots/S4-05 – Medium vs High integrity comparison.png>)

## 6. Sysmon Process Creation Investigation

I examined the Sysmon Operational event log to identify the PowerShell processes associated with the Stage 4 activity.

The investigation focused on Sysmon Event ID 1, which records process creation activity.

The captured events included:

- Process Name: `powershell.exe`
- Executed Script: `Stage4-Elevation.ps1`
- User: `DESKTOP-B6GVH9G\kirth`
- Parent Process: `powershell.exe`

The event evidence also showed the execution of `whoami.exe` with the `/all` argument.

This helped identify the PowerShell script execution and the related privilege-context information collection activity.

### Evidence 4 – Sysmon Event ID 1

![Stage 4 Sysmon Event ID 1](<./Screenshots/S4-06 – Sysmon Event ID 1.png>)

## 7. Elastic Security Investigation

After reviewing the local Sysmon records, I investigated the related process creation activity in Elastic Security.

I used Kibana Discover and ES|QL to examine the available Sysmon Event ID 1 records associated with the Stage 4 PowerShell activity.

The investigation focused on:

- Event timestamp.
- Hostname.
- Event code.
- Process name.
- Process ID.
- Parent process information.
- Process command line.
- Username.

The captured Elastic investigation view displayed one matching result for the selected query.

This provided SIEM-level visibility into the PowerShell process execution associated with the privilege elevation simulation.

### Evidence 5 – Elastic Privilege Elevation Investigation

![Elastic Privilege Elevation Investigation](<./Screenshots/S4-08_Elastic_Privilege_Escalation_Investigation.png>)

## 8. Investigation Findings

The Stage 4 investigation demonstrated:

- Creation of the `Stage4-Elevation.ps1` PowerShell script.
- Collection of privilege-context information.
- Display of the Windows UAC elevation prompt.
- Comparison of Medium and High integrity levels.
- Sysmon Event ID 1 process creation records.
- Identification of the PowerShell script execution.
- Investigation of the related process event in Elastic Security.

The combined evidence demonstrated how Windows privilege elevation and associated process execution activity can be examined using endpoint telemetry and SIEM investigation.

## 9. Conclusion

In this stage, I performed a controlled privilege elevation simulation on the Windows 10 endpoint.

I created a PowerShell script to collect privilege-context information, examined the UAC elevation prompt, and compared the Medium and High integrity levels of PowerShell sessions.

I then investigated the related process creation activity using Sysmon Event ID 1 and Elastic Security.

This exercise provided practical experience in Windows privilege-context analysis, UAC investigation, Sysmon process monitoring, and SIEM-based security investigation.

All activities were performed in a controlled virtual lab environment for educational and defensive security learning purposes.
