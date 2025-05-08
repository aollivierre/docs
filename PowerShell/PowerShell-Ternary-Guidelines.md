# PowerShell Ternary Operator Guidelines

## Overview

This document provides guidance on when and how to use conditional expressions in PowerShell, with a focus on compatibility between PowerShell 5.1 and PowerShell 7+.

## General Rule

**Prefer using the PowerShell 5.1 compatible one-line if-else syntax over the PowerShell 7+ ternary operator for maximum compatibility.**

## PowerShell Conditional Expression Options

### 1. Traditional Multi-line If-Else (All PowerShell Versions)
```powershell
if ($condition) {
    $result = $trueValue
}
else {
    $result = $falseValue
}
```

### 2. One-line If-Else Expression (All PowerShell Versions)
```powershell
$result = if ($condition) { $trueValue } else { $falseValue }
```

### 3. True Ternary Operator (PowerShell 7+ Only)
```powershell
$result = $condition ? $trueValue : $falseValue
```

## Decision Flowchart

1. Does the project need to support PowerShell 5.1?
   - **Yes**: Use the one-line if-else expression (Option 2)
   - **No**: Either Option 2 or Option 3 is acceptable

2. Is the condition complex or does it involve multiple steps?
   - **Yes**: Use traditional multi-line if-else (Option 1)
   - **No**: Use one-line if-else or ternary

3. Is code readability the highest priority?
   - **Yes**: Use the format that makes the code most understandable in context
   - **No**: Use the most concise option appropriate for the PowerShell version

## When to Use Each Approach

### Traditional Multi-line If-Else
- For complex conditions with multiple lines of code
- When each branch has multiple operations
- When maximum readability is important
- When the logic flow needs to be very explicit

### One-line If-Else Expression
- For simple conditions that return a value
- When PowerShell 5.1 compatibility is required
- When you want an expression-based approach that works everywhere
- For moderate code conciseness while maintaining readability

### Ternary Operator
- Only in PowerShell 7+ projects with no backward compatibility needs
- For very simple conditions where extreme conciseness is valued
- When you want code that looks more like other programming languages
- For developers familiar with C#, JavaScript, or other languages with ternary operators

## Best Practices

1. **Prioritize Readability**: Choose the approach that makes your code most readable
2. **Consistency**: Use the same approach throughout a project
3. **Compatibility**: Default to the one-line if-else for maximum compatibility
4. **Documentation**: If using ternary operators, note the PS7+ requirement in documentation
5. **Complexity**: As complexity increases, prefer traditional if-else structures

## Examples

### Variables Based on Conditions

```powershell
# Compatible with all PowerShell versions (PREFERRED)
$logLevel = if ($isVerbose) { "VERBOSE" } else { "INFO" }

# PowerShell 7+ only (AVOID unless PS7-only project)
$logLevel = $isVerbose ? "VERBOSE" : "INFO"
```

### Inline Conditions

```powershell
# Compatible with all PowerShell versions (PREFERRED)
Write-Host "Status: $(if ($isOnline) { "Online" } else { "Offline" })"

# PowerShell 7+ only (AVOID unless PS7-only project)
Write-Host "Status: $($isOnline ? "Online" : "Offline")"
```

### Function Parameters with Defaults

```powershell
# Compatible with all PowerShell versions (PREFERRED)
function Get-Thing {
    param ($id = (if ($script:lastId) { $script:lastId } else { 1 }))
    # Function body
}

# PowerShell 7+ only (AVOID unless PS7-only project)
function Get-Thing {
    param ($id = ($script:lastId ? $script:lastId : 1))
    # Function body
}
```
