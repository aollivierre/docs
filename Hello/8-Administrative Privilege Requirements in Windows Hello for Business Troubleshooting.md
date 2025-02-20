# Administrative Privilege Requirements in Windows Hello for Business Troubleshooting

Troubleshooting Windows Hello for Business (WHfB) often requires a combination of **administrator** and **user-level** actions, depending on the component being diagnosed. Below is a structured breakdown of common troubleshooting tasks and their privilege requirements:

## PowerShell Commands

## **Device Registration Verification**

- **`dsregcmd /status`**:
    
    - **Run as**: Administrator
        
    - **Reason**: Accesses system-wide Azure AD/AD join status and TPM details1[5](https://serverfault.com/questions/1159162/problems-with-windows-hello-for-business-in-hybrid-cloud-trust-scenario-but-onl).
        

## **TPM Status Checks**

- **`Get-Tpm`**, **`tpm.msc`**:
    
    - **Run as**: Administrator
        
    - **Reason**: TPM management requires elevated privileges to query or reset hardware security modules1[14](https://dirteam.com/sander/2023/02/23/howto-troubleshoot-windows-hello-for-business-hybrid-access/).
        

## **Certificate Management**

- **`certutil -verify`**, **`certutil -DeleteHelloContainer`**:
    
    - **Run as**: Administrator
        
    - **Reason**: Modifies certificate stores and NGC containers, which are system-protected[17](https://learn.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/hello-deployment-issues)[18](https://forum.level1techs.com/t/windows-hello-in-ad-pin-becomes-unavailable-and-other-musings/220881).
        

## **NGC Folder Operations**

- **`Remove-Item -Path C:\...\Ngc -Recurse -Force`**:
    
    - **Run as**: Administrator
        
    - **Reason**: Requires ownership/permissions to modify protected system directories[2](https://superuser.com/questions/1113638/cant-enable-windows-hello-some-settings-are-managed-by-your-organization)[18](https://forum.level1techs.com/t/windows-hello-in-ad-pin-becomes-unavailable-and-other-musings/220881).
        

## **Event Log Analysis**

- **`Get-WinEvent -LogName "Microsoft-Windows-HelloForBusiness/Operational"`**:
    
    - **Run as**: User or Administrator
        
    - **Reason**: User-level logs (e.g., provisioning errors) are accessible to users, but system-wide logs require admin rights[14](https://dirteam.com/sander/2023/02/23/howto-troubleshoot-windows-hello-for-business-hybrid-access/)[17](https://learn.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/hello-deployment-issues).
        

## Diagnostic Tools

## **`dsregcmd` (Device Registration)**

- **Run as**: Administrator
    
- **Reason**: Queries hybrid join status and Azure AD connectivity, which are system-level configurations[5](https://serverfault.com/questions/1159162/problems-with-windows-hello-for-business-in-hybrid-cloud-trust-scenario-but-onl)[17](https://learn.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/hello-deployment-issues).
    

## **`certutil` (Certificate Validation)**

- **Run as**: Administrator
    
- **Reason**: Validates enterprise certificates tied to domain authentication[16](https://learn.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/deploy/on-premises-cert-trust-enroll)[17](https://learn.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/hello-deployment-issues).
    

## Health Check Script

A comprehensive script combining TPM checks, NGC validation, and event analysis **must run as Administrator** to:

1. Access TPM hardware (e.g., `Get-TpmEndorsementKeyInfo`).
    
2. Modify NGC folder permissions.
    
3. Retrieve system-wide event logs[5](https://serverfault.com/questions/1159162/problems-with-windows-hello-for-business-in-hybrid-cloud-trust-scenario-but-onl)[14](https://dirteam.com/sander/2023/02/23/howto-troubleshoot-windows-hello-for-business-hybrid-access/).
    

## Reset Procedures

## **Basic PIN Reset**

- **`dsregcmd /leave` + `dsregcmd /join`**:
    
    - **Run as**: Administrator
        
    - **Reason**: Modifies device registration state in Azure AD/AD[5](https://serverfault.com/questions/1159162/problems-with-windows-hello-for-business-in-hybrid-cloud-trust-scenario-but-onl)[17](https://learn.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/hello-deployment-issues).
        

## **NGC Container Cleanup**

- **`certutil -DeleteHelloContainer`**:
    
    - **Run as**: Administrator
        
    - **Reason**: Deletes cryptographic keys stored in protected system folders[18](https://forum.level1techs.com/t/windows-hello-in-ad-pin-becomes-unavailable-and-other-musings/220881).
        

## **Complete Device Reset**

- **`Clear-Tpm -AllowClear $true`**:
    
    - **Run as**: Administrator
        
    - **Reason**: Resets TPM hardware to factory state[14](https://dirteam.com/sander/2023/02/23/howto-troubleshoot-windows-hello-for-business-hybrid-access/)[18](https://forum.level1techs.com/t/windows-hello-in-ad-pin-becomes-unavailable-and-other-musings/220881).
        

## Error Code Analysis

- **User-Level Errors** (e.g., `0x80070490`):
    
    - **Run as**: User
        
    - **Reason**: Often relate to user-specific provisioning failures (e.g., expired certificates)[17](https://learn.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/hello-deployment-issues).
        
- **System-Level Errors** (e.g., `0x801C03ED`):
    
    - **Run as**: Administrator
        
    - **Reason**: Indicate TPM/NGC corruption or policy misconfigurations[17](https://learn.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/hello-deployment-issues)[18](https://forum.level1techs.com/t/windows-hello-in-ad-pin-becomes-unavailable-and-other-musings/220881).
        

## Best Practices

1. **Standard User Testing**:
    
    - Reproduce issues logged in as the affected user to isolate policy conflicts (e.g., PIN complexity rules)[9](https://www.techtarget.com/searchenterprisedesktop/tip/A-complete-guide-to-troubleshooting-Windows-Hello)[12](https://www.idmanagement.gov/implement/whfb/).
        
2. **Administrative Scripts**:
    
    - Use `Start-Process -Verb RunAs` to elevate PowerShell commands when scripting[3](https://stackoverflow.com/questions/12903629/how-do-i-run-a-program-from-command-prompt-as-a-different-user-and-as-an-admin)[4](https://www.ninjaone.com/blog/open-an-elevated-powershell-prompt/).
        
3. **Event Log Filtering**:
    
    - User logs: Filter `Microsoft-Windows-HelloForBusiness/Operational` by the user’s SID.
        
    - System logs: Use admin privileges to analyze `Microsoft-Windows-User Device Registration/Admin`[14](https://dirteam.com/sander/2023/02/23/howto-troubleshoot-windows-hello-for-business-hybrid-access/)[17](https://learn.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/hello-deployment-issues).
        

## Privilege Exceptions

- **User Certificate Enrollment**:
    
    - Certificates tied to WHfB (e.g., AD CS-issued) can be managed via user-initiated MFA workflows without admin rights[16](https://learn.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/deploy/on-premises-cert-trust-enroll).
        
- **PIN Reset via Entra ID**:
    
    - Users can reset PINs via Entra ID Self-Service Password Reset (SSPR) if configured[6](https://deviceadvice.io/2020/09/17/how-to-disable-the-your-organization-requires-windows-hello-prompt-during-oobe/)[12](https://www.idmanagement.gov/implement/whfb/).
        

## Conclusion

Most WHfB troubleshooting tasks—particularly those involving TPM, NGC, or device registration—**require administrative privileges**. User-level actions are limited to PIN resets and basic log analysis. Always:

1. **Reproduce issues** in both user and admin contexts.
    
2. **Leverage Intune/Group Policy** to enforce TPM requirements and block WHfB on non-compliant devices[6](https://deviceadvice.io/2020/09/17/how-to-disable-the-your-organization-requires-windows-hello-prompt-during-oobe/)[12](https://www.idmanagement.gov/implement/whfb/).
    
3. **Audit registry keys** (e.g., `HKLM\SOFTWARE\Policies\Microsoft\PassportForWork`) to validate policy enforcement[2](https://superuser.com/questions/1113638/cant-enable-windows-hello-some-settings-are-managed-by-your-organization)[5](https://serverfault.com/questions/1159162/problems-with-windows-hello-for-business-in-hybrid-cloud-trust-scenario-but-onl).
    

By adhering to these guidelines, enterprises can streamline WHfB diagnostics while maintaining security boundaries.

### Citations:

1. [https://www.youtube.com/watch?v=5LJIv4-034E](https://www.youtube.com/watch?v=5LJIv4-034E)
2. [https://superuser.com/questions/1113638/cant-enable-windows-hello-some-settings-are-managed-by-your-organization](https://superuser.com/questions/1113638/cant-enable-windows-hello-some-settings-are-managed-by-your-organization)
3. [https://stackoverflow.com/questions/12903629/how-do-i-run-a-program-from-command-prompt-as-a-different-user-and-as-an-admin](https://stackoverflow.com/questions/12903629/how-do-i-run-a-program-from-command-prompt-as-a-different-user-and-as-an-admin)
4. [https://www.ninjaone.com/blog/open-an-elevated-powershell-prompt/](https://www.ninjaone.com/blog/open-an-elevated-powershell-prompt/)
5. [https://serverfault.com/questions/1159162/problems-with-windows-hello-for-business-in-hybrid-cloud-trust-scenario-but-onl](https://serverfault.com/questions/1159162/problems-with-windows-hello-for-business-in-hybrid-cloud-trust-scenario-but-onl)
6. [https://deviceadvice.io/2020/09/17/how-to-disable-the-your-organization-requires-windows-hello-prompt-during-oobe/](https://deviceadvice.io/2020/09/17/how-to-disable-the-your-organization-requires-windows-hello-prompt-during-oobe/)
7. [https://winsides.com/5-ways-to-open-run-msdt-as-administrator-windows-11/](https://winsides.com/5-ways-to-open-run-msdt-as-administrator-windows-11/)
8. [https://superuser.com/questions/661979/run-as-different-user-and-elevate](https://superuser.com/questions/661979/run-as-different-user-and-elevate)
9. [https://www.techtarget.com/searchenterprisedesktop/tip/A-complete-guide-to-troubleshooting-Windows-Hello](https://www.techtarget.com/searchenterprisedesktop/tip/A-complete-guide-to-troubleshooting-Windows-Hello)
10. [https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc771525(v=ws.11)](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc771525\(v=ws.11\))
11. [https://petervanderwoude.nl/post/requiring-the-use-of-windows-hello-for-business-for-interactive-logons/](https://petervanderwoude.nl/post/requiring-the-use-of-windows-hello-for-business-for-interactive-logons/)
12. [https://www.idmanagement.gov/implement/whfb/](https://www.idmanagement.gov/implement/whfb/)
13. [https://community.citrix.com/tech-zone/build/deployment-guides/cwa-windows-hello-sso/](https://community.citrix.com/tech-zone/build/deployment-guides/cwa-windows-hello-sso/)
14. [https://dirteam.com/sander/2023/02/23/howto-troubleshoot-windows-hello-for-business-hybrid-access/](https://dirteam.com/sander/2023/02/23/howto-troubleshoot-windows-hello-for-business-hybrid-access/)
15. [https://itexperience.net/run-as-administrator-how-to/](https://itexperience.net/run-as-administrator-how-to/)
16. [https://learn.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/deploy/on-premises-cert-trust-enroll](https://learn.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/deploy/on-premises-cert-trust-enroll)
17. [https://learn.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/hello-deployment-issues](https://learn.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/hello-deployment-issues)
18. [https://forum.level1techs.com/t/windows-hello-in-ad-pin-becomes-unavailable-and-other-musings/220881](https://forum.level1techs.com/t/windows-hello-in-ad-pin-becomes-unavailable-and-other-musings/220881)
19. [https://learn.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/faq](https://learn.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/faq)
20. [https://www.reddit.com/r/Intune/comments/wj2hqq/windows_hello_enable_admin_rights/](https://www.reddit.com/r/Intune/comments/wj2hqq/windows_hello_enable_admin_rights/)
21. [https://knowledge.digicert.com/solution/integration-for-windows-hello-for-busines-pki-platform](https://knowledge.digicert.com/solution/integration-for-windows-hello-for-busines-pki-platform)
22. [https://learn.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/deploy/](https://learn.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/deploy/)
23. [https://learn.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/configure](https://learn.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/configure)
24. [https://www.gradenegger.eu/en/login-error-with-windows-hello-for-business-contact-the-system-administrator-and-inform-him-that-the-kdc-certificate-could-not-be-verified/](https://www.gradenegger.eu/en/login-error-with-windows-hello-for-business-contact-the-system-administrator-and-inform-him-that-the-kdc-certificate-could-not-be-verified/)
25. [https://www.sevenforums.com/general-discussion/235987-run-cmd-exe-given-user-administrator-command-line-2.html](https://www.sevenforums.com/general-discussion/235987-run-cmd-exe-given-user-administrator-command-line-2.html)
26. [https://www.reddit.com/r/Intune/comments/wicq2o/how_to_enable_windows_hello_for_business_for_a/](https://www.reddit.com/r/Intune/comments/wicq2o/how_to_enable_windows_hello_for_business_for_a/)
27. [https://www.reddit.com/r/Windows10/comments/13jqdab/what_is_the_command_to_change_to_a_non_admin_user/](https://www.reddit.com/r/Windows10/comments/13jqdab/what_is_the_command_to_change_to_a_non_admin_user/)
28. [https://amaxra.com/articles/windows-hello-for-business](https://amaxra.com/articles/windows-hello-for-business)
29. [https://learn.microsoft.com/en-us/answers/questions/1652190/windows-hello-for-business-elevate-to-domain-admin](https://learn.microsoft.com/en-us/answers/questions/1652190/windows-hello-for-business-elevate-to-domain-admin)
30. [https://answers.microsoft.com/en-us/windows/forum/all/windows-11-pro-admin-privileges/de1320ce-fb94-4f55-bad0-6f0c31831edb](https://answers.microsoft.com/en-us/windows/forum/all/windows-11-pro-admin-privileges/de1320ce-fb94-4f55-bad0-6f0c31831edb)
31. [https://h30434.www3.hp.com/t5/Desktop-Software-and-How-To-Questions/Set-HpHwDiag-As-Administrator/td-p/7726785](https://h30434.www3.hp.com/t5/Desktop-Software-and-How-To-Questions/Set-HpHwDiag-As-Administrator/td-p/7726785)
32. [https://www.tenforums.com/user-accounts-family-safety/138005-lost-admin-privileges.html](https://www.tenforums.com/user-accounts-family-safety/138005-lost-admin-privileges.html)
33. [https://www.dell.com/support/kbdoc/en-ca/000193593/how-to-run-sddc-diagnostics-in-powershell-and-windows-admin-center](https://www.dell.com/support/kbdoc/en-ca/000193593/how-to-run-sddc-diagnostics-in-powershell-and-windows-admin-center)
34. [https://www.reddit.com/r/Intune/comments/18wpgur/windows_hello_for_business_prompt_for/](https://www.reddit.com/r/Intune/comments/18wpgur/windows_hello_for_business_prompt_for/)
35. [https://stackoverflow.com/questions/2532769/how-to-start-a-process-as-administrator-mode-in-c-sharp](https://stackoverflow.com/questions/2532769/how-to-start-a-process-as-administrator-mode-in-c-sharp)
36. [https://www.reddit.com/r/Intune/comments/1da4m33/whfb_does_not_work_for_domain_admins_only/](https://www.reddit.com/r/Intune/comments/1da4m33/whfb_does_not_work_for_domain_admins_only/)
37. [https://www.reddit.com/r/sysadmin/comments/s2eqye/windows_hello_for_business_and_mfa_questions/](https://www.reddit.com/r/sysadmin/comments/s2eqye/windows_hello_for_business_and_mfa_questions/)
38. [https://learn.microsoft.com/en-us/answers/questions/642143/your-administrator-or-organization-has-set-the-dia](https://learn.microsoft.com/en-us/answers/questions/642143/your-administrator-or-organization-has-set-the-dia)
39. [https://learn.microsoft.com/en-gb/answers/questions/1655457/windows-hello-for-business-entra-id-sync-issue](https://learn.microsoft.com/en-gb/answers/questions/1655457/windows-hello-for-business-entra-id-sync-issue)
40. [https://answers.microsoft.com/en-us/windows/forum/all/diagnostics/993fba2b-ffda-494b-99f7-770976a8787e](https://answers.microsoft.com/en-us/windows/forum/all/diagnostics/993fba2b-ffda-494b-99f7-770976a8787e)