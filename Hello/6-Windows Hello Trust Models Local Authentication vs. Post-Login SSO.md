# Windows Hello Trust Models: Local Authentication vs. Post-Login SSO

Windows Hello for Business (WHfB) separates **local device authentication** (PIN/biometric login) from **post-login SSO workflows** (accessing on-premises resources). The trust models—**certificate trust**, **key trust**, and **cloud Kerberos trust**—govern how authentication to **on-premises applications** occurs after the user unlocks their device.

## Local Authentication: NGC and TPM-Centric Process

## Windows Login Experience

WHfB credentials (PIN/biometrics) are validated locally through:

1. **NGC Folder** (`C:\Windows\...\Microsoft\Ngc`): Stores encrypted metadata (user SIDs, policy configurations) but **not cryptographic keys**.
    
2. **TPM**: Secures the private key used for authentication. The TPM ensures:
    
    - Anti-hammering protections (lockouts after failed attempts).
        
    - Hardware-bound key storage (keys never leave the TPM).
        

**Login Flow**:

- User enters PIN → TPM decrypts the private key → Signs authentication challenge → Grants local access.
    
- **No trust model involvement**: This process is identical regardless of certificate/key/cloud trust configurations.
    

## Post-Login SSO: Role of Trust Models

## Trust Models Define On-Premises Authentication

After local login, trust models determine how WHfB authenticates to **on-premises Active Directory resources**:

|**Trust Model**|**SSO Mechanism**|**Key Dependency**|
|---|---|---|
|**Key Trust**|Uses asymmetric key pair (public key in `msDS-KeyCredentialLink`) for Kerberos TGTs.|Azure AD Connect syncs public keys to AD.|
|**Certificate Trust**|AD CS-issued certificates authenticate to AD via PKINIT.|PKI infrastructure (AD CS, CRLs).|
|**Cloud Kerberos Trust**|Entra ID issues "cloud TGTs" using Microsoft Entra Kerberos.|No PKI; relies on Entra ID/AD trust chain.|

**Example Workflow (Cloud Kerberos Trust)**:

1. User accesses `\\fileserver\share` after logging in with WHfB.
    
2. Device requests a TGT from Entra ID (using cloud Kerberos trust).
    
3. Entra ID validates user identity and issues a partial TGT.
    
4. On-premises domain controller authorizes access using the TGT.
    

## Why Trust Models Don’t Affect Local Login

- **Local Authentication** relies on **TPM/Ngc** infrastructure, which operates independently of domain trust configurations.
    
- **Trust models** are only invoked when accessing resources requiring Kerberos/NTLM authentication (e.g., file shares, legacy apps).
    

## Common Misconceptions Clarified

## Myth: "Cloud Trust Changes How Users Log In"

**Reality**:

- Cloud Kerberos trust affects **SSO to on-premises resources**, not the login screen. Users still unlock devices via PIN/biometrics stored in TPM/Ngc.
    

## Myth: "Certificate Trust Requires Certificates for Local Login"

**Reality**:

- Certificates are used **only** for authenticating to AD post-login. The local login still uses TPM-backed WHfB keys.
    

## Technical Validation

## Evidence from Search Results

1. **Search Result 1 (Microsoft Learn)**:
    
    > _"The trust type defines how Windows Hello for Business clients authenticate to Active Directory [...] not applicable to cloud-only deployments."_
    
    - Confirms trust models govern **AD authentication**, not local login.
        
2. **Search Result 5 (MSEndpointMgr)**:
    
    > _"Cloud Kerberos Trust [...] simplifies hybrid authentication by eliminating PKI dependencies [...] for SSO to domain resources."_
    
    - Highlights SSO focus, not local authentication.
        
3. **Search Result 9 (KatysTech)**:
    
    > _"WHfB keys aren’t present in AD [...] domain controllers won’t validate WHfB logins without trust configurations."_
    
    - Explains why trust models are needed **after** local authentication.
        

## Best Practices for Administrators

1. **Enforce TPM 2.0**: Mandate TPM usage via Intune/GPO to secure local credentials.
    
2. **Audit NGC Permissions**: Restrict access to `C:\Windows\...\Ngc` to SYSTEM/admins.
    
3. **Prefer Cloud Kerberos Trust**: Simplifies SSO without PKI and reduces sync delays.
    

## Conclusion

**Local Login**:

- Handled by TPM/Ngc infrastructure (device-bound keys, PIN/biometrics).
    
- **Unaffected** by certificate/key/cloud trust models.
    

**Post-Login SSO**:

- Trust models define how WHfB authenticates to on-premises AD:
    
    - **Key Trust**: Syncs public keys via Azure AD Connect.
        
    - **Certificate Trust**: Uses AD CS certificates.
        
    - **Cloud Trust**: Leverages Entra-issued TGTs.
        

Enterprises should adopt **Cloud Kerberos Trust** for modern hybrid environments, reserving certificate/key trust for legacy use cases requiring PKI.

### Citations:

