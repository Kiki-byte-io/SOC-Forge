# Initial Detection Scenarios (Candidate List)

**Sprint 1 · Milestone 1C** · Status: **Candidates, nothing implemented or tested** · 2026-11-11

These are **hypotheses**, not working detections. Each one gets a full detection specification in Phase 4 and positive/negative tests in Phase 5. All statuses start at `NOT_RUN`.

> **ATT&CK mappings are *candidates*.** A mapping is only kept if the validated detection actually observes that behaviour. Technique IDs below were checked on attack.mitre.org (v20.1) on 2026-11-11.
>
> **Built-in rules first.** Wazuh already ships rules for many Windows and Linux events. In Phase 4, for each scenario we first check what the built-in ruleset already alerts on. A custom rule is written only where it adds something specific, such as a threshold, a particular group, or a narrower pattern.

---

## Summary

| ID | Scenario | Endpoint | Key telemetry | Candidate ATT&CK | Status |
|---|---|---|---|---|---|
| SOCF-DET-001 | Repeated failed Windows logons | Windows | Security **4625** | T1110.001 | `NOT_RUN` |
| SOCF-DET-002 | New local account created | Windows | Security **4720** | T1136.001 | `NOT_RUN` |
| SOCF-DET-003 | Account added to a privileged local group | Windows | Security **4732** | T1098.007 | `NOT_RUN` |
| SOCF-DET-004 | Linux SSH authentication burst | Linux | SSH auth log entries | T1110.001 | `NOT_RUN` |
| SOCF-DET-005 | Successful login after a burst of failures | Windows / Linux | 4625 → **4624**; SSH fail → success | T1110 (TBD) | `NOT_RUN` (stretch) |
| SOCF-DET-006 | Narrow process-creation pattern | Windows | Sysmon **Event ID 1** | TBD with pattern | `NOT_RUN` |

---

## SOCF-DET-001 — Repeated failed Windows logons

| Field | Plan |
|---|---|
| **Objective** | Detect several failed logons to one account on one host within a short time window |
| **Hypothesis** | Password guessing produces a burst of 4625 events for the same `TargetUserName` |
| **Telemetry** | Windows Security channel, event **4625** "An account failed to log on" |
| **Prerequisite** | Audit policy **Audit Logon** (Failure) enabled |
| **Key fields** | `TargetUserName`, `LogonType`, `Status`, `SubStatus`, `IpAddress`, `WorkstationName` |
| **Positive test (idea)** | N wrong-password attempts for a disposable lab account within T seconds (N and T set in Phase 4) |
| **Negative test (idea)** | A single typo followed by a correct logon must **not** fire the burst rule |
| **Candidate ATT&CK** | T1110.001 Brute Force: Password Guessing (Credential Access) |

## SOCF-DET-002 — New local account created

| Field | Plan |
|---|---|
| **Objective** | Alert whenever a new local user account is created |
| **Hypothesis** | Persistence via a new local account produces event 4720 |
| **Telemetry** | Windows Security channel, event **4720** "A user account was created" |
| **Prerequisite** | Audit policy **Audit User Account Management** (Success) enabled |
| **Key fields** | `TargetUserName` (new account), `SubjectUserName` (who created it), `TargetSid` |
| **Positive test (idea)** | Create a disposable local account, then delete it afterwards |
| **Negative test (idea)** | Changing an *existing* account (a different event, 4738) must not fire this rule |
| **Candidate ATT&CK** | T1136.001 Create Account: Local Account (Persistence) |

## SOCF-DET-003 — Account added to a privileged local group

| Field | Plan |
|---|---|
| **Objective** | Detect a member being added to a high-value local group, e.g. built-in Administrators |
| **Hypothesis** | Privilege escalation or persistence via group membership produces event 4732 |
| **Telemetry** | Windows Security channel, event **4732** "A member was added to a security-enabled local group" |
| **Prerequisite** | Audit policy **Audit Security Group Management** (Success) enabled |
| **Key fields** | `TargetUserName` (group name), `MemberSid`, `SubjectUserName`. Note: for local groups `MemberName` is typically `-`, so the rule must rely on `MemberSid` |
| **Positive test (idea)** | Add the disposable lab account to local Administrators, then remove it |
| **Negative test (idea)** | Adding the account to a non-privileged lab group must **not** fire the privileged-group rule |
| **Candidate ATT&CK** | T1098.007 Account Manipulation: Additional Local or Domain Groups (Persistence, Privilege Escalation) |

