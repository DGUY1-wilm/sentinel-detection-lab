# Screenshot Checklist

Save images in `docs/images/` using these names, then uncomment the matching lines in the README.

Before publishing any screenshot, blur: subscription ID, tenant name, email addresses, and the VM's public IP.

| File name | What to capture |
|---|---|
| 00-architecture.png | Simple diagram: VM, agent, workspace, Sentinel |
| 01-resource-group.png | `rg-sentinel-lab` listing the lab resources |
| 02-vm-overview.png | VM overview page |
| 03-audit-policy.png | `auditpol` output showing the audited categories |
| 04-data-connector.png | Windows Security Events via AMA connected |
| 05-event-id-chart.png | Event ID counts from the verification query |
| 06-d1-bruteforce.png | D1 query results |
| 07-d2-spray.png | D2 query results |
| 08-d3-success-after-failures.png | D3 query results |
| 09-d4-encoded-powershell.png | D4 query results |
| 10-d5-d6-account-changes.png | D5 and D6 query results |
| 11-d7-scheduled-task.png | D7 query results |
| 12-d8-log-cleared.png | D8 query results |

Take these before deleting the lab. They cannot be recreated afterward.
