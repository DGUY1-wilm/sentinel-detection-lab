# Attack Simulation Steps

These commands create harmless, local activity on a **disposable lab VM** so each detection has something to find.

> Run them only on your own lab VM, never on any other machine or account.
> Open **PowerShell as Administrator** inside the VM (right-click, Run as administrator). Replace every `<choose-a-strong-password>` with a real, unique password.

Note the time you run each one so you can find the events later. Events can take 5 to 10 minutes to reach Sentinel.

## Setup

Create a test account to attack. Do this before building any rules, so it does not trigger D5.

```
net user labvictim '<choose-a-strong-password>' /add
```

## D1 Brute force (event 4625)

```
1..8 | ForEach-Object { net use \\127.0.0.1\IPC$ /user:labvictim "WrongPass$_!"; Start-Sleep -Seconds 2 }
```

Expected: several 4625 events for `labvictim`.
Note: in testing, an earlier version that reused the same wrong password eight times with no pause produced only 2 recorded failures. Check the count with:

```
SecurityEvent
| where EventID == 4625
| summarize Attempts = count() by TargetAccount
| sort by Attempts desc
```

## D2 Password spray (event 4625)

```
'alice','bob','carol','dave','erin','frank' | ForEach-Object { net use \\127.0.0.1\IPC$ /user:$_ 'Winter2026!' }
```

Expected: one 4625 per account name, all from the same source.

## D3 Success after failures (events 4625 then 4624)

Run the D1 loop, then log in with the real password within a couple of minutes:

```
net use \\127.0.0.1\IPC$ /delete
net use \\127.0.0.1\IPC$ /user:labvictim '<the-real-password>'
net use \\127.0.0.1\IPC$ /delete
```

Expected: a 4624 for `labvictim` shortly after the failures.
If you see system error 1219, an older connection is blocking the login. Run the `/delete` line first.

## D4 Encoded PowerShell (event 4688)

```
$cmd = [Convert]::ToBase64String([Text.Encoding]::Unicode.GetBytes('Write-Output lab-test'))
powershell.exe -NoProfile -EncodedCommand $cmd
```

Expected: a 4688 event whose command line contains `-EncodedCommand`. Requires command-line capture to be enabled.

## D5 New local user (event 4720)

```
net user labtempuser '<choose-a-strong-password>' /add
```

## D6 Added to Administrators (event 4732)

```
net localgroup Administrators labtempuser /add
```

Clean up afterward:

```
net localgroup Administrators labtempuser /delete
net user labtempuser /delete
```

## D7 Scheduled task (event 4698)

```
schtasks /create /tn LabTestTask /tr 'cmd.exe /c echo hi' /sc daily /st 03:00 /ru SYSTEM
schtasks /delete /tn LabTestTask /f
```

Requires the "Other Object Access Events" audit policy.

## D8 Security log cleared (event 1102). Run this last.

```
wevtutil cl Security
```

This wipes the local Security log. Sentinel keeps the copies already sent, and Windows records the clear itself as event 1102.

## Cleanup

```
net user labvictim /delete
```

Then delete the whole resource group in the Azure portal when you finish the lab.
