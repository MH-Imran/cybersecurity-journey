# Windows Event Logs — Day 12  
**Focus:** Process evidence (4688) after login — CMD, PowerShell, Chrome.  
Goal: see how “normal admin work” looks in logs, and how it could also look like attacker behavior.

---

## The scenario (what I actually did)
After getting a successful logon, I launched a few tools:

- `cmd.exe`
- `powershell.exe`
- `chrome.exe`

Then I checked the Security log to confirm **process creation events**.

---

## Evidence Collected (Event Viewer → Security)

### 1) CMD launched (Event ID: 4688)

- **Time:** 12/26/2025 10:16:32 AM  
- **New process:** `C:\Windows\System32\cmd.exe`  
- **Target user:** `WIN10\labuser`  
- **Target Logon ID:** `0x1A8D27`  
- **Parent process:** `C:\Windows\explorer.exe`  
- **Token elevation type:** `%%1937`  
- **Mandatory label:** High Mandatory Level  

#### My interpretation
- Parent is `explorer.exe`, which fits a normal “user opened CMD from desktop/start menu”.
- High integrity + elevation token signals tell me this is likely running elevated (admin context).

---

### 2) PowerShell launched (Event ID: 4688)

- **Time:** 12/26/2025 10:17:14 AM  
- **New process:** `C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe`  
- **Target user:** `WIN10\labuser`  
- **Target Logon ID:** `0x1A8D27`  
- **Parent process:** `C:\Windows\System32\RuntimeBroker.exe`  
- **Token elevation type:** `%%1937`  
- **Mandatory label:** High Mandatory Level  

#### My interpretation
This one is interesting:
- RuntimeBroker as parent is not “wrong”, but it’s less obvious than Explorer.
- In real SOC work, PowerShell is high-signal, so I would check:
  - command line arguments (unfortunately empty here)
  - any script execution policy changes
  - network connections right after this event

---

### 3) Chrome launched (Event ID: 4688)

- **Time:** 12/26/2025 10:17:56 AM  
- **New process:** `C:\Program Files\Google\Chrome\Application\chrome.exe`  
- **Creator / Subject:** `WIN10\labuser` (Logon ID shown: `0x1A8D44`)  
- **Parent process:** `C:\Windows\explorer.exe`  
- **Token elevation type:** `%%1938`  
- **Mandatory label:** Medium Mandatory Level  

#### My interpretation
- This looks like a standard user-launched browser.
- Medium integrity is normal for Chrome (usually not elevated).

---

## Mini timeline (the story in plain words)
- I had failed logons first (wrong password).  
- Then I successfully logged in as `labuser`.  
- After login, I started CMD and PowerShell (high integrity).  
- Then I opened Chrome (medium integrity).

From a SOC view:
- CMD + PowerShell right after login can be totally normal…  
  but it’s also exactly what attackers do after gaining access.  
- That’s why the missing **command line** field matters — it would show intent.

---

## Note about 4800/4801 (why I didn’t find them)
I searched for **4800/4801** (workstation lock/unlock), but they didn’t show up.

Most likely reasons:
- Lock/unlock auditing wasn’t triggered (I didn’t lock the screen), or
- the system policy for those events is not enabled / not generating in this lab flow.

I’ll generate them next time by:
- locking the screen (Win + L) → waiting 10–20 seconds → unlocking.

---
