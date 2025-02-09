# Layer Next Engineer Onboarding Guide
## Canada Computing Infrastructure Access

Welcome to Canada Computing infrastructure! This guide provides step-by-step instructions for accessing and using the infrastructure set up for Layer Next engineers.

## System Information
- Domain: cci.local
- Application Server: CCI-LAYERNEXT01 (10.224.10.9)
- SQL Server: CCI-SQL01 (10.224.10.11)
- Domain Controller: CCI-DC01 (10.224.10.10)

## Access Setup Process

### Step 1: VPN Setup
1. Download and install FortiClient VPN (free version)
   - Latest version: 7.4
   - Available for both Windows and macOS
   - Download link for Windows: https://links.fortinet.com/forticlient/win/vpnagent
   - Download link for macOS: https://links.fortinet.com/forticlient/mac/vpnagent

2. VPN Connection Settings
   - Server IP and configuration details will be sent separately via secure email
   - Use the provided VPN credentials (separate from domain credentials)
   - Enable auto-connect and browser settings as specified in the secure email

### Step 2: SSH Access to Ubuntu Application Server
After connecting to VPN, you can SSH into the Ubuntu application server:

```bash
ssh admin-layernext@cci.local@10.224.10.9
```

**Credentials:**
- Username: `admin-layernext@cci.local` (AD domain account)
- Password: [Provided separately]
- Port: 22

### Step 3: Sudo Access
Your account has been configured with sudo privileges:
- Can execute commands with sudo (e.g., `sudo whoami`)
- No password required for sudo commands
- Full administrative access on the Ubuntu server

### Step 4: SQL Server Access
The application server has been configured to access the SQL Server:
- SQL Server: cci-sql01.cci.local
- Port: 1433
- Authentication: Use your domain credentials
- Firewall rules are configured to allow access

## Testing Your Access

1. **VPN Connection**
```bash
# After connecting to VPN, verify network access
ping cci-layernext01.cci.local
```

2. **SSH Connection**
```bash
# Test SSH access
ssh admin-layernext@cci.local@10.224.10.9
```

3. **Sudo Privileges**
```bash
# Test sudo access
sudo whoami  # Should return 'root' without password prompt
```

4. **SQL Server Connection**
```bash
# Test SQL Server connectivity
telnet cci-sql01.cci.local 1433
```

## Important Notes

1. **Authentication**
   - VPN credentials are separate from domain credentials
   - Domain account (admin-layernext@cci.local) is used for:
     - SSH access
     - SQL Server authentication
     - System administration

2. **Security**
   - Always connect through VPN before accessing any resources
   - Keep your credentials secure
   - Follow security best practices

3. **Network Access**
   - All access is through the VPN
   - DNS resolution is configured automatically
   - Firewall rules are pre-configured

## Support and Additional Assistance

For any issues or additional requirements:
1. Contact: Abdullah
2. Available via:
   - Email
   - Microsoft Teams
   - Phone

Common requests that can be handled:
- Additional software installation
- Permission adjustments
- Access to additional resources
- Infrastructure modifications

## Related Documentation
- [Ubuntu Server Passwordless Sudo Configuration Guide](https://github.com/aollivierre/docs/blob/main/Linux/1-Ubuntu%20Server%20Passwordless%20Sudo%20Configuration%20Guide.md)
- [Ubuntu Server Active Directory Integration Guide](https://github.com/aollivierre/docs/blob/main/Linux/2-Ubuntu%20Server%20Active%20Directory%20Integration%20Guide.md)
- [Windows Server DNS and SQL Firewall Configuration Guide](https://github.com/aollivierre/docs/blob/main/Linux/3-Windows%20Server%20DNS%20and%20SQL%20Firewall%20Configuration%20Guide.md)

## Maintenance and Updates

1. **Regular Tasks**
   - Keep FortiClient VPN updated
   - Change passwords according to policy
   - Monitor system access and performance

2. **Best Practices**
   - Always disconnect VPN when not in use
   - Report any access issues immediately
   - Follow security protocols

## Troubleshooting

### Common Issues and Solutions

1. **VPN Connection Issues**
   - Verify FortiClient version
   - Check network connectivity
   - Ensure correct credentials

2. **SSH Access Problems**
   - Confirm VPN is connected
   - Verify domain credentials
   - Check network connectivity

3. **SQL Server Connection Issues**
   - Verify DNS resolution
   - Check domain authentication
   - Confirm firewall access

### Getting Help
If you encounter any issues:
1. Check the related documentation
2. Try the troubleshooting steps
3. Contact support if issues persist 