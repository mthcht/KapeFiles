# KAPE Modules Skills Guide

This document describes the skills and knowledge required to understand, analyze, and create KAPE Module files (`.mkape`).

## Overview

KAPE Modules are YAML-like configuration files that define parsing and processing steps for collected artifacts. Each Module specifies one or more processors (executables and command-line arguments) that transform raw artifacts into processed outputs in various formats (CSV, JSON, XML, etc.).

## File Format and Structure

Module files use a YAML-like format with the following structure:

```
Description: <string>
Category: <string>
Author: <string>
Version: <float>
Id: <GUID>
ExportFormat: <format>
[Optional fields]
Processors:
    - Executable: <string>
      CommandLine: <string>
      ExportFormat: <format>
    - [Additional processors...]

# Documentation
# <Links and references>
```

## Required Fields

### Top-Level Fields

| Field | Type | Description | Example |
|-------|------|-------------|---------|
| `Description` | String | Name/purpose of the Module | `PowerShell Event Log Dump` |
| `Category` | String | Functional category; folder name for output | `Persistence`, `Logs`, `Registry` |
| `Author` | String | Creator of the Module | `Max Zabuty` |
| `Version` | Float | Version number; increment on revisions | `1.0` |
| `Id` | GUID | Unique identifier for this Module | `e3444190-b58e-4fe7-8048-e0bb1f40b3c7` |
| `ExportFormat` | String | Default format when user doesn't specify | `csv` |
| `Processors` | Array | List of processing commands | See below |

### Optional Top-Level Fields

| Field | Type | Description | Example |
|-------|------|-------------|---------|
| `BinaryUrl` | String | URL to download binary if not present | `https://github.com/user/tool/releases/` |
| `WaitTimeout` | Integer | Minutes to wait for Module to finish | `0` (no timeout) |
| `FileMask` | String | Filter input files; regex or glob | `regex:(2019\|DSC).+\.(jpg\|txt)` |

### Processor Item Fields

| Field | Type | Required | Description | Example |
|-------|------|----------|-------------|---------|
| `Executable` | String | Yes | Binary name/path to execute | `EvtxECmd.exe`, `Folder\tool.exe` |
| `CommandLine` | String | Yes | Arguments; includes format placeholders | `-d %sourceDirectory% --csv %destinationDirectory%` |
| `ExportFormat` | String | Yes | Output format this processor generates | `csv`, `json`, `xml` |

## Understanding Export Formats

Modules support multiple output formats. Each processor can generate one format:

| Format | Extension | Use Case |
|--------|-----------|----------|
| `csv` | `.csv` | Spreadsheet applications, easy pivot tables |
| `json` | `.json` | Structured data, programmatic access |
| `xml` | `.xml` | Enterprise systems, XSLT transformations |
| `txt` | `.txt` | Human-readable reports |

## Path Variables and Placeholders

Modules use the following placeholders in CommandLine arguments:

| Placeholder | Description | Expands To |
|------------|-------------|-----------|
| `%sourceDirectory%` | Input folder containing artifacts to process | Directory of collected artifacts from Target |
| `%destinationDirectory%` | Output folder for processed results | User-specified Module output directory |

### Example

```
CommandLine: -d %sourceDirectory% --csv %destinationDirectory%
```

When executed:
- `-d` flag points to the folder containing collected artifacts
- `--csv` flag tells the tool to output CSV format
- Results are written to `%destinationDirectory%`

## Understanding Artifact Parsing

To determine what artifacts a Module parses:

1. **Read the Description**: Summarizes what the Module processes
2. **Check the Category**: Indicates the output data type
3. **Examine the Executable**: Identifies the parsing tool
4. **Review CommandLine**: Shows which input artifacts/options are used
5. **Check ExportFormats**: Determines available output formats
6. **Review Documentation links**: References provide deeper context

### Example: What does a Module parse?

```yaml
# From Modules/Windows/PowerShell_AccessibilityFeatures.mkape
Description: Checks for Debugger registry value and file integrity
Executable: C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
CommandLine: -Command "$features = @(...); ..."
ExportFormat: csv
```

This Module:
1. Executes a PowerShell script
2. Reads Windows registry for accessibility features
3. Checks file integrity using `sfc`
4. Outputs results as CSV

## Processor Types

### Binary Executables

Most Modules use pre-compiled tools:

```yaml
Processors:
    -
        Executable: EvtxECmd.exe
        CommandLine: -d %sourceDirectory% --csv %destinationDirectory%
        ExportFormat: csv
```

