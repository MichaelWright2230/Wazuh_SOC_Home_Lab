# Wazuh Lab Screenshot Guide

[Back to README](README.md) | [Step-by-step guide](step-by-step-lab-guide.md) | [Investigation report](investigation-report.md)

These are the captions used throughout the project. They explain what each screenshot shows and what I was trying to verify. Most screenshots are embedded directly beneath the relevant step in the lab guide, with investigation examples also shown in the investigation report. The architecture page contains the lab setup and connection images.

| Figure | Caption |
|---|---|
| 01 | Installed the Ubuntu virtual machine that I used to host the Wazuh server. |
| 02 | Connected to the Ubuntu Wazuh server over SSH to manage the installation. |
| 03 | Downloaded and started installing the Wazuh components on the server. |
| 04 | Opened the Wazuh web dashboard for the first time to confirm access. |
| 05 | Began configuring Wazuh archives so I could collect more than alert-only events. |
| 06 | Edited the Wazuh archive-related settings as part of the logging setup. |
| 07 | Continued the archive configuration and checked the relevant settings. |
| 08 | Updated the archive configuration needed to make raw events searchable. |
| 09 | Opened Wazuh's interface to continue setting up archived event access. |
| 10 | Checked the archive-related settings and the available event data. |
| 11 | Completed the archive setup and confirmed the logs could be investigated in Wazuh. |
| 12 | Set up the Windows 10 Wazuh agent to send endpoint events to the manager. |
| 13 | Checked the newly connected agent in the Wazuh interface. |
| 14 | Verified that both my Windows and Ubuntu agents were visible in Wazuh. |
| 15 | Reviewed the agent connection details before starting the detection tests. |
| 16 | Downloaded Microsoft Sysmon to improve visibility of Windows endpoint activity. |
| 17 | Updated the Windows Wazuh agent's ossec.conf to collect the Sysmon Operational event channel. |
| 18 | Searched Wazuh for Sysmon events to confirm that collection was working. |
| 19 | Expanded a Sysmon event to inspect the individual fields and event details. |
| 20 | Reviewed the endpoint log data in Wazuh after integrating Sysmon. |
| 21 | Created, edited and deleted test Windows accounts to generate security events. |
| 22 | Found the account-management events in Wazuh after producing the test activity. |
| 23 | Expanded a Windows event to investigate its account and event information. |
| 24 | Filtered the Windows logs for user-account creation events. |
| 25 | Built a search for an account being added to a security-enabled local group. |
| 26 | Filtered the Windows Security logs for user logon activity. |
| 27 | Generated invalid logon attempts using a test account in my lab. |
| 28 | Searched the archived logs for the fakeuser account and found related events. |
| 29 | Started building a dashboard to visualise the security events I had collected. |
| 30 | Finished the dashboard with views for Windows logon failures, account changes and Linux SSH failures. |
| 31 | Configured Windows file integrity monitoring for my C:\CompanyData test directory. |
| 32 | Modified payroll.txt and observed a Wazuh file integrity alert reporting that the checksum changed. |
| 33 | Configured real-time file integrity monitoring on my Ubuntu endpoint. |
| 34 | Generated Linux test file activity and reviewed the related Wazuh events. |
| 35 | Created custom Wazuh rule 100200 to alert when the Windows Guest account is enabled. |
| 36 | Triggered the Guest-account detection and confirmed the custom alert appeared in Discover. |
| 37 | Created rule 100101 to detect repeated SSH login failures from the same source IP within 120 seconds. |
| 38 | Linked the SSH detection rule to the firewall-drop active-response command. |
| 39 | Checked that firewall-drop was listed as an available Wazuh active-response command. |
| 40 | Generated repeated failed SSH attempts and viewed the correlation alert followed by Wazuh reporting a firewall-drop response. This shows the response was invoked, but not an independent packet-level block test. |

## Screenshot notes

The figures document a home lab, not a live incident. File-integrity alerts identify changes, not necessarily malicious activity. The SSH firewall-drop event confirms that Wazuh reported running the active response; an independent firewall inspection or connection attempt would be needed to verify the actual network block. Figure 03 is documented here, although its image is not currently embedded in the main lab guide.

## Adding any missing image

Where needed, upload the screenshot using GitHub's **Edit → drag image into the Markdown editor** workflow. GitHub will insert an image link. Put the matching *Figure NN* caption directly below the image.
