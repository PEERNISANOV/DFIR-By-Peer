# Windows Prefetch as a DFIR Artifact

> How does Windows "remember" which programs were executed on a system?

One of the most useful Windows artifacts for investigating program execution is:

**Prefetch**

## Overview

Prefetch was not originally designed for digital forensics.

It was created to improve application startup performance.

When an application is executed, Windows can monitor which files, libraries, and other resources are accessed during the application's startup process. This information can then be used to improve the application's startup time during future executions.

However, like many Windows artifacts created for performance or usability purposes, Prefetch can also become a valuable source of forensic evidence.

Prefetch files are typically located under:

```text
C:\Windows\Prefetch\
```

The files use the `.pf` extension and commonly follow a naming structure similar to:

```text
EXECUTABLE.EXE-HASH.pf   (Note: The "hash" is an 8-character code derived from the full directory path where the executable was located during launch).
```

For example:

```text
MIMIKATZ.EXE-XXXXXXXX.pf
```

## What Can Prefetch Tell Us?

A Prefetch file may provide information such as:

- The name of the executed application
- The application's execution count
- The most recent execution time
- Volume information
- Files and directories referenced during application startup
- DLLs and other resources associated with the execution

This information can be valuable when reconstructing activity on a Windows system.

During an investigation, Prefetch can help answer questions such as:

- Was a suspicious executable actually executed?
- Is there evidence of a program that has since been deleted?
- When was the program executed?
- Can the execution be correlated with the forensic timeline?
- Were unusual files or directories accessed during application startup?
- Does the execution appear consistent with legitimate user activity?

## Analyzing Prefetch with PECmd

One tool that can be used to analyze Windows Prefetch files is **PECmd**, created by Eric Zimmerman.

A single Prefetch file can be analyzed using:

```powershell
PECmd.exe -f "C:\Windows\Prefetch\MIMIKATZ.EXE-XXXXXXXX.pf"
```

For larger investigations, an entire Prefetch directory can also be processed. The results can be exported to CSV file for easier filtering and timeline analysis:

```powershell
PECmd.exe -d "C:\Windows\Prefetch" --csv "C:\Temp\Prefetch" --csvf Prefetch.csv
```

PECmd parses the Prefetch structure and presents useful information such as execution timestamps, run count, volume information, and referenced files.

## Practical Example

In the simulation below, `Mimikatz.exe` was executed manually on a Windows system.

After execution, Windows created a corresponding Prefetch file under:

```text
C:\Windows\Prefetch\
```

The resulting file appeared similar to:

```text
MIMIKATZ.EXE-XXXXXXXX.pf
```

<!-- Add screenshot of Mimikatz execution here -->

The Prefetch file was then analyzed using PECmd:

```powershell
PECmd.exe -f "C:\Windows\Prefetch\MIMIKATZ.EXE-XXXXXXXX.pf"
```

<!-- Add screenshot of PECmd analysis here -->

From the parsed Prefetch data, an investigator can identify information associated with the execution, including:

```text
Executable Name
Run Count
Last Execution Time
Volume Information
Referenced Files and Directories
```

This provides evidence that Windows recorded the execution of the executable and allows the activity to be correlated with other forensic artifacts.

## Investigative Value

Prefetch can be particularly useful when:

- A suspicious executable is no longer present on disk
- Investigators need evidence that a specific application was executed
- Execution timestamps need to be added to a forensic timeline
- Malware execution is suspected
- An executable was launched from an unusual location
- Other execution artifacts are missing or incomplete
- Investigators need additional evidence to support activity found in other artifacts

For example, discovering:

```text
MIMIKATZ.EXE-XXXXXXXX.pf
```

may be highly relevant during an investigation.

However, the existence of the Prefetch file should not automatically be treated as proof of malicious activity.

The executable may have been used legitimately by an administrator, security researcher, penetration tester, or incident responder.

Context matters.

Prefetch should therefore be correlated with other evidence such as:

- Windows Event Logs
- Amcache
- Shimcache / AppCompatCache
- Program Compatibility Assistant (PCA)
- UserAssist
- SRUM
- Registry artifacts
- File system timestamps
- EDR telemetry
- Authentication activity

## Limitations

Prefetch is a valuable artifact, but it has several important limitations.

- Prefetch may be disabled through system configuration.
- Prefetch files may have been deleted.
- Older Prefetch entries may no longer be available.
- Behavior can vary depending on the Windows version and system configuration.
- Windows Server systems most likely have application prefetching disabled depending on the version and configuration.
- The absence of a Prefetch file does **not** prove that an executable was never executed.


## Key Takeaway

Prefetch is a good example of how artifacts created for system performance can become valuable sources of forensic evidence.

It can help investigators determine:

> **What ran, when it ran, how often it ran, and what resources were associated with the execution.**

On its own, Prefetch provides only part of the picture.

Combined with other Windows artifacts, it can become an important piece of the forensic timeline.

---
