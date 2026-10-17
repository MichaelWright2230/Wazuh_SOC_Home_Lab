# Step-by-Step Wazuh SOC Home Lab Guide

[Back to README](README.md) | [Architecture](architecture.md) | [Investigation report](investigation-report.md)

This page describes how I completed and tested the Wazuh lab, following the **Build a SOC Lab with Wazuh** tutorial and adjusting it to match my own environment and the screenshots I took.

## Wazuh server setup and archived logging

First I configured the Wazuh server on Ubuntu and connected to it using SSH. I then opened the Wazuh web interface to confirm that the installation was working.

By default, Wazuh focuses on events that produce alerts. I also worked through the archive configuration so I could search additional raw activity without needing every single event to match an alert rule. This was usefull when investigating Windows account events and logon activity.

The Wazuh documentation describes the manager options `logall` and `logall_json`; archived JSON events can be indexed and searched in Discover. **The screenshots document my archive setup and use of the archive index; exact option values should be checked against the saved configuration if reproducing this lab.**

Example manager-side configuration (reference, not a verbatim copy of my complete file):

```xml
<global>
  <logall>yes</logall>
  <logall_json>yes</logall_json>
</global>
```

After changing manager configuration, restart the relevant service and verify events appear. Archiving more telemetry uses additional storage, so this needs retention controls in a real environment.

**Evidence:** 05–11 (archive configuration), 22–28 (searching archived Windows events).

<img width="1392" height="656" alt="5 config archives part1" src="https://github.com/user-attachments/assets/55a9c348-7294-4343-9013-bcf767250ebc" />

*Figure 05: Began configuring Wazuh archives so I could collect more than alert-only events.*<img width="873" height="953" alt="6 config archives part2" src="https://github.com/user-attachments/assets/49789ad3-0a7b-41a0-9922-01a1af00d6bc" />

*Figure 06: Edited the Wazuh archive-related settings as part of the logging setup.*<img width="680" height="288" alt="7 config archives part3" src="https://github.com/user-attachments/assets/2a7b38cb-dd99-4115-8fdc-639af6118dd1" />

*Figure 07: Continued the archive configuration and checked the relevant settings.*<img width="657" height="887" alt="8 config archives part4" src="https://github.com/user-attachments/assets/6466b8d5-0a38-4b33-83ad-f065f8d1d693" />

*Figure 08: Updated the archive configuration needed to make raw events searchable.*<img width="1919" height="1030" alt="9 config archives part5" src="https://github.com/user-attachments/assets/f2ccafb1-729f-480e-afb0-483ef176547f" />

*Figure 09: Opened Wazuh's interface to continue setting up archived event access.*<img width="1916" height="1029" alt="10 config archives part6" src="https://github.com/user-attachments/assets/748bb8e2-38ed-4fe9-9a48-76b8365d414b" />

*Figure 10: Checked the archive-related settings and the available event data.*<img width="1918" height="1038" alt="11 config archives part7" src="https://github.com/user-attachments/assets/da8e7d1f-9da9-44b6-b823-8d6f35a5c260" />

*Figure 11: Completed the archive setup and confirmed the logs could be investigated in Wazuh.*## Connecting the Windows and Linux agents

Next I installed and connected a Wazuh agent on my Windows 10 virtual machine and connected the Ubuntu endpoint. This let me monitor two different operating systems within the same dashboard.

I used the Wazuh agent views to check connectivity and identify the endpoints before moving onto security testing.

**Result:** My screenshots show both Windows and Linux agent connections in Wazuh.

<img width="1717" height="948" alt="12 connected to wazuh as a agent with windows 10" src="https://github.com/user-attachments/assets/67904fa2-846c-48c9-9d01-fb97b504889c" />

*Figure 12: Set up the Windows 10 Wazuh agent to send endpoint events to the manager.*<img width="1916" height="656" alt="13  wazuh viw of agent connection" src="https://github.com/user-attachments/assets/5db40f6e-324e-4039-bcab-1bf23f3b2663" />

