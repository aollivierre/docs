## Overview

This document outlines the code signing process for PowerShell scripts in our environment. Code signing is required due to the SCCM/MECM policy implemented on our computers and servers that enforces script execution policies.

## What Requires Code Signing

### Scripts That Require Signing:
- PowerShell scripts (.ps1) that are manually pushed through MECM
- Scripts published to Software Center (available or required)
- Platform scripts used for deployment
- Automated scripts (scheduled tasks, remediation scripts)

### Scripts That Don't Require Signing:
- PS App Deploy Toolkit (PSADT) scripts (Any scripts using deploy-application.exe as the execution method)
- Batch scripts
- SQL scripts
- WQL scripts
- VBScripts


## Certificate Location and Access

The code signing certificate is located on the Azure SCCM01 primary site server: (`insert UNC Path \\`)
- Location: Root of C: drive
- Access Requirements:
  - Device Management Admin account required
  - If working remotely: Cisco VPN connection required
  - If in office: Direct network access

## Certificate Requirements

- Format: .pfx file (contains private key)
- Do not use: .cer or .crt files (public key only)
- Certificate expiration: Every 2 years
- Issued by: AD Certificate Services

## Certificate Import Process

1. Access Requirements:
   - Launch Terminal or PowerShell as admin first
   - Launch certificate manager (certmgr.msc) from the elevated Terminal/PowerShell prompt
   - Use admin account (Windows Hello biometric authentication do not work with standard accounts)

2. Import Location:
   - Use certmgr.msc (user store), NOT certlm.msc (machine store)
   - Import to Personal store under user certificates
   - Must be imported to admin user's certificate store

3. Import Steps:
   - Open certmgr.msc as administrator
   - Navigate to Personal store
   - Right-click > All Tasks > Import
   - Select the .pfx file
   - Enter the certificate password (Check with Device Management team member or consult password vault)
   - Use default options for remaining prompts

## Code Signing Process

1. Import the certificate as described above
2. Use the provided PowerShell signing script (location to be specified)
3. The script will:
   - Locate the certificate in your store
   - Retrieve certificate metadata and thumbprint
   - Sign all .ps1 files in the specified directory
   - Provide a summary of signed files

## Important Considerations

1. Certificate Expiration:
   - Certificates expire every 2 years
   - Expired certificates will cause signed scripts to stop working
   - All affected scripts must be re-signed with the new certificate

2. Automated Scripts:
   - Avoid signing automated scripts (remediation, scheduled tasks)
   - Signed automated scripts will break when certificates expire
   - Focus signing on manually deployed scripts

3. Security Considerations:
   - Keep private key (.pfx) secure
   - Only import on administrator workstations
   - Public key can be freely distributed
   - Certificate password is separate from the private key

## Development Impact

- Adds minimal complexity to PowerShell development
- Additional step required during script creation
- Not required for PSADT-based deployments

## Future Improvements

The following areas have been identified for potential improvement:
- Automation of the signing process
- Interactive folder selection for batch signing
- Enhanced documentation with specific paths and script names
- Improved certificate renewal process

## Notes

- This document will be updated as processes are refined
- Contact the Device Management team for additional clarification
- Regular reviews of signed scripts should be conducted before certificate expiration