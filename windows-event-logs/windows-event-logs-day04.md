# Windows Event Logs — Day 4 (My First Real 4625 “Failed Logon” Note)

Today I finally captured a clean **Event ID 4625** while testing wrong passwords on purpose.  
I’ve seen people talk about 4625 a lot, but seeing it in my own VM made it feel much more real.

My goal wasn’t to memorize everything — I just wanted to pull out the few details a SOC analyst would care about.

---

## The event I looked at
- **Log:** Security
- **Event ID:** 4625 (Audit Failure)
- **Computer:** win10
- **Time (local):** **12/16/2025 11:59:27 PM**

This happened right after I typed a wrong password.

---

## What I pulled out (the “SOC useful” parts)

### Username
- **Username:** `vboxuser`
- **Domain:** `WIN10`

So the failed login attempt was for the account `vboxuser`.

---

### Logon type
- **Logon Type:** `2`

From what I understand, **Type 2 = interactive logon**, meaning it was a local login attempt (like on the Windows login screen / VM console), not a remote login over the network.

That already makes it feel less “attacker-ish” and more like a normal user mistake.

---

### Failure reason
The event message says:
- **Failure Reason:** “Unknown user name or bad password.”

But the more specific part is here:
- **Sub Status:** `0xC000006A`

From what I learned, `0xC000006A` points to **bad password**.

So in simple words: the username was valid, but the password was wrong.

---

### Time pattern
- This looks like **a single failed attempt**
- No rapid repeats in this snippet
- It happened **late at night**, but without repetition it still looks like normal testing / normal mistake

If I saw this many times in a row (especially over the network), I would start thinking about brute force.  
But one attempt like this by itself doesn’t scream “attack”.

---

## Extra details that I noticed (but I’m not 100% sure yet)
- **Caller Process Name:** `C:\Windows\System32\consent.exe`
- **Source Network Address:** `::1`

`::1` is the IPv6 loopback address (basically “this machine itself”), which again fits the idea that it was local activity in my VM.

I’m still learning how to interpret `consent.exe` in this context, but I’m keeping it noted.

---

## What this helped me understand
Before this, “4625” was just a number.

Now it’s clearer to me that a failed login event can be broken down into:
- **who** tried to log in
- **how** they tried (logon type)
- **why** it failed (substatus)
- **pattern** (one time vs repeated attempts)

This is the kind of thinking I need for SOC investigations.

---

## Next thing I want to test
- Create **3–5 failed logons in a short time**
- Then do **one successful logon**
- Compare:
  - How repeated 4625 events look
  - Whether anything changes when 4624 happens after failures

That should feel closer to a real “investigation” scenario.
