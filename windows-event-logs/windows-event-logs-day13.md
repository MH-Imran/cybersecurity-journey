# windows-event-logs - Day 13

Today I practiced a simple SOC-style story chain in **Windows Security logs**:
a failed login → a successful login → a process created later.

I’m keeping it small on purpose: **one story, clean fields, short timeline**.

---

## The logs I collected

### 1) Event ID 4625 — Failed login
- Time (UTC): **11:07:31**
- User: **labuser**
- Host: **win10**
- Logon type: **2 (Interactive)**
- Source IP: **127.0.0.1**
- Status/SubStatus: **0xC000006D / 0xC000006A**
- Process: **svchost.exe**

My quick read:
- Logon Type 2 = someone tried to log in **interactively** (keyboard/console/RDP-like context).
- SubStatus **0xC000006A** usually means **bad password** (most likely a typo in a lab).

---

### 2) Event ID 4624 — Successful login
- Time (UTC): **11:08:08**
- User: **labuser**
- Host: **win10**
- Logon type: **2 (Interactive)**
- Source IP: **127.0.0.1**
- Status: **Success**
- Process: **svchost.exe**
- Elevation: **No**

My quick read:
- Same user + same host + same source IP.
- Happened about **37 seconds** after the failure → very consistent with “typed wrong password once, then correct.”

---

### 3) Event ID 4688 — Process creation
- Time (UTC): **15:16:32** *(this time doesn’t line up with the login time — I need to validate)*
- User: **labuser**
- Host: **win10**
- Process: **cmd.exe**
- Parent: **explorer.exe**
- Elevation: **Yes (Admin)**

My quick read:
- Parent **explorer.exe** usually means the user launched it normally (Start menu / Run / click).
- **Admin elevation** matters. If this were a real incident, an elevated shell right after a suspicious login would be a red flag.
- But here there’s a **big time gap**, so I’m not assuming it’s connected until I confirm context.

---

## Timeline (simple)
- **11:07:31** — 4625: `labuser` failed an interactive login on `win10` from `127.0.0.1`.
- **11:08:08** — 4624: `labuser` successfully logged in interactively from the same source.
- **15:16:32** — 4688: `cmd.exe` created under `explorer.exe` with **Admin** elevation.
- Note to self: the **4688 time gap** is big, so I should check surrounding events before linking it to the login.

---

## Mini SOC ticket (practice)

**Title:** Failed interactive login followed by success; later elevated CMD observed  
**Time window:** 11:07–11:09 UTC (logon) + 15:16 UTC (process)  
**Host/User:** win10 / labuser

**What happened (plain):**  
One failed interactive login was followed shortly by a successful interactive login for the same user from the same source. Later, an elevated command prompt was launched by the same user, but the timestamp gap needs validation.

**Evidence:**
- 4625 @ 11:07:31 — interactive logon failed (bad password pattern)
- 4624 @ 11:08:08 — interactive logon success (same user/source)
- 4688 @ 15:16:32 — `cmd.exe` created (parent `explorer.exe`, elevated)

**Assessment:** Probably benign in a lab (password typo), **but** elevated shell + time gap means “verify before concluding.”  
**Next steps (what I’d check in a real SOC):**
- Confirm whether 15:16 is same session/day/timezone
- Look for activity around 15:16 (other 4688, new services/tasks, downloads, outbound connections)
- If suspicious: reset creds, review other hosts for the same login pattern

---

## What I’m taking from this
- A 4625 → 4624 sequence can be normal, but it’s still a useful pattern to recognize fast.
- 4688 becomes more interesting when it’s **elevated** and near a risky login.
- **Time alignment matters**. If my timestamps don’t match, I pause and validate instead of forcing a story.
