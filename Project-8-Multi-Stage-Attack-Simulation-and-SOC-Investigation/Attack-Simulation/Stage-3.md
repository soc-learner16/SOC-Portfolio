
# Stage 3 – Scheduled Task Persistence Simulation

## 1. Overview

In this stage, I performed a controlled scheduled task persistence simulation on the Windows 10 endpoint.

I created a PowerShell script and configured a Windows Scheduled Task to execute it at regular two-minute intervals.

The script recorded its execution timestamp, username, and computer name in a local log file.

The objective was to understand how scheduled task execution works and investigate the resulting activity using Windows Task Scheduler logs, Sysmon, and Elastic Security.

## 2. Lab Environment

**Operating System:** Windows 10

**Hostname:** `DESKTOP-B6GVH9G`

**User Account:** `kirth`

**Monitoring Tool:** Microsoft Sysmon

**SIEM Platform:** Elastic Security

**PowerShell Script:** `Stage3-Task.ps1`

**Scheduled Task Name:** `Stage3-Lab-Persistence`

## 3. PowerShell Script Creation

I created a working directory at `C:\Lab\Stage3` and prepared a PowerShell script named `Stage3-Task.ps1`.

The script was designed to record the following information whenever it executed:

- Execution timestamp.
- Current username.
- Computer name.

The execution details were saved in:

`C:\Lab\Stage3\execution.log`

I used PowerShell's `Get-Date` and `Add-Content` functionality to record the execution information.

### Evidence 1 – PowerShell Script Creation

![PowerShell Script Creation](<./Screenshots/S3-01 – PowerShell script.png>)

## 4. Scheduled Task Creation

After preparing the PowerShell script, I created a Windows Scheduled Task named `Stage3-Lab-Persistence`.

The task was configured to execute the PowerShell script every two minutes.

I used the following command:

`schtasks /Create /TN "Stage3-Lab-Persistence" /SC MINUTE /MO 2 /TR "powershell.exe -NoProfile -ExecutionPolicy Bypass -File C:\Lab\Stage3\Stage3-Task.ps1" /F`

The command returned a success message confirming that the scheduled task was created.

### Evidence 2 – Scheduled Task Creation

![Scheduled Task Creation](<./Screenshots/S3-02 – Scheduled Task creation.png>)

## 5. Scheduled Task Configuration Verification

I verified the scheduled task configuration using:

`schtasks /Query /TN "Stage3-Lab-Persistence" /V /FO LIST`

The output showed the task name, hostname, run-as user, task status, execution action, and configured repetition interval.

The important configuration details were:

- Task Name: `\Stage3-Lab-Persistence`
- Hostname: `DESKTOP-B6GVH9G`
- Run As User: `kirth`
- Task Status: `Ready`
- Execution Interval: Every 2 minutes
- Action: Execute `Stage3-Task.ps1` using PowerShell.

This confirmed that the task was configured with the intended execution interval and script path.

### Evidence 3 – Scheduled Task Details

![Scheduled Task Details](<./Screenshots/S3-03-Task details.png>)

## 6. Scheduled Task Execution Verification

After verifying the configuration, I manually triggered the task for testing.

I used:

`schtasks /Run /TN "Stage3-Lab-Persistence"`

I then waited for the task to execute and examined the generated log using:

`Get-Content C:\Lab\Stage3\execution.log`

The output displayed multiple execution entries containing timestamps, username `kirth`, and computer name `DESKTOP-B6GVH9G`.

This verified that the PowerShell script executed and recorded its activity in the log file.

### Evidence 4 – Scheduled Task Execution Log

![Scheduled Task Execution Log](<./Screenshots/S3-04 – Execution log.png>)

## 7. Windows Task Scheduler Event Investigation

I examined the Windows Task Scheduler Operational event log to investigate the scheduled task activity.

The investigation focused on:

- **Event ID 200:** Task Scheduler launched the task action.
- **Event ID 201:** Task Scheduler recorded completion of the task action.

The captured records showed the `powershell.exe` action associated with `Stage3-Lab-Persistence`.

The event messages also displayed task execution return codes. The captured events included different return-code values, so individual task results were reviewed rather than assuming every execution completed successfully.

### Evidence 5 – Task Scheduler Events

![Task Scheduler Events](<./Screenshots/S3-05 – Task Scheduler events.png>)

## 8. Sysmon Process Creation Investigation

I examined Sysmon Event ID 1 to identify the processes associated with scheduled task creation and execution.

The captured evidence included:

- `schtasks.exe` process creation during scheduled task configuration.
- `powershell.exe` process creation associated with `Stage3-Task.ps1`.
- The PowerShell command line referencing the Stage 3 script.
- Parent process information associated with the Task Scheduler service.

The parent process information helped establish the relationship between Windows Task Scheduler and the PowerShell process.

### Evidence 6 – Sysmon Event ID 1

![Sysmon Event ID 1](<./Screenshots/S3-06 – Sysmon Event ID 1.png>)

## 9. Elastic Security Investigation

After reviewing the local Windows events, I investigated the related Sysmon process creation records in Elastic Security using ES|QL.

I used the following query:

FROM logs-windows.sysmon_operational-*
| WHERE event.code == "1"
    AND process.name == "powershell.exe"
    AND process.command_line LIKE "*Stage3-Task.ps1*"

The query filtered process creation events associated with the Stage 3 PowerShell script.

I reviewed the available fields, including:

- Timestamp.
- Hostname.
- Event code.
- Process name.
- Process ID.
- Parent process name.
- Process command line.
- Username.

The captured Elastic investigation view displayed eight results matching the query.

This provided SIEM-level evidence of the PowerShell process activity associated with the scheduled task.

### Evidence 7 – Elastic Scheduled Task Investigation

![Elastic Scheduled Task Investigation](<./Screenshots/S3-07_Elastic_Scheduled_Task_Investigation.png>)

## 10. Investigation Findings

The Stage 3 investigation demonstrated:

- Creation of the `Stage3-Task.ps1` PowerShell script.
- Successful creation of the `Stage3-Lab-Persistence` scheduled task.
- Configuration of a two-minute execution interval.
- Verification of the task configuration.
- Multiple execution entries in `execution.log`.
- Task Scheduler Event IDs 200 and 201.
- Sysmon Event ID 1 process creation evidence.
- Elastic investigation of the associated PowerShell activity.

The combined evidence demonstrated how recurring PowerShell execution through Windows Scheduled Tasks can be examined during a security investigation.

## 11. Conclusion

In this stage, I implemented a controlled scheduled task persistence simulation on the Windows 10 endpoint.

I created a PowerShell script, configured a scheduled task with a two-minute execution interval, and verified its execution through local log records.

I then examined Task Scheduler events, investigated the related Sysmon process creation activity, and used ES|QL in Elastic Security to review the associated events.

This exercise provided practical experience in Windows Scheduled Task investigation, PowerShell process monitoring, Sysmon event analysis, and SIEM-based investigation.

All activities were performed in a controlled virtual lab environment for educational and defensive security learning purposes.