Binary executables must be placed in `.\KAPE\Modules\bin\` with subdirectory support:

- Root binary: `Executable: tool.exe`
- In subfolder: `Executable: folder\tool.exe` (path relative to `bin\`)

### PowerShell Scripts

Modules can execute PowerShell commands:

```yaml
Processors:
    -
        Executable: C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
        CommandLine: -Command "script code here"
        ExportFormat: csv
```

### Python Scripts

Some Modules execute Python:

```yaml
Processors:
    -
        Executable: python.exe
        CommandLine: script.py %sourceDirectory% %destinationDirectory%
        ExportFormat: json
```

## Creating New Modules

### Step 1: Identify the Parser

- Determine what artifact you want to parse
- Find or develop a tool that parses this artifact
- Ensure the tool is available and can be distributed
- Verify the tool supports your desired output format(s)

### Step 2: Determine Command Syntax

Test the tool locally with actual artifacts:

```bash
tool.exe -d input_folder --csv output_folder
```

Document:
- Input parameter(s)
- Output format options
- Output location parameter(s)
- Any required file types/extensions

### Step 3: Plan Output Formats

Identify which export formats the tool supports:
- CSV: Spreadsheet analysis
- JSON: Programmatic processing
- XML: Enterprise integration
- TXT: Human-readable reports

Create a Processor for each format:

```yaml
Processors:
    -
        Executable: tool.exe
        CommandLine: -d %sourceDirectory% --csv %destinationDirectory%
        ExportFormat: csv
    -
        Executable: tool.exe
        CommandLine: -d %sourceDirectory% --json %destinationDirectory%
        ExportFormat: json
    -
        Executable: tool.exe
        CommandLine: -d %sourceDirectory% --xml %destinationDirectory%
        ExportFormat: xml
```

### Step 4: Write the Module File

```yaml
Description: <Tool Name> - <Artifact Type>
Category: <Category>
Author: Your Name
Version: 1.0
Id: <Generate GUID via: kape.exe --guid>
BinaryUrl: https://github.com/user/tool/releases/
ExportFormat: csv
Processors:
    -
        Executable: tool.exe
        CommandLine: -d %sourceDirectory% --csv %destinationDirectory%
        ExportFormat: csv
    -
        Executable: tool.exe
        CommandLine: -d %sourceDirectory% --json %destinationDirectory%
        ExportFormat: json

# Documentation
# https://github.com/user/tool
# <Add additional documentation links>
```

### Step 5: Validation

- Verify YAML syntax is valid
- Test with actual collected artifacts
- Verify output in all formats
- Run the KAPE Pull Request template checks
- Ensure blank line after last comment before end of file

## Common Module Patterns

### Single Format Output

```yaml
Processors:
    -
        Executable: tool.exe
        CommandLine: -d %sourceDirectory% --csv %destinationDirectory%
        ExportFormat: csv
```

### Multiple Format Support

```yaml
Processors:
    -
        Executable: tool.exe
        CommandLine: -d %sourceDirectory% --csv %destinationDirectory%
        ExportFormat: csv
    -
        Executable: tool.exe
        CommandLine: -d %sourceDirectory% --json %destinationDirectory%
        ExportFormat: json
    -
        Executable: tool.exe
        CommandLine: -d %sourceDirectory% --xml %destinationDirectory%
        ExportFormat: xml
```

### PowerShell-Based Processing

```yaml
Processors:
    -
        Executable: C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
        CommandLine: >
            -Command "Get-ChildItem %sourceDirectory% | 
            ConvertTo-Csv | 
            Out-File %destinationDirectory%\output.csv"
        ExportFormat: csv
```

### File Type Filtering

```yaml
FileMask: regex:.*\.(evtx|log)$
Processors:
    -
        Executable: parser.exe
        CommandLine: -d %sourceDirectory% --csv %destinationDirectory%
        ExportFormat: csv