*Figure 13: Checked the newly connected agent in the Wazuh interface.*<img width="1913" height="696" alt="14  wazuh viw of agent connection with linux and windows machine" src="https://github.com/user-attachments/assets/7b1c355e-881d-4746-982a-186f2bd28570" />

*Figure 14: Verified that both my Windows and Ubuntu agents were visible in Wazuh.*<img width="1915" height="1027" alt="15  agent connection in wazuh" src="https://github.com/user-attachments/assets/d277d608-b9b3-4097-874c-c9c3a4fa70ab" />

*Figure 15: Reviewed the agent connection details before starting the detection tests.*## Windows Sysmon integration

I downloaded Microsoft Sysmon for the Windows endpoint and updated the Windows Wazuh agent's `ossec.conf` configuration to include the Sysmon Operational event channel. This is useful because the normal Windows Security logs and Sysmon events provide different kinds of host information.

Example of the event-channel collection block:

```xml
<localfile>
  <location>Microsoft-Windows-Sysmon/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>
```

After the agent configuration was applied, I checked Wazuh to verify Sysmon-related logs were arriving.

**Result:** Sysmon telemetry appeared in Wazuh and I was able to inspect the expanded event information.

<img width="1699" height="934" alt="16  downloaded sysmon" src="https://github.com/user-attachments/assets/7a03966a-13cf-415d-8490-60b444917653" />

*Figure 16: Downloaded Microsoft Sysmon to improve visibility of Windows endpoint activity.*<img width="1719" height="898" alt="17 config the ossec conf to accept sysmon" src="https://github.com/user-attachments/assets/462de781-61a8-4905-bd2d-ee4c5001053d" />

*Figure 17: Updated the Windows Wazuh agent's ossec.conf to collect the Sysmon Operational event channel.*<img width="1917" height="1034" alt="18  sysmon working on wazuh" src="https://github.com/user-attachments/assets/b0f9cc01-7fe1-4d07-971f-3f0efa327ebf" />

*Figure 18: Searched Wazuh for Sysmon events to confirm that collection was working.*<img width="1918" height="1030" alt="19  sysmon working on wazuh expanded" src="https://github.com/user-attachments/assets/0e71e8c1-5e09-4005-9b34-31afed430efd" />

*Figure 19: Expanded a Sysmon event to inspect the individual fields and event details.*<img width="1911" height="1033" alt="20  sysmon working for winows and linux in wazuh" src="https://github.com/user-attachments/assets/547904ed-ed7c-4ca2-bc0c-e793a72f5de8" />

*Figure 20: Reviewed the endpoint log data in Wazuh after integrating Sysmon.*## Windows account activity investigation

I generated test Windows account activity involving creating, changing and deleting users, then looked for the results in Discover. I also built seperate searches for account creation, local security-group membership changes and logon events.

Some important Windows Security Event IDs used for this kind of analysis are:

| Event ID | Meaning | Why it matters |
|---|---|---|
| 4720 | User account created | Identifies new local or domain accounts |
| 4722 | User account enabled | May indicate an account was re-enabled |
| 4723 | Attempt to change account password | Helps investigate password changes |
| 4724 | Attempt to reset an account password | Useful for privileged account changes |
| 4726 | User account deleted | Tracks removal of accounts |
| 4732 | Member added to a security-enabled local group | Useful for monitoring changes in access |
| 4733 | Member removed from a security-enabled local group | Tracks access removal |
| 4738 | User account changed | Helps identify unexpected account modification |
| 4624 | Successful logon | Useful authentication context |
| 4625 | Failed logon | Possible credential guessing when repeated |

Event IDs provide leads, not proof of malicious activity by themselves. I checked the time, target username, host and other event fields to understand each result.

<img width="578" height="846" alt="21 createing events on windows (creating, editing and deleting users)" src="https://github.com/user-attachments/assets/780e4aa7-9186-420f-a95d-dcd08e0fc262" />

