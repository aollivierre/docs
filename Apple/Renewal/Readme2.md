
# Apple Certificate and Token Renewal Guide 🍎

[![MIT License](https://img.shields.io/badge/License-MIT-green.svg)](https://choosealicense.com/licenses/mit/)
[![Contributions Welcome](https://img.shields.io/badge/contributions-welcome-brightgreen.svg?style=flat)](https://github.com/yourusername/apple-cert-renewal/issues)

A comprehensive guide for IT administrators on renewing Apple certificates and tokens for MDM management.

## Table of Contents
- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Certificate and Token Types](#certificate-and-token-types)
- [Renewal Processes](#renewal-processes)
  - [APNs Certificate Renewal](#apns-certificate-renewal)
  - [VPP Token Renewal](#vpp-token-renewal)
  - [ADE Token Renewal](#ade-token-renewal)
- [File Formats](#file-formats)
- [Important Notes](#important-notes)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)

## Overview

This guide provides step-by-step instructions for renewing three critical Apple management files that typically require annual renewal:

- Apple Push Notification service (APNs) Certificate
- Volume Purchase Program (VPP) Token
- Automated Device Enrollment (ADE) Token

## Prerequisites

- Access to Apple Business Manager (ABM)
- Access to your MDM solution (e.g., Jamf Pro)
- Valid Apple ID with administrative access
- Access to identity.apple.com
- Access to business.apple.com

## Certificate and Token Types

| File/Certificate | Description | Validity | Purpose | File Format |
|-----------------|-------------|----------|----------|-------------|
| APNs Certificate | Enables MDM server to send push notifications | 1 Year | Manages communication between MDM and Apple devices | .cer |
| CSR | Certificate Signing Request for APNs | N/A | Required for APNs certificate generation | .plist |
| VPP Token | Manages app distribution | 1 Year | Enables bulk app purchasing and distribution | .vpptoken |
| ADE Token | Enables automated enrollment | 1 Year | Automates device enrollment process | .p7m |

## Renewal Processes

### APNs Certificate Renewal

1. Contact Apple Support if needed:
   - Phone: 1-866-902-7144 (Canada)
   - Hours: Monday-Friday, 8 AM - 7 PM CST
   - Web: support.apple.com

2. Generate CSR from MDM:
   - Access your MDM solution
   - Navigate to certificate settings
   - Generate new CSR (.plist file)

3. Obtain New Certificate:
   - Visit identity.apple.com
   - Sign in with Apple ID
   - Upload CSR
   - Download new certificate (.cer file)

4. Upload to MDM:
   - Access your MDM solution
   - Upload new certificate
   - Verify renewal success

### VPP Token Renewal

1. Access Apple Business Manager:
   - Visit business.apple.com
   - Sign in with administrator account

2. Navigate to Token Settings:
   - Click your name (bottom-left corner)
   - Select "Preferences"
   - Choose "Payments & Billing"
   - Select "Apps and Books"
   - Scroll to "Content Tokens"

3. Download New Token:
   - Click "Download" next to required token
   - Save .vpptoken file

4. Upload to MDM:
   - Access your MDM solution
   - Upload new VPP token
   - Verify renewal success

### ADE Token Renewal

1. Access Apple Business Manager:
   - Visit business.apple.com
   - Sign in with administrator account

2. Navigate to Server Settings:
   - Click your name (bottom-left corner)
   - Select "Preferences"
   - Scroll to "MDM Servers"

3. Update Server Configuration:
   - Select your MDM server
   - Click "Edit"
   - Click "Upload New..."
   - Upload public key certificate from MDM
   - Click "Apply"

4. Download and Upload Token:
   - Click "Download Token"
   - Save .p7m file
   - Upload to your MDM solution
   - Verify renewal success

## Important Notes

- Start renewal process when receiving 30-day expiration notice
- Keep track of renewal dates for all certificates/tokens
- Consider using a shared service account for certificate management
- Always verify successful renewal in MDM solution
- Maintain backup copies of current certificates/tokens

## Troubleshooting

If you encounter issues during renewal:

1. Verify Apple ID permissions
2. Check network connectivity
3. Clear browser cache if using web portals
4. Contact Apple Support if needed
5. Consult MDM provider documentation

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request. For major changes, please open an issue first to discuss what you would like to change.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---
📝 Created and maintained by Abdullah Ollivierre
```