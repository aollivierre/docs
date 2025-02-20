# Storage Architecture of Windows Hello for Business Credentials: NGC Metadata and TPM Integration

Windows Hello for Business (WHfB) employs a layered storage architecture where **metadata** resides in the **Ngc folder**, while **cryptographic keys** are secured by the **Trusted Platform Module (TPM)** when available. This division ensures both operational functionality and hardware-backed security. Below is a technical breakdown of how WHfB credentials are managed across these components.

## The Role of the NGC Folder

## Metadata Storage

The Ngc folder (`C:\Windows\ServiceProfiles\LocalService\AppData\Local\Microsoft\Ngc`) stores configuration data and encrypted artifacts required for WHfB operations, including:

1. **User SIDs**: Identifiers linking credentials to specific accounts (as seen in `1.dat` files)[6](https://www.forensicfocus.com/forums/general/windows-10-login-pin/)[9](https://blog.matrixpost.net/set-up-windows-hello-for-business-hybrid-azure-ad-joined-devices/).
    
2. **Protectors**: Encrypted data blobs securing PIN-derived keys (via DPAPI-NG or TPM-backed encryption)[2](https://www.synacktiv.com/en/publications/whfb-and-entra-id-say-hello-to-your-new-cache-flow)[7](https://dirkjanm.io/assets/raw/Abusing%20Windows%20Hello%20Without%20a%20Severed%20Hand_v3.pdf).
    
3. **Crypto Provider Metadata**: Information about the key storage provider (e.g., TPM vs. software-based)[4](https://helgeklein.com/blog/checking-windows-hello-for-business-whfb-key-storage-tpm-hardware-or-software/)[9](https://blog.matrixpost.net/set-up-windows-hello-for-business-hybrid-azure-ad-joined-devices/).
    

Example Ngc folder structure:

text

`Ngc   ├── {GUID_1}   │   ├── 1.dat (User SID: S-1-5-21-xxx)   │   ├── 7.dat (Crypto Provider: "Microsoft Platform Crypto Provider")   │   └── Protectors   │       └── {GUID_A}   │           └── 1.dat (Encrypted PIN derivation parameters)   └── {GUID_2}       └── ...`  

## Non-Key Material

Critically, the Ngc folder **does not store raw cryptographic keys**. Instead, it holds:

- **Encrypted PIN data**: Protected via TPM-bound encryption when available[2](https://www.synacktiv.com/en/publications/whfb-and-entra-id-say-hello-to-your-new-cache-flow)[7](https://dirkjanm.io/assets/raw/Abusing%20Windows%20Hello%20Without%20a%20Severed%20Hand_v3.pdf).
    
- **Policy configurations**: Device-specific WHfB settings (e.g., PIN complexity rules)[9](https://blog.matrixpost.net/set-up-windows-hello-for-business-hybrid-azure-ad-joined-devices/).
    

## TPM’s Role in Key Storage

## Hardware-Bound Key Protection

When a TPM is present and functional:

1. **Private Keys**: Generated and stored exclusively within the TPM’s secure hardware[1](https://www.reddit.com/r/eGPU/comments/1da7j6f/egpu_windows_hello_and_tpm/)[4](https://helgeklein.com/blog/checking-windows-hello-for-business-whfb-key-storage-tpm-hardware-or-software/).
    
2. **Key Operations**: Signing/decryption actions occur inside the TPM, preventing key extraction[7](https://dirkjanm.io/assets/raw/Abusing%20Windows%20Hello%20Without%20a%20Severed%20Hand_v3.pdf)[9](https://blog.matrixpost.net/set-up-windows-hello-for-business-hybrid-azure-ad-joined-devices/).
    
3. **NgcKeyImplType**: The `certutil` command reveals storage location:
    
    powershell
    
    `certutil -csp "Microsoft Passport Key Storage Provider" -key -v   # TPM-stored key:   NgcKeyImplType: 1 (0x1)   # Software-stored key:   NgcKeyImplType: 2 (0x2)`  
    
    This output confirms whether keys are TPM-bound (Type 1) or disk-resident (Type 2)[1](https://www.reddit.com/r/eGPU/comments/1da7j6f/egpu_windows_hello_and_tpm/)[4](https://helgeklein.com/blog/checking-windows-hello-for-business-whfb-key-storage-tpm-hardware-or-software/).
    

## TPM-Attested Security

- **Anti-hammering**: TPMs enforce PIN attempt limits, bricking after repeated failures[2](https://www.synacktiv.com/en/publications/whfb-and-entra-id-say-hello-to-your-new-cache-flow)[7](https://dirkjanm.io/assets/raw/Abusing%20Windows%20Hello%20Without%20a%20Severed%20Hand_v3.pdf).
    
- **Secure PCRs**: Platform Configuration Registers ensure keys unlock only when system state matches provisioning conditions[7](https://dirkjanm.io/assets/raw/Abusing%20Windows%20Hello%20Without%20a%20Severed%20Hand_v3.pdf).
    

## Scenarios Without TPM

## Software-Based Key Storage

On devices lacking TPM 2.0 (or with TPM disabled in BIOS):

1. **Keys on Disk**: Private keys encrypt using DPAPI-NG with PIN-derived keys[2](https://www.synacktiv.com/en/publications/whfb-and-entra-id-say-hello-to-your-new-cache-flow)[4](https://helgeklein.com/blog/checking-windows-hello-for-business-whfb-key-storage-tpm-hardware-or-software/).
    
2. **Vulnerability**: Attackers can brute-force PINs offline to decrypt keys (hashcat achieves ~1M guesses/sec)[5](https://hashcat.net/forum/thread-10461.html)[6](https://www.forensicfocus.com/forums/general/windows-10-login-pin/).
    

Example attack workflow:

1. Extract `EncryptedPassword` from registry (`HKLM\SOFTWARE\Microsoft\...\NgcPin\Credentials\<SID>`)[6](https://www.forensicfocus.com/forums/general/windows-10-login-pin/).
    
2. Derive PBKDF2-SHA256 key from candidate PIN.
    
3. Decrypt DPAPI-NG blobs to recover cloud PRT tokens or local credentials[2](https://www.synacktiv.com/en/publications/whfb-and-entra-id-say-hello-to-your-new-cache-flow)[5](https://hashcat.net/forum/thread-10461.html).
    

## Security Implications

## TPM vs. NGC Folder Dependencies

|**Component**|**TPM Present**|**No TPM**|
|---|---|---|
|Private Key Storage|TPM hardware (inaccessible)|Encrypted on disk (vulnerable)|
|PIN Brute-Forcing|TPM locks after 5-10 attempts|Unlimited offline attempts|
|Key Extraction|Physically impossible|Possible via disk/Memory attacks|

## Administrative Best Practices

1. **Enforce TPM 2.0**: Use Intune/Group Policy to block WHfB enrollment on non-TPM devices[9](https://blog.matrixpost.net/set-up-windows-hello-for-business-hybrid-azure-ad-joined-devices/).
    
2. **Monitor Ngc Folder**: Audit permissions to prevent unauthorized access (SYSTEM/admin only)[7](https://dirkjanm.io/assets/raw/Abusing%20Windows%20Hello%20Without%20a%20Severed%20Hand_v3.pdf)[8](https://www.isunshare.com/windows-10/fixed-windows-10-pin-setup-and-login-not-working.html).
    
3. **Rotate Keys Periodically**: Force WHfB re-enrollment via PowerShell:
    
    powershell
    
    `Get-WindowsHelloContainer -User <SID> | Remove-WindowsHelloContainer`  
    

## Troubleshooting Insights

## When Deleting the NGC Folder Helps

Resetting WHfB often requires Ngc folder deletion to clear corrupted metadata:

powershell

`takeown /f C:\Windows\...\Ngc /r /d y   icacls C:\Windows\...\Ngc /grant administrators:F /t   Remove-Item -Path C:\Windows\...\Ngc -Recurse -Force`  

However, this **does not affect TPM-stored keys**—re-enrollment generates new keys[3](https://superuser.com/questions/1757603/unable-to-sign-in-to-windows-11-due-to-failed-sign-in-attempts)[8](https://www.isunshare.com/windows-10/fixed-windows-10-pin-setup-and-login-not-working.html).

## TPM-Specific Errors

- **eGPU Conflicts**: Connecting external GPUs may reset TPM measurements, invalidating WHfB keys[1](https://www.reddit.com/r/eGPU/comments/1da7j6f/egpu_windows_hello_and_tpm/).
    
- **Firmware Updates**: TPM firmware patches (e.g., addressing ROCA vulnerability) can orphan keys, requiring re-enrollment[1](https://www.reddit.com/r/eGPU/comments/1da7j6f/egpu_windows_hello_and_tpm/)[9](https://blog.matrixpost.net/set-up-windows-hello-for-business-hybrid-azure-ad-joined-devices/).
    

## Conclusion

WHfB credentials rely on a symbiotic relationship between the **Ngc folder** (metadata/encrypted parameters) and **TPM** (hardware-secured keys). While the Ngc folder is essential for operational data, the actual cryptographic keys either reside in the TPM (preferred) or on disk (less secure). Enterprises must mandate TPM usage and regularly audit Ngc permissions to maintain robust authentication hygiene.

Future developments in **Cloud Kerberos Trust** aim to further decouple WHfB from on-premises dependencies, but TPM remains critical for endpoint security in hybrid environments.

### Citations:

1. [https://www.reddit.com/r/eGPU/comments/1da7j6f/egpu_windows_hello_and_tpm/](https://www.reddit.com/r/eGPU/comments/1da7j6f/egpu_windows_hello_and_tpm/)
2. [https://www.synacktiv.com/en/publications/whfb-and-entra-id-say-hello-to-your-new-cache-flow](https://www.synacktiv.com/en/publications/whfb-and-entra-id-say-hello-to-your-new-cache-flow)
3. [https://superuser.com/questions/1757603/unable-to-sign-in-to-windows-11-due-to-failed-sign-in-attempts](https://superuser.com/questions/1757603/unable-to-sign-in-to-windows-11-due-to-failed-sign-in-attempts)
4. [https://helgeklein.com/blog/checking-windows-hello-for-business-whfb-key-storage-tpm-hardware-or-software/](https://helgeklein.com/blog/checking-windows-hello-for-business-whfb-key-storage-tpm-hardware-or-software/)
5. [https://hashcat.net/forum/thread-10461.html](https://hashcat.net/forum/thread-10461.html)
6. [https://www.forensicfocus.com/forums/general/windows-10-login-pin/](https://www.forensicfocus.com/forums/general/windows-10-login-pin/)
7. [https://dirkjanm.io/assets/raw/Abusing%20Windows%20Hello%20Without%20a%20Severed%20Hand_v3.pdf](https://dirkjanm.io/assets/raw/Abusing%20Windows%20Hello%20Without%20a%20Severed%20Hand_v3.pdf)
8. [https://www.isunshare.com/windows-10/fixed-windows-10-pin-setup-and-login-not-working.html](https://www.isunshare.com/windows-10/fixed-windows-10-pin-setup-and-login-not-working.html)
9. [https://blog.matrixpost.net/set-up-windows-hello-for-business-hybrid-azure-ad-joined-devices/](https://blog.matrixpost.net/set-up-windows-hello-for-business-hybrid-azure-ad-joined-devices/)
10. [https://h30434.www3.hp.com/t5/Notebook-Operating-System-and-Recovery/Validity-Fingerprint-Sensor-no-longer-fuctioning-with/td-p/6224464](https://h30434.www3.hp.com/t5/Notebook-Operating-System-and-Recovery/Validity-Fingerprint-Sensor-no-longer-fuctioning-with/td-p/6224464)
11. [https://www.reddit.com/r/Intune/comments/kx9p0b/windows_hello_for_business_this_option_is/](https://www.reddit.com/r/Intune/comments/kx9p0b/windows_hello_for_business_this_option_is/)
12. [https://learn.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/faq](https://learn.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/faq)
13. [https://answers.microsoft.com/en-us/windows/forum/all/tpm-full/e2fa4ad7-cc7c-427f-b63c-204928564383](https://answers.microsoft.com/en-us/windows/forum/all/tpm-full/e2fa4ad7-cc7c-427f-b63c-204928564383)
14. [https://forum.level1techs.com/t/windows-hello-in-ad-pin-becomes-unavailable-and-other-musings/220881](https://forum.level1techs.com/t/windows-hello-in-ad-pin-becomes-unavailable-and-other-musings/220881)
15. [https://www.stellarinfo.com/blog/your-pin-not-available-windows-11/](https://www.stellarinfo.com/blog/your-pin-not-available-windows-11/)
16. [https://www.mdpi.com/2410-387X/7/1/9](https://www.mdpi.com/2410-387X/7/1/9)
17. [https://hit.skku.edu/?page_id=2233](https://hit.skku.edu/?page_id=2233)
18. [https://www.youtube.com/watch?v=v-E0Xe9LM4E](https://www.youtube.com/watch?v=v-E0Xe9LM4E)
19. [https://www.thewindowsclub.com/windows-hello-for-business-setup-for-pin-0x80040154](https://www.thewindowsclub.com/windows-hello-for-business-setup-for-pin-0x80040154)
20. [https://www.kensington.com/en-ca/news-index---blogs--press-center/security-blog/windows-hello-for-business-what-it-is-how-it-works-and-why-use-it/](https://www.kensington.com/en-ca/news-index---blogs--press-center/security-blog/windows-hello-for-business-what-it-is-how-it-works-and-why-use-it/)
21. [https://www.samuraj-cz.com/en/article/windows-hello-for-business-cloud-kerberos-trust-deployment/](https://www.samuraj-cz.com/en/article/windows-hello-for-business-cloud-kerberos-trust-deployment/)
22. [https://msendpointmgr.com/2023/03/04/cloud-kerberos-trust-part-1/](https://msendpointmgr.com/2023/03/04/cloud-kerberos-trust-part-1/)
23. [https://github.com/MicrosoftDocs/SupportArticles-docs/blob/main/support/windows-client/user-profiles-and-logon/windows-hello-errors-during-pin-creation-in-windows-10.md](https://github.com/MicrosoftDocs/SupportArticles-docs/blob/main/support/windows-client/user-profiles-and-logon/windows-hello-errors-during-pin-creation-in-windows-10.md)
24. [https://www.reddit.com/r/windows/comments/117imrh/does_anyone_know_the_fix_for_this/](https://www.reddit.com/r/windows/comments/117imrh/does_anyone_know_the_fix_for_this/)
25. [https://dirkjanm.io/assets/raw/Windows%20Hello%20from%20the%20other%20side_nsec_v1.0.pdf](https://dirkjanm.io/assets/raw/Windows%20Hello%20from%20the%20other%20side_nsec_v1.0.pdf)
26. [https://www.youtube.com/watch?v=u22XC01ewn0](https://www.youtube.com/watch?v=u22XC01ewn0)