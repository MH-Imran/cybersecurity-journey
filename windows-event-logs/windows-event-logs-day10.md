# Windows Event Logs — Day 10 (Logon → Privilege signal → Process evidence)

**Date:** 2025-12-22  
**Host:** `win10`  
**User tested:** `WIN10\labuser`  
**Log source:** Event Viewer → Windows Logs → Security

Today I repeated the same idea as Day 9, but I paid extra attention to one thing:  
**what changes when “Elevated Token = Yes” shows up in the successful logon event.**

---

## What I did (quick lab steps)
- Typed the wrong password once for `labuser`
- Logged in successfully right after
- Opened **CMD**
- Opened **Notepad**

I kept it simple so the logs stay readable.

---

## 1) Failed logon — Event ID 4625

This is the moment I intentionally entered the wrong password.

**Key fields I noted**
- **Event ID:** 4625 (Audit Failure)
- **Target user:** `labuser` (Domain: `WIN10`)
- **Logon Type:** `2` (Interactive / local)
- **Status / SubStatus:** `0xC000006D` / `0xC000006A` (bad username or bad password / bad password)
- **Source address:** `127.0.0.1` (localhost)
- **Caller process:** `C:\Windows\System32\svchost.exe`
- **Time (UTC):** `2025-12-22T09:45:08Z`

**My take**
One failed interactive logon from localhost is almost always just a typo or wrong password entry. No remote source, no spray pattern.

---

## 2) Successful logon — Event ID 4624

This is the “correct password” login right after the failure.

**Key fields I noted**
- **Event ID:** 4624 (Audit Success)
- **New Logon:** `WIN10\labuser`
- **Logon Type:** `2` (Interactive)
- **Target Logon ID:** `0xBFC68`
- **Linked Logon ID:** `0xBFC85`
- **Elevated Token:** **Yes**
- **Source address:** `127.0.0.1`
- **Process name:** `C:\Windows\System32\svchost.exe`
- **Time (UTC):** `2025-12-22T09:45:19Z`

**My take**
This is the part I would not ignore in a real SOC:  
**“Elevated Token: Yes”** means the session has elevated rights (or was created in a context that results in elevation). It doesn’t automatically mean malicious, but it raises the “check what happened next” priority.

---

## 3) Process creation — Event ID 4688

After the successful logon, I checked what processes spawned under the session.

### A) CMD started
- **Event ID:** 4688
- **User:** `labuser`
- **Subject Logon ID:** `0xBFC85`
- **New process:** `C:\Windows\System32\cmd.exe`
- **Parent:** `C:\Windows\explorer.exe`
- **Integrity:** Medium
- **Time (UTC):** `2025-12-22T09:45:43Z`

### B) Notepad started
- **Event ID:** 4688
- **User:** `labuser`
- **Subject Logon ID:** `0xBFC85`
- **New process:** `C:\Windows\System32\notepad.exe`
- **Parent:** `C:\Windows\explorer.exe`
- **Integrity:** Medium
- **Time (UTC):** `2025-12-22T09:46:15Z`

**Note I noticed**
My 4688 events still show an empty **CommandLine** field. So I can prove the process started, but not what was typed inside CMD. In a real environment I’d enable command-line auditing or use Sysmon for better visibility.

---

## Simple timeline (UTC)
- **09:45:08Z** → 4625 failed logon (labuser, Logon Type 2, localhost)
- **09:45:19Z** → 4624 successful logon (labuser, elevated token = Yes)
- **09:45:43Z** → 4688 cmd.exe (parent explorer.exe)
- **09:46:15Z** → 4688 notepad.exe (parent explorer.exe)

---

## What I learned today (in my own words)
- The **same user** can generate both failed and successful logons within seconds, and that’s usually normal.
- **Elevated Token = Yes** is worth noticing, because that’s where privilege-related investigations often begin.
- Process creation logs (4688) are useful, but without command-line logging, I’m only seeing “what started,” not “what was done.”

---

## Mini SOC Ticket (6–8 lines)

**Title:** Local failed logon followed by elevated interactive session and normal process launches  
**Time (UTC):** 2025-12-22 09:45–09:46  
**Host:** win10  
**User:** WIN10\labuser  
**Summary:** 4625 shows one failed interactive logon for `labuser` from localhost (127.0.0.1) with bad password status/substatus.  
**Follow-up:** 4624 shows a successful interactive logon for `labuser` shortly after with **Elevated Token = Yes** (Logon ID 0xBFC68 / Linked 0xBFC85).  
**Process evidence:** 4688 shows `cmd.exe` and `notepad.exe` spawned from `explorer.exe` under Subject Logon ID 0xBFC85.  
**Assessment/Action:** Likely benign lab activity; note elevated token and continue monitoring; enable command-line auditing (or Sysmon) for deeper CMD visibility.
