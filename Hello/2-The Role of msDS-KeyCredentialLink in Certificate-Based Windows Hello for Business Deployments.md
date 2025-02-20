# The Role of msDS-KeyCredentialLink in Certificate-Based Windows Hello for Business Deployments  

Windows Hello for Business (WHfB) offers two primary authentication models: **key trust** and **certificate trust**. The `msDS-KeyCredentialLink` attribute plays a critical role in key trust deployments but remains largely irrelevant in certificate-based environments. This report examines the technical relationship between these authentication methods and the attribute’s behavior, providing clarity on its usage across deployment models.  

---

## Authentication Models in Windows Hello for Business  

### Key Trust Authentication  
In key trust deployments, WHfB generates an asymmetric key pair (RSA 2048-bit) stored in the **Ngc folder** and protected by the Trusted Platform Module (TPM). The public key synchronizes from Microsoft Entra ID (Azure AD) to on-premises Active Directory via Azure AD Connect, populating the `msDS-KeyCredentialLink` attribute of the user object. This attribute enables Kerberos authentication by allowing domain controllers to verify the client’s public key during PKINIT exchanges[1][5][12].  

Common scenarios requiring `msDS-KeyCredentialLink`:  
1. **Hybrid key trust**: Azure AD Connect synchronizes the public key to on-premises AD, enabling seamless SSO to on-premises resources[7][8].  
2. **Shadow credential attacks**: Attackers exploit write permissions to this attribute to implant rogue keys, granting persistent access[2][11].  

### Certificate Trust Authentication  
Certificate trust deployments rely on certificates issued by an enterprise certificate authority (CA) for authentication. Unlike key trust:  
- Certificates are provisioned via **Active Directory Certificate Services (AD CS)**.  
- Authentication uses the certificate’s private key (TPM-protected) rather than an asymmetric key pair[8][15].  
- The `msDS-KeyCredentialLink` attribute remains **unused** because certificates do not require synchronization of public keys to this attribute[13][15].  

---

## Analyzing the Statement: *“We Don’t Use Keys; We’re Certificate-Based, So That Attribute Will Be Empty”**  

### Validity in Pure Certificate Trust Deployments  
The assertion holds true in environments **exclusively** using certificate trust:  
1. **No synchronization of public keys**: Since certificate trust relies on AD CS-issued certificates, Azure AD Connect does not populate `msDS-KeyCredentialLink` with key material[8][15].  
2. **Attribute remains empty**: Querying the attribute via PowerShell (`Get-ADUser -Properties msDS-KeyCredentialLink`) returns no entries for users authenticating via certificates[6][13].  

**Example**:  
```powershell  
# Certificate trust user  
Get-ADUser "UserA" -Properties msDS-KeyCredentialLink | Select-Object msDS-KeyCredentialLink  
# Output:   
```

### Exceptions and Edge Cases  
1. **Hybrid deployments with both models**: If an organization transitions from key trust to certificate trust, residual entries in `msDS-KeyCredentialLink` may persist until manually cleared[6][14].  
2. **Misconfigured synchronization**: Incorrect Azure AD Connect rules might erroneously synchronize certificate public keys to the attribute, though this does not impact certificate trust authentication[4][13].  

---

## Security and Operational Implications  

### Risks of an Empty msDS-KeyCredentialLink in Certificate Trust  
While the attribute’s emptiness is expected, administrators should:  
1. **Audit permissions**: Ensure no unauthorized accounts (e.g., compromised users in the **Key Admins** group) retain write access to `msDS-KeyCredentialLink`, preventing shadow credential attacks[3][11].  
2. **Monitor synchronization**: Validate Azure AD Connect rules to prevent unintended synchronization of certificate-related data to the attribute[4][8].  

### Troubleshooting Certificate Trust Without msDS-KeyCredentialLink  
Common certificate trust issues unrelated to the attribute include:  
- **Expired certificates**: Use `certutil -verify` to check certificate validity.  
- **Misconfigured certificate templates**: Ensure AD CS templates include the **Smart Card Logon** extended key usage (EKU)[15].  
- **TPM provisioning failures**: Diagnose with `Get-TpmEndorsementKeyInfo` and Event Viewer logs (`Microsoft-Windows-HelloForBusiness/Operational`)[5][8].  

---

## Comparative Analysis: Key Trust vs. Certificate Trust  

| **Criteria**               | **Key Trust**                                  | **Certificate Trust**                          |  
|----------------------------|-----------------------------------------------|------------------------------------------------|  
| **Authentication mechanism** | Asymmetric keys in `msDS-KeyCredentialLink` | AD CS-issued certificates                      |  
| **Domain Controller OS**   | Requires Windows Server 2016 or later[5][12] | Compatible with legacy Server OS (e.g., 2012R2)|  
| **PKI dependency**          | No (uses Azure AD-synced keys)               | Yes (requires AD CS or third-party CA)        |  
| **Attribute usage**         | `msDS-KeyCredentialLink` populated           | Attribute unused/empty                        |  
| **Attack surface**           | Vulnerable to shadow credential attacks[11]  | Vulnerable to certificate theft/forgery        |  

