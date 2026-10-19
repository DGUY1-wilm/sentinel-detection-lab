# Build Your Own Security Alarm Lab: A Simple Step-by-Step Tutorial

No security background needed. If you can follow a recipe, you can do this.

## What you will build

Think of a smoke detector, but for a computer. You will set up a practice Windows computer in the cloud, send its activity diary to a security tool called Microsoft Sentinel, and write a few "alarm rules". Then you will do harmless pretend break-ins to see whether the alarms notice.

## Before you start

- **Time:** about 8 hours, spread over a few days.
- **Cost:** a free trial or student credit usually covers it, but a practice computer costs a little each hour it runs. You will set a spending alert and delete everything at the end.
- **You need:** a Microsoft account, a way to verify your identity (a card, or a student email), and a computer with internet. Windows has Remote Desktop built in. On a Mac, install the free **Windows App** from the App Store.
- **Safety rules:**
  - Only do the pretend break-ins on your own practice computer.
  - Use a long password you do not use anywhere else.
  - Delete everything when you finish.

## Words you will see

| Word | Plain meaning |
|---|---|
| Azure | Microsoft's cloud, where you rent computers |
| Resource group | A folder that holds everything for this lab, so you can delete it in one click |
| VM (virtual machine) | The practice computer in the cloud |
| Log | A diary of what happened on a computer |
| Sentinel | The security tool that reads those diaries |
| KQL | The simple search language used to ask Sentinel questions |
| Detection | A saved search that raises a flag when something suspicious appears |

---

## Step 1: Create your Azure account

1. Go to portal.azure.com and sign up for the free account (or sign in with your student account).
2. Complete the identity check.

**You will know it worked when** you can see the Azure portal home page.

## Step 2: Set a spending alert

1. In the search bar at the top, type **Cost Management** and open it.
2. Click **Budgets**, then **Add**.
3. Make a budget of about 10 dollars a month. Add email alerts at 50%, 80% and 100%.

**You will know it worked when** the budget appears in the list. The alert only warns you. It does not stop spending.

## Step 3: Make a folder for the lab

1. Search for **Resource groups** and click **Create**.
2. Name it `rg-sentinel-lab`.
3. Pick a region close to you and click **Review + create**, then **Create**.

**You will know it worked when** `rg-sentinel-lab` shows up in the list.

## Step 4: Make the log workspace

1. Search for **Log Analytics workspaces** and click **Create**.
2. Choose your resource group, name it `law-sentinel-lab`, and pick a region.
3. Click **Review + create**, then **Create**.

If you see an error that says `RequestDisallowedByAzure`, your account does not allow that region. Pick a different one and try again. Student accounts often have this limit.

**You will know it worked when** the workspace appears inside your resource group.

## Step 5: Make the practice computer

1. Search for **Virtual machines** and click **Create**, then **Azure virtual machine**.
2. Choose your resource group and name it `vm-win-lab`.
3. For the image, pick **Windows Server Datacenter** (the plain one, not "Core", not "smalldisk").
4. For the size, pick a small one such as **Standard B2s**.
5. Choose a username that is not "admin" or "administrator". Make a long, unique password and save it in a password manager.
6. Click **Review + create**, then **Create**. Wait a few minutes.

**You will know it worked when** the VM page says **Running**.

## Step 6: Lock the front door

By default, the practice computer accepts connection attempts from anywhere. Limit it to just you.

1. On your own computer, search the web for "what is my IP" and copy the **IPv4** address (it does not start with 192.168).
2. On the VM page, click **Networking** in the left menu.
3. Click the **RDP** rule (port 3389).
4. Change the **Source** to **IP addresses** and paste your address followed by `/32`, for example `203.0.113.7/32`.
5. Click **Save**.
6. In the left menu, open **Operations**, then **Auto-shutdown**, and turn it on so the VM turns off every night.

**You will know it worked when** the saved rule shows your address.

## Step 7: Connect to the practice computer