*Figure 21: Created, edited and deleted test Windows accounts to generate security events.*<img width="1919" height="1034" alt="22 viewing events from  windows (creating, editing and deleting users) on wazuh" src="https://github.com/user-attachments/assets/24d599ea-29a2-4a80-a565-9a2ef5469a61" />

*Figure 22: Found the account-management events in Wazuh after producing the test activity.*<img width="1512" height="882" alt="23  exploring infomation into the traffic i created" src="https://github.com/user-attachments/assets/07cfa537-8f74-433e-a6e3-b36595a84b06" />

*Figure 23: Expanded a Windows event to investigate its account and event information.*<img width="1918" height="1037" alt="24  creting a search for a new user creation" src="https://github.com/user-attachments/assets/ee1f1a4d-e969-4242-bec9-e906ccf096fa" />

*Figure 24: Filtered the Windows logs for user-account creation events.*<img width="1916" height="1031" alt="25  creting a search for a user added to a secuirty local group" src="https://github.com/user-attachments/assets/bf1f96be-ef74-4ad0-b8c2-fd068da67314" />

*Figure 25: Built a search for an account being added to a security-enabled local group.*<img width="1915" height="1029" alt="26  creting a search for user logons" src="https://github.com/user-attachments/assets/260ea13d-a751-4da3-81d2-3ecb6b0a158c" />

*Figure 26: Filtered the Windows Security logs for user logon activity.*### Failed Windows logon test

I also made deliberate invalid sign-in attempts against my test Windows environment and searched for the username `fakeuser`. In the screenshots I can see multiple authentication-related results after the testing.

**Result:** The archive search returned events matching `fakeuser`. The dashboard also contains a failed Windows logon metric.

<img width="1110" height="622" alt="27  creating fake logon attempts traffic" src="https://github.com/user-attachments/assets/b8883fa9-5fec-4c93-89ca-db1c1548c789" />

*Figure 27: Generated invalid logon attempts using a test account in my lab.*<img width="1916" height="1028" alt="28  searching for fakeuser logon attempts" src="https://github.com/user-attachments/assets/e53d4b46-c10f-47a0-8ff8-bb62d136cc65" />

*Figure 28: Searched the archived logs for the fakeuser account and found related events.*## SOC monitoring dashboard

To make the event data easier to review, I created a custom dashboard in Wazuh/OpenSearch Dashboards. It includes a Windows failed-logon metric, a time-series graph for account-management events and a table of Linux failed SSH authentications.

This brought multiple investigations into one place and helped me see authentication and account activity without repeating each search individually.

**Result shown in my screenshots:** The dashboard displays a failed-Windows-logon metric of **3** for the selected time range, a Windows account-change timeline and Linux SSH-related entries. That number is a captured snapshot, not a fixed lab total.

<img width="1915" height="1039" alt="29  creating a dash board" src="https://github.com/user-attachments/assets/b7d886e8-175a-4aea-9a93-b715e95262e7" />

*Figure 29: Started building a dashboard to visualise the security events I had collected.*<img width="1916" height="1032" alt="30 creation of a dashboard" src="https://github.com/user-attachments/assets/0cc87b04-44d4-479b-9353-558da9a8d54c" />

*Figure 30: Finished the dashboard with views for Windows logon failures, account changes and Linux SSH failures.*## File integrity monitoring (FIM)

I enabled real-time file monitoring in the agent configurations so Wazuh could identify changes to selected test directories.

### Windows FIM

I configured the Windows agent to monitor a test company-data directory:

```xml
<directories realtime="yes">C:\CompanyData</directories>
```

I then edited a file and looked for the change in Wazuh. The FIM event view shows:

- **Agent:** `MYDIF_WINDOWS` (name as displayed in the lab screenshot)
- **File:** `c:\companydata\payroll.txt`
- **Event:** `modified`
- **Rule ID:** `550`
- **Description:** `Integrity checksum changed.`

