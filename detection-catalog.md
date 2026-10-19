# Detection Catalog

Status values: update after each query has been run against simulated activity.

| ID | Detection | ATT&CK | Event | Starting logic | Likely false positives | Status |
|---|---|---|---|---|---|---|
| D1 | Brute-force failed logons | T1110.001 | 4625 | 5 or more failures per account in 5 minutes | Mistyped passwords, expired service credentials | ☐ |
| D2 | Password spray | T1110.003 | 4625 | 5 or more distinct accounts from one source in 10 minutes, 3 or fewer attempts each | Scanners, misconfigured apps | ☐ |
| D3 | Success after repeated failures | T1110, T1078 | 4625, 4624 | 5 or more failures, then a network or RDP logon within 10 minutes | A user who forgot then remembered a password | ☐ |
| D4 | Encoded PowerShell | T1059.001 | 4688 | PowerShell process with an encoded-command argument and a long base64 string | Deployment and management tools | ☐ |
| D5 | New local user | T1136.001 | 4720 | Any account creation | IT onboarding, imaging tools | ☐ |
| D6 | Added to privileged group | T1098 | 4728, 4732, 4756 | Member added to Administrators or a domain admin group | Planned admin changes | ☐ |
| D7 | Scheduled task created | T1053.005 | 4698 | Any task creation | Windows and software updaters | ☐ |
| D8 | Security log cleared | T1070.001 | 1102 | Any clear event | Planned log maintenance (rare) | ☐ |

## Tuning log

Record each change and how many results there were before and after.

| Detection | Noise source | Tuning applied | Before | After |
|---|---|---|---|---|
|  |  |  |  |  |

## Notes

- Each query keeps a `lookback` value at the top for testing in the Logs page. A scheduled analytics rule uses its own lookup period instead.
- D6: for local accounts the member name is often `-`, so match `MemberSid` against `NewUserSid` from D5.
- D7: if the task name comes back empty, inspect one raw event with `SecurityEvent | where EventID == 4698 | take 1 | project EventData` and adjust the extraction.
- D8 covers the Security log only. Clearing the System log creates event 104 in a different table.