1. On the VM page, copy the **Public IP address**.
2. On Windows, press the Windows key, type `mstsc`, and open **Remote Desktop Connection**. On Mac, use **Windows App**.
3. Paste the IP address and click **Connect**.
4. When asked for a sign-in, click **More choices**, then **Use a different account**. Enter `.\` followed by your VM username, for example `.\labadmin`, and your VM password.
5. Accept the certificate warning.

**You will know it worked when** you see a Windows Server desktop.

If it times out, your home IP may have changed. Repeat Step 6 with the new address. Some school and public networks also block this kind of connection, so try home internet or a phone hotspot.

## Step 8: Turn on the diary settings

By default, Windows does not write down everything we need. Inside the practice computer, right-click the PowerShell icon in the Start menu and choose **Run as administrator**. Then copy and paste these lines, one at a time:

```
auditpol /set /subcategory:"Logon" /success:enable /failure:enable
auditpol /set /subcategory:"User Account Management" /success:enable /failure:enable
auditpol /set /subcategory:"Security Group Management" /success:enable /failure:enable
auditpol /set /subcategory:"Process Creation" /success:enable
auditpol /set /subcategory:"Other Object Access Events" /success:enable
```

Next, record the full command for every program that starts:

1. Press the Windows key, type `gpedit.msc`, and open it.
2. Go to **Computer Configuration**, **Administrative Templates**, **System**, **Audit Process Creation**.
3. Open **Include command line in process creation events**, choose **Enabled**, and click **OK**.
4. In PowerShell, run `gpupdate /force`.

**You will know it worked when** each `auditpol` line says "The command was successfully executed."

## Step 9: Turn on Sentinel and connect the practice computer

Do this part on your own computer, in the Azure portal, not inside the VM.

1. Search for **Microsoft Sentinel**, click **Create**, pick `law-sentinel-lab`, and click **Add**. A free trial may start now, so write down the date.
2. Open the workspace, then **Content management**, then **Content hub**. Search `Windows Security Events` and click **Install**.
3. Go to **Configuration**, then **Data connectors**. Open **Windows Security Events via AMA** and click **Open connector page**.
4. Click **Create data collection rule**. Give it a name, add `vm-win-lab` as the resource, and on the Collect tab choose **All Security Events**. Click **Create**.
5. Wait 10 to 20 minutes with the VM running.
6. Go to **Logs**. If it says "Simple mode", switch it to **KQL mode**. Paste this and click **Run**:

```
SecurityEvent
| summarize count() by EventID
| sort by count_ desc
```

**You will know it worked when** a table or chart of numbers appears. That means the practice computer's diary is arriving in Sentinel.

## Step 10: Do the pretend break-ins and check the alarms

1. Open [attack-simulation.md](attack-simulation.md). Run each pretend break-in inside the practice computer's PowerShell (as administrator), one at a time. Run the last one (clearing the log) at the very end.
2. Wait about 5 to 10 minutes after each one.
3. On your own computer, open the matching file in the `detections` folder (for example `D1_bruteforce_failed_logons.kql`). Copy **all** of the text, including the lines at the top that start with `let`.
4. Paste it into Sentinel's **Logs** box, with the time range set to **Last 24 hours**, and click **Run**.
5. Take a screenshot of each result.

**You will know it worked when** a detection shows rows that match what you just did. If a result is empty, see the tips below.

## Step 11: Clean up so nothing keeps billing

1. Save your screenshots and your detection files somewhere safe first. They cannot be recovered afterward.
2. In the portal, go to **Resource groups** and open `rg-sentinel-lab`.
3. Click **Delete resource group**, type the name to confirm, and click **Delete**.
4. Check **All resources** a bit later and confirm nothing from the lab remains.

---

## If something goes wrong

| Problem | What to try |
|---|---|
| "Region not allowed" error | Choose a different region and try again |
| Remote Desktop times out | Your home IP changed, or your network blocks it. Redo Step 6 with your current IP, or try a hotspot |
| Sign-in is rejected | Use `.\` plus the VM username. Do not use your Microsoft account |
| Sentinel shows no data | Wait 20 to 30 minutes, check the VM is running, and check the VM is listed in the data collection rule |
| A detection returns nothing | Check you pasted every line, including the `let` lines, and wait a few more minutes after the pretend break-in |
| Error about a missing name like `lookback` | The `let` lines at the top were left out. Paste the whole file |
| Unexpected charges | Stop the VM, then delete the resource group |

## Where to learn more

- [README.md](README.md) explains what each detection looks for.
- [detection-catalog.md](detection-catalog.md) lists the reasoning, false positives, and tuning ideas.
