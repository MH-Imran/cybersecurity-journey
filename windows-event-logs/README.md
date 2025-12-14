# Windows Event Logs

This folder contains my hands-on notes and observations while learning how to analyze Windows event logs from a SOC analyst’s perspective.

After completing the TryHackMe Pre-Security path, I started focusing on Windows logs because most real SOC alerts are built on Windows telemetry. The goal here is to understand how activity is recorded and how analysts use logs to investigate suspicious behavior.

## What I am focusing on
At this stage, I’m keeping things simple and practical. My focus includes:
- Security and System logs
- Authentication events (successful and failed logins)
- Process and service-related events
- Understanding what “normal” activity looks like

I’m not trying to memorize every event ID — I’m learning how to read logs with context.

## How these notes are used
Each file usually represents a single practice session. In the notes, I record:
- Which logs I looked at
- Which event IDs stood out
- What felt normal vs unusual
- Why a SOC analyst might care about those events

The explanations are written in my own words and may change as my understanding improves.

## Why this matters for SOC work
Windows event logs are one of the main data sources used in SIEM platforms.  
Learning where alerts come from and what they actually mean is an important step toward effective alert triage and incident investigation.

## What’s next
As I progress, I plan to:
- Use PowerShell more for log analysis
- Connect Windows events to MITRE ATT&CK techniques
- Correlate logs with network traffic
- Apply this knowledge in SOC-style labs

Learning in progress.
