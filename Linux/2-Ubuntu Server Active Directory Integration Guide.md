# Ubuntu Server Active Directory Integration Guide

This guide provides step-by-step instructions for integrating Ubuntu Server with Active Directory (AD) for SSH authentication using SSSD.

## Prerequisites

- Ubuntu Server (20.04 or newer)
- Active Directory domain controller
- Domain administrator credentials
- DNS resolution to the domain controller

## System Information

- Domain: cci.local
- Domain Controller: 10.224.10.10
- Ubuntu Server: 10.224.10.9

## Step 1: Install Required Packages

```bash
sudo apt update
sudo apt install -y realmd sssd sssd-tools libnss-sss libpam-sss adcli samba-common-bin oddjob oddjob-mkhomedir packagekit
```

## Step 2: Configure DNS Resolution

Ensure your server can resolve the domain. Add the domain controller as DNS server:

```bash
echo -e "[Resolve]\nDNS=10.224.10.10\nDomains=cci.local" | sudo tee /etc/systemd/resolved.conf
sudo systemctl restart systemd-resolved
```

Verify DNS resolution:
```bash
ping -c 2 cci.local
```

## Step 3: Join the Domain

Join the domain using realm:
```bash
sudo realm join -U Administrator cci.local --install=/
```

## Step 4: Configure SSSD

Create SSSD configuration:
```bash
sudo tee /etc/sssd/sssd.conf << EOF
[sssd]
domains = cci.local
config_file_version = 2
services = nss, pam

[domain/cci.local]
ad_domain = cci.local
krb5_realm = CCI.LOCAL
cache_credentials = True
id_provider = ad
access_provider = ad
use_fully_qualified_names = True
fallback_homedir = /home/%u
default_shell = /bin/bash
EOF

sudo chmod 600 /etc/sssd/sssd.conf
```

## Step 5: Configure PAM

Configure PAM for SSH:
```bash
sudo tee /etc/pam.d/sshd << EOF
@include common-auth
@include common-account
@include common-password
@include common-session
session required pam_mkhomedir.so skel=/etc/skel/ umask=0022
EOF
```

## Step 6: Configure SSH Server

Update SSH server configuration:
```bash
sudo tee /etc/ssh/sshd_config << EOF
Include /etc/ssh/sshd_config.d/*.conf
PasswordAuthentication yes
ChallengeResponseAuthentication yes
UsePAM yes
X11Forwarding yes
PrintMotd no
AcceptEnv LANG LC_*
Subsystem sftp /usr/lib/openssh/sftp-server
EOF
```

## Step 7: Restart Services

```bash
sudo systemctl restart sssd
sudo systemctl restart ssh
```

## Step 8: Configure Client SSH

On your Windows/Linux client, create or update `~/.ssh/config`:
```
Host cci-layernext01
    HostName 10.224.10.9
    User admin-layernext@cci.local
    PasswordAuthentication yes
    PubkeyAuthentication no
    GSSAPIAuthentication no
    PreferredAuthentications password
    StrictHostKeyChecking no
    UserKnownHostsFile /dev/null
```

## Verification

1. Check domain join status:
```bash
realm list
```

2. Verify user lookup:
```bash
getent passwd username@cci.local
```

3. Test SSH connection:
```bash
ssh admin-layernext@cci.local@10.224.10.9
```

## Troubleshooting

### Common Issues

1. DNS Resolution
```bash
nslookup cci.local
```

2. SSSD Status
```bash
sudo systemctl status sssd
```

3. Authentication Logs
```bash
sudo tail -f /var/log/auth.log
```

4. SSSD Logs
```bash
sudo journalctl -u sssd
```

### Common Fixes

1. Clear SSSD Cache:
```bash
sudo sssctl cache-remove
sudo systemctl restart sssd
```

2. Verify Group Membership:
```bash
id username@cci.local
```

3. Check PAM Configuration:
```bash
sudo pam-auth-update
```

## Notes

- Always use fully qualified usernames (username@domain.local)
- Home directories are automatically created on first login
- Group memberships are synchronized from AD
- Password changes must be done through AD

## Security Considerations

1. Consider enabling SSH key authentication after initial setup
2. Review and adjust SSH configuration for your security requirements
3. Consider implementing SSH allow/deny lists
4. Monitor authentication logs regularly





After completing these steps, any AD user will be able to SSH into the server with a few important points to remember:

1. The user must use their full username format: `username@cci.local`
2. They must use their AD password
3. A home directory will be automatically created for them on their first login
4. They will inherit their group permissions from AD

However, if you want to restrict which AD users can access the server, you have a few options:

1. Use `realm permit` to allow specific users:
```bash
sudo realm permit -U username@cci.local
```

2. Allow specific AD groups:
```bash
sudo realm permit -g "Group Name@cci.local"
```

3. Or add restrictions in the SSH config by adding `AllowGroups` to `/etc/ssh/sshd_config`:
```bash
AllowGroups sudo "Domain Admins@cci.local" "IT Staff@cci.local"
```











By default AD users cannot use sudo. To allow specific AD users or groups to use sudo, you'll need to configure sudo permissions. Here are the steps:

1. Create a new sudo configuration file for AD users:
```bash
sudo visudo -f /etc/sudoers.d/domain-admins
```

You have a few options for granting sudo access:

1. For specific AD users:
```bash
# Allow specific AD user
admin-layernext@cci.local ALL=(ALL:ALL) ALL
```

2. For AD groups (recommended approach):
```bash
# Allow members of Domain Admins group
%Domain\ Admins@cci.local ALL=(ALL:ALL) ALL

# Allow members of specific AD group
%IT\ Admins@cci.local ALL=(ALL:ALL) ALL
```

3. For all members of a specific group with no password:
```bash
# Allow without password prompt (use with caution)
%Domain\ Admins@cci.local ALL=(ALL:ALL) NOPASSWD:ALL
```







## Maintenance

Regular maintenance tasks:
1. Keep system packages updated
2. Monitor SSSD and SSH logs
3. Review and update security configurations
4. Test AD authentication periodically

## Support

For issues:
1. Check system logs (/var/log/auth.log)
2. Verify network connectivity
3. Ensure AD credentials are valid
4. Verify DNS resolution
5. Check SSSD service status 