```

## Analyzing Existing Modules

### Finding Modules by Type

Search for Modules in the appropriate category:
- `Modules/Windows/` - Windows native tools and artifacts
- `Modules/Apps/` - Third-party application parsers
- `Modules/EZTools/` - Eric Zimmerman's forensic tools
- `Modules/KapeResearch/` - Research and development modules
- `Modules/KapeSync/` - KAPE update and synchronization
- `Modules/Compound/` - Modules that reference other Modules

### Extracting Information from Module Files

1. **List all processors**: Parse the YAML and iterate through `Processors` array
2. **Find modules by category**: Filter by `Category` field
3. **Find modules using specific tool**: Search for `Executable` values
4. **Determine output formats**: Check each Processor's `ExportFormat`
5. **Identify artifacts parsed**: Examine tool documentation and descriptions

### Example: What does an EvtxECmd Module parse?

```yaml
# Hypothetical Module for Event Log parsing
Description: Windows Event Logs - CSV Format
Category: EventLogs
Executable: EvtxECmd.exe
CommandLine: -d %sourceDirectory% --csv %destinationDirectory%
ExportFormat: csv
```

This Module:
1. Uses EvtxECmd tool to parse Event Logs
2. Takes all `.evtx` files from the source directory
3. Outputs parsed data as CSV for analysis in spreadsheets
4. Writes results to the destination directory

## Compound Modules

Compound Modules reference other Modules rather than defining processors:

```yaml
Description: Complete Windows Analysis Suite
Category: Analysis
Author: Someone
Version: 1.0
Id: <GUID>
Processors:
    -
        Name: Event Logs
        Category: EventLogs
        Path: EventLogs_Parsing.mkape
    -
        Name: Registry Analysis
        Category: Registry
        Path: Registry_Analysis.mkape
```

Use Compound Modules to group related Modules for convenience.

## Best Practices

1. **Test thoroughly**: Verify the Module works with typical collected artifacts
2. **Support multiple formats**: Provide CSV, JSON, and XML when possible
3. **Document the tool**: Include URL to tool repository/documentation
4. **Use meaningful categories**: Maintain consistency with existing categorization
5. **Provide clear descriptions**: Help analysts understand what's being parsed
6. **Include command-line options**: Document any special flags or requirements
7. **Handle errors gracefully**: Plan for missing or malformed input files
8. **Version your Modules**: Track changes as tool versions change
9. **Test on different systems**: Verify Module works across OS versions

## Common Pitfalls and Solutions

| Issue | Solution |
|-------|----------|
| "File not found" for binary | Verify binary is in `.\KAPE\Modules\bin\` or subdirectory; use correct relative path |
| Output not generated | Verify output directory exists; check tool has write permissions; review command syntax |
| Incorrect path placeholders | Use `%sourceDirectory%` for input; `%destinationDirectory%` for output |
| Format mismatch | Ensure `CommandLine` output format matches declared `ExportFormat` |
| Module times out | Add `WaitTimeout` field; verify tool isn't waiting for input; optimize command |
| YAML syntax errors | Check indentation (2 spaces); verify quotes around multi-line commands; test in YAML validator |

## Integration with Targets

Modules work together with Targets:

1. **Targets collect** raw artifacts (files from disk)
2. **Modules parse** collected artifacts into structured data
3. **User chooses** which Modules to run and output format

Example workflow:

```
Target: Windows/WindowsDefender.tkape
  ↓ Collects: Defender logs and event files
  ↓
Module: Windows/Windows_EventLogs_Defender.mkape
  ↓ Parses: Event log files with EvtxECmd
  ↓
Output: parsed_output/EventLogs/Defender.csv
```

## Resources

- [Full KAPE Module Documentation](https://ericzimmerman.github.io/KapeDocs/#!Pages\2.2-Modules.md)
- [KAPE GitHub Repository](https://github.com/EricZimmerman/KapeFiles)
- [EvtxECmd Documentation](https://github.com/EricZimmerman/evtxECmd)
- [Eric Zimmerman's Tools](https://ericzimmerman.github.io/)

## Tools and Commands

### Generate a GUID for New Modules

```bash
kape.exe --guid
```

### Test Module Locally

1. Place parsed tool output in a test directory
2. Create Module with test command
3. Run with: `kape.exe --module module_name --msource test_dir --mdest output_dir`
4. Verify output format and content

### Common KAPE Module Tools

- **EvtxECmd**: Parse Windows Event Logs
- **RECmd**: Parse Windows Registry hives
- **SQLECmd**: Parse SQLite databases
- **JLECmd**: Parse Jump List artifacts
- **MFTECmd**: Parse MFT records
- **ShellBagsExplorer**: Shell Bag analysis
- **AmcacheParser**: Amcache hive parsing
- **AppCompatCacheParser**: AppCompatCache parsing

## Executable Best Practices

### Organizing Binaries

```
.\KAPE\Modules\bin\
  ├── EvtxECmd.exe
  ├── RECmd.exe
  ├── YourTool\
  │   ├── tool.exe
  │   └── dependency.dll
```

### Executable Path Reference

- Root binary: `Executable: YourTool.exe`
- In subfolder: `Executable: YourTool\tool.exe`
- System PATH: `Executable: system_tool.exe`

### PowerShell Execution

Always use full path for PowerShell:

```
Executable: C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
```

Or on PowerShell 6+:

```
Executable: C:\Program Files\PowerShell\7\pwsh.exe
```
