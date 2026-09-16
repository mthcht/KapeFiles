# KAPE Targets Skills Guide

This document describes the skills and knowledge required to understand, analyze, and create KAPE Target files (`.tkape`).

## Overview

KAPE Targets are YAML-like configuration files that define what artifacts to collect from a system. Each Target specifies one or more artifacts with their file paths, patterns, and collection rules.

## File Format and Structure

Target files use a YAML-like format with the following structure:

```
Description: <string>
Author: <string>
Version: <float>
Id: <GUID>
RecreateDirectories: <boolean>
Targets:
    - Name: <string>
      Category: <string>
      Path: <string>
      [Optional fields]
    - [Additional targets...]

# Documentation
# <Links and references>
```

## Required Fields

### Top-Level Fields

| Field | Type | Description | Example |
|-------|------|-------------|---------|
| `Description` | String | Name of application/artifact; visible in gKape | `Windows Defender Data` |
| `Author` | String | Creator of the Target | `Drew Ervin` |
| `Version` | Float | Version number; increment on revisions | `1.0` |
| `Id` | GUID | Unique identifier for this Target | `061aa929-292b-4d7f-a4af-a3fe2673a3e5` |
| `RecreateDirectories` | Boolean | Preserve directory structure in output | `true` |
| `Targets` | Array | List of artifact definitions | See below |

### Target Item Fields

| Field | Type | Required | Description | Example |
|-------|------|----------|-------------|---------|
| `Name` | String | Yes | Human-readable artifact name | `Windows Defender Logs` |
| `Category` | String | Yes | Functional category; groups related artifacts | `Antivirus`, `EventLogs` |
| `Path` | String | Yes | File system path with wildcards/variables | `C:\ProgramData\Microsoft\Windows Defender\*` |
| `Recursive` | Boolean | No | Search subdirectories (default: false) | `true` |
| `FileMask` | String | No | File name pattern (glob or regex) | `*.evtx`, `log*.txt`, `regex:(2019\|DSC).+\.log` |
| `AlwaysAddToQueue` | Boolean | No | Defer collection until other targets finish; use for system-locked files | `true` |
| `SaveAsFileName` | String | No | Rename collected file to this name | `output.csv` |
| `MinSize` | Integer | No | Minimum file size in bytes | `1000` |
| `MaxSize` | Integer | No | Maximum file size in bytes | `10000` |
| `Comment` | String | No | Brief description of what is collected | `Windows Defender detection history` |

## Path Variables and Wildcards

### Variables

KAPE supports the following variables in paths:

| Variable | Description | Example |
|----------|-------------|---------|
| `%user%` | Current user profile folder | `C:\Users\%user%\AppData\Local\` |
| `%userprofile%` | User profile root | `C:\Users\%userprofile%\` |
| `%allusersprofile%` | All Users profile folder | `%allusersprofile%\Microsoft\` |
| `%systemroot%` | Windows system root | `%systemroot%\System32\` |
| `%programfiles%` | Program Files folder | `%programfiles%\App\` |
| `%programfiles(x86)%` | Program Files (x86) folder | `%programfiles(x86)%\App\` |

### Wildcards

| Wildcard | Meaning | Example |
|----------|---------|---------|
| `*` | Any characters (folder or file) | `C:\Users\*\AppData\*` |
| `?` | Single character | `file?.txt` |

### FileMask Patterns

FileMask supports glob patterns and regex:

- **Glob patterns**: `*.txt`, `log*.log`, `file?.txt`
- **Regex patterns**: `regex:(2019|DSC|Log).+\.(jpg|txt)`
- **Common extensions**: `SOFTWARE.logX` matches `SOFTWARE.log1`, `SOFTWARE.log2`, etc.

## Understanding Artifact Collection

To determine what artifacts a Target collects:

1. **Read the Description**: Summarizes the application/artifact
2. **Check the Category**: Indicates the type of artifact (Antivirus, EventLogs, Registry, etc.)
3. **Examine the Path**: Shows where artifacts are located on the system
4. **Review the Comment**: Provides context on forensic significance
5. **Check Documentation links**: References provide deeper context

## Creating New Targets

### Step 1: Plan Your Artifact

- Identify what you want to collect (application data, logs, configuration files)
- Determine where it's stored on the system
- Research alternative storage locations (user-specific, system-wide, etc.)

### Step 2: Test Paths

Create a test Target with your planned paths on a sample system:

- Verify paths exist and contain expected files
- Test wildcard patterns to ensure correct matching
- Test on systems with different folder structures (multiple users, different OS versions)
- Verify recursive settings work as expected

### Step 3: Write the Target File

```yaml
Description: <Application Name> <Artifact Type>
Author: Your Name
Version: 1.0
Id: <Generate GUID via: kape.exe --guid>
RecreateDirectories: true
Targets:
    -
        Name: <Artifact Name>
        Category: <Category>
        Path: <Verified Path>
        Recursive: true
        Comment: "Brief description of forensic value"

