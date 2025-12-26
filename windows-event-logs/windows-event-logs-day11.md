# Windows Event Logs — Day 11  
**Focus:** Failed → Successful interactive logon (4625 vs 4624) + what the pattern “feels like” from a SOC view.

---

## What I tried to simulate
Today I intentionally created a simple authentication story:

1) Try the wrong password for **labuser** (to generate failures)  
2) Log in successfully as **labuser** (to confirm what “normal success” looks like)  
3) Compare the fields that matter in a SOC investigation

---

## Evidence Collected (Event Viewer → Security)

### 1) Failed Logons (Event ID: 4625)

I got **two failed interactive logons** for the same account.

#### Failed #1  
- **Time:** 12/26/2025 10:13:39 AM  
- **Target username:** `labuser`  
- **Logon type:** `2` (Interactive)  
- **Failure reason (text):** Unknown user name or bad password  
- **Status/SubStatus:** `0xC000006D / 0xC000006A`  
- **Source IP:** `127.0.0.1` (local machine)  
- **Caller process:** `C:\Windows\System32\svchost.exe`  

#### Failed #2  
- **Time:** 12/26/2025 10:14:18 AM  
- **Target username:** `labuser`  
- **Logon type:** `2` (Interactive)  
- **Failure reason (text):** Unknown user name or bad password  
- **Status/SubStatus:** `0xC000006D / 0xC000006A`  
- **Source IP:** `127.0.0.1`  
- **Caller process:** `C:\Windows\System32\svchost.exe`  

##### What this means (my notes)
- The **same username**, **same logon type**, **same status codes**, **same source (127.0.0.1)** strongly suggests:
  - this was **local user error** (me typing wrong password)  
  - not a remote brute force attempt  
- Also, the time gap is about **39 seconds**, so it looks like a human retry, not automated spray.

---

### 2) Successful Logons (Event ID: 4624)

I got **two successful interactive logons** for `WIN10\labuser`.

#### Success #1  
- **Time:** 12/26/2025 10:14:22 AM  
- **Account logged in:** `WIN10\labuser`  
- **Logon type:** `2` (Interactive)  
- **Logon ID:** `0x1A8D27`  
- **Linked Logon ID:** `0x1A8D44`  
- **Elevated token:** `Yes`  
- **Source IP:** `127.0.0.1`  
- **Process:** `C:\Windows\System32\svchost.exe`

#### Success #2  
- **Time:** 12/26/2025 10:15:34 AM  
- **Account logged in:** `WIN10\labuser`  
- **Logon type:** `2` (Interactive)  
- **Logon ID:** `0x2F8111`  
- **Linked Logon ID:** `0x2F813B`  
- **Elevated token:** `Yes`  
- **Source IP:** `127.0.0.1`  
- **Process:** `C:\Windows\System32\svchost.exe`

##### What I noticed
- The key difference vs failures is obvious:  
  - 4624 confirms `WIN10\labuser` with a real **Logon ID** that I can later use to correlate activity.
- The **Elevated Token = Yes** matters:
  - it suggests this session could run admin-level actions (depending on UAC + context).
  - if I see suspicious process launches after this, I’d take it more seriously.

---

## Pattern check (quick SOC thinking)
- **Repeated attempts:** 2 failures, then success  
- **Time rhythm:** looks human (tens of seconds), not machine spam  
- **Source:** `127.0.0.1` makes it local, not remote  
- **Verdict:** very likely “user typed wrong password twice and then succeeded.”

---

## What I want to do next
- Correlate the successful **Logon ID** with later process events (4688) to build a timeline:
  - login → process start → suspicious command → network actions

---
