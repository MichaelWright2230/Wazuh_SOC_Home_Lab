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

**[SCREENSHOT PLACEHOLDER — 21–26: Account changes and event searches]**

## 2. Invalid Windows logons

**Purpose:** Practice spotting unsuccessful authentications.

**Procedure:** Deliberately attempted invalid credentials against my own lab, including the username `fakeuser`.

**Evidence:** Screenshot 27 shows the attempts; 28 shows five search hits matching `fakeuser`. The dashboard screenshot 30 shows a failed Windows logon count of **3** in its selected time range. These totals are from different filtered views and should not be conflated.

**Assessment:** The returned logs show invalid attempts but do not by themselves establish a real brute-force attack or a compromised account. I would pivot by username, source IP, host and time.

**[SCREENSHOT PLACEHOLDER — 27–28: Failed attempts and search]**

## 3. File integrity event

**Purpose:** Confirm that Wazuh could notice modifications to watched files.

**Procedure:** Added realtime monitoring for `C:\CompanyData`, then modified `payroll.txt`.

**Evidence:** Screenshot 31 captures the Windows configuration. Screenshot 32 shows the file `c:\companydata\payroll.txt`, event `modified`, Wazuh rule `550`, description `Integrity checksum changed.`

**Assessment:** This demonstrates detection of a changed file, not who made the change or whether it was unauthorised. A SOC analyst would inspect user/session context and the contents or hash difference.

**[SCREENSHOT PLACEHOLDER — 31–32: Configuration and detected modification]**

## 4. Linux integrity monitoring

**Purpose:** Configure a test folder in Linux and review file-integrity events.

**Procedure:** Edited Wazuh's `syscheck` settings to add real-time monitoring under `/opt/soc-lab`, generated test activity and viewed the related events.

**Evidence:** Screenshot 33 shows the configuration, screenshot 34 shows the event review.

**Assessment:** Confirming baseline file state and the exact monitored path is an important next step before this could be trusted as a production file-change alert.

**[SCREENSHOT PLACEHOLDER — 33–34: Linux file integrity]**

## 5. Windows Guest account custom alert

**Purpose:** Detect when the built-in Guest account becomes enabled.

**Rule:** `100200`, severity 12, Windows event 4722, target username `Guest`, MITRE technique `T1078` Valid Accounts.

**Evidence:** Screenshot 35 shows `local_rules.xml`. Screenshot 36 shows the alert description `MYDFIR-Steven Windows Guest account was enabled.` in Discover.

**Assessment:** Alert successfully generated in the lab. Before using it elsewhere, I would validate the parent rule and expected account names, and tune the response to account for approved administrative actions.

**[SCREENSHOT PLACEHOLDER — 35–36: Guest rule and its alert]**

## 6. SSH brute-force style authentication activity and response

**Purpose:** Detect repeated failed SSH authentications from the same source and have Wazuh invoke an active response.

**Rule:** `100101` uses three matching failures within 120 seconds with `same_source_ip` correlation and MITRE `T1110`.

**Test activity:** Repeated invalid SSH authentication attempts against a lab system.

**Evidence:** Screenshot 37 shows the custom correlation rule. Screenshot 38 shows the `firewall-drop` active-response configuration and screenshot 39 shows the available response. Screenshot 40 shows individual SSH failures, `Multiple SSH login failures observed from the same source IP`, then `Host Blocked by firewall-drop Active Response`.

**Assessment:** Both the correlation alert and active-response event are visible. The Wazuh view is good evidence of the detection/response pipeline. A command-level firewall inspection and repeat connectivity test would be needed to independently verify the actual packet block and duration.

**[SCREENSHOT PLACEHOLDER — 37–40: Rule, active response and Wazuh result]**

## MITRE ATT&CK and SOC skills

| Technique | Relevance |
|---|---|
| [T1110 — Brute Force](https://attack.mitre.org/techniques/T1110/) | Repeated SSH authentication failures |
| [T1078 — Valid Accounts](https://attack.mitre.org/techniques/T1078/) | Investigating enabling of Guest accounts |

In a real investigation I would check for any successful logons after the failures, identify whether the source IP belongs to approved testing, and scope activity across other endpoints. This was only simulated activity in my own lab.

## Conclusion

This project let me practise the workflow between endpoint data, a central SIEM, rule tuning, incident investigation and active response. The most important result was being able to capture both the failed SSH logins and the resulting custom alert and response record. I'd still want additional validation of response effectiveness before calling this a production-ready control.