# Documentation
# <Add URLs to documentation, blog posts, etc. or write "N/A">
```

### Step 4: Validation

- Verify YAML syntax is valid
- Test on sample systems before submitting PR
- Run the KAPE Pull Request template checks
- Ensure blank line after last comment before end of file

## Common Target Patterns

### Collect All Files in a Folder

```yaml
Path: C:\Users\%user%\AppData\Local\Application\
Recursive: true
```

### Collect Specific File Extensions

```yaml
Path: C:\Users\%user%\Documents\
FileMask: "*.docx"
Recursive: true
```

### Collect Files Matching a Pattern

```yaml
Path: C:\Logs\
FileMask: "log*.txt"
```

### Collect Files with Size Constraints

```yaml
Path: C:\Temp\
MinSize: 1024
MaxSize: 5242880  # 5 MB
```

### Collect System-Locked Files

```yaml
AlwaysAddToQueue: true  # Collect after other targets finish
```

## Analyzing Existing Targets

### Finding Targets by Artifact Type

Search for Targets in the appropriate category:
- `Targets/Antivirus/` - Antivirus software data
- `Targets/Apps/` - Third-party applications
- `Targets/Browsers/` - Web browsers
- `Targets/Windows/` - Native Windows artifacts
- `Targets/Logs/` - Log files
- `Targets/P2P/` - Peer-to-peer applications

### Extracting Information from Target Files

1. **List all artifacts**: Parse the YAML and iterate through `Targets` array
2. **Find artifacts in a location**: Search for Targets with matching `Path` values
3. **Find artifacts by category**: Filter by `Category` field
4. **Understand collection scope**: Combine `Path`, `FileMask`, and `Recursive` to determine scope

### Example: What artifacts does Windows Defender target collect?

```yaml
# From Targets/Antivirus/WindowsDefender.tkape
Targets:
    - Name: Windows Defender Logs
      Path: C:\ProgramData\Microsoft\Microsoft AntiMalware\Support\
      Recursive: true
    - Name: Windows Defender Logs
      Path: C:\ProgramData\Microsoft\Windows Defender\Support\
      Recursive: true
    - Name: Windows Defender Event Logs
      Path: C:\Windows\System32\winevt\Logs\
      FileMask: Microsoft-Windows-Windows Defender*.evtx
```

This target collects:
1. Support logs from Microsoft AntiMalware folder
2. Support logs from Windows Defender folder  
3. Windows event logs matching the "Microsoft-Windows-Windows Defender" pattern

## Compound Targets

Compound Targets reference other Targets rather than defining paths directly:

```yaml
Description: Multiple Antivirus Applications
Author: Someone
Version: 1.0
Id: <GUID>
RecreateDirectories: true
Targets:
    -
        Name: Windows Defender
        Category: Antivirus
        Path: WindowsDefender.tkape  # References filename
    -
        Name: McAfee
        Category: Antivirus
        Path: McAfee.tkape
```

Use Compound Targets to group related targets for convenience.

## Best Practices

1. **Test on multiple systems**: Verify paths work on different Windows versions and configurations
2. **Use variables**: Prefer `%user%` and system variables over hardcoded paths
3. **Provide meaningful comments**: Help other analysts understand what's being collected
4. **Include documentation**: Link to relevant blog posts, vendor documentation, or research
5. **Increment version numbers**: Track Target revisions
6. **Use appropriate categories**: Maintain consistency with existing categorization
7. **Consider user-specific paths**: Use `%user%` to capture per-user artifacts
8. **Handle locked files**: Use `AlwaysAddToQueue: true` for system files that may be in use

## Resources

- [Full KAPE Documentation](https://ericzimmerman.github.io/KapeDocs/#!Pages\2.1-Targets.md)
- [KAPE GitHub Repository](https://github.com/EricZimmerman/KapeFiles)
- [Windows Artifact Locations](https://github.com/EricZimmerman/KapeDocs)

## Tools and Commands

### Generate a GUID for New Targets

```bash
kape.exe --guid
```

### Validate Target Files

Test your Target on a system before submitting:

1. Copy the `.tkape` file to the appropriate `Targets/` subfolder
2. Open in gKape or use KAPE CLI
3. Verify artifacts are collected as expected
4. Check for error messages

## Common Issues and Solutions

| Issue | Solution |
|-------|----------|
| Files not being collected | Verify path exists; test path variables on target system; check FileMask syntax |
| Too many files collected | Add FileMask to narrow results; add MaxSize limit; check Recursive setting |
| Folder structure not preserved | Ensure `RecreateDirectories: true` at top level |
| System reports "file in use" | Add `AlwaysAddToQueue: true` to defer collection |
| YAML syntax errors | Validate indentation; check for missing colons; test in YAML validator |