---

## Recommendations for Certificate Trust Environments  

1. **Disable unnecessary synchronization**: Modify Azure AD Connect rules to exclude `msDS-KeyCredentialLink` if migrating from key trust[13][15].  
2. **Regularly audit AD attributes**: Use PowerShell to identify residual entries:  
   ```powershell  
   Get-ADUser -Filter * -Properties msDS-KeyCredentialLink | Where-Object {$_.'msDS-KeyCredentialLink'}  
   ```
3. **Enforce TPM-bound certificates**: Configure AD CS templates to require TPM attestation, reducing private key extraction risks[8][15].  

---

## Conclusion  

In pure certificate trust deployments, the `msDS-KeyCredentialLink` attribute remains empty, as synchronization of asymmetric keys is unnecessary. However, organizations must ensure proper configuration of Azure AD Connect and AD CS to avoid unintended interactions between authentication models. By adhering to certificate trust best practices—such as TPM enforcement and PKI hygiene—enterprises can mitigate risks while maintaining compatibility with legacy infrastructure.  

Future considerations include monitoring Microsoft’s evolving documentation for changes in attribute behavior and automating audits of Active Directory permissions to prevent credential-based attacks.

Citations:
[1] https://brookspeppin.com/2021/08/13/how-to-setup-windows-hello-for-business-key-trust-method/
[2] https://cyberstoph.org/posts/2022/03/detecting-shadow-credentials/
[3] https://pentestlab.blog/tag/msds-keycredentiallink/
[4] https://www.reddit.com/r/sysadmin/comments/i4jyy7/whfb_msdskeycredential_attribute_issue/
[5] https://learn.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/hello-deployment-issues
[6] https://www.reddit.com/r/activedirectory/comments/13m1kok/how_to_remove_msdskeycredentiallink_value/
[7] https://stephanwaelde.com/2020/02/24/azure-ad-join-single-sign-on-follow-the-key/
[8] https://blog.matrixpost.net/set-up-windows-hello-for-business-hybrid-azure-ad-joined-devices/
[9] https://cloudbrothers.info/en/windows-business-cloud-trust-kdc-proxy/
[10] https://github.com/MicrosoftDocs/windows-itpro-docs/blob/public/windows/security/identity-protection/hello-for-business/hello-deployment-issues.md
[11] https://posts.specterops.io/shadow-credentials-abusing-key-trust-account-mapping-for-takeover-8ee1a53566ab
[12] https://dirteam.com/sander/2022/09/16/why-everyones-talking-about-hybrid-cloud-trust/
[13] https://ericonidentity.com/2022/03/23/windows-hello-for-business-hybrid-cloud-trust/
[14] https://jairocadena.com/2018/04/02/windows-hello-for-business-registration-and-authentication-with-azuread/
[15] https://github.com/MicrosoftDocs/windows-itpro-docs/blob/public/windows/security/identity-protection/hello-for-business/deploy/hybrid-key-trust.md
[16] https://identity-man.eu/2020/02/13/password-less-3-of-5-going-password-less-with-windows-hello-for-business-hybrid/
[17] https://learn.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/deploy/hybrid-cert-trust
[18] https://argonsys.com/microsoft-cloud/library/azure-ad-mailbag-windows-hello-for-business/
[19] https://msendpointmgr.com/2023/03/04/cloud-kerberos-trust-part-1/
[20] https://detection.fyi/elastic/detection-rules/windows/credential_access_shadow_credentials/
[21] https://techcommunity.microsoft.com/blog/microsoft-entra-blog/azure-ad-mailbag-windows-hello-for-business/445349
[22] https://learn.microsoft.com/en-us/answers/questions/206785/windows-hello-hybrid-azure-ad-joined-with-key-trus
[23] https://cloudbrothers.info/en/going-passwordless-whfb-scril/
[24] https://katystech.blog/azure/azure-ad-and-windows-hello-sso-to-on-premise-resources
[25] https://identity-man.eu/2022/02/17/improving-your-windows-hello-for-business-hybrid-password-less-setup-by-using-cloud-trust/
[26] https://lazyadmin.nl/it/windows-hello-for-business-cloud-trust/
[27] https://www.linkedin.com/pulse/journey-passwordless-experience-windows-hello-business-christophe-d--4ivye
[28] https://www.dsinternals.com/assets/documents/eu-19-Grafnetter-Exploiting-Windows-Hello-for-Business.pdf
[29] https://msendpointmgr.com/2023/03/04/cloud-kerberos-trust-part-3/
[30] https://serverfault.com/questions/1029701/disabling-synchronization-rule-out-to-ad-user-ngckey-in-azuread-connect