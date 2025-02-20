# Windows Hello for Business Key Storage on Devices Without TPM

When a device lacks a Trusted Platform Module (TPM), Windows Hello for Business (WHfB) defaults to **software-based key storage**, which introduces distinct security characteristics and storage mechanisms. Below is a detailed technical analysis of how WHfB handles credentials in such environments.

## Key Storage Architecture Without TPM

## NGC Folder vs. Key Material

The **Ngc folder** (`C:\Windows\...\Microsoft\Ngc`) stores **metadata** (e.g., user SIDs, policy configurations, encrypted PIN parameters) but **does not directly store cryptographic keys**. Instead:

1. **Private Keys**: Encrypted using **DPAPI-NG** (Data Protection API – Next Generation) and stored in the Windows Registry or disk-based containers.
    
2. **Key Derivation**: PIN-derived secrets generate encryption keys via PBKDF2-SHA256 (100,000 iterations), securing the DPAPI-NG blobs.
    

Example registry path for encrypted PIN data:

text

`HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\NgcPin\Credentials\<UserSID>`  

## Security Implications of Software-Based Storage

|**Aspect**|**TPM-Protected Keys**|**Software-Stored Keys**|
|---|---|---|
|**Storage Location**|TPM hardware (inaccessible)|Disk/registry (encrypted via DPAPI-NG)|
|**Brute-Force Resistance**|TPM locks after 5-10 failed PIN attempts|No hardware rate-limiting; vulnerable to offline attacks (1M+ guesses/sec)|
|**Key Extraction**|Physically impossible|Possible via disk/registry extraction and brute-forcing|

## Attack Vectors on TPM-Less Systems

## 1. Offline PIN Brute-Forcing

Tools like **Elcomsoft System Recovery** can:

1. Extract encrypted DPAPI-NG blobs from the registry or disk.
    
2. Use GPU acceleration to brute-force 4/6-digit PINs (hashcat achieves ~1M guesses/sec).
    
3. Decrypt WHfB credentials and access BitLocker keys, Azure AD PRT tokens, or on-premises Kerberos TGTs.
    

**Mitigation**:

- Enforce **alphanumeric PINs** (12+ characters) via Group Policy/Intune.
    
- Enable **Full Disk Encryption (BitLocker)** to prevent offline disk access.
    

## 2. Credential Theft via Malware

Malicious processes running as SYSTEM can:

1. Dump DPAPI-NG masterkeys from memory.
    
2. Decrypt WHfB credentials stored in `HKEY_LOCAL_MACHINE`.
    

**Mitigation**:

- Restrict local admin privileges.
    
- Deploy **Microsoft Defender Credential Guard**.
    

## Administrative Best Practices

## Policy Configuration for TPM-Less Devices

1. **Block WHfB Enrollment**: Use Intune/Group Policy to prevent WHfB use on non-TPM devices:
    
    powershell
    
    `# Intune Settings Catalog:   ConfigureWindowsHello = 0`  
    
2. **Mandate Alphanumeric PINs**:
    
    powershell
    
    `New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\PassportForWork" -Name "PINComplexity" -Value 3 -Type DWORD`  
    
3. **Audit Ngc Permissions**: Ensure only SYSTEM/Administrators have access to `C:\Windows\...\Ngc`.
    

## Comparative Analysis: TPM vs. Software Storage

|**Criteria**|TPM|Software Storage|
|---|---|---|
|**Key Protection**|Hardware-bound (TPM 2.0)|Encrypted via DPAPI-NG (disk/registry)|
|**SSO Scope**|Hybrid/cloud Kerberos trust|Limited to Entra ID-cloud resources|
|**Recovery Post-Image**|Keys lost after system reimaging|Keys persist in disk image backups|
|**Compliance**|FIPS 140-2/3 compliant|Non-compliant for high-security needs|

## Conclusion

On devices without TPM:

1. **Keys are stored on disk/registry** using DPAPI-NG encryption, **not directly in the Ngc folder**.
    
2. **Security risks escalate** due to offline brute-force vulnerabilities and malware susceptibility.
    
3. **Mitigation requires**:
    
    - Enforcing complex PINs.
        
    - Avoiding WHfB on non-TPM devices where possible.
        

For enterprises, mandating TPM 2.0 remains critical to maintaining Zero Trust standards. Legacy devices without TPM should transition to certificate-based authentication or be phased out.

### Citations:

1. [https://www.stigviewer.com/stig/microsoft_windows_10/2024-06-13/finding/V-220846](https://www.stigviewer.com/stig/microsoft_windows_10/2024-06-13/finding/V-220846)
2. [https://learn.microsoft.com/en-us/answers/questions/452782/tpm-needed-for-windows-hello-windows-hello-for-bus](https://learn.microsoft.com/en-us/answers/questions/452782/tpm-needed-for-windows-hello-windows-hello-for-bus)
3. [https://github.com/MicrosoftDocs/windows-itpro-docs/blob/public/windows/security/identity-protection/hello-for-business/how-it-works.md](https://github.com/MicrosoftDocs/windows-itpro-docs/blob/public/windows/security/identity-protection/hello-for-business/how-it-works.md)
4. [https://www.reddit.com/r/Bitwarden/comments/13qlg96/you_may_want_to_avoid_windows_hello_if_your/](https://www.reddit.com/r/Bitwarden/comments/13qlg96/you_may_want_to_avoid_windows_hello_if_your/)
5. [https://www.kensington.com/en-ca/news-index---blogs--press-center/security-blog/windows-hello-for-business-what-it-is-how-it-works-and-why-use-it/](https://www.kensington.com/en-ca/news-index---blogs--press-center/security-blog/windows-hello-for-business-what-it-is-how-it-works-and-why-use-it/)
6. [https://blog.elcomsoft.com/2022/08/windows-hello-no-tpm-no-security/](https://blog.elcomsoft.com/2022/08/windows-hello-no-tpm-no-security/)
7. [https://www.elevenforum.com/t/are-windows-hello-passkeys-device-bound-if-you-have-no-tpm.21286/](https://www.elevenforum.com/t/are-windows-hello-passkeys-device-bound-if-you-have-no-tpm.21286/)
8. [https://learn.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/faq](https://learn.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/faq)
9. [https://www.idmanagement.gov/implement/whfb/](https://www.idmanagement.gov/implement/whfb/)