# Windows Event Logs (Practice Notes)

This folder is where I’m saving my Windows event log practice notes as I learn SOC-style investigation.

I’m not trying to write perfect documentation here. The goal is to keep honest notes from real log review sessions and slowly get better at answering questions like:
- Who did what?
- When did it happen?
- What process started it?
- Does it look normal or suspicious?

Most of my focus right now is on the **Security** log, and I’m learning how to read common events with context instead of memorizing random IDs.

---

## Files in this folder

- **windows-event-logs-day1.md**  
  First session with Event Viewer. Focused on getting comfortable with Security/System logs and spotting basic logon activity.

- **windows-event-logs-day2.md**  
  Focused on **process creation (Event ID 4688)** and what details Windows records (process name, creator, SYSTEM context, etc.).

- **windows-event-logs-day3.md**  
  Focused on **successful logon (Event ID 4624)**. Also fixed a mistake in my notes (I labeled a success event as “failed” at first), which helped me learn to trust the Event ID and keywords.

---

## What I’m focusing on (for now)
- Event IDs: **4624, 4625, 4688** (and basic service-related events)
- “Normal vs suspicious” patterns
- Reading logs using **Event Viewer** first (PowerShell comes next)

---

## What’s next
As I continue, I plan to:
- Generate my own test events (failed logins, new process runs) and capture them in logs
- Practice PowerShell filtering for faster log review
- Connect Windows events to **MITRE ATT&CK** techniques later, once my basics are solid

Learning in progress.
