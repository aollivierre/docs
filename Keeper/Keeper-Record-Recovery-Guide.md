# Keeper Record Recovery Guide

This guide outlines the process for recovering deleted records from the Keeper vault using the Keeper Commander CLI.

## Prerequisites

- Keeper Commander CLI installed (`pip install keepercommander`)
- Access to a Keeper Enterprise or Business account
- SSO authentication credentials
- Record UID of the deleted item (From the Admin Console > Reports)

## Installation

Install Keeper Commander CLI using pip:

```bash
pip install keepercommander
```

## Authentication Process

1. Start the login process:
   ```bash
   keeper login your.email@domain.com
   ```

2. The system will display an SSO Login URL. The URL will open in your default browser.

3. Complete the SSO authentication in your browser.

4. Click the "Copy login token" button on the SSO Connect page.

5. Return to the CLI and select option 'p' to paste the SSO token.

6. Paste the copied token when prompted.

## Record Recovery Process

### 1. Restore from Trash
To restore a deleted record, use the `trash restore` command with the record's UID:

```bash
keeper trash restore <RECORD_UID>
```

Example:
```bash
keeper trash restore WVKEElwq6mupeG09rdx89g
```

### 2. Verify Restoration
Verify the record has been restored by retrieving its details:

```bash
keeper get <RECORD_UID>
```

## Record Properties After Recovery

When a record is restored:
- It returns to its original folder location
- All sharing permissions are preserved
- Record history is maintained
- All custom fields and attachments are restored

## Vault Backup Process

### Creating a Full Vault Backup

To create a complete backup of your vault including all shared records:

```bash
keeper export keeper_vault_backup_YYYY-MM-DD.json --format=json
```

Example:
```bash
keeper export keeper_vault_backup_2025-02-06.json --format=json
```

### Backup Contents
The JSON backup includes:
- All vault records (including shared records)
- Record metadata and sharing permissions
- Custom fields and attachments
- Folder structure and organization

### Backup Best Practices

1. **Regular Backups**
   - Schedule monthly vault backups
   - Use consistent naming convention with dates
   - Keep multiple versions of backups

2. **Secure Storage**
   - Store backups in a secure, encrypted location
   - Consider offline storage for critical backups
   - Maintain backups in multiple secure locations

3. **Backup Verification**
   - Test backup files periodically
   - Verify backup completeness by checking record count
   - Try importing backup into a test vault

4. **Backup Naming Convention**
   ```
   keeper_vault_backup_YYYY-MM-DD.json
   ```
   Example: `keeper_vault_backup_2025-02-06.json`

### Backup File Information
- Format: JSON
- Typical Size: 1-2 MB (varies with vault size)
- Encryption: Maintains Keeper's zero-knowledge encryption

### Restoring from Backup

To restore from a backup file:
```bash
keeper import --format=json keeper_vault_backup_YYYY-MM-DD.json
```

*Note: Always verify the backup file's integrity before overwriting any existing vault data.*

## Troubleshooting

### Common Issues

1. **Team Key Decryption Warnings**
   - Warnings about team key decryption may appear but usually don't affect record restoration
   - Example warning: `Could not decrypt team <TEAM_ID> key`

2. **SSO Token Issues**
   - If SSO token authentication fails, try generating a new token
   - Ensure you're using the most recent token from the SSO Connect page

### Best Practices

1. **Record Documentation**
   - Keep track of important record UIDs
   - Document the folder structure of critical records
   - Maintain a list of shared users and their permissions

2. **Recovery Verification**
   - Always verify restored records using the `keeper get` command
   - Check that all important fields and permissions are intact
   - Confirm the record appears in its intended folder

## Support

For additional assistance:
- Contact Keeper Support: https://www.keepersecurity.com/support.html
- Keeper Documentation: https://docs.keeper.io/
- Enterprise Support: Contact your Keeper Security account representative

## Version Information

- Last Updated: February 6, 2025
- Keeper Commander Version: 17.0.6
- Document Author: Cascade AI

---
*Note: This guide assumes you have the necessary permissions and access levels in your Keeper account to perform record recovery operations.*