1. [https://learn.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/deploy/](https://learn.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/deploy/)
2. [https://infusedinnovations.com/blog/secure-intelligent-workplace/windows-hello-for-business-cloud-trust-in-preview](https://infusedinnovations.com/blog/secure-intelligent-workplace/windows-hello-for-business-cloud-trust-in-preview)
3. [https://blog.matrixpost.net/set-up-windows-hello-for-business-hybrid-azure-ad-joined-devices/](https://blog.matrixpost.net/set-up-windows-hello-for-business-hybrid-azure-ad-joined-devices/)
4. [https://www.kensington.com/en-ca/news-index---blogs--press-center/security-blog/windows-hello-for-business-what-it-is-how-it-works-and-why-use-it/](https://www.kensington.com/en-ca/news-index---blogs--press-center/security-blog/windows-hello-for-business-what-it-is-how-it-works-and-why-use-it/)
5. [https://msendpointmgr.com/2023/03/04/cloud-kerberos-trust-part-1/](https://msendpointmgr.com/2023/03/04/cloud-kerberos-trust-part-1/)
6. [https://learn.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/deploy/hybrid-cloud-kerberos-trust](https://learn.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/deploy/hybrid-cloud-kerberos-trust)
7. [https://intunestuff.com/2025/01/24/cloud-kerberos-trust-wfhb-intune/](https://intunestuff.com/2025/01/24/cloud-kerberos-trust-wfhb-intune/)
8. [https://dirkjanm.io/assets/raw/Windows%20Hello%20from%20the%20other%20side_TR23_final.pdf](https://dirkjanm.io/assets/raw/Windows%20Hello%20from%20the%20other%20side_TR23_final.pdf)
9. [https://katystech.blog/azure/azure-ad-and-windows-hello-sso-to-on-premise-resources](https://katystech.blog/azure/azure-ad-and-windows-hello-sso-to-on-premise-resources)
10. [https://www.reddit.com/r/Intune/comments/1fgzxn5/windows_hello_for_business_cloud_kerberos_trust/](https://www.reddit.com/r/Intune/comments/1fgzxn5/windows_hello_for_business_cloud_kerberos_trust/)
11. [https://hideez.com/blogs/news/windows-hello-for-business](https://hideez.com/blogs/news/windows-hello-for-business)
12. [https://msendpointmgr.com/2023/03/04/cloud-kerberos-trust-part-3/](https://msendpointmgr.com/2023/03/04/cloud-kerberos-trust-part-3/)
13. [https://amaxra.com/articles/windows-hello-for-business](https://amaxra.com/articles/windows-hello-for-business)
14. [https://www.computerworld.com/article/1712315/windows-hello-for-business-passwordless-authentication-for-windows-shops.html](https://www.computerworld.com/article/1712315/windows-hello-for-business-passwordless-authentication-for-windows-shops.html)
15. [https://blog.thomasmarcussen.com/setting-up-windows-hello-cloud-kerberos-trust/](https://blog.thomasmarcussen.com/setting-up-windows-hello-cloud-kerberos-trust/)
16. [https://blog.auth360.net/tag/windows-hello-for-business/](https://blog.auth360.net/tag/windows-hello-for-business/)
17. [https://www.reddit.com/r/Intune/comments/10dm9i6/windows_hello_for_business_do_you_need_a_trust/](https://www.reddit.com/r/Intune/comments/10dm9i6/windows_hello_for_business_do_you_need_a_trust/)
18. [https://petervanderwoude.nl/post/configuring-windows-hello-for-business-cloud-kerberos-trust/](https://petervanderwoude.nl/post/configuring-windows-hello-for-business-cloud-kerberos-trust/)
19. [https://learn.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/faq](https://learn.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/faq)
20. [https://www.encryptionconsulting.com/education-center/windows-hello-for-business-deployment-models/](https://www.encryptionconsulting.com/education-center/windows-hello-for-business-deployment-models/)
21. [https://community.citrix.com/tech-zone/build/deployment-guides/cwa-windows-hello-sso/](https://community.citrix.com/tech-zone/build/deployment-guides/cwa-windows-hello-sso/)
22. [https://learn.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/deploy/hybrid-cert-trust](https://learn.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/deploy/hybrid-cert-trust)
23. [https://learn.microsoft.com/en-us/answers/questions/1294301/how-to-properly-configure-sso-using-whfb-hybrid-ke](https://learn.microsoft.com/en-us/answers/questions/1294301/how-to-properly-configure-sso-using-whfb-hybrid-ke)
24. [https://petri.com/windows-hello-for-business-single-sign-on/](https://petri.com/windows-hello-for-business-single-sign-on/)
25. [https://identity-man.eu/2022/02/17/improving-your-windows-hello-for-business-hybrid-password-less-setup-by-using-cloud-trust/](https://identity-man.eu/2022/02/17/improving-your-windows-hello-for-business-hybrid-password-less-setup-by-using-cloud-trust/)
26. [https://lazyadmin.nl/it/windows-hello-for-business-cloud-trust/](https://lazyadmin.nl/it/windows-hello-for-business-cloud-trust/)
27. [https://mjshellenberger.com/2023/07/03/how-to-guide-deploying-windows-hello-for-business-cloud-kerberos-trust/](https://mjshellenberger.com/2023/07/03/how-to-guide-deploying-windows-hello-for-business-cloud-kerberos-trust/)
28. [https://www.samuraj-cz.com/en/article/windows-hello-for-business-cloud-kerberos-trust-deployment/](https://www.samuraj-cz.com/en/article/windows-hello-for-business-cloud-kerberos-trust-deployment/)
29. [https://jairocadena.com/2016/01/18/how-domain-join-is-different-in-windows-10-with-azure-ad/](https://jairocadena.com/2016/01/18/how-domain-join-is-different-in-windows-10-with-azure-ad/)
30. [https://www.reddit.com/r/sysadmin/comments/p7bmvs/windows_hello_cloud_trust_any_insiders_have_more/](https://www.reddit.com/r/sysadmin/comments/p7bmvs/windows_hello_cloud_trust_any_insiders_have_more/)