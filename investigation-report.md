# Wazuh SOC Investigation Report

[Back to README](README.md) | [Step-by-step guide](step-by-step-lab-guide.md)

## Investigation summary

I completed several small security investigations in a controlled environment to practise identifying suspicious behaviour from endpoint and authentication telemetry. This report records what triggered each test, what I saw in Wazuh, my interpretation and what the screenshots cannot independently prove.

| Investigation | Endpoint | Wazuh evidence | Result |
|---|---|---|---|
| Windows account changes | Windows 10 | Account events and filtered searches | Account-management activity was searchable |
| Failed account login | Windows 10 | `fakeuser` archive results and dashboard | Invalid logon activity appeared in Wazuh |
| Monitored-file modification | Windows 10 | File integrity rule 550 for `payroll.txt` | A changed file was detected |
| Linux FIM test | Ubuntu | Syscheck configuration and FIM event view | Watched-folder activity was investigated |
| Guest account enabled | Windows 10 | Rule `100200` and matching alert | Custom rule triggered |
| Repeated SSH failures | Linux/Wazuh | Rule `100101`, SSH alerts and active-response alert | Detection and response event recorded |

## 1. Windows account-management activity

**Purpose:** Recognise account creation and security group changes that might be important in a SOC investigation.

**Procedure:** Created and modified test accounts, then searched for relevant Windows security events in Discover.

**Evidence:** Screenshot 21 (test actions), 22–23 (logs), 24–26 (searches). Important Windows event IDs include 4720 (account creation), 4732 (member added to local security group), and 4624/4625 (logon results).

**Assessment:** Account-management telemetry was present, but the occurrence of an Event ID alone isn't proof of suspicious intent. I would check the acting account, target account, source system and whether an approved change existed.

<img width="578" height="846" alt="21 createing events on windows (creating, editing and deleting users)" src="https://github.com/user-attachments/assets/26e21d95-eb3d-441f-a7d9-28f1a069070f" />

<img width="1919" height="1034" alt="22 viewing events from  windows (creating, editing and deleting users) on wazuh" src="https://github.com/user-attachments/assets/3a82f383-34f8-498e-bded-df45efb13532" />

<img width="1512" height="882" alt="23  exploring infomation into the traffic i created" src="https://github.com/user-attachments/assets/95a2bd4a-048f-420c-9c40-4c6135c771ed" />

<img width="1918" height="1037" alt="24  creting a search for a new user creation" src="https://github.com/user-attachments/assets/d018540c-98ec-4205-893d-268ad235f261" />

<img width="1916" height="1031" alt="25  creting a search for a user added to a secuirty local group" src="https://github.com/user-attachments/assets/82a33f25-f338-4435-ac8a-5981e1a64b13" />

<img width="1915" height="1029" alt="26  creting a search for user logons" src="https://github.com/user-attachments/assets/a87b777e-225f-480c-9583-b6520c3a2bc7" />

## 2. Invalid Windows logons

**Purpose:** Practice spotting unsuccessful authentications.

**Procedure:** Deliberately attempted invalid credentials against my own lab, including the username `fakeuser`.

**Evidence:** Screenshot 27 shows the attempts; 28 shows five search hits matching `fakeuser`. The dashboard screenshot 30 shows a failed Windows logon count of **3** in its selected time range. These totals are from different filtered views and should not be conflated.

**Assessment:** The returned logs show invalid attempts but do not by themselves establish a real brute-force attack or a compromised account. I would pivot by username, source IP, host and time.

<img width="1110" height="622" alt="27  creating fake logon attempts traffic" src="https://github.com/user-attachments/assets/aeaa8b6e-a3a8-4c14-9724-0ecbe6e5898d" />

<img width="1916" height="1028" alt="28  searching for fakeuser logon attempts" src="https://github.com/user-attachments/assets/cf7a5000-9a48-4a08-8716-8dd9fac6c5bf" />

## 3. File integrity event

**Purpose:** Confirm that Wazuh could notice modifications to watched files.

**Procedure:** Added realtime monitoring for `C:\CompanyData`, then modified `payroll.txt`.

**Evidence:** Screenshot 31 captures the Windows configuration. Screenshot 32 shows the file `c:\companydata\payroll.txt`, event `modified`, Wazuh rule `550`, description `Integrity checksum changed.`

**Assessment:** This demonstrates detection of a changed file, not who made the change or whether it was unauthorised. A SOC analyst would inspect user/session context and the contents or hash difference.

<img width="1710" height="937" alt="31 creating comsum rule for a file companydata file i made" src="https://github.com/user-attachments/assets/623b537e-844a-4f44-a63c-27d0976c3df1" />

