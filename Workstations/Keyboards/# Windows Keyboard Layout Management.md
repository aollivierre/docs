# Windows Keyboard Layout Management Script

This repository contains PowerShell scripts to manage keyboard layouts in Windows 11 (also works in Windows 10). The scripts allow you to list, remove, and set keyboard layouts without requiring administrator privileges.

## Key Features

- List current keyboard layouts
- Remove unwanted keyboard layouts
- Add specific keyboard layouts
- No administrator privileges required
- Works in user context

## Background

Sometimes Windows systems end up with multiple keyboard layouts, which can be frustrating when switching between them accidentally. This script helps manage these layouts efficiently.

Common scenarios where this is useful:
- Removing unwanted keyboard layouts that were automatically added
- Cleaning up after system updates that may have added additional layouts
- Standardizing keyboard layouts across user profiles

## Usage

### Viewing Current Keyboard Layouts

```powershell
Get-WinUserLanguageList | Format-List *
```

This command shows your current language and keyboard settings, including:
- LanguageTag
- InputMethodTips (keyboard layouts)
- Other language-related settings

### Resetting to US English Layout

```powershell
# Get the current language list
$LanguageList = Get-WinUserLanguageList

# Clear existing keyboard layouts
$LanguageList[0].InputMethodTips.Clear()

# Add back US English QWERTY (standard US layout)
$LanguageList[0].InputMethodTips.Add("0409:00000409")

# Apply the changes
Set-WinUserLanguageList $LanguageList -Force

# Verify the changes
Get-WinUserLanguageList | Format-List *
```

## Common Keyboard Layout Codes

- US English QWERTY: `0409:00000409`
- Canadian Multilingual: `0c0c:00011009`
- US English Dvorak: `0409:00010409`

## Lessons Learned

1. **User Context**: These commands work in the current user context without requiring elevation (admin rights).

2. **Property Access**: Earlier versions of the script tried to access `InputMethodTips` directly, which caused errors. The correct approach is to access it through the language list object.

3. **Clear and Add**: Sometimes it's more reliable to clear all layouts and add back the desired one rather than trying to remove specific layouts.

4. **Verification**: Always verify changes after applying them using `Get-WinUserLanguageList | Format-List *`

## Troubleshooting

If you encounter the error "Property cannot be found":
- This usually means you're trying to access properties directly instead of through the language list object
- Use the full object path as shown in the scripts above

If new layouts keep appearing:
- Check your Windows language settings
- Verify no group policies are automatically adding layouts
- Consider running the script again after system updates

## Contributing

Feel free to submit issues and enhancement requests!

## License

MIT License - feel free to use and modify as needed.

## Credits

Created based on real-world experience dealing with multiple keyboard layouts in Windows 11. Special thanks to the PowerShell community for guidance on language management cmdlets.