This demonstrated that the system could identify an alteration to a watched file, rather than just collect login events.

<img width="1710" height="937" alt="31 creating comsum rule for a file companydata file i made" src="https://github.com/user-attachments/assets/c05b66a6-8157-4d9c-a328-442b910bb374" />

*Figure 31: Configured Windows file integrity monitoring for my C:\CompanyData test directory.*<img width="1918" height="1035" alt="32  creating an event by trigering the new rule" src="https://github.com/user-attachments/assets/2a594083-b340-406d-ba6c-28d3fb40d8ef" />

*Figure 32: Modified payroll.txt and observed a Wazuh file integrity alert reporting that the checksum changed.*### Linux FIM

I also configured Linux directories for integrity monitoring, including a test path under `/opt/soc-lab`. The configuration screenshot includes real-time monitoring settings. I generated test file changes and used Wazuh to investigate the resulting events.

<img width="778" height="852" alt="33  creating an new rule on ubuntu for a file intergrity" src="https://github.com/user-attachments/assets/22d96609-a2d9-4de3-a656-c429fbc9fff5" />

*Figure 33: Configured real-time file integrity monitoring on my Ubuntu endpoint.*<img width="1917" height="1033" alt="34  triggeriing new lunix rule and view on wazuh" src="https://github.com/user-attachments/assets/0b76ad56-1f33-4e7c-a060-9d1eb8631d88" />

*Figure 34: Generated Linux test file activity and reviewed the related Wazuh events.*> FIM tells an analyst that a monitored file changed. It does **not** automatically establish that the change was malicious. Further investigation would be required.


## Custom Wazuh detection rules

Wazuh includes built-in rules, but I wanted to try making my own detections for events I cared about. I edited `local_rules.xml` through the Wazuh Rules interface.

### Rule 100200 — Windows Guest account enabled

I added a custom rule for Windows Security Event ID **4722** when the target account name was **Guest**. This is worth investigating because a newly enabled Guest account can increase exposure if it was not authorised.

The configuration shown in my screenshot uses:

- Custom rule ID: `100200`
- Severity: `12`
- Parent rule: `60103`
- Event ID: `4722`
- Target username: `Guest`
- MITRE mapping entered: `T1078` (Valid Accounts)

For readability, the core of the screenshot rule is:

```xml
<group name="windows,windows_security,account_changed,adduser,">
  <rule id="100200" level="12">
    <if_sid>60103</if_sid>
    <field name="win.system.eventID">^4722$</field>
    <field name="win.eventdata.targetUserName">^Guest$</field>
    <description>MYDFIR-Steven Windows Guest account was enabled.</description>
    <mitre><id>T1078</id></mitre>
    <group>windows_account_management,account_enabled,guest_account,</group>
  </rule>
</group>
```

The custom alert description follows the tutorial naming visible in my screenshot. It isn't the Windows hostname.

**Result:** The Discover screenshot includes the new custom Guest-account alert as well as a related account-enabled event. This shows the custom rule was firing in my lab.

<img width="834" height="806" alt="35  new created rule in wazuh about the guess account" src="https://github.com/user-attachments/assets/3b85a951-f3fc-4244-af9e-7419f190af95" />

*Figure 35: Created custom Wazuh rule 100200 to alert when the Windows Guest account is enabled.*<img width="1914" height="737" alt="36  triggering the alert create in wazuh" src="https://github.com/user-attachments/assets/4e560e02-376e-4710-b25b-dcbab38a2c2b" />

*Figure 36: Triggered the Guest-account detection and confirmed the custom alert appeared in Discover.*### Rule 100101 — Multiple SSH login failures

I then created a seperate SSH correlation rule for repeated failed login attempts against the Linux side of the lab.

The rule in the screenshot is configured with:
- ID `100101`
- Level `10`
- `frequency="3"`
- `timeframe="120"`
- `if_matched_sid>5760</if_matched_sid>`
- `same_source_ip` to correlate attempts from the same originating IP
- MITRE `T1110` (Brute Force)

