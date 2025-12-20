# Windows Event Logs — Day 8 (Same test, but written cleaner)

For Day 8 I used the **same set of logs** from my controlled login test.  
My goal this time wasn’t to collect more events — it was to write the story in a cleaner way so I can review it later and quickly understand what happened.

I’m basically training myself to answer one simple SOC question:

> “If I saw these logs at work, what would I conclude?”

---

## What I did (on purpose)
On my Windows 10 VM I intentionally:
- typed the wrong password **two times**
- logged in successfully as `labuser`
- opened **Notepad**
- later opened **Chrome**

Then I went to **Event Viewer → Security** and pulled the matching events.

---

## Evidence I collected (the important events only)

### 1) Two failed interactive logons — Event ID 4625
I got **two** 4625 events within a few seconds:

**Failed logon #1**
- **Time:** 12/20/2025 4:22:40 AM
- **Account:** `labuser`
- **Logon Type:** 2 (Interactive)
- **Reason shown:** Unknown user name or bad password
- **SubStatus:** `0xC000006A` (bad password)
- **Source:** `127.0.0.1` (local)

**Failed logon #2**
- **Time:** 12/20/2025 4:22:44 AM
- **Account:** `labuser`
- **Logon Type:** 2 (Interactive)
- **SubStatus:** `0xC000006A`
- **Source:** `127.0.0.1` (local)

**My simple read:**  
Two quick failures + local source + bad password substatus = looks like normal user mistake, not an attack.

If this was thousands of attempts, or coming from a remote IP, then the story changes.

---

### 2) Successful interactive logon — Event ID 4624
A few seconds after the failures, I captured the success:

- **Time:** 12/20/2025 4:22:48 AM
- **Event ID:** 4624 (Audit Success)
- **Account:** `WIN10\labuser`
- **Logon Type:** 2 (Interactive)
- **Logon ID:** `0x1F8488`
- **Linked Logon ID:** `0x1F84A5`
- **Source:** `127.0.0.1` (local)

**Why I like this event:**  
This is the kind of 4624 I actually care about right now — it’s a real user logon, not a background SYSTEM/service logon.

---

### 3) Process creation after login — Event ID 4688

#### Notepad started
- **Time:** 12/20/2025 4:23:15 AM
- **Event ID:** 4688
- **Process:** `C:\Windows\System32\notepad.exe`
- **Parent:** `C:\Windows\explorer.exe`
- **User context:** `labuser` appears in the event
- **Related Logon ID:** `0x1F8488`

This looks normal: user logs in → Explorer → Notepad.

#### Chrome started
- **Time:** 12/20/2025 4:30:51 AM
- **Event ID:** 4688
- **Process:** `C:\Program Files\Google\Chrome\Application\chrome.exe`
- **Creator:** `WIN10\labuser`
- **Creator Logon ID:** `0x1F84A5`

This also looks normal: user session continues → Chrome launched.

One thing I’m still learning: Windows sometimes uses **linked logon IDs**, so I’m trying to be careful not to assume one ID = one event forever.

---

## The timeline (what happened, in order)
- **04:22:40** — failed login for `labuser` (wrong password)
- **04:22:44** — failed login for `labuser` again
- **04:22:48** — successful login for `labuser` (interactive)
- **04:23:15** — Notepad created (4688)
- **04:30:51** — Chrome created (4688)

This matches my actions almost perfectly.

---

## Mini SOC ticket (my conclusion)
**Title:** Local failed logons followed by successful logon and normal user activity  
**Summary:** Two failed interactive logons for `labuser` were observed, followed immediately by a successful interactive logon from the local host.  
**Evidence:** 4625 events at 04:22:40 and 04:22:44 (Logon Type 2, bad password substatus), then 4624 at 04:22:48 (Logon Type 2 for `labuser`).  
**Post-logon activity:** 4688 shows Notepad (04:23:15) and Chrome (04:30:51) created under `labuser`.  
**Assessment:** Likely benign user error; pattern is low volume, local, and followed by expected user activity.  
**Next checks:** If this was a real case, I’d hunt for repeated 4625 spikes, unusual timing, different source IPs, or suspicious processes right after login.
