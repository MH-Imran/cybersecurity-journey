# windows-event-logs-day14.md 

Today I practiced a simple SOC-style investigation chain in **Windows Security logs**:
**failed login → successful login → process creation**.

I generated the events in my Windows VM so I can train on real evidence instead of “imaginary logs”.

---

## Evidence (Security log)

### 1) Failed login — 4625
- **Time:** 2026-01-14 12:11:55  
- **User:** labuser  
- **Host:** win10  
- **Logon Type:** 2 (Interactive)  
- **Source:** 127.0.0.1 (Local)  
- **Status/SubStatus:** 0xC000006D / 0xC000006A *(wrong password)*  
- **Process:** C:\Windows\System32\svchost.exe  

What I think this means:  
A local interactive login failed, and the codes match a typical “wrong password” situation.

---

### 2) Successful login — 4624
- **Time:** 2026-01-14 12:12:01  
- **User:** labuser  
- **Host:** win10  
- **Logon Type:** 2 (Interactive)  
- **Source:** 127.0.0.1 (Local)  
- **Status:** Success  
- **Process:** C:\Windows\System32\svchost.exe  

This happened **6 seconds** after the failure, which fits a normal human mistake → then success.

---

### 3) Process creation — 4688
- **Time:** 2026-01-14 12:16:32  
- **User:** labuser  
- **Host:** win10  
- **Process:** C:\Windows\System32\cmd.exe *(Elevated)*  
- **Parent:** C:\Windows\explorer.exe  
- **Logon ID:** 0x1A8D27  

What I think this means:  
`cmd.exe` was launched interactively (parent `explorer.exe`) and it was **elevated (admin)**.  
In a real environment, an elevated shell is worth checking because it can be used for discovery, persistence, or tool execution.

---

## Timeline (clean story)
- **12:11:55** — 4625: `labuser` failed an interactive login on `win10` (local), wrong password pattern.  
- **12:12:01** — 4624: `labuser` successfully logged in interactively (same host/source).  
- **12:16:32** — 4688: elevated `cmd.exe` launched by `explorer.exe` (same user context).  

---

## Mini SOC ticket (practice)

**Title:** Failed login followed by success; elevated CMD observed shortly after  
**Time window:** 2026-01-14 12:11–12:17  
**Host/User:** win10 / labuser  

**Summary:**  
A failed local interactive login was followed by a successful login 6 seconds later. A few minutes after the login, an elevated command prompt was launched interactively.

**Evidence:**
- 4625 @ 12:11:55 — interactive logon failure (wrong password codes)
- 4624 @ 12:12:01 — interactive logon success (same user/host/source)
- 4688 @ 12:16:32 — `cmd.exe` created (parent `explorer.exe`, elevated)

**Assessment:** Likely benign (lab typo + normal admin action), but elevated shell would be a pivot point in a real case.  
**Next steps (real SOC checks):**
- Check for additional 4688 events after the elevated shell (suspicious tools/commands)
- Look for persistence indicators (scheduled tasks/services) if activity looks unusual
- Correlate by **Logon ID** to confirm the process belongs to the same session

---

## What I learned
- The **4625 → 4624** pattern can be normal, but it’s still a good “story starter” for triage.
- **Elevation** is a big context clue; it increases impact if the session is compromised.
- The **Logon ID** is useful for tying actions back to the same session.
