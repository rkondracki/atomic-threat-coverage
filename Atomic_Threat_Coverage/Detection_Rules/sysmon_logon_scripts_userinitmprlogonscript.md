| Title                    | Logon Scripts (UserInitMprLogonScript)       |
|:-------------------------|:------------------|
| **Description**          | Detects creation or execution of UserInitMprLogonScript persistence method |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0003: Persistence](https://attack.mitre.org/tactics/TA0003)</li><li>[TA0008: Lateral Movement](https://attack.mitre.org/tactics/TA0008)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1037: Logon Scripts](https://attack.mitre.org/techniques/T1037)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0002_4688_windows_process_creation_with_commandline](../Data_Needed/DN_0002_4688_windows_process_creation_with_commandline.md)</li><li>[DN_0003_1_windows_sysmon_process_creation](../Data_Needed/DN_0003_1_windows_sysmon_process_creation.md)</li><li>[DN_0015_11_windows_sysmon_FileCreate](../Data_Needed/DN_0015_11_windows_sysmon_FileCreate.md)</li><li>[DN_0016_12_windows_sysmon_RegistryEvent](../Data_Needed/DN_0016_12_windows_sysmon_RegistryEvent.md)</li><li>[DN_0017_13_windows_sysmon_RegistryEvent](../Data_Needed/DN_0017_13_windows_sysmon_RegistryEvent.md)</li><li>[DN_0018_14_windows_sysmon_RegistryEvent](../Data_Needed/DN_0018_14_windows_sysmon_RegistryEvent.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1037: Logon Scripts](../Triggers/T1037.md)</li></ul>  |
| **Severity Level**       | high |
| **False Positives**      | <ul><li>exclude legitimate logon scripts</li><li>penetration tests, red teaming</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://attack.mitre.org/techniques/T1037/](https://attack.mitre.org/techniques/T1037/)</li></ul>  |
| **Author**               | Tom Ueltschi (@c_APT_ure) |


## Detection Rules

### Sigma rule

```
action: global
title: Logon Scripts (UserInitMprLogonScript)
id: 0a98a10c-685d-4ab0-bddc-b6bdd1d48458
status: experimental
description: Detects creation or execution of UserInitMprLogonScript persistence method
references:
    - https://attack.mitre.org/techniques/T1037/
tags:
    - attack.t1037
    - attack.persistence
    - attack.lateral_movement
author: Tom Ueltschi (@c_APT_ure)
date: 2019/01/12
falsepositives:
    - exclude legitimate logon scripts
    - penetration tests, red teaming
level: high
---
logsource:
    category: process_creation
    product: windows
detection:
    exec_selection:
        ParentImage: '*\userinit.exe'
    exec_exclusion1:
        Image: '*\explorer.exe'
    exec_exclusion2:
        CommandLine:
            - '*\netlogon.bat'
            - '*\UsrLogon.cmd'
    condition: exec_selection and not exec_exclusion1 and not exec_exclusion2
---
logsource:
    category: process_creation
    product: windows
detection:
    create_keywords_cli:
        CommandLine: '*UserInitMprLogonScript*'
    condition: create_keywords_cli
---
logsource:
    product: windows
    service: sysmon
detection:
    create_selection_reg:
        EventID:
            - 11
            - 12
            - 13
            - 14
    create_keywords_reg:
        TargetObject: '*UserInitMprLogonScript*'
    condition: create_selection_reg and create_keywords_reg

```





### powershell
    
```
Get-WinEvent | where {(($_.message -match "ParentImage.*.*\\\\userinit.exe" -and  -not ($_.message -match "Image.*.*\\\\explorer.exe")) -and  -not (($_.message -match "CommandLine.*.*\\\\netlogon.bat" -or $_.message -match "CommandLine.*.*\\\\UsrLogon.cmd"))) } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message\nGet-WinEvent | where {$_.message -match "CommandLine.*.*UserInitMprLogonScript.*" } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message\nGet-WinEvent -LogName Microsoft-Windows-Sysmon/Operational | where {(($_.ID -eq "11" -or $_.ID -eq "12" -or $_.ID -eq "13" -or $_.ID -eq "14") -and $_.message -match "TargetObject.*.*UserInitMprLogonScript.*") } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
((winlog.event_data.ParentImage.keyword:*\\\\userinit.exe AND (NOT (winlog.event_data.Image.keyword:*\\\\explorer.exe))) AND (NOT (winlog.event_data.CommandLine.keyword:(*\\\\netlogon.bat OR *\\\\UsrLogon.cmd))))\nwinlog.event_data.CommandLine.keyword:*UserInitMprLogonScript*\n(winlog.channel:"Microsoft\\-Windows\\-Sysmon\\/Operational" AND winlog.event_id:("11" OR "12" OR "13" OR "14") AND winlog.event_data.TargetObject.keyword:*UserInitMprLogonScript*)
```


### xpack-watcher
    
```
curl -s -XPUT -H \'Content-Type: application/json\' --data-binary @- localhost:9200/_watcher/watch/0a98a10c-685d-4ab0-bddc-b6bdd1d48458 <<EOF\n{\n  "metadata": {\n    "title": "Logon Scripts (UserInitMprLogonScript)",\n    "description": "Detects creation or execution of UserInitMprLogonScript persistence method",\n    "tags": [\n      "attack.t1037",\n      "attack.persistence",\n      "attack.lateral_movement"\n    ],\n    "query": "((winlog.event_data.ParentImage.keyword:*\\\\\\\\userinit.exe AND (NOT (winlog.event_data.Image.keyword:*\\\\\\\\explorer.exe))) AND (NOT (winlog.event_data.CommandLine.keyword:(*\\\\\\\\netlogon.bat OR *\\\\\\\\UsrLogon.cmd))))"\n  },\n  "trigger": {\n    "schedule": {\n      "interval": "30m"\n    }\n  },\n  "input": {\n    "search": {\n      "request": {\n        "body": {\n          "size": 0,\n          "query": {\n            "bool": {\n              "must": [\n                {\n                  "query_string": {\n                    "query": "((winlog.event_data.ParentImage.keyword:*\\\\\\\\userinit.exe AND (NOT (winlog.event_data.Image.keyword:*\\\\\\\\explorer.exe))) AND (NOT (winlog.event_data.CommandLine.keyword:(*\\\\\\\\netlogon.bat OR *\\\\\\\\UsrLogon.cmd))))",\n                    "analyze_wildcard": true\n                  }\n                }\n              ],\n              "filter": {\n                "range": {\n                  "timestamp": {\n                    "gte": "now-30m/m"\n                  }\n                }\n              }\n            }\n          }\n        },\n        "indices": [\n          "winlogbeat-*"\n        ]\n      }\n    }\n  },\n  "condition": {\n    "compare": {\n      "ctx.payload.hits.total": {\n        "not_eq": 0\n      }\n    }\n  },\n  "actions": {\n    "send_email": {\n      "email": {\n        "to": "root@localhost",\n        "subject": "Sigma Rule \'Logon Scripts (UserInitMprLogonScript)\'",\n        "body": "Hits:\\n{{#ctx.payload.hits.hits}}{{_source}}\\n================================================================================\\n{{/ctx.payload.hits.hits}}",\n        "attachments": {\n          "data.json": {\n            "data": {\n              "format": "json"\n            }\n          }\n        }\n      }\n    }\n  }\n}\nEOF\ncurl -s -XPUT -H \'Content-Type: application/json\' --data-binary @- localhost:9200/_watcher/watch/0a98a10c-685d-4ab0-bddc-b6bdd1d48458-2 <<EOF\n{\n  "metadata": {\n    "title": "Logon Scripts (UserInitMprLogonScript)",\n    "description": "Detects creation or execution of UserInitMprLogonScript persistence method",\n    "tags": [\n      "attack.t1037",\n      "attack.persistence",\n      "attack.lateral_movement"\n    ],\n    "query": "winlog.event_data.CommandLine.keyword:*UserInitMprLogonScript*"\n  },\n  "trigger": {\n    "schedule": {\n      "interval": "30m"\n    }\n  },\n  "input": {\n    "search": {\n      "request": {\n        "body": {\n          "size": 0,\n          "query": {\n            "bool": {\n              "must": [\n                {\n                  "query_string": {\n                    "query": "winlog.event_data.CommandLine.keyword:*UserInitMprLogonScript*",\n                    "analyze_wildcard": true\n                  }\n                }\n              ],\n              "filter": {\n                "range": {\n                  "timestamp": {\n                    "gte": "now-30m/m"\n                  }\n                }\n              }\n            }\n          }\n        },\n        "indices": [\n          "winlogbeat-*"\n        ]\n      }\n    }\n  },\n  "condition": {\n    "compare": {\n      "ctx.payload.hits.total": {\n        "not_eq": 0\n      }\n    }\n  },\n  "actions": {\n    "send_email": {\n      "email": {\n        "to": "root@localhost",\n        "subject": "Sigma Rule \'Logon Scripts (UserInitMprLogonScript)\'",\n        "body": "Hits:\\n{{#ctx.payload.hits.hits}}{{_source}}\\n================================================================================\\n{{/ctx.payload.hits.hits}}",\n        "attachments": {\n          "data.json": {\n            "data": {\n              "format": "json"\n            }\n          }\n        }\n      }\n    }\n  }\n}\nEOF\ncurl -s -XPUT -H \'Content-Type: application/json\' --data-binary @- localhost:9200/_watcher/watch/0a98a10c-685d-4ab0-bddc-b6bdd1d48458-3 <<EOF\n{\n  "metadata": {\n    "title": "Logon Scripts (UserInitMprLogonScript)",\n    "description": "Detects creation or execution of UserInitMprLogonScript persistence method",\n    "tags": [\n      "attack.t1037",\n      "attack.persistence",\n      "attack.lateral_movement"\n    ],\n    "query": "(winlog.channel:\\"Microsoft\\\\-Windows\\\\-Sysmon\\\\/Operational\\" AND winlog.event_id:(\\"11\\" OR \\"12\\" OR \\"13\\" OR \\"14\\") AND winlog.event_data.TargetObject.keyword:*UserInitMprLogonScript*)"\n  },\n  "trigger": {\n    "schedule": {\n      "interval": "30m"\n    }\n  },\n  "input": {\n    "search": {\n      "request": {\n        "body": {\n          "size": 0,\n          "query": {\n            "bool": {\n              "must": [\n                {\n                  "query_string": {\n                    "query": "(winlog.channel:\\"Microsoft\\\\-Windows\\\\-Sysmon\\\\/Operational\\" AND winlog.event_id:(\\"11\\" OR \\"12\\" OR \\"13\\" OR \\"14\\") AND winlog.event_data.TargetObject.keyword:*UserInitMprLogonScript*)",\n                    "analyze_wildcard": true\n                  }\n                }\n              ],\n              "filter": {\n                "range": {\n                  "timestamp": {\n                    "gte": "now-30m/m"\n                  }\n                }\n              }\n            }\n          }\n        },\n        "indices": [\n          "winlogbeat-*"\n        ]\n      }\n    }\n  },\n  "condition": {\n    "compare": {\n      "ctx.payload.hits.total": {\n        "not_eq": 0\n      }\n    }\n  },\n  "actions": {\n    "send_email": {\n      "email": {\n        "to": "root@localhost",\n        "subject": "Sigma Rule \'Logon Scripts (UserInitMprLogonScript)\'",\n        "body": "Hits:\\n{{#ctx.payload.hits.hits}}{{_source}}\\n================================================================================\\n{{/ctx.payload.hits.hits}}",\n        "attachments": {\n          "data.json": {\n            "data": {\n              "format": "json"\n            }\n          }\n        }\n      }\n    }\n  }\n}\nEOF\n
```


### graylog
    
```
((ParentImage.keyword:*\\\\userinit.exe AND (NOT (Image.keyword:*\\\\explorer.exe))) AND (NOT (CommandLine.keyword:(*\\\\netlogon.bat *\\\\UsrLogon.cmd))))\nCommandLine.keyword:*UserInitMprLogonScript*\n(EventID:("11" "12" "13" "14") AND TargetObject.keyword:*UserInitMprLogonScript*)
```


### splunk
    
```
((ParentImage="*\\\\userinit.exe" NOT (Image="*\\\\explorer.exe")) NOT ((CommandLine="*\\\\netlogon.bat" OR CommandLine="*\\\\UsrLogon.cmd")))\nCommandLine="*UserInitMprLogonScript*"\n(source="WinEventLog:Microsoft-Windows-Sysmon/Operational" (EventCode="11" OR EventCode="12" OR EventCode="13" OR EventCode="14") TargetObject="*UserInitMprLogonScript*")
```


### logpoint
    
```
((ParentImage="*\\\\userinit.exe"  -(Image="*\\\\explorer.exe"))  -(CommandLine IN ["*\\\\netlogon.bat", "*\\\\UsrLogon.cmd"]))\nCommandLine="*UserInitMprLogonScript*"\n(event_id IN ["11", "12", "13", "14"] TargetObject="*UserInitMprLogonScript*")
```


### grep
    
```
grep -P '^(?:.*(?=.*(?:.*(?=.*.*\\userinit\\.exe)(?=.*(?!.*(?:.*(?=.*.*\\explorer\\.exe))))))(?=.*(?!.*(?:.*(?=.*(?:.*.*\\netlogon\\.bat|.*.*\\UsrLogon\\.cmd))))))'\ngrep -P '^.*UserInitMprLogonScript.*'\ngrep -P '^(?:.*(?=.*(?:.*11|.*12|.*13|.*14))(?=.*.*UserInitMprLogonScript.*))'
```



