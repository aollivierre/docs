> **Disclaimer:** This content was created with the assistance of Generative AI. While efforts have been made to ensure accuracy and relevance, users are advised to review and verify the information provided according to their specific needs and context.

> **Disclaimer:** The 2024 approach discussed in this article is currently available only for Windows 11 Insider Canary builds. It is not recommended for production environments. Please evaluate the stability and security implications before deploying this method in any critical systems.


### Table of Contents

1. [Introduction](#1-introduction)
    
    -   Overview of LAPS
    -   Importance of Secure Local Admin Account Management

2. [2020 Approach vs. 2024 Approach](#2-2020-approach-vs-2024-approach)
    
    -   Manual Account Creation using CSP (2020)
    -   Automated Account Management with LAPS CSP (2024) _Windows 11 (Insider Canary)_
    -   Key Differences and Use Cases

3. [Detailed Implementation Guide](#3-detailed-implementation-guide)
    
    -   2020 Approach
        -   Step-by-Step Configuration in Intune
        -   OMA-URI Settings for Generic Local Admin Account Creation
        -   Assigning to Devices
    -   2024 Approach Windows 11 (Insider Canary)
        -   Step-by-Step Configuration in Intune
        -   OMA-URI Settings for Automated Local Admin Account Creation
        -   Assigning to Devices
        -   **Disclaimer:** This approach is currently available only in the Windows 11 Insider Canary Channel and not suitable for production environments.

4. [Best Practices](#4-best-practices)
    
    -   Naming Conventions for Policies
    -   Security Considerations
    -   Scope Limitation with RBAC Roles

5. [FAQs](#5-faqs)
    
    -   Can I specify the exact account name in the 2024 approach?
    -   How does the LAPS policy manage and auto-rotate passwords?
    -   Does this approach work on both Windows 10 and Windows 11?
    -   Should I assign the policy to users or devices?
    -   What is the difference between prefixing and randomizing account names?

6. [Credits and References](#6-credits-and-references)
    
    -   Placeholder for Original Articles
    -   Acknowledgment of Authors

7. [Conclusion](#7-conclusion)
    
    -   Summary of Approaches
    -   Recommendations for Implementing LAPS in Production Environments




## 1. Introduction

### Overview of LAPS

The Local Administrator Password Solution (LAPS) is a Microsoft feature that automatically manages and rotates the passwords of local administrator accounts on Windows devices. LAPS provides an added layer of security by ensuring that each device has a unique, regularly updated password for its local admin account, reducing the risk of lateral movement by attackers within a network.

### Importance of Secure Local Admin Account Management

Local administrator accounts are critical for managing Windows devices, but they also pose a significant security risk if not properly managed. Using a common password across multiple devices or neglecting to update passwords regularly can lead to vulnerabilities. LAPS addresses these concerns by automating the management and rotation of local admin passwords, helping to protect against unauthorized access and potential attacks.



### 2. 2020 Approach vs. 2024 Approach

#### Manual Account Creation using CSP (2020)

In 2020, managing local administrator accounts on Windows devices via Microsoft Intune involved a more manual approach using Configuration Service Provider (CSP) policies. This method required the administrator to specify each detail, such as the account name, password, and group membership, through custom OMA-URI settings in Intune. While effective, this approach required careful configuration and ongoing management to ensure security.

_Credit for this approach: https://inthecloud247.com/create-a-local-user-account-on-windows-10-with-microsoft-intune/

#### Automated Account Management with LAPS CSP (2024) _*Windows 11 Insider Canary*_

The 2024 approach introduces a more automated method of managing local administrator accounts using the LAPS CSP. Available only in the Windows 11 Insider Canary Channel, this method allows administrators to automate the creation, management, and rotation of local admin account passwords. With LAPS CSP, administrators can set up policies that automatically generate secure local admin accounts, manage their passwords, and enforce password complexity, significantly reducing manual overhead.

**Disclaimer:** The 2024 approach is currently available only in the Windows 11 Insider Canary Channel and is not recommended for production environments. Administrators should carefully evaluate the stability and security implications before deploying this method.

_Credit for this approach: https://ourcloudnetwork.com/how-to-enable-automatic-account-creation-with-laps-in-intune/

#### Key Differences and Use Cases

-   **Manual vs. Automated:** The 2020 approach is manual, requiring specific configuration for each account, while the 2024 approach automates account creation and password management.
-   **Flexibility:** The 2020 method allows for detailed customization of the account name and properties, whereas the 2024 approach focuses on automation and may use randomized account names.
-   **Availability:** The 2020 approach is suitable for all production environments, while the 2024 approach is limited to the Windows 11 Insider Canary Channel.








The main difference in the OMA-URI between the 2020 article by Peter Klapwijk and the 2024 article by Daniel Bradley lies in the specific CSPs (Configuration Service Providers) being used to manage different aspects of local account creation and management on Windows devices.

### 2020 Article:

-   **OMA-URI:**
    
    -   `./Device/Vendor/MSFT/Accounts/Users/LocalUser/Password`
    -   `./Device/Vendor/MSFT/Accounts/Users/LocalUser/LocalUserGroup`
-   **Purpose:**
    
    -   The OMA-URI in this context is used to manually create a local user account and optionally assign it to the local administrators group. The process requires setting the password and specifying the group membership directly in the custom profile.

### 2024 Article:

-   **OMA-URI:**
    
    -   `./Device/Vendor/MSFT/LAPS/Policies/AutomaticAccountManagementEnableAccount`
    -   `./Device/Vendor/MSFT/LAPS/Policies/AutomaticAccountManagementEnabled`
    -   `./Device/Vendor/MSFT/LAPS/Policies/AutomaticAccountManagementNameOrPrefix`
    -   `./Device/Vendor/MSFT/LAPS/Policies/AutomaticAccountManagementRandomizeName`
    -   `./Device/Vendor/MSFT/LAPS/Policies/AutomaticAccountManagementTarget`
    -   `./Device/Vendor/MSFT/LAPS/Policies/PasswordComplexity`
-   **Purpose:**
    
    -   The OMA-URI settings are part of the Windows LAPS (Local Administrator Password Solution) CSP and are used to automatically create and manage a local administrator account, including setting policies for automatic account creation, name randomization, and password complexity. This process is more automated and integrated within the Windows LAPS framework, reducing the need for manual configuration.

### Summary:

-   **2020 Approach:** Focused on manually setting up a local user account using the Accounts CSP with basic configurations.
-   **2024 Approach:** Leverages the advanced capabilities of Windows LAPS to automate the creation and management of local admin accounts with additional options for name randomization, password complexity, and automatic account management.










Here's a table that compares the two approaches:

| **Aspect**                      | **2020 Approach**                                                                 | **2024 Approach** (Windows 11 Insider Preivew)                                               |
|---------------------------------|-----------------------------------------------------------------------------------|-----------------------------------------------------------------------------------|
| **Method**                      | Manual creation using the Accounts CSP via OMA-URI                                | Automated creation using LAPS CSP via OMA-URI                                     |
| **OMA-URI**                     | `./Device/Vendor/MSFT/Accounts/Users/{UserName}/LocalUserGroup`                   | Various LAPS settings, e.g., `./Device/Vendor/MSFT/LAPS/Policies/AutomaticAccountManagementEnableAccount` |
| **Process**                     | - Admin manually specifies username, password, and group membership.<br>- Requires creating a custom configuration profile in Intune. | - LAPS automates account creation and management.<br>- Configures password complexity, name randomization, and account management. |
| **Customization**               | Direct input required for each local account (username, password, admin rights).  | Automated account management with predefined settings in LAPS CSP policies.      |
| **Automation**                  | Low: Manual setup required for each account.                                      | High: Automatic account creation and management via LAPS.                         |
| **Password Management**         | Manual entry of password.                                                         | Automated password complexity and management via LAPS.                            |
| **Account Name Management**     | Manually specified in OMA-URI.                                                    | Can be automated with options for name randomization.                             |
| **Applicability**               | Works on all production-level Windows 10 and 11 machines.                         | Currently only available on Windows 11 Insider Preivew Builds (Canary Channel).      |
| **Admin Rights Assignment**     | Optional, by specifying `LocalUserGroup` OMA-URI with a value of `2`.             | Automatic, based on LAPS CSP settings.                                            |
| **Outcome**                     | Creates a custom local user account with optional admin rights.                   | Creates and manages a custom local admin account automatically.                   |

This table should make it easier to cross-reference and understand the key differences between the two approaches.






Here are examples for each approach:

### 2020 Approach Example

This example shows how to manually create a local admin account using the Accounts CSP.

**Scenario:** Create a local user account named `LocalUser` and assign it as an administrator on a Windows 10 machine.

1.  **Create Configuration Profile in Intune:**
    
    -   **Platform:** Windows 10 and later
    -   **Profile Type:** Custom
2.  **OMA-URI for Creating the Local User:**
    
    -   **Name:** `RestrictedGroups – ConfigureGroupMembership`
    -   **OMA-URI:** `./Device/Vendor/MSFT/Accounts/Users/LocalUser/Password`
    -   **Data Type:** String
    -   **Value:** `YourPassword123`
3.  **OMA-URI for Assigning Admin Rights:**
    
    -   **Name:** `RestrictedGroups – ConfigureGroupMembership`
    -   **OMA-URI:** `./Device/Vendor/MSFT/Accounts/Users/LocalUser/LocalUserGroup`
    -   **Data Type:** Integer
    -   **Value:** `2` (This assigns the user to the Administrators group)

**Outcome:** A new local user account named `LocalUser` is created on the device with the password `YourPassword123`, and the account is added to the local Administrators group.

### 2024 Approach Example (Windows 11 Insider Preivew)

This example shows how to automatically create and manage a local admin account using LAPS CSP.

**Scenario:** Automatically create a local admin account with the prefix `CorpAdmin-`, randomize the name, and enforce password complexity.

1.  **Create Configuration Profile in Intune:**
    
    -   **Platform:** Windows 10 and later
    -   **Profile Type:** Custom
2.  **OMA-URI Settings:**
    
    -   **Enable Automatic Account Creation:**
        
        -   **OMA-URI:** `./Device/Vendor/MSFT/LAPS/Policies/AutomaticAccountManagementEnableAccount`
        -   **Data Type:** Bool
        -   **Value:** `True`
    -   **Enable Account Management:**
        
        -   **OMA-URI:** `./Device/Vendor/MSFT/LAPS/Policies/AutomaticAccountManagementEnabled`
        -   **Data Type:** Bool
        -   **Value:** `True`
    -   **Specify Account Name Prefix:**
        
        -   **OMA-URI:** `./Device/Vendor/MSFT/LAPS/Policies/AutomaticAccountManagementNameOrPrefix`
        -   **Data Type:** String
        -   **Value:** `CorpAdmin-`
    -   **Randomize Account Name:**
        
        -   **OMA-URI:** `./Device/Vendor/MSFT/LAPS/Policies/AutomaticAccountManagementRandomizeName`
        -   **Data Type:** Bool
        -   **Value:** `True`
    -   **Password Complexity:**
        
        -   **OMA-URI:** `./Device/Vendor/MSFT/LAPS/Policies/PasswordComplexity`
        -   **Data Type:** Int
        -   **Value:** `3` (Assuming 3 corresponds to a desired level of complexity, such as a strong password)

**Outcome:** A new local administrator account is automatically created with a name that starts with `CorpAdmin-` and is randomized. The account's password adheres to the specified complexity requirements, and LAPS manages the account, including password updates.






Here's a table that presents the two examples side by side:

| **Aspect**                        | **2020 Approach Example**                                               | **2024 Approach Example (Windows 11 Insider Preivew)**                                |
|-----------------------------------|-------------------------------------------------------------------------|----------------------------------------------------------------------------|
| **Scenario**                      | Create a local user account named `LocalUser` with admin rights.        | Automatically create a local admin account with prefix `CorpAdmin-`, randomize the name, and enforce password complexity. |
| **Platform**                      | Windows 10 and later                                                    | Windows 10 and later (Windows 11 Insider Preivew)                                     |
| **Profile Type**                  | Custom                                                                  | Custom                                                                     |
| **OMA-URI for Account Creation**  | `./Device/Vendor/MSFT/Accounts/Users/LocalUser/Password`                | `./Device/Vendor/MSFT/LAPS/Policies/AutomaticAccountManagementEnableAccount` |
| **Data Type**                     | String                                                                  | Bool                                                                       |
| **Value**                         | `YourPassword123`                                                       | `True`                                                                     |
| **OMA-URI for Admin Rights**      | `./Device/Vendor/MSFT/Accounts/Users/LocalUser/LocalUserGroup`          | `./Device/Vendor/MSFT/LAPS/Policies/AutomaticAccountManagementEnabled`      |
| **Data Type**                     | Integer                                                                 | Bool                                                                       |
| **Value**                         | `2` (Assigns to Administrators group)                                   | `True`                                                                     |
| **Additional Settings**           | None                                                                    | - **Account Name Prefix:** `CorpAdmin-`<br>- **Randomize Name:** `True`<br>- **Password Complexity:** `3` |
| **Outcome**                       | - Creates a local user account named `LocalUser`.<br>- Account is added to the Administrators group. | - Automatically creates a local admin account with a randomized name starting with `CorpAdmin-`.<br>- Enforces password complexity and manages the account using LAPS. |

This table should help you quickly compare and contrast the two examples.








### 3. Detailed Implementation Guide

#### 2020 Approach

##### Step-by-Step Configuration in Intune

1.  **Sign in to the Endpoint Manager Admin Center:**
    
    -   Navigate to **Devices > Configuration profiles** and click **+ Create profile**.
2.  **Choose Platform and Profile Type:**
    
    -   **Platform:** Windows 10 and later.
    -   **Profile Type:** Custom.
3.  **Create the Configuration Profile:**
    
    -   Provide a **Name** for the profile (e.g., "Local Admin Account Creation").
    -   Optionally, add a **Description** to explain the purpose of the profile.
4.  **Add the OMA-URI Settings:**
    
    -   **Number of OMA-URI Settings Required:** 2
        
    -   **OMA-URI for Creating the Local Admin Account:**
        
        -   **Name:** `Create Local Admin Account`
        -   **OMA-URI:** `./Device/Vendor/MSFT/Accounts/Users/LocalAdmin/Password`
        -   **Data Type:** `String`
        -   **Value:** `YourSecurePassword123`
        -   **Description:** This setting creates a new local administrator account with the username `LocalAdmin` and sets the specified password.
    -   **OMA-URI for Assigning Admin Rights:**
        
        -   **Name:** `Assign Admin Rights to Local Admin Account`
        -   **OMA-URI:** `./Device/Vendor/MSFT/Accounts/Users/LocalAdmin/LocalUserGroup`
        -   **Data Type:** `Integer`
        -   **Value:** `2` (This assigns the user to the Administrators group)
        -   **Description:** This setting adds the `LocalAdmin` account to the local Administrators group, granting it full administrative privileges on the device.
5.  **Assign the Profile:**
    
    -   Assign the profile to **Device Groups** where you want the local admin account to be created.
6.  **Monitor Deployment:**
    
    -   Monitor the deployment in the **Intune Admin Center** to ensure the profile is applied successfully.

##### Assigning to Devices

-   Ensure the policy is assigned to the appropriate **Device Groups** in Intune.
-   This approach is designed to apply to all targeted devices, creating a local administrator account with the specified settings.

#### 2024 Approach Windows 11 (Insider Canary)

##### Step-by-Step Configuration in Intune

1.  **Sign in to the Endpoint Manager Admin Center:**
    
    -   Navigate to **Devices > Configuration profiles** and click **+ Create profile**.
2.  **Choose Platform and Profile Type:**
    
    -   **Platform:** Windows 10 and later.
    -   **Profile Type:** Custom.
3.  **Create the Configuration Profile:**
    
    -   Provide a **Name** for the profile (e.g., "Automated Local Admin Account Creation").
    -   Optionally, add a **Description** to explain the purpose of the profile.
4.  **Add the OMA-URI Settings:**
    
    -   **Number of OMA-URI Settings Required:** 3
        
    -   **OMA-URI for Enabling Account Management:**
        
        -   **Name:** `Enable LAPS Account Management`
        -   **OMA-URI:** `./Device/Vendor/MSFT/LAPS/Policies/AutomaticAccountManagementEnableAccount`
        -   **Data Type:** `Bool`
        -   **Value:** `True`
        -   **Description:** This setting enables the automatic management of a local administrator account by LAPS on the device.
    -   **OMA-URI for Setting the Account Name or Prefix:**
        
        -   **Name:** `Set LAPS Account Name Prefix`
        -   **OMA-URI:** `./Device/Vendor/MSFT/LAPS/Policies/AutomaticAccountManagementNameOrPrefix`
        -   **Data Type:** `String`
        -   **Value:** `AdminPrefix-`
        -   **Description:** This setting specifies a prefix for the LAPS-managed local administrator account name. The actual account name will start with this prefix, and additional characters may be randomized.
    -   **OMA-URI for Managing Password Complexity:**
        
        -   **Name:** `Set LAPS Password Complexity`
        -   **OMA-URI:** `./Device/Vendor/MSFT/LAPS/Policies/PasswordComplexity`
        -   **Data Type:** `Int`
        -   **Value:** `4` (Choose the complexity level according to your organization's security policies)
        -   **Description:** This setting defines the password complexity requirements for the LAPS-managed local administrator account. A higher value corresponds to more complex passwords.
5.  **Assign the Profile:**
    
    -   Assign the profile to **Device Groups** where you want the LAPS-managed local admin account to be created.
6.  **Disclaimer:** This approach is currently available only in the Windows 11 Insider Canary Channel and is not recommended for production environments. Monitor the deployment and account management through the **Intune Admin Center**.
    

##### Assigning to Devices

-   Similar to the 2020 approach, assign the policy to the appropriate **Device Groups** in Intune.
-   This approach automates the creation and management of local admin accounts on the targeted devices.

### 4. Best Practices

#### Naming Conventions for Policies

When managing multiple Intune policies, especially those related to local administrator accounts, it's crucial to use a consistent and descriptive naming convention. This practice helps in organizing policies, making them easier to identify and manage over time.

**Recommended Naming Convention:**

-   **Format:** `PolicyType_SequenceNumber_Description_TargetPlatform`
-   **Example:** `CSP_01_LocalAdmin_AccountCreation_Windows10-11`

**Explanation:**

-   **PolicyType:** Indicates the type of policy, such as `CSP` for Configuration Service Provider policies.
-   **SequenceNumber:** A numerical value to help organize and sequence related policies.
-   **Description:** A brief description of the policy’s purpose, such as `LocalAdmin_AccountCreation`.
-   **TargetPlatform:** Specifies the target operating system, e.g., `Windows10-11`.

#### Security Considerations

Managing local administrator accounts is a critical security task. Here are some best practices to ensure secure management:

1.  **Unique Passwords:** Ensure each local admin account has a unique initial password across devices.
    
    -   In the **2020 approach**, the initial password must be manually set by the administrator during configuration.
    -   In the **2024 approach**, LAPS automates the creation of the initial password.
2.  **Password Complexity:** Implement strong password complexity requirements to prevent brute-force attacks. This is particularly important in the 2024 approach, where LAPS manages the initial password.
    
3.  **Ongoing Password Management:**
    
    -   While the **2020 approach** requires manual entry of the initial password, and no further automation is provided, the **2024 approach** automates the initial password.
    -   However, ongoing password management is handled by the LAPS policy under Account Protection in Endpoint Security. The password can then be viewed in the Entra or Intune portal for the specific device.
4.  **Limit Scope:** Avoid using broad administrative roles such as Global Admin or Device Admin in Entra ID unless absolutely necessary. Local admin accounts should be scoped as narrowly as possible to reduce the attack surface.
    
5.  **Disable Unnecessary Accounts:** Disable the built-in `.\administrator` account if it’s not needed, as it has a known SID that attackers can target.
    

#### Scope Limitation with RBAC Roles

Role-Based Access Control (RBAC) in Entra ID allows for granular control over administrative access. However, it’s important to limit the scope of RBAC roles to minimize security risks:

-   **Avoid Over-Privileged Roles:** Use least privilege principles to assign only the necessary permissions to each role.
-   **Custom Roles:** Consider creating custom roles with specific, limited permissions that match the needs of your environment, rather than using broad roles like Global Admin.
-   **Local Admins:** Use local administrator accounts managed by LAPS for tasks that don’t require cloud-level administrative access, keeping Global Admin and Device Admin roles reserved for situations where they are truly necessary.



Here's the breakdown of the description into its key components:

### 1. **Author Information:**
   - **"Created by: Author Name"**
     - This identifies the author of the policy, ensuring that anyone reviewing the policy later knows who was responsible for its creation.

### 2. **Creation Date:**
   - **"Created on: Sept 2, 2024"**
     - This provides the date the policy was created, which is important for tracking changes, updates, and the relevance of the policy over time.

### 3. **Purpose of the Policy:**
   - **"Created for: Creating a local administrator account named 'LocalUser' on Intune-managed Windows 10 and Windows 11 devices."**
     - This section clearly states the primary purpose of the policy: to create the `LocalUser` local admin account on devices managed through Intune.

### 4. **Security Context:**
   - **"This account is intended to serve as an alternative to the built-in .\administrator, which is disabled by default due to its known SID that can be exploited by attackers."**
     - This explains why the policy is necessary, emphasizing the security risks associated with the built-in administrator account and the importance of creating a secure alternative.

### 5. **Usage in Safe Mode:**
   - **"The 'LocalUser' account can be used during safe mode from WinRE, providing a secure method for administrative access."**
     - This details the specific use case for the `LocalUser` account, highlighting its utility in scenarios like safe mode or WinRE (Windows Recovery Environment).

### 6. **Scope Limitation:**
   - **"Without relying on the Global Admin or Device Admin RBAC roles in Entra, which have a broad scope."**
     - This part explains that the policy helps avoid using broader administrative roles (Global Admin or Device Admin) in Entra, which are less secure due to their wide-reaching permissions.

### Summary:
- **The Description:** is comprehensive and covers who created the policy, when it was created, and the specific purpose and security rationale behind it. 
- **Security & Administrative Considerations:** It emphasizes the importance of avoiding reliance on more privileged roles like Global Admin, thus narrowing the scope of administrative access to reduce potential security risks.

This breakdown should give you a clear understanding of each component of the description and how it contributes to a thorough and well-documented policy.



Simpler Version
---

**Created by:** Author Name  
**Created on:** Sept 2, 2024  

**Created for:**  
**Purpose:**  
- Creating a local administrator account named **'LocalUser'** on Intune-managed Windows 10 and Windows 11 devices.

**Security Considerations:**  
- This account serves as an alternative to the built-in **.\administrator**, which is disabled by default due to its known SID that can be exploited by attackers.

**Usage:**  
- Both the built-in **.\administrator** and **'LocalUser'** accounts can be used during safe mode from **WinRE** (Windows Recovery Environment), providing secure methods for administrative access.

**Scope Limitation:**  
- Avoids using the **Global Admin** or **Device Admin** RBAC roles in **Entra**, as these roles have a broad scope.

---




### 5. FAQs

#### **Q1: Can I specify the exact account name in the 2024 approach?**

**A:** Yes, in the 2024 approach, you can specify the exact account name by using the `AutomaticAccountManagementNameOrPrefix` OMA-URI. If you do not enable name randomization (`AutomaticAccountManagementRandomizeName` set to `False`), the account will be created with the exact name you specify.

#### **Q2: How does the LAPS policy manage and auto-rotate passwords?**

**A:** The 2020 approach does not automate password management; the initial password is manually set by the administrator. In contrast, the 2024 approach automates the creation of the initial password, but ongoing password management and rotation are handled by the LAPS policy under Account Protection in Endpoint Security. The passwords can be viewed and managed through the Entra or Intune portal for each specific device.

#### **Q3: Does this approach work on both Windows 10 and Windows 11?**

**A:** The 2020 approach works on both Windows 10 and Windows 11. However, the 2024 approach is currently limited to Windows 11 devices running the Windows 11 Insider Canary Channel.

#### **Q4: Should I assign the policy to users or devices?**

**A:** The policies should be assigned to **devices** rather than users. This is because the policies are intended to create and manage local administrator accounts on the devices themselves, independent of who is logged in.

#### **Q5: What is the difference between prefixing and randomizing account names in the 2024 approach?**

**A:**

-   **Prefixing:** When you set a prefix using the `AutomaticAccountManagementNameOrPrefix` OMA-URI, the LAPS policy will create an account name that starts with this prefix. You can choose to use a static prefix or allow for additional randomization.
-   **Randomizing:** If name randomization is enabled (`AutomaticAccountManagementRandomizeName` set to `True`), LAPS will add a randomized string after the specified prefix, creating a unique account name on each device.




### 6. Credits and References

This section is reserved for acknowledging the original authors and sources that contributed to the approaches discussed in this article. It’s important to credit the individuals and resources that provided the foundational knowledge and methods you have implemented.

#### 2020 Approach

-   **Original Article:** https://inthecloud247.com/create-a-local-user-account-on-windows-10-with-microsoft-intune/
-   **Author:** Peter Klapwijk

#### 2024 Approach Windows 11 (Insider Canary)

-   **Original Article:** https://ourcloudnetwork.com/how-to-enable-automatic-account-creation-with-laps-in-intune/
-   **Author:** Daniel Bradley

_Note: Replace the placeholders with the actual URLs and author names to properly credit the sources._

### 7. Conclusion

Both the 2020 and 2024 approaches to managing local administrator accounts on Windows devices provide valuable methods for ensuring secure and effective account management.

-   **2020 Approach:** Suitable for production environments across Windows 10 and Windows 11, this method involves manual setup and configuration, giving administrators full control over the account details.
-   **2024 Approach:** An automated method available exclusively for Windows 11 Canary builds, this approach simplifies the creation and management of local admin accounts, including initial password automation. However, ongoing password management and rotation must be handled through LAPS policies under Account Protection in Endpoint Security.

**Recommendations:**

-   For production environments, continue using the 2020 approach or wait for broader release support if considering the 2024 method.
-   Ensure that all local admin accounts are properly managed with unique, complex passwords and that ongoing password management is in place through LAPS policies.

By following the best practices and guidelines provided, you can effectively manage local administrator accounts in your organization, enhancing both security and operational efficiency.


**Section Headers in the Document:**



