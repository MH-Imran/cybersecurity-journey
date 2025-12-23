# Windows Event Logs — Day 2 (Process Creation)

Today I focused mainly on **process creation events** in the Windows Security log. My goal was not to understand every single field, but to recognize what “normal” looks like and what kind of details Windows actually records.

---

## Event I reviewed (Security → Event ID 4688)

**Event ID:** 4688 (A new process has been created)  
**Process name:** `C:\Windows\System32\smss.exe`  
**Computer:** `win10`  
**Level:** Information (Audit Success)

This event showed a process being created, and it was related to `smss.exe`.

---

## What I understood from this event (in my own words)

### `smss.exe` looks like a normal Windows process
From the name and the location (`System32`), this seems like a legitimate Windows component. I also noticed something interesting:

- **New Process Name:** `smss.exe`
- **Creator Process Name:** `smss.exe`

So basically, it looks like `smss.exe` started and the creator process is also `smss.exe`. That seems weird at first, but I think it can happen during early system startup or system-managed process creation where Windows is booting / initializing.

---

## Who created it?
The **Creator Subject Security ID** was:

- `SYSTEM` (SID: `S-1-5-18`)
- Logon ID: `0x3E7`

From what I’ve seen so far, `SYSTEM` appearing here often means “this was created by the operating system itself,” not by a normal user.

So right now, this event looks like **normal OS behavior**, not suspicious activity.

---

## Fields that confused me today
These were the parts I didn’t fully understand yet (and that’s okay):

- **Token Elevation Type:** `%%1936`  
  I saw Windows explain token types (full / elevated / limited), but the value in the event is still confusing to me. I understand the *idea*, but I’m not fully confident reading it from real logs yet.

- **Command line:** empty  
  That made me realize that not every 4688 event will show a clean command line, especially for system processes.

---

## SOC takeaway (simple)
This log helped me understand why Event ID 4688 is useful:

- It records **what process started**
- It can show **parent process**
- It can show **who created it** (user vs SYSTEM)
- In real investigations, unusual parent/child relationships would be a big clue

For now, `smss.exe` started by SYSTEM looks normal.

---

## Next time
- Try to find 4688 events for apps I open myself (like Notepad or Chrome)
- Compare the “SYSTEM created process” vs “User created process”
- Try to capture a 4688 that includes a visible command line