## SOCF-DET-004 — Linux SSH authentication burst

| Field | Plan |
|---|---|
| **Objective** | Detect repeated failed SSH logins against the Linux endpoint |
| **Hypothesis** | Password guessing over SSH produces a burst of failed-authentication log lines |
| **Telemetry** | SSH daemon authentication messages. Exact source (`/var/log/auth.log` vs systemd journal) **[To verify in Phase 3]** |
| **Prerequisite** | SSH server running on `socf-lnx`; agent collecting the correct auth source |
| **Key fields** | Source IP, target user, result. Exact message format **[To verify from real logs]** |
| **Positive test (idea)** | N wrong-password SSH attempts from the Windows host or Wazuh VM (lab network only) |
| **Negative test (idea)** | One failed attempt then a successful login must not fire the burst rule |
| **Candidate ATT&CK** | T1110.001 Brute Force: Password Guessing (Credential Access) |

## SOCF-DET-005 — Successful login after a burst of failures (stretch goal)

| Field | Plan |
|---|---|
| **Objective** | Raise a *review lead* when failures for an account are followed by a success |
| **Hypothesis** | A successful guess appears as failures (4625) followed by success (event **4624** "An account was successfully logged on"), or SSH failures followed by an accepted login |
| **Telemetry** | Windows 4625 + 4624, or Linux SSH fail + success |
| **Prerequisite** | DET-001 / DET-004 working; reliable shared fields (account, source) and consistent timestamps |
| **Caveat** | Only kept if correlation is **reliable** in testing. Otherwise documented as a limitation. |
| **Negative test (idea)** | Failures without a later success, and a success without prior failures, must not fire |
| **Candidate ATT&CK** | T1110 Brute Force. Exact sub-technique decided after validation |

## SOCF-DET-006 — Narrow process-creation pattern (Sysmon)

| Field | Plan |
|---|---|
| **Objective** | Detect one narrowly defined, benign-but-suspicious-looking process execution |
| **Hypothesis** | A specific executable/command-line pattern is visible in Sysmon process-creation events |
| **Telemetry** | Sysmon **Event ID 1** "Process creation" in `Microsoft-Windows-Sysmon/Operational` (logs the full command line and parent process) |
| **Prerequisite** | Sysmon installed with a **focused** config that includes `ProcessCreate` |
| **Key fields** | `Image`, `CommandLine`, `ParentImage`, `User`, hashes. Exact names to confirm from real events |
| **Pattern** | **To be chosen in Phase 4.** It must be safe, repeatable and narrow enough to keep false positives low |
| **Negative test (idea)** | The same program run in an expected, benign way must not fire |
| **Candidate ATT&CK** | Decided once the pattern is chosen |

---

## Sources

- Microsoft Learn, "4625(F) An account failed to log on" — https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/auditing/event-4625 (accessed 2026-11-11)
- Microsoft Learn, "4720(S) A user account was created" — https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/auditing/event-4720 (accessed 2026-11-11)
- Microsoft Learn, "4732(S) A member was added to a security-enabled local group" — https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/auditing/event-4732 (accessed 2026-11-11)
- Microsoft Learn, "Sysmon" (Event ID 1, channel, supported OS) — https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon (accessed 2026-11-11)
- MITRE ATT&CK, T1110.001 — https://attack.mitre.org/techniques/T1110/001/ (accessed 2026-11-11)
- MITRE ATT&CK, T1136.001 — https://attack.mitre.org/techniques/T1136/001/ (accessed 2026-11-11)
- MITRE ATT&CK, T1098.007 — https://attack.mitre.org/techniques/T1098/007/ (accessed 2026-11-11)
