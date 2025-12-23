# Windows Event Logs — Day 9 (Logon → Privilege signal → Process evidence)

Today I followed the Day 9 plan exactly on my Windows 10 VM.  
I wanted to create a clean “story” in the Security log that I could explain like a real SOC triage case.

What I did (on purpose):
- 1 failed login for `labuser` (wrong password)
- 1 successful login for `labuser`
- opened **Task Manager**
- opened **cmd.exe** and ran a couple commands
- opened **Chrome** and visited a site

Then I went to **Event Viewer → Windows Logs → Security** and pulled only the events that prove each step.

---

## 1) Failed logon — Event ID 4625

**Event:** 4625 (Audit Failure)  
- **Time:** 12/21/2025 4:33:01 AM  
- **Account:** `labuser`  
- **Logon Type:** 2 (Interactive)  
- **Failure reason:** Unknown user name or bad password  
- **Status/SubStatus:** `0xC000006D` / `0xC000006A` (bad password)  
- **Source address:** `127.0.0.1` (local)  
- **Process noted:** `C:\Windows\System32\svchost.exe`

**How I interpret it:**  
This looks like a simple user mistake. It’s one attempt, it’s interactive, and it’s coming from the local machine.

---

## 2) Successful logon — Event ID 4624

**Event:** 4624 (Audit Success)  
- **Time:** 12/21/2025 4:33:07 AM  
- **Account:** `WIN10\labuser`  
- **Logon Type:** 2 (Interactive)  
- **Logon ID:** `0x115AE4`  
- **Linked Logon ID:** `0x115AC7`  
- **Workstation / Source:** `WIN10` / `127.0.0.1` (local)  
- **Elevated Token:** **No** (not an admin-elevated session)

**Why I care about this event:**  
This is the key “proof” that the login succeeded for my actual user.  
Also, seeing **Elevated Token = No** is useful — it tells me this session wasn’t created with admin elevation.

---

## 3) Process creation — Event ID 4688 (post-login activity)

After the successful login, I looked for process creation events that match what I opened.

### A) Task Manager started
**Event:** 4688  
- **Time:** 12/21/2025 4:33:31 AM  
- **New process:** `C:\Windows\System32\Taskmgr.exe`  
- **User:** `WIN10\labuser`  
- **Parent process:** `C:\Windows\explorer.exe`  
- **Mandatory Label:** Medium

**What it tells me:**  
Explorer spawning Task Manager under the logged-in user is normal.

---

### B) Command Prompt started
**Event:** 4688  
- **Time:** 12/21/2025 4:33:36 AM  
- **New process:** `C:\Windows\System32\cmd.exe`  
- **User:** `WIN10\labuser`  
- **Parent process:** `C:\Windows\explorer.exe`  
- **Mandatory Label:** Medium

**What it tells me:**  
This matches my test step (open cmd). In a real SOC case, cmd.exe is a normal tool, but it becomes interesting if it’s launched by unusual parents (like Office apps, script hosts, or browsers).

---

### C) Chrome started
**Event:** 4688  
- **Time:** 12/21/2025 4:35:01 AM  
- **New process:** `C:\Program Files\Google\Chrome\Application\chrome.exe`  
- **User:** `WIN10\labuser`  
- **Parent process:** `C:\Windows\explorer.exe`  
- **Mandatory Label:** Medium

**What it tells me:**  
Chrome was launched normally by the user from Explorer, which fits the test (open Chrome and browse).

---

## Timeline (the story, in order)

- **04:33:01** — 4625 failed login (`labuser`, wrong password, local)
- **04:33:07** — 4624 successful login (`labuser`, interactive, **not elevated**)
- **04:33:31** — 4688 Task Manager created (Explorer → Taskmgr)
- **04:33:36** — 4688 cmd.exe created (Explorer → cmd)
- **04:35:01** — 4688 Chrome created (Explorer → Chrome)

This is exactly the chain I was aiming for: login attempt → login success → user processes after login.

---

## Mini SOC Ticket (6–8 lines)

**Title:** Local failed logon followed by successful interactive logon and normal user activity  
**Summary:** A single failed interactive logon attempt for `labuser` was observed, followed shortly by a successful interactive logon.  
**Evidence:** 4625 at 04:33:01 (Logon Type 2, SubStatus 0xC000006A), then 4624 at 04:33:07 for `WIN10\labuser` (Logon Type 2).  
**Privilege context:** Successful session shows **Elevated Token = No**, suggesting standard user session.  
**Post-logon activity:** 4688 shows Taskmgr.exe, cmd.exe, and chrome.exe launched by `explorer.exe` under `labuser`.  
**Assessment:** Likely benign user behavior (local source 127.0.0.1, expected process parents, no suspicious burst pattern).  
**Next checks:** If this was an alert, I’d look for repeated 4625 failures over time, remote source IPs, or unusual parent processes spawning cmd.exe.
