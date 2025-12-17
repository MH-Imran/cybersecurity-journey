# Windows Event Logs — Day 5 (Two Wrong Passwords in a Row)

Today I tried something simple on purpose: I entered the wrong password a couple of times and then checked what Windows recorded. I’m doing this because I want to get used to reading logs like a SOC analyst — not just “seeing an Event ID”, but understanding what story the events are telling.

---

## What happened (in real life)
I tried to log in as `vboxuser` and typed the wrong password twice.

After that, Windows kept doing its normal background stuff (services starting etc.), and I captured one successful logon event too.

---

## The logs I saved (in time order)

### 1) Failed logon (4625)
- **Time:** 12:01:31 AM
- **Username:** `vboxuser`
- **Logon Type:** 2
- **Failure reason:** “Unknown user name or bad password”
- **SubStatus:** `0xC000006A` (bad password)
- **Caller process:** `C:\Windows\System32\svchost.exe`
- **Source address:** `127.0.0.1`

### 2) Failed logon again (4625)
- **Time:** 12:01:42 AM
- Same username, same logon type, same failure reason
- Still shows `127.0.0.1`

So basically: I messed up the password twice, and Windows recorded two failures.

### 3) Successful logon event (4624)
- **Time:** 12:02:17 AM
- **Logon Type:** 5
- **Account:** SYSTEM
- **Process:** `services.exe`

This one is important because it’s *not* me logging in successfully. It’s Windows doing Windows things.

---

## The details I’m training myself to read

### Failure reason (what it really means)
The text says “unknown user name or bad password,” but the useful clue is the **SubStatus**:

- `0xC000006A` = bad password

So this wasn’t a random username being guessed — it looks like the account is real, I just typed the password wrong.

---

### Logon type (how the attempt happened)
Both failures are:

- **Logon Type 2** = interactive login

That means it was a local login attempt (like at the VM console / login screen).  
If I saw **Logon Type 3**, I would start thinking about network attempts (more suspicious).

---

### Repeated attempts vs single attempt
I saw **two failed attempts** back to back.

Two attempts by itself doesn’t scare me. It feels very normal (people mistype passwords).  
If this was 20 attempts, or multiple usernames, or happening every minute, then it would look different.

---

### Time pattern (this is the “story” part)
Here’s the timing:

- 12:01:31 AM → first failure
- 12:01:42 AM → second failure (**11 seconds later**)
- 12:02:17 AM → successful SYSTEM logon (**about 35 seconds after**)

This timing feels human.  
It looks like: “wrong password → try again quickly → done.”

It does *not* look like an automated brute-force tool.

---

## My judgment: user mistake or attack?
Based on:
- Logon Type 2 (interactive)
- Only two attempts
- Bad password (not unknown user)
- Source address is local (`127.0.0.1`)
- No weird remote IP

This looks like **normal user behavior / testing**, not an attack.

---

## What I learned today
This was a good lesson for me:

Just because I see a “successful logon” (4624) doesn’t mean the user got in.  
The successful one here was **SYSTEM, Logon Type 5** — basically background service activity.

So I need to be careful with assumptions and always check:
- who logged in
- logon type
- what process is involved
- what happened right before and after

---

## What I want to try next
Next time I want to push it a bit:
- create **more** failed attempts and see how the pattern looks
- test a **network logon failure** (Logon Type 3) if possible
- practice writing a short “SOC conclusion” like a real alert/ticket summary
