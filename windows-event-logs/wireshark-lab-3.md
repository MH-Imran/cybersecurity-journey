# Windows Event Logs — Day 3 (Successful Logon + A Mistake I Noticed)

Today I focused on **logon events** in the Security log. I wanted to understand what a “successful logon” looks like and what details Windows records (logon type, process name, etc.).

---

## Event I reviewed (Security → Event ID 4624)

**Event ID:** 4624 (An account was successfully logged on)  
**Logon Type:** 5  
**Account:** SYSTEM (`NT AUTHORITY\SYSTEM`)  
**Process Name:** `C:\Windows\System32\services.exe`  
**Computer:** `win10`

---

## What I understood from this log

### This logon looks like a service logon (not a human login)
The logon type is **5**, and from my understanding so far, this is related to a **service** starting (not me typing a password at the login screen).

That matches the process name too:

- `services.exe` is involved
- User is `SYSTEM`
- No workstation name, no IP address

So this looks like normal Windows activity during boot or service startup.

---

## What stood out to me
- **Elevated Token: Yes**  
  That makes sense because SYSTEM has high privileges.

- **Source network address / port: blank**
  This helped me understand that not all logon events are “from a remote attacker.” Many are just internal system behavior.

---

## Important correction: I accidentally labeled something “failed logon”
In my notes, I wrote “Logon Failed,” but when I look closely at the event:

- It is still **Event ID 4624**
- It says **Audit Success**
- The description says **successfully logged on**

So that wasn’t a failed logon — it was another successful one.

If I want a real failed logon event, I should look for:

✅ **Event ID 4625** (failed login)

That mistake actually helped me learn something important:
I can’t trust my labels — I must trust the **Event ID + Keywords + description**.

---

## SOC takeaway (simple)
For SOC work, Event ID 4624 is useful, but it needs context:

- A 4624 for SYSTEM with Logon Type 5 looks normal
- A 4624 for a real user at weird times, repeated, or from a network source might be suspicious
- Without the IP field, this one doesn’t look like an external login attempt

This is exactly why analysts look for patterns instead of a single event.

---

## Next steps
- Generate an intentional failed login (wrong password) and capture **4625**
- Compare 4624 vs 4625 side by side
- Start noting logon types that appear most often in my system
