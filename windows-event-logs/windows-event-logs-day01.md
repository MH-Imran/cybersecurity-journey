# Windows Event Logs — Day 1

## Why I started working with Windows logs
After completing the TryHackMe Pre-Security path, I wanted to move beyond theory and actually look at how Windows records activity.

Since most SOC alerts are based on Windows logs, this felt like the right next step.  
The goal of this first session was simple: **get comfortable reading logs**, not mastering everything at once.

---

## Logs I focused on today
I only worked with two log types to keep things manageable:

- **Security log**
- **System log**

I ignored the rest for now.

---

## Security log observations
This log immediately stood out because of how much activity it records.

Things I looked at:
- User logins and logouts
- Failed login attempts
- Process-related events

### Event ID 4624 — Successful logon
- Shows when a user logs in successfully
- Includes the account name and logon type
- Helped me understand how normal logins look

### Event ID 4625 — Failed logon
- Shows failed authentication attempts
- Useful for spotting brute-force or password guessing
- Even normal systems generate these occasionally

Seeing both side by side helped me understand the difference between *normal noise* and something that might need investigation.

---

## System log observations
The System log felt less about users and more about the operating system itself.

Things I noticed:
- Service start and stop events
- System-level messages
- Background activity that users usually don’t see

### Service-related events
- Services start automatically when Windows boots
- Some events are expected and happen regularly
- Unexpected service behavior could be interesting from a SOC perspective

---

## What felt confusing at first
- There are a lot of events, even on a clean system
- Many events look similar at first glance
- Not every event is important

Instead of trying to understand everything, I focused on:
- Event ID
- Time
- Account name (if any)
- Short description

That made things much clearer.

---

## Why this matters for SOC work
This session helped me understand why SOC analysts rely so heavily on logs:
- Logs tell the story of what happened
- Alerts usually point back to specific event IDs
- Understanding “normal” behavior is necessary before spotting attacks

It also made SIEM alerts feel less abstract, since I now know where the data comes from.

---

## Next steps
- Practice filtering logs more efficiently
- Use PowerShell to read logs instead of only Event Viewer
- Connect log events to MITRE ATT&CK techniques
- Combine log analysis with network traffic observations

---

*Status: First hands-on session with Windows Event Logs completed*  
*Learning is ongoing, and these notes will evolve with practice.*