This checks for repeated failure activity within a short time window. A few failures could be a user mistake, but a pattern from the same source deserves investigation.

```xml
<group name="local,syslog,sshd,authentication_failed,">
  <rule id="100101" level="10" frequency="3" timeframe="120">
    <if_matched_sid>5760</if_matched_sid>
    <same_source_ip />
    <description>Multiple SSH login failures observed from the same source IP</description>
    <mitre><id>T1110</id></mitre>
    <group>authentication_failed,ssh_bruteforce,credential_access,</group>
  </rule>
</group>
```

**Result:** My Discover screen includes the rule description **Multiple SSH login failures observed from the same source IP** and individual SSH authentication failures.

<img width="824" height="985" alt="37  created a rule which will detect multiple failed ssh failures" src="https://github.com/user-attachments/assets/fcc1823b-1b30-4135-b75c-fd155f885a8c" />

*Figure 37: Created rule 100101 to detect repeated SSH login failures from the same source IP within 120 seconds.*## Automated incident response — firewall-drop

I configured an active-response block tied to rule `100101`, using Wazuh's `firewall-drop` command. I restarted the Wazuh manager and checked the list of active response commands before testing it.

The active response section in the screenshot includes:

```xml
<active-response>
  <disabled>no</disabled>
  <command>firewall-drop</command>
  <location>local</location>
  <rules_id>100101</rules_id>
</active-response>
```

I generated multiple unsuccessful SSH login attempts from my lab system, and checked the Wazuh alerts. The final screenshot shows the sequence of authentication failures, the custom multiple-failure alert, and **Host Blocked by firewall-drop Active Response**.

### Test sequence
1. Attempted SSH authentication with invalid credentials/users from the lab.
2. Wazuh recorded SSH login failures.
3. Custom rule `100101` generated a multiple-failures alert.
4. An active-response alert reported `firewall-drop` execution.

**Observed result:** The Wazuh alerts show that the automated response was invoked. I did not capture a separate packet-level connectivity test or firewall ruleset output proving the block duration, so I am careful not to claim that as independently verified.

<img width="315" height="113" alt="38  created a rule which will act when the detection of multiple failed ssh failures" src="https://github.com/user-attachments/assets/eef73c74-84f2-40cd-9d06-f65e32272f24" />

*Figure 38: Linked the SSH detection rule to the firewall-drop active-response command.*<img width="774" height="311" alt="39  viewing the control" src="https://github.com/user-attachments/assets/dc2738ed-709b-4335-b2ab-cf1f46320f31" />

*Figure 39: Checked that firewall-drop was listed as an available Wazuh active-response command.*<img width="1910" height="1032" alt="40  new rule(firwall) blocked the user afetr multiple failed attempt to login on wazuh" src="https://github.com/user-attachments/assets/2fae784e-4e94-4dab-af00-210a826397df" />

*Figure 40: Generated repeated failed SSH attempts and viewed the correlation alert followed by Wazuh reporting a firewall-drop response. This shows the response was invoked, but not an independent packet-level block test.*> **Safety:** Active response can disconnect legitimate clients or administrators. Test against non-essential systems and define exclusions, expiry and recovery procedures before deploying similar rules outside a lab.


## What I learned

The most valuable part of this project was working through the entire monitoring process: **generating an event → collecting it → searching the logs → writing detection logic → validating an alert → observing a response**.

I got practical experience working with Windows Security Event IDs, Linux SSH authentication records, Wazuh rules, Sysmon telemetry, FIM and the Wazuh dashboard. It was also helpful to see how a high-level detection can be traced back to specific log events.

Some challenges were getting the agents and logs to show the right data, finding fields that matched the search queries, and setting up custom rules so they would trigger on the intended activity. Troubleshooting these issues helped me better understand where events come from and how Wazuh handles them.

