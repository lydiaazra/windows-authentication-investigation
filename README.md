# Windows Authentication Investigation

A blue-team log analysis exercise reviewing Windows Security event logs to identify authentication patterns, investigate failed logon attempts, and distinguish normal activity from events that require escalation.

## Objective

Review Windows Security event logs to identify authentication patterns, investigate failed logon attempts, and distinguish normal activity from events requiring escalation.

## Report Details

- **Analyst:** Lydia Azra
- **Host Analysed:** AZRA-LAPTOP / Windows 11
- **Investigation Period:** 28 August 2026 – 4 September 2026
- **Tools Used:** Windows Event Viewer, Windows Security Event Log

## Event IDs Investigated

| Event ID | Event Name | What It Means / What to Look For |
|---|---|---|
| 4624 | Successful Logon | Logon Type (2=interactive, 3=network, 10=remote), username, source IP, timestamp. |
| 4625 | Failed Logon | Username, failure reason (wrong password vs unknown user), source IP. Repeated failures = brute-force indicator. |
| 4740 | Account Lockout | Which account was locked, from which machine. Often follows repeated 4625 events. |
| 4648 | Logon with Explicit Credentials | User authenticated using different credentials (e.g. `runas`). Can indicate lateral movement or privilege escalation. |

## Methodology

1. Filtered Windows Security logs (via Event Viewer) for the four Event IDs above.
2. Recorded username, source machine/IP, logon type, and timestamp for each relevant event.
3. Cross-referenced failed logons (4625) and lockouts (4740) to look for brute-force patterns.
4. Reviewed successful logons (4624), flagging Logon Type 10 (RDP) and Type 3 (network) as higher risk than Type 2 (interactive) and Type 7 (unlock).
5. Reviewed explicit-credential events (4648) to separate normal Windows internals from real user lateral movement.
6. Built a chronological investigation timeline to tell the story of the investigation period.

## Findings

**Failed logon activity:** Two failed logon attempts (4625) were identified, four seconds apart, both wrong-password failures (`0xC000006A`), immediately followed by a successful logon 14 seconds later — consistent with a mistyped password rather than a brute-force attempt.

**Account lockout activity:** No account lockout events (4740) were identified during the investigation period.

**Notable successful logons:** All successful logons (4624) were Logon Type 2 (interactive) or Type 7 (workstation unlock), originating exclusively from the local machine. No RDP (Type 10) or anomalous network (Type 3) logons were detected.

**Explicit credential use:** All 118 events (4648) were attributed to normal Windows internal processes (Font Driver Host / `UMFD-9`), not user-initiated lateral movement.

## Assessment

| Category | Classification |
|---|---|
| Failed logons (4625) | Normal — 2 failed attempts, correct password on next try, no brute-force indicators |
| Account lockouts (4740) | N/A — none detected |
| Successful logons (4624) | Normal — local machine only, expected logon types, within active hours |
| Explicit credential use (4648) | Normal — all attributed to Windows internal processes |

**Conclusion:** No activity requiring escalation was identified. The only anomalous events were two consecutive failed logon attempts immediately followed by a successful authentication, consistent with a mistyped password rather than unauthorised access.

## Key Learnings

- Windows Security logs require filtering by Event ID to be useful — raw logs are too large to review manually.
- A single 4625 event is not an alert; repeated 4625 events for the same user within seconds is the signal worth escalating.
- Logon Type matters: Type 10 (RDP) and Type 3 (network) carry more risk than Type 2 (interactive) and Type 7 (unlock).
- 4648 events from `SYSTEM` using UMFD (Font Driver Host) accounts are normal Windows internals — the real signal is a 4648 event from a real user account authenticating to a different machine.

## Files

- `windows-authentication-report.pdf` — the full investigation report, including Event Viewer screenshots and the investigation timeline.
