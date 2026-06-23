# Investigation

## Initial Assessment

Based on the user's history of startup BSODs, repeated Windows reinstalls, and recovery environment boot loops, the initial hypothesis was storage corruption or storage device failure.

The investigation focused on identifying evidence of file system corruption, operating system corruption, memory instability, or hardware failure.

---

## Event Viewer Analysis

Review of Event Viewer identified multiple Kernel-Boot startup failures:

| Date | Event |
|--------|--------|
| 06/19/2026 | Error Status 0xC0000001 |
| 06/16/2026 | Error Status 0xC000A002 |
| 06/15/2026 | Error Status 0xC00000D4 |
| 06/11/2026 | Error Status 0xC000A002 |

These events indicated recurring startup failures affecting the Windows boot process.

---

## NTFS Corruption Discovery

Shortly after one of the Kernel-Boot failures, Event Viewer recorded an NTFS corruption event:

> A corruption was discovered in the file system structure on volume C:.

The event specifically identified corruption within:

```text
\Windows\SystemApps
```

and referenced a corrupted index allocation structure:

```text
:$I30:INDEX_ALLOCATION
```

This strengthened the theory that file system corruption was contributing to system instability.

---

## System Integrity Testing

### SFC Scan

Command:

```powershell
sfc /scannow
```

Initial Result:

No integrity violations detected.

---

## CHKDSK Repair

Attempts to run CHKDSK from Windows resulted in access denied errors despite administrative privileges.

The system was booted into the Windows Recovery Environment where CHKDSK was successfully executed.

Result:

- File system corrections were performed
- Corruption was repaired

A follow-up SFC scan later identified and repaired integrity violations.

---

## Additional BSOD Encountered

On June 22, 2026, the system generated a BSOD during normal operation while loading applications.

Stop Code:

```text
IRQL_NOT_LESS_OR_EQUAL
```

Event Viewer recorded:

```text
BugCheck 0x0000000A
```

This expanded the investigation to include:

- Memory instability
- Driver-related issues
- Operating system corruption

---

## Storage Health Verification

Samsung Magician was used to verify SSD health.

Results:

- Drive health reported as Good
- No warnings reported
- No evidence of SSD hardware failure identified

This reduced confidence in the original storage-failure hypothesis.

---

## Memory Investigation

Windows Memory Diagnostic was executed.

Results:

- Hardware problems detected during testing
- Event ID 1102 generated

Because both the stop code and diagnostic results suggested memory instability, memory became a primary area of investigation.

---

## Windows Servicing Investigation

Additional operating system repair attempts revealed further issues.

Command:

```powershell
DISM /Online /Cleanup-Image /RestoreHealth
```

Result:

```text
The system cannot find the file specified
```

Investigation of DISM logs and CBS logs revealed:

- Error 0x80070002
- Missing package references
- Invalid package entries
- Missing Windows system files

Examples included:

```text
Package for KB3025096
```

and missing Windows runtime implementation files.

These findings suggested corruption within the Windows servicing infrastructure rather than a simple file system issue.

---

## Investigation Pivot

At this stage, multiple indicators pointed toward Windows operating system corruption:

- NTFS corruption events
- Failed servicing operations
- Missing package references
- CBS log errors
- Historical success after Windows reinstalls

Rather than continue investigating individual corruption artifacts, an in-place Windows upgrade was selected as the most efficient remediation strategy.