<img width="1918" height="1035" alt="32  creating an event by trigering the new rule" src="https://github.com/user-attachments/assets/7eb89420-54ea-4671-ab53-0b3ea07cb37f" />

## 4. Linux integrity monitoring

**Purpose:** Configure a test folder in Linux and review file-integrity events.

**Procedure:** Edited Wazuh's `syscheck` settings to add real-time monitoring under `/opt/soc-lab`, generated test activity and viewed the related events.

**Evidence:** Screenshot 33 shows the configuration, screenshot 34 shows the event review.

**Assessment:** Confirming baseline file state and the exact monitored path is an important next step before this could be trusted as a production file-change alert.

<img width="778" height="852" alt="33  creating an new rule on ubuntu for a file intergrity" src="https://github.com/user-attachments/assets/e091e23d-922e-4958-b8c1-0679773997c7" />

<img width="1917" height="1033" alt="34  triggeriing new lunix rule and view on wazuh" src="https://github.com/user-attachments/assets/4f5f9275-c934-4266-baeb-df2fa4bc137d" />

## 5. Windows Guest account custom alert

**Purpose:** Detect when the built-in Guest account becomes enabled.

**Rule:** `100200`, severity 12, Windows event 4722, target username `Guest`, MITRE technique `T1078` Valid Accounts.

**Evidence:** Screenshot 35 shows `local_rules.xml`. Screenshot 36 shows the alert description `MYDFIR-Steven Windows Guest account was enabled.` in Discover.

**Assessment:** Alert successfully generated in the lab. Before using it elsewhere, I would validate the parent rule and expected account names, and tune the response to account for approved administrative actions.

<img width="834" height="806" alt="35  new created rule in wazuh about the guess account" src="https://github.com/user-attachments/assets/24df007a-c1eb-4375-9848-b1d204ea66a9" />

<img width="1914" height="737" alt="36  triggering the alert create in wazuh" src="https://github.com/user-attachments/assets/db1879b3-32e3-448b-b920-1ded60db544b" />

## 6. SSH brute-force style authentication activity and response

**Purpose:** Detect repeated failed SSH authentications from the same source and have Wazuh invoke an active response.

**Rule:** `100101` uses three matching failures within 120 seconds with `same_source_ip` correlation and MITRE `T1110`.

**Test activity:** Repeated invalid SSH authentication attempts against a lab system.

**Evidence:** Screenshot 37 shows the custom correlation rule. Screenshot 38 shows the `firewall-drop` active-response configuration and screenshot 39 shows the available response. Screenshot 40 shows individual SSH failures, `Multiple SSH login failures observed from the same source IP`, then `Host Blocked by firewall-drop Active Response`.

**Assessment:** Both the correlation alert and active-response event are visible. The Wazuh view is good evidence of the detection/response pipeline. A command-level firewall inspection and repeat connectivity test would be needed to independently verify the actual packet block and duration.

<img width="824" height="985" alt="37  created a rule which will detect multiple failed ssh failures" src="https://github.com/user-attachments/assets/10973a10-7452-4e53-bc31-e5e998f944b3" />

<img width="315" height="113" alt="38  created a rule which will act when the detection of multiple failed ssh failures" src="https://github.com/user-attachments/assets/832f0161-612b-49e5-819f-8ba406f938d9" />

<img width="774" height="311" alt="39  viewing the control" src="https://github.com/user-attachments/assets/2546175f-980d-4e74-bd3c-4505003aa6bb" />

<img width="1910" height="1032" alt="40  new rule(firwall) blocked the user afetr multiple failed attempt to login on wazuh" src="https://github.com/user-attachments/assets/7d3cf67d-b944-4f20-bf35-0a7c86de9839" />

## MITRE ATT&CK and SOC skills

| Technique | Relevance |
|---|---|
| [T1110 — Brute Force](https://attack.mitre.org/techniques/T1110/) | Repeated SSH authentication failures |
| [T1078 — Valid Accounts](https://attack.mitre.org/techniques/T1078/) | Investigating enabling of Guest accounts |

In a real investigation I would check for any successful logons after the failures, identify whether the source IP belongs to approved testing, and scope activity across other endpoints. This was only simulated activity in my own lab.

## Conclusion

This project let me practise the workflow between endpoint data, a central SIEM, rule tuning, incident investigation and active response. The most important result was being able to capture both the failed SSH logins and the resulting custom alert and response record. I'd still want additional validation of response effectiveness before calling this a production-ready control.
