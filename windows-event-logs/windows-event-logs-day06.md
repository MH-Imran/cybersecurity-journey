# Windows Event Logs — Day 6 (My Own Event Chain: 4625 → 4624 → 4688)

Today I tried to build a simple “event chain” from things I actually did, instead of reading random events and guessing.  
My goal was very practical:

1) Create a failed login (wrong password)  
2) Confirm something logged in successfully  
3) Open an app and catch the process creation

I’m treating this like a mini SOC exercise: **what happened, when, and does it look normal?**

---

## The three events I captured

### 1) Failed logon — Event ID 4625
- **Time:** 12/18/2025 2:09:34 AM
- **Event ID:** 4625 (Audit Failure)
- **User attempted:** `labuser`
- **Logon Type:** 2 (Interactive)
- **Failure Reason:** “Unknown user name or bad password”
- **Status:** `0xC000006D`
- **SubStatus:** `0xC000006A` (bad password)
- **Process:** `C:\Windows\System32\svchost.exe`
- **Source Address:** `127.0.0.1`

**How I understand it:**  
This is exactly what I expected when I typed the wrong password.  
It’s an interactive login attempt (so local / VM console), and the substatus points to “bad password”.

Nothing about this looks like a remote attack — it looks like me testing.

---

### 2) “Success” logon — Event ID 4624 (but it’s not my user)
- **Time:** 12/18/2025 2:13:28 AM
- **Event ID:** 4624 (Audit Success)
- **Logged on account:** `NT AUTHORITY\SYSTEM`
- **Logon Type:** 5 (Service)
- **Process:** `C:\Windows\System32\services.exe`

**Important thing I noticed:**  
At first glance, I thought “okay, this is the successful login after my failure.”

But when I read the details, it’s not `labuser`. It’s **SYSTEM**, and Logon Type is **5** (service logon).  
So this is most likely normal Windows service activity, not me logging in successfully.

This was a good reminder:
> A “successful logon” event doesn’t always mean a human user logged in.

---

### 3) Process creation — Event ID 4688 (Chrome)
- **Time:** 12/18/2025 2:18:29 AM
- **Event ID:** 4688 (Process Creation)
- **Creator Subject:** `WIN10\labuser`
- **New Process Name:** `C:\Program Files\Google\Chrome\Application\chrome.exe`
- **Parent Process:** `chrome.exe` (Chrome starting another Chrome process)
- **Logon ID:** `0x473516`
- **Mandatory Label:** Low Mandatory Level

**How I understand it:**  
This is the strongest “human activity” evidence in this chain.  
It clearly shows *my user* (`labuser`) creating a process, and that process is Chrome.

Also, the parent process being Chrome itself makes sense because Chrome spawns multiple processes.

---

## My “event chain” summary (the story)
Here’s the simple timeline:

- **2:09 AM** → failed login attempt for `labuser` (wrong password)  
- **2:13 AM** → SYSTEM service logon event (background activity)  
- **2:18 AM** → `labuser` starts Chrome (process creation)

This chain looks **normal** and matches exactly what I was doing during the test.

---

## Mini SOC ticket-style conclusion (short)
**Finding:** A local interactive login attempt failed for `labuser` due to bad password, followed later by normal system activity and then normal user activity (Chrome launched).  
**Impact:** No signs of brute force or external access.  
**Why:** Logon Type 2 + local source `127.0.0.1` + single attempt + expected user behavior afterward.  
**Next checks (if this was a real case):**
- Look for more 4625 events around the same time (repetition matters)
- Confirm if there is a 4624 for `labuser` (interactive) near the session start
- Check if any unusual processes were launched after authentication

---

## What I learned today
The biggest lesson for me was this:

I can’t just look for “a 4624” and assume it’s the successful login I’m expecting.  
I have to check:
- **who** logged in (SYSTEM vs my user)
- **logon type**
- **what process caused it**

This is exactly the kind of mistake that could confuse an investigation if I’m not careful.

---

## Next thing I want to improve
- Try to capture a 4624 event where **labuser** successfully logs on (Logon Type 2)
- Correlate that user logon to process creation using the same Logon ID (if possible)
- Keep practicing “event chains” instead of isolated events
