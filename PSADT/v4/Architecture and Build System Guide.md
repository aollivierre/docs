# PSADT v4: Architecture and Build System Guide

## Table of Contents

1. [Project Overview](#project-overview)
2. [Architecture](#architecture)
3. [UI Framework and Technologies](#ui-framework-and-technologies)
4. [Build System](#build-system)
5. [Runtime Requirements](#runtime-requirements)
6. [PowerShell Compatibility](#powershell-compatibility)

## Project Overview

PowerShell App Deployment Toolkit (PSADT) v4 represents a significant architectural evolution from previous versions. Version 4.0.5 introduces a hybrid approach combining C# WPF components with PowerShell deployment logic.

## Architecture

PSADT v4 follows a full-stack application architecture:

| Layer | Technology | Purpose |
|-------|------------|---------|
| **Frontend** | C# + XAML (WPF) | User interface components, dialogs, notifications |
| **Backend** | PowerShell | Deployment logic, system management, business rules |
| **Interface** | PowerShell wrappers | Bridge between UI components and deployment logic |

### Component Interaction

1. PowerShell functions (`Show-InstallationPrompt`, `Show-InstallationProgress`) act as API wrappers
2. These functions call compiled C# assemblies to render UI elements
3. WPF handles all visual rendering and user interactions
4. Results are passed back to PowerShell for processing

## UI Framework and Technologies

### Technology Stack

| Component | Technology | Description |
|-----------|------------|-------------|
| UI Framework | Windows Presentation Foundation (WPF) | Native Windows UI framework |
| Language | C# | Core UI logic and event handling |
| Markup | XAML | UI layout and styling definitions |
| PowerShell Role | Wrapper functions only | No direct UI rendering |

### Key Features

- ✅ High DPI support
- ✅ Modern Windows 10+ toast notifications
- ✅ Internationalization support
- ✅ Custom branding and styling
- ✅ Professional dialog interfaces

### UI Implementation

```
PowerShell Script → PowerShell Wrapper Function → C# Assembly → WPF UI
```

PowerShell is **not involved** in UI rendering. It only:
- Calls C# functions with parameters
- Processes user input results
- Manages deployment workflow

## Build System

### Build Tool

PSADT v4 uses **Invoke-Build**, an external PowerShell build automation module.

| Aspect | Details |
|--------|---------|
| Build Script | `PSAppDeployToolkit.build.ps1` |
| Required Module | `Invoke-Build` |
| Execution Method | `Invoke-Build -File PSAppDeployToolkit.build.ps1` |

### Build Process

| Task | Purpose | Dependencies |
|------|---------|--------------|
| Clean | Reset artifacts directory | None |
| ValidateRequirements | Check PowerShell version ≥ 5.1.0 | None |
| DotNetBuild | Compile C# solutions | ValidateRequirements |
| TestModuleManifest | Validate module manifest | None |
| EncodingCheck | Ensure UTF-8 with BOM | None |
| FormattingCheck | PSScriptAnalyzer formatting | None |
| Analyze | PSScriptAnalyzer code analysis | None |
| Test | Pester unit tests with coverage | None |
| Build | Merge functions, sign files | Multiple |
| IntegrationTest | Pester integration tests | Build |

### C# Components Built

1. **PSADT.dll** - Core functionality
2. **PSADT.UserInterface.dll** - WPF UI components  
3. **Invoke-AppDeployToolkit.exe** - Main executable
4. **Deploy-Application.exe** - Legacy compatibility

## Runtime Requirements

### .NET Targeting

PSADT v4 uses multi-targeting for maximum compatibility:

| Target Framework | Purpose | Windows Compatibility |
|------------------|---------|----------------------|
| .NET Framework 4.6.2 | Legacy compatibility | Windows 10 v1607+ (built-in) |
| .NET 8.0 Windows | Modern runtime | Requires separate installation |

### Runtime Configuration

| Setting | Value | Impact |
|---------|-------|--------|
| SelfContained | `false` | Smaller package size, requires runtime |
| PublishSingleFile | `false` | Multiple assembly files |
| PlatformTarget | `AnyCPU` | Cross-architecture compatibility |

### Availability

| Runtime | Windows Inclusion | Installation Required |
|---------|-------------------|----------------------|
| .NET Framework 4.6.2+ | ✅ Built into Windows 10 v1607+ (often 4.8+) | ❌ No |
| .NET 8.0 | ❌ Not included | ✅ Yes |

## PowerShell Compatibility

### Version Support

| PowerShell Version | .NET Runtime Used | Status |
|-------------------|-------------------|--------|
| PowerShell 5.1 | .NET Framework 4.6.2 | ✅ Fully supported |
| PowerShell 7.x | .NET 8.0 Windows | ✅ Supported with .NET 8 installed |

### .NET 9 Compatibility

| Scenario | Compatibility | Notes |
|----------|---------------|-------|
| .NET 9 installed (no .NET 8) | ⚠️ Potentially works | Forward compatibility not guaranteed |
| .NET 8 and 9 both installed | ✅ Full compatibility | Recommended configuration |
| Only .NET Framework | ✅ Works with PS 5.1 | Limited to legacy features |

### Best Practices

- .NET Framework 4.6.2+ is backward compatible (4.8+ versions commonly installed work fine)
- Install .NET 8.0 runtime for PowerShell 7.x compatibility
- Test compatibility thoroughly when using newer .NET versions
- Use PowerShell 5.1 for maximum compatibility in enterprise environments
- Use PowerShell 7.x with .NET 8.0 for modern features and performance