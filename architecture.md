# Wazuh SOC Lab Architecture

[Back to README](README.md) | [Step-by-step guide](step-by-step-lab-guide.md)

## Overview

I used three virtual machines on a VMware lab network to bring Windows and Linux events into a central Wazuh server. Windows Security events, Windows Sysmon telemetry and file integrity changes could be investigated in Wazuh alongside SSH-related Linux activity.

## 2. Lab architecture

```mermaid
flowchart TB
    HOST["Host PC / VMware virtual network"]
    WIN["Windows 10 endpoint<br/>192.168.128.132<br/>Wazuh Agent + Sysmon"]
    LIN["Ubuntu Linux endpoint<br/>192.168.128.133<br/>Wazuh Agent"]
    MGR["Wazuh server (Ubuntu)<br/>192.168.128.131<br/>Manager + Indexer + Dashboard"]
    UI["Wazuh web dashboard<br/>Discover / Rules / FIM / Dashboards"]

    HOST --- WIN
    HOST --- LIN
    HOST --- MGR
    WIN -->|"Windows Event Logs + Sysmon + FIM"| MGR
    LIN -->|"Linux security logs + FIM"| MGR
    MGR --> UI
    MGR -.->|"Active response on configured host"| LIN
```

| Component | Lab IP | Purpose |
|---|---|---|
| Wazuh server | `192.168.128.131` | Central security-event collection, correlation, indexing and dashboard |
| Windows 10 VM | `192.168.128.132` | Windows event, account-management, Sysmon, login and FIM testing |
| Ubuntu Linux VM | `192.168.128.133` | Linux log collection, FIM and SSH authentication testing |

Wazuh agents forward telemetry from the endpoints to the central Wazuh deployment. On the Windows system, Sysmon gives extra host activity visibility alongside regular Windows event logs. On the Linux system, SSH and file events supplied useful detection data. The server processes events into searchable records and alerts.

**Evidence:** [01 — Ubuntu VM](#architecture-screenshots), [02 — SSH access](#architecture-screenshots), [03 — Wazuh installation](SCREENSHOT-GUIDE.md), [04 — initial dashboard](#lab-architecture), [12–15 — agent connections](step-by-step-lab-guide.md#connecting-the-windows-and-linux-agents).

<img width="1917" height="1039" alt="4 first login to wazuh" src="https://github.com/user-attachments/assets/704e5539-415c-4cac-a189-46961dac09a8" />

*Figure 04: Opened the Wazuh web dashboard for the first time to confirm access.*<img width="1913" height="696" alt="14  wazuh viw of agent connection with linux and windows machine" src="https://github.com/user-attachments/assets/8c44d434-819c-4017-bef0-168d7c022ee1" />

*Figure 14: Verified that both my Windows and Ubuntu agents were visible in Wazuh.*## How events travelled through the lab

1. **Windows VM (192.168.128.132):** Windows Security and Sysmon logs are collected by the agent. It also monitors the test `C:\CompanyData` folder.
2. **Ubuntu VM (192.168.128.133):** The Wazuh agent provides endpoint log and file-monitoring coverage, including Linux security activity.
3. **Wazuh server (192.168.128.131):** Receives events, evaluates rules and makes alerts available to search using the dashboard.
4. **Analyst investigation:** I used Discover for event fields and queries, the FIM interface for watched files, and dashboards for account activity and failed logons.
5. **Response:** Custom rule `100101` was linked to the `firewall-drop` active response. The alert page captured the resulting response message.

## Network and security notes

These are private lab IPs, not public-facing addresses. Connectivity uses the virtual network. Wazuh normally uses agent communication on TCP 1514 and agent enrolment on TCP 1515, although the screenshots do not prove these specific port settings. Sysmon collects endpoint details locally and Wazuh sends the resulting events to the manager. The SSH tests used port 22.

**Important:** A Wazuh active-response alert confirms that the configured response was invoked. Independent proof that a firewall rule blocked the remote host would require checking the actual firewall state and testing connectivity.

## Architecture screenshots

<img width="1916" height="1035" alt="1 installed ubuntu and named it wazah" src="https://github.com/user-attachments/assets/4e385c50-8263-42ce-80e0-bd9ff14c7ded" />

*Figure 01: Installed the Ubuntu virtual machine that I used to host the Wazuh server.*<img width="1110" height="624" alt="2 sshing into wazah server" src="https://github.com/user-attachments/assets/2a2a8685-4fc7-4d5b-8c4b-f19e909f0134" />

*Figure 02: Connected to the Ubuntu Wazuh server over SSH to manage the installation.*<img width="1916" height="1032" alt="30 creation of a dashboard" src="https://github.com/user-attachments/assets/22e89778-047a-4d7f-9c6b-cfe6c17a58c9" />

*Figure 30: Finished the dashboard with views for Windows logon failures, account changes and Linux SSH failures.*<img width="1913" height="696" alt="14  wazuh viw of agent connection with linux and windows machine" src="https://github.com/user-attachments/assets/158b2aff-0e0b-40ab-80ef-51c81bba65f7" />

*Figure 14: Verified that both my Windows and Ubuntu agents were visible in Wazuh.*See [SCREENSHOT-GUIDE.md](SCREENSHOT-GUIDE.md) for all 40 filenames.
