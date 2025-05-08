## Overview
This article contains documentation for the Windows Firewall configuration policy specifically designed to enable SFTP connections using WinSCP for business-critical file transfers.

## Service Desk Instructions

### Granting User Access
To enable a user to connect using WinSCP:
1. Access the [Entra ID Admin Center](https://entra.microsoft.com)
2. Navigate to Groups > All groups
3. Locate the cloud group: "MEM Firewall rule for sftp outbound"
4. Select the group and go to Members
5. Click "Add members" and search for the user's device requiring WinSCP access
6. Select the device and click "Select" to add it to the group
7. The policy will automatically apply through one of these methods:
   - **Natural Sync:** Intune client automatically checks for new policies every 8 hours
   - **Device Check-in:** Occurs automatically when:
     - User signs in
     - Device restarts
     - Network state changes
     - Windows Update scan
     - Every 8 hours on Windows 10/11

   To force immediate policy application, you can use:
   - **Option 1:** Restart the computer
   - **Option 2:** Force MDM policy sync through Windows Settings:
     1. Open Settings (Windows + I)
     2. Navigate to Accounts > Access work or school
     3. Select your connected account (with briefcase icon)
     4. Click Info
     5. Under Device Action Status, click Sync
   - **Option 3:** Administrator method through Intune Admin Center:
     1. Sign in to [Intune Admin Center](https://intune.microsoft.com)
     2. Go to Devices > All Devices
     3. Select the target device(s)
     4. Use "Bulk Device Actions" to force sync

Note: Even though these are co-managed devices, Windows Firewall rules configured in Endpoint Security are delivered through Intune's MDM channel. The policy sync must be initiated through Windows Settings or the Intune admin portal, not through MECM methods.

### Important Policy Sync Notes
- Firewall rules can be merged from multiple profiles if they don't conflict
- Each profile supports up to 150 custom rules
- Policy conflicts between different types (device configuration vs. endpoint security) can prevent settings from being applied
- Monitor policy status through Endpoint Security > Firewall > Summary in Intune portal
- No manual action is required in most cases as policies will sync automatically through the natural device check-in cycle

### Troubleshooting Access
If a user cannot connect via WinSCP:
1. Verify user's device is in the correct cloud group
2. Check if user is not in the exclusion group "MEM Firewall Troubleshooting"
3. Confirm policy application by checking Windows Firewall on user's machine
4. Verify Windows Defender Firewall is enabled (should not be disabled locally)
5. If issues persist, force an Intune policy sync through one of the methods described above

## Important Security Notes

### Windows Firewall Status
- **Critical:** Windows Defender Firewall must remain enabled for this policy to function
- Users can technically disable the firewall locally through Control Panel (firewall.cpl)
- **Warning:** Disabling the firewall locally can cause inconsistent behavior:
  - Intune policy and local settings may conflict
  - Network traffic might still be blocked despite firewall appearing disabled
  - System may show different states across Windows Security GUI and Intune
  - Unpredictable packet filtering behavior may occur
- Regular compliance monitoring should be performed through Intune's Endpoint Security > Firewall section

### Security Best Practices
1. **Never disable Windows Defender Firewall**
   - Disabling firewall increases security risks
   - Creates compliance issues with security policies
   - May expose systems to unauthorized network traffic

2. **Policy Enforcement**
   - Intune policies should be configured to prevent local policy merge
   - Regular compliance checks should be performed
   - Security teams should monitor for disabled firewalls

3. **Troubleshooting Steps for Disabled Firewall**
   - Use PowerShell to re-enable: `netsh advfirewall set allprofiles state on`
   - Verify registry values for policy conflicts
   - Check Windows Security GUI for status inconsistencies

## Policy Details

### General Information
- **Policy Name:** Firewall Rules for SFTP
- **Created:** May 30, 2022
- **Last Modified:** December 3, 2024
- **Platform:** Windows
- **Technologies:** MDM (Mobile Device Management), Microsoft Sense
- **Policy ID:** 8b48d8dc-fcad-4c73-ac91-3a4a511c9238
- **Direct Link:** [View Policy in Intune Portal](https://intune.microsoft.com/#view/Microsoft_Intune_Workflows/PolicySummaryBlade/policyId/8b48d8dc-fcad-4c73-ac91-3a4a511c9238/templateId/19c8aa67-f286-4861-9aa0-f23541d31680_1/templateTypeName/Windows%20Firewall%20Rules/platformName/windows10/isAssigned~/true/technology/mdm%2CmicrosoftSense/templateFamilyName/endpointSecurityFirewall)

### Firewall Rule Configuration
- **Rule Name:** SFTP: all outbound TCP
- **Direction:** Outbound
- **Protocol:** TCP (6)
- **Remote Port:** 22 (Standard SFTP port)
- **Action:** Allow
- **Status:** Enabled
- **Network Profiles:** Applied to all (Domain, Private, and Public)

### Network Profiles
The rule is applied to all network types:
1. **Domain Profile (FW_PROFILE_TYPE_DOMAIN)**
   - For networks connected to domains
2. **Private Profile (FW_PROFILE_TYPE_PRIVATE)**
   - Standard profile for private networks
   - Typically behind NAT devices, routers, and other edge devices
   - Common in home or office environments
3. **Public Profile (FWPROFILETYPEPUBLIC)**
   - For networks classified as public
   - Typically in airports, coffee shops, and other public places
   - Used when network peers or administrators are not trusted

### Purpose
This firewall rule is configured to enable secure SFTP connections using WinSCP application to:
- Medalia FTP server
- filetransfter.wawanesalife.com
- Any other SFTP server using standard port 22

**Important Note**: This policy allows outbound connections to ANY SFTP server on TCP port 22, not only the specifically listed servers above. No modification to the policy is needed when adding new SFTP server destinations.

### Applications
#### WinSCP
- **Purpose:** Primary SFTP client application used for secure file transfers
- **Protocol:** SFTP (SSH File Transfer Protocol)
- **Port:** TCP 22 (standard SFTP port)
- **Usage:** Enables secure file transfers between local systems and remote SFTP servers
- **Security:** Utilizes SSH protocol for encrypted data transmission

### Policy Assignment
The policy is managed through Microsoft Intune, with groups in Entra ID:
- **Included Group:** "MEM Firewall rule for sftp outbound"
  - Filter: None
  - Filter Mode: None
  - Pure cloud Entra ID group (not synchronized from MECM)
  - Contains **devices** as members, not users
- **Excluded Group:** "MEM Firewall Troubleshooting"
  - Pure cloud Entra ID group (not synchronized from MECM)
- **Scope Tags:** Default

## Implementation
The policy is implemented using:
- Microsoft Intune (Microsoft Endpoint Manager)
- Modern Device Management (MDM)
- Windows Defender Firewall Configuration Service Provider (CSP)
- Microsoft Sense for monitoring and security

### Configuration Service Provider (CSP) Details
The Firewall CSP enables:
- Management of non-domain devices
- Reduction of network security threats
- Configuration of Windows Defender Firewall global settings
- Per-profile settings management
- Custom rules enforcement across devices

## Security Considerations
- Rule is specifically limited to SFTP traffic (TCP port 22)
- Outbound-only connections are permitted
- Group-based assignment ensures controlled distribution
- All network profiles are covered for consistent security
- Comprehensive coverage across all network profiles (Domain, Private, Public)
- Regular deployment status monitoring available through Intune portal

## Maintenance
For any modifications to this policy:
1. Access Microsoft Endpoint Manager admin center
2. Navigate to Endpoint Security > Firewall
3. Locate "Firewall Rules for SFTP" policy
4. Make necessary adjustments following change management procedures

## Support
For different types of issues:
- **Service Desk:** Add devices to appropriate cloud groups for access
- **Security Team:** Policy configuration and rule modifications
- **System Administrator:** MECM and Entra ID synchronization issues
- **Security Incidents:** Report immediately if firewall is found disabled
