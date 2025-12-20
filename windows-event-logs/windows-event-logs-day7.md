# Windows Event Logs — Day 7 (My “failed → success → activity” test)

Today I didn’t want random Windows logs. I wanted a clean, controlled story that I could explain like a junior SOC analyst.

So I did this on purpose in my Windows 10 VM:

- typed the wrong password **two times**
- logged in correctly
- opened **Notepad**
- later opened **Chrome**

Then I went into **Event Viewer → Security** and tried to prove each step using real Event IDs.

---

## What I was trying to answer
If someone asked me: “What happened on this machine?”, I wanted to be able to say:

- there were failed login attempts
- then a successful login
- then normal user activity

Not a guess — evidence.

---

## 1) Failed logons (Event ID 4625)

### Attempt #1 — failed login
- **Time:** 12/20/2025 4:22:40 AM  
- **Event:** 4625 (Audit Failure)  
- **Account:** `labuser`  
- **Logon Type:** 2 (Interactive)  
- **Reason shown:** “Unknown user name or bad password”  
- **SubStatus:** `0xC000006A` (bad password)  
- **Source:** `127.0.0.1` (local)  

### Attempt #2 — failed login again
- **Time:** 12/20/2025 4:22:44 AM  
- **Event:** 4625 (Audit Failure)  
- **Account:** `labuser`  
- **Logon Type:** 2 (Interactive)  
- **SubStatus:** `0xC000006A` (bad password)  
- **Source:** `127.0.0.1` (local)  

**How it looked to me:**  
This felt like normal “human mistake” behavior. Two quick failures, both local, same account, and the error points to the password.

If this was a real investigation, I’d only get worried if:
- it kept repeating for minutes/hours
- it came from a remote IP
- the account was an admin account
- or the pattern looked automated

---

## 2) Successful logon (Event ID 4624)

- **Time:** 12/20/2025 4:22:48 AM  
- **Event:** 4624 (Audit Success)  
- **Account:** `WIN10\labuser`  
- **Logon Type:** 2 (Interactive)  
- **Logon ID:** `0x1F8488`  
- **Linked Logon ID:** `0x1F84A5`  
- **Source:** `127.0.0.1` (local)  

**Why this mattered:**  
This is the event I was looking for earlier in my practice days — a real successful login for my user, not a service/system login.

This one actually matches my test.

---

## 3) Process creation (Event ID 4688)

### Notepad launched
- **Time:** 12/20/2025 4:23:15 AM  
- **Event:** 4688 (Process Creation)  
- **New process:** `C:\Windows\System32\notepad.exe`  
- **Parent process:** `C:\Windows\explorer.exe`  
- **User in event:** `labuser` (shown in the event fields)  
- **Related Logon ID (target):** `0x1F8488`  

**My takeaway:**  
Explorer starting Notepad right after login looks totally normal.

### Chrome launched
- **Time:** 12/20/2025 4:30:51 AM  
- **Event:** 4688 (Process Creation)  
- **New process:** `C:\Program Files\Google\Chrome\Application\chrome.exe`  
- **Creator:** `WIN10\labuser`  
- **Logon ID (creator):** `0x1F84A5`  

**My takeaway:**  
Chrome shows up later, still under my user, which matches what I did.  
Also interesting: the logon IDs are not identical, but Windows links sessions (Logon ID vs Linked Logon ID), so I’m starting to see how correlations can get tricky if I don’t pay attention.

---

## The timeline (the story in plain language)

- **04:22:40** — I typed the wrong password (failed login)
- **04:22:44** — wrong password again (failed login)
- **04:22:48** — I logged in correctly (successful interactive login)
- **04:23:15** — I opened Notepad (process creation)
- **04:30:51** — I opened Chrome (process creation)

That is exactly the “chain” I wanted to capture.

---

## Mini SOC Ticket (short and realistic)

**Case:** Two failed logins followed by successful login and normal user activity  
**What I saw:** `labuser` had two failed interactive logon attempts, then successfully logged on locally.  
**Evidence:** 4625 at 04:22:40 + 04:22:44 (Logon Type 2, SubStatus 0xC000006A), then 4624 at 04:22:48 (Logon Type 2).  
**Follow-up activity:** 4688 shows Notepad at 04:23:15 and Chrome at 04:30:51 under `labuser`.  
**My call:** Looks like normal user error (small count, tight time gap, local source 127.0.0.1).  
**If this was work:** I’d check if this pattern repeats, if the same account gets hit again later, or if any remote IPs show up in similar failures